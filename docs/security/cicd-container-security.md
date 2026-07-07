# CI/CD and Container Security

Ref: OWASP Top 10 CI/CD, OWASP Docker/Kubernetes Top 10.

## Containers

- Dockerfiles run as a non-root user (`USER` with UID ≥ 1000) — never run the application
  process as root inside the container.
- Set a restrictive `securityContext` in the deployment manifest:
  `allowPrivilegeEscalation: false`, all Linux capabilities dropped,
  `readOnlyRootFilesystem: true` where the application doesn't need to write to its own
  filesystem, and explicit CPU/memory resource limits.
- Never copy a secret into the image layer (build args, `COPY .env`, etc.) — see
  `secret-management.md`.
- Never pin to an unpinned `latest` tag for a base or dependency image; pin a specific,
  reproducible digest or version.
- Prefer a distroless or minimal base image, and pin it by immutable digest rather than
  by tag alone.

## CI pipeline gates

- Every pipeline includes, at minimum: a secrets scanner, SAST, dependency/SCA scanning
  (see `dependency-policy.md`'s audit gate), and a container-image scan — each as a
  blocking gate, not an informational-only report.
- Jobs run with the minimum permissions they need; secrets are never exposed to pipelines
  triggered from a fork/external merge request.
