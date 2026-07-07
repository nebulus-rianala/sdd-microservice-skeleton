# Input Validation

## Rule: every external input goes through a strict server-side validator

No handler may read raw request input directly and act on it. All input — path params,
query params, body, headers used for anything beyond Kong trust — is validated by a
schema-driven validator before it reaches business logic.

- Laravel: a `FormRequest` class per endpoint. No controller may call
  `$request->input()/query()/all()` unvalidated.
- FastAPI: a Pydantic model per endpoint (request schema), `additionalProperties`-strict
  behavior (Pydantic's default of rejecting unknown fields unless explicitly configured
  otherwise).

## Required rules for all validators

- Every field has an explicit type and, where applicable, a bounded length/range/enum.
- Unknown/unexpected fields are rejected, not silently dropped or accepted
  (anti mass-assignment / anti over-posting).
- No `$guarded = []` (Laravel) or a permissive `model_config = {"extra": "allow"}`
  (Pydantic) on any inbound schema.

## SQL injection prevention

Access the database via the ORM's query builder or parameterized statements — never
string concatenation to build a query. Raw SQL is permitted only for aggregate/statistics
queries where the ORM's expressiveness is insufficient, and only ever with bound
parameters, never interpolated values.

## Mass assignment protection

Output DTOs/resources are just as strict as input: expose only the fields the endpoint's
contract (OpenAPI spec) declares. Never return an ORM model directly; map it through a
resource/schema class so adding a column to the database doesn't silently expose it.

## Query parameter filtering (index/list endpoints)

Every filterable/sortable query parameter is validated against an explicit allowlist of
column names — never pass a raw query parameter as a column name or raw `ORDER BY`
fragment.

## File uploads

- Rename the file server-side (e.g. a generated ID); never reuse the client-supplied name.
- Validate file type by magic bytes, not just the extension or `Content-Type` header.
- Store outside the webroot; forbid execution of uploaded files.
- Enforce a maximum file size and, where the threat model requires it, an async
  antivirus scan before the file is usable.

## JSON columns

Structured data uses `JSON`/`JSONB` columns with an explicit expected shape documented in
the technical spec's data-model section — validate the shape on write, don't store
arbitrary caller-supplied JSON unchecked.

## Max request size

Enforce a maximum request body size at the application layer in addition to whatever
limit Kong applies upstream — do not rely solely on the gateway.

## Server-side request forgery (SSRF) (OWASP A10, API7:2023)

Any code path that makes an outbound HTTP call to a caller-influenced or caller-supplied
URL (webhook target registration, "fetch this resource" style features, URL preview) must:

- restrict destinations to an explicit allowlist rather than accepting any URL;
- block requests to private IP ranges (RFC 1918), loopback, and cloud metadata endpoints
  (`169.254.169.254`) — a caller must never be able to make this service call its own
  cloud metadata service or an internal-only host;
- not follow redirects to an unvalidated destination automatically;
- enforce a connect/read timeout and a maximum response size.

## Unsafe deserialization and parsing (OWASP A08/A05)

Never deserialize untrusted data with a format that can execute code as a side effect of
parsing (PHP `unserialize()` on caller input, Python `pickle.loads()`, unsafe YAML
loaders). Use the safe/strict variant (`json_decode`, `yaml.safe_load`, Pydantic/DTO
parsing) for anything originating outside the service.

XML parsers must have external entity resolution disabled (anti-XXE) — this is not
optional even for internal/trusted-looking XML sources, since the source's trust level
can change without this service being told. Enforce depth and size limits on any
recursive parser (JSON, XML, YAML) to prevent a small payload from causing
disproportionate CPU/memory use.
