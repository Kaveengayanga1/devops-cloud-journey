Date: 2026-05-23
Topic: Git History Rewrite and Recovery - Reset Branch to `add data.txt file` Commit
Environment: KodeKloud Stratos DC - Storage Server (`ststor01`)

Summary

- Objective: Clean the Git repository history in `/usr/src/kodekloudrepos/blog` so only two commits remain:
  - `initial commit`
  - `add data.txt file`
- Action performed: Used `git reset --hard 3d695ca` and force-pushed to remote.
- Validation: `git log --oneline` showed only the required two commits.
- Recovery test: Demonstrated how to move back to `Test Commit10` using `git reflog`, then reset again to target commit.

Task Statement (English)

The Nautilus application development team was working on a Git repository at `/usr/src/kodekloudrepos/blog` on the Storage Server in Stratos DC. Test commits were pushed and now the team wants to clean both commit history and working tree, pointing `HEAD` and the branch back to commit message `add data.txt file`.

Required final state:

- Only two commits in history:
  - `initial commit`
  - `add data.txt file`
- Push the cleaned history to remote.

Complete Terminal Session (Observed Output)

```bash
thor@jumphost ~$ ssh natasha@ststor01
The authenticity of host 'ststor01 (10.244.97.235)' can't be established.
ED25519 key fingerprint is SHA256:yEyN8qvzhNxfcKVE+H05zwQPmQMKCXj4JyGWuOP1HIg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01' (ED25519) to the list of known hosts.
natasha@ststor01's password:
[natasha@ststor01 ~]$ cd /usr/src/kodekloudrepos/blog
[natasha@ststor01 blog]$ ls
blog.txt  info.txt
[natasha@ststor01 blog]$ sudo su
[root@ststor01 blog]# git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
[root@ststor01 blog]# git log --oneline
11121fc (HEAD -> master, origin/master) Test Commit10
cf3cace Test Commit9
5306a86 Test Commit8
435855c Test Commit7
862d653 Test Commit6
e23d29d Test Commit5
3cade69 Test Commit4
fb98976 Test Commit3
85e32ba Test Commit2
a5ffc91 Test Commit1
3d695ca add data.txt file
2ed3714 initial commit
[root@ststor01 blog]# git reset --hard 3d695ca
HEAD is now at 3d695ca add data.txt file
[root@ststor01 blog]# git log --oneline
3d695ca (HEAD -> master) add data.txt file
2ed3714 initial commit
[root@ststor01 blog]# git branch
* master
[root@ststor01 blog]# git push origin master --force
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To /opt/blog.git
 + 11121fc...3d695ca master -> master (forced update)
[root@ststor01 blog]# ls
blog.txt
[root@ststor01 blog]# git reflog
3d695ca (HEAD -> master, origin/master) HEAD@{0}: reset: moving to 3d695ca
11121fc HEAD@{1}: commit: Test Commit10
cf3cace HEAD@{2}: commit: Test Commit9
5306a86 HEAD@{3}: commit: Test Commit8
435855c HEAD@{4}: commit: Test Commit7
862d653 HEAD@{5}: commit: Test Commit6
e23d29d HEAD@{6}: commit: Test Commit5
3cade69 HEAD@{7}: commit: Test Commit4
fb98976 HEAD@{8}: commit: Test Commit3
85e32ba HEAD@{9}: commit: Test Commit2
a5ffc91 HEAD@{10}: commit: Test Commit1
3d695ca (HEAD -> master, origin/master) HEAD@{11}: commit: add data.txt file
2ed3714 HEAD@{12}: commit (initial): initial commit
[root@ststor01 blog]# git reset --hard 11121fc
HEAD is now at 11121fc Test Commit10
[root@ststor01 blog]# git reflog
11121fc (HEAD -> master) HEAD@{0}: reset: moving to 11121fc
3d695ca (origin/master) HEAD@{1}: reset: moving to 3d695ca
11121fc (HEAD -> master) HEAD@{2}: commit: Test Commit10
cf3cace HEAD@{3}: commit: Test Commit9
5306a86 HEAD@{4}: commit: Test Commit8
435855c HEAD@{5}: commit: Test Commit7
862d653 HEAD@{6}: commit: Test Commit6
e23d29d HEAD@{7}: commit: Test Commit5
3cade69 HEAD@{8}: commit: Test Commit4
fb98976 HEAD@{9}: commit: Test Commit3
85e32ba HEAD@{10}: commit: Test Commit2
a5ffc91 HEAD@{11}: commit: Test Commit1
3d695ca (origin/master) HEAD@{12}: commit: add data.txt file
2ed3714 HEAD@{13}: commit (initial): initial commit
[root@ststor01 blog]# git log --oneline
11121fc (HEAD -> master) Test Commit10
cf3cace Test Commit9
5306a86 Test Commit8
435855c Test Commit7
862d653 Test Commit6
e23d29d Test Commit5
3cade69 Test Commit4
fb98976 Test Commit3
85e32ba Test Commit2
a5ffc91 Test Commit1
3d695ca (origin/master) add data.txt file
2ed3714 initial commit
[root@ststor01 blog]# git reset --hard 3d695ca
HEAD is now at 3d695ca add data.txt file
[root@ststor01 blog]# git reflog
3d695ca (HEAD -> master, origin/master) HEAD@{0}: reset: moving to 3d695ca
11121fc HEAD@{1}: reset: moving to 11121fc
3d695ca (HEAD -> master, origin/master) HEAD@{2}: reset: moving to 3d695ca
11121fc HEAD@{3}: commit: Test Commit10
cf3cace HEAD@{4}: commit: Test Commit9
5306a86 HEAD@{5}: commit: Test Commit8
435855c HEAD@{6}: commit: Test Commit7
862d653 HEAD@{7}: commit: Test Commit6
e23d29d HEAD@{8}: commit: Test Commit5
3cade69 HEAD@{9}: commit: Test Commit4
fb98976 HEAD@{10}: commit: Test Commit3
85e32ba HEAD@{11}: commit: Test Commit2
a5ffc91 HEAD@{12}: commit: Test Commit1
3d695ca (HEAD -> master, origin/master) HEAD@{13}: commit: add data.txt file
2ed3714 HEAD@{14}: commit (initial): initial commit
[root@ststor01 blog]# git log --oneline
3d695ca (HEAD -> master, origin/master) add data.txt file
2ed3714 initial commit
[root@ststor01 blog]# git push origin master --force
Everything up-to-date
[root@ststor01 blog]#
```

