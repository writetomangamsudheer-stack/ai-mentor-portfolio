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
