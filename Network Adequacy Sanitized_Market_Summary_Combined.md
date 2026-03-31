# Network Adequacy Engine Pipeline Overview

## Overview
TODO
The Network Adequacy Engine pipeline processes issuer-submitted and provider data to generate distance-based insights for compliance analysis and reporting. It is organized into three core engines that identify eligible consumer–provider pairs, extract the road network, and compute shortest path distances to support network adequacy evaluation.

## Data Flow

```mermaid
graph TD
    subgraph INPUTS["Input Data Sources"]
        A1[Criteria Files<br/>CRITERIA_SPEC</br>CRITERIA_TYPE]
        A2[Issuer Networks Data]
        A3[Recipients/Population Data]
        A4[Provider Data]
        A5[(PostgreSQL DB<br/>coordinate_pairs table)]
    end

    subgraph PROCESSING["ETL Processing"]
        P1[Data Preparation<br/>Merge criteria, networks, recipients, providers]
        P2[Pair Generation<br/>Create consumer-provider pairs with criteria]
        P3[Market Analysis<br/>County-level access calculations]
        P4[Issuer Analysis<br/>Per-network coverage metrics]
        P5[Fillable Gap Indicator Calculation<br/>Identify fillable coverage gaps]
        P6[Time/Distance Metrics<br/>Average travel times by coverage status]
    end

    subgraph OUTPUTS["Output Reports"]
        O1[<strong>Market Summary:</strong><br/>CVS & PKL]
        O2[<strong>Issuer View Dashboard Summary:</strong><br/>CSV & PKL]
        O3[<strong>Executive Dashboards:</strong><br/>CSV]
        O4[<strong>Fillable Gap Results:</strong><br/>CSV & PKL]
        O5[<strong>Avg Time/Distance Report:</strong><br/>CSV]
        O6[<strong>Network Excel Reports</strong><br/><strong>ZIP with formatted workbooks</strong>]
    end


    subgraph ROW1 [ ]
        direction LR
        CONSUMER[Consumer Data]
        PROVIDER[Provider Data]
        CRITERIA[Criteria Data]
    end

    OSM[US OSM Data]

    subgraph ROW2 [ ]
        direction LR
        PAIR[<strong>Determine Pair Engine:</strong><br/>Pair consumers with nearby providers]
        EXTRACT[<strong>Road Network Extraction Engine:</strong><br/>Extract state road network]
    end

    GRAPH[(Road Network Graph)]
    NODE[(Graph Nodes)]
    
    ROUTER[<strong>Router Calculation Engine:</strong><br/>Compute shortest paths & distances]
    
    OUTPUT[(Update Shortest Paths & Distances in PostgreSQL)]

    CONSUMER --> PAIR
    PROVIDER --> PAIR
    CRITERIA --> PAIR
    CRITERIA --> EXTRACT
    OSM --> EXTRACT
    EXTRACT --> GRAPH
    EXTRACT --> NODE
    PAIR --> ROUTER
    GRAPH --> ROUTER
    NODE --> ROUTER
    CONSUMER --> ROUTER
    ROUTER --> OUTPUT


    A1 --> P1
    A2 --> P1
    A3 --> P1
    A4 --> P1
    OUTPUT --> P3
    OUTPUT --> P4
    OUTPUT --> P6

    P1 --> P2
    P2 --> P3
    P2 --> P4
    
    P3 --> O1
    P4 --> O2
    P4 --> O3
    P4 --> P5
    P5 --> O4
    P6 --> O5
    
    O1 --> O6
    O2 --> O6
    O3 --> O6
    O4 --> O6
    O5 --> O6

    style ROW1 fill:none,stroke:none
    style ROW2 fill:none,stroke:none

    style CONSUMER fill:#bbdefb,stroke:#0d47a1,stroke-width:2px,color:#000
    style PROVIDER fill:#bbdefb,stroke:#0d47a1,stroke-width:2px,color:#000
    style CRITERIA fill:#bbdefb,stroke:#0d47a1,stroke-width:2px,color:#000
    style OSM fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px,color:#000
    style PAIR fill:#e1bee7,stroke:#6a1b9a,stroke-width:2px,color:#000
    style EXTRACT fill:#e1bee7,stroke:#6a1b9a,stroke-width:2px,color:#000
    style GRAPH fill:#f8bbd9,stroke:#c2185b,stroke-width:2px,color:#000
    style NODE fill:#f8bbd9,stroke:#c2185b,stroke-width:2px,color:#000
    style ROUTER fill:#e1bee7,stroke:#6a1b9a,stroke-width:2px,color:#000
    style OUTPUT fill:#fff9c4,stroke:#f57f17,stroke-width:2px,color:#000

    classDef input fill:#e1f5fe,stroke:#01579b,color:#000000
    classDef process fill:#e8f5e9,stroke:#1b5e20,color:#000000
    classDef output fill:#f3e5f5,stroke:#4a148c,color:#000000
    
    class A1,A2,A3,A4,A5 input
    class P1,P2,P3,P4,P5,P6 process
    class O1,O2,O3,O4,O5,O6 output
```

## Input Sources

### Primary Inputs: 

1. Criteria Files
   * CRITERIA_SPEC: 20250417_T_NET_ADQ_CRITERIA_SPEC_PA.csv 
   * CRITERIA_TYPE: 20250417_T_NET_ADQ_CRITERIA_TYPE_PA.csv <br/> Columns: 'GRP_CDE_SPECLTY_PRIM','DISPLAY_CAT','CDE_SPECLTY_PRIM','TYPE_SPECLTY_PRIM','NAM_CRITERIA'
   * Merge CRITERIA_SPEC and CRITERIA_TYPE on 'NAM_CRITERIA' and finally selecting 'NAM_CRITERIA','CDE_SPECLTY_PRIM','criteria_category' columns as input for Data Preparation step.

2. Issuer Network Data: 
   * For each individual issuer (issuer_identifier_number) take the first non duplicated data.
   * Columns: ['period','issuer_name','issuer_identifier_number','any_dental','Updated File Name','PID_network_name']
   * Table: 20250815_Network_Information_Report.xlsx <br/> sheet_name='Network Information Report'

3. Recipients/Population Data
   * Read csv file
   * Columns: [consumer_id','zip', 'countyssa', 'countyfips']
   * Table: 20250811_QHP_Sample_Population_PY26_PA_Final_v3.csv

4. Provider Data
   * Table: 20251114_PID_Providers.csv

### Supporting Data Sources

- **Postgres Database**: Database 


---

## Data Preparation
* Prepare data for the further analysis by merging the respctive input files

## Market Analysis Metrics
* all_summaries_market_final_{model run date}.pkl
* gap_report_results_all_market_final_{model run date}.csv

## Issuer Analysis
* gap_report_results_all_{model run date}.csv 
* all_summaries_{model run date}.pkl
* {model run date}_PID_Network_Adequacy_Executive_Dashboard_Data.csv

## Fillable Gap Indicator Calculation
* fillable_gap_results_final_{model run date}.csv
* fillable_gap_results_final_{model run date}.pkl

## Time/Distance Metrics
* avg_time_distance_report_{model run date}.csv




**Outputs**:
* 20251205_Summary_Reports.zip
* Updated_Gap_Reports.zip