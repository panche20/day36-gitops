# 🔄 Week 6, Day 36 — GitOps with ArgoCD (Complete from Scratch)

## Interview Foundation — Understand This First
Before touching any code, you need to understand the concepts deeply. Every senior DevOps interview will probe these.

### 🧠 What is GitOps? (The Philosophy)
GitOps is an operating model where Git is the single source of truth for both application code AND infrastructure state.
The word breaks down as:

- **Git** — version control system
- **Ops** — operations (deployments, infrastructure changes)

Combined: all operational changes happen through Git.

### The Core Principle
Traditional Ops:
Engineer → runs command → infrastructure changes (no audit trail, no review, hard to reproduce)

GitOps:
Engineer → commits to git → automated system applies change (full audit trail, peer review, reproducible)

### The Four GitOps Principles (CNCF definition)
1. Declarative
   - Describe WHAT you want, not HOW to get there.
   - Bad: `run kubectl scale --replicas=3`
   - Good: `replicas: 3` in a YAML file in git

2. Versioned and Immutable
   - Git stores every state the system was ever in.
   - You can see: who changed what, when, why (commit message).
   - You can recover: `git revert` to any previous state.

3. Pulled Automatically
   - The system pulls desired state from git.
   - Nothing pushes into the cluster from outside. This is the key security improvement over push-based CI/CD.

4. Continuously Reconciled
   - An agent constantly compares desired state (git) vs actual state (cluster) and fixes differences.
   - This is called reconciliation — not a one-time operation.

### Interview Answer: "What is GitOps?"
GitOps is an operational pattern where Git serves as the single source of truth for system state. Instead of engineers running `kubectl` commands or CI/CD pipelines pushing changes to clusters, a GitOps operator like ArgoCD runs inside the cluster, continuously watches a Git repository, and reconciles the cluster state to match what's in Git. This gives you a complete audit trail through git history, peer review through pull requests, and the ability to recover from any state by reverting a commit.


## 🧠 Push vs Pull Model (Critical Interview Topic)
This is the most important conceptual difference in GitOps.

### Push Model (Traditional CI/CD)
Problems:
1. CI system holds cluster credentials → If CI is compromised, cluster is compromised
2. No drift detection → Someone runs `kubectl` manually → cluster silently diverges from git
3. No self-healing → Someone deletes a deployment → it stays deleted until CI notices
4. CI needs network access to cluster → Opens firewall holes or cluster must be public

### Pull Model (GitOps)
Benefits:
1. Cluster credentials stay INSIDE the cluster → Nothing external can reach in
2. Drift detection is automatic → ArgoCD detects manual `kubectl` changes and self-heals
3. Cluster doesn't need to be reachable from CI → Works behind NAT/VPN/firewalls
4. Complete audit trail → Every change = git commit with author and message

### Interview Answer: "Push vs Pull in CI/CD"
In a push model, the CI/CD system holds credentials and pushes changes into the cluster. In a pull model, an agent like ArgoCD runs inside the cluster, polls the git repository, and pulls changes in. The cluster credentials never leave the cluster, there's no external system that can reach in, and any drift between desired and actual state is detected and corrected automatically.


## 🧠 ArgoCD Architecture (Deep Dive)
ArgoCD Components:

- API Server — REST/gRPC API, Web UI, CLI interface, Auth/RBAC
- Repo Server — clones git repos, renders Helm/Kustomize/plain YAML
- Application Controller — watches Kubernetes, compares desired vs actual, applies differences
- Redis — caches rendered manifests and cluster state
- Dex — optional OIDC provider for SSO

API Server handles user interactions and enforces RBAC. Repo Server clones and renders. Application Controller is the core engine that reconciles continuously.


### ArgoCD Core Concepts

- **Application** — the fundamental unit answering: "deploy THIS (source) to THERE (destination)."
- **Project** — groups Applications and enforces access control (who can deploy what, where, from which repos).
- **Sync** — make actual state match desired state. Can be manual or automated.
- **Health Status vs Sync Status** — Sync = matches git; Health = resource runtime condition.

