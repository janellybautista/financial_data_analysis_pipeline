# Financial Data Analysis Pipeline

A Python project that ingests financial data from CSV, Excel, and PDF files, cleans and normalizes the records, calculates business KPIs, stores the results in SQLite, and exports reporting-ready outputs for analysis.

## Project Overview

This project turns messy financial statements into a consistent analytical dataset. The workflow is designed for real-world finance data where source files vary by format, company, and reporting period. Instead of manually reviewing each document, the pipeline automates extraction, validation, enrichment, and reporting.

## Clear Architecture Diagram

```text
Raw source files (.csv, .xlsx, .xls, .pdf)
                │
                ▼
        File discovery + metadata extraction
                │
                │  Example: company, year, month, period
                ▼
    Extract values from tables or text using pandas + regex
                │
                ▼
       Standardize column names and data types
                │
                ▼
      Validate financial values and prevent invalid math
                │
                ▼
      Clean dataset with normalized KPI-ready schema
                │
                ▼
               SQLite database
                │
                ├── Store raw/cleaned records
                └── Query summary metrics and rankings
                │
                ▼
      Compute KPIs (profit margin, liquidity, etc.)
                │
                ▼
      Export CSV summaries + charts for business review
                │
                ▼
       Final analysis outputs for Power BI / reporting
```

## How the Pipeline Works

### 1. Ingest raw files
The project scans the input folder for files in multiple formats:

- CSV
- Excel
- PDF

Each file is processed and assigned metadata based on its filename, such as:

- company name
- report year
- report month
- reporting period
- source file name
- source type

This metadata extraction is handled in the filename parsing logic in [financial_pipeline.py](financial_pipeline.py).

### 2. Extract financial values
The code reads each file and extracts the key fields it needs from the data, including:

- service revenue
- sales revenue
- net income
- operating expenses
- total assets
- current assets
- liabilities
- retained earnings

For PDFs, text is extracted with `pdfplumber`. For Excel files, all sheets are read with pandas. For CSVs, the structured table is parsed directly.

### 3. Clean and normalize the data
After extraction, the pipeline standardizes the schema and ensures all rows are aligned to the same columns. This includes:

- lowercasing and normalizing column names
- converting currency strings to numeric values
- filling missing columns with nulls where needed
- ensuring metadata is consistent across files

This step is important because the source files are not guaranteed to use the same formatting or naming conventions.

### 4. Calculate KPIs
Once the rows are standardized, the project computes several financial KPIs:

- Total revenue = service revenue + sales revenue
- Profit margin = net income / total revenue
- Operating expense ratio = operating expenses / total revenue
- Current ratio = current assets / liabilities
- Asset-to-liability ratio = total assets / liabilities

These KPIs are then attached to each record and used for comparisons by company, period, and source.

### 5. Save to the database
The cleaned dataset is written to a SQLite database. This provides a durable storage layer and allows the project to run analytical SQL queries without reprocessing the original files.

### 6. Export reporting outputs
The project writes multiple CSV outputs and chart files to the output directory, including:

- financial_data_with_kpis.csv
- all_records.csv
- sql_calculated_kpis.csv
- highest_profit_margin.csv
- strongest_liquidity.csv
- largest_total_assets.csv
- revenue_by_company.csv
- revenue_by_period.csv
- average_profit_margin_by_company.csv

These outputs are useful for dashboards, BI tools, and stakeholder review.

## Data Validation

The project includes practical quality checks to prevent bad or misleading numbers from being loaded into the analysis layer. In the current implementation, the pipeline validates and normalizes data in several ways:

- money values are cleaned before conversion, removing formatting such as `$` and `,`
- invalid numbers are coerced to nulls instead of crashing the pipeline
- division-based KPI calculations only run when the denominator is present and non-zero
- metadata is filled from the filename when row-level data is missing
- source information is preserved so analysts can trace each record back to the original file

Examples of the validation logic in practice:

- `pd.to_numeric(..., errors="coerce")` prevents malformed values from breaking numeric analysis
- `NULLIF` logic in SQL avoids divide-by-zero errors when liabilities or revenue are zero
- Only records with valid KPI inputs are used in ratio calculations

This means the pipeline is designed to ensure that no malformed financial values are silently treated as valid numbers, and that no invalid KPI is calculated from a zero-value denominator.

## Storage Choice

The project uses SQLite as the local analytical data store.

### Why SQLite was a good choice here

- lightweight and easy to set up for a local analysis project
- portable and file-based, no separate server required
- sufficient for a moderate-sized financial dataset
- easy to query with SQL for summary analysis and rankings
- simple to export to CSV for downstream tools such as Power BI

This is a good fit for a project whose goal is to process a batch of company statements and generate reporting outputs without requiring a production-grade warehouse. For a much larger historical time-series workload, a columnar engine like DuckDB could be even faster for analytical queries, especially for large-scale aggregations and exploratory analysis.

In short:

- SQLite: ideal for a self-contained analytics pipeline and local reporting workflow
- DuckDB: better for very large analytical workloads and heavier query performance

## Project Structure

```text
financial_data_analysis_pipeline/
├── financial_pipeline.py        # main data pipeline
├── analysis_agent.py            # separate AI/finance prototype
├── pyproject.toml              # project configuration
├── README.md                   # project overview and workflow explanation
├── input_files/                # raw financial source files
├── output/                     # generated CSV reports, charts, and database
├── LICENSE
└── .gitignore
```

## KPI Definitions

The project calculates and compares business health using these key financial metrics:

- Total revenue = service revenue + sales revenue
- Profit margin = net income / total revenue
- Operating expense ratio = operating expenses / total revenue
- Current ratio = current assets / liabilities
- Asset-to-liability ratio = total assets / liabilities

## Outputs and Business Use

The generated files allow analysts to answer business questions such as:

- Which company has the highest total revenue?
- Which company has the strongest operating margin?
- Which period had the strongest revenue growth?
- Which company appears to have the best liquidity position?

The project is useful for financial reporting, portfolio analysis, and data engineering demonstrations because it combines data ingestion, metric creation, database storage, and output generation in a single workflow.

## Setup

Using Poetry:

```bash
poetry install
poetry run python financial_pipeline.py
```

Using pip:

```bash
pip install pandas matplotlib pdfplumber openpyxl
python financial_pipeline.py
```

## Future Improvements

Possible next steps for the project include:

- improving regex extraction for more document variations
- adding stronger validation rules for malformed or incomplete records
- expanding support for scanned PDFs and OCR workflows
- creating automated tests for KPI accuracy and data quality
- adding a Streamlit dashboard for interactive exploration
- connecting outputs to Power BI or a warehouse environment

## Repository Notes

The main project logic is in [financial_pipeline.py](financial_pipeline.py). The companion [analysis_agent.py](analysis_agent.py) is a separate prototype focused on AI-assisted financial analysis and report generation.
