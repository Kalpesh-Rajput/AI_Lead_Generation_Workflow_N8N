
# 🤖 AI Lead Generation & Outreach Workflow using n8n  

### 🚀 End-to-End Intelligent Automation (Apify + Hugging Face + Google Sheets)

---

<img width="1700" height="509" alt="N8N" src="https://github.com/user-attachments/assets/ec8e08ea-507b-46fd-b8c8-e6b420fc2da4" />


> **Automate your lead generation, website summarization, and outreach emails using AI — all inside n8n.**

---

## 🧠 Project Overview

This project demonstrates how to build a **fully automated lead generation and outreach system** using **n8n**, integrating multiple APIs:
- **Apify** for company lead scraping  
- **Hugging Face** for summarizing websites and generating outreach emails  
- **Google Sheets** for storing and tracking all generated leads  

The workflow automatically:
1. Fetches business leads via API  
2. Cleans & extracts domain names  
3. Scrapes and summarizes company websites  
4. Generates personalized outreach emails using AI  
5. Stores everything neatly in Google Sheets  

---

## 🧰 Tech Stack & Tools Used

| Tool | Purpose | Description |
|------|----------|-------------|
| 🧩 **n8n** | Workflow Automation | Visual automation platform to connect APIs without coding |
| 🌐 **Apify API** | Lead Data Extraction | Fetches company and website data programmatically |
| 🤖 **Hugging Face Inference API** | AI Text Generation | Summarizes websites and generates personalized outreach emails |
| 📊 **Google Sheets API** | Data Storage | Saves lead info, summaries, and emails |
| 🐳 **Docker** | Environment Setup | Runs n8n locally in a container for stability |

---

## ⚙️ Project Setup (Step-by-Step)

### 🪜 1. Install Docker
Download and install Docker Desktop from:  
🔗 [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)

Ensure it’s running before continuing.

---

### 🪜 2. Run n8n in Docker

In **Command Prompt (CMD)**, run the following commands:

```bash
docker volume create n8n_data
````

Then start n8n:

```bash
docker run -d ^
  --name n8n ^
  -p 5678:5678 ^
  -e N8N_HOST=localhost ^
  -e N8N_PORT=5678 ^
  -e N8N_PROTOCOL=http ^
  -v "%UserProfile%\.n8n:/home/node/.n8n" ^
  n8nio/n8n:latest
