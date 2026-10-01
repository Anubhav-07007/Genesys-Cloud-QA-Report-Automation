# Genesys Cloud QA Report Automation

Automatically extract voice interaction data, survey results, and transcripts from Genesys Cloud and generate an Excel report for the Quality team, with no manual work.

## The problem

The Quality team needed a report of all voice interactions with completed surveys, containing:

- Conversation ID
- Date and time
- Agent name
- Survey score
- Survey status
- Transcript

This combined view is not available in the Genesys Cloud UI. The data lives behind several different APIs, so building it by hand for a week or a month of interactions was not practical.

## The solution

1. **Prototyped in Postman.** Identified and tested the 4 Genesys Cloud APIs needed to get the data.
2. **Automated in Python.** Built a script (with AI-assisted code generation, then reviewed and tested line by line) that:
   - Authenticates with Genesys Cloud using OAuth (Client Credentials)
   - Calls the Analytics API with pagination to fetch conversations
   - Filters for voice interactions with finished surveys
   - Resolves agent names
   - Retrieves, downloads, and parses transcripts
   - Writes everything to an Excel report with a timestamped filename
3. **Scheduled it.** A batch file runs the script, and Windows Task Scheduler triggers it automatically.

## Workflow

```
Task Scheduler -> run_report.bat -> Python script
                                        |
        OAuth token -> Analytics API (paginated) -> filter voice + finished surveys
                                        |
        Agent name lookup -> Transcript URL -> Download + parse transcript
                                        |
                      Excel report (timestamped filename)
```

## Tech stack

- Genesys Cloud Platform API (OAuth Client Credentials)
- Python 3 (`requests`, `pandas`, `openpyxl`, `python-dotenv`)
- Postman (API discovery and testing)
- VS Code
- Windows Task Scheduler + batch file

## Setup

1. Clone the repo and create a virtual environment:
   ```
   python -m venv venv
   venv\Scripts\activate
   pip install -r requirements.txt
   ```
2. Create an OAuth client in Genesys Cloud (Client Credentials grant) with only the read permissions the script needs.
3. Copy `.env.example` to `.env` and fill in your values:
   ```
   GENESYS_REGION=your-region-domain
   GENESYS_CLIENT_ID=your-client-id
   GENESYS_CLIENT_SECRET=your-client-secret
   ```
4. Run:
   ```
   python main.py
   ```

## Scheduling (Windows)

1. Create `run_report.bat` that activates the environment and runs `main.py`.
2. In Task Scheduler, create a task that runs the batch file on your chosen schedule.
3. Note: the PC must be on and awake at the scheduled time, or the run may be missed.

## Security

- Credentials live only in `.env`, which is excluded by `.gitignore`.
- Use a dedicated OAuth client with least-privilege, read-only roles.
- Never commit reports, transcripts, or any customer data.
- Rotate the client secret if it is ever exposed.

## Project structure

```
.
├── main.py
├── run_report.bat
├── requirements.txt
├── .env.example
├── .gitignore
└── README.md
```

## Lessons learned

- Postman is great for discovering and validating APIs, but Python is needed for large volumes of data.
- Pagination is essential when pulling large analytics datasets.
- AI tools can speed up scripting, but reviewing and testing every step is what makes the result reliable.
- Start with one week of data to validate, then scale up.

## Possible improvements

- Email the report automatically
- Re-run missed schedules
- Logging and retry handling
- Move to a server or cloud scheduler so it does not depend on a laptop

## Disclaimer

This is a personal learning project. All data shown is sample or anonymized. It is not affiliated with or endorsed by Genesys.
