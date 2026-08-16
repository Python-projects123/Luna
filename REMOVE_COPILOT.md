Remove GitHub Copilot / Co-authored-by trailer

This document explains safe, non-destructive steps an admin or maintainer can take to remove the Copilot/GitHub App (if installed) and to stop the Copilot "Co-authored-by" trailer from appearing in future commits.

1) Verify whether Copilot is installed for this repository or organization

- Web UI (recommended):
  - Repo Settings → Installed GitHub Apps
  - If not found, check Organization Settings → Installed GitHub Apps

- gh CLI (requires admin auth):
  - List repo-level installations (may be empty):
    gh api repos/OWNER/REPO/installation --jq '.'
  - List org installations:
    gh api orgs/ORG/installations --jq '.[] | {id: .id, app: .app_slug}'

2) Uninstall the GitHub Copilot app (admin required)

- Web UI:
  - Go to the Installed GitHub Apps page shown above.
  - Find the entry for "GitHub Copilot" or "Copilot App".
  - Click Uninstall → choose this repository (or the org) → Confirm.

- API (advanced, admin token required):
  - Find installation id from the org installations command above, then run:
    gh api --method DELETE /user/installations/INSTALLATION_ID

3) Remove a repository collaborator (if Copilot is listed as a collaborator)

- Web UI:
  - Repo Settings → Manage access → Manage → find the collaborator → Remove

- gh CLI (admin):
  - gh api repos/OWNER/REPO/collaborators --jq '.[] | .login'
  - gh api --method DELETE repos/OWNER/REPO/collaborators/COLLABORATOR_LOGIN

4) Stop adding Co-authored-by to future commits (non-destructive)

- Identify the source that appends the trailer:
  - Check CI workflows in .github/workflows for any commit message edits.
  - Check repository automation, commit-msg hooks, or third-party integrations.
  - Look for references to the trailer text: "Co-authored-by: Copilot App <223556219+Copilot@users.noreply.github.com>".

- Remove or disable that integration or script. Common places:
  - .github/workflows/*.yml
  - .git/hooks/ (local hooks; remove from contributors' machines if present)
  - Any server-side automation that runs edits on default branch

5) (Optional, destructive) Remove trailer from past commits — DO NOT RUN without team consent

- This rewrites history and requires all collaborators to re-clone. Consider only if legal/compliance requires removing the trailer from history.

- Using git-filter-repo (recommended):
  - Backup: git bundle create repo-backup.bundle --all
  - pip install git-filter-repo
  - git clone --mirror <repo_url> repo-mirror
  - cd repo-mirror
  - git filter-repo --replace-message-contents 'Co-authored-by: Copilot App <223556219+Copilot@users.noreply.github.com>'::''
  - Push: git push --force --all && git push --force --tags
  - Notify all contributors to re-clone

- Using filter-branch (legacy, slower):
  - git filter-branch --msg-filter "sed '/Co-authored-by: Copilot App <223556219+Copilot@users.noreply.github.com>/d'" -- --all

6) If you want me to perform admin actions

- Grant repo admin permissions or provide an installation token with admin scopes.
- I will only proceed with non-destructive uninstallation steps. I will never rewrite history without explicit, signed confirmation.

7) Contact & verification

- After uninstalling Copilot or removing automation, verify:
  - No Copilot entry on Installed GitHub Apps page
  - New commits no longer contain the Co-authored-by trailer

---

If you want, I can now:
- Attempt to uninstall the app using gh (requires admin token)
- Search the repository for scripts that append the trailer and propose targeted edits

Choose one and I will proceed.