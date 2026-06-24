# Nullable Types

- Treat every nullable type as a special union type.
- When writing a nullable type, add KDoc explaining why it is nullable.
- Use `@property` for nullable properties, `@param` for nullable parameters, and `@return` for nullable return values.
- The comment must state what `null` means at that site.
- Do not use nullable types to avoid modeling a real state explicitly.
- Prefer a sealed model when `null` would hide multiple distinct states.
- Existing nullable types should be documented when touched for related changes.
