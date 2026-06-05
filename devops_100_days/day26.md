# Day 26: Updating Git Remotes and Workflow

**Date:** May 13, 2026
**Topic:** Git Remote Management and Content Update
**Server:** Storage Server (`ststor01`)

## Task Overview

The xFusionCorp development team needed to update the Git workflow for a project maintained under `/opt/media.git` (cloned at `/usr/src/kodekloudrepos/media`). Due to changes on the Git server hosted on the Storage server, a new remote needed to be added, content updated, and changes pushed to the new destination.

## Infrastructure Details

| Server Name | Hostname | User | Password | Purpose |
| | | | | |
| Jump Host | jump-host | thor | mjolnir123 | Secure Access |
| Storage Server | ststor01 | natasha | Bl@kW | Data Storage / Git Host |
| App Server 1 | stapp01 | tony | Ir0nM@n | App Hosting |
| App Server 2 | stapp02 | steve | Am3ric@ | App Hosting |
| App Server 3 | stapp03 | banner | BigGr33n | App Hosting |

## Theoretical Concepts

1.  **Git Remote:** A connection to a version of your repository hosted on the internet or a network server.
2.  **Git Remotes (Multiple):** A single local repository can track multiple remote repositories (e.g., `origin`, `dev_media`).
3.  **Git Push:** Uploading local repository content to a remote repository.

## Mistakes & Solutions

- **Mistake:** Attempting to push to `origin` when the task specified pushing to a new remote `dev_media`.
- **Why it happened:** Habitual use of `git push origin master`.
- **Fix:** Explicitly defining the remote name in the push command: `git push dev_media master`.
- **Mistake:** Permission denied when copying files or adding remotes.
- **Why it happened:** Working as a standard user (`natasha`) in system directories (`/usr/src`).
- **Fix:** Switched to root user using `sudo su`.

## Implementation Log

```bash
# Connect to Storage Server
ssh natasha@ststor01 # Password: Bl@kW
sudo su

# Navigate to the repository
cd /usr/src/kodekloudrepos/media

# Add new remote
git remote add dev_media /opt/xfusioncorp_media.git

# Verify remotes
git remote -v

# Add new content
cp /tmp/index.html .
git add index.html
git commit -m "Add index.html to the repository"

# Push to the new remote
git push dev_media master
```

## Terminal Output

```text
[root@ststor01 media]# git commit -m "Add index.html to the repository"
[master 21e3567] Add index.html to the repository
 1 file changed, 10 insertions(+)
 create mode 100644 index.html
[root@ststor01 media]# git push dev_media master
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Writing objects: 100% (6/6), 598 bytes | 598.00 KiB/s, done.
To /opt/xfusioncorp_media.git
 * [new branch]      master -> master
```
