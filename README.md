# COVID-19-Automated-Reporting-System
n8n + GenAI Automation Workflow

## Automation Problem Statement
Manual analysis of COVID-19 data is time-consuming and requires constant human intervention. This project automates the entire pipeline — from reading raw COVID data to generating AI-powered summaries and alerts — and delivers the report directly to Slack every day without any manual effort.

## The Problem:
-COVID data needs daily monitoring
-Manually reading CSV and writing summaries takes time
-Alerts for rising cases can be missed
-No automated reporting system exists

## The Solution:
-n8n reads COVID CSV automatically every day at 9AM IST
-Groq AI (LLaMA 3.3) analyzes the data and generates a summary
-Report is saved as a text file AND sent to Slack automatically
-Zero manual intervention required

## Tech Stack
n8n (Self Hosted) -- Workflow Automation
Groq AI (LLaMA 3.3-70b) -- GenAI Summary Generation
Slack API -- Report Delivery
COVID-19 CSV Dataset -- Data Source
Power BI -- Dashboard Visualization

## n8n Workflow Steps
Complete Workflow:
Schedule Trigger (9AM IST)
→ Read File From Disk
→ Extract From File (CSV Parser)
→ Code Node 1 (Data Formatter)
→ HTTP Request (Groq AI API)
→ Code Node 2 (Report Builder)
→ Write File (Save Report)
→ Slack (Send DM Report)

### Node 1 — 🕐 Schedule Trigger
Purpose: Automatically starts the workflow every day at 9AM IST
Configuration:

Trigger: Every Day
Time: 09:00 AM
Timezone: Asia/Kolkata

Why: Ensures daily automated reporting without any manual trigger

### Node 2 — 📁 Read File From Disk
Purpose: Opens and reads the COVID CSV file from local storage
Configuration:

Operation: Read File
File Path: C:\Users\nikhi\.n8n-files\covid_eda_task4.csv

Why: Fetches raw data that will be analyzed by AI

### Node 3 — 📊 Extract From File
Purpose: Converts raw binary CSV into structured JSON rows
Configuration:

Operation: CSV
Input Binary Field: data

Why: n8n cannot process raw CSV — needs to be converted to JSON first

### Node 4 — 💻 Code Node 1 (Data Formatter)
Purpose: Picks first 10 rows and formats them into clean readable text for AI

### Node 5 — 🌐 HTTP Request (Groq AI)
Purpose: Sends COVID data to Groq AI and gets intelligent analysis
Configuration:

Method: POST
URL: https://api.groq.com/openai/v1/chat/completions
Model: llama-3.3-70b-versatile
Max Tokens: 300

### Node 6 — 💻 Code Node 2 (Report Builder)
Purpose: Extracts AI response and builds formatted report with alert level

### Node 7 — 📝 Write File
Purpose: Saves the final AI report as a text file on local system
Configuration:

Operation: Write File to Disk
File Path: C:\Users\nikhi\.n8n-files\covid_report.txt
Input Binary Field: covid_report

Why: Keeps a local record of every daily report generated

### Node 8 — 💬 Slack
Purpose: Sends the AI report as a Direct Message on Slack
Configuration:

Resource: Message
Operation: Send a Message
Channel: Personal DM (Member ID)
Message: {{ $json.report }}

Why: Delivers report directly to user without needing to check files manually

##  GenAI Usage
Input → Prompt → Output
INPUT (COVID Data Sample):
State: Andaman and Nicobar Islands, Date: 16-01-2021, 
Confirmed: 4,979, Deaths: 62, Recovered: 4,895, Daily Cases: 3
State: Andhra Pradesh, Date: 16-01-2021, 
Confirmed: 885,710, Deaths: 7,190, Recovered: 877,474, Daily Cases: 204

### PROMPT:
System: You are a COVID data analyst. Provide brief summary, 
        key trends and alerts in under 200 words.

User: Analyze this COVID India data: [formatted data]

### OUTPUT (AI Generated):
COVID DASHBOARD REPORT
Generated: 2026-04-22T07:49:33.061Z
Alert Level: 🔴 HIGH ALERT

### AI ANALYSIS:
Based on the COVID data analysis:
- Declining cases observed across most states
- Recovery rate remains high at 91%+
- Alert: Rising cases in select regions
- Continued monitoring recommended

