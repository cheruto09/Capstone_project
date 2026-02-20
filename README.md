# Capstone_project
The data integrity project
1. Background Story
QuickCart is a fast-growing e-commerce startup. A P0 incident has just occurred: Marketing’s “Total Sales” dashboard does not match the bank settlement statement.

Finance cannot close the month. The CEO has escalated this and given you:

Access to the production database
Raw transaction logs 
5 days to establish a single, bank-reconcilable Source of Truth
Your job is not to build dashboards. Your job is to prove what the real revenue is. 

2. The Core Problems
You are dealing with two broken sources of truth:

A) Raw JSON Transaction Logs (Python Problem)
Data is stored in nested JSON
The Amount field is inconsistent:
“$10.00” (string with symbol)
“10.00” (string without symbol)
1000 (integer in cents)
null, missing, or empty string
Test/sandbox transactions are mixed with real ones
Some records are structurally valid but financially meaningless
B) Production Database (SQL Problem)
orders and payments do not have a clean 1-to-1 relationship
An order may have multiple payment attempts (retries)
Some payments have no associated order (orphans)
Bank settlements may contain:
duplicates
partial settlements
rows without internal payment references
3. Your Objective
You must produce a defensible, auditable, finance-grade reconciliation report that shows:

Total successful sales (deduplicated and cleaned)
Orphan payments (money received but no matching order)
Discrepancy Gap = Expected sales − Bank settled amount
Your result must be explainable to Finance and robust to edge cases.

4. Learning Objectives
Part A — Python: Data Cleaning & Standardization
You must demonstrate:

Navigating nested dictionaries in JSON
Currency normalization
Convert “$10.00”, “10.00”, and 1000 → 10.00 (float, dollars)
Type casting and sanitization
Filtering
Remove test transactions
Drop incomplete or unrecoverable records
Part B — SQL: Data Reconciliation
You must demonstrate:

CTEs (Common Table Expressions) to build staged clean datasets
NULL handling using COALESCE
Window functions
Use ROW_NUMBER() to deduplicate multiple payment attempts per order
Reconciliation logic
Compare internal truth vs bank truth
5. Deliverables
You must submit two files only:

1) clean_transactions.py
A standalone script that:

Reads raw_data.jsonl
Extracts relevant fields from nested JSON
Normalizes all currency formats into a single float column (amount_usd)
Filters out:
test/sandbox transactions
records without valid amounts or identifiers
Outputs a clean dataset (CSV or JSON) for analysis
2) reconciliation.sql
A SQL script that produces:

Total successful sales
List of orphan payments
The discrepancy gap between:
cleaned internal transactions
bank settlements 
3) Write the Raw JSON Transaction Logs to mongodb for archival purposes. 

6. Database Schema
You will be working with these tables:

orders
order_id (TEXT, PK)
customer_id (TEXT)
customer_email (TEXT)
order_total_cents (INT)
currency (TEXT)
is_test (INT)
created_at (TIMESTAMP)
payments
payment_id (TEXT, PK)
order_id (TEXT, nullable)
attempt_no (INT)
provider (TEXT)
provider_ref (TEXT)
status (TEXT: SUCCESS / FAILED / PENDING)
amount_cents (INT)
attempted_at (TIMESTAMP)
bank_settlements
settlement_id (TEXT, PK)
payment_id (TEXT, nullable)
provider_ref (TEXT, nullable)
status (TEXT: SETTLED)
settled_amount_cents (INT)
currency (TEXT)
settled_at (TIMESTAMP)
7. Dataset Generation Instructions
A generator script is provided: generate_quickcart_data.py: https://drive.google.com/file/d/1-zMQsOqNq9Np_zz4KaFkuqI0Y72XYyh4/view?usp=sharing 

Step 1: Generate Data
python generate_quickcart_data.py –outdir quickcart_data
This will create:

quickcart_data/raw_data.jsonl (messy nested logs)
quickcart_data/seed_orders.sql
quickcart_data/seed_payments.sql
quickcart_data/seed_bank_settlements.sql
Default size:

~50,000 orders
~75,000 payments
~70,000 bank settlement rows
8. Load Data into Postgres
First create tables:

psql “$DATABASE_URL” -f schema.sql
Then load data:

psql “$DATABASE_URL” -f quickcart_data/seed_orders.sql
psql “$DATABASE_URL” -f quickcart_data/seed_payments.sql
psql “$DATABASE_URL” -f quickcart_data/seed_bank_settlements.sql
9. Success Criteria
Your solution is correct if:

Your numbers reconcile with the bank within explainable differences
Your logic is explicit and auditable
There are no silent assumptions
Finance could sign off on your result
10. Evaluation Focus
You are being evaluated on:

Data reasoning under ambiguity
Correct use of Python for data cleaning
Correct use of SQL for reconciliation
Ability to build a defensible source of truth
If this breaks in production, the company loses trust in its data. Treat it accordingly.
