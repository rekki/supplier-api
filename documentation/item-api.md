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

