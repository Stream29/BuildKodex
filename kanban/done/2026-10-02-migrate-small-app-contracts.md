# Task Tree

- `Inventory the two pure app contract projects`()
- `Make path-picker and session-catalog vertical app components`()
- `Retarget all Gradle consumers`()
- `Run contract tests and downstream JVM compilation`()
- `Update active planning records`()
- `Accept the component spec and implementation`()

# Details

## Authorization and scope

- This is the next low-risk batch after the completed `agent-session`
  migration.
- Treat `path-picker` and `session-catalog` as the first vertical app
  components rather than as isolated data-contract projects.
- Give each component an `app/component/<name>/{spec,impl/<topic>}` tree.
  The spec owns the ViewModel dependency ports, interaction protocol,
  state/effect semantics, and renderer contract. Implementations may remain
  hosted by existing broad modules temporarily, but that hosting debt must be
  explicit.
- Do not move unrelated application composition, CLI, settings, migration, or
  shared filesystem projects in this task.
- Preserve existing Kotlin package identities, serialization, and observable
  behavior. The component factory API is intentionally expanded to typed
  dependency injection as described below.

## Target mapping

| Current project | Target project | Responsibility |
| --- | --- | --- |
| `app/contract/path-picker` + existing picker View/ViewModel | `app/component/path-picker/spec` plus `app/component/path-picker/impl/{viewmodel,view}` | Directory picker dependencies, navigation/confirmation interactions, state/effects, and rendering semantics; filesystem and Mosaic code stay in impl topics. |
| `app/contract/session-catalog` + existing RPC/UI hosts | `app/component/session-catalog/spec` plus `impl/viewmodel` | Catalog backend dependency port, archive/fork/delete interactions, snapshot state, row semantics, and renderer contract. RPC adapts the port; the UI remains hosted by application View pending extraction. |

The spec is the component's fact source. Its KDoc must say which dependency
operations are required, which interactions are serialized or no-ops, which
state/effect transitions are observable, and how a renderer maps every state
variant to UI. It must not import Mosaic or concrete filesystem types.

## Dependency and validation gates

- Retarget every Gradle consumer to the component spec or implementation topic
  that it actually consumes (`:app-component-path-picker-*` and
  `:app-component-session-catalog-*`).
- Keep the existing conventional plugins and target sets.
- Check all public KDoc and serialization annotations after the move.
- Run component spec/impl JVM compilation and tests.
- Compile direct downstream consumers, including app contract/application,
  settings, ViewModel/session/application, View/application, RPC contract,
  and integration consumers where applicable.
- Run `git diff --check` in both repositories and scan active documents for
  stale paths or old Gradle coordinates.
- Record Native/JS/CLI validation separately if not run.

## Stop conditions

- Stop if a consumer requires a concrete implementation dependency that is not
  represented by a spec port, or if the move changes a serialized field,
  public Kotlin name, or ViewModel behavior.
- Do not create placeholder implementation projects solely for directory
  symmetry; staged hosts must be recorded as debt.

## Execution checkpoint

- `path-picker` is now a vertical component with
  `app/component/path-picker/spec` and `impl/{viewmodel,view}` topics.
- `session-catalog` is now a component spec under
  `app/component/session-catalog/spec` with a framework-free state machine
  under `impl/viewmodel`. RPC now adapts the declared dependency port, and
  Application supplies the fork/delete ownership operations. Its renderer
  remains embedded in `app/view/application`; extracting that UI is a
  follow-up, not a completed dedicated View topic.
- Component specs retain Kotlin packages, public names, and serialization
  declarations while adding dependency, interaction, state, and renderer KDoc.
- The catalog factory now accepts the complete typed dependency port rather
  than two standalone callbacks; the composition root and parent ViewModel
  have been migrated together. This is the intentional API change authorized
  by the component-spec design, not a wire or persisted-model change.
- JVM tests passed: 24 component tests across the Path Picker ViewModel/View
  and Session Catalog spec/ViewModel, plus the RPC, Session, Application
  ViewModel and Application View regression suites.
- JVM compilation passed for the component projects and direct consumers,
  including application/settings contracts, RPC contract, Application/Session/
  Settings ViewModels and Application/Settings Views.
- New dependency-only tests cover picker loading/retry/filtering/confirmation,
  duplicate in-flight confirmation, disposal, lazy catalog construction,
  mutation/reload order, serialization with in-flight refresh, and failed
  reload restoration/retry. These do not require a filesystem or RPC server.
- Old generated project directories and old Gradle coordinates were removed.
- Native/JS/CLI validation remains outstanding.
- The user accepted continuation along this component design on 2026-10-02;
  the scoped implementation and review checkpoint are complete. The user later
  authorized submission of the accepted component batches, recorded together
  in inner commit `fc9f1936`. Session Catalog renderer extraction remains
  a follow-up.
