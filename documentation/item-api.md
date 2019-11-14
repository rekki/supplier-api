# REKKI - CATALOGUE API

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

### Validation

Client-submitted data goes through several validations.

When validation fails, the response code will be `422 Unprocessable Entity`.
The response body will be a JSON document with an `errors` property that contains a list of error objects.

Each error object will contain a `message` property. Depending on the validation, there may be an optional `uri` property, which represents a [JSON Pointer][] URI fragment to identify where the violation occurred.

For example, a payload such as
```json
{
  "product_code": 123,
  "units_prices": [
    {"price_cents": 100},
    {"price_cents": 1.00, "unit": "kg"},
  ]
}
```

Could produce an error response like

```json
{
  "errors": [
    {
      "uri": "#",
      "message": "Required property name was not present."
    },
    {
      "uri": "#/product_code",
      "message": "Type mismatch. Expected String but got Integer."
    },
    {
      "uri": "#/units_prices/0",
      "message": "Required property unit was not present."
    },
    {
      "uri": "#/units_prices/1/price_cents",
      "message": "Type mismatch. Expected Integer but got Number."
    }
  ]
}
```

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

---

## ADD ITEM
### `POST /api/catalog/integration/v1/items`
<!-- <details><summary>Show details</summary> -->

Creates an item on your catalog.

### Parameters

- **`product_code`**  <span style="font-size: 12px; font-weight: 500;">required</span>  
  Product code for the item that maps to the supplier's catalog.  
  Suppliers can modify the product code for future orders at https://tulip.rekki.com
- **`name`**  <span style="font-size: 12px; font-weight: 500;">required</span>  
  Item name as would be defined on the customer's product list.
- **`currency`**  <span style="font-size: 12px; font-weight: 500;">optional, default is GBP</span>  
  Currency code for the price. In [ISO 4217][] three-letter format.
- **`units_prices`**  <span style="font-size: 12px; font-weight: 500;">required</span>  
  List of units and their prices that the item can be ordered in. 
- **`units_prices.unit`**  <span style="font-size: 12px; font-weight: 500;">required</span>  
  A unit that the item can be ordered in.
- **`units_prices.price_cents`**  <span style="font-size: 12px; font-weight: 500;">optional, default is 0</span>  
  The order price in cents for the item per unit.  
  For example, a currency of GBP with unit 5L and price 850 means a 5L item can be ordered for £8.50.

### Response

Status: `201 Created`

Body: JSON object of the item that was added to the catalog of the authenticated supplier.

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
curl -X POST "https://backend.live.rekki.com/api/catalog/integration/v1/items" \
     -H "Authorization: Bearer $API_TOKEN" \
     -H "Content-Type: application/json" \
     -d @item_data.json
```

Where `item_data.json` contains the payload
```json
{
  "product_code": "C1235",
  "name": "Organic Banana",
  "currency": "GBP",
  "units_prices": [
    {"unit": "each", "price_cents": 100},
    {"unit": "kg", "price_cents": 1000}
  ]
}
```

### Example Response

```json
{
  "id": 44724,
  "product_code": "C1235",
  "name": "Organic Banana",
  "currency": "GBP",
  "units_prices": [
    {
      "unit": "each",
      "price_cents": 100
    },
    {
      "unit": "kg",
      "price_cents": 1000
    }
  ],
  "created_at": "2019-11-10T20:57:22Z",
  "updated_at": "2019-11-10T20:57:22Z"
}
```

</details>

[ISO 4217]: https://en.wikipedia.org/wiki/ISO_4217
[ISO 8601]: https://en.wikipedia.org/wiki/ISO_8601
[JSON Pointer]: https://tools.ietf.org/html/rfc6901#section-6
