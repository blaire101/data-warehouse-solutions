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

## 2. ToC Business - Global Remittances to China

Partner with overseas remittance providers (e.g. Panda Remit, Wise) to bring foreign currency into China  

> This diagram provides a basic outline, — the actual **information flow** and **funds flow** is much more complex.

```mermaid
flowchart LR

%% Node styles
classDef user fill:#D1C4E9,stroke:#673AB7,stroke-width:2px;
classDef product fill:#C8E6C9,stroke:#388E3C,stroke-width:2px;
classDef infra fill:#FFF9C4,stroke:#FBC02D,stroke-width:2px;
classDef spacer fill:none,stroke:none;

%% Subgraph background styles
style overseasBox fill:#EBF5FB,stroke:#85C1E9,stroke-width:2px
style chinaBox fill:#FEF9E7,stroke:#F7DC6F,stroke-width:2px

%% Nodes
Sender["Sender"]:::user
SI["Sending Institution - SI"]:::product
API["Remittance Services - API"]:::infra
RI["Receiving Institution - RI"]:::product
Recipient["Recipient"]:::user

%% Subgraph: Overseas
subgraph overseasBox["Overseas or HK"]
    direction LR
    Sender -->|Initiate Transfer<br>Make Payment| SI
    SI -->|Forward Transfer<br>Prefund| API
    OV_SPACER1[" "]:::spacer
end

%% Subgraph: Onshore China
subgraph chinaBox["Onshore China"]
    direction LR
    API -->|Forward Transfer<br>Settlement| RI
    RI -.->|Notify| Recipient
    CN_SPACER1[" "]:::spacer
end
```

**Subject-Specifc Analysis model**

> covering `Sending Institution (Remittance Providers)`, `Orders`, and `Users`.

1. **Remittance Provider** : Analysed key metrics (countries, currencies and transaction, user volumes, fees, FX income ...), produced reports of transaction & profits to guide improvements.
2. **Orders** : 3 stage life-cycle funnel to measure conversion rates and bottleneck. - Order status: A: Provider order | B: Account opening process | C:  Complete fund receipt.
3. **Users** : basic info, behaviour, life-cycle, preferences for targeted campaigns.

| No. | Business Processes  | Description |
|:---:|---------------------|-------------|
| pre 1   | Partner Onboarding  | Partners complete a onboarding process. Risk & compliance teams perform due diligence. |
| pre 2   | Institution Funding | Partners pre-fund a designated account to ensure sufficient liquidity for remittance. |
| pre 3   | Currency Exchange   | Based on settlement needs, foreign currency is converted into RMB. |
| 4   | Remittance Service | End-users initiate remittance via the provider's app by submitting sender and recipient info.<br>1. If the recipient is new, an SMS prompts setup of a receiving card.<br>2. The provider calls the remittance API to submit the order.<br>3. Funds are routed into local settlement accounts. |
| 5   | Payment Collection  | Recipients collect RMB via digital wallets or linked bank-cards. |

## 3. ToB Business - Cross-border E-commerce Collection and Payment

> Background:
>
> - Under the standard collection model, Shopee currently only supports local settlement of sales proceeds—meaning funds from sold goods can only be settled into local overseas bank accounts.
> - However, for **cross-border sellers, it is not feasible to open overseas bank accounts**. As a result, sellers face challenges in **receiving payments and accessing their earnings freely**.
> - Ten-Pay is great at getting money back to China and distributing it efficiently.

So, Provide offshore accounts (AS Shopee official wallet) and **fund repatriation** services `[riːˌpætriˈeɪʃən]` for Shopee cross-border sellers based in Mainland China, Hong Kong, and South Korea.