```

✅ n8n will be available at:
👉 [[http://localhost:5678](http://localhost:5678](http://localhost:5678/workflow/wGwzZssdikXbzV2z)

---

### 🪜 3. Open the n8n Editor

* Visit the URL above.
* Create a new workflow and start connecting your nodes.

---

## 🧩 Workflow Architecture

The workflow follows **five main stages**:

---

### 🧠 Step 1 — Lead Data Ingestion (Apify API)

**Goal:** Fetch company leads (100–500) from Apify.

* Used the **Apify Dataset API** with Dataset ID.
* Retrieved fields like `company_name`, `domain`, `email`, and `website`.
* Stored output in Google Sheets using the **Google Sheet Append node**.

<img width="1857" height="802" alt="N8N2" src="https://github.com/user-attachments/assets/469319ba-4213-4fa4-8b01-e2ea2d9d875f" />

---
### 🗂️ Step 2 — Map Fields (Edit Fields)

**Goal:** To prepare and rename incoming data fields from Apify so that the next nodes (Domain Extractor, Hugging Face Summary, etc.) can use clean and consistent column names.

**Node Used:** 🧩 *Set / Edit Fields Node* in n8n

<img width="1857" height="802" alt="N8N2" src="https://github.com/user-attachments/assets/9d7d27ed-4dc6-492c-883c-9725888ee149" />


---

### 🌐 Step 3 — Domain Extraction

**Goal:** Identify or extract the domain name for each lead.

* Used **JavaScript Function node** to:

  * Check if domain exists
  * If missing, extract from email or website
  * Log missing domains into a separate “Error Log” tab

<img width="1905" height="852" alt="N8N4" src="https://github.com/user-attachments/assets/c2e5411b-3100-4b3f-9dc4-4d9d8a1742b1" />


---

### 🧾 Step 4 — Website Summarization (Hugging Face)

**Goal:** Summarize company website content in 2–3 lines.

* API: `facebook/bart-large-cnn` (via Hugging Face Inference API)

* Model prompt example:

  ```json
  {
    "inputs": "Summarize this company website content in 2–3 sentences: {{ $json.website }}"
  }
  ```

* Output field: `summary_text`

<img width="1903" height="875" alt="N8N5" src="https://github.com/user-attachments/assets/770888d0-39f5-412c-b574-80993b0089b0" />


---

### ✂️ Step 5 — Clean the Summary

**Goal:** Clean and format AI summaries for readability.

**Node:** Code (JavaScript)

**Code Used:**

```js
const items = $input.all();
return items.map(item => {
  let summary = item.json.summary_text || "";
  summary = summary
    .replace(/https?:\/\/[^\s]+/g, "")
    .replace(/www\.[^\s]+/g, "")
    .replace(/\s+/g, " ")
    .trim();

  if (summary.length > 0) {
    summary = summary.charAt(0).toUpperCase() + summary.slice(1);
  }

  return { json: { ...item.json, clean_summary: summary || "No clear summary available" } };
});
```

<img width="1905" height="855" alt="N8N6" src="https://github.com/user-attachments/assets/788c874a-52f7-4b22-8b33-a910b6ba4250" />


---

### 💌 Step 6 — Personalized Email Generation (Hugging Face)

**Goal:** Generate outreach emails automatically.

**Model:** `mistralai/Mistral-7B-Instruct-v0.2`
**Prompt Used:**

```json
{
  "inputs": "You are an SDR for E2M Solutions. Write a warm 2–3 sentence outreach email for {{ $json.company_name }} (domain: {{ $json.domain }}). Use this summary: {{ $json.clean_summary }}."
}
```

Output: `generated_text` (email body)

<img width="1890" height="822" alt="N8N7" src="https://github.com/user-attachments/assets/ee5b1a64-1b75-4836-915c-57b7e88da609" />

---

### 📊 Step 7 — Save Data in Google Sheets

**Goal:** Store final output.

* Used **Google Sheets Append Node**
* Saved columns:

  * `Company Name`
  * `Domain`
  * `Summary`
  * `Generated Email`
* Sheet auto-updates every time new data is processed.

<img width="1874" height="782" alt="N8N8" src="https://github.com/user-attachments/assets/e3368553-ba78-4b86-b470-fffb56c63409" />


---

## 🧾 Deliverables

| File                                      | Description                                 |
| ----------------------------------------- | ------------------------------------------- |
| `AI_Lead_Generation_Workflow.json`        | Exported n8n workflow                       |
| `API_Pricing_Justification_Document.docx` | Free API comparison & pricing justification |
| `Kalpesh_AI_Intern_Walkthrough.mp4`       | 3–6 min workflow demo video                 |
| `Google_Sheet_Link.txt`                   | Final lead data stored in Sheets            |


---

## 💰 API Pricing & Justification (Summary)

| API           | Type | Cost | Why Used                                                 |
| ------------- | ---- | ---- | -------------------------------------------------------- |
| Apify         | Free | $0   | Free dataset access for leads                            |
| Hugging Face  | Free | $0   | Open models like Bart & Mistral for summaries and emails |
| Google Sheets | Free | $0   | Cloud storage for all data                               |
| Docker        | Free | $0   | Run n8n locally                                          |

---

## 💡 Future Enhancements

* Integrate with **CRM tools** (HubSpot / Zoho)
* Auto-send outreach emails via **Gmail API**
* Add analytics for open rate tracking
* Host workflow on **n8n Cloud or Railway**

---


## 🏁 Conclusion

This project demonstrates the power of combining **AI + Automation** using free tools.
From lead extraction to smart email generation — everything runs hands-free.

> ✨ “Small automations today can create big business impact tomorrow.”



