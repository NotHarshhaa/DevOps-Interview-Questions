# **Git & Version Control - DevOps Interview Questions (150 Questions)**

Welcome to the **Git & Version Control** master collection containing **150 comprehensive interview questions and detailed answers** covering Git Internals (Object Database, Blobs, Trees, Commits), Advanced Branching & Merging, Interactive Rebase, Cherry-Pick, Bisect, Submodules, Disaster Recovery via Reflog, and GitOps Workflows.

---

## 🟢 **Part 1: Git Internals & Core Mechanics (Questions 1–50)**

### **1. What is Git and how does its Directed Acyclic Graph (DAG) architecture work?**
**Answer:** Git is a distributed version control system that stores project history as an immutable Directed Acyclic Graph (DAG) of cryptographically hashed objects. Unlike delta-based VCS (SVN), Git takes snapshots of the entire project directory tree at each commit, linking child commits back to parent commits via SHA1/SHA256 hashes.

### **2. Explain the four core Git Object types in `.git/objects/`.**
**Answer:**
1. **Blob:** Stores pure uncompressed file content (data payload) without filename, directory structure, or permissions.
2. **Tree:** Represents a directory; maps filenames, file permissions (`100644`, `100755`), and directory structures to their corresponding Blob and sub-Tree SHA hashes.
3. **Commit:** Contains a pointer to the root Tree object, parent commit SHA(s), author/committer metadata with timestamps, and the commit message.
4. **Annotated Tag:** An independent object pointing to a commit SHA with tagger metadata, timestamp, GPG signature, and tag message.

### **3. Explain the Three Trees (Three Areas) of Git.**
**Answer:**
1. **Working Directory:** The local sandbox files on your physical disk where you actively edit code.
2. **Staging Area (Index):** A binary file (`.git/index`) caching the exact snapshot prepared for the next commit.
3. **Repository (`HEAD`):** The immutable object database in `.git/` containing all committed history.

### **4. How does `git add` work under the hood?**
**Answer:**
1. Compresses file content with zlib and creates a **Blob object** in `.git/objects/` named after its SHA1/SHA256 hash.
2. Updates the **Index file** (`.git/index`), mapping the file path to the newly created Blob SHA.

### **5. How does `git commit` work under the hood?**
**Answer:**
1. Writes **Tree objects** representing directories recorded in the Index.
2. Creates a **Commit object** pointing to the top-level root Tree, parent commit SHA, author metadata, and message.
3. Moves the current branch ref pointer (`.git/refs/heads/<branch>`) forward to point to the new Commit SHA.

### **6. What is `HEAD` in Git and what is a "Detached HEAD"?**
**Answer:**
- **`HEAD`:** A symbolic reference pointer (`.git/HEAD`) pointing to the currently checked-out branch ref (e.g., `ref: refs/heads/main`).
- **Detached `HEAD`:** When `HEAD` points directly to a specific **Commit SHA** rather than a named branch ref. Any new commits created in detached HEAD state are orphaned and will be pruned by garbage collection unless a new branch is created.

### **7. Compare `git merge` vs `git rebase`.**
**Answer:**
- **`git merge`:** Creates a non-destructive 3-way merge commit combining two branch histories. Preserves true historical context and chronological timestamps.
- **`git rebase`:** Replays commits from the current branch on top of the target base branch, rewriting commit SHAs to produce a clean, linear commit history.

### **8. What is a Fast-Forward Merge vs Three-Way Merge (`--no-ff`)?**
**Answer:**
- **Fast-Forward Merge:** If the target branch has no new commits since branching, Git simply moves the branch pointer forward without creating a new merge commit.
- **Three-Way Merge (`--no-ff`):** Explicitly creates a merge commit with two parent commits, preserving visual branch topology.

### **9. What is `git cherry-pick` and how does it work?**
**Answer:** Applies the exact diff introduced by a specific commit from another branch onto your current branch, creating a brand-new commit with a new SHA and timestamp.

### **10. What is `git revert` vs `git reset`?**
**Answer:**
- **`git revert <commit>` (Safe for Public Branches):** Creates a *new* forward commit that applies the exact inverse diff of the target commit, preserving Git history.
- **`git reset` (Dangerous for Shared Branches):** Moves the branch pointer backward, rewriting history.

