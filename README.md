# ACCOUNT RECEIVABLES ANALYSIS
![](https://github.com/nheesele/account_receivables_analysis/blob/main/accounts-receivable-la-gi.jpeg)

> Source: https://bizzi.vn/wp-content/uploads/2025/04/accounts-receivable-la-gi.jpeg

This project utilizes a historical invoice and payment transactions dataset from a finance factoring context to explore billing methods, disputes, settlement timing, and payment delays through data analysis and visualization.

The dataset contains information from 2,466 invoices, with invoices issued from January 2012 to December 2013 and payments settled from January 2012 to January 2014. Each record represents a unique invoice, identified by the `invoiceNumber`

## Data source
This data was provided by IBM for testing out their analytics tools. It can be assessed via Kaggle
Dataset source: [Kaggle - Finance Factoring IBM Late Payment Histories](https://www.kaggle.com/datasets/hhenry/finance-factoring-ibm-late-payment-histories)

***How to get the data?***
```sql
import kagglehub
path = kagglehub.dataset_download("hhenry/finance-factoring-ibm-late-payment-histories")
df = pd.read_csv("/kaggle/input/finance-factoring-ibm-late-payment-histories/WA_Fn-UseC_-Accounts-Receivable.csv")
```

## Dataset Overview

The dataset contains historical invoice and payment transactions from a finance factoring context, including details on billing methods, disputes, settlement timing, and payment delays.

- **Data Period**:
  - Invoices issued from January 2012 to December 2013.
  - Payments settled from January 2012 to January 2014.

- **Number of Records**: 2,466 records

- **Record Level**: Each record represents one invoice, unique by `invoiceNumber`.

## Data Structure
Below is a detailed description of the columns in the datasets:

| Column Name      | Description                                                                                          |
|------------------|------------------------------------------------------------------------------------------------------|
| countryCode      | Numeric code representing the customer’s country or region.                                          |
| customerID       | Unique identifier for each customer.                                                                 |
| PaperlessDate    | Date when the customer switched to paperless billing (if applicable).                                |
| invoiceNumber    | Unique identifier for each invoice (primary key of the dataset).                                     |
| InvoiceDate      | Date when the invoice was issued.                                                                    |
| DueDate          | Contractual payment due date for the invoice.                                                        |
| InvoiceAmount    | Total monetary value of the invoice.                                                                 |
| Disputed         | Indicator whether the invoice was disputed (`Yes` or `No`).                                          |
| SettledDate      | Actual date when the payment was settled.                                                            |
| PaperlessBill    | Billing method used for the invoice (`Paper` or `Electronic`).                                       |
| DaysToSettle     | Number of days taken from invoice issuance to payment settlement.                                    |
| DaysLate         | Number of days the payment was late compared to the due date (0 if paid on time or early).           |

## Data Cleaning
Let's perform initial data inspection to understand structure, data types, and missing values.

```sql
df.sample(3)
```

|   countryCode | customerID   | PaperlessDate   |   invoiceNumber | InvoiceDate   | DueDate    |   InvoiceAmount | Disputed   | SettledDate   | PaperlessBill   |   DaysToSettle |   DaysLate |
|--------------:|:-------------|:----------------|----------------:|:--------------|:-----------|----------------:|:-----------|:--------------|:----------------|---------------:|-----------:|
|      818 | 7946-HJDUR   | 2/21/2012       |      9373791288 | 3/31/2012     | 4/30/2012  |           60.57 | No         | 4/21/2012     | Electronic      |             21 |          0 |
|     770 | 9323-NDIOV   | 7/9/2012        |      8996690503 | 9/18/2012     | 10/18/2012 |           61.32 | Yes        | 10/25/2012    | Electronic      |             37 |          7 |
|    406 | 6833-ETVHD   | 4/30/2012       |      8844419761 | 5/10/2013     | 6/9/2013   |           54.69 | No         | 6/1/2013      | Electronic      |             22 |          0 |


```sql
df.info()
```

### Data Summary

```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 2466 entries, 0 to 2465
Data columns (total 12 columns):
 #   Column         Non-Null Count  Dtype         
---  --------------  --------------  -------------- 
  0   countryCode    2466 non-null   int64  
 1   customerID     2466 non-null   object 
 2   PaperlessDate  2466 non-null   object 
 3   invoiceNumber  2466 non-null   int64  
 4   InvoiceDate    2466 non-null   object 
 5   DueDate        2466 non-null   object 
 6   InvoiceAmount  2466 non-null   float64
 7   Disputed       2466 non-null   object 
 8   SettledDate    2466 non-null   object 
 9   PaperlessBill  2466 non-null   object 
 10  DaysToSettle   2466 non-null   int64  
 11  DaysLate       2466 non-null   int64  
dtypes: float64(1), int64(4), object(7)
```
Upon reviewing df.info(), the dataset has 2,466 records across 12 columns with no missing values. However, a few data types should be adjusted for better analysis:

* Convert date columns (`PaperlessDate`, `InvoiceDate`, `DueDate`, `SettledDate`) from object to datetime.
* Change `invoiceNumber`, `countryCode` from int64 to object (string), as it is an identifier.

```sql
date_cols = ['PaperlessDate','InvoiceDate', 'DueDate', 'SettledDate']

for col in date_cols:
    df[col] = pd.to_datetime(df[col])

df['invoiceNumber'] = df['invoiceNumber'].astype(str)
df['countryCode'] = df['countryCode'].astype(str)
```

- Converted `Disputed` and `PaperlessBill` to appropriate formats for analysis.
- Verified no missing values and no invalid dates (e.g., settlement before invoice).
- No duplicates found.

## Exploratory Data Analysis (EDA)
- Generated summary statistics for numerical variables (`InvoiceAmount`, `DaysToSettle`, `DaysLate`).

```sql
numeric_cols = df.select_dtypes(include=['number'])
```
```sql
numeric_summary = numeric_cols.describe().round(2)
print(numeric_summary)
```

```
      InvoiceAmount  DaysToSettle  DaysLate
count        2466.00       2466.00   2466.00
mean           59.90         26.44      3.44
std            20.44         12.33      6.29
min             5.26          0.00      0.00
25%            46.40         18.00      0.00
50%            60.56         26.00      0.00
75%            73.76         35.00      5.00
max           128.28         75.00     45.00
```

- Visualized:
  - Distribution of payment delays (histogram)
  - Correlation heatmap of key behavioral factors
  - Aging bucket distribution
  - Average delay by country and by invoice month
  - Monthly invoiced amount time-series (seasonal pattern)
- Customer-level aggregation: late payment ratio, average delay, transaction frequency.

## Analysis Performed
* **Correlation analysis**: 

```sql
  heatmap_cols = [
    'DaysLate',
    'TransactionFrequency',
    'InvoiceAmount',
    'Disputed',
    'PaperlessBill']
  ```
```sql
heatmap_data = df[heatmap_cols].copy()
heatmap_data['Disputed'] = heatmap_data['Disputed'].map({'Yes': 1, 'No': 0})
heatmap_data['PaperlessBill'] = heatmap_data['PaperlessBill'].map({'Paper': 0, 'Electronic': 1})
```
```sql
plt.figure(figsize=(8,6))
sns.heatmap(
    heatmap_data.corr(),
    annot=True,
    cmap='YlOrBr',
    fmt='.2f'
)
plt.title('Correlation Heatmap of Customer Behavior and Payment Risk')
plt.show()
```

<img width="398" height="334" alt="image" src="https://github.com/user-attachments/assets/1ffde93d-2068-4187-b0a7-844ad70aedbd" />

> Strongest positive correlation between disputes and delays (0.44); moderate negative correlation with electronic billing (−0.16).
 
- **Group comparisons**: Disputed invoices average ~10 days late vs. ~2 days for non-disputed; electronic bills paid ~2 days earlier.
- **Seasonality**: Clear annual cycle with peak invoicing in March–April and sharp decline toward year-end.
- **Customer segmentation**: Created risk groups (High/Medium/Low) based on late payment ratio and average delay.
- **Statistical modeling**: Multiple linear regression (OLS) predicting `DaysLate`:
  `DaysLate = 3.9573 - 0.0008×InvoiceAmount + 6.6684×Disputed - 2.0422×PaperlessBill - 0.0387×TransactionFrequency`
  
- R-squared = 0.225
- Significant predictors: `Disputed` (p < 0.001) and `PaperlessBill` (p < 0.001)

## Key Insights
- Most invoices are paid on time; average delay (~3.4 days) is driven by outliers.
- **Disputes** are the primary driver of late payments (+6.67 days on average).
- **Electronic billing** significantly reduces delays (~2 days faster).
- Country variation: Country 818 has highest average delay (4.82 days); Country 391 is most reliable (1.85 days).
- Strong seasonal pattern in invoicing volume affects cash flow predictability.

## Hypotheses Supported by Data
- Disputes cause significant payment delays → confirmed (strongest predictor).
- Digital billing improves timeliness → confirmed (negative coefficient, statistically significant).
- Higher transaction frequency slightly reduces delays → weakly supported (small negative effect).

## Recommendations
1. **Accelerate dispute resolution** processes — greatest potential to reduce delays.
2. **Promote electronic billing** aggressively to all customers to shorten payment cycles.
3. **Implement country-specific collection strategies**:
 - Early reminders and dedicated monitoring for high-delay regions (e.g., Country 818).
 - Offer incentives or extended credit to reliable regions (e.g., Country 391).
4. **Prepare for seasonality**: Build cash reserves in low-volume periods and intensify collections during peak months.
5. **Monitor high-risk customers** identified in the segmentation for proactive follow-up.

**Author**: Nhi Le
**Date**: December 2025

