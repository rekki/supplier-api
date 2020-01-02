# REKKI - ORDER API

The base URL for all API endpoints is `https://backend.live.rekki.com`

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
you must not do `since: last_order.inserted_at_ts + 1`.

### Example Request

```bash
curl -X POST "https://backend.live.rekki.com/api/catalog/integration/list_orders_by_supplier" \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer $API_TOKEN" \
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

* `customer_account_no` is the account number for customer within the supplier system, this can be setup in REKKI supplier app ( https://tulip.rekki.com ). 
* `confirmed_at` the time at which the supplier confirmed the order (via email or from the REKKI supplier app)
* `location_name` the name of the location that placed the order, can be NULL
* `delivery_address` delivery address for this specific order (address, postcode)
* `delivery_on` expected delivery date (when users place orders they specify for which day it is supposed to be delivered)
* `inserted_at_ts` when was the order created by the customer (timestamp)
* `notes` defined by the user at the moment of making an order and usually refer to that specific order (e.g. "please send fresher tomatoes")
* `supplier_notes` notes define by the user for the supplier, usually being common across orders (e.g.: "please use the side entrance for delivery")
* `reference` REKKI's order reference
  
&nbsp;
* `item.product_code` product code that maps to the supplier catalog, suppliers can modify the product code for the future orders for this customer at https://tulip.rekki.com
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
    "https://backend.live.rekki.com/api/catalog/integration/list_orders_by_supplier",
    {
      method: "POST",
      headers: {
        Authorization: "Bearer " + token,
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
