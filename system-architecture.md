# System Architecture – Olympic Data Processing Pipeline

```text
┌──────────────────────────────────────────────────────────────┐
│                        DATA SOURCES                          │
│──────────────────────────────────────────────────────────────│
│                                                              │
│  Original Olympic Datasets            Paris 2024 Datasets    │
│  ──────────────────────────           ─────────────────────  │
│  • olympic_athlete_bio.csv             • athletes.csv        │
│  • olympic_athlete_event_results.csv   • medallists.csv      │
│  • olympics_country.csv                • events.csv          │
│  • olympics_games.csv                  • teams.csv           │
│                                       • nocs.csv             │
│                                                              │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────┐
│                   PARIS DATA HANDLER                         │
│            (Integration & Reconciliation Layer)              │
│──────────────────────────────────────────────────────────────│
│ • Merges Paris data into existing datasets                   │
│ • Resolves athlete identity using (Name + NOC)               │
│ • Assigns consistent athlete_id values                       │
│ • Prevents duplicate athlete and country records             │
│ • Detects and labels team-based events                       │
│                                                              │
│  Architecture Role:                                          │
│  → Data Ingestion + Data Reconciliation                      │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────┐
│                       DATA CLEANER                           │
│             (Validation & Normalization Layer)               │
│──────────────────────────────────────────────────────────────│
│ • Cleans fuzzy and inconsistent birth dates                  │
│ • Normalizes all date fields to dd-Mon-yyyy                  │
│ • Validates medal values and athlete positions               │
│ • Deduplicates country and NOC records                       │
│ • Standardizes formatting across all datasets                │
│                                                              │
│  Architecture Role:                                          │
│  → Data Quality Assurance / Sanitization                     │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────┐
│                     AGE CALCULATOR                           │
│                (Derived Data / Enrichment Layer)             │
│──────────────────────────────────────────────────────────────│
│ • Builds cached lookup tables for athlete DOBs               │
│ • Builds cached lookup tables for Olympic start/end dates    │
│ • Calculates athlete age per Olympic edition                 │
│ • Applies birthday-during-Olympics rule                      │
│                                                              │
│  Architecture Role:                                          │
│  → Data Enrichment / Feature Derivation                      │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────┐
│                 MEDAL TALLY CALCULATOR                       │
│               (Aggregation & Analytics Layer)                │
│──────────────────────────────────────────────────────────────│
│ • Aggregates medals by Edition and Country (NOC)             │
│ • Deduplicates medals for team-based events                  │
│ • Counts unique athletes per country and edition             │
│ • Computes total medal counts                                │
│                                                              │
│  Architecture Role:                                          │
│  → Analytical Aggregation / Reporting                        │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────┐
│                     FINAL OUTPUT LAYER                       │
│──────────────────────────────────────────────────────────────│
│ ✔ new_olympic_athlete_bio.csv                                │
│ ✔ new_olympic_athlete_event_results.csv                      │
│ ✔ new_olympics_country.csv                                   │
│ ✔ new_olympics_games.csv                                     │
│ ✔ new_medal_tally.csv                                        │
│                                                              │
│  Architecture Role:                                          │
│  → Persistent, Clean, Query-Ready Data                       │
└──────────────────────────────────────────────────────────────┘
