# n8n Resume Evaluation Workflow

This repository documents an **n8n** automation that evaluates uploaded resumes. It watches a Google Drive folder, downloads the newest PDF, extracts text with **Typhoon OCR**, analyzes it with **Vertex AI (Gemini 2.5-flash)**, parses a JSON result, and appends selected fields to **Google Sheets**.

---

## Overview
In this workflow you need:
- **n8n** installed and running (VM, Docker, or server). 
- Required **APIs & credentials** (Google Drive, Vertex AI, Google Sheets).
- Installed **Typhoon OCR** for text extraction.

---

## VM & Hosting Setup
Personally I used a **Google Cloud VM** for deployment.  
<figure>
  <img alt="GCE instances list" src="https://github.com/user-attachments/assets/f4025af4-6037-49a7-994a-c1882fcb0db7" />
  <figcaption>GCE instances overview</figcaption>
</figure>

<figure>
  <img alt="Create VM - machine type" src="https://github.com/user-attachments/assets/36fb6880-0f8e-4075-93df-f4dbe2c20936" />
  <figcaption>Create VM options</figcaption>
</figure>

<figure>
  <img alt="Firewall / network config" src="https://github.com/user-attachments/assets/1f886a9e-34b4-43d9-852b-819aa1f93182" />
  <figcaption>Firewall / network config</figcaption>
</figure>

<figure>
  <img alt="Startup / boot settings" src="https://github.com/user-attachments/assets/24897c15-22fc-48f0-a734-eead9b9e2fb2" />
  <figcaption>Startup settings</figcaption>
</figure>

<figure>
  <img alt="VM created" src="https://github.com/user-attachments/assets/c2d11419-aac9-4314-8c68-37c2c362c7ac" />
  <figcaption>VM created</figcaption>
</figure>

- Set up the VM engine and connect via SSH.  
<figure>
  <img alt="SSH to VM" src="https://github.com/user-attachments/assets/4b968705-3f1e-4bac-944f-d38f0f67d392" />
  <figcaption>SSH connection</figcaption>
</figure>

- Install dependencies, n8n, and Typhoon OCR (https://docs.opentyphoon.ai/en/ocr/)
- Configure DNS with a free domain from **noip.com** for hosting. (https://my.noip.com/dns/records)  
<figure>
  <img alt="no-ip DNS records" src="https://github.com/user-attachments/assets/fecd436e-52c0-4e4a-9b9e-41ef4b33b2dc" />
  <figcaption>no-ip DNS</figcaption>
</figure>

---

## Automation Workflow Overview
This is the overall automation workflow for resume evaluation.  
<figure>
  <img alt="n8n workflow overview" src="https://github.com/user-attachments/assets/f45c61db-bc8c-4e2c-b056-5243d3835786" />
  <figcaption>n8n workflow overview</figcaption>
</figure>

---

## Google Cloud APIs & Credentials
Before running the workflow:

1) Enable required APIs in your Google Cloud project:
- Drive API  
- Sheets API  
- Vertex AI API  
<figure>
  <img alt="Enable Google APIs" src="https://github.com/user-attachments/assets/b230f5b1-9b58-4933-95e4-08a3359d5a1d" />
  <figcaption>Enable APIs</figcaption>
</figure>

2) Prepare:
- API Key  
- OAuth Client ID  
- Service Account  
<figure>
  <img alt="Credentials dashboard" src="https://github.com/user-attachments/assets/830a9cd7-b3e0-4082-aac0-f8d7be704de2" />
  <figcaption>Credentials dashboard</figcaption>
</figure>

3) Add credentials in **n8n** for Google Drive, Vertex AI, and Google Sheets.  
<figure>
  <img alt="n8n credentials" src="https://github.com/user-attachments/assets/f025e0d7-3e91-4dac-ad81-e368d641cd03" />
  <figcaption>n8n → Credentials</figcaption>
</figure>

---

## Workflow Steps

