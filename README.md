# Data Warehouse Solutions

> 🛡️ **Disclaimer:**  
> The following content represents generalized industry knowledge and anonymized case practices.  
> It does **not contain any confidential, proprietary, or internal information** from any specific company.

---

## 1. Data Warehouse Architecture 

Built a layered data warehouse (ODS > DIL > DML > DAL) to ingest, clean, and transform data into fact and dimension tables. Defined data domains, granularity, metrics, and embedded business logic for subject-oriented, multi-dimensional analysis.

<div align="center">
  <img src="docs/dwh-1.jpg" alt="Diagram" width="700">
</div>

**Data Domains**

| No. | Domain Name    | Description |
|-----|----------------|-------------|
| 1   | Customer (USR) | Covers individuals and merchants, including user info, identity data, and credit profiles. |
| 2   | Transaction (TRD) | Order lifecycle, including creation, payment, completion, and closure. |
| 3   | Event (EVT)    | Risk signals, marketing campaigns, click logs, etc. |
| …   | …              | … |

---

## 2. ToC Business - Global Remittances to China

Partner with overseas remittance providers (e.g. Panda Remit, Wise) to bring foreign currency into China  

```mermaid
flowchart LR
    %% Style definitions
    classDef user      fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px
    classDef product   fill:#FFEBEE,stroke:#C62828,stroke-width:2px
    classDef infra     fill:#E3F2FD,stroke:#1565C0,stroke-width:2px

    classDef boxOverseas fill:#E0F7FA,stroke:#00838F,stroke-width:3px
    classDef boxDomestic fill:#FCE4EC,stroke:#AD1457,stroke-width:3px

    %% Actual nodes
    Sender["Sender"]:::user
    SI["Sending Institution - SI"]:::product
    TRS["Remittance Services - TRS"]:::infra

    RI["Receiving Institution - RI"]:::product
    Recipient["Recipient"]:::user

    %% Flow lines
    Sender -->|initiate transfer <br> make payment| SI
    SI -->|Forward Transfer <br> Prefund | TRS
    TRS -->|Forward Transfer <br> Settlement| RI
    RI -.->| Notify| Recipient

    %% Invisible bounding boxes for region grouping
    subgraph Overseas [Overseas]
        direction LR
        O1[ ]:::boxOverseas
        Sender
        SI
        TRS
        O2[ ]:::boxOverseas
    end

    subgraph "Onshore China"
        direction LR
        D1[ ]:::boxDomestic
        RI
        Recipient
        D2[ ]:::boxDomestic
    end
```

| No. | Business Processes  | Description |
|:---:|---------------------|-------------|
| pre 1   | Partner Onboarding  | Partners complete a onboarding process. Risk & compliance teams perform due diligence. |
| pre 2   | Institution Funding | Partners pre-fund a designated account to ensure sufficient liquidity for remittance. |
| 3   | Currency Exchange   | Based on settlement needs, foreign currency is converted into RMB. |
| 4   | Remittance Service | End-users initiate remittance via the provider's app by submitting sender and recipient info.<br>1. If the recipient is new, an SMS prompts setup of a receiving card.<br>2. The provider calls the remittance API to submit the order.<br>3. Funds are routed into local settlement accounts. |
| 5   | Payment Collection  | Recipients collect RMB via digital wallets or linked bank-cards. |

**Subject-Specifc Analysis model**, covering `Remittance Providers (Institution)`, `Orders`, and `Users`.

## 3. ToB Business - Cross-border E-commerce Collection and Payment

