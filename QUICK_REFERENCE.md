# LocalAI Quick Reference Guide

## 🚀 Essential Commands

### Service Management
```bash
# Check all services
docker compose -p localai ps

# View logs (all services)
docker compose -p localai logs -f

# View specific service logs
docker compose -p localai logs n8n --tail=100

# Restart a service
docker compose -p localai restart n8n

# Stop all services
docker compose -p localai down

# Start all services
python3 start_services.py --profile gpu-nvidia --environment public
```

### Quick Diagnostics
```bash
# Check service health
~/health-check.sh

# Check resource usage
docker stats --no-stream

# Diagnose startup issues
~/diagnose-startup.sh

# Analyze logs
~/analyze-logs.sh n8n error
```

### Backup Operations
```bash
# Create backup
~/backup-localai.sh

# Restore from backup
~/rollback-localai.sh 20241220_020000
```

### Updates
```bash
# Update all services
~/update-localai.sh

# Update Ollama models only
~/update-ollama-models.sh

# Check for updates (dry run)
docker compose -p localai pull --dry-run
```

## 🌐 Service URLs

| Service | URL | Default Login |
|---------|-----|---------------|
| n8n | https://n8n.yourdomain.com | Set on first access |
| Open WebUI | https://openwebui.yourdomain.com | Set on first access |
| Flowise | https://flowise.yourdomain.com | From .env file |
| Supabase | https://supabase.yourdomain.com | From .env file |
| Langfuse | https://langfuse.yourdomain.com | Set on first access |

## 🔧 Common Fixes

### SSL Certificate Issues
```bash
~/fix-ssl.sh
```

### Database Connection Issues
```bash
~/fix-database.sh
```

### Memory Issues
```bash
~/fix-memory.sh
```

### Service Won't Start
```bash
# Check ports
sudo netstat -tlnp | grep -E "80|443|5678|8080"

# Check environment
grep -E "^[A-Z_]+=$" .env

# Fix permissions
sudo chown -R $USER:docker ~/local-ai-packaged
```

## 📊 Monitoring

### Real-time Performance
```bash
~/performance-monitor.sh
```

### Service Metrics
```bash
# Prometheus metrics
curl http://localhost:9090/metrics

# Grafana dashboards
http://localhost:3005
```

### Container Stats
```bash
docker compose -p localai exec n8n cat /proc/meminfo
docker compose -p localai exec ollama nvidia-smi
```

## 🆘 Emergency Procedures

### 1. Service Outage
```bash
# Quick restart
docker compose -p localai restart

# Full restart
docker compose -p localai down
docker compose -p localai up -d
```

### 2. Database Lock
```bash
docker compose -p localai exec db psql -U postgres -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE state = 'idle';"
```

### 3. Out of Memory
```bash
# Clear Docker cache
docker system prune -af --volumes

# Restart with limits
docker compose -p localai up -d
```

### 4. Corrupted Container
```bash
# Remove and recreate
docker compose -p localai rm -f [service-name]
docker compose -p localai up -d [service-name]
```

## 📝 Configuration Files

| File | Purpose |
|------|---------|
| `.env` | Environment variables |
| `docker-compose.yml` | Service definitions |
| `docker-compose.*.yml` | Profile overrides |
| `caddy-addon/Caddyfile` | Reverse proxy config |
| `prometheus.yml` | Monitoring config |

## 🔑 Important Paths

| Path | Contents |
|------|----------|
| `~/local-ai-packaged/` | Main installation |
| `~/backups/` | Backup files |
| `/var/lib/docker/volumes/` | Docker volumes |
| `~/.n8n/` | n8n config (in container) |

---
**Need more help?** Check the full [Production Deployment Guide](PRODUCTION_DEPLOYMENT_GUIDE.md) 