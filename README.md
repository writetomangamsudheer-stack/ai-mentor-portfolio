# ai-mentor-portfolio-Mangam Sudheer
## Day 1 — Setup complete

- ✅ Google AI Studio API key provisioned
- ✅ Groq API key provisioned
- ✅ Hello-Gemini call working — see [Day1_Setup.ipynb](Day1_Setup.ipynb)
- 4-tool comparison matrix from Lab 1A: see screenshot below

![Gemini first call]
<img width="832" height="263" alt="gemini_first_call" src="https://github.com/user-attachments/assets/5b8fb8ec-2061-47df-a4fe-39f02134f04b" />

## Day 4 — Productivity sprint

**Company:** TCS

### Edit notes

1. Verified hiring statistics before finalizing slides.
2. Removed unverifiable package information.
3. Edited cover slide to make it company-specific.

## Day 5 — resume scorer completed 
https://resume-scorer-ckg4q6mvrorwmtauqvx2pf.streamlit.app/

Hugging Face NLP Lab complete

| Method                | Minimum Time | Average Time |
| --------------------- | ------------ | ------------ |
| Hugging Face API      | 0.01s        | 0.14s        |
| Local Model Inference | 0.82s        | 0.87s        |

Reflection
API inference is faster to start with and easier to deploy because the model is hosted remotely.
Local inference is slower initially but gives better privacy, offline capability, and no API dependency.
In production, APIs are ideal for quick prototyping and low infrastructure cost, while local models are better for high-scale, privacy-sensitive, or offline systems.


## Day 6 Lab 6A — Errors handled

1. **Markdown fence wrapping** (` ```json ... ``` `)
   Gemini sometimes wraps responses inside markdown code fences even when JSON-only output is requested. The retry logic re-prompts the model to return raw JSON without markdown formatting.

2. **Missing optional fields**
   Some résumés did not contain phone numbers. Using `Optional[str] = None` in the Pydantic schema allowed Gemini to return `null` while still passing validation safely.

3. **Empty / whitespace-only input**
   Gemini unexpectedly generated placeholder or empty résumé objects instead of failing validation automatically. This showed that schema validation alone is insufficient because empty strings and empty lists still satisfy type constraints.

4. **Hallucination on garbage input**
   For non-résumé text such as "the quick brown fox jumps over the lazy dog", Gemini generated an empty or fabricated résumé structure instead of rejecting the input. This demonstrated LLM hallucination behaviour and the need for input validation before model calls.

### Defence Added

To improve reliability:
- validate minimum input length before sending to Gemini
- reject whitespace-only input
- optionally check for résumé-like patterns (email, skills, education keywords)
- add semantic validation after Pydantic validation (e.g., non-empty name/email)

### Key Learning

`response_schema` guarantees JSON structure, but not semantic correctness. Production-grade AI pipelines require:
- input validation
- schema validation
- semantic validation
- hallucination defence

# Day 6 — Capstone Sprint 1: PlacementDataProcessor

## Engineer Answer

### 1. PROBLEM

Job Descriptions from company websites are messy HTML pages. Placement teams need structured data to filter jobs based on skills, role requirements, and eligibility criteria.

Manual extraction is slow and difficult for large numbers of JDs.

---

### 2. ARCHITECTURE

JD URL  
→ BeautifulSoup scraper  
→ Gemini structured extraction (`response_schema`)  
→ Pydantic validation  
→ `data/jds.jsonl`

The system converts unstructured job pages into clean structured JSON.

---

### 3. TRADE-OFFS

- Scraping is fragile because some websites block bots.
- Dynamic JavaScript pages reduce extraction quality.
- Gemini provides structured JSON but may still miss semantic details.
- Most latency comes from Gemini API calls.

---

### 4. SCALE

- Small batches work easily on free Gemini API.
- Large-scale systems require retry logic, queues, and paid APIs.
- JSONL datasets can later be indexed into RAG systems.

---

### 5. INTERVIEW ANSWER

"I built a schema-first pipeline that converts scraped Job Descriptions into structured JSON using Gemini structured outputs and Pydantic validation."

---

## Extraction Observations

