# 🔐 QuantumICA

**Enterprise-Grade Quantum-Safe Intermediate Certificate Authority**

[![License](https://img.shields.io/badge/license-Proprietary-blue.svg)]()
[![API](https://img.shields.io/badge/API-142%20endpoints-green.svg)]()
[![PQC](https://img.shields.io/badge/PQC-Dilithium%20%7C%20Kyber-purple.svg)]()
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED.svg)]()

QuantumICA is a next-generation Certificate Authority platform built for the post-quantum era. It provides complete certificate lifecycle management with native support for NIST-approved post-quantum cryptographic algorithms (Dilithium, Kyber) alongside classical algorithms (RSA, ECDSA).

---

## 📑 Table of Contents

- [Features](#-features)
- [Quick Start](#-quick-start)
- [API Documentation](#-api-documentation)
- [Certificate Operations Guide](#-certificate-operations-guide)
- [Architecture](#-architecture)
- [Configuration](#-configuration)
- [Development](#-development)
- [Testing](#-testing)
- [Troubleshooting](#-troubleshooting)

---

## ✨ Features

### Core PKI Capabilities
- **Certificate Issuance** - Issue X.509 certificates with CSR support
- **Certificate Revocation** - Revoke certificates with RFC 5280 reason codes
- **CRL Management** - Automatic CRL generation and distribution
- **OCSP Responder** - Real-time certificate status checking
- **Certificate Templates** - Pre-configured issuance policies

### Quantum-Safe Cryptography
- **Dilithium-2/3/5** - NIST-approved digital signatures
- **Kyber-512/768/1024** - NIST-approved key encapsulation
- **Hybrid Certificates** - Classical + PQC for migration path
- **Crypto Agility** - Hot-swappable algorithm support

### Enterprise Protocols
- **ACME** - Automated Certificate Management (RFC 8555)
- **EST** - Enrollment over Secure Transport (RFC 7030)
- **SCEP** - Simple Certificate Enrollment Protocol
- **Certificate Transparency** - CT log monitoring and submission

### Advanced Features
- **Workload Identity** - Cloud-native identity for Kubernetes/VMs
- **Code Signing** - Software supply chain security
- **IoT Device Management** - Fleet certificate provisioning
- **Multi-Region Support** - Distributed CA deployment
- **Audit Logging** - Tamper-evident audit chain

---

## 🚀 Quick Start

### Prerequisites

- Docker & Docker Compose
- Git
- 4GB RAM minimum
- Ports: 8000 (API), 8080 (UI), 5432 (PostgreSQL), 6379 (Redis)

### 1. Clone the Repository

```bash
git clone https://github.com/SulabhAgarwal/QuantumICA.git
cd QuantumICA
```

### 2. Start the Services

```bash
# Start all services
docker-compose -f docker-compose.local.yml up -d

# Check service status
docker-compose -f docker-compose.local.yml ps
```

### 3. Verify Services are Running

| Service | URL | Purpose |
|---------|-----|---------|
| **API Backend** | http://localhost:8000 | REST API |
| **API Docs (Swagger)** | http://localhost:8000/docs | Interactive API documentation |
| **API Docs (ReDoc)** | http://localhost:8000/redoc | Alternative API docs |
| **Frontend UI** | http://localhost:8080 | Operator Console |
| **Health Check** | http://localhost:8000/api/health | Service health |

### 4. Quick Health Check

```bash
# Check API health
curl http://localhost:8000/api/health

# Expected response:
# {"status":"healthy","timestamp":"2025-11-28T..."}
```

---

## 📚 API Documentation

### Interactive Documentation

QuantumICA provides **live, interactive API documentation** at:

| Documentation | URL | Description |
|---------------|-----|-------------|
| **Swagger UI** | [http://localhost:8000/docs](http://localhost:8000/docs) | Interactive API testing |
| **ReDoc** | [http://localhost:8000/redoc](http://localhost:8000/redoc) | Clean API reference |
| **OpenAPI JSON** | [http://localhost:8000/openapi.json](http://localhost:8000/openapi.json) | Machine-readable spec |

### API Overview

The API exposes **142 endpoints** organized into these categories:

| Category | Base Path | Description |
|----------|-----------|-------------|
| **Health** | `/api/health`, `/api/ready` | Service health checks |
| **Certificates** | `/api/v1/certificates` | Issue, list, view certificates |
| **Revocation** | `/api/v1/revocation` | Revoke/unrevoke certificates |
| **CRL** | `/api/v1/crl` | CRL management |
| **OCSP** | `/api/v1/ocsp` | OCSP responder |
| **Templates** | `/api/v1/templates` | Certificate templates |
| **Audit** | `/api/v1/audit` | Audit logs |
| **ACME** | `/api/v1/acme` | ACME protocol |
| **EST** | `/est` | EST protocol |
| **SCEP** | `/scep` | SCEP protocol |
| **Lifecycle** | `/api/v1/lifecycle` | Auto-renewal policies |
| **Workload** | `/api/v1/workload` | Workload identity |
| **Supply Chain** | `/api/v1/supply-chain` | Code signing |
| **CT** | `/api/v1/certificate-transparency` | CT monitoring |
| **Crypto** | `/api/v1/crypto` | Key generation, CSR creation |

---

## 🔑 Certificate Operations Guide

This guide walks you through the complete certificate lifecycle: **Generate → Issue → View → Revoke → Check Status**

### Step 1: Generate a Key Pair and CSR

You can generate keys and CSRs using OpenSSL or the API.

#### Option A: Using OpenSSL (RSA)

```bash
# Generate RSA private key
openssl genrsa -out my_private.key 2048

# Generate CSR
openssl req -new -key my_private.key -out my_request.csr \
  -subj "/CN=myserver.example.com/O=My Organization/C=US"

# View CSR content (for API)
cat my_request.csr
```

#### Option B: Using OpenSSL (ECDSA)

```bash
# Generate ECDSA private key
openssl ecparam -genkey -name prime256v1 -out my_private.key

# Generate CSR
openssl req -new -key my_private.key -out my_request.csr \
  -subj "/CN=myserver.example.com/O=My Organization/C=US"
```

#### Option C: Using the API

```bash
# Generate key pair via API
curl -X POST http://localhost:8000/api/v1/crypto/generate-keypair \
  -H "Content-Type: application/json" \
  -d '{
    "algorithm": "RSA",
    "key_size": 2048
  }'

# Generate CSR via API
curl -X POST http://localhost:8000/api/v1/crypto/generate-csr \
  -H "Content-Type: application/json" \
  -d '{
    "common_name": "myserver.example.com",
    "organization": "My Organization",
    "country": "US",
    "algorithm": "RSA",
    "key_size": 2048
  }'
```

---

### Step 2: Issue a Certificate

#### Using curl (Linux/Mac)

```bash
# Read your CSR file and issue certificate
curl -X POST http://localhost:8000/api/v1/certificates/issue_sync \
  -H "Content-Type: application/json" \
  -d "{
    \"csr_pem\": \"$(cat my_request.csr | sed ':a;N;$!ba;s/\n/\\n/g')\",
    \"validity_days\": 365
  }"
```

#### Using PowerShell (Windows)

```powershell
# Read CSR content
$csrContent = Get-Content -Path "my_request.csr" -Raw

# Issue certificate
$body = @{
    csr_pem = $csrContent
    validity_days = 365
} | ConvertTo-Json

$response = Invoke-RestMethod -Uri "http://localhost:8000/api/v1/certificates/issue_sync" `
    -Method POST `
    -ContentType "application/json" `
    -Body $body

# Display the issued certificate
$response.certificate_pem

# Save the serial number for later
$serialNumber = $response.serial_number
Write-Host "Serial Number: $serialNumber"
```

#### Request Body Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `csr_pem` | string | ✅ Yes | PEM-encoded PKCS#10 CSR |
| `template_id` | string | No | Template ID to apply |
| `validity_days` | integer | No | Validity period (default: 365) |
| `metadata` | object | No | Custom metadata |

#### Example Response

```json
{
  "serial_number": "7A8B9C0D1E2F3A4B",
  "certificate_pem": "-----BEGIN CERTIFICATE-----\nMIID...\n-----END CERTIFICATE-----",
  "issuer": "CN=QuantumICA Intermediate CA",
  "subject": "CN=myserver.example.com,O=My Organization,C=US",
  "not_before": "2025-11-28T00:00:00Z",
  "not_after": "2026-11-28T00:00:00Z",
  "algorithm": "RSA-2048"
}
```

---

### Step 3: View Certificate Details

```bash
# Get certificate by serial number
curl http://localhost:8000/api/v1/certificates/7A8B9C0D1E2F3A4B

# List all certificates
curl http://localhost:8000/api/v1/certificates/

# List with pagination
curl "http://localhost:8000/api/v1/certificates/?limit=10&offset=0"
```

---

### Step 4: Revoke a Certificate

```bash
curl -X POST http://localhost:8000/api/v1/revocation/revoke \
  -H "Content-Type: application/json" \
  -d '{
    "serial_number": "7A8B9C0D1E2F3A4B",
    "reason": "keyCompromise",
    "comment": "Private key was exposed in a security incident"
  }'
```

#### PowerShell

```powershell
$body = @{
    serial_number = "7A8B9C0D1E2F3A4B"
    reason = "keyCompromise"
    comment = "Private key was exposed"
} | ConvertTo-Json

Invoke-RestMethod -Uri "http://localhost:8000/api/v1/revocation/revoke" `
    -Method POST `
    -ContentType "application/json" `
    -Body $body
```

#### Revocation Reasons (RFC 5280)

| Reason | Code | When to Use |
|--------|------|-------------|
| `unspecified` | 0 | No specific reason |
| `keyCompromise` | 1 | Private key exposed or stolen |
| `caCompromise` | 2 | Issuing CA key compromised |
| `affiliationChanged` | 3 | Subject's org changed |
| `superseded` | 4 | New certificate issued |
| `cessationOfOperation` | 5 | Service decommissioned |
| `certificateHold` | 6 | Temporary suspension |
| `privilegeWithdrawn` | 9 | Access rights revoked |

---

### Step 5: Check Revocation Status

#### Via API

```bash
# Check revocation status
curl http://localhost:8000/api/v1/revocation/7A8B9C0D1E2F3A4B
```

#### Via OCSP

```bash
# Get OCSP response
curl http://localhost:8000/api/v1/ocsp/7A8B9C0D1E2F3A4B.der --output status.der
```

#### Via CRL

```bash
# Get latest CRL
curl http://localhost:8000/api/v1/crl/latest --output crl.pem

# Get delta CRL (changes since last full CRL)
curl http://localhost:8000/api/v1/crl/delta
```

---

### Step 6: Unrevoke a Certificate (from Hold)

If a certificate was revoked with reason `certificateHold`, it can be restored:

```bash
curl -X POST http://localhost:8000/api/v1/revocation/unrevoke \
  -H "Content-Type: application/json" \
  -d '{
    "serial_number": "7A8B9C0D1E2F3A4B"
  }'
```

---

## 🏗 Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                       QuantumICA Stack                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐      ┌──────────────┐      ┌──────────────┐   │
│  │   Frontend   │      │   Backend    │      │  PostgreSQL  │   │
│  │  (React UI)  │─────▶│  (FastAPI)   │─────▶│  (Database)  │   │
│  │   :8080      │      │   :8000      │      │   :5432      │   │
│  └──────────────┘      └──────┬───────┘      └──────────────┘   │
│                               │                                  │
│                               ▼                                  │
│                        ┌──────────────┐                          │
│                        │    Redis     │                          │
│                        │  (Caching)   │                          │
│                        │   :6379      │                          │
│                        └──────────────┘                          │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                     PKI Services                            │ │
│  ├────────────────────────────────────────────────────────────┤ │
│  │  • Certificate Issuance     • OCSP Responder               │ │
│  │  • CRL Generation           • CT Log Submission            │ │
│  │  • Key Management           • HSM Integration (PKCS#11)    │ │
│  │  • PQC (Dilithium/Kyber)    • Hybrid Crypto                │ │
│  │  • ACME/EST/SCEP            • Workload Identity            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Technology Stack

| Layer | Technology | Version |
|-------|------------|---------|
| **Frontend** | React, TypeScript, Ant Design | 18.x |
| **Backend** | Python, FastAPI, Uvicorn | 3.11+, 0.110+ |
| **Database** | PostgreSQL | 15+ |
| **Cache** | Redis | 7+ |
| **Queue** | Kafka (optional) | 3.x |
| **Crypto** | cryptography, liboqs | 42.x |
| **Container** | Docker, Docker Compose | 24.x |

### Project Structure

```
QuantumICA/
├── backend/
│   ├── src/quantumica/
│   │   ├── domain/                    # Domain models (DDD)
│   │   │   ├── certificate_authority/ # CA operations
│   │   │   ├── quantum_security/      # PQC algorithms
│   │   │   ├── workload_identity/     # Cloud identity
│   │   │   └── ...                    # 22 bounded contexts
│   │   ├── application/               # Application services
│   │   ├── infrastructure/
│   │   │   ├── crypto/                # PQC, HSM, keystore
│   │   │   │   ├── pqc_liboqs/        # Dilithium, Kyber
│   │   │   │   └── hsm/               # PKCS#11
│   │   │   ├── persistence/orm/       # SQLAlchemy models
│   │   │   └── pki_services/          # CA operations
│   │   └── interfaces/api/rest/       # 142 REST endpoints
│   └── tests/                         # Unit, integration, UAT
├── frontend/
│   └── operator-console/              # React UI
│       ├── src/pages/                 # 18 page modules
│       └── src/api/                   # API client
├── platform/
│   └── docker/                        # Dockerfiles
├── docs/                              # Documentation
└── docker-compose.local.yml           # Local dev stack
```

---

## ⚙️ Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `DATABASE_URL` | `postgresql://...` | PostgreSQL connection |
| `REDIS_URL` | `redis://redis:6379` | Redis connection |
| `SECRET_KEY` | (required) | JWT signing key |
| `LIBOQS_ENABLED` | `true` | Enable PQC algorithms |
| `HSM_ENABLED` | `false` | Enable HSM integration |

### Docker Services

```yaml
# docker-compose.local.yml services:
services:
  postgres:           # PostgreSQL database (:5432)
  redis:              # Redis cache (:6379)
  kafka:              # Event streaming (:9092)
  quantumica-backend: # FastAPI backend (:8000)
  quantumica-frontend: # React frontend (:8080)
```

---

## 💻 Development

### Running Locally

```bash
# Start the full stack
docker-compose -f docker-compose.local.yml up -d

# View logs
docker-compose -f docker-compose.local.yml logs -f quantumica-backend

# Rebuild after code changes
docker-compose -f docker-compose.local.yml build --no-cache quantumica-backend
docker-compose -f docker-compose.local.yml up -d
```

### Backend Development

```bash
cd backend
python -m venv venv
source venv/bin/activate  # or .\venv\Scripts\activate on Windows
pip install -r requirements.txt
uvicorn quantumica.main:app --reload --port 8000
```

### Frontend Development

```bash
cd frontend/operator-console
npm install
npm run dev
```

---

## 🧪 Testing

### Quick API Tests

```bash
# Health check
curl http://localhost:8000/api/health

# List algorithms
curl http://localhost:8000/api/v1/crypto/algorithms

# Get CA certificate
curl http://localhost:8000/api/v1/system/ca/certificate
```

### Run Backend Tests

```bash
cd backend
pytest                              # Run all tests
pytest -v                           # Verbose output
pytest --cov=quantumica             # With coverage
pytest tests/unit/                  # Unit tests only
```

---

## 🔧 Troubleshooting

### Services Won't Start

```bash
# Check logs
docker-compose -f docker-compose.local.yml logs

# Restart all services
docker-compose -f docker-compose.local.yml down
docker-compose -f docker-compose.local.yml up -d
```

### Database Issues

```bash
# Connect to database
docker exec -it quantumica-postgres-1 psql -U quantumica -d quantumica

# Check tables
\dt
```

### Port Conflicts

```bash
# Check what's using a port
netstat -ano | findstr :8000   # Windows
lsof -i :8000                   # Linux/Mac
```

### Rebuild Everything

```bash
docker-compose -f docker-compose.local.yml down -v
docker-compose -f docker-compose.local.yml build --no-cache
docker-compose -f docker-compose.local.yml up -d
```

---

## 📊 API Quick Reference

### Core Operations

| Operation | Method | Endpoint |
|-----------|--------|----------|
| Issue Certificate | `POST` | `/api/v1/certificates/issue_sync` |
| List Certificates | `GET` | `/api/v1/certificates/` |
| Get Certificate | `GET` | `/api/v1/certificates/{serial}` |
| Revoke Certificate | `POST` | `/api/v1/revocation/revoke` |
| Unrevoke | `POST` | `/api/v1/revocation/unrevoke` |
| Get CRL | `GET` | `/api/v1/crl/latest` |
| OCSP Check | `GET` | `/api/v1/ocsp/{serial}.der` |

### Key Generation

| Operation | Method | Endpoint |
|-----------|--------|----------|
| Generate Key Pair | `POST` | `/api/v1/crypto/generate-keypair` |
| Generate CSR | `POST` | `/api/v1/crypto/generate-csr` |
| List Algorithms | `GET` | `/api/v1/crypto/algorithms` |

### Templates & Policies

| Operation | Method | Endpoint |
|-----------|--------|----------|
| List Templates | `GET` | `/api/v1/templates/` |
| Create Template | `POST` | `/api/v1/templates/` |
| List Policies | `GET` | `/api/v1/lifecycle/policies` |

---

## 📄 License

Copyright © 2025 QuNulabs Private Limited. All rights reserved.

---

## 📧 Contact

- **Issues**: [GitHub Issues](https://github.com/SulabhAgarwal/QuantumICA/issues)
- **Email**: support@qunulabs.com

---

<p align="center">
  <b>QuantumICA</b> - Securing the Post-Quantum Future 🔐
</p>