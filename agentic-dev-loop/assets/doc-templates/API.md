# API Reference

## `<METHOD> /path`
**Auth:** none | bearer token | session | api key
**Description:** one line.

**Request**
```json
{ "field": "type — description" }
```

**Response `200`**
```json
{ "field": "type — description" }
```

**Errors**
| Status | Condition |
|---|---|
| 400 | invalid input, missing field X |
| 401 | not authenticated |
| 403 | authenticated but not authorized for this resource |
| 404 | resource not found |
| 429 | rate limit exceeded |

---

(repeat per endpoint)
