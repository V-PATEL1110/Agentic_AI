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
