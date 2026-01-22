# Local n8n with Docker Compose

Run n8n workflow automation platform locally using Docker Compose with PostgreSQL for production-ready data persistence. Perfect for development, testing, and production workflows.

## 🚀 Quick Start

```bash
# Clone the repository
git clone https://github.com/denis911/local-n8n-with-docker-compose.git
cd local-n8n-with-docker-compose

# Copy environment configuration
cp .env.example .env

# Generate a secure encryption key (recommended)
openssl rand -hex 32

# Edit .env file and update N8N_ENCRYPTION_KEY with the generated key

# Start n8n with PostgreSQL
docker compose -f docker-compose-n8n-local.yml up -d

# Access n8n at http://localhost:5678
```

## 📋 Features

- **Latest n8n**: Always uses the most recent n8n version
- **PostgreSQL Database**: Production-ready data persistence
- **Health Checks**: Automatic service health monitoring
- **Security**: Encryption key for credential protection
- **Community Nodes**: Full support for community packages
- **File Storage**: Local file system integration
- **Workflow Persistence**: Automatic backup and restore

## 🏗️ Architecture

```bash
┌─────────────────┐    ┌─────────────────┐
│      n8n        │────│   PostgreSQL    │
│   (port 5678)   │    │   (internal)    │
└─────────────────┘    └─────────────────┘
         │                       │
         └───────────────────────┘
                 │
          ┌──────────────┐
          │   Volumes    │
          │ - ./data     │
          │ - ./local-files│
          │ - postgres_data│
          └──────────────┘
```

## 📁 Project Structure

```bash
local-n8n-with-docker-compose/
├── docker-compose-n8n-local.yml    # Docker Compose configuration
├── .env.example                    # Environment variables template
├── .env                           # Your environment variables (create from .env.example)
├── README.md                      # This file
├── workflows/                     # n8n workflow files (organized by workflow)
│   └── lead-capture-form-auto-email-telegram-approval/
│       ├── README.md              # Workflow-specific documentation
│       └── Lead-capture-form-5-minutes-auto.json
├── cloudflared/                   # Cloudflared tunnel configuration
│   └── config.yml                 # Tunnel configuration (create from example)
├── data/                          # n8n data persistence (created automatically)
└── local-files/                   # File storage (created automatically)
```

## 🌐 Webhook Tunneling Setup (ngrok)

**Required for Telegram webhooks**: Since you're running n8n locally, Telegram needs a public HTTPS URL to send webhook updates.

### Windows 11 Installation & Setup

1. **Install Chocolatey (package manager for Windows):**
   ```powershell
   # Run PowerShell as Administrator:
   Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
   ```

2. **Install ngrok via Chocolatey:**
   ```cmd
   # Open Command Prompt (no admin needed):
   choco install ngrok
   ```

3. **Sign up for ngrok account:**
   - Go to https://dashboard.ngrok.com/signup
   - Create free account
   - Copy your auth token from the dashboard

4. **Authenticate ngrok:**
   ```cmd
   ngrok config add-authtoken YOUR_AUTH_TOKEN_HERE
   ```

5. **Start tunnel for n8n:**
   ```cmd
   ngrok http 5678
   ```

6. **Copy the HTTPS URL:**
   - ngrok will show: `Forwarding https://abc123.ngrok.io -> http://localhost:5678`
   - Copy the `https://abc123.ngrok.io` URL

7. **Update environment variables:**
   ```bash
   # Edit your .env file and add:
   N8N_EDITOR_BASE_URL=https://your-ngrok-url.ngrok.io
   WEBHOOK_URL=https://your-ngrok-url.ngrok.io

   # ...if any problems with ngrok or token is missing - go to
   # https://dashboard.ngrok.com/get-started/your-authtoken
   # and check / copy / paste token...
   ```

8. **Restart n8n:**
   ```bash
   docker compose -f docker-compose-n8n-local.yml down
   docker compose -f docker-compose-n8n-local.yml up -d
   ```

### Alternative: Keep Tunnel Running

