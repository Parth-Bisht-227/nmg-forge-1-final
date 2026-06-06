# Architecture: SEO Command Center

## Overview
The SEO Command Center is a system designed to ingest Screaming Frog SEO exports, detect technical SEO issues based on a deterministic rulebook, prioritize those issues using AI agents, generate content fixes (titles/metas), and provide a real-time visualization dashboard and a final deliverable report.

## Repository Tree
```text
.
├── report.schema.json          # Schema for the final report.json
├── rulebook.md                 # The SEO rules the detectors implement
└── seo-command-center/
    ├── run.py                  # Headless runner for the full pipeline
    ├── agents/                  # AI Agent definitions
    │   ├── auditor.md           # Prioritizes and analyzes issues
    │   ├── fixer.md             # Generates content fixes
    │   ├── ingest.md            # Handles data loading and cleaning
    │   └── reporter.md         # Synthesizes the final report
    ├── commands/               # Defined commands for Claude Code
    │   └── seo-audit.md
    ├── dashboard/               # Live dashboard frontend
    │   ├── app.js               # Frontend logic & SSE handling
    │   └── index.html            # Dashboard UI
    ├── mcp/                     # MCP Server & Pipeline Coordination
    │   └── server.py            # Core state management & tool exposure
    ├── outputs/                 # Generated audit results
    │   ├── report.json          # Structured audit data
    │   └── report.html          # Client-facing HTML report
    ├── seo/                     # Deterministic detection logic
    │   └── detector.py           # Rulebook implementation (CSV processing)
    └── skills/                  # Claude Code skill definitions
        └── seo-audit/
            └── SKILL.md
```

## Execution Flow (`run.py`)
`run.py` acts as a headless orchestrator for the full pipeline. It executes the following sequence via the `server` module:

1.  **`start_dashboard()`**: (Optional) Launches the HTTP/SSE server for real-time monitoring.
2.  **`seo_load(export_dir)`**: Ingests the `internal_all.csv` from the Screaming Frog export.
3.  **`seo_detect()`**: Runs deterministic detectors against the loaded data.
4.  **`seo_recommend(recs)`**: Attaches starter recommendations (prioritizes top 5 issues).
5.  **`seo_report()`**: Generates and saves `outputs/report.json`.
6.  **`seo_export()`**: Renders and saves `outputs/report.html`.

## Agent Responsibilities
The system uses a multi-agent architecture to move from raw detection to actionable fixes:

| Agent | Responsibility | Primary Action |
| :--- | :--- | :--- |
| **Ingest** | Data loading and cleaning | Calls `seo_load` |
| **Auditor** | Prioritization and impact analysis | Analyzes results of `seo_detect` |
| **Fixer** | Generates model-driven fixes (Titles, Metas) | Calls `seo_set_fixes` |
| **Reporter** | Final synthesis and report formatting | Calls `seo_report` & `seo_export` |

## MCP Server Responsibilities (`mcp/server.py`)
The MCP server serves as the "Brain" of the system, providing both a tool-based API for agents and a real-time API for the dashboard.

- **State Management**: Maintains a global `RUN` dictionary containing the current site, URLs, detected issues, fixes, and metadata.
- **Tool Exposure**: Exposes the following tools to Claude Code:
    - `load`: Load export directory.
    - `detect_issues`: Run rulebook detectors.
    - `set_fixes`: Attach AI-generated title/redirect fixes.
    - `recommend`: Attach prioritized recommendations.
    - `write_report`: Save JSON output.
    - `export_report`: Save HTML output.
- **Dashboard Hosting**: Serves a static frontend and an SSE event stream.
- **Event Emission**: Pushes state changes to the dashboard via an internal `_emit` mechanism.

## Dashboard Event Flow
The dashboard uses Server-Sent Events (SSE) to update in real-time without polling:

1.  **Connection**: Frontend connects to `/events`.
2.  **Snapshot**: Server immediately sends a `snapshot` event with current `RUN` state.
3.  **Events**: As the pipeline progresses, the server emits:
    - `loaded`: Site and URL count.
    - `issue`: Individual detected issue (allows the UI to "stream" issues).
    - `summary`: Final counts by severity.
    - `fixes`: Updated titles/redirects.
    - `recommendations`: The priority list.
    - `saved`/`exported`: Notification that files are ready.

## Report Generation Flow
Reports are generated in two formats:

1.  **`report.json`**:
    - The `_report_obj()` function aggregates state from the `RUN` dictionary.
    - The resulting object is validated against `report.schema.json` and written to disk.
2.  **`report.html`**:
    - The report object is passed to `_render_html`.
    - A standalone HTML page is generated with embedded CSS for a professional client deliverable.

## Detector Flow (`seo/detector.py`)
The detection process is deterministic and uses only standard Python (CSV) to ensure speed and reliability:

1.  **`load_rows()`**: Reads `internal_all.csv` into a list of dictionaries.
2.  **`detect()`**: 
    - Filters rows into subsets: `html` (Content-Type check) and `idx200` (Status 200 + Indexable).
    - Applies specific rules (e.g., Missing Title, Duplicate Title, 4xx/5xx Errors).
    - Collects affected URLs and counts per issue type.
3.  **`summarize()`**: Aggregates the issues into a total count and a breakdown by severity (High, Medium, Low).

## TODO Locations
- **`seo-command-center/run.py`**: Line 12-13 — Implementation of model-driven fixes (title rewriting, redirect maps).
- **`seo-command-center/seo/detector.py`**: Line 93 — Implementation of the remaining rulebook detectors (too short titles, missing metas, etc.).