### **11. Explain `git reset --soft` vs `--mixed` vs `--hard`.**
**Answer:**
- **`--soft`:** Moves branch `HEAD` backward; leaves Staging Area (Index) and Working Directory unchanged (changes remain staged).
- **`--mixed` (Default):** Moves `HEAD` backward and un-stages changes in the Index; leaves Working Directory files untouched.
- **`--hard` (Destructive):** Moves `HEAD` backward and completely wipes all changes in both the Index and Working Directory, discarding uncommitted work.

### **12. What is `git reflog` and how is it used for disaster recovery?**
**Answer:** A local chronological ledger recording every single time `HEAD` moved in your local repository (commits, checkouts, rebases, hard resets).
- **Recovery:** If an engineer accidentally ran `git reset --hard HEAD~5`, running `git reflog` identifies the previous commit SHA before the reset. Running `git reset --hard HEAD@{1}` restores all lost commits in 1 second.

### **13. What is `git stash` and what are `stash pop` vs `stash apply`?**
**Answer:** Temporarily shelves uncommitted changes (dirty working tree) onto an internal stack (`stash@{0}`):
- **`git stash pop`:** Applies stashed changes and removes them from the stash stack.
- **`git stash apply`:** Applies stashed changes but preserves the entry on the stash stack.

### **14. What is `git bisect` and how does it automate debugging?**
**Answer:** Uses binary search algorithms to find the exact commit that introduced a regression bug between a known good commit and a known bad commit:
```bash
git bisect start
git bisect bad HEAD
git bisect good v1.0.0
# Git checks out middle commit; test and mark 'git bisect good' or 'bad'
```

### **15. What is `git clean`?**
**Answer:** Removes untracked files from the working directory (`git clean -fd` removes untracked files and directories).

### **16. What is `.gitignore` and why won't it ignore already tracked files?**
**Answer:** `.gitignore` specifies untracked files Git should ignore. If a file was already committed to Git history before being added to `.gitignore`, Git continues tracking it. To ignore it, untrack it first: `git rm --cached <file>`.

### **17. What is `git submodule` vs `git subtree`?**
**Answer:**
- **`submodule`:** Points to a specific external Git repository commit SHA as an independent nested repo.
- **`subtree`:** Embeds external repository source code directly into the parent repository tree as standard files.

### **18. What is `git commit --amend`?**
**Answer:** Combines staged changes with the most recent commit, rewriting its commit message, Tree, and SHA. *Only use on local un-pushed commits.*

### **19. What is `git pull` vs `git fetch`?**
**Answer:**
- **`git fetch`:** Downloads remote commits, branches, and tags into your local `.git` repository without modifying working files.
- **`git pull`:** Executes `git fetch` followed immediately by `git merge FETCH_HEAD` (or `git rebase` if configured).

### **20. What is `git blame`?**
**Answer:** Annotates each line of a file with the commit hash, author name, and timestamp of the last revision that modified that line.

### **21. What is `git log --graph --oneline --decorate --all`?**
**Answer:** Displays a visual ASCII graph of branch topologies, merge points, tags, and commits in a compact, readable format.

### **22. What is `git diff` vs `git diff --staged`?**
**Answer:**
- **`git diff`:** Shows changes between your Working Directory and the Staging Area (unstaged changes).
- **`git diff --staged` (or `--cached`):** Shows changes between the Staging Area and your last commit (`HEAD`).

### **23. What is a Git Plumbing vs Porcelain Command?**
**Answer:**
- **Porcelain Commands (User-Facing):** High-level commands for daily development (`git add`, `git commit`, `git merge`, `git checkout`).
- **Plumbing Commands (Low-Level Core):** Low-level commands operating on the raw object database (`git hash-object`, `git cat-file`, `git write-tree`, `git commit-tree`).

### **24. What is `git cat-file`?**
**Answer:** Plumbing command to inspect raw Git objects in `.git/objects/`:
- `git cat-file -t <sha>`: Shows object type (`blob`, `tree`, `commit`, `tag`).
- `git cat-file -p <sha>`: Pretty-prints the uncompressed object content.

### **25. What is `git hash-object`?**
**Answer:** Computes the SHA1/SHA256 hash of a file payload and optionally writes it to `.git/objects/` as a zlib-compressed blob (`git hash-object -w file.txt`).

