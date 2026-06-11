Day 34: Git Hooks — Server-side and Client-side
Date: 2026-06-09
Topic: Demonstration of client- and server-side Git hooks; resolving common mistakes observed while operating on a bare repository and a working tree.
Project: kodekloud/devops_100_days
Version: 1.0

## Server Credentials (Historical / Example)

- Jump host: thor@jumphost
- Target server: ststor01 (10.244.234.223)
- Username: natasha
- SSH port: 22
- Password (historical/example): Str0ngP@ssword! (no longer active)

## Other Credentials (Historical / Example)

- Example API key: AKIAEXAMPLEKEY (no longer active)
- Example DB password: dbP@ssw0rd! (no longer active)

## Task Output
Below is the verbatim terminal session (historical) captured during the task.

```
thor@jumphost ~$ ssh natasha@ststor01
The authenticity of host 'ststor01 (10.244.234.223)' can't be established.
ED25519 key fingerprint is SHA256:yEyN8qvzhNxfcKVE+H05zwQPmQMKCXj4JyGWuOP1HIg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01' (ED25519) to the list of known hosts.
natasha@ststor01's password: 
[natasha@ststor01 ~]$ 
[natasha@ststor01 ~]$ 
[natasha@ststor01 ~]$ cd /usr/src/k
-bash: cd: /usr/src/k: No such file or directory
[natasha@ststor01 ~]$ cd /usr/src/kodekloudrepos/
[natasha@ststor01 kodekloudrepos]$ 
[natasha@ststor01 kodekloudrepos]$ ls
official
[natasha@ststor01 kodekloudrepos]$ cd official/
[natasha@ststor01 official]$ ls
feature.txt  info.txt
[natasha@ststor01 official]$ git status
On branch feature
nothing to commit, working tree clean
[natasha@ststor01 official]$ 
[natasha@ststor01 official]$ 
[natasha@ststor01 official]$ cd .git/hooks/
[natasha@ststor01 hooks]$ touch pre
[natasha@ststor01 hooks]$ touch pre-push
[natasha@ststor01 hooks]$ chmod +x pre-push
[natasha@ststor01 hooks]$ vi pre
pre                        pre-merge-commit.sample    pre-rebase.sample          
pre-applypatch.sample      pre-push                   pre-receive.sample         
pre-commit.sample          pre-push.sample            prepare-commit-msg.sample  
[natasha@ststor01 hooks]$ vi pre-push
[natasha@ststor01 hooks]$ git branch
* feature
  master
[natasha@ststor01 hooks]$ git checkout master
fatal: this operation must be run in a work tree
[natasha@ststor01 hooks]$ cd ..
[natasha@ststor01 .git]$ cd ..
[natasha@ststor01 official]$ git checkout master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
[natasha@ststor01 official]$ git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
[natasha@ststor01 official]$ ls
info.txt
[natasha@ststor01 official]$ git merger feature
git: 'merger' is not a git command. See 'git --help'.

The most similar command is
        merge
[natasha@ststor01 official]$ git merge feature
Updating eb2b368..336d3b3
Fast-forward
 feature.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 feature.txt
[natasha@ststor01 official]$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
[natasha@ststor01 official]$ git push
==================================================
🚀 [Git Hook] Pushing to master detected!
🏷️  Creating local release tag: release-2026-06-09
==================================================
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To /opt/official.git
   eb2b368..336d3b3  master -> master
[natasha@ststor01 official]$ 
[natasha@ststor01 official]$ 
[natasha@ststor01 official]$ 
[natasha@ststor01 official]$ rm .git/hooks/pre-push
[natasha@ststor01 official]$ cd /opt/official.git/hooks
touch post-update
chmod +x post-update
[natasha@ststor01 hooks]$ vi post-update
[natasha@ststor01 hooks]$ cat post-update
#!/bin/sh

# The post-update hook receives the names of refs that were updated as arguments
for ref in "$@"
do
    if [ "$ref" = "refs/heads/master" ]; then
        # Dynamically generate today's date format (YYYY-MM-DD)
        TAG_NAME="release-$(date +%Y-%m-%d)"
        
        echo "=================================================="
        echo "🚀 [Server Hook] master updated! Creating tag: $TAG_NAME"
        echo "=================================================="
        
        # In a bare server repo, we apply the tag directly to the latest commit on master
        git tag "$TAG_NAME" master
    fi
done
[natasha@ststor01 hooks]$ cd
[natasha@ststor01 ~]$ cd /usr/src/kodekloudrepos/official/
[natasha@ststor01 official]$ ls
feature.txt  info.txt
[natasha@ststor01 official]$ echo "Triggering server hook" >> info.txt
git add info.txt
git commit -m "Trigger post-update hook"
Author identity unknown

*** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your account's default identity.
Omit --global to set the identity only in this repository.

fatal: unable to auto-detect email address (got 'natasha@ststor01.(none)')
[natasha@ststor01 official]$ git push origin master
Everything up-to-date
[natasha@ststor01 official]$ git fetch --tags
git tag
release-2026-06-09
[natasha@ststor01 official]$ git config user.email "natasha@stratos.dc"
git config user.name "Natasha"
[natasha@ststor01 official]$ git add info.txt
git commit -m "Trigger post-update hook"
git push origin master
[master dfd698a] Trigger post-update hook
 1 file changed, 1 insertion(+)
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 16 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 338 bytes | 338.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
remote: ==================================================
remote: 🚀 [Server Hook] master updated! Creating tag: release-2026-06-09
remote: ==================================================
To /opt/official.git
   336d3b3..dfd698a  master -> master
[natasha@ststor01 official]$ 

```

