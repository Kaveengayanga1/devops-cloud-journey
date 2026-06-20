Day 36: Docker NGINX Container Deployment and Troubleshooting
Date: 2026-06-20
Topic: Running an `nginx:alpine` container, handling port conflicts, and resolving container name reuse conflicts.
Project: kodekloud/devops_100_days
Version: 1.0

## Server Credentials (Historical / Example)

| Item                   | Value                                              | Status                                    |
| ---------------------- | -------------------------------------------------- | ----------------------------------------- |
| Jump host user         | thor                                               | Historical value (inactive)               |
| Jump host hostname     | jump-host                                          | Historical value (inactive)               |
| Target server user     | tony                                               | Historical value (inactive)               |
| Target server hostname | stapp01                                            | Historical value (inactive)               |
| Target server IP       | 10.244.49.58                                       | Historical value (inactive)               |
| SSH port               | 22                                                 | Historical value (inactive)               |
| SSH password           | Example: `T0ny@123!`                               | Historical/example only; no longer active |
| Host key fingerprint   | SHA256:b+8h5o8jTMB11DMBkoOPD9jac5Z236sotKmKp8mcHBA | Historical value                          |

## Other Credentials (Historical / Example)

| Credential Type            | Example Value                                            | Status                                    |
| -------------------------- | -------------------------------------------------------- | ----------------------------------------- |
| Docker registry credential | Not required for public `nginx:alpine` pull in this task | N/A                                       |
| API key                    | `API_KEY_EXAMPLE_123456`                                 | Historical/example only; no longer active |
| Database password          | `DB_Pass_Example!2026`                                   | Historical/example only; no longer active |

## Task Output

The following is the verbatim terminal output captured during the task.

```bash
thor@jump-host ~$ ssh tony@stapp01
The authenticity of host 'stapp01 (10.244.49.58)' can't be established.
ED25519 key fingerprint is SHA256:b+8h5o8jTMB11DMBkoOPD9jac5Z236sotKmKp8mcHBA.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password:
[tony@stapp01 ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[tony@stapp01 ~]$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[tony@stapp01 ~]$ docker run --name nginx_1 -p 8080:80 -d nginx:alpine
Unable to find image 'nginx:alpine' locally
alpine: Pulling from library/nginx
6a0ac1617861: Pull complete
9d80b52aca3c: Pull complete
696fe85c76cd: Pull complete
f982d9d7587e: Pull complete
2ff91eb02cdc: Pull complete
d4676175f322: Pull complete
e0b71e755d67: Pull complete
747ffc4966dd: Pull complete
Digest: sha256:20316569d8f81a160065d7d2a5eeffc7ca97d79022462ee255fd23fa103a6b5c
Status: Downloaded newer image for nginx:alpine
bb3a900af4184c376021b2b8161636a5113ef757a37c837e16efb3b6c725f00b
docker: Error response from daemon: driver failed programming external connectivity on endpoint nginx_1 (41093987cc791f8ea5e9324f83b0b2d25aa2d5ab71eeb93404e6723ba19db2c3): Error starting userland proxy: listen tcp4 0.0.0.0:8080: bind: address already in use.
[tony@stapp01 ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[tony@stapp01 ~]$ docker run --name nginx_1 -p 8081:80 -d nginx:alpine
docker: Error response from daemon: Conflict. The container name "/nginx_1" is already in use by container "bb3a900af4184c376021b2b8161636a5113ef757a37c837e16efb3b6c725f00b". You have to remove (or rename) that container to be able to reuse that name.
See 'docker run --help'.
[tony@stapp01 ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[tony@stapp01 ~]$ docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS    PORTS     NAMES
bb3a900af418   nginx:alpine   "/docker-entrypoint.…"   30 seconds ago   Created             nginx_1
[tony@stapp01 ~]$ docker rm bb3a900af418
bb3a900af418
[tony@stapp01 ~]$ docker run --name nginx_1 -p 8081:80 -d nginx:alpine
9da7fa8b26e82a4ada927dc59d7a731a9bf16fc7dd2bab0a62288c6e74ef9c5e
[tony@stapp01 ~]$
[tony@stapp01 ~]$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                                   NAMES
9da7fa8b26e8   nginx:alpine   "/docker-entrypoint.…"   6 seconds ago   Up 5 seconds   0.0.0.0:8081->80/tcp, :::8081->80/tcp   nginx_1
[tony@stapp01 ~]$
```

## Theories

### Core Command Used

To create and run an NGINX container from the Alpine-based image:

```bash
docker run --name my-nginx-container -p 8080:80 -d nginx:alpine
```

### Command Breakdown

- `docker run`: Creates and starts a new container.
- `--name my-nginx-container`: Assigns a readable container name.
- `-p 8080:80`: Maps host port `8080` to container port `80`.
- `-d`: Runs the container in detached (background) mode.
- `nginx:alpine`: Uses the lightweight Alpine variant of NGINX.

### Why the Pull Happened

The image was not available locally, so Docker pulled `nginx:alpine` from Docker Hub before attempting to start the container.

### Port Binding Theory

A host port can be bound by only one process/container at a time. If port `8080` is already occupied, Docker fails with `bind: address already in use`.

### Container Name Uniqueness Theory

Docker container names are unique on a host. Even a `Created` or `Exited` container still reserves its name until removed or renamed.

### Verification Method

After successful deployment, container state can be verified with:

```bash
docker ps
```

A healthy output shows the container in `Up` status and expected port mapping, for example `0.0.0.0:8081->80/tcp`.

## Mistakes and Fixes

### Mistake 1: Attempting to use occupied host port 8080

- What happened: Container start failed with `bind: address already in use` when mapping `8080:80`.
- Why it happened (root cause): Port `8080` was already used by another process or service on the host.
- How it was fixed:
  1. Recognized the port allocation error from Docker daemon output.
  2. Switched to an available host port (`8081`) while still mapping to container port `80`.
  3. Re-ran the container command with `-p 8081:80`.

### Mistake 2: Reusing the same container name without cleanup

- What happened: Second run failed with `Conflict. The container name "/nginx_1" is already in use...`.
- Why it happened (root cause): The previous failed-start container existed in `Created` state and still held the name `nginx_1`.
- How it was fixed:
  1. Listed all containers including non-running ones using `docker ps -a`.
  2. Removed the stale container using `docker rm bb3a900af418`.
  3. Re-executed `docker run --name nginx_1 -p 8081:80 -d nginx:alpine` successfully.

### Mistake 3: Not validating all container states immediately

- What happened: `docker ps` initially showed no running containers, which could be misinterpreted as no container artifacts existing.
- Why it happened (root cause): `docker ps` shows only running containers; it does not include `Created` containers.
- How it was fixed:
  1. Used `docker ps -a` to view all container states.
  2. Identified the `Created` container and removed it.
  3. Adopted the habit of checking both `docker ps` and `docker ps -a` during troubleshooting.

## Final Outcome

- NGINX container `nginx_1` started successfully.
- Active port mapping: host `8081` to container `80`.
- Runtime status confirmed as `Up` in `docker ps`.

---

Day 36 documentation completed.
