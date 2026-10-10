# Data Dictionary — Customer Shopping Behavior Analysis

## 1. Dataset overview

| Property | Details |
|---|---|
| Source file | `customer_shopping_behavior.csv` |
| Dataset grain | One row per customer/purchase record in the supplied dataset |
| Rows | 3,900 |
| Original columns | 18 |
| Missing values | 37, all in `Review Rating` |
| Unique customer IDs | 3,900 |
| Main use | Descriptive analysis of purchase amounts, product preferences, discounts, shipping, subscriptions, and customer characteristics |

**Important interpretation note:** Although the source contains one record for each unique `Customer ID`, the dataset does not provide transaction dates or a transaction identifier. Treat the records as the dataset's observed shopping records; do not infer a customer's complete lifetime history from this file alone.

## 2. Original CSV fields

The field names below match the original CSV headers exactly.

| Original field | Data type in CSV | Description | Example / valid values | Data quality notes |
|---|---|---|---|---|
| `Customer ID` | Integer | Unique identifier assigned to the customer record. | `1`, `2`, `3` | 3,900 unique values; no missing values. It is an identifier, not a measure. |
| `Age` | Integer | Age of the customer in years. | `19`, `50`, `55` | 53 distinct age values; no missing values. |
| `Gender` | Text | Gender category recorded for the customer. | `Male`, `Female` | Two distinct values in the supplied CSV; no missing values. |
| `Item Purchased` | Text | Name of the product purchased. | `Blouse`, `Sweater`, `Jeans` | 25 distinct items; no missing values. |
| `Category` | Text | Broad product category. | `Clothing`, `Accessories`, `Footwear`, `Outerwear` | Four distinct values; no missing values. |
| `Purchase Amount (USD)` | Integer | Recorded purchase amount in US dollars. | `53`, `64`, `73` | 81 distinct amounts; no missing values. This is purchase amount, not profit or margin. |
| `Location` | Text | US state/location associated with the record. | `Kentucky`, `Maine`, `Massachusetts` | 50 distinct values; no missing values. |
| `Size` | Text | Size of the purchased item. | `S`, `M`, `L`, `XL` | Four distinct values; no missing values. Applicability may vary by item. |
| `Color` | Text | Recorded product color. | `Gray`, `Maroon` | 25 distinct values; no missing values. |
| `Season` | Text | Season associated with the purchase record. | `Winter`, `Spring`, `Summer`, `Fall` | Four distinct values; no missing values. The field is not a transaction date. |
| `Review Rating` | Decimal | Customer review rating associated with the item/record. | `3.1` | 37 missing values; 26 distinct non-null values. Averages should be calculated from available ratings unless a documented imputation method is applied. |
| `Subscription Status` | Text | Indicates whether the customer is recorded as a subscriber. | `Yes`, `No` | Two distinct values; no missing values. |
| `Shipping Type` | Text | Shipping option recorded for the purchase. | `Express`, `Free Shipping`, `Standard`, `Next Day Air`, `2-Day Shipping`, `Store Pickup` | Six distinct values; no missing values. |
| `Discount Applied` | Text | Indicates whether a discount was applied to the purchase. | `Yes`, `No` | Two distinct values; no missing values. |
| `Promo Code Used` | Text | Indicates whether a promo code was used. | `Yes`, `No` | Two distinct values; no missing values in the source. The notebook checks this field against `Discount Applied` and then drops it from the prepared DataFrame. |
| `Previous Purchases` | Integer | Number of previous purchases recorded for the customer. | `1`, `2`, `14`, `23` | 50 distinct values; no missing values. The file does not supply dates or a complete customer history to independently validate this count. |
| `Payment Method` | Text | Payment method recorded for the purchase. | `Venmo`, `Cash`, `Credit Card` | Six distinct values; no missing values. |
| `Frequency of Purchases` | Text | Categorical description of how frequently the customer shops. | `Fortnightly`, `Weekly`, `Monthly` | Seven distinct values; no missing values. The category is mapped to approximate days in the notebook. |

## 3. Prepared DataFrame fields

The notebook modifies column names and creates two fields before loading the DataFrame into the SQL Server table `customer`.

| Prepared field | Origin / transformation | Description | Notes |
|---|---|---|---|
| `customer_id` | `Customer ID` converted to lowercase with spaces replaced by underscores | Customer record identifier | Identifier; do not sum or average. |
| `age` | `Age` after column-name standardization | Customer age in years | Integer in the source. |
| `gender` | `Gender` after standardization | Recorded gender category | Text category. |
| `item_purchased` | `Item Purchased` after standardization | Purchased item | Text category. |
| `category` | `Category` after standardization | Product category | Text category. |
| `purchase_amount` | `Purchase Amount (USD)` is standardized and then renamed from `purchase_amount_(usd)` to `purchase_amount` | Purchase amount in USD | Renamed in the notebook for easier SQL use. |
| `location` | `Location` after standardization | Recorded location | Text category. |
| `size` | `Size` after standardization | Product size | Text category. |
| `color` | `Color` after standardization | Product color | Text category. |
| `season` | `Season` after standardization | Recorded season | Not a date or time field. |
| `review_rating` | `Review Rating` after standardization | Review rating | 37 nulls remain; the notebook does not impute them. |
| `subscription_status` | `Subscription Status` after standardization | Subscription indicator | `Yes` / `No`. |
| `shipping_type` | `Shipping Type` after standardization | Shipping method | Text category. |
| `discount_applied` | `Discount Applied` after standardization | Discount indicator | `Yes` / `No`. |
| `previous_purchases` | `Previous Purchases` after standardization | Number of previous purchases | Used by the SQL customer-segmentation and repeat-buyer queries. |
| `payment_method` | `Payment Method` after standardization | Recorded payment method | Text category. |
| `frequency_of_purchases` | `Frequency of Purchases` after standardization | Shopping-frequency category | Used to derive `purchase_frequency_days`. |
| `age_group` | Created with `pandas.qcut(df['age'], q=4, labels=...)` | Quartile-based age grouping | Labels used by the notebook: `Young Adult`, `Adult`, `Middle_aged`, `Senior`. Boundaries are calculated from the data, not set as fixed age ranges. |
| `purchase_frequency_days` | Created by mapping `frequency_of_purchases` to day counts | Approximate numeric representation of shopping frequency | Mapping is documented below. It is a simplified proxy, not an observed exact number of days between purchases. |

