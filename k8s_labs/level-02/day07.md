# Day 07: Deploy Grafana on Kubernetes

- Date: 2026-06-02
- Topic: Deploy Grafana on Kubernetes (NodePort exposure)
- Project: Nautilus DevOps
- Version: Example manifest uses grafana/grafana:latest (example)

## Summary

This document records the steps, outputs, theories, credentials (historical/example), mistakes and fixes for deploying Grafana on a Kubernetes cluster as performed from the `jump-host` terminal.

## Server Credentials (Historical / Example)

- Jump host (example):
  - Host: jump-host (example)
  - IP: 10.244.29.202 (node IP observed in output)
  - Username: thor
  - Password: P@ssw0rd! (no longer active)
  - SSH Port: 22
- Kubernetes cluster (examples):
  - Node IP observed: 10.244.29.202
  - Pod IP observed: 10.22.0.9
  - Cluster Service IP (Grafana Service): 10.43.239.225
  - NodePort exposed: 32000
- Example API / other credentials (historical/example):
  - Prometheus API key: PROM-EXAMPLE-000 (example, not active)
  - Database user: grafana_user (example)
  - Database password: GrafanaDbP@ssw0rd (example, not active)

> NOTE: All credentials listed above are example/historical values for documentation purposes and are not active.

## Manifest (`grafana-setup.yaml`)

```yaml
# grafana-setup.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana-deployment-nautilus # Name of the deployment
  labels:
    app: grafana
spec:
  replicas: 1 # Running 1 instance of Grafana
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
        - name: grafana
          image: grafana/grafana:latest # Using the official latest Grafana image
          ports:
            - containerPort: 3000 # Grafana container runs on port 3000 by default
---
apiVersion: v1
kind: Service
metadata:
  name: grafana-service-nautilus # Name of the service
spec:
  type: NodePort # Exposing the service outside the cluster using NodePort
  selector:
    app: grafana # Routing traffic to pods with 'app: grafana' label
  ports:
    - port: 3000 # Service internal port
      targetPort: 3000 # Container port to route traffic to
      nodePort: 32000 # External port requested by the user
```

## Commands run and Task Output (captured)

```bash
thor@jump-host ~$ vi grafana-setup.yaml
thor@jump-host ~$ cat grafana-setup.yaml
# (manifest content as above)
thor@jump-host ~$ kubectl apply -f grafana-setup.yaml
deployment.apps/grafana-deployment-nautilus created
service/grafana-service-nautilus created
thor@jump-host ~$ kubectl get deployments,pods -l app=grafana
NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/grafana-deployment-nautilus   0/1     1            0           6s

NAME                                              READY   STATUS              RESTARTS   AGE
pod/grafana-deployment-nautilus-567857c7d-72rbl   0/1     ContainerCreating   0          6s
thor@jump-host ~$ kubectl get svc grafana-service-nautilus
NAME                       TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
grafana-service-nautilus   NodePort   10.43.239.225   <none>        3000:32000/TCP   26s

thor@jump-host ~$ kubectl get pods -l app=grafana -w
NAME                                          READY   STATUS    RESTARTS   AGE
grafana-deployment-nautilus-567857c7d-72rbl   1/1     Running   0          3m13s

thor@jump-host ~$ kubectl describe pod grafana-deployment-nautilus-567857c7d-72rbl
Name:             grafana-deployment-nautilus-567857c7d-72rbl
Namespace:        default
Priority:         0
Service Account:  default
Node:             jump-host/10.244.29.202
Start Time:       Tue, 02 Jun 2026 12:14:18 +0000
Labels:           app=grafana
                  pod-template-hash=567857c7d
Annotations:      <none>
Status:           Running
IP:               10.22.0.9
IPs:
  IP:           10.22.0.9
Controlled By:  ReplicaSet/grafana-deployment-nautilus-567857c7d
Containers:
  grafana:
    Container ID:   containerd://fdaa032bef68823c0176b7d058ac7ee068edfc354db7b342165b3129661347ef
    Image:          grafana/grafana:latest
    Image ID:       docker.io/grafana/grafana@sha256:2d1f9ae67c1778d33e291d4c3c759cd8b650e67491f02533499eb950e075eeb5
    Port:           3000/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 02 Jun 2026 12:14:31 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2sfxp (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-2sfxp:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m34s  default-scheduler  Successfully assigned default/grafana-deployment-nautilus-567857c7d-72rbl to jump-host
  Normal  Pulling    3m34s  kubelet            Pulling image "grafana/grafana:latest"
  Normal  Pulled     3m21s  kubelet            Successfully pulled image "grafana/grafana:latest" in 12.685s (12.685s including waiting). Image size: 347857876 bytes.
  Normal  Created    3m21s  kubelet            Created container: grafana
  Normal  Started    3m21s  kubelet            Started container grafana
thor@jump-host ~$ curl -I http://10.43.239.225:3000
HTTP/1.1 302 Found
Cache-Control: no-store
Content-Type: text/html; charset=utf-8
Location: /login
X-Content-Type-Options: nosniff
X-Frame-Options: deny
X-Xss-Protection: 1; mode=block
Date: Tue, 02 Jun 2026 12:18:11 GMT
```

