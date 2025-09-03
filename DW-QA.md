# Data Warehouse

The core purpose of a data warehouse is to integrate and store large amounts of internal and external data, providing accurate, reliable data for analysis, reporting, and decision-making, while addressing issues like **fragmentation**, and difficult historical data management.

## Q1. Data Warehouse Architecture - Hourglass

Built a layered data warehouse (ODS > DIL > DML > DAL) to ingest, clean, and transform data into fact and dimension tables. Defined data domains, granularity, metrics, and embedded business logic for subject-oriented, multi-dimensional analysis

<div align="center">
  <img src="docs/dwh-1.jpg" alt="Diagram" width="700">
</div>

## Q2. How is your data warehouse built?

### 1. Architecture

We follow a **business-driven layered architecture**: **ODS → DIL/DIM → DWS → ADS**.

- **ODS**: Ingest raw data via binlog subscription with hourly batch loading.  
- **DIL/DIM**: Clean, deduplicate, and normalize data; build fact and dimension tables.  
- **DML (Data Mart Layer)**: Perform <mark>subject-oriented</mark> modeling around <mark>business entities</mark> (e.g., Merchant, Order) and processes (e.g., Top-up, settlement, payment/withdrawal), delivering reusable wide tables and standardized metrics for **<mark>multi-dimensional and thematic analysis.</mark>**
- **ADS (Application Data Service Layer)**: Deliver application-level wide tables to support Finance, Risk, and BI reporting.  

### 2. Modeling

#### Business Case 1 – Cross-border E-commerce Collection

- **Process-oriented**: Model fact tables around fund flow processes:  
  *Settlement(Fund Distribution) → Withdrawal/Payment/Subscription.*  
- **Entity-oriented**: Build subject tables around merchants, shops, and orders.  
  Design metrics to enable multi-dimensional analysis, subject-area analytics, and monitoring of core business KPIs.  

<details>
<summary><strong><mark>Amazon - Cross-border E-commerce Collection - Data Warehouse Modeling</mark></strong></summary>

**Core idea:** Amazon settles **per shop** into **sub-VA** (real bank sub-account); provider internally aggregates to **main VA** for the merchant. We model **settlement** and **cash-out/payments**; the internal sub-VA→main-VA aggregation is automatic and **not** a business fact.

### 1) Business Process (for context)
1. **Merchant onboarding** → register, KYC pass  
2. **Shop authorization & binding** →
3. **Amazon settlement** → **Amazon → sub-VA (shop-level)**  
4. *(Internal aggregation)* sub-VA → main VA *(system auto, non-analytical)*  
5. **Cash-out / payment** → main VA → bank account / supplier

### 2) Layered Architecture Mapping

| Layer | What we build | Examples |
|------|----------------|----------|
| **ODS** | Raw pulls from Amazon/shop-binding/KYC/account systems | shop binding logs, settlement raw, payout raw |
| **DIL/FACT + DIM** | Cleaned **facts** (atomic events) + standard dims | `FCT_SETTLEMENT`, `FCT_WITHDRAWAL`, `FCT_PAYMENT`; `DIM_MERCHANT`, `DIM_SHOP` |
| **DML / Subject** | Subject-oriented wide tables & metrics | `DML_MERCHANT_SUBJECT`, `DML_SHOP_SUBJECT`, `DML_ORDER_SUBJECT` |
| **ADS** | App/report views & cubes | Finance/Risk/Product dashboards |

> Naming note: DIL≈DWD, DML≈DWS in other orgs.

### 3) Fact Tables (Process-oriented)

#### 3.1 `FCT_SETTLEMENT` — *Amazon → sub-VA (shop-level settlement)*
- **Grain:** one **settlement line** per **shop × currency × settlement_id**
- **Keys:** `settlement_id`, `shop_id`, `merchant_id (fspid)`, `sub_va_id`
- **Core fields:**  
  - `fdate` (partition), `settlement_time`  
  - `currency`, `amount_settled`, `fx_to_cnh`, `amount_cnh`, `amount_usd`  
  - `platform_id='amazon'`, `site_id`  
- **Notes:** True external inflow; **drives revenue KPIs** at shop & merchant.

#### 3.2 `FCT_WITHDRAWAL` — *main VA → merchant bank account*
- **Grain:** one withdrawal order
- **Keys:** `withdraw_order_id`, `merchant_id`, `bank_account_id`
- **Core fields:** `withdraw_time`, `currency`, `amount`, `fee`, `status`

