# Day 23: Fork an Existing Git Repository in Gitea

**Date:** May 6, 2026  
**Task:** Log in to Gitea as `jon` and fork `sarah/story-blog` under the `jon` account

## Infrastructure Details

| Server Name | IP Address | Hostname | User | Password | Purpose |
|---|---|---|---|---|---|
| Application Server 1 | Dynamic | stapp01 | tony | Ir0nM@n | Hosts Nautilus Application 1 |
| Application Server 2 | Dynamic | stapp02 | steve | Am3ric@ | Hosts Nautilus Application 2 |
| Application Server 3 | Dynamic | stapp03 | banner | BigGr33n | Hosts Nautilus Application 3 |
| LoadBalancer Server | Dynamic | stlb01 | loki | Mischi3f | Distributes traffic for Nautilus HTTP |
| Database Server | Dynamic | stdb01 | peter | Sp!dy | Hosts Nautilus Database |
| Storage Server | Dynamic | ststor01 | natasha | Bl@kW | Stores data for Nautilus Servers |
| Backup Server | Dynamic | stbkp01 | clint | H@wk3y3 | Manages backups for Nautilus Servers |
| Mail Server | Dynamic | stmail01 | groot | Gr00T123 | Manages email services for Nautilus Servers |
| Jump Host Server | Dynamic | jumphost / jump-host | thor | mjolnir123 | Provides secure access to Stork DC |
| Jenkins Server | Dynamic | jenkins | jenkins | j@rv!s | Runs Jenkins for CI/CD pipeline |

**Note:** These credentials are preserved here because the original lab environment no longer exists and the values may be needed for documentation or review.

## Task Requirements

- Open the Gitea UI from the top bar.
- Log in with username `jon` and password `Jon_pass123`.
- Locate `sarah/story-blog`.
- Fork the repository under the `jon` account.
- Capture screenshots of the completed web UI action for review.
- Optionally record the workflow with a screen recorder such as loom.com.

## Theory and Concepts

### 1. Version Control System
A version control system tracks changes to source code over time. It helps teams collaborate, review changes, and restore earlier versions when needed.

### 2. Git
Git is a distributed version control system. Each clone can contain the full repository history, which makes branching, merging, and offline work efficient.

### 3. Gitea
Gitea is a lightweight, self-hosted Git service similar to GitHub or GitLab. It provides repository hosting, user management, forks, pull requests, and web-based code browsing.

### 4. Forking
Forking creates a personal copy of someone else’s repository under your own account. This lets you work independently without changing the original project immediately.

### 5. Pull Request Flow
After changes are made in a fork, a pull request can be opened to propose those changes back to the original repository. This is the normal review path in collaborative Git workflows.

### 6. Web UI Validation
When a task is performed through a browser interface, screenshots are often required as proof of completion. This is especially important in lab and review environments where command output alone is not enough.

### 7. Credential Handling in Labs
Training labs often provide temporary usernames and passwords. These should be documented carefully for the exercise, but in production they should be stored securely and rotated regularly.

### 8. Repository Ownership
The fork should be created under the correct account, `jon`, because ownership determines where future commits, branches, and pull requests will originate.

## Professional DevOps / Cloud Execution Steps

### Step 1: Open the Gitea UI
Click the Gitea UI button on the top bar to open the Git service page.

### Step 2: Authenticate
Use the provided credentials:

- Username: `jon`
- Password: `Jon_pass123`

Professional practice: confirm that you are logging into the intended environment before entering any credentials, especially in shared lab systems.

### Step 3: Find the Source Repository
Use the search bar or Explore page in Gitea to locate `sarah/story-blog`.

### Step 4: Fork the Repository
Open the repository page and select the Fork option. Ensure the target owner is `jon`.

If the repository name is offered for editing, keep it aligned with the original unless the task explicitly requests a rename.

### Step 5: Verify the Fork
After forking, confirm that the repository appears under the `jon` namespace, typically as `jon/story-blog`.

### Step 6: Capture Evidence
Take a screenshot of the fork result page. If required by the review process, add a short screen recording showing the login and fork flow.

### Step 7: Optional Clone Check
If you want to confirm the fork URL from the command line, use the Git clone format shown below.

```bash
git clone http://<gitea-server-ip>/jon/story-blog.git
```

## Mistakes Made, Why They Happened, and How They Were Fixed

### 1. Logging in With the Wrong Account Would Break Ownership
If the repository is forked while signed in as the wrong user, the fork will be created under the wrong namespace.

Why it happened:
- Web UI tasks depend on the active session identity.
- It is easy to assume the account is correct without confirming the logged-in user.

How it was fixed:
- The login credentials were checked first, and the session was verified as `jon` before forking.

### 2. Searching the Wrong Repository Name Would Lead to the Wrong Target
Using an incorrect search term could bring up a different project or no result at all.

Why it happened:
- Repository names in lab environments are often similar.
- A small typo in organization or project name can point to the wrong page.

How it was fixed:
- The exact repository path `sarah/story-blog` was used when locating the source.

### 3. Forgetting the Screenshot Would Leave the Task Incomplete
The fork action in a browser is not always enough on its own for review.

Why it happened:
- UI tasks can feel complete once the visible action succeeds.
- The lab also requires proof for verification.

How it was fixed:
- A screenshot was planned immediately after the fork confirmation page, and a screen recording was recommended for auditability.

### 4. Treating Lab Credentials Like Production Credentials
The task includes shared lab credentials that should not be reused outside the exercise.

Why it happened:
- Lab documentation often lists many usernames and passwords together.
- It is easy to copy them into notes without considering their temporary nature.

How it was fixed:
- The credentials were documented only for the lab context and clearly identified as inactive historical values.

## Professional DevOps Notes

- Always confirm the active user before performing any ownership-sensitive web action.
- Use exact repository paths when searching for source projects.
- Keep UI evidence such as screenshots or recordings when the task depends on browser changes.
- Treat lab credentials as temporary training data, not reusable access information.

## Validation Checklist

- Gitea UI was opened from the top bar.
- Login was performed with `jon` / `Jon_pass123`.
- `sarah/story-blog` was located.
- The repository was forked under `jon`.
- The result was ready for screenshot-based review.

## Final Outcome

The `sarah/story-blog` repository was forked in Gitea under the `jon` account. The task is complete once the fork is visible as `jon/story-blog` and the screenshot evidence is captured for review.