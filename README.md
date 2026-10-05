# mamaearth-growth-analytics
Mamaearth Growth Analytics project covering SQL reporting, data cleaning, EDA, visualization, and narrative insights.
This project analyzes e-commerce orders to identify revenue trends, data-quality issues, return-rate patterns, and operational risk segments. It combines SQL reporting, Python data cleaning/EDA, visualizations, and an optional Gemini-generated business narrative.

## Project Structure

```text
.
├── README.md
├── requirements.txt
├── sql/
│   ├── schema.sql
│   ├── seed_data.sql
│   └── reports.sql
├── data/
│   ├── customers.csv
│   ├── products.csv
│   └── orders.csv
├── analysis/
│   ├── clean_and_eda.py
│   └── visualize.py
├── visualizations/
│   ├── return_rate_by_payment.png
│   └── monthly_revenue_trend.png
└── narrator/
    ├── findings.json
    ├── generate_narrative.py
    └── sample_output.txt
```

---

# 1. SQL Setup and Reports

The SQL files create the database tables, load the source data, and generate the analytical reports.

## Step 1. Create the database schema

Run:

```sql
\i sql/schema.sql
```

This creates the required tables for customers, products, and orders.

## Step 2. Load the seed data

Run:

```sql
\i sql/seed_data.sql
```

This inserts the project data into the tables.

## Step 3. Run the SQL reports

Run:

```sql
\i sql/reports.sql
```

The report queries provide the SQL-side analysis used to inspect revenue, orders, returns, and other business metrics.

For PostgreSQL, the three files can be run from `psql` in this order:

```bash
psql -d <database_name>
```

Then inside `psql`:

```sql
\i sql/schema.sql
\i sql/seed_data.sql
\i sql/reports.sql
```

Replace `<database_name>` with the name of your PostgreSQL database.

**Important:** Run the files in this order. `seed_data.sql` depends on the tables created by `schema.sql`, and `reports.sql` depends on the loaded data.

---

# 2. Python Cleaning, EDA, and Visualizations

The Python workflow cleans the order data, validates the results, performs exploratory analysis, and creates the project visualizations.

## Install dependencies

From the project root:

```bash
pip install -r requirements.txt
```

The Python scripts expect the CSV files in:

```text
data/customers.csv
data/products.csv
data/orders.csv
```

## Step 1. Run data cleaning and EDA

Run:

```bash
python analysis/clean_and_eda.py
```

This step:

* Loads the customers, products, and orders data.
* Standardizes fields such as `payment_method`.
* Removes duplicate orders using the project's natural key.
* Fills missing discount values with `0`.
* Fills missing ratings using the median rating.
* Calculates order revenue.
* Checks quantity outliers using the IQR rule.
* Reconciles cleaned revenue against the raw revenue.
* Calculates return rates by payment method.
* Identifies the highest-risk payment-method/city-tier segment.
* Calculates monthly revenue.
* Tests the relationship between discounts and returns.
* Writes the validated findings required by the narrative stage.

### Task 5 output

Task 5 of Part 2 writes:

```text
narrator/findings.json
```

This file is the single structured input for the narrative generator. It contains the validated business findings rather than requiring the narrative script to recalculate or hard-code them.

The expected key findings include:

* Cleaned total revenue: **₹97,358.30**
* Raw total revenue: **₹99,860.20**
* Duplicate reconciliation delta: **₹2,501.90**
* COD return rate: **44.4%**
* Highest-risk segment: **COD + Tier-2 at 54.5%**
* True peak month: **March 2026 at ₹20,318.90**
* January's apparent revenue is inflated by quantity outliers.

The two quantity outliers are flagged rather than deleted from the cleaned dataset. This distinction is important because the cleaned revenue and the outlier-corrected monthly comparison answer different analytical questions.

## Step 2. Generate visualizations

After the cleaning/EDA step, run:

```bash
python analysis/visualize.py
```

The visualization script uses the same cleaning and normalization logic as the analysis workflow.

It produces the charts in:

```text
visualizations/
```

including:

```text
visualizations/return_rate_by_payment.png
visualizations/monthly_revenue_trend.png
```

