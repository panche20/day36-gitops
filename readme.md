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

If you'd like, I can also:

- add this README as `README.md` (capitalized) as well for visibility,
- or commit and push to the GitHub repo if you provide credentials/access.
