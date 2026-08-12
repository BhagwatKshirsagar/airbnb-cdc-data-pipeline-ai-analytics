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
                    │ Personalized Email  │
                    └──────────────────────┘
