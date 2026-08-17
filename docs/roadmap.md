# Roadmap

The roadmap prioritises failure recovery over adding more services or making the diagram look more sophisticated.

## Current baseline

As of 2026-08-17:

- one Proxmox mini PC runs all workloads;
- all active compute workloads are unprivileged Debian LXC containers;
- access is private through the home network and Tailscale;
- metrics and application checks are established;
- scheduled local recovery artifacts exist;
- the detected backup artifacts still share the production disk;
- there is no complete offsite copy, UPS, NAS, or external node monitor.

## Priority 1: separate the backup failure domain

The first storage improvement is a backup target on different physical hardware. It may be a small dedicated backup host, a two-bay NAS, or another maintainable storage system.

Acceptance criteria:

- backups no longer depend on the production NVMe;
- bind-mounted data is included explicitly;
- retention is defined for high-value and replaceable datasets;
- at least one file-level and one workload-level restore are tested;
- backup failure notifications are tested, not merely configured.

A mirrored disk pair may reduce interruption after one disk failure, but it is not a replacement for versioned or offsite backups.

## Priority 2: protect high-value data offsite

Documents, application databases, notes, and selected photographs should have an encrypted offsite copy. Replaceable media can use a different policy to control storage and transfer costs.

The offsite design needs:

- explicit dataset scope;
- encryption before upload;
- retention and pruning rules;
- measured initial and recurring transfer requirements;
- a restore procedure tested from a clean location.

## Priority 3: power-loss handling

A UPS should cover the Proxmox host and the networking equipment required for controlled shutdown and recovery. The goal is not long runtime; it is enough time to survive short interruptions or shut down cleanly.

Acceptance criteria:

- actual power draw is measured;
- runtime and low-battery thresholds are selected from evidence;
- graceful shutdown is automated;
- recovery after power returns is documented and tested.

## Priority 4: external monitoring

At least one check should run outside the Proxmox node. It should distinguish application failure from full-node or home-connectivity failure and deliver alerts through a path that does not depend entirely on the failed node.

## Priority 5: service lifecycle cleanup

The reading stack is under review. Any retirement will preserve selected data, verify the copy, stop the workload reversibly, and remove dependencies before deletion.

The platform and personal application groups also need maintained dependency maps so recovery order is clear when DNS, proxying, monitoring, data services, or automation are unavailable.

## Later, when justified

The following are useful learning projects but not current reliability priorities:

- a second Proxmox node;
- clustering and migration experiments;
- managed network segmentation;
- a dedicated firewall appliance;
- local GPU workloads;
- higher-speed networking.

A second node can improve maintenance flexibility and compute availability. It does not make data safe unless replication, quorum, storage, and recovery behaviour are designed and tested as well.

## Definition of “ready to expand”

Expansion becomes reasonable after:

- important data exists outside the production failure domain;
- a restore drill has succeeded;
- power-loss behaviour is controlled;
- full-node failure can be detected externally;
- current workloads have clear lifecycle and recovery priorities.

Until then, the most valuable homelab project is making the existing system recoverable.
