# Intelligent CV Ranking Pipeline

## Overview
End-to-end n8n automation workflow that continuously monitors a Google Drive folder for CV uploads, extracts resume text, evaluates candidates using LLM-powered reasoning, and outputs a ranked list of top 3 candidates back to Drive.

---

## Architecture

### Workflow Flow

```
Google Drive Trigger (CV Folder)
    ↓
[Parallel Paths]
├─ Download Job Description (from Drive)
│  └─ Parse Job Description (text extraction)
│     ↓
└─ List CVs (query Google Drive folder)
   ├─ Download each CV (PDF)
   ├─ Extract text (PDF → text)
   ├─ Clean data (prepare for LLM)
   │
   ↓
[Merge] - Combine job description + all CV data
   ↓
AI Agent (LLM-powered evaluation)
   ├─ Model: DeepSeek R1 Distill Llama 70B (via OpenRouter)
   ├─ Logic: Compare candidates against job requirements
   └─ Output: JSON array of top 3 candidates with Drive links
   ↓
Parse & Structure Results
   ↓
Generate Output (best_candidates_links.txt)
   ↓
Upload to Results Folder
   ↓
Cleanup (remove old result file if exists)
```

---

## Node Breakdown

| Node | Type | Purpose |
|------|------|---------|
| **Google Drive Trigger** | Event Trigger | Polls CV folder every minute for new files |
| **HTTP Request** | API Call | Queries Google Drive API for PDF files in folder |
| **Download file** | File Download | Fetches the job description document |
| **HTTP Request1** | API Call | Downloads each CV PDF from Drive |
| **Extract from File** | PDF Processing | Extracts text from PDF using n8n's built-in parser |
| **save to binary** | Data Transform | Preserves file metadata (name, ID) during extraction |
| **job description parser** | Code (Node.js) | Converts job description binary to UTF-8 text |
| **clean data** | Code (Node.js) | Structures CV data: `{fileName, id, pdfText}` |
| **Merge** | Orchestration | Combines job description + CV data for LLM |
| **AI Agent** | LangChain Agent | LLM-powered candidate evaluation & ranking |
| **OpenRouter Chat Model** | LLM Integration | DeepSeek-based reasoning engine |
| **CV link parser** | Code (Node.js) | Parses LLM JSON output, validates format |
| **result builder** | Code (Node.js) | Formats results as TXT file with Drive links |
| **Search files and folders** | File Query | Finds existing result files for cleanup |
| **Delete a file** | File Management | Removes stale result files (deduplication) |
| **Upload file** | File Upload | Saves ranked results back to Drive |
| **Merge1** | Branch Logic | Routes result file for upload |

---

## LLM Evaluation Logic

**Model:** DeepSeek-R1-Distill-Llama-70B (free tier via OpenRouter)

**Prompt Strategy:**
- Receives: Job description + all candidate resumes (as JSON)
- Compares: Quality, clarity, relevance to role, achievements
- Outputs: JSON array of exactly 3 top candidates
- Format: `[{fileName, driveLink}, ...]`

**Why this approach:**
- Structured output (JSON) ensures parseable results
- Deterministic ranking based on job requirements
- Free model tier keeps costs minimal
- DeepSeek R1 provides reasoning-grade evaluation

---

## Data Flow

```
Input:
  - CV files (*.pdf) in Google Drive folder
  - Job description (*.pdf) in Drive

Processing:
  1. Trigger: Polls for new CVs every minute
  2. Extract: PDF → raw text
  3. Clean: Structure data for LLM
  4. Evaluate: LLM ranks against job description
  5. Parse: Extract top 3 from LLM output
  6. Format: Generate shareable Drive links

Output:
  - best_candidates_links.txt (in results folder)
  - Format: "FileName: https://drive.google.com/file/d/{ID}/view"
```

---

## Setup Instructions

### Prerequisites
- n8n instance (self-hosted or n8n Cloud)
- Google Drive API credentials (OAuth2)
- OpenRouter API key (free tier available)
- Two Google Drive folders:
  - CV folder (watch for new PDFs)
  - Results folder (output destination)
- One job description PDF in Drive

### Configuration Steps

1. **Google Drive Setup**
   - Create two folders in Drive: `/cv-manager` and `/cv-manager-results`
   - Upload job description PDF to a known location
   - Replace placeholders in workflow:
     - `YOUR_CV_FOLDER_ID` → CV folder ID
     - `YOUR_RESULTS_FOLDER_ID` → Results folder ID
     - `YOUR_JOB_DESCRIPTION_FILE_ID` → Job description file ID

2. **Credentials**
   - Connect Google Drive OAuth2 (n8n will handle scopes)
   - Add OpenRouter API key to OpenRouter Chat Model node
   - Test authentication

3. **Deploy**
   - Import workflow JSON into n8n
   - Enable "Active" toggle
   - Monitor execution logs for errors

### Extracting Folder/File IDs from Drive URLs
```
Folder: https://drive.google.com/drive/folders/{FOLDER_ID}
File:   https://drive.google.com/file/d/{FILE_ID}/view
```

---

## Key Features

**Continuous Monitoring** - Triggers automatically on new CV uploads  
**Scalable** - Handles 10+ CVs per run (configurable)  
**LLM-Powered** - Reasoning-based candidate ranking  
**Cloud-Native** - Leverages Google Drive as storage  
**Fault-Tolerant** - Cleanup logic prevents duplicate results  
**Production-Grade** - Error handling, data validation, structured output  

---

## Customization

### Change Candidate Count
In **AI Agent** node, modify prompt:
```
Return only a JSON array with THREE objects for the best three candidates.
```
Change "THREE" to desired number.

### Change LLM Model
In **OpenRouter Chat Model** node, select different model:
- Claude 3 (via OpenRouter)
- GPT-4 (via OpenRouter)
- Llama 2/3 variants

### Add Filtering Logic
Add a **Code** node before AI Agent to pre-filter CVs:
```javascript
// Only process CVs with certain keywords
return $input.all().filter(item => 
  item.json.pdfText.toLowerCase().includes('python')
);
```

---

## Monitoring & Debugging

**View Logs:**
- n8n UI → Workflow → "Executions" tab
- Check each node's output for errors
- Common issues:
  - API rate limits (slow down trigger interval)
  - PDF parsing failures (malformed PDF format)
  - LLM timeout (increase timeout in model settings)

**Test Manually:**
- Click "Test Workflow" button
- Upload a test CV to trigger folder
- Check results folder for output

---

## Technologies Used

- **n8n** - Workflow orchestration
- **LangChain** - LLM agent integration
- **DeepSeek R1** - LLM backbone (via OpenRouter)
- **Google Drive API** - Cloud storage integration
- **Node.js** - Custom data transformation code nodes
- **PDF.js** - PDF text extraction

---

## Performance Notes

- **Execution time:** ~30-60 seconds per batch (depends on CV count)
- **Trigger interval:** Every 1 minute (adjustable)
- **Cost:** Free tier (DeepSeek + Google Drive quota)
- **Scalability:** Tested with 10-50 CVs per batch

---
