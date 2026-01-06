
# Reboot sequence

## **Pre-Reboot:**

```bash
docker compose -f docker-compose-n8n-local.yml down
```

## **Post-Reboot:**

1. Start ngrok: `ngrok http 5678`
2. Update .env with new URL (if changed)
3. Start n8n: `docker compose up -d`
4. Verify: `docker compose ps`
5. Access: http://localhost:5678

### **Important Notes:**

- Docker volumes persist
- ngrok URLs change on free tier
- Workflows remain active

## 🎯 **Now Complete:**

Yes, **stopping n8n is sufficient** for a Windows reboot. Docker containers will automatically stop when Windows shuts down, and ngrok will also terminate.

Here's the complete reboot procedure:

## 🛑 **Pre-Reboot Steps:**

```bash
# Stop n8n services (this is the key step)
docker compose -f docker-compose-n8n-local.yml down

# Optional: Close ngrok terminal (but not required)
# Windows will terminate it automatically
```

## 🔄 **After Reboot:**

```bash
# Start ngrok first (in one terminal)
ngrok http 5678
# Copy the new HTTPS URL

# Update .env with new ngrok URL (if changed)
# N8N_EDITOR_BASE_URL=https://new-ngrok-url.ngrok.io
# WEBHOOK_URL=https://new-ngrok-url.ngrok.io

# Start n8n (in another terminal)
docker compose -f docker-compose-n8n-local.yml up -d

# Check status
docker compose -f docker-compose-n8n-local.yml ps
```

## ⚠️ **Important Notes:**

- **ngrok URLs change** on reboot (free tier), so you'll need to update `.env` if the URL changed
- **Docker volumes persist** - your n8n data and PostgreSQL database will be preserved
- **Workflows remain active** - they'll pick up where they left off after restart

## 🚀 **Quick Post-Reboot Checklist:**

1. ✅ Start ngrok: `ngrok http 5678`
2. ✅ Update `.env` with new ngrok URL (if changed)
3. ✅ Start n8n: `docker compose -f docker-compose-n8n-local.yml up -d`
4. ✅ Access n8n: http://localhost:5678
5. ✅ Test workflow: Send a test email

**Just stopping n8n with `docker compose down` is all you need before rebooting!** 🎯
