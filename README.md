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


### 2. ARCHITECTURE

JD URL  
→ BeautifulSoup scraper  
→ Gemini structured extraction (`response_schema`)  
→ Pydantic validation  
→ `data/jds.jsonl`

The system converts unstructured job pages into clean structured JSON.


### 3. TRADE-OFFS

- Scraping is fragile because some websites block bots.
- Dynamic JavaScript pages reduce extraction quality.
- Gemini provides structured JSON but may still miss semantic details.
- Most latency comes from Gemini API calls.


### 4. SCALE

- Small batches work easily on free Gemini API.
- Large-scale systems require retry logic, queues, and paid APIs.
- JSONL datasets can later be indexed into RAG systems.


### 5. INTERVIEW ANSWER

"I built a schema-first pipeline that converts scraped Job Descriptions into structured JSON using Gemini structured outputs and Pydantic validation."


## Extraction Observations

- Amazon and Meta job pages produced the strongest extraction quality because they exposed detailed qualification text in static HTML.
- Google Careers pages were partially JavaScript-rendered, reducing extraction quality in some cases.
- The pipeline handled partial scraping failures gracefully and continued processing remaining JDs.


## Files

- `Day6_PlacementProcessor.ipynb`
- `data/jds.jsonl`


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




# Day 7 — Capstone Sprint 2: PlacementKnowledgeRAG

## Overview
Built a citation-enforcing RAG (Retrieval-Augmented Generation) chatbot using:
- MiniLM embeddings
- ChromaDB vector database
- LangChain RetrievalQA
- Gemini API

The system indexes placement Job Descriptions (JDs) and syllabus documents, retrieves relevant chunks using semantic similarity, and answers student questions with citations.

---

# Engineer Answer

## 1. PROBLEM
Frontier LLMs do not know private institutional data such as placement JDs and syllabus documents. Students need a chatbot that answers ONLY from the placement corpus and provides verifiable citations.

---

## 2. ARCHITECTURE

5-stage RAG pipeline:

1. Embed  
   - Model: `sentence-transformers/all-MiniLM-L6-v2`
   - 384-dimensional embeddings

2. Index  
   - ChromaDB persistent vector collection
   - Metadata stored for company, source, CGPA, etc.

3. Retrieve  
   - Top-4 cosine similarity retrieval

4. Augment  
   - Citation-enforcing prompt
   - Strict instruction: "Do NOT guess"

5. Generate  
   - Gemini / LangChain QA chain

---

## 3. TRADE-OFFS

- Cost:
  - MiniLM embeddings are free and local
  - Gemini API has quota limits

- Accuracy:
  - Retrieval works well for placement-related queries

- Latency:
  - Retrieval: <1 second
  - LLM generation: depends on API quota

- Complexity:
  - Chunking strategy affects retrieval quality

- Caveat:
  - Out-of-corpus detection requires strict prompting

---

## 4. SCALE

- 50 documents:
  - Runs easily on Colab/local machine

- 5,000 documents:
  - Still manageable using ChromaDB

- 1M+ documents:
  - Requires optimized vector databases like:
    - Pinecone
    - Weaviate
    - FAISS server mode

---

## 5. INTERVIEW ANSWER

"I built a citation-enforcing RAG chatbot over placement JDs and syllabus documents using MiniLM embeddings, ChromaDB, LangChain RetrievalQA, and Gemini. The system retrieves top-k relevant chunks and either answers with citations or refuses out-of-corpus questions to avoid hallucinations."

---

# 5 Cited Q&A Pairs

| # | Question | Answer (excerpt) | Sources cited |
|---|---|---|---|
| 1 | Which companies want Java + DSA + CGPA 7+? | TCS Digital requires Java + DSA with CGPA 7.0. Goldman Sachs also requires Java + DSA with CGPA 8.5. | TCS Digital, Goldman Sachs |
| 2 | What are the Sem 5 OS topics? | Retrieval issue occurred because syllabus chunks were not returned correctly. Needs chunk tuning/re-indexing. | No syllabus chunks retrieved |
| 3 | Which JDs require Python? | Cognizant, Goldman Sachs, and Accenture require Python in must-have skills. | Cognizant, Goldman Sachs, Accenture |
| 4 | Companies hiring in Hyderabad? | Cognizant, TCS Digital, Accenture, and Deloitte USI are hiring in Hyderabad. | Cognizant, TCS Digital, Accenture, Deloitte USI |
| 5 | What is TCS Codevita? | I do not know — not found in indexed corpus. | None |

---

# Retrieval Examples

## Example 1

### Question
Which JDs require Python?

### Retrieved Results
- Cognizant
- Goldman Sachs
- Accenture

### Observation
Semantic retrieval correctly matched Python-related JDs.

---

## Example 2

### Question
Companies hiring in Hyderabad?

### Retrieved Results
- Cognizant
- TCS Digital
- Accenture
- Deloitte USI

### Observation
Location-based retrieval worked correctly using metadata-rich documents.

---

# Issues Faced

