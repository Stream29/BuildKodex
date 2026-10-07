# Task Tree

- `Accept the independently reviewed moduleTree and immutable SDK consumer batches`()
- `Prepare the isolated Kotlin2.4.20 and Gradle9.6.1 candidate`()
- **`Observe actual plugin classpaths and compare the unchanged full model`()**
- `Validate daemon-first and separate in-process compilation controls`()
- `Validate real JVM JS Native consumers and explicit source-composite compatibility`()
- `if (allVersionGatesPass()) {`
  - `Review the exact candidate independently before production adoption`()
- `} else {`
  - `Stop the version and build-logic migration and return the concrete failure to the user`()
- `}`

# Details

- Parent: [Gradle development experience](2026-10-02-rescue-gradle-development-experience.md).
  Prerequisite: [moduleTree integration](../done/2026-10-08-integrate-explicit-module-tree.md)
  and [SDK root-name/package gate](../done/2026-10-08-maintain-mcp-sdk-composite-root-name.md).
- User-approved compatibility target is Kotlin/KGP2.4.20 with Gradle9.6.1.
  Official fully-supported range includes9.6.1; this establishes eligibility,
  not actual Kodex/plugin compatibility.
  [Kotlin compatibility table](https://kotlinlang.org/docs/gradle-configure-project.html).
- Isolated first candidate changes the canonical Kotlin catalog version,
  replaces the independent buildSrc KGP literal with a catalog library alias,
  and updates the wrapper URL with its official distribution checksum.
  Build-logic migration, target/profile, kRPC and resource defaults remain unchanged.
- Existing TestBalloon, kRPC, Koin and Poko versions are not silently upgraded
  to make the compatibility test pass. Observe their actual compiler/plugin
  behavior; any concrete compatibility failure stops this line for a user decision.
- Default binary mode does not load fork Gradle plugins. Explicit source mode
  remains a separate required gate. Any fork adaptation belongs only to its
  `kodex-submodule` maintenance branch and a newly reviewed immutable version,
  never an upstream cooperation branch or overwritten package.
- Heavy execution only on Xiaoxin under the existing device lock. A wrapper
  transition occurs only after the completed, verified owned daemon is released;
  then explicitly select the actual JVM. No incompatible available daemon may
  be ignored while a new server is launched.
- Model equivalence compares IDs/directories, targets, source roots/parents and
  dependency edges; expected plugin codeSource and compiler version changes
  are separately recorded. CLI `help` and model extraction do not certify IDEA Sync.
- No production version commit, build-logic implementation or performance claim
  before the specified gate succeeds. Preserve failed outputs and exact hashes.

## Exact first experiment

- Accepted product predecessor: settings-only9458be6f and SDK consumer2e5c0df4.
  Isolated source archive9458be6f has the subsequently accepted51ca6e catalog
  suffix plus exactly the three compatibility candidate files.
- Kotlin2.4.20 catalog + real `libs.kotlin.gradle.plugin` buildSrc dependency;
  official Gradle9.6.1 archive SHA256
  `9c0f7faeeb306cb14e4279a3e084ca6b596894089a0638e68a07c945a32c9e14`.
  [Official distribution checksum](https://services.gradle.org/distributions/gradle-9.6.1-bin.zip.sha256).
- Our completed owned9.5.1 daemon PID1100746 was explicitly released only after
  all SDK/model/source/PTY gates finished. No available Gradle server remained
  before the9.6.1 invocation; selected JVM stays Temurin25.0.4.
- First isolated `help` passed in118s. Actual full model/classpath observation
  and representative compiler/plugin tests are running on its reused9.6.1
  daemon PID1120008. `help` alone does not pass the compatibility line; no
  production version change is declared yet.
