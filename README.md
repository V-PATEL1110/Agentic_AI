## 🔑 Prerequisites

<details>
<summary>🔑 Prerequisite 1: Obtain Google Gemini API Key</summary>

### Creating Your Google Gemini API Key

Step 1: Open Google AI Studio
- Go to: https://aistudio.google.com/

Step 2: Get API Key
- Click "Get API key" from the left sidebar or top menu.

Step 3: Create API Key in Project
- Click "Create API key".
- Choose "Create API key in new project" (or select your existing project).

Step 4: Copy and Save Your API Key
- Copy the generated API key string.
- Save it securely in a text file or environment variable (Example: AIzaSyxxxxxxxxxxxxxxxxxxxx).
- Important Security Note: Do NOT commit your API key directly to GitHub or share it publicly.

Step 5: Use in n8n Workflows
- This Gemini API Key will be used in each agent whenever you configure and connect the Google Gemini Chat Model node.

</details>

<details>
<summary>🔑 Prerequisite 2: Google OAuth 2.0 Setup Guide</summary>

### Creating Google OAuth Client ID & Client Secret

Step 1: Open Google Cloud Console
- Go to: https://console.cloud.google.com/

Step 2: Create a New Project
1. Click the Project Selector at the top.
2. Click New Project.
3. Enter a project name (Example: Neuralize Gmail Agent).
4. Click Create.
5. Select the newly created project.

Step 3: Enable Gmail API
1. Open the left sidebar.
2. Go to: APIs & Services ➔ Library.
3. Search for Gmail API.
4. Open Gmail API and click Enable.

Step 4: Configure OAuth Consent Screen
1. Go to: APIs & Services ➔ OAuth consent screen.
2. Click Get Started.
3. Fill in the details:
   - App Name: Neuralize AI Agent
   - User Support Email: Select your Gmail account.
   - Developer Contact Email: Select your Gmail account.
4. Click Next until completed.

Step 5: Add Test Users
1. Go to Audience ➔ Choose External.
2. Go to Test Users ➔ Click Add Users.
3. Add the Gmail account that will be used in n8n (Example: yourname@gmail.com).
4. Click Save.

Step 6: Configure Data Access
1. Go to Data Access ➔ Click Add or Remove Scopes.
2. Search and add these scopes:
   - Gmail API ➔ .../auth/gmail.modify
   - Gmail API ➔ .../auth/gmail.send
3. Click Update, then click Save.

Step 7: Create OAuth Client
1. Go to: APIs & Services ➔ Credentials.
2. Click + Create Credentials ➔ Choose OAuth Client ID.

Step 8: Select Application Type
- Choose Web application.

Step 9: Give It a Name
- Example: n8n Gmail OAuth

Step 10: Add Authorized Redirect URI
1. Copy the Redirect URL shown by n8n in the Gmail credential window.
   - Example: https://your-n8n-instance/rest/oauth2-credential/callback
   - Or for n8n Cloud: https://xxxxx.app.n8n.cloud/rest/oauth2-credential/callback
2. Paste it under Authorized Redirect URIs.
3. Click Create.

Step 11: Copy Credentials
- Google will display Client ID and Client Secret. Copy both credentials.

Step 12: Connect in n8n
1. In n8n, open the Gmail Credential window.
2. Paste the Client ID and Client Secret.
3. Click Sign in with Google.
4. Choose your Gmail account and click Allow.

</details>

---

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

## 🛠️ Step-by-Step n8n Setup Guide

### Step 1: Create a New Blank Workflow
1. Click on **Workflows** in the left sidebar menu.
2. Click the **Create Workflow** button (or `+` icon) at the top right to start with a blank canvas.

### Step 2: Configure the Gmail Trigger
1. Add the **Gmail Trigger** node to your canvas.
2. Set **Event**: `Message Received`.
3. Set **Poll Times**: `Every Minute`.
4. Set **Max Emails Per Poll**: `1`.
5. Under **Filters ➔ Search**, enter:
   ```text
   is:unread -from:me
   ```

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

### Step 5: Configure the Gmail Tool Node
1. Search for the **Gmail** tool node in n8n and add it to your canvas.
2. Connect its **Tool** port to the **Tool** port of the **AI Agent** node.
3. Configure all node parameters as follows:
   - **Credential**: Select your connected Gmail OAuth Credential
   - **Tool Description**: Set Automatically
   - **Resource**: Message
   - **Operation**: Reply
   - **Message ID**: `{{ $json.id }}`
   - **Email Type**: Text
   - **Message**: Defined automatically by the model
   - **Options**: Click **Add option** ➔ select **Reply to Sender Only** and set it to **True / Enabled**.

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
You are an expert AI Resume Reviewer and Technical Recruiter named Neuralize AI.

