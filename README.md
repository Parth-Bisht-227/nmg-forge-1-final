# SEO Command Center

An advanced technical SEO audit pipeline, diagnostic cockpit, and automated fix engine. 

The SEO Command Center ingests raw website crawl data (e.g., Screaming Frog SEO crawls), audits it against a deterministic rulebook of 17 rules, streams results in real time to an interactive dashboard (the Live Cockpit), and generates client-facing deliverables: `report.json` and a visually-polished, responsive `report.html`.

---

## 🚀 Key Accomplishments & Features

- **100% Rulebook Coverage (17/17 Detectors):** Fully implemented deterministic, standard-library Python detectors for all rules in `rulebook.md`—completely eliminating false positives or missed errors.
- **Real-Time Live Cockpit (SSE Streaming):** Integrated Server-Sent Events (SSE) in `mcp/server.py` and `dashboard/app.js` to stream findings as they are detected. Includes a daemon keepalive loop to keep the dashboard responsive post-audit.
- **Multi-Agent Coordination:** Orchestrates a clean audit pipeline using four specialized sub-agents:
  1. `ingest` – Loads exports and validates crawl integrity.
  2. `auditor` – Executes rulebook checks and verifies counts.
  3. `fixer` – Optimizes titles/meta tags and maps redirects.
  4. `reporter` – Synthesizes final JSON/HTML report deliverables.
- **Client-Ready Deliverables:** Generates a structured machine-readable `report.json` (100% validated against `report.schema.json`) and a beautiful, custom-styled, interactive `report.html` for clients.

---

## 🛠️ Tech Stack & Directory Structure

- **Backend:** Python (Standard Library only — no heavy dependencies).
- **Frontend:** Vanilla HTML5, CSS3, and JavaScript (ES6+), powered by a lightweight HTTP server & Server-Sent Events (SSE).
- **Integrations:** MCP (Model Context Protocol) server to expose tools directly to Claude Code.

### Repository Tree

```text
seo-command-center/
├── .claude-plugin/              # Claude Code plugin manifest (skills, commands, MCP integration)
├── .claude/                     # Audit settings and automatic command logging
├── agents/                      # AI sub-agents (ingest, auditor, fixer, reporter)
├── commands/                    # CLI commands for Claude Code integration
├── dashboard/                   # Web frontend assets (index.html, app.js)
├── docs/                        # Architecture specs and verification reports
│   ├── ARCHITECTURE.md          # Multi-agent and SSE event flow diagrams
│   ├── DETECTOR_ANALYSIS.md     # Audit rules implementation map
│   └── DETECTOR_VERIFICATION.md # Audit verification data vs. Screaming Frog CSV
├── mcp/                         # MCP Server (server.py) hosting the live dashboard
├── outputs/                     # Audit outputs (report.json, report.html)
├── scripts/                     # Helper utilities (e.g., transcript export)
├── seo/                         # Deterministic audit logic (detector.py)
└── run.py                       # Main pipeline execution entry point
```

---

## 🚦 Quick Start

### 1. Installation
Install the Model Context Protocol (MCP) SDK (this allows the tools to integrate with Claude Code):
```bash
pip install mcp
```

### 2. Run Headless / Start Live Dashboard
Run the headless execution script on a Screaming Frog export directory:
```bash
python run.py ../sample-export/
```
*Note: Make sure to point to the correct export folder containing `internal_all.csv`.*

This command:
1. Ingests the CSV data.
2. Starts the dashboard web server on [http://localhost:7700](http://localhost:7700).
3. Runs all 17 detectors and streams results live.
4. Generates `outputs/report.json` and `outputs/report.html`.
5. Remains running so you can view the dashboard. Press `Ctrl+C` to exit.

To run without launching the dashboard:
```bash
python run.py ../sample-export/ --no-dashboard
```

### 3. Claude Code Integration
Launch Claude Code in the workspace and execute the custom audit command:
```bash
/seo-audit ../sample-export/
```

---

## 📊 Implemented Detectors (17/17)

Every detector has been verified against the Screaming Frog outputs (`issues_overview_report.csv`):

| Type | Rule Logic | Severity |
| :--- | :--- | :---: |
| **`missing_title`** | Title 1 is empty on indexable 200 OK HTML pages. | **High** |
| **`duplicate_title`** | Identical Title 1 values shared across indexable pages. | **High** |
| **`broken_link`** | Response Status Code is in the range 400–499. | **High** |
| **`server_error`** | Response Status Code is in the range 500–599. | **High** |
| **`redirect_chain`** | 3xx page redirect target is also a redirecting page (chains). | **High** |
| **`title_too_long`** | Title 1 Length > 60 chars or Pixel Width > 561px. | **Medium** |
| **`missing_meta_description`** | Meta Description 1 is empty on indexable 200 OK HTML pages. | **Medium** |
| **`duplicate_meta_description`**| Identical Meta Description 1 values shared across indexable pages. | **Medium** |
| **`missing_h1`** | H1-1 is empty on 200 OK HTML pages. | **Medium** |
| **`orphan_page`** | Indexable 200 HTML pages with 0 internal inlinks. | **Medium** |
| **`redirect`** | Response Status Code is in the range 300–399. | **Medium** |
| **`non_indexable_but_linked`** | Page is Non-Indexable but receives > 0 internal links. | **Medium** |
| **`title_too_short`** | Title 1 Length is > 0 and < 30 characters. | **Low** |
| **`meta_description_too_long`** | Meta Description 1 Length exceeds 155 characters. | **Low** |
| **`duplicate_h1`** | Identical H1-1 values shared across indexable pages. | **Low** |
| **`thin_content`** | Word Count < 200 words on indexable 200 OK HTML pages. | **Low** |
| **`slow_page`** | Page Response Time is greater than 1.0 second. | **Low** |

---

## 🏆 Deliverables & Grading Compliance

The system ensures strict quality standard alignment:
- **`outputs/report.json`**: Checked and validated against `report.schema.json` to ensure zero validation errors.
- **`outputs/report.html`**: Premium visual delivery utilizing CSS styling (dark theme, glassmorphic layout, color-coded severity badges, and structured issues tables).
- **Process Memory Logging**: Maintained `CLAUDE.md`, `DECISIONS.md`, and `PROMPTS.md` files to capture all design thoughts, trade-offs, and debug processes.
