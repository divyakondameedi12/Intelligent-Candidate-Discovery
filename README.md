# 🔍 Intelligent Candidate Discovery
**Redrob Data & AI Challenge 2026**

[![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python)](https://python.org)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)
[![Challenge](https://img.shields.io/badge/Redrob-Data%20%26%20AI%20Challenge-orange)](https://redrob.in)

> A multi-signal AI pipeline that reads a Job Description and a pool of candidate profiles, then returns a **ranked, colour-coded, fully explained Excel file** — identifying the best-fit candidates without pure keyword matching.

**Author:** Divya Kondameedi — [github.com/divyakondameedi12](https://github.com/divyakondameedi12)

---

## ✨ What It Does

Most ATS tools rank candidates by counting keyword overlaps. This pipeline goes further:

| Signal | Weight | How |
|---|---|---|
| 🔧 Skills Match | **35%** | Extracts required skills from JD → scans all candidate text fields |
| ⏱ Experience | **25%** | Regex-parses numeric years from JD → tiered comparison |
| 💬 Semantic Similarity | **20%** | TF-IDF cosine between full JD and full profile (CPU-only) |
| 🎓 Education | **10%** | Degree-tier keyword matching (PhD → Master → Bachelor) |
| 📋 Profile Quality | **10%** | Penalises sparse, inconsistent, or inflated profiles |

Every candidate gets a **0–100 final score**, a **tier label** (Strong Fit / Potential Fit / Weak Fit / Not Recommended), and a **plain-English explanation** of why.

---

## 🚀 Quick Start

```bash
git clone https://github.com/divyakondameedi12/redrob-candidate-discovery
cd redrob-candidate-discovery
pip install -r requirements.txt
```

**Place the challenge dataset in `/data/`:**
```
data/
├── job_description.txt     ← paste JD text here
└── candidates.json         ← candidate pool (or candidates.csv)
```

**Run the pipeline:**
```bash
python main.py
```

Output is written to `output/ranked_candidates.xlsx` — colour-coded and ready to submit.

> **No dataset yet?** Running `python main.py` without the `/data/` files automatically uses built-in sample data (10 realistic candidates for a Senior Data Scientist role) so you can see the output immediately.

---

## 📁 Project Structure

```
redrob-candidate-discovery/
│
├── main.py                  # Entry point — loads data, runs pipeline, saves Excel
│
├── src/
│   └── ranker.py            # Core logic
│                            #   JDParser       — extracts structured requirements from raw JD
│                            #   CandidateScorer — 5-signal weighted scorer
│                            #   CandidateDiscoveryPipeline — orchestrates end-to-end
│
├── data/                    # Place challenge dataset files here (gitignored)
│
├── output/                  # ranked_candidates.xlsx written here
│
├── requirements.txt         # pandas, scikit-learn, openpyxl
└── README.md
```

---

## 📊 Output Format

The Excel file `ranked_candidates.xlsx` includes one row per candidate:

| Column | Description |
|---|---|
| `rank` | 1 = best fit |
| `candidate_id` | From source data |
| `name` | Candidate name |
| `final_score` | Weighted score 0–100 |
| `recommendation` | Strong Fit / Potential Fit / Weak Fit / Not Recommended |
| `skills_match` | Component score 0–100 |
| `experience` | Component score 0–100 |
| `semantic_similarity` | Component score 0–100 |
| `education` | Component score 0–100 |
| `profile_quality` | Component score 0–100 |
| `explanation` | Plain-English reasoning for the score |

Rows are **colour-coded** by recommendation tier (green → amber → orange → red) for fast visual scanning.

---

## 💡 Explainability

Every score is traceable. Example explanation for Rank 1:

```
Matched 11/14 required skills | 6 yrs experience (exceeds 4+ req) |
Education: M.Tech IIT Madras | Clean profile | Semantic alignment: 0.71
```

No black box. Every number maps to a specific field in the source data.

---

## 🛡 Hallucination Prevention

- Scores are computed **only from candidate profile text** — the model never invents attributes
- Profile Quality signal actively **penalises suspicious profiles**: sparse summary (−30 pts), no skills listed (−20 pts), high experience with thin profile (−20 pts)
- All component scores are bounded to **[0, 1]** before aggregation
- No generative AI in the ranking output path — fully deterministic

---

## ⚙️ Supported Input Formats

| File | Format |
|---|---|
| `data/job_description.txt` | Plain text JD |
| `data/candidates.json` | Array of objects, or `{"candidates": [...]}` |
| `data/candidates.csv` | One row per candidate |

Required candidate fields: `name`, `skills`, `years_of_experience`, `education`, `profile_summary`
Optional: `candidate_id`, `experience_details`

---

## 🔧 Requirements

```
Python  3.10+
pandas  >= 2.0
scikit-learn >= 1.3
openpyxl >= 3.1
```

Install: `pip install -r requirements.txt`

No GPU required. Runtime under 30 seconds for 1,000 candidates on a standard laptop.

---

## 🔮 Extending the Pipeline

The scorer is designed to be swapped out. To upgrade from TF-IDF to SBERT:

```python
# In src/ranker.py, replace _tfidf_cosine with:
from sentence_transformers import SentenceTransformer, util
model = SentenceTransformer("all-MiniLM-L6-v2")

def _semantic_score(self, candidate, jd):
    emb_jd   = model.encode(jd["raw_text"], convert_to_tensor=True)
    emb_cand = model.encode(cand_text,      convert_to_tensor=True)
    sim = float(util.cos_sim(emb_jd, emb_cand))
    return sim, f"SBERT similarity: {sim:.2f}"
```

All other pipeline code stays identical.

---

## 📄 License

MIT — free to use, extend, and build on.
