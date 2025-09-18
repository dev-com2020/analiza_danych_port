## AI/ML: Pozyskiwanie informacji o genach i białkach (NCBI/UniProt)

Cel: Zaprojektować i zademonstrować przepływ danych, który pobiera informacje o genach i kodowanych przez nie białkach z NCBI i UniProt, normalizuje je do wspólnego modelu i udostępnia warstwę zapytań wspieraną przez wektorowe wyszukiwanie (semantic search) i klasyczne filtry.

Źródła przykładowe:
- NCBI Datasets API: `https://www.ncbi.nlm.nih.gov/datasets/docs/v2/api/languages/`
- NCBI Protein FASTA (przykład): `https://www.ncbi.nlm.nih.gov/protein/NP_001316607.1?report=fasta`
- UniProt (gapdh): `https://www.uniprot.org/uniprotkb?query=gapdh`

### Architektura rozwiązania
1) Ingest: pobieranie danych z NCBI/UniProt (REST/HTTP), ratelimiting, retry, cache.
2) Normalizacja: wspólny schemat `Gene`, `Transcript`, `Protein`, `CrossRef` (ID mapping: Entrez, RefSeq, UniProt, Ensembl).
3) Przetwarzanie NLP: embedding opisów/sekcji funkcjonalnych; wektorowy indeks (FAISS/Annoy/pgvector).
4) Funkcje ML: klasyfikacja funkcji białka (zero-shot/LLM labeling), podobieństwo sekwencji (MinHash/k-mer, opcjonalnie ESM embeddingi), rankowanie dokumentów.
5) API zapytań: hybrydowe (filtry metadanych + top-k wektorowo) i generator raportów.

### Minimalny model danych (schemat)
```python
from dataclasses import dataclass
from typing import List, Optional, Dict

@dataclass
class CrossRef:
    source: str  # "Entrez", "RefSeq", "UniProt", "Ensembl"
    accession: str

@dataclass
class Protein:
    uniprot_id: Optional[str]
    refseq_id: Optional[str]
    name: Optional[str]
    sequence_fasta: Optional[str]
    functions: List[str]
    organisms: Optional[str]
    xrefs: List[CrossRef]

@dataclass
class Gene:
    symbol: str
    entrez_id: Optional[str]
    description: Optional[str]
    organism: Optional[str]
    proteins: List[Protein]
    xrefs: List[CrossRef]
```

### Ingest NCBI/UniProt – przykładowe pobranie
Wymagania: `pip install requests pandas polars beautifulsoup4 lxml sentence-transformers faiss-cpu`
```python
import requests

# NCBI Protein FASTA przykład
fasta = requests.get("https://www.ncbi.nlm.nih.gov/protein/NP_001316607.1?report=fasta", timeout=30).text
print(fasta.splitlines()[:3])

# UniProt REST (search API)
u = requests.get(
    "https://rest.uniprot.org/uniprotkb/search",
    params={"query": "gapdh", "format": "json", "size": 5},
    timeout=30,
).json()
entries = u.get("results", [])
```

### Normalizacja i łączenie rekordów
```python
def uniprot_to_protein(entry: dict) -> Protein:
    pri = entry.get("primaryAccession")
    org = entry.get("organism", {}).get("scientificName")
    name = (entry.get("proteinDescription", {})
            .get("recommendedName", {})
            .get("fullName", {})
            .get("value"))
    xrefs = [CrossRef(source="UniProt", accession=pri)]
    return Protein(uniprot_id=pri, refseq_id=None, name=name, sequence_fasta=None, functions=[], organisms=org, xrefs=xrefs)

proteins = [uniprot_to_protein(e) for e in entries]
gene = Gene(symbol="GAPDH", entrez_id=None, description="Glyceraldehyde-3-phosphate dehydrogenase", organism=proteins[0].organisms if proteins else None, proteins=proteins, xrefs=[])
```

### Embedding opisów i wyszukiwanie wektorowe
```python
from sentence_transformers import SentenceTransformer
import numpy as np

model = SentenceTransformer("sentence-transformers/all-MiniLM-L6-v2")

docs = []
for p in proteins:
    text = " ".join(filter(None, [p.name, p.organisms]))
    docs.append(text)

emb = model.encode(docs, normalize_embeddings=True)

# prosty top-k kosinusowy
def top_k(query: str, k: int = 3):
    qv = model.encode([query], normalize_embeddings=True)[0]
    scores = np.dot(emb, qv)
    idx = np.argsort(-scores)[:k]
    return [(docs[i], float(scores[i])) for i in idx]

print(top_k("glycolysis enzyme human"))
```

### Rozszerzenia ML (opcjonalnie)
- Funkcyjne etykietowanie opisów: zero-shot classification (np. `facebook/bart-large-mnli`) z etykietami GO/KEGG.
- Podobieństwo sekwencji: embeddingi ESM (`facebook/esm2_t6_8M_UR50D`) dla FASTA i wektorowe porównania.
- Rankowanie hybrydowe: BM25 (metadane) + embedding scoring (re-ranking).

### Warstwa zapytań (schemat REST)
```python
# /search?text=... -> zwraca top-k geny/białka
# /gene/{symbol} -> szczegóły genu i powiązanych białek
# /protein/{accession} -> szczegóły białka (sekwencja, funkcje, xrefs)
# /similar?accession=... -> podobne białka wg embeddingów lub k-mer
```

### Ćwiczenia
- Pobierz 10 rekordów z UniProt dla zapytania "gapdh", znormalizuj do klasy `Protein` i zbuduj wektorowy indeks.
- Pobierz FASTA z NCBI dla 3 akcesji, dodaj do indeksu i przetestuj zapytania semantyczne (top-k).
- Zaprojektuj prosty scorer hybrydowy: 0.7*similarity + 0.3*BM25 na polu opisu.
