# Task Tree

- `Inventory login state machine, renderer and OAuth adapters`()
- `Extract OpenAI Login spec and implementation topics`()
- `Document dependencies, interactions, rendering and cancellation`()
- `Retarget composition and direct consumers`()
- `Run component and downstream JVM regression suites`()
- `Record results and obtain user acceptance`()

# Details

## Authorization and scope

- The user authorized the next batch along the confirmed app-component design.
- Select the independently owned OpenAI Login popup as one bounded batch.
  Do not migrate all Settings, History or Agent interaction modules together.
- Preserve the previous uncommitted Path Picker and Session Catalog batch.
  Do not commit, switch branches, or touch unrelated user files.
- Use [Frontend Application Boundary](../../checklist/frontend-application-boundary.md)
  and [Spec/Impl Module Boundaries](../../checklist/spec-impl-module-boundaries.md).

## Component mapping

| Existing source | Target | Responsibility |
| --- | --- | --- |
| Login contract within `app/contract/settings` | `app/component/openai-login/spec` | Dependency port, state/effects, interactions, factory, lifecycle and renderer semantics. |
| Login state machine/tests within `app/viewmodel/settings` | `app/component/openai-login/impl/viewmodel` | Component-owned attempt sequencing and state/effect publication. |
| Login popup/tests within `app/view/settings` | `app/component/openai-login/impl/view` | Mosaic rendering, opening an external URL and reporting its outcome. |

## Dependency and ownership boundary

- The spec declares the login preparation capability and consumes the existing
  `KodexAuthLoginAttempt`; do not duplicate OAuth callback/token protocols.
- `app-shared-auth-contract` is a legacy shared module containing both the
  attempt interface and other auth declarations, including an in-memory
  implementation. Record this staged dependency rather than claiming a fully
  spec-only repository graph.
- The ViewModel must not depend on Settings, RPC, auth-store implementations
  or browser launching. The renderer must not depend on Settings View or the
  concrete ViewModel; consume only the component spec and shared TUI/URL capabilities.
- Keep Kotlin package/public symbol identities and the existing zero-argument
  bound factory. Add typed dependency injection to the ViewModel creation
  seam; the bound factory captures dependencies and the owner lifecycle.
- Preserve credential-source binding at child creation. Browser retry reuses
  the exact pending attempt and never restarts OAuth.
- Browser success means only launch success; completion means the dependency
  finished authorization/commit. URLs live in one-shot effects, never state,
  logs or persisted snapshots.
- Leave RPC preparation, listener/timeout/cancellation, token union results,
  credential persistence and MCP OAuth unchanged.
- Use the same TUI theme roles and button interaction styles directly; do not
  duplicate Settings or introduce a Settings View dependency just for styling.

## Contract and validation gates

- Describe every state variant's rendering and permitted actions in spec KDoc.
- Document start/retry/cancel, stale host callbacks, effect consumption,
  completion/failure, close, required thread confinement and `@throws`.
- Test dependency-only execution, duplicate start suppression, exact-attempt
  browser retry, stale callbacks/effects, cancellation/restart, failure/retry,
  late preparation cleanup and disposal. Render all state variants without
  starting a real browser or contacting a provider.
- Run login spec/impl JVM compilation/tests plus Settings, RPC, Application
  ViewModel/View regressions and direct downstream compilation.
- Check unchanged public package identities, no unexpected serialization or
  OAuth behavior changes, spec/view dependency directions, and inner/outer
  `git diff --check`.
- Record Native/JS/CLI validation as outstanding when not run.

## Checkpoint

- All three component projects are implemented on `refactor/spec`; public
  package identities and the bound factory are preserved.
- Composition explicitly supplies `OpenAiLoginDependencies`. The ViewModel
  receives no Settings/RPC/store implementation, and the renderer uses the
  same TUI theme roles without importing Settings styles.
- 20 component JVM tests passed (2 spec, 9 ViewModel, 9 View), including all
  state branches, launcher outcomes/stale-effect filtering and explicit
  removal from composition. The initial renderer disposal assertion assumed
  test-harness shutdown timing; it was corrected to test actual composition
  removal, without changing production disposal behavior.
- Direct downstream JVM compilation passed through Settings, RPC,
  Application ViewModel and Application/Settings Views.
- 212 downstream JVM tests passed: Settings ViewModel (5), Settings View (23),
  RPC ViewModel (59), Application ViewModel (24), Application View (101).
  All 232 scoped tests passed without failures or skips.
- Public package/symbol identities, source relocation, dependency direction
  and inner/outer `diff --check` were verified. Login no longer has source
  declarations in the three broad Settings projects; there are no duplicate
  or placeholder projects.
- Native/JS/CLI validation was not run. The mixed legacy auth-contract
  dependency remains recorded above; this is not global spec closure.
- The user accepted continuation and subsequently authorized submission after
  reviewing the session-action family. Inner commit `fc9f1936` contains the
  accepted component batches. No pushes or branch switches were performed.
