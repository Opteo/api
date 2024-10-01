<h1 align="center">Opteo API</h1>

<p align="center">
  <a href="https://opteo.com">
    <img src="https://app.opteo.com/img/opteo-icon-120.png" width="60">
  </a>
</p>

The Opteo API allows you to retrieve and modify your Opteo account data programmatically, for example listing improvements or updating the budget of an account. The API is based on [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) principles.

> ⚠️ The Opteo API is currently invite only and still a work in progress

## Table of Contents

- [Introduction](#introduction)
- [Authentication](#authentication)
- [Errors](#errors)
- [Customers](#customers)
  - [List](#list-customers)
- [Budgets](#budgets)
  - [Get](#get-a-budget)
  - [Update](#update-a-budget)
- [Improvements](#improvements)
  - [Get active](#get-active-improvements)
  - [Get completed](#get-completed-improvements)
- [Reports](#reports)
  - [Get all reports](#get-all-reports)
  - [Get PDF](#get-a-report-pdf)
- [GAQL](#gaql)

## Introduction

**Base URL**: https://api.opteo.dev

You must specify the API version and base customer resource when building request URLs for your desired endpoint. For example:

```
https://api.opteo.dev/v0/customers/{customer-id}/{resource}
```

Note: the customer id refers to the Google Ads customer id. [This is found in Google Ads, not Opteo](https://support.google.com/google-ads/answer/1704344?hl=en).

---

## Authentication

To start using the API you will need to correctly authenticate all requests.

> As the API is still invite only, authentication tokens are provided by the team upon request. If you don't have an API key and want to start using the Opteo API, [please contact our support team.](mailto:support@opteo.com)

To authenticate a request, you must use bearer auth and set a `Authorization` header on your HTTP request.

**cURL**

```bash
curl -i \
    -H "Accept: application/json" \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer {TOKEN}" \
    https://api.opteo.dev/v0/...
```

**JavaScript**

```js
const rawResponse = await fetch("https://api.opteo.dev/v0/...", {
  method: "POST",
  headers: {
    Accept: "application/json",
    "Content-Type": "application/json",
    Authorization: "Bearer {TOKEN}"
  },
  body: JSON.stringify(...),
});
const content = await rawResponse.json();
```

All API requests must be made over HTTPS. Calls made over plain HTTP will fail.

## Errors

The Opteo API uses standard [HTTP response codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status). All successful requests will return a `200` status code and any errors will be in the range `400` (client) to `500` (server).

All failing requests will return the `status` code and a descriptive error `message`. For example, if you attempt to access resources for a non-active customer:

```json
{
  "status": 403,
  "message": "Forbidden - Client not active in Opteo, this may be due to a billing problem, or because it isn't selected in Linked Accounts"
}
```

## Customers

### List customers

Get the list of customers (aka google ads accounts) that are accessible using the supplied API token. These customers must be [linked in Opteo](https://opteo.com/docs/en/articles/823760-link-unlink-accounts-to-your-account-centre).

**URL**

```
GET https://api.opteo.dev/v0/customers
```

**Response**

```json
{
  "status": 200,
  "data": [
    {
      "customerId": "123-456-7890",
      "name": "Plumbers United"
    },
    {
      "customerId": "098-765-4321",
      "name": "Electricians Online"
    }
  ]
}
```

## Budgets

### Get a budget

Get the set Opteo budget for a connected Google Ads account. No ID is required to get the budget.

**URL**

```
GET https://api.opteo.dev/v0/customers/{customer-id}/budget
```

**Response**

```json
{
  "status": 200,
  "data": {
    "lastUpdated": "2021-09-09T15:02:24.000Z",
    "budget": 100
  }
}
```

In the case where no budget has ever been set in Opteo, the `budget` will be `0` and `lastUpdated` will be `null`.

### Update a budget

Update the Opteo budget for a connected Google Ads account. You must specify the new `budget` in the JSON request body.

**URL**

```
POST https://api.opteo.dev/v0/customers/{customer-id}/budget
```

**Parameters**

```json
{
  "budget": 123.45
}
```

**Response**

```json
{
  "status": 200,
  "data": {
    "lastUpdated": "2021-09-09T15:02:30.000Z",
    "budget": 123.45
  }
}
```

## Improvements

### Get active improvements

Get a list of improvements that are currently available for a linked Google Ads account. This list
will reflect the contents of the "Active" tab in the Opteo interface, and will be sorted by "priority", which is a rough measure of an improvement's impact to the account.

Improvements that are completed or dismissed will not be returned.

**URL**

```
GET https://api.opteo.dev/v0/customers/{customer-id}/improvements
```

**Response**

```javascript
{
  "status": 200,
  "data": [
    {
      "improvement_id": 1234,
      "url": "https://app.opteo.com/user/:user_id/account/4567/improvements/active/1234",
      "title": "Write New Ad", // Title of the improvement. Is sometimes dynamic based on the contents of the improvement.
      "type": "ad_comparison_v2", // Type of the improvement, as an enum.
      "rec_action": "write_ad_v2", // The action that we're recommending, as an enum.
      "created_ts": "2022-03-15T04:44:35.000Z", // When the improvement was created.
      "last_updated_ts": "2022-04-12T09:30:35.000Z", // When the improvement was last updated with new data.
      "location": [
        // The location in the Google Ads account that the improvement is concerned with.
        {
          "entity": "campaign",
          "label": "My Campaign Name"
        },
        {
          "entity": "adgroup",
          "label": "My Adgroup Name"
        },
        // The "text_description" and "text_variables" are designed to help surface key
        // information about an improvement.
        "text_description": "The ad group My Ad Group only contains one ad. Create a new ad for this ad group so you can test for the best-performing creative approach.",
        "text_variables": {
          "Campaign": "My Campaign",
          "Ad Group": "My Ad Group",
        },
      ]
    },
    ...
  ]
}
```

### Get completed improvements

Get a list of improvements that have been completed for a linked Google Ads account.

This list will reflect the contents of the "Completed" tab in the Opteo interface.

It is sorted by `completed_ts`, which is the time when the improvement was pushed (descending).

**URL**

```
GET https://api.opteo.dev/v0/customers/{customer-id}/improvements/completed
```

**Response**

```javascript
{
  "status": 200,
  "data": [
    {
      "improvement_id": 5678,
      "url": "https://app.opteo.com/user/:user_id/account/1234/improvements/completed/5678",
      "title": "Add New Keyword", // Title of the improvement. Is sometimes dynamic based on the contents of the improvement.
      "type": "sqr", // Type of the improvement, as an enum.
      "rec_action": "check_query_relevance", // The action that we're recommending, as an enum.
      "completed_ts": "2021-09-07T14:00:50.000Z", // When the improvement was pushed.
      // The location in the Google Ads account that the improvement is concerned with.
      "location": [
        {
          "entity": "search-query",
          "label": "hotels near me"
        }
      ],
      // The "text_description" and "text_variables" are designed to help surface key
      // information about an improvement.
      "text_description": "The search term `hotels near me` is triggering your ads, but has not been added to your account as a keyword. You may want to add `hotels near me` as a new keyword or a negative keyword.",
      "text_variables": {
        "Cost": "$12.00",
        "Conversions": "2",
        "CPA": "$6.00"
      },
      "completed_by_email": "steven@agency.com"
    },
    ...
  ]
}
```

## Reports

### Get all reports

Get a list of all reports for all linked accounts.
Each item in the list will include a `pdfPath`, which
can be used to fetch the report's PDF export.

Archived or deleted reports will not be returned.

**URL**

```
GET https://api.opteo.dev/v0/reports
```

**Response**

```javascript
{
  "status": 200,
  "data": [
    {
      "accountId": 1234,
      "accountName": "Plumbers Ltd",
      "title": "Google Ads Report",
      "fromDate": "2023-04-01",
      "toDate": "2023-04-30",
      "pdfPath": "/v0/reports/pdf/09440bbe-1638-4084-4378-c0d20e3ffefe"
    },
    {
      "accountId": 1234,
      "accountName": "Plumbers Ltd",
      "title": "Google Ads Report",
      "fromDate": "2023-03-01",
      "toDate": "2023-03-30",
      "pdfPath": "/v0/reports/pdf/daf46eb4-4901-45c4-9e75-de58db6effff"
    },
    {
      "accountId": 3456,
      "accountName": "Electricians Online",
      "title": "Google Ads Report",
      "fromDate": "2023-04-01",
      "toDate": "2023-04-30",
      "pdfPath": "/v0/reports/pdf/f1be9043-17a0-4a3d-8689-91d40deeffff"
    }
  ]
}
```

### Get a report PDF

Get the rendered PDF for a single report.

**URL**

```
GET https://api.opteo.dev/v0/reports/pdf/{report-uuid}
```

**Response**

```
The PDF will be returned as a binary response with the `Content-Type` header set to `application/pdf`.
```

## GAQL

This API supports making [GAQL queries](https://developers.google.com/google-ads/api/docs/query/overview) to retrieve arbitrary data from Google Ads. GAQL queries are similar to SQL queries, but are specific to the Google Ads API.

- For details on how to write GAQL queries, see the [GAQL reference](https://developers.google.com/google-ads/api/docs/query/overview).
- For a complete list of what can be fetched via GAQL, see the [Google Ads API reference](https://developers.google.com/google-ads/api/fields/v14/overview).
- In the example below, we use [the `campaign` resource](https://developers.google.com/google-ads/api/fields/v14/campaign).

**URL**

```
GET https://api.opteo.dev/v0/customers/{customer-id}/gaql?query={YOUR_GAQL_QUERY}
```

**Parameters**

The `query` parameter is required and should be a valid GAQL query. The query should be URL encoded.

For example:

query: `SELECT campaign.name, campaign.id, metrics.impressions FROM campaign`

GET request: `https://api.opteo.dev/v0/customers/1234567890/gaql?query=SELECT%20campaign.name,%20campaign.id,%20metrics.impressions%20FROM%20campaign`

**Response**

```javascript
{
  "status": 200,
  "data": [
    {
      "campaign": {
        "name": "First Campaign",
        "id": 685312434,
        "resource_name": "customers/1234567890/campaigns/685312434"
      },
      "metrics": {
        "impressions": 5951
      }
    },
    {
      "campaign": {
        "name": "Second Campaign",
        "id": 685608890,
        "resource_name": "customers/1234567890/campaigns/685608890"
      },
      "metrics": {
        "impressions": 68004
      }
    },
  ]
}
```

## Alerts

### Get all alerts

Get a list of all available alerts for a linked Ads accounts.

This list will reflect the contents of the "Alerts" notifications panel in the Opteo interface.

The list is sorted by `created_ts` (descending).

Returns latest alerts generated in the last 90d, up to 500.

**URL**

```
GET https://api.opteo.dev/v0/alerts
```

**Response**

```javascript
{
	"status": 200,
	"data": [
		{
			"accountId": "123-456-7891", // Google Ads account Id
			"alertId": "f24579d8-5bc9-4260-863e-fb9bf1ed2cc5", // Opteo alert id
			"url": "https://app.opteo.com/user/:user_id/alerts/f24579d8-5bc9-4260-863e-fb9bf1ed2cc5",//Url to the alert in the Opteo UI
			"type": "conversion", // Alert type
			"created_ts": "2024-09-23T09:39:17.000Z", // Alert creation date
			"text_description": "Last week the `Brand Campaigns` campaign group earned a record `254` conversions. That roughly equates to a `25%` improvement when compared with an average week. Congratulations!" // text description of the alert. Matches the Slack notification text.
		},
		{
		  "accountId": "423-456-7892",
			"alertId": "28e4f436-000e-4f66-8be3-252ba88b4f28",
			"url": "https://app.opteo.com/user/:user_id/alerts/28e4f436-000e-4f66-8be3-252ba88b4f28",
			"type": "delta",
			"created_ts": "2024-09-18T07:11:19.000Z",
			"text_description": "Opteo detected an unexpected change in the campaign `Best Campaign`. Between `September 16th` and `September 17th`, cost increased by `94.8%`. This change could be due to seasonality or because of a recent adjustment you made. If something seems out of place, consider checking up on this campaign."
		}
  ]
}
```

### Get all alerts for an account

Get a list of all available alerts for your linked Google Ads account.

This list will reflect the contents of the "Alerts" notifications panel in the Opteo interface for the specified account.

The list is sorted by `created_ts` (descending).

Returns latest alerts generated in the last 90d, up to 500.

**URL**

```
GET https://api.opteo.dev/v0/customers/123-456-7891/alerts
```

**Response**

```javascript
{
	"status": 200,
	"data": [
		{
			"accountId": "123-456-7891",
			"alertId": "f24579d8-5bc9-4260-863e-fb9bf1ed2cc5",
			"url": "https://app.opteo.com/user/:user_id/alerts/f24579d8-5bc9-4260-863e-fb9bf1ed2cc5", 
			"type": "conversion",
			"created_ts": "2024-09-23T09:39:17.000Z",
			"text_description": "Last week the `Brand Campaigns` campaign group earned a record `254` conversions. That roughly equates to a `25%` improvement when compared with an average week. Congratulations!"
		},
		{
		  "accountId": "123-456-7891",
			"alertId": "28e4f436-000e-4f66-8be3-252ba88b4f28",
			"url": "https://app.opteo.com/user/:user_id/alerts/28e4f436-000e-4f66-8be3-252ba88b4f28",
			"type": "budget",
			"created_ts": "2024-09-18T07:11:19.000Z",
			"text_description": "So far this August, you have spent about `$13515.94` of your `$18680` monthly budget. You are `94%` through the month, but you have spent about `72.36%` of the budget."
		}
  ]
}
