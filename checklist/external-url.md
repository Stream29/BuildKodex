# External URL Opening

Use this checklist when changing system external URL launching.

- Keep the public API as the global `openExternalUrl`; do not add an `ExternalUrlOpener` interface without multiple policies or implementations.
- Keep platform launch commands in `Kodex/utils/external-url/impl` and invoke them directly through `:utils-process-client-impl`'s scope factory and the real `:utils-process-client-spec` contract, never through a shell.
- Treat `OpenExternalUrlResult.Started` as successful handoff to the host URL launcher, not proof that the target application loaded the URL.
- Do not include the URL in launcher failure messages, logs, or persisted UI state; OAuth URLs may contain transient state.
- Keep `OpenAiLoginClient` and ViewModels free of host browser-launch behavior; invoke the utility at the platform UI-effect boundary.
