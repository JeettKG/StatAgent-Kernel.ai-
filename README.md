# StatAgent-Kernel.ai- 

🇦🇹 StatAgent – Sozialversicherung Statistics & QA Engine

StatAgent is an automated, high-performance statistics calculation and quality assurance platform designed for the Dachverband der österreichischen Sozialversicherungsträger. It processes raw employment delivery files (raw_deliveries) and pension reports (raw_pensions), applies deterministic counting rules, performs automated QA anomaly detection, and exposes interactive analytics via a local executive dashboard and CLI.

📄 Key Features & Compliance Highlights

Strict Deterministic Processing: Zero language-model dependency in statistic calculations. Two runs with the same data and rules yield identical numbers (COUNTING_CONDITIONS mapped via SQL over clean_period).

Verifiable Audit Trail: Human-in-the-loop approvals backed by database logs (audit_log) and Git commits tracking authorized names, dates, and target checksums.

Change Request (CR 1–3) Support:

CR 1: Harmonization of regular employment contracts and free service contracts (Arbeiter, Angestellte, Beamte).

CR 2: Transition from snapshot/reference-date counting to full-month coverage (any relationship active $\ge 1$ day in the reporting period).

CR 3: Ministry Special Delivery (Sonderauswertung) filtering individuals with $\ge 200$ days in ÖNACE Section C over a 12-month window.

Zero Fake Data Policy: All dashboard components, classifications (ÖNACE, Region), and quality reports stream live from PostgreSQL.

Safe Read-Only Query Explorer: Ad-hoc SQL execution guarded against destructive operations (DROP, DELETE, UPDATE).

🏗 System Architecture & Repository Structure

├── app_nicegui.py       # Main NiceGUI Executive Web Interface (Port 8080)
├── run_2.py             # CLI Entry point for orchestrating runs, approvals & exports
├── engine_2.py          # Deterministic calculation engine, counting rules & query handler
├── ingest_2.py          # Data ingestion pipeline & rule verification
├── common.py            # Database connection, date math, logging & cryptographic hashing
├── qa/                  # Quality Assurance framework
│   ├── checks.py        # Automated anomaly detection logic
│   └── report.py        # Markdown Quality Report compiler
├── database/            # Database schema initialization scripts (01_schema.sql, 02_laden.sql)
├── lieferungen/         # Input delivery archive directory (.csv.gz or .csv)
├── pensionen/           # Input pension report directory (.xlsx)
├── reports/             # Auto-generated monthly quality reports (.md)
├── docker-compose.yml   # Docker setup for local PostgreSQL database container
└── requirements.txt     # Python dependencies


🛠 Quick Start & Installation

Prerequisites

Python 3.10+

Docker Desktop (or a local PostgreSQL 16 instance)

1. Clone the Repository & Install Dependencies

git clone https://github.com/your-username/statagent.git
cd statagent
pip install -r requirements.txt


(Required packages: nicegui, pandas, psycopg2-binary, sqlalchemy, openpyxl)

2. File Structure Verification

Ensure raw delivery files are placed directly inside lieferungen/ as flat files or .csv.gz archives (do not keep them as unzipped sub-folders):

lieferungen/
  ├── lieferung_2024-08.csv.gz
  ├── lieferung_2024-09.csv.gz
  └── ...


3. Database Initialization

Start the PostgreSQL database container:

docker compose down -v
docker compose up -d


Verify that the container is ready:

docker logs -f statagent-db


Initialize system schemas, classifications, and default rules:

python run_2.py --init


🚀 Usage Guide

Human Approval & Rule Activation

Before running monthly pipelines, authorize the validation rules and rule sets with your full name:

# Approve rules as a verified individual
python run_2.py --approve rules --by "Jeet Kumar Gupta"
python run_2.py --approve ruleset:v1_reference_date --by "Jeet Kumar Gupta"


Running Monthly Production Batches via CLI

Process deliveries sequentially (chronological execution enforced):

# Run single reporting month
python run_2.py --delivery 2025-03

# Run full time-series (2024-08 to 2026-07) with auto-approval
python run_2.py --all --approve-by "Jeet Kumar Gupta"


Generating Aggregation Sheets & Special Deliveries

# Export consolidated aggregation sheet
python run_2.py --sheet v1_reference_date --out output/aggregation_v1.csv

# Generate Ministry Delivery (CR 3)
python run_2.py --ministry 2026-07


🖥 Executive Dashboard (NiceGUI Web UI)

Start the interactive web portal:

python app_nicegui.py


Navigate to http://localhost:8080 in your browser to access:

📊 Monatslauf & Qualitätssicherung: Select delivery periods, run pipelines, and review markdown quality reports.

📖 Klassifikationen & Metadaten: Inspect live ÖNACE systems, regional classifications, and active validation rules.

🔍 Ad-hoc Abfragen & SQL Explorer: Natural-language data inquiries and safe, read-only SQL querying.

📜 Revisionssicherer Audit-Trail: View real-time database-backed log entries and issue formal human approvals.

🔒 Security & Safe Execution

Read-Only SQL Explorer: Restricts queries strictly to SELECT / WITH statements, with timeouts and row caps.

Data Integrity: Duplicate rows are tracked in occurrences without losing record counts. Missing/invalid fields default to official standards (UNB, 0) rather than arbitrary text guesses.

📜 License & Compliance

Designed strictly in accordance with the technical guidelines and delivery criteria for the Austrian Social Insurance (Dachverband der Österreichischen Sozialversicherungen).
