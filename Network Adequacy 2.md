# Network Adequacy ETL Pipeline

## Overview

The Network Adequacy ETL pipeline processes issuer-submitted files and national provider data to produce datasets for the network adequacy calculation engine and dashboard reporting. The pipeline consists of three core ETL processes that transform raw submissions into geocoded, normalized datasets ready for compliance analysis.

## Data Flow

```mermaid
flowchart TD
    subgraph ROW1 [ ]
        direction LR
        CONSUMER[Consumer Data]
        PROVIDER[Provider Data]
        CRITERIA[Criteria Data]
    end

    OSM[US OSM Data]

    subgraph ROW2 [ ]
        direction LR
        PAIR[Determine Pair:<br/>Pair consumers with nearby providers]
        EXTRACT[Road Network Extraction:<br/>Extract state road network]
    end

    GRAPH[(Road Network Graph)]
    NODE[(Graph Nodes)]
    
    ROUTER[Router Calculation:<br/>Compute shortest paths & distances]
    
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

    linkStyle default stroke:#90a4ae,stroke-width:1.5px
```

## Input Sources

### Primary Inputs: 

1. **"Providers Data"** (Provider network information)
   - Provider NPIs, names, addresses, and specialty codes

2. **"Consumer Data"** (Service area coverage)
   - Geographic coverage definitions (full county or ZIP-code-based)

3. **"Criteria Data"** (Service area coverage)
   - Drive time and distance requirements for provider specialty criteria and county type
   - CDE_COUNTY, Criteria, TME_DRVE, DISTANCE


### Supporting Data Sources

- **Administrative Boundaries**: US counties shapefile for spatial analysis


---

## Network Adequacy Calculation Engine (Downstream)
```bash
cd scripts
python Determine_Pairs_PA.py
python Road_Network_Extraction_PA.py
python Route_Calculation_PA.py
```


**Outputs**:
- Network adequacy compliance analysis results
- Time/distance calculations by specialty and geography
- Files for Tableau dashboard visualization
- Regulatory compliance reports

---

### Determine_Pairs_PA.py
   - Find consumer and provider pair that lies within the criteria distance.
   - Update all provider and consumer pair within all unique distance and criteria data in the postgres database in table: *Coordinate_pairs_pa*

### Road_Network_Extraction_PA.py
   - Extract the road network for the specified state.
   - Store the network graph as a serialized (pickle) file and the associated nodes dataset in Parquet format.
   - Graph_pa.pkl: Contains the extracted state road network graph
   - Graph_nodes_pa.parquet: Contains the nodes of the extracted state road network

### Route_Calculation_PA.py;
   - Compute shortest paths and distances using the road network graph and nodes.
   - Update the computed results in the postgres database table *Coordinate_pairs_pa*

