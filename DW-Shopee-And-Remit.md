
## Data Warehouse How to Built

```mermaid
flowchart TB
  %% ============ Business Entities ============
  subgraph BIZ["Business Entities"]
    direction TB
    M[Merchant]:::biz
    S[Store]:::biz
    O[Order]:::biz
    VAM[Main VA - Real Bank Account]:::biz
    VAS[Sub-VA - Store and Currency]:::biz
    SUP[Supplier]:::biz
    CNBK[Bank Card in China]:::biz
  end

  %% ============ ODS ============
  ODS[ODS Layer]:::ods

  %% ============ DIL ============
  subgraph DIL["DIL Layer"]
    direction TB
    FACT[Fact Table]:::fact
  end

  %% ============ DIM ============
  subgraph DIM["DIM Layer"]
    direction TB
    DIM_T[Dim Table]:::dim
  end

  %% ============ DWS ============
  subgraph DWS["DWS - Subject Tables"]
    direction TB
    DWS_M[Merchant Subject]:::dws
    DWS_S[Store Subject]:::dws
    DWS_O[Order Subject]:::dws
  end

  %% ============ ADS ============
  ADS["ADS Layer<br>(tables for dashboards)"]:::ads

  %% ============ Mappings ============
  BIZ-->ODS
  ODS-->DIL
  ODS-->DIM
  DIL-->DWS
  DIM-->DWS
  DWS-->ADS
  DIM-->ADS

  %% ============ Styles ============
  classDef biz fill:#e0f2fe,stroke:#0284c7,stroke-width:2px,color:#075985;   %% light blue
  classDef ods fill:#f5f5f5,stroke:#424242,stroke-width:2px,color:#212121;   %% gray
  classDef dil fill:#ede9fe,stroke:#7c3aed,stroke-width:2px,color:#4c1d95;   %% purple
  classDef fact fill:#ddd6fe,stroke:#5b21b6,stroke-width:2px,color:#3730a3; %% darker purple
  classDef dim fill:#ccfbf1,stroke:#14b8a6,stroke-width:2px,color:#0f766e;  %% purple
  classDef dws fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#166534;  %% green
  classDef ads fill:#ffedd5,stroke:#ea580c,stroke-width:2px,color:#7c2d12;  %% orange
```


## 4. Other Info

**Shopee Official Wallet**

> In Shopee’s official wallet model, Shopee itself acts as the settlement entity. After sellers onboard and bind stores, Shopee credits their **official wallet account** (white-label offshore account powered by Tenpay).  
There is **no sub-VA per store** — store-level differentiation comes from Shopee’s internal transaction system. Funds can be disbursed (fees, supplier payments, subscription plans) or withdrawn to bank accounts.  

| No. | Amazon Standard Collection                           | Shopee Official Wallet                                      |
|-----|------------------------------------------------------|-------------------------------------------------------------|
| 1   | **Merchant Onboarding** – Merchant registers and KYC | **Merchant Onboarding** – Merchant registers and KYC        |
| 2   | **VA Assignment** – Main VA created, sub-VA per shop | **Shop Binding** – Merchant links their shops               |
| 3   | **Shop Authorization & Binding** – Sub-VA assigned   | **Funds Inflow (Top-up)** – Shopee credits merchant wallet  |
| 4   | **Amazon Pays Store VA** – Funds flow into sub-VA    | **Funds Flow & Deduction** – Payouts/deductions processed   |
| 5   | **Transaction Details via API** – Collect order data | **Merchant Card Binding** – Bank card linked for withdrawal |
| 6   | **Merchant Card Binding** – Settlement card binding  | **Payout - Withdrawal/Payment** – Merchant withdraws/pays   |
| 7   | **Withdrawal & Payout** – From Main VA to bank/supplier | **Payout - Merchant Ops** – e.g., annual subscription plan   |

### Subject-Specifc Table

**Subject-Specifc Analysis model**, covering `Merchant`, `Shop`, and `Orders`.

> Shopee's official wallet business leverages multi-dimensional data analysis to support merchant lifecycle management, transaction insights, and revenue optimization. From churn monitoring to cross-site transaction trend analysis, comprehensive dashboards and thematic tables provide strong data support for business growth, product experience enhancement, and precision operations.

</details>

<details>
<summary><strong style="color:#1E90FF;">Merchant Subject Sample - Data Metric</strong></summary>

| --Category-- | Field Name | Data_Type | Description |
|-----------------------------------------|--------------------------------------|-----------|-----------------------------------------------------------------------------|
| **Partition Field**   | fdate                       | BIGINT  | Partition date                                                              |
| **Primary Key**       | fgid                        | STRING  | Merchant GID (Global ID)                                                   |
| **Primary Key**       | fspid                       | STRING  | Merchant SPID (Sub-platform ID)                                            |
| **Merchant_Basic_Info** | fcompany_name       | STRING  | Company name       |
| **<mark>Time Funnel</mark>** |  -  |  -  |  -  |
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

</details>


<details>
<summary><strong style="color:#1E90FF;">Order Subject Sample - Data Metric</strong></summary>