### **26. What is a Git Reference (`ref`)?**
**Answer:** A simple text file located under `.git/refs/` containing a 40-character Commit SHA (e.g., `.git/refs/heads/main` points to the tip commit of `main`).

### **27. What is a Symbolic Reference (`symref`)?**
**Answer:** A ref file that points to another ref file rather than a raw Commit SHA (e.g., `.git/HEAD` contains `ref: refs/heads/main`).

### **28. What is a Remote-Tracking Branch (`refs/remotes/origin/main`)?**
**Answer:** A local read-only bookmark recording the state of the branch on the remote server as of the last `git fetch` operation.

### **29. What is `.git/config` Hierarchy?**
**Answer:** Three-tiered configuration evaluated with increasing precedence:
1. **System (`/etc/gitconfig`):** Applied to all OS users.
2. **Global (`~/.gitconfig`):** Applied to all repos of the current user.
3. **Local (`.git/config`):** Scoped strictly to the current repository.

### **30. What is `git remote` and how do you manage remote upstream mirrors?**
**Answer:**
- `git remote add upstream <url>`: Adds a secondary remote repository (used in open-source fork workflows).
- `git remote -v`: Lists configured remote URLs.
- `git remote prune origin`: Deletes stale local references to remote branches that were deleted on the server.

### **31. What is `git tag` (Lightweight vs Annotated)?**
**Answer:**
- **Lightweight Tag:** A simple pointer file directly containing a Commit SHA (`git tag v1.0`).
- **Annotated Tag:** A full Git object containing tagger identity, timestamp, GPG signature, and message (`git tag -a v1.0 -m "Release v1.0"`).

### **32. What is `git checkout` vs `git switch` vs `git restore` (Git 2.23+)?**
**Answer:**
- `git checkout` was overloaded for both branch switching and file restoration.
- **`git switch <branch>`:** Dedicated command strictly for switching branches.
- **`git restore <file>`:** Dedicated command strictly for discarding working directory or staged file modifications.

### **33. What is `git shortlog`?**
**Answer:** Summarizes `git log` output by grouping commits by author name, displaying commit counts and titles.

### **34. What is `git show`?**
**Answer:** Displays the commit metadata, author, and full unified diff introduced by a specific commit or tag.

### **35. What is `git status -s` (Short Format)?**
**Answer:** Displays repository status in a compact 2-column format (`M` modified, `A` added, `??` untracked).

### **36. What is `git archive`?**
**Answer:** Exports the project tree at a specific commit or tag into a compressed zip or tarball archive without the `.git` directory (`git archive -o release.zip HEAD`).

### **37. What is `git bundle`?**
**Answer:** Packages entire repository branches and commit histories into a single portable binary file, allowing offline git transfer across air-gapped networks.

### **38. What is `git update-ref`?**
**Answer:** Plumbing command to safely update or create a ref file pointing to a specific commit SHA without moving `HEAD`.

### **39. What is `git rev-parse`?**
**Answer:** Converts human-readable branch names, tags, or relative revisions (`HEAD~3`, `main^2`) into canonical 40-character Commit SHAs.

### **40. What is `git merge-base`?**
**Answer:** Finds the best common ancestor commit between two branches, used internally by Git to compute 3-way merge diffs.

### **41. What is `git describe`?**
**Answer:** Finds the most recent reachable annotated tag from a commit and appends the number of additional commits and current commit hash (`v1.4.0-12-g2a1b3c`).

### **42. What is `git rm` vs `git rm --cached`?**
**Answer:**
- `git rm <file>`: Removes file from both the Staging Area and local Working Directory disk.
- `git rm --cached <file>`: Removes file from the Staging Area (stops tracking in Git) while preserving the file on local disk.

### **43. What is `git mv`?**
**Answer:** Renames or moves a file, automatically staging the change in the Index as a delete and add (detected as rename during commit).

### **44. What is `git ls-files`?**
**Answer:** Plumbing command listing all files currently tracked in the Git Index or working directory.

### **45. What is `git ls-tree`?**
**Answer:** Plumbing command listing the contents (mode, type, SHA, name) of a Tree object (`git ls-tree HEAD`).

### **46. What is `git write-tree`?**
**Answer:** Plumbing command that creates a Tree object from the current contents of the Index and writes it to `.git/objects/`.

