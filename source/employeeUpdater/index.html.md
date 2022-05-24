---
title: Advanced employee updater

language_tabs:
  - json
  - shell

search: false
---

# Advanced employee updater introduction

The advanced employee updater is designed to allow dynamic and customizable employee provisioning and de-provisioning in Expensify. It will allow you to:

   - Provision employees into Expensify policies based on customizable fields, such as employee department, country, job code, etc.
   - De-provision employees from Expensify, based on customizable fields, such as termination date
   - Assign employees to Expensify domain groups based on customizable fields, such as employee department, country, job code, etc.
   - Automatically invite managers to policies they have subordinates on
   - Automatically detect employee email address changes, and merge both addresses into the same account
   - Import additional information from the employee feed into Expensify, to reuse for custom data export

<aside class="notice">
This document assumes that you are already familiar with the Expensify API. To get more general information on authentication and request formats, please check out <a href="../">this documentation</a>.
</aside>

# API Principles

> Employee feed examples with various amounts of information provided:

```json
{
    "Employees": [
        {
            "employeeEmail": "employee@domain.com",
            "managerEmail": "manager@domain.com",
            "policyID": "0123456789ABCDEF",
            "employeeID": "12345"
        },
        {
            "employeeEmail": "manager@domain.com",
            "managerEmail": "ceo@domain.com",
            "policyID": "0123456789ABCDEF",
            "employeeID": "34567"
        }
    ]
}
```

```json
{
    "Employees": [
        {
            "employeeEmail": "employee@domain.com",
            "managerEmail": "manager@domain.com",
            "policyID": "0123456789ABCDEF",
            "employeeID": "12345",
            "firstName": "John",
            "lastName": "Doe",
            "customField2": "ABC123",
            "approvalLimit": 12300,
            "overLimitApprover": "audit@domain.com",
            "isTerminated": false
        },
        {
            "employeeEmail": "manager@domain.com",
            "managerEmail": "ceo@domain.com",
            "policyID": "0123456789ABCDEF",
            "employeeID": "34567",
            "firstName": "Michael",
            "lastName": "Scott",
            "customField2": "BCD345",
            "isTerminated": false
        }
    ]
}
```

The job uses a JSON list of objects representing employees to provision.

### Required employee fields

Field name | Type | Description
---------- | ---- | -----------
employeeEmail | String | The email address of the employee
managerEmail | String | Who the employee should submit reports to
employeeID | String | Unique, constant external identifier of the employee. It is used to detect email address changes
policyID | String | The ID of the policy the employee should be invited to

### Optional employee fields

Field name | Type | Description
---------- | ---- | -----------
firstName | String | First name of the employee in Expensify. Does not overwrite values manually set by the employee in their Expensify account
lastName | String | Last name of the employee in Expensify. Does not overwrite values manually set by the employee in their Expensify account
customField2 | String | Custom field, displayed in the "Custom Field 2"
approvalLimit | Integer | If passed, determines who the employee should forward the report to when they approve reports over `overLimitApprover`
overLimitApprover | String | Who the manager should forward reports to if a report is over `approvalLimit`. Required if an `approvalLimit` is specified
limitApprover | String | Alias for `overLimitApprover`
isTerminated | Boolean | If set to true, the employee will be removed from the `policyID`
approvesTo | String | If a valid email address is passed, determines who the employee should forward the report to
role | String | One of `user`, `auditor`, `admin`. If passed, specifies the role of the account in the policy in Expensify. Policy owners and the account running the request cannot have their role demoted from `admin`.

<aside class="notice">
    <strong>Important note on <code>employeeID</code></strong>: the <code>employeeID</code> field is used to detect when an employee's email address changes on your end. We set the new email as the primary login and switch the old email to a <a href="https://community.expensify.com/discussion/4432/how-to-add-a-secondary-login" target="_blank">secondary login</a>. Be careful using production employee IDs for testing purposes. Additionally, this field will automatically populate Custom Field 1 with its value in the Policy Members table for a given user, so ensure Custom Field 1 is not utilized for other data in the policy before implementing the Advanced Employee Updater.
</aside>

# Request format

> Request payload

> - Via URL

```json
{
    "type": "update",
    "dry-run" : false,
    "credentials": {
        "partnerUserID": "aa_api_domain_com",
        "partnerUserSecret": "xxx",
        "feedUrl": "https://somedomain.com/path/to/employeeData.json",
        "feedUser": "expensify",
        "feedPassword": "xxx"
    },
    "dataSource" : "download",
    "inputSettings": {
        "type": "employees",
        "entity": "generic"
    },
    "onFinish":[
        {"actionName": "email", "recipients":"admin1@domain.com,admin2@domain.com"}
    ]
}
```

```shell
curl -X POST 'https://integrations.expensify.com/Integration-Server/ExpensifyIntegrations' \
    -d 'requestJobDescription={
        "type": "update",
        "dry-run" : false,
        "credentials": {
            "partnerUserID": "aa_api_domain_com",
            "partnerUserSecret": "xxx",
            "feedUrl": "https://somedomain.com/path/to/employeeData.json",
            "feedUser": "expensify",
            "feedPassword": "xxx"
        },
        "dataSource" : "download",
        "inputSettings": {
            "type": "employees",
            "entity": "generic"
        },
        "onFinish":[
            {"actionName": "email", "recipients":"admin1@domain.com,admin2@domain.com"}
        ]
    }'
```

> - Directly in the request (parameter `data`)

