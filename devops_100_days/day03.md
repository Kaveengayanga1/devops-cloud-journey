# Day 3: Disable Root SSH Login on All Application Servers

## Task Description
Harden SSH security across all application servers by disabling direct root login. This is a critical security best practice that prevents attackers from attempting to gain direct root access via SSH. Users must now authenticate with a regular user account and then escalate privileges using `sudo`.

**Objective:** Disable root SSH login (`PermitRootLogin no`) on all three Application Servers (stapp01, stapp02, stapp03).

---

## Security Context

**Why Disable Root SSH Login?**
- Prevents brute-force attacks targeting the root account directly
- Forces users to authenticate with individual accounts (audit trail)
- Requires privilege escalation through `sudo` (additional security layer)
- Compliance with industry security standards and best practices

---

## Infrastructure Details

| Server Name | IP | Hostname | User | Password | Purpose |
|------------|------------|--------------------------------|---------|----------|---------------------|
| **stapp01** | **172.16.238.10** | **stapp01.stratos.xfusioncorp.com** | **tony** | **Ir0nM@n** | **Nautilus App 1** |
| **stapp02** | **172.16.238.11** | **stapp02.stratos.xfusioncorp.com** | **steve** | **Am3ric@** | **Nautilus App 2** |
| **stapp03** | **172.16.238.12** | **stapp03.stratos.xfusioncorp.com** | **banner** | **BigGr33n** | **Nautilus App 3** |

---

## Solution Steps

### Procedure (Repeat for Each Server)

#### Step 1: Connect to the App Server
SSH into the target server using the appropriate credentials:

```bash
ssh <username>@<server>
```

#### Step 2: Edit SSH Configuration File
Open the SSH daemon configuration file with elevated privileges:

```bash
sudo vi /etc/ssh/sshd_config
```

