## Rozwiązania – 13 Bio: geny i białka

```python
import requests, numpy as np
from sentence_transformers import SentenceTransformer
from dataclasses import dataclass
from typing import List, Optional

@dataclass
class CrossRef:
    source: str
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

def fetch_uniprot_gapdh(n=10):
    r = requests.get("https://rest.uniprot.org/uniprotkb/search", params={"query":"gapdh","format":"json","size":n}, timeout=30)
    r.raise_for_status()
    return r.json().get("results", [])

def uniprot_to_protein(entry: dict) -> Protein:
    pri = entry.get("primaryAccession")
    org = entry.get("organism", {}).get("scientificName")
    name = (entry.get("proteinDescription", {})
            .get("recommendedName", {})
            .get("fullName", {})
            .get("value"))
    return Protein(uniprot_id=pri, refseq_id=None, name=name, sequence_fasta=None, functions=[], organisms=org, xrefs=[CrossRef("UniProt", pri)])

def fetch_fasta(ref: str) -> str:
    return requests.get(f"https://www.ncbi.nlm.nih.gov/protein/{ref}?report=fasta", timeout=30).text

entries = fetch_uniprot_gapdh(10)
proteins = [uniprot_to_protein(e) for e in entries]

model = SentenceTransformer("sentence-transformers/all-MiniLM-L6-v2")
docs = [" "+" ".join(filter(None,[p.name, p.organisms])) for p in proteins]
emb = model.encode(docs, normalize_embeddings=True)

def top_k(query: str, k=5):
    q = model.encode([query], normalize_embeddings=True)[0]
    scores = emb @ q
    idx = np.argsort(-scores)[:k]
    return [(proteins[i].uniprot_id, docs[i], float(scores[i])) for i in idx]

print(top_k("human glycolysis enzyme"))

# Hybrydowy scorer (przykład szkicu – BM25 wymagane osobno)
def hybrid_score(embed_score: float, bm25_score: float, alpha: float = 0.7) -> float:
    return alpha * embed_score + (1 - alpha) * bm25_score
```