Your goal is to evaluate the candidate's resume PDF and generate a clear, professional, and structured evaluation report.

Use simple, clear, and professional language so students can easily understand the feedback. Do not use hashtags (#) or dash bullet points (-) in your response.

Format your output strictly as follows:

📄 ATS RESUME EVALUATION REPORT

👤 Candidate Summary:
• Candidate Name: [Extract name or state "Not Specified"]
• Targeted Level: Entry-Level / Internship

📊 Overall ATS Score:
• Score: [Insert Score]/100

🌟 Key Strengths:
• [Strength 1: Mention a strong skill or technical area]
• [Strength 2: Mention good project work or practical experience]
• [Strength 3: Mention good formatting or resume structure]

⚠️ Skill Gaps & Areas to Improve:
• [Improvement 1: Mention missing keywords or technical skills for entry-level roles]
• [Improvement 2: Mention formatting or layout issues if any]
• [Improvement 3: Mention bullet points that need clearer achievements or outcomes]

🚀 Step-by-Step Action Plan:
1. [Clear, simple step to fix formatting or layout]
2. [Clear, simple step to improve project descriptions]
3. [Simple suggestion for a skill or concept to learn]

🤖 Evaluated by Neuralize AI Resume Agent.
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
{{ $('On form submission').item.json.email }}
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

## 📄 Sample Resume PDF for Testing

Students can directly download and use the pre-built sample resume PDF for testing the ATS evaluation workflow:

- 📄 **Sample Resume File**: [Download Arjun_Sharma_Resume.pdf](./Arjun_Sharma_Resume.pdf)

> **Note:** Download and upload this PDF file into the **n8n Form Trigger** test URL to test the AI Resume Analyzer & ATS Agent.

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
You are a Senior Software Engineer and Mentor named Neuralize AI.

Your goal is to review the code provided by the developer, identify bugs, check performance, and give clear, constructive feedback alongside clean refactored code.

Use simple, clear, and professional language so students can quickly grasp where they made mistakes and how to fix them. Do not use hashtags (#) or dash bullet points (-) in your response.

Format your output strictly as follows:

💻 AI CODE REVIEW & SECURITY REPORT

📊 Summary:
• Language: [Specified Language]
• Health Score: [Score]/100
• Status: [Critical Bugs Found / Small Fixes Needed / Production Ready]

🐛 Bugs & Security Issues:
• [Bug 1 Title]: Explanation of what went wrong in simple terms.
• [Bug 2 Title]: Explanation of edge cases or bad coding practices.

⚡ Performance & Complexity:
• Time Complexity: O(...) ➔ Explanation of execution speed in plain English.
• Space Complexity: O(...) ➔ Explanation of memory usage in plain English.
• Optimization Tip: Simple explanation of how to make the code faster.

🛠️ Refactored & Production-Ready Code:
Provide the completely fixed, clean, and commented code inside a standard code block:

```[language]
// Refactored and commented implementation
```

💡 Recommendations:
1. [Simple coding habit or style recommendation]
2. [Simple security or bug-prevention recommendation]

🤖 Reviewed by Neuralize AI Code Agent.
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
{{ $('On form submission').item.json.email }}
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

---

# ⚙️ Steps to Install Community Version of n8n

> [!NOTE]
> **Future Reference / Workshop Note:** For workshops, participants were instructed to sign up for the **n8n Cloud version** (`n8n.cloud`) only. The local community edition installation steps below are provided for local development and self-hosted testing.

### Step 1: Install Node.js

#### Why do we need it?
n8n runs on Node.js, so it must be installed first.

#### Installation
* Visit [https://nodejs.org](https://nodejs.org)
* Download the LTS (Long Term Support) version.
* Run the installer.
* Keep all options as default.
* Click **Next** until installation finishes.
* Restart your computer (recommended).

#### Verify Installation
1. Open Command Prompt:
   - Press `Windows + R`
   - Type `cmd` and press Enter
2. Run:
```cmd
node -v
```
You should see something similar to:
```text
v22.x.x
```
3. Now check `npm`:
```cmd
npm -v
```
Output:
```text
10.x.x
```
✅ **Node.js is installed successfully if both commands work.**

---

### Step 2: Install n8n Community Edition

1. Open Command Prompt.
2. Run:
```cmd
npm install -g n8n
```
*(This may take a few minutes.)*

3. After installation finishes, verify it:
```cmd
n8n --version
```
Example output:
```text
1.xx.x
```
✅ **n8n is installed.**

---

### Step 3: Start n8n

1. Run:
```cmd
n8n
```
The terminal will show:
```text
Editor is now accessible via:
http://localhost:5678
```
2. Leave this terminal open.
3. Open Chrome.
4. Visit [http://localhost:5678](http://localhost:5678)

🎉 **Congratulations! Your local n8n server is running.**