For development, keep ngrok running in a separate Command Prompt window:
```cmd
# Terminal 1: Run ngrok
C:\ngrok\ngrok.exe http 5678

# Terminal 2: Run n8n (after updating .env)
docker compose -f docker-compose-n8n-local.yml up -d
```

### Troubleshooting ngrok

- **Connection issues**: Try `ngrok config check`
- **Port conflicts**: Make sure port 5678 is free
- **URL changes**: Free tier URLs change on restart (upgrade for stable URLs)
- **Rate limits**: Free tier limited to 40 requests/minute

### Testing Webhooks with PowerShell

After setting up ngrok, test webhooks directly:

```powershell
# Test ngrok bypass header
Invoke-WebRequest -Uri "https://your-ngrok-url.ngrok.io" -Headers @{ "ngrok-skip-browser-warning" = "true" }

# Test actual webhook URLs (replace with your URLs)
Invoke-WebRequest -Uri "https://your-ngrok-url.ngrok.io/webhook-waiting/YOUR_ID?approved=true&signature=YOUR_SIG" -Headers @{ "ngrok-skip-browser-warning" = "true" }
```

---

## 🌐 Webhook Tunneling Setup (Cloudflared - Free Alternative)

**Cloudflared is a great free alternative to ngrok** - it offers stable URLs and no rate limits on the free tier. This is the recommended option for local development.

### Windows 11 Installation & Setup

1. **Download and install cloudflared:**
   - Go to https://github.com/cloudflare/cloudflared/releases
   - Download the latest `cloudflared-stable-windows-amd64.exe`
   - Rename to `cloudflared.exe` and move to a folder (e.g., `C:\cloudflared`)
   - Add the folder to your system PATH:
     - Search "Edit the system environment variables"
     - Click "Environment Variables"
     - Under "User variables", select "Path" → Click "Edit" → "New"
     - Add `C:\cloudflared` → Click "OK" for all dialogs

2. **Authenticate with Cloudflare:**
   ```cmd
   cloudflared tunnel login
   ```
   - This will open a browser for authentication
   - Select your Cloudflare account and authorize cloudflared
   - Credentials are saved to `%USERPROFILE%\.cloudflared\`

3. **Create a tunnel:**
   ```cmd
   cloudflared tunnel create n8n-local
   ```
   - Output shows tunnel ID and credentials file location
   - Example: `Created tunnel n8n-local with ID abc123-uuid...`
   - Credentials saved to: `%USERPROFILE%\.cloudflared\abc123-uuid.json`

4. **Configure the tunnel (create config.yml):**

   Create `%USERPROFILE%\.cloudflared\config.yml` with:
   ```yaml
   tunnel: abc123-uuid
   credentials-file: C:\Users\YOUR_USERNAME\.cloudflared\abc123-uuid.json
   logDirectory: C:\cloudflared\logs
   loglevel: info

   ingress:
     - hostname: n8n-local.yourdomain.com
       service: http://localhost:5678
       originRequest:
         connectTimeout: 30s
         noTLSVerify: false

     - service: http_status:404
   ```

   **Note:** For free tier without custom domain, use the auto-generated URL approach below.

5. **Route DNS (if using custom domain):**
   ```cmd
   cloudflared tunnel route dns n8n-local n8n-local.yourdomain.com
   ```
   - Creates a CNAME record in Cloudflare DNS

6. **Run the tunnel:**
   ```cmd
   cloudflared tunnel run n8n-local
   ```

### Free Tier: Quick Tunnel (No Login Required)

For immediate access without creating an account:

```cmd
cloudflared tunnel --url http://localhost:5678
```

This starts a temporary tunnel with a random URL like `https://random-name.trycloudflare.com`.

⚠️ **IMPORTANT - URL Changes on Every Restart:**
The quick tunnel URL is **ephemeral** and will change:
- After PC reboot
- When you stop and restart the tunnel
- Every time you run the command

**If your URL changes, you must:**
1. Copy the new URL from the terminal output
2. Update your `.env` file:
   ```bash
   N8N_EDITOR_BASE_URL=https://new-random-url.trycloudflare.com
   WEBHOOK_URL=https://new-random-url.trycloudflare.com
   ```
