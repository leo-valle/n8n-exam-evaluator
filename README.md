# 📘 PDF Exam Autograder (n8n + OpenAI)

A complete n8n workflow for **automatically grading PDF exams**, comparing student answers with an official answer key using **OpenAI**.  
Supports two grading modes: *exact* (strict, binary) and *lenient* (semantic, flexible).

---

## 🚀 Overview

This workflow:

1. Receives **two PDFs** via an n8n web form:
   - The exam (student answers)
   - The answer key
2. Extracts text from both PDFs
3. Parses and structures questions by number
4. Sends everything to OpenAI with strict evaluation rules
5. Returns:
   - Score per question
   - Brief and objective feedback
   - Final score (0–10)
6. Displays the results using a Form Ending node in HTML

---

## 🧩 Features

- PDF text extraction (Extract From File)
- Robust regex parsing of question numbers
- Two grading modes:
  - **exact → binary grading (1.0 or 0.0)**
  - **lenient → allows intermediate scores**
- Clean JSON output
- Works for any text-based exam
- Automatic normalization of whitespace and formatting

---

## 🛠️ How to Run

### 1. Requirements
- n8n (Cloud or Self-hosted)
- OpenAI Chat node enabled
- Your OpenAI API key

### 2. Environment Variables
Configure in *Settings → Variables* or your environment:

| Variable | Description |
|---------|-------------|
| `OPENAI_API_KEY` | Your OpenAI API key |

### 3. Import the workflow
- Import `workflow.json` from this repository into n8n.

### 4. Running the workflow
1. Open the **Form Trigger** public URL
2. Upload:
   - `test_pdf` = exam (student answers)
   - `answer_key_pdf` = answer key
   - `grading_mode` = `exact` or `lenient`
3. Execute
4. View results on the final page

---

## 🧠 Prompts Used (OpenAI Node)

### **System Prompt**
```text
You are an expert evaluator of academic responses in Systems Engineering.

Your task is to evaluate each student answer by comparing it to the expected answer from the answer key.

Follow ALL rules below exactly.

=====================================================
GRADING MODES
=====================================================

1. EXACT (strict, binary)
- The answer must match the expected conceptual meaning very closely.
- Penalize any omission, simplification, or conceptual deviation.
- Small wording changes are acceptable; conceptual differences are not.
- IMPORTANT: In EXACT mode, the score must be ONLY:
    - 1.0 (correct)
    - 0.0 (incorrect)
  No intermediate values are allowed in EXACT mode.

2. LENIENT (semantic / flexible)
- Evaluate based on the overall semantic meaning.
- Accept simplified but correct answers.
- Penalize only major omissions or conceptual errors.
- Scores may use the full scoring scale.

=====================================================
SCORING SCALE (required)
=====================================================

Allowed scores:

1.00 = fully correct  
0.75 = mostly correct (small omissions)  
0.50 = partially correct  
0.25 = minimally relevant  
0.00 = incorrect or missing

NOTE:
- In EXACT mode → ONLY 1.00 or 0.00 allowed.
- In LENIENT mode → all the above are allowed.

=====================================================
EVALUATION RULES
=====================================================

For EACH question:
- Compare semantic meaning between student answer and expected answer.
- Identify correct points, omissions, and errors.
- Choose the allowed score based on the grading mode.
- Provide short, objective, specific feedback, translated in portuguese.

=====================================================
OUTPUT FORMAT (STRICT JSON ONLY)
=====================================================

Return ONLY this JSON:

{
  "grade_per_question": {
    "1": number,
    "2": number,
    ...
  },
  "comments": {
    "1": "feedback",
    "2": "feedback",
    ...
  },
  "final_grade": number
}

Where:
final_grade = (sum(scores) / total_questions) * 10

=====================================================
INPUT DATA
=====================================================

STUDENT_ANSWERS:
{{JSON.stringify($json.questions)}}

ANSWER_KEY:
{{JSON.stringify($json.answer_key)}}

GRADING_MODE:
{{ $json.grading_mode }}
```
### **User Prompt**
```text
Questions: {{JSON.stringify($json.questions)}}
Answer key: {{JSON.stringify($json.answer_key)}}  
Grading mode: {{ $json.grading_mode }}

```
---

## ⚠️ Known Limitations

  - PDF size limit: constrained by n8n memory and node limits.
  - OCR: scanned PDFs without embedded text may extract poorly.
  - Formatting assumptions: regex relies on consistent numbering (e.g., “1.”, “Question 1:”, “Resposta 1:”).
  - Token limits: very long exams may exceed OpenAI context size.
  - Missing question detection: assumes both PDFs contain matching question numbers.

---

## 📂 Repository Structure
/
├─ workflow.json          # exported n8n workflow
├─ README.md              # documentation
└─ examples/              # example PDFs
