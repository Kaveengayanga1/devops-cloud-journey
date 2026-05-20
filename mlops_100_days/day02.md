# Day 02: Configuring JupyterLab Server for xFusionCorp Industries

**Date:** May 13, 2026
**Topic:** Troubleshooting and Configuring JupyterLab Infrastructure

## Task Overview

The objective was to configure and start a JupyterLab server for the xFusionCorp Industries data science team. The server had specific requirements regarding its listening port, IP binding, and root directory for notebooks.

## Implementation Details

### Server Environment & Credentials

- **Server:** `controlplane`
- **User:** `root`
- **Virtual Environment Path:** `/root/code/ml-env/`
- **Configuration File:** `/root/code/jupyter_lab_config.py`
- **Notebook Root Directory:** `/root/notebooks/`

### Terminal Output

```bash
root@controlplane ~/code via 🐍 v3.12.3 ✖ source /root/code/ml-env/bin/activate
root@controlplane ~/code via 🐍 v3.12.3 (ml-env) ➜  jupyter lab --config=/root/code/jupyter_lab_config.py --allow-root --no-browser &
[1] 4853
[W 2026-05-12 23:01:13.050 ServerApp] ServerApp.token config is deprecated in 2.0. Use IdentityProvider.token.
[W 2026-05-12 23:01:13.050 ServerApp] notebook_dir is deprecated, use root_dir
[I 2026-05-12 23:01:13.301 ServerApp] Serving notebooks from local directory: /root/notebooks
[I 2026-05-12 23:01:13.301 ServerApp] Jupyter Server 2.18.2 is running at:
[I 2026-05-12 23:01:13.301 ServerApp] http://controlplane:8888/lab
[I 2026-05-12 23:01:13.301 ServerApp]      http://127.0.0.1:8888/lab
```

## Troubleshooting: Mistakes & Fixes

| Problem                           | Cause                                                     | Resolution                                                                    |
| :-------------------------------- | :-------------------------------------------------------- | :---------------------------------------------------------------------------- |
| **Server inaccessible via Proxy** | Bind address was likely set to `127.0.0.1`.               | Changed `c.ServerApp.ip` to `'0.0.0.0'`.                                      |
| **Missing Directory**             | The directory `/root/notebooks/` did not exist.           | Created using `mkdir -p /root/notebooks/`.                                    |
| **Wrong Port**                    | The server was not listening on the required port `8888`. | Updated `c.ServerApp.port = 8888` in config.                                  |
| **Command Not Found**             | `netstat` was missing from the terminal path.             | Used standard server logs to verify startup or alternatively use `ss -tulpn`. |
| **Deprecated Config Items**       | Used `notebook_dir` instead of `root_dir`.                | Updated configuration to use `c.ServerApp.root_dir`.                          |

## Professional DevOps Instructions

1. **Prepare the Filesystem:**
   Ensure the notebook storage location exists.

   ```bash
   mkdir -p /root/notebooks/
   ```

2. **Revise the Configuration:**
   Edit `/root/code/jupyter_lab_config.py` to ensure the following parameters are set:

   ```python
   c.ServerApp.ip = '0.0.0.0'
   c.ServerApp.port = 8888
   c.ServerApp.root_dir = '/root/notebooks/'
   c.ServerApp.allow_root = True
   ```

3. **Activate Environment and Launch:**
   ```bash
   source /root/code/ml-env/bin/activate
   jupyter lab --config=/root/code/jupyter_lab_config.py --allow-root --no-browser &
   ```

## Theoretical Background

### IP Binding (0.0.0.0 vs 127.0.0.1)

Binding to `127.0.0.1` restricts access to the local machine only. Binding to `0.0.0.0` allows the server to listen on all network interfaces, making it reachable from outside (e.g., via a lab proxy or external IP).

### Virtual Environments

Essential for isolating Python dependencies. In this task, JupyterLab was installed specifically within the `ml-env` environment to avoid conflicts with system-wide packages.

---

_Created by GitHub Copilot_