Self-heal: ArgoCD detects drift (e.g., `kubectl scale`) and re-applies git state.
Prune: deleting a file from git with `prune: true` causes ArgoCD to delete the resource in-cluster.


## 🛠️ Complete From-Scratch Lab (Summary)
Below is a concise lab workflow used to build this project from scratch.

1. Project setup: repository layout for `app`, `helm/url-shortener`, `argocd/` and `scripts/`.
2. Application: FastAPI URL shortener (`app/main.py`) and `requirements.txt`.
3. Dockerfile: multi-stage build with `uvicorn` and production user.
4. Helm chart: `helm/url-shortener` with `values.yaml`, `values.dev.yaml`, `values.prod.yaml` and templates (ConfigMap, Secret, Deployment, Redis, Services).
5. Git: repository initialization, GitHub repo creation, push.
6. Minikube: `minikube start` for a local cluster (optional), enable ingress.
7. Install ArgoCD: apply official manifest into `argocd` namespace and wait for pods.
8. ArgoCD CLI: install and login using the initial admin secret.
9. Create ArgoCD Project: `argocd/projects/url-shortener-project.yaml` to restrict repos and destinations.
10. Create ArgoCD Application: `argocd/apps/url-shortener-dev.yaml` pointing at `helm/url-shortener` with `values.dev.yaml`.
11. Watch sync, verify pods and services, and port-forward to test the app.
12. Demonstrate GitOps: change `values.dev.yaml` in git (e.g., scale replicas) and let ArgoCD pull and reconcile.
13. Self-heal demo: manually break cluster state with `kubectl` and observe ArgoCD revert to git state.
14. Rollback via git: use `git revert` to undo a bad commit and let ArgoCD deploy the revert.
15. App of Apps: add `argocd/root-app.yaml` managing other Application YAMLs under `argocd/apps/`.
16. CI + GitOps integration: GitHub Actions builds image, pushes to GHCR, updates Helm values in git (CI never runs `kubectl`).
17. ApplicationSet: `argocd/appsets/url-shortener-appset.yaml` to generate multiple Applications from a template for multiple environments.


## Interview Q&A — Complete Preparation (Highlights)

Q1: What is GitOps and how does it differ from traditional CI/CD?
- GitOps: Git is single source of truth, pull-based reconciliation (ArgoCD). Push CI/CD pushes into cluster and holds credentials.

Q2: What is the App of Apps pattern?
- One ArgoCD Application watches a directory of Application YAMLs and manages them all.

Q3: What's the difference between Sync Status and Health Status?
- Sync = matches git vs actual; Health = runtime health (pods, containers). They are independent.

Q4: What does `prune: true` do and what's the risk?
- Deletes resources removed from git. Risk: accidental deletion if files are removed unintentionally.

Q5: How do you handle secrets in GitOps?
- Use Sealed Secrets, External Secrets, or Vault — never commit plaintext secrets.

Q6: How does ArgoCD handle multiple clusters?
- Register external clusters with `argocd cluster add`, then Applications can target those clusters.

Q7: What is ApplicationSet and when would you use it?
- Generates multiple Applications from a template for multiple environments/clusters/tenants.

Q8: How do you do a rollback in GitOps?
- Use `git revert` to create a new commit that undoes the bad commit; ArgoCD deploys the reverted state.


## Summary — What You Built
GitOps Platform:

- GitHub Repo ← single source of truth
- ArgoCD ← GitOps operator in cluster
  - Project ← access control boundary
  - Application ← deploy url-shortener to dev
  - Root App ← App of Apps pattern
  - ApplicationSet ← multi-environment generator
- CI Pipeline ← builds image, updates git tag (never touches cluster)
- Self-Healing ← manual changes auto-reverted

Demonstrated Concepts:

- ✅ Pull model vs push model
- ✅ Self-healing (manual changes reverted automatically)
- ✅ Rollback via git revert
- ✅ App of Apps pattern
- ✅ ApplicationSet for multi-environment
- ✅ CI updating git, not cluster
- ✅ Project-based access control
- ✅ Sync vs Health status