#### 3.3 `FCT_PAYMENT` — *main VA → supplier / subscription / fees*
- **Grain:** one payment order
- **Keys:** `payment_order_id`, `merchant_id`, `payee_id`, `biz_type`
- **Core fields:** `payment_time`, `currency`, `amount`, `fee`, `biz_type` *(supplier / subscription / platform fee)*

> 🚫 **No `FCT_DISTRIBUTION`** for sub-VA→main-VA: it’s internal aggregation; usually not modeled as a business fact.

### 4) Dimension Tables (Entity-oriented)

#### 4.1 `DIM_MERCHANT`
- **Keys:** `merchant_id (fspid)`, `fgid`
- **Attrs:** register channel, KYC status, country/region, wallet type, lifecycle flags
- **Timestamps (merchant funnel):** `create_time_enter`, `register_time`, `kyc_apply_time`, `kyc_approve_time`, …

#### 4.2 `DIM_SHOP`
- **Keys:** `shop_id`
- **Attrs:** platform (`amazon`), `site_id` (e.g., `amzNA`, `amzEU`), country, default currency, status
- **Timestamps (shop funnel):**  
  - **`shop_apply_time`**: merchant **applied** in provider console to bind this shop  
  - **`shop_auth_time`**: mid-state (API authorization / token ready)  
  - **`shop_bind_time`**: **binding completed** (ownership verified, sub-VA allocated)  
  - **`first_settlement_time`** (first inflow), **`first_withdrawal_time`**

#### 4.3 Other Dims
- `DIM_CURRENCY`, `DIM_ACCOUNT` (main/sub-VA), `DIM_PAYEE`, `DIM_DATE`, …

### 5) Subject Tables (DML / Wide models)

#### 5.1 `DML_MERCHANT_SUBJECT` — *Merchant-centric KPIs*
- **Keys:** `merchant_id`, `fdate`
- **Horizontal (funnel timestamps):** registration/KYC times, **first_settlement_time**, **first_withdrawal_time**
- **Vertical (rolling metrics & tags):**  
  - Totals: `total_settlement_amt_usd/cnh`, `total_settlement_cnt`  
  - Recent 28d: `settl_amt_28d`, `settl_cnt_28d`  
  - Shops: `shop_cnt`, `active_shop_cnt_28d`  
  - Lifecycle tag: *1 Unfunded / 2 New / 3 Retained / 4 Lost / 5 Recovered / 0 Default*  
    - **Lost rule (example):** *first_settlement exists* AND *no settlement in last 28 days*.

#### 5.2 `DML_SHOP_SUBJECT` — *Shop-centric KPIs*
- **Keys:** `shop_id`, `fdate`
- **Funnel timestamps (shop-level):** `shop_apply_time`, `shop_auth_time`, `shop_bind_time`, `first_settlement_time`, `first_withdrawal_time`
- **Metrics:**  
  - Totals: `total_settlement_amt_usd/cnh`, `total_settlement_cnt`  
  - 28d: `settl_amt_28d`, `settl_cnt_28d`  
  - Activity tag: *active if 28d has settlement; else inactive*  
  - Site/platform breakdown via `platform_id`, `site_id`

#### 5.3 `DML_ORDER_SUBJECT` — *Order/transaction lens*
- Join across facts to trace **settlement → (internal aggregation) → cash-out/payment**  
- Use for **lag analysis**, **amount bucket distributions**, **anomaly checks**.

### 6) Typical KPIs

- **Merchant level:** total inflow (USD/CNH), **active shops**, 28-day settlement, lifecycle tag, cash-out ratio  
- **Shop level:** first settlement latency (bind→first inflow), monthly settlement, 28-day activity, site mix  
- **Operational:** settlement frequency, long-tail shops, withdrawal cadence, fee rate trends

### 7) Data Quality (examples)
- **Timeliness:** partitions landed by SLA; alert if delayed  
- **Integrity:** `settlement_id` uniqueness; **shop_id/merchant_id** FK valid  
- **Consistency:** currency FX conversion reproducible; totals match finance reconciliation  
- **Security:** sensitive identifiers encrypted; access controlled

### 8) Field Name Alignment (mapping highlights)
- **Merchant funnel:**  
  - `fcreate_time_enter` → entry time  
  - `fcreate_time_register` → register time  
  - `fcreate_time_kyc_apply` / `fcreate_time_kyc_info` / `fcreate_time_kyc_reject` / `fkyc_first_approved_time`