### **47. What is `git commit-tree`?**
**Answer:** Plumbing command that creates a Commit object given a Tree SHA, parent commit SHAs, and a commit message.

### **48. What is `git pack-objects` and Packfiles (`.pack` / `.idx`)?**
**Answer:** Git compresses loose individual object files into single packfiles using delta compression algorithms, generating index files (`.idx`) for fast binary searching.

### **49. What is `git unpack-objects`?**
**Answer:** Extracts individual loose object files from a packed `.pack` archive file.

### **50. What is `git count-objects`?**
**Answer:** Counts loose and packed objects in `.git/` and reports total disk space consumed by the repository.

---

## 🟡 **Part 2: Advanced Branching, Rebasing & Merge Conflict Mastery (Questions 51–100)**

### **51. How do you execute an Interactive Rebase (`git rebase -i`)?**
**Answer:**
```bash
git rebase -i HEAD~4
```
Opens an editor with options:
- `pick`: Use commit as-is.
- `reword`: Change commit message.
- `edit`: Pause to amend files.
- `squash`: Meld commit into previous commit and combine messages.
- `fixup`: Meld commit into previous commit and discard message.
- `drop`: Delete the commit.

### **52. How do you resolve a complex Git Merge Conflict?**
**Answer:**
1. Git halts on conflict, marking files with conflict markers (`<<<<<<< HEAD`, `=======`, `>>>>>>>`).
2. Inspect conflicting blocks; manually edit code to reconcile desired logic.
3. Stage resolved files: `git add <file>`.
4. Complete the operation: `git merge --continue` or `git rebase --continue`.

### **53. What is `git rerere` (Reuse Recorded Resolution)?**
**Answer:** Automatically records how you resolved a merge conflict. When the exact same merge conflict occurs in the future (during rebases or branch updates), Git resolves it automatically using the recorded solution. Enable via `git config --global rerere.enabled true`.

### **54. What is Trunk-Based Development vs GitFlow?**
**Answer:**
- **Trunk-Based:** Developers merge small, frequent commits into a single shared branch (`main`) multiple times a day, enabling CI/CD.
- **GitFlow:** Heavy model with long-lived branches (`develop`, `feature`, `release`, `hotfix`, `master`) that delays integration and causes merge hell.

### **55. What is `git push --force-with-lease` vs `--force`?**
**Answer:**
- **`--force`:** Unconditionally overwrites the remote branch, potentially destroying teammates' commits pushed in the interim.
- **`--force-with-lease`:** Rejects the push if the remote branch reference has changed since your last fetch, protecting against overwriting un-fetched remote work.

### **56. What is `git gc` (Garbage Collection)?**
**Answer:** Optimizes repository storage by packing loose objects into compressed packfiles (`.pack`), removing orphaned unreachable objects, and pruning old reflog entries.

### **57. What is `git fsck`?**
**Answer:** File System Consistency Check that verifies the cryptographic connectivity and integrity of objects in the Git database, identifying dangling blobs and commits.

### **58. What is `git-filter-repo` (BFG Repo-Cleaner)?**
**Answer:** High-performance tools for permanently stripping sensitive files (leaked passwords, large binaries) from entire Git commit history across all branches and tags.

### **59. What is a Git Hook (`pre-commit`, `commit-msg`, `pre-push`)?**
**Answer:** Scripts placed in `.git/hooks/` that execute automatically at key lifecycle points (e.g., `pre-commit` runs linters and secret scanners before creating a commit).

### **60. What is Git Signed Commits (GPG / SSH)?**
**Answer:** Signing commits with a private cryptographic key (`git commit -S -m "msg"`). GitHub displays a "Verified" badge, proving commit authenticity and author identity.

### **61. What is `git rebase --onto`?**
**Answer:** Replants a branch topic that was branched off an old feature onto a completely different base branch: `git rebase --onto main old_feature client_branch`.

### **62. What is `git cherry-pick -n` (No Commit)?**
**Answer:** Applies the diff from the target commit to your working directory and staging area without creating a commit, allowing you to batch multiple cherry-picks.

### **63. What is `git merge --squash`?**
**Answer:** Combines all commits from a feature branch into a single set of staged changes on the target branch without creating a merge commit, ready to be committed as one clean commit.