---

## How to test the app (local lab)
Follow these commands to run and verify the project locally using Minikube and ArgoCD. Run each block in a terminal.

```
export GITHUB_USER=your-github-username
export GITHUB_TOKEN=your-github-pat
```

1) Start Minikube

```bash
minikube start --driver=docker --memory=4096 --cpus=3 --kubernetes-version=v1.29.0
minikube addons enable ingress
kubectl cluster-info
kubectl get nodes
```

2) Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s
kubectl get pods -n argocd
```

3) Install ArgoCD CLI (optional but useful)

```bash
curl -sSL -o /tmp/argocd "https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64"
chmod +x /tmp/argocd
sudo mv /tmp/argocd /usr/local/bin/argocd
argocd version --client
```

4) Access ArgoCD and login (local port-forward)

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443 &
sleep 2
ARGOCD_PASS=$(kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath='{.data.password}' | base64 -d)
echo "ArgoCD URL: https://localhost:8080"
echo "Username: admin"
echo "Password: $ARGOCD_PASS"
argocd login localhost:8080 --username admin --password "$ARGOCD_PASS" --insecure
argocd account update-password --current-password "$ARGOCD_PASS" --new-password "ArgoCD@Day36!" --insecure 2>/dev/null || true
```

5) Create an ArgoCD Project

```bash
kubectl apply -f argocd/projects/url-shortener-project.yaml
argocd proj list
argocd proj get url-shortener
```

6) Create application namespace and deploy the ArgoCD Application from this repo

```bash
kubectl create namespace url-shortener || true
kubectl apply -f argocd/apps/url-shortener-dev.yaml
argocd app get url-shortener-dev
argocd app list
argocd app sync url-shortener-dev
argocd app wait url-shortener-dev --health --timeout 300
argocd app resources url-shortener-dev
argocd app diff url-shortener-dev
argocd app history url-shortener-dev
kubectl get pods -n url-shortener
```

6) Port-forward the app service and test endpoints

```bash
kubectl port-forward svc/url-shortener-app 9999:80 -n url-shortener &
sleep 3
curl http://localhost:9999/health

# Create a short URL
curl -s -X POST http://localhost:9999/shorten -H "Content-Type: application/json" -d '{"url":"https://argoproj.github.io"}' | jq

# Use the returned short_code in the stats endpoint:
# curl http://localhost:9999/stats/<SHORT_CODE>
```

7) Prove GitOps — Change via Git

```bash
# This is the core demonstration.
# We make a change in git and watch ArgoCD deploy it.
# NO kubectl commands. NO helm commands. Just git.

# Change the replica count
sed -i 's/replicas: 1/replicas: 2/' \
  helm/url-shortener/values.dev.yaml

git add helm/url-shortener/values.dev.yaml
git commit -m "scale: increase dev to 2 replicas for load testing"
git push origin main

echo "Change pushed to git. Watching ArgoCD sync..."
echo "(ArgoCD polls every 3 minutes, or force sync below)"
echo ""

# Force immediate sync (instead of waiting 3 minutes)
argocd app sync url-shortener-dev

# Watch pods scale up
kubectl get pods -n url-shortener -w &
sleep 20
kill %1

# Verify
kubectl get deployment url-shortener-app -n url-shortener
# READY: 2/2
```

8) Prove Self-Healing

