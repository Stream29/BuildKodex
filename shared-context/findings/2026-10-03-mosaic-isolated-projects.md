# Mosaic Isolated Projects research

**Date:** 2026-10-03  
**Device:** Xiaoxin Ubuntu  
**Scope:** isolated exact-main validation copies; no public fork or gitlink changes

## Finding

Mosaic's root `allprojects {}` block was the direct Isolated Projects blocker.
Replacing it with project-local convention application is sufficient to remove
the Mosaic-side cross-project configuration violations while preserving the
existing 18-project graph.

The next blocker was not Mosaic's root script but the external
`de.infolektuell.jextract` 1.0.0 plugin. Its implementation registers a shared
service with `project.rootProject.layout`, which violates Isolated Projects when
applied to `mosaic-tty`. Plugin 1.4.0 still emitted that violation in this
environment and introduced a KMP/Java plugin incompatibility.

An isolated source copy of the plugin with that path made project-local passed
the Mosaic IP `help` diagnostic. The jextract task remained present and a
compile of `:mosaic-tty:compileKotlinJvm` succeeded under the existing KGP
2.3.21 line.

## Required production shape

- Move Mosaic-wide defaults into a convention applied by each project; do not
  replace `allprojects` with `subprojects`.
- Maintain the same project graph and target/source-set semantics.
- Either upstream an IP-safe jextract fix or carry a clearly identified
  Kodex/Mosaic-local compatibility implementation in the fork.
- Keep the jextract shared-service cache under the current project, not the
  root project.
- Re-run normal help, IP help, jextract task model, JVM compile, and
  host-specific Native checks before changing `kodex-submodule`.

## Version-line gate

Mosaic IP `help` passed with KGP 2.4.20, but its JVM compile failed in
`app.cash.burst` with an
`IrGenerationExtension`/`ProjectExtensionDescriptor` `ClassCastException`.
The user previously required that a failed KGP 2.4.20 + Gradle 9.6.1
validation stop the migration and return for a decision. Therefore this is a
separate unresolved version gate, not evidence against the Mosaic IP design.

## Evidence

- Research report/task checkpoint:
  [Mosaic IP task](../../kanban/done/2026-10-03-migrate-mosaic-for-isolated-projects.md)
- Xiaoxin prototype:
  `~/ACodeSpace/demo/kodex-gradle-research-445/main-validation-445/mosaic-ip-local-jextract-445/`
- KGP 2.4.20 failure:
  `.../mosaic-ip-local-jextract-445/results/mosaic-tty-compile-kgp2420.log`
