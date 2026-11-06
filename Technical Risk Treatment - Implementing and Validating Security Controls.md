# INTERNATIONAL CYBERSECURITY AND DIGITAL FORENSICS ACADEMY

## GRC103: Risk Assessment and Management Techniques

### Week 3 Practical Laboratory:
# Technical Risk Treatment - Implementing and Validating Security Controls

**Full Name:** Venesha Adhiambo Ochieng  
**Registration Number:** C11/26/CGRCE/17566  
**Email Address:** c11.cgrce2617566@icdfa.edu.ng  
**Cohort / Batch:** Cohort11  
**Date Submitted:** 6th November 2026

---

You are the GRC analyst and technical control implementer for a fictional organisation, Northbridge Services. Its internal document server exposes a legacy endpoint and a directory of dummy files. Management requires measurable risk treatment without interrupting the approved web service.

## Required Local Environment

Use two VMs on your own computer: Ubuntu Server 24.04 LTS (2 vCPUs, 2–4 GB RAM, 25 GB disk) and Kali Linux (2 vCPUs, 2–4 GB RAM, 25 GB disk). Use the VM console for Ubuntu administration. A third VM is optional; blocked-client testing can instead use a second source IP on Kali.

Connect both VMs to the same isolated internal or host-only network. Use DHCP where available or assign unique addresses on the same subnet. Record the actual Ubuntu and Kali addresses. Do not use Metasploitable for these hardening steps. Take a clean VM checkpoint before starting.

Use only fictional files and your own VMs. A temporary NAT adapter may be used to install packages; disconnect it before creating the intentionally exposed lab services. Do not use bridged networking or expose this lab publicly.

# 1 Build the Baseline Environment

## Step 1 Prepare Ubuntu

While temporary NAT access is available, install the required packages. Then disconnect NAT, confirm the isolated adapter is active and record both VM addresses with `ip -br addr`. Keep the Ubuntu VM console open throughout the exercise.

