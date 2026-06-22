# Day 37: Copying Files from Docker Host to Container

**Date:** 2026-06-22

**Topic:** Docker File Transfer - Copying Files from Host to Running Containers

**Project:** KodeKloud DevOps 100 Days Challenge

---

## Server Credentials (Historical)

| Parameter          | Value                       | Status                             |
| ------------------ | --------------------------- | ---------------------------------- |
| Jump Host          | thor@jump-host              | No longer active                   |
| Application Server | steve@stapp02               | No longer active                   |
| Host IP            | 10.244.29.239               | N/A                                |
| SSH Password       | (Historical, not disclosed) | No longer active                   |
| Container ID       | 9cf18af3409d                | Sample container for documentation |
| Container Image    | ubuntu:latest               | Example image                      |

---

## Task Output

### Initial Attempt on Jump Host

```bash
thor@jump-host ~$ docker ps
bash: docker: command not found
```

**Observation:** Docker command was not available on the jump host, as it lacked Docker installation.

### SSH Connection to Application Server

```bash
thor@jump-host ~$ ssh steve@stapp02
The authenticity of host 'stapp02 (10.244.29.239)' can't be established.
ED25519 key fingerprint is SHA256:74C2NPS23CTFGtSBIrIMcE52AkMrWWl+Bel/j+hC4fc.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
steve@stapp02's password:
[steve@stapp02 ~]$
```

**Observation:** Successfully established SSH connection to stapp02 server.

### Docker Operations on Application Server

```bash
[steve@stapp02 ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS              PORTS     NAMES
9cf18af3409d   ubuntu    "/bin/bash"   About a minute ago   Up About a minute             ubuntu_latest
[steve@stapp02 ~]$
```

**Result:** Successfully listed running containers. One Ubuntu container was active with ID `9cf18af3409d`.

### File Copy Operation

```bash
[steve@stapp02 ~]$ docker cp /tmp/nautilus.txt.gpg 9cf18af3409d:/tmp/
Successfully copied 2.05kB to 9cf18af3409d:/tmp/
[steve@stapp02 ~]$
```

**Result:** Successfully copied encrypted file (`nautilus.txt.gpg`, 2.05kB) from the Docker host to the running container's `/tmp/` directory.

---

## Theories

### Concept: Copying Files to Docker Containers

When working with Docker containers, there are scenarios where you need to transfer files between the host system and running containers. Docker provides built-in mechanisms for this purpose, each with specific use cases and implications.

### Method 1: Copying to a Running Container (docker cp)

**What it does:** The `docker cp` command allows you to copy files or directories between the Docker host and a container without stopping the container.

**Syntax:**

```bash
docker cp /path/on/host/file.txt <container_id_or_name>:/path/in/container/
```

**Example:**

```bash
docker cp ./config.json my_web_app:/usr/src/app/config.json
```

**Key Characteristics:**

- **Scope:** Applies only to the specific container instance
- **Persistence:** Changes are not permanent; if the container is stopped and removed, the file is lost
- **Overhead:** Minimal; no need to rebuild the image
- **Use Case:** Temporary file transfers, configuration updates, runtime data injection
- **Bidirectional:** Can copy from host to container or container to host using the same command

**Advantages:**

- Fast and immediate
- No image rebuild required
- Useful for debugging and temporary modifications
- Works on stopped containers as well

**Limitations:**

- Not suitable for permanent application files
- Changes are isolated to the container instance
- Not recommended for production application code

### Method 2: Baking a File into a Docker Image (Dockerfile)

**What it does:** The `COPY` or `ADD` instruction in a Dockerfile allows you to embed files permanently into the Docker image during the build process.

**Step A: Create a Dockerfile**

```dockerfile
# Start from your base image
FROM ubuntu:latest

# Copy the file from your host machine to the image
COPY ./my-file.txt /app/my-file.txt
```

**Step B: Build the new image**

```bash
docker build -t my-custom-image:latest .
```

**Result:** Every container created from `my-custom-image:latest` will automatically include `my-file.txt` in the `/app/` directory.

