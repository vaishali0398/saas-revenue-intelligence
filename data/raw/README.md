# CloudFlow SaaS Revenue Intelligence Platform — Synthetic Starter Data

All records are fictional and generated for portfolio / learning use. No real customer or company data is included.

## Files
- customers.csv — CRM customer master (initial/current snapshot)
- customer_updates.csv — synthetic CRM attribute changes for SCD Type 2 history
- products.csv — product catalog and indicative list prices
- contracts.csv — subscription contracts; monthly_recurring_amount is already normalized to monthly USD
- usage_monthly.csv — monthly product usage
- invoices.csv — monthly or annual invoices
- payments.csv — payment transactions (one invoice may have multiple payments)
- data_dictionary.json — table grains, keys and column descriptions

## Coverage
- Customers: 100
- Products: 4
- Contract history starts during 2025 and extends through October 2026
- Usage / billing periods: January 2025 through October 2026 where a contract is active

## Intentional data quality defects
The usage_monthly.csv file contains a small number of deliberately introduced defects:
1. One duplicated usage record.
2. One missing customer_id.
3. One negative event_count.

These are intentionally retained in the raw source so the Bronze-to-Silver pipeline can demonstrate detection, quarantine and remediation. Do not silently delete them during ingestion.

## Important modeling notes
- Use contract_start_date and contract_end_date to calculate whether a contract is active for a given reporting month.
- Do not rely only on the source contract_status field for historical as-of reporting.
- monthly_recurring_amount is the normalized monthly recurring value even for annual billing contracts.
- Annual invoices are issued annually and have invoice_amount = monthly_recurring_amount * 12. Therefore, do not calculate MRR by summing invoice_amount.
- Payments are cash collection events and must not be treated as recurring revenue.
- Currency is USD in this first release to keep the initial model focused. Multi-currency conversion can be a later enhancement.
- Churn and NRR require explicit month-over-month definitions and should be implemented in Gold models using contract history.

## Suggested first checks
- customers.customer_id is unique and not null.
- customer_updates.source_update_id is unique and customer_id resolves; apply updates in effective_date order to build SCD Type 2 history.
- products.product_id is unique and not null.
- contracts.contract_id is unique and customer_id/product_id resolve.
- usage_monthly.usage_id is unique; contract_id + usage_month should be reviewed for duplicates.
- usage_monthly.event_count >= 0 and customer_id is not null.
- invoices.invoice_id is unique and contract_id resolves.
- payments.payment_id is unique and invoice_id resolves.
