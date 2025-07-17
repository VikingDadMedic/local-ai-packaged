# LocalAI Production Deployment Checklist

**Deployment Date:** _______________  
**Deployed By:** _______________  
**VM IP Address:** _______________  
**Domain:** _______________

## Pre-Deployment Phase

### VM Setup
- [ ] VM provisioned with Ubuntu 22.04 LTS
- [ ] Minimum specs verified:
  - [ ] 8+ vCPUs
  - [ ] 32GB+ RAM  
  - [ ] 200GB+ SSD
  - [ ] GPU (if applicable): _______________
- [ ] Root SSH access established
- [ ] VM IP noted: _______________

### Domain & DNS
- [ ] Domain registered/available
- [ ] DNS management access confirmed
- [ ] Planned subdomains:
  - [ ] n8n._______________
  - [ ] openwebui._______________
  - [ ] flowise._______________
  - [ ] supabase._______________
  - [ ] langfuse._______________

## Security Setup Phase

### User & Access
- [ ] Non-root user created: _______________
- [ ] User added to docker group
- [ ] SSH key authentication configured
- [ ] Password authentication disabled

### Firewall
- [ ] UFW installed
- [ ] Default deny incoming
- [ ] Port 22 (SSH) allowed from: _______________
- [ ] Port 80 (HTTP) allowed
- [ ] Port 443 (HTTPS) allowed
- [ ] UFW enabled

### Security Tools
- [ ] fail2ban installed
- [ ] fail2ban SSH jail configured
- [ ] unattended-upgrades configured

## Installation Phase

### Docker Setup
- [ ] Docker installed (version: _______________)
- [ ] Docker Compose installed (version: _______________)
- [ ] Docker daemon configured
- [ ] NVIDIA runtime installed (if GPU)

### Repository Setup
- [ ] Repository cloned to: _______________
- [ ] Ownership set correctly
- [ ] .env file created from example

### Environment Configuration
**CRITICAL: No @ symbols in passwords!**

- [ ] POSTGRES_PASSWORD: _______________
- [ ] N8N_ENCRYPTION_KEY (32 hex): Generated
- [ ] N8N_USER_MANAGEMENT_JWT_SECRET: Generated
- [ ] JWT_SECRET (Supabase): Generated
- [ ] ANON_KEY: Generated
- [ ] SERVICE_ROLE_KEY: Generated
- [ ] All hostnames configured
- [ ] LETSENCRYPT_EMAIL: _______________

## Deployment Phase

### Service Deployment
- [ ] Deployment command: `python3 start_services.py --profile _______________ --environment _______________`
- [ ] All containers started successfully
- [ ] No error messages in logs

### DNS Configuration
- [ ] A records created:
  - [ ] n8n → _______________
  - [ ] openwebui → _______________
  - [ ] flowise → _______________
  - [ ] supabase → _______________
  - [ ] langfuse → _______________
- [ ] DNS propagation verified

### SSL Verification
- [ ] All services accessible via HTTPS
- [ ] SSL certificates valid
- [ ] No certificate warnings

## Post-Deployment Phase

### Service Configuration
- [ ] n8n admin account created
- [ ] Open WebUI admin account created
- [ ] Flowise login tested
- [ ] Ollama models downloaded:
  - [ ] qwen2.5:7b-instruct-q4_K_M
  - [ ] nomic-embed-text
  - [ ] _______________ (optional)

### Integration Setup
- [ ] n8n workflows imported
- [ ] n8n webhook URLs documented
- [ ] Open WebUI n8n function configured
- [ ] Flowise custom tools imported
- [ ] Test queries successful

### Backup Configuration
- [ ] Backup script tested
- [ ] Backup location verified
- [ ] Backup encryption password stored securely
- [ ] Cron job configured

### Monitoring Setup
- [ ] Health check script tested
- [ ] Monitoring stack deployed (optional)
- [ ] Alerts configured (optional)

## Documentation

### Credentials Documentation
**Store in secure password manager!**

- [ ] VM SSH credentials
- [ ] Service admin passwords
- [ ] Database passwords
- [ ] API keys and tokens
- [ ] Backup encryption password

### Operational Documentation
- [ ] Team members trained
- [ ] Runbook created
- [ ] Emergency contacts listed
- [ ] Support channels documented

## Sign-off

**Deployment completed successfully:** ⬜ Yes ⬜ No

**Issues encountered:** 
_________________________________
_________________________________
_________________________________

**Notes:**
_________________________________
_________________________________
_________________________________

**Deployed by:** _______________ **Date:** _______________

**Reviewed by:** _______________ **Date:** _______________

---

## Quick Support

- **Documentation:** [PRODUCTION_DEPLOYMENT_GUIDE.md](PRODUCTION_DEPLOYMENT_GUIDE.md)
- **Quick Reference:** [QUICK_REFERENCE.md](QUICK_REFERENCE.md)
- **Community:** https://thinktank.ottomator.ai/c/local-ai/18
- **Emergency Contact:** _______________ 