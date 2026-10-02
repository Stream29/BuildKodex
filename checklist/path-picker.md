# Path Picker

- During the first same-process RPC migration, keep directory browsing local to the frontend, following [the directory-selection boundary](rpc-architecture.md#工作目录选择); do not add a directory RPC.
- Keep the component contract in `Kodex/app/component/path-picker/spec`, including its ViewModel dependencies, factory, interactions, state/effects, and renderer semantics. Its KDoc is the component fact source; follow [the component boundary](frontend-application-boundary.md).
- Keep filesystem-backed state in `Kodex/app/component/path-picker/impl/viewmodel`; inject the spec's browser port rather than expose the concrete filesystem to the renderer. Neither spec nor ViewModel depends on application, session, or renderer modules.
- Keep the Mosaic popup and terminal rendering in `Kodex/app/component/path-picker/impl/view/src/mosaicMain`; it depends on the picker spec but not on application or session modules.
- Use `CoroutineFileSystem`; expand current-user `~` and `~/...` shorthand before resolving the initial path, so accepted paths are absolute runtime paths. Do not expand `~user`.
- Show only direct child directories, sorted case-insensitively by name; files are never selectable. Use listed child paths directly and skip entries with absent metadata, so a dangling symbolic link does not fail the directory.
- Let the user navigate to a child or parent directory and explicitly confirm the current directory. Keep `Select` above the directory list as the default focus target, including while child directories load; it is a browser action, not a trailing dialog confirmation beside `Cancel`. Selecting validates and resolves the current path independently of listing its children, so listing latency or failure does not disable confirmation. An invalid path must not be returned.
- In the CLI picker popup, map a `Button8` press inside the popup to the same enabled `Up` action used by the button.
- Treat unmodified letter input in the CLI picker as a case-insensitive directory-name substring filter; show the query and focus the first match so Enter enters it. After successful navigation clears the filter and removes that child target, restore focus to the leading `Select` action so a second Enter selects the entered directory.
- Let Backspace edit the active filter, let Escape clear it before dismissing the popup, and clear it when directory navigation starts a new request.
- Treat filesystem failures as displayable picker state; dismissing the popup or cancelling never changes the caller's path.
- Expose selection through a callback, leaving session persistence and any caller-specific side effects outside the picker module.
- Working-directory callers use `app/component/working-directory` to own the
  browser and bind selection to their exact target/handle/revision. Its renderer
  borrows the picker without independently disposing it. Standalone picker
  rendering retains its existing three-argument API and disposal ownership.
