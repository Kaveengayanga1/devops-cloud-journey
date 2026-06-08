Date: 2026-05-22
Topic: Git PR workflow — Merge `story/fox-and-grapes` into `master` (Gitea)

Summary

- Demonstration of a developer (Max) pushing a feature branch to a remote Gitea repo, creating a Pull Request (PR), assigning a reviewer (Tom), and merging into `master` after review.

Terminal session (observed output)

```
thor@jumphost ~$ ssh max@ststor01
The authenticity of host 'ststor01 (10.244.29.235)' can't be established.
ED25519 key fingerprint is SHA256:yEyN8qvzhNxfcKVE+H05zwQPmQMKCXj4JyGWuOP1HIg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01' (ED25519) to the list of known hosts.
max@ststor01's password:
[max@ststor01 ~]$ sudo su

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

For security reasons, the password you type will not be visible.

[sudo] password for max:
max is not in the sudoers file.
[max@ststor01 ~]$ ls
story-blog
[max@ststor01 ~]$ cd story-blog/
[max@ststor01 story-blog]$ ls
fox-and-grapes.txt  frogs-and-ox.txt  lion-and-mouse.txt
[max@ststor01 story-blog]$ git log
commit eb9965f82e6bfcccb3b4398841baa6a911c06644 (HEAD -> story/fox-and-grapes, origin/story/fox-and-grapes)
Author: Max <max@stratos.xfusioncorp.com>
Date:   Fri May 22 14:25:26 2026 +0000

    Added fox-and-grapes story

commit 0b8d77896ab68035f965de165cf4687fa63ad641 (origin/master, origin/HEAD, master)
Merge: a1f4d2c 6c662c6
Author: sarah <sarah@stratos.xfusioncorp.com>
Date:   Fri May 22 14:25:25 2026 +0000

    Merge branch 'story/frogs-and-ox'

commit a1f4d2cba0eab26e84c1f505bf2c1b8b7875271e
Author: sarah <sarah@stratos.xfusioncorp.com>
Date:   Fri May 22 14:25:25 2026 +0000

    Fix typo in story title

commit 6c662c60338e850e5bea57c4080ef6fba08cc7c5
Author: sarah <sarah@stratos.xfusioncorp.com>
Date:   Fri May 22 14:25:25 2026 +0000

    Completed frogs-and-ox story

commit 6e540041b5cf6f16888530f1b222427278f65c23
Author: sarah <sarah@stratos.xfusioncorp.com>
Date:   Fri May 22 14:25:25 2026 +0000

    Added the lion and mouse story

commit 59a84126397de2299b8ddb478e522382d3636010
Author: sarah <sarah@stratos.xfusioncorp.com>
Date:   Fri May 22 14:25:25 2026 +0000

    Add incomplete frogs-and-ox story
[max@ststor01 story-blog]$ git fetch origin
remote: Enumerating objects: 1, done.
remote: Counting objects: 100% (1/1), done.
remote: Total 1 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
Unpacking objects: 100% (1/1), 313 bytes | 313.00 KiB/s, done.
From http://gitea:3000/sarah/story-blog
   0b8d778..8ed7b96  master     -> origin/master
[max@ststor01 story-blog]$ git log origin/master --oneline -n 5
8ed7b96 (origin/master, origin/HEAD) Merge pull request 'Added fox-and-grapes story' (#1) from story/fox-and-grapes into master
eb9965f (HEAD -> story/fox-and-grapes, origin/story/fox-and-grapes) Added fox-and-grapes story
0b8d778 (master) Merge branch 'story/frogs-and-ox'
a1f4d2c Fix typo in story title
6c662c6 Completed frogs-and-ox story
[max@ststor01 story-blog]$
```

Credentials (used / relevant)

- SSH session initiated from Jump Host user `thor`.
- SSH into storage server: `ssh max@ststor01` — password used during the task: `Max_pass123` (as specified in the task instructions).
- Gitea UI login used to create PR and assign reviewers:
  - `max` / `Max_pass123` (create PR)
  - `tom` / `Tom_pass123` (review & merge)
