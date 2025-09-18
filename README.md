# -n8n Resume Evaluation- by Chottanut Wannasri

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

<figure style="text-align:center;">
  <figcaption><strong>GCE instances overview</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/f4025af4-6037-49a7-994a-c1882fcb0db7" alt="GCE instances list" width="900"></p>
</figure>

<figure style="text-align:center;">
  <figcaption><strong>Create VM options</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/36fb6880-0f8e-4075-93df-f4dbe2c20936" alt="Create VM - machine type" width="900"></p>
</figure>

<figure style="text-align:center;">
  <figcaption><strong>Firewall / network config</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/1f886a9e-34b4-43d9-852b-819aa1f93182" alt="Firewall / network config" width="900"></p>
</figure>

<figure style="text-align:center;">
  <figcaption><strong>Startup settings</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/24897c15-22fc-48f0-a734-eead9b9e2fb2" alt="Startup / boot settings" width="900"></p>
</figure>

<figure style="text-align:center;">
  <figcaption><strong>VM created</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/c2d11419-aac9-4314-8c68-37c2c362c7ac" alt="VM created" width="900"></p>
</figure>

- Set up the VM engine and connect via SSH.  

<figure style="text-align:center;">
  <figcaption><strong>SSH connection</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/4b968705-3f1e-4bac-944f-d38f0f67d392" alt="SSH to VM" width="900"></p>
</figure>

- Install dependencies, n8n, and Typhoon OCR (https://docs.opentyphoon.ai/en/ocr/)
- Configure DNS with a free domain from **noip.com** for hosting. (https://my.noip.com/dns/records)  

<figure style="text-align:center;">
  <figcaption><strong>no-ip DNS</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/fecd436e-52c0-4e4a-9b9e-41ef4b33b2dc" alt="no-ip DNS records" width="900"></p>
</figure>

---

## Automation Workflow Overview
This is the overall automation workflow for resume evaluation.  

<figure style="text-align:center;">
  <figcaption><strong>n8n workflow overview</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/f45c61db-bc8c-4e2c-b056-5243d3835786" alt="n8n workflow overview" width="900"></p>
</figure>

---

## Google Cloud APIs & Credentials
Before running the workflow:

1) Enable required APIs in your Google Cloud project:
- Drive API  
- Sheets API  
- Vertex AI API  

<figure style="text-align:center;">
  <figcaption><strong>Enable APIs</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/b230f5b1-9b58-4933-95e4-08a3359d5a1d" alt="Enable Google APIs" width="900"></p>
</figure>

2) Prepare:
- API Key  
- OAuth Client ID  
- Service Account  

<figure style="text-align:center;">
  <figcaption><strong>Credentials dashboard</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/830a9cd7-b3e0-4082-aac0-f8d7be704de2" alt="Credentials dashboard" width="900"></p>
</figure>

3) Add credentials in **n8n** for Google Drive, Vertex AI, and Google Sheets.  

<figure style="text-align:center;">
  <figcaption><strong>n8n → Credentials</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/f025e0d7-3e91-4dac-ad81-e368d641cd03" alt="n8n credentials" width="900"></p>
</figure>

---

## Workflow Steps

### 1) Google Drive Trigger

<figure style="text-align:center;">
  <figcaption><strong>Drive Trigger node</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/3df181d6-b984-48bc-9b85-f8846ef8f861" alt="Drive Trigger node" width="900"></p>
</figure>

- The trigger checks the folder every minute for the **latest uploaded file**.  

<figure style="text-align:center;">
  <figcaption><strong>Trigger configuration</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/ed6d3c7c-d997-429c-a3f0-e822028c3702" alt="Trigger configuration" width="900"></p>
</figure>

- Make the Drive folder public so resumes can be uploaded.  

<figure style="text-align:center;">
  <figcaption><strong>Share settings</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/7dea8b3c-fee1-46a5-8255-4c60633a5068" alt="Drive share settings" width="900"></p>
</figure>

- One file per run.

### 2) File Download

<figure style="text-align:center;">
  <figcaption><strong>Download node</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/ea492370-4104-41a3-bbfe-14e1d18ec13e" alt="Download node" width="900"></p>
</figure>

<figure style="text-align:center;">
  <figcaption><strong>Input data from trigger</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/60f583e6-3b1d-4d65-bbfa-686f2379aad2" alt="New Resume Uploaded input" width="900"></p>
</figure>

- The workflow downloads the most recent file uploaded to the folder.  

<figure style="text-align:center;">
  <figcaption><strong>File details</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/76a21b39-e293-48d7-b75a-ed65c122d6d3" alt="File details" width="900"></p>
</figure>

- Confirms successful download in the node output.  

<figure style="text-align:center;">
  <figcaption><strong>Download output</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/4d137626-2733-4cd1-a51c-d482c033ba82" alt="Download output" width="900"></p>
