# SecretSifter — User Guide

**Version 1.1.0**
Developed by Hemanth Gorijala · gorijala2k16@gmail.com

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [System Requirements](#2-system-requirements)
3. [Installation](#3-installation)
   - 3.1 [macOS — DMG Installer](#31-macos--dmg-installer)
   - 3.2 [Run the JAR Directly](#32-run-the-jar-directly)
   - 3.3 [First Launch — Gatekeeper Warning](#33-first-launch--gatekeeper-warning)
4. [Interface Overview](#4-interface-overview)
5. [Scan Tab](#5-scan-tab)
   - 5.1 [Entering Target URLs](#51-entering-target-urls)
   - 5.2 [Importing a URL List from File](#52-importing-a-url-list-from-file)
   - 5.3 [Scanning a HAR File](#53-scanning-a-har-file)
   - 5.4 [Threads (Concurrency)](#54-threads-concurrency)
   - 5.5 [Follow Script Sources](#55-follow-script-sources)
   - 5.6 [Starting and Stopping a Scan](#56-starting-and-stopping-a-scan)
6. [Understanding Results](#6-understanding-results)
   - 6.1 [Results Table Columns](#61-results-table-columns)
   - 6.2 [Severity Filter](#62-severity-filter)
   - 6.3 [Severity Levels](#63-severity-levels)
   - 6.4 [Confidence Levels](#64-confidence-levels)
   - 6.5 [Editing Severity and Confidence](#65-editing-severity-and-confidence)
   - 6.6 [Deleting a Finding](#66-deleting-a-finding)
   - 6.7 [Target Status Table](#67-target-status-table)
7. [Exporting Results](#7-exporting-results)
   - 7.1 [Export HTML Report](#71-export-html-report)
   - 7.2 [Per-Domain Reports ZIP](#72-per-domain-reports-zip)
   - 7.3 [Export CSV](#73-export-csv)
   - 7.4 [HTML Report Features](#74-html-report-features)
8. [Browser Mode — Playwright / Chromium](#8-browser-mode--playwright--chromium)
   - 8.1 [When to Use Browser Mode](#81-when-to-use-browser-mode)
   - 8.2 [Installing Chromium](#82-installing-chromium)
   - 8.3 [Browser Mode Settings](#83-browser-mode-settings)
9. [Settings Reference](#9-settings-reference)
   - 9.1 [Scanner Controls](#91-scanner-controls)
   - 9.2 [CDN / Third-Party Blocklist](#92-cdn--third-party-blocklist)
   - 9.3 [Key Name Blocklist](#93-key-name-blocklist)
   - 9.4 [Key Name Allowlist](#94-key-name-allowlist)
   - 9.5 [Saving and Resetting Settings](#95-saving-and-resetting-settings)
10. [Detection Engine](#10-detection-engine)
    - 10.1 [Scan Tiers](#101-scan-tiers)
    - 10.2 [Shannon Entropy Analysis](#102-shannon-entropy-analysis)
    - 10.3 [Noise Suppression](#103-noise-suppression)
    - 10.4 [Vendor Coverage](#104-vendor-coverage)
11. [Logs Tab](#11-logs-tab)
12. [Privacy and Security](#12-privacy-and-security)
13. [Troubleshooting](#13-troubleshooting)
14. [License](#14-license)

---

## 1. Introduction

SecretSifter is a standalone desktop application for scanning live web targets — URLs, JavaScript files, API responses, and HAR archives — for exposed credentials, API keys, tokens, and other sensitive secrets.

It is designed for security professionals and penetration testers who need to identify secret leakage in running web applications, without sending any data to third-party services.

**Key capabilities at a glance:**

- Bulk URL scanning with configurable concurrency
- JavaScript source analysis (inline scripts and external files)
- HAR archive import (Burp Suite, browser DevTools)
- Optional headless browser crawling via Playwright/Chromium for SPA/JS-heavy targets
- 160+ detection patterns across CRITICAL, HIGH, MEDIUM, LOW, and INFORMATION severities
- Shannon entropy analysis to catch unknown high-entropy secrets
- PII detection — SSNs, credit card numbers, email addresses, phone numbers
- Request header scanning (Authorization, X-Api-Key, Cookie, etc.)
- Fully offline — no secrets or scan data ever leave your machine

---

## 2. System Requirements

| Requirement | Minimum |
|---|---|
| Operating System | macOS 11+, Windows 10+, Linux (x86-64 or ARM) |
| Java | 17 or later (bundled in the macOS .app) |
| Disk space (app) | ~165 MB |
| Disk space (Browser Mode) | +~120 MB for Chromium (one-time download) |
| RAM | 512 MB recommended |
| Network | Required for URL scanning; not required for HAR-only use |

> **Note:** The macOS `.app` bundle includes a bundled Java runtime — no separate JDK installation is needed on macOS.

---

## 3. Installation

### 3.1 macOS — DMG Installer

1. Open `SecretSifter-1.1.0.dmg`.
2. In the installer window, drag **SecretSifter.app** onto the **Applications** folder.
3. Launch SecretSifter from your Applications folder or Spotlight.

### 3.2 Run the JAR Directly

If you have Java 17+ installed, you can run the JAR on any platform:

```bash
java -jar secretsifter-1.1.0.jar
```

### 3.3 First Launch — Gatekeeper Warning

Because SecretSifter is distributed outside the Mac App Store, macOS may show:

> *"SecretSifter cannot be opened because the developer cannot be verified."*

**To open it the first time:**

1. In Finder, locate **SecretSifter.app** (in Applications or the DMG).
2. **Right-click** (or Control-click) on SecretSifter.app.
3. Select **Open** from the context menu.
4. Click **Open** in the confirmation dialog.

You only need to do this once. After the first launch, SecretSifter opens normally by double-clicking.

---

## 4. Interface Overview

SecretSifter has three main tabs:

| Tab | Purpose |
|---|---|
| **Scan** | Enter targets, run scans, view findings, export results |
| **Settings** | Configure scan behaviour, tiers, entropy, blocklists |
| **Logs** | Live output — fetch status, match counts, errors |

A small **ⓘ** button in the bottom bar opens the **About** dialog with version and contact information.

---

## 5. Scan Tab

### 5.1 Entering Target URLs

Paste one URL per line into the **Target URLs** text area on the left side of the Scan tab. URLs should include the scheme:

```
https://example.com
https://api.example.com/v2/config
https://cdn.example.com/assets/app.js
```

SecretSifter will fetch each URL, extract inline JavaScript blocks and external script references, and scan all content for secrets.

### 5.2 Importing a URL List from File

Click **Import File…** to load a plain-text file containing one URL per line. The file contents are appended to the current URL list.

Accepted file types: `.txt`, `.csv` (first column used), any plain text file.

### 5.3 Scanning a HAR File

Click **Scan HAR…** to import a HAR (HTTP Archive) file. SecretSifter scans all response bodies recorded in the HAR for secrets — including responses that are not normally visible in the browser.

**How to export a HAR file:**

- **Burp Suite:** *Proxy → HTTP History → right-click → Save items* (use Burp's HAR export or the Logger++ extension)
- **Chrome / Edge:** DevTools → Network tab → right-click any request → *Save all as HAR with content*
- **Firefox:** DevTools → Network tab → gear icon → *Save All As HAR*

HAR scanning does not require a network connection.

### 5.4 Threads (Concurrency)

The **Threads** spinner controls how many URLs are fetched in parallel. Default: **25**. Range: 1–50.

- Increase for faster scans on targets with many URLs and low latency.
- Decrease if the target rate-limits requests or if you observe connection errors in the Logs tab.

### 5.5 Follow Script Sources

When **Follow script src** is enabled (checkbox next to the Threads spinner), SecretSifter fetches all external JavaScript files referenced in `<script src="...">` tags found in the HTML response of each target.

This is recommended for sites that load most of their code via external bundles. Disable it to scan only the initial HTML response body.

### 5.6 Starting and Stopping a Scan

- Click **Start Scan** to begin. The button is disabled during scanning.
- Click **Stop** to cancel an in-progress scan. Findings collected so far remain in the results table.
- The **Scanned / Failed / Auth** counters in the toolbar update in real time.

---

## 6. Understanding Results

### 6.1 Results Table Columns

| Column | Description |
|---|---|
| **#** | Row number |
| **Severity** | CRITICAL / HIGH / MEDIUM / LOW / INFORMATION — editable via inline dropdown |
| **Confidence** | CERTAIN / FIRM / TENTATIVE — editable via inline dropdown |
| **Rule ID** | Internal identifier of the matched detection rule (e.g. `AWS_ACCESS_KEY`) |
| **Key** | The matched variable or key name (e.g. `api_key`, `Authorization`) |
| **Value** | The secret value, partially masked by default |
| **URL** | Source URL where the finding was discovered |
| **Line** | Line number within the source file |
| **Context** | Surrounding code snippet (up to 120 characters) |
| **×** | Delete button — removes the row from the table |

Click any column header to sort the table by that column. Severity sorts by rank (CRITICAL → HIGH → MEDIUM → LOW → INFORMATION), not alphabetically. Click again to reverse the order.

### 6.2 Severity Filter

A **Show:** dropdown above the results table lets you filter findings by minimum severity. This is especially useful when scanning many targets where INFORMATION and LOW findings create noise.

| Filter | Shows |
|---|---|
| **All Severities** | Every finding |
| **Critical+** | CRITICAL only |
| **High+** | CRITICAL and HIGH |
| **Medium+** | CRITICAL, HIGH, and MEDIUM |
| **Low+** | Everything except INFORMATION |

The filter applies instantly to the current results without discarding any data. Switching back to **All Severities** restores all findings.

### 6.3 Severity Levels

| Severity | Meaning | Examples |
|---|---|---|
| **CRITICAL** | Credential types that give direct system access | Plaintext passwords, PEM private keys, RSA/EC keys, master passwords, encryption keys |
| **HIGH** | Known vendor secret format matched by an anchored pattern | AWS access key, GitHub personal access token, Stripe secret key, Twilio auth token |
| **MEDIUM** | Generic high-entropy value in a secret-looking key name | `secret`, `api_token`, `private_key` with a random-looking value |
| **LOW** | Lower-confidence match — review manually | Short tokens, partial matches, ambiguous key names |
| **INFORMATION** | Low-risk or contextual findings | Public keys, informational headers, low-entropy heuristic matches |

### 6.4 Confidence Levels

| Confidence | Meaning |
|---|---|
| **CERTAIN** | Matched an anchored vendor-specific pattern AND value passes entropy check |
| **FIRM** | Matched a pattern with good structural confidence but without full entropy gating |
| **TENTATIVE** | Heuristic match — lower confidence, higher false-positive risk |

### 6.5 Editing Severity and Confidence

Click any cell in the **Severity** or **Confidence** column to open a dropdown and override the value. This is useful for triaging findings during a pentest engagement. The severity and confidence badge counts in the toolbar update automatically.

### 6.6 Deleting a Finding

Click the **×** button in the last column of any row to remove that finding from the results table. This does not affect exported reports that were generated before deletion.

### 6.7 Target Status Table

The lower-left panel shows per-target scan outcomes:

| Icon | Status | Meaning |
|---|---|---|
| ✅ | OK / Crawled | Target fetched and scanned successfully |
| ❌ | Failed / No resources | Could not fetch — DNS failure, timeout, connection refused |
| 🔒 | Auth (401/403) | Target requires authentication — no content was scanned |

The **Scanned**, **Failed**, and **Auth** counters in the toolbar summarise the totals across all targets.

---

## 7. Exporting Results

### 7.1 Export HTML Report

Click **Export HTML ▾** and select **HTML Report** to save a self-contained HTML file. The report includes:

- Summary cards showing finding counts per severity
- A sortable, filterable table of all findings
- Severity filter buttons, domain/rule dropdowns, and a free-text search bar
- Masked values and URLs by default (unmask with a button click)
- Built-in CSV and JSON export buttons
- No external dependencies — the file works fully offline

### 7.2 Per-Domain Reports ZIP

Click **Export HTML ▾** and select **Per-Domain Reports ZIP** to generate one HTML report per target domain, packaged into a single ZIP file. Each report contains only the findings for that domain.

This is the recommended export format for multi-target engagements where you need to deliver separate artefacts per client asset.

### 7.3 Export CSV

Click **Export CSV** to save all current findings as a CSV file. Columns exported:

`Severity, Confidence, Rule ID, Rule Name, Key, Value, URL, Line, Context`

The CSV includes unmasked values. Handle the output file with appropriate care.

### 7.4 HTML Report Features

The exported HTML report is fully interactive in any modern browser:

| Feature | How to use |
|---|---|
| **Severity filter** | Click All / High / Medium / Low / Info buttons |
| **Domain filter** | Select a domain from the Domain dropdown |
| **Rule filter** | Select a rule ID from the Rule dropdown |
| **Free-text search** | Type in the search box — matches key, value, URL, context |
| **Sort columns** | Click any column header; click again to reverse |
| **Unmask values** | Click the strikethrough-eye Value button |
| **Unmask URLs** | Click the strikethrough-eye URL button |
| **Export CSV from report** | Click the ↓ CSV button inside the report |
| **Export JSON from report** | Click the ↓ JSON button inside the report |

---

## 8. Browser Mode — Playwright / Chromium

### 8.1 When to Use Browser Mode

Standard scanning fetches the initial HTTP response only. Many modern web applications load the bulk of their JavaScript dynamically — via webpack chunks, lazy imports, or API-driven rendering. In these cases, a plain HTTP fetch returns little or no JavaScript to scan.

Enable **Browser Mode** when:

- The target is a Single Page Application (SPA) built with React, Angular, Vue, or similar frameworks
- Standard scanning returns 0 findings on a target you expect to have secrets
- The target uses dynamic JS bundle loading (webpack, Rollup, Vite)

Browser Mode launches a headless Chromium browser, navigates to each seed URL, intercepts all network responses (including dynamically loaded JS chunks), and scans them all.

### 8.2 Installing Chromium

Chromium (~120 MB) must be installed separately before Browser Mode can be used. SecretSifter will prompt you automatically on first use.

**To install manually, run one of the following in a terminal:**

Option 1 — using the SecretSifter JAR:
```bash
java -cp secretsifter-1.1.0.jar com.microsoft.playwright.CLI install chromium
```

Option 2 — using npx (requires Node.js):
```bash
npx playwright install chromium
```

Chromium is cached at:

| OS | Cache location |
|---|---|
| macOS | `~/Library/Caches/ms-playwright/` |
| Windows | `%LOCALAPPDATA%\ms-playwright\` |
| Linux | `~/.cache/ms-playwright/` |

You can also click **Install Instructions…** in the Settings tab for a reminder of these commands.

### 8.3 Browser Mode Settings

| Setting | Description |
|---|---|
| **Enable headless browser** | Turns Browser Mode on/off for Bulk Scan |
| **Max pages** | Maximum HTML pages Chromium will crawl per seed URL (default: 100, max: 2000) |
| **Chromium path** | Override the auto-detected Chromium binary with a custom path. Leave blank for auto-detect. |

---

## 9. Settings Reference

Open the **Settings** tab to configure scanning behaviour. Click **Save Settings** to apply changes. Settings persist across restarts.

### 9.1 Scanner Controls

| Setting | Default | Description |
|---|---|---|
| **Enable scanning** | On | Master on/off switch. When off, no scanning occurs. |
| **Scan Tier** | FULL | Controls which detection checks run. See [Section 10.1](#101-scan-tiers). |
| **Entropy threshold** | 3.5 | Shannon entropy threshold (bits/char) for generic high-entropy checks. Range: 0.0–6.0. See [Section 10.2](#102-shannon-entropy-analysis). |
| **Enable PII detection** | On | Detect SSNs, credit card numbers (Luhn-validated), email addresses, and phone numbers. |
| **Scan request headers** | Off | Scan outbound request headers (e.g. `X-Api-Key`, `Authorization`) for hardcoded vendor tokens. JWT Bearer tokens are automatically excluded. |

### 9.2 CDN / Third-Party Blocklist

Enter one entry per line. Any URL whose host contains a matching substring is skipped entirely during scanning.

Use this to suppress noise from known CDN domains that serve third-party libraries (e.g. `cloudflare.com`, `googleapis.com`, `cdnjs.cloudflare.com`). Partial host matches are used — entering `cloudflare` will match any subdomain of cloudflare.

**Example entries:**
```
cloudflare.com
googleapis.com
bootstrapcdn.com
jquery.com
```

### 9.3 Key Name Blocklist

Enter one entry per line. Any finding whose matched key name contains a blocklisted substring is suppressed from results.

Use this to hide false positives caused by non-secret key names that happen to match patterns. The Key Name Allowlist overrides this list.

**Example entries:**
```
PUBLIC_KEY
STORAGE_KEY_
VERSION_KEY
FONT_
```

### 9.4 Key Name Allowlist

Enter one entry per line. Any finding whose matched key name contains an allowlisted substring is always reported, even if the value fails entropy checks or other gates.

Use this to ensure specific keys are never missed — for example, internal API subscription key names used by your target.

**Example entries:**
```
apimSubscriptionKey
x-internal-token
CLIENT_SECRET
```

### 9.5 Saving and Resetting Settings

- **Save Settings** — persists all current values to disk. The status bar shows the saved time and active configuration.
- **Reset to Defaults** — restores all settings to factory defaults after a confirmation prompt.

---

## 10. Detection Engine

### 10.1 Scan Tiers

Select the tier that balances thoroughness against scan speed.

| Tier | What is checked | Best for |
|---|---|---|
| **FAST** | Anchored vendor API token patterns + URL embedded credentials | Quick reconnaissance, large URL lists |
| **LIGHT** | FAST + database connection strings + private keys (PEM, RSA, EC) | General-purpose scanning |
| **FULL** | LIGHT + generic high-entropy strings + PII + request header secrets + JSON key walk + SSR state inspection | Thorough engagements, JS-heavy applications |

### 10.2 Shannon Entropy Analysis

SecretSifter calculates the Shannon entropy (bits per character) of candidate values. High entropy indicates a random, machine-generated string — characteristic of API keys, tokens, and cryptographic material.

The **Entropy threshold** setting (default 3.5 bits/char) controls the minimum entropy required for generic (non-anchored) matches. Tuning guidance:

| Threshold | Effect |
|---|---|
| **3.0–3.5** | More sensitive — catches lower-entropy secrets, more false positives |
| **3.5** | Default — good balance for most web targets |
| **4.0–4.5** | More precise — reduces false positives on targets with noisy configuration data |
| **5.0+** | Very strict — only catches very high-entropy values (base64 keys, hex secrets) |

Anchored vendor-specific patterns (e.g. AWS access keys) are not affected by the entropy threshold — they are always reported when matched.

### 10.3 Noise Suppression

SecretSifter uses multiple layers to suppress false positives before reporting a finding:

| Layer | What it removes |
|---|---|
| **Placeholder detection** | Values matching patterns like `your_api_key`, `example_token`, `changeme`, `${VAR}`, `<placeholder>`, all-same-character strings |
| **Minimum length** | Generic values shorter than 20 characters; credential values shorter than 10 characters |
| **Probable value check** | Rejects values that look like CSS class names, file paths, common English words, or HTML attributes |
| **Entropy gate** | Generic checks require the value to meet the configured entropy threshold |
| **Cross-rule deduplication** | When the same value is caught by both an anchored pattern and an entropy check, only the anchored finding is kept |
| **JWT exclusion** | JWT Bearer tokens in Authorization headers are excluded from header scanning (they are session tokens, not secrets) |
| **CSRF/XSRF suppression** | `x-csrf-token`, `x-xsrf-token`, and related anti-forgery headers are never reported — they rotate per request and carry no credential value |

### 10.4 Vendor Coverage

SecretSifter includes anchored detection patterns for 40+ services, including:

AWS, GitHub, GitLab, Bitbucket, Google Cloud, Azure, Stripe, Twilio, SendGrid, Mailgun, Slack, Discord, Telegram, Firebase, Shopify, HubSpot, Salesforce, Okta, Datadog, PagerDuty, Heroku, Cloudflare, HashiCorp Vault, OpenAI, Anthropic, Notion, Intercom, Algolia, Mapbox, PayPal, Square, Braintree, Plaid, Zendesk, Freshdesk, Linear, and more.

Generic detection covers:
- PEM and RSA/EC private keys
- Database connection strings (PostgreSQL, MySQL, MongoDB, Redis, MSSQL)
- JWT tokens
- OAuth client secrets
- Bcrypt password hashes
- Age encryption keys
- SSH private keys
- URL-embedded credentials (`https://user:password@host`)

---

## 11. Logs Tab

The Logs tab shows real-time output from the scanner:

- **Fetch status** — HTTP response codes, redirect chains, skipped CDN domains
- **Match counts** — number of findings discovered per URL
- **Browser Mode events** — Playwright page load and interception events
- **Errors** — connection timeouts, TLS failures, HAR parse errors

Log output can be useful for debugging why a target returned 0 findings (e.g. CDN blocklist hit, 401 response, fetch timeout).

---

## 12. Privacy and Security

SecretSifter is designed for use in sensitive environments where data confidentiality is critical.

- **Fully offline detection** — all pattern matching and entropy analysis runs locally on your machine. No scan data, URLs, or matched secrets are sent to any external service.
- **No telemetry** — SecretSifter does not collect usage data, crash reports, or analytics.
- **No key validation** — SecretSifter does not make outbound API calls to verify whether discovered keys are active. This is by design: making such calls would create an unauthorised audit trail on client infrastructure and violate most penetration testing rules of engagement.
- **Obfuscated binary** — the distributed JAR is processed with ProGuard to protect intellectual property and detection logic.
- **Masked output** — the HTML report masks secret values and URLs by default, reducing accidental exposure when sharing reports.

---

## 13. Troubleshooting

### Gatekeeper blocks the app on first launch

Right-click → Open → Open. See [Section 3.3](#33-first-launch--gatekeeper-warning).

### App icon shows as a generic document icon in Finder

Re-register the app with Launch Services, then restart Finder:

```bash
/System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/LaunchServices.framework/Versions/A/Support/lsregister -f /Applications/SecretSifter.app
killall Finder
```

If you installed to a different location, replace `/Applications/SecretSifter.app` with the actual path to the `.app` bundle. Then relaunch SecretSifter.

### Scan returns 0 results on a JS-heavy SPA

Enable **Browser Mode** in the Settings tab. SPAs load most JavaScript dynamically — the plain HTTP fetcher only sees the initial HTML shell.

### High false-positive rate

- Increase the **Entropy threshold** (try 4.0–4.5).
- Add noisy key names to the **Key Name Blocklist** (e.g. `PUBLIC_KEY`, `VERSION_`).
- Add noisy CDN domains to the **CDN Blocklist**.
- Switch to the **LIGHT** or **FAST** scan tier to disable generic entropy checks.

### Chromium fails to install or launch

Ensure Java is on your `PATH` and the JAR is on a writable path. Run the install command manually:

```bash
java -cp secretsifter-1.1.0.jar com.microsoft.playwright.CLI install chromium
```

If Chromium is already installed but not detected, set the full path to the Chromium executable in the **Chromium path** field in Settings.

### Target returns 401 / 403

SecretSifter does not handle authentication. To scan authenticated targets:

1. Use a browser to authenticate and capture the session as a HAR file.
2. Import the HAR using **Scan HAR…** — SecretSifter will scan all response bodies recorded in the session.

### Results table is empty after scan

Check the Logs tab for errors. Common causes: all targets returned non-200 responses, all targets matched the CDN blocklist, or the scan tier is set to FAST and the target only contains generic (non-vendor) secrets.

---

## 14. License

Use of SecretSifter is governed by the End-User License Agreement in `LICENSE.txt`.

Copyright (c) 2026 Hemanth Gorijala. All rights reserved.

Contact: gorijala2k16@gmail.com