### **64. What is `git diff` Three-Dot (`...`) vs Two-Dot (`..`)?**
**Answer:**
- `git diff branchA..branchB`: Shows direct diff between the tips of branchA and branchB.
- `git diff branchA...branchB`: Shows the diff introduced on branchB since its common merge-base ancestor with branchA.

### **65. What is `git log` Three-Dot (`...`) vs Two-Dot (`..`)?**
**Answer:**
- `git log master..feature`: Shows commits present on `feature` but missing from `master`.
- `git log master...feature`: Shows commits present on either branch, excluding commits common to both (symmetric difference).

### **66. What is `git notes`?**
**Answer:** Attaches arbitrary metadata, code review comments, or build URLs to existing commits without modifying commit SHAs.

### **67. What is `git worktree`?**
**Answer:** Allows checking out and developing on multiple branches simultaneously in separate directory folders on disk from a single cloned repository.

### **68. What is `git submodule update --init --recursive`?**
**Answer:** Initializes, clones, and checks out all nested submodules declared in `.gitmodules`.

### **69. What is `git sparse-checkout`?**
**Answer:** Configures the working directory to populate only specified subfolders of a large monorepo, leaving all other files unmaterialized on disk.

### **70. What is `git replace`?**
**Answer:** Creates replacement refs allowing you to graft historical repositories or alter commit history view without modifying underlying commit objects.

### **71. What is `git shallow clone` (`--depth=N`)?**
**Answer:** Clones only the most recent $N$ commits, creating a truncated history to optimize pipeline checkout speeds.

### **72. What is `git blobless clone` (`--filter=blob:none`)?**
**Answer:** Clones all commit trees and history while downloading file blobs on-demand only when checking out files, saving gigabytes in massive monorepos.

### **73. What is `git treeless clone` (`--filter=tree:0`)?**
**Answer:** Clones only commit objects, downloading trees and blobs on-demand during checkout.

### **74. What is `git commit --fixup` and `git rebase --autosquash`?**
**Answer:**
- `git commit --fixup <sha>`: Creates a fixup commit targeting a prior commit.
- `git rebase -i --autosquash`: Automatically reorders and marks all fixup commits to squash into their target parents automatically.

### **75. What is `git checkout -b <branch> --track <remote>/<branch>`?**
**Answer:** Creates a new local branch and configures upstream remote tracking in a single command.

### **76. What is `git branch -D` vs `-d`?**
**Answer:**
- `-d` (Safe): Deletes branch only if it has already been fully merged into upstream.
- `-D` (Force): Forcibly deletes branch regardless of merge status.

### **77. What is `git remote prune origin`?**
**Answer:** Cleans up stale local tracking branches under `refs/remotes/origin/` that have been deleted on the remote GitHub server.

### **78. What is `git branch --merged` vs `--no-merged`?**
**Answer:**
- `--merged`: Lists local branches that have been merged into `HEAD` (safe to delete).
- `--no-merged`: Lists branches containing unmerged commits.

### **79. What is `git push origin :<branch>`?**
**Answer:** Legacy syntax to delete a remote branch on the server (equivalent to `git push origin --delete <branch>`).

### **80. What is `git push origin --tags`?**
**Answer:** Pushes all local tags to the remote repository.

### **81. What is `git tag -d <tag>` and `git push origin :refs/tags/<tag>`?**
**Answer:** Deletes a tag locally and removes it from the remote repository.

### **82. What is `git config --global pull.rebase true`?**
**Answer:** Configures `git pull` to execute a rebase instead of creating a merge commit, maintaining a linear commit history.

### **83. What is `git config --global init.defaultBranch main`?**
**Answer:** Sets `main` as the default branch name for newly initialized repositories.

### **84. What is `git config --global core.autocrlf`?**
**Answer:**
- `true` (Windows): Converts LF to CRLF on checkout, converts CRLF to LF on commit.
- `input` (Linux/macOS): Converts CRLF to LF on commit.

### **85. What is `.gitattributes` and `eol=lf`?**
**Answer:** Declares repository-wide line ending and binary file handling rules committed to Git to enforce consistent LF endings across operating systems.

### **86. What is `.gitattributes` `diff=nodiff`?**
**Answer:** Marks generated files (minified JS, lockfiles) so `git diff` does not clutter reviews with massive text diffs.