**Key Characteristics:**

- **Scope:** Applies to all containers created from the new image
- **Persistence:** Permanent; files are part of the image specification
- **Overhead:** Requires image rebuild
- **Use Case:** Application code, configuration templates, static assets
- **Deployment:** Ensures consistency across all instances

**Advantages:**

- Permanent and reproducible
- Consistent across all container instances
- Better for version control and CI/CD pipelines
- Ideal for production deployments

**COPY vs ADD Instruction:**

Both instructions copy files into the image, but they differ in behavior:

| Feature             | COPY            | ADD                     |
| ------------------- | --------------- | ----------------------- |
| Copies local files  | ✓               | ✓                       |
| Extracts .tar files | ✗               | ✓                       |
| Downloads from URLs | ✗               | ✓                       |
| Predictability      | ✓ (Recommended) | ✗ (Complex)             |
| Best Practice       | ✓               | Avoid for basic copying |

**Recommendation:** Use `COPY` for local files as it is more predictable and safer. Use `ADD` only when you specifically need its extra features.

### Workflow Decision Tree

```
Need to transfer files?
│
├─ Temporary/debugging? → Use docker cp
│
└─ Permanent/production? → Use Dockerfile COPY/ADD
```

---

## Mistakes and Fixes

### Mistake 1: Attempting Docker Commands on Jump Host

**Description:** Initially attempted to use `docker ps` on the jump host without ensuring Docker was installed.

**Why it happened:**

- The jump host is a bastion/gateway server with minimal tools
- Docker is not installed on every server in an infrastructure
- Assumed Docker would be available on all servers

**Root Cause:** Lack of infrastructure awareness; incorrect server target selection.

**How it was fixed:**

1. Identified that the jump host does not have Docker
2. Determined that Docker was needed on the application server (stapp02)
3. Established SSH connection to the application server
4. Executed Docker commands on the correct server where Docker was installed

**Learning:** Always verify tool availability on the target system before attempting to use commands. For Docker operations, identify which servers in your infrastructure have Docker installed and operational.

---

### Mistake 2: Not Verifying Container State Before File Transfer

**Description:** Could have verified if the target container was running before attempting the copy operation.

**Why it happened:**

- Assumed the container was already running based on the task description
- Did not explicitly check container status as a validation step

**Root Cause:** Skipped validation step in the workflow; could lead to errors in production scenarios.

**How it was fixed:**

1. Executed `docker ps` command to verify container status
2. Confirmed that container `9cf18af3409d` was in "Up" state
3. Only proceeded with file copy after validation

**Best Practice:** Always verify preconditions (container running, file exists, permissions correct) before performing operations. This prevents unnecessary troubleshooting and ensures reliable automation.

---

### Mistake 3: Choosing Between docker cp and Dockerfile COPY Without Clear Criteria

**Description:** Did not initially clarify which method (docker cp vs Dockerfile COPY) was appropriate for the specific use case.

**Why it happened:**

- Both methods accomplish the goal of getting files into containers
- Lack of distinction between temporary and permanent file requirements
- No explicit guidance on production vs development scenarios

**Root Cause:** Insufficient architectural decision-making; missing use-case analysis.

**How it was fixed:**

1. Analyzed the requirement: temporary file transfer to an existing container
2. Determined that `docker cp` was appropriate for this scenario
3. Recognized that for production, permanent application files should use Dockerfile
4. Documented both approaches to enable informed decisions in future tasks

**Recommendation Practice:**

- Use `docker cp` for: debugging, temporary modifications, configuration adjustments, maintenance tasks
- Use Dockerfile COPY for: application code, static assets, configuration templates, production images

---

## Summary

Successfully copied an encrypted file from the Docker host to a running Ubuntu container. The task demonstrated the use of `docker cp` command for temporary file transfer to an active container. Key learning includes understanding the appropriate contexts for using `docker cp` versus embedding files in Docker images through Dockerfile, and the importance of verifying infrastructure and container states before executing operations.
