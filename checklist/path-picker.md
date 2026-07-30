# Path Picker

- Keep the directory picker in the independent `cli/path-picker` module; it must not depend on the app or session modules.
- Use `CoroutineFileSystem` and resolve the initial path before presenting it, so accepted paths are absolute runtime paths.
- Show only direct child directories, sorted case-insensitively by name; files are never selectable.
- Let the user navigate to a child or parent directory and explicitly confirm the current directory.
- Treat filesystem failures as displayable picker state; dismissing the popup or cancelling never changes the caller's path.
- Expose selection through a callback, leaving session persistence and any caller-specific side effects outside the picker module.
