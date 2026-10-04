# Spec/Impl Module Boundaries

- Organize each domain's specification projects under `spec/` and implementation projects under `impl/`; Gradle project paths must reflect those directories.
- Every implementation Gradle project must have a corresponding, meaningful spec project. When no contract can be identified, reconsider the project boundary rather than create an empty interface, marker type, or copied DTO.
- A spec is the authoritative source for its code-level contract **and the KDoc attached to that contract**: values, operations, observable states, lifecycle and error semantics, and compatibility guarantees.
- Document contractually observable exceptions on the throwing spec operation with KDoc `@throws`, including the condition that triggers each exception; do not leave that behavior documented only on an implementation or exception class.
- Implementation code may carry KDoc for its concrete behavior and mechanism. That KDoc is not a second normative source and must not contradict the spec; do not remove it merely because a spec exists.
- Move KDoc with declarations that move. When extracting a spec interface from concrete code, state cross-implementation guarantees on the spec; retain implementation-specific details with the implementation.
- Target dependency direction is `spec -> spec`, `impl -> spec`, and `impl -> impl` when composition requires it. Track dependencies on mixed projects outside the current migration batch explicitly; do not claim the final graph before those domains are migrated.
- Preserve persisted and RPC serialization shapes, ownership, and behavior during module migration. A necessary contract or wire change requires its own review rather than being hidden in a move.
- Component migrations are hard cutovers: move the real ViewModel contract and implementation, remove the replaced declarations, and update all consumers. Do not introduce a parallel public model plus adapter, resolver, alias, or forwarding project merely to compile against the old host. Ordinary storage/RPC dependency binding and renderer-local visual composition remain valid boundaries.
