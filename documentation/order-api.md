# REKKI

## List orders

To retrieve a list of orders from a supplier, please use the following endpoint:

`POST https://backend.live.rekki.com/api/catalog/integration/list_orders_by_supplier`

There are three mandatory parameters for the call: 

* `XXXXXXX-XXXX-XXXX-XXXXX-XXXXXXXXXXXX` is the token for API authentication that is attributed by REKKI to each supplier, please contact integrations@rekki.com to request one. Include it in the request header `authorization: Bearer <TOKEN>`.

* `{"since":1565610869}` send as POST body, **since** is the epoch timestamp (seconds since 01/01/1970 00:00) and will filter the orders **after or equal** to the specified timestamp. A very common use case is to `poll` for orders since the last successful request to avoid losing any information. The requested timestamp **must** be within the last 30 days and it **must** be an integer not a string.

* `Content-Type: application/json` header is mandatory as well

### Example request

```bash
curl -X POST "https://backend.live.rekki.com/api/catalog/integration/list_orders_by_supplier" \
     -H "Content-Type: application/json" \
     -H "authorization: Bearer XXXXXXX-XXXX-XXXX-XXXXX-XXXXXXXXXXXX" \
     -d "{\"since\":1565610869}"
```

### Example response

The JSON response includes a list of orders that were created since the given timestamp.  
Each order has metadata such as delivery date and order items.

```json
{
  "orders": [
    {
      "customer_account_no": "R8813", 
      "confirmed_at": "2019-08-12T12:20:10.968294",
      "delivery_address": "Test, Test",
      "delivery_on": "2019-08-29",
      "inserted_at_ts": 1565458065,
      "notes": "please pick yellow peppers",
      "reference": "A16915",
      "supplier_notes": "please use back entrance",
      "items": [
        {
          "id": "4fae0a5d3ee045922bae04eb2aee3d52",
          "name": "tomatoes",
          "price": "2.0",
          "product_code": "tm12",
          "quantity": 1,
          "units": "kg"
        }
      ]
    }
  ]
}
```

### Response properties

* `customer_account_no` is the account number for customer within the supplier system, this can be setup in REKKI supplier app ( https://tulip.rekki.com ). 
* `confirmed_at` the time at which the supplier confirmed the order (via email or from the REKKI supplier app)
* `delivery_address` delivery address for this specific order (address, postcode)
* `delivery_on` expected delivery date (when users place orders they specify for which day it is supposed to be delivered)
* `inserted_at_ts` when was the order created by the customer (timestamp)
* `notes` defined by the user at the moment of making an order and usually refer to that specific order (e.g. "please send fresher tomatoes")
* `supplier_notes` , notes define by the user for the supplier, usually being common across orders (e.g.: "please use the side entrance for delivery")
* `reference` REKKI's order reference

* `item.product_code` product code that maps to the supplier catalog, suppliers can modify the product code for the future orders for this customer at https://tulip.rekki.com
* `item.id` REKKI's item id, for REKKI internal reference 
* `item.price` the item price as set in REKKI 
* `item.quantity` quantity
* `item.name` item name as defined on the customer product list 
* `item.units` item unit as defined on the customer product list 

### Example usage

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
