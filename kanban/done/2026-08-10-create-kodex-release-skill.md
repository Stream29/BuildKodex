# Task Tree

- [done] Simplify the project-local Kodex release skill
  - [done] Retain the established release steps and commit exception
  - [done] Replace the skill body with one unordered list
  - [done] Remove auxiliary skill and findings files
  - [done] Validate the single-file skill

# Details

- Create the skill under `.agents/skills/release-kodex/` in the BuildKodex repository.
- Permit the agent to create the exact `chore: bump version` commit in both `Kodex/` and BuildKodex.
- Do not broaden the commit exception to unrelated changes or other commit messages.
- Keep only `.agents/skills/release-kodex/SKILL.md`.
- Use one plain unordered list for the workflow.
- `quick_validate.py` passed and IntelliJ IDEA reported no file problems.
- No Git or release operation was performed.