3. Restart n8n:
   ```bash
   docker compose -f docker-compose-n8n-local.yml down
   docker compose -f docker-compose-n8n-local.yml up -d
   ```
4. Update any external services (Telegram webhook URL, etc.) with the new URL

For stable URLs, use a **named tunnel** with a custom domain (requires paid Cloudflare plan).

### Create Cloudflared Config File for Quick Tunnel

Create `cloudflared\config.yml` in your project directory:

```yaml
# Project: local-n8n-with-docker-compose
# Run: cloudflared tunnel --config cloudflared\config.yml url http://localhost:5678

tunnel: ""
credentials-file: ""

# Quick tunnel configuration (no tunnel creation needed)
ingress:
  - service: http://localhost:5678
```

Then run:
```cmd
cloudflared tunnel --config cloudflared\config.yml url http://localhost:5678
```

### Keep Tunnel Running in Background

For development, run cloudflared in a separate terminal:

```cmd
# Terminal 1: Run cloudflared
cloudflared tunnel --url http://localhost:5678

# Terminal 2: Run n8n (after updating .env)
docker compose -f docker-compose-n8n-local.yml up -d
```

### Update n8n Environment Variables

After getting your cloudflared URL (e.g., `https://random-name.trycloudflare.com`):

```bash
# Edit your .env file and add:
N8N_EDITOR_BASE_URL=https://random-name.trycloudflare.com
WEBHOOK_URL=https://random-name.trycloudflare.com
```

Restart n8n:
```bash
docker compose -f docker-compose-n8n-local.yml down
docker compose -f docker-compose-n8n-local.yml up -d
```

### Cloudflared vs ngrok Comparison

| Feature | Cloudflared (Free) | ngrok (Free) |
|---------|-------------------|--------------|
| URL Stability | Changes each restart | Changes each restart |
| Rate Limits | None | 40 requests/minute |
| Custom Domains | Requires paid plan | Requires paid plan |
| Concurrent Tunnels | 1 | 1 |
| HTTP/2 Support | ✅ | ✅ |
| Websocket Support | ✅ | ✅ |
| Account Required | Optional (for named tunnels) | Required |

### Troubleshooting Cloudflared

- **Connection issues**: Check logs in `%USERPROFILE%\.cloudflared\logs`
- **Port conflicts**: Ensure port 5678 is free
- **Permission errors**: Run terminal as Administrator
- **URL not working**: Verify n8n is running on localhost:5678

### Testing Webhooks with Cloudflared

```powershell
# Test cloudflared tunnel
Invoke-WebRequest -Uri "https://your-tunnel-url.trycloudflare.com"

# Test actual webhook URLs
Invoke-WebRequest -Uri "https://your-tunnel-url.trycloudflare.com/webhook-waiting/YOUR_ID?approved=true&signature=YOUR_SIG"
```

## 🔐 Google Cloud Platform (GCP) OAuth Setup

**Important**: GCP OAuth requires `localhost` as the host (not `0.0.0.0`). Make sure your `.env` file has `N8N_HOST=localhost`.

### Step-by-Step GCP OAuth Setup

