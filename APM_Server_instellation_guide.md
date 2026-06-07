# APM Server Integration with ELK Stack

> Full setup guide for integrating Elastic APM Server with an existing ELK Stack on **Rocky Linux**, and connecting a Node.js application using the APM Agent.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1 — Install and Configure APM Server](#step-1--install-and-configure-apm-server)
- [Step 2 — Open Firewall Port and Verify Connectivity](#step-2--open-firewall-port-and-verify-connectivity)
- [Step 3 — Add APM Agent to the Application](#step-3--add-apm-agent-to-the-application)
- [Step 4 — Verify Data in Kibana](#step-4--verify-data-in-kibana)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

Before starting, make sure the following are in place:

- A running **ELK Stack** (Elasticsearch + Logstash + Kibana) — version `9.x`
- **Rocky Linux** on both the ELK server and application host
- Elasticsearch is accessible and secured with a username/password
- Node.js application running on the head/main node
- `sudo` access on all servers

---

## Step 1 — Install and Configure APM Server

> Run these commands on the **ELK server**.

### 1.1 Download and Install

```bash
curl -L -O https://artifacts.elastic.co/downloads/apm-server/apm-server-9.4.2-x86_64.rpm
sudo rpm -ivh apm-server-9.4.2-x86_64.rpm
```

### 1.2 Find the Config File

```bash
find / -name "apm-server.yml"
```

It is usually located at:

```
/etc/apm-server/apm-server.yml
```

### 1.3 Edit the Configuration

```bash
sudo nano /etc/apm-server/apm-server.yml
```

Update the file with the following:

```yaml
apm-server:
  host: "0.0.0.0:8200"

output.elasticsearch:
  hosts: ["https://<ELK-Server-IP>:9200"]
  username: "elastic"
  password: "YourElasticPassword"
  ssl:
    verification_mode: none
```

> **Note:** Replace `<ELK-Server-IP>` and `YourElasticPassword` with your actual values.

### 1.4 Enable and Start APM Server

```bash
sudo systemctl enable apm-server
sudo systemctl start apm-server
sudo systemctl status apm-server
```

You should see `active (running)` in the status output.

---

## Step 2 — Open Firewall Port and Verify Connectivity

> Run these commands on the **ELK server**.

### 2.1 Open Port 8200

APM Server listens on port `8200`. Open it on the firewall:

```bash
sudo firewall-cmd --permanent --add-port=8200/tcp
sudo firewall-cmd --reload
```

Confirm the port is open:

```bash
sudo firewall-cmd --list-ports
```

### 2.2 Verify APM Server is Reachable

From the **application host**, run:

```bash
curl http://<ELK-Server-IP>:8200
```

Expected response:

```json
{
  "name": "apm-server",
  "version": "9.4.2",
  "build_date": "...",
  "build_sha": "...",
  "go_version": "..."
}
```

> If this fails, the APM agent on the application side will **not** be able to send data. Fix connectivity before moving to Step 3.

---

## Step 3 — Add APM Agent to the Application

> Run these commands on the **application host (head/main node)**.

### 3.1 Install the APM Agent

Inside your Node.js project directory:

```bash
npm install elastic-apm-node --save
```

### 3.2 Add Agent to Application Entry File

Open the **first file that loads** when your application starts (e.g., `server.js`, `app.js`, `index.js`).

Add the following as the **very first lines** — before any other `require` statements:

```javascript
var apm = require('elastic-apm-node').start({
  serviceName: 'my-service-name-55',
  secretToken: '',
  serverUrl: 'http://<ELK-Server-IP>:8200',
  environment: 'my-environment'
});
```

> ⚠️ **Important:** This must be placed before any other `require` or `import` in the file. If any other module loads first, APM will not be able to instrument it correctly.

### 3.3 Restart the Application

After adding the agent, restart your application:

```bash
pm2 restart all
# or
node server.js
```

---

## Step 4 — Verify Data in Kibana

Once the application is running and receiving traffic, verify that APM data is flowing into Kibana.

1. Open **Kibana** in your browser
2. Navigate to:

```
Observability → APM → Services
```

3. You should see your service (e.g., `my-service-name-55`) listed with:
   - Transactions
   - Traces
   - Error logs

---

## Troubleshooting

| Issue | What to Check |
|---|---|
| Service not showing in Kibana | Verify `serverUrl` in agent config matches APM server IP and port |
| `curl` to port 8200 fails | Check firewall rules and make sure APM server is running |
| APM server not starting | Check logs: `sudo journalctl -u apm-server -f` |
| No traces showing up | Make sure the APM agent `require` is the first line in the entry file |
| Elasticsearch connection error | Verify ELK IP, username, password, and SSL config in `apm-server.yml` |

---

## Notes

- APM Server version used: `9.4.2`
- OS: Rocky Linux
- Stack: ELK 9.x
- Agent: `elastic-apm-node`
<img width="1632" height="882" alt="Overview" src="https://github.com/user-attachments/assets/c51ec942-4f53-448e-a405-8453132ebc75" />
<img width="1632" height="882" alt="Transactions" src="https://github.com/user-attachments/assets/c2ba2871-ff0d-4ee3-862c-6e2a5223804b" />
<img width="1632" height="882" alt="Metrics" src="https://github.com/user-attachments/assets/33e9c3cb-bdfa-4403-8e3a-9b8353951ed5" />
