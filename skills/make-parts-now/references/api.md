# Make Parts Now REST API: the calls this skill uses

Base URL `https://makepartsnow.com/api/agent/v1`. Every call but `/capabilities` and `/accounts` sends `Authorization: Bearer $MPN_API_KEY`. The full reference is https://makepartsnow.com/api/agent/v1/docs (OpenAPI: `/api/agent/v1/openapi.json`); the plain-text guide is https://makepartsnow.com/llms.txt. The API changes additively within v1.

```bash
MPN=https://makepartsnow.com/api/agent/v1
AUTH="Authorization: Bearer $MPN_API_KEY"
```

## Account (once, with the user's consent)

```bash
curl -sS -X POST "$MPN/accounts" -H 'Content-Type: application/json' \
  -d '{"name": "<user name>", "email": "<user email>", "accept_terms": true}'
```

Returns `api_key` (shown once) and `webhook_secret`. `accept_terms: true` states that the user accepted https://makepartsnow.com/legal/terms; send it only after they did. Optional fields: `organization`, `ship_to` (US address: `{name, line1, line2?, city, state, postal_code, country: "US", phone?}`), `callback_url` (signed webhooks; see llms.txt).

## Catalog

```bash
curl -sS "$MPN/capabilities"
```

`processes[]` each have `id` (`sls`, `fdm`, `cnc`), `materials[]` (`id`, `colors[]`), `finishes[]` (`id`, `group`, `materials`), `services[]`, `speeds[]`, `build_volumes_mm` and `min_wall_mm`. `config_schema` describes the quote `config`; `limits` has the rate and size limits.

## Upload and analysis

```bash
curl -sS -X POST "$MPN/uploads" -H "$AUTH" -H "Idempotency-Key: <unique>" \
  -F "file=@path/to/part.step" -F "no_controlled_data=true"
```

Returns the part with `id` (`prt_...`) and `status`. Files up to 95 MB: STEP/STP, STL, 3MF, OBJ. For a public https link instead: `-H 'Content-Type: application/json' -d '{"url": "https://...", "no_controlled_data": true}'`.

```bash
curl -sS "$MPN/parts/<part_id>?wait=20" -H "$AUTH"
```

Waits up to `wait` seconds (max 30) for the analysis. Repeat until `status` is `ready`; `failed` comes with `error`. A ready part has `geometry` (`bbox` in the file's units, `volume`, `watertight`, `min_wall`), `suggested_units` (`mm` or `in`, a guess from the size), and `capable_processes`: each process id maps to `null` when it can make the part, or to the reason it can't (for example `"cnc": "CNC needs a STEP file"`).

## Quote

```bash
curl -sS -X POST "$MPN/quotes" -H "$AUTH" -H 'Content-Type: application/json' -H "Idempotency-Key: <unique>" \
  -d '{"items": [{"part_id": "<part_id>", "quantity": 10, "config": {"units": "mm", "process": "sls", "material": "sls_pa12"}}]}'
```

`config`: `units` (`mm` or `in`, the file's units), `process`, `material`, `color` (optional), `finishes` (ids, at most one per group), `infill` and `layer_height` (FDM), `services` (`[{id, count?}]`, for example heat-set inserts). Optional quote fields: `speed`, `ship_to`, `contact_email`, `po_number`.

The response (`QuoteView`):

- `id` (`qt_...`), `number`, `status`, `expires_at`, `share_url` (the quote's page for a person).
- `ready`: true when it can be checked out. If false, `blocking[]` names each problem with its `field` and `allowed` values.
- `items[]`: `id`, `part`, `config`, `quantity`, and `pricing` with `unit_price`, `line_total`, `warnings` (design notes, such as thin walls), `price_breaks` (`[{qty, unit_price}]` at 1, 2, 10, 50, 100 and 1000) and `needs_review` (a person prices it; the shop emails the quote).
- `tiers[]`, one per speed: `id` (`economy`, `standard`, `fast`, `faster`, `fastest`), `label`, `eligible` (and `reason` when not), `ship_date`, `arrive_date`, `total_before_tax`, `rush_fee`, `discount`, `shipping`.
- `totals`, `speed` (the selected one, `standard` by default), `next_steps[]` (plain-English instructions), `tax_note`.

Without `ship_to`, shipping in `tiers` is an estimate; `PATCH /quotes/<quote_id>` with `ship_to` makes it exact. The checkout page also asks the person for the address.

## Change the quote

```bash
curl -sS -X PATCH "$MPN/quotes/<quote_id>/items/<item_id>" -H "$AUTH" -H 'Content-Type: application/json' -d '{"quantity": 50}'
curl -sS -X PATCH "$MPN/quotes/<quote_id>/items/<item_id>" -H "$AUTH" -H 'Content-Type: application/json' -d '{"config": {"units": "mm", "process": "cnc", "material": "al6061"}}'
curl -sS -X PATCH "$MPN/quotes/<quote_id>" -H "$AUTH" -H 'Content-Type: application/json' -d '{"speed": "standard"}'
```

Each returns the repriced quote, with the same `id`. `POST /quotes/<quote_id>/items` adds another part (up to 25 per quote); `DELETE /quotes/<quote_id>/items/<item_id>` removes one.

## Checkout

```bash
curl -sS -X POST "$MPN/quotes/<quote_id>/checkout" -H "$AUTH" -H "Idempotency-Key: <unique>"
```

Returns `checkout_url` (Stripe Checkout on makepartsnow.com, for a person to pay by card), `amount_before_tax` and `quote_url`. Sales tax is added at checkout. The link is the only way this skill orders: the person pays on the website.

## Orders and housekeeping

```bash
curl -sS "$MPN/orders?quote_id=<quote_id>" -H "$AUTH"
curl -sS "$MPN/orders/<order_id>" -H "$AUTH"
curl -sS "$MPN/quotes?status=open" -H "$AUTH"
curl -sS -X POST "$MPN/quotes/<quote_id>/discard" -H "$AUTH"
```

## Errors and limits

Errors are JSON: `{"error": true, "status", "code", "message", "field"?, "allowed"?, "next_steps"?, "retry_after_s"?, "request_id"}`. Branch on `code`: `INVALID_CONFIG` (see `allowed`), `QUOTE_NOT_READY` (see `next_steps`), `AUTH_REQUIRED`, `RATE_LIMITED` and `UPLOAD_QUOTA` (wait `retry_after_s`), `OPEN_QUOTE_LIMIT` (discard old quotes), `FILE_TOO_LARGE`, `UNSUPPORTED_FILE_TYPE`. Limits: 60 requests a minute per key, 50 open quotes, 25 items per quote, files up to 95 MB.
