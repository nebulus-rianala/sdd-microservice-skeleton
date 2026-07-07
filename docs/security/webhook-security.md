# Webhook and Third-Party Integration Security

Ref: OWASP API10:2023.

## Outbound webhooks (this service notifying a subscriber)

Sign every outbound webhook payload with HMAC and include a timestamp the receiver can
use for anti-replay validation. Treat webhook delivery as at-least-once — the receiver's
processing (and this service's own retry logic) must be idempotent.

## Inbound webhooks (a third party calling this service)

Verify the sender's signature and timestamp before processing — an unsigned or
stale-timestamped webhook is rejected, not processed "just in case it's legitimate."
Validate the payload against a strict schema exactly as any other external input (see
`input-validation.md`); a webhook body is not a trusted source merely because it arrived
on a dedicated endpoint.

## Third-party API responses

Treat every response from a third-party API as untrusted input: validate its shape,
apply a timeout, and bound retries (with backoff) rather than retrying indefinitely on a
failing third party.
