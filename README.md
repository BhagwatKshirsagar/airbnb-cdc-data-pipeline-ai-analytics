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
---

## 🛠️ Technology Stack

| Technology | Purpose |
|---|---|
| **Azure Data Lake Storage Gen2** | Customer raw-file storage and archival |
| **Azure Data Factory** | Data ingestion, pipeline orchestration and transformation |
| **ADF Mapping Data Flow** | Booking transformation, validation and insert/update processing |
| **Azure Cosmos DB for NoSQL** | Booking event source |
| **Cosmos DB Change Feed** | Incremental CDC event ingestion |
| **Azure Synapse Analytics** | Analytical warehouse and dimensional data model |
| **SQL** | Aggregation logic and stored procedures |
| **n8n** | AI workflow orchestration |
| **OpenAI** | Personalized email generation |
| **Microsoft Outlook** | Automated email delivery |

---

## 🎯 Business Problem

An Airbnb-style booking platform generates customer and booking information from multiple operational sources.

The data engineering solution needs to:

1. Ingest customer information from cloud storage.
2. Keep the customer dimension up to date.
3. Capture incremental booking changes using CDC.
4. Validate incoming booking events.
5. Insert new bookings and update existing bookings.
6. Maintain analytical tables in Azure Synapse Analytics.
7. Generate business-level booking aggregations.
8. Use processed data to automate personalized customer communication.

---

## 🔄 Customer Data Pipeline

Customer data is stored in **Azure Data Lake Storage Gen2 (ADLS Gen2)** and processed through Azure Data Factory.

### Storage Structure

```text
ADLS Gen2
└── airbnb
    ├── customer-raw-data
    └── customer-data-archive
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

---

## 🔁 Booking CDC Pipeline

Booking events are stored in **Azure Cosmos DB for NoSQL**.

### Cosmos DB Source

```text
Azure Cosmos DB
└── Container: bookings

Cosmos DB Change Feed

---

## 🔍 Booking Transformation & Data Quality

The **`New_BookingTransformation`** Mapping Data Flow processes incremental booking events received from the Cosmos DB Change Feed.

### Transformation Flow

```text
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
          ↓
New_BookingTransformation
          ↓
BookingAggregation Stored Procedure
          ↓
Synapse fact_booking