```bash
# This demonstrates one of GitOps' most powerful features.
# Manually change the cluster — ArgoCD reverts it automatically.

echo "=== Self-Healing Demonstration ==="
echo ""
echo "Current state: 2 replicas (from git)"
kubectl get deployment url-shortener-app \
  -n url-shortener \
  -o jsonpath='{.spec.replicas}'
echo ""

echo "Manually scaling to 5 (breaking GitOps rules)..."
kubectl scale deployment url-shortener-app \
  --replicas=5 \
  -n url-shortener

echo "Replicas immediately after manual change:"
kubectl get deployment url-shortener-app \
  -n url-shortener \
  -o jsonpath='{.spec.replicas}'
echo " (shows 5)"

echo ""
echo "Waiting for ArgoCD to detect and revert..."
echo "(This takes up to 3 minutes)"

# Poll until ArgoCD reverts it
for i in $(seq 1 20); do
  sleep 10
  REPLICAS=$(kubectl get deployment url-shortener-app \
    -n url-shortener \
    -o jsonpath='{.spec.replicas}' 2>/dev/null)
  echo "  Check $i: replicas = $REPLICAS"
  if [ "$REPLICAS" = "2" ]; then
    echo "  ✅ ArgoCD reverted to git state (2 replicas)"
    break
  fi
done

# Check ArgoCD's view
argocd app get url-shortener-dev | grep -E "Sync|Health"
```

9) Rollback via Git

```bash
# Interview question: "How do you rollback in GitOps?"
# Answer: git revert — not helm rollback, not kubectl.
# The git history IS the deployment history.

echo "=== GitOps Rollback Demonstration ==="
echo ""

# See current git history
git log --oneline -5

# Make a "bad" change
cat >> helm/url-shortener/values.dev.yaml << 'EOF'

# Simulate bad config
badConfig:
  enabled: true
  brokenSetting: "this-will-cause-issues"
EOF

git add helm/url-shortener/values.dev.yaml
git commit -m "feat: add new feature (accidentally broken)"
git push origin main

argocd app sync url-shortener-dev

echo ""
echo "Bad commit deployed. Rolling back via git revert..."
echo ""

# Rollback = git revert
# This creates a NEW commit that undoes the bad one.
# The git history is preserved — you can see what happened.
git revert HEAD --no-edit
git push origin main

# ArgoCD picks up the revert and deploys the previous state
argocd app sync url-shortener-dev
argocd app wait url-shortener-dev --health --timeout 120

echo ""
echo "✅ Rolled back via git revert"
echo ""
git log --oneline -5
```

10) App of Apps Pattern

```bash
git add argocd/
git commit -m "feat: add App of Apps root application"
git push origin main
kubectl apply -f argocd/root-app.yaml
sleep 30
argocd app list
```

11) CI + GitOps Integration

```bash
git add .github/
git commit -m "feat: add CI+GitOps integration pipeline"
git push origin main
kubectl apply -f argocd/appsets/url-shortener-appset.yaml
sleep 10
argocd app list
```

12) Verification and Exploration

```
echo "=== ArgoCD Status ==="
argocd app list

echo ""
echo "=== Application Details ==="
argocd app get url-shortener-dev

echo ""
echo "=== Sync History ==="
argocd app history url-shortener-dev

echo ""
echo "=== Managed Resources ==="
argocd app resources url-shortener-dev

echo ""
echo "=== Current Diff (should be empty) ==="
argocd app diff url-shortener-dev

echo ""
echo "=== Pods ==="
kubectl get pods -n url-shortener
kubectl get pods -n url-shortener-dev 2>/dev/null

echo ""
echo "=== Test the app ==="
kubectl port-forward svc/url-shortener-app 9999:80 \
  -n url-shortener &
sleep 3

curl -s http://localhost:9999/health | python3 -m json.tool

RESPONSE=$(curl -s -X POST http://localhost:9999/shorten \
  -H "Content-Type: application/json" \
  -d '{"url":"https://argoproj.github.io"}')
echo $RESPONSE | python3 -m json.tool

CODE=$(echo $RESPONSE | python3 -c \
  "import sys,json; print(json.load(sys.stdin)['short_code'])")

curl -s http://localhost:9999/stats/$CODE | python3 -m json.tool

kill %1 2>/dev/null
```

Notes:
- If ArgoCD doesn't sync automatically wait a few minutes or run `argocd app sync url-shortener-dev`.
- For CI-driven image updates the GitHub Actions pipeline in `.github/workflows/ci-gitops.yml` updates the Helm values in git; ArgoCD will pull and deploy the change.
- For local image testing you can load images into Minikube with `minikube image load <image:tag>`.

