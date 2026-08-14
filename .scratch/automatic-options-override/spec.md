# Per-Route Automatic OPTIONS Override

Status: ready-for-agent

## Problem Statement

Flask applications can globally disable Automatic OPTIONS. When a route's view
explicitly enables `provide_automatic_options`, that route currently still
rejects an OPTIONS request before Flask can produce its generated response.
This contradicts the documented per-route override and prevents applications
from opting in selectively.

## Solution

Make an explicit per-route enablement of Automatic OPTIONS take precedence over
the global default. The route must accept OPTIONS and Flask must return its
generated OPTIONS response. Existing opt-out, explicitly declared OPTIONS, and
default configuration behavior remain unchanged.

## User Stories

1. As a Flask application author, I want to disable Automatic OPTIONS globally, so that routes do not opt in by default.
2. As a Flask application author, I want a function view to explicitly enable Automatic OPTIONS, so that one route can opt in despite the global default.
3. As a Flask application author, I want a class-based view to explicitly enable Automatic OPTIONS, so that the override works consistently across supported view styles.
4. As a Flask application author, I want an opted-in route to accept an OPTIONS request, so that clients can discover the methods the route allows.
5. As a Flask application author, I want Flask to generate the OPTIONS response for an opted-in route, so that I do not need to write a duplicate handler.
6. As a Flask application author, I want explicit per-route disablement to remain effective, so that I can continue to provide a custom OPTIONS response when needed.
7. As a Flask application author, I want routes that explicitly declare OPTIONS to retain their existing behavior, so that this fix does not change established method declarations.
8. As a Flask application author, I want the global default to remain unchanged for views without an explicit setting, so that existing application-wide configuration remains reliable.
9. As a Flask maintainer, I want all route-registration entry points to resolve the override consistently, so that identical public settings do not behave differently by registration style.

## Implementation Decisions

- Route registration remains the single place that resolves Automatic OPTIONS behavior and constructs the route's accepted method set.
- An explicit `provide_automatic_options=True`, whether supplied by a view or the route-registration argument, is an override of the global `PROVIDE_AUTOMATIC_OPTIONS` default.
- When that resolved value is true, OPTIONS is included in the route's required methods so that request matching reaches Flask's generated OPTIONS response.
- Explicit `False` continues to suppress generated OPTIONS responses and does not add OPTIONS to a route.
- Views without an explicit setting continue to derive their behavior from the global configuration.
- No new configuration keys, public APIs, or documentation changes are required because the documented override already promises this behavior.

## Testing Decisions

- Test externally observable behavior through the existing application route-registration and test-client seam; assertions should cover response status and the `Allow` header rather than internal route attributes.
- Add coverage for a function view that explicitly enables Automatic OPTIONS while the global default is disabled.
- Add coverage for a class-based view under the same configuration, using its existing view-construction path.
- Verify that the generated OPTIONS response advertises OPTIONS together with the route's normal methods.
- Preserve the existing regression coverage for explicit opt-out, explicitly declared OPTIONS methods, and the normal global default.
- Follow the existing Automatic OPTIONS tests as prior art and keep the regression focused on public HTTP behavior.

## Out of Scope

- Changing Flask's default value for `PROVIDE_AUTOMATIC_OPTIONS`.
- Introducing a new custom OPTIONS-response API.
- Altering behavior for routes that explicitly implement their own OPTIONS handling.
- Refactoring unrelated routing or view-registration code.

## Further Notes

The defect is limited to the combination of a disabled global default and an
explicit per-route enablement. The intended behavior is already represented in
Flask's public configuration and view documentation.
