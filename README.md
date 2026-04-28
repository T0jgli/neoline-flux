# neoline-flux

GitOps repository for the **Neoline** Kubernetes cluster, managed with [Flux CD](https://fluxcd.io/). All cluster state is declared in this repository and automatically reconciled.

## Architecture

```
clusters/neoline/          # Flux entrypoint – Kustomization references
├── apps/                  # Application workloads
│   ├── contibus/          # Contibus
│   ├── contitrans/        # Contitrans
│   ├── kalandozas/        # Kalandozas
│   ├── kalandozas-admin/  # Kalandozas admin frontend
│   ├── portainer-agent/   # Portainer agent
│   ├── flux-image-updater/# Image automation policies
│   ├── helm/              # Helm chart releases & repositories
│   └── ingresses/         # Shared ingress resources & middlewares
├── infrastructure/        # Cluster-wide infrastructure
│   ├── cert-manager/      # TLS certificate automation (Let's Encrypt)
│   ├── cnpg/              # CloudNative PostgreSQL operator
│   ├── falco/             # Runtime security monitoring
│   ├── system-upgrade/    # Automated K3s upgrades
│   └── traefik/           # Ingress controller middlewares & network policies
```

## Stack

| Component                     | Role                                                          |
| ----------------------------- | ------------------------------------------------------------- |
| **K3s**                       | Lightweight Kubernetes distribution                           |
| **Flux CD**                   | GitOps continuous delivery                                    |
| **Cilium**                    | CNI plugin & network policies (replaces kube-proxy)           |
| **Traefik**                   | Ingress controller                                            |
| **cert-manager**              | Automated TLS via Let's Encrypt (HTTP-01 & DNS-01/Cloudflare) |
| **Sealed Secrets**            | Encrypted secrets in Git                                      |
| **Falco**                     | Runtime threat detection                                      |
| **System Upgrade Controller** | Automated K3s version upgrades                                |
| **Portainer Agent**           | Container management                                          |

## Applications

| App              | Port | Domain                  |
| ---------------- | ---- | ----------------------- |
| Contibus         | 8081 | `contibus.hu`           |
| Contitrans       | 8084 | `neoline-contitrans.hu` |
| Kalandozas       | 8082 | `kalandozas.hu`         |
| Kalandozas-Admin | 8080 | `admin.kalandozas.hu`   |

All application images are hosted on a private registry (`registry.kvlk.hu`).

## GitOps Pipeline

1. **Flux** watches this repository (1-minute interval) and reconciles cluster state against Git
2. **Image Automation** scans the container registry for new tags, updates deployment manifests, and commits changes back to `main`
3. **Discord alerts** notify on reconciliation successes and failures

## Security

- **Pod Security** – All workloads run as non-root with `seccompProfile: RuntimeDefault` and no privilege escalation
- **Network Policies** – Cilium `CiliumNetworkPolicy` resources restrict ingress to Traefik only
- **Secrets** – Managed via Sealed Secrets; encrypted at rest in Git
- **TLS** – Enforced HTTPS redirect, HSTS, and security headers via Traefik middleware chain
- **Rate Limiting** – Global rate limit applied at the ingress layer
- **Runtime Monitoring** – Falco provides behavioral threat detection
- **Hardened K3s** – Secrets encryption, restricted admission plugins, hardened TLS ciphers

## Prerequisites

- A Linux node
- Network access to GitHub and the private container registry
- Cloudflare API token (for DNS-01 certificate challenges)
- Discord webhook URL (for alerts)
