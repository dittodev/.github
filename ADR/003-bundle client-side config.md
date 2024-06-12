# Bundle client-side configuration with the SPA bundle

## Status

proposed

## Context

Client side code needs runtime configuration - things like what API endpoints to call, which cognito pool, and so forth. Typically these start out as a few line of configuration per stage. While initially this config trivial, and easy to test, this approach invites accidental complexity in the long run: configuration may be abused for a quick feature toggle, increases in complexity, introducing runtime risks that require additional testing and monitoring.

We identify the following options:

1. Bundle all the runtime config - We bundle all runtime config, adding compile time checks for consistency and schema validation. On the client, we implement simple heuristics to select the right configuration (for example the domain).
2. Bundle runtime config per stage - Bundle runtime config per stage, compiling different bundles per stage.
3. Runtime config injection - Common in the industry this is a two-stage approach: we ship config and bundle separately and inject configuration at "bootstrap" time. Often this happens in the shape of an index.html generated server-side, or a dedicated service like AWS AppConfig, or Optimizely.

## Decision

We go with option (1): Bundle all runtime config, add compile time checks, implement simple client-side heuristics.

## Consequences

We are aware of the limitations of this approach, and will revisit this decision later, as we identify requirements or scaling concerns.
