[![Apache License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0) 
# QuickBooks ([docs](https://dbt-quickbooks.netlify.app/))

This package models QuickBooks data from [Fivetran's connector](https://fivetran.com/docs/applications/quickbooks). It uses data in the format described by [this ERD](https://fivetran.com/docs/applications/quickbooks#schemainformation).

The main focus of this package is to provide users insights into their QuickBooks data that can be used for financial statement reporting and deeper analysis. The package achieves this by:
  - Creating a comprehensive general ledger that can be used to create financial statements with additional flexibility.
  - Providing historical general ledger month beginning balances, ending balances, and net change for each account.
  - Enhancing Accounts Payable and Accounts Receivables data by providing past and present aging of bills and invoices.
  - Pairing all expense and sales transactions in one table with accompanying data to provide enhanced analysis.

## Compatibility

> Please be aware that the [dbt_quickbooks](https://github.com/fivetran/dbt_quickbooks) and [dbt_quickbooks_source](https://github.com/fivetran/dbt_quickbooks_source) packages were developed with single currency company data. As such, the package models will not reflect accurate totals if your QuickBooks account has Multi-Currency enabled.

## Models

This package contains transformation models designed to work simultaneously with our [QuickBooks source package](https://github.com/fivetran/dbt_quickbooks_source). A dependency on the source package is declared in this package's `packages.yml` file, so it will automatically download when you run `dbt deps`. The primary outputs of this package are described below. Intermediate models are used to create these output models.

| **model**                | **description**                                                                                                                                |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| [quickbooks__general_ledger](https://github.com/fivetran/dbt_quickbooks/blob/master/models/quickbooks__general_ledger.sql) | Table containing a comprehensive list of all transactions with offsetting debit and credit entries to accounts. |
| [quickbooks__general_ledger_by_period](https://github.com/fivetran/dbt_quickbooks/blob/master/models/quickbooks__general_ledger_by_period.sql) | Table containing the beginning balance, ending balance, and net change of the dollar amount for each month since the first transaction. This table can be used to generate a balance sheet and income statement for your business. |
| [quickbooks__profit_and_loss](https://github.com/fivetran/dbt_quickbooks/blob/master/models/quickbooks__profit_and_loss.sql) | Table containing all revenue and expense account classes by calendar year and month enriched with account type, class, and parent information. |
| [quickbooks__balance_sheet](https://github.com/fivetran/dbt_quickbooks/blob/master/models/quickbooks__balance_sheet.sql) | Table containing all asset, liability, and equity account classes by calendar year and month enriched with account type, class, and parent information. |
| [quickbooks__ap_ar_enhanced](https://github.com/fivetran/dbt_quickbooks/blob/master/models/quickbooks__ap_ar_enhanced.sql) | Table providing the amount, amount paid, due date, and days overdue of all bills and invoices your company has received and paid along with customer, vendor, department, and address information for each invoice or bill. |
| [quickbooks__expenses_sales_enhanced](https://github.com/fivetran/dbt_quickbooks/blob/master/models/quickbooks__expenses_sales.sql) | Table providing enhanced customer, vendor, and account details for each expense and sale transaction. |

## Installation Instructions
Check [dbt Hub](https://hub.getdbt.com/) for the latest installation instructions, or [read the dbt docs](https://docs.getdbt.com/docs/package-management) for more information on installing packages.

Include in your `packages.yml`

```yaml
packages:
  - package: fivetran/quickbooks
    version: [">=0.5.0", "<0.6.0"]
```

## Configuration

By default, this package looks for your QuickBooks data in the `quickbooks` schema of your [target database](https://docs.getdbt.com/docs/running-a-dbt-project/using-the-command-line-interface/configure-your-profile). 
If this is not where your QuickBooks data is, add the below configuration to your `dbt_project.yml` file.

```yml
# dbt_project.yml

...
config-version: 2

vars:
    quickbooks_database: your_database_name
    quickbooks_schema: your_schema_name
```

### Changing the Build Schema
By default this package will build the QuickBooks staging models within a schema titled (<target_schema> + `_quickbooks_staging`) and QuickBooks final models within a schema titled (<target_schema> + `_quickbooks`) in your target database. If this is not where you would like your modeled QuickBooks data to be written to, add the following configuration to your `dbt_project.yml` file:

```yml
# dbt_project.yml

...
models:
    quickbooks:
      +schema: my_new_schema_name # leave blank for just the target_schema
    quickbooks_source:
      +schema: my_new_schema_name # leave blank for just the target_schema
```
### Disabling models

This package takes into consideration that not every QuickBooks account utilizes the same transactional tables, and allows you to disable the corresponding functionality. By default, most variables' values are assumed to be `true` (with exception of purchase orders). Add variables for only the tables you want to disable or enable respectively:

```yml
# dbt_project.yml

...
vars:
  using_address:        false         #disable if you don't have addresses in QuickBooks
  using_bill:           false         #disable if you don't have bills or bill payments in Quickbooks
  using_credit_memo:    false         #disable if you don't have credit memos in Quickbooks
  using_department:     false         #disable if you don't have departments in Quickbooks
  using_deposit:        false         #disable if you don't have deposits in Quickbooks
  using_estimate:       false         #disable if you don't have estimates in Quickbooks
  using_invoice:        false         #disable if you don't have invoices in Quickbooks
  using_invoice_bundle: false         #disable if you don't have invoice bundles in Quickbooks
  using_journal_entry:  false         #disable if you don't have journal entries in Quickbooks
  using_payment:        false         #disable if you don't have payments in Quickbooks
  using_refund_receipt: false         #disable if you don't have refund receipts in Quickbooks
  using_transfer:       false         #disable if you don't have transfers in Quickbooks
  using_vendor_credit:  false         #disable if you don't have vendor credits in Quickbooks
  using_sales_receipt:  false         #disable if you don't have sales receipts in QuickBooks
  using_purchase_order: true          #enable if you want to include purchase orders in your staging models
```

## Analysis

After running the models within this package, you may want to compare the baseline financial statement totals from the data provided against what you expect. You can make use of the [analysis functionality of dbt](https://docs.getdbt.com/docs/building-a-dbt-project/analyses/) and run pre-written SQL to test these values. The SQL files within the [analysis](https://github.com/fivetran/dbt_quickbooks/blob/master/analysis) folder contain SQL queries you may compile to generate balance sheet and income statement values. You can then tie these generated values to your expected ones and confirm the values provided in this package are accurate.

## Contributions

Don't see a model or specific metric you would have liked to be included? Notice any bugs when installing 
and running the package? If so, we highly encourage and welcome contributions to this package! 
Please create issues or open PRs against `main`. Check out [this post](https://discourse.getdbt.com/t/contributing-to-a-dbt-package/657) on the best workflow for contributing to a package.

## Database Support

This package has been tested on BigQuery, Snowflake, Redshift, and Postgres.

## Resources:
- Provide [feedback](https://www.surveymonkey.com/r/DQ7K7WW) on our existing dbt packages or what you'd like to see next
- Have questions or feedback, or need help? Book a time during our office hours [here](https://calendly.com/fivetran-solutions-team/fivetran-solutions-team-office-hours) or shoot us an email at solutions@fivetran.com.
- Find all of Fivetran's pre-built dbt packages in our [dbt hub](https://hub.getdbt.com/fivetran/)
- Learn how to orchestrate your models with [Fivetran Transformations for dbt Core™](https://fivetran.com/docs/transformations/dbt)
- Learn more about Fivetran overall [in our docs](https://fivetran.com/docs)
- Check out [Fivetran's blog](https://fivetran.com/blog)
- Learn more about dbt [in the dbt docs](https://docs.getdbt.com/docs/introduction)
- Check out [Discourse](https://discourse.getdbt.com/) for commonly asked questions and answers
- Join the [chat](http://slack.getdbt.com/) on Slack for live discussions and support
- Find [dbt events](https://events.getdbt.com) near you
- Check out [the dbt blog](https://blog.getdbt.com/) for the latest news on dbt's development and best practices
