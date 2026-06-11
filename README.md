# platform-lab-gitops

Argo CD-managed manifests for [platform-lab](https://github.com/amekala2514/platform-lab).

## Layout

```
.
├── apps/                          # Argo Applications (app-of-apps)
│   ├── inference.yaml             # ExternalName Service → host Ollama
│   ├── inference-gateway.yaml     # Go gateway (v0.2.0)
│   ├── open-webui.yaml            # chat UI
│   ├── grafana-dashboards.yaml    # LLM Inference dashboard
│   ├── observability-ingress.yaml # Grafana ingress
│   ├── platform-api.yaml          # reference workload
│   └── platform-api-monitoring.yaml
└── workloads/
    ├── inference-gateway/         # Deployment + Service + Ingress + ServiceMonitor
    ├── open-webui/                # PVC + Deployment + Service + Ingress
    ├── grafana-dashboards/        # ConfigMaps with grafana_dashboard=1 label
    ├── observability-ingress/     # Grafana Ingress
    └── platform-api/              # reference workload manifests
```

## How sync works

1. The bootstrap `root` Application (created out-of-band by `platform-lab/make argocd`) watches `apps/`.
2. Every file in `apps/` becomes an Argo Application via the app-of-apps pattern.
3. Each child Application syncs its `workloads/<name>/` directory.
4. All apps use `automated: { prune: true, selfHeal: true }` and `ServerSideApply=true`.

## Local hostnames

| Host                             | Ingress in                       |
|----------------------------------|----------------------------------|
| `argocd.platform-lab.test`       | argocd namespace (manual)        |
| `grafana.platform-lab.test`      | workloads/observability-ingress  |
| `chat.platform-lab.test`         | workloads/open-webui             |
| `inference.platform-lab.test`    | workloads/inference-gateway      |
| `platform-api.platform-lab.test` | workloads/platform-api           |

## Tags

- `v0.2.0` — Phase C: inference gateway + Open WebUI + LLM dashboard
- `v0.1.0` — Phase B: platform-api + observability