- Additional infrastructure credentials provided in the environment reference (listed here for completeness):
  - Application Server 1: `stapp01` user `tony` password `Ir0nM@n`
  - Application Server 2: `stapp02` user `steve` password `Am3ric@`
  - Application Server 3: `stapp03` user `banner` password `BigGr33n`
  - LoadBalancer: `stlb01` user `loki` password `Mischi3f`
  - Database Server: `stdb01` user `peter` password `Sp!dy`
  - Storage Server: `ststor01` user `natasha` password `Bl@kW` (note: the task used `max`/`Max_pass123` for the storage-server session)
  - Backup Server: `stbkp01` user `clint` password `H@wk3y3`
  - Mail Server: `stmail01` user `groot` password `Gr00T123`
  - Jump Host: `jump-host` user `thor` password `mjolnir123` (task context shows `thormjolnir123` in one place; use the task-provided `thormjolnir123` when specified)
  - Jenkins Server: `jenkins` user `jenkins` password `j@rv!s`

(Above list repeats provided infrastructure credentials and the specific credentials used during the exercise.)

Theory (concise)

- Never allow direct pushes to `master` for production-ready branches: enable branch protection and require PRs + reviews.
- Use feature/topic branches (e.g., `story/fox-and-grapes`) to isolate work.
- Create a Pull Request (PR) from the feature branch into `master` to request merge and trigger review, CI, and checks.
- Assign a reviewer (Tom) to approve; upon approval an authorized user merges to `master`.
- Verify merge from the terminal using `git fetch origin` and `git log origin/master --oneline -n 5`.

Commands used / verification steps (run on storage server repo)

```bash
# inspect current branch and commits
git log

# fetch remote updates (after PR merged in UI)
git fetch origin

# check recent commits on remote master
git log origin/master --oneline -n 5

# to update local master and view files
git checkout master
git pull origin master
ls -la
cat fox-and-grapes.txt
```

Mistakes observed, root causes and fixes

- Mistake: Attempted to run `sudo su` as `max` but `max` is not in the `sudoers` file.
  - Why: `max` lacks sudo privileges on the storage server.
  - Fix: Use the account with the correct privileges or perform privileged actions via an authorized admin (do not escalate with incorrect credentials). For this task no privileged actions were necessary to create PRs or inspect repo.

- Mistake: Confusing credential sets in documentation (multiple password variants for some hosts).
  - Why: The infrastructure sheet contained slightly different passwords than the task instructions (e.g., `Max_pass123` vs `natashaBl@kW` for storage server). This causes operator confusion.
  - Fix: Standardize credential documentation; clearly state which credential is for which user. When following lab instructions, prefer the task-specific credentials (explicitly called out in steps) and annotate any conflicts.

- Mistake: Expecting feature branch commits to appear on `master` without performing a PR + merge.
  - Why: Feature branch and `master` are separate refs; pushing a feature branch does not modify `master` until merged.
  - Fix: Create a Pull Request, have it reviewed, and merge (or use protected-branch merge policy). Verify with `git fetch origin` and `git log origin/master`.

- Mistake: Not assigning or notifying a reviewer.
  - Why: Workflow requires review for protected branches; forgetting to request/assign a reviewer stalls merges.
  - Fix: In the Gitea UI add the reviewer (Tom) when creating the PR or afterwards in PR settings; notify via comments or team chat.

Notes and best-practices

- Use branch protection rules in Gitea (require at least one approval, require passing CI checks, disallow force pushes to `master`).
- Keep credentials documented and rotated; for real systems use secrets management (Vault, cloud KMS) rather than plaintext notes.
- Use SSH keys for server access where possible, and protect private keys.

Files created for this exercise

- [day29.md](day29.md)

---

If you want, I can now:

- add screenshots/placeholders for the Gitea UI steps,
- create a short checklist for repository branch-protection settings.

Do you want me to update the TODO list to mark the content files as completed and add a final step to commit these notes to the repo?
