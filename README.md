# Job Application Autopilot 🤖💼

An automated, AI-powered job application processing workflow built on **n8n**.

---

## 🚀 What it does

**Job Application Autopilot** is an n8n automation workflow designed to streamline the job application process from CV submission to follow-up.

The workflow:
- Collects candidate and job information through an n8n form.
- Downloads the candidate's CV from Google Drive.
- Extracts and validates text from the CV PDF.
- Uses **Mistral AI** to compare the CV against the job description.
- Calculates an evidence-based 0–100 fit score.
- Identifies matching and missing skills.
- Generates a personalized cover letter.
- Emails the analysis and cover letter to the candidate via Gmail.
- Logs the application to Google Sheets.
- Creates a Google Calendar follow-up event for 7 days later.
- Provides a fallback email path if the AI analysis or CV extraction fails.

---

## 🚀 How to Use

### Step 1: Import the Workflow
Import `workflow.json` provided in this repository into your n8n instance.

### Step 2: Configure Credentials
Open the relevant nodes and select your own credentials for:
- Google Drive OAuth2
- Mistral Cloud API
- Gmail OAuth2
- Google Sheets OAuth2
- Google Calendar OAuth2

*Note: Do not rely on credential IDs from the exported workflow. Credential references are specific to the original n8n environment.*

### Step 3: Configure Google Sheets
Create or select the spreadsheet where applications should be stored.

Make sure the sheet contains the columns required by the logging node, including:
- `application_id`
- `candidate_name`
- `candidate_email`
- `phone`
- `company`
- `job_title`
- `portfolio_url`
- `cv_link`
- `submitted_at`
- `follow_up_date`
- `fit_score`
- `confidence`
- `fit_summary`
- `matching_skills`
- `missing_skills`
- `reasoning`
- `cover_letter`
- `score_breakdown`
- `ai_status`
- `failure_reason`

### Step 4: Configure Google Drive Access
The applicant must provide a Google Drive CV URL that the configured Google Drive credential can access.

### Step 5: Test the Workflow
Submit a test application containing:
- A real or test CV on Google Drive
- A sample job description
- Candidate details
- Company and position information

Then verify:
- CV downloads correctly from Google Drive.
- CV text is extracted and validated.
- Mistral AI returns structured output.
- Email is delivered via Gmail.
- Application is logged into Google Sheets.
- Calendar follow-up is created for 7 days post-submission.

### Step 6: Activate the Workflow
After testing all integrations, activate the n8n workflow.

---

## ✨ Key Features

### 📝 Application Intake
The workflow starts with an n8n form titled **Job Application Autopilot**.

The form collects:
- **Full Name** — required
- **Email** — required
- **Phone** — optional
- **Company** — required
- **Job Position** — required
- **Job Description** — required (textarea)
- **Portfolio / GitHub URL** — optional
- **CV Link (Google Drive URL)** — required

The form is designed so a candidate can submit their CV and target-job information in a single step.

### 🧠 AI-Powered CV Analysis
The core evaluation is performed by **Mistral AI** (`magistral-small-latest` chat model).

The AI is explicitly instructed to use only information supported by the provided CV and job description. It must not invent skills, experience, projects, certifications, education, achievements, tools, or responsibilities.

#### Scoring Rubric
The final fit score is calculated out of 100 points:

| Criterion | Maximum Score |
| :--- | :---: |
| Core / Required Skills | 30 |
| Relevant Experience | 20 |
| Projects / Practical Work | 15 |
| Education / Degree | 10 |
| Job Responsibilities Match | 10 |
| Certifications | 5 |
| Tools / Technologies | 5 |
| ATS / Important JD Terminology | 5 |
| **Total** | **100** |

The analysis also produces a separate confidence score from 0–100, representing how strongly the evaluation is supported by explicit CV evidence.

#### Evidence Rules
The AI evaluation distinguishes between:
- **Matched**
- **Partial Match**
- **Not Found**

For missing information, the workflow instructs the AI to use:
`Not found in the provided CV.`

Each scoring criterion is expected to contain supporting reasoning and evidence.

### ✉️ Personalized Cover Letter
After evaluating the candidate, the AI generates a personalized professional cover letter (250–350 words).

The successful email delivered to the candidate includes:
- Candidate name
- Company
- Job position
- AI fit score
- Fit summary
- Matching skills
- Missing / weaker areas
- Detailed reasoning
- Personalized cover letter
- Confirmation that the application was logged
- Notice that a 7-day follow-up is scheduled

---

## 🔄 Workflow Architecture

### 🏗️ Workflow Components
The imported n8n workflow contains 15 nodes.