The payment-method visualization uses standardized payment-method values (`COD`, `CARD`, and `UPI`) so differently capitalized raw values such as `cod`, `COD`, `card`, `CARD`, `Card`, `upi`, and `UPI` are treated as the same payment method.

The expected cleaned return rates are:

| Payment method | Return rate |
| -------------- | ----------: |
| COD            |       44.4% |
| CARD           |       14.7% |
| UPI            |       18.9% |

The monthly revenue analysis also distinguishes the apparent January peak from the validated peak after quantity-outlier review. The true peak month is **March 2026**, with revenue of **₹20,318.90**.

---

# 3. Business Narrative Generation

The narrative stage reads the structured findings produced by Task 5:

```text
narrator/findings.json
```

and generates a three-part Situation–Complication–Resolution business narrative.

Run:

```bash
python narrator/generate_narrative.py
```

The script supports two modes:

1. **Online Gemini generation** when a Gemini API key is available.
2. **Deterministic offline generation** when no API key is available or the online request fails.

## Option A: Run with a Gemini API key

The recommended approach is to store the key as an environment variable rather than putting it directly into the source code.

### Linux/macOS

Set the environment variable in the terminal:

```bash
export GEMINI_API_KEY="your_api_key_here"
```

Then run:

```bash
python narrator/generate_narrative.py
```

### Windows PowerShell

Set the environment variable with:

```powershell
$env:GEMINI_API_KEY="your_api_key_here"
```

Then run:

```powershell
python narrator/generate_narrative.py
```

The Python script reads the key with:

```python
os.getenv("GEMINI_API_KEY")
```

Do **not** commit the API key to Git or place it directly in `generate_narrative.py`.

When the key is available, the script attempts the Gemini generation path. The model is instructed to use only the supplied values in `findings.json` and to return exactly three sections:

```text
Situation
Complication
Resolution
```

The generated narrative is saved to:

```text
narrator/sample_output.txt
```

## Option B: Run with no API key

A Gemini API key is **not required** to reproduce the project.

Simply run:

```bash
python narrator/generate_narrative.py
```

without setting `GEMINI_API_KEY`.

The script detects that the environment variable is missing and uses the deterministic offline fallback.

The offline path reads the values directly from:

```text
narrator/findings.json
```

and creates the same required three-section structure without making a network/API request.

This makes the project reproducible even when:

* No Gemini account is configured.
* No API key is available.
* Internet access is unavailable.
* The Gemini request fails.

The offline result is also written to:

```text
narrator/sample_output.txt
```

---

# End-to-End Reproduction

A new user can reproduce the project from start to finish in this order:

## 1. Run the SQL workflow

```bash
psql -d <database_name>
```

Then:

```sql
\i sql/schema.sql
\i sql/seed_data.sql
\i sql/reports.sql
```

## 2. Install Python dependencies

From the project root:

```bash
pip install -r requirements.txt
```

## 3. Run cleaning and EDA

```bash
python analysis/clean_and_eda.py
```

This creates:

```text
narrator/findings.json
```

## 4. Run visualizations

```bash
python analysis/visualize.py
```

This creates/updates the files under:

```text
visualizations/
```

## 5. Generate the narrative

### With Gemini

Set the environment variable first:

```bash
export GEMINI_API_KEY="your_api_key_here"
```

Then:

```bash
python narrator/generate_narrative.py
```

### Without Gemini

Simply run:

```bash
python narrator/generate_narrative.py
```

The script automatically uses the offline fallback.

## Reproducible Results

Following the workflow above should reproduce the validated findings used in the project brief, including:

* **₹97,358.30** cleaned total revenue.
* **₹2,501.90** duplicate reconciliation delta.
* **44.4%** COD return rate.
* **54.5%** return rate for the highest-risk COD + Tier-2 segment.
* **March 2026** as the true peak month.
* **₹20,318.90** revenue in March 2026.

The January revenue figure of **₹29,582.10** is the apparent monthly total before quantity-outlier correction; the corrected January figure is **₹11,637.10**. The difference is why March, rather than January, is treated as the validated peak month.

The final narrative is stored in:

```text
narrator/sample_output.txt
```

so the project can be reviewed without requiring a live Gemini API call.
