# GES Streamlit App - Modular Version

## Folder structure

- `app.py` - Streamlit UI and orchestration
- `services/config.py` - app configuration
- `services/auth.py` - Google access token helper
- `services/http_utils.py` - retry-enabled HTTP session
- `services/gcs_service.py` - Cloud Storage upload and download
- `services/speech_service.py` - Speech-to-Text submit and polling
- `services/dlp_service.py` - DLP masking
- `services/bigquery_service.py` - BigQuery table creation and inserts
- `services/parsers.py` - transcript parsing and field extraction

## Why this structure

This keeps the app easier to explain to a manager:
- UI layer is separated from cloud integrations
- each cloud service has its own file
- parsing and masking logic are isolated
- retry/network handling is centralized

## Run

```bash
cd ges_streamlit_modular
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python -m streamlit run app.py
```

## Notes

- keep `Create dataset/tables if missing` unchecked for faster tests
- use short `.wav` files for demo
- connect Looker Studio to:
  `ges-poc-490514.ges_demo.call_transcripts_analytics`