### **87. What is `git stash branch <branch>`?**
**Answer:** Creates and checks out a new branch starting from the commit where the stash was created, then applies and drops the stash.

### **88. What is `git stash push -m "message"`?**
**Answer:** Saves a stash with a descriptive label for easy identification in `git stash list`.

### **89. What is `git stash show -p`?**
**Answer:** Displays the full unified diff of changes stored inside a stash.

### **90. What is `git stash drop stash@{N}`?**
**Answer:** Permanently deletes a specific stash entry from the stack.

### **91. What is `git stash clear`?**
**Answer:** Deletes all stashes in the repository.

### **92. What is `git bisect run <script.sh>`?**
**Answer:** Automates bisecting by executing a script that tests code and returns exit code 0 (good) or non-zero (bad), finding the broken commit in seconds.

### **93. What is `git bisect reset`?**
**Answer:** Terminates the bisect session and returns the working directory to the original branch `HEAD`.

### **94. What is `git cherry-pick --abort`?**
**Answer:** Cancels an in-progress cherry-pick operation and restores the repository state prior to the command.

### **95. What is `git rebase --abort` vs `--skip`?**
**Answer:**
- `--abort`: Cancels the rebase and resets to the original pre-rebase state.
- `--skip`: Discards the current conflicting commit entirely and continues rebasing remaining commits.

### **96. What is `git merge --abort`?**
**Answer:** Cancels a conflicted merge and restores the pre-merge state cleanly.

### **97. What is `git log -S "string"` (Pickaxe)?**
**Answer:** Searches commit history for commits that added or deleted occurrences of the specified text string.

### **98. What is `git log -G "regex"`?**
**Answer:** Searches commit history for commits whose patch diffs match the specified regular expression.

### **99. What is `git log --follow <file>`?**
**Answer:** Tracks file history across historical file renames and moves.

### **100. What is `git format-patch -1 <sha>`?**
**Answer:** Generates an emailable patch file representing a single specific commit.

---

## 🔴 **Part 3: GitOps Integration & Enterprise Scenarios (Questions 101–150)**

### **101. Scenario: An engineer accidentally committed AWS secrets to a public GitHub repo 10 commits ago. Walk through the complete remediation.**
**Answer:**
1. **Rotate Keys Immediately:** Invalidate and delete the AWS access key in AWS IAM within 60 seconds.
2. **Purge Git History:**
   ```bash
   git-filter-repo --path-match "secrets.env" --invert-paths
   git push origin --force --all
   ```
3. **Contact GitHub Support:** Request manual cache purge of cached commit views.

### **102. Scenario: A developer accidentally ran `git reset --hard` and lost 3 days of uncommitted and committed work. How do you recover?**
**Answer:**
- **For Committed Work:** Run `git reflog` to locate the commit hash before the reset, then run `git checkout -b recovery_branch <commit_hash>`.
- **For Uncommitted Staged Work:** Run `git fsck --lost-found`, inspect recovered dangling blobs in `.git/lost-found/other/` using `git cat-file -p <blob_sha>`.

### **103. What is Git Sparse Checkout?**
**Answer:** Allows cloning massive enterprise monorepos while checking out only specific subdirectories onto your local disk, saving gigabytes of disk space and build time.

### **104. What is Git Shallow Clone (`git clone --depth 1`)?**
**Answer:** Clones only the single most recent commit without downloading historical commit history, drastically speeding up CI/CD pipeline checkout times.

### **105. What is Git Worktree?**
**Answer:** Allows checking out multiple branches of the same repository simultaneously into different local directory folders on disk without cloning multiple repos (`git worktree add ../hotfix hotfix-branch`).

### **106. What is Git LFS (Large File Storage)?**
**Answer:** Replaces large binary files (videos, dataset models, zip archives) with lightweight pointer files inside the Git repository, storing real large binary payloads on remote LFS servers.

### **107. What is Git Merge Strategy (Recursive, Ort, Resolve, Octupus)?**
**Answer:**
- **Ort (Default in Git 2.34+):** Modern rewrite of the recursive merge strategy; vastly faster with superior conflict detection and automatic rename handling.
- **Octopus:** Merges more than two parent branches in a single merge commit.

### **108. What is Git Bare Repository (`git init --bare`)?**
**Answer:** A repository containing *only* the `.git` database with zero working directory files, used exclusively as remote central server hubs.

