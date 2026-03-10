# AWS EC2 — Python Web Page Delivery Architecture (End to End)

> A complete guide to deploying a Python web application on AWS EC2 and serving it to the outside world — from DNS resolution to the browser.

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Full Architecture Diagram](#2-full-architecture-diagram)
3. [Component Breakdown](#3-component-breakdown)
4. [End-to-End Request Flow](#4-end-to-end-request-flow)
5. [Step-by-Step Setup](#5-step-by-step-setup)
6. [Security Layers](#6-security-layers)
7. [Python App Stack Detail](#7-python-app-stack-detail)
8. [Sample Configs & Code](#8-sample-configs--code)
9. [Flow Summary Cheat Sheet](#9-flow-summary-cheat-sheet)

---

## 1. Architecture Overview

```
Outside World (User's Browser)
         │
         ▼
  [ DNS Resolution ]         ← Route 53 / Domain Registrar
         │
         ▼
  [ Internet ]               ← Public Internet
         │
         ▼
  [ Internet Gateway ]       ← AWS VPC Entry Point
         │
         ▼
  [ Load Balancer ]          ← ALB (Application Load Balancer)  [OPTIONAL]
         │
         ▼
  [ Public Subnet ]          ← EC2 Instance (Nginx + Gunicorn + Python/Flask/Django)
         │
         ▼
  [ Private Subnet ]         ← RDS Database / S3 / ElastiCache  [OPTIONAL]
```

---

## 2. Full Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            OUTSIDE WORLD                                    │
│                                                                             │
│   👤 User Browser                                                           │
│   https://myapp.com                                                         │
└──────────────────────────────────┬──────────────────────────────────────────┘
                                   │  DNS Lookup
                                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         AWS Route 53 (DNS)                                  │
│                  myapp.com  →  54.210.10.220 (EIP)                          │
└──────────────────────────────────┬──────────────────────────────────────────┘
                                   │  HTTPS Request (Port 443)
                                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        AWS Region: us-east-1                                │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                    VPC  (10.0.0.0/16)                                 │  │
│  │                                                                       │  │
│  │   ┌─── Internet Gateway ──────────────────────────────────────────┐  │  │
│  │   │         igw-0abc123                                            │  │  │
│  │   └───────────────────────────┬───────────────────────────────────┘  │  │
│  │                               │                                       │  │
│  │   ┌─── Public Subnet (10.0.1.0/24) — AZ: us-east-1a ─────────────┐  │  │
│  │   │                           │                                   │  │  │
│  │   │   ┌─── Security Group (web-sg) ────────────────────────────┐  │  │  │
│  │   │   │  Inbound:  Port 443 (HTTPS) from 0.0.0.0/0            │  │  │  │
│  │   │   │  Inbound:  Port 80  (HTTP)  from 0.0.0.0/0            │  │  │  │
│  │   │   │  Inbound:  Port 22  (SSH)   from Admin IP only        │  │  │  │
│  │   │   │  Outbound: All traffic allowed                        │  │  │  │
│  │   │   │                                                        │  │  │  │
│  │   │   │   ┌────────────────────────────────────────────────┐  │  │  │  │
│  │   │   │   │         EC2 Instance (t3.micro)                │  │  │  │  │
│  │   │   │   │         EIP: 54.210.10.220                     │  │  │  │  │
│  │   │   │   │                                                │  │  │  │  │
│  │   │   │   │   ┌──────────────────────────────────────┐    │  │  │  │  │
│  │   │   │   │   │  NGINX (Port 80/443)                 │    │  │  │  │  │
│  │   │   │   │   │  Reverse Proxy + SSL Termination     │    │  │  │  │  │
│  │   │   │   │   └──────────────┬───────────────────────┘    │  │  │  │  │
│  │   │   │   │                  │ proxy_pass                  │  │  │  │  │
│  │   │   │   │   ┌──────────────▼───────────────────────┐    │  │  │  │  │
│  │   │   │   │   │  GUNICORN (Port 8000)                │    │  │  │  │  │
│  │   │   │   │   │  WSGI Application Server             │    │  │  │  │  │
│  │   │   │   │   │  Workers: 4                          │    │  │  │  │  │
│  │   │   │   │   └──────────────┬───────────────────────┘    │  │  │  │  │
│  │   │   │   │                  │ WSGI Call                   │  │  │  │  │
│  │   │   │   │   ┌──────────────▼───────────────────────┐    │  │  │  │  │
│  │   │   │   │   │  PYTHON APP (Flask / Django)         │    │  │  │  │  │
│  │   │   │   │   │  app.py / wsgi.py                    │    │  │  │  │  │
│  │   │   │   │   │  Renders HTML Templates              │    │  │  │  │  │
│  │   │   │   │   └──────────────────────────────────────┘    │  │  │  │  │
│  │   │   │   │                                                │  │  │  │  │
│  │   │   │   └────────────────────────────────────────────────┘  │  │  │  │
│  │   │   └────────────────────────────────────────────────────────┘  │  │  │
│  │   └───────────────────────────────────────────────────────────────┘  │  │
│  │                                                                       │  │
│  │   ┌─── Private Subnet (10.0.2.0/24) — AZ: us-east-1a ────────────┐  │  │
│  │   │                                                               │  │  │
│  │   │   ┌─── Security Group (db-sg) ─────────────────────────────┐  │  │  │
│  │   │   │  Inbound: Port 5432 (PostgreSQL) from web-sg only      │  │  │  │
│  │   │   │                                                         │  │  │  │
│  │   │   │   ┌─────────────────────────────────────────────────┐  │  │  │  │
│  │   │   │   │         RDS PostgreSQL (Optional)               │  │  │  │  │
│  │   │   │   │         Private IP: 10.0.2.100                  │  │  │  │  │
│  │   │   │   └─────────────────────────────────────────────────┘  │  │  │  │
│  │   │   └─────────────────────────────────────────────────────────┘  │  │  │
│  │   └───────────────────────────────────────────────────────────────┘  │  │
│  │                                                                       │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Component Breakdown

### Outside World

| Component | Role |
|-----------|------|
| **User Browser** | Initiates the HTTP/HTTPS request via domain name |
| **DNS (Route 53)** | Resolves `myapp.com` → EC2 Elastic IP address |
| **Public Internet** | Carries the request packets to AWS |

---

### AWS Networking Layer

| Component | Role |
|-----------|------|
| **VPC** (`10.0.0.0/16`) | Isolated virtual network for all resources |
| **Internet Gateway (IGW)** | Entry/exit point for internet traffic into the VPC |
| **Public Subnet** (`10.0.1.0/24`) | Subnet where EC2 lives; routes traffic to IGW |
| **Private Subnet** (`10.0.2.0/24`) | Isolated subnet for database; no internet route |
| **Route Table (Public)** | `0.0.0.0/0 → IGW` — routes internet traffic in/out |
| **Elastic IP (EIP)** | Static public IP attached to EC2 instance |

---

### Security Layer

| Component | Role |
|-----------|------|
| **Security Group (web-sg)** | Allows 80, 443 from world; 22 from admin only |
| **Security Group (db-sg)** | Allows 5432 only from `web-sg` (EC2) |
| **NACL** | Subnet-level stateless firewall (optional hardening) |
| **SSL Certificate (ACM)** | HTTPS encryption via Let's Encrypt or AWS ACM |

---

### EC2 Software Stack

| Layer | Software | Port | Role |
|-------|----------|------|------|
| **OS** | Amazon Linux 2 / Ubuntu 22.04 | — | Host OS |
| **Web Server** | Nginx | 80 / 443 | Reverse proxy, SSL termination, static files |
| **App Server** | Gunicorn | 8000 | WSGI server, manages Python worker processes |
| **App Framework** | Flask / Django | — | Python web application logic |
| **Runtime** | Python 3.11 | — | Language runtime |
| **Process Manager** | Systemd | — | Keeps Gunicorn running, auto-restart on crash |

---

## 4. End-to-End Request Flow

### Step-by-Step: User visits `https://myapp.com`

```
STEP 1 — DNS Resolution
───────────────────────
User types: https://myapp.com
Browser asks DNS: "What IP is myapp.com?"
Route 53 responds:  54.210.10.220  (EC2 Elastic IP)

STEP 2 — TCP Connection + TLS Handshake
────────────────────────────────────────
Browser connects to 54.210.10.220:443
TLS handshake occurs (SSL certificate verified)
Encrypted HTTPS session established

STEP 3 — Request Hits Internet Gateway
────────────────────────────────────────
Packet enters AWS VPC via Internet Gateway
IGW routes packet to the public subnet
EC2 Security Group checks: "Is port 443 allowed?" → YES ✅

STEP 4 — Nginx Receives the Request
─────────────────────────────────────
Nginx (Port 443) accepts the HTTPS request
SSL is terminated at Nginx (decrypted here)
Nginx checks: Is this a static file? → Serve directly
              Is this dynamic?       → Proxy to Gunicorn

STEP 5 — Gunicorn Processes the Request
─────────────────────────────────────────
Nginx forwards to Gunicorn at 127.0.0.1:8000
Gunicorn picks an available worker process
WSGI interface calls your Python app

STEP 6 — Python App Generates Response
────────────────────────────────────────
Flask/Django route handler executes
(Optional) Queries RDS database in private subnet
Renders HTML template with Jinja2
Returns HTTP 200 with HTML content

STEP 7 — Response Travels Back
────────────────────────────────
Python → Gunicorn → Nginx (re-encrypts) → IGW → Internet → Browser
Browser renders the HTML page
```

### Flow Diagram

```
Browser
  │
  │  1. DNS: myapp.com → 54.210.10.220
  ▼
Route 53
  │
  │  2. HTTPS GET / on port 443
  ▼
Internet Gateway (igw)
  │
  │  3. Security Group: Port 443 ✅ ALLOW
  ▼
EC2 Instance (54.210.10.220)
  │
  │  4. Nginx receives request, terminates SSL
  ▼
Nginx [:443]
  │
  │  5. proxy_pass → localhost:8000
  ▼
Gunicorn [:8000]
  │
  │  6. WSGI call to Python app
  ▼
Flask / Django App
  │
  │  7. (Optional) SQL query via private subnet
  ▼
RDS PostgreSQL (10.0.2.100)  ← Private Subnet
  │
  │  8. Data returned to Python app
  ▼
Flask renders HTML
  │
  │  9. HTTP 200 OK + HTML body
  ▼
Gunicorn → Nginx → IGW → Internet → Browser ✅
```

---

## 5. Step-by-Step Setup

### Phase 1 — VPC & Networking

```bash
# 1. Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# 2. Create Public Subnet
aws ec2 create-subnet \
  --vpc-id vpc-XXXXXX \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a

# 3. Create & Attach Internet Gateway
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --internet-gateway-id igw-XXXX --vpc-id vpc-XXXX

# 4. Create Route Table with internet route
aws ec2 create-route-table --vpc-id vpc-XXXX
aws ec2 create-route --route-table-id rtb-XXXX \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-XXXX
aws ec2 associate-route-table --route-table-id rtb-XXXX --subnet-id subnet-XXXX

# 5. Allocate & Associate Elastic IP to EC2
aws ec2 allocate-address --domain vpc
aws ec2 associate-address --instance-id i-XXXX --allocation-id eipalloc-XXXX
```

---

### Phase 2 — Security Group

```bash
# Create Security Group
aws ec2 create-security-group \
  --group-name web-sg \
  --description "Python Web App SG" \
  --vpc-id vpc-XXXX

# Allow HTTP
aws ec2 authorize-security-group-ingress \
  --group-id sg-XXXX --protocol tcp --port 80 --cidr 0.0.0.0/0

# Allow HTTPS
aws ec2 authorize-security-group-ingress \
  --group-id sg-XXXX --protocol tcp --port 443 --cidr 0.0.0.0/0

# Allow SSH from your IP only
aws ec2 authorize-security-group-ingress \
  --group-id sg-XXXX --protocol tcp --port 22 --cidr YOUR.IP.HERE/32
```

---

### Phase 3 — Launch EC2

```bash
aws ec2 run-instances \
  --image-id ami-0c02fb55956c7d316 \        # Amazon Linux 2
  --instance-type t3.micro \
  --key-name my-keypair \
  --subnet-id subnet-XXXX \
  --security-group-ids sg-XXXX \
  --associate-public-ip-address \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=python-web-server}]'
```

---

### Phase 4 — EC2 Software Setup

```bash
# SSH into EC2
ssh -i my-keypair.pem ec2-user@54.210.10.220

# Update packages
sudo yum update -y                          # Amazon Linux
# sudo apt update && sudo apt upgrade -y   # Ubuntu

# Install Python, pip, Nginx
sudo yum install python3 python3-pip nginx git -y

# Install Python packages
pip3 install flask gunicorn

# Create project directory
mkdir ~/myapp && cd ~/myapp
```

---

### Phase 5 — Configure Nginx

```bash
sudo nano /etc/nginx/conf.d/myapp.conf
```

---

### Phase 6 — Configure Systemd Service

```bash
sudo nano /etc/systemd/system/myapp.service
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start myapp
sudo systemctl start nginx
```

---

## 6. Security Layers

```
Layer 1 — DNS & Network Edge
  └── Route 53 → Resolves only to your EIP
  └── AWS Shield (DDoS protection, basic is free)

Layer 2 — VPC & Subnet
  └── Public subnet: EC2 exposed to internet (Port 80/443 only)
  └── Private subnet: RDS never exposed to internet

Layer 3 — Security Group (Stateful Firewall)
  └── web-sg: 80, 443 open; 22 from admin IP only
  └── db-sg:  5432 only from web-sg (not from internet!)

Layer 4 — OS Level (EC2)
  └── SSH key pair authentication only (no password)
  └── Fail2ban to block brute-force SSH attempts
  └── Regular OS patching (yum/apt update)

Layer 5 — Application Level (Nginx)
  └── SSL/TLS termination with strong ciphers
  └── Hide server version (server_tokens off)
  └── Rate limiting to prevent abuse

Layer 6 — Python App Level
  └── Input validation & sanitization
  └── CSRF protection (Django middleware / Flask-WTF)
  └── SQL injection prevention (ORM queries only)
  └── Environment variables for secrets (never hardcoded)
```

---

## 7. Python App Stack Detail

```
┌─────────────────────────────────────────────────────────┐
│                   EC2 Instance                          │
│                                                         │
│  Port 443/80                                            │
│  ┌───────────────────────────────────────────────────┐  │
│  │                    NGINX                          │  │
│  │  • Receives HTTPS requests from internet         │  │
│  │  • Terminates SSL (decrypts traffic)             │  │
│  │  • Serves /static/ files directly               │  │
│  │  • proxy_pass → 127.0.0.1:8000 (Gunicorn)       │  │
│  └───────────────────────┬───────────────────────────┘  │
│                           │ HTTP (internal only)         │
│  Port 8000                │                             │
│  ┌───────────────────────▼───────────────────────────┐  │
│  │                   GUNICORN                        │  │
│  │  • WSGI server (bridges Nginx ↔ Python)          │  │
│  │  • Manages multiple worker processes             │  │
│  │  • Workers = (2 × CPU cores) + 1                 │  │
│  │  • Handles concurrent requests                   │  │
│  └───────────────────────┬───────────────────────────┘  │
│                           │ Python function call         │
│  ┌───────────────────────▼───────────────────────────┐  │
│  │              FLASK / DJANGO APP                   │  │
│  │                                                   │  │
│  │  app.py / wsgi.py                                 │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │  Routes / Views                             │  │  │
│  │  │    @app.route('/')   → index()              │  │  │
│  │  │    @app.route('/about') → about()           │  │  │
│  │  └───────────────┬─────────────────────────────┘  │  │
│  │                  │                                 │  │
│  │  ┌───────────────▼─────────────────────────────┐  │  │
│  │  │  Jinja2 Templates                           │  │  │
│  │  │    templates/index.html                     │  │  │
│  │  │    templates/base.html                      │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  │                                                   │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │  Static Files (served by Nginx directly)    │  │  │
│  │  │    static/css/style.css                     │  │  │
│  │  │    static/js/app.js                         │  │  │
│  │  │    static/images/                           │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────┘  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 8. Sample Configs & Code

### Flask Application (`app.py`)

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html', title='My AWS App')

@app.route('/health')
def health():
    return {'status': 'healthy'}, 200

if __name__ == '__main__':
    app.run()
```

---

### WSGI Entry Point (`wsgi.py`)

```python
from app import app

if __name__ == '__main__':
    app.run()
```

---

### Gunicorn Start Command

```bash
gunicorn \
  --workers 4 \
  --bind 127.0.0.1:8000 \
  --access-logfile /var/log/gunicorn/access.log \
  --error-logfile /var/log/gunicorn/error.log \
  wsgi:app
```

---

### Nginx Config (`/etc/nginx/conf.d/myapp.conf`)

```nginx
server {
    listen 80;
    server_name myapp.com www.myapp.com;

    # Redirect HTTP → HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name myapp.com www.myapp.com;

    # SSL Certificates (Let's Encrypt)
    ssl_certificate     /etc/letsencrypt/live/myapp.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/myapp.com/privkey.pem;

    # Hide server version
    server_tokens off;

    # Static files served directly by Nginx (fast!)
    location /static/ {
        alias /home/ec2-user/myapp/static/;
        expires 30d;
    }

    # Dynamic requests → Gunicorn
    location / {
        proxy_pass         http://127.0.0.1:8000;
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
    }
}
```

---

### Systemd Service (`/etc/systemd/system/myapp.service`)

```ini
[Unit]
Description=Gunicorn instance for Python Web App
After=network.target

[Service]
User=ec2-user
Group=ec2-user
WorkingDirectory=/home/ec2-user/myapp
Environment="PATH=/home/ec2-user/myapp/venv/bin"
ExecStart=/home/ec2-user/myapp/venv/bin/gunicorn \
          --workers 4 \
          --bind 127.0.0.1:8000 \
          wsgi:app
Restart=always

[Install]
WantedBy=multi-user.target
```

---

### HTML Template (`templates/index.html`)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{ title }}</title>
    <link rel="stylesheet" href="/static/css/style.css">
</head>
<body>
    <h1>Hello from AWS EC2 + Python Flask! 🚀</h1>
    <p>Delivered via: EC2 → Nginx → Gunicorn → Flask</p>
</body>
</html>
```

---

### SSL Certificate Setup (Let's Encrypt)

```bash
# Install Certbot
sudo yum install certbot python3-certbot-nginx -y

# Obtain SSL certificate
sudo certbot --nginx -d myapp.com -d www.myapp.com

# Auto-renewal (runs twice daily)
sudo systemctl enable certbot-renew.timer
```

---

## 9. Flow Summary Cheat Sheet

```
USER ACTION           AWS COMPONENT            EC2 PROCESS
─────────────────────────────────────────────────────────────────────────
User types URL   →   Route 53 (DNS)        →   Resolves to EIP
Browser connects →   Internet Gateway      →   Routes to public subnet
HTTPS request    →   Security Group        →   Checks port 443 ✅
Packet arrives   →   EC2 Elastic IP        →   Nginx receives request
Nginx handles    →   SSL Termination       →   Decrypts HTTPS → HTTP
Nginx proxies    →   Gunicorn (Port 8000)  →   WSGI worker selected
Python runs      →   Flask/Django route    →   Handler function called
DB query (opt.)  →   RDS (private subnet)  →   SQL query executed
HTML rendered    →   Jinja2 template       →   Response built
Response sent    →   Nginx re-encrypts     →   HTTPS response
Travels back     →   IGW → Internet        →   Browser receives HTML
Browser renders  →   ✅ Page displayed!
```

---

### Port Reference

| Port | Protocol | Who Listens | Source |
|------|----------|-------------|--------|
| 443 | HTTPS | Nginx | 0.0.0.0/0 (internet) |
| 80 | HTTP | Nginx | 0.0.0.0/0 (redirects to 443) |
| 8000 | HTTP | Gunicorn | 127.0.0.1 (internal only) |
| 22 | SSH | OS/sshd | Admin IP only |
| 5432 | PostgreSQL | RDS | web-sg (EC2) only |

---

### Tech Stack Summary

```
DNS          →  AWS Route 53
CDN          →  AWS CloudFront (optional, for caching)
Load Balancer→  AWS ALB (optional, for scaling)
Compute      →  AWS EC2 (t3.micro or higher)
OS           →  Amazon Linux 2 / Ubuntu 22.04
Web Server   →  Nginx 1.24
App Server   →  Gunicorn 21+
Framework    →  Python Flask 3.x / Django 5.x
Runtime      →  Python 3.11+
Database     →  AWS RDS PostgreSQL (private subnet)
SSL          →  Let's Encrypt (Certbot) or AWS ACM
Process Mgmt →  Systemd
```

---

*Architecture Version 1.0 | AWS Region: us-east-1 | Last Updated: March 2026*