### Fields removed during preparation

| Field | Reason / handling |
|---|---|
| `promo_code_used` | The notebook checks whether `promo_code_used` equals `discount_applied` for every row using `(df['discount_applied'] == df['promo_code_used']).all()`, then drops `promo_code_used`. Keep in mind this is a project-specific data-preparation decision; reconsider it if the fields differ in another version of the data. |

### Column-name standardization

The notebook applies these operations in order:

1. Converts column names to lowercase.
2. Replaces spaces with underscores.
3. Renames `purchase_amount_(usd)` to `purchase_amount`.

For example, `Customer ID` becomes `customer_id`, and `Purchase Amount (USD)` becomes `purchase_amount`.

## 4. Transformation mappings used by the notebook

### Age group

The notebook creates quartile-based groups using:

```python
labels = ['Young Adult', 'Adult', 'Middle_aged', 'Senior']
df['age_group'] = pd.qcut(df['age'], q=4, labels=labels)
```

| Label | Meaning in this project |
|---|---|
| `Young Adult` | First age quartile generated by `qcut` |
| `Adult` | Second age quartile generated by `qcut` |
| `Middle_aged` | Third age quartile generated by `qcut` |
| `Senior` | Fourth age quartile generated by `qcut` |

These labels represent relative groups within this dataset. They should not be interpreted as standard demographic age definitions. Exact boundaries depend on the age distribution in the data.

### Purchase frequency to approximate days

The notebook uses the following mapping:

| `frequency_of_purchases` value | `purchase_frequency_days` |
|---|---:|
| `Fortnightly` | 14 |
| `Weekly` | 7 |
| `Monthly` | 30 |
| `Quarterly` | 90 |
| `Bi-Weekly` | 14 |
| `Annually` | 365 |
| `Every 3 Months` | 90 |

This mapping uses simplified approximations (for example, a month is treated as 30 days). Do not describe the resulting field as a measured purchase interval.

## 5. SQL analysis fields and derived business logic

The SQL script queries the prepared `customer` table and uses these fields:

| Analysis | Fields used | Calculation / rule |
|---|---|---|
| Revenue by gender | `gender`, `purchase_amount` | Sum of purchase amounts grouped by gender. |
| Discount users with high purchase amounts | `customer_id`, `discount_applied`, `purchase_amount` | Selects records with `discount_applied = 'Yes'` and purchase amount greater than or equal to the overall average. |
| Highest-rated items | `item_purchased`, `review_rating` | Average review rating per item; null ratings are ignored by SQL aggregate averages. |
| Shipping comparison | `shipping_type`, `purchase_amount` | Average purchase amount for `Standard` and `Express` shipping only. |
| Subscription comparison | `subscription_status`, `customer_id`, `purchase_amount` | Count of records, average purchase amount, and sum of purchase amounts by subscription status. |
| Discount rate by item | `item_purchased`, `discount_applied` | Discounted records divided by all records for each item, multiplied by 100. |
| Customer segments | `customer_id`, `previous_purchases` | `1` = `New`; `2–10` = `Returning`; all other values = `Loyal`. These are script-defined labels, not independently validated lifecycle segments. |
| Top items within category | `category`, `item_purchased`, `customer_id` | Counts records per item and category; ranks within each category using `ROW_NUMBER()`. |
| Repeat buyers and subscription | `subscription_status`, `customer_id`, `previous_purchases` | Counts records by subscription status where `previous_purchases > 5`. |
| Revenue by age group | `age_group`, `purchase_amount` | Sum of purchase amounts by the quartile-based age group created in Python. |

## 6. Data quality summary

| Check | Result |
|---|---|
| Row count | 3,900 |
| Original column count | 18 |
| Unique `Customer ID` values | 3,900 |
| Missing values in `Review Rating` | 37 |
| Missing values in other original columns | 0 |
| Missing-rating treatment in notebook | Values remain missing; no imputation is performed |
| Duplicate ID values | None observed in the supplied CSV based on the unique-ID count matching the row count |

## 7. Important limitations

- The dataset contains no purchase/transaction date, so it cannot support a reliable time-series or trend-over-time analysis.
- The dataset does not include costs, profit, or margin, so purchase amount should not be described as profit.
- The dataset does not have a direct online/offline channel field.
- `purchase_frequency_days` is a mapped approximation.
- `age_group` boundaries are data-dependent quartiles.
- The customer-segment definitions in the SQL script are simple rules based on `previous_purchases`; they should not be treated as verified business definitions.
- Findings are descriptive and do not establish causation or necessarily generalize to all shoppers.

