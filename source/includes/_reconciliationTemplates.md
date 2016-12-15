---
title: Templates for Reconciliation

language_tabs:
  - freemarker

search: true

---

# Templates for Reconciliation

## Introduction

Templates for reconciliation do not follow the same format as [templates for the report exporter](./export_report_template.html). The syntax rules are the same, as they both are based on [Freemarker](http://freemarker.org/docs/). However the data that you can access and how you access it are very different.

For more information beyond this documentation, please contact us at <help@expensify.com>. Enjoy!

## Basic format

> Basic reconciliation template skeleton

```freemarker
<#-- Header Line -->
<#if addHeader>
  <#-- Print header information here -->
</#if>
<#list cards as card, reports>
  <#-- Print information on a per card basis here -->
  <#list reports as report>
    <#-- Print information on a per report basis here -->
    <#list report.transactionList as expense>
      <#-- Print information on a per expense basis here -->
    </#list>
  </#list>
</#list>
<#if addHeader>
  <#-- Print footer information here -->
</#if>
```

Reconciliation templates hold a map of cards to reports (`cards`) as their basic reference data structure. Reports contain transactions, held in the report's `transactionList`. Any transaction that is not reported, but still from a card, will be placed on a report with an id of `0` in that card's set of reports.

In the archetypical example to the right, we iterate over each entry of the `cards` map. For each `card`, we iterate over the `reports` mapped to that card. For each `report` we iterate over each `expense` that belongs to the report. Again, any expenses on a card that are unreported will show up under the report with `id=0`. At any point during each iteration we can print information on a per-card, per-report, or per-expense basis.

## Object Reference

> Default reconciliation template - When you run a reconciliation report in Expensify via Loading Dock in Domain Control, this is the template that we use to generate your reconciliation report:

```freemarker
<#-- Functions -->
<#function quoteCsv string>
   <#if isCsv && string?contains(",") && !string?starts_with("\"")>
       <#return "\"" + string?replace("\"", "\\\"") + "\"">
   </#if>
    <#return string>
</#function>
<#function unescapeHtml string>
    <#return string?replace("&lt;", "<")?replace("&gt;", ">")?replace("&amp;", "&")?replace("&quot;", "\"")?replace("&apos;", "'")>
</#function>
<#-- Header Line -->
<#if addHeader>
    Original Merchant,<#t>
    Modified Merchant,<#t>
    Posted date,<#t>
    Sales date,<#t>
    Modified Sales date,<#t>
    Original Amount,<#t>
    Modified Amount,<#t>
    Currency,Split,<#t>
    Category,<#t>
    Tag,<#t>
    Email,<#t>
    Card name,<#t>
    Expense status,<#t>
    Report name,<#t>
    Report status,<#t>
    Submitted to,<#t>
    Report URL,<#t>
    Comment,<#t>
    External ID,<#t>
    Report ID<#lt>
</#if>
<#list cards as card, reports>
    <#list reports as report>
        <#list report.transactionList as transaction>
            <#-- Assign an expense status based off of the report ID -->
            <#if transaction.reportID lt 0>
                <#assign status = "deleted">
            <#elseif transaction.reportID gt 0>
                <#assign status = "reported">
            <#else>
                <#assign status = "unreported">
            </#if>
            <#-- Do not print the modified created if it is the same as the original -->
            <#if transaction.originalCreated == transaction.modifiedCreated>
                <#assign modifiedCreated = "">
            <#else>
                <#assign modifiedCreated = transaction.modifiedCreated>
            </#if>
            <#-- Only print the reportID if it greater than 0 -->
            <#if report.ID gt 0>
                <#assign reportID = report.ID?c>
            <#else>
                <#assign reportID = "">
            </#if>
            ${quoteCsv(unescapeHtml(transaction.originalMerchant))},<#t>
            ${quoteCsv(unescapeHtml(transaction.modifiedMerchant))},<#t>
            ${transaction.posted},<#t>
            ${transaction.originalCreated},<#t>
            ${modifiedCreated},<#t>
            ${(-transaction.originalAmount/100)?string("0.##")},<#t>
            ${transaction.amountModified?then((-transaction.modifiedAmount/100)?string("0.##"), "")},<#t>
            ${transaction.currency},<#t>
            ${transaction.split?then("YES","NO")},<#t>
            ${quoteCsv(unescapeHtml(transaction.category))},<#t>
            ${quoteCsv(unescapeHtml(transaction.tag))},<#t>
            ${card.owner},<#t>
            ${quoteCsv(unescapeHtml(card.name))},<#t>
            ${status},<#t>
            ${quoteCsv(unescapeHtml(report.name))},<#t>
            ${report.status!""},<#t>
            ${report.managerEmail},<#t>
            ${report.url},<#t>
            ${quoteCsv(unescapeHtml(transaction.comment))},<#t>
            ${quoteCsv(unescapeHtml(transaction.externalID))},<#t>
            ${reportID}<#lt>
        </#list>
    </#list>
</#list>
```

### Card level (card)

Name  | Description
-------- | ---------
bank | The bank or the bank feed of the card (Example: "vcf")
ID | The ID of the card
name | The name of the card, as seen in Expensify
owner | The email address of the Expensify account that owns the card, or that the card is assigned to.

### Report level (report)

Name  | Description
-------- | ---------
currency | The currency of the report.
getApproved() | The approved date of the report, as a string (format `yyyy-MM-dd hh:mm:ss`), or `null` if the report wasn't submitted. 
getSubmitted() | The submitted date of the report, as a string (format `yyyy-MM-dd hh:mm:ss`),  or `null` if the report wasn't submitted.
hasAttachment() | Whether the report contains any attachment.
hasBeenApproved() | Whether or not the report has been approved. 
ID | The ID of the report
isACHReimbursed() | Whether the report has been reimbursed via ACH
isClosed() | Whether or not the report is approved.
isOpen() | Whether or not the report is in the `OPEN` state
isReimbursed() | Whether or not the report is reimbursed.
isReimbursementComplete() | Whether the report reimbursement is complete
isSubmitted() | Whether or not the report is in the `SUBMITTED` state
managerEmail | The report manager's email
name | The name of the report (The report title in Expensify)
policyID | The policy ID of the report
submitterEmail | The email address of the person that created the report.
submitterID | The account ID of the creator of the report
state | The state of the report (Example: `AUTOREIMBURSED`, `BILLING`, `MANUALREIMBURSED`, `OPEN`, `SUBMITTED`)
status | The status of the report (Example: "Open", "Processing", "Approved", "Reimbursed", or "Archived")
total | The report's total, in cents
transactionList | The list of expenses on the report. *See the Expense level reference below for more information on printing information about Expenses.*
url | The url of the report

### Transaction level (expense)

Name  | Description
-------- | ---------
amount | The amount of the expense, in cents
bank | The bank of the expense
billable | Whether or not the expense is billable
cardID | The ID of the card for the expense
category | The category of the expense.
comment | The comment on the expense.
convertedAmount | The converted amount of the expense, in cents
created | The expense's created date, in format `yyyy-MM-dd`
currency | The currency of the expense
currencyConversionRate | The conversion rate of the expense.
externalID | The External ID of the expense.
ID | The ID of the expense
isAmountModified() | Whether or not the expense has a numerical modified amount present.
isTagged() | Whether or not the expense has a tag
originalAmount | The original (non-modified) amount of the expense, in its original currency.
originalCreated | The expense's original created date, in format `yyyy-MM-dd`
originalCurrencyAmount | The amount of the expense, in cents, in its original currency
originalMerchant | The original merchant name of the Expense
maskedPAN | The masked account number of the card for the transaction.
merchant | The merchant of the expense
modifiedAmount | The modified amount of the expense, will be 0 if the amount of the expense has not been modified, or if it has been modified to 0.
modifiedCreated | The expense's modified created date, in format `yyyy-MM-dd`
modifiedMerchant | The modified merchant of the Expense
posted | The Expense's posted date, in format `yyyy-MM-dd`
receiptID | The Receipt ID of the expense
receiptURL | The URL for the receipt attached to the expense.
reimbursable | Whether or not the expense is reimbursable
reportID | The Report ID of the expense
split | Whether or not the transaction originated from a split expense action.
tag | The tag of the expense
taxAmount | The tax amount on the expense.
taxExternalID | The externalID of the tax on the expense.
taxRate | The tax rate on the expense.

## Imports

> You can import extra information using import statements like so:

```
<!-- use _REPLACE_ -->
```

> The bundles currently accessible right now are:

```
<!-- use cardOwnerData -->
<!-- use reportPolicyData -->
<!-- use categoryGLCodeMap -->
<!-- use repordFieldData[reportFieldName1, reportFieldName2] -->
```

In addition to all the data in the `cards` map, you can use import statements in your template to bring in extra data.

Accessible data packages are:

- `cardOwnerData`:  Employee information about the set of card owners.
- `reportPolicyData`: Policy data for all the reports in the reconciliation report.
- `categoryGLCodeMap`: Map of category names to their GL codes.
- `reportFieldsMap`: Map of reportIDs to their report fields under a given report field name.

### Card Owner Data

> You can access your Employee Data under the `cardOwnerData` object:

```
<!-- use cardOwnerData -->
...
<#assign individualEmployeeWorkdayObject = cardOwnerData[report.submitterEmail]!{}>
<#assign individualEmployeeData = individualEmployeeWorkdayObject.data!{}>
EmployeeID: ${individualEmployeeWorkdayObject.employeeID}
Some Custom Field: ${individualEmployeeData["customFieldName1"]}
```

This is for companies that have enterprise employee import set up. With this enabled you can bring in your custom employee information to be used in your reconciliation report.

All custom employee information is stored in a map: `cardOwnerData`. The key-value pairs in the map are the custom field names mapped to their values.

### Report Policy Data

> You can access your relevant Policy Data under the `reportPolicyData` object:

```
<!-- use reportPolicyData -->
...
<#assign policy = reportPolicyData[report.policyID]!{}>
${policy.name!""}
${policy.approver!""}
```

You can load information about the relevant policies of reports included in your reconciliation report. The policy information will be stored in a map: `reportPolicyData`. The map will be keyed by the policyID - which can be accessed by referencing `report.policyID`. The value will contain an object with relevant policy information:

Name  | Description
-------- | ---------
approver | The final approver of the policy, if set.
name | The name of the policy.
owner | The owner of the policy.
outputCurrency | The default currency of the policy.
technicalContact | The technical contact of the policy.

### Category GL Codes

> You can access GL Codes for Categories under the `categoryGLCodeMap` object:

```
<!-- use categoryGLCodeMap -->
...
<#assign categoryGlCode = categoryGLCodeMap[expense.category]!"">
${expense.category} : ${categoryGlCode}
```

You can access the GL codes of expense categories. They will be stored in a map: `categoryGLCodeMap`.  The map will be keyed by the category name, which is accessed normally through `expense.category`. The value will be the GL code for that category name.

### Report Field Data

> You need to include the name of each Report Field you want to bring in via the import like so:

```
<!-- use repordFieldData[reportFieldName1, reportFieldName2] -->
```

> Then, you can access the report field per report via:

```
<#assign reportID = report.ID?c>
reportFieldData.reportFieldName1[reportID]
```

You can access the report fields for reports. The import statement works a bit differently from the regular report information. You need to specify each report field that you want to bring in directly in the import. The `reportFieldData` will contain a map, with each report field name as a key pointing to a map of reportIDs to report field values on those reports.
