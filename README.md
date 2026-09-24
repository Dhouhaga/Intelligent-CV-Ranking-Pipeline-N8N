# Intelligent CV Ranking Pipeline

An n8n workflow that polls a Google Drive folder for new CVs, evaluates each candidate against a job description using an LLM, and writes a ranked list of the top 3 candidates back to Drive.

## How It Works

1. **Poll** – The Google Drive Trigger checks the CV folder every minute for new files
2. **Extract** – Downloads the job description and any new CVs, then parses text from each PDF
3. **Evaluate** – An LLM compares candidates against the job description
4. **Rank** – Returns the top 3 candidates as JSON
5. **Output** – Writes `best_candidates_links.txt` (with Drive links) to the results folder, deleting any previous result file first

## Nodes Overview

| Stage | Nodes |
|---|---|
| Trigger | Google Drive Trigger (polls CV folder every minute) |
| Ingest | HTTP Request (list/download CVs), Download file (job description) |
| Extract | Extract from File (PDF → text), save to binary, job description parser |
| Prepare | clean data (structures `{fileName, id, pdfText}`), Merge |
| Evaluate | AI Agent + OpenRouter Chat Model (DeepSeek R1 Distill Llama 70B) |
| Output | CV link parser, result builder, Search files and folders, Delete a file, Upload file |

## Setup

### Prerequisites

- n8n instance
- Google Drive OAuth2 credentials
- OpenRouter API key
- Two Drive folders: one for CVs, one for results
- Job description PDF in Drive

### Configuration

1. Import the workflow JSON into n8n
2. Replace placeholders:
   - `YOUR_CV_FOLDER_ID`
   - `YOUR_RESULTS_FOLDER_ID`
   - `YOUR_JOB_DESCRIPTION_FILE_ID`
3. Connect Google Drive and OpenRouter credentials
4. Activate the workflow

To get IDs from Drive URLs:
```
Folder: https://drive.google.com/drive/folders/{FOLDER_ID}
File:   https://drive.google.com/file/d/{FILE_ID}/view
```

## Output

`best_candidates_links.txt`, written to the results folder:

```
FileName: https://drive.google.com/file/d/{ID}/view
```

## Customization

- **Candidate count** – Edit the AI Agent prompt (default: 3)
- **LLM model** – Change the model in the OpenRouter Chat Model node
- **Pre-filtering** – Add a Code node before the AI Agent to filter CVs by keyword

## Notes

- Trigger interval is 1 minute (adjustable in the trigger node)
- Execution time: roughly 30–60 seconds per batch
- Uses a free-tier model via OpenRouter
