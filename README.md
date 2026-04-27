# SecretSifter Desktop v1.7.7

First release since v1.1.0. Six months of focused work, with the detection engine, capture pipeline, and automation surface all rebuilt around real-world Burp comparison data.

## Highlights since v1.1.0

- **Burp-comparable detection on Angular / webpack bundles.** Dedup is now line-aware and the GENERIC_KV scanner runs on the full text of multi-MB JS bundles, so the same key+value at lines 14, 15, 33, … 169754 each surfaces as a separate finding (matching Burp's per-line reporting instead of collapsing them all into one row).
- **Akamai / Cloudflare bypass.** The crawler now launches real Google Chrome via `setChannel("chrome")` with `--disable-blink-features=AutomationControlled`. Falls back to bundled Chromium / system Edge if Chrome is absent. No user setup required.
- **Burp-style Details / Request / Response side panel** with live capture store backing it. Click any finding to see the full HTTP transaction.
- **Custom rules with raw mode.** Ship a curated org-specific regex pack with a "Use custom rules ONLY" toggle that bypasses every default noise filter — every match is reported, false positives included, exactly as the user's regex defines.
- **Bearer-authenticated REST + MCP server** for automation. Full route table (`/api/scan`, `/api/findings`, `/api/report`, `/api/settings`, `/api/rules`, `/api/logs`) plus JSON-RPC at `/mcp` for Claude Desktop / Claude Code integration.
- **Single-click report bundles.** All-in-One ZIP (HTML + CSV + JSON + suppressed CSV) and Per-Domain ZIP, every artifact timestamped `yyyyMMdd_HHmmss`. CSV column order matches the HTML report.
- **8× smaller installer.** v1.1.0 shipped as 1.83 GB; v1.7.7 is 238 MB. Stale per-version JARs in `build/libs/` were silently being bundled into the installer — that's fixed.

## Per-version log

### v1.7.7
- Project root is now self-contained. Icon and supporting assets moved inside the project directory; `build.gradle` no longer references the parent folder.

### v1.7.6
- Mask Value / Mask URL fixed — uses model-column index, so masking works regardless of column reordering. Mask URL covers both URL and Target columns.
- Start Scan auto-clears prior findings, dedup state, and capture store.
- Copy buttons in the Details panel changed from "Copy" text to compact 📋 icon.
- Center divider has hard min-widths so dragging can't hide either pane.
- New "AI Provider API Key" field in Settings → AI / MCP — masked by default, 👁 reveal toggle, persisted to user prefs.

### v1.7.5
- Custom rules ONLY mode now runs in **raw mode** — bypasses placeholder, entropy, hex-too-short, SRI, and keyName blocklist filters. Every regex match is reported, false positives included.

### v1.7.4
- Details tab: outer scroll-pane allows horizontal scroll on overflow.
- MCP port spinner: comma stripped (matches the proxy port fix).
- "Use ONLY" mode now force-enables custom rules so it never silently runs zero rules.
- "Enable custom rules" tooltip clarifies that custom rules run *in addition to* built-in detection.
- Proxy now logs every intercepted request (`[Proxy] METHOD url`) to the Logs tab.
- New menu: Export Data → Export Suppressed (NOISE) CSV.
- Both HTML export options produce a single timestamped ZIP bundling HTML + CSV + JSON + suppressed CSV.
- Every export filename carries `yyyyMMdd_HHmmss`.
- CSV column order matches the HTML report.

### v1.7.3
- New "Use custom rules ONLY" toggle in Settings → Custom Rules.

### v1.7.2
- Crawler launches real Chrome via `setChannel("chrome")` + `--disable-blink-features=AutomationControlled`. Falls back silently to bundled Chromium if Chrome is absent.
- Port spinner displays as `8888` — no comma, copy/pasteable into browser proxy fields.
- Proxy moved into Settings as a 4th sub-tab beside AI / MCP.

### v1.7.1
- Two-pass dedup is now line-aware: same key+value at distinct lines are preserved as separate findings (matches Burp's per-line behaviour on multi-MB Angular bundles).

### v1.7.0
- Findings details side panel with Details / Request / Response tabs.
- `CaptureStore` for HTTP request/response data backing the Details panel.
- UI refinements: column filter location, mask button placement, clear-results behaviour, font sizing, search bars in Request / Response panes, severity / confidence sync, NOISE marker, copy buttons.
- Detection improvements: `vapidKey` prefix, placeholder filter, dedup Pass 2 fix, subscription / resource scoring, JWT severity by content type.

### v1.2 — v1.6
- Initial Windows desktop port from the Burp extension.
- Bouncy Castle MITM proxy with per-host certificate generation.
- Playwright crawler integration.
- HAR import support.
- Initial MCP / REST server.
- jpackage Windows EXE installer.

## System requirements

- Windows 10 / 11 (x64)
- ~500 MB disk space (installer + bundled JRE + Chromium fallback)
- No admin rights required (per-user install)
- Java runtime is bundled — no external JRE needed
- Google Chrome recommended for best WAF-bypass coverage; the crawler falls back to bundled Chromium or system Edge if Chrome is absent

## Installation

1. Download `SecretSifter-1.7.7.exe`.
2. Double-click. Windows SmartScreen may warn — click *More info → Run anyway*.
3. Default install path: `%LOCALAPPDATA%\SecretSifter`. Start Menu shortcut and Desktop icon are created.
4. First launch generates a random MCP token; visible in Settings → AI / MCP.

## Known limitations

- Proxy is HTTP/1.1 only; HTTP/2 sites may not capture cleanly.
- macOS / Linux not supported in this build (Burp extension edition available for cross-platform).
- HSTS-preloaded sites refuse the proxy's per-host certs even after manual CA import — use bulk scan or HAR import for those targets.

## Documentation

Two companion documents are available:

- `SecretSifter_Desktop_Engineering_Reference_v1.7.6.html` — pipeline internals, every rule family, dedup architecture, threading, REST surface, hardcoded thresholds, full version history.
- `SecretSifter_Desktop_User_Documentation_v1.7.6.html` — installation guide, every workflow, severity model, triage guide, settings reference, REST API quickstart with PowerShell + Python snippets, troubleshooting.

---

Author: Hemanth Gorijala · MIT License