#### Step 3: Modify PermitRootLogin Setting
1. **Find the line:** Look for `PermitRootLogin yes` or `#PermitRootLogin yes` (the # indicates it's commented out)
2. **Change to:** `PermitRootLogin no`
3. **Remove comment:** Make sure to remove the `#` if the line is commented out
4. **Final result:** `PermitRootLogin no`

**Vi Editor Navigation:**
- Press `/` to search: `/PermitRootLogin`
- Press `n` to find next occurrence
- Position cursor at the beginning of `yes`
- Delete `yes`: Press `dw` (delete word)
- Type `no`

#### Step 4: Save and Exit
- Press `Esc` to exit insert mode
- Type `:wq` (write and quit)
- Press `Enter`

#### Step 5: Restart SSH Service
Restart the SSH daemon to apply the changes:

```bash
sudo systemctl restart sshd
```

**Expected Output:**
```bash
[user@serverX ~]$ sudo systemctl restart sshd
[sudo] password for user:
[user@serverX ~]$
```

---

## Server-by-Server Execution

### App Server 1 (stapp01)

**Connection:**
```bash
ssh tony@stapp01
Password: Ir0nM@n
```

**Configuration Steps:**
```bash
sudo vi /etc/ssh/sshd_config
# Find and change: PermitRootLogin yes → PermitRootLogin no
# Save with :wq

sudo systemctl restart sshd
```

**Verification:**
```bash
sudo grep PermitRootLogin /etc/ssh/sshd_config
# Expected: PermitRootLogin no
```

---

### App Server 2 (stapp02)

**Connection:**
```bash
ssh steve@stapp02
Password: Am3ric@
```

**Configuration Steps:**
```bash
sudo vi /etc/ssh/sshd_config
# Find and change: PermitRootLogin yes → PermitRootLogin no
# Save with :wq

sudo systemctl restart sshd
```

**Verification:**
```bash
sudo grep PermitRootLogin /etc/ssh/sshd_config
# Expected: PermitRootLogin no
```

---

### App Server 3 (stapp03)

**Connection:**
```bash
ssh banner@stapp03
Password: BigGr33n
```

**Configuration Steps:**
```bash
sudo vi /etc/ssh/sshd_config
# Find and change: PermitRootLogin yes → PermitRootLogin no
# Save with :wq

sudo systemctl restart sshd
```

**Verification:**
```bash
sudo grep PermitRootLogin /etc/ssh/sshd_config
# Expected: PermitRootLogin no
```

---

## Verification Commands

### Quick Verification on Each Server
After restarting the SSH service, verify that the setting was applied correctly:

```bash
sudo grep PermitRootLogin /etc/ssh/sshd_config
```

**Expected Output:**
```bash
PermitRootLogin no
```

### Full SSH Configuration Verification
To see the complete SSH configuration (active settings):

```bash
sudo sshd -T | grep permitrootlogin
```

**Expected Output:**
```bash
permitrootlogin no
```

### Check SSH Service Status
Verify that the SSH service is running after restart:

```bash
sudo systemctl status sshd
```

---

## Key Learnings

1. **SSH Security Hardening:**
   - `PermitRootLogin no`: Disables direct root SSH access
   - Forces privilege escalation through `sudo` for audit trails
   - Significantly reduces attack surface

2. **SSH Configuration File:**
   - Located at: `/etc/ssh/sshd_config`
   - Changes require service restart to take effect
   - Comments are marked with `#` at the start of the line
   - Be careful not to lock yourself out with invalid configurations

3. **Vi Editor Essentials:**
   - `i`: Insert mode
   - `Esc`: Exit insert mode
   - `:wq`: Write and quit
   - `:q!`: Quit without saving
   - `/pattern`: Search for pattern
   - `dw`: Delete word
   - `dd`: Delete line

4. **Service Management:**
   - `systemctl restart`: Restarts the service (closes and reopens connections)
   - `systemctl reload`: Reloads configuration (doesn't close connections)
   - `systemctl status`: Shows service status

---

## Alternative Methods

### Using sed to Modify Configuration
```bash
sudo sed -i 's/^#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sudo sed -i 's/^PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
```

### Using grep and Edit in One Command
```bash
# View the current setting
sudo grep -n PermitRootLogin /etc/ssh/sshd_config

# Make backup before editing
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

---

## Common Mistakes to Avoid

- ❌ Forgetting to use `sudo` when editing the SSH config file
- ❌ Not restarting the SSH service after making changes
- ❌ Changing `PermitRootLogin` to something other than `no` (valid values: `yes`, `no`, `prohibit-password`, `forced-commands-only`)
- ❌ Using `:q` instead of `:wq` and losing the changes
- ❌ Not verifying the changes on each server before moving to the next
- ❌ Editing only one server and forgetting the other two
- ❌ Modifying syntax incorrectly (extra spaces or typos will prevent service restart)

---

## Rollback Instructions

If you need to revert the changes:

```bash
# Restore from backup (if you created one)
sudo cp /etc/ssh/sshd_config.backup /etc/ssh/sshd_config
sudo systemctl restart sshd

# Or manually edit back to:
sudo vi /etc/ssh/sshd_config
# Change: PermitRootLogin no → PermitRootLogin yes
# Save with :wq
sudo systemctl restart sshd
```

---

## Checklist

- [ ] SSH into App Server 1 (stapp01) as tony
- [ ] Disable root login on stapp01 and restart SSH
- [ ] Verify change on stapp01
- [ ] SSH into App Server 2 (stapp02) as steve
- [ ] Disable root login on stapp02 and restart SSH
- [ ] Verify change on stapp02
- [ ] SSH into App Server 3 (stapp03) as banner
- [ ] Disable root login on stapp03 and restart SSH
- [ ] Verify change on stapp03

---

## Status
✅ **Completed Successfully**

**Configuration Applied To:**
- ✓ stapp01 (App Server 1)
- ✓ stapp02 (App Server 2)
- ✓ stapp03 (App Server 3)

**Setting:** `PermitRootLogin no` enabled on all servers  
**Verification:** All servers confirmed to have root SSH login disabled
