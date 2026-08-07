# Path Picker

- Keep filesystem browsing and picker state in `Kodex/app/shared/path-picker`; it must not depend on application, session, or frontend modules.
- Keep the Mosaic popup and terminal rendering in `Kodex/app/cli/path-picker`; it depends on the shared picker but not on application or session modules.
- Use `CoroutineFileSystem`; expand current-user `~` and `~/...` shorthand before resolving the initial path, so accepted paths are absolute runtime paths. Do not expand `~user`.
- Show only direct child directories, sorted case-insensitively by name; files are never selectable.
- Let the user navigate to a child or parent directory and explicitly confirm the current directory.
- Treat filesystem failures as displayable picker state; dismissing the popup or cancelling never changes the caller's path.
- Expose selection through a callback, leaving session persistence and any caller-specific side effects outside the picker module.
