---
description: Create branch, stage changes (excluding binaries), commit, and push
argument-hint: [branch-name]
allowed-tools: Bash(git:*), Bash(file:*), Read, Edit, Bash(cat:*), Bash(echo:*)
---

Create a new git branch, intelligently stage files, commit changes, push to remote, then return to main and rebase. Follow these steps:

1. **Generate branch name:**
   - If $ARGUMENTS is provided, use it as the branch name exactly as given
   - If no arguments, analyze the git diff to create a descriptive branch name (kebab-case, e.g., "update-auth-flow")
   - Fetch all branches with `git branch -a` and ensure the name is unique
   - If conflict exists, append a numeric suffix (e.g., "fix-bug-2")

2. **Create and checkout the new branch:**
   - Run `git checkout -b <branch-name>`

3. **Intelligently stage files:**
   - Get list of all modified and untracked files with `git status --porcelain`
   - For each file:
     - If it's already tracked and modified, check if it's a binary using `file` command
     - If it's untracked, check if it's a binary using `file` command
     - Binary detection: look for "executable" or "binary" in file output, but EXCLUDE image files (PNG, JPEG, GIF, etc.)
     - For non-image binaries: add to .gitignore if not already there, do NOT stage
     - For all other files (text, source code, images): stage them with `git add`
   - Show summary of what was staged vs ignored

4. **Commit the changes:**
   - Analyze the staged changes with `git diff --cached`
   - Create a concise, descriptive commit message based on the changes
   - Commit with the message, including Claude Code attribution:
   ```
   git commit -m "$(cat <<'EOF'
   <Your commit message here>

   🤖 Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>
   EOF
   )"
   ```

5. **Push to remote:**
   - Push the branch with `git push -u origin <branch-name>`
   - Display the branch name and success message

6. **Return to main and rebase:**
   - Checkout main branch: `git checkout main`
   - Pull latest changes: `git pull origin main`
   - Rebase the feature branch onto main: `git rebase main <branch-name>`
   - If rebase conflicts occur, report them to the user

IMPORTANT: Be thorough in binary detection. Common binaries to exclude: executables, .so, .dylib, .dll, .a, .o files. But DO stage images like .png, .jpg, .gif, .svg, .ico, .webp.