---

## Interview Q&A — Complete Preparation

# Q1: "What is GitOps and how does it differ from traditional CI/CD?"

GitOps is an operational model where Git is the single source of truth for desired system state. Traditional CI/CD is push-based — the pipeline holds cluster credentials and runs kubectl/helm to push changes in. GitOps is pull-based — an agent like ArgoCD runs inside the cluster, watches git, and pulls changes. The key differences are: security (credentials stay in the cluster), drift detection (ArgoCD continuously compares desired vs actual state), self-healing (any manual change is automatically reverted), and auditability (every change is a git commit with author, timestamp, and message).

# Q2: "What is the App of Apps pattern?"

The App of Apps pattern uses one ArgoCD Application to manage other ArgoCD Applications. The root Application watches a directory of Application YAML files in git. When you add a new Application YAML to that directory, the root app deploys it. When you delete one, the root app removes it. This lets you bootstrap an entire cluster's application stack from a single kubectl apply on the root app. It's how you scale GitOps from one app to dozens.

# Q3: "What's the difference between Sync Status and Health Status?"

Sync Status answers "does the cluster match git?" — Synced means yes, OutOfSync means there's a difference. Health Status answers "are the deployed resources actually working?" — Healthy means all pods are running, Degraded means something is broken. They're independent. You can be Synced but Degraded — you successfully deployed a broken configuration. You can be OutOfSync but Healthy — someone scaled the deployment manually to a higher count, pods are running, but it doesn't match git.

# Q4: "What does prune: true do and what's the risk?"

With prune true, when you delete a resource from git, ArgoCD deletes it from the cluster. Without it, deleted resources accumulate as orphans. The risk is accidental deletion — if someone accidentally deletes a file from git and it gets merged, the production resource gets deleted. Mitigations: branch protection requiring PRs and approvals, sync windows restricting when auto-sync runs in production, and manual sync for production instead of automated.

# Q5: "How do you handle secrets in GitOps? You can't commit secrets to git."

Several approaches: Sealed Secrets (encrypt secrets with a cluster-specific key, commit the encrypted version — only the cluster can decrypt), External Secrets Operator (ArgoCD deploys a reference to AWS Secrets Manager/Vault, the operator fetches the actual secret at runtime), or Vault Agent Injection (Vault injects secrets directly into pods as files). The common principle: never commit plaintext secrets. The secret reference or encrypted form goes in git, the actual secret value lives elsewhere.

# Q6: "How does ArgoCD handle multiple clusters?"

ArgoCD can manage multiple Kubernetes clusters from a single ArgoCD installation. You register external clusters using argocd cluster add, which installs a ServiceAccount in the remote cluster and stores its credentials in ArgoCD. Then Application destinations reference the remote cluster's API endpoint. ApplicationSets with the Cluster generator automatically create one Application per registered cluster. This is the standard pattern for managing dev/staging/prod clusters from one control plane.

# Q7: "What is ApplicationSet and when would you use it?"

ApplicationSet is an ArgoCD extension that generates multiple Applications from a single template. Instead of maintaining 10 nearly-identical Application YAMLs for 10 environments, you write one template and use generators to create them all. Generators include: List (explicit values), Git (read from directory structure), Cluster (one per cluster), and Matrix (combinations). Use cases: same app across multiple clusters, same app across multiple namespaces/tenants, or generating apps from directory structure in a monorepo.

# Q8: "How do you do a rollback in GitOps?"

In GitOps, rollback is git revert. You create a new commit that undoes the changes from the bad commit, push it, and ArgoCD deploys the reverted state. This is better than helm rollback because: the git history is preserved (you can see what happened), the revert itself is reviewed and approved through the PR process, and you're guaranteed to get exactly the previous state (not a stale cached Helm release state). ArgoCD also has its own rollback via argocd app rollback, but this is a last resort — git revert is preferred.

