# Task Tree

- [done] Remove global settings schema version
  - [done] Define schema-less compatibility semantics
  - [done] Replace versioned loading and migration models
  - [done] Preserve defaults and canonical writes
  - [done] Add tolerant loading regression tests
  - [done] Update durable global settings rules
  - [done] Run relevant validation

# Details

- Unknown YAML keys are ignored.
- Missing settings use current application defaults.
- Removed or renamed keys do not require conversion.
- Loading does not rewrite `settings.yml`; normal settings updates write the current canonical shape.
- Known fields with invalid values remain configuration errors.
- IntelliJ lint reported no problems in the changed Kotlin files.
- JVM and Linux x64 settings-store tests passed.
- The Linux x64 release executable linked successfully.
- An isolated CLI smoke test accepted schema 2 and unknown keys, opened the application, and left the file unchanged.
