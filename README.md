# 🤖 COVID-19 Automated Risk Alert & Reporting System
### n8n + Groq GenAI Automation Workflow

---

## 📌 Automation Problem Statement

Manual analysis of COVID-19 data is time-consuming and requires constant human intervention. Delays in identifying risk conditions like rising cases, high positivity rates, or low vaccination coverage can lead to poor decision-making.

**The Problem:**
- COVID data needs daily monitoring across multiple states
- Manually reading CSV and writing summaries takes significant time
- Risk alerts for rising cases can be missed without automation
- No automated reporting or alert system exists for the team

**The Solution:**
- n8n reads COVID CSV automatically every day at 9AM IST
- Groq AI (LLaMA 3.3-70b) analyzes the data and generates a summary
- IF/ELSE logic detects risk conditions automatically
- 🔴 HIGH ALERT or 🟢 NORMAL status sent to Slack instantly
- Report saved as text file for record keeping
- Zero manual intervention required

---

## 🔧 Tech Stack

| Tool | Purpose |
|------|---------|
| n8n (Self Hosted) | Workflow Automation |
| Groq AI (LLaMA 3.3-70b) | GenAI Summary & Analysis |
| Slack API | Alert & Report Delivery |
| COVID-19 CSV Dataset | Data Source |

---

## 🔄 n8n Workflow Steps

### Complete Workflow Diagram:
```
Schedule Trigger (9AM IST)
→ Read File From Disk
→ Extract From File (CSV Parser)
→ Code Node 1 (Data Formatter)
→ HTTP Request (Groq AI API)
→ Code Node 2 (Report Builder)
→ Code Node 3 (Risk Calculator)
→ IF Node (Risk Condition Check)
   ├── TRUE  (HIGH Risk) → Slack 🔴 → Write File
   └── FALSE (LOW Risk)  → Slack 🟢 → Write File
```

---

### Node 1 — 🕐 Schedule Trigger
**Purpose:** Automatically starts the workflow every day at 9AM IST

**Configuration:**
```
Trigger:  Every Day
Time:     09:00 AM
Timezone: Asia/Kolkata
```

**Why:** Ensures daily automated reporting without any manual trigger

---

### Node 2 — 📁 Read File From Disk
**Purpose:** Opens and reads the COVID CSV file from local storage

**Configuration:**
```
Operation: Read File
File Path: C:\Users\nikhi\.n8n-files\covid_eda_task4.csv
```

**Why:** Fetches raw data that will be analyzed by AI

---

### Node 3 — 📊 Extract From File
**Purpose:** Converts raw binary CSV into structured JSON rows

**Configuration:**
```
Operation:          CSV
Input Binary Field: data
```

**Why:** n8n cannot process raw CSV — needs to be converted to JSON first

---

### Node 4 — 💻 Code Node 1 (Data Formatter)
**Purpose:** Picks first 10 rows and formats them into clean text for AI

**Code:**
```javascript
const rows = $input.all();
let summaryText = "Analyze this COVID India data: ";

for(let i = 0; i < rows.length && i < 10; i++) {
  const row = rows[i];
  if(row && row.json) {
    const data = row.json;
    summaryText += `State: ${data.State}, Date: ${data.Date}, 
    Confirmed: ${data.Confirmed}, Deaths: ${data.Deaths}, 
    Recovered: ${data.Cured}, Daily Cases: ${data['Daily New Cases']}. `;
  }
}

return [{ json: { summaryText: summaryText.trim() } }];
```

**Why:** AI needs clean formatted text — not raw JSON objects

---

### Node 5 — 🌐 HTTP Request (Groq AI)
**Purpose:** Sends COVID data to Groq AI and gets intelligent analysis

**Configuration:**
```
Method: POST
URL:    https://api.groq.com/openai/v1/chat/completions
Model:  llama-3.3-70b-versatile
Max Tokens: 300
```

**Prompt Structure:**
```
System: "You are a COVID data analyst. Provide brief summary, 
         key trends and alerts in under 200 words."

User:   [Formatted COVID data from Code Node 1]
```

**Why:** Groq AI is free, fast and produces accurate analytical summaries

