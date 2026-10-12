# JVM Toolchain

- Keep regular Kodex JVM toolchains on Java 25.
- Require JetBrains Runtime 25 for Desktop compilation, tests, execution, and native packaging.
- Keep the Foojay resolver enabled so Gradle can provision a JBRSDK with `jpackage`.
- Use a Java 25 SDK for the IDEA project and Gradle JVM.
- Check JVM file-facade collisions across every source set in the same compilation, including `specMain`, `commonMain`, and platform layers. Files in the same package with the same basename and JVM-emitting top-level declarations can collide; rename an implementation file when necessary. A file containing only interfaces or classes does not by itself emit a top-level file facade.
