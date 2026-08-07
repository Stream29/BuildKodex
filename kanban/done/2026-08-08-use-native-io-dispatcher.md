# Task Tree

- [done] Use the Kotlin/Native IO dispatcher for blocking operations
  - [done] Inventory Native dispatcher actuals backed by `Dispatchers.Default`
  - [done] Replace blocking Native lanes with `Dispatchers.IO`
  - [done] Update the durable dispatcher constraint
  - [done] Validate affected Native modules

# Details

The user selected kotlinx.coroutines 1.11's built-in Native `Dispatchers.IO`, which grows workers on demand and retains its internal 2048-worker safety limit. Keep downstream APIs unchanged. The cancellation-time process-session ownership leak is outside this focused change.

Validated Linux and MinGW Native compilation for the affected modules. Linux Native tests passed for `utils-kotlinx-io-coroutines`, `utils-process-client`, and `utils-shell-client`.
