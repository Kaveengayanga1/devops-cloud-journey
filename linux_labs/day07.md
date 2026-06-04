# Linux User Data Archiving

## Scenario
The jump host server hosts a directory named `/data`, serving as a repository for various developers' non-confidential data. Developer `siva` has requested a copy of their data stored in `/data/siva`. The System Admin team has requested to archive this data.

### Tasks
a. Create a compressed archive named `siva.tar.gz` of the `/data/siva` directory.
b. Transfer the archive to the `/home` directory on the Jump Host Server.

### Infrastructure Details
* **Server**: Jump Host Server (stapp01/stapp02 etc. irrelevant here, executed on jump host)
* **User**: `thor`
* **Password**: `mjolnir123` (Reference from previous lab)
* **Target User Directory**: `siva`

## Theory & Key Concepts

* **Archiving vs Compression**:
  * *Archiving* combines multiple files and directories into a single file. (`tar`)
  * *Compression* reduces the size of the file. (`gzip`, `bzip2`)
* **tar Command Flags**:
  * `-c`: Create a new archive.
  * `-z`: Compress the archive using gzip.
  * `-v`: Verbose output (shows files being processed).
  * `-f`: Specify the filename of the archive.
  * `-C`: Change directory before performing the operation (avoids storing absolute paths).

## Practical Steps (DevOps Approach)

1. **Log in to the Jump Host**
   ```bash
   ssh thor@jump-host
   # Password: mjolnir123
   ```

2. **Navigate and verify the directory**
   ```bash
   cd /data/
   ls
   cd siva/
   ls  # Should show nautilus1.txt, nautilus2.txt, nautilus3.txt
   cd ..
   ```

3. **Create the compressed archive**
   *Using the `-C` flag ensures we don't capture the absolute `/data` path within the tarball.*
   ```bash
   sudo tar -czvf siva.tar.gz -C /data siva
   ```

4. **Move the archive to `/home`**
   ```bash
   sudo mv siva.tar.gz /home/
   ```

5. **Verify the task**
   ```bash
   ls -lh /home/siva.tar.gz
   ```

## Output Log
```bash
thor@jump-host ~$ cd /data/
thor@jump-host /data$ ls
siva
thor@jump-host /data$ cd siva/
thor@jump-host /data/siva$ ls
nautilus1.txt  nautilus2.txt  nautilus3.txt
thor@jump-host /data/siva$ cd ..
thor@jump-host /data$ sudo tar -czvf siva.tar.gz -C /data siva
siva/
siva/nautilus2.txt
siva/nautilus3.txt
siva/nautilus1.txt
thor@jump-host /data$ sudo mv siva.tar.gz /home/
thor@jump-host /data$ ls -lh /home/siva.tar.gz
-rw-r--r-- 1 root root 181 Mar 28 15:45 /home/siva.tar.gz
thor@jump-host /data$ 
```