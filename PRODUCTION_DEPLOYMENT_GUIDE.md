# Local AI Package - Production Deployment Guide

> **Version**: 1.0  
> **Last Updated**: December 2024  
> **Deployment Time**: ~2-3 hours  
> **Difficulty**: Intermediate to Advanced  

## Executive Summary

This guide provides comprehensive instructions for deploying the Local AI Package to a production environment. The deployment includes a complete AI and automation stack featuring n8n, Supabase, Ollama, Open WebUI, and other essential services, all secured with automatic SSL/TLS and optimized for production workloads.

### Key Features
- 🔒 **Security-First**: SSL/TLS encryption, firewall configuration, secrets management
- 🚀 **Production-Ready**: Monitoring, backups, performance optimization
- 🤖 **AI-Powered**: Local LLM inference, RAG implementation, vector search
- 🔧 **Automation**: Low-code workflows, webhook integrations, custom tools
- 📊 **Observability**: Comprehensive logging, metrics, and monitoring

### Prerequisites Summary
- **VM**: Ubuntu 22.04 LTS, 8+ vCPUs, 32GB+ RAM, 200GB+ SSD
- **Domain**: Full DNS control with ability to create subdomains
- **Skills**: Basic Linux, Docker, and networking knowledge

## Table of Contents

1. [Overview](#overview)
   - [What's Included](#whats-included)
   - [Quick Start](#quick-start)
2. [VM Requirements and Prerequisites](#vm-requirements-and-prerequisites)
   - [Minimum VM Specifications](#minimum-vm-specifications)
   - [Resource Allocation Breakdown](#resource-allocation-breakdown)
   - [Software Prerequisites](#software-prerequisites)
   - [Network Requirements](#network-requirements)
   - [Cloud Provider Recommendations](#cloud-provider-recommendations)
3. [Security Hardening](#security-hardening)
   - [Pre-Deployment Security](#pre-deployment-security)
   - [System Hardening](#system-hardening)
   - [Docker Security](#docker-security)
   - [Network Security](#network-security)
   - [SSL/TLS Configuration](#ssltls-configuration)
   - [Secrets Management](#secrets-management)
4. [Step-by-Step Deployment](#step-by-step-deployment)
   - [Phase 1: VM Setup](#phase-1-vm-setup)
   - [Phase 2: System Configuration](#phase-2-system-configuration)
   - [Phase 3: Docker Installation](#phase-3-docker-installation)
   - [Phase 4: Repository Setup](#phase-4-repository-setup)
   - [Phase 5: Environment Configuration](#phase-5-environment-configuration)
   - [Phase 6: Service Deployment](#phase-6-service-deployment)
   - [Phase 7: Verification](#phase-7-verification)
5. [Post-Deployment Configuration](#post-deployment-configuration)
   - [Service Integration Setup](#service-integration-setup)
   - [Webhook Configuration](#webhook-configuration)
   - [Advanced Integration Setup](#advanced-integration-setup)
6. [Maintenance and Operations](#maintenance-and-operations)
   - [Daily Tasks](#daily-tasks)
   - [Weekly Tasks](#weekly-tasks)
   - [Monitoring and Alerting Setup](#monitoring-and-alerting-setup)
   - [Update and Upgrade Procedures](#update-and-upgrade-procedures)
   - [Monthly Tasks](#monthly-tasks)
   - [Performance Tuning Guide](#performance-tuning-guide)
   - [Disaster Recovery](#disaster-recovery)
7. [Troubleshooting and FAQ](#troubleshooting-and-faq)
   - [Service-Specific Troubleshooting](#service-specific-troubleshooting)
   - [Log Analysis and Debugging Guide](#log-analysis-and-debugging-guide)
   - [Common Issues](#common-issues)
   - [Performance Optimization](#performance-optimization)
   - [FAQ](#faq)
8. [Architecture Diagrams](#architecture-diagrams)
   - [System Architecture](#system-architecture)
   - [Data Flow](#data-flow)
   - [Backup Architecture](#backup-architecture)
   - [Network Topology](#network-topology)
9. [Additional Resources](#additional-resources)
10. [Appendix](#appendix)
    - [Environment Variables Reference](#environment-variables-reference)
    - [Port Reference](#port-reference)
    - [Command Quick Reference](#command-quick-reference)

## Overview

This guide provides comprehensive instructions for deploying the Local AI Package to a production environment. The deployment includes multiple AI and low-code services orchestrated through Docker Compose, secured with Caddy reverse proxy, and optimized for production workloads.

### What's Included

- **n8n** - Low-code workflow automation platform
- **Supabase** - Open source Firebase alternative (PostgreSQL, Auth, Storage)
- **Ollama** - Local LLM runtime
- **Open WebUI** - ChatGPT-like interface
- **Flowise** - No/low code AI agent builder
- **Qdrant** - High-performance vector database
- **Neo4j** - Graph database for knowledge graphs
- **Langfuse** - LLM observability platform
- **SearXNG** - Privacy-respecting metasearch engine
- **Caddy** - Automatic HTTPS reverse proxy

### Quick Start

For experienced users who want to deploy quickly:

```bash
# 1. SSH into your VM
ssh root@YOUR_VM_IP

# 2. Clone and setup
git clone https://github.com/yourusername/local-ai-packaged.git
cd local-ai-packaged
cp .env.example .env

# 3. Configure essentials (edit .env)
# - Set all passwords (avoid @ symbol)
# - Configure domain names
# - Generate secure keys

# 4. Deploy
python3 start_services.py --profile gpu-nvidia --environment public

# 5. Configure DNS
# Point *.yourdomain.com to your VM IP

# 6. Access services
# - n8n: https://n8n.yourdomain.com
# - Open WebUI: https://openwebui.yourdomain.com
# - Flowise: https://flowise.yourdomain.com
```

⚠️ **Important**: This quick start assumes you have:
- Ubuntu 22.04 LTS with Docker installed
- A domain with DNS control
- Basic knowledge of Docker and Linux administration

For detailed instructions, continue reading below.

### Deployment Checklist

Use this checklist to track your deployment progress:

**Pre-Deployment**
- [ ] VM provisioned with required specifications
- [ ] Domain registered and DNS access configured
- [ ] SSH access to VM established
- [ ] Backup of any existing data completed

**Security Setup**
- [ ] Non-root user created with sudo privileges
- [ ] SSH hardened (key-only authentication)
- [ ] UFW firewall configured
- [ ] Fail2ban installed and configured
- [ ] All passwords generated (32+ characters, no @ symbol)

**Installation**
- [ ] Docker and Docker Compose installed
- [ ] NVIDIA drivers installed (if using GPU)
- [ ] Repository cloned
- [ ] Environment variables configured
- [ ] Services deployed successfully

**Post-Deployment**
- [ ] DNS records created for all subdomains
- [ ] SSL certificates generated and verified
- [ ] All services accessible via HTTPS
- [ ] n8n workflows imported
- [ ] Ollama models downloaded
- [ ] Backup script configured and tested
- [ ] Monitoring stack deployed
- [ ] Health checks passing

**Documentation**
- [ ] Admin credentials documented securely
- [ ] Backup procedures documented
- [ ] Team members trained on basic operations

## VM Requirements and Prerequisites

### Minimum VM Specifications

#### Basic Deployment (CPU-only)
- **CPU**: 8 vCPUs (Intel/AMD x86_64)
- **RAM**: 32GB
- **Storage**: 200GB SSD
- **Network**: 100 Mbps dedicated bandwidth
- **OS**: Ubuntu 22.04 LTS

#### GPU-Accelerated Deployment
- **CPU**: 8 vCPUs (Intel/AMD x86_64)
- **RAM**: 32GB (48GB recommended)
- **GPU**: NVIDIA GPU with 8GB+ VRAM (T4, V100, A10, RTX 3080+)
- **Storage**: 500GB SSD (NVMe preferred)
- **Network**: 100 Mbps dedicated bandwidth
- **OS**: Ubuntu 22.04 LTS with NVIDIA drivers

#### Production Recommended Specifications
- **CPU**: 16 vCPUs
- **RAM**: 64GB
- **GPU**: NVIDIA GPU with 16GB+ VRAM (for better LLM performance)
- **Storage**: 1TB NVMe SSD
- **Network**: 1 Gbps dedicated bandwidth
- **OS**: Ubuntu 22.04 LTS

### Resource Allocation Breakdown

| Service | CPU (cores) | RAM (GB) | Storage (GB) | Notes |
|---------|------------|----------|--------------|-------|
| n8n | 2 | 4 | 20 | Scales with workflow complexity |
| Supabase (PostgreSQL) | 2 | 8 | 50 | Database for all services |
| Ollama | 4 | 16 | 100 | More for larger models |
| Open WebUI | 1 | 2 | 10 | Frontend only |
| Flowise | 1 | 2 | 10 | Agent builder |
| Qdrant | 2 | 4 | 50 | Vector storage |
| Neo4j | 2 | 4 | 30 | Graph database |
| Langfuse | 2 | 4 | 20 | Includes ClickHouse, MinIO |
| SearXNG | 1 | 1 | 5 | Search engine |
| Caddy | 0.5 | 0.5 | 5 | Reverse proxy |
| Redis | 0.5 | 2 | 10 | Caching layer |
| **Total** | **18.0** | **47.5** | **310** | Plus OS overhead |

### Software Prerequisites

#### Required Software
1. **Operating System**: Ubuntu 22.04 LTS (recommended) or 20.04 LTS
2. **Docker Engine**: Version 24.0.0 or higher
3. **Docker Compose**: Version 2.20.0 or higher
4. **Git**: Version 2.34.0 or higher
5. **Python**: Version 3.8 or higher (for start_services.py)
6. **OpenSSL**: For generating secure keys
7. **UFW**: Ubuntu firewall (pre-installed)

#### Optional Software
- **htop**: System monitoring
- **ncdu**: Disk usage analysis
- **fail2ban**: Brute force protection
- **unattended-upgrades**: Automatic security updates

### Network Requirements

#### DNS Requirements
You'll need a domain with the ability to create multiple A records:
- `n8n.yourdomain.com`
- `openwebui.yourdomain.com`
- `flowise.yourdomain.com`
- `supabase.yourdomain.com`
- `langfuse.yourdomain.com`
- `neo4j.yourdomain.com`
- `searxng.yourdomain.com` (optional)

#### Firewall Ports
- **Inbound (Required)**:
  - Port 80 (HTTP - redirects to HTTPS)
  - Port 443 (HTTPS - all services)
  - Port 22 (SSH - restrict to your IP)

- **Outbound (Required)**:
  - Port 443 (HTTPS - for external APIs, Docker Hub)
  - Port 80 (HTTP - for package updates)
  - Port 53 (DNS)

#### SSL/TLS Requirements
- Valid email address for Let's Encrypt certificates
- Caddy will automatically handle certificate generation and renewal

### Cloud Provider Recommendations

#### AWS
- **Instance Type**: t3.2xlarge (basic) or g4dn.xlarge (GPU)
- **Storage**: GP3 SSD with 3000 IOPS
- **Region**: Choose based on user location

#### Google Cloud
- **Instance Type**: n2-standard-8 (basic) or n1-standard-8 with T4 GPU
- **Storage**: SSD persistent disk
- **Region**: Choose based on user location

#### DigitalOcean
- **Droplet**: General Purpose 32GB/8vCPU
- **GPU Droplets**: For GPU workloads
- **Storage**: Add block storage volumes as needed

#### Azure
- **Instance Type**: Standard_D8s_v3 (basic) or Standard_NC6s_v3 (GPU)
- **Storage**: Premium SSD
- **Region**: Choose based on user location 

## Security Hardening

### Environment Variable Security

#### Generating Secure Secrets

Always generate cryptographically secure secrets for production. Never use default or example values.

```bash
# Generate secure passwords and keys
generate_secret() {
    openssl rand -hex 32
}

# Generate all required secrets
export POSTGRES_PASSWORD=$(generate_secret)
export JWT_SECRET=$(generate_secret)
export N8N_ENCRYPTION_KEY=$(generate_secret)
export N8N_USER_MANAGEMENT_JWT_SECRET=$(generate_secret)
export LANGFUSE_SALT=$(generate_secret)
export NEXTAUTH_SECRET=$(generate_secret)
export ENCRYPTION_KEY=$(generate_secret)
export CLICKHOUSE_PASSWORD=$(generate_secret)
export MINIO_ROOT_PASSWORD=$(generate_secret)
export NEO4J_AUTH="neo4j/$(generate_secret)"
export DASHBOARD_PASSWORD=$(generate_secret)
export FLOWISE_PASSWORD=$(generate_secret)
```

#### Supabase API Keys Generation

Generate Supabase JWT keys using their official tool:

```bash
# Install Supabase CLI
npm install -g @supabase/cli

# Generate API keys
supabase bootstrap gen keys

# Or use the online tool: https://supabase.com/docs/guides/self-hosting/docker#generate-api-keys
```

#### Password Requirements

- Minimum 32 characters for all secrets
- Avoid special characters in PostgreSQL password (especially '@', '%', '#')
- Use only alphanumeric characters for database passwords
- Store all credentials in a secure password manager
- Never commit secrets to version control

### System Security Hardening

#### 1. Create Non-Root User

```bash
# Create a dedicated user for the application
sudo adduser localai --disabled-password --gecos ""
sudo usermod -aG docker localai
sudo usermod -aG sudo localai

# Set strong password
sudo passwd localai
```

#### 2. SSH Hardening

```bash
# Backup SSH config
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# Edit SSH configuration
sudo nano /etc/ssh/sshd_config
```

Add/modify these settings:
```
# Disable root login
PermitRootLogin no

# Use SSH key authentication only
PasswordAuthentication no
PubkeyAuthentication yes

# Limit user access
AllowUsers localai

# Change default port (optional)
Port 2222

# Disable empty passwords
PermitEmptyPasswords no

# Limit authentication attempts
MaxAuthTries 3

# Disable X11 forwarding
X11Forwarding no
```

```bash
# Restart SSH service
sudo systemctl restart sshd
```

#### 3. Firewall Configuration (UFW)

```bash
# Enable UFW
sudo ufw enable

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (adjust port if changed)
sudo ufw allow from YOUR_IP_ADDRESS to any port 22
# Or if you changed SSH port:
# sudo ufw allow from YOUR_IP_ADDRESS to any port 2222

# Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Reload firewall
sudo ufw reload

# Check status
sudo ufw status verbose
```

**WARNING**: Docker bypasses UFW rules. To secure Docker ports:

```bash
# Create Docker UFW rules file
sudo nano /etc/ufw/after.rules
```

Add at the end:
```
# BEGIN UFW AND DOCKER
*filter
:ufw-user-forward - [0:0]
:DOCKER-USER - [0:0]
-A DOCKER-USER -j RETURN -s 10.0.0.0/8
-A DOCKER-USER -j RETURN -s 172.16.0.0/12
-A DOCKER-USER -j RETURN -s 192.168.0.0/16
-A DOCKER-USER -j ufw-user-forward
-A DOCKER-USER -j DROP -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -d 192.168.0.0/16
-A DOCKER-USER -j DROP -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -d 10.0.0.0/8
-A DOCKER-USER -j DROP -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -d 172.16.0.0/12
-A DOCKER-USER -j DROP -p udp -m udp --dport 0:32767 -d 192.168.0.0/16
-A DOCKER-USER -j DROP -p udp -m udp --dport 0:32767 -d 10.0.0.0/8
-A DOCKER-USER -j DROP -p udp -m udp --dport 0:32767 -d 172.16.0.0/12
-A DOCKER-USER -j RETURN
COMMIT
# END UFW AND DOCKER
```

#### 4. Fail2ban Installation

```bash
# Install fail2ban
sudo apt-get update
sudo apt-get install -y fail2ban

# Create local config
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# Edit configuration
sudo nano /etc/fail2ban/jail.local
```

Add custom jails:
```ini
[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600

[nginx-http-auth]
enabled = true
filter = nginx-http-auth
port = http,https
logpath = /var/log/nginx/error.log

[nginx-noscript]
enabled = true
port = http,https
filter = nginx-noscript
logpath = /var/log/nginx/access.log
maxretry = 6

[nginx-badbots]
enabled = true
port = http,https
filter = nginx-badbots
logpath = /var/log/nginx/access.log
maxretry = 2

[nginx-noproxy]
enabled = true
port = http,https
filter = nginx-noproxy
logpath = /var/log/nginx/access.log
maxretry = 2
```

```bash
# Start and enable fail2ban
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

### Docker Security

#### 1. Docker Daemon Security

```bash
# Create Docker daemon config
sudo nano /etc/docker/daemon.json
```

Add:
```json
{
  "icc": false,
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "userland-proxy": false,
  "no-new-privileges": true,
  "live-restore": true
}
```

```bash
# Restart Docker
sudo systemctl restart docker
```

#### 2. Docker Compose Security

Ensure the `--environment public` flag is used:
```bash
python3 start_services.py --profile gpu-nvidia --environment public
```

This ensures:
- Only ports 80 and 443 are exposed
- All other service ports are internal only
- Services communicate through Docker networks

### SSL/TLS Configuration

#### 1. Domain Setup

Configure DNS A records for all subdomains:
```
n8n.yourdomain.com          → YOUR_SERVER_IP
openwebui.yourdomain.com    → YOUR_SERVER_IP
flowise.yourdomain.com      → YOUR_SERVER_IP
supabase.yourdomain.com     → YOUR_SERVER_IP
langfuse.yourdomain.com     → YOUR_SERVER_IP
neo4j.yourdomain.com        → YOUR_SERVER_IP
```

#### 2. Caddy Configuration

Update `.env` with your domain information:
```bash
# Production Caddy configuration
N8N_HOSTNAME=n8n.yourdomain.com
WEBUI_HOSTNAME=openwebui.yourdomain.com
FLOWISE_HOSTNAME=flowise.yourdomain.com
SUPABASE_HOSTNAME=supabase.yourdomain.com
LANGFUSE_HOSTNAME=langfuse.yourdomain.com
NEO4J_HOSTNAME=neo4j.yourdomain.com
LETSENCRYPT_EMAIL=admin@yourdomain.com
```

Caddy automatically:
- Obtains Let's Encrypt certificates
- Renews certificates before expiration
- Redirects HTTP to HTTPS
- Implements modern TLS settings

#### 3. Additional Caddy Security Headers

Create `caddy-addon/security.conf`:
```
(security) {
    header {
        # HSTS
        Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
        
        # Prevent clickjacking
        X-Frame-Options "DENY"
        
        # Prevent MIME type sniffing
        X-Content-Type-Options "nosniff"
        
        # XSS Protection (legacy browsers)
        X-XSS-Protection "1; mode=block"
        
        # Referrer Policy
        Referrer-Policy "strict-origin-when-cross-origin"
        
        # Permissions Policy
        Permissions-Policy "geolocation=(), microphone=(), camera=()"
        
        # Remove server header
        -Server
    }
}

# Apply to all sites
*.yourdomain.com {
    import security
}
```

### Application-Level Security

#### 1. Service Authentication

Configure authentication for each service:

```bash
# n8n - Set in first-time setup
# Open WebUI - Set in first-time setup
# Flowise
FLOWISE_USERNAME=admin
FLOWISE_PASSWORD=$(generate_secret)

# Supabase Studio
DASHBOARD_USERNAME=supabase
DASHBOARD_PASSWORD=$(generate_secret)

# Neo4j
NEO4J_AUTH="neo4j/$(generate_secret)"
```

#### 2. API Security

- Always use bearer tokens for API authentication
- Rotate API keys regularly
- Implement rate limiting where possible
- Use webhook signatures for verification

#### 3. Database Security

```sql
-- Create read-only user for analytics
CREATE USER analytics_user WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE postgres TO analytics_user;
GRANT USAGE ON SCHEMA public TO analytics_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO analytics_user;

-- Revoke unnecessary permissions
REVOKE CREATE ON SCHEMA public FROM PUBLIC;
```

### Security Monitoring

#### 1. Log Monitoring

```bash
# Monitor authentication attempts
sudo tail -f /var/log/auth.log

# Monitor Caddy access logs
docker compose -p localai logs -f caddy

# Monitor all services
docker compose -p localai logs -f
```

#### 2. Automated Security Updates

```bash
# Install unattended-upgrades
sudo apt-get install unattended-upgrades

# Enable automatic updates
sudo dpkg-reconfigure --priority=low unattended-upgrades

# Configure update schedule
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

### Security Checklist

- [ ] All default passwords changed
- [ ] Secure secrets generated (32+ characters)
- [ ] Non-root user created
- [ ] SSH hardened and key-only authentication
- [ ] Firewall enabled with minimal ports
- [ ] Docker daemon secured
- [ ] SSL/TLS certificates configured
- [ ] Service authentication enabled
- [ ] Automatic updates configured
- [ ] Backup encryption enabled
- [ ] Log monitoring in place
- [ ] Fail2ban configured
- [ ] No sensitive data in environment files committed to git

### Incident Response Plan

1. **Detection**: Monitor logs for suspicious activity
2. **Containment**: Isolate affected services
3. **Investigation**: Analyze logs and system state
4. **Remediation**: Apply fixes and patches
5. **Recovery**: Restore from backups if needed
6. **Documentation**: Document incident and response

### Regular Security Tasks

- **Daily**: Check service logs for errors/warnings
- **Weekly**: Review authentication logs
- **Monthly**: Update all containers
- **Quarterly**: Rotate all secrets and API keys
- **Annually**: Full security audit

## Step-by-Step Deployment

### Phase 1: Initial VM Setup

#### 1.1 Create VM Instance

**AWS EC2:**
```bash
# Using AWS CLI
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \  # Ubuntu 22.04 LTS
  --instance-type t3.2xlarge \
  --key-name your-key-pair \
  --security-group-ids sg-xxxxxxxx \
  --subnet-id subnet-xxxxxxxx \
  --block-device-mappings '[{"DeviceName":"/dev/sda1","Ebs":{"VolumeSize":200,"VolumeType":"gp3"}}]'
```

**DigitalOcean:**
```bash
doctl compute droplet create localai-prod \
  --size s-8vcpu-32gb \
  --image ubuntu-22-04-x64 \
  --region nyc3 \
  --ssh-keys your-ssh-key-id
```

**Manual Setup:**
- Choose Ubuntu 22.04 LTS
- Select appropriate instance size (see VM Requirements)
- Configure security group/firewall (ports 22, 80, 443)
- Attach SSH key for access

#### 1.2 Initial System Configuration

```bash
# Connect to VM
ssh ubuntu@YOUR_VM_IP

# Update system
sudo apt-get update && sudo apt-get upgrade -y

# Set hostname
sudo hostnamectl set-hostname localai-prod

# Configure timezone
sudo timedatectl set-timezone UTC

# Create swap file (recommended for memory-intensive operations)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Install essential packages
sudo apt-get install -y \
  curl \
  wget \
  git \
  nano \
  htop \
  ncdu \
  net-tools \
  software-properties-common \
  apt-transport-https \
  ca-certificates \
  gnupg \
  lsb-release \
  python3-pip \
  unzip
```

#### 1.3 Create Application User

```bash
# Create dedicated user
sudo adduser localai --disabled-password --gecos "Local AI Application"

# Add to required groups
sudo usermod -aG sudo localai
sudo usermod -aG docker localai  # Will add docker group after Docker installation

# Set up SSH for new user
sudo mkdir -p /home/localai/.ssh
sudo cp ~/.ssh/authorized_keys /home/localai/.ssh/
sudo chown -R localai:localai /home/localai/.ssh
sudo chmod 700 /home/localai/.ssh
sudo chmod 600 /home/localai/.ssh/authorized_keys

# Switch to new user
sudo su - localai
```

### Phase 2: Security Hardening

#### 2.1 SSH Configuration

```bash
# Backup original config
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# Create new config
sudo tee /etc/ssh/sshd_config > /dev/null <<EOF
# SSH Server Configuration - Production Hardened
Port 22
Protocol 2
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key

# Logging
SyslogFacility AUTH
LogLevel INFO

# Authentication
LoginGraceTime 30
PermitRootLogin no
StrictModes yes
MaxAuthTries 3
MaxSessions 10

PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys

PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no

# Disable unused authentication methods
KerberosAuthentication no
GSSAPIAuthentication no
UsePAM yes

# Security restrictions
AllowUsers localai
X11Forwarding no
PrintMotd no
PrintLastLog yes
TCPKeepAlive yes
UsePrivilegeSeparation yes
Compression delayed
ClientAliveInterval 300
ClientAliveCountMax 2
UseDNS no

# Subsystems
Subsystem sftp /usr/lib/openssh/sftp-server
EOF

# Restart SSH
sudo systemctl restart sshd
```

#### 2.2 Firewall Setup

```bash
# Install and enable UFW
sudo apt-get install -y ufw

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (restrict to your IP if possible)
sudo ufw allow from any to any port 22 proto tcp
# Or restrict to specific IP: sudo ufw allow from YOUR_IP to any port 22

# Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable firewall
sudo ufw --force enable

# Add Docker-specific rules
sudo tee -a /etc/ufw/after.rules > /dev/null <<'EOF'

# BEGIN UFW AND DOCKER
*filter
:ufw-user-forward - [0:0]
:DOCKER-USER - [0:0]
-A DOCKER-USER -j RETURN -s 10.0.0.0/8
-A DOCKER-USER -j RETURN -s 172.16.0.0/12
-A DOCKER-USER -j RETURN -s 192.168.0.0/16
-A DOCKER-USER -j ufw-user-forward
-A DOCKER-USER -j DROP -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -d 192.168.0.0/16
-A DOCKER-USER -j DROP -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -d 10.0.0.0/8
-A DOCKER-USER -j DROP -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -d 172.16.0.0/12
-A DOCKER-USER -j DROP -p udp -m udp --dport 0:32767 -d 192.168.0.0/16
-A DOCKER-USER -j DROP -p udp -m udp --dport 0:32767 -d 10.0.0.0/8
-A DOCKER-USER -j DROP -p udp -m udp --dport 0:32767 -d 172.16.0.0/12
-A DOCKER-USER -j RETURN
COMMIT
# END UFW AND DOCKER
EOF

# Reload UFW
sudo ufw reload
```

#### 2.3 Fail2ban Installation

```bash
# Install fail2ban
sudo apt-get install -y fail2ban

# Create local configuration
sudo tee /etc/fail2ban/jail.local > /dev/null <<'EOF'
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3
destemail = admin@yourdomain.com
sendername = Fail2Ban
action = %(action_mwl)s

[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 3

[sshd-ddos]
enabled = true
port = 22
filter = sshd-ddos
logpath = /var/log/auth.log
maxretry = 6
EOF

# Start and enable fail2ban
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

### Phase 3: Docker Installation

#### 3.1 Install Docker Engine

```bash
# Remove old versions
sudo apt-get remove -y docker docker-engine docker.io containerd runc

# Install prerequisites
sudo apt-get update
sudo apt-get install -y \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg \
  lsb-release

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Set up stable repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Add user to docker group
sudo usermod -aG docker localai

# Configure Docker daemon
sudo tee /etc/docker/daemon.json > /dev/null <<'EOF'
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 64000,
      "Soft": 64000
    }
  },
  "live-restore": true,
  "userland-proxy": false,
  "no-new-privileges": true
}
EOF

# Restart Docker
sudo systemctl restart docker
sudo systemctl enable docker

# Verify installation
docker --version
docker compose version
```

#### 3.2 Install NVIDIA Docker (for GPU instances)

```bash
# Only run this section if you have a GPU instance

# Install NVIDIA driver
sudo apt-get update
sudo apt-get install -y ubuntu-drivers-common
sudo ubuntu-drivers autoinstall

# Add NVIDIA Docker repository
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

# Install NVIDIA Docker
sudo apt-get update
sudo apt-get install -y nvidia-docker2

# Restart Docker
sudo systemctl restart docker

# Test NVIDIA Docker
docker run --rm --gpus all nvidia/cuda:11.6.2-base-ubuntu20.04 nvidia-smi
```

### Phase 4: Repository Setup

#### 4.1 Clone Repository

```bash
# Switch to localai user if not already
sudo su - localai

# Clone the repository
cd ~
git clone -b stable https://github.com/coleam00/local-ai-packaged.git
cd local-ai-packaged

# Create required directories
mkdir -p caddy-addon
```

#### 4.2 Environment Configuration

```bash
# Copy example environment file
cp .env.example .env

# Generate secure secrets
generate_secret() {
    openssl rand -hex 32
}

# Create secure environment file
cat > .env.production <<EOF
############
# N8N Configuration
############
N8N_ENCRYPTION_KEY=$(generate_secret)
N8N_USER_MANAGEMENT_JWT_SECRET=$(generate_secret)

############
# Supabase Secrets
############
POSTGRES_PASSWORD=$(generate_secret | tr -d '@#%&*(){}[]|\\/<>?~`')
JWT_SECRET=$(generate_secret)
DASHBOARD_USERNAME=supabase
DASHBOARD_PASSWORD=$(generate_secret)
POOLER_TENANT_ID=1000
POOLER_DB_POOL_SIZE=5

############
# Neo4j Secrets
############
NEO4J_AUTH=neo4j/$(generate_secret)

############
# Langfuse credentials
############
CLICKHOUSE_PASSWORD=$(generate_secret)
MINIO_ROOT_PASSWORD=$(generate_secret)
LANGFUSE_SALT=$(generate_secret)
NEXTAUTH_SECRET=$(generate_secret)
ENCRYPTION_KEY=$(generate_secret)

############
# Flowise credentials
############
FLOWISE_USERNAME=admin
FLOWISE_PASSWORD=$(generate_secret)

############
# Caddy Config - UPDATE THESE WITH YOUR DOMAIN
############
N8N_HOSTNAME=n8n.yourdomain.com
WEBUI_HOSTNAME=openwebui.yourdomain.com
FLOWISE_HOSTNAME=flowise.yourdomain.com
SUPABASE_HOSTNAME=supabase.yourdomain.com
LANGFUSE_HOSTNAME=langfuse.yourdomain.com
NEO4J_HOSTNAME=neo4j.yourdomain.com
LETSENCRYPT_EMAIL=admin@yourdomain.com

# Optional services
# SEARXNG_HOSTNAME=searxng.yourdomain.com
EOF

# Generate Supabase keys
echo "Generating Supabase API keys..."
echo "Visit: https://supabase.com/docs/guides/self-hosting/docker#generate-api-keys"
echo "Add the generated ANON_KEY and SERVICE_ROLE_KEY to .env.production"

# Merge production env with example
grep -E '^(ANON_KEY|SERVICE_ROLE_KEY)=' .env.example >> .env.production

# Replace .env with production version
mv .env .env.backup
mv .env.production .env

# Edit .env to add your domain and Supabase keys
nano .env
```

#### 4.3 Create Caddy Security Configuration

```bash
# Create Caddy addon directory
mkdir -p caddy-addon

# Create security configuration
cat > caddy-addon/security.conf <<'EOF'
(security) {
    header {
        # HSTS
        Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
        
        # Prevent clickjacking
        X-Frame-Options "SAMEORIGIN"
        
        # Prevent MIME type sniffing
        X-Content-Type-Options "nosniff"
        
        # XSS Protection
        X-XSS-Protection "1; mode=block"
        
        # Referrer Policy
        Referrer-Policy "strict-origin-when-cross-origin"
        
        # Permissions Policy
        Permissions-Policy "geolocation=(), microphone=(), camera=()"
        
        # Remove server header
        -Server
    }
}
EOF
```

### Phase 5: Service Deployment

#### 5.1 Pre-deployment Checks

```bash
# Verify Docker is running
docker info

# Check disk space
df -h

# Verify environment variables
grep -E '^[A-Z_]+=' .env | wc -l  # Should show 20+ variables

# Test DNS resolution
nslookup n8n.yourdomain.com
```

#### 5.2 Deploy Services

```bash
# Deploy with appropriate profile
# For CPU-only deployment:
python3 start_services.py --profile cpu --environment public

# For NVIDIA GPU deployment:
# python3 start_services.py --profile gpu-nvidia --environment public

# Monitor deployment
docker compose -p localai ps

# Watch logs during startup
docker compose -p localai logs -f
```

#### 5.3 Verify Service Health

```bash
# Check all services are running
docker compose -p localai ps --format "table {{.Service}}\t{{.Status}}\t{{.Ports}}"

# Test internal connectivity
docker compose -p localai exec n8n ping -c 1 db
docker compose -p localai exec n8n ping -c 1 ollama

# Check Caddy is obtaining certificates
docker compose -p localai logs caddy | grep -i "certificate"

# Verify no ports are exposed except 80/443
sudo netstat -tlnp | grep -E ':(80|443)\s'
```

### Phase 6: Post-Deployment Verification

#### 6.1 Service Accessibility

```bash
# Test HTTPS endpoints
curl -I https://n8n.yourdomain.com
curl -I https://openwebui.yourdomain.com
curl -I https://flowise.yourdomain.com
curl -I https://supabase.yourdomain.com
curl -I https://langfuse.yourdomain.com
curl -I https://neo4j.yourdomain.com

# Check SSL certificates
echo | openssl s_client -servername n8n.yourdomain.com -connect n8n.yourdomain.com:443 2>/dev/null | openssl x509 -noout -dates
```

#### 6.2 Initial Service Configuration

Access each service and complete initial setup:

1. **n8n** (https://n8n.yourdomain.com)
   - Create admin account
   - Note down credentials securely

2. **Open WebUI** (https://openwebui.yourdomain.com)
   - Create admin account
   - Configure Ollama connection

3. **Flowise** (https://flowise.yourdomain.com)
   - Login with credentials from .env
   - Verify authentication is working

4. **Supabase Studio** (https://supabase.yourdomain.com)
   - Login with DASHBOARD_USERNAME/DASHBOARD_PASSWORD
   - Verify database connectivity

5. **Langfuse** (https://langfuse.yourdomain.com)
   - Create initial organization
   - Generate API keys

6. **Neo4j** (https://neo4j.yourdomain.com)
   - Login with neo4j/password from .env
   - Change default password if prompted

### Phase 7: Final Security Checks

```bash
# Verify firewall rules
sudo ufw status verbose

# Check fail2ban status
sudo fail2ban-client status

# Review authentication logs
sudo tail -n 50 /var/log/auth.log

# Scan for exposed ports
sudo nmap -sT -O localhost

# Check Docker security
docker scout cves local://localai

# Verify SSL configuration
testssl.sh https://n8n.yourdomain.com
```

### Deployment Complete!

Your Local AI Package is now deployed in production. Next steps:
1. Complete the Post-Deployment Configuration
2. Set up monitoring and backups
3. Configure service integrations
4. Test all workflows 

## Post-Deployment Configuration

### Service Integration Setup

#### 1. n8n Configuration

Access n8n at `https://n8n.yourdomain.com`:

```bash
# 1. Create initial admin account
# 2. Import the included workflow
#    - Go to Workflows → Import from File
#    - Upload: Local_RAG_AI_Agent_n8n_Workflow.json

# 3. Configure credentials:
```

**Ollama Credentials:**
- Type: Ollama
- Name: "Ollama Local"
- Base URL: `http://ollama:11434`

**Postgres Credentials:**
- Type: Postgres  
- Name: "Supabase DB"
- Host: `db`
- Database: `postgres`
- User: `postgres`
- Password: (from .env POSTGRES_PASSWORD)
- Port: `5432`

**Qdrant Credentials:**
- Type: Qdrant
- Name: "Qdrant Local"
- URL: `http://qdrant:6333`
- API Key: `localai` (any value works internally)

#### 2. Open WebUI Configuration

Access Open WebUI at `https://openwebui.yourdomain.com`:

```bash
# 1. Create admin account
# 2. Configure n8n integration:
#    - Go to Workspace → Functions
#    - Click "+" to add new function
#    - Name: "n8n Agent"
#    - Paste contents of n8n_pipe.py

# 3. Configure function settings:
#    - Click gear icon
#    - Set n8n_url to webhook URL from n8n
#    - Save and enable function
```

#### 3. Ollama Model Setup

```bash
# Connect to Ollama container
docker compose -p localai exec ollama bash

# Pull required models
ollama pull qwen2.5:7b-instruct-q4_K_M
ollama pull nomic-embed-text
ollama pull llama3.1:8b  # Optional, for better performance

# List available models
ollama list
```

#### 4. Database Initialization

```bash
# Connect to PostgreSQL
docker compose -p localai exec db psql -U postgres

# Create vector extension
CREATE EXTENSION IF NOT EXISTS vector;

# Create dedicated databases (optional)
CREATE DATABASE n8n_data;
CREATE DATABASE langfuse;
CREATE DATABASE flowise;

# Exit psql
\q
```

### Service-Specific Configurations

#### Flowise Setup

1. Access at `https://flowise.yourdomain.com`
2. Login with FLOWISE_USERNAME/FLOWISE_PASSWORD
3. Import custom tools:
   ```bash
   # From the UI: Tools → Import
   - create_google_doc-CustomTool.json
   - get_postgres_tables-CustomTool.json
   - send_slack_message_through_n8n-CustomTool.json
   - summarize_slack_conversation-CustomTool.json
   ```
4. Import example chatflow:
   - Chatflows → Import → `Web Search + n8n Agent Chatflow.json`

#### Langfuse Configuration

1. Access at `https://langfuse.yourdomain.com`
2. Create organization and project
3. Generate API keys:
   ```
   Public Key: pk_xxxxx
   Secret Key: sk_xxxxx
   ```
4. Configure tracing in n8n/Open WebUI

#### Neo4j Setup

1. Access at `https://neo4j.yourdomain.com`
2. Login with credentials from .env
3. Initialize database:
   ```cypher
   // Create indexes for performance
   CREATE INDEX node_name IF NOT EXISTS FOR (n:Node) ON (n.name);
   CREATE INDEX node_type IF NOT EXISTS FOR (n:Node) ON (n.type);
   ```

### Webhook Configuration

#### n8n Webhooks

1. **Production Webhook Setup**:
   ```bash
   # In n8n, for each workflow:
   1. Open the workflow
   2. Click on the Webhook trigger node
   3. Switch from "Test" to "Production" mode
   4. Copy the Production URL
   5. Format: https://n8n.yourdomain.com/webhook/[unique-id]
   
   # Important webhook endpoints:
   - RAG Agent: /webhook/rag-agent
   - Google Docs: /webhook/create-doc
   - Postgres Query: /webhook/postgres-tables
   - Slack Integration: /webhook/slack-message
   ```

2. **Webhook Security**:
   ```javascript
   // Configure authentication in webhook node
   {
     "authentication": "headerAuth",
     "headerAuth": {
       "name": "Authorization",
       "value": "Bearer {{$env.N8N_WEBHOOK_TOKEN}}"
     }
   }
   ```

3. **Import Tool Workflows**:
   ```bash
   # Import n8n tool workflows
   cd ~/local-ai-packaged/n8n-tool-workflows
   
   # For each workflow file:
   - Workflows → Import from File
   - Select: Create_Google_Doc.json
   - Select: Get_Postgres_Tables.json
   - Select: Post_Message_to_Slack.json
   - Select: Summarize_Slack_Conversation.json
   
   # Activate each workflow after import
   ```

#### Integration Points

```javascript
// Open WebUI → n8n Integration
{
  "url": "https://n8n.yourdomain.com/webhook/rag-agent",
  "headers": {
    "Authorization": "Bearer YOUR_N8N_WEBHOOK_TOKEN",
    "Content-Type": "application/json"
  },
  "body": {
    "sessionId": "{{chat_id}}",
    "chatInput": "{{message}}"
  }
}

// Flowise → n8n Custom Tools
{
  "create_google_doc": {
    "webhookUrl": "https://n8n.yourdomain.com/webhook/create-doc",
    "method": "POST",
    "headers": {
      "Authorization": "Bearer {{$vars.headerauth}}"
    }
  },
  "get_postgres_tables": {
    "webhookUrl": "https://n8n.yourdomain.com/webhook/postgres-tables",
    "method": "POST",
    "headers": {
      "Authorization": "Bearer {{$vars.headerauth}}"
    }
  }
}

// n8n → Ollama Configuration
{
  "baseURL": "http://ollama:11434",
  "model": "qwen2.5:7b-instruct-q4_K_M",
  "temperature": 0.7,
  "maxTokens": 4096
}

// n8n → Qdrant Configuration
{
  "url": "http://qdrant:6333",
  "collectionName": "documents",
  "vectorDimension": 768,
  "distance": "Cosine"
}
```

### Advanced Integration Setup

#### 1. n8n Workflow Import and Configuration

**Step 1: Import Base Workflows**
```bash
# Access n8n at https://n8n.yourdomain.com
# Import workflows in this order:

1. V1_Local_RAG_AI_Agent.json (Basic RAG)
   - Simple document Q&A
   - PDF/text processing
   - Basic vector search

2. V2_Local_Supabase_RAG_AI_Agent.json (Advanced RAG)
   - Supabase vector store integration
   - Multi-document support
   - Metadata filtering

3. V3_Local_Agentic_RAG_AI_Agent.json (Full Agent)
   - Multiple tool support
   - SQL query capabilities
   - Google Drive integration
   - Web search functionality
```

**Step 2: Configure Workflow Variables**
```javascript
// In each workflow, set environment variables:
{
  "OLLAMA_MODEL": "qwen2.5:7b-instruct-q4_K_M",
  "EMBEDDING_MODEL": "nomic-embed-text",
  "VECTOR_STORE": "qdrant",  // or "supabase"
  "CHUNK_SIZE": "1000",
  "CHUNK_OVERLAP": "200",
  "MAX_TOKENS": "4096",
  "TEMPERATURE": "0.7"
}
```

**Step 3: Test Workflow Endpoints**
```bash
# Test RAG agent webhook
curl -X POST https://n8n.yourdomain.com/webhook/rag-agent \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "sessionId": "test-session",
    "chatInput": "What is this document about?"
  }'
```

#### 2. Flowise Custom Tools Integration

**Step 1: Configure Flowise Environment**
```bash
# In Flowise, go to Settings → Variables
# Add the following:

headerauth: "Bearer YOUR_N8N_WEBHOOK_TOKEN"
n8n_base_url: "https://n8n.yourdomain.com"
```

**Step 2: Import Custom Tools**
```javascript
// Each custom tool configuration:

// create_google_doc Tool
{
  "name": "create_google_doc",
  "description": "Creates a Google Doc via n8n integration",
  "code": `
    const fetch = require('node-fetch');
    const url = $vars.n8n_base_url + '/webhook/create-doc';
    
    const response = await fetch(url, {
      method: 'POST',
      headers: {
        'Authorization': $vars.headerauth,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        title: $title,
        content: $content
      })
    });
    
    return await response.json();
  `
}

// get_postgres_tables Tool
{
  "name": "get_postgres_tables",
  "description": "Retrieves PostgreSQL table schema",
  "code": `
    const fetch = require('node-fetch');
    const url = $vars.n8n_base_url + '/webhook/postgres-tables';
    
    const response = await fetch(url, {
      method: 'POST',
      headers: {
        'Authorization': $vars.headerauth,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        database: $database || 'postgres',
        schema: $schema || 'public'
      })
    });
    
    return await response.json();
  `
}

// send_slack_message Tool
{
  "name": "send_slack_message_through_n8n",
  "description": "Sends messages to Slack via n8n webhook",
  "code": `
    const fetch = require('node-fetch');
    const url = $vars.n8n_base_url + '/webhook/slack-message';
    
    const response = await fetch(url, {
      method: 'POST',
      headers: {
        'Authorization': $vars.headerauth,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        channel: $channel,
        message: $message,
        username: $username || 'Flowise Bot'
      })
    });
    
    return await response.json();
  `
}

// summarize_slack_conversation Tool
{
  "name": "summarize_slack_conversation",
  "description": "Summarizes recent Slack channel history",
  "code": `
    const fetch = require('node-fetch');
    const url = $vars.n8n_base_url + '/webhook/slack-summary';
    
    const response = await fetch(url, {
      method: 'POST',
      headers: {
        'Authorization': $vars.headerauth,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        channel: $channel,
        limit: $limit || 50,
        summarizationType: $type || 'brief'
      })
    });
    
    return await response.json();
  `
}
```

**Step 3: Create Agent Chatflow**
```javascript
// Import Web Search + n8n Agent Chatflow
// Configure the following nodes:

1. Tool Agent Node:
   - Model: Ollama (qwen2.5:7b-instruct-q4_K_M)
   - Tools: Select all imported n8n tools
   - System Message: "You are an AI assistant with access to various tools..."

2. Buffer Memory Node:
   - Session ID: {{sessionId}}
   - Window Size: 10

3. Vector Store Node:
   - Type: Qdrant
   - Collection: documents
   - Embedding: nomic-embed-text
```

**Step 4: Test Flowise Integration**
```bash
# Test custom tools from Flowise API
curl -X POST https://flowise.yourdomain.com/api/v1/prediction/[chatflow-id] \
  -H "Authorization: Bearer YOUR_FLOWISE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "question": "Create a Google Doc titled Test Document with content Hello World",
    "overrideConfig": {
      "sessionId": "test-session"
    }
  }'

# Expected response should show tool execution
```

**Step 5: Configure Tool Permissions**
```javascript
// In Flowise, set tool execution policies:
{
  "toolExecutionPolicy": {
    "requireConfirmation": false,  // Set to true for production
    "maxToolCalls": 10,
    "timeout": 30000,
    "retryOnFailure": true,
    "retryCount": 3
  },
  "securitySettings": {
    "allowedDomains": ["n8n.yourdomain.com"],
    "validateSSL": true,
    "requireAuth": true
  }
}
```

#### 5. Complete Integration Testing

**Step 1: End-to-End Test Flow**
```bash
# Create test script
cat > ~/test-integration.sh <<'EOF'
#!/bin/bash

echo "Testing Local AI Integration..."

# Test 1: n8n Webhook
echo "1. Testing n8n webhook..."
curl -X POST https://n8n.yourdomain.com/webhook/rag-agent \
  -H "Authorization: Bearer ${N8N_WEBHOOK_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"sessionId": "test", "chatInput": "Hello"}'

# Test 2: Flowise API
echo -e "\n2. Testing Flowise API..."
curl -X POST https://flowise.yourdomain.com/api/v1/prediction/${FLOWISE_CHATFLOW_ID} \
  -H "Content-Type: application/json" \
  -d '{"question": "What can you help me with?"}'

# Test 3: Open WebUI Function
echo -e "\n3. Testing Open WebUI n8n function..."
# This requires manual testing through the UI

# Test 4: Vector Store
echo -e "\n4. Testing Qdrant vector store..."
curl https://qdrant.yourdomain.com/collections

echo -e "\nIntegration tests complete!"
EOF

chmod +x ~/test-integration.sh
```

**Step 2: Monitor Integration Logs**
```bash
# Create monitoring script
cat > ~/monitor-integration.sh <<'EOF'
#!/bin/bash

# Monitor all integration points
docker compose -p localai logs -f n8n flowise open-webui | grep -E "(webhook|integration|error|connected)"
EOF

chmod +x ~/monitor-integration.sh
```

#### 6. Production Security Hardening

**Webhook Security**
```nginx
# Add to Caddy configuration for webhook rate limiting
n8n.yourdomain.com {
    rate_limit {
        zone webhook_zone {
            key {remote_host}
            events 100
            window 1m
        }
    }
    
    handle /webhook/* {
        rate_limit webhook_zone
        reverse_proxy n8n:5678
    }
}
```

**API Key Management**
```bash
# Generate secure API keys
N8N_WEBHOOK_TOKEN=$(openssl rand -hex 32)
FLOWISE_API_KEY=$(openssl rand -hex 32)
LANGFUSE_PUBLIC_KEY=$(openssl rand -hex 16)
LANGFUSE_SECRET_KEY=$(openssl rand -hex 32)

# Store in environment
cat >> .env <<EOF
N8N_WEBHOOK_TOKEN=${N8N_WEBHOOK_TOKEN}
FLOWISE_API_KEY=${FLOWISE_API_KEY}
LANGFUSE_PUBLIC_KEY=pk_${LANGFUSE_PUBLIC_KEY}
LANGFUSE_SECRET_KEY=sk_${LANGFUSE_SECRET_KEY}
EOF
```

#### 3. Open WebUI n8n Function Setup

**Step 1: Configure n8n_pipe.py Function**
```python
# In Open WebUI Workspace → Functions
# Paste the following configuration:

"""
title: n8n AI Agent
author: localai
version: 1.0
requirements: requests
environment_variables: n8n_url, n8n_bearer_token
"""

import os
import requests
import json
from typing import Iterator, Any

class Pipe:
    def __init__(self):
        self.name = "n8n AI Agent"
        self.n8n_url = os.getenv("N8N_URL", "https://n8n.yourdomain.com/webhook/rag-agent")
        self.bearer_token = os.getenv("N8N_BEARER_TOKEN", "")
        
    def pipe(self, prompt: str, **kwargs) -> Iterator[str]:
        headers = {
            "Authorization": f"Bearer {self.bearer_token}",
            "Content-Type": "application/json"
        }
        
        data = {
            "sessionId": kwargs.get("chat_id", "default"),
            "chatInput": prompt,
            "context": kwargs.get("context", {})
        }
        
        try:
            response = requests.post(
                self.n8n_url,
                headers=headers,
                json=data,
                timeout=30
            )
            
            if response.status_code == 200:
                result = response.json()
                yield result.get("output", "No response from n8n")
            else:
                yield f"Error: {response.status_code} - {response.text}"
                
        except Exception as e:
            yield f"Error connecting to n8n: {str(e)}"
```

**Step 2: Enable Function**
```bash
# In Open WebUI:
1. Go to function settings (gear icon)
2. Set environment variables:
   - n8n_url: https://n8n.yourdomain.com/webhook/rag-agent
   - n8n_bearer_token: YOUR_WEBHOOK_TOKEN
3. Toggle function ON
4. Function appears in model dropdown as "n8n AI Agent"
```

#### 4. Service Health Checks

```bash
# Create health check endpoints
cat > ~/health-check.sh <<'EOF'
#!/bin/bash

# Colors for output
GREEN='\033[0;32m'
RED='\033[0;31m'
NC='\033[0m'

echo "Checking Local AI Services Health..."

# Check each service
services=(
  "n8n:https://n8n.yourdomain.com/healthz"
  "openwebui:https://openwebui.yourdomain.com/health"
  "flowise:https://flowise.yourdomain.com/api/v1/ping"
  "supabase:https://supabase.yourdomain.com/rest/v1/"
  "langfuse:https://langfuse.yourdomain.com/api/public/health"
)

for service in "${services[@]}"; do
  IFS=':' read -r name url <<< "$service"
  if curl -s -f "$url" > /dev/null; then
    echo -e "${GREEN}✓${NC} $name is healthy"
  else
    echo -e "${RED}✗${NC} $name is not responding"
  fi
done

# Check Ollama models
echo -e "\nOllama Models:"
docker compose -p localai exec ollama ollama list
EOF

chmod +x ~/health-check.sh
```

## Maintenance and Operations

### Daily Tasks

#### 1. Health Monitoring

```bash
# Check service status
docker compose -p localai ps

# View service logs
docker compose -p localai logs --tail=100

# Check resource usage
docker stats --no-stream

# Monitor disk usage
df -h
du -sh /var/lib/docker/
```

#### 2. Log Rotation

```bash
# Configure Docker log rotation (already in daemon.json)
# Manually rotate if needed
docker compose -p localai logs --tail=0 -f > /dev/null

# System logs
sudo journalctl --vacuum-time=7d
```

### Weekly Tasks

#### 1. Backup Procedures

```bash
# Create backup script
cat > ~/backup-localai.sh <<'EOF'
#!/bin/bash
BACKUP_DIR="/home/localai/backups/$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"

# Backup PostgreSQL
docker compose -p localai exec -T db pg_dumpall -U postgres > "$BACKUP_DIR/postgres_backup.sql"

# Backup volumes
docker run --rm -v localai_n8n_storage:/data -v "$BACKUP_DIR":/backup alpine tar czf /backup/n8n_storage.tar.gz -C /data .
docker run --rm -v localai_qdrant_storage:/data -v "$BACKUP_DIR":/backup alpine tar czf /backup/qdrant_storage.tar.gz -C /data .
docker run --rm -v localai_open-webui:/data -v "$BACKUP_DIR":/backup alpine tar czf /backup/openwebui.tar.gz -C /data .

# Backup configurations
cp -r ~/local-ai-packaged/.env "$BACKUP_DIR/"
cp -r ~/local-ai-packaged/caddy-addon "$BACKUP_DIR/"

# Compress and encrypt
tar czf "$BACKUP_DIR.tar.gz" -C "$(dirname "$BACKUP_DIR")" "$(basename "$BACKUP_DIR")"
openssl enc -aes-256-cbc -salt -in "$BACKUP_DIR.tar.gz" -out "$BACKUP_DIR.tar.gz.enc" -k YOUR_ENCRYPTION_PASSWORD

# Clean up
rm -rf "$BACKUP_DIR" "$BACKUP_DIR.tar.gz"

# Keep only last 7 backups
find /home/localai/backups -name "*.tar.gz.enc" -mtime +7 -delete
EOF

chmod +x ~/backup-localai.sh

# Add to crontab
crontab -e
# Add: 0 2 * * * /home/localai/backup-localai.sh
```

#### 2. Update and Upgrade Procedures

**Pre-Update Checklist**
```bash
# 1. Create full backup before any updates
~/backup-localai.sh

# 2. Check current versions
docker compose -p localai ps --format "table {{.Name}}\t{{.Image}}\t{{.Status}}"

# 3. Review release notes
# Check GitHub repos for breaking changes:
# - https://github.com/n8n-io/n8n/releases
# - https://github.com/supabase/supabase/releases
# - https://github.com/ollama/ollama/releases

# 4. Test in staging environment if available
```

**Safe Update Procedure**
```bash
# Create update script
cat > ~/update-localai.sh <<'EOF'
#!/bin/bash
set -e

# Colors for output
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
NC='\033[0m'

echo -e "${YELLOW}Starting LocalAI update procedure...${NC}"

# Step 1: Health check before update
echo -e "\n${YELLOW}1. Checking service health...${NC}"
~/health-check.sh

# Step 2: Create backup
echo -e "\n${YELLOW}2. Creating backup...${NC}"
~/backup-localai.sh

# Step 3: Export current image versions
echo -e "\n${YELLOW}3. Saving current versions...${NC}"
docker compose -p localai ps --format json > ~/localai-versions-$(date +%Y%m%d).json

# Step 4: Pull new images
echo -e "\n${YELLOW}4. Pulling latest images...${NC}"
docker compose -p localai pull

# Step 5: Stop services in correct order
echo -e "\n${YELLOW}5. Stopping services...${NC}"
# Stop dependent services first
docker compose -p localai stop open-webui flowise langfuse n8n
sleep 5
# Stop data services
docker compose -p localai stop ollama qdrant neo4j
sleep 5
# Stop infrastructure
docker compose -p localai stop kong auth meta storage realtime
sleep 5
# Stop databases last
docker compose -p localai stop db redis

# Step 6: Update database schemas if needed
echo -e "\n${YELLOW}6. Checking for database migrations...${NC}"
# n8n migrations
docker compose -p localai run --rm n8n n8n db:migrate

# Step 7: Start services in correct order
echo -e "\n${YELLOW}7. Starting services...${NC}"
cd ~/local-ai-packaged
python3 start_services.py --profile ${PROFILE:-gpu-nvidia} --environment ${ENVIRONMENT:-public}

# Step 8: Wait for services to be healthy
echo -e "\n${YELLOW}8. Waiting for services to stabilize...${NC}"
sleep 30

# Step 9: Post-update health check
echo -e "\n${YELLOW}9. Running post-update health check...${NC}"
~/health-check.sh

# Step 10: Clean up old images
echo -e "\n${YELLOW}10. Cleaning up old images...${NC}"
docker image prune -a -f

echo -e "\n${GREEN}Update completed successfully!${NC}"
EOF

chmod +x ~/update-localai.sh
```

**Rolling Update Strategy**
```bash
# For zero-downtime updates (requires load balancer)
cat > ~/rolling-update.sh <<'EOF'
#!/bin/bash

# Function to update single service
update_service() {
    local service=$1
    echo "Updating $service..."
    
    # Scale up new version
    docker compose -p localai up -d --scale ${service}=2 --no-recreate ${service}
    
    # Wait for health check
    sleep 30
    
    # Remove old container
    old_container=$(docker ps --filter "label=com.docker.compose.service=${service}" --format "{{.ID}}" | tail -1)
    docker stop $old_container
    docker rm $old_container
}

# Update services one by one
for service in n8n open-webui flowise; do
    update_service $service
done
EOF

chmod +x ~/rolling-update.sh
```

**Service-Specific Update Procedures**

**Ollama Model Updates**
```bash
# Update Ollama models
cat > ~/update-ollama-models.sh <<'EOF'
#!/bin/bash

echo "Updating Ollama models..."

# List current models
echo "Current models:"
docker compose -p localai exec ollama ollama list

# Update specific model
update_model() {
    local model=$1
    echo "Updating $model..."
    docker compose -p localai exec ollama ollama pull $model
}

# Update all models
for model in qwen2.5:7b-instruct-q4_K_M nomic-embed-text llama3.1:8b; do
    update_model $model
done

# Remove old versions
docker compose -p localai exec ollama ollama prune

echo "Model update complete!"
EOF

chmod +x ~/update-ollama-models.sh
```

**Database Update Procedures**
```bash
# PostgreSQL major version upgrade
cat > ~/upgrade-postgres.sh <<'EOF'
#!/bin/bash

OLD_VERSION=15
NEW_VERSION=16

echo "Upgrading PostgreSQL from $OLD_VERSION to $NEW_VERSION"

# 1. Dump all databases
docker compose -p localai exec -T db pg_dumpall -U postgres > postgres_upgrade_backup.sql

# 2. Stop PostgreSQL
docker compose -p localai stop db

# 3. Backup data directory
sudo cp -r /var/lib/docker/volumes/localai_db_data /var/lib/docker/volumes/localai_db_data_backup

# 4. Update docker-compose.yml to new version
sed -i "s/postgres:$OLD_VERSION/postgres:$NEW_VERSION/" docker-compose.yml

# 5. Remove old data directory
docker compose -p localai down db
docker volume rm localai_db_data

# 6. Start new PostgreSQL
docker compose -p localai up -d db

# 7. Wait for startup
sleep 20

# 8. Restore data
docker compose -p localai exec -T db psql -U postgres < postgres_upgrade_backup.sql

echo "PostgreSQL upgrade complete!"
EOF

chmod +x ~/upgrade-postgres.sh
```

**Rollback Procedures**
```bash
# Create rollback script
cat > ~/rollback-localai.sh <<'EOF'
#!/bin/bash

BACKUP_DATE=$1
if [ -z "$BACKUP_DATE" ]; then
    echo "Usage: $0 YYYYMMDD_HHMMSS"
    exit 1
fi

echo "Rolling back to backup from $BACKUP_DATE..."

# 1. Stop all services
docker compose -p localai down

# 2. Restore from backup
cd /home/localai/backups
openssl enc -aes-256-cbc -d -in ${BACKUP_DATE}.tar.gz.enc -out ${BACKUP_DATE}.tar.gz -k YOUR_ENCRYPTION_PASSWORD
tar xzf ${BACKUP_DATE}.tar.gz

# 3. Restore database
docker compose -p localai up -d db
sleep 20
docker compose -p localai exec -T db psql -U postgres < ${BACKUP_DATE}/postgres_backup.sql

# 4. Restore volumes
docker run --rm -v localai_n8n_storage:/data -v $(pwd)/${BACKUP_DATE}:/backup alpine tar xzf /backup/n8n_storage.tar.gz -C /data
docker run --rm -v localai_qdrant_storage:/data -v $(pwd)/${BACKUP_DATE}:/backup qdrant_storage.tar.gz -C /data

# 5. Restore configuration
cp ${BACKUP_DATE}/.env ~/local-ai-packaged/

# 6. Start services
cd ~/local-ai-packaged
python3 start_services.py --profile gpu-nvidia --environment public

echo "Rollback complete!"
EOF

chmod +x ~/rollback-localai.sh
```

**Automated Update Schedule**
```bash
# Create automated update cron job
cat > ~/auto-update-check.sh <<'EOF'
#!/bin/bash

# Check for updates but don't apply automatically
docker compose -p localai pull --dry-run 2>&1 | grep -E "Pulling|Already" > /tmp/update-check.log

# If updates are available, send notification
if grep -q "Pulling" /tmp/update-check.log; then
    echo "Updates available for LocalAI:" | mail -s "LocalAI Updates Available" admin@yourdomain.com
    cat /tmp/update-check.log | mail -s "LocalAI Update Details" admin@yourdomain.com
fi
EOF

chmod +x ~/auto-update-check.sh

# Add to crontab (weekly check)
(crontab -l 2>/dev/null; echo "0 2 * * 1 /home/localai/auto-update-check.sh") | crontab -
```

### Monitoring and Alerting Setup

#### 1. Deploy Monitoring Stack

**Option A: Lightweight Monitoring with cAdvisor + Prometheus**
```bash
# Create monitoring docker-compose file
cat > ~/local-ai-packaged/docker-compose.monitoring.yml <<'EOF'
version: '3.8'

services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: localai-cadvisor
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    ports:
      - "8080:8080"
    devices:
      - /dev/kmsg
    privileged: true
    restart: unless-stopped
    networks:
      - localai

  prometheus:
    image: prom/prometheus:latest
    container_name: localai-prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
    ports:
      - "9090:9090"
    restart: unless-stopped
    networks:
      - localai

  grafana:
    image: grafana/grafana:latest
    container_name: localai-grafana
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources
    environment:
      - GF_SECURITY_ADMIN_USER=${GRAFANA_USER:-admin}
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD:-admin}
      - GF_USERS_ALLOW_SIGN_UP=false
    ports:
      - "3005:3000"
    restart: unless-stopped
    networks:
      - localai

  alertmanager:
    image: prom/alertmanager:latest
    container_name: localai-alertmanager
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
      - alertmanager_data:/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
    ports:
      - "9093:9093"
    restart: unless-stopped
    networks:
      - localai

volumes:
  prometheus_data:
  grafana_data:
  alertmanager_data:

networks:
  localai:
    external: true
    name: localai_default
EOF

# Create Prometheus configuration
cat > ~/local-ai-packaged/prometheus.yml <<'EOF'
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093

rule_files:
  - "alerts.yml"

scrape_configs:
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'docker'
    static_configs:
      - targets: ['172.17.0.1:9323']

  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres-exporter:9187']

  - job_name: 'n8n'
    static_configs:
      - targets: ['n8n:5678']
    metrics_path: /metrics
EOF

# Create alert rules
cat > ~/local-ai-packaged/alerts.yml <<'EOF'
groups:
  - name: container_alerts
    interval: 30s
    rules:
      - alert: ContainerDown
        expr: up{job="cadvisor"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Container is down"
          description: "Container {{ $labels.instance }} is down for more than 1 minute."

      - alert: HighMemoryUsage
        expr: (container_memory_usage_bytes / container_spec_memory_limit_bytes) > 0.9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage detected"
          description: "Container {{ $labels.container_name }} memory usage is above 90%"

      - alert: HighCPUUsage
        expr: rate(container_cpu_usage_seconds_total[5m]) > 0.9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage detected"
          description: "Container {{ $labels.container_name }} CPU usage is above 90%"

      - alert: DiskSpaceLow
        expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes) < 0.1
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Low disk space"
          description: "Disk space is below 10% on {{ $labels.device }}"

      - alert: PostgreSQLDown
        expr: pg_up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "PostgreSQL is down"
          description: "PostgreSQL database is not responding"
EOF

# Create Alertmanager configuration
cat > ~/local-ai-packaged/alertmanager.yml <<'EOF'
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname', 'severity']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'email-notifications'

receivers:
  - name: 'email-notifications'
    email_configs:
      - to: 'alerts@yourdomain.com'
        from: 'alertmanager@yourdomain.com'
        smarthost: 'smtp.gmail.com:587'
        auth_username: 'alertmanager@yourdomain.com'
        auth_password: 'YOUR_APP_PASSWORD'
        headers:
          Subject: 'LocalAI Alert: {{ .GroupLabels.alertname }}'

  - name: 'slack-notifications'
    slack_configs:
      - api_url: 'YOUR_SLACK_WEBHOOK_URL'
        channel: '#alerts'
        title: 'LocalAI Alert'
        text: '{{ range .Alerts }}{{ .Annotations.summary }}\n{{ .Annotations.description }}\n{{ end }}'
EOF

# Start monitoring stack
docker compose -f docker-compose.monitoring.yml up -d
```

**Option B: Production-Grade Monitoring with Datadog/New Relic**
```bash
# Datadog Agent Setup
docker run -d --name datadog-agent \
  -e DD_API_KEY=YOUR_DATADOG_API_KEY \
  -e DD_SITE="datadoghq.com" \
  -e DD_LOGS_ENABLED=true \
  -e DD_LOGS_CONFIG_CONTAINER_COLLECT_ALL=true \
  -e DD_CONTAINER_EXCLUDE="name:datadog-agent" \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -v /proc/:/host/proc/:ro \
  -v /sys/fs/cgroup/:/host/sys/fs/cgroup:ro \
  -v /var/lib/docker/containers:/var/lib/docker/containers:ro \
  datadog/agent:latest
```

#### 2. Service-Specific Monitoring

**n8n Monitoring**
```javascript
// Add to n8n environment variables
N8N_METRICS=true
N8N_METRICS_PREFIX=n8n_

// Access metrics at http://n8n:5678/metrics
```

**PostgreSQL Monitoring**
```bash
# Deploy postgres_exporter
docker run -d \
  --name postgres-exporter \
  --network localai_default \
  -e DATA_SOURCE_NAME="postgresql://postgres:${POSTGRES_PASSWORD}@db:5432/postgres?sslmode=disable" \
  quay.io/prometheuscommunity/postgres-exporter
```

**Ollama Monitoring**
```bash
# Create Ollama metrics script
cat > ~/monitor-ollama.sh <<'EOF'
#!/bin/bash
while true; do
  # Get model list and sizes
  models=$(docker compose -p localai exec ollama ollama list | tail -n +2)
  
  # Get current GPU usage if available
  if command -v nvidia-smi &> /dev/null; then
    gpu_usage=$(nvidia-smi --query-gpu=utilization.gpu,memory.used,memory.total --format=csv,noheader,nounits)
  fi
  
  # Write to Prometheus textfile
  echo "# HELP ollama_models_count Number of loaded models" > /tmp/ollama_metrics.prom
  echo "# TYPE ollama_models_count gauge" >> /tmp/ollama_metrics.prom
  echo "ollama_models_count $(echo "$models" | wc -l)" >> /tmp/ollama_metrics.prom
  
  sleep 30
done
EOF
chmod +x ~/monitor-ollama.sh
```

#### 3. Custom Dashboards

**Grafana Dashboard Configuration**
```json
{
  "dashboard": {
    "title": "LocalAI Production Monitoring",
    "panels": [
      {
        "title": "Container CPU Usage",
        "targets": [
          {
            "expr": "rate(container_cpu_usage_seconds_total{container_name=~\"localai.*\"}[5m])"
          }
        ]
      },
      {
        "title": "Container Memory Usage",
        "targets": [
          {
            "expr": "container_memory_usage_bytes{container_name=~\"localai.*\"}"
          }
        ]
      },
      {
        "title": "HTTP Request Rate",
        "targets": [
          {
            "expr": "rate(caddy_http_requests_total[5m])"
          }
        ]
      },
      {
        "title": "Database Connections",
        "targets": [
          {
            "expr": "pg_stat_database_numbackends{datname=\"postgres\"}"
          }
        ]
      }
    ]
  }
}
```

#### 4. Health Check Automation

```bash
# Create comprehensive health check script
cat > ~/health-monitor.sh <<'EOF'
#!/bin/bash

# Configuration
SLACK_WEBHOOK="YOUR_SLACK_WEBHOOK_URL"
EMAIL="alerts@yourdomain.com"
CHECK_INTERVAL=300  # 5 minutes

# Function to send alerts
send_alert() {
    local service=$1
    local status=$2
    local details=$3
    
    # Send to Slack
    curl -X POST $SLACK_WEBHOOK \
      -H 'Content-type: application/json' \
      --data "{\"text\":\"🚨 LocalAI Alert: $service is $status\\n$details\"}"
    
    # Send email
    echo -e "Subject: LocalAI Alert: $service is $status\\n\\n$details" | \
      sendmail $EMAIL
}

# Function to check service health
check_service() {
    local name=$1
    local url=$2
    local expected_code=${3:-200}
    
    response=$(curl -s -o /dev/null -w "%{http_code}" "$url")
    
    if [ "$response" != "$expected_code" ]; then
        send_alert "$name" "unhealthy" "Expected $expected_code, got $response"
        return 1
    fi
    return 0
}

# Main monitoring loop
while true; do
    echo "$(date): Starting health checks..."
    
    # Check each service
    check_service "n8n" "https://n8n.yourdomain.com/healthz"
    check_service "OpenWebUI" "https://openwebui.yourdomain.com/health"
    check_service "Flowise" "https://flowise.yourdomain.com/api/v1/ping"
    check_service "Supabase" "https://supabase.yourdomain.com/rest/v1/" 200
    
    # Check disk space
    disk_usage=$(df -h / | awk 'NR==2 {print $5}' | sed 's/%//')
    if [ "$disk_usage" -gt 90 ]; then
        send_alert "Disk Space" "critical" "Disk usage is at ${disk_usage}%"
    fi
    
    # Check memory
    mem_usage=$(free | grep Mem | awk '{print ($2-$7)/$2 * 100.0}')
    if (( $(echo "$mem_usage > 90" | bc -l) )); then
        send_alert "Memory" "critical" "Memory usage is at ${mem_usage}%"
    fi
    
    sleep $CHECK_INTERVAL
done
EOF

chmod +x ~/health-monitor.sh

# Run as systemd service
sudo cat > /etc/systemd/system/localai-monitor.service <<EOF
[Unit]
Description=LocalAI Health Monitor
After=docker.service

[Service]
Type=simple
User=localai
ExecStart=/home/localai/health-monitor.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable localai-monitor
sudo systemctl start localai-monitor
```

### Monthly Tasks

#### 1. Security Audit

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Check for security updates
sudo unattended-upgrades --dry-run

# Review user access
sudo last -20

# Check fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd

# Scan for vulnerabilities
docker scout cves --only-fixed

# Review firewall rules
sudo ufw status numbered
```

#### 2. Performance Optimization Review

```bash
# Analyze slow queries
docker compose -p localai exec db psql -U postgres -c "SELECT * FROM pg_stat_statements ORDER BY total_time DESC LIMIT 10;"

# Check container resource limits
docker compose -p localai exec n8n cat /proc/meminfo

# Clean up Docker
docker system prune -a -f --volumes
```

### Performance Tuning Guide

#### 1. Service-Specific Performance Tuning

**PostgreSQL Optimization**
```bash
# Create PostgreSQL tuning script
cat > ~/tune-postgres.sh <<'EOF'
#!/bin/bash

# Connect to PostgreSQL
docker compose -p localai exec db psql -U postgres <<SQL

-- Memory Settings (for 8GB allocated to PostgreSQL)
ALTER SYSTEM SET shared_buffers = '2GB';
ALTER SYSTEM SET effective_cache_size = '6GB';
ALTER SYSTEM SET maintenance_work_mem = '512MB';
ALTER SYSTEM SET work_mem = '64MB';
ALTER SYSTEM SET wal_buffers = '16MB';

-- Checkpoint Settings
ALTER SYSTEM SET checkpoint_completion_target = '0.9';
ALTER SYSTEM SET checkpoint_timeout = '15min';
ALTER SYSTEM SET max_wal_size = '4GB';
ALTER SYSTEM SET min_wal_size = '1GB';

-- Query Planner
ALTER SYSTEM SET random_page_cost = '1.1';  -- For SSD storage
ALTER SYSTEM SET effective_io_concurrency = '200';  -- For SSD
ALTER SYSTEM SET default_statistics_target = '100';

-- Connection Pooling
ALTER SYSTEM SET max_connections = '200';

-- Parallel Query Execution
ALTER SYSTEM SET max_parallel_workers_per_gather = '4';
ALTER SYSTEM SET max_parallel_workers = '8';
ALTER SYSTEM SET max_parallel_maintenance_workers = '4';

-- Apply changes
SELECT pg_reload_conf();

-- Create indexes for common queries
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_n8n_execution_finished ON execution_entity(finished);
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_n8n_workflow_active ON workflow_entity(active);
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_chat_messages_session ON chat_messages(session_id, created_at);

SQL

echo "PostgreSQL tuning complete. Restart may be required for some settings."
EOF

chmod +x ~/tune-postgres.sh
```

**n8n Performance Optimization**
```bash
# Add to n8n environment variables in docker-compose.yml
environment:
  - N8N_PUSH_BACKEND=websocket  # More efficient than SSE
  - EXECUTIONS_PROCESS=main      # Or 'own' for heavy workflows
  - N8N_CONCURRENCY_PRODUCTION_LIMIT=10
  - N8N_PAYLOAD_SIZE_MAX=16     # MB
  - N8N_METRICS=true
  - NODE_OPTIONS=--max-old-space-size=4096  # 4GB heap

# For high-volume workflows, use queue mode
  - QUEUE_BULL_REDIS_HOST=redis
  - QUEUE_BULL_REDIS_PORT=6379
  - EXECUTIONS_MODE=queue
```

**Ollama Optimization**
```bash
# Create Ollama optimization script
cat > ~/optimize-ollama.sh <<'EOF'
#!/bin/bash

# Set Ollama environment for optimal performance
cat >> ~/local-ai-packaged/.env <<ENV

# Ollama Performance Settings
OLLAMA_NUM_PARALLEL=2          # Parallel model execution
OLLAMA_MAX_LOADED_MODELS=2     # Models kept in memory
OLLAMA_GPU_OVERHEAD=500        # GPU memory overhead in MB
OLLAMA_CONTEXT_LENGTH=4096     # Default context window
OLLAMA_BATCH_SIZE=512          # Batch processing size
OLLAMA_NUM_THREAD=8            # CPU threads for inference
OLLAMA_FLASH_ATTENTION=true    # Enable flash attention (if supported)

# GPU-specific optimizations
CUDA_VISIBLE_DEVICES=0         # Specify GPU device
OLLAMA_CUDA_COMPUTE_CAP=75     # For RTX 3080 (adjust for your GPU)
ENV

# Optimize model loading
docker compose -p localai exec ollama bash -c '
# Pre-load frequently used models
ollama run qwen2.5:7b-instruct-q4_K_M --keepalive 24h
ollama run nomic-embed-text --keepalive 24h
'
EOF

chmod +x ~/optimize-ollama.sh
```

**Qdrant Vector Database Optimization**
```bash
# Qdrant performance configuration
cat > ~/local-ai-packaged/qdrant-config.yaml <<'EOF'
storage:
  # Performance optimizations
  wal:
    wal_capacity_mb: 32
    wal_segments_ahead: 2
  
  optimizers:
    # Indexing optimization
    indexing_threshold: 50000
    # Vacuum optimization  
    vacuum_min_vector_number: 10000
    # Segment optimization
    max_segment_size: 200000
  
  performance:
    # HNSW index parameters
    hnsw_index:
      m: 16                # Higher = better recall, more memory
      ef_construct: 200    # Higher = better quality, slower indexing
      ef: 100             # Higher = better search quality, slower
      
service:
  max_request_size_mb: 32
  max_workers: 0  # Auto-detect CPU cores
  
# Enable metrics
metrics:
  enabled: true
  port: 6334
EOF

# Apply configuration
docker compose -p localai restart qdrant
```

**Redis Performance Tuning**
```bash
# Redis optimization
docker compose -p localai exec redis redis-cli <<'REDIS'
# Memory optimization
CONFIG SET maxmemory 2gb
CONFIG SET maxmemory-policy allkeys-lru

# Persistence settings (for performance)
CONFIG SET save ""  # Disable RDB snapshots
CONFIG SET appendonly no  # Disable AOF for max performance

# Network optimization
CONFIG SET tcp-backlog 511
CONFIG SET tcp-keepalive 300

# Save configuration
CONFIG REWRITE
REDIS
```

#### 2. Container Resource Optimization

```yaml
# Add to docker-compose.yml for each service
services:
  n8n:
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 4G
        reservations:
          cpus: '2'
          memory: 2G
    
  ollama:
    deploy:
      resources:
        limits:
          cpus: '8'
          memory: 16G
        reservations:
          cpus: '4'
          memory: 8G
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
              
  db:
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 8G
        reservations:
          cpus: '2'
          memory: 4G
```

#### 3. Network Performance Optimization

```bash
# Optimize Docker network performance
cat > ~/optimize-network.sh <<'EOF'
#!/bin/bash

# Increase network buffer sizes
sudo sysctl -w net.core.rmem_max=134217728
sudo sysctl -w net.core.wmem_max=134217728
sudo sysctl -w net.ipv4.tcp_rmem="4096 87380 134217728"
sudo sysctl -w net.ipv4.tcp_wmem="4096 65536 134217728"

# Enable TCP BBR congestion control
sudo sysctl -w net.core.default_qdisc=fq
sudo sysctl -w net.ipv4.tcp_congestion_control=bbr

# Optimize connection handling
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=4096
sudo sysctl -w net.core.somaxconn=4096
sudo sysctl -w net.ipv4.tcp_fin_timeout=15

# Make permanent
sudo cat >> /etc/sysctl.conf <<SYSCTL
# LocalAI Network Optimizations
net.core.rmem_max=134217728
net.core.wmem_max=134217728
net.ipv4.tcp_rmem=4096 87380 134217728
net.ipv4.tcp_wmem=4096 65536 134217728
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
net.ipv4.tcp_max_syn_backlog=4096
net.core.somaxconn=4096
net.ipv4.tcp_fin_timeout=15
SYSCTL
EOF

chmod +x ~/optimize-network.sh
```

#### 4. Storage Performance Optimization

```bash
# Optimize Docker storage
cat > ~/optimize-storage.sh <<'EOF'
#!/bin/bash

# Use dedicated fast storage for performance-critical volumes
FAST_STORAGE="/mnt/nvme"  # Adjust to your fast storage path

# Create optimized volumes
docker volume create --driver local \
  --opt type=none \
  --opt o=bind \
  --opt device=$FAST_STORAGE/qdrant \
  localai_qdrant_storage_fast

docker volume create --driver local \
  --opt type=none \
  --opt o=bind \
  --opt device=$FAST_STORAGE/postgres \
  localai_db_data_fast

# Set optimal filesystem parameters
sudo tune2fs -o journal_data_writeback $FAST_STORAGE
sudo mount -o remount,noatime,nodiratime $FAST_STORAGE

# Configure I/O scheduler for NVMe
echo none | sudo tee /sys/block/nvme0n1/queue/scheduler

# For regular SSDs
echo deadline | sudo tee /sys/block/sda/queue/scheduler
EOF

chmod +x ~/optimize-storage.sh
```

#### 5. Monitoring Performance Metrics

```bash
# Create performance monitoring dashboard
cat > ~/performance-monitor.sh <<'EOF'
#!/bin/bash

while true; do
    clear
    echo "=== LocalAI Performance Monitor ==="
    echo "Time: $(date)"
    echo ""
    
    # CPU and Memory
    echo "=== System Resources ==="
    docker stats --no-stream --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.MemPerc}}"
    echo ""
    
    # Database Performance
    echo "=== Database Metrics ==="
    docker compose -p localai exec -T db psql -U postgres -c "
    SELECT datname, numbackends as connections, 
           pg_size_pretty(pg_database_size(datname)) as size
    FROM pg_stat_database 
    WHERE datname NOT IN ('template0', 'template1');"
    echo ""
    
    # Ollama Models
    echo "=== Loaded Models ==="
    docker compose -p localai exec ollama ollama list
    echo ""
    
    # Redis Memory
    echo "=== Redis Memory ==="
    docker compose -p localai exec redis redis-cli INFO memory | grep used_memory_human
    
    sleep 5
done
EOF

chmod +x ~/performance-monitor.sh
```

#### 6. Performance Benchmarking

```bash
# Create benchmarking script
cat > ~/benchmark-localai.sh <<'EOF'
#!/bin/bash

echo "Starting LocalAI performance benchmark..."

# Test 1: Database performance
echo -e "\n1. Testing PostgreSQL performance..."
docker compose -p localai exec -T db pgbench -i -s 10 postgres
docker compose -p localai exec -T db pgbench -c 10 -j 2 -t 1000 postgres

# Test 2: Vector search performance
echo -e "\n2. Testing Qdrant performance..."
curl -X POST "http://localhost:6333/collections/benchmark/points/search" \
  -H "Content-Type: application/json" \
  -d '{
    "vector": [0.1, 0.2, 0.3, ...],
    "limit": 10
  }'

# Test 3: LLM inference performance
echo -e "\n3. Testing Ollama inference..."
time docker compose -p localai exec ollama ollama run qwen2.5:7b-instruct-q4_K_M "Hello world"

# Test 4: API response times
echo -e "\n4. Testing API endpoints..."
ab -n 100 -c 10 https://n8n.yourdomain.com/healthz

echo -e "\nBenchmark complete!"
EOF

chmod +x ~/benchmark-localai.sh
```

#### 7. Auto-Scaling Configuration

```yaml
# docker-compose.scale.yml for horizontal scaling
version: '3.8'

services:
  n8n-worker:
    image: n8nio/n8n
    command: worker
    environment:
      - EXECUTIONS_MODE=queue
      - QUEUE_BULL_REDIS_HOST=redis
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '2'
          memory: 2G
          
  open-webui:
    deploy:
      replicas: 2
      labels:
        - "traefik.enable=true"
        - "traefik.http.services.webui.loadbalancer.server.port=3000"
```

### Disaster Recovery

#### Restore from Backup

```bash
# Decrypt backup
openssl enc -aes-256-cbc -d -in backup_20240120_020000.tar.gz.enc -out backup.tar.gz -k YOUR_ENCRYPTION_PASSWORD

# Extract backup
tar xzf backup.tar.gz

# Stop services
docker compose -p localai down

# Restore PostgreSQL
docker compose -p localai up -d db
docker compose -p localai exec -T db psql -U postgres < postgres_backup.sql

# Restore volumes
docker run --rm -v localai_n8n_storage:/data -v $(pwd):/backup alpine tar xzf /backup/n8n_storage.tar.gz -C /data
docker run --rm -v localai_qdrant_storage:/data -v $(pwd):/backup alpine tar xzf /backup/qdrant_storage.tar.gz -C /data

# Restore configuration
cp backup/.env ~/local-ai-packaged/

# Start services
python3 start_services.py --profile gpu-nvidia --environment public
```

## Troubleshooting and FAQ

### Service-Specific Troubleshooting

#### n8n Troubleshooting

**Issue: Workflows not executing**
```bash
# Check n8n logs
docker compose -p localai logs n8n --tail=100

# Common issues and solutions:
# 1. Webhook URL incorrect
docker compose -p localai exec n8n cat /root/.n8n/config

# 2. Database connection issues
docker compose -p localai exec n8n n8n db:status

# 3. Redis connection (if using queue mode)
docker compose -p localai exec redis redis-cli ping

# 4. Execution timeout
# Add to environment:
EXECUTIONS_TIMEOUT=300
EXECUTIONS_TIMEOUT_MAX=3600

# 5. Memory issues with large workflows
NODE_OPTIONS=--max-old-space-size=8192
```

**Issue: n8n UI not loading**
```bash
# Check if n8n is running
docker compose -p localai ps n8n

# Verify internal connectivity
docker compose -p localai exec caddy wget -O- http://n8n:5678/healthz

# Check for JavaScript errors
docker compose -p localai logs n8n | grep -E "ERROR|FATAL"

# Reset n8n user
docker compose -p localai exec n8n n8n user-management:reset
```

#### Supabase Troubleshooting

**Issue: Supabase services not starting**
```bash
# Check all Supabase services
docker compose -p localai ps | grep -E "kong|auth|meta|storage|realtime"

# Common issues:

# 1. Kong failing to start
docker compose -p localai logs kong
# Solution: Avoid @ in POSTGRES_PASSWORD

# 2. Auth service issues
docker compose -p localai logs auth
# Solution: Regenerate JWT secrets
openssl rand -hex 32  # For JWT_SECRET

# 3. Storage service issues
docker compose -p localai logs storage
# Solution: Check S3 credentials and bucket permissions

# 4. Database migrations failed
docker compose -p localai exec db psql -U postgres -d postgres -c "\dt"
```

**Issue: Supabase pooler restarting**
```bash
# Known issue: https://github.com/supabase/supabase/issues/30210
# Temporary fix:
docker compose -p localai restart db
sleep 10
docker compose -p localai restart kong auth meta storage realtime

# Permanent fix: Set in .env
POOLER_DB_POOL_SIZE=5
```

#### Ollama Troubleshooting

**Issue: Models not loading**
```bash
# Check Ollama logs
docker compose -p localai logs ollama --tail=50

# Common issues:

# 1. Insufficient GPU memory
nvidia-smi
# Solution: Use smaller models or CPU

# 2. Model download failed
docker compose -p localai exec ollama ollama list
docker compose -p localai exec ollama ollama pull qwen2.5:7b-instruct-q4_K_M

# 3. GPU not detected
docker compose -p localai exec ollama nvidia-smi
# Solution: Check NVIDIA runtime configuration

# 4. Context length errors
# Set OLLAMA_CONTEXT_LENGTH=2048 for lower memory usage
```

**Issue: Slow inference**
```bash
# Check if using GPU
docker compose -p localai exec ollama ollama ps

# Monitor GPU usage during inference
watch -n 1 nvidia-smi

# Use quantized models for better performance
docker compose -p localai exec ollama ollama pull qwen2.5:7b-instruct-q4_K_M
```

#### Open WebUI Troubleshooting

**Issue: Can't connect to Ollama**
```bash
# Verify Ollama connection
docker compose -p localai exec open-webui wget -O- http://ollama:11434/api/tags

# Check environment variables
docker compose -p localai exec open-webui env | grep OLLAMA

# For Mac users running Ollama locally:
# Set OLLAMA_BASE_URL=http://host.docker.internal:11434
```

**Issue: Authentication not working**
```bash
# Check session storage
docker compose -p localai exec open-webui ls -la /app/backend/data

# Reset admin password
docker compose -p localai exec open-webui sqlite3 /app/backend/data/webui.db "UPDATE users SET password='NEW_HASH' WHERE email='admin@example.com';"
```

#### Flowise Troubleshooting

**Issue: Custom tools not working**
```bash
# Check Flowise logs
docker compose -p localai logs flowise --tail=100

# Verify n8n webhook connectivity
docker compose -p localai exec flowise curl -X POST http://n8n:5678/webhook/test

# Check environment variables
docker compose -p localai exec flowise env | grep -E "headerauth|n8n_base_url"

# Test tool execution
curl -X POST http://localhost:3001/api/v1/prediction/[chatflow-id] \
  -H "Content-Type: application/json" \
  -d '{"question": "test"}'
```

#### Qdrant Troubleshooting

**Issue: Vector search not returning results**
```bash
# Check Qdrant health
curl http://localhost:6333/health

# List collections
curl http://localhost:6333/collections

# Check collection details
curl http://localhost:6333/collections/documents

# Rebuild index if corrupted
docker compose -p localai exec qdrant rm -rf /qdrant/storage/collections/documents
docker compose -p localai restart qdrant
```

#### Caddy Troubleshooting

**Issue: SSL certificates not generating**
```bash
# Check Caddy logs in detail
docker compose -p localai logs caddy -f

# Verify DNS propagation
for domain in n8n openwebui flowise supabase langfuse; do
  echo "Checking $domain.yourdomain.com:"
  dig +short $domain.yourdomain.com
done

# Force certificate renewal
docker compose -p localai exec caddy caddy renew --force

# Check Caddy storage
docker compose -p localai exec caddy ls -la /data/caddy
```

### Log Analysis and Debugging Guide

#### Centralized Log Analysis

```bash
# Create log analysis script
cat > ~/analyze-logs.sh <<'EOF'
#!/bin/bash

SERVICE=$1
PATTERN=$2
LINES=${3:-100}

if [ -z "$SERVICE" ]; then
    echo "Usage: $0 <service> [pattern] [lines]"
    echo "Services: n8n, ollama, open-webui, flowise, caddy, db, redis, etc."
    exit 1
fi

echo "=== Analyzing logs for $SERVICE ==="

# Get logs with timestamps
docker compose -p localai logs --timestamps --tail=$LINES $SERVICE > /tmp/${SERVICE}_logs.txt

# Extract errors and warnings
echo -e "\n=== Errors and Warnings ==="
grep -E "ERROR|WARN|FATAL|CRITICAL" /tmp/${SERVICE}_logs.txt | tail -20

# Extract patterns if provided
if [ ! -z "$PATTERN" ]; then
    echo -e "\n=== Pattern matches for: $PATTERN ==="
    grep -i "$PATTERN" /tmp/${SERVICE}_logs.txt
fi

# Show log statistics
echo -e "\n=== Log Statistics ==="
echo "Total lines: $(wc -l < /tmp/${SERVICE}_logs.txt)"
echo "Errors: $(grep -c ERROR /tmp/${SERVICE}_logs.txt)"
echo "Warnings: $(grep -c WARN /tmp/${SERVICE}_logs.txt)"

# Show recent activity
echo -e "\n=== Recent Activity (last 10 lines) ==="
tail -10 /tmp/${SERVICE}_logs.txt
EOF

chmod +x ~/analyze-logs.sh
```

#### Debug Mode Configuration

```bash
# Enable debug logging for services
cat >> ~/local-ai-packaged/.env <<'EOF'

# Debug flags
N8N_LOG_LEVEL=debug
OLLAMA_DEBUG=true
FLOWISE_LOG_LEVEL=debug
WEBUI_LOG_LEVEL=debug
POSTGRES_LOG_LEVEL=debug
CADDY_DEBUG=true
EOF

# Restart services to apply debug mode
docker compose -p localai restart
```

#### Interactive Debugging

```bash
# Create interactive debug session
cat > ~/debug-service.sh <<'EOF'
#!/bin/bash

SERVICE=$1

if [ -z "$SERVICE" ]; then
    echo "Usage: $0 <service>"
    exit 1
fi

echo "Starting debug session for $SERVICE..."

# Get container ID
CONTAINER=$(docker compose -p localai ps -q $SERVICE)

# Start interactive debug
case $SERVICE in
    n8n)
        docker exec -it $CONTAINER /bin/sh -c "npm install -g node-inspect && node inspect /usr/local/lib/node_modules/n8n/dist/index.js"
        ;;
    ollama)
        docker exec -it $CONTAINER /bin/bash -c "OLLAMA_DEBUG=true ollama serve"
        ;;
    db)
        docker exec -it $CONTAINER psql -U postgres
        ;;
    redis)
        docker exec -it $CONTAINER redis-cli monitor
        ;;
    *)
        docker exec -it $CONTAINER /bin/sh
        ;;
esac
EOF

chmod +x ~/debug-service.sh
```

#### Performance Profiling

```bash
# Create performance profiling script
cat > ~/profile-service.sh <<'EOF'
#!/bin/bash

SERVICE=$1
DURATION=${2:-60}

echo "Profiling $SERVICE for $DURATION seconds..."

# Start profiling
docker stats --no-stream --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}" > /tmp/profile_start.txt

# Collect metrics
for i in $(seq 1 $DURATION); do
    docker stats --no-stream --format "{{.Container}},{{.CPUPerc}},{{.MemUsage}},{{.NetIO}},{{.BlockIO}}" | grep $SERVICE >> /tmp/profile_$SERVICE.csv
    sleep 1
done

# Analyze results
echo -e "\n=== Profile Results for $SERVICE ==="
awk -F',' '{cpu+=$2; mem+=$3; count++} END {print "Avg CPU: " cpu/count "%\nAvg Memory: " mem/count}' /tmp/profile_$SERVICE.csv
EOF

chmod +x ~/profile-service.sh
```

### Common Issues

#### 1. Service Won't Start

```bash
# Comprehensive service startup diagnosis
cat > ~/diagnose-startup.sh <<'EOF'
#!/bin/bash

echo "=== LocalAI Startup Diagnosis ==="

# Check Docker
echo -e "\n1. Docker Status:"
docker version
docker compose version

# Check ports
echo -e "\n2. Port Availability:"
for port in 80 443 3000 3001 5678 6333 7474 8080 11434; do
    if lsof -i :$port > /dev/null 2>&1; then
        echo "Port $port: IN USE"
        lsof -i :$port | grep LISTEN
    else
        echo "Port $port: Available"
    fi
done

# Check resources
echo -e "\n3. System Resources:"
free -h
df -h /
docker system df

# Check environment
echo -e "\n4. Environment Variables:"
grep -c "^[A-Z_]*=" .env
grep -E "^[A-Z_]+=$" .env && echo "Found empty variables!"

# Check network
echo -e "\n5. Docker Network:"
docker network ls | grep localai
docker network inspect localai_default | jq '.[0].Containers | length'

echo -e "\n=== Diagnosis Complete ==="
EOF

chmod +x ~/diagnose-startup.sh
```

#### 2. SSL Certificate Issues

```bash
# Advanced SSL troubleshooting
cat > ~/fix-ssl.sh <<'EOF'
#!/bin/bash

echo "=== SSL Certificate Troubleshooting ==="

# Check Caddy certificate storage
echo -e "\n1. Certificate Storage:"
docker compose -p localai exec caddy ls -la /data/caddy/certificates

# Test ACME challenge
echo -e "\n2. Testing ACME HTTP Challenge:"
docker compose -p localai exec caddy curl -I http://localhost/.well-known/acme-challenge/test

# Check rate limits
echo -e "\n3. Let's Encrypt Rate Limits:"
curl -s https://crt.sh/?q=yourdomain.com | grep -c "Let's Encrypt"

# Force new certificate
echo -e "\n4. Forcing new certificate..."
docker compose -p localai exec caddy rm -rf /data/caddy/certificates
docker compose -p localai restart caddy

echo "Wait 30 seconds for certificate generation..."
sleep 30

# Verify certificates
echo -e "\n5. Verifying certificates:"
for domain in n8n openwebui flowise supabase langfuse; do
    echo -n "$domain.yourdomain.com: "
    echo | openssl s_client -servername $domain.yourdomain.com -connect $domain.yourdomain.com:443 2>/dev/null | openssl x509 -noout -dates
done
EOF

chmod +x ~/fix-ssl.sh
```

#### 3. Database Connection Errors

```bash
# Advanced database troubleshooting
cat > ~/fix-database.sh <<'EOF'
#!/bin/bash

echo "=== Database Connection Troubleshooting ==="

# Test basic connectivity
echo -e "\n1. Testing PostgreSQL connectivity:"
docker compose -p localai exec -T db pg_isready -U postgres

# Check connections
echo -e "\n2. Current connections:"
docker compose -p localai exec -T db psql -U postgres -c "SELECT datname, count(*) FROM pg_stat_activity GROUP BY datname;"

# Check locks
echo -e "\n3. Checking for locks:"
docker compose -p localai exec -T db psql -U postgres -c "SELECT pid, usename, application_name, state, query FROM pg_stat_activity WHERE state != 'idle';"

# Reset connections
echo -e "\n4. Resetting stale connections:"
docker compose -p localai exec -T db psql -U postgres -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE state = 'idle' AND state_change < NOW() - INTERVAL '1 hour';"

# Verify migrations
echo -e "\n5. Checking database migrations:"
docker compose -p localai exec -T db psql -U postgres -c "\dt *.*" | grep -E "migration|version"

echo -e "\n=== Database diagnosis complete ==="
EOF

chmod +x ~/fix-database.sh
```

#### 4. Memory Issues

```bash
# Advanced memory troubleshooting
cat > ~/fix-memory.sh <<'EOF'
#!/bin/bash

echo "=== Memory Optimization Script ==="

# Show current memory usage
echo -e "\n1. Current Memory Status:"
free -h
docker stats --no-stream --format "table {{.Container}}\t{{.MemUsage}}\t{{.MemPerc}}"

# Clear caches
echo -e "\n2. Clearing system caches:"
sync && echo 3 | sudo tee /proc/sys/vm/drop_caches

# Optimize Docker
echo -e "\n3. Cleaning Docker resources:"
docker system prune -af --volumes
docker builder prune -af

# Configure swap if needed
SWAP_SIZE=$(free -h | grep Swap | awk '{print $2}')
if [ "$SWAP_SIZE" = "0B" ]; then
    echo -e "\n4. Creating swap file:"
    sudo fallocate -l 16G /swapfile
    sudo chmod 600 /swapfile
    sudo mkswap /swapfile
    sudo swapon /swapfile
    echo "/swapfile none swap sw 0 0" | sudo tee -a /etc/fstab
fi

# Set memory limits
echo -e "\n5. Applying memory limits to containers..."
cat > docker-compose.override.yml <<YAML
version: '3.8'
services:
  ollama:
    deploy:
      resources:
        limits:
          memory: 8G
  n8n:
    deploy:
      resources:
        limits:
          memory: 4G
  db:
    deploy:
      resources:
        limits:
          memory: 4G
YAML

docker compose -p localai up -d

echo -e "\n=== Memory optimization complete ==="
EOF

chmod +x ~/fix-memory.sh
```

### Performance Optimization

#### 1. Ollama Optimization

```bash
# Use quantized models
docker compose -p localai exec ollama ollama pull qwen2.5:7b-instruct-q4_K_M

# Adjust context length in docker-compose.yml
environment:
  - OLLAMA_NUM_PARALLEL=2
  - OLLAMA_MAX_LOADED_MODELS=1
  - OLLAMA_CONTEXT_LENGTH=4096
```

#### 2. PostgreSQL Tuning

```sql
-- Connect to PostgreSQL
docker compose -p localai exec db psql -U postgres

-- Adjust settings
ALTER SYSTEM SET shared_buffers = '2GB';
ALTER SYSTEM SET effective_cache_size = '6GB';
ALTER SYSTEM SET maintenance_work_mem = '512MB';
ALTER SYSTEM SET work_mem = '32MB';

-- Reload configuration
SELECT pg_reload_conf();
```

#### 3. Docker Performance

```bash
# Use local volume driver for better performance
# In docker-compose.yml:
volumes:
  n8n_storage:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /mnt/fast-ssd/n8n_storage
```

### FAQ

**Q: How do I add a new user to n8n?**
A: Go to Settings → User Management → Invite User

**Q: Can I use external PostgreSQL?**
A: Yes, update the connection strings in .env and remove the db service from docker-compose.yml

**Q: How do I enable GPU for specific services only?**
A: Use Docker Compose profiles and specify GPU requirements per service

**Q: How can I monitor service metrics?**
A: Consider adding Prometheus + Grafana to the stack

**Q: Is it safe to expose Ollama publicly?**
A: No, keep it internal. Use Open WebUI as the public interface

## Architecture Diagrams

### System Architecture

The LocalAI package uses a microservices architecture with all services communicating through a Docker bridge network. Caddy handles SSL termination and reverse proxying for all services.

#### High-Level Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                        Internet                                  │
└──────────────────┬──────────────────────────┬───────────────────┘
                   │                          │
                   ▼                          ▼
              ┌────────┐                 ┌────────┐
              │ HTTP   │                 │ HTTPS  │
              │ (80)   │                 │ (443)  │
              └───┬────┘                 └────┬───┘
                  │                           │
                  └──────────┬────────────────┘
                             │
                    ┌────────▼────────┐
                    │     Caddy       │
                    │ Reverse Proxy   │
                    └────────┬────────┘
                             │
        ┌────────────────────┴────────────────────┐
        │          Docker Network (localai)        │
        │                                          │
        │  ┌─────────┐  ┌──────────┐  ┌────────┐ │
        │  │   n8n   │  │ Open     │  │Flowise │ │
        │  │  :5678  │  │ WebUI    │  │ :3001  │ │
        │  └────┬────┘  │  :8080   │  └───┬────┘ │
        │       │       └─────┬────┘      │      │
        │       │             │           │      │
        │  ┌────▼─────────────▼───────────▼────┐ │
        │  │         PostgreSQL (Supabase)      │ │
        │  │              :5432                 │ │
        │  └────────────────────────────────────┘ │
        │                                          │
        │  ┌─────────┐  ┌──────────┐  ┌────────┐ │
        │  │ Ollama  │  │  Qdrant  │  │ Neo4j  │ │
        │  │ :11434  │  │  :6333   │  │ :7474  │ │
        │  └─────────┘  └──────────┘  └────────┘ │
        │                                          │
        └──────────────────────────────────────────┘
```

#### Detailed Component Architecture

<!-- See the interactive Mermaid diagram above for a comprehensive view of all services and their connections -->

### Data Flow

The system implements a RAG (Retrieval-Augmented Generation) pattern for intelligent responses:

```
User Request → Caddy → Service → Database/Vector Store → Response
                ↓
            SSL/TLS
           Termination
                ↓
            Service
            Routing
                ↓
          Authentication
                ↓
            Process
                ↓
            Response
```

#### Request Processing Sequence

1. **User Query Flow**:
   - User submits question via Open WebUI
   - Request proxied through Caddy (HTTPS)
   - Open WebUI calls n8n webhook
   - n8n retrieves chat history from PostgreSQL
   - n8n performs vector search in Qdrant
   - n8n sends context + query to Ollama
   - Ollama returns AI-generated response
   - Response flows back to user

2. **Document Ingestion Flow**:
   - User uploads document
   - n8n processes and chunks document
   - Ollama generates embeddings (nomic-embed-text)
   - Vectors stored in Qdrant with metadata
   - Document indexed for future retrieval

<!-- See the sequence diagram above for detailed interaction flow -->

### Backup Architecture

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│   Volumes   │────▶│  Backup      │────▶│  Encrypted  │
│   - n8n     │     │  Script      │     │  Storage    │
│   - Qdrant  │     │              │     │             │
│   - WebUI   │     │  - Compress  │     │  - Local    │
└─────────────┘     │  - Encrypt   │     │  - S3       │
                    │  - Rotate    │     │             │
┌─────────────┐     │              │     └─────────────┘
│  Database   │────▶│              │
│  - Postgres │     └──────────────┘
└─────────────┘
```

### Network Topology

```
┌─────────────────────────────────────────────────────────┐
│                    Public Internet                       │
│                         │                                │
│    ┌────────────────────┼────────────────────┐          │
│    │                    │                    │          │
│    ▼                    ▼                    ▼          │
│ *.yourdomain.com   api.yourdomain.com   cdn.domain     │
└─────────────────────┬───────────────────────────────────┘
                      │
                 ┌────▼────┐
                 │ Firewall │
                 │ (UFW)    │
                 └────┬────┘
                      │
              ┌───────▼────────┐
              │  Port 80/443   │
              └───────┬────────┘
                      │
         ┌────────────▼────────────┐
         │      Caddy Proxy        │
         │  (SSL Termination)      │
         └────────────┬────────────┘
                      │
    ┌─────────────────┼─────────────────┐
    │         Docker Bridge Network      │
    │          (172.20.0.0/16)          │
    │                                    │
    │  ┌──────────┐  ┌──────────┐      │
    │  │ n8n      │  │ Supabase │      │
    │  │ .20.10   │  │ .20.20   │      │
    │  └──────────┘  └──────────┘      │
    │                                    │
    │  ┌──────────┐  ┌──────────┐      │
    │  │ Ollama   │  │ Qdrant   │      │
    │  │ .20.30   │  │ .20.40   │      │
    │  └──────────┘  └──────────┘      │
    └────────────────────────────────────┘
```

## Additional Resources

- [n8n Documentation](https://docs.n8n.io/)
- [Supabase Self-Hosting Guide](https://supabase.com/docs/guides/self-hosting)
- [Ollama Documentation](https://github.com/ollama/ollama)
- [Open WebUI Documentation](https://docs.openwebui.com/)
- [Caddy Documentation](https://caddyserver.com/docs/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)

For support, visit the [Local AI Community Forum](https://thinktank.ottomator.ai/c/local-ai/18).

## Appendix

### Environment Variables Reference

| Variable | Description | Example | Required |
|----------|-------------|---------|----------|
| **n8n Configuration** | | | |
| N8N_ENCRYPTION_KEY | Encryption key for credentials | 32 char hex string | ✅ |
| N8N_USER_MANAGEMENT_JWT_SECRET | JWT secret for auth | 32 char hex string | ✅ |
| N8N_WEBHOOK_TOKEN | Token for webhook auth | 32 char hex string | ✅ |
| **Supabase Configuration** | | | |
| POSTGRES_PASSWORD | PostgreSQL password (no @) | StrongPass123! | ✅ |
| JWT_SECRET | Supabase JWT secret | 32 char string | ✅ |
| ANON_KEY | Supabase anonymous key | JWT token | ✅ |
| SERVICE_ROLE_KEY | Supabase service key | JWT token | ✅ |
| POOLER_DB_POOL_SIZE | Connection pool size | 5 | ✅ |
| **Ollama Configuration** | | | |
| OLLAMA_MODELS | Default models to load | qwen2.5:7b | ❌ |
| OLLAMA_NUM_PARALLEL | Parallel requests | 2 | ❌ |
| OLLAMA_MAX_LOADED_MODELS | Models in memory | 2 | ❌ |
| **Caddy Configuration** | | | |
| N8N_HOSTNAME | n8n domain | n8n.yourdomain.com | ✅ |
| WEBUI_HOSTNAME | Open WebUI domain | openwebui.yourdomain.com | ✅ |
| FLOWISE_HOSTNAME | Flowise domain | flowise.yourdomain.com | ✅ |
| LETSENCRYPT_EMAIL | SSL cert email | admin@yourdomain.com | ✅ |

### Port Reference

| Service | Internal Port | External Port | Protocol | Notes |
|---------|--------------|---------------|----------|-------|
| Caddy | 80, 443 | 80, 443 | HTTP/HTTPS | Public facing |
| n8n | 5678 | - | HTTP | Internal only |
| PostgreSQL | 5432 | - | TCP | Internal only |
| Ollama | 11434 | - | HTTP | Internal only |
| Open WebUI | 8080 | - | HTTP | Internal only |
| Flowise | 3001 | - | HTTP | Internal only |
| Qdrant | 6333 | - | HTTP/gRPC | Internal only |
| Neo4j | 7474, 7687 | - | HTTP/Bolt | Internal only |
| Langfuse | 3000 | - | HTTP | Internal only |
| SearXNG | 8080 | - | HTTP | Internal only |
| Redis | 6379 | - | TCP | Internal only |

### Command Quick Reference

```bash
# Service Management
docker compose -p localai ps                    # Check service status
docker compose -p localai logs [service]        # View logs
docker compose -p localai restart [service]     # Restart service
docker compose -p localai down                  # Stop all services
python3 start_services.py --profile gpu-nvidia --environment public  # Start all

# Backup & Restore
~/backup-localai.sh                            # Create backup
~/rollback-localai.sh YYYYMMDD_HHMMSS         # Restore from backup

# Updates
~/update-localai.sh                           # Update all services
~/update-ollama-models.sh                     # Update AI models

# Monitoring
~/health-check.sh                             # Check service health
~/performance-monitor.sh                      # Monitor performance
~/analyze-logs.sh [service] [pattern]         # Analyze logs

# Troubleshooting
~/diagnose-startup.sh                         # Diagnose startup issues
~/fix-ssl.sh                                 # Fix SSL certificates
~/fix-database.sh                            # Fix database issues
~/fix-memory.sh                              # Fix memory issues

# Performance
~/tune-postgres.sh                           # Optimize PostgreSQL
~/optimize-ollama.sh                         # Optimize Ollama
~/benchmark-localai.sh                       # Run benchmarks
```

## Best Practices

### Security Best Practices
1. **Regular Updates**: Schedule weekly update checks and monthly security audits
2. **Principle of Least Privilege**: Only expose necessary services publicly
3. **Secrets Rotation**: Rotate passwords and API keys quarterly
4. **Backup Encryption**: Always encrypt backups before storage
5. **Access Logs**: Regularly review Caddy access logs for suspicious activity

### Operational Best Practices
1. **Documentation**: Maintain runbooks for common operations
2. **Change Management**: Test all changes in staging first
3. **Monitoring**: Set up alerts for critical metrics
4. **Capacity Planning**: Monitor resource usage trends
5. **Disaster Recovery**: Test restore procedures monthly

### Performance Best Practices
1. **Model Selection**: Use quantized models for better performance
2. **Resource Allocation**: Allocate resources based on actual usage
3. **Caching**: Enable Redis caching where appropriate
4. **Database Optimization**: Run VACUUM and ANALYZE regularly
5. **Container Limits**: Set memory and CPU limits to prevent resource exhaustion

### Development Best Practices
1. **Version Control**: Track all configuration changes in Git
2. **Environment Separation**: Maintain separate dev/staging/prod environments
3. **API Versioning**: Version your n8n webhooks and APIs
4. **Testing**: Test workflows thoroughly before production deployment
5. **Documentation**: Document all custom workflows and integrations

## Conclusion

Congratulations on deploying the Local AI Package! This production deployment provides you with a powerful, self-hosted AI and automation platform. 

### Next Steps
1. Import the provided n8n workflows to get started quickly
2. Customize the Open WebUI interface for your brand
3. Build custom Flowise agents for your use cases
4. Integrate with your existing tools via webhooks
5. Join the community forum for support and sharing

### Getting Help
- **Documentation**: Check the service-specific docs linked in Additional Resources
- **Community Forum**: https://thinktank.ottomator.ai/c/local-ai/18
- **Issues**: Report bugs on the GitHub repository
- **Updates**: Watch the repository for new features and updates

Remember to regularly update your deployment, monitor performance, and maintain backups. The Local AI Package is designed to grow with your needs, so don't hesitate to customize and extend it.

---

📝 **Document Version**: 1.0  
📅 **Last Updated**: December 2024  
👤 **Maintained by**: LocalAI Community  
📧 **Contact**: support@yourdomain.com  

**[⬆ Back to Top](#table-of-contents)** 