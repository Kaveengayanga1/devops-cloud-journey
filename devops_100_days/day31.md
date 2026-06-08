Date: 2026-05-24
Topic: Git Stash Restore, Commit, and Push Workflow
Environment: KodeKloud Stratos DC - Storage Server (`ststor01`)

Summary

- This task restored the stashed changes identified as `stash@{1}` from the Git repository at `/usr/src/kodekloudrepos/games`.
- The changes were applied, committed with an appropriate message, and pushed to the remote `origin` on `master`.
- The visible result was the addition of `welcome.txt` to the repository history.

Task Statement

The Nautilus application development team was working on the Git repository `/usr/src/kodekloudrepos/games` on the Storage Server in Stratos DC. One of the developers had stashed in-progress changes and later needed to restore the second most recent stash entry.

Required outcome:

- Locate the stashed changes in `/usr/src/kodekloudrepos/games`.
- Restore the stash with identifier `stash@{1}`.
- Commit the restored changes.
- Push the commit to the remote origin.

Observed Terminal Session

```bash
thor@jumphost ~$ ssh natasha@ststor01
The authenticity of host 'ststor01 (10.244.81.25)' can't be established.
ED25519 key fingerprint is SHA256:yEyN8qvzhNxfcKVE+H05zwQPmQMKCXj4JyGWuOP1HIg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01' (ED25519) to the list of known hosts.
natasha@ststor01's password:
[natasha@ststor01 ~]$ cd /usr/src/kodekloudrepos/games
[natasha@ststor01 games]$ git stash list
fatal: detected dubious ownership in repository at '/usr/src/kodekloudrepos/games'
To add an exception for this directory, call:

        git config --global --add safe.directory /usr/src/kodekloudrepos/games
[natasha@ststor01 games]$ sudo su
[root@ststor01 games]# git stash list
stash@{0}: WIP on master: 8a3f596 initial commit
stash@{1}: WIP on master: 8a3f596 initial commit
[root@ststor01 games]# git loge --online
git: 'loge' is not a git command. See 'git --help'.

The most similar commands are
        clone
        log
[root@ststor01 games]# git log --online
fatal: unrecognized argument: --online
[root@ststor01 games]# git log --oneline
8a3f596 (HEAD -> master, origin/master) initial commit
[root@ststor01 games]# git stash apply stash@{1}
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   welcome.txt

[root@ststor01 games]# ls
info.txt  welcome.txt
[root@ststor01 games]# git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   welcome.txt

[root@ststor01 games]# git add .
[root@ststor01 games]# git commit -m "Restore and apply in-progress changes from stash@{1}"
[master 61de047] Restore and apply in-progress changes from stash@{1}
 1 file changed, 1 insertion(+)
 create mode 100644 welcome.txt
[root@ststor01 games]# git push origin master
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 16 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 337 bytes | 337.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To /opt/games.git
   8a3f596..61de047  master -> master
[root@ststor01 games]#
```

Infrastructure Details and Credentials

| Server               | Hostname    | User      | Password         | Purpose                                     |
| -------------------- | ----------- | --------- | ---------------- | ------------------------------------------- |
| Application Server 1 | `stapp01`   | `tony`    | `Ir0nM@n`        | Hosts Nautilus Application 1                |
| Application Server 2 | `stapp02`   | `steve`   | `Am3ric@`        | Hosts Nautilus Application 2                |
| Application Server 3 | `stapp03`   | `banner`  | `BigGr33n`       | Hosts Nautilus Application 3                |
| LoadBalancer Server  | `stlb01`    | `loki`    | `Mischi3f`       | Distributes traffic for Nautilus HTTP       |
| Database Server      | `stdb01`    | `peter`   | `Sp!dy`          | Hosts Nautilus Database                     |
| Storage Server       | `ststor01`  | `natasha` | `Bl@kW`          | Stores data for Nautilus Servers            |
| Backup Server        | `stbkp01`   | `clint`   | `H@wk3y3`        | Manages backups for Nautilus Servers        |
| Mail Server          | `stmail01`  | `groot`   | `Gr00T123`       | Manages email services for Nautilus Servers |
| Jump Host Server     | `jump-host` | `thor`    | `thormjolnir123` | Provides secure access to Stork DC          |
| Jenkins Server       | `jenkins`   | `jenkins` | `j@rv!s`         | Runs Jenkins for CI/CD pipeline             |

Task-specific SSH and host details

- Storage server IP observed in the session: `10.244.81.25`
- Host key type: `ED25519`
- Fingerprint: `SHA256:yEyN8qvzhNxfcKVE+H05zwQPmQMKCXj4JyGWuOP1HIg`
- SSH login used in the task: `natasha@ststor01`

Required Theory

1. Git stash concept

- `git stash` temporarily stores uncommitted changes so that the working tree can be cleaned without losing work.
- It is typically used when a developer must switch context quickly and return later.

2. Stash ordering

- `git stash list` displays all stash entries.
- `stash@{0}` is the most recent stash.
- `stash@{1}` is the second most recent stash.
- Git manages stashes as a stack, so the newest entry is placed on top.

3. Apply versus pop

- `git stash apply` restores changes but keeps the stash entry in the list.
- `git stash pop` restores changes and removes the stash entry if the operation succeeds.
- For a specific stash entry such as `stash@{1}`, `apply` is the safer choice because it avoids losing the reference if follow-up validation is needed.

4. Repository safety checks

- Newer Git versions can block operations in repositories with ownership mismatches and report a dubious ownership error.
- In such cases, the repository can be marked as safe after verifying that the location is trusted.

5. Commit and push workflow

- After restoration, changes must be staged with `git add`.
- A clear commit message documents why the change exists.
- `git push origin master` publishes the commit to the remote repository.

Mistakes, Root Causes, and Fixes

1. Mistake: Git refused to list stashes because of dubious ownership.

- Why it happened: The repository owner and the user executing the command did not match, so Git treated the repository as potentially unsafe.
- Fix: Switch to an account with proper privilege context and, if required by policy, mark the repository as safe using the documented Git setting.

2. Mistake: The command `git loge --online` was typed incorrectly.

- Why it happened: The command name and option were both mistyped.
- Fix: Use the correct command `git log --oneline`.

3. Mistake: `git log --online` was attempted next and failed.

- Why it happened: The option name was still incorrect; Git expects `--oneline`.
- Fix: Re-run the command with the correct spelling.

4. Mistake: Assuming the stash restore alone would complete the task.

- Why it happened: Restoring the stash only places the changes into the working tree or index; it does not create a permanent history entry.
- Fix: Stage the restored file, commit it, and then push the commit to `origin`.

Professional Execution Guide

1. Connect to the storage server from the jump host.

```bash
ssh natasha@ststor01
```

2. Move to the target repository.

```bash
cd /usr/src/kodekloudrepos/games
```

3. Inspect the stash list and identify the required entry.

```bash
git stash list
```

4. Restore the required stash entry.

```bash
git stash apply stash@{1}
```

5. Verify the restored files and repository state.

```bash
git status
ls
```

6. Stage and commit the restored changes.

```bash
git add .
git commit -m "Restore and apply in-progress changes from stash@{1}"
```

7. Push the commit to the remote repository.

```bash
git push origin master
```

Operational Notes

- When multiple stash entries exist, always confirm the index before applying or popping a stash.
- Prefer `apply` over `pop` when the stash must remain available for further review.
- Keep credentials documented carefully in lab notes, but treat them as sensitive in real environments.

Files created for this exercise

- [day31.md](day31.md)
