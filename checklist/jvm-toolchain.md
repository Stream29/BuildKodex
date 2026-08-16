# JVM Toolchain

- Keep regular Kodex JVM toolchains on Java 25.
- Require JetBrains Runtime 25 for Desktop compilation, tests, execution, and native packaging.
- Keep the Foojay resolver enabled so Gradle can provision a JBRSDK with `jpackage`.
- Use a Java 25 SDK for the IDEA project and Gradle JVM.
- In Kotlin Multiplatform modules, keep JVM-emitting common top-level declarations out of files that share a package and basename with JVM source files; use a separate common file to avoid duplicate JVM file facades.
