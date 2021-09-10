# REKKI - ORDER API

---

Order API can be used to fetch the information about orders

The base URL for all API endpoints is **`https://api.rekki.com`**.

### Authentication
<!-- <details><summary>Show details</summary> -->

Requests _must_ be authenticated via an authorization header <strong><code>Authorization: Bearer <em>TOKEN</em></code></strong>.
Unauthenticated requests will receive a `401 Unauthorized` response.

Requests _must_ also specify the token type via the header <strong><code>X-REKKI-Authorization-Type: supplier_api_token</code></strong>.

Contact integrations@rekki.com for an API token.
</details>

---

### Endpoints

Please consult out [swagger page](https://api.rekki.com/swagger/index.html) for the current api information