| Node | Purpose |
| :--- | :--- |
| **Job Application Form** | Collects candidate and job information |
| **Prepare Application Data** | Normalizes form data, assigns application ID, and sets follow-up date |
| **Google Drive - Download CV** | Downloads the CV from the supplied Google Drive URL |
| **Extract CV Text** | Extracts readable text from the CV PDF |
| **Validate CV Text** | Validates text length (checks text >= 200 characters) |
| **CV Readable?** | Conditional IF node checking if CV text is valid |
| **AI - Score Fit & Write Cover Letter** | Performs CV/job matching and generates the cover letter |
| **Mistral Chat Model** | Provides the Mistral AI chat model to the AI chain |
| **Analysis Output Parser** | Forces the AI result into a structured JSON format |
| **Combine AI Result** | Combines application data with the successful AI result |
| **Prepare Fallback Data** | Builds fallback application data when CV extraction or AI processing fails |
| **Gmail - Send Cover Letter** | Sends the analysis and generated cover letter to the candidate |
| **Gmail - Fallback Message** | Notifies the candidate that AI analysis is temporarily unavailable |
| **Google Sheets - Log Application** | Appends application information to the tracking spreadsheet |
| **Google Calendar - 7 Day Follow-up** | Creates a 30-minute follow-up calendar event 7 days after submission |

### 🔢 Data Flow

#### 1. Form Submission
The candidate submits their details through the n8n form.

#### 2. Data Preparation
The workflow creates normalized fields including:
- `application_id` (`APP-YYYYMMDD-HHMMSS`)
- `candidate_name`
- `candidate_email`
- `phone`
- `company`
- `job_title`
- `job_description`
- `portfolio_url`
- `cv_link`
- `submitted_at`
- `follow_up_date` (7 days after submission at 10:00 AM)

#### 3. CV Retrieval & Validation
The Google Drive node downloads the CV using the submitted URL. PDF text is extracted and validated for readability (length >= 200 chars).

#### 4. AI Evaluation
Mistral AI receives:
- Candidate CV text
- Job description
- Candidate name
- Company
- Position

It then performs the structured evaluation and cover-letter generation.

#### 5. Structured Output
The Analysis Output Parser expects a structured JSON result containing:
```json
{
  "fit_score": 85,
  "confidence": 92,
  "fit_summary": "...",
  "score_breakdown": {},
  "matching_skills": [],
  "missing_skills": [],
  "evidence": [],
  "reasoning": "...",
  "cover_letter": "..."
}
```

#### 6. Success Path
```
Mistral AI
  ↓
Combine AI Result
  ↓
├── Gmail - Send Cover Letter
├── Google Sheets - Log Application
└── Google Calendar - 7 Day Follow-up
```

#### 7. Failure Path
If CV text extraction or the AI node returns an error:
```
CV / AI Error
    ↓
Prepare Fallback Data
    ↓
├── Gmail - Fallback Message
├── Google Sheets - Log Application
└── Google Calendar - 7 Day Follow-up
```
The candidate is informed that automated AI analysis is temporarily unavailable while their application is recorded for manual review.

---

## 📊 Output Data

The workflow produces application records containing the following fields:

| Field | Description |
| :--- | :--- |
| `application_id` | Unique ID (`APP-YYYYMMDD-HHMMSS`) |
| `candidate_name` | Candidate's full name |
| `candidate_email` | Candidate email address |
| `phone` | Candidate phone number |
| `company` | Target company |
| `job_title` | Target position |
| `portfolio_url` | Portfolio or GitHub URL |
| `cv_link` | Google Drive CV URL |
| `submitted_at` | Application submission timestamp |
| `follow_up_date` | Scheduled follow-up timestamp (7 days later) |
| `fit_score` | AI fit score from 0–100 |
| `confidence` | AI confidence score from 0–100 |
| `fit_summary` | Overall suitability summary |
| `matching_skills` | Skills supported by CV evidence |
| `missing_skills` | Missing or weaker skills |
| `reasoning` | Detailed scoring explanation |
| `cover_letter` | Generated cover letter |
| `score_breakdown` | JSON string of itemized criterion scores |
| `ai_status` | Status (`Success` or `Failed`) |
| `failure_reason` | Error message if processing failed |

---

## 📈 Google Sheets Logging

The workflow uses Google Sheets to maintain an application log.

The application record includes:
- Date & Time (`submitted_at`)
- Candidate Name & Email
- Company & Position
- Fit Score & Confidence Score
- Fit Summary
- Matching Skills & Missing Skills
- Detailed Reasoning & Generated Cover Letter
- Follow-up Date & AI Status

This creates a centralized record of applications and their AI-generated evaluations.

---

## 📅 Automated Follow-Up

