# Customer Shopping Behavior Analysis

An end-to-end data analytics portfolio project exploring customer shopping behavior using **Python in Jupyter Notebook, Microsoft SQL Server with SQL Server Management Studio (SSMS) 22, and Power BI Desktop**. The workflow inspects and prepares the source CSV, loads the prepared DataFrame into SQL Server, uses T-SQL queries to investigate customer spending and shopping patterns, and presents the analysis in a Power BI report.

> **Project status:** The notebook and SQL script are the working analysis artifacts. The Power BI `.pbix` file is included separately. Open it in Power BI Desktop and verify that the report and its data connections work in your environment. Add screenshots and query-result exports only after capturing them from the actual applications.

## Project goals

- Inspect and prepare customer shopping data for analysis.
- Explore purchase amounts across customer groups.
- Examine discount use, product ratings, shipping types, and subscription status.
- Use SQL aggregations, subqueries, `CASE`, CTEs, and window functions to answer business questions.
- Present findings in a dashboard/report and communicate practical business questions and recommendations.

## Tools and technologies

| Purpose | Tool |
|---|---|
| Python analysis and data preparation | **Jupyter Notebook** |
| Data manipulation | Python, Pandas |
| Database | **Microsoft SQL Server** |
| SQL editor and query execution | **SQL Server Management Studio (SSMS) 22** |
| Python-to-SQL connection | SQLAlchemy, `pyodbc` |
| SQL Server ODBC connection | Microsoft ODBC Driver 17 for SQL Server (as specified in the notebook) |
| Dashboard/report | **Power BI Desktop** |

SSMS is the SQL client used to connect to and query Microsoft SQL Server. It is not the database engine itself. Power BI Desktop is used to open and inspect the supplied `.pbix` report.

## Repository contents

Use this structure if you have organized the files into folders. If your actual filenames differ, update the names here and in any code paths accordingly.

```text
Customer-Shopping-Behavior-Analysis/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   └── customer_shopping_behavior.csv
├── notebooks/
│   └── customer_shopping_behavior_analysis.ipynb
├── sql/
│   └── customer_shopping_behavior_analysis.sql
├── powerbi/
│   └── customer_shopping_behavior_dashboard.pbix
├── reports/
│   ├── business_problem_statement.pdf
│   ├── customer_shopping_behavior_analysis_report.pdf
│   └── customer_shopping_behavior_analysis_presentation.pptx
└── docs/
    └── data_dictionary.md
```

Only include files and folders that actually exist in your repository. Rename the paths in this README if you keep the original filenames.

## Dataset overview

The supplied CSV contains **3,900 rows and 18 columns**. Fields include customer ID, age, gender, item purchased, category, purchase amount, location, size, color, season, review rating, subscription status, shipping type, discount applied, promo code used, previous purchases, payment method, and purchase frequency.

The `Review Rating` column contains **37 missing values** in the supplied dataset. The notebook checks missing values, standardizes column names, renames `purchase_amount_(usd)` to `purchase_amount` after normalizing the headers, creates an age-group column, maps purchase-frequency labels to approximate day counts, and drops `promo_code_used` after checking it against `discount_applied`.

### Important data assumptions and limitations

- `purchase_frequency_days` is a simplified mapping from text categories to day counts. It is an approximation, not an observed exact interval for each customer.
- The notebook does **not** fill in missing review ratings. SQL averages over `review_rating` ignore `NULL` values.
- The notebook creates age groups with `pandas.qcut`, which divides the rows into quantile-based groups. These are sample-relative groups, not fixed business-defined age bands.
- The source data does not include transaction dates, profit/cost, or a direct online/offline channel field. Do not claim time-series, profitability, or online-versus-offline comparisons from this dataset.
- The CSV includes a `Payment Method` field, but it does not include transaction-level payment amounts or enough information to establish that a payment method causes a particular spending pattern.
- The SQL customer-segment query labels `previous_purchases = 1` as `New`, `2–10` as `Returning`, and values above 10 as `Loyal`. These are rule-based labels from the script, not verified customer lifecycle stages.

## Setup and run

### 1. Place the project files

Keep the dataset, notebook, SQL script, and dashboard in their corresponding folders. The notebook currently reads the CSV using a local absolute path similar to:

```python
df = pd.read_csv('D:/Downloads/customer_shopping_behavior.csv')
```

Change the path to match where you saved the CSV. If the notebook is in `notebooks/` and the CSV is in `data/`, a relative path from the notebook's working directory may be:

```python
df = pd.read_csv('../data/customer_shopping_behavior.csv')
```

The exact relative path depends on the folder from which Jupyter runs. Confirm it before running the notebook.

### 2. Install the Python packages

Open a terminal in the Python environment used by Jupyter, navigate to the project root, and run:

```bash
python -m pip install -r requirements.txt
```

The `requirements.txt` file lists these Python packages:

```text
pandas>=2.0,<4.0
SQLAlchemy>=2.0,<3.0
pyodbc>=5.0,<6.0
jupyter>=1.0,<2.0
```