```mermaid
graph TD
    %% Left side entities - Shopee above Shopee Bank Card
    Shopee(S Shopee)
    style Shopee fill:#FFA07A,stroke:#333,stroke-width:2px %% Adjusted to comfortable orange-yellow
    ShopeeBankCard([Shopee Bank Card])
    style ShopeeBankCard fill:#ccf,stroke:#333,stroke-width:2px %% Keep light blue for the card

    %% Central green box - White-label Official Wallet
    subgraph SG_Wallet_T["White-label Wallet System"]
        direction LR %% Internal layout is more left-to-right
        style SG_Wallet_T fill:#98FB98,stroke:#333,stroke-width:2px %% Changed to light green
        SMA([Shopee Main Account])
        style SMA fill:#F0FFF0,stroke:#333,stroke-width:2px %% Keep very pale green
        SA_A([Seller Account A])
        style SA_A fill:#F0FFF0,stroke:#333,stroke-width:2px %% Keep very pale green
        SA_B([Seller Account B])
        style SA_B fill:#F0FFF0,stroke:#333,stroke-width:2px %% Keep very pale green
        SA_C([Seller Account C])
        style SA_C fill:#F0FFF0,stroke:#333,stroke-width:2px %% Keep very pale green

        SMA -- "Funds distribute" --> SA_A
        SMA -- "Funds distribute" --> SA_B
        SMA -- "Funds distribute" --> SA_C
        linkStyle 0,1,2 stroke:#333,stroke-width:1px
    end

    %% Right blue box - Seller Bank Accounts
    subgraph SG_SellerBanks["Seller Bank Accounts"]
        direction TB %% Internal layout is top-to-bottom
        style SG_SellerBanks fill:#ADD8E6,stroke:#333,stroke-width:2px %% Changed to light blue
        SBA_A([Seller Bank Account A])
        style SBA_A fill:#F5F5DC,stroke:#333,stroke-width:2px %% Keep beige
        SBA_B([Seller Bank Account B])
        style SBA_B fill:#F5F5DC,stroke:#333,stroke-width:2px %% Keep beige
        SBA_C([Seller Bank Account C])
        style SBA_C fill:#F5F5DC,stroke:#333,stroke-width:2px %% Keep beige
    end

   %% Right blue box - Seller Product Supplier
    subgraph SZ_ProductSupplier["I"]
        direction TB %% Internal layout is top-to-bottom
        style SZ_ProductSupplier fill:#ADD8,stroke:#333,stroke-width:2px %% Changed to light blue
        SPS_A([Product Supplier])
        style SPS_A fill:#FF23,stroke:#333,stroke-width:2px %% Keep beige
    end

    %% Right blue box - Seller Annual Subscription Plan
    subgraph Subs["I"]
        direction TB %% Internal layout is top-to-bottom
        style Subs fill:#ADD5,stroke:#333,stroke-width:1px %% Changed to light blue
        SASP_C([annual membership])
        style SASP_C fill:#F5F5,stroke:#333,stroke-width:1px %% Keep beige
    end

    %% Connections between main sections
    ShopeeBankCard --> |E-commerce Top-up| SMA
    linkStyle 3 stroke:#333,stroke-width:1px,color:#000

    SMA --> |balance can be withdrawn to bank account| ShopeeBankCard
    linkStyle 4 stroke:#333,stroke-width:1px,color:#000

    SA_A --> |Withdrawal| SBA_A
    SA_A --> |Payment| SPS_A
    SA_C --> |Subscription| SASP_C
    SA_B --> |Withdrawal| SBA_B
    SA_C --> |Withdrawal| SBA_C
    linkStyle 5,6,7 stroke:#333,stroke-width:1px
```

| No. | Business Process                    | Description                                                                                  |
|-----|-------------------------------------|----------------------------------------------------------------------------------------------|
| 1   | **Merchant Onboarding**            | Merchant registers on the platform and completes kyc.   |
| 2   | **Merchant Shop Binding**         | Merchant links their shops. |
| 3   | **Funds Inflow (E-commerce Top-up)** | E-commerce Top-up |
| 4   | **Funds Payout (Disbursement & Deduction)** | The platform processes payouts or automatic deductions on behalf of the merchant (e.g. platform fees, commission). |
| 5   | **Merchant Shop Card Binding**          | Merchant binds a settlement bank card for receiving withdrawals.     |
| 6   | **Merchant Operations (e.g., Annual Subscription Plan)** | Merchant performs business-related actions such as purchasing subscription plans or value-added services. |
| 7   | **Withdrawal and Payment**         | Merchant initiates fund withdrawal to their own bank account or makes payments to external suppliers. |

**Subject-Specifc Analysis model**, covering `Merchant`, `Shop`, and `Orders`.

### Merchant Subject Table

> Shopee's official wallet business leverages multi-dimensional data analysis to support merchant lifecycle management, transaction insights, and revenue optimization. From churn monitoring to cross-site transaction trend analysis, comprehensive dashboards and thematic tables provide strong data support for business growth, product experience enhancement, and precision operations.

