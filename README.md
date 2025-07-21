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

- 1. Subject - Sending Institution (Remittance Providers)
- 2. Subject - Orders (Order Full Lifecycle Funnel Analysis, From institution placing the order → Order Stage 1 → Order Stage 2 → Order Stage 3 → ends with Payment Success or Failure.)
- 3. Subject - Users （Sender & Payee）


## 3. ToB Business - Cross-border E-commerce Collection and Payment

Provide offshore accounts (Shopee official wallet) and fund repatriation services for Shopee cross-border sellers based in Mainland China, Hong Kong, and South Korea.

Background: Under the standard collection model, Shopee currently only supports local settlement of sales proceeds—meaning funds from sold goods can only be settled into local overseas bank accounts. However, for cross-border sellers, it is generally not feasible to open overseas bank accounts due to high entry barriers and operational costs. As a result, sellers face challenges in receiving payments and difficulties in capital circulation.

| No. | Business Process                    | Description                                                                                  |
|-----|-------------------------------------|----------------------------------------------------------------------------------------------|
| 1   | **Merchant Onboarding**            | Merchant registers on the platform and completes kyc.   |
| 2   | **Merchant Shop Binding**         | Merchant links their shops. |
| 3   | **Funds Inflow (E-commerce Top-up)** | E-commerce Top-up |
| 4   | **Funds Payout (Disbursement & Deduction)** | The platform processes payouts or automatic deductions on behalf of the merchant (e.g. platform fees, commission). |
| 5   | **Merchant Shop Card Binding**          | Merchant binds a settlement bank card for receiving withdrawals.     |
| 6   | **Merchant Operations (e.g., Annual Subscription Plan)** | Merchant performs business-related actions such as purchasing subscription plans or value-added services. |
| 7   | **Withdrawal and Payment**         | Merchant initiates fund withdrawal to their own bank account or makes payments to external suppliers. |

### 3.1 Merchant Subject Table

| Category                                | Field Name                          | Data Type | Description                                                                 |
|-----------------------------------------|--------------------------------------|-----------|-----------------------------------------------------------------------------|
| **Partition Field**   | fdate                       | BIGINT  | Partition date                                                              |
| **Primary Key**       | fgid                        | STRING  | Merchant GID (Global ID)                                                   |
| **Primary Key**       | fspid                       | STRING  | Merchant SPID (Sub-platform ID)                                            |
| **Merchant Basic Attributes** | fcompany_name       | STRING  | Company name                                                                |
| **Time-based**    | fkyc_first_submit_time          | STRING  | First KYC submission time                                                   |
| **Time-based**    | fkyc_first_approved_time        | STRING  | First KYC approval time                                                     |
| **Time-based** | fshop_apply_time                   | STRING  | Store application time                                                      |
| **Time-based** | fshop_first_bind_time              | STRING  | First store binding time                                                    |
| **Time-based** | fcard_first_bind_time              | STRING  | First card binding time                                                     |
| **Time-based** | ffirst_disbursement_time           | STRING  | First disbursement time (funds distributed on behalf of merchant)          |
| **Time-based** | ffirst_withdraw_time               | STRING  | First withdrawal to merchant bank account                                  |
| **Time-based** | ffirst_payment_time                | STRING  | First payment to external supplier                                         |
| **Time-based** | fsubs_plan_first_buy_time          | STRING  | First annual plan purchase time                                             |
| **Time-based** | fsubs_plan_first_use_time          | STRING  | First annual plan usage time                                                |
| **Vertical - Tag** | fsite_count                    | BIGINT  | Number of sites (e.g., Shopee-TW, Shopee-SG)                               |
| **Vertical - Tag** | fshop_count                    | BIGINT  | Number of stores bound to merchant                                         |
| **Vertical - Tag** | faccount_count                 | BIGINT  | Number of accounts under this merchant                                     |
| **Vertical - Tag** | fpayee_count                   | BIGINT  | Unique payee count (withdrawal or supplier payments)                       |
| **Vertical - Tag** | fpayee_count_30d               | BIGINT  | Payee count in the last 30 days                                            |
| **Vertical - Statistical** | ftrd_cnt_month                 | BIGINT  | Total transaction count this month                                         |
| **Vertical - Statistical** | ftrd_cnt_year                  | BIGINT  | Total transaction count this year                                          |
| **Vertical - Statistical** | flast_disbursement_amount_cny_1d    | DOUBLE    | Disbursement amount in CNY (today)                                         |
| **Vertical - Statistical** | flast_disbursement_amount_usd_1d    | DOUBLE    | Disbursement amount in USD (today)                                         |
| **Vertical - Statistical** | flast_disbursement_amount_cny_28d   | DOUBLE    | Disbursement amount in CNY (last 28 days)                                  |
| **Vertical - Statistical** | flast_disbursement_amount_usd_28d   | DOUBLE    | Disbursement amount in USD (last 28 days)                                  |
| **Vertical - Statistical** | ftrd_amt_month                      | DOUBLE    | Total transaction amount this month                                        |
| **Vertical - Statistical** | ftrd_amt_year                       | DOUBLE    | Total transaction amount this year                                         |
| **Vertical - Statistical** | fmax_trd_amt_month                  | DOUBLE    | Max single transaction amount this month                                   |
| **Vertical - Statistical** | fmax_trd_amt_year                   | DOUBLE    | Max single transaction amount this year                                    |
| **Lifecycle Tag** | fmerchant_lifecycle_tag   | BIGINT | Merchant lifecycle status tag:<br>1. Not disbursed<br>2. New<br>3. Retained<br>4. Lost<br>5. Recovered<br>0. Default |
