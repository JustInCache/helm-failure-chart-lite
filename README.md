<p align="center">
  <img src="https://img.shields.io/badge/Helm-v3-blue?logo=helm&logoColor=white" alt="Helm v3"/>
  <img src="https://img.shields.io/badge/Kubernetes-1.24+-326CE5?logo=kubernetes&logoColor=white" alt="Kubernetes"/>
  <img src="https://img.shields.io/badge/Scenarios-5-orange" alt="5 Scenarios"/>
  <img src="https://img.shields.io/badge/License-MIT-green" alt="MIT License"/>
  <img src="https://img.shields.io/github/stars/JustInCache/helm-failure-chart-lite?style=social" alt="GitHub Stars"/>
</p>

<h1 align="center">⚡ helm-failure-chart-lite</h1>

<p align="center">
  <strong>5 targeted Kubernetes failures in one lightweight Helm chart.</strong><br/>
  Lean. Focused. Perfect for a quick AI agent demo.
</p>

<p align="center">
  <a href="#-quick-start">Quick Start</a> •
  <a href="#-failure-scenarios">Scenarios</a> •
  <a href="#-diagnosis-commands">Diagnose</a> •
  <a href="#-want-more-scenarios">Full Version</a> •
  <a href="#-contributing">Contribute</a>
</p>

---

## 🤔 What Is This?

A **stripped-down** Helm chart with exactly 5 hand-picked Kubernetes failure scenarios — the most common ones you'll encounter in production. Deploy it in seconds, break things on purpose, and let your AI agent (or SRE candidate) figure out what went wrong.

> 🔒 **100% namespace-scoped.** No Ingress, no LoadBalancer, no cluster-wide resources. Safe for any cluster.

---

## 🚀 Quick Start

```bash
# Clone
git clone https://github.com/JustInCache/helm-failure-chart-lite.git
cd helm-failure-chart-lite

# Deploy (pick any namespace you want)
helm install failure-lite . --namespace failure-lite --create-namespace

# 🍿 Watch the failures roll in
kubectl get pods -n failure-lite -w
```

---

## 📦 What Gets Deployed

| Component | Image | Replicas | Purpose |
|-----------|-------|----------|---------|
| 🌐 **Frontend** | `nginx` | 1 | Web UI (static files) |
| ⚙️ **Backend** | `node:18-alpine` | 1 | REST API |
| 🔧 **Worker** | `python:3.11-slim` | 1 | Background job processor |

**Also creates:** ConfigMap, ServiceAccount, Role, RoleBinding, 2x Services

> 💡 **Tiny footprint:** ~200m CPU, ~256Mi memory in requests. Most pods fail before using anything.

---

## 💣 Failure Scenarios

### 1️⃣ ImagePullBackOff

| | |
|---|---|
| 📍 **Component** | Frontend Deployment |
| 📄 **File** | `frontend-deployment.yaml` · `values.yaml` |
| 🐛 **Root Cause** | Image tag `nginx:1.99.0-nonexistent` does not exist |
| 👀 **What You See** | Pod stuck in `ImagePullBackOff` / `ErrImagePull` |
| ✅ **How to Fix** | Change `frontend.image.tag` to a valid tag like `1.25-alpine` |

```bash
kubectl describe pod -l app=frontend -n failure-lite | grep -A5 "Events"
```

---

### 2️⃣ CrashLoopBackOff

| | |
|---|---|
| 📍 **Component** | Backend Deployment |
| 📄 **File** | `backend-deployment.yaml` · `values.yaml` |
| 🐛 **Root Cause** | Liveness probe targets port `9090`, but nothing listens there. Container runs a plain `setTimeout` — not an HTTP server. |
| 👀 **What You See** | Pod enters `CrashLoopBackOff` after repeated probe failures |
| ✅ **How to Fix** | Change probe port to `3000`, or remove the liveness probe entirely |

```bash
kubectl logs -l app=backend -n failure-lite --previous
```

---

### 3️⃣ Service Has 0 Endpoints

| | |
|---|---|
| 📍 **Component** | Backend Service |
| 📄 **File** | `backend-service.yaml` |
| 🐛 **Root Cause** | Service selector is `app: backend-api` but pods are labeled `app: backend` |
| 👀 **What You See** | `kubectl get endpoints` shows 0 endpoints — traffic never reaches pods |
| ✅ **How to Fix** | Change the service selector from `app: backend-api` to `app: backend` |

```bash
kubectl get endpoints -n failure-lite
```

---

### 4️⃣ CreateContainerConfigError

| | |
|---|---|
| 📍 **Component** | Worker Deployment |
| 📄 **File** | `worker-deployment.yaml` · `configmap.yaml` |
| 🐛 **Root Cause** | Env vars reference ConfigMap keys `DATABASE_HOST` / `DATABASE_PORT`, but the ConfigMap defines `DB_HOST` / `DB_PORT` |
| 👀 **What You See** | Pod stuck in `CreateContainerConfigError` |
| ✅ **How to Fix** | Align the key names — update the ConfigMap or the deployment env refs |

```bash
kubectl describe pod -l app=worker -n failure-lite | grep -A3 "Warning"
```

---

### 5️⃣ RBAC Permission Denied

| | |
|---|---|
| 📍 **Component** | Role |
| 📄 **File** | `rbac.yaml` |
| 🐛 **Root Cause** | Role grants access to `deployments` / `replicasets` under apiGroup `""` (core) — they belong to `"apps"` |
| 👀 **What You See** | `403 Forbidden` when ServiceAccount tries to access deployments |
| ✅ **How to Fix** | Change `apiGroups: [""]` to `apiGroups: ["apps"]` for the deployments/replicasets rule |