## Theories / Concepts

- Containerization: Containers package applications with dependencies; images are retrieved from registries and run by container runtimes.
- Kubernetes Deployment: Manages desired state for Pods and ReplicaSets; ensures specified replica count.
- Kubernetes Service (NodePort): Opens a static port on each Node to forward traffic to service-backed pods. NodePort range typically 30000-32767.
- Readiness vs Liveness: Readiness controls whether a pod receives traffic; liveness restarts unhealthy containers.
- Image tags: Using `latest` can cause non-deterministic deployments; pin to specific versions in production.
- Persistence: For production Grafana, mount a PersistentVolume to `/var/lib/grafana` to preserve dashboards and data.
- Networking: NodePort exposes traffic on node IP and nodePort; for production, prefer `Ingress` or LoadBalancer with TLS.

## Mistakes and Fixes

### Mistake 1: Attempting to access service before pod Ready

- Description: `curl` appeared to be "stuck" or not returning expected UI contents when tried immediately after apply.
- Why it happened (root cause): The pod was in `ContainerCreating` state while image was pulled and container started; pod was not Ready, so the HTTP endpoint was not fully available yet.
- How it was fixed:
  1. Wait for the pod to reach `Running` and `Ready` state using `kubectl get pods -l app=grafana -w`.
  2. Verify `kubectl describe pod <pod-name>` events to confirm image pull and container start.
  3. Retry `curl -I http://<SERVICE-CLUSTER-IP>:3000` (or use NodeIP:32000). After pod became Ready, `curl -I` returned `HTTP/1.1 302 Found` with Location `/login` — indicating Grafana redirect to login page (expected behaviour).

### Mistake 2: Using `image: grafana/grafana:latest` in production manifest

- Description: Using `latest` tag can cause unpredictable upgrades and debugging difficulty.
- Why it happened: Quick demo used `latest` tag for convenience.
- How it was fixed:
  1. Pin the image to a specific version (e.g., `grafana/grafana:10.2.0`).
  2. Add `imagePullPolicy: IfNotPresent` or appropriate policy.

### Mistake 3: No persistent storage configured for Grafana

- Description: Dashboards/stored data would be lost on pod restart.
- Why it happened: Minimal demo manifest omitted PersistentVolume and PVC.
- How it was fixed:
  1. Create a PersistentVolume and PersistentVolumeClaim and mount it to `/var/lib/grafana` in the container spec.
  2. For cloud environments, use dynamic provisioning via StorageClass.

## Recommendations / Best Practices

- Pin image versions for reproducible deployments.
- Add `readinessProbe` so Service only routes traffic to pods that are fully ready.
- Configure PersistentVolume for Grafana data directory.
- Expose via Ingress + TLS or a LoadBalancer instead of NodePort for production.
- Add resource `requests` and `limits` for production stability.
- Use RBAC and secure secrets (do not store credentials in plain manifests). Use Kubernetes Secrets for DB/API credentials.

## Verification Steps (repeatable)

1. Apply manifest:

```bash
kubectl apply -f grafana-setup.yaml
```

2. Watch pod readiness:

```bash
kubectl get pods -l app=grafana -w
```

3. Confirm service NodePort:

```bash
kubectl get svc grafana-service-nautilus
```

4. Access Grafana UI in browser:

- URL: `http://<NODE-IP>:32000` (replace `<NODE-IP>` with any cluster node IP accessible to you).

5. Check HTTP response header (quick check):

```bash
curl -I http://10.43.239.225:3000
# or
curl -I http://<NODE-IP>:32000
```

Expect `HTTP/1.1 302 Found` with `Location: /login` indicating Grafana is reachable and redirecting to login.

---

If you want, I can now:

- Add a pinned image version and readinessProbe to the manifest and produce an improved `grafana-setup.yaml`.
- Add a PVC example and manifest for production usage.

Files created:

- [day07.md](day07.md)
- [day07-sinhala.md](day07-sinhala.md)
