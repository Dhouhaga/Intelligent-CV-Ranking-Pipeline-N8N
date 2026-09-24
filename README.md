# Intelligent CV Ranking Pipeline

An n8n automation that monitors a Google Drive folder for CV uploads, evaluates candidates against a job description using LLM reasoning, and outputs a ranked list of the top 3 candidates.

## Features

- **Automatic Monitoring** – Triggers on new CV uploads (polls every minute)
- **PDF Text Extraction** – Parses resumes and job descriptions directly from Drive
- **LLM-Powered Ranking** – Uses DeepSeek R1 (via OpenRouter) to evaluate and rank candidates
- **Structured Output** – Generates `best_candidates_links.txt` with Drive links to top candidates
- **Self-Cleaning** – Removes stale result files before uploading new ones

## How It Works

1. **Trigger** – Detects new PDFs in the CV folder
2. **Extract** – Downloads and parses text from CVs and job description
3. **Evaluate** – LLM compares candidates against job requirements
4. **Rank** – Outputs top 3 candidates as JSON
5. **Upload** – Saves formatted results with shareable Drive links

## Setup

### Prerequisites

- n8n instance
- Google Drive OAuth2 credentials
- OpenRouter API key (free tier works)
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

## Customization

- **Candidate count** – Edit the AI Agent prompt (default: 3)
- **LLM model** – Swap models in the OpenRouter Chat Model node
- **Pre-filtering** – Add a Code node before the AI Agent to filter CVs by keywords

## Tech Stack

- **n8n** – Workflow orchestration
- **LangChain** – LLM agent integration
- **DeepSeek R1** – Reasoning engine via OpenRouter
- **Google Drive API** – Storage and file management

## Performance

- Execution time: ~30–60 seconds per batch
- Tested with 10–50 CVs per run
