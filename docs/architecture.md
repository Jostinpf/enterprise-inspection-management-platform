# Architectural Overview

This document details the system architecture, relational database design, data flow automation, and core structural modules of the **Enterprise Inspection Management Platform**.

---

## 1. System Topology & Technology Stack

The platform implements a decoupled serverless architecture utilizing the Google Workspace ecosystem and the AppSheet Core engine. Core responsibilities are distributed across the following layers:

| Layer | Component | Implementation |
| :--- | :--- | :--- |
| **Frontend / Client** | AppSheet Native Framework | Responsive mobile/tablet interface, client-side data validation, dynamic UI formatting, and local caching. |
| **Backend / Database** | Google Sheets & Cloud Engines | Relational relational schema, AppSheet virtual columns for real-time calculations, and row-level security (RLS) filters. |
| **Automation Engine** | AppSheet Automation Bots | Asynchronous document synthesis pipeline, event-driven webhooks, and trigger-based data synchronization. |
| **Storage & Assets** | Google Drive Enterprise | Structured cloud binary storage acting as the primary repository for evidence photographs and compiled PDF outputs. |

---

## 2. Relational Schema & Data Modeling

The relational database enforces a strict hierarchical structural design (One-to-Many). The Inspection Master entity controls metadata and synchronization states, routing specific transactional data to individual engineering submodules.

```text
[Project Entity]
   │
   └───► [Inspection Master Record] (Status & Metadata Control)
            │
            ├───► [Electrical Inspection Module] ───────► [Evidence Photographs]
            ├───► [Black Water Inspection Module] ──────► [Evidence Photographs]
            ├───► [Storm Water Inspection Module] ──────► [Evidence Photographs]
            ├───► [Fire Suppression Module] ────────────► [Evidence Photographs]
            ├───► [Service Entrance Isolation Module] ──► [Evidence Photographs]
            └───► [Potable Water Inspection Module] ────► [Evidence Photographs]