## 1. Gemini API Quota Exhausted
Encountered:
- `429 RESOURCE_EXHAUSTED`

Fix:
- Switched temporarily to retrieval-only testing using `similarity_search()`

---

## 2. Dependency Conflicts
Issues with:
- protobuf
- tensorflow
- sentence_transformers
- langchain versions

Fix:
- Installed compatible package versions manually

---

## 3. Poor Retrieval for OS Topics
Sem 5 OS question returned JD chunks instead of syllabus chunks.

Reason:
- Chunking/retrieval overlap issue

Planned Fix:
- Reduce chunk size
- Increase syllabus chunk density
- Use metadata filtering

---

# Final Outcome

Successfully built:
- Persistent ChromaDB vector store
- Semantic search pipeline
- Citation-based retrieval system
- LangChain RetrievalQA workflow

The project demonstrates production-style RAG architecture suitable for:
- Placement assistants
- Internal knowledge bots
- College academic assistants
- Enterprise document QA systems
- 
## Day 9 Lab 9A — Hello-LangGraph

- 1-tool ReAct agent with DuckDuckGo web_search
- 4-message trace on a live-fact question (TCS 2026 hiring)
- Failure case: bad URL → agent reported domain does not exist

### Reflection (3 lines)

1. The trace IS the explanation. Print every step.
2. The doc-string IS the prompt. Bad doc-string = bad tool selection.
3. Real agents handle tool failures gracefully.

## Day 9 — Capstone Sprint 4: Career Agent

### 3 tools wired
1. jd_fetcher
2. skills_gap
3. answer_scorer

### Successful runs
- Student 1 completed
- Student 2 completed
- Student 3 completed

### Tool usage observed
- answer_scorer tool invoked successfully
- Agent selected tools using doc-strings
- Agent avoided hallucination when data was missing

### 1 failure-recovery analysis
Bad URL passed to jd_fetcher.
Agent correctly reported URL fetch failure.

### Engineer Answer

1. Built a 3-tool LangGraph agent.
2. Agent performs JD analysis and interview-answer scoring.
3. Tools are selected using doc-strings.
4. Failure handling prevents hallucination.
5. Agent supports placement preparation.

However, the local retrieval pipeline using MiniLM embeddings + ChromaDB worked successfully.

## Day 10 Lab 10A — Hello-CrewAI

### Goal
Built a 2-agent CrewAI system that generates a 1-page TCS Digital placement preparation brief.

### Agents
1. **Placement Researcher** — prepares factual placement notes.
2. **Placement Brief Writer** — converts notes into a student-friendly markdown brief.

### Workflow
Researcher → Writer → Final Markdown Brief

### Files Generated
- `day10_lab10a_transcript.txt`
- `tcs_digital_brief.md`

### Reflection
1. The handoff between agents is the design quality.
2. `expected_output` is the contract between agents.
3. Verbose mode helps debug multi-agent workflows.


# Day10-Capstone Sprint_ Placement Prep Crew

## Overview

Placement Prep Crew is a multi-agent AI workflow built using CrewAI, Gemini API, ChromaDB, and Sentence Transformers. The system automates placement preparation by generating research notes, mock interview questions, coaching answers, and progress summaries for students.

---

# Architecture

Researcher → Interviewer → Coach → Tracker

The workflow uses sequential multi-agent orchestration where each agent performs a specialized task and passes outputs to the next agent.

---

# Agents

## Researcher Agent

Uses RAG-based retrieval from ChromaDB to collect placement preparation information.

## Interviewer Agent

Generates technical and HR interview questions.

## Coach Agent

Creates strong sample answers and preparation guidance.

## Tracker Agent

Generates structured JSON-style progress summaries.

---

# Features

* Multi-agent orchestration
* Sequential workflow execution
* RAG integration
* Automated interview preparation
* JSON transcript generation
* Markdown report generation

---

# Workflow

1. Student profile is provided
2. Researcher gathers preparation information
3. Interviewer generates mock questions
4. Coach creates sample answers
5. Tracker generates structured summaries
6. Reports are saved as JSON and Markdown files

---

# Outputs

* day10_sprint5_transcripts.json
* day10_sprint5_report.md

# Learning Outcomes

* CrewAI orchestration
* Multi-agent system design
* RAG implementation
* Sequential task pipelines
* Structured AI workflows

# Conclusion

This project demonstrates how multi-agent AI systems can automate placement preparation workflows using modular orchestration, retrieval-based reasoning, and structured reporting.

## Day 11 Lab 11A — Ollama Offline + Hybrid Fallback

### Completed
- ✅ Ollama installed locally
- ✅ llama3.2 model downloaded
- ✅ Offline AI tested after Wi-Fi disconnect
- ✅ Gemini → Groq → Ollama fallback chain implemented
- ✅ Force-failure testing completed
- ✅ Local fallback verified

<img width="1914" height="1079" alt="image" src="https://github.com/user-attachments/assets/027465a5-d109-40c2-b5eb-12ee731168fc" />


