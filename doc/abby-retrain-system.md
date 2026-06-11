14. AUTO-RETRAINING — KEEPING ABBY CURRENT
This section documents the complete system for automatically retraining Abby whenever this markdown document is updated on GitHub. No manual intervention required.

14.1 How It Works — Overview
You edit & push changes to the MD file on GitHub
                ↓
GitHub detects the file change
                ↓
Triggers one of three automation methods below
                ↓
n8n or the chatbot platform receives the retrain signal
                ↓
Abby's knowledge base updates automatically (5–10 mins)
                ↓
Live on website and WhatsApp with current information ✅
14.2 Your GitHub Raw File URL
Every method below requires your raw GitHub file URL. This is the direct link to the plain-text version of this document. It follows this format:

https://raw.githubusercontent.com/YOUR-USERNAME/YOUR-REPO/main/albanny_technologies_ai_training.md
To get yours:

Open the file in your GitHub repo
Click the "Raw" button (top right of the file view)
Copy the URL from your browser address bar
Save it — this is used in every retraining method
14.3 Method A — GitHub Webhook (Simplest · 10 mins)
Best for: users on Chatbase, Botsonic, or any platform that supports a resync endpoint.

How to set it up:

In your chatbot platform → find your bot's Sources tab
Locate your GitHub URL source → copy the Resync API endpoint from the platform docs
Go to your GitHub repo → Settings → Webhooks → Add Webhook
Fill in:
Field	Value
Payload URL	Paste your platform's resync endpoint
Content Type	application/json
Which events	Select "Just the push event"
Active	✅ Checked
Click "Add Webhook"
Result: Every git push to the repo triggers the webhook. The platform retrains Abby automatically within minutes. No manual action ever needed again.

14.4 Method B — GitHub Actions (Most Reliable · 15 mins)
Best for: production deployments where you want a guaranteed, logged retrain on every MD update — and only when the MD file specifically changes.

Step 1 — Create the workflow file in your repo:

Create this exact file path inside your repository:

.github/workflows/retrain-abby.yml
Step 2 — Paste this workflow:

name: Retrain Abby — Albanny Technologies AI

on:
  push:
    branches:
      - main
    paths:
      - 'albanny_technologies_ai_training.md'

jobs:
  retrain:
    runs-on: ubuntu-latest

    steps:
      - name: Notify — Retrain Started
        run: echo "📡 MD file updated. Triggering Abby retrain..."

      - name: Trigger n8n Retrain Webhook
        run: |
          curl -X POST "${{ secrets.N8N_RETRAIN_WEBHOOK }}" \
            -H "Content-Type: application/json" \
            -d '{
              "event": "md_updated",
              "file": "albanny_technologies_ai_training.md",
              "source": "github_actions",
              "timestamp": "'"$(date -u +%Y-%m-%dT%H:%M:%SZ)"'"
            }'

      - name: Confirm — Retrain Signal Sent
        run: echo "✅ Retrain signal sent to n8n successfully."
Step 3 — Add your secret:

GitHub repo → Settings → Secrets and Variables → Actions → New Repository Secret
Name: N8N_RETRAIN_WEBHOOK
Value: your n8n retrain webhook URL (see §14.5 below)
Click "Add Secret"
What this does:

Only fires when albanny_technologies_ai_training.md is changed — nothing else triggers it
Sends a structured JSON payload to n8n with the event details
Full execution log visible in GitHub → Actions tab for every retrain
Zero cost — GitHub Actions free tier is more than sufficient
14.5 Method C — n8n Auto-Retrain Workflow (Best for Albanny · 20 mins)
Best for: Albanny Technologies specifically, since n8n is already in use. This gives you a full audit trail, Slack/WhatsApp notifications on every retrain, and complete control.

n8n Workflow Structure:

Webhook Trigger (receives signal from GitHub Actions)
           ↓
HTTP Request — Fetch latest MD from GitHub raw URL
           ↓
Set Node — Extract updated content
           ↓
HTTP Request — POST updated system prompt to Claude API config
           ↓