| Description | Field Name | Type | Remarks |
|-------------|------------|------|---------|
| **Partition** | fdate | BIGINT | Date partition field |
| **Primary Key** | Ftransaction_scene | BIGINT | 1: Collection Transaction Scene (Top-up to Ten-HK )<br>2: Disbursement Scene – Disbursement <br> 3: Payout Scene (Withdrawal / Pay Supplier / Subs Plan) |
| **Primary Key** | Flistid | STRING | Order ID |
| transaction_scene | Ftransaction_scene_type | trans_type | 1: Collection<br> 2: Disbursement<br> 3: Withholding<br>4: Withdrawal<br>5: Payment<br>6: Subs Plan |
| Merchant SPID | fspid | STRING | Used to join with merchant dimension table |
| - | fsite_id | STRING | One seller may have multiple sites |
| - | fshop_id | STRING | Present only in Disbursement Scene; ignored in Payout Scene |
| **Payout** <br> (Withdrawal/Pay/Subs) | fpayee_id | STRING | Applicable in payout scenarios |
| **Payout** | fpayee_type | BIGINT | Domestic: 1 - Personal Bank Account, 2 - Corporate Account<br>Overseas: 1 - Same-name Account, 2 - Supplier Account |
| **Payout** | fbiz_type | BIGINT | 1: FX purchase inbound (domestic)<br>2: FX purchase payment (overseas)<br>3: FX payment (overseas)<br>4: Annual Subs |
| **Payout** | Fsell_cur_type | STRING | Outgoing currency, ISO 4217 format |
| **Payout** | Fbuy_cur_type | STRING | Incoming currency, ISO 4217 format |
| **Payout**  | Fbank_country | STRING | Destination country of funds |
| **Payout**  | Fproduct_code | STRING | Product code, used in annual card purchase |
| **Payout**  | Fbiz_fee_cur_type | STRING | Currency of transaction fee |
| **Payout**  | Fbiz_fee_amount | BIGINT | Transaction fee in original currency (unit: yuan) |
| **Payout**  | Fbiz_fee_amount_usd | BIGINT | Fee amount (USD) |
| **Payout** <br> (Withdrawal/Pay/Subs) | Fbiz_fee_amount_cny | BIGINT | Transaction fee converted to CNY |
| - | - | - |
| **General Transaction** | Fcur_type | STRING | Transaction currency |
| **General Transaction** | Famount | BIGINT | Transaction amount in original currency (unit: yuan) |
| **General Transaction** | Famount_usd | BIGINT | Converted amount in USD |
| **General Transaction** | Famount_cny | BIGINT | Converted amount in CNY |
| **General Transaction** | Ftransaction_initiation_time | STRING | Time when the transaction was initiated |

</details>


## 4. ToC Business - Global Remittances to China

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

| No. | Business Processes  | Description |
|:---:|---------------------|-------------|
| pre 1   | Partner Onboarding  | Partners complete a onboarding process. Risk & compliance teams perform due diligence. |
| pre 2   | Institution Funding | Partners pre-fund a designated account to ensure sufficient liquidity for remittance. |
| pre 3   | Currency Exchange   | Based on settlement needs, foreign currency is converted into RMB. |
| 4   | Remittance Service | End-users initiate remittance via the provider's app by submitting sender and recipient info.<br>1. If the recipient is new, an SMS prompts setup of a receiving card.<br>2. The provider calls the remittance API to submit the order.<br>3. Funds are routed into local settlement accounts. |
| 5   | Payment Collection  | Recipients collect RMB via digital wallets or linked bank-cards. |

---

**Subject-Specifc Analysis model**

> covering `Sending Institution (Remittance Providers)`, `Orders`, and `Users`.

1. **Sending Institution (SI)** : Analysed key metrics (countries, currencies and transaction, user volumes, fees, FX income ...), produced reports of transaction & profits to guide improvements.
2. **Orders** : 3 stage life-cycle funnel to measure conversion rates and bottleneck. - Order status: A: Provider order | B: Account opening process | C:  Complete fund receipt.
3. **Users** : basic info, behaviour, life-cycle, preferences for targeted campaigns.


**Regulatory Reporting** 

In cross-border inbound remittance to China, user orders are recorded individually (1-to-1), while actual fund settlements arrive in batches. For regulatory reconciliation—especially under HK MSO, MAS MPI, or SAFE/PBoC oversight—we ensure each fund batch (gather_result_list) is accurately matched to valid user orders (remit_list). This ensures auditability, mitigates AML risks, and enables accurate compliance reporting.

<details>
<summary>DML：fundin_order_regulation</summary>

| Field                  | Source | Description                                                                 |
|-----------------------|--------|-----------------------------------------------------------------------------|
| `Flistid`              | T1     | Remittance order ID (primary key)                                           |
| `Forg_id` / `Forg_name` | T1     | Sending institution ID / name                                              |
| `Ftran_amt`            | T1     | Remitted amount in CNY                                                      |
| `Ftran_ccy`            | T1     | Currency type (fixed as CNY)                                                |
| `Ffund_inflow_time`    | T5     | Fund arrival time in China (core time field)                                |
| `Fname_cn_fid`         | T1     | Encrypted recipient name                                                    |
| `Fcre_type` / `Fcre_id_fid` | T1     | Recipient ID type / encrypted ID (used to identify domestic residency and deduplicate recipients) |
| `Fpurpose_code`        | T1     | Purpose of remittance (e.g., education, salary, family support)            |
| `Fpay_final_time`      | T1     | Final timestamp of remittance success or failure                            |

</details>