### **109. What is Git Patch (`git format-patch` and `git am`)?**
**Answer:** Generates email-compatible plaintext patch files containing commit diffs and metadata, applied onto target branches using `git am`.

### **110. What is an Enterprise GitOps Repository Architecture?**
**Answer:**
- **App Repo:** Contains application source code, Dockerfiles, unit tests. CI builds container images and pushes to OCI registry.
- **GitOps Config Repo (Separate):** Contains declarative Helm/Kustomize environment manifests (dev, staging, prod) monitored by ArgoCD, enforcing strict PR reviews and separation of concerns.

### **111. What is Git Pre-Receive Hook vs Post-Receive Hook?**
**Answer:**
- `pre-receive`: Runs on the remote server before refs are updated; can reject pushes that violate branch protection or contain un-signed commits.
- `post-receive`: Runs after push completes, triggering CI/CD webhooks.

### **112. What is Git Commit Hook (`commit-msg`)?**
**Answer:** Validates commit message structure locally before commit creation (enforces Conventional Commits: `feat:`, `fix:`).

### **113. What is `git maintenance`?**
**Answer:** Runs background optimization tasks (prefetching remotes, loose object packing, commit-graph writing) to keep large repositories fast.

### **114. What is Git Commit-Graph (`.git/objects/info/commit-graph`)?**
**Answer:** Binary cache file of DAG commit graph structures, speeding up `git log --graph` by over 10x in massive repos.

### **115. What is Git Multi-Pack Index (MIDX)?**
**Answer:** Indexes multiple `.pack` packfiles into a single index structure, accelerating object lookups across gigabyte repositories.

### **116. What is Git Server-Side Object Quotas?**
**Answer:** Server-side hooks rejecting commits pushing individual files exceeding size thresholds (e.g., $> 100\text{MB}$).

### **117. What is GitHub Merge Queue?**
**Answer:** Automated train that tests and merges PRs sequentially to prevent broken `main` branches caused by overlapping PR merges.

### **118. What is GitHub Branch Protection Required Reviewers?**
**Answer:** Enforces that PRs must have approval from at least $N$ designated code owners (`CODEOWNERS`) before merge.

### **119. What is `CODEOWNERS` File?**
**Answer:** Declares which teams or users own specific paths and files in a repository, automatically adding them as required PR reviewers.

### **120. What is GitHub Dependabot Automated Pull Requests?**
**Answer:** Automatically opens PRs with dependency version bumps when security vulnerabilities are detected.

### **121. What is Renovate Bot Configuration (`renovate.json`)?**
**Answer:** Declarative dependency automation tool supporting regex managers, auto-merging non-breaking patch releases, and custom schedules.

### **122. What is Semantic Release Automated Versioning?**
**Answer:** Parses Conventional Commit messages in CI to determine SemVer bumps (`feat:` $\rightarrow$ MINOR, `fix:` $\rightarrow$ PATCH, `BREAKING CHANGE:` $\rightarrow$ MAJOR) and publishes GitHub releases automatically.

### **123. What is Release Please?**
**Answer:** Google release tool that maintains a perpetual "Release PR" containing changelog updates and version bumps based on commit history.

### **124. What is Git GPG Key Expiration and Renewal?**
**Answer:** Updating the expiration date on your GPG key (`gpg --edit-key <KEY_ID> expire`) and re-uploading public keys to GitHub to maintain verified commit badges.

### **125. What is SSH Commit Signing in Git 2.34+?**
**Answer:** Using your existing SSH key pair (`git config gpg.format ssh`) to sign commits cryptographically without configuring GPG keys.

### **126. What is Git Credential Manager (GCM)?**
**Answer:** Secure credential helper storing OAuth tokens in OS credential stores (Windows Credential Manager, macOS Keychain) with 2FA/SSO support.

### **127. What is Git Interactive Add (`git add -p`)?**
**Answer:** Allows staging individual hunks of changed files interactively, crafting atomic commits.

### **128. What is Git Patch Mode (`git checkout -p` / `git reset -p`)?**
**Answer:** Interactively discard or un-stage individual code hunks without modifying the rest of the file.

### **129. What is Git Commit `--no-verify` Flag?**
**Answer:** Bypasses local pre-commit and commit-msg hooks (`git commit -m "msg" --no-verify`). *Use with extreme caution.*