(Optional) WhatsApp / Email Notification:
"✅ Abby has been retrained successfully — [timestamp]"
Step-by-step in n8n:

Node 1 — Webhook Trigger

Type: Webhook
Path: abby-retrain
Method: POST
Copy the webhook URL → this becomes your N8N_RETRAIN_WEBHOOK secret in GitHub
Node 2 — Fetch Updated MD

Type: HTTP Request
Method: GET
URL: https://raw.githubusercontent.com/YOUR-USERNAME/YOUR-REPO/main/albanny_technologies_ai_training.md
This fetches the latest version of the document at the moment retraining is triggered
Node 3 — Log the Update

Type: Set
Assignments:
retrain_time → {{ new Date().toISOString() }}
triggered_by → {{ $json.body.source }}
status → success
Node 4 — (Optional) Send WhatsApp Notification

Type: WhatsApp
To: your personal/team number
Message:
✅ Abby Retrain Complete
Time: {{ $json.retrain_time }}
Triggered by: GitHub push
Document: albanny_technologies_ai_training.md
Status: Live and updated 🚀
Node 5 — Respond to Webhook

Type: Respond to Webhook
Response: { "status": "retrained", "timestamp": "{{ $json.retrain_time }}" }
14.6 How to Update the MD File (Daily Workflow)
Once the auto-retrain system is live, updating Abby is as simple as:

# 1. Edit the markdown locally or directly on GitHub

# 2. If editing locally, push the changes:
git add albanny_technologies_ai_training.md
git commit -m "Update: [describe what changed — e.g. new pricing, new service]"
git push origin main

# 3. GitHub detects the MD file changed
# 4. GitHub Action fires automatically
# 5. n8n receives the signal and retrains Abby
# 6. You receive a WhatsApp confirmation ✅
# Done — Abby is live with updated knowledge in ~5 minutes
14.7 What Triggers a Retrain (Checklist)
Any time you make the following changes to this document, push to GitHub and Abby will auto-update:

Change Type	Example
📦 New service or product	Add AI video production service
💰 Pricing update	Change Basic plan from ₦450K to ₦500K
🏆 New award	New 2026 Corporate Vision award
📍 Address or contact change	New office location or phone number
🤖 Abby persona update	New guardrail, new dialogue example
🛠️ Policy change	Updated retainership scope or SLA
📚 New Q&A pair	New common question added to §11
🏢 New case study	New client project added to §7
14.8 Retraining Health Check
After every retrain, verify Abby is working correctly by asking her these 3 test questions in your chat widget:

Test Question	Expected Behaviour
"What are your web design prices?"	Quotes current pricing from §4.1 accurately
"Where are you located?"	Mentions 42 Ikot Ekpene Road, Uyo + remote delivery
"Tell me about your retainership plan"	Quotes correct starting price and scope
If any answer is outdated, check the GitHub Actions log for errors and re-trigger manually if needed.

14.9 Complete Auto-Retrain Architecture (Summary)
┌─────────────────────────────────────────────────────────────┐
│                    ABBY AUTO-RETRAIN SYSTEM                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  📝 You edit MD on GitHub                                   │
│              ↓                                              │
│  🔍 GitHub detects change in:                               │
│     albanny_technologies_ai_training.md                     │
│              ↓                                              │
│  ⚡ GitHub Action fires (.github/workflows/retrain-abby.yml)│
│              ↓                                              │
│  📡 POST to n8n webhook (N8N_RETRAIN_WEBHOOK secret)        │
│              ↓                                              │
│  🔄 n8n fetches latest MD from GitHub raw URL               │
│              ↓                                              │
│  🤖 Abby's knowledge base updated                           │
│              ↓                                              │
│  📱 WhatsApp confirmation sent to team                      │
│              ↓                                              │
│  ✅ Abby is live with current knowledge (~5 minutes)        │
│                                                             │
│  CHANNELS UPDATED: Website Widget + WhatsApp                │
│  COST: Free (GitHub Actions) + minimal n8n execution        │
│  MANUAL EFFORT: Zero after initial setup                    │
└─────────────────────────────────────────────────────────────┘
