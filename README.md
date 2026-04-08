# Google Cloud Enterprise Services (GES) - Streamlit Application

A modular Python application that integrates Google Cloud services for audio transcription, data privacy masking, and analytics. This application uses Streamlit for the UI layer and orchestrates multiple Google Cloud APIs.

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Features](#features)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Running the Application](#running-the-application)
- [Workflows](#workflows)
- [Configuration](#configuration)
- [Project Structure](#project-structure)
- [Best Practices](#best-practices)

## 🎯 Project Overview

The GES Streamlit App is a cloud-native application designed to process audio files through an intelligent pipeline that includes:

- **Speech-to-Text Conversion**: Transform audio files into transcripts using Google Cloud Speech-to-Text API
- **Data Privacy**: Mask sensitive information using Google Cloud DLP (Data Loss Prevention)
- **Data Storage**: Store and retrieve files from Google Cloud Storage
- **Analytics**: Load processed data into BigQuery for analysis and reporting

The application follows a modular architecture that separates UI concerns from cloud service integrations, making it easy to maintain, test, and explain to stakeholders.

## ✨ Features

- **Modular Architecture**: Clean separation between UI layer and cloud service integrations
- **Retry-Enabled HTTP**: Robust network handling with automatic retries
- **Cloud Storage Integration**: Upload and download files from Google Cloud Storage
- **Speech Recognition**: Automatic speech-to-text processing with polling support
- **Data Masking**: Sensitive information detection and masking using DLP
- **BigQuery Analytics**: Automatic table creation and data insertion for analytics
- **Transcript Parsing**: Intelligent parsing and field extraction from transcripts
- **OAuth Authentication**: Secure Google Cloud authentication via OAuth 2.0

## 🏗️ Architecture

The application is organized into clear, maintainable components:

### Folder Structure

- `app.py` - Streamlit UI and orchestration
- `services/config.py` - app configuration
- `services/auth.py` - Google access token helper
- `services/http_utils.py` - retry-enabled HTTP session
- `services/gcs_service.py` - Cloud Storage upload and download
- `services/speech_service.py` - Speech-to-Text submit and polling
- `services/dlp_service.py` - DLP masking
- `services/bigquery_service.py` - BigQuery table creation and inserts
- `services/parsers.py` - transcript parsing and field extraction

### Why This Structure

This keeps the app easier to explain to a manager:
- UI layer is separated from cloud integrations
- Each cloud service has its own file
- Parsing and masking logic are isolated
- Retry/network handling is centralized

| Service | Purpose |
|---------|---------|
| `config.py` | Centralized configuration and environment variables |
| `auth.py` | Manages Google Cloud authentication tokens |
| `http_utils.py` | HTTP session with retry policies for reliability |
| `gcs_service.py` | Upload/download files from Cloud Storage |
| `speech_service.py` | Submit audio for transcription and poll results |
| `dlp_service.py` | Detect and mask sensitive information |
| `bigquery_service.py` | Create datasets/tables and insert data |
| `parsers.py` | Parse transcripts and extract structured data |

## 📦 Prerequisites

Before you begin, ensure you have the following:

- **Python 3.8+** installed on your system
- **Google Cloud Project** with the following APIs enabled:
  - Cloud Speech-to-Text API
  - Cloud Storage API
  - BigQuery API
  - Cloud Data Loss Prevention API
- **Google Cloud Service Account** with appropriate permissions
- **Service Account Key** (JSON file) or configured Application Default Credentials
- **pip** (Python package manager)

## 🚀 Installation

### Step 1: Clone the Repository

```bash
git clone https://github.com/paulrajchellappa/google-ces.git
cd google-ces
```

### Step 2: Create Virtual Environment

```bash
python3 -m venv .venv
```

### Step 3: Activate Virtual Environment

**On macOS/Linux:**
```bash
source .venv/bin/activate
```

**On Windows:**
```bash
.venv\Scripts\activate
```

### Step 4: Install Dependencies

```bash
pip install -r requirements.txt
```

### Step 5: Set Up Google Cloud Authentication

Set the `GOOGLE_APPLICATION_CREDENTIALS` environment variable to point to your service account key:

**On macOS/Linux:**
```bash
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/your/service-account-key.json"
```

**On Windows:**
```bash
set GOOGLE_APPLICATION_CREDENTIALS=C:\path\to\your\service-account-key.json
```

Alternatively, use Application Default Credentials:
```bash
gcloud auth application-default login
```

## ▶️ Running the Application

Once installation is complete, start the Streamlit application:

```bash
python -m streamlit run app.py
```

The application will start and be accessible at `http://localhost:8501` in your browser.

### First Run Tips

- **Uncheck "Create dataset/tables if missing"** during initial testing for faster iterations
- **Use short `.wav` files** for demo and testing purposes
- The application will guide you through the workflow via the Streamlit UI

## 🔄 Workflows

### Main Application Workflow

1. **Upload Audio File**
   - User uploads an audio file (`.wav`, `.mp3`, etc.) via Streamlit UI
   - File is temporarily stored locally

2. **Upload to Cloud Storage**
   - Audio file is uploaded to Google Cloud Storage
   - Reference URI is stored for later retrieval

3. **Submit to Speech-to-Text**
   - Audio file URI is submitted to Cloud Speech-to-Text API
   - Receives operation ID for polling

4. **Poll Speech-to-Text Results**
   - Application polls the Speech-to-Text API for completion
   - Retrieves transcript once processing is complete

5. **Apply DLP Masking**
   - Transcript is scanned using Cloud DLP for sensitive information
   - PII (Personally Identifiable Information) is masked
   - Masked transcript is returned

6. **Parse and Extract Fields**
   - Transcript is parsed to extract structured fields
   - Speaker information, timestamps, and topics are identified

7. **Store in BigQuery**
   - Processed data is inserted into BigQuery tables
   - Tables are created automatically if they don't exist
   - Data becomes available for analytics and reporting

### BigQuery Configuration

Connect Looker Studio to `ges-poc-490514.ges_demo.call_transcripts_analytics` for visualization.

## ⚙️ Configuration

### Environment Variables

Configure the application using environment variables:

```bash
# Google Cloud Project ID
GOOGLE_CLOUD_PROJECT=your-project-id

# Cloud Storage Bucket
GCS_BUCKET=your-bucket-name

# BigQuery Dataset
BQ_DATASET=your_dataset_name

# Authentication
GOOGLE_APPLICATION_CREDENTIALS=/path/to/key.json
```

All configurable parameters are centralized in `services/config.py`:
- API timeouts
- Retry policies
- Bucket names
- Dataset names
- Default speech recognition language

## 💡 Best Practices

### Development

1. **Use Short Audio Files**: Test with 10-30 second audio clips for faster feedback
2. **Skip Table Creation**: During development, uncheck "Create dataset/tables if missing" to speed up iterations
3. **Monitor Quotas**: Keep track of Google Cloud API quotas to avoid unexpected limits
4. **Enable Logging**: Add logging to track API calls and debug issues

### Production

1. **Error Handling**: Implement comprehensive error handling for API failures
2. **Retry Logic**: Use the built-in retry mechanisms in `http_utils.py`
3. **Authentication**: Use service accounts with minimal required permissions
4. **Monitoring**: Set up Cloud Logging and Cloud Monitoring for production deployments
5. **Cost Optimization**: Configure appropriate API quotas and rate limits

### Security

1. **Credentials**: Never commit service account keys to version control
2. **Access Control**: Use Google Cloud IAM to restrict service account permissions
3. **Data Privacy**: Ensure DLP masking rules match your data classification requirements
4. **Audit Logging**: Enable Cloud Audit Logs for compliance tracking

## 📚 Dependencies

The application uses the following Python packages:

- **streamlit** (≥1.37.0): Web UI framework
- **google-cloud-storage** (≥2.18.0): Cloud Storage operations
- **google-cloud-bigquery** (≥3.25.0): BigQuery operations
- **google-auth** (≥2.34.0): Google Cloud authentication
- **requests** (≥2.32.0): HTTP library
- **urllib3** (≥2.2.0): HTTP client library

See `requirements.txt` for the complete list.

---

**Last Updated**: April 8, 2026
