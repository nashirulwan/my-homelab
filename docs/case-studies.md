# Case Studies

The most useful parts of this homelab are the incidents that changed how it is operated. These notes focus on symptoms, evidence, decisions, and lessons rather than publishing the private recovery commands.

## 1. Moving the server to a different home network (2026-06-27)

### Situation

After a physical move, the server was connected to a router using a different private network. The Ethernet link was active, but the Proxmox host, all workloads, and remote overlay access appeared offline.

### Diagnosis

A working link only proved that the physical connection existed. The server and workloads still expected their previous static gateway, so they had no usable route through the new router. Overlay access failed as a consequence because it still depended on working underlying internet connectivity.

The diagnosis proceeded from a laptop on the new network:

1. establish the network currently provided by the router;
2. temporarily reach the previous private network;
3. confirm that the server and workloads were still running;
4. inspect the configured route and identify the missing gateway assumption.

### Decision

Renumbering every workload would also require coordinated changes to internal DNS, reverse-proxy targets, application settings, and documentation. Instead, the Proxmox host was made responsible for forwarding and NAT between the retained workload network and the new home network.

### Result

Connectivity returned without changing each application individually. The approach also reduced the impact of a future physical move, although it increased the importance of documenting the host's gateway role.

### Lesson

When everything disappears at once, begin with the shared dependency. Physical link, addressing, routing, DNS, and overlay access are different layers; a healthy light on the network interface proves only the first.

## 2. Tailscale route acceptance broke incoming connections (2026-06-28)

### Situation

After enabling route acceptance on selected workloads, their applications stopped responding to incoming connections while outbound connectivity still worked.

### Diagnosis

The affected workloads already lived on the physical network being advertised through Tailscale. Accepting an overlay route for that same network introduced an alternate return path.

The normal routing table looked correct. The useful evidence was in Tailscale's policy-routing table: replies were being directed through the overlay interface instead of the local interface. The connection therefore became asymmetric.

### Decision and result

Route acceptance was disabled on workloads that were already directly attached to the advertised network. Incoming connectivity recovered immediately.

### Lesson

A route can be individually valid and still be wrong for the machine receiving it. When inbound and outbound behaviour differ, inspect policy rules and alternate routing tables rather than relying only on the default route display.

## 3. The backup failed, then its alert failed too (2026-07)

### Situation

An external storage enclosure failed after a power event, stopping the scheduled backup. The failure was not noticed through alerting; it was found during a manual audit.

### Diagnosis

Two independent-looking controls shared an undocumented lifecycle problem:

1. the backup could no longer write to its external target;
2. the notification still pointed to a service that had already been removed.

The backup and the mechanism intended to report its failure were both broken at the same time.

### Response

Alerting was consolidated to a maintained destination, and notification consumers became part of the service-removal checklist. An interim local backup was established while replacement storage remained unresolved.

### Remaining limitation

The interim copy reduces the cost of an application or configuration failure, but it shares the physical disk with production and therefore does not solve disaster recovery.

### Lesson

A configured alert is not evidence that an alert can arrive. Backup verification must test three separate outcomes:

- the job completed;
- the expected data was captured;
- success and failure notifications reached a currently maintained destination.

## Patterns carried forward

Across all three incidents, the same operating rules emerged:

- verify the failing layer before changing configuration;
- prefer the smallest reversible change;
- treat consumers and dependencies as part of a service lifecycle;
- distinguish a successful job from a recoverable result;
- record known gaps instead of hiding them behind “production-ready” language.