Infrastructure and Credentials (Provided)

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

Task-specific host fingerprint and endpoint details

- Storage server IP observed: `10.244.97.235`
- Host key type: `ED25519`
- Fingerprint: `SHA256:yEyN8qvzhNxfcKVE+H05zwQPmQMKCXj4JyGWuOP1HIg`

What Happened: Mistakes, Root Causes, and Fixes

1. Mistake: History was rewritten with `git reset --hard` and force-push without preserving a safety tag/backup branch first.

- Why it happened: The goal was to clean the repo quickly, and in lab workflows engineers often skip a backup step.
- Impact: Commits `Test Commit1` to `Test Commit10` disappeared from visible branch history.
- Fix applied: Used `git reflog` to locate lost commit (`11121fc`) and successfully restored with:

```bash
git reset --hard 11121fc
```

Then reset again to required task target:

```bash
git reset --hard 3d695ca
```

2. Mistake: Force push can overwrite remote history for all collaborators.

- Why it happened: Rewriting history requires remote alignment; normal `git push` is rejected after reset.
- Impact: Team members can be confused if their local branches still reference removed commits.
- Fix applied: Explicit force push was used intentionally:

```bash
git push origin master --force
```

Best-practice safer alternative:

```bash
git push origin master --force-with-lease
```

3. Mistake: Possible confusion after recovering to `Test Commit10` and then resetting again.

- Why it happened: Recovery validation and final target enforcement were both tested in the same session.
- Impact: Operator may think state is unstable.
- Fix applied: Final validation repeated with `git log --oneline`; end state confirmed to exactly two commits.

Required and Additional Theory (English)

1. Git object model basics

- `HEAD` points to the current commit of the checked-out branch.
- Branch (`master`) is just a movable pointer to a commit.
- `git reset` moves pointers; `--hard` also rewrites index and working tree.

2. Reset types and practical applications

- `git reset --soft <commit>`
  - Moves only `HEAD`.
  - Use case: squash/rewrite recent commit messages while keeping staged changes.

- `git reset --mixed <commit>` (default)
  - Moves `HEAD` and updates staging area.
  - Use case: unstage files but keep code changes in working tree.

- `git reset --hard <commit>`
  - Moves `HEAD`, updates staging area, and resets working tree.
  - Use case: discard local changes and return exactly to a known commit.

3. Reflog recovery concept

- `git reflog` tracks local `HEAD` movements even after resets.
- Lost commits are usually recoverable as long as garbage collection has not cleaned them.

4. Force push mechanics

- After rewriting local history, remote still has old commit graph.
- `git push --force` replaces remote ref with local ref.
- In shared repos, prefer `--force-with-lease` to avoid overwriting others' unseen updates.

5. Operational risk controls for DevOps/Cloud teams

- Create rollback markers before rewrite:

```bash
git tag backup-before-rewrite-$(date +%F-%H%M%S)
# or
git branch backup/master-before-reset
```

- Announce freeze window in team channel before force push.
- Protect critical branches and allow force push only for release admins.
- Keep audit notes of old/new commit IDs in incident/task documentation.

Professional Execution Guide (English)

1. Connect through jump host to storage server

```bash
ssh natasha@ststor01
```

2. Go to repository and verify current state

```bash
cd /usr/src/kodekloudrepos/blog
git status
git log --oneline
```

3. Identify target commit (`add data.txt file` = `3d695ca`)

4. Reset local branch and working tree to target commit

```bash
git reset --hard 3d695ca
```

5. Confirm commit history has exactly two commits

```bash
git log --oneline
```

Expected:

- `3d695ca add data.txt file`
- `2ed3714 initial commit`

6. Confirm branch name

```bash
git branch
```

7. Push rewritten history to remote

```bash
git push origin master --force
```

8. If accidental rollback is required later, recover using reflog

```bash
git reflog
git reset --hard 11121fc
# optional if remote must match recovered state
git push origin master --force
```

Final Verification Checklist

- [x] `HEAD` at `3d695ca`
- [x] Branch is `master`
- [x] `git log --oneline` shows only two commits
- [x] Remote updated (`master -> master (forced update)` seen earlier)
- [x] Repository working tree clean

Short Answer: "Can I now go back to Test Commit10?"

Yes. Use `git reflog` to find commit `11121fc`, then:

```bash
git reset --hard 11121fc
```

If remote must also reflect that state:

```bash
git push origin master --force
```
