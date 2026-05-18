# Vuln Daily Brief

**Vuln Daily Brief** is a Python-based daily vulnerability intelligence and triage pipeline.  
It collects recent cybersecurity news, identifies potentially important vulnerabilities, enriches them with **CISA KEV** and optionally **VulnCheck KEV**, uses the **OpenAI API** to filter out noise, and generates a clean **HTML + PDF report**.

The goal is simple:

> **Do not report every CVE. Report only the vulnerability news that may actually matter operationally.**

---

## Features

- Collects articles from:
  - The Hacker News
  - BleepingComputer
  - SecurityWeek

- Looks back over a configurable time window, by default:
  - **last 24 hours**

- Extracts:
  - CVE identifiers
  - article text
  - high-risk keywords such as:
    - active exploitation
    - zero-day
    - PoC / exploit
    - RCE
    - auth bypass
    - privilege escalation
    - ransomware

- Enriches findings with:
  - **CISA Known Exploited Vulnerabilities Catalog**
  - **VulnCheck KEV** — optional, requires API token

- Uses the **OpenAI API** to:
  - classify items as `CRITICAL`, `HIGH`, `MEDIUM`, `LOW`, or `DROP`
  - keep only **CRITICAL** and **HIGH** findings in the main report
  - produce concise Polish-language summaries
  - generate:
    - VM recommendations
    - evidence status: PoC / exploitation / KEV
    - threat intelligence context
    - detection / hunting notes for Elastic

- Exports:
  - JSON
  - HTML
  - PDF

---

## Example output

The report contains:

- report metadata and counters,
- the most important vulnerability items from the last 24 hours,
- explanation of why each item matters,
- VM recommendation,
- PoC / exploitation / KEV status,
- threat intelligence context,
- **Detection / Hunting (Elastic)** notes,
- a separate list of lower-priority or rejected items.

Only **CRITICAL** and **HIGH** items appear in the main report.

---

## Project structure

```text
vuln-daily-brief/
├─ src/
│  └─ vuln_report/
│     ├─ ai.py
│     ├─ config.py
│     ├─ enrich.py
│     ├─ main.py
│     ├─ models.py
│     ├─ report.py
│     ├─ sources.py
│     └─ utils.py
│
├─ templates/
│  └─ report.html.j2
│
├─ output/
│  ├─ vuln_daily_report_YYYY-MM-DD_HHMM.json
│  ├─ vuln_daily_report_YYYY-MM-DD_HHMM.html
│  └─ vuln_daily_report_YYYY-MM-DD_HHMM.pdf
│
├─ .env.example
├─ requirements.txt
├─ setup.ps1
├─ run.ps1
├─ build.ps1
└─ README.md
Requirements
Windows
PowerShell
Python 3.11+ recommended
OpenAI API key
Optional: VulnCheck API token

The project has been designed for a PowerShell-oriented workflow.

Installation

Clone the repository:

git clone https://github.com/YOUR_USERNAME/vuln-daily-brief.git
cd .\vuln-daily-brief

Run the setup script:

.\setup.ps1

This should:

create the virtual environment,
install Python dependencies,
prepare the runtime environment.
Configuration

Copy or create the .env file:

Copy-Item .\.env.example .\.env
notepad .\.env

Minimum required configuration:

OPENAI_API_KEY=your_openai_api_key

Optional VulnCheck integration:

VULNCHECK_API_TOKEN=your_vulncheck_api_token

Example full configuration:

OPENAI_API_KEY=your_openai_api_key
OPENAI_MODEL=gpt-5.4-mini

VULNCHECK_API_TOKEN=

LOOKBACK_HOURS=24
MAX_ITEMS_PER_SOURCE=25
MAX_ARTICLE_CHARS=12000
REQUEST_TIMEOUT_SECONDS=25

REPORT_TIMEZONE=Europe/Warsaw
GENERATE_PDF=true
Running the report

Run:

.\run.ps1

Generated files will appear in:

.\output\

Example:

vuln_daily_report_2026-05-19_0715.json
vuln_daily_report_2026-05-19_0715.html
vuln_daily_report_2026-05-19_0715.pdf
How the triage works

The pipeline has two layers.

1. Heuristic scoring

Articles receive points for signals such as:

CVE present
CISA KEV match
VulnCheck KEV match
phrases indicating:
active exploitation
zero-day
exploit / PoC
RCE
authentication bypass
ransomware
presence of high-value enterprise vendors or products
2. AI-based triage

The OpenAI API evaluates the article and enrichment data and returns structured JSON.

The model is instructed to:

use only supplied material,
avoid fabricating facts,
explicitly mark unconfirmed information,
write all report-ready fields in Polish,
retain only:
CRITICAL
HIGH

Medium and low-priority items are automatically excluded from the main report.

Report sections

Each important vulnerability entry may include:

Podsumowanie
Identyfikatory i źródła
Dlaczego to ważne
Rekomendacja VM
Status dowodowy
PoC
Eksploatacja
KEV
Informacje wywiadowcze / kontekst
Detekcja / hunting (Elastic)
Niepewności
Data sources

The project currently uses:

RSS / feed content from:
The Hacker News
BleepingComputer
SecurityWeek
Enrichment from:
CISA KEV JSON feed
VulnCheck KEV backup API, when configured

Article pages are fetched to obtain more complete content for triage.

Notes on detection / hunting output

The report does not attempt to automatically write full production Elastic rules.

Instead, it provides practical analyst-grade guidance such as:

which logs may be worth checking,
which process relationships or behaviors may matter,
which network traces could be relevant,
whether detections should focus on behavior rather than hard IOC matching.

This section is intended as a starting point for detection engineering or threat hunting.

Known limitations
Source websites may change their HTML structure over time.
Some news articles contain many CVEs in one text, which may require future grouping improvements.
VulnCheck enrichment requires a valid token.
AI output quality depends on:
source article quality,
available enrichment,
chosen model.
The report is a prioritization aid, not a substitute for direct verification in vendor advisories, NVD, CISA KEV, or product-specific documentation.
Suggested future improvements

Potential enhancements:

add vendor advisory feeds,
deduplicate multiple articles describing the same vulnerability,
group report entries by CVE instead of by article,
add EPSS enrichment,
add NVD / CVSS enrichment,
add email delivery of the generated PDF,
add scheduled daily execution through Windows Task Scheduler,
add a local HTML dashboard with history,
add explicit “observed exploitation / PoC / KEV” filters.
Git ignore

Recommended .gitignore:

.env
.venv/
__pycache__/
*.pyc
output/
Disclaimer

This project is intended for defensive cybersecurity, vulnerability management, and security research workflows.

Generated reports should be reviewed before operational use or escalation.

License

Choose a license appropriate for your use case, for example:

MIT
Apache-2.0
GPL-3.0
