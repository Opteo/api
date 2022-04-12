<h1 align="center">Opteo API</h1>

<p align="center">
  <a href="https://opteo.com">
    <img src="https://app.opteo.com/img/opteo-icon-120.png" width="60">
  </a>
</p>

The Opteo API allows you to retrieve and modify your Opteo account data programmatically, for example listing improvements or updating the budget of an account. The API is based on [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) principles.

> ⚠️ The Opteo API is currently invite only and still a work in progress

**Base URL**: https://api.opteo.dev

You must specify the API version and base customer resource when building request URLs for your desired endpoint

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

## Resources

---

## Budgets

### Retrieve a budget

Retrieve the set Opteo budget for a connected Google Ads account. No ID is required to retrieve the budget.

**URL**

```
GET https://api.opteo.dev/v0/customers/{customer-id}/budget
```

**Parameters**

_none required_

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

### Get Active Improvements

Get a list of improvements that are currently available for a linked Google Ads account. This list
will reflect the contents of the "Active" tab in the Opteo interface.

Improvements that are completed or dismissed will no longer be available in this list.

**URL**

```
GET https://api.opteo.dev/v0/customers/{customer-id}/improvements
```

**Parameters**

_none required_

**Response**

```json
{
  "status": 200,
  "data": [
    {
      "improvement_id": 1234,
      "url": "https://app.opteo.com/user/:user_id/domain/4567/improvements/active/1234",
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
        }
      ]
    },
    ...
  ]
}
```

### Get Completed Improvements

_coming soon_