A Google Calendar node creates a follow-up event using `follow_up_date`.
- Event Duration: 30 minutes
- Date Calculation: Current submission time + 7 days (scheduled for 10:00 AM)
- Summary: `Follow up: [Job Title] at [Company]`

Every successfully processed application automatically receives a scheduled follow-up reminder.

---

## 🔐 Required Integrations

The workflow depends on the following n8n integrations:

- **Google Drive**: Used to retrieve the candidate CV from the supplied Google Drive URL.
- **Mistral AI**: Used for CV analysis, job matching, fit scoring, evidence generation, missing-skill detection, and cover-letter generation (temperature: 0.3, max tokens: 4000).
- **Gmail**: Used to send successful AI analysis + cover letter emails and fallback notifications.
- **Google Sheets**: Used to log application records into a central tracker.
- **Google Calendar**: Used to create the 7-day follow-up calendar event.

---

## 🛠️ Prerequisites

Before importing and running the workflow, make sure you have:
- n8n instance (Cloud or Self-Hosted)
- Configured Google Drive OAuth2 credential
- Configured Mistral Cloud API credential
- Configured Gmail OAuth2 credential
- Configured Google Sheets OAuth2 credential
- Configured Google Calendar OAuth2 credential
- A Google Sheet prepared for application logging
- A Google Calendar available for follow-up scheduling
- CVs accessible through Google Drive links

---

## 🧩 AI Prompt Design

The AI agent follows an evidence-first evaluation strategy.

Important constraints include:
- Use only explicit CV and job-description information.
- Never invent candidate qualifications.
- Do not assume knowledge of related technologies.
- Do not award points for unsupported claims.
- Separate exact matches, partial matches, and missing requirements.
- Ensure the final score equals the sum of the rubric criteria (max 100).
- Keep confidence score separate from fit score.
- Generate a personalized cover letter without inventing information.

---

## ⚠️ Important Configuration Notes

The exported workflow contains environment-specific n8n credential references and Google resource IDs. When importing it into another n8n instance, review every integration node and select the appropriate credentials/resources for your environment.

Verify your Google Sheets column mappings and target calendar ID before enabling the workflow in production.

---

## 🔒 Privacy & Security

This workflow processes sensitive candidate information (name, contact info, CV text, job details).

Before deploying for live applications:
- Ensure proper access permissions on Google Drive & Sheets.
- Restrict access to candidate CV files.
- Avoid sharing workflow exports containing private credential IDs.
- Review applicable data-privacy regulations (e.g., GDPR) regarding automated processing and AI evaluations.

---

## 🧪 Testing Checklist

Before production deployment, verify:
- [x] Form accepts all required fields.
- [x] Invalid email addresses are rejected appropriately.
- [x] Google Drive CV downloads successfully.
- [x] PDF text extraction & validation work.
- [x] Mistral AI credentials are valid.
- [x] Mistral AI returns valid structured JSON.
- [x] Fit score is between 0 and 100.
- [x] Score components total 100 maximum.
- [x] Matching and missing skills are populated correctly.
- [x] Cover letter is generated cleanly.
- [x] Success email is delivered via Gmail.
- [x] Fallback email is delivered when processing fails.
- [x] Google Sheets receives all intended columns.
- [x] Google Calendar creates the follow-up event 7 days later.
- [x] Production credentials replace environment-specific references.

---

## 🗺️ End-to-End Workflow

```
Candidate
   │
   ▼
Job Application Form
   │
   ▼
Prepare Application Data
   │
   ▼
Download CV from Google Drive
   │
   ▼
Extract CV Text & Validate
   │
   ▼
Mistral AI Analysis
   │
   ├─────────────── Success ───────────────┐
   │                                       │
   ▼                                       ▼
Combine AI Result                    Prepare Fallback Data
   │                                       │
   ├──► Gmail: Send Cover Letter           ├──► Gmail: Fallback Notice
   ├──► Google Sheets: Log Application     ├──► Google Sheets: Log Application
   └──► Google Calendar: 7-Day Event       └──► Google Calendar: 7-Day Event
```

---

## 📁 Workflow File

The n8n workflow can be imported directly from:
- `workflow.json`

The workflow contains 15 nodes and uses n8n's standard workflow connection structure.

---

## 💡 Future Improvements

Potential improvements for a production-ready version include:
- Add automatic application status tracking (e.g. Applied, Interviewing, Rejected).
- Add Slack or Microsoft Teams notifications for instant team alerts.
- Add Gmail labels for processed application emails.
- Add duplicate-application detection based on candidate email.
- Add a recruiter dashboard for application analytics.

---

## 📄 License

No explicit open-source license is defined in the provided workflow export. Add a `LICENSE` file if you plan to share this repository publicly.
