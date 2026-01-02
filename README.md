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

```
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

```
local-n8n-with-docker-compose/
├── docker-compose-n8n-local.yml    # Docker Compose configuration
├── .env.example                    # Environment variables template
├── .env                           # Your environment variables (create from .env.example)
├── README.md                      # This file
├── workflows/                     # n8n workflow files
│   └── Lead capture form 5 minutes auto.json
├── data/                          # n8n data persistence (created automatically)
└── local-files/                   # File storage (created automatically)
```

## ⚙️ Configuration

### Environment Variables (.env)

Copy `.env.example` to `.env` and configure:

```bash
# Basic Configuration
N8N_HOST=0.0.0.0
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

- Docker and Docker Compose installed
- At least 2GB free RAM
- Ports 5678 and 5432 available (PostgreSQL internal)

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

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## 📝 License

This project is provided as-is for educational and development purposes.

## 🔗 Useful Links

- [n8n Documentation](https://docs.n8n.io/)
- [n8n Community](https://community.n8n.io/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
