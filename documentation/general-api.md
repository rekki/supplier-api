# REKKI - ORDER API

The base URL for all API endpoints is `https://api.rekki.com`
This page is devoted to api endpoints that do not belong to a specific domain

## Send logging information
### `POST /api/integration/v1/log`
<!-- <details><summary>Show details</summary> -->

Post a log message for the supplier for internal debugging. There is no need to handle response from this endpoint.
Please chat with integrations@rekki.com before using it.

### Parameters

	Level   string `json:"level"`
	Message string `json:"message"`

- **`order`**  <span style="font-size: 12px; font-weight: 500;">required</span>
  Log level. possible values are: "panic", "error", "fatal", "warn", "warning", "info", "debug", "trace"

- **`message`**  <span style="font-size: 12px; font-weight: 500;">required</span>
  Log message

### Response

Status: `200 OK`
Body: empty

### Example Request

```bash
curl -X POST -d '{"level":"info", "message": "[WD10034] integration took 120 seconds"}' "https://api.rekki.com/api/integration/v1/log" \
     -H "Authorization: Bearer $API_TOKEN" \
     -H "X-REKKI-Authorization-Type: supplier_api_token" \
     -H "Content-Type: application/json"
```

<sup><a href="#">Back to top ↰</a></sup>
