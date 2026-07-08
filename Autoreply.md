# Gmail AI Draft Reply Workflow

<img width="1512" height="701" alt="AI agent" src="https://github.com/user-attachments/assets/1f7b6bac-fc1a-4c61-a92a-c9243406fb7c" />


This n8n workflow automatically creates a **Gmail draft reply** whenever a new email is received.

The workflow uses an AI Agent to read the incoming email, prepare a professional response, and save it as a draft in Gmail. The email is **not sent automatically**, so a human can review, edit, and approve the reply before sending.

---

## Workflow Overview

```text

Gmail Trigger
→ Check Junk Email
   ├─ true  → Mapping
   └─ false → Stop
Mapping / Edit Fields
      ↓
AI Agent
 ├── Google Gemini Chat Model
 ├── Simple Memory
 └── Gmail Tool: Create Draft


##Purpose

The purpose of this workflow is to reduce manual email-writing time while keeping human control over final communication.

The workflow follows a safe human-review process:

New email received
→ Email details are extracted
→ AI writes a professional draft reply
→ Gmail draft is created
→ Human reviews and sends manually

Key Features
Automatically detects new Gmail messages
Extracts important email details
Uses AI to generate a professional draft reply
Creates a Gmail draft in the original email thread
Does not send emails automatically
Allows human review before sending
Reduces repetitive reply-writing work
Improves response consistency and professionalism


| Step | Node                      | Purpose                                      |
| ---- | ------------------------- | -------------------------------------------- |
| 1    | Gmail Trigger             | Detects a new incoming Gmail message         |
| 2    | Mapping / Edit Fields     | Cleans and organises the email data          |
| 3    | AI Agent                  | Reads the email and prepares the reply       |
| 4    | Google Gemini Chat Model  | Generates the AI response                    |
| 5    | Simple Memory             | Provides short-term context for the AI Agent |
| 6    | Gmail Tool — Create Draft | Creates a Gmail draft reply                  |