---

### Node 6 — 💻 Code Node 2 (Report Builder)
**Purpose:** Extracts AI response and builds formatted report

**Code:**
```javascript
const response = $input.first().json;
const aiSummary = response.choices[0].message.content;
const timestamp = new Date().toISOString();

const alertLevel = aiSummary.toLowerCase().includes('rising') || 
                   aiSummary.toLowerCase().includes('alert') ? 
                   '🔴 HIGH ALERT' : '🟢 Normal';

const report = `COVID DASHBOARD REPORT
Generated: ${timestamp}
Alert Level: ${alertLevel}

AI ANALYSIS:
${aiSummary}`;

const base64Report = Buffer.from(report, 'utf8').toString('base64');

return [{ 
  json: { report, aiSummary, alertLevel },
  binary: {
    'covid_report': {
      data: base64Report,
      mimeType: 'text/plain',
      fileExtension: 'txt',
      fileName: 'covid_report.txt'
    }
  }
}];
```

**Why:** Formats AI response into professional report with alert detection

---

### Node 7 — 💻 Code Node 3 (Risk Calculator)
**Purpose:** Applies conditional risk logic based on 3 risk conditions

**Risk Conditions Monitored:**
```
Condition 1: Daily cases rising     (rising/increase keywords in AI output)
Condition 2: High positivity rate   (positivity/positive rate keywords)
Condition 3: Low vaccination        (vaccination + low keywords combined)
             + rising cases
```

**Code:**
```javascript
const reportData = $input.first().json;

const aiSummary = reportData.aiSummary ? 
                  reportData.aiSummary.toLowerCase() : 
                  reportData.report ? 
                  reportData.report.toLowerCase() : '';

const risingCases = aiSummary.includes('rising') || 
                    aiSummary.includes('increase');

const highPositivity = aiSummary.includes('positivity') || 
                       aiSummary.includes('positive rate');

const lowVaccination = aiSummary.includes('vaccination') && 
                       aiSummary.includes('low');

const overallRisk = (risingCases || highPositivity || lowVaccination) ? 
                    'HIGH' : 'LOW';

return [{ 
  json: { 
    ...reportData,
    riskConditions: {
      risingCases,
      highPositivity,
      lowVaccination,
      overallRisk
    }
  }
}];
```

**Why:** Automates risk detection without any human review

---

### Node 8 — ⚠️ IF Node (Conditional Logic)
**Purpose:** Routes workflow based on detected risk level

**Condition:**
```
Value 1:  {{ $json.riskConditions.overallRisk }}
Operator: Equal
Value 2:  HIGH
```

```
TRUE  → Risk detected   → Send 🔴 HIGH ALERT to Slack
FALSE → No risk found   → Send 🟢 NORMAL status to Slack
```

**Why:** Ensures different messages sent based on actual risk conditions automatically

---

### Node 9a — 💬 Slack (TRUE Branch — HIGH ALERT)
**Purpose:** Sends urgent risk alert message to Slack DM

**Message Sent:**
```
🚨 RISK CONDITIONS DETECTED!
Rising Cases:    true/false
High Positivity: true/false
Low Vaccination: true/false
🔴 IMMEDIATE ATTENTION REQUIRED!
+ Full AI Report
```

---

### Node 9b — 💬 Slack (FALSE Branch — NORMAL)
**Purpose:** Sends normal daily status report to Slack DM

**Message Sent:**
```
✅ COVID RISK STATUS: NORMAL
🟢 No immediate action required.
+ Full AI Report
```

---

### Node 10 — 📝 Write File
**Purpose:** Saves the final AI report as a text file on local system

**Configuration:**
```
Operation:          Write File to Disk
File Path:          C:\Users\nikhi\.n8n-files\covid_report.txt
Input Binary Field: covid_report
```

**Why:** Keeps permanent local record of every daily report generated

---

## 🤖 GenAI Usage

### Input → Prompt → Output Flow:

**INPUT (Formatted CSV Data):**
```
State: Andaman and Nicobar Islands, Date: 16-01-2021,
Confirmed: 4,979, Deaths: 62, Recovered: 4,895, Daily Cases: 3
State: Andhra Pradesh, Date: 16-01-2021,
Confirmed: 885,710, Deaths: 7,190, Recovered: 877,474, Daily Cases: 204
```

