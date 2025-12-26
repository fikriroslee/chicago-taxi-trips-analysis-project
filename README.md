# Chicago Taxi Trips Analytics Pipeline

## Overview
Production-ready data pipeline analyzing Chicago taxi trips using GCP Dataform and BigQuery. This project transforms the public Chicago Taxi dataset into actionable business insights for operational optimization and driver welfare monitoring.

## Author
Muhammad Fikri Bin Mohd Roslee 

---

## Table of Contents
- [Architecture](#architecture)
- [Dataset Information](#dataset-information)
- [Project Structure](#project-structure)
- [Setup Instructions](#setup-instructions)
- [Data Models](#data-models)
- [Analytical Questions & Findings](#analytical-questions--findings)
- [Data Quality Notes](#data-quality-notes)
- [Looker Studio Dashboard](#looker-studio-dashboard)
- [CI/CD Implementation](#cicd-implementation)
- [Technologies Used](#technologies-used)
- [Future Enhancements](#future-enhancements)

---

## Architecture
```
┌─────────────────────┐
│  BigQuery Public    │
│  Chicago Taxi Data  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   GCP Dataform      │
│   Transformations   │
│                     │
│  ┌───────────────┐  │
│  │   Staging     │  │
│  │  (Cleaning &  │  │
│  │   Dedup)      │  │
│  └───────┬───────┘  │
│          │          │
│          ▼          │
│  ┌───────────────┐  │
│  │    Marts      │  │
│  │  (Analytics)  │  │
│  └───────────────┘  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Looker Studio      │
│  Dashboard          │
└─────────────────────┘
```

**Components:**
- **Source Data:** BigQuery Public Dataset (`bigquery-public-data.chicago_taxi_trips.taxi_trips`)
- **Transformation Tool:** GCP Dataform
- **Data Warehouse:** Google BigQuery
- **Visualization:** Looker Studio
- **Version Control:** GitHub
- **CI/CD:** GitHub Actions

---

## Dataset Information

**Source:** Chicago Taxi Trips - Public BigQuery Dataset  
**Full Dataset Range:** January 1, 2013 - December 31, 2023  
**Total Records:** ~200+ million trips (entire dataset)

**Analysis Scope:**  
**Staging Layer:** January 1, 2023 - December 31, 2023 (1 complete year)  
**Records Analyzed:** ~13 million trips from 2023

**Analysis Period by Question:**

| Question | Time Period | Rationale |
|----------|-------------|-----------|
| **Question 1: Top Tip Earners** | Q4 2023 (Oct-Dec 2023) | "Last 3 months" of available data |
| **Question 2: Top Overworkers** | Q4 2023 (Oct-Dec 2023) | "Last 3 months" of available data |
| **Question 3: Holiday Impact** | 2023 (1 year) | Whole-year analysis for robust trend identification |

**Important Notes:**
- The public dataset's most recent data is from **December 31, 2023**
- All temporal references (e.g., "last 3 months") are relative to this end date, not the current date
- Holiday analysis uses a whole year (2023) to ensure statistical significance and identify consistent patterns
- Staging layer includes only the year 2023 to support recent operational analysis

---

## Project Structure
```
time-dotcom-chicago-taxi-trips-analysis-project/
├── definitions/
│   ├── staging/
│   │   └── staging_taxi_trips.sqlx       # Cleaned source data (2023) with deduplication
│   └── marts/
│       ├── top_100_tip_earners.sqlx          # Question 1: Top 100 tip earners (Q4 2023)
│       ├── top_100_overworkers.sqlx          # Question 2: Top 100 overworkers (Q4 2023)
│       ├── holiday_impact_on_trips.sqlx      # Question 3: Holiday impact analysis (2023)
├── .github/
│   └── workflows/
│       └── dataform-deploy.yml           # CI/CD automation pipeline
├── workflow_settings.yaml                # Dataform project configuration
└── README.md                             # This file
```

---

## Setup Instructions

### Prerequisites
- Google Cloud Platform account with billing enabled (free $300 credits available)
- GitHub account
- Access to BigQuery and Dataform APIs

### Step 1: Clone Repository
```bash
git clone https://github.com/fikriroslee/time-dotcom-chicago-taxi-trips-analysis-project.git
cd time-dotcom-chicago-taxi-trips-analysis-project
```

### Step 2: Enable GCP APIs
Navigate to GCP Console and enable:
- BigQuery API
- Dataform API
- Cloud Build API (for CI/CD)

### Step 3: Create Dataform Repository
1. In GCP Console, navigate to Dataform
2. Create new repository: `time-chicago-taxi-pipeline`
3. Link to this GitHub repository
4. Create development workspace: `dev-workspace`

### Step 4: Run the Pipeline
1. In Dataform workspace, click **"COMPILE"** to validate all models
2. Click **"RUN"** to execute transformations
3. Verify tables created in BigQuery:
   - `[project].staging.staging_taxi_trips`
   - `[project].marts.top_100_tip_earners`
   - `[project].marts.top_100_overworkers`
   - `[project].marts.holiday_impact_on_trips`

### Step 5: View Dashboard
Access the Looker Studio dashboard: https://lookerstudio.google.com/reporting/fe3148f5-7261-45b1-bae8-5cb32a17ef94

---

## Data Models

### Staging Layer

#### `staging_taxi_trips`
**Purpose:** Foundation layer providing clean, deduplicated trip records from 2023

**Time Coverage:** January 1, 2023 - December 31, 2023 (1 complete year)

**Transformations Applied:**
- ✅ Removed duplicate records (same unique_key appearing multiple times)
- ✅ Filtered for data quality (non-null values, positive amounts)
- ✅ Calculated trip_hours from trip_seconds
- ✅ Extracted date components (year, month, day_of_week, hour_of_day)

**Output:** ~6 million cleaned trip records from 2023

**Key Fields:**
- `unique_key` - Unique trip identifier
- `taxi_id` - Taxi medallion ID
- `trip_hours` - Trip duration in hours (calculated)
- `trip_date` - Date of trip
- `trip_year` - Year extracted for filtering
- `hour_of_day` - Hour when trip started (0-23)

**Design Decision:** Includes 1 year of data to support both recent operational analysis (Q4 2023 for tips/overworkers) and a whole-year trend analysis (2023 for holidays).

---

### Mart Layer (Analytics Models)

#### 1. `top_100_tip_earners`
**Business Question:** Who are the top 100 "tip earners" - taxi IDs that earn more money in tips than others for the last 3 months?

**Time Period:** Q4 2023 (October 1 - December 31, 2023)

**Methodology:**
- Aggregated tips by taxi_id for Q4 2023 (most recent 3 months in dataset)
- Calculated total tips, average tip per trip, and tip percentage
- Filtered for taxis with at least 10 trips (statistical significance)
- Ranked by total tips earned

**Key Metrics:**
- `total_tips` - Sum of all tips earned
- `avg_tip_per_trip` - Average tip amount per trip
- `tip_percentage` - Tips as % of fare (tips/fare * 100)
- `total_trips` - Number of trips completed
- `total_hours_driven` - Total driving hours

**Business Value:** 
- Identify high-performing drivers for recognition programs
- Study service patterns of top earners to train other drivers
- Understand characteristics that lead to higher tips
- Benchmark performance metrics for driver evaluation

---

#### 2. `top_100_overworkers`
**Business Question:** Who are the top 100 "overworkers" - taxi IDs that work more hours than others without taking at least 8 hours break and regularly have long shifts?

**Time Period:** Q4 2023 (October 1 - December 31, 2023)

**Methodology & Assumptions:**
- **Insufficient break:** Less than 8 hours between trip end and next trip start
- **Long shift:** Single trip lasting more than 10 hours
- **"Regular overworker":** At least 10 instances of insufficient breaks OR 5+ long shifts
- Used window functions (`LEAD`) to calculate break time between consecutive trips per taxi
- Calculated percentage of shifts without proper rest

**Key Metrics:**
- `shifts_without_8hr_break` - Count of trips followed by <8hr break
- `pct_shifts_without_break` - Percentage of total shifts with insufficient rest
- `total_hours_worked` - Sum of all trip hours
- `longest_single_shift_hours` - Maximum continuous driving time
- `avg_hours_per_day` - Average daily working hours
- `long_shifts_over_10hrs` - Count of exceptionally long single trips

**Business Value:** 
- Monitor driver welfare and regulatory compliance
- Prevent burnout and safety incidents
- Identify taxis needing intervention or rest requirements
- Support labor law compliance and driver health programs
- Reduce accident risk from fatigued drivers

#### 3. `holiday_impact_on_trips`
**Business Question:** Do US public holidays have an impact on increasing/decreasing taxi trips?

**Time Period:** 2023 (1 complete year)

**Holidays Analyzed:**
- New Year's Day (January 1)
- Independence Day (July 4)
- Labor Day (First Monday in September)
- Thanksgiving (4th Thursday in November)
- Day After Thanksgiving (Black Friday)
- Christmas (December 25)
- Observed holidays when falling on weekends

**Methodology:**
- Compared daily trip counts, fares, and tips on holidays vs non-holidays
- Calculated percentage change from non-holiday averages
- Analyzed 1 year of data (2023) for pattern consistency and statistical significance

**Metrics Compared:**
- Trip volume (count)
- Average fare per trip
- Average tips per trip
- Trip distance
- Total revenue (fare + tips)

**Business Value:**
- Optimize fleet deployment strategies around holidays
- Adjust pricing and surge policies based on demand patterns
- Plan driver scheduling for high/low demand periods
- Forecast revenue impacts of holiday calendar
- Inform marketing and promotional strategies

---

## Data Quality Notes

### Issues Identified & Resolved

**1. Duplicate Trip Records**
- **Issue:** Source dataset contains duplicate rows with identical `unique_key` values
- **Impact:** Without correction, would inflate trip counts, distort break time calculations, and skew all analytics
- **Root Cause:** Data collection/reporting process occasionally creates duplicate entries for the same physical trip
- **Solution:** Applied `SELECT DISTINCT` in staging model to remove exact duplicate rows
- **Verification:** Post-deduplication query confirms zero duplicate unique_keys in staging layer
- **Documentation:** Logged in staging model comments and this README for transparency
- **Query Used for Detection:**
```sql
SELECT unique_key, COUNT(*) as occurrences
FROM source_table
GROUP BY unique_key
HAVING COUNT(*) > 1
```

**2. trip_total Field Inconsistencies**
- **Issue:** `trip_total` field doesn't always equal `fare + tips + tolls + extras`
- **Scale:** Approximately [X]% of records show calculation discrepancies
- **Impact:** Minimal - this field is not used in core analysis metrics
- **Analysis:** Verified that `fare`, `tips`, `tolls`, and `extras` fields individually are reliable
- **Decision:** Documented but not corrected, as our analysis focuses on individual components (tips, fare) not the pre-calculated total
- **Production Recommendation:** 
  - Investigate with City of Chicago data provider
  - Implement data quality tests to flag discrepancies
  - Consider calculating trip_total as derived field: `fare + tips + tolls + extras`
  - Add validation rules in data ingestion pipeline
 
### Data Filters Applied

All models apply these quality filters in the staging layer:

**Completeness Filters:**
- ✅ Removed rows with NULL values in critical fields (taxi_id, trip_start_timestamp, trip_end_timestamp)
- ✅ Ensured all records have valid trip identifiers

**Validity Filters:**
- ✅ Filtered out negative or zero values for:
  - `trip_seconds` (must be > 0)
  - `trip_miles` (must be > 0)
  - `fare` (must be >= 0)
  - `tips` (must be >= 0)
- ✅ Removed logical impossibilities (e.g., trips ending before they started)

**Consistency Filters:**
- ✅ **Deduplicated by unique_key** using SELECT DISTINCT
- ✅ Ensured trip_end_timestamp >= trip_start_timestamp

**Temporal Filters:**
- ✅ Limited to 2021-2023 data (most recent 3 complete years)
- ✅ Provides sufficient history for trend analysis while maintaining query performance

**Impact of Filters:**
- Original records (2023): 12,901,871 million
- After quality filters: 5,776,365 million (45% retention rate)
- Records removed: ~7,125,506 million (55% rejection rate)

### Assumptions & Definitions

**Overworker Definition:**
- **Insufficient break:** Less than 8 hours rest time between end of one trip and start of next trip
- **Rationale:** 8 hours aligns with standard sleep requirements and common labor regulations
- **Long shift:** Single continuous trip lasting more than 10 hours
- **Rationale:** 10+ hours of continuous driving poses significant safety and fatigue risks
- **Regular overworker:** Taxi meeting at least ONE of these criteria:
  - 10+ instances of insufficient breaks in Q4 2023, OR
  - 5+ long shifts (>10 hours) in Q4 2023
- **Rationale:** Occasional violations may be circumstantial; regular pattern indicates systemic issue

**Holiday Definition:**
- **Included:** Federal holidays plus commonly observed days
  - New Year's Day, Independence Day, Thanksgiving, Christmas
  - Day after Thanksgiving (Black Friday) - widely observed
  - Observed dates when holidays fall on weekends
- **Excluded:** 
  - Minor holidays (Presidents Day, Columbus Day, etc.)
  - Religious holidays not federally recognized
  - Local Chicago-specific holidays
- **Rationale:** Focused on holidays with nationwide recognition and likely business impact

**"Last 3 Months" Definition:**
- **Defined as:** Q4 2023 (October 1 - December 31, 2023)
- **Rationale:** Most recent complete quarter in available dataset
- **Alternative considered:** Rolling 90 days from Dec 31, 2023 (would be Oct 2 - Dec 31)
- **Decision:** Used calendar quarter for cleaner date boundaries and easier interpretation

**Trip Duration Calculation:**
- **Formula:** `trip_seconds / 3600` to convert seconds to hours
- **Assumption:** trip_seconds represents actual driving/occupied time, not total time taxi is active
- **Note:** Does not include idle time, refueling, or breaks between trips

---

## Looker Studio Dashboard

**📊 Public Dashboard Link:** https://lookerstudio.google.com/reporting/fe3148f5-7261-45b1-bae8-5cb32a17ef94