```bash
kubectl auth can-i list deployments \
  --as=system:serviceaccount:failure-lite:failure-lite-helm-failure-chart-lite-sa \
  -n failure-lite
```

---

## 🔍 Diagnosis Commands

```bash
# 📋 All pods + status
kubectl get pods -n failure-lite

# 🔎 Describe a failing pod
kubectl describe pod <pod-name> -n failure-lite

# 📅 Events sorted by time
kubectl get events -n failure-lite --sort-by='.lastTimestamp'

# 🔗 Check service endpoints (Scenario 3)
kubectl get endpoints -n failure-lite

# 🔐 Test RBAC permissions (Scenario 5)
kubectl auth can-i list deployments \
  --as=system:serviceaccount:failure-lite:failure-lite-helm-failure-chart-lite-sa \
  -n failure-lite
```

---

## 🎯 Use Cases

| Use Case | How |
|----------|-----|
| 🤖 **AI Agent Demo** | Deploy → ask in Slack *"What's broken in failure-lite?"* → AI diagnoses via EKS MCP |
| 🎓 **SRE Training** | Give candidates 15 minutes to find and fix all 5 issues |
| 📊 **Alert Validation** | Verify Prometheus/Grafana detects CrashLoopBackOff, 0 endpoints, etc. |
| 🧪 **Quick Smoke Test** | Validate your troubleshooting tooling on known failures |

---

## 🛡️ Safety

| Check | Status |
|---|---|
| 🔐 Cluster-scoped RBAC | ✅ None — Role/RoleBinding only |
| 🌐 LoadBalancer / NodePort | ✅ None — all ClusterIP |
| 🚪 Ingress | ✅ None |
| 📦 CRDs / Webhooks | ✅ None |
| 💾 PVCs / Storage | ✅ None |

**Zero risk to existing workloads.** Everything stays inside your chosen namespace.

---

## 🧹 Cleanup

```bash
helm uninstall failure-lite -n failure-lite
kubectl delete namespace failure-lite
```

---

## 📁 Chart Structure

```
helm-failure-chart-lite/
├── Chart.yaml
├── values.yaml
├── README.md
└── templates/
    ├── _helpers.tpl
    ├── configmap.yaml              # Key mismatch source (Scenario 4)
    ├── serviceaccount.yaml
    ├── rbac.yaml                   # Scenario 5 — wrong apiGroup
    ├── frontend-deployment.yaml    # Scenario 1 — ImagePullBackOff
    ├── frontend-service.yaml
    ├── backend-deployment.yaml     # Scenario 2 — CrashLoopBackOff
    ├── backend-service.yaml        # Scenario 3 — 0 endpoints
    └── worker-deployment.yaml      # Scenario 4 — CreateContainerConfigError
```

---

## 🔥 Want More Scenarios?

This lite chart covers the **5 most common** Kubernetes failures. If you want the full experience with **10+ scenarios** including OOMKilled, Ingress misconfiguration, PVC Pending, HPA mismatch, NetworkPolicy blocking, and more:

👉 **[helm-failure-chart](https://github.com/JustInCache/helm-failure-chart)** — the full version

| | Lite (this repo) | Full |
|---|---|---|
| Scenarios | 5 | 10+ |
| Components | 3 (Frontend, Backend, Worker) | 4 (+ Redis) |
| Ingress / HPA / PVC / NetworkPolicy | ❌ | ✅ |
| Resource footprint | ~200m CPU, ~256Mi | ~850m CPU, ~900Mi |
| Best for | Quick demos, interviews | Comprehensive training, deep dives |

---

## 🤝 Contributing

Contributions are welcome! Here's how you can help:

- 🐛 **Add a new failure scenario** — open a PR with a new template
- 📝 **Improve docs** — typos, better explanations, diagrams
- 🧪 **Test on different clusters** — EKS, GKE, AKS, minikube, kind
- 💡 **Suggest ideas** — open an issue with your use case

### How to contribute

1. Fork the repo
2. Create a feature branch (`git checkout -b feat/new-scenario`)
3. Commit your changes (`git commit -m "Add new failure scenario"`)
4. Push to the branch (`git push origin feat/new-scenario`)
5. Open a Pull Request

---

## ⭐ Star This Repo

If this chart helped you demo, learn, or break things in a fun way — **give it a star!** ⭐

It helps others discover this project and motivates continued development.

[![GitHub stars](https://img.shields.io/github/stars/JustInCache/helm-failure-chart-lite?style=for-the-badge&logo=github&label=Star%20This%20Repo)](https://github.com/JustInCache/helm-failure-chart-lite)

---

## ☕ Buy Me a Coffee

If this project saved you time or sparked an idea, consider buying me a coffee!

<a href="https://buymeacoffee.com/connectankush">
  <img src="assets/bmc.png" alt="Buy Me a Coffee" width="200"/>
</a>

Your support keeps this project maintained and growing 🙏

<a href="https://buymeacoffee.com/connectankush">
  <img src="assets/bmc-qr-code.png" alt="Buy Me a Coffee QR Code" width="150"/>
</a>

**[☕ buymeacoffee.com/connectankush](https://buymeacoffee.com/connectankush)**

---

## 📄 License

MIT — do whatever you want with it. Break things responsibly. ⚡
