# Day 18: PostgreSQL Database Setup

## Task Overview
The Nautilus application development team is deploying a new application in the Stratos DC. This application requires a PostgreSQL database. The goal is to set up the database and a dedicated user with specific privileges on the Nautilus database server (`stdb01`).

## Infrastructure Details
| Server Name | Hostname | User | Password | Purpose |
|-------------|----------|------|----------|---------|
| Jump Host | `jump-host` | `thor` | `mjolnir123` | Secure Access |
| Database Server | `stdb01` | `peter` | `Sp!dy` | PostgreSQL Host |

## Requirements
1. Create a database user `kodekloud_joy` with password `LQfKeWWxWD`.
2. Create a database `kodekloud_db8`.
3. Grant full permissions (`ALL PRIVILEGES`) to `kodekloud_joy` on `kodekloud_db8`.
4. **Do not** restart the PostgreSQL service.

## Implementation Steps

### 1. Access the Database Server
Login to the Jump Host and then SSH into the database server:
```bash
ssh peter@stdb01
# Password: Sp!dy
```

### 2. Enter PostgreSQL Shell
Access the PostgreSQL CLI using the administrative user `postgres`:
```bash
sudo -u postgres psql
# Password for peter: Sp!dy
```

### 3. Create User and Database
Run the following SQL commands:
```sql
-- Create the user
CREATE USER kodekloud_joy WITH PASSWORD 'LQfKeWWxWD';

-- Create the database
CREATE DATABASE kodekloud_db8;

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db8 TO kodekloud_joy;
```

### 4. Verification
Check the database and user lists:
```sql
\l
\du
\q
```

## Mistakes and Troubleshooting

### 1. Directory Permission Warning
**Issue:** When running `sudo -u postgres psql`, the error `could not change directory to "/home/peter": Permission denied` appeared.
**Cause:** The `postgres` user does not have permission to access Peter's home directory. Since the command was executed while the current working directory was `/home/peter`, PostgreSQL tried to reference it.
**Fix:** This is a non-fatal warning. The command still executes successfully and enters the `psql` shell. Alternatively, moving to a public directory like `/tmp` before running the command would avoid this message.

### 2. Privilege Scope
**Mistake:** Forgetting to grant privileges after creating the database.
**Why:** Many beginners assume creating a user and a database automatically links them. In PostgreSQL, access must be explicitly granted.
**Fix:** Always run the `GRANT` command to ensure the application user can actually interact with the data.

## Full Terminal Output
```text
thor@jump-host ~$ ssh peter@stdb01
peter@stdb01's password: 
[peter@stdb01 ~]$ sudo -u postgres psql

[sudo] password for peter: 
could not change directory to "/home/peter": Permission denied
psql (13.23)
Type "help" for help.

postgres=# CREATE USER kodekloud_joy WITH PASSWORD 'LQfKeWWxWD';
CREATE ROLE
postgres=# CREATE DATABASE kodekloud_db8;
CREATE DATABASE
postgres=# GRANT ALL PRIVILEGES ON DATABASE kodekloud_db8 TO kodekloud_joy;
GRANT
postgres=# \l
postgres=# \du
postgres=# \q
```
