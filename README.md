# 🏠 Airbnb CDC Data Pipeline & AI-Powered Analytics on Azure

An end-to-end data engineering and AI automation project built using **Azure Data Factory, Azure Data Lake Storage Gen2, Azure Cosmos DB, Azure Synapse Analytics, n8n, and OpenAI**.

The solution demonstrates **customer dimension processing, Cosmos DB Change Feed-based CDC ingestion, data-quality validation, insert/update processing, dimensional analytics, pipeline orchestration, and AI-powered personalized email automation**.

---

## 📌 Project Overview

This project implements an Airbnb-inspired booking data platform that integrates customer and booking data from different sources into an analytical data warehouse on Azure Synapse Analytics.

The solution consists of two primary ingestion paths:

- **Customer Data Pipeline:** Ingests customer files from ADLS Gen2 and performs an SCD Type-1-style upsert into the customer dimension.
- **Booking CDC Pipeline:** Consumes booking events from the Cosmos DB Change Feed, validates and transforms the events using an ADF Mapping Data Flow, and loads them into the booking fact table.

The processed data is then aggregated in Synapse and consumed by an **n8n + OpenAI workflow** to generate personalized customer communication automatically through Microsoft Outlook.

---

## 🏗️ Solution Architecture

<p align="center">
  <img src="Architecture/architecture_diagram.png" width="950"/>
</p>

### High-Level Data Flow

