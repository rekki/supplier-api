# REKKI - CATALOGUE API

- [LIST ITEMS](#list-items)
- [RETRIEVE ITEM](#retrieve-item)
- [DELETE ITEM](#delete-item)
- [ADD ITEM](#add-item)

---

Suppliers can use the Item API to manage their item catalog.

The base URL for all API endpoints is **`https://api.rekki.com`**.

### Authentication
<!-- <details><summary>Show details</summary> -->

Requests _must_ be authenticated via an authorization header <strong><code>Authorization: Bearer <em>TOKEN</em></code></strong>.  
Unauthenticated requests will receive a `401 Unauthorized` response.

Requests _must_ also specify the token type via the header <strong><code>X-REKKI-Authorization-Type: supplier_api_token</code></strong>.

Contact integrations@rekki.com for an API token.
</details>

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
  Suppliers can modify the product code for future orders at https://supplier.rekki.com
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

### Example Request

```bash
curl -X GET "https://api.rekki.com/api/integration/v1/catalog/items" \
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

Body: JSON object of the item that was retrieved.

- **`id`**  
  REKKI's ID to uniquely identify the catalog item (for REKKI internal reference).
- **`product_code`**  
  Product code for the item that maps to the supplier's catalog.  
  Suppliers can modify the product code for future orders at https://supplier.rekki.com
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

### Example Request

```bash
curl -X GET "https://api.rekki.com/api/integration/v1/catalog/items/17238" \
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
}
```

<sup><a href="#">Back to top ↰</a></sup>

## DELETE ITEM
### `DELETE /api/integration/v1/catalog/items/:id`
<!-- <details><summary>Show details</summary> -->

Delete an item from your catalog by its unique ID.

### Parameters

- **`id`**  <span style="font-size: 12px; font-weight: 500;">required (query parameter)</span>  
  ID of the item to retrieve. Item IDs are discoverable when [listing items](#list-items).

### Response

Body: JSON object of the item that was retrieved.

- **`success`**  
  true if delete was successfull

### Example Request

```bash
curl -X DELETE "https://api.rekki.com/api/integration/v1/catalog/items/17238" \
     -H "Authorization: Bearer $API_TOKEN" \
     -H "X-REKKI-Authorization-Type: supplier_api_token" \
     -H "Content-Type: application/json"
```

### Example Response

```json
{
  "success": true
}
```

<sup><a href="#">Back to top ↰</a></sup>


---

## ADD/UPDATE ITEM
### `POST /api/integration/v1/catalog/items`
<!-- <details><summary>Show details</summary> -->

Creates or Updates an item on your catalog. If item with this
`product_code` already exists, you can update this item.


### Parameters

- **`id`**  
  REKKI's ID to uniquely identify the catalog item (for REKKI internal reference).
  If `id` is specified the item will be update, if not it will attempt to insert it.
- **`product_code`**  <span style="font-size: 12px; font-weight: 500;">required</span>  
  Product code for the item that maps to the supplier's catalog.  
  Suppliers can modify the product code for future orders at https://supplier.rekki.com
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

Body: JSON object of the item that was added to the catalog of the authenticated supplier.

- **`id`**  
  REKKI's ID to uniquely identify the catalog item (for REKKI internal reference).
- **`product_code`**  
  Product code for the item that maps to the supplier's catalog.  
  Suppliers can modify the product code for future orders at https://supplier.rekki.com
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

### Example Request

```bash
curl -X POST "https://api.rekki.com/api/integration/v1/catalog/items" \
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



[ISO 4217]: https://en.wikipedia.org/wiki/ISO_4217
[ISO 8601]: https://en.wikipedia.org/wiki/ISO_8601
[JSON Pointer]: https://tools.ietf.org/html/rfc6901#section-6