1. **Create a Google Cloud Project**
   - Go to [Google Cloud Console](https://console.cloud.google.com/)
   - Create a new project or select existing one

2. **Enable Required APIs**
   - Enable the Gmail API: APIs & Services → Library → Search "Gmail" → Enable
   - Enable the Google+ API (for user info): Search "Google+" → Enable

3. **Create OAuth 2.0 Credentials**
   - Go to APIs & Services → Credentials
   - Click "Create Credentials" → "OAuth 2.0 Client IDs"
   - Choose "Web application"
   - Set name: "n8n Local Development"
   - Add authorized redirect URIs:
     - `http://localhost:5678/rest/oauth2-credential/callback`
   - Save and copy Client ID and Client Secret

4. **Configure n8n Credentials**
   - Start n8n and go to Settings → Credentials
   - Create new credential: "Gmail OAuth2"
   - Fill in:
     - **Client ID**: Your GCP OAuth Client ID
     - **Client Secret**: Your GCP OAuth Client Secret
   - Save the credential

5. **Test the Connection**
   - Create a test workflow with Gmail Trigger node
   - Select your Gmail OAuth2 credential
   - The OAuth flow will redirect to Google for authorization

### Common OAuth Issues

- **"invalid_request" error**: Ensure `N8N_HOST=localhost` in your `.env` file
- **Redirect URI mismatch**: Make sure the callback URL matches exactly: `http://localhost:5678/rest/oauth2-credential/callback`
- **Scopes not granted**: Ensure Gmail API is enabled and user grants necessary permissions

## 🔐 Google Cloud Platform (GCP) Service Account Setup (For Google Sheets)

For backend processing where no user interaction is required (like reading/writing Google Sheets automatically), a **Service Account** is often easier and more stable than OAuth.

### Step-by-Step Setup

1.  **Create Project & Enable APIs**
    - Go to [Google Cloud Console](https://console.cloud.google.com/).
    - Create a new project or select an existing one.
    - **Enable APIs**:
        - Go to "APIs & Services" > "Library".
        - Search for **"Google Sheets API"** -> Enable.
        - Search for **"Google Drive API"** -> Enable.

2.  **Create Service Account**
    - Go to "APIs & Services" > "Credentials".
    - Click "**Create Credentials**" > "**Service Account**".
    - **Name**: e.g., "n8n-automation".
    - **Description**: "Access for n8n local instance".
    - Click "Create and Continue".
    - **Grant this service account access to project**:
        - For simple Sheet reading/writing where you share specific files, **no specific role is required** here. You can click "Continue" without selecting roles.
        - *Optional*: If you want the account to manage all files in the project, you could add "Editor", but *Sharing* per file is safer and typically sufficient.
    - Click "Done".

3.  **Generate & Secure Key**
    - In the Credentials list, click on the newly created Service Account (email looks like `n8n-automation@project-id.iam.gserviceaccount.com`).
    - Go to the **"Keys"** tab.
    - Click "Add Key" > "Create new key".
    - Select **JSON** and click "Create".
    - **⚠️ IMPORTANT**: The JSON file will download automatically.
        - **Do NOT save this file inside this repository**.
        - Save it in a secure location on your machine (e.g., a dedicated `secrets` folder outside your code, or a password manager).
        - If you commit this file to GitHub, your Google Cloud resources could be compromised.

4.  **Grant Permissions (Share the Sheet)**
    - Service Accounts do not have access to your personal files by default.
    - Open the Google Sheet you want n8n to access.
    - Click the **"Share"** button (top right).
    - In the "Add people" field, paste the **Service Account Email** (from the JSON file's `client_email` field, or GCP Console).
    - Set permission to **Editor**.
    - Uncheck "Notify people" (optional) and click "Send".

5.  **Configure in n8n**
    - In n8n, go to **Credentials** > **New**.
    - Search for **"Google API"** (or "Google Sheets" depending on version).
    - Credential Type: **Service Account**.
    - **Service Account Email**: Copy `client_email` from your JSON file.
    - **Private Key**: Copy `private_key` from your JSON file.
        - It must include the header `-----BEGIN PRIVATE KEY...` and footer `...END PRIVATE KEY-----`.
        - *Tip*: Open the JSON file with any text editor (Notepad, VS Code) to copy these values correctly.
    - Save the credential.

Now you can use the **Google Sheets** node with this "Google API" credential to read/write the shared sheet.

## ⚙️ Configuration

### Environment Variables (.env)

Copy `.env.example` to `.env` and configure:

```bash
# Basic Configuration (localhost required for GCP OAuth)
N8N_HOST=localhost
N8N_PORT=5678
N8N_PROTOCOL=http

# Security - Generate a strong key: openssl rand -hex 32
N8N_ENCRYPTION_KEY=your-super-secure-encryption-key-here-change-this

# Database Configuration
DB_TYPE=postgresdb
POSTGRES_DB=n8n
POSTGRES_USER=n8n
POSTGRES_PASSWORD=n8n_password

# Community Nodes
N8N_COMMUNITY_PACKAGES_ENABLED=true
N8N_UNVERIFIED_PACKAGES_ENABLED=true
N8N_VERIFIED_PACKAGES_ENABLED=true

# Executions
EXECUTIONS_DATA_PRUNE=true
EXECUTIONS_DATA_MAX_AGE=336

# Performance
N8N_DIAGNOSTICS_ENABLED=false
N8N_METRICS_ENABLED=false
```

### Important Security Notes

- **Never commit `.env` file** to version control
- **Generate a unique encryption key** for each deployment
- **Change default PostgreSQL password** in production
- **Use strong passwords** for all services

## 🐳 Docker Services

### n8n Service

- **Image**: `docker.n8n.io/n8nio/n8n:latest`
- **Port**: `5678` (configurable via `N8N_PORT`)
- **Volumes**:
  - `./data:/home/node/.n8n` - n8n configuration and data
  - `./local-files:/files` - file storage

### PostgreSQL Service

- **Image**: `postgres:15`
- **Database**: `n8n`
- **User**: `n8n`
- **Volume**: `postgres_data` - database persistence

## 📖 Setup Instructions

### Prerequisites

#### System Requirements
- Docker and Docker Compose installed
- At least 2GB free RAM
- Ports 5678 and 5432 available (PostgreSQL internal)

#### Required Accounts & API Keys

##### 1. **Telegram Bot Token**
**Cost**: Free
**Setup Time**: 5 minutes

1. Open Telegram and search for `@BotFather`
2. Send `/newbot` and follow the prompts
3. Save the bot token (starts with `123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11`)
4. **Important**: Start a chat with your bot by sending `/start` - this is required for the bot to send you messages

##### 2. **OpenAI API Key**
**Cost**: Pay-as-you-go (starts free, then ~$0.03 per 1K tokens for GPT-4)
**Setup Time**: 5 minutes

1. Go to [OpenAI Platform](https://platform.openai.com/)
2. Sign up/Login to your account
3. Navigate to API Keys section
4. Create a new secret key
5. **Important**: Add credits to your account (minimum $5) to use GPT-4

##### 3. **Google Cloud Platform (GCP)**
**Cost**: Free tier available
**Setup Time**: 10-15 minutes

Required for Google Sheets/Drive/Gmail nodes. You can use either **OAuth** (user interaction) or **Service Account** (backend automation).
- See [OAuth Setup](#-google-cloud-platform-gcp-oauth-setup) for Gmail/interactive use.
- See [Service Account Setup](#-google-cloud-platform-gcp-service-account-setup-for-google-sheets) for automated Sheets/Drive access.

### Step-by-Step Setup

1. **Clone Repository**

   ```bash
   git clone https://github.com/denis911/local-n8n-with-docker-compose.git
   cd local-n8n-with-docker-compose
   ```

2. **Configure Environment**

   ```bash
   # Copy environment template
   cp .env.example .env

   # Generate secure encryption key
   ENCRYPTION_KEY=$(openssl rand -hex 32)
   echo "Generated encryption key: $ENCRYPTION_KEY"

   # Edit .env file and replace N8N_ENCRYPTION_KEY value
   # You can use: sed -i "s/your-super-secure-encryption-key-here-change-this/$ENCRYPTION_KEY/" .env
   ```

3. **Start Services**

   ```bash
   # Start in detached mode (background)
   docker compose -f docker-compose-n8n-local.yml up -d

   # Or start with logs visible
   docker compose -f docker-compose-n8n-local.yml up
   ```

4. **Wait for Services**

   ```bash
   # Check service status
   docker compose -f docker-compose-n8n-local.yml ps

   # View logs
   docker compose -f docker-compose-n8n-local.yml logs -f n8n
   ```

5. **Access n8n**
   - Open http://localhost:5678 in your browser
   - Create your admin account on first access

## 🔄 Workflow Management

### Import Existing Workflows

```bash
# Workflows are stored in the workflows/ directory
# Import via n8n UI: Settings → Workflows → Import from File
```

### Available Workflows

- **Lead Capture Form**: AI-powered workflow that:
  - Monitors Gmail for web form messages
  - Uses OpenAI GPT-4 to extract lead information
  - Generates personalized reply suggestions
  - Sends messages to Telegram for approval
  - Sends approved/rejected emails automatically

**⚠️ Important:** After importing the workflow, **activate it** in n8n UI (toggle the "Active" switch in the workflow editor).

## 🛠️ Management Commands

### Start Services

```bash
docker compose -f docker-compose-n8n-local.yml up -d
```

### Stop Services

```bash
docker compose -f docker-compose-n8n-local.yml down
```

### View Logs

```bash
# All services
docker compose -f docker-compose-n8n-local.yml logs -f

# Specific service
docker compose -f docker-compose-n8n-local.yml logs -f n8n
docker compose -f docker-compose-n8n-local.yml logs -f postgres
```

### Update n8n

```bash
# Pull latest images
docker compose -f docker-compose-n8n-local.yml pull

# Restart services
docker compose -f docker-compose-n8n-local.yml down
docker compose -f docker-compose-n8n-local.yml up -d
```

### Reset Everything (⚠️ Destroys Data)

```bash
# Stop and remove all data
docker compose -f docker-compose-n8n-local.yml down -v

# Clean up unused images
docker image prune -f
```

## 🔧 Troubleshooting

### Common Issues

1. **Port Already in Use**

   ```bash
   # Change port in .env file
   N8N_PORT=5679

   # Restart services
   docker compose -f docker-compose-n8n-local.yml down
   docker compose -f docker-compose-n8n-local.yml up -d
   ```

2. **Database Connection Failed**

   ```bash
   # Check PostgreSQL logs
   docker compose -f docker-compose-n8n-local.yml logs postgres

   # Verify .env database settings
   cat .env | grep -E "(POSTGRES|DB_)"
   ```

3. **n8n Won't Start**

   ```bash
   # Check n8n logs
   docker compose -f docker-compose-n8n-local.yml logs n8n

   # Verify encryption key is set
   grep N8N_ENCRYPTION_KEY .env
   ```

4. **Slow Performance**
   - Ensure at least 2GB RAM available
   - Check disk space: `df -h`
   - Monitor resource usage: `docker stats`

### Health Checks

```bash
# Check service health
curl http://localhost:5678/healthz

# Check database connectivity
docker compose -f docker-compose-n8n-local.yml exec postgres pg_isready -U n8n -d n8n
```

## 🔒 Security Considerations

- **Change default passwords** in `.env`
- **Use HTTPS in production** with reverse proxy
- **Regular backups** of `./data` directory
- **Monitor logs** for security events
- **Keep n8n updated** with latest security patches

## 📊 Monitoring

### Resource Usage

```bash
# Monitor Docker containers
docker stats

# Check disk usage
du -sh data/ local-files/
```

### n8n Metrics

- Access n8n UI at http://localhost:5678
- View execution history and logs
- Monitor workflow performance

## 🔄 Backup and Restore

### Backup

```bash
# Create backup directory
mkdir -p backups/$(date +%Y%m%d)

# Backup n8n data
cp -r data/ backups/$(date +%Y%m%d)/

# Backup workflows
cp -r workflows/ backups/$(date +%Y%m%d)/

# Backup database (if needed)
docker compose -f docker-compose-n8n-local.yml exec postgres pg_dump -U n8n n8n > backups/$(date +%Y%m%d)/n8n.sql
```

### Restore

```bash
# Stop services
docker compose -f docker-compose-n8n-local.yml down

# Restore data
cp -r backups/20231201/data/* data/

# Start services
docker compose -f docker-compose-n8n-local.yml up -d
```

## 🔗 Useful Links

- [n8n Documentation](https://docs.n8n.io/)
- [n8n Community](https://community.n8n.io/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
