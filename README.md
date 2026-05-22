I can see the issue — your Markdown formatting isn't rendering properly because the code blocks and headings aren't using correct Markdown syntax. Let me fix it for you.Here's your content properly formatted as clean Markdown. You can copy this into your `.md` file:Here's the clean Markdown source you can paste into your `.md` file on GitHub:

```markdown
## Additional Kibana configuration

Edit `/etc/kibana/kibana.yml` and ensure the following lines are set:

```yaml
server.port: 5601
server.host: 0.0.0.0
elasticsearch.hosts: ["https://151.185.412.86 <OR> LOCALHOST:9200"]
elasticsearch.username: "kibana_system"
elasticsearch.password: "ejxtpWEDFCVBHJKwUdJ=W9Ytd7X"
elasticsearch.ssl.certificateAuthorities: ["/etc/kibana/certs/http_ca.crt"]
elasticsearch.ssl.verificationMode: certificate
` ` `

## Fixing the "Unable to initialize Fleet" error

### Step 1 — Generate a secure 32+ character key

```bash
openssl rand -hex 32
` ` `

### Step 2 — Add the keys to `kibana.yml`

```yaml
xpack.encryptedSavedObjects.encryptionKey: "your-generated-32-char-key-here"
xpack.security.encryptionKey: "another-32-char-key-here"
xpack.reporting.encryptionKey: "yet-another-32-char-key-here"
` ` `

### Step 3 — Restart Kibana

```bash
sudo systemctl restart kibana
` ` `

> ⏳ Wait a moment after restarting — the Kibana dashboard will load shortly and Fleet agents can then be added.
```

The key fixes were: wrapping all commands in fenced code blocks with language tags (` ```yaml `, ` ```bash `), using proper `##` / `###` heading syntax, and adding a blockquote `>` for the final note.
