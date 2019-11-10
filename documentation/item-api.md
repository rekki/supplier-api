# REKKI - ITEM API

Suppliers can use the Item API to manage their item catalog.

The base URL for all API endpoints is **`https://backend.live.rekki.com`**.

### Authentication

Requests _must_ be authenticated via an authorization header <strong><code>Authorization: Bearer <em>TOKEN</em></code></strong>.
Unauthenticated requests will receive a `401 Unauthorized` response.

Contact integrations@rekki.com for an API token.

### Parameters

Some endpoints may take parameters in the URL, such as segments in the path or key-values in a query string.
For example `GET https://example.com/api/resource/:id?limit=10`.

Some endpoints may take parameters outside the URL, typically `POST`, `PATCH`, `PUT`, and `DELETE` requests.
These requests _must_ specify a JSON encoding with a content-type header <strong>`Content-Type: application/json`</strong>.
Unencoded requests to endpoints that expect an encoding for the payload, will receive a `400 Bad Request` response.
Requests with an encoding that's not supported by the endpoint, will receive a `406 Not Acceptable` response.

All endpoints return data in JSON format. Media types specifed in the `Accept` header, if provided, will be ignored.

---

## LIST ITEMS
### `GET /api/catalog/integration/v1/items`
<!-- <details><summary>Show details</summary> -->

Lists all your catalog items. Sorted by creation date, with the oldest appearing first.

### Parameters

None. Options for pagination and sorting direction may be introduced later.

### Response

Status: `200 OK`

Body: JSON object with a `data` property that contains catalog items of the authenticated supplier.

- **`id`**
  REKKI's ID to uniquely identify the catalog item (for REKKI internal reference).
- **`product_code`**
  Product code for the item that maps to the supplier's catalog.
  Suppliers can modify the product code for future orders at https://tulip.rekki.com
- **`name`**
  Item name as would be defined on the customer's product list.
- **`currency`**
  Currency code for the price. In [ISO 4217][] three-letter format. Defaults to GBP.
- **`units_prices`**
  List of units and their prices that the item can be ordered in.
- **`units_prices.unit`**
  A unit that the item can be ordered in.
- **`units_prices.price_cents`**
  The order price in cents for the item per unit.
  For example, a currency of GBP with unit 5L and price 850 means a 5L item can be ordered for £8.50.
  Items without price data will show a default placeholder of 0.
- **`created_at`**
  Datetime when the catalog item was created on REKKI.
  In [ISO 8601][] UTC format `YYYY-MM-DDTHH:MM:SSZ`.
- **`updated_at`**
  Datetime when the catalog item was last updated on REKKI.
  In [ISO 8601][] UTC format `YYYY-MM-DDTHH:MM:SSZ`.

### Example Request

```bash
curl -X GET "https://backend.live.rekki.com/api/catalog/integration/v1/items" \
     -H "Authorization: Bearer $API_TOKEN"
```

### Example Response

```json
{
  "data": [
    {
      "id": 44285,
      "product_code": "2026",
      "name": "Bear Multipack Yo Yos Mango (6 x 5x20G)",
      "currency": "EUR",
      "units_prices": [
        {
          "unit": "Case",
          "price_cents": 0
        }
      ],
      "created_at": "2019-11-04T08:38:48Z",
      "updated_at": "2019-11-04T08:38:57Z"
    },
    {
      "id": 34858,
      "product_code": "2101",
      "name": "Premium Cooks Vegetable Oil",
      "currency": "GBP",
      "units_prices": [
        {
          "unit": "5L",
          "price_cents": 800
        },
        {
          "unit": "10L",
          "price_cents": 1200
        }
      ],
      "created_at": "2019-10-11T09:26:13Z",
      "updated_at": "2019-10-11T09:26:13Z"
    }
  ]
}
```

</details>


[ISO 4217]: https://en.wikipedia.org/wiki/ISO_4217
[ISO 8601]: https://en.wikipedia.org/wiki/ISO_8601