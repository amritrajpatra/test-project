# Network Adequacy ETL Pipeline

## Overview

The Network Adequacy ETL pipeline processes issuer-submitted files and national provider data to produce datasets for the network adequacy calculation engine and dashboard reporting. The pipeline consists of three core ETL processes that transform raw submissions into geocoded, normalized datasets ready for compliance analysis.

## Data Flow

```mermaid
flowchart TD
    subgraph INPUT [📥 INPUT DATA SOURCES]
        direction LR
        C[Consumer Data]
        P[Provider Data]
        CR[Criteria Data]
    end

    subgraph PAIRING [🔗 DETERMINE PAIR PA]
        A[Pair consumers with<br/>nearby providers]
    end

    subgraph NETWORK [🗺️ ROAD NETWORK EXTRACTION PA]
        B[Extract state road<br/>network data]
        G[(graph_pa.pkl)]
        N[(graph_nodes_pa.parquet)]
        B --> G
        B --> N
    end

    subgraph ROUTING [⚡ ROUTER CALCULATION PA]
        R[Compute shortest paths & distances<br/>Update PA coordinates]
    end

    subgraph OUTPUT [💾 OUTPUT]
        T[(coordinate_pairs_pa)]
    end

    INPUT --> PAIRING
    PAIRING --> ROUTING
    NETWORK --> ROUTING
    C -.->|Consumer data input| ROUTING
    
    ROUTING --> OUTPUT

    style INPUT fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#1a237e
    style PAIRING fill:#fff3e0,stroke:#f57c00,stroke-width:2px,color:#bf360c
    style NETWORK fill:#e8f5e9,stroke:#388e3c,stroke-width:2px,color:#1b5e20
    style ROUTING fill:#e0f2fe,stroke:#0284c7,stroke-width:2px,color:#0c4a6e
    style OUTPUT fill:#fff8e1,stroke:#ffa000,stroke-width:2px,color:#e65100

    style C fill:#90caf9,stroke:#1976d2,stroke-width:1.5px,color:#1a237e,font-weight:500
    style P fill:#90caf9,stroke:#1976d2,stroke-width:1.5px,color:#1a237e,font-weight:500
    style CR fill:#90caf9,stroke:#1976d2,stroke-width:1.5px,color:#1a237e,font-weight:500
    style A fill:#ffe0b2,stroke:#f57c00,stroke-width:1.5px,color:#bf360c,font-weight:500
    style B fill:#c8e6c9,stroke:#388e3c,stroke-width:1.5px,color:#1b5e20,font-weight:500
    style G fill:#a5d6a7,stroke:#2e7d32,stroke-width:1.5px,color:#1b5e20,font-family:monospace
    style N fill:#a5d6a7,stroke:#2e7d32,stroke-width:1.5px,color:#1b5e20,font-family:monospace
    style R fill:#bae6fd,stroke:#0284c7,stroke-width:1.5px,color:#0c4a6e,font-weight:500
    style T fill:#ffecb3,stroke:#ffa000,stroke-width:1.5px,color:#e65100,font-weight:500,font-family:monospace

    linkStyle default stroke:#90a4ae,stroke-width:2px,fill:none
    linkStyle 4 stroke:#ffa000,stroke-width:2px,stroke-dasharray:5,fill:none
```

## Input Sources

### Primary Inputs: 

1. **Providers Data** (Provider network information)
   - Provider NPIs, names, addresses, and specialty codes

2. **"Consumer Data** (Service area coverage)
   - Geographic coverage definitions (full county or ZIP-code-based)

3. **"Criteria Data** (Service area coverage)
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

### Route_Calculation_PA.py;
   - Compute shortest paths and distances using the road network graph.
   - Update the computed results in the database table *Coordinate_pairs_pa*

