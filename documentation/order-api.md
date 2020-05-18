# REKKI - ORDER API

- [LIST ORDERS](#list-orders)
- [CONFIRM ORDER](#confirm-order)
- [GET ORDERS THAT ARE NOT MARKED AS INTEGRATED](#get-orders-that-are-not-marked-as-integrated)
- [MARK ORDERS AS SUCCESSFULLY INTEGRATED](#mark-orders-as-integrated)
- [MARK AN ORDER AS FAILED TO INTEGRATE](#mark-an-order-as-failed-to-integrate)

---

The base URL for all API endpoints is `https://api.rekki.com`

## LIST ORDERS
### `POST /api/catalog/integration/list_orders_by_supplier`

Lists all orders placed for the supplier that were placed through REKKI.

Orders are limited to max 30 days old (i.e. timestamp must be within 30 days).

We recommend polling for orders by setting the new request timestamp to the time of the last successful request.


### Request Headers

| Value | Description | Required? |
| --- | --- | --- |
| `Content-Type: application/json` | Indicates that the request body is of JSON media type | Yes|
| `Authorization: Bearer <api_token>` | Authentication. Contact integrations@rekki.com for a token | Yes |
| `X-REKKI-Authorization-Type: supplier_api_token` | Authentication Token type | Yes |

### Request Parameters

| Property | Type | Description | Required? |
| --- | --- | --- | --- |
| `since` | integer ([UNIX timestamp]) | Filters orders created at or after the given timestamp | Yes |

[UNIX timestamp]: https://www.epochconverter.com/

After you start using the API, you should request orders *since* last
received order's `inserted_at_ts`, since the API returns orders created **>=**
of the requested timestamp, you will **always** get at order from which
you took the timestamp in the response. This will be explained again
in the provided [example](#example-usage).

Keep in mind that since you can have more than one order per `since`,
you must not do `since: last_order.inserted_at_ts + 1`, but keep the
last order you received's `reference` and ignore the duplicate.

### Example Request

```bash
curl -X POST "https://api.rekki.com/api/catalog/integration/list_orders_by_supplier" \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer $API_TOKEN" \
     -H "X-REKKI-Authorization-Type: supplier_api_token" \
     -d "{\"since\":1565610869}"
```

### Example Response

The JSON response includes a list of orders that were created since the given timestamp.
Each order has metadata such as delivery date and order items.

```json
{
  "orders": [
    {
      "customer_account_no": "R8813",
      "confirmed_at": "2019-08-12T12:20:10.968294",
      "contact_info": "+447123456789",
      "contact_name": "John Doe",
      "location_name": "Coffee & Cake Cafe",
      "delivery_address": "123 Fake Street, Test",
      "delivery_on": "2019-08-29",
      "inserted_at_ts": 1565458065,
      "notes": "fresh tomato please",
      "reference": "A16915",
      "supplier_notes": "please use back entrance",
      "items": [
        {
          "id": "4fae0a5d3ee045922bae04eb2aee3d52",
          "name": "tomatoes",
          "price": "2.0",
          "product_code": "tm12",
          "quantity": 1,
          "units": "kg",
          "spec": "customer needs vines on tomatoes"
        }
      ]
    }
  ]
}
```

### Response Properties

* `customer_account_no` is the account number for customer within the supplier system, this can be setup in REKKI supplier app ( https://supplier.rekki.com ).
* `confirmed_at` the time at which the supplier confirmed the order (via email, or via the REKKI supplier app, or via the REKKI API)
* `contact_info` the phone number or email address for the person who placed the order
* `contact_name` the full name of the person who placed the order
* `location_name` the name of the location that placed the order, can be NULL
* `delivery_address` delivery address for this specific order (address, postcode)
* `delivery_on` expected delivery date (when users place orders they specify for which day it is supposed to be delivered)
* `inserted_at_ts` when was the order created by the customer (timestamp)
* `notes` defined by the user at the moment of making an order and usually refer to that specific order (e.g. "please send fresher tomatoes")
* `supplier_notes` notes define by the user for the supplier, usually being common across orders (e.g.: "please use the side entrance for delivery")
* `reference` REKKI's order reference

&nbsp;
* `item.product_code` product code that maps to the supplier catalog, suppliers can modify the product code for the future orders for this customer at https://supplier.rekki.com
* `item.id` REKKI's item id, for REKKI internal reference
* `item.price` the item price as set in REKKI
* `item.quantity` quantity
* `item.name` item name as defined on the customer product list
* `item.units` item unit as defined on the customer product list
* `item.spec` details/notes provided by the supplier for the item

### Example Usage

In this JavaScript example, all orders are retrieved. Then it keeps pulling for new orders since the last order, every hour.

```js
const fetch = require("node-fetch");

const sleep = function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
};

const fetch_orders = async function(token, since) {
  let r = await fetch(
    "https://api.rekki.com/api/catalog/integration/list_orders_by_supplier",
    {
      method: "POST",
      headers: {
        Authorization: "Bearer " + token,
        "X-REKKI-Authorization-Type": "supplier_api_token",
        "Content-Type": "application/json",
        Accept: "application/json"
      },
      body: JSON.stringify({ since })
    }
  );
  return await r.json();
};

const poll = async function(token, last_rekki_order_time) {
  let last_order_reference = undefined;

  while (true) {
    console.log("requesting orders since " + last_rekki_order_time);
    let response = await fetch_orders(token, last_rekki_order_time);

    for (let order of response.orders) {
      if (order.reference == last_order_reference) {
        // here is where we are ignoring the order we
        // took the inserted_at_ts from
        // but since we can have more orders in the same inserted_at_ts
        // you can't just do since: inserted_at_ts+1
        continue;
      }
      if (order.inserted_at_ts >= last_rekki_order_time) {
        last_rekki_order_time = order.inserted_at_ts;
        last_order_reference = order.reference;
      }

      // process(order)
      console.log(order);
    }
    await sleep(3600 * 1000); // wait 1 hour
  }
};

poll("XXXXXXX-XXXX-XXXX-XXXXX-XXXXXXXXXXXX", parseInt((+new Date() /1000) - 3600 * 24 * 30));
```

---

## CONFIRM ORDER
### `POST /api/integration/v1/orders/confirm`
<!-- <details><summary>Show details</summary> -->

Confirm a pending order by its reference code.

Notifies the buyer that the order has been acknowledged.

### Parameters

- **`orders`**  <span style="font-size: 12px; font-weight: 500;">required</span>
  Array of References of the orders to confirm. Order refs are discoverable when [listing orders](#list-orders).

### Response

Status: `200 OK`
Body: JSON object of the confirmed order

- **`success`**
  if the operation is successful

Status: `400 Conflict`
Body: `{"error":"Order already confirmed","order_id":...}`

Status: `400 Not Found`
Body: `{"error":"Order not found","order_id":...}`

in errors order_id denotes the order that failed to be confirmed

**the processing stops at first error**

### Example Request

```bash
curl -X POST -d '{"orders":["W2978351"]}' "https://api.rekki.com/api/integration/v1/orders/confirm" \
     -H "Authorization: Bearer $API_TOKEN" \
     -H "X-REKKI-Authorization-Type: supplier_api_token" \
     -H "Content-Type: application/json"
```

### Example Response

```json
{
  "reference": "W2978341",
  "notes": "Deliver to back door",
  "supplier_notes": null,
  "location_name": "Flour Power Pizza",
  "delivery_on": "2020-02-01",
  "delivery_address": "Borehamwood, WD6 1GP",
  "customer_account_no": "R8813",
  "created_at": "2020-01-31T18:08:05Z",
  "contact_name": "Jane Doe",
  "contact_info": "+447123456789",
  "confirmed_at": "2020-01-31T18:12:55Z",
  "items": [
    {
      "id": "b285af6ba51010750bba1e955996b0ad",
      "product_code": "APL31",
      "name": "Granny Smith Apples",
      "units": "each",
      "quantity": 50,
      "spec": "customer requested apples with leaf on stem",
      "price": null
    }
  ]
}
```

<sup><a href="#">Back to top ↰</a></sup>

---

## Get orders, that are not marked as integrated

### `POST /api/integration/v1/orders/list_not_integrated`
<!-- <details><summary>Show details</summary> -->

Lists all orders placed for the supplier that were placed through REKKI and not marked as integrated.

### Parameters

- **`since`**  <span style="font-size: 12px; font-weight: 500;">required</span>
  A unix timestamp (in seconds). Response will contrain only orders created after this time.

### Example Request

```bash
curl -X POST "https://api.rekki.com/api/integration/v1/orders/list_not_integrated" \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer $API_TOKEN" \
     -H "X-REKKI-Authorization-Type: supplier_api_token" \
     -d "{\"since\":1565610869}"
```

### Example Response

The JSON response includes a list of orders that were created after given timestamp and are not marked as integrated.
Each order has metadata such as delivery date and order items.

```json
{
  "orders": [
    {
      "customer_account_no": "R8813",
      "confirmed_at": "2019-08-12T12:20:10.968294",
      "contact_info": "+447123456789",
      "contact_name": "John Doe",
      "location_name": "Coffee & Cake Cafe",
      "delivery_address": "123 Fake Street, Test",
      "delivery_on": "2019-08-29",
      "inserted_at_ts": 1565458065,
      "notes": "fresh tomato please",
      "reference": "A16915",
      "supplier_notes": "please use back entrance",
      "items": [
        {
          "id": "4fae0a5d3ee045922bae04eb2aee3d52",
          "name": "tomatoes",
          "price": "2.0",
          "product_code": "tm12",
          "quantity": 1,
          "units": "kg",
          "spec": "customer needs vines on tomatoes"
        }
      ]
    }
  ]
}
```

### Response Properties

* `customer_account_no` is the account number for customer within the supplier system, this can be setup in REKKI supplier app ( https://supplier.rekki.com ).
* `confirmed_at` the time at which the supplier confirmed the order (via email, or via the REKKI supplier app, or via the REKKI API)
* `contact_info` the phone number or email address for the person who placed the order
* `contact_name` the full name of the person who placed the order
* `location_name` the name of the location that placed the order, can be NULL
* `delivery_address` delivery address for this specific order (address, postcode)
* `delivery_on` expected delivery date (when users place orders they specify for which day it is supposed to be delivered)
* `inserted_at_ts` when was the order created by the customer (timestamp)
* `notes` defined by the user at the moment of making an order and usually refer to that specific order (e.g. "please send fresher tomatoes")
* `supplier_notes` notes define by the user for the supplier, usually being common across orders (e.g.: "please use the side entrance for delivery")
* `reference` REKKI's order reference

&nbsp;
* `item.product_code` product code that maps to the supplier catalog, suppliers can modify the product code for the future orders for this customer at https://supplier.rekki.com
* `item.id` REKKI's item id, for REKKI internal reference
* `item.price` the item price as set in REKKI
* `item.quantity` quantity
* `item.name` item name as defined on the customer product list
* `item.units` item unit as defined on the customer product list
* `item.spec` details/notes provided by the supplier for the item

<sup><a href="#">Back to top ↰</a></sup>

---

## Mark orders as integrated
### `POST /api/integration/v1/orders/set_integrated`
<!-- <details><summary>Show details</summary> -->

Mark orders as integrated.

### Parameters

- **`orders`**  <span style="font-size: 12px; font-weight: 500;">required</span>
  Array of References of the orders to confirm. Order refs are discoverable when getting not integrated orders

### Response

Status: `200 OK`
Body: JSON object of the confirmed order

Body: JSON object that identifies the result of operation

- **`affected`**
  Number of items actually changed in the catalog
- **`success`**
  Result of update, `true` if successful

in errors order_id denotes the order that failed to be confirmed

**the processing stops at first error**

### Example Request

```bash
curl -X POST -d '{"orders":["W2978351"]}' "https://api.rekki.com/api/integration/v1/orders/set_integrated" \
     -H "Authorization: Bearer $API_TOKEN" \
     -H "X-REKKI-Authorization-Type: supplier_api_token" \
     -H "Content-Type: application/json"
```

### Example Response

```json
{
  "affected": 1,
  "success": true
}
```

<sup><a href="#">Back to top ↰</a></sup>

---

## Mark an order as failed to integrate
### `POST /api/integration/v1/orders/set_error`
<!-- <details><summary>Show details</summary> -->

Notify that an order has failed to integrate. This action is required, since such order will be handled differently
in this case to make sure that the supplier gets the information

### Parameters

- **`order`**  <span style="font-size: 12px; font-weight: 500;">required</span>
  Object that identifies an order to be marked as failed. The only field that needs to be set in the object is `reference`
  which is a reference of the order in rekki.

- **`error`**  <span style="font-size: 12px; font-weight: 500;">required</span>
  A string containg the reason of failure

- **`attempts`**  <span style="font-size: 12px; font-weight: 500;">required</span>
  Number of attempts made to push the order into an external system

### Response

Status: `200 OK`
Body: JSON object of the confirmed order

- **`success`**
  if the operation is successful

### Example Request

```bash
curl -X POST -d '{"orders":["W2978351"]}' "https://api.rekki.com/api/integration/v1/orders/set_error" \
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


[ISO 4217]: https://en.wikipedia.org/wiki/ISO_4217
[ISO 8601]: https://en.wikipedia.org/wiki/ISO_8601
