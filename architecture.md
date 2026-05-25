# Architecture — Weekly Churn Prediction System

Scenario: **B — Weekly churn predictions for a B2B SaaS company**

Serving pattern: **Batch inference**

```mermaid
flowchart TB
    subgraph Sources["Data Sources"]
        CRM["CRM<br/>accounts, stages, owner"]
        Product["Product Analytics<br/>usage events, logins, seats"]
        Billing["Billing System<br/>invoices, plan, payments"]
    end

    subgraph DataLayer["Offline Data Layer"]
        Ingestion["Ingestion Jobs"]
        Warehouse["Data Warehouse"]
        FeaturePipeline["Feature Pipeline<br/>weekly account features"]
        FeatureTable["Feature Table<br/>account_id, week, features"]
    end

    subgraph TrainingLayer["Training + Registry"]
        Training["Training Pipeline"]
        Registry["Model Registry<br/>approved churn model"]
    end

    subgraph ServingLayer["Batch Serving Boundary"]
        Scoring["Weekly Batch Scoring Job<br/>runs every Monday"]
        Scores["Churn Scores Table<br/>account_id, score, rank, reason codes"]
    end

    subgraph Consumers["Downstream Consumers"]
        Dashboard["CRM Dashboard<br/>ranked at-risk accounts"]
        Playbooks["Outreach Playbooks<br/>CSM follow-up tasks"]
    end

    subgraph Feedback["Monitoring + Feedback Loop"]
        Monitoring["Monitoring<br/>data drift, score drift, accuracy, job health"]
        Outcomes["Outcome Labels<br/>actual churn, renewals, outreach results"]
    end

    CRM -->|"account snapshots"| Ingestion
    Product -->|"usage events batch"| Ingestion
    Billing -->|"billing records batch"| Ingestion

    Ingestion -->|"cleaned batch writes"| Warehouse
    Warehouse -->|"historical account data"| FeaturePipeline
    FeaturePipeline -->|"weekly feature rows"| FeatureTable

    FeatureTable -->|"training dataset"| Training
    Outcomes -->|"labels"| Training
    Training -->|"model artifact + metrics"| Registry

    Registry -->|"approved model version"| Scoring
    FeatureTable -->|"latest weekly features"| Scoring
    Scoring -->|"batch predictions"| Scores

    Scores -->|"ranked churn-risk list"| Dashboard
    Scores -->|"risk segments + reason codes"| Playbooks

    Scoring -->|"prediction logs"| Monitoring
    Scores -->|"score distribution"| Monitoring
    Warehouse -->|"freshness + schema checks"| Monitoring
    Monitoring -->|"drift / quality signals"| Training
    Dashboard -->|"sales actions + notes"| Outcomes
    Playbooks -->|"outreach results"| Outcomes
```

## Notes

This architecture uses **batch inference** because the business only needs updated churn scores once per week. The sales team receives a ranked list every Monday morning and uses the same list throughout the week.

The serving boundary is the weekly batch scoring section: the approved model is loaded from the model registry, the latest account features are read from the feature table, and the output is written to the churn scores table.

The feedback loop comes from monitoring signals, actual churn outcomes, renewals, and sales outreach results. These outputs feed the next training cycle so the model can improve over time.