- **Shop funnel:**  
  - `fcreate_time_shop_apply` → **shop_apply_time**  
  - `fcreate_time_shop_auth` → **shop_auth_time**  
  - `fcreate_time_shop_bind` → **shop_bind_time**  
  - `ffirst_recharge_time` *(rename)* → **first_settlement_time**  
  - `ffirst_withdrawal_time` → **first_withdrawal_time**
- **Rolling metrics:**  
  - `Ftotal_recharge_amount_usd/cnh` → **total_settlement_amt_usd/cnh**  
  - `Flast_recharge_amount_*_28d` → **settl_amt_28d / settl_cnt_28d**

> ✅ Replace legacy “recharge” naming with **settlement** to reflect Amazon→sub-VA semantics.

</details>

#### Business Case 2 – Cross-border Remittances

- **Process-oriented**: Model fact tables around the remittance lifecycle:  
  *Sender initiates transfer → Remittance → Recipient account setup → Recipient collects funds.*  
- **Entity-oriented**: Build subject tables around sending institutions, remittance orders (3 lifecycle stages), recipients, and senders.  
  Design metrics to support funnel analysis, compliance monitoring, and business insights.  

### 3. Governance

- **Data Asset Scoring**: Evaluate each table across four dimensions: standards, data quality (DQC), security, and cost.  
- **Data Quality Monitoring (DQC)**: Enforce rules for completeness, uniqueness, timeliness, and reconciliation.  
- **Security & Compliance**: Detect unencrypted sensitive fields and enforce ownership accountability.  

### 4. Development Process：

1. Defined business goals and requirements.
2. Collected data into ODS and integrated into fact and dimension tables (DIL/DIM).
3. Organised data domains, determined data granularity, and designed key metrics.
4. Abstracted business and data subject analyses into DML tables.
5. Delivered reporting, supporting subject-specific and multi-dimensional analysis


## Amazon Standard Collection vs Shopee Official Wallet

### 1. Background 背景

#### Amazon Standard Collection (English)
In Amazon’s standard collection model, cross-border sellers cannot easily open overseas bank accounts. Payment service providers like Tenpay issue one **main VA (real bank account)** for settlement and create **sub-VAs (child accounts with unique identifiers)** for each store bound under the merchant.  
Amazon pays into the sub-VA (store level), which technically maps back to the main VA. This allows tracking of funds per store and per currency.  

#### 亚马逊标准收款 (中文)
在亚马逊标准收款模式下，跨境卖家很难在海外开立银行账户。收款服务商（如 Tenpay）会为商户开立一个 **主VA（真实银行账户）**，并在商户绑定店铺时分配 **子VA**。  
亚马逊将货款打入子VA（每个店铺一个），而子VA最终归集到主VA，用于资金清算和提现。这样服务商可以通过子VA识别不同店铺的资金来源。

#### Shopee Official Wallet (English)
In Shopee’s official wallet model, Shopee itself acts as the settlement entity. After sellers onboard and bind stores, Shopee credits their **official wallet account** (white-label offshore account powered by Tenpay).  
There is **no sub-VA per store** — store-level differentiation comes from Shopee’s internal transaction system. Funds can be disbursed (fees, supplier payments, subscription plans) or withdrawn to bank accounts.  

#### Shopee 官方钱包 (中文)
在 Shopee 官方钱包模式下，Shopee 与 Tenpay 深度合作，Shopee 自身作为结算主体。商户入驻并绑定店铺后，货款直接进入商户的 **Shopee 官方钱包账户**。  
这里 **没有子VA**，店铺的区分由 Shopee 内部交易系统完成。资金可以用于平台代扣（佣金、年卡）、供应商付款，或提现至银行账户。


### 3. Data Warehouse Construction 

### Amazon Standard Collection
**Business Process**  
1. Merchant onboarding (商户入驻)  
2. VA assignment (主VA开户)  
3. Store authorization & binding & sub-VA assignment (店铺绑定 + 子VA发放)  
4. Amazon pays store VA (亚马逊打款 → 子VA)  
5. Transaction details via API (获取交易明细)  
6. Merchant card binding (商户绑卡)  
7. Withdrawal & payout (提现/付款)  

**Modeling / 建模**  
- **Fact tables**: 收款订单事实表、提现订单事实表  
- **Dimension tables**: 商户维度、店铺维度、收款人维度、账户维度  
- **DML subject tables**: Merchant， Shop， Order
  - Horizontal timeline fields (first recharge, first withdrawal)  
  - Vertical tags (active stores, total inflow, retention metrics)  

<details>
<summary><strong>Merchant Subject Sample - Data Metric</strong></summary>

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

