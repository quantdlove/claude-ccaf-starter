# /publish-staging

Publish files from the workspace to a staging site for team review.

## When to Use
Human invokes `/publish-staging <file-path>` when a file needs to be viewable by teammates via a rendered URL.

## Inputs
- File path relative to workspace root
- `--dry-run` (optional): run gates without pushing

## Step 1: Path Validation
File must NOT be in any of these directories:
- `memory/` (internal context)
- `scheduled-agents/` (agent configs)
- `business-development/` (pipeline state, sensitive leads)

Blocked files: `CLAUDE.md`, `TASKS.md`, any `.json` dotfile, any `*-deal-state.json`.

## Step 2: Content Scan
Block if file contains:
- `api.key`, `token`, `password`, `secret`, `bearer` (credentials)
- Dollar amounts in financial context (real revenue figures)
- Internal email addresses (`@yourcompany.com`)

If blocked: print the matched pattern and line number. Stop. Human fixes the source file.

## Step 3: CSS Dependency Check
For HTML files, scan for external stylesheet references. If the stylesheet isn't on the staging site yet, upload it first.

## Step 4: Push
Upload file via GitHub's "Add file" interface. Write a descriptive commit message.

## Step 5: Verify
Check that the deployment succeeded. Navigate to the rendered URL and confirm it loads.

## Error Handling
- If content scan finds violations, STOP. Do not offer to "just remove that line." Human fixes the source.
- If push fails, retry once. If retry fails, log and stop.
