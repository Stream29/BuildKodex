# JVM Toolchain

- Keep regular Kodex JVM toolchains on Java 25.
- Require JetBrains Runtime 25 for Desktop compilation, tests, execution, and native packaging.
- Keep the Foojay resolver enabled so Gradle can provision a JBRSDK with `jpackage`.
- Use a Java 25 SDK for the IDEA project and Gradle JVM.
