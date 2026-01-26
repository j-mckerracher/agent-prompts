# Common Pitfalls to Avoid

> Note: This is an in-repo copy of team guidance (originally maintained in an external vault).

- Don’t change files outside the assignment scope.
- Don’t introduce new dependencies without strong reason.
- Don’t hardcode secrets; use placeholders like `{{API_KEY}}`.
- Don’t swallow errors; return useful context and map to user-safe messages.
- Don’t break API contracts (request/response shapes) without updating all consumers.
- Don’t write flaky tests (timing-dependent, random ordering, reliance on clock/network).
- Don’t leak PII in logs; keep logging minimal and sanitized.