| --Category-- | Field Name | Data_Type | Description |
|-----------------------------------------|--------------------------------------|-----------|-----------------------------------------------------------------------------|
| **Partition Field**   | fdate                       | BIGINT  | Partition date                                                              |
| **Primary Key**       | fgid                        | STRING  | Merchant GID (Global ID)                                                   |
| **Primary Key**       | fspid                       | STRING  | Merchant SPID (Sub-platform ID)                                            |
| **Merchant_Basic_Info** | fcompany_name       | STRING  | Company name                                                                |
| **Horizontal_Time** | fkyc_first_submit_time          | STRING  | First KYC submission time                                                   |
| **Horizontal Time** | fkyc_first_approved_time        | STRING  | First KYC approval time                                                     |
| **Horizontal Time** | fshop_apply_time                   | STRING  | Store application time                                                      |
| **Horizontal Time** | fshop_first_bind_time              | STRING  | First store binding time                                                    |
| **Horizontal Time** | fcard_first_bind_time              | STRING  | First card binding time                                                     |
| **Horizontal Time** | ffirst_disbursement_time           | STRING  | First disbursement time (funds distributed on behalf of merchant)          |
| **Horizontal Time** | ffirst_withdraw_time               | STRING  | First withdrawal to merchant bank account                                  |
| **Horizontal Time** | ffirst_payment_time                | STRING  | First payment to external supplier                                         |
| **Horizontal Time** | fsubs_plan_first_buy_time          | STRING  | First annual plan purchase time                                             |
| **Horizontal Time** | fsubs_plan_first_use_time          | STRING  | First annual plan usage time                                                |
| **Vertical - Tag** | fsite_count                    | BIGINT  | Number of sites (e.g., Shopee-TW, Shopee-SG)                               |
| **Vertical - Tag** | fshop_count                    | BIGINT  | Number of stores bound to merchant                                         |
| **Vertical - Tag** | faccount_count                 | BIGINT  | Number of accounts under this merchant                                     |
| **Vertical - Tag** | fpayee_count                   | BIGINT  | Unique payee count (withdrawal or supplier payments)                       |
| **Vertical - Tag** | fpayee_count_30d               | BIGINT  | Payee count in the last 30 days                                            |
| **Vertical - Calc** | ftrd_cnt_month                 | BIGINT  | Total transaction count this month                                         |
| **Vertical - Calc** | ftrd_cnt_year                  | BIGINT  | Total transaction count this year                                          |
| **Vertical - Calc** | flast_disbursement_amount_cny_1d    | DOUBLE    | Disbursement amount in CNY (today)                                         |
| **Vertical - Calc** | flast_disbursement_amount_usd_1d    | DOUBLE    | Disbursement amount in USD (today)                                         |
| **Vertical - Calc** | flast_disbursement_amount_cny_28d   | DOUBLE    | Disbursement amount in CNY (last 28 days)                                  |
| **Vertical - Calc** | flast_disbursement_amount_usd_28d   | DOUBLE    | Disbursement amount in USD (last 28 days)                                  |
| **Vertical - Calc** | ftrd_amt_month                      | DOUBLE    | Total transaction amount this month                                        |
| **Vertical - Calc** | ftrd_amt_year                       | DOUBLE    | Total transaction amount this year                                         |
| **Vertical - Calc** | fmax_trd_amt_month                  | DOUBLE    | Max single transaction amount this month                                   |
| **Vertical - Calc** | fmax_trd_amt_year                   | DOUBLE    | Max single transaction amount this year                                    |
| **Lifecycle Tag** | fmerchant_lifecycle_tag   | BIGINT | Merchant lifecycle status tag:<br>1. Not disbursed<br>2. New<br>3. Retained<br>4. Lost<br>5. Recovered<br>0. Default |

### Orders Subject Table

| Description | Field Name | Type | Remarks |
|-------------|------------|------|---------|
| **Partition** | fdate | BIGINT | Date partition field |
| **Primary Key** | Ftransaction_scene | BIGINT | 1: Collection Transaction Scene (Top-up to Ten-HK )<br>2: Disbursement Scene – Disbursement <br> 3: Payment Scene (Withdrawal / Payment / Subs Plan) |
| **Primary Key** | Flistid | STRING | Order ID |
| transaction_scene | Ftransaction_scene_type | trans_type | 1: Collection<br> 2: Disbursement<br> 3: Withholding<br>4: Withdrawal<br>5: Payment<br>6: Subs Plan |
| Merchant SPID | fspid | STRING | Used to join with merchant dimension table |
| - | fsite_id | STRING | One seller may have multiple sites |
| - | fshop_id | STRING | Present only in Disbursement Scene; ignored in Payment Scene |
| **Pay_Transaction** <br> (Withdrawal/Pay/Subs) | fpayee_id | STRING | Applicable in payment scenarios |
| **Pay_Transaction** | fpayee_type | BIGINT | Domestic: 1 - Personal Bank Account, 2 - Corporate Account<br>Overseas: 1 - Same-name Account, 2 - Supplier Account |
| **Pay_Transaction** | fbiz_type | BIGINT | 1: FX purchase inbound (domestic)<br>2: FX purchase payment (overseas)<br>3: FX payment (overseas)<br>4: Annual Subs |
| **Pay_Transaction** | Fsell_cur_type | STRING | Outgoing currency, ISO 4217 format |
| **Pay_Transaction** | Fbuy_cur_type | STRING | Incoming currency, ISO 4217 format |
| **Pay_Transaction**  | Fbank_country | STRING | Destination country of funds |
| **Pay_Transaction**  | Fproduct_code | STRING | Product code, used in annual card purchase |
| **Pay_Transaction**  | Fbiz_fee_cur_type | STRING | Currency of transaction fee |
| **Pay_Transaction**  | Fbiz_fee_amount | BIGINT | Transaction fee in original currency (unit: yuan) |
| **Pay_Transaction**  | Fbiz_fee_amount_usd | BIGINT | Fee amount (USD) |
| **Pay_Transaction** <br> (Withdrawal/Pay/Subs) | Fbiz_fee_amount_cny | BIGINT | Transaction fee converted to CNY |
| - | - | - |
| **General Transaction** | Fcur_type | STRING | Transaction currency |
| **General Transaction** | Famount | BIGINT | Transaction amount in original currency (unit: yuan) |
| **General Transaction** | Famount_usd | BIGINT | Converted amount in USD |
| **General Transaction** | Famount_cny | BIGINT | Converted amount in CNY |
| **General Transaction** | Ftransaction_initiation_time | STRING | Time when the transaction was initiated |

### Dashboard
