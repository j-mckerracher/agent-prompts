# Code Standards

> Note: This is an in-repo copy of the team’s standards (originally maintained in an external vault).

## General
- Follow existing patterns in the codebase.
- Prefer small, focused diffs and avoid unrelated refactors.
- Keep public APIs stable unless the task requires change.
- Use clear naming and keep code self-documenting.

## Go (backend)
- Keep packages small and cohesive.
- Use context-aware functions where appropriate.
- Return typed errors for domain conditions and map them at the API boundary.
- Validate input at the edge (handlers) and enforce invariants in the domain/service layer.
- Prefer table-driven tests.

## Dart/Flutter (mobile)
- Follow existing project architecture (BLoC/repository patterns used in the app).
- Keep widgets small; extract reusable pieces.
- Keep domain entities immutable where possible.
- Map network errors to failure types; avoid throwing raw exceptions across layers.
- Add unit/widget tests for core behavior.

## Testing
- Tests should be deterministic and not rely on network.
- Cover happy path + at least 1–2 edge cases.
- Name tests clearly: what is being tested and expected outcome.
