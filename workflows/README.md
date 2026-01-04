# n8n Workflows Documentation

## Lead Capture Form Workflow

**File:** `Lead-capture-form-5-minutes-auto.json`

### Overview

This workflow automates lead capture and response generation using AI. It monitors Gmail for incoming web form messages, processes them with OpenAI GPT-4, and sends approval requests to Telegram with inline keyboard buttons.

### Workflow Flow

```
Gmail Trigger → AI Lead Extraction → AI Reply Generation → AI Human-like Text → Telegram Approval → Email Send
```

### Node Details

#### 1. Gmail Trigger (`On message from web form Gmail trigger`)
- **Purpose**: Monitors Gmail inbox for new messages
- **Configuration**:
  - Polls every 5 minutes
  - Filters by sender: `denis911@gmail.com`
  - Triggers on new web form submissions

#### 2. OpenAI Node (`Extract lead info`)
- **Purpose**: Extracts structured data from incoming emails
- **AI Prompt Logic**:
  ```
  Extract company name, person name, and any comments from the message
  Return STRICTLY as JSON with fields: name, company, text
  ```
- **Output**: `{"name": "John Doe", "company": "Acme Corp", "text": "Interested in services"}`

#### 3. OpenAI Node (`Write a reply`)
- **Purpose**: Generates personalized email response
- **AI Prompt Logic**:
  1. Research company using web search
  2. Create personalized reply template
  3. Include relevant company information (1-2 facts)
  4. Structure: Greeting → Context → Question → Call to action
- **Web Search**: Enabled for company research
- **Output**: Professional email draft

#### 4. OpenAI Node (`Make text look more human`)
- **Purpose**: Converts AI-generated text to natural human communication
- **AI Prompt Logic**:
  1. Remove all links, UTM parameters, markup
  2. Structure: Thank → Context → Clarifying question → Zoom offer
  3. Make it conversational and natural
- **Output**: Natural-sounding email text

#### 5. Telegram Node (`Send a text message`)
- **Purpose**: Sends approval request with inline keyboard buttons
- **Configuration**:
  - Operation: `sendAndWait` (creates interactive buttons)
  - `approvalOptions.approvalType: "double"` (Approve/Reject buttons)
- **Message Format**:
  ```
  🚨 New lead captured from: [extracted data]
  📝 Reply text message for approval: [generated reply]

  [Approve/Reject buttons appear here]
  ```

#### 6. Switch Node (`Switch`)
- **Purpose**: Routes workflow based on approval decision
- **Logic**: Checks `$json.data.approved`
  - `true` → Send approved email
  - `false` → Send rejected email

#### 7. Gmail Nodes (`APPROVED - Send a message` / `REJECTED message`)
- **Purpose**: Sends final emails to designated address
- **Configuration**:
  - Approved: Subject "Approved email ready to send - leads capture form"
  - Rejected: Subject "Rejected message - lead capture form" + "ATTENTION - THIS MESSAGE WAS REJECTED !!!"

### Technical Implementation

#### AI Prompts Strategy
- **Lead Extraction**: Structured JSON output for reliable parsing
- **Reply Generation**: Multi-step process with web research
- **Human-like Text**: Focus on natural conversation flow

#### Error Handling
- Workflow includes try/catch logic in AI nodes
- Telegram webhook failures are handled gracefully
- Email sending failures are logged

#### Security Considerations
- API keys stored in n8n credentials system
- No sensitive data logged in workflow
- Encrypted communication via HTTPS

### Configuration Requirements

#### Environment Variables
- `N8N_HOST=localhost` (required for GCP OAuth)
- `N8N_ENCRYPTION_KEY` (generate with `openssl rand -hex 32`)

#### External Services
- **OpenAI API**: GPT-4 access with credits
- **Google Cloud Platform**: Gmail API enabled, OAuth credentials
- **Telegram Bot**: Created via @BotFather, token configured
- **ngrok**: For webhook tunneling in development

#### n8n Credentials
- `openAiApi`: OpenAI API key
- `gmailOAuth2`: GCP OAuth credentials
- `telegramApi`: Telegram bot token

### Testing the Workflow

1. **Setup ngrok**: `ngrok http 5678`
2. **Update .env**: Add ngrok URLs for `WEBHOOK_URL` and `N8N_EDITOR_BASE_URL`
3. **Import workflow**: Upload JSON file in n8n UI
4. **Activate workflow**: Toggle "Active" switch
5. **Test**: Send email to trigger address
6. **Approve/Reject**: Use Telegram buttons

### Monitoring & Debugging

- **n8n UI**: View execution history and logs
- **Docker logs**: `docker compose logs -f n8n`
- **Webhook testing**: Use curl with ngrok-skip-browser-warning header

### Performance Considerations

- **AI API calls**: Rate limited by OpenAI quotas
- **Gmail polling**: 5-minute intervals to avoid rate limits
- **Telegram webhooks**: Require public HTTPS URL (ngrok in development)

This workflow demonstrates advanced AI integration with practical business automation, combining multiple APIs into a cohesive lead management system.
