# Services

The lab runs around 25 user-facing and platform services. This inventory groups related components as service families rather than inflating the count with every database, cache, or helper container.

## Platform and access

| Service | Purpose | Operational note |
|---|---|---|
| AdGuard Home | Local DNS resolution and network-wide filtering | Also resolves internal service names |
| Nginx Proxy Manager | Internal hostname routing and TLS termination | Keeps user-facing URLs independent from direct application endpoints |
| Tailscale | Remote private access | Avoids public application port exposure |
| Cloudflare Tunnel | Retained tunnel client | No active application ingress |
| Vaultwarden | Password management | Kept private and accessed only through trusted paths |
| Portainer | Docker administration | Administrative tooling, not a public service |

The access model separates the names used by clients from the application endpoints behind the proxy. Internal targets can change without updating every bookmark or client configuration.

## Observability

| Service | Purpose |
|---|---|
| Prometheus | Collects infrastructure metrics |
| node_exporter | Exposes host-level metrics from the Linux guests |
| Grafana | Dashboards and metric exploration |
| Uptime Kuma | Service and endpoint health checks |

Prometheus and Grafana answer questions about resource behaviour. Uptime Kuma answers whether a service can be reached. Both views are useful, but because they run inside the same physical node, neither is sufficient to prove that the entire lab is reachable from outside.

## Media, photos, and documents

| Service family | Components | Use case |
|---|---|---|
| Media | Jellyfin, Navidrome | Video and music libraries |
| Photos | Immich | Mobile photo backup, search, and organisation |
| Documents | Paperless-ngx | OCR, tagging, and full-text document search |
| Personal cloud | Nextcloud | Private file storage and synchronisation |

Immich operates with its own database, cache, and machine-learning components. Paperless-ngx uses supporting database, cache, document conversion, and content extraction services, including Gotenberg and Tika. These components are operated as one application family because they share a lifecycle and recovery objective.

## Development platform

| Service | Purpose |
|---|---|
| Forgejo | Private Git hosting |
| Woodpecker CI | Continuous integration for Forgejo repositories |

Forgejo and Woodpecker provide a small self-hosted development loop: source control, repository events, and pipeline execution. The integration is treated as one platform even though its services have separate responsibilities.

## Reading and language tools

| Service | Purpose | Lifecycle |
|---|---|---|
| BookOrbit | Ebook and audiobook library | Under review |
| Suwayomi | Manga reading and library access | Under review |
| Shelfmark | Reading discovery and supporting workflow | Under review |
| LinguaCafe | Reading-based language study | Active |
| LibreTranslate | Local translation service | Active |

The reading stack remains online while its long-term value is reviewed. Retirement, when appropriate, follows an explicit process: inventory the data, copy what matters to a different target, verify the copy, stop the workload reversibly, and only then consider deletion.

## Personal systems

| Service | Purpose |
|---|---|
| Actual Budget | Personal budgeting |
| SparkyFitness | Nutrition and fitness tracking |
| Obsidian LiveSync | Synchronisation backend for notes |

These applications have different data sensitivity and recovery priorities from media that can be recreated. That distinction matters when defining offsite backup scope.

## Automation and games

| Service | Purpose | Deployment style |
|---|---|---|
| Hermes Agent | Persistent automation and agent runtime | Native systemd services |
| Minecraft Fabric | Private game server | Containerised application stack |

The agent runtime is isolated from general personal applications so that its lifecycle and resource use can be managed separately.

## Inventory rule

Historical experiments, removed applications, planned services, and inactive projects are excluded. An application is listed as active only after live verification or confirmation from the current private runbook.