If you use Anaconda, launch Jupyter Notebook from the same environment where you installed these packages.

**Important:** `requirements.txt` installs Python packages only. It does not install Microsoft SQL Server, SSMS, the Microsoft ODBC driver, or Power BI Desktop.

### 3. Install the required desktop tools and ODBC driver

Install or confirm that you have:

- **Microsoft SQL Server** — the database engine.
- **SQL Server Management Studio (SSMS) 22** — to connect to SQL Server and run the SQL script.
- **Microsoft ODBC Driver 17 for SQL Server** — the notebook's connection string specifies this driver. Install it separately on Windows. If a different driver version is installed, update the notebook connection string to match.
- **Power BI Desktop** — to open the `.pbix` dashboard/report.

### 4. Configure SQL Server

1. Start your SQL Server instance.
2. Create a database named `customer_behavior`, or change the notebook's `database` setting to your database name.
3. Ensure your Windows account has permission to connect to the database and create/write tables.
4. The notebook currently uses a server setting equivalent to `localhost`. If your SQL Server uses a named instance or a different server name, update the setting.
5. Confirm that the connection string uses Windows Authentication (`Trusted_Connection=yes`) and the correct installed ODBC driver.

### 5. Run the Jupyter Notebook

Open `notebooks/customer_shopping_behavior_analysis.ipynb` and run the cells in order. The notebook reads the CSV, inspects and transforms the data, and writes the prepared DataFrame to a SQL Server table named `customer`.

**Warning:** The notebook uses `if_exists="replace"` when writing the table. Running that cell replaces the existing `customer` table in the selected database. Back up any table you need to keep, or change this behavior before rerunning.

### 6. Execute the SQL script in SSMS 22

1. Open SSMS 22.
2. Connect to the same SQL Server instance used by the notebook.
3. Select the `customer_behavior` database.
4. Open `sql/customer_shopping_behavior_analysis.sql`.
5. Confirm that the notebook has created and populated the `customer` table.
6. Execute the queries and review the returned results.

The supplied SQL script contains ten business questions. Capture query outputs or screenshots only after executing the queries and checking the results.

### 7. Open the Power BI report

Open `powerbi/customer_shopping_behavior_dashboard.pbix` in Power BI Desktop. Check that the report opens, visuals load, and data connections or paths are valid in your environment. If prompted, configure the data source to match your SQL Server setup.

A `.pbix` file is a Power BI report file, regardless of the original filename.

## SQL questions covered

The supplied SQL script addresses:

1. Total purchase amount by gender.
2. Customers who used a discount and spent at least the overall average purchase amount.
3. Products with the highest average review ratings.
4. Average purchase amount for Standard versus Express shipping.
5. Customer counts, average spend, and total purchase amount by subscription status.
6. Products with the highest proportion of purchases with discounts applied.
7. Counts of customers in the script-defined New, Returning, and Loyal segments.
8. Top three purchased items within each category using `ROW_NUMBER()`.
9. Subscription-status counts among customers with more than five previous purchases.
10. Total purchase amount by the age groups created in Python.

## Quality checks and interpretation notes

- Confirm that the CSV path, SQL Server instance, database name, ODBC driver, and permissions match your machine.
- Validate row counts after loading the data into SQL Server.
- Check missing values and category spellings before interpreting averages or rankings.
- The discount-related query uses `>=` (at least the average purchase amount).
- The query using `ROW_NUMBER()` ranks items within each category; add `ORDER BY category, item_rank` to the final query if you want the output consistently ordered.
- The customer-segment labels are defined by previous-purchase counts: exactly 1 is `New`, 2–10 is `Returning`, and above 10 is `Loyal`. Review these definitions before using them for business decisions.
- Results describe this dataset and should not be treated as causal evidence or as representative of all shoppers.
- Verify that numbers and recommendations in the final report, presentation, and dashboard match the executed SQL results.

## Key findings to validate

The dataset contains **3,900 records** and the total purchase amount is **$233,081**. The mean purchase amount is approximately **$59.76**, and the median is **$60**. Treat these as descriptive statistics for this dataset, not profit or net revenue.

The data has **37 missing review ratings**. Rating-based results should be interpreted using available ratings, and missing values should not be silently treated as actual ratings.

## Limitations

This is a descriptive analysis of the supplied customer-shopping dataset. It does not establish why customers behave a certain way, estimate profit, or track changes over time because the CSV does not provide the fields needed for those analyses. Although the dataset includes `Payment Method` and `Season`, it does not include transaction dates, cost/margin fields, or a direct online/offline channel field.

## Future improvements

- Validate the Power BI visuals against executed SQL results.
- Add screenshots captured from the actual dashboard and query results.
- Add verified business insights and recommendations based on query outputs.
- Add automated data-quality checks and document the SQL Server/ODBC configuration.
- Explore profitability, time trends, or channel performance only if suitable additional data becomes available.