```text
                    ┌──────────────────────┐
                    │      ADLS Gen2       │
                    │   Customer Raw Data  │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   Azure Data Factory │
                    │ Customer Ingestion   │
                    └──────────┬───────────┘
                               │
                         SCD Type 1
                           Upsert
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Azure Synapse        │
                    │   dim_customer       │
                    └──────────────────────┘


                    ┌──────────────────────┐
                    │   Azure Cosmos DB   │
                    │    Booking Events   │
                    └──────────┬───────────┘
                               │
                         Change Feed / CDC
                               │
                               ▼
                    ┌──────────────────────┐
                    │   Azure Data Factory │
                    │   Booking Pipeline   │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │  Mapping Data Flow   │
                    │                      │
                    │ • Data Quality       │
                    │ • Lookup             │
                    │ • Insert / Update    │
                    │ • Column Mapping     │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Azure Synapse        │
                    │    fact_booking      │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Booking Aggregation  │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │         n8n          │
                    │    AI Automation     │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   OpenAI Chat Model  │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Microsoft Outlook    │
                    │ Personalized Email   │
                    └──────────────────────┘
🛠️ Technology Stack
Technology	Purpose
Azure Data Lake Storage Gen2	Customer raw-file storage and archival
Azure Data Factory	Data ingestion, orchestration and transformation
ADF Mapping Data Flow	Booking transformation, validation and insert/update processing
Azure Cosmos DB for NoSQL	Booking event source
Cosmos DB Change Feed	Incremental CDC event ingestion
Azure Synapse Analytics	Analytical warehouse and dimensional data model
SQL	Table creation, aggregation and stored procedures
n8n	AI workflow orchestration
OpenAI	Personalized email generation
Microsoft Outlook	Automated email delivery
🎯 Business Problem

An Airbnb-style booking platform generates customer and booking information from multiple operational sources.

The data engineering solution needs to:

Ingest customer information from cloud storage.
Keep the customer dimension up to date.
Capture incremental booking changes.
Validate incoming booking events.
Insert new bookings and update existing bookings.
Maintain analytical tables in Synapse.
Generate useful business-level booking aggregations.
Use processed data to automate personalized customer communication.
🔄 Customer Data Pipeline

Customer data is stored in Azure Data Lake Storage Gen2.

Storage Structure
ADLS Gen2
└── airbnb
    ├── customer-raw-data
    └── customer-data-archive
Pipeline

ADF Pipeline: New_LoadCustomerDim

Get Metadata
      ↓
ForEach Customer File
      ↓
Copy Customer Data
      ↓
Synapse dim_customer
      ↓
Archive Processed File
      ↓
Delete Source File
Processing Logic
Get Metadata identifies files available in the raw folder.
ForEach processes each customer file.
Customer records are loaded into airbnb.dim_customer.
customer_id is used as the upsert key.
Successfully processed files are copied to the archive location.
The original source file is deleted after successful archival.

This provides controlled file processing while preventing already-processed files from remaining in the raw ingestion area.

🔁 Booking CDC Pipeline

Booking events are stored in Azure Cosmos DB for NoSQL.

Cosmos DB
└── Database: Airbnb
    └── Container: bookings

The ADF Mapping Data Flow consumes the Cosmos DB Change Feed for incremental booking-event processing.

Pipeline

ADF Pipeline: New_LoadBookingFact

Cosmos DB Change Feed
          ↓
New_BookingTransformation
          ↓
BookingAggregation Stored Procedure
🔍 Booking Transformation

Mapping Data Flow: New_BookingTransformation

The transformation performs the following operations:

Cosmos DB Change Feed
          ↓
Data Quality Check
          ↓
Accepted Records
          ↓
Lookup Existing booking_id
          ↓
Generate Insert / Update Flags
          ↓
Final Column Mapping
          ↓
Synapse fact_booking
1. Cosmos DB Change Feed

The source is configured to consume the Cosmos DB Change Feed.

enableChangeFeed = true
changeFeedStartFromTheBeginning = true

This allows booking changes to be processed incrementally.

2. Data Quality Validation

A data-quality split is applied to booking events.

One validation condition checks:

checkout_date < checkin_date

Records failing the validation are separated as bad records, while accepted records continue through the pipeline.

3. Existing Booking Lookup

Accepted records are compared with the existing Synapse booking fact table using:

booking_id

The lookup determines whether the incoming event represents an existing booking.

4. Insert / Update Decision

The pipeline dynamically determines the required database operation:

If booking_id does not exist
        → INSERT

If booking_id already exists
        → UPDATE

This allows incremental booking events to maintain the latest state of the booking fact table.

🏢 Synapse Data Model

The analytical layer is implemented in Azure Synapse Analytics using the airbnb schema.

Customer Dimension
airbnb.dim_customer

Important attributes include:

customer_id
first_name
last_name
email
phone_number
address
city
state
country
zip_code
signup_date
last_login
total_bookings
total_spent
preferred_language
referral_code
account_status
Booking Fact
airbnb.fact_booking

Important attributes include:

booking_id
customer_id
listing_id
status
booking_created_at
checkin_date
checkout_date
nights
lead_time_days
guests_adults
guests_children
guests_infants
price_nightly
cleaning_fee
total_amount
currency
country_code
city
channel
device_type
cancellation_ts
cancellation_reason
updated_at
📊 Booking Aggregation

After successful booking transformation, the pipeline executes:

airbnb.BookingAggregation

The procedure populates:

airbnb.BookingCustomerAggregation

The aggregation layer provides metrics such as:

Total bookings
Confirmed bookings
Cancelled bookings
Total booking amount
Cancellation rate
Average booking amount
Minimum booking amount
Maximum booking amount
Distinct customers
Average stay duration

This creates a business-oriented analytical layer on top of the booking fact data.

🔗 Pipeline Orchestration

The project uses a master ADF pipeline:

New_FinalAirBNBPipeline

The master pipeline executes the child pipelines sequentially.

New_FinalAirBNBPipeline
          │
          ▼
New_LoadCustomerDim
          │
      Succeeded
          │
          ▼
New_LoadBookingFact

The booking pipeline starts only after the customer pipeline completes successfully.

This provides dependency-based orchestration using reusable child pipelines.

🤖 AI-Powered Customer Communication

The project extends the data platform with an AI automation workflow using n8n and OpenAI.

n8n Workflow
Microsoft SQL
      ↓
OpenAI Chat Model
      ↓
Structured Output Parser
      ↓
Microsoft Outlook

The workflow queries processed booking/customer information from Synapse.

Relevant contextual information is passed to the OpenAI Chat Model to generate a personalized communication.

The structured output contains:

{
  "email": "...",
  "subject": "...",
  "body": "..."
}

The generated email is then delivered through Microsoft Outlook.

AI Flow
Synapse Processed Data
        ↓
Business Context
        ↓
Custom Prompt
        ↓
OpenAI Chat Model
        ↓
Structured Email Output
        ↓
Microsoft Outlook

This demonstrates how an analytical data platform can be extended into an AI-powered business automation workflow.

📸 Project Screenshots
Master ADF Pipeline
<p align="center"> <img src="Images/01_final_airbnb_pipeline.png" width="900"/> </p>
Customer Ingestion Pipeline
<p align="center"> <img src="Images/02_customer_ingestion_pipeline.png" width="900"/> </p>
Booking CDC Pipeline
<p align="center"> <img src="Images/03_booking_cdc_pipeline.png" width="900"/> </p>
Booking Transformation Data Flow
<p align="center"> <img src="Images/04_booking_transformation_dataflow.png" width="900"/> </p>
Synapse Customer Dimension
<p align="center"> <img src="Images/05_dim_customer.png" width="900"/> </p>
Synapse Booking Fact
<p align="center"> <img src="Images/06_fact_booking.png" width="900"/> </p>
Booking Aggregation
<p align="center"> <img src="Images/07_booking_aggregation.png" width="900"/> </p>
Cosmos DB Booking Container
<p align="center"> <img src="Images/08_cosmos_booking_container.png" width="900"/> </p>
n8n AI Automation
<p align="center"> <img src="Images/09_n8n_ai_workflow.png" width="900"/> </p>
AI-Generated Email
<p align="center"> <img src="Images/10_ai_generated_email.png" width="900"/> </p>
📁 Repository Structure
airbnb-cdc-data-pipeline-ai-analytics/
│
├── Architecture/
│   └── architecture_diagram.png
│
├── Images/
│   ├── 01_final_airbnb_pipeline.png
│   ├── 02_customer_ingestion_pipeline.png
│   ├── 03_booking_cdc_pipeline.png
│   ├── 04_booking_transformation_dataflow.png
│   ├── 05_dim_customer.png
│   ├── 06_fact_booking.png
│   ├── 07_booking_aggregation.png
│   ├── 08_cosmos_booking_container.png
│   ├── 09_n8n_ai_workflow.png
│   └── 10_ai_generated_email.png
│
├── ADF/
│   ├── Pipelines/
│   ├── DataFlows/
│   ├── Datasets/
│   └── LinkedServices/
│
├── Synapse/
│   └── SQLScripts/
│
├── n8n/
│   └── Airbnb_AI_Email_Automation.json
│
├── .gitignore
├── LICENSE
└── README.md
⭐ Key Engineering Features
Data Engineering
Azure Data Factory pipeline orchestration
Dynamic file processing using Get Metadata and ForEach
ADLS Gen2 ingestion
Cosmos DB Change Feed / CDC processing
Mapping Data Flow transformations
Data-quality validation
Lookup-based insert/update processing
SCD Type-1-style customer upsert
Synapse dimensional modeling
Stored-procedure-based aggregation
AI & Automation
n8n workflow automation
OpenAI Chat Model integration
Context-aware prompt generation
Structured LLM output
Automated Outlook email delivery
Architecture
Reusable child pipelines
Master pipeline orchestration
Dependency-based execution
Separation of raw, analytical and automation layers
Incremental booking-event processing
🔐 Security Considerations

Sensitive credentials and secrets are intentionally excluded from this repository.

The repository does not contain:

Azure storage keys
Cosmos DB keys
OpenAI API keys
SQL passwords
OAuth tokens
Connection secrets

Azure and n8n configuration files included in the repository are sanitized to remove sensitive credential information.

🚀 Project Outcome

The solution demonstrates an end-to-end cloud data engineering architecture capable of:

Processing customer data from ADLS Gen2
Maintaining a customer dimension using upsert processing
Capturing incremental booking events through Cosmos DB Change Feed
Performing data-quality validation
Detecting new and existing booking records
Maintaining a Synapse booking fact table
Generating analytical booking aggregations
Orchestrating multiple data pipelines
Using processed business data as context for an LLM
Generating personalized customer communication automatically
🔮 Future Enhancements

Potential production enhancements include:

Automated ADF trigger-based scheduling
Dead-letter storage for rejected booking events
Centralized monitoring and alerting
Azure Key Vault integration for secret management
CI/CD deployment using Azure DevOps or GitHub Actions
Data-quality metrics and operational dashboards
Additional business intelligence reporting
More advanced AI-driven customer segmentation
👨‍💻 Project

Airbnb CDC Data Pipeline & AI-Powered Analytics on Azure

Built as an end-to-end cloud data engineering and AI automation project demonstrating Azure-based ingestion, CDC processing, dimensional analytics, orchestration, and LLM-powered workflow automation.