</figure>

### 3) OCR (Typhoon)

<figure style="text-align:center;">
  <figcaption><strong>Typhoon OCR node</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/071bddfa-ca44-4eb5-b809-c9fb24da36ae" alt="OCR node" width="900"></p>
</figure>

- Extracts text from PDF using **Typhoon OCR**.  

<figure style="text-align:center;">
  <figcaption><strong>OCR parameters</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/66cace93-5390-46e5-a8be-bce77db47c7f" alt="OCR parameters" width="900"></p>
</figure>

- Confirms successful extract text from PDF in the node output.  

<figure style="text-align:center;">
  <figcaption><strong>OCR output</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/9f67786a-a138-4846-a961-5a3dbc9efb1e" alt="OCR output" width="900"></p>
</figure>

- Details of code stored in `code_inside_n8n.txt`.

### 4) LLM Analysis (Gemini 2.5-flash)

<figure style="text-align:center;">
  <figcaption><strong>LLM: Analyze Resume with JD</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/1a8d2a70-fb88-45e1-a1ba-44e8b3de9178" alt="LLM analyze node" width="900"></p>
</figure>

<figure style="text-align:center;">
  <figcaption><strong>Parameters</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/db2f2840-912d-465d-bfd3-ed7a0a6870b4" alt="LLM parameters" width="900"></p>
</figure>

<figure style="text-align:center;">
  <figcaption><strong>Input from OCR</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/e93cf719-9da0-46ec-9590-34ef53e59c39" alt="LLM input (OCR stdout)" width="900"></p>
</figure>

- Sends OCR text to **Gemini 2.5-flash** for analysis.  

<figure style="text-align:center;">
  <figcaption><strong>LLM Helper</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/5391a76e-5471-420e-a18b-10382cb9a022" alt="LLM helper node" width="900"></p>
</figure>

<figure style="text-align:center;">
  <figcaption><strong>Model config</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/387a6873-1daa-40d2-87ab-298342204db0" alt="Vertex AI model config" width="900"></p>
</figure>

- Confirms successful analysis in node output.  

<figure style="text-align:center;">
  <figcaption><strong>LLM output</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/205b3a5f-b0dc-4bc0-aa98-8e089d5512df" alt="LLM output" width="900"></p>
</figure>

- Prompt stored in `code_inside_n8n.txt`.

### 5) JSON Parsing

<figure style="text-align:center;">
  <figcaption><strong>Parser node</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/e0aaedb4-840d-407f-95bb-535036f90e7b" alt="Parser node" width="900"></p>
</figure>

- Parses model output into structured JSON schema (see `code_inside_n8n.txt`).  

<figure style="text-align:center;">
  <figcaption><strong>Schema preview</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/6b1658fa-e336-4cd5-bf73-d9fc89b4593f" alt="Schema preview" width="900"></p>
</figure>

<figure style="text-align:center;">
  <figcaption><strong>JSON schema (full)</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/9f95bec3-b4eb-4268-be25-6042221511b0" alt="JSON schema in node" width="900"></p>
</figure>

- Confirms successful JSON parsing in the node output.  

<figure style="text-align:center;">
  <figcaption><strong>Parser output</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/e1665cea-854f-4fea-b30e-a1eb90aa314a" alt="Parser output" width="900"></p>
</figure>

- Currently extracting only: `name`, `point`, `reason`.  

<figure style="text-align:center;">
  <figcaption><strong>Selected fields</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/3937513e-34ec-49fd-bd43-dda8ec9f96b8" alt="Selected fields" width="900"></p>
</figure>

### 6) Append to Google Sheets

<figure style="text-align:center;">
  <figcaption><strong>Google Sheets node</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/437b6cde-92c8-4435-b1e2-460ed06f5511" alt="Sheets node" width="900"></p>
</figure>

- Appends structured results into a **Google Sheet** with specific topic columns.  

<figure style="text-align:center;">
  <figcaption><strong>Append Row parameters</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/2013bcdc-b798-4a13-9849-8bdcfcc548c3" alt="Append Row parameters" width="900"></p>
</figure>

<figure style="text-align:center;">
  <figcaption><strong>Values to Send mapping</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/838d236b-ce93-4c79-87ea-b9d660000e1b" alt="Values to send mapping" width="900"></p>
</figure>

- Example: [Google Sheet link](https://docs.google.com/spreadsheets/d/1ggK24tksPicvpG6D4k0iuTKjoBIHJcD76gijGTpmx3A/edit?gid=0#gid=0)  

<figure style="text-align:center;">
  <figcaption><strong>Sheet result</strong></figcaption>
  <p align="center"><img src="https://github.com/user-attachments/assets/dbbe8624-22c1-4f05-a68b-389414dbf7fe" alt="Sheet result" width="900"></p>
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
