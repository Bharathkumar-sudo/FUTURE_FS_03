# Resume / Candidate Screening System

A lightweight Python pipeline that screens resumes against a job description,
ranks candidates by fit, and reports missing (gap) skills — built with
spaCy/NLTK for text handling and scikit-learn for similarity scoring.

## Features

1. **Resume text cleaning & parsing** — strips URLs, emails, phone numbers,
   and punctuation; lowercases and normalizes whitespace.
2. **Skill extraction & matching** — matches resume/job-description text
   against a configurable skill taxonomy (supports multi-word skills like
   "machine learning").
3. **Candidate ranking** — combines TF-IDF cosine similarity with skill-match
   percentage into a blended `final_score`.
4. **Skill gap identification** — lists which required skills each candidate
   is missing.

## Requirements

```
pandas
scikit-learn
```

Optional (used automatically if installed, otherwise falls back to built-in
alternatives so the script still runs standalone):

```
spacy            # + en_core_web_sm model
nltk             # + stopwords corpus
```

Install:

```bash
pip install pandas scikit-learn
# optional extras
pip install spacy nltk
python -m spacy download en_core_web_sm
```

## Usage

### Run the built-in demo

```bash
python resume_screening_system.py
```

This runs a sample job description against three example resumes and prints:
- Skills detected in the job description
- A ranked candidate table
- A per-candidate skill gap report

### Use it in your own code

```python
from resume_screening_system import ResumeScreeningSystem

system = ResumeScreeningSystem()  # or pass a custom skill_taxonomy=[...]

job_description = "We need a Python developer skilled in SQL and AWS..."

resumes = {
    "Candidate A": "Full resume text here...",
    "Candidate B": "Full resume text here...",
}

results = system.rank_candidates(resumes, job_description)
print(results)
```

`rank_candidates` returns a `pandas.DataFrame` (indexed by rank) with columns:

| Column | Description |
|---|---|
| `candidate` | Candidate name |
| `text_similarity` | TF-IDF cosine similarity to the job description |
| `skill_match_score` | Fraction of required skills present in the resume |
| `final_score` | `0.6 * text_similarity + 0.4 * skill_match_score` |
| `matched_skills` | Skills found in both resume and job description |
| `missing_skills` | Required skills missing from the resume |

### Individual methods

- `clean_text(text)` — static text cleaner
- `extract_skills(text)` — returns skills found in a text
- `skill_match_score(resume_skills, jd_skills)` — 0–1 match fraction
- `skill_gaps(resume_skills, jd_skills)` — list of missing required skills

## Customizing the skill taxonomy

Pass your own list of skills when creating the system to target a specific
role or industry:

```python
system = ResumeScreeningSystem(skill_taxonomy=[
    "python", "aws", "kubernetes", "terraform", "ci/cd"
])
```

If omitted, a default general tech-skill taxonomy is used (Python, SQL,
ML/DL/NLP libraries, cloud platforms, dev tools, and soft skills).

## Notes

- Matching is keyword/phrase-based with word-boundary regex, so it won't
  catch synonyms or heavily paraphrased skill mentions.
- The scoring weights (0.6 / 0.4) can be adjusted directly in
  `rank_candidates` for different use cases.