```bash
sudo apt update
sudo apt install -y nginx ufw python3 curl
ip -br addr
```
![Step 1.1](images/1.1.png

### For Kali

```bash
ip -br addr
```
![Step 1.2](images/1.2.png)

## Step 2 Create Fictional Content and a Legacy Service

```bash
sudo mkdir -p /var/www/grc-lab/public /var/www/grc-lab/archive /srv/grc-legacy

printf 'Approved internal portal\n' | sudo tee /var/www/grc-lab/public/index.html

printf 'DUMMY RECORD A001 - training only\n' | sudo tee /var/www/grc-lab/archive/dummy-records.txt

printf 'Legacy training endpoint\n' | sudo tee /srv/grc-legacy/index.html

sudo chmod -R a+rX /var/www/grc-lab /srv/grc-legacy
```
![Step 2.1](images/2.1.png)

Create a lab-only systemd unit. This service is deliberately unnecessary; it contains no real data.

```bash
sudo tee /etc/systemd/system/grc-legacy.service > /dev/null <<'EOF'
[Unit]
Description=GRC lab legacy endpoint

[Service]
User=www-data
ExecStart=/usr/bin/python3 -m http.server 8080 --bind 0.0.0.0 --directory /srv/grc-legacy

[Install]
WantedBy=multi-user.target
EOF
```
![Step 2.2](images/2.2.png)

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now grc-legacy
```
![Step 2.3](images/2.3.png)

### Verification
![Step 2.4](images/2.4.png)

## Step 3 Configure the Lab Web Service

```bash
sudo tee /etc/nginx/sites-available/grc-lab > /dev/null <<'EOF'
server {
    listen 80;
    server_name _;
    root /var/www/grc-lab;
    access_log /var/log/nginx/grc_access.log combined;
    error_log /var/log/nginx/grc_error.log;
    location / { autoindex on; }
}
EOF
```
![Step 3.1](images/3.1.png)

```bash
sudo ln -s /etc/nginx/sites-available/grc-lab /etc/nginx/sites-enabled/grc-lab
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t && sudo systemctl reload nginx
```
![Step 3.2](images/3.2.png)

# 2 Measure Risk and Remove Exposure

## Step 4 Collect Baseline Evidence from Kali

Set SERVER_IP to your Ubuntu isolated-network address. Do not include angle brackets. Kali requires curl and Nmap. Save all evidence in a dedicated folder.

```bash
mkdir -p ~/grc-treatment-evidence
cd ~/grc-treatment-evidence

SERVER_IP=192.168.56.20
date -Is | tee baseline-time.txt
```
![Step 4.1](images/4.1.png)

```bash
nmap -sT -p 80,8080 "$SERVER_IP" -oN baseline-ports.txt
```
![Step 4.2](images/4.2.png)

```bash
curl -i "http://$SERVER_IP/" | tee baseline-web.txt
```
![Step 4.3](images/4.3.png)

```bash
curl -i "http://$SERVER_IP/archive/" | tee baseline-listing.txt
```
![Step 4.4](images/4.4.png)

```bash
curl -i "http://$SERVER_IP/archive/dummy-records.txt" | tee baseline-file.txt
```
![Step 4.5](images/4.5.png)

```bash
curl -i "http://$SERVER_IP:8080/" | tee baseline-legacy.txt
```
![Step 4.6](images/4.6.png)

Expected baseline: ports 80 and 8080 are reachable; the root and archive directories list files; the dummy file is retrievable.

On Ubuntu:

```bash
sudo ufw status verbose
sudo ss -lntp
```

If the clean lab already has firewall restrictions, resolve them on this disposable VM before collecting the baseline; do not alter a production firewall.

![Step 4.7](images/4.7.png)

## Step 5 Create a baseline risk register
Create four entries: R1 unnecessary legacy service, R2 exposed archive, R3 unauthorised network access and R4 loss of portal content. For each record the asset, threat event, observed weakness, impact, likelihood, evidence reference and treatment owner.
For this exercise use likelihood 1–5 and impact 1–5; score = likelihood × impact. Bands: 1–4 Low, 5–9 Medium, 10–16 High, 17–25 Critical. These are classroom decision thresholds, not an external standard. Scores are ordinal and must not be interpreted as monetary loss or statistical probabilities.

# 5.1 Risk Assessment Methodology

The assessment applies a qualitative 5 × 5 risk matrix to evaluate the likelihood and potential business impact of identified vulnerabilities before the implementation of security controls.

\[
\text{Risk Score}=\text{Likelihood}\times\text{Impact}
\]

The following classification thresholds are applied as specified in the assignment.

| Risk Score | Risk Rating | Recommended Response |
|------------|------------|----------------------|
| 1-4 | Low | Accept or monitor |
| 5-9 | Medium | Planned treatment |
| 10-16 | High | Prioritised risk treatment |
| 17-25 | Critical | Immediate escalation and treatment |

The ratings represent classroom risk prioritisation thresholds and are not externally prescribed standards. Likelihood and impact scores are ordinal assessments and must not be interpreted as financial losses or statistical probabilities.

## 5.2 Baseline Risk Register

The following risk register uses verified technical findings from the lab wherever available. The likelihood and impact scores are professional assessment judgments based on those observations and the fictional business scenario.

| Risk ID | Asset | Threat Event | Observed Weakness | Impact (1-5) | Likelihood (1-5) | Risk Score | Risk Rating | Evidence Reference | Treatment Owner |
|----------|--------|-------------|-------------------|--------------|------------------|------------|------------|-------------------|----------------|
| R1 - Unnecessary Legacy Service | Legacy HTTP endpoint - TCP 8080 | Unauthorised interaction with or exploitation of an unnecessary legacy service, potentially compromising server security or availability. | Nmap confirmed TCP 8080 open on 192.168.224.133. A direct HTTP request returned 200 OK, identifying SimpleHTTP/0.6 Python/3.12.3 and exposing the legacy training endpoint. | 4 | 4 | 16 | High | baseline-ports.txt, baseline-legacy.txt; report pp. 6 and 8 | IT Infrastructure / Server Administrator |
| R2 - Exposed Archive Directory | Internal document archive and dummy records | Unauthorised browsing and retrieval of archived documents, resulting in potential confidentiality breaches. | Nginx directory listing was enabled. Requests to /archive/ and /archive/dummy-records.txt returned 200 OK, allowing retrieval of DUMMY RECORD A001 - training only without authentication. | 4 | 4 | 16 | High | baseline-web.txt, baseline-listing.txt, baseline-file.txt; report pp. 7-8 | Information Asset Owner / Web Administrator |
| R3 - Unauthorised Network Access | Internal web server and network services | An unauthorised network device accesses exposed services without approved source-IP restrictions. | UFW reported Status: inactive. Ubuntu showed services listening on 0.0.0.0:80 and 0.0.0.0:8080, while Kali successfully connected to both ports. | 4 | 3 | 12 | High | sudo ufw status verbose, sudo ss -lntp, baseline-ports.txt; report pp. 6 and 8 | Network Security Administrator |
| R4 - Loss of Portal Content | Approved internal portal - /var/www/grc-lab/public/index.html | Accidental deletion or corruption of portal files causes service unavailability and disruption of approved internal operations. | The approved portal file was created and the web service was accessible. However, the submitted baseline evidence does not demonstrate a protected backup or successful restoration capability. Recovery effectiveness remains unverified. | 4 | 3 | 12 | High | Step 2 portal-file creation, p. 4; Step 4 baseline-web.txt, p. 7; recovery testing pending | IT Operations / Backup and Recovery Administrator |

## 5.3 Evidence-Based Risk Analysis

### R1 - Unnecessary Legacy Service (High Risk: 16)

The baseline Nmap scan performed from Kali Linux against Ubuntu Server (192.168.224.133) confirmed that TCP ports 80 and 8080 were open. The subsequent HTTP request to port 8080 returned HTTP/1.0 200 OK, identifying a Python SimpleHTTP server and displaying the legacy training endpoint.

This confirms that the unnecessary service was remotely accessible within the lab network, unnecessarily expanding the server's attack surface.

Although no exploitation was demonstrated, continued exposure introduces an avoidable opportunity for unauthorised interaction, service misuse or exploitation of future vulnerabilities.

**Recommended treatment - Risk Avoidance:** Retire the legacy service using `sudo systemctl disable --now grc-legacy` while preserving the approved Nginx service on port 80. Validate the change through post-treatment Nmap and HTTP tests.

### R2 - Exposed Archive Directory (High Risk: 16)

The baseline HTTP tests confirmed that Nginx permitted directory listing at the root and archive paths. The request to `/archive/` returned HTTP/1.1 200 OK, exposing a directory index containing `dummy-records.txt`.

A direct request to `/archive/dummy-records.txt` also returned HTTP/1.1 200 OK, revealing the fictional record DUMMY RECORD A001 - training only.

These results demonstrate that archived content could be discovered and retrieved without authentication or access restrictions.

**Recommended treatment - Risk Mitigation:** Relocate the archive outside the Nginx web root, restrict filesystem permissions, disable directory listing and configure explicit HTTP 403 denial for the retired archive path.

### R3 - Unauthorised Network Access (High Risk: 12)

The Ubuntu baseline firewall assessment returned Status: inactive, confirming that UFW was not enforcing host-based filtering rules.

The `ss -lntp` output identified services listening on `0.0.0.0:80` and `0.0.0.0:8080`. Kali's Nmap scan independently confirmed that both ports were accessible from the test machine.

These observations establish that the server lacked host-based restrictions limiting access to an explicitly approved source address. They do not establish exposure to the public internet.

**Recommended treatment - Risk Mitigation:** Implement a default-deny UFW policy, permit HTTP access only from the approved Kali IP address and validate both permitted and blocked requests using firewall rules, connectivity tests and log evidence.

### R4 - Loss of Portal Content (High Risk: 12)

The baseline implementation created the approved portal content at `/var/www/grc-lab/public/index.html`, and the Nginx service was confirmed to be accessible through HTTP.

However, the evidence provided does not establish that the portal has a protected backup or that its content can be successfully restored following deletion or corruption.

This presents a potential availability and recoverability risk. The assessment reflects the absence of demonstrated recovery assurance rather than evidence that a data-loss incident has occurred.

**Recommended treatment - Risk Mitigation and Transfer Evaluation:** Create a protected local backup, record the original file's SHA-256 hash, simulate deletion and perform a verified restoration. Evaluate managed backup services as a potential risk-sharing arrangement while retaining organisational responsibility for recovery assurance.
