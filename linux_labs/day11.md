# Linux String Substitution (sed)

## Scenario
At xFusionCorp Industries, the Stratos Datacenter houses a jump host server that stores template XML files essential for the Nautilus application. Prior to their use, these files need to be populated with valid data. The system administration team utilizes string and file manipulation commands to prepare these templates.

### Task
Substitute all occurrences of the string `About` with `LUSV` within the XML file located at `/root/nautilus.xml` on the jump host server.

### Infrastructure Details
* **Server**: Jump Host Server (`jump-host`)
* **User**: `thor`
* **Password**: `mjolnir123`
* **Target File**: `/root/nautilus.xml`

## Theory & Key Concepts

* **Stream Editor (`sed`)**: A powerful command-line tool in Linux used to perform basic text transformations on an input stream (a file or input from a pipeline). It is often used for find-and-replace operations.
* **Global Replacement Syntax**: `s/old_string/new_string/g`
  * `s`: Substitute command.
  * `/About/LUSV/`: Finds 'About' and replaces it with 'LUSV'.
  * `g`: Global flag. Without it, `sed` would only replace the first occurrence on each line.
* **In-place Editing (`-i` flag)**: By default, `sed` outputs the transformed text to standard output. The `-i` flag tells `sed` to edit the file in-place (saving the changes directly to the file).
* **Root Access**: Since the file is located in the `/root/` directory, superuser (`sudo`) privileges are required to access and modify it.

## Practical Steps (DevOps Approach)

1. **Log in to the Jump Host**
   ```bash
   ssh thor@jump-host
   # Password: mjolnir123
   ```

2. **Verify the file and occurrences (Using Sudo)**
   ```bash
   sudo grep "About" /root/nautilus.xml
   ```

3. **Take a Backup (DevOps Best Practice)**
   ```bash
   sudo cp /root/nautilus.xml /root/nautilus.xml.bak
   ```

4. **Perform String Substitution using `sed`**
   ```bash
   sudo sed -i 's/About/LUSV/g' /root/nautilus.xml
   ```

5. **Verify the changes**
   ```bash
   sudo grep "LUSV" /root/nautilus.xml
   sudo grep "About" /root/nautilus.xml # This should return empty now
   ```

## Output Log
```bash
thor@jump-host ~$ cd /root/
bash: cd: /root/: Permission denied
thor@jump-host ~$ sudo su
[root@jump-host /]# cd /root/
[root@jump-host ~]# ls
anaconda-ks.cfg  anaconda-post-nochroot.log  anaconda-post.log  nautilus.xml  original-ks.cfg
[root@jump-host ~]# exit
exit
thor@jump-host ~$ sudo cp /root/nautilus.xml /root/nautilus.xml.bak
thor@jump-host ~$ sudo sed -i 's/About/LUSV/g' /root/nautilus.xml
thor@jump-host ~$ sudo grep "About" /root/nautilus.xml
thor@jump-host ~$ 
```
