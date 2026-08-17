# Architecture

The homelab is intentionally small: one Proxmox node, unprivileged Linux containers, local storage, and private access. The design optimizes for understandable operations and useful learning rather than enterprise appearance.

## Goals and constraints

The system is expected to:

- host services used regularly;
- provide clear workload boundaries;
- remain reachable from home and while travelling;
- expose failures through monitoring and useful alerts;
- be maintainable by one operator;
- document limitations without presenting planned work as completed work.

Its current constraints are equally important:

- one physical compute node;
- one active internal NVMe for the operating system, workloads, and primary data;
- no UPS, NAS, or second compute node yet;
- no complete offsite recovery path;
- internal monitoring cannot independently observe a total host outage.

## Hardware

| Component | Current setup |
|---|---|
| System | HP ProDesk 400 G4 DM |
| CPU | Intel Core i5-8500T, 6 cores |
| Memory | 32 GB DDR4 |
| Active storage | 1 TB NVMe |
| Hypervisor | Proxmox VE 9 |

This hardware is sufficient for the present workloads. Additional compute is not the current bottleneck; recovery infrastructure is the more important investment.

## Compute model

The node runs 11 unprivileged Debian 12 LXC containers and no virtual machines.

Nine containers currently have Docker installed. Docker is used for multi-component application stacks and straightforward lifecycle management. Native systemd services are retained where they fit the application better, including selected platform and automation workloads.

Workloads are separated by operational role rather than one container per process. The role boundaries include:

- platform and access services;
- observability;
- media and personal data;
- development tooling;
- personal applications;
- automation and agent runtime;
- game hosting.

This provides useful container-level isolation without pretending that separate containers remove the shared host-level failure domain.

## Access and network shape

The normal access paths are:

1. **Home network:** direct private access or internal names resolved through the local DNS service and reverse proxy.
2. **Remote devices:** Tailscale provides authenticated overlay access without public application ports.
3. **Public internet:** no application route is intentionally enabled. The installed tunnel client returns no application ingress.

The Proxmox host also performs routing and NAT between the workload network and the current home network. This was introduced after a physical move changed the available gateway network. Keeping translation at one boundary avoided coordinated addressing changes across DNS, proxies, and application configurations.

This is pragmatic rather than ideal. The host already represents the main failure domain, so the added gateway role does not create a second independent dependency, but it does make recovery documentation important.

## Storage model

Proxmox uses an LVM thin pool on the internal NVMe. Container root filesystems hold operating systems and application configuration. Larger stateful datasets use separate logical volumes that are bind-mounted into the relevant workloads.

The separation provides several benefits:

- container roots remain smaller;
- large datasets can be managed independently;
- application state is easier to identify;
- restoring a container does not require treating every large dataset as part of its root filesystem.

It also creates a recovery requirement: a Proxmox container snapshot does not automatically include arbitrary bind-mounted data. Backup procedures therefore need both workload snapshots and explicit data archives.

## Failure domains

### Container-level failure

A broken application, Docker stack, package update, or container filesystem can often be repaired or restored without affecting every workload.

### Host-level failure

A failed NVMe, motherboard, power event, or host network failure affects the entire lab. More containers do not reduce this risk.

### Site-level failure

Theft, fire, or a larger electrical event can remove both the server and any storage beside it. Only an encrypted offsite copy addresses that class of failure.

These boundaries determine the roadmap: separate backup storage, restore testing, power protection, and external monitoring come before clustering.

## Architecture principles

- **Use before expansion:** services must have a real use case.
- **Private by default:** remote access does not require public application exposure.
- **Simple boundaries:** split workloads when recovery, lifecycle, security, or resource behaviour justifies it.
- **Recovery before availability theatre:** a second node is not a substitute for a tested backup.
- **Documentation follows verified state:** plans and historical experiments are not listed as active infrastructure.