### 1) Google Drive Trigger
<figure>
  <img alt="Drive Trigger node" src="https://github.com/user-attachments/assets/3df181d6-b984-48bc-9b85-f8846ef8f861" />
  <figcaption>Drive Trigger node</figcaption>
</figure>

- The trigger checks the folder every minute for the **latest uploaded file**.  
<figure>
  <img alt="Trigger configuration" src="https://github.com/user-attachments/assets/ed6d3c7c-d997-429c-a3f0-e822028c3702" />
  <figcaption>Trigger configuration</figcaption>
</figure>

- Make the Drive folder public so resumes can be uploaded.  
<figure>
  <img alt="Drive share settings" src="https://github.com/user-attachments/assets/7dea8b3c-fee1-46a5-8255-4c60633a5068" />
  <figcaption>Share settings</figcaption>
</figure>

- One file per run.

### 2) File Download
<figure>
  <img alt="Download node" src="https://github.com/user-attachments/assets/ea492370-4104-41a3-bbfe-14e1d18ec13e" />
  <figcaption>Download node</figcaption>
</figure>

<figure>
  <img alt="New Resume Uploaded input" src="https://github.com/user-attachments/assets/60f583e6-3b1d-4d65-bbfa-686f2379aad2" />
  <figcaption>Input data from trigger</figcaption>
</figure>

- The workflow downloads the most recent file uploaded to the folder.  
<figure>
  <img alt="File details" src="https://github.com/user-attachments/assets/76a21b39-e293-48d7-b75a-ed65c122d6d3" />
  <figcaption>File details</figcaption>
</figure>

- Confirms successful download in the node output.  
<figure>
  <img alt="Download output" src="https://github.com/user-attachments/assets/4d137626-2733-4cd1-a51c-d482c033ba82" />
  <figcaption>Download output</figcaption>
</figure>

### 3) OCR (Typhoon)
<figure>
  <img alt="OCR node" src="https://github.com/user-attachments/assets/071bddfa-ca44-4eb5-b809-c9fb24da36ae" />
  <figcaption>Typhoon OCR node</figcaption>
</figure>

- Extracts text from PDF using **Typhoon OCR**.  
<figure>
  <img alt="OCR parameters" src="https://github.com/user-attachments/assets/66cace93-5390-46e5-a8be-bce77db47c7f" />
  <figcaption>OCR parameters</figcaption>
</figure>

- Confirms successful extract text from PDF in the node output.  
<figure>
  <img alt="OCR output" src="https://github.com/user-attachments/assets/9f67786a-a138-4846-a961-5a3dbc9efb1e" />
  <figcaption>OCR output</figcaption>
</figure>

- Details of code stored in `code_inside_n8n/ocr`.

### 4) LLM Analysis (Gemini 2.5-flash)
<figure>
  <img alt="LLM analyze node" src="https://github.com/user-attachments/assets/1a8d2a70-fb88-45e1-a1ba-44e8b3de9178" />
  <figcaption>LLM: Analyze Resume with JD</figcaption>
</figure>

<figure>
  <img alt="LLM parameters" src="https://github.com/user-attachments/assets/db2f2840-912d-465d-bfd3-ed7a0a6870b4" />
  <figcaption>Parameters</figcaption>
</figure>

<figure>
  <img alt="LLM input (OCR stdout)" src="https://github.com/user-attachments/assets/e93cf719-9da0-46ec-9590-34ef53e59c39" />
  <figcaption>Input from OCR</figcaption>
</figure>

- Sends OCR text to **Gemini 2.5-flash** for analysis.  
<figure>
  <img alt="LLM helper node" src="https://github.com/user-attachments/assets/5391a76e-5471-420e-a18b-10382cb9a022" />
  <figcaption>LLM Helper</figcaption>
</figure>

<figure>
  <img alt="Vertex AI model config" src="https://github.com/user-attachments/assets/387a6873-1daa-40d2-87ab-298342204db0" />
  <figcaption>Model config</figcaption>
</figure>

- Confirms successful analysis in node output.  
<figure>
  <img alt="LLM output" src="https://github.com/user-attachments/assets/205b3a5f-b0dc-4bc0-aa98-8e089d5512df" />
  <figcaption>LLM output</figcaption>
