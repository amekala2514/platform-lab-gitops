# platform-lab-gitops

GitOps source-of-truth for the [platform-lab](https://github.com/amekala2514/platform-lab) homelab cluster.

Watched by Argo CD running in-cluster. Any change merged to `main` is automatically reconciled into the cluster within ~3 minutes (or instantly if a webhook is configured).

## Layout

```
.
├── apps/                     # Argo CD Application CRs (the "app of apps")
│   └── platform-api.yaml     # one Application per workload
├── bootstrap/                # Root Application that points Argo CD at apps/
│   └── root-app.yaml
└── workloads/                # Actual K8s manifests Argo CD applies
    └── platform-api/
        ├── deployment.yaml
        ├── service.yaml
        ├── ingress.yaml
        └── servicemonitor.yaml
```

## How it works

1. Argo CD is bootstrapped by Terraform in [`platform-lab`](https://github.com/amekala2514/platform-lab/tree/main/terraform).
2. Terraform also creates a single root `Application` CR pointing at this repo's `apps/` directory.
3. The root app discovers and creates child `Application` CRs for each workload (app-of-apps pattern).
4. Each child app watches a directory under `workloads/` and syncs it.

## Sync policy

All apps use:
- **automated sync** with `prune: true` and `selfHeal: true`
- **3-minute reconciliation interval** (Argo CD default polling)
- **CreateNamespace=true** sync option

To force an immediate sync: `argocd app sync <app-name>` or click "Sync" in the UI.

## Adding a new workload

1. Create `workloads/<name>/` with K8s manifests
2. Create `apps/<name>.yaml` (an Argo CD Application CR pointing at `workloads/<name>`)
3. Commit and push to `main`
4. Root app picks it up within 3 min, creates the child app, syncs the workload

## Related

- Cluster + Argo CD provisioning: https://github.com/amekala2514/platform-lab
- v0.2 design plan: see `PLATFORM_LAB_V0.2_PLAN.md` in main repo