```json
{
    "type": "update",
    "dry-run" : false,
    "credentials": {
        "partnerUserID": "aa_api_domain_com",
        "partnerUserSecret": "xxx"
    },
    "dataSource" : "request",
    "inputSettings": {
        "type": "employees",
        "entity": "generic"
    },
    "onFinish":[
        {"actionName": "email", "recipients":"admin1@domain.com,admin2@domain.com"}
    ]
}
```

```shell
curl -X POST 'https://integrations.expensify.com/Integration-Server/ExpensifyIntegrations' \
    -d 'requestJobDescription={
        "type": "update",
        "dry-run" : false,
        "credentials": {
            "partnerUserID": "aa_api_domain_com",
            "partnerUserSecret": "xxx"
        },
        "dataSource" : "request",
        "inputSettings": {
            "type": "employees",
            "entity": "generic"
        },
        "onFinish":[
            {"actionName": "email", "recipients":"admin1@domain.com,admin2@domain.com"}
        ]
    }' \
    --data-urlencode 'data@myEmployeeData.json'
```

> - Via SFTP

```json
{
    "type": "update",
    "dry-run" : false,
    "credentials": {
        "partnerUserID": "aa_api_domain_com",
        "partnerUserSecret": "xxx",
        "sftp": {
            "host": "sftp.domain.com",
            "login": "expensify",
            "password": "[xxx]",
            "port": 22,
            "filename": "/path/to/employeeFile.json"
        }
    },
    "dataSource" : "sftp",
    "inputSettings": {
        "type": "employees",
        "entity": "generic"
    },
    "onFinish":[
        {"actionName": "email", "recipients":"admin1@domain.com,admin2@domain.com"}
    ]
}
```

```shell
curl -X POST 'https://integrations.expensify.com/Integration-Server/ExpensifyIntegrations' \
    -d 'requestJobDescription={
        "type": "update",
        "dry-run" : false,
        "credentials": {
            "partnerUserID": "aa_api_domain_com",
            "partnerUserSecret": "xxx",
            "sftp": {
                "host": "sftp.domain.com",
                "login": "expensify",
                "password": "[xxx]",
                "port": 22,
                "filename": "/path/to/employeeFile.json"
            }
        },
        "dataSource" : "sftp",
        "inputSettings": {
            "type": "employees",
            "entity": "generic"
        },
        "onFinish":[
            {"actionName": "email", "recipients":"admin1@domain.com,admin2@domain.com"}
        ]
    }'
```

The API call should be made to the regular endpoint `https://integrations.expensify.com/Integration-Server/ExpensifyIntegrations`. There are three ways to pass the employee data:

- Directly as a parameter of the request
- As a URL, that Expensify will load the data from (using Basic Authentication)
- Hosted on a SFTP server, that Expensify will load the data from

### `requestJobDescription` format

Name | Format | Valid values | Description
--------- | --------- | --------- | ---------
type | String | `update` | |
credentials | JSON object | | Contains your Expensify [API credentials](../#authentication), and information to access the feed by URL if needed.
dataSource | String | `download`, `request`, or `sftp` | Dictates how Expensify should retrieve your employee feed.
inputSettings | JSON object | See `inputSettings` below. |
**Optional elements** |
dry-run | Boolean | true, false | If set to true, employees will not actually be provisioned or updated. Use this for development. |
onFinish | JSON array | See onFinish | You can configure the `recipients` list of email addresses that should receive a summary email of the changes made by the API.
setEmployeePrimaryPolicy | String | `none` (default), `new_employees`, `all_employees` | `none`: we will never update an employee's primary policy.<br>`new_employees`: we will set an employee's primary policy to the `policyID` parameter if they were not a member of that policy yet.<br>`all_employees`: we will update the primary policy for every employee.

- `inputSettings`

Name | Format | Valid values | Description
--------- | --------- | --------- | ---------
type | String | `employees` |
entity | String |  | Should always be set to `generic`.

# Response format

```json
{
    "responseCode": 200,
    "dry-run": false,
    "updatedEmployeesCount": 3,
    "diff": {
        "diffToAdd": {
            "0123456789ABCDEF": [
                "employee1@domain.com",
                "employee2@domain.com"
            ],
            "ABCDEF0123456789": [
                "employee3@domain.com"
            ]
        },
        "diffToRemove": {
            "B1C7903C636F4A51": [
                "terminatedEmployee@domain.com"
            ]
        }
    },
    "securityGroupEmployeesMap": {
        "407184": [
            "employee1@domain.com",
            "employee2@domain.com"
        ],
        "830936": [
            "employee3@domain.com"
        ]
    },
    "skippedEmployees": [
        {
            "email": "employee6@domain.com",
            "reason": "No policy found for 'Marketing'"
        },
        {
            "email": "employee7@domain.com",
            "reason": "Invalid manager email address 'manager@domain '"
        }
    ]
}
```

#### Response codes

Response code | Meaning
----- | -----
200 | Success
410 | Invalid input data, e.g. employee data is missing
500 | Other

#### Response data

Key | Meaning
----- | -----
`dry-run` | Whether the job was run in dry-run mode, meaning employee data was not actually updated in Expensify
`updatedEmployeesCount` | Number of employees that were or would have been updated, depending on if the job was run in dry-run mode
`diff`, `diffToAdd`, `diffToRemove` | Email addresses of employees that were or would have been invited or removed from policies, grouped by policy IDs
`securityGroupEmployeesMap` | Email addresses of employees that were or would have been assigned to Domain Groups, grouped by domain group IDs
`skippedEmployees` | List of employees that could not be updated, with a reason why
