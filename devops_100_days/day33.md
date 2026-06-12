# Day 33: Git Remote Synchronization, Merge Conflict Resolution, and Push Recovery

**Date:** 2026-06-08  
**Topic:** Git remote synchronization, branch comparison, conflict resolution, and push recovery  
**Project:** story-blog  
**Environment:** Stratos Data Center  
**Status:** Completed successfully

---

## Task Summary

This task documented a Git workflow on the `story-blog` repository after a local commit had already been created on the `master` branch. The session covered SSH access, repository inspection, remote verification, branch history review, a failed push caused by remote divergence, a merge conflict in `story-index.txt`, and the final successful push after conflict resolution.

---

## Server Credentials and Historical Access Details

| System         | Hostname / URL                           | Username | Password                 | Notes                                           |
| -------------- | ---------------------------------------- | -------- | ------------------------ | ----------------------------------------------- |
| Jump host      | `jumphost`                               | `thor`   | Historical/example value | Source system used to start the session         |
| Storage server | `ststor01`                               | `max`    | Historical/example value | Target system accessed by SSH                   |
| Git remote     | `http://gitea:3000/sarah/story-blog.git` | `max`    | Historical/example value | Remote repository used for push/pull operations |

The SSH host key prompt also displayed the server fingerprint for `ststor01`, which was accepted during the session after manual verification.

---

## Task Output

### 1. SSH Connection and Host Verification

```bash
thor@jumphost ~$ ssh max@stsor01
ssh: Could not resolve hostname stsor01: Name or service not known
thor@jumphost ~$ ssh max@ststor01
The authenticity of host 'ststor01 (10.244.234.214)' can't be established.
ED25519 key fingerprint is SHA256:yEyN8qvzhNxfcKVE+H05zwQPmQMKCXj4JyGWuOP1HIg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01' (ED25519) to the list of known hosts.
max@ststor01's password:
```

### 2. Repository Navigation

```bash
[max@ststor01 ~]$ cd /home/max/
[max@ststor01 ~]$ ls
story-blog
[max@ststor01 ~]$ cs story-blog/
-bash: cs: command not found
[max@ststor01 ~]$ cs story-blog/cd story-blog/
-bash: cs: command not found
[max@ststor01 ~]$ cd story-blog/
[max@ststor01 story-blog]$ ls
fox-and-grapes.txt  frogs-and-ox.txt  lion-and-mouse.txt  story-index.txt
```

### 3. Privilege Escalation Attempt

```bash
[max@ststor01 story-blog]$ sudo su

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

For security reasons, the password you type will not be visible.

[sudo] password for max:
max is not in the sudoers file.
```

### 4. Repository State and Remote Checks

```bash
[max@ststor01 story-blog]$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean

[max@ststor01 story-blog]$ git remote -v
origin  http://gitea:3000/sarah/story-blog.git (fetch)
origin  http://gitea:3000/sarah/story-blog.git (push)

[max@ststor01 story-blog]$ git branch
* master

[max@ststor01 story-blog]$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
```

### 5. Authentication and Push Failures

```bash
[max@ststor01 story-blog]$ git push
Username for 'http://gitea:3000': mac
Password for 'http://mac@gitea:3000':
remote: Failed to authenticate user
fatal: Authentication failed for 'http://gitea:3000/sarah/story-blog.git/'

[max@ststor01 story-blog]$ git push
Username for 'http://gitea:3000': max
Password for 'http://max@gitea:3000':
To http://gitea:3000/sarah/story-blog.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'http://gitea:3000/sarah/story-blog.git'
hint: Updates were rejected because the remote contains work that you do not
hint: have locally.
```

### 6. Branch Checkout Error and Detached HEAD

```bash
[max@ststor01 story-blog]$ git checkout origin/mater
error: pathspec 'origin/mater' did not match any file(s) known to git

[max@ststor01 story-blog]$ git checkout origin/master
Note: switching to 'origin/master'.

You are in 'detached HEAD' state.
...
HEAD is now at 1f876c8 Merge branch 'story/frogs-and-ox'

[max@ststor01 story-blog]$ ls
frogs-and-ox.txt  lion-and-mouse.txt
```

### 7. Commit History Review

```bash
[max@ststor01 story-blog]$ git logs -online
git: 'logs' is not a git command. See 'git --help'.

The most similar command is
        log

[max@ststor01 story-blog]$ git logs -onleine
git: 'logs' is not a git command. See 'git --help'.

The most similar command is
        log

[max@ststor01 story-blog]$ git log -oneline
fatal: unrecognized argument: -oneline

[max@ststor01 story-blog]$ git log --oneline
1f876c8 (HEAD, origin/master, origin/HEAD) Merge branch 'story/frogs-and-ox'
cf9e81f Fix typo in story title
8d09315 Completed frogs-and-ox story
d3e240c Added the lion and mouse story
3e7ce5e Add incomplete frogs-and-ox story
```

### 8. Return to Local Branch and Conflict Discovery

```bash
[max@ststor01 story-blog]$ git checkout master
Previous HEAD position was 1f876c8 Merge branch 'story/frogs-and-ox'
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 1 commit.

[max@ststor01 story-blog]$ git log --oneline
4c57a62 (HEAD -> master) Added the fox and grapes story
1f876c8 (origin/master, origin/HEAD) Merge branch 'story/frogs-and-ox'
cf9e81f Fix typo in story title
8d09315 Completed frogs-and-ox story
d3e240c Added the lion and mouse story
3e7ce5e Add incomplete frogs-and-ox story
```

