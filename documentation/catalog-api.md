# REKKI - CATALOGUE API

- [LIST ITEMS](#list-items)
- [RETRIEVE ITEM](#retrieve-item)
- [ADD ITEM](#add-item)
- [UPDATE ITEM](#update-item)

---

Suppliers can use the Item API to manage their item catalog.

The base URL for all API endpoints is **`https://backend.live.rekki.com`**.

### Authentication
<!-- <details><summary>Show details</summary> -->

Requests _must_ be authenticated via an authorization header <strong><code>Authorization: Bearer <em>TOKEN</em></code></strong>.  
Unauthenticated requests will receive a `401 Unauthorized` response.

Requests _must_ also specify the token type via the header <strong><code>X-REKKI-Authorization-Type: supplier_api_token</code></strong>.

Contact integrations@rekki.com for an API token.
</details>

### Parameters
<!-- <details><summary>Show details</summary> -->

Some endpoints may take parameters in the URL, such as segments in the path or key-values in a query string.  
For example `GET https://example.com/api/resource/:id?limit=10`.

Some endpoints may take parameters outside the URL, typically `POST`, `PATCH`, `PUT`, and `DELETE` requests.  
These requests _must_ specify a JSON encoding with a content-type header <strong>`Content-Type: application/json`</strong>.  
Unencoded requests to endpoints that expect an encoding for the payload, will receive a `400 Bad Request` response.  
Requests with an encoding that's not supported by the endpoint, will receive a `406 Not Acceptable` response.

All endpoints return data in JSON format. Media types specifed in the `Accept` header, if provided, will be ignored.
</details>

### Validation
<!-- <details><summary>Show details</summary> -->

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
</details>

<sup><a href="#">Back to top ↰</a></sup>

---

## LIST ITEMS
### `GET /api/integration/v1/catalog/items`
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
- **`availability`**  
  Availability status of the item. Defaults to in_stock.
- **`created_at`**  
  Datetime when the catalog item was created on REKKI.  
  In [ISO 8601][] UTC format `YYYY-MM-DDTHH:MM:SSZ`.
- **`updated_at`**  
  Datetime when the catalog item was last updated on REKKI.  
  In [ISO 8601][] UTC format `YYYY-MM-DDTHH:MM:SSZ`.

### Example Request

```bash
curl -X GET "https://backend.live.rekki.com/api/integration/v1/catalog/items" \
     -H "Authorization: Bearer $API_TOKEN" \
     -H "X-REKKI-Authorization-Type: supplier_api_token"
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
      "availability": "in_stock",
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
      "availability": "out_of_stock",
      "created_at": "2019-10-11T09:26:13Z",
      "updated_at": "2019-10-11T09:26:13Z"
    }
  ]
}
```

</details>

<sup><a href="#">Back to top ↰</a></sup>

---

## RETRIEVE ITEM
### `GET /api/integration/v1/catalog/items/:id`
<!-- <details><summary>Show details</summary> -->

Retrieve an item from your catalog by its unique ID.

### Parameters

- **`id`**  <span style="font-size: 12px; font-weight: 500;">required (query parameter)</span>  
  ID of the item to retrieve. Item IDs are discoverable when [listing items](#list-items).

### Response

Status: `200 OK`

Body: JSON object of the item that was retrieved.

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
- **`availability`**  
  Availability status of the item. Defaults to in_stock.
- **`created_at`**  
  Datetime when the catalog item was created on REKKI.  
  In [ISO 8601][] UTC format `YYYY-MM-DDTHH:MM:SSZ`.
- **`updated_at`**  
  Datetime when the catalog item was last updated on REKKI.  
  In [ISO 8601][] UTC format `YYYY-MM-DDTHH:MM:SSZ`.

### Example Request

```bash
curl -X GET "https://backend.live.rekki.com/api/integration/v1/catalog/items/17238" \
     -H "Authorization: Bearer $API_TOKEN" \
     -H "X-REKKI-Authorization-Type: supplier_api_token" \
     -H "Content-Type: application/json"
```

### Example Response

```json
{
  "id": 17238,
  "product_code": "RT1001",
  "name": "Dark Red Kidney Beans",
  "currency": "USD",
  "units_prices": [
    {
      "unit": "2Lbs bags (910g)",
      "price_cents": 759
    }
  ],
  "availability": "discontinued",
  "created_at": "2019-11-10T20:57:22Z",
  "updated_at": "2019-11-10T20:57:22Z"
}
```

<sup><a href="#">Back to top ↰</a></sup>

---

## ADD ITEM
### `POST /api/integration/v1/catalog/items`
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
- **`units_prices.stock_count`**  <span style="font-size: 12px; font-weight: 500;">optional</span>  
  The number of items in stock for the related unit.
- **`availability`**  <span style="font-size: 12px; font-weight: 500;">optional, default is "in_stock"</span>  
  Availability status of the item. Can be "in_stock", "out_of_stock", or "discontinued".
- **`description`**  <span style="font-size: 12px; font-weight: 500;">optional</span>  
  Short description of the item.
- **`allergens`**  <span style="font-size: 12px; font-weight: 500;">optional</span>  
  List of allergens for the item, if any.
- **`allergens.type`**  <span style="font-size: 12px; font-weight: 500;">required when allergens is given</span>  
  Type of allergy. For example "contains peanuts" or "may contain peanuts".
- **`allergens.symptoms`**  <span style="font-size: 12px; font-weight: 500;">optional</span>  
  List of symptoms for the allergy.
- **`order_cutoff_times`**  <span style="font-size: 12px; font-weight: 500;">optional</span>  
  Cutt-off times are the minimum amount of time before delivery when the item can still be ordered.
- **`order_cutoff_times.{mon,tue,wed,thu,fri,sat,sun}`**  <span style="font-size: 12px; font-weight: 500;">optional</span>  
  Minimum amount of time, in hours, that an item needs to be ordered in advance of delivery for the given day.
- **`replacement_products`**   <span style="font-size: 12px; font-weight: 500;">optional</span>  
  List of product codes for alternative items when this item is not available.
- **`seasonality`**   <span style="font-size: 12px; font-weight: 500;">optional</span>  
  List of date ranges when the item is in-season. 
- **`seasonality.start_date`**  <span style="font-size: 12px; font-weight: 500;">required when seasonality is given</span>  
  The start date when the item is in season. In [ISO 8601][] calendar date format `YYYY-MM-DD`.
- **`seasonality.end_date`**  <span style="font-size: 12px; font-weight: 500;">required when seasonality is given</span>  
  The end date when the item is in season. In [ISO 8601][] calendar date format `YYYY-MM-DD`.

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
- **`availability`**  
  Availability status of the item. Defaults to in_stock.
- **`created_at`**  
  Datetime when the catalog item was created on REKKI.  
  In [ISO 8601][] UTC format `YYYY-MM-DDTHH:MM:SSZ`.
- **`updated_at`**  
  Datetime when the catalog item was last updated on REKKI.  
  In [ISO 8601][] UTC format `YYYY-MM-DDTHH:MM:SSZ`.

### Example Request

```bash
curl -X POST "https://backend.live.rekki.com/api/integration/v1/catalog/items" \
     -H "Authorization: Bearer $API_TOKEN" \
     -H "X-REKKI-Authorization-Type: supplier_api_token" \
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
    {"unit": "kg", "price_cents": 1000, "stock_count": 300}
  ],
  "availability": "in_stock",
  "description": "Ripe. Gluten free. Vegan. Organic. Origin Costa Rica",
  "allergens": [
    {
      "type": "Banana allergy",
      "symptoms": [
        "itching of the mouth and throat",
        "itchy rash (hives, urticaria)",
        "skin or mucosal swellings (angioedema)",
        "in rare cases narrowing of the throat, wheezing, and even collapse"
      ]
    }
  ],
  "order_cutoff_times": {
    "mon": 48,
    "tue": 24,
    "wed": 24,
    "thu": 24,
    "fri": 24
  },
  "replacement_products": ["C1236", "C1237", "C1238"],
  "seasonality": [{"start_date": "2020-01-01", "end_date": "2020-12-31"}]
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
  "availability": "in_stock",
  "created_at": "2019-11-10T20:57:22Z",
  "updated_at": "2019-11-10T20:57:22Z"
}
```

</details>

<sup><a href="#">Back to top ↰</a></sup>

---

## UPDATE ITEM
### `POST /api/integration/v1/catalog/items/:id`
<!-- <details><summary>Show details</summary> -->

Updates an item on your catalog.

Only certain properties of an item can be updated.  
An item's product code cannot be updated for example.
Instead the item can be discontinued and a new item can be created.

### Parameters

- **`id`**  <span style="font-size: 12px; font-weight: 500;">required (query parameter)</span>  
  ID of the item to update. Item IDs are discoverable when [listing items](#list-items).
- **`name`**  <span style="font-size: 12px; font-weight: 500;">optional</span>  
  Item name as would be defined on the customer's product list.
- **`currency`**  <span style="font-size: 12px; font-weight: 500;">optional, default is GBP</span>  
  Currency code for the price. In [ISO 4217][] three-letter format.
- **`units_prices`**  <span style="font-size: 12px; font-weight: 500;">optional</span>  
    List of units and their prices that the item can be ordered in. 
- **`units_prices.unit`**  <span style="font-size: 12px; font-weight: 500;">required when changing units_prices</span>  
  A unit that the item can be ordered in.
- **`units_prices.price_cents`**  <span style="font-size: 12px; font-weight: 500;">optional, default is 0</span>  
  The order price in cents for the item per unit.  
  For example, a currency of GBP with unit 5L and price 850 means a 5L item can be ordered for £8.50.
- **`units_prices.stock_count`**  <span style="font-size: 12px; font-weight: 500;">optional</span>  
  The number of items in stock for the related unit.
- **`availability`**  <span style="font-size: 12px; font-weight: 500;">optional</span>  
  Availability status of the item. Can be "in_stock", "out_of_stock", or "discontinued".
- **`description`**  <span style="font-size: 12px; font-weight: 500;">optional</span>  
  Short description of the item.
- **`allergens`**  <span style="font-size: 12px; font-weight: 500;">optional</span>  
  List of allergens for the item, if any.
- **`allergens.type`**  <span style="font-size: 12px; font-weight: 500;">required when allergens is given</span>  
  Type of allergy. For example "contains peanuts" or "may contain peanuts".
- **`allergens.symptoms`**  <span style="font-size: 12px; font-weight: 500;">optional</span>  
  List of symptoms for the allergy.
- **`order_cutoff_times`**  <span style="font-size: 12px; font-weight: 500;">optional</span>  
  Cutt-off times are the minimum amount of time before delivery when the item can still be ordered.
- **`order_cutoff_times.{mon,tue,wed,thu,fri,sat,sun}`**  <span style="font-size: 12px; font-weight: 500;">optional</span>  
  Minimum amount of time, in hours, that an item needs to be ordered in advance of delivery for the given day.
- **`replacement_products`**   <span style="font-size: 12px; font-weight: 500;">optional</span>  
  List of product codes for alternative items when this item is not available.
- **`seasonality`**   <span style="font-size: 12px; font-weight: 500;">optional</span>  
  List of date ranges when the item is in-season. 
- **`seasonality.start_date`**  <span style="font-size: 12px; font-weight: 500;">required when seasonality is given</span>  
  The start date when the item is in season. In [ISO 8601][] calendar date format `YYYY-MM-DD`.
- **`seasonality.end_date`**  <span style="font-size: 12px; font-weight: 500;">required when seasonality is given</span>  
  The end date when the item is in season. In [ISO 8601][] calendar date format `YYYY-MM-DD`.

### Response

Status: `200 OK`

Body: JSON object of the updated item.

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
- **`availability`**  
  Availability status of the item. Defaults to in_stock.
- **`created_at`**  
  Datetime when the catalog item was created on REKKI.  
  In [ISO 8601][] UTC format `YYYY-MM-DDTHH:MM:SSZ`.
- **`updated_at`**  
  Datetime when the catalog item was last updated on REKKI.  
  In [ISO 8601][] UTC format `YYYY-MM-DDTHH:MM:SSZ`.

### Example Request

In this example, an existing item is marked as discontinued.

```bash
curl -X POST "https://backend.live.rekki.com/api/integration/v1/catalog/items/44724" \
     -H "Authorization: Bearer $API_TOKEN" \
     -H "X-REKKI-Authorization-Type: supplier_api_token" \
     -H "Content-Type: application/json" \
     -d '{"availability": "discontinued"}'
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
  "availability": "discontinued",
  "created_at": "2019-11-10T20:57:22Z",
  "updated_at": "2020-01-27T12:31:43Z"
}
```

</details>


[ISO 4217]: https://en.wikipedia.org/wiki/ISO_4217
[ISO 8601]: https://en.wikipedia.org/wiki/ISO_8601
[JSON Pointer]: https://tools.ietf.org/html/rfc6901#section-6