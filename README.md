# gitlab-sync

`gitlab-sync` is a centralized, automated backup environment designed to mirror an entire GitHub profile straight into a GitLab archive organization. Powered completely by a zero-dependency GitHub Actions shell script, it bypasses individual repository hooks to batch-sync your codebases on an intervals-based routine.

---

## ⚙️ Architecture & Mirroring Flow

The sync process leverages high-fidelity Git mirroring protocols to preserve all commit history, tags, references, and remote branches:


1. **Bare Clone**: Pulls down full remote metadata using the temporary internal `GITHUB_TOKEN`.
2. **Safety Exclusions**: Scans repository profiles and selectively filters out outsized targets to stay within free tier limits.
3. **Primary Mirror**: Executes `git push --mirror` to match the remote target frame perfectly.
4. **Resilient Fallback**: If server-side branch rules interrupt the mirror pipeline, it triggers forced-head ref-pushes (`refs/heads/*`, `refs/tags/*`) to prevent sync degradation.

---

## 🚀 How to Set Up Your Own Backup Sync

The workflow configuration in this repo is a **generic template built for your convenience**. Follow these steps to configure it for your own profiles:

### 1. Provision Your GitHub Secrets & Permissions

#### A. Configure the GitLab Token (`BACKUP_TOKEN`)
This token gives your GitHub Action runner permission to write code and create repositories on your GitLab account. Follow these exact steps:

1. **Log in** to your GitLab account.
2. On your **left-hand sidebar**, click on **Access > Personal Access Tokens**.
3. Scroll down the page and click on **Generate token > Legacy token**.
4. Configure the token details precisely:
   * **Name**: Type **`BACKUP_TOKEN`** exactly.
   * **Description**: Add a description if you like.
   * **Expiration date**: Set the date to **exactly one year more than the current date** (e.g., if today is 15 March 2021, set the expiration date to 15 March 2022).
5. Select the **`write_repository`** and **`api`** checkboxes in the scopes section.
6. Click on **Generate token** and **please copy the token immediately**—you won't see it again!
7. Finally, click on **Done**.
8. **Save it in GitHub**: Go to your GitHub repository hosting this workflow. Navigate to **Settings > Secrets and variables > Actions**, click **New repository secret**, and save the value with the secret name **`BACKUP_TOKEN`**.

#### B. Update GitHub Workflow Permissions
GitHub Actions automatically generates a temporary token named `${{ secrets.GITHUB_TOKEN }}` every time your workflow spins up so the runner can clone your repositories. For the script to execute cleanly, you must update its access level:

1. In your GitHub repository, go to **Settings > Actions > General**.
2. Scroll down to the **Workflow permissions** section.
3. Ensure it is set to **Read and write permissions**.
4. Click on **Save**.

### 2. Activate the Workflow Template
1. Navigate to `.github/workflows/` in your repository.
2. Copy the template file `sync-all.yml.example` and rename the copy to `sync-all.yml`.
3. Open `sync-all.yml` and modify the fields to point to your target accounts and repository names:

```bash
# 🟢 CHANGE THESE FIELDS TO MATCH YOUR TARGET PROFILES:
GH_USER="YOUR_GITHUB_USERNAME"            # The GitHub profile you are backing up
GL_USER="YOUR_GITLAB_BACKUP_USERNAME"     # The GitLab user/group where backups go

REPOS=(
  "repo-one"
  "repo-two"
  "repo-three"
  "monorepo-one"
  "monorepo-two"
  "my-awesome-project"
)
```

*(Note: The default names left inside the `.example` file are just placeholder references for your convenience!)*

---

## 📅 Automation Engine Schedule

Once renamed and activated, you can customize the execution loop inside `sync-all.yml`:
*   **Automated Interval**: Set your chosen execution interval inside the `schedule: - cron:` string at the top of the file (e.g., `'0 */6 * * *'` to run every 6 hours).
*   **Manual Override**: Built-in `workflow_dispatch` support lets you execute a complete sync pipeline via the GitHub Actions dashboard UI with a single click at any time.

---

## 📄 License

Distributed safely under the terms of the [MIT License](LICENSE).
