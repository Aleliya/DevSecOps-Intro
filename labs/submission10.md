# Lab 10

## Task 1

### 1.1: Clone and Start DefectDojo

**Results:**

- Repository cloned successfully  
- Build completed successfully  
- Container status:  
  - All containers healthy and running  
  - nginx exposed on port 8080  
  - postgres running  
  - redis/valkey running  
  - uwsgi running  
  - celery workers running  

### 1.2: Get Admin Credentials

**Admin credentials obtained:**

- Username: admin  
- Password: ***

**Source:** Extracted from initializer container logs using:
docker compose logs initializer


## Task 2

### 2.1: Get API Token and Set Variables

**API Configuration:**

- API URL: http://localhost:8080/api/v2  
- API Token: 60ddae754963d6d9ede193cb1ea8fe51692f5f11  

**Environment variables:**

export DD_API="http://localhost:8080/api/v2"  
export DD_TOKEN="60ddae754963d6d9ede193cb1ea8fe51692f5f11"  
export DD_PRODUCT_TYPE="Engineering"  
export DD_PRODUCT="Juice Shop"  
export DD_ENGAGEMENT="Labs Security Testing"  


### 2.3: Run the Importer Script

**Import results:**

- ZAP: Imported successfully — 0 findings  
- Semgrep: Imported successfully — 25 findings  
- Trivy: Imported successfully — 74 findings  
  (9 critical, 28 high, 33 medium, 4 low)  
- Nuclei: Imported successfully — 18 findings  
- Grype: Imported successfully — 65 findings  

Total findings imported: 182 across 5 tools  

Import response files saved to:  
labs/lab10/imports/


## Task 3

### 3.1: Create a Baseline Progress Snapshot

Metrics snapshot created:  
labs/lab10/report/metrics-snapshot.md  

Date captured: April 10, 2026  

**Active findings by severity:**

- Critical: 17  
- High: 55  
- Medium: 76  
- Low: 5  
- Informational: 29  

Total active findings: 182  

**Status overview:**

- Open findings: 182  
- Closed findings: 0  
- All findings are currently active and unverified  
- No findings have been mitigated or closed  


### 3.2: Generate Governance-Ready Artifacts

Generated artifacts:

- labs/lab10/report/dojo-report.html — HTML engagement report with all findings  
- labs/lab10/report/findings.csv — CSV export of all findings with key details  


### 3.3: Extract Key Metrics

**Key metrics summary:**

- Total findings: 182 active findings imported from 5 security tools  

**Findings by tool:**

- Trivy: 74  
- Grype: 65  
- Semgrep: 25  
- Nuclei: 18  
- ZAP: 0  

**Findings by severity:**

- 17 critical  
- 55 high  
- 76 medium  
- 5 low  
- 29 informational  

**Status overview:**

- All 182 findings are active and verified  
- No findings have been mitigated, closed, or marked as false positives  

**SLA status:**

- 17 SLA breaches or items due within the next 14 days  

**Risk acceptance:**

- No findings have been risk-accepted  

**Top recurring CWE categories:**

- CWE-89  
- CWE-79  
- CWE-73  
- CWE-548  
- CWE-674  