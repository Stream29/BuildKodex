# Task Tree

- `Trace draft materialization and repeated entry points`()
- `Determine whether normal in-memory CLI can retain a retryable draft after cancellation`()
- `Record deferred defensive risk without changing frontend behavior`()

# Details

## Finding and applicability

- [RpcSessionDraft.materialize](../../Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionDraft.kt#L43-L65) sets `submitted` only after `appendUserMessage` returns. A reply can be lost after backend commit; another `materialize()` can then append the same text again.
- The existing [lost-reply test](../../Kodex/app/viewmodel/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionViewsTest.kt#L341-L365) **artificially** throws in a test client after a successful delegate call. It proves the local retry path, not that the normal in-memory connection drops replies.
- The production [in-memory connection](../../Kodex/rpc/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/rpc/inmemory/WithInMemoryRpc.kt#L26-L35) is one process with channel transport; unexpected endpoint death terminates the host instead of allowing routine retry. There is no append-specific timeout in the frontend.
- A narrower ambiguity remains in principle: [BackendSessionHost.inSession](../../Kodex/rpc/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHost.kt#L121-L136) detaches accepted work from its waiter. If the frontend waiting coroutine is cancelled after acceptance while the app and draft remain live, the backend can commit without a successful return. Whether normal UI navigation produces a **retryable surviving draft** in that case has not been reproduced.
- The [Application replacement](../../Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L155-L175) happens only after `draft.materialize()` returns. On the lost reply it leaves the draft tab and text in place; the [submit key](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L470-L481) invokes the same method again.
- Creation and initial append are separate commits. [Session design](../../checklist/rpc-session.md#创建与前端关闭) requires keeping the created Session and retryable content after append failure, while [state design](../../checklist/rpc-state.md#全量-cas) rejects blind replay of unknown writes.

## Conclusion

- User agreed to **defer** a frontend fix for the synthetic lost-reply case and proceed to the next verified finding. There is no demonstrated normal UI path in which a cancelled waiter leaves a live, retryable draft.
- The single-process in-memory transport does not randomly drop a reply; endpoint failure ends the host. Do not treat the injected test as production reproduction or add an unapproved warning/transfer workflow.
- Reopen this issue if a surviving-draft cancellation path is reproduced or if a later transport supports reconnect after a command outcome becomes unknown.

- The frozen append RPC has no idempotency key or authoritative commit receipt in a failed response. Do not “fix” this by silently retrying, by marking every failed append successful, or by changing the RPC contract.
