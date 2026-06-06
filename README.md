# Vision Moderation as a Service — API Spec

OpenAPI 3.1 specification for the Vision Moderation SaaS API.

## Validating the spec

```bash
npx @redocly/cli lint openapi.yaml
```

Or install globally first:

```bash
npm install -g @redocly/cli
redocly lint openapi.yaml
```

Expected output: **"Your API description is valid."** (two informational warnings about
the `example.com` hostname and missing `license` field are expected in a lab context).

## Example payloads

Files in `examples/` are valid instances of their corresponding schemas.

| File | Schema |
|------|--------|
| `predict-request.json` | `PredictRequest` |
| `predict-response.json` | `PredictResponse` |
| `predict-error-413.json` | `Error` |
| `batch-request.json` | `BatchPredictRequest` |
| `batch-response.json` | `BatchPredictResponse` |

> **Note on `image_b64` placeholders:** The base64 strings in `predict-request.json`
> and `batch-request.json` are real, valid base64-encoded 1×1 pixel images (JPEG, PNG,
> WebP respectively). They are far smaller than 5 MB. In production, this field would
> contain a full image encoded as a base64 string; a 5 MB JPEG becomes approximately
> 6.67 MB of base64 text.

## Makefile

```makefile
lint:
	redocly lint openapi.yaml
```

Run with `make lint`.
