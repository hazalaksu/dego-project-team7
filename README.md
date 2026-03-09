# DEGO Project – Team 7

## Team Members
- [Hazal Aksu]
- [Thomas Teichmann]
- [Mariana Temudo]
- [Xaver Baeppler]

## Project Description

Credit scoring bias analysis for DEGO course.

## Structure
```
├── data/              # Raw and processed datasets
├── notebooks/         # Jupyter notebooks for data quality and analysis
├── presentation/      # PowerPoint slides and presentation materials
├── src/               # Structural placeholder for future Python package deployment
└── requirements.txt   # List of dependencies for the project
```

## Setup and Execution
### Dependencies 
All required libraries are listed in the `requirements.txt` file.  
You can install them via pip:

`pip install -r requirements.txt`

### Data Processing 
Executing the `01-data-quality.ipynb` notebook processes the raw data and generates the following two files:

* `processed_credit_applications.json`

* `credit_applications_with_age.csv`


## Executive Summary
This report outlines the findings of the Data Governance Task Force following a comprehensive audit of NovaCred's credit application dataset. The analysis spanned data engineering, data science, algorithmic fairness, and privacy compliance. We successfully resolved initial data quality issues, but identified a statistically significant gender bias in the loan approval process that violates regulatory thresholds. Additionally, exposed Personally Identifiable Information (PII) required immediate pseudonymization to ensure compliance with GDPR and the EU AI Act.





### 1. Data Quality & Engineering Findings

* **Data Ingestion & Deduplication:** Ingested 502 raw nested JSON records into MongoDB. Identified and removed 1 duplicate entry using the `app_id` field, yielding a dataset of 501 unique records.
* **Schema & Type Correction:** Standardized mixed data types across the dataset. Most notably, parsed inconsistent string and integer formats in `financials.annual_income` into uniform floats, and converted `date_of_birth` strings into proper datetime objects with the format "YYYY-MM-DD". Standardized `gender` values into the two categories “Female” and “Male”.
* **Missing Value Handling & Imputation:** Took a business-logic-driven approach to deleting and imputing missing values. Observed that the missing values in the `interest_rate`, `approved_amount`, and `rejection_reason` fields were intentional and corresponded perfectly to the 502 approved versus rejected loan decisions and thus decided not to delete any rows missing these specific fields. Instead of deleting rows for the `notes` field, which was almost entirely empty (99.6% missing), opted to remove the field altogether during curation.  The documents with missing `gender` and `date_of_birth` values were deleted to not interfere with analysis in the Data Scientist’s part. The `annual_salary` field was mapped to the `annual_income` field and dropped.
* **Data Transformation:** Flattened complex nested dictionaries (`applicant_info`, `financials`, `decision`) into a structured tabular form and exported the entire dataset into a CSV file. Engineered a new `age` feature from the date of birth to enable downstream algorithmic bias testing.
* **Outcome:** Exported a fully clean, standardized, and analysis-ready dataset (`credit_applications_with_age.csv`) containing 494 records.

### 2. Algorithmic Bias & Fairness Findings

* **Gender Disparity:** Identified a statistically significant gender bias in loan approvals. The female approval rate (50.0%) compared to the male approval rate (66.1%) results in a Disparate Impact (DI) ratio of 0.756. This falls below the 0.80 regulatory "four-fifths rule" threshold, triggering a formal fairness warning.
* **Age-Based Penalties:** Applicants under 30 face a significant penalty, with an approval rate of 41.2% and a DI ratio of 0.667 when compared to the prime-age reference group (30-49). Older applicants (50+) do not face systemic barriers and maintain near parity (DI: 0.967).
* **Proxy Variable Risk:** Geographic location (`zip_code`) acts as an indirect channel transmitting bias. There is a significant approval disparity between New York and Los Angeles (DI: 0.799) that strongly correlates with gender composition, raising proxy-discrimination risks.
* **Intersectional Disadvantage:** Bias is intersectionally compounded for younger women. The largest approval gap occurs in the 25–34 age bracket, where females face a severe 23.4 percentage point penalty compared to males in the same cohort.
* **Governance Directives:** Recommended immediate actions include establishing formal fairness triggers, investigating proxy variables (ZIP code) driving the disparities, and standardizing intersectional KPIs (Gender × Age) for all future model monitoring.

### 3. Privacy, Security & Compliance Findings

* **PII Exposure & Audit Risks:** Initial scans revealed that 100% of the dataset exposed Direct Identifiers (names, emails, SSNs in plain text), violating GDPR Article 32. The audit also flagged excessive Online Identifiers (IP addresses) and Quasi-Identifiers (ZIP codes) that presented severe geographic "redlining" risks.
* **Cryptographic Pseudonymization:** Implemented SHA-256 cryptographic hashing to mask sensitive identifiers (SSNs and emails). This de-coupled applicant identities from the processing activity while preserving referential integrity for the credit scoring algorithm.
* **Data Minimization:** Irreversibly purged non-essential and sensitive fields, including IP addresses and specific spending behaviors (e.g., 'Alcohol' transactions), ensuring compliance with GDPR Article 5.1(c).
* **Governance Metadata Injection:** Embedded a comprehensive regulatory compliance layer directly into the NoSQL document structure:
* **EU AI Act Traceability & Oversight:** Logged model versions (Art. 12) and instituted a mandatory human-in-the-loop oversight flag (`human_oversight_required: True`) to prevent fully automated bias in this high-risk credit system (Art. 14).
* **GDPR Lifecycle Controls:** Injected explicit consent tracking mechanisms (Art. 7) and implemented an automated 5-year expiration date on all records to enforce strict storage limitation (Art. 5.1.e).