# Operations

The homelab is operated as a real system, but its reliability claims are kept proportional to the evidence. Monitoring is established. Local recovery exists. Complete disaster recovery does not.

## Monitoring

The observability stack combines three perspectives:

- **Prometheus and node_exporter** collect resource and operating-system metrics.
- **Grafana** provides dashboards for capacity and behaviour over time.
- **Uptime Kuma** checks whether internal applications and selected network services respond.

Alerts are delivered through a Discord webhook. This replaced an older notification path that had become stale while dependent services were being removed.

A remaining limitation is placement: the monitoring stack lives inside the same physical node as the applications. It can detect application failures, but an external monitor is still needed to detect total node, power, or site connectivity loss independently.

## Backup posture

The backup design has changed through several stages.

### Stage 1: configuration copies on the production disk

The first approach protected against accidental configuration edits but not physical disk failure.

### Stage 2: external storage

A later job created container snapshots, separate archives for bind-mounted data, and host configuration copies on external storage. A restore test exposed an important behaviour: container snapshots did not automatically include the bind-mounted application data, so recovery required both artifacts.

The external enclosure later failed after a power event. Its backup path stopped being a trustworthy target.

### Current interim stage

Scheduled backup jobs are active. They create selected workload snapshots, separate data archives, and notifications. During the read-only verification on 2026-08-17, recent backup artifacts were present.

However, every detected artifact was still on the same physical filesystem as the running system. The current process is useful for recovering from a broken container or configuration mistake, but it does not protect against loss of the internal NVMe.

Large datasets and high-value personal data also need explicit scope decisions because bind mounts are not covered merely by snapshotting a container.

### What the current backup can and cannot claim

| Failure scenario | Current posture |
|---|---|
| Bad configuration or broken selected workload | Local recovery artifact available |
| Loss of the internal NVMe | Production and detected backup artifacts are lost together |
| Theft, fire, or site-wide loss | No complete offsite recovery path yet |
| Bind-mounted data | Must be included explicitly; container snapshots alone are insufficient |

The next acceptable milestone is not “a backup job ran.” It is a copy in a separate failure domain followed by a successful restore test.

## Updates and maintenance

Security-focused unattended upgrades are enabled on most host and guest systems. They reduce the delay for routine operating-system patches but do not replace deliberate maintenance for hypervisor, kernel, Docker, application, or database changes.

The preferred change sequence is:

1. identify the current state and dependency;
2. confirm that the relevant recovery artifact exists;
3. make one scoped change;
4. verify the intended behaviour and important access paths;
5. update the private runbook;
6. update public documentation only if the architecture or portfolio evidence changed.

This prevents a maintenance session from turning into an undocumented multi-service rewrite.

## Service lifecycle

Services are classified by use rather than by whether they still start:

- **Active:** regularly used or holding important data.
- **Parked:** retained in a reversible state but not routinely operated.
- **Retire:** going through controlled decommissioning.
- **Lab:** experimental and allowed to have a different failure tolerance.

Decommissioning includes data inventory, recovery decisions, dependency cleanup, and a reversible stop period before deletion. Removing an application also requires checking dashboards, proxy routes, DNS records, backup jobs, and notification consumers.

## Current operational risks

1. **Single physical node:** every workload shares host, power, and primary storage failure domains.
2. **Same-disk interim backup:** useful locally, insufficient for disk loss.
3. **No UPS:** an abrupt power loss can interrupt databases and storage writes.
4. **No external node monitor:** internal monitoring cannot independently observe a total host outage.
5. **Recovery coverage varies by dataset:** personal data, photographs, documents, and replaceable media do not have equal priorities or costs.

These risks drive the [roadmap](roadmap.md). New services and cluster experiments come after the recovery path becomes credible.
