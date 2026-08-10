# AI Research Assistant

**Scientific Paper Recommendation using LLM-Extracted Sections and Citation Context**

MSc dissertation, School of Computing, Newcastle University.

Ajay Adarsh Sivakumar &nbsp;·&nbsp; Supervisor: Dr Huizhi Liang &nbsp;·&nbsp; 2025–2026

---

## Overview

Scientific paper recommenders typically encode a paper from its title and
abstract, discarding both the methodological detail in its body and the
independent characterisation supplied by the literature that cites it. This
project encodes both.

**Section content** is extracted and summarised in a single pass by
Qwen2.5-7B-Instruct reading raw PDF text, raising section coverage from
52–83% under rule-based parsing to 93–95%. The largest gain is on results
sections, where table-dense formatting defeats heading-anchored extraction.

**Citation context** is retrieved from Semantic Scholar. Raw fragments embed
generically, placing unrelated papers at 0.86–0.93 cosine similarity, so they
are reconstructed by the same model into coherent prose before encoding.

Both are encoded with SPECTER2 and fused at α = 0.90, γ = 0.10.

### Results

Evaluated over 9,999 queries against BGE-M3 relevance judgements, chosen to be
independent of the retrieval encoder and so avoid circularity.

| Method | MRR@10 | Hit@10 | NDCG@10 |
|:---|---:|---:|---:|
| BM25 | 0.5836 | 0.8403 | 0.2575 |
| Abstract-only (SPECTER2) | 0.5910 | 0.8690 | 0.2688 |
| Section-aware (regex) | 0.5920 | 0.8680 | 0.2696 |
| Section-aware (LLM) | 0.6021 | 0.8741 | 0.2775 |
| Three-way fusion (regex) | 0.6120 | **0.8827** | 0.2832 |
| **Sec(LLM) + citation** | **0.6165** | 0.8821 | **0.2856** |

A qualitative LLM-as-judge study over 30 queries prefers the proposed method
on relevance, usefulness and diversity against every baseline, winning
diversity 24–0 against abstract-only.

### Central finding

Five architectural extensions, each with an established motivation in prior
work, yielded no further improvement. **Representation quality, rather than
the sophistication of the mechanism combining representations, is the
operative constraint.**

| Extension | MRR@10 | Diagnosed cause |
|:---|---:|:---|
| Temporal decay | 0.5517 | two-year span insufficient |
| Contrastive alignment | 0.6027 | 8,770 pairs, overfits |
| Author-graph fusion | 0.4752 | proximity–relevance mismatch |
| Diversity re-ranking | −0.3% | no redundancy to correct |
| Claim-level MaxSim | 0.3141 | representation collapse |

A sixth direction, inductive graph representation learning, was assessed but
not implemented: a reachability analysis found fewer than 2% of relevant
papers reachable through in-corpus graph edges, bounding any structural
method independently of architecture.

---

## Repository structure

```
notebooks/
  00_master_restore.ipynb        session restore, shared helpers
  01_corpus_construction.ipynb   arXiv fetch, PDF download, regex baseline
  02_section_extraction.ipynb    LLM extraction and SPECTER2 encoding
  03_citation_context.ipynb      Semantic Scholar fetch, reconstruction
  04_ground_truth.ipynb          three judgement strategies, one adopted
  05_evaluation.ipynb            full ablation, LLM-as-judge
  06_extensions.ipynb            the five extensions and viability analysis
  07_demo.ipynb                  interactive query interface

data/
  ground_truth.json                       9,999 BGE-M3 judgements
  paper_claims.json                       21,691 claims over 9,526 papers
  paper_community.json                    Louvain community per paper
  citation_contexts_reconstructed.json    LLM-reconstructed descriptions
  results/                                all evaluation output

figures/                          dissertation figures and plotting code
dissertation/                     LaTeX source, bibliography, compiled PDF
requirements.txt
```
 
## How to Run

Run the notebooks in the below order. `00_master_restore.ipynb` must be run first after
any session restart, as it defines the helpers the others depend on.

  00_master_restore.ipynb        
  01_corpus_construction.ipynb   
  02_section_extraction.ipynb    
  03_citation_context.ipynb      
  04_ground_truth.ipynb          
  05_evaluation.ipynb            
  06_extensions.ipynb            
  07_demo.ipynb                

---

## Data availability

Small artifacts are committed under `data/`. Large artifacts exceed GitHub's
100 MB per-file limit and are hosted separately.

| Artifact | Size | Location |
|:---|---:|:---|
| Judgements, claims, communities, results | < 5 MB | `data/` |
| SPECTER2 and BGE-M3 embeddings (`.npy`) | 29–63 MB each | *[Drive link]* |
| Raw citation contexts (13,747 records) | 127 MB | *[Drive link]* |
| Corpus metadata with LLM sections | 25 MB | *[Drive link]* |
| Source PDFs (10,453 files) | ~30 GB | not distributed |

**PDFs are not redistributed.** They remain under their original arXiv
licences. `01_corpus_construction.ipynb` downloads them given the identifiers
in the corpus metadata.

---

## Reproduction

Developed on Google Colab with an A100 (80 GB) and Google Drive for persistent
storage. Every long-running stage is checkpointed and resumes automatically
after interruption.

```bash
pip install -r requirements.txt
```

| Purpose | Model |
|:---|:---|
| Section extraction, citation reconstruction, judging | `Qwen/Qwen2.5-7B-Instruct` |
| Document encoding | `allenai/specter2_base` |
| Relevance judgements | `BAAI/bge-m3` |

**To reproduce the headline result** without re-running LLM inference,
download the embedding archive from the Drive link above and run
`05_evaluation.ipynb`. Minutes rather than hours.

**To reproduce from scratch**, run notebooks 01 through 05 in order. Roughly
20 GPU-hours in total, dominated by LLM inference: about 14 hours for section
extraction and 3 for citation reconstruction. The Semantic Scholar fetch adds
around 4 hours of wall-clock time at 1 request per second.

`03_citation_context.ipynb` requires a Semantic Scholar API key. Set
`S2_API_KEY` at the top of the notebook.

---

## Notes

The regex extraction in notebook 01 and the abandoned ground-truth strategies
in notebook 04 are retained deliberately. Both produce figures reported in the
dissertation, the regex coverage baseline and the 1.3% in-corpus citation
rate respectively, and neither claim is verifiable without them.

Artifacts from abandoned approaches are suffixed `_ABANDONED` so they cannot
be loaded by mistake. The judgements used throughout are `ground_truth.json`.

---

## Licence

Code released under the MIT Licence. Corpus metadata derives from the arXiv
API and the Semantic Scholar Graph API, subject to their respective terms.
