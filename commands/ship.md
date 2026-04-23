---
description: Create a PR, wait for CI to pass, then merge to main
argument-hint: [pr-title]
allowed-tools: Bash(git:*), Bash(gh:*), Bash(cargo:*)
---

Create a pull request, watch CI checks until they complete, and merge to main. Follow these steps:

1. **Preflight checks:**
   - Run `git status` to check for uncommitted changes
   - Get the current branch name with `git rev-parse --abbrev-ref HEAD`

2. **Handle uncommitted changes:**
   - If there are uncommitted changes (staged or unstaged), stage them all with `git add -A`
   - Generate a concise commit message based on the diff
   - Commit the changes

3. **Ensure on a feature branch:**
   - If on `main`, create a new branch:
     - Generate a descriptive branch name from the latest commit message (e.g., `update-cargo-lock`, `fix-auth-bug`)
     - Run `git checkout -b <branch-name>`
   - Ensure the branch is pushed to remote: `git log origin/<branch>..HEAD --oneline 2>/dev/null` — if there are unpushed commits, run `git push -u origin <branch>`

4. **Run quality checks:**
   - Run `cargo fmt --check` — if it fails, run `cargo fmt` to fix, then stage and amend the last commit
   - Run `cargo clippy --all-features -- -D warnings` — if it fails, stop and show the errors for the user to fix
   - Run `cargo test --all-features` — if it fails, stop and show the errors for the user to fix
   - If any check fails and cannot be auto-fixed, stop here and do NOT create the PR

5. **Create the pull request:**
   - Run `git log main..HEAD --oneline` to see all commits on this branch
   - Run `git diff main...HEAD --stat` for a summary of changes
   - If $ARGUMENTS is provided, use it as the PR title
   - If no arguments, generate a concise PR title from the commits and diff
   - Create the PR:
   ```
   gh pr create --title "<title>" --body "$(cat <<'EOF'
   ## Summary
   <1-3 bullet points summarizing the changes>

   ## Test plan
   - [ ] CI checks pass

   Generated with [Claude Code](https://claude.com/claude-code)
   EOF
   )"
   ```
   - Capture and display the PR URL

6. **Wait for checks to register, then watch them:**
   - First, wait for at least one check to appear. CI systems take a few seconds to register checks on a new PR. Poll in a loop:
     ```
     for i in $(seq 1 20); do
       CHECKS=$(gh pr checks <pr-number> 2>&1)
       if echo "$CHECKS" | grep -qv "no checks reported"; then
         break
       fi
       sleep 10
     done
     ```
   - If after 20 attempts (200 seconds) no checks have appeared, warn the user: "No CI checks registered on this PR. The repo may not have CI configured for pull requests. Stopping before merge — verify manually and merge with `gh pr merge <pr-number> --squash --delete-branch`." Then **stop — do NOT merge.**
   - Once checks are visible, watch them: `gh pr checks <pr-number> --watch --fail-fast`
   - If `--watch` is not available, fall back to polling manually:
     - Run `gh pr checks <pr-number>` in a loop
     - Wait 30 seconds between polls
     - Continue until all checks have a final status (pass/fail, no "pending")
   - Maximum wait time: 15 minutes (30 polls). If exceeded, report timeout and stop.

7. **Handle results:**
   - **All checks pass:** proceed to step 7
   - **Any check fails:** display the failed checks and their URLs, then stop. Tell the user to fix the failures and run `/ship` again.
   - **IMPORTANT:** Never merge without confirmed passing checks. If you could not determine check status, stop and ask the user.

8. **Merge the PR:**
   - Merge with: `gh pr merge <pr-number> --squash --delete-branch`
   - If squash merge is not allowed by repo settings, try `--merge` instead
   - If branch protection blocks the merge, try with `--admin` flag
   - Display confirmation that the PR was merged

9. **Return to main:**
   - `git checkout main`
   - `git pull origin main`
   - Display a final summary: PR URL, merge status, and current branch
