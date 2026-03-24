# SecretSifter — User Guide
**Version 1.1.0 · Windows Edition**

---

## Table of Contents

1. [Installation](#1-installation)
2. [Overview](#2-overview)
3. [Quick Start](#3-quick-start)
4. [Scan Tab](#4-scan-tab)
   - 4.1 [Entering Targets](#41-entering-targets)
   - 4.2 [Scan Modes](#42-scan-modes)
   - 4.3 [Running a Scan](#43-running-a-scan)
   - 4.4 [Target Status Panel](#44-target-status-panel)
   - 4.5 [Results Table](#45-results-table)
   - 4.6 [Filtering & Searching Results](#46-filtering--searching-results)
   - 4.7 [Masking Sensitive Data](#47-masking-sensitive-data)
   - 4.8 [Exporting Results](#48-exporting-results)
   - 4.9 [HAR File Import](#49-har-file-import)
5. [Settings Tab](#5-settings-tab)
   - 5.1 [Scan Tier](#51-scan-tier)
   - 5.2 [Browser Mode](#52-browser-mode)
   - 5.3 [Entropy Threshold](#53-entropy-threshold)
   - 5.4 [PII Detection](#54-pii-detection)
   - 5.5 [Scan Request Headers](#55-scan-request-headers)
   - 5.6 [CDN Blocklist](#56-cdn-blocklist)
   - 5.7 [Key Name Blocklist](#57-key-name-blocklist)
   - 5.8 [Key Name Allowlist](#58-key-name-allowlist)
6. [Logs Tab](#6-logs-tab)
7. [What SecretSifter Detects](#7-what-secretsifter-detects)
8. [Scan Tiers Explained](#8-scan-tiers-explained)
9. [Browser Mode Deep Dive](#9-browser-mode-deep-dive)
10. [Understanding Results](#10-understanding-results)
11. [HTML Reports](#11-html-reports)
12. [Tips & Best Practices](#12-tips--best-practices)
13. [Troubleshooting](#13-troubleshooting)
14. [License](#14-license)

---

## 1. Installation

### System Requirements
- Windows 10 or Windows 11 (64-bit)
- 4 GB RAM minimum (8 GB recommended for browser mode)
- ~500 MB disk space
- Internet connection (for scanning remote targets)

### Installing SecretSifter
1. Double-click **SecretSifter-1.1.0-windows-Setup.exe**
2. Click **Yes** on the UAC prompt if shown
3. Follow the installer wizard (Next → Install → Finish)
4. SecretSifter will appear in your Start Menu under **SecretSifter**

### First Launch
- On first launch, Windows SmartScreen may show a warning — click **More info → Run anyway**
- The app opens directly to the **Scan** tab

---

## 2. Overview

SecretSifter scans websites for accidentally exposed credentials, API keys, tokens, and other secrets in JavaScript files, API responses, and page source code.

**Three main tabs:**

| Tab | Purpose |
|-----|---------|
| **Scan** | Enter targets, run scans, view and export results |
| **Settings** | Configure scan depth, browser mode, filters, noise suppression |
| **Logs** | Real-time activity log with search and download |

---

## 3. Quick Start

1. Open SecretSifter
2. Paste one or more URLs into the target box (one per line)
3. Click **Start Scan**
4. Watch the Target Status panel and Results table populate in real time
5. When the scan finishes, click **Export → HTML Report** to save findings

---

## 4. Scan Tab

### 4.1 Entering Targets

Paste URLs into the large text area at the top — **one URL per line**.

```
https://example.com
https://app.acmecorp.com/login
https://api.target.com
```

**Supported URL types:**
- Public websites
- Web applications (login pages, dashboards)
- API endpoints
- Any HTTP/HTTPS URL

> **Tip:** Include both the main site and any known API subdomains for thorough coverage.

---

### 4.2 Scan Modes

SecretSifter has two scan modes selectable via a toggle in the Scan tab:

#### Simple Mode (HTTP Fetch)
- Fetches the page HTML over HTTP/HTTPS
- Extracts and scans all inline `<script>` blocks
- Follows all `<script src="...">` references
- Optionally follows webpack chunk imports
- Fast and lightweight — no browser required
- **Best for:** Public pages, APIs, quick reconnaissance

#### Browser Mode (Playwright / Headless Chromium)
- Launches a real headless browser (Microsoft Edge or Chromium)
- Renders each page fully, including JavaScript-rendered content
- Captures **all** network requests and responses in real time
- Crawls linked pages (BFS) up to the configured page limit
- Detects secrets in dynamically loaded JS chunks, API calls, and POST bodies
- **Best for:** Single-page apps (React, Vue, Angular), apps requiring login simulation, comprehensive audits

> **Note:** Browser mode downloads ~120 MB of Chromium on first use if Microsoft Edge is not installed.

---

### 4.3 Running a Scan

1. Enter your target URLs
2. Select **Simple** or **Browser** mode
3. Click **Start Scan**

**During the scan:**
- The progress bar shows files scanned vs. total discovered
- The status label shows current activity (e.g., "Crawling & scanning… 12 / ~45 resource(s)")
- The current file label shows the URL being processed
- Summary counters update in real time:
  - **Scanned** — URLs successfully processed (green)
  - **Failed** — URLs that errored (red)
  - **Auth (401)** — URLs returning authentication errors (orange)
  - **JS Found / Fetched** — JavaScript resources discovered and downloaded
  - **Elapsed** — scan timer

4. Click **Stop** at any time to cancel
5. Results remain in the table after stopping

---

### 4.4 Target Status Panel

The Target Status panel (left side) shows a row per scanned URL:

| Column | Meaning |
|--------|---------|
| **✔** (green) | URL scanned successfully |
| **✖** (red) | URL failed (network error, timeout, DNS failure) |
| **●** (amber) | Authentication error (HTTP 401 / 403) |
| **URL** | The target URL |
| **Detail** | Additional context (e.g., error message, HTTP status) |

---

### 4.5 Results Table

Each row represents one detected finding:

| Column | Description |
|--------|-------------|
| **Severity** | HIGH / MEDIUM / LOW / INFO |
| **Confidence** | CERTAIN / FIRM / TENTATIVE |
| **Rule** | The detection rule that matched (e.g., `github-pat`, `aws-access-key`) |
| **Key** | The variable/key name where the secret was found |
| **Value** | The detected secret value (masked by default) |
| **URL** | Source URL where the secret was found |
| **Line** | Line number within the source file |
| **Context** | Surrounding code snippet (masked by default) |

**Right-click any row** for:
- Copy value to clipboard
- Copy entire row
- Open source URL in browser

---

### 4.6 Filtering & Searching Results

Use the filter bar above the results table:

| Filter | How to Use |
|--------|-----------|
| **Severity** | Dropdown — All, Critical+, High+, Medium+, Low+ |
| **Domain** | Dropdown — filter by target domain |
| **Rule** | Dropdown — filter by detection rule ID |
| **Search** | Free-text — searches across all columns |

Click any **column header** to sort results by that column.

---

### 4.7 Masking Sensitive Data

Three toggle buttons control what is visible in the results table:

| Button | Effect |
|--------|--------|
| **Mask Values** | Shows first 4 + last 4 characters only (e.g., `AIza••••••••1234`) |
| **Mask URLs** | Replaces URLs with `••••••••••••` |
| **Mask Context** | Shows first 6 + last 6 characters of context snippets |

> **Tip:** Enable all three masks before taking screenshots for reports or sharing findings with others.

---

### 4.8 Exporting Results

Click the **Export** button for output options:

| Format | Description |
|--------|-------------|
| **HTML Report** | Single self-contained HTML file — best for sharing |
| **Per-Domain HTML (ZIP)** | One HTML file per domain, bundled in a zip |
| **CSV** | Spreadsheet-compatible, all columns |
| **JSON** | Structured data for integration with other tools |

All exports include the full finding metadata: severity, confidence, rule, key name, value, URL, line number, and context.

---

### 4.9 HAR File Import

HAR (HTTP Archive) files capture all browser network traffic and are useful for scanning traffic recorded during a manual browsing session.

**How to record a HAR file:**
1. Open Chrome/Edge DevTools (`F12`)
2. Go to the **Network** tab
3. Check **Preserve log**
4. Browse the target application normally
5. Right-click in the Network tab → **Save all as HAR with content**

**How to import into SecretSifter:**
1. Click **Import HAR** in the Scan tab
2. Select your `.har` file
3. Click **Scan HAR**

SecretSifter will scan all captured request headers, request bodies, and response bodies from the HAR recording.

---

## 5. Settings Tab

All settings are saved automatically when you close the app.

---

### 5.1 Scan Tier

Controls the depth and breadth of secret detection.

| Tier | Speed | What It Scans |
|------|-------|--------------|
| **FAST** | Fastest | Anchored vendor tokens (50+ API key patterns), URL-embedded credentials |
| **LIGHT** | Balanced | All FAST + database/broker connection strings, context-gated rules |
| **FULL** | Most thorough | All LIGHT + generic key=value pairs, SSR state blobs (Next.js, Redux), high-entropy tokens, JavaScript getter secrets, JSON deep walk, XML scanning, PII |

> **Recommendation:** Use **LIGHT** for routine scans. Use **FULL** for in-depth security assessments.

---

### 5.2 Browser Mode

| Setting | Description |
|---------|-------------|
| **Enable headless browser** | Toggle Playwright-based browser crawling on/off |
| **Max pages per target** | How many pages to crawl per URL (1–2000, default 100) |
| **Custom browser path** | Path to a specific Edge or Chrome executable |

**First-time setup:**
- If Microsoft Edge is installed, SecretSifter uses it automatically (no setup needed)
- If not, click **Install Chromium** — a one-time ~120 MB download

---

### 5.3 Entropy Threshold

Controls how random/complex a value must be to be reported as a secret.

- **Default:** 3.5 bits/character (Shannon entropy)
- **Lower value** → more findings, more noise
- **Higher value** → fewer findings, less noise
- **Range:** 0.0 – 6.0

> **Tip:** If you're getting too many false positives, increase to 4.0. If you're missing secrets, lower to 3.0.

---

### 5.4 PII Detection

When enabled, SecretSifter also scans for:
- **Social Security Numbers** (SSN) — in context (e.g., `ssn: 123-45-6789`)
- **Credit card numbers** — Luhn-validated

> **Note:** Disabled by default. Enable only when explicitly authorized to scan for PII.

---

### 5.5 Scan Request Headers

When enabled (Browser mode), SecretSifter scans HTTP request headers sent by the browser, such as:
- `Authorization: Bearer <token>`
- `X-API-Key: <key>`
- `X-Auth-Token: <token>`

Common non-credential headers (Host, Accept, Content-Type, etc.) are automatically skipped.

---

### 5.6 CDN Blocklist

A list of hostnames whose JavaScript files are skipped during scanning. Pre-configured with 20+ common CDN and analytics hosts:

- `cdnjs.cloudflare.com`, `cdn.jsdelivr.net`, `unpkg.com`
- `google-analytics.com`, `googletagmanager.com`, `hotjar.com`
- `segment.com`, `doubleclick.net`, `sentry.io`
- And many more

**To add a host:**
1. Type the hostname in the input box
2. Click **Add**

**To remove a host:**
1. Select it from the list
2. Click **Remove**

**To search:**
- Use the search box above the list

> **Tip:** Add internal CDN hosts or third-party script hosts you don't own to reduce noise.

---

### 5.7 Key Name Blocklist

Suppresses findings where the key name contains any of the listed substrings. Pre-configured with:
- `STORAGE_KEY_`, `STATE_KEY_` — React/Redux state management keys
- `NEXT_PUBLIC_` — Next.js public environment variables (intentionally public)
- `REACT_APP_PUBLIC_`, `VUE_APP_PUBLIC_` — Framework public variables

**To add:** Type a substring and click **Add**
**To remove:** Select the entry and click **Remove**

> **Example:** Add `TEST_KEY` to suppress all test API key findings.

---

### 5.8 Key Name Allowlist

Forces SecretSifter to always report a finding even if it fails entropy or other checks, as long as the key name contains a listed substring.

**To add:** Type a substring and click **Add**
**To remove:** Select the entry and click **Remove**

> **Example:** Add `apimSubscriptionKey` to ensure Azure API Management keys are always reported regardless of entropy score.

---

## 6. Logs Tab

The Logs tab shows a real-time activity log of everything SecretSifter does.

| Control | Description |
|---------|-------------|
| **Search box** | Filter log lines by keyword |
| **Auto-scroll** | Toggle automatic scrolling to latest entry |
| **Line count** | Shows total number of log entries |
| **Download** | Save the entire log as a `.txt` file |

Log lines are color-coded:
- **Gray** — informational messages
- **Red** — errors and warnings

> **Tip:** If a URL fails, check the Logs tab for the exact error message (e.g., SSL error, DNS resolution failure, timeout).

---

## 7. What SecretSifter Detects

SecretSifter detects **100+ secret types** across these categories:

### Cloud & Infrastructure
AWS Access Keys, Google API Keys, Google Cloud Service Accounts, Azure Storage connections, Azure App Insights, DigitalOcean tokens, Alibaba Cloud keys, GCP OAuth tokens

### Source Control & CI/CD
GitHub PATs (classic, OAuth, Actions, fine-grained), GitLab PATs and deploy tokens, CircleCI tokens, Terraform Cloud tokens, Buildkite tokens, Harness tokens

### Payment Processors
Stripe (secret, restricted, publishable, webhook), PayPal, Razorpay, Square, PayStack, Braintree, Flutterwave

### Communication & Messaging
Slack (bot, user, app, webhook tokens), Twilio SIDs, Discord bots and webhooks, Telegram bot tokens, Microsoft Teams webhooks

### Email & Marketing
SendGrid, Mailchimp, Mailgun, Klaviyo, SendinBlue/Brevo

### Monitoring & Observability
New Relic, Datadog, Dynatrace, Sentry, Grafana, Elastic

### AI & ML Platforms
OpenAI API keys, Anthropic API keys, Groq, Replicate, Hugging Face, DeepSeek, LangSmith, Langfuse, xAI

### Security & Identity
HashiCorp Vault tokens, Okta SSWS tokens, Atlassian API tokens, Contentful tokens, 1Password service accounts

### Databases & Brokers
MySQL, PostgreSQL, MongoDB, Redis, RabbitMQ connection strings, SQL Server ADO.NET strings

### Development & Productivity
Figma tokens, Postman API keys, Airtable PATs, Notion tokens, Asana tokens, Linear API keys, Hubspot tokens, Shopify tokens, Dropbox tokens, Netlify tokens, Fly.io tokens, PlanetScale tokens

### Generic Secrets
High-entropy tokens in semantic context (`api_key`, `client_secret`, `auth_token`, `db_password`, etc.)

### Framework State Blobs
Next.js `__NEXT_DATA__`, Redux `__REDUX_STATE__`, `window.__CONFIG__`, `window._env_`, inline JSON script blocks

### Private Keys & Certificates
PEM private keys (RSA, EC, DSA, OpenSSH), bcrypt password hashes

### Credentials in URLs
`https://username:password@hostname`

### PII (when enabled)
Social Security Numbers, Credit card numbers (Luhn-validated)

---

## 8. Scan Tiers Explained

### FAST
Catches the most common, high-value secrets quickly:
- Vendor-anchored tokens with strong prefixes (e.g., `ghp_`, `AKIA`, `sk-`)
- URL-embedded credentials

### LIGHT
Everything in FAST plus:
- Database and broker connection strings
- Context-gated patterns (value must appear near a semantic keyword like `api`, `token`, `secret` within 80 characters)

### FULL
Everything in LIGHT plus:
- Generic key=value scanning with entropy analysis
- SSR/SPA state blob scanning (`__NEXT_DATA__`, `window.__CONFIG__`, etc.)
- JavaScript getter function secrets
- Deep JSON walk (up to 20 levels, up to 50 findings per blob)
- XML element value scanning
- PII (if also enabled in settings)

---

## 9. Browser Mode Deep Dive

### How It Works

1. SecretSifter launches a headless browser (Edge or Chromium)
2. For each target URL, it opens a new isolated browser context
3. Before navigating, it intercepts all network requests to capture:
   - Request headers
   - POST/PUT/PATCH request bodies
   - Response bodies (JS, JSON, XML, HTML)
4. The page is loaded and allowed to settle (up to 30 seconds)
5. Light scrolling is performed to trigger lazy-loaded content
6. All discovered links on the same domain are added to the crawl queue (BFS)
7. Steps 3–6 repeat for each queued page up to the configured page limit
8. Scanning runs in parallel with crawling for real-time results

### Scope Control

- Crawling is **scoped to the registrable domain** of each seed URL
  - e.g., seeding `app.example.com` will crawl `*.example.com` but not `other.com`
- JavaScript files from CDN hosts (in the CDN blocklist) are automatically skipped
- JSON/XML API responses are allowed from cross-origin hosts

### Destructive Link Avoidance

Links containing these words are never followed:
`logout`, `signout`, `delete`, `remove`

---

## 10. Understanding Results

### Severity Levels

| Level | Meaning |
|-------|---------|
| **HIGH** | Known secret vector — vendor token, database password, private key |
| **MEDIUM** | Generic credential or high-entropy value in semantic context |
| **LOW** | Low-confidence match — possible credential, needs manual review |
| **INFO** | Metadata finding — informational, not directly exploitable |

### Confidence Levels

| Level | Meaning |
|-------|---------|
| **CERTAIN** | Strong structural match — high-entropy or vendor-anchored token |
| **FIRM** | Semantic key name + reasonable entropy value |
| **TENTATIVE** | Context-based match — lower confidence, worth manual review |

### Noise Suppression

SecretSifter automatically suppresses:
- Known placeholder values (`password`, `secret`, `test`, `example`, `REDACTED`)
- JWT tokens (they appear in every authenticated request)
- Blockchain/crypto hashes (SHA-256, Keccak)
- Template expressions (`${var}`, `{{ x }}`, `$event`)
- CSS class names without digits
- OID dotted decimals (certificate fields)
- Device fingerprinting endpoints (`/fp/*` with `org_id=`)

---

## 11. HTML Reports

HTML reports are the recommended export format for sharing findings.

### Features
- **Summary bar** at the top — CRITICAL / HIGH / MEDIUM / LOW / INFO counts
- **Color-coded severity and confidence badges** for at-a-glance triage
- **Sortable columns** — click any header to sort
- **Filter buttons** — filter by severity with one click
- **Search box** — search across all fields client-side
- **Domain dropdown** — filter to a single target
- **Masked values by default** — click the eye icon to reveal
- **Remediation guidance** — click the info icon on any finding for actionable fix steps
- **Export from report** — download findings as CSV or JSON directly from the HTML file
- **Fully self-contained** — no internet connection required to view

### Per-Domain Reports
The **Per-Domain HTML (ZIP)** export creates one separate HTML file per target domain. Useful when you need to send individual reports to separate teams or clients.

---

## 12. Tips & Best Practices

### For Faster Scans
- Use **Simple mode** for initial reconnaissance
- Set scan tier to **FAST** for a quick overview
- Reduce **Max pages** in browser mode to 25–50 for large applications

### For Thorough Scans
- Use **Browser mode** with scan tier **FULL**
- Set Max pages to 200+ for large applications
- Enable **Scan Request Headers**
- Import a HAR file recorded during a manual walkthrough of the application

### Reducing False Positives
- Increase **Entropy threshold** to 4.0
- Add irrelevant CDN hosts to the **CDN blocklist**
- Add noisy key name prefixes to the **Key Name blocklist**

### Ensuring You Catch Everything
- Add critical key name patterns to the **Key Name allowlist**
- Lower **Entropy threshold** to 3.0
- Use **FULL** tier

### Reporting
- Export as **HTML Report** for client-ready reports
- Use **Mask Values + Mask URLs** when sharing screenshots
- Use **Per-Domain ZIP** when delivering findings per team/owner

---

## 13. Troubleshooting

### "Browser not found" error
**Cause:** Microsoft Edge is not installed and Chromium has not been downloaded.
**Fix:** Go to **Settings → Browser Mode → Install Chromium** and wait for the ~120 MB download to complete.

### Browser mode scan stops immediately on first use (TargetClosedError)
**Cause:** On some Windows machines, Microsoft Edge launches but immediately closes when Playwright first attempts to use it in headless mode. This is a one-time initialisation issue caused by Edge's security policies conflicting with headless automation on first run.
**Fix:** Click **Start Scan** again — SecretSifter will automatically fall back to Chromium and the scan will proceed normally. This issue does not repeat after the first attempt.

### URL shows ✖ (red) in Target Status
**Cause:** Network error, DNS failure, SSL error, or timeout.
**Fix:** Check the **Logs tab** for the exact error. Verify the URL is accessible from your machine.

### URL shows ● (amber) in Target Status
**Cause:** The server returned HTTP 401 or 403 (authentication required).
**Fix:** Use Browser mode and log in manually, then import the session as a HAR file.

### No results for a known single-page app
**Cause:** The app requires JavaScript execution to load content.
**Fix:** Switch to **Browser mode** — it renders the full page and captures dynamic requests.

### Too many false positives
**Fix:**
1. Increase Entropy threshold (Settings → 4.0 or higher)
2. Add noisy CDN hosts to the CDN blocklist
3. Add noisy key prefixes to the Key Name blocklist
4. Switch from FULL to LIGHT scan tier

### Scan is very slow
**Fix:**
1. Reduce Max pages per target (Settings → Browser Mode → 25–50)
2. Use Simple mode instead of Browser mode
3. Reduce the thread count if your machine is under heavy load

### Settings not saving
**Fix:** Close and reopen the app. Settings are saved when the window closes. If the problem persists, check that the app has write access to the Windows Registry (`HKCU\Software\JavaSoft\Prefs`).

### "Windows protected your PC" warning on install
**Cause:** The installer is not signed by a certificate authority trusted by SmartScreen.
**Fix:** Click **More info → Run anyway**. This is expected for software not yet registered with Microsoft's SmartScreen reputation service.

---

## 14. License

Use of SecretSifter is governed by the End-User License Agreement in `LICENSE.txt`.

Copyright (c) 2026 Hemanth Gorijala. All rights reserved.
