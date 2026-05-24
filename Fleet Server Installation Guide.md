# 🚀 Fleet Server Installation Guide  <IF ANY ISSUES IN ADDING FLEET AGENT FROM DEFAULT METHOD>

<p align="center">
Self-signed SSL deployment for Elastic Fleet Server (ELK 9.3.3)
</p>

---

## 📑 Table of Contents

- [📋 Prerequisites](#-prerequisites)
- [🔐 Step 1 — Get Elasticsearch CA Fingerprint](#-step-1--get-elasticsearch-ca-fingerprint)
- [🎫 Step 2 — Get Fleet Service Token](#-step-2--get-fleet-service-token)
- [⚙️ Step 3 — Install Fleet Server](#️-step-3--install-fleet-server)
- [✅ Step 4 — Verify Fleet Server](#-step-4--verify-fleet-server)
- [🖥️ Step 5 — Install Agent on Another Machine](#️-step-5--install-agent-on-another-machine)
- [🚨 Troubleshooting](#-troubleshooting)

---

## 📋 Prerequisites

Ensure the following before starting:

- Elasticsearch running
- Kibana accessible
- Fleet enabled
- Port `9200` → Elasticsearch
- Port `8220` → Fleet Server

---

## 🔐 Step 1 — Get Elasticsearch CA Fingerprint

Run this on the ELK server:

```bash
 openssl x509 -fingerprint -sha256 -in /etc/elasticsearch/certs/http_ca.crt -noout     -->   RUN THIS CMD IN THE ELK SERVER..
```

Expected output:

```text
sha256 Fingerprint=29:1D:B2:C0:75:A6:78:E3:21:66:80:4A:94:F0:AD:2F
```

Convert the fingerprint:

✅ Remove `:` characters

✅ Convert to lowercase

Final output:

```text
291db2c075a678e32166804a94f0ad2f3ba0ccf3f916d9a8420b93a84b4956d5
```

---

## 🎫 Step 2 — Get Fleet Service Token

Navigate in Kibana:

```text
Kibana
└── Management
    └── Fleet
        └── Add Fleet Server
```

Copy the generated service token:

```text
AAEAAWVsYXN0aWMvZmxlZXQ...   <fleet-server-service-token=AAEAAWVs.....................>
```

---

## ⚙️ Step 3 — Install Fleet Server

Download Elastic Agent:

```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-9.3.3-linux-x86_64.tar.gz

tar xzvf elastic-agent-9.3.3-linux-x86_64.tar.gz

cd elastic-agent-9.3.3-linux-x86_64
```

Install Fleet Server:

```bash
sudo ./elastic-agent install \
--fleet-server-es=https://YOUR_ELK_IP:9200 \
--fleet-server-service-token=<TOKEN FREOM STEP 2> \
--fleet-server-policy=fleet-server-policy \
--fleet-server-es-ca-trusted-fingerprint=<FINGERPRINT FROM STEP 1> \
--fleet-server-port=8220
```

---

## ✅ Step 4 — Verify Fleet Server

Check Elastic Agent service:

```bash
sudo systemctl status elastic-agent
```

WE CAN USE THIS TO ADD FLEET AGENT TO THE SERVER OR ANY OTHER INSTENSE.




### .
### .
### .
### .
### .
### .
### .
### .
### .
### .



# IF FLEET AGENT IS "OFFLINE"

# Elastic Agent — SSL / Offline Troubleshooting Runbook

> Use this guide when your agent shows **Offline** and the log contains `x509: certificate signed by unknown authority`.

---

## 1  Confirm the Problem

```bash
sudo tail -f /var/log/elastic-agent/elastic-agent.log
```

Look for:

```
x509: certificate signed by unknown authority
```

If you see that error, pick one of the three fixes below.

---

## 2  Fix Options

### Option A — Disable SSL Verification (Quick / Temporary)

Edit the agent config:

```bash
sudo nano /etc/elastic-agent/elastic-agent.yml
```

Update the `outputs` block:

```yaml
outputs:
  default:
    type: elasticsearch
    hosts:
      - https://<your-elasticsearch-ip>:9200
    ssl:
      verification_mode: none   # ← add this line
```

> ⚠️ This disables certificate validation entirely. Use only for testing or in a trusted private network.

---

### Option B — Disable Verification via Fleet UI (Recommended for Managed Agents)

1. Open **Kibana → Fleet → Settings** (gear icon, top right)
2. Find the **Outputs** section → click **Edit** on the default output
3. Scroll to **SSL / Advanced YAML** settings
4. Add the following line and save:

```yaml
ssl.verification_mode: none
```

Changes pushed through Fleet propagate to all enrolled agents automatically.

---

### Option C — Trust the CA Certificate (Most Secure)

**Step 1 — Copy the CA cert from the Elasticsearch server:**

```bash
sudo cat /etc/elasticsearch/certs/ca/ca.crt
```

**Step 2 — Place it on the agent machine:**

```bash
sudo mkdir -p /etc/elastic-agent/certs
sudo nano /etc/elastic-agent/certs/ca.crt
# Paste the certificate content and save
```

**Step 3 — Point the agent config at the cert:**

```bash
sudo nano /etc/elastic-agent/elastic-agent.yml
```

```yaml
outputs:
  default:
    type: elasticsearch
    hosts:
      - https://<your-elasticsearch-ip>:9200
    ssl:
      verification_mode: certificate
      certificate_authorities:
        - /etc/elastic-agent/certs/ca.crt
```

---

## 3  Restart and Verify

After applying **any** of the options above:

```bash
sudo systemctl restart elastic-agent
sudo elastic-agent status
```

The agent status should change from `STARTING` / `FAILED` to **`HEALTHY`**.

---

## 4  Still Not Working? Re-enroll the Agent

If the agent stays stuck on `STARTING`, or port `8220` (Fleet Server) returns a `404`, the agent cannot reach Fleet to pull the updated config. Re-enrollment is required.

### Why this happens

| Symptom | Cause |
|---|---|
| Status stuck on `STARTING` | Agent cannot contact Fleet Server |
| Port 8220 returns `404` | Fleet Server unreachable or misconfigured |
| Kibana fix applied but agent still fails | Agent never received the new config |

### Re-enrollment Command

```bash
sudo /opt/Elastic/Agent/elastic-agent enroll \
  --url=https://<your-fleet-server-ip>:8220 \
  --enrollment-token=<YOUR_ENROLLMENT_TOKEN> \
  --insecure \
  --force
```

Then restart and check status:

```bash
sudo systemctl restart elastic-agent
sudo elastic-agent status
```

---

## 5  Quick Reference

```
Agent offline
    └── Check logs → x509 error?
            ├── Yes → Choose a fix
            │       ├── Option A: ssl.verification_mode: none  (in yml)
            │       ├── Option B: Fleet UI → Output → Advanced YAML
            │       └── Option C: Copy CA cert → reference in yml
            │
            └── Restart agent
                    └── Still STARTING / 404 on 8220?
                            └── Re-enroll with --insecure --force
```

---

> **Tip:** For production environments, Option C (CA certificate trust) is strongly preferred over disabling verification entirely.