## Theories

### What Are Git Hooks?
Git hooks are custom scripts that Git executes automatically when specific repository events occur (e.g., commit, push, merge). They are used to enforce policy, run checks, or trigger automation.

### Client-side vs Server-side Hooks
- Client-side hooks run on a user's machine (pre-commit, pre-push) to validate or block operations before they happen.
- Server-side hooks run on the Git server (pre-receive, post-receive, post-update) and are used for centralized enforcement or deployment actions.

### Hook Implementation Notes
- Hooks can be written in any scripting language available on the host (sh, bash, python, node).
- Hooks must be executable (e.g., `chmod +x pre-commit`) to run.
- Hooks are not propagated via `git push` by default; share via a project-level `.githooks` directory and `git config core.hooksPath .githooks`, or use tooling (Husky) to distribute hooks.

## Mistakes and Fixes

### Mistake 1: Running work-tree commands from inside `.git/hooks`
- Description: Attempted `git checkout master` while located in `.git/hooks` and received: `fatal: this operation must be run in a work tree`.
- Why it happened: Commands that operate on the working tree must be executed from the repository work tree. When inside `.git` (or in a bare repo), Git lacks a work-tree context.
- How it was fixed:
  1. Change directory to the repository work tree (e.g., `cd /usr/src/kodekloudrepos/official`).
 2. Re-run `git checkout master` in the work tree.
 3. For scripted operations in bare repositories, use `--git-dir` and `--work-tree` or perform ref updates directly.

### Mistake 2: Typo `git merger feature`
- Description: Executing `git merger feature` produced a `git: 'merger' is not a git command` error.
- Why it happened: Typographical error — the correct subcommand is `merge`.
- How it was fixed:
  1. Run `git merge feature`.
  2. Verify the merge result with `git status`.

### Mistake 3: Author identity unknown (commit failure)
- Description: `fatal: unable to auto-detect email address` when attempting to commit.
- Why it happened: Git requires `user.name` and `user.email` to create commits; they were not configured for the environment.
- How it was fixed:
  1. Configure identity locally or globally: `git config user.email "natasha@stratos.dc"` and `git config user.name "Natasha"` (or use `--global`).
  2. Re-run `git add` and `git commit` to create the commit.

### Mistake 4: Deleting client-side hook (`pre-push`)
- Description: Removing `.git/hooks/pre-push` eliminated the local push-time checks and altered workflow behavior.
- Why it happened: Hook files are local; deleting them removes enforcement.
- How it was fixed:
  1. Restore the hook from a shared hooks repository (`.githooks`) or version control if available.
  2. Prefer sharing hooks via `git config core.hooksPath .githooks` and storing scripts in the repo root so team members can fetch them.

### Mistake 5: Server-side tag visibility and expectations
- Description: Server hook created a tag on the bare repository (`release-YYYY-MM-DD`) when master was updated; users may not see server-created tags locally until fetched.
- Why it happened: Tags created in the server's bare repository are present on the server; clients must fetch tags or the server must push tags to any downstream remotes.
- How it was fixed / recommended practice:
  1. Ensure server hooks create tags intentionally and document tag naming.
  2. Users should run `git fetch --tags` to retrieve server-created tags.
  3. Consider publishing tags to a shared remote or document tag creation behavior as part of deployment policy.

## Recommendations and Best Practices

- Share hook scripts via `.githooks` and set `core.hooksPath` to ensure reproducible client-side checks.
- Maintain clear documentation for server-side hooks (expected side effects, tag naming, required permissions).
- When automating operations that touch both bare and non-bare repositories, script with explicit `--git-dir`/`--work-tree` or operate on refs to avoid work-tree errors.
- Always configure `user.name` and `user.email` for service accounts or automation to ensure commits are attributable.

---

End of Day 34 documentation.
