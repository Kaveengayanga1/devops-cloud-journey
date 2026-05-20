# Day 01: Jenkins Installation and Setup

## Task Description
The DevOps team at xFusionCorp Industries is initiating the setup of CI/CD pipelines and has decided to utilize Jenkins as their server. The objective is to install and configure Jenkins on the provided server.

### Requirements
1. Install Jenkins on the `jenkins` server using the `apt` utility only, and start it using the `service` command.
   - *Note: If you face a timeout issue while starting the Jenkins service, first check the service status with `service jenkins status`. Then review the logs in `/var/log/jenkins/jenkins.log` to identify the cause.*
2. Create an admin user with the following details:
   - **Username**: `theadmin`
   - **Password**: `Adm!n321`
   - **Full Name**: `Siva`
   - **Email**: `siva@jenkins.stratos.xfusioncorp.com`

### Credentials Used
- **Jump Host to Jenkins Server Login**: `root` user with password `S3curePass`
- **Jenkins Initial Admin Password**: `fcd289ebc79c401ea988b59d9002e2cf`

---

## Technical Theories

- **CI/CD (Continuous Integration & Continuous Deployment)**: The process of automating the building, testing, and deployment of code changes.
- **Jenkins**: A popular open-source automation server that enables the implementation of CI/CD pipelines through various plugins.
- **Java Runtime Environment (JRE/JDK)**: Jenkins is a Java-based application, so a compatible Java version (like OpenJDK 17 or 21) must be installed prior to setting up Jenkins.
- **Apt Utility**: The default package manager for Debian and Ubuntu-based Linux systems, used to install, update, and manage software packages.
- **Service Management**: In containerized environments or environments without `systemd` as PID 1, the `service` command is used instead of `systemctl` to manage background services.

---

## Execution Steps

1. **Access the Jenkins Server**
   ```bash
   ssh root@jenkins
   # Password: S3curePass
   ```
2. **Install Java**
   ```bash
   sudo apt update
   sudo apt install openjdk-21-jdk -y
   ```
3. **Add Jenkins Repository and Key**
   ```bash
   curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
   echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
   ```
4. **Install Jenkins**
   ```bash
   sudo apt update
   sudo apt install jenkins -y
   ```
5. **Start and Verify Jenkins Service**
   ```bash
   service jenkins start
   service jenkins status
   ```
6. **Retrieve Initial Admin Password**
   ```bash
   cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
7. **UI Configuration**
   - Click the "Jenkins" button on the top bar to access the Web UI.
   - Input the retrieved initial password: `fcd289ebc79c401ea988b59d9002e2cf`.
   - Complete the setup by creating the first admin user using the required credentials (`theadmin` / `Adm!n321`).

---

## Mistakes Made & How They Were Fixed

1. **SSH Authentication Failure**
   - **Mistake**: Entered the wrong password `Permission denied, please try again.` a few times when connecting to `root@jenkins`.
   - **Fix**: The correct password `S3curePass` was provided from the Jump Host instructions.

2. **Downloading the Wrong Jenkins GPG Key**
   - **Mistake**: Ran `wget -O /usr/share/keyrings/jenkins-keyring.asc https://jenkins.io`. This downloaded the Jenkins homepage HTML file instead of the actual ASCII GPG key.
   - **Resulting Error**: When running `apt update`, it showed `The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 7198F4B714ABFC68`.
   - **Fix**: The correct key URL `https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key` should be used to download the keyring.

3. **Using `systemctl` in a Non-Systemd Environment**
   - **Mistake**: Ran `systemctl status jenkins`.
   - **Resulting Error**: `System has not been booted with systemd as init system (PID 1). Can't operate. Failed to connect to bus: Host is down.`
   - **Fix**: Used the `service jenkins status` command format instead, as the container setup uses standard SysVinit or custom init scripts rather than `systemd`.
