# Day 11: Deploying a Java Application on Tomcat Server

## Theory

The Nautilus application development team recently finished the beta version of a Java-based application, planned for deployment on one of the app servers in Stratos DC. They decided to use the Tomcat application server.

**Requirements:**
1. Install Tomcat server on App Server 1.
2. Configure it to run on port 5002.
3. Deploy the `ROOT.war` file located at `/tmp` on the Jump host to the Tomcat server.
4. Ensure the webpage works directly on the base URL (e.g., `curl http://stapp01:5002`).

**Required Knowledge:**
* **Apache Tomcat:** An open-source Servlet Container used to run Java-based web applications.
* **WAR File (Web ARchive):** A compressed file containing all files (classes, images, XML files, etc.) of a Java web application.
* **ROOT.war:** Deploying an application as `ROOT.war` makes it accessible at the base URL (`http://ip:port/`) instead of `http://ip:port/app-name`.
* **Java Runtime Environment (JRE/JDK):** Since Tomcat is a Java application, Java must be installed on the server.
* **server.xml:** The main configuration file for Tomcat where port numbers (like changing 8080 to 5002) are configured in the `<Connector>` tag.

## Implementation Steps

### 1. Connect to App Server 1 from Jump Host
SSH into App Server 1 (`stapp01`) from the Jump host.
```bash
thor@jump-host ~$ ssh tony@stapp01
# Provide password when prompted
```

### 2. Install Java and Tomcat
Install Java and the Tomcat package using the package manager.
```bash
[tony@stapp01 ~]$ sudo yum install java-1.8.0-openjdk -y
[tony@stapp01 ~]$ sudo yum install tomcat -y
```

### 3. Configure Tomcat Port
Edit `server.xml` to change the default port from 8080 to 5002.
```bash
[tony@stapp01 ~]$ sudo vi /etc/tomcat/server.xml
```
Change the `<Connector>` tag:
```xml
<Connector port="5002" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```

### 4. Deploy the WAR File
Transfer `ROOT.war` from the Jump host to App Server 1, remove the existing ROOT folder, and copy the new WAR file.

From Jump Host:
```bash
thor@jump-host ~$ scp /tmp/ROOT.war tony@stapp01:/tmp/
```

From App Server 1:
```bash
[tony@stapp01 ~]$ sudo rm -rf /var/lib/tomcat/webapps/ROOT
[tony@stapp01 ~]$ sudo cp /tmp/ROOT.war /var/lib/tomcat/webapps/
```

### 5. Start Service and Verify
Restart and enable the Tomcat service, then verify the deployment.
```bash
[tony@stapp01 ~]$ sudo systemctl restart tomcat
[tony@stapp01 ~]$ sudo systemctl enable tomcat
[tony@stapp01 ~]$ curl http://stapp01:5002
```
If successful, you will see the HTML output of the web application.

## Output Log
```
thor@jump-host ~$ ssh ssh tony@stapp01
ssh: Could not resolve hostname ssh: Name or service not known
thor@jump-host ~$ ssh tony@stapp01
The authenticity of host 'stapp01 (10.244.29.208)' can't be established.
ED25519 key fingerprint is SHA256:Gg5jKT30SeiQSBz+pgnxSJomftpGbxqiTc8KQtMkMqY.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password: 
[tony@stapp01 ~]$ sudo yum install java-1.8.0-openjdk -y
...
[tony@stapp01 ~]$ sudo yum install tomcat -y
...
[tony@stapp01 ~]$ sudo vi /etc/tomcat/server.xml
...
<Connector port="5002" protocol="HTTP/1.1" ... />
...
thor@jump-host ~$ scp /tmp/ROOT.war tony@stapp01:/tmp/
...
[tony@stapp01 ~]$ sudo rm -rf /var/lib/tomcat/webapps/ROOT
[tony@stapp01 ~]$ sudo cp /tmp/ROOT.war /var/lib/tomcat/webapps/
[tony@stapp01 ~]$ sudo systemctl restart tomcat
[tony@stapp01 ~]$ sudo systemctl enable tomcat
[tony@stapp01 ~]$ curl http://stapp01:5002
<!DOCTYPE html>
<html>
    <head>
        <title>SampleWebApp</title>
    </head>
    <body>
        <h2>Welcome to xFusionCorp Industries!</h2>
    </body>
</html>
```