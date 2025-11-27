# AXV Operating Modes (Land / Sea / Degraded / Autopilot)

**Author:** Aster × Wojtek  
**Version:** v0.1  
**Status:** Draft (living document, evolves with AXV)

---

## 1. Purpose

This document defines **how AXV behaves in different environments and conditions**:

- full connectivity on land (HQ / CORE),
- limited / expensive connectivity at sea (@Sea),
- degraded / emergency conditions,
- situations where the **Captain is not on the bridge**, and Autopilot must hold the line.

It translates the principles from the **AXV Manifest (North Star)** into concrete operating modes.

---

## 2. Mode Overview

AXV recognizes four primary operating modes:

1. **Land Mode (HQ / CORE)**  
   - Full connectivity, full compute, Captain usually available.
2. **Sea Mode (@Sea)**  
   - Limited / expensive connectivity (Starlink, mobile), Captain often mobile.
3. **Degraded Mode**  
   - Partial failure of cluster / services, but AXV still partially available.
4. **Autopilot Mode**  
   - Captain not available; AXV keeps essential functions running and avoids risky changes.

These modes are **conceptual** – they may be implemented as flags, configs, or environment presets in the future.

---

## 3. Land Mode (HQ / CORE)

**Definition:**  
AXV is running with **normal / full connectivity** and good local conditions:

- HQ / CORE network is reachable,
- cluster nodes (Ruderka, KVMs, Pi5, etc.) are mostly healthy,
- internet uplink is stable and reasonably cheap,
- the Captain is usually reachable and in good shape.

**Capabilities in Land Mode:**

- Full dashboards (status page, incident log, Cargo Bay UI),
- Heavy RAG queries (Atlas), long-running analysis tasks,
- Code deployments and infrastructure changes (with normal safeguards),
- Syncs between CORE / HQ / cloud (Git, backups, media).

**Guiding principles:**

- Optimize for **productivity and learning**.
- Prefer **rich feedback** (dashboards, logs, visualizations).
- Autopilot may perform **safe routine tasks**, but high-impact changes should still be confirmed by the Captain.

---

## 4. Sea Mode (@Sea)

**Definition:**  
AXV is accessed from **ship / mobile environment** with:

- limited / expensive connectivity (e.g. Starlink, ship Wi-Fi),
- higher latency,
- the Captain often on iPhone / tablet / laptop in a cabin.

**Primary constraints:**

- Every megabyte costs money or comfort,
- Connectivity may drop or fluctuate,
- Time windows for focused work may be short and unpredictable.

**Capabilities in Sea Mode:**

- Lightweight status checks (API / status page zoomed-out view),
- Critical alerts only (P1/P2) – low-noise, high-signal,
- Access to **essential runbooks** and **Cargo Bay** (what is where, what is packed),
- Low-bandwidth interfaces (text first, no heavy media unless necessary).

**What Sea Mode should *avoid*:**

- Large model calls (unless absolutely necessary),
- Bulk data transfers (full backups, large syncs),
- Non-essential updates and experiments.

**Guiding principles:**

- **Bandwidth is a critical resource** – AXV should treat it like limited power on a spaceship.
- Prefer **summaries over full details**.
- Defer non-urgent work to Land Mode whenever possible.

---

## 5. Degraded Mode

**Definition:**  
AXV or its infrastructure is partially broken, but not completely offline.

Examples:

- Some cluster nodes are down (e.g. Ruderka unavailable),
- Atlas DEEP not reachable, but Atlas EDGE / KVM2 still works,
- Monitoring system sees inconsistent signals,
- Hardware issues, partial power/network failures.

**Capabilities in Degraded Mode:**

- Use whatever is still **safe and reliable**:
  - minimal status endpoints,
  - simplified dashboards,
  - offline copies of critical docs (runbooks, manifests).
- Suspend or limit:
  - heavy automations,
  - non-essential cron jobs,
  - risky changes.

**Guiding principles:**

- **Fail-safe over “keep everything alive at all costs”**.
- Prefer:
  - reduce functionality,
  - log everything,
  - avoid irreversible actions.
- The system should clearly **signal** that it is in Degraded Mode so that the Captain does not expect full capabilities.

---

## 6. Autopilot Mode

**Definition:**  
The Captain is not actively available:

- offline (sleep, travel, sickness),
- busy with real life,
- temporarily out of reach.

AXV must:

- **keep the ship alive**,
- avoid dangerous behavior,
- wake the Captain (or a human) **only when necessary**.

### 6.1. What Autopilot *does*

Autopilot should maintain:

1. **Monitoring & logging**
   - Status page stays up (as much as infra allows),
   - K8 incident log keeps receiving normalized snapshots,
   - Health checks continue with safe intervals.

2. **Self-healing (safe subset)**
   - Retry transient requests (with backoff),
   - Restart services known to be safe to bounce (idempotent, well-tested),
   - Switch to Degraded Mode if core components misbehave.

3. **Scheduled safe tasks**
   - Backups and log rotations (where failure is unlikely to corrupt data),
   - Data syncs that are:
     - verified,
     - and easily reversible.

4. **Alerts**
   - Send P1/P2 alerts when:
     - AXV is significantly broken,
     - there is risk of data loss,
     - or prolonged outage of critical components.

### 6.2. What Autopilot *does not do*

Autopilot must **not**:

- modify or rotate critical secrets,
- apply risky infrastructure changes (firewall, routing, storage layout),
- deploy new, untested versions of core services,
- run scripts that have not been explicitly whitelisted for Autopilot execution.

If Autopilot is unsure whether something is safe:

> It **does nothing**, logs the situation,  
> and waits for a human decision.

---

## 7. Mode Switching

Mode switching can be:

- **Implicit** – based on signals:
  - connectivity (latency, bandwidth, packet loss),
  - health checks of key nodes,
  - time of day / known schedules.
- **Explicit** – manually set by the Captain via:
  - a toggle in AXV UI,
  - a config file / environment variable,
  - a CLI or API call.

### 7.1. Examples

- HQ → Sea Mode:
  - triggered when the Captain flags “@Sea” in the UI or  
    when a device connects from known ship networks and data is expensive.
- Normal → Degraded:
  - triggered when key health checks fail (e.g. cluster nodes offline for N minutes).
- Any → Autopilot:
  - triggered when the Captain explicitly sets “AFK / offline”,  
    or inferred from schedule (sleep hours, travel windows).

---

## 8. Interaction with the Captain’s State

The Captain’s physical and mental state is not a “nice to know” – it shapes mode behavior:

- When you are tired / overloaded:
  - AXV should propose **smaller steps**,
  - avoid suggesting risky changes,
  - log more, decide less.
- When you are sick or intentionally offline:
  - AXV should lean more on Autopilot Mode,
  - alert only when there is a serious reason.

Future versions may formalize this via a simple status, e.g.:

- `captain_state = normal | tired | sick | offline | high_focus`

…which can influence recommendations, prompts, and autopilot thresholds.

---

## 9. Implementation Notes (Future)

This document is **conceptual** first. In future iterations, it can be used to define:

- mode flags in configuration (`mode = land | sea | degraded | autopilot`),
- n8n workflows that:
  - adjust polling intervals,
  - switch which alerts are sent,
  - adapt the level of detail in notifications,
- frontend behavior:
  - different dashboards or UI layouts per mode,
  - indicators of current mode (e.g. color, icon),
- runbook sections:
  - “How to act in Degraded Mode”,
  - “What Autopilot is allowed to do for this service”.

---

## 10. Relation to the Manifest

This document is a **companion** to:

- `01_AXV_Manifest_north star.md`

The Manifest defines **why** and **at which principles** AXV operates.  
This file defines **how** those principles play out in practice across different operating modes.

Both are **living documents** and should evolve together as AXV grows.