- Amazon and Meta job pages produced the strongest extraction quality because they exposed detailed qualification text in static HTML.
- Google Careers pages were partially JavaScript-rendered, reducing extraction quality in some cases.
- The pipeline handled partial scraping failures gracefully and continued processing remaining JDs.

---

## Files

- `Day6_PlacementProcessor.ipynb`
- `data/jds.jsonl`

---

## Pair

- Mentor 1: Mangam Sudheer
- Mentor 2: Surimalli Koteswara Rao

# Day 7 Lab 7A — ChromaDB Hello-World

## Tasks Completed

- Generated 384-dim embeddings using `all-MiniLM-L6-v2`
- Indexed syllabus paragraphs in ChromaDB
- Ran semantic similarity queries
- Visualized embeddings using PCA
- Added a food outlier sentence

## Observations

- Similar engineering topics clustered together
- Unrelated queries returned nearest semantic matches
- Food outlier appeared far from syllabus topics

## Reflection

Semantic search returns the nearest semantic match, not exact truth.
This lab demonstrated the basics of embeddings, vector databases, semantic search, and RAG systems.

## Files

- `Day7_RAG_Chatbot.ipynb`

# PlacementKnowledgeRAG

## Day 7 — Capstone Sprint 2

A Retrieval-Augmented Generation (RAG) system built using:
- MiniLM embeddings
- ChromaDB vector database
- LangChain
- Placement JDs + syllabus documents

The system retrieves relevant placement-related information with citations.

---

# Problem Statement

LLMs do not know private placement data such as:
- company job descriptions
- syllabus documents
- internal placement preparation material

This project solves that problem using RAG (Retrieval-Augmented Generation).

---

# Architecture

User Question
↓
MiniLM Embeddings
↓
ChromaDB Vector Store
↓
Top-k Similarity Search
↓
Retrieved Chunks with Citations

---

# Tech Stack

- Python
- Sentence Transformers
- ChromaDB
- LangChain
- Google Gemini API
- HuggingFace Embeddings

---

# Features

- Indexed 50+ placement documents
- Vector similarity search
- Citation-based retrieval
- JD search
- Syllabus topic retrieval
- Out-of-corpus detection

---

# Chunking Strategy

- Chunk Size: 500
- Chunk Overlap: 50
- Embedding Model:
  `sentence-transformers/all-MiniLM-L6-v2`

---

# Sample Questions

## 1. Which companies want Java + DSA + CGPA 7+?

Retrieved JD chunks mentioning:
- Java
- DSA
- CGPA requirements

Sources:
- jd_0
- jd_5

---

## 2. What are the Sem 5 OS topics?

Retrieved syllabus chunks mentioning:
- paging
- scheduling
- deadlocks
- virtual memory

Sources:
- cse_sem5_2
- cse_sem5_5

---

## 3. Which JDs require Python?

Retrieved JD chunks mentioning Python requirements.

Sources:
- jd_3
- jd_5
- jd_9

---

## 4. Companies hiring in Hyderabad?

Retrieved JD chunks mentioning Hyderabad location.

Sources:
- jd_2
- jd_6

---

## 5. What is TCS Codevita?

Answer:
I do not know.

Reason:
Information not present in indexed corpus.

---

# Trade-offs

## Advantages
- Fast retrieval
- Free local embeddings
- Citation-based answers
- No retraining needed

## Limitations
- Retrieval quality depends on chunking
- Gemini API quota issues during testing
- No answer generation when API quota exhausted

---

# Scale

- 50 docs → works easily locally
- 5K docs → manageable with ChromaDB
- 1M docs → requires optimized vector DBs

---

# Interview Answer

“I built a placement-focused RAG system using MiniLM embeddings, ChromaDB, and LangChain. The system indexes placement JDs and syllabus documents, retrieves relevant chunks using vector similarity search, and provides citation-based answers.”

---

# Project Status

✅ Embedding pipeline working  
✅ ChromaDB indexing working  
✅ Retrieval pipeline working  
✅ Citation retrieval working  
✅ Local RAG search working

---

# Note

Gemini API quota was exhausted during testing.
However, the local retrieval pipeline using MiniLM embeddings + ChromaDB worked successfully.

