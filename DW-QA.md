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
  *Top-up → Distribution → Withdrawal/Payment/Subscription.*  
- **Entity-oriented**: Build subject tables around merchants, shops, and orders.  
  Design metrics to enable multi-dimensional analysis, subject-area analytics, and monitoring of core business KPIs.  

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

### 2. Comparison 表格对比

| Aspect / 维度         | Amazon Standard Collection (English) | 亚马逊标准收款 (中文)                   | Shopee Official Wallet (English)      | Shopee 官方钱包 (中文)               |
|-----------------------|--------------------------------------|-----------------------------------------|---------------------------------------|--------------------------------------|
| **Account Structure / 账户结构** | Main VA + sub-VA per store          | 主VA + 店铺子VA                          | One official wallet per merchant       | 每个商户一个官方钱包账户              |
| **Fund Inflow / 资金入账**       | Amazon pays into sub-VA (per store) | 亚马逊打款到子VA（店铺维度）             | Shopee credits merchant wallet directly| Shopee 直接打款到商户钱包             |
| **Store Differentiation / 区分店铺** | By sub-VA number                     | 通过子VA编号区分                         | By Shopee transaction system           | 通过 Shopee 内部交易系统区分          |
| **Settlement / 清算归集**        | Sub-VA → Main VA → Withdrawal        | 子VA → 主VA → 商户提现                   | Wallet balance → Deduction → Withdrawal| 钱包余额 → 扣费 → 提现               |
| **Complexity / 模式复杂度**      | Higher (need VA management per store)| 较高（需为每个店铺管理VA）               | Lower (platform internal accounting)   | 较低（平台内部记账）                  |

---

### 3. Data Warehouse Construction 数仓建设

### Amazon Standard Collection
**Business Process / 业务过程**  
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

> 收款订单事实表 : 电商平台打款 → 店铺 VA → 主账户 的资金入账交易


<details>
<summary><strong style="color:#1E90FF;">Merchant Subject Sample - Data Metric</strong></summary>

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