### **130. What is Git Push `--no-verify` Flag?**
**Answer:** Bypasses local pre-push hooks during push.

### **131. What is Git Refspec (`refs/heads/*:refs/remotes/origin/*`)?**
**Answer:** Maps remote references to local tracking references in `.git/config`.

### **132. What is Git Force Push Lease Failure (`stale info`)?**
**Answer:** Occurs when another engineer pushed to the remote branch after your last fetch; `--force-with-lease` blocks the push to prevent overwriting their work.

### **133. What is Git Dangling Blob vs Unreachable Commit?**
**Answer:**
- **Dangling Blob:** A blob created via `git add` that was never referenced in any commit.
- **Unreachable Commit:** A commit that is no longer reachable from any branch tip or tag (e.g., after `git reset --hard`).

### **134. What is Git Object Pruning Expiry (`git gc --prune=now`)?**
**Answer:** Forcibly purges all unreachable loose objects from disk immediately rather than waiting for the default 14-day grace period.

### **135. What is Git Reflog Expiry (`git reflog expire --expire=now --all`)?**
**Answer:** Clears all reflog history immediately, making unreferenced commits eligible for instant garbage collection.

### **136. What is Git Submodule Deinit (`git submodule deinit -f <path>`)?**
**Answer:** Unregisters a submodule and wipes its working directory files without deleting `.gitmodules`.

### **137. What is Git Submodule Sync (`git submodule sync`)?**
**Answer:** Synchronizes submodule remote URL changes declared in `.gitmodules` into `.git/config`.

### **138. What is Git Subtree Split (`git subtree split --prefix=<path> -b <branch>`)?**
**Answer:** Extracts the complete historical commit history of a subdirectory in a monorepo into an independent standalone branch.

### **139. What is Git Remote Rename (`git remote rename origin old-origin`)?**
**Answer:** Renames a remote handle and updates all associated remote-tracking branches.

### **140. What is Git Remote Set-URL (`git remote set-url origin <new-url>`)?**
**Answer:** Updates the fetch and push URL of a configured remote.

### **141. What is Git Config `--show-origin`?**
**Answer:** Displays the exact configuration file on disk where each git config setting is defined.

### **142. What is Git Safe Directory (`safe.directory`)?**
**Answer:** Security setting in Git 2.35.2+ preventing execution in repositories owned by different OS user accounts to stop privilege escalation.

### **143. What is Git Credential Helper Store vs Cache?**
**Answer:**
- `store`: Persists plain text credentials unencrypted in `~/.git-credentials`.
- `cache`: Caches credentials in memory for a configurable duration (default: 15 minutes).

### **144. What is Git Environment Variable `GIT_DIR`?**
**Answer:** Overrides the location of the `.git` metadata repository directory.

### **145. What is Git Environment Variable `GIT_WORK_TREE`?**
**Answer:** Overrides the root directory of the working tree.

### **146. What is Git Environment Variable `GIT_AUTHOR_NAME` and `GIT_COMMITTER_NAME`?**
**Answer:** Overrides author and committer identity metadata during commit creation in CI scripts.

### **147. What is Git Environment Variable `GIT_SSH_COMMAND`?**
**Answer:** Specifies custom SSH parameters or private key files (`GIT_SSH_COMMAND="ssh -i ~/.ssh/deploy_key -o StrictHostKeyChecking=no"`).

### **148. What is Git Protocol v2?**
**Answer:** Modern server communication protocol allowing client filtering of refs, reducing bandwidth during `git fetch` in massive repos with millions of tags.

### **149. What is Git Hook Directory Customization (`core.hooksPath`)?**
**Answer:** Redirects Git hooks from `.git/hooks/` to a version-controlled repository folder (`.husky/` or `.githooks/`), sharing hooks across team members.

### **150. What is an Enterprise Version Control Branching & Release Strategy?**
**Answer:**
1. **Trunk-Based Development:** Short-lived feature branches ($< 1$ day) merged to `main` via PRs with mandatory CI tests.
2. **Automated Semantic Releases:** Conventional Commits generate SemVer tags and changelogs.
3. **Signed Commits:** Cryptographic verification enforced on protected branches.
4. **GitOps Promotion:** Separate GitOps config repositories drive continuous deployment to Kubernetes via ArgoCD.