**PROMPT:**
```
System: You are a COVID data analyst. Provide brief summary,
        key trends and alerts in under 200 words.

User:   Analyze this COVID India data: [formatted data above]
```

**OUTPUT (AI Generated):**
```
COVID DASHBOARD REPORT
Generated: 2026-04-22T07:49:33.061Z
Alert Level: 🔴 HIGH ALERT

AI ANALYSIS:
Based on COVID data analysis:
- Declining cases observed across most states
- Recovery rate remains high at 91%+
- Alert: Rising cases detected in select regions
- Continued monitoring recommended
```

**Risk Logic Applied:**
```
Rising Cases:    TRUE  ← AI mentioned "rising"
High Positivity: FALSE
Low Vaccination: FALSE
Overall Risk:    HIGH  → 🔴 Alert sent to Slack!
```

---

## 📸 Screenshots

### 1. Complete n8n Workflow
![n8n Workflow](screenshots/n8n_workflow.png)

### 2. Code Node 1 Output — Formatted Data
![Code Node 1](screenshots/code_node1_output.png)

### 5. Generated Text Report
![Text Report](screenshots/text_report.png)

### 6. Slack Alert Message Received
![Slack Message](screenshots/slack_message.png)


## ❓ Theoretical Questions & Answers

### Q1. What is n8n and why use it?
n8n is an open source workflow automation tool that connects different apps and services without heavy coding. We used it because it is free, self-hosted, supports custom JavaScript nodes, and easily integrates with AI APIs like Groq.

### Q2. What is GenAI and how is it used here?
Generative AI refers to AI models that generate human-like text. Here we used Groq's LLaMA 3.3-70b model to automatically analyze COVID data and generate summaries, trend analysis and risk alerts without human intervention.

### Q3. Why Groq AI over OpenAI?
```
✅ Completely free to use
✅ Faster response times
✅ LLaMA 3.3-70b is highly capable
✅ No credit card required
✅ Perfect for student projects
```

### Q4. How does the IF/ELSE risk alert system work?
The Risk Calculator (Code Node 3) scans the AI generated summary for keywords like "rising", "increase", "positivity", and "low vaccination". If any risk condition is detected, the overall risk is marked HIGH. The IF node then routes the workflow to either a HIGH ALERT Slack message or a NORMAL status message automatically.

### Q5. What are the 3 risk conditions monitored?
```
1. Rising Cases:     Daily cases increasing detected in AI summary
2. High Positivity:  Positive test rate mentioned as concerning
3. Low Vaccination   Both low vaccination AND rising cases
   + Rising Cases:   detected simultaneously
```

### Q6. How would this work in real production?
In production the CSV would be updated daily by a data pipeline pulling from government health APIs. n8n would process the latest data, generate AI reports, and send alerts to the entire team Slack channel. The system would scale to monitor all states independently with state-wise risk scoring.

### Q7. What is the connection between Power BI and n8n?
Both tools use the same COVID CSV dataset. Power BI provides interactive visual analysis for human exploration while n8n automates daily AI report generation and risk alerting. Together they form a complete end-to-end analytics and automated reporting pipeline.

### Q8. Why save the report as text file AND send to Slack?
```
Text File → Permanent record keeping and audit trail
Slack     → Instant notification and team awareness
Both      → Complete reporting solution
```

---

## 🚀 How to Run This Project

### Prerequisites:
```
Node.js v18+
n8n: npm install -g n8n
Groq API key: console.groq.com
Slack Bot Token: api.slack.com
```

### Setup Steps:
```
1. Clone this repository
2. Copy CSV to: C:\Users\{username}\.n8n-files\
3. Start n8n: n8n start
4. Open: localhost:5678
5. Import: workflow/covid_workflow.json
6. Update file path in Read File node
7. Add Groq API key in HTTP Request headers
8. Add Slack token in Slack node credentials
9. Activate workflow toggle ✅
```
> Reports generate automatically every day at 9AM IST!

---