</figure>

- Prompt stored in `code_inside_n8n/prompts`.

### 5) JSON Parsing
<figure>
  <img alt="Parser node" src="https://github.com/user-attachments/assets/e0aaedb4-840d-407f-95bb-535036f90e7b" />
  <figcaption>Parser node</figcaption>
</figure>

- Parses model output into structured JSON schema (see `code_inside_n8n/schema`).  
<figure>
  <img alt="Schema preview" src="https://github.com/user-attachments/assets/6b1658fa-e336-4cd5-bf73-d9fc89b4593f" />
  <figcaption>Schema preview</figcaption>
</figure>

<figure>
  <img alt="JSON schema in node" src="https://github.com/user-attachments/assets/9f95bec3-b4eb-4268-be25-6042221511b0" />
  <figcaption>JSON schema (full)</figcaption>
</figure>

- Confirms successful JSON parsing in the node output.  
<figure>
  <img alt="Parser output" src="https://github.com/user-attachments/assets/e1665cea-854f-4fea-b30e-a1eb90aa314a" />
  <figcaption>Parser output</figcaption>
</figure>

- Currently extracting only: `name`, `point`, `reason`.  
<figure>
  <img alt="Selected fields" src="https://github.com/user-attachments/assets/3937513e-34ec-49fd-bd43-dda8ec9f96b8" />
  <figcaption>Selected fields</figcaption>
</figure>

### 6) Append to Google Sheets
<figure>
  <img alt="Sheets node" src="https://github.com/user-attachments/assets/437b6cde-92c8-4435-b1e2-460ed06f5511" />
  <figcaption>Google Sheets node</figcaption>
</figure>

- Appends structured results into a **Google Sheet** with specific topic columns.  
<figure>
  <img alt="Append Row parameters" src="https://github.com/user-attachments/assets/2013bcdc-b798-4a13-9849-8bdcfcc548c3" />
  <figcaption>Append Row parameters</figcaption>
</figure>

<figure>
  <img alt="Values to send mapping" src="https://github.com/user-attachments/assets/838d236b-ce93-4c79-87ea-b9d660000e1b" />
  <figcaption>Values to Send mapping</figcaption>
</figure>

- Example: [Google Sheet link](https://docs.google.com/spreadsheets/d/1ggK24tksPicvpG6D4k0iuTKjoBIHJcD76gijGTpmx3A/edit?gid=0#gid=0)  
<figure>
  <img alt="Sheet result" src="https://github.com/user-attachments/assets/dbbe8624-22c1-4f05-a68b-389414dbf7fe" />
  <figcaption>Sheet result</figcaption>
</figure>

---

## Folder Structure
<figure>
  <img alt="Folder structure" src="https://github.com/user-attachments/assets/15e08f24-f7d6-4383-8efe-807f8228161a" />
  <figcaption>Project folders</figcaption>
</figure>

---

## How to Run
1. Upload a resume PDF to the Google Drive folder.  
2. Trigger workflow (automatically every minute or manually with **Execute Workflow**).  
3. Check the outputs step by step:
   - File downloaded  
   - OCR text extracted  
   - LLM analysis completed  
   - JSON parsed  
   - Data appended to Google Sheets  

---

## Notes
- Upload **one file at a time** (workflow is per-file).  
- Make sure the Google Sheet and Drive folder have correct access permissions.  
- API keys and credentials should be stored securely in **n8n Credentials**.  

---

## Troubleshooting
- **Trigger not firing:** Check folder ID and webhook/SSL if using Watch mode.  
- **File not found:** Ensure correct folder permissions.  
- **OCR errors:** Verify Typhoon OCR installation.  
- **LLM errors:** Confirm Vertex AI API enabled and quota available.  
- **Sheets issues:** Ensure sharing and sheet/tab names are correct.  

---

## Security Notes
- Keep API keys and secrets in n8n Credentials only.  
- Use HTTPS and authentication if exposing n8n to the internet.  
