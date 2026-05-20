# Day 2: Create User with Account Expiry Date

## Task Description
Securely provision a temporary user account on a specific application server with an automatic expiry date. Managing user access lifecycles is a critical security practice that ensures temporary access for contractors or short-term projects is automatically revoked, preventing unauthorized access after the assignment ends.

**Objective:** Create a user named `javed` on App Server 2 with an expiry date of **2027-01-28**.

---

## Key Requirements

- **Target Server:** App Server 2 (stapp02)
- **Target User:** javed (lowercase)
- **Expiry Date:** 2027-01-28 (YYYY-MM-DD)
- **Purpose:** Temporary access for Nautilus project

---

## Infrastructure Details

**Target Server:**
- **Server Name:** stapp02
- **IP:** 172.16.238.11
- **Hostname:** stapp02.stratos.xfusioncorp.com
- **User:** steve
- **Password:** Am3ric@
- **Purpose:** Nautilus App 2

---

## Solution Steps

### Step 1: Connect to App Server 2
SSH into App Server 2 using the correct credentials for `steve`:

```bash
ssh steve@stapp02
```

**Password:** `Am3ric@`

**Expected Output:**
```bash
thor@jumphost ~$ ssh steve@stapp02
The authenticity of host 'stapp02 (172.16.238.11)' can't be established.
ED25519 key fingerprint is SHA256:+umvZkYkKrAygc1C4iWzyXME3NZSVjw2jALJ7DGB0WU.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
steve@stapp02's password: 
[steve@stapp02 ~]$
```

### Step 2: Create User with Expiry Date
Use the `useradd` command with the `-e` flag to specify the expiration date in YYYY-MM-DD format:

```bash
sudo useradd -e 2027-01-28 javed
```

**Password:** `Am3ric@` (when prompted for sudo password)

**Expected Output:**
```bash
[steve@stapp02 ~]$ sudo useradd -e 2027-01-28 javed

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for steve: 
[steve@stapp02 ~]$
```

### Step 3: Verify Account Expiry Configuration
Verify that the account was created with the correct expiry date using the `chage` command:

```bash
sudo chage -l javed
```

**Expected Output:**
```bash
[steve@stapp02 ~]$ sudo chage -l javed
Last password change                                    : Jan 08, 2026
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : Jan 28, 2027
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
[steve@stapp02 ~]$
```

**Verification Point:** The line `Account expires : Jan 28, 2027` confirms the expiry date is set correctly.

---

## Key Learnings

1. **Account Expiry Management:** Setting expiry dates during user creation is a security best practice that automates access revocation for temporary users.

2. **useradd Command with Expiry:**
   - `-e` or `--expiredate`: Sets the account expiration date in YYYY-MM-DD format
   - The account will be disabled after the specified date
   - Prevents the need for manual account deletion

3. **chage Command for Verification:**
   - `chage -l <username>`: Lists password and account aging information
   - Shows detailed information about password expiry, account expiry, and password change policies
   - Essential for auditing user account lifecycle configurations

4. **Security Best Practices:**
   - Always set expiry dates for temporary accounts (contractors, vendors, short-term projects)
   - Regularly audit account expiry dates using `chage -l`
   - Automates security compliance by preventing stale accounts

---

## Alternative Commands

### Modify Expiry Date for Existing User
If you need to change the expiry date for an existing user:
```bash
sudo usermod -e 2027-01-28 javed
```

### Set Account to Never Expire
To remove an expiry date:
```bash
sudo usermod -e "" javed
```

### Check Expiry Date Quickly
For a quick check without full details:
```bash
sudo chage -l javed | grep "Account expires"
```

---

## Common Mistakes to Avoid

- ❌ Using incorrect date format (must be YYYY-MM-DD)
- ❌ Forgetting to verify the expiry date after creation
- ❌ Using uppercase in username (requirement specifies lowercase)
- ❌ Not setting expiry dates for temporary accounts (security risk)
- ❌ Setting expiry dates in the past (account will be immediately disabled)

---

## Status
✅ **Completed Successfully**

**User:** javed  
**Server:** stapp02  
**Expiry Date:** January 28, 2027  
**Verified:** ✓