### 9. Pull Failure and Merge Conflict

```bash
[max@ststor01 story-blog]$ git pull
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0, pack-reused 0 (from 0)
From http://gitea:3000/sarah/story-blog
   1f876c8..70a481f  master     -> origin/master
Auto-merging story-index.txt
CONFLICT (add/add): Merge conflict in story-index.txt
Automatic merge failed; fix conflicts and then commit the result.

[max@ststor01 story-blog]$ git status
On branch master
Your branch and 'origin/master' have diverged,
and have 1 and 1 different commits each, respectively.

You have unmerged paths.
        both added:      story-index.txt
```

### 10. Conflict Content and Final Push

```bash
[max@ststor01 story-blog]$ cat story-index.txt
<<<<<<< HEAD
1. The Lion and the Mooose
2. The Frogs and the Ox
3. The Fox and the Grapes
4. The Donkey and the Dog
=======
1. The Lion and the Mouse
2. The Frogs and the Ox
3. The Fox and the Grapes
>>>>>>> 70a481ff19eda85a5a429312fc43f4d4f9e01012

[max@ststor01 story-blog]$ git add story-index.txt
[max@ststor01 story-blog]$ git commit -m "Fix the story-index.txt"
[master 3b40ae8] Fix the story-index.txt

[max@ststor01 story-blog]$ git pull
Already up to date.

[max@ststor01 story-blog]$ git push
Enumerating objects: 10, done.
Counting objects: 100% (10/10), done.
Delta compression using all threads
Writing objects: 100% (7/7), 1.15 KiB | 1.15 MiB/s, done.
To http://gitea:3000/sarah/story-blog.git
   70a481f..3b40ae8  master -> master
```

---

## Theories and Concepts

### SSH Host Verification

The SSH client verifies the target host identity using a key fingerprint before establishing trust. The first attempt failed because the hostname was mistyped as `stsor01`. The second attempt succeeded after connecting to the correct host name `ststor01` and accepting the fingerprint prompt.

### Git Remote Origin

`origin` is the default name Git uses for the primary remote repository. The command `git remote -v` confirms the fetch and push URLs for the remote. In this task, the remote repository was `http://gitea:3000/sarah/story-blog.git`.

### Branch Divergence and Non-Fast-Forward Pushes

A push can be rejected when the local branch is ahead of the remote but does not contain the remote’s latest commit history. This is known as a non-fast-forward rejection. Git requires the user to synchronize with the remote first, usually by pulling and resolving any conflicts.

### Detached HEAD State

Checking out `origin/master` directly moves the repository into detached HEAD state. In this mode, the user is not on a normal local branch, so new commits would not advance a branch reference unless a new branch is created.

### Merge Conflicts

The `CONFLICT (add/add)` message indicates both the local branch and the remote branch added the same file independently. Git could not choose one version automatically, so the file required manual resolution before the merge could continue.

### Merge In Progress

After a conflict appears, `MERGE_HEAD` remains present until the user resolves the conflict, stages the file, and creates a merge commit. While a merge is incomplete, Git rejects additional pull attempts.

### Command Precision

Several failures were caused by typing incorrect commands such as `cs`, `git logs`, `git log -oneline`, `git checkout origin/mater`, and `git sttatus`. Git command names and options must be entered precisely because Git does not infer misspelled subcommands or flags.

---

## Mistakes and Fixes

### Mistake 1: Incorrect Hostname

**Mistake:** `ssh max@stsor01` used a misspelled hostname.

**Why it happened:** The hostname was typed incorrectly, so DNS resolution failed.

**How it was fixed:** The correct hostname `ststor01` was used in the next SSH command.

### Mistake 2: Incorrect Command Names

**Mistake:** Commands such as `cs`, `git logs`, `git log -oneline`, and `git sttatus` were entered.

**Why it happened:** These were typographical errors or invalid Git command forms.

**How it was fixed:** The correct commands were re-entered, including `cd`, `git log --oneline`, and `git status`.

### Mistake 3: Wrong Branch Name During Checkout

**Mistake:** `git checkout origin/mater` was executed.

**Why it happened:** The branch name was misspelled as `mater` instead of `master`.

**How it was fixed:** The correct remote branch `origin/master` was checked out.

### Mistake 4: Attempting to Use sudo Without Permission

**Mistake:** `sudo su` was attempted on the storage server.

**Why it happened:** The account `max` was not a member of the sudoers file.

**How it was fixed:** The task continued without privilege escalation because the required Git operations were available to the current user.

### Mistake 5: Push Rejected Because the Remote Had New Work

**Mistake:** `git push` was rejected with a fetch-first / non-fast-forward error.

**Why it happened:** The remote repository contained commits that the local branch did not yet have.

**How it was fixed:** `git pull` was used to bring remote changes locally, the merge conflict in `story-index.txt` was resolved, and the changes were committed before pushing again.

### Mistake 6: Merge Conflict in story-index.txt

**Mistake:** `git pull` produced an `add/add` conflict in `story-index.txt`.

**Why it happened:** Both local and remote histories introduced the file independently with different content.

**How it was fixed:** The file was edited to keep the correct final list, staged with `git add story-index.txt`, committed with `git commit -m "Fix the story-index.txt"`, and pushed successfully.

---

## Final Outcome

The repository history was synchronized successfully. The local `master` branch incorporated the remote changes, the conflict in `story-index.txt` was resolved, and the final push updated `origin/master` from `70a481f` to `3b40ae8`.
