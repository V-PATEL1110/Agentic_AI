# 🤖 Agent 1: AI Customer Support Gmail Agent

An automated n8n workflow that monitors unread emails, analyzes customer inquiries using Google Gemini 3.1 Flash Lite, and automatically sends polite, professional support replies via Gmail.

---

## 🏗️ Workflow Architecture

<details>
<summary>Click to Expand / View Code</summary>

```text
[Gmail Trigger] ──► [AI Agent Node] ──► [Gmail Tool (Reply)]
                          ▲
                          │ (Chat Model)
              [Google Gemini Chat Model]
```

</details>

---

## 🔑 Prerequisite: Google OAuth 2.0 Setup Guide

<details>
<summary>🔑 Prerequisite: Google OAuth 2.0 Setup Guide</summary>

### Creating Google OAuth Client ID & Client Secret

#### Step 1: Open Google Cloud Console
Go to: [https://console.cloud.google.com/](https://console.cloud.google.com/)

#### Step 2: Create a New Project
1. Click the Project Selector at the top.
2. Click **New Project**.
3. Enter a project name (e.g., `Neuralize Gmail Agent`).
4. Click **Create**.
5. Select the newly created project.

#### Step 3: Enable Gmail API
1. Open the left sidebar.
2. Go to: **APIs & Services** ➔ **Library**.
3. Search for **Gmail API**.
4. Open **Gmail API** and click **Enable**.

#### Step 4: Configure OAuth Consent Screen
1. Go to: **APIs & Services** ➔ **OAuth consent screen**.
2. Click **Get Started**.
3. Fill in the details:
   - **App Name**: `Neuralize AI Agent`
   - **User Support Email**: Select your Gmail account.
   - **Developer Contact Email**: Select your Gmail account.
4. Click **Next** until completed.

#### Step 5: Add Test Users
1. Go to **Audience** ➔ Choose **External**.
2. Go to **Test Users** ➔ Click **Add Users**.
3. Add the Gmail account that will be used in n8n (e.g., `yourname@gmail.com`).
4. Click **Save**.

#### Step 6: Configure Data Access
1. Go to **Data Access** ➔ Click **Add or Remove Scopes**.
2. Search and add these scopes:
   - `Gmail API` ➔ `.../auth/gmail.modify`
   - `Gmail API` ➔ `.../auth/gmail.send`
   *(These allow the workflow to read emails and send replies.)*
3. Click **Update**, then click **Save**.

#### Step 7: Create OAuth Client
1. Go to: **APIs & Services** ➔ **Credentials**.
2. Click **+ Create Credentials** ➔ Choose **OAuth Client ID**.

#### Step 8: Select Application Type
Choose **Web application**.

#### Step 9: Give It a Name
Example: `n8n Gmail OAuth`

#### Step 10: Add Authorized Redirect URI
1. Copy the Redirect URL shown by n8n in the Gmail credential window.
   - Example: `https://your-n8n-instance/rest/oauth2-credential/callback`
   - Or for n8n Cloud: `https://xxxxx.app.n8n.cloud/rest/oauth2-credential/callback`
2. Paste it under **Authorized Redirect URIs**.
3. Click **Create**.

#### Step 11: Copy Credentials
Google will display:
- **Client ID**
- **Client Secret**

Copy both credentials.

#### Step 12: Connect in n8n
1. In n8n, open the Gmail Credential window.
2. Paste the **Client ID** and **Client Secret**.
3. Click **Sign in with Google**.
4. Choose your Gmail account and click **Allow**.

</details>

---

## 🛠️ Step-by-Step n8n Setup Guide

### Step 1: Create a New Blank Workflow
1. Click on **Workflows** in the left sidebar menu.
2. Click the **Create Workflow** button (or `+` icon) at the top right to start with a blank canvas.

### Step 2: Add, Test, and Configure the Gmail Trigger
1. Click the **Add first step** button on the blank canvas.
2. Search for **Gmail Trigger** and select it.
3. Select your connected Gmail account under **Credential to connect with**.
4. Set **Event** to `Message Received`.
5. Under **Filters**, click **Add Filter** ➔ select **Search**, and type:
<details>
<summary>Click to Expand / View Code</summary>

```text
is:unread -from:me
```
</details>

6. **Crucial Step**: Click the yellow **Test step** button at the top right of the panel (make sure you have at least one email in your inbox). *(Running this once loads the input schema so `id`, `From`, `Subject`, and `snippet` show up in the left panel for drag-and-drop).*
7. Close the Gmail Trigger panel.

### Step 3: Add the AI Agent Node (Set System & User Prompts)
1. Hover over the Gmail Trigger node on the canvas and click the small `+` icon on its right side.
2. Search for **AI Agent** and select it.
3. Double-click the AI Agent node to open its settings:
   - Set **Agent** to `Tools Agent`.
4. Set the **User Prompt** in the main Prompt / Text field:

<details>
<summary>Click to Expand Prompt</summary>

```text
Read the incoming email from trigger and send a polite,helpful reply back using reply to a message in gmail tool

From: {{ $json.From }}
Subject: {{ $json.Subject }}
Email: {{ $json.snippet }}
```
</details>

5. Set the **System Prompt**:
   - Scroll down to **Options** at the bottom of the AI Agent panel and click **Add Option**.
   - Select **System Prompt**.
   - Paste your full system prompt inside that box:

<details>
<summary>Click to Expand Prompt</summary>

```text
You are the official AI Customer Support Assistant for Neuralize AI.

Neuralize AI provides:
• AI & Machine Learning Workshops
• Agentic AI Development
• AI Chatbot Development
• Website Development
• Automation using n8n
• Technical Training Programs

Whenever a new email is received:

1. Read the customer's email carefully.
2. Understand the customer's query.
3. Generate a professional, polite and concise reply.
4. Use the Gmail Reply tool exactly once to reply to the received email.
5. If the customer asks about pricing, politely mention that a team member will contact them with a customized quotation.
6. If the customer asks about workshops, mention that Neuralize conducts hands-on AI workshops for schools, colleges and organizations.
7. Never invent pricing, dates or commitments.
8. After calling the Gmail Reply tool successfully, stop. Do not call it again.

End every reply with:

Best Regards,
Neuralize AI Support Team
```
</details>

6. Close the AI Agent panel.

### Step 4: Connect the Google Gemini Model
1. On the canvas, find the bottom-left connector port on the AI Agent node labeled **Chat Model**.
2. Click and drag out a line from **Chat Model** onto the canvas.
3. Search for **Google Gemini Chat Model** and select it.
4. Select your Gemini API key under **Credential**.
5. Choose your preferred model (e.g., `gemini-3.1-flash-lite`).
6. Close the Gemini node panel.

### Step 5: Connect and Drag-and-Drop Configure the Gmail Tool
1. On the canvas, find the bottom-right connector port on the AI Agent node labeled **Tool**.
2. Click and drag out a line from **Tool** onto the canvas.
3. Search for **Gmail** and select the Gmail tool node.
4. In the settings:
   - **Credential**: Select your Gmail account.
   - **Resource**: Select `Message`.
   - **Operation**: Select `Reply`.
5. Set the **Message ID**:
   - Clear out any existing text in the Message ID box.
   - Look at the left **INPUT** panel (populated from Step 2).
   - Click and drag the top `id` field straight into the Message ID box so it displays:
<details>
<summary>Click to Expand / View Code</summary>

```text
{{ $json.id }}
```
</details>

6. Set **Message**: Keep it set to `Defined automatically by the model`.
7. Prevent cross-replying:
   - Scroll to **Options** at the bottom, click **Add option**, and select **Reply to Sender Only**.
   - Toggle it to **True / Enabled**.
8. Close the Gmail Tool panel.

### Step 6: Test Manually & Publish
1. Send a test email to your Gmail address from a second email account.
2. Click the yellow **Execute workflow** button at the bottom of the n8n canvas to test the complete run.
3. Check the second email account to verify the Neuralize AI reply arrived!
4. Switch the top-right toggle from **Inactive** to **Active / Published** so it replies automatically in the background.

---

# 📄 Agent 2: AI Resume Analyzer & ATS Agent

An automated n8n workflow that receives candidate resume PDFs via a web form, processes the document directly using Google Gemini's multimodal capabilities, and sends a detailed ATS score and evaluation report to the candidate's email.

---

## 🏗️ Workflow Architecture

<details>
<summary>Click to Expand / View Code</summary>

```text
[n8n Form Trigger] ──► [AI Agent Node] ──► [Gmail Node]
                           ▲
                           │ (Chat Model)
               [Google Gemini Chat Model]
```

</details>

---

## 🚀 Setup & Installation Steps

### Step 1: Configure the "n8n Form Trigger" Node
1. Add the **n8n Form Trigger** node to your canvas.
2. Set **Form Title**: `AI Resume Analyzer`
3. Set **Form Description**: `Upload your PDF resume to receive a real-time ATS score and detailed feedback.`
4. Add 3 Form Elements:
   - **Element 1 (Name)**: Label = `Your Name` | Type = `Text Input` | Custom Field Name = `name`
   - **Element 2 (Email)**: Label = `Your Email Address` | Type = `Email` | Custom Field Name = `email`
   - **Element 3 (Resume File)**: Label = `Upload Resume (PDF)` | Type = `File` | Custom Field Name = `resume`

### Step 2: Configure the "AI Agent" Node
1. Connect **n8n Form Trigger** ➔ **AI Agent**.
2. Set **Prompt / Text (User Prompt)**:

<details>
<summary>Click to Expand Prompt</summary>

```text
Please analyze the attached candidate resume document.
```
</details>

3. Scroll down to **Options** ➔ Click **Add Option**:
   - Select **Automatically Passthrough Binary PDFs** and toggle it **ON**.
4. Click **Add Option** ➔ Select **System Prompt**, then paste:

<details>
<summary>Click to Expand Prompt</summary>

```text
You are an expert AI Resume Reviewer and Applicant Tracking System (ATS) Evaluator named Neuralize AI.

Your task is to analyze the candidate's resume PDF provided to you and generate a clear, professional, and actionable evaluation report.

Format your output using clean Markdown as follows:

# 📄 ATS Resume Evaluation Report

### Candidate Overview
* **Candidate Name:** [Extract from resume or say "Not Specified"]
* **Target Role Level:** Entry-Level / Software Engineering Intern

---

### 📊 Overall ATS Score
**[Score]/100** 

---

### Key Strengths
* [Strength 1]
* [Strength 2]
* [Strength 3]

---

### Areas for Improvement & Missing Keywords
* [Improvement 1]
* [Improvement 2]
* [Missing critical technical skill/keyword 3]

---

### Actionable Recommendations
1. [Clear step to improve formatting or content]
2. [Project or skill enhancement suggestion]

---
*Evaluated automatically by Neuralize AI Resume Agent.*
```
</details>

### Step 3: Connect "Google Gemini Chat Model"
1. Connect **Google Gemini Chat Model** to the **Chat Model** port on the AI Agent node.
2. Select your Gemini API Key credential.
3. Set **Model** to `gemini-3.1-flash-lite`.

### Step 4: Configure the "Gmail" Node
1. Connect **AI Agent** ➔ **Gmail Node**.
2. Settings:
   - **Resource**: `Message`
   - **Operation**: `Send`
   - **To**: Switch to Expression mode (`fx`) and set:
<details>
<summary>Click to Expand / View Code</summary>

```text
{{ $('n8n Form Trigger').item.json.email }}
```
</details>

   - **Subject**: `Your AI Resume Analysis & ATS Score Report - Neuralize AI`
   - **Message**:
<details>
<summary>Click to Expand / View Code</summary>

```text
{{ $json.output }}
```
</details>

### Step 5: Test the Workflow
1. Click **Execute workflow**.
2. Open the Form Trigger Test URL, fill in your details, and upload a sample resume PDF.
3. Verify that the candidate receives the structured ATS score report in their email inbox.
4. Toggle the workflow to **Active / Published**.

---

## 📄 HTML Template for Resume Test PDF

You can save this HTML file and convert it to a clean, text-selectable PDF for testing ATS evaluation workflows:

<details>
<summary>📄 HTML Template for Resume Test PDF</summary>

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<style>
  @page { size: A4; margin: 15mm; }
  body { font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; color: #2b2b2b; line-height: 1.4; font-size: 9.5pt; }
  .header { text-align: center; border-bottom: 2px solid #1a365d; padding-bottom: 8px; margin-bottom: 12px; }
  .name { font-size: 18pt; font-weight: bold; color: #1a365d; text-transform: uppercase; }
  .contact-info { font-size: 9pt; color: #4a5568; }
  .section { margin-bottom: 12px; }
  .section-title { font-size: 11pt; font-weight: bold; color: #1a365d; text-transform: uppercase; border-bottom: 1px solid #cbd5e0; padding-bottom: 3px; margin-bottom: 6px; }
  ul { margin: 3px 0 0 0; padding-left: 16px; }
  li { margin-bottom: 3px; }
</style>
</head>
<body>

  <div class="header">
    <div class="name">Arjun Sharma</div>
    <div class="contact-info">
      Vadodara, Gujarat, India | +91 98765 43210 | arjun.sharma@email.com<br>
      https://linkedin.com/in/arjun-sharma | https://github.com/arjun-sharma
    </div>
  </div>

  <div class="section">
    <div class="section-title">Professional Summary</div>
    <p>Driven 2nd-year Computer Science student with expertise in Data Structures, Relational Databases, Python, and AI automation workflows.</p>
  </div>

  <div class="section">
    <div class="section-title">Technical Skills</div>
    <ul>
      <li><strong>Languages:</strong> Python, JavaScript, SQL, C++</li>
      <li><strong>Frameworks & Tools:</strong> React.js, Node.js, n8n Automation, Google Gemini API, Git</li>
    </ul>
  </div>

  <div class="section">
    <div class="section-title">Projects</div>
    <p><strong>AI Resume Analyzer & ATS Agent</strong> (n8n, Python, Gemini API)</p>
    <ul>
      <li>Automated PDF parsing and real-time evaluation using Gemini LLM and Gmail API.</li>
    </ul>
  </div>

</body>
</html>
```
</details>

---

# 🐍 Agent 3: AI Code Reviewer & Bug Hunter Agent

An automated n8n workflow that receives code snippets via a web form, evaluates them using Google Gemini for bugs, security vulnerabilities, and $O(n)$ time/space complexity issues, and sends a refactored code report directly to the developer's email.

---

## 🏗️ Workflow Architecture

<details>
<summary>Click to Expand / View Code</summary>

```text
[n8n Form Trigger] ──► [AI Agent Node] ──► [Gmail Node]
                           ▲
                           │ (Chat Model)
               [Google Gemini Chat Model]
```

</details>

---

## 🚀 Setup & Installation Steps

### Step 1: Configure "n8n Form Trigger" Node
1. Add the **n8n Form Trigger** node to your canvas.
2. Set **Form Title**: `🐍 AI Code Reviewer & Bug Hunter`
3. Set **Form Description**: `Paste your code snippet below to get instant bug detection, complexity analysis, and clean refactored code.`
4. Click **+ Add Form Element** 3 times:
   - **Element 1 (Language)**: Label = `Programming Language` | Type = `Text Input` | Custom Field Name = `language`
   - **Element 2 (Code Snippet)**: Label = `Paste Your Code Here` | Type = `Text Area` | Custom Field Name = `code`
   - **Element 3 (Email)**: Label = `Your Email Address` | Type = `Email` | Custom Field Name = `email`

### Step 2: Configure "AI Agent" Node
1. Connect **n8n Form Trigger** ➔ **AI Agent**.
2. In the **Prompt / Text** box (User Prompt), paste:

<details>
<summary>Click to Expand Prompt</summary>

```text
Language: {{ $json.language }}

Code Snippet to Review:
{{ $json.code }}
```
</details>

3. Scroll to **Options** at the bottom ➔ Click **+ Add Option**:
   - Select **System Prompt** and paste this exact system prompt:

<details>
<summary>Click to Expand Prompt</summary>

````text
You are an expert Senior Software Engineer and Tech Lead specializing in code quality, algorithm optimization, security, and clean code principles.

Your job is to review the code provided by the developer, identify bugs, analyze time and space complexity, and provide refactored, production-ready code.

Format your review strictly using clean Markdown as follows:

# 🛠️ AI Code Review & Security Analysis

### 🎯 Summary & Health Rating
* **Language:** [Specified Language]
* **Code Health Score:** [Score]/100
* **Status:** [e.g., 🔴 Critical Bugs Found / 🟡 Refactoring Suggested / 🟢 Production Ready]

---

### 🐛 Identified Bugs & Security Issues
* **[Bug 1 Title]:** Explanation of why it fails or crashes.
* **[Bug 2 Title]:** Explanation of bad practice or edge-case failure.

---

### ⚡ Performance & Complexity Analysis
* **Time Complexity:** O(...) - [Brief explanation]
* **Space Complexity:** O(...) - [Brief explanation]
* **Optimization Potential:** [How memory/speed can be improved]

---

### 💡 Refactored & Optimized Code
Provide the fixed, clean, and production-ready code inside a proper markdown code block:

```[language]
// Refactored code goes here
```

### 📝 Key Takeaways & Best Practices
1. [Clear takeaway 1]
2. [Clear takeaway 2]

Reviewed automatically by Neuralize AI Code Reviewer Agent.
````
</details>

### Step 3: Connect "Google Gemini Chat Model"
1. Click the **Chat Model** port on the bottom of the AI Agent node.
2. Select **Google Gemini Chat Model**.
3. Choose your credential and select `gemini-3.1-flash-lite`.

### Step 4: Configure "Gmail Node"
1. Connect **AI Agent** ➔ **Gmail Node**.
2. Configure settings:
   - **Resource**: `Message`
   - **Operation**: `Send`
   - **To**: Switch to Expression mode (`fx`) and set to:
<details>
<summary>Click to Expand / View Code</summary>

```text
{{ $('n8n Form Trigger').item.json.email }}
```
</details>

   - **Subject**: `Your AI Code Review & Security Report - Neuralize AI`
   - **Message**:
<details>
<summary>Click to Expand / View Code</summary>

```text
{{ $json.output }}
```
</details>

### Step 5: Test the Workflow & Publish
1. Click the yellow **Execute workflow** button at the bottom of the n8n canvas to test the complete run.
2. Open the Form Trigger Test URL, fill in your details, select a language, and paste the sample buggy Python code.
3. Verify that the developer receives the structured AI Code Review & Security Report in their email inbox!
4. Switch the top-right toggle from **Inactive** to **Active / Published** so it runs automatically in the background.

---


## 🐍 Sample Buggy Python Code (For Testing)

Paste this code snippet into your form to test the agent:

<details>
<summary>🐍 Sample Buggy Python Code (For Testing)</summary>

```python
def process_user_data(user_list=[]):  # Mutable default argument
    results = {}
    
    # Intended: Find duplicate IDs and calculate average score
    for i in range(len(user_list)):
        for j in range(len(user_list)): # O(n^2) nested loop bug
            if user_list[i]['id'] == user_list[j]['id']:
                print("Found duplicate: " + user_list[i]['id']) # TypeError if id is int
                
        # UnboundLocalError: total_score used before declaration
        total_score = total_score + user_list[i]['score'] 
        
    avg = total_score / len(user_list) # ZeroDivisionError if list is empty
    return avg

# Test run
users = [
    {"id": 101, "name": "Arjun", "score": 85},
    {"id": 102, "name": "Priya", "score": 92},
    {"id": 101, "name": "Arjun", "score": 85}
]

print(process_user_data(users))
```
</details>


