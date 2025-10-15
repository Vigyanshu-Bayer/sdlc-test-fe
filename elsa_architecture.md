# ELSA System Architecture
 
## System Flow Diagram
 
```mermaid
graph TB
    subgraph "Data Source"
        SNB[Scientific Notebook]
    end
 
    subgraph "AWS Lambda Functions"
        EA[External Actions]
        EAA[External Actions Authorizer]
        FE[Fetch Experiment & Send for Ingest]
        EM[Extract Metadata]
        EP[Extract PDF]
        FMP[Fetch Missing PDF]
       
        subgraph "Consolidated Archive Service" %% Highlight new changes
            AS[archive_service.py]
            AS -->|Replaced| CD[cleanupData.py]
            AS -->|Replaced| IN[ingest.py]
            AS -->|Replaced| PR[pollResults.py]
            AS -->|Replaced| SI[searchIP.py]
        end
       
        EL[Error Logger]
        PM[Push Metadata]
    end
 
    subgraph "Storage & External Services"
        AD[Azure AD]
        S3[AWS S3]
        DB[DynamoDB]
        AS3[Azure Storage]
    end
 
    %% Data Flow
    SNB -->|Experiment Data| EA
    EA -->|Authorized Data| EAA
    EAA -->|Validated Data| FE
    FE -->|Raw Data| EM
    EM -->|Metadata| EP
    EP -->|PDF Data| AS
   
    %% Archive Service Flow
    AS -->|Authentication| AD
    AS -->|Store Files| AS3
    AS -->|Update Status| DB
    AS -->|Store Metadata| S3
   
    %% Error Handling
    EL -->|Log Errors| S3
    FMP -->|Recover Missing| EP
    PM -->|Store| S3
 
    classDef highlight fill:#f96,stroke:#333,stroke-width:4px;
    class AS,CD,IN,PR,SI highlight;
```
 
## Architecture Overview
 
1. **Data Ingestion Layer**
   - Scientific Notebook (SNB) serves as the primary data source
   - External Actions handle initial data reception and authorization
   - Fetch Experiment service prepares data for processing
 
2. **Processing Layer**
   - Extract Metadata service processes experiment metadata
   - Extract PDF service handles document extraction
   - Fetch Missing PDF service provides recovery capabilities
 
3. **Archive Layer (New Consolidated Service)**
   - `archive_service.py` now handles multiple functions previously split across:
     * Data ingestion (formerly ingest.py)
     * Status monitoring (formerly pollResults.py)
     * IP search functionality (formerly searchIP.py)
     * Cleanup operations (formerly cleanupData.py)
   - Integrates directly with Azure Storage for file archival
   - Manages authentication through Azure AD
   - Updates status in DynamoDB
 
4. **Storage Layer**
   - AWS S3 for metadata and error logs
   - Azure Storage for archived files
   - DynamoDB for status tracking
 
5. **Monitoring & Error Handling**
   - Error Logger tracks issues across all services
   - Push Metadata service ensures data consistency
   - Status monitoring integrated into archive service
 
The new architecture simplifies the workflow by consolidating multiple services into a single archive service, reducing complexity and improving maintainability while maintaining all necessary functionality.
