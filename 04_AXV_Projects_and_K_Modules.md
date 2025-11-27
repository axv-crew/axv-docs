# AXV Projects and K-Modules

**Author:** Aster × Wojtek  
**Version:** v0.1  
**Status:** Draft (living document, evolves with AXV)

---

## 1. Purpose

This document defines **how AXV structures work into projects and K-modules**:

- how we name and track projects,
- what “K1 / K2 / K3 …” mean,
- how these map to:
  - MASTER HUB,
  - runbooks and docs,
  - infra and automations.

Goal: make AXV projects **legible and repeatable**, not reinvented from scratch every time.

---

## 2. What Is a Project in AXV?

A **project** in AXV is any coherent initiative that:

- has a clear **goal / outcome**,
- has **bounded scope** (even if evolving),
- requires more than one or two ad-hoc tasks.

Examples:

- AXV internal:
  - `AXV Cluster – Hybrid Option C`,
  - `AXV Cargo Bay — Gear & Inventory System`,
  - `AXV @ Sea – Mission & Scope`,
  - `AXV 3way Communications`.
- External / client:
  - `MyShip.guru v1.5 – Cleanup & Stabilization`,
  - client rebranding / websites,
  - event productions.
- Creative / family:
  - `Retro2Pro`,
  - `Lumiverde`,
  - songwriting / audio projects.

Each project should have:

1. **Name** – human-readable, stable over time.  
2. **Owner** – usually Wojtek; in future, possibly shared.  
3. **Type / domain** – e.g. AXV infra, client work, creative.  
4. **Priority** – how important it is vs other active projects.  
5. **Status** – Planning / In Progress / Paused / Done / Archived.  
6. **Storage / links** – where the main assets live (Drive, repos, docs).

MASTER HUB is the **index of all such projects**.

---

## 3. What Is a K-Module?

A **K-module** is a structured **phase or sub-project**, usually numbered:

- `K1`, `K2`, `K3`, … for major phases,
- `K1.1`, `K1.2` etc. for sub-steps if needed.

Think of a K-module as:

> “A chapter in the story of this project, with a clear outcome and a finite set of steps.”

Examples:

- For **AXV Cargo Bay**:
  - `K1 – Inventory & Data Model`,
  - `K2 – MariaDB & API`,
  - `K3 – Web UI`,
  - `K4 – QR & Labeling`,
  - `K5 – Mobile & Scanning`.
- For **MyShip.guru v1.5**:
  - `K1 – Audit & Backup`,
  - `K2 – Plugin Cleanup`,
  - `K3 – Theme / Layout fixes`,
  - `K4 – Performance & Caching`.

K-modules are:

- small enough to **reason about**,
- big enough to feel like a meaningful step.

---

## 4. Typical Life-Cycle of a Project

A simple, repeatable life-cycle:

1. **Discovery / Draft**
   - Name, rough scope, context.
   - Basic notes in MASTER HUB + maybe a scratch doc/canvas.

2. **Planning**
   - Define first K-modules (K1, K2, …).
   - Decide priority and status (e.g. High/Medium/Low, Planning/In Progress).

3. **Execution**
   - Work done K-module by K-module.
   - Each K-module has:
     - clear **goal**,
     - **steps** (often Klaudiusz-style),
     - a **log** of what was done and when.

4. **Stabilization / Runbook**
   - Once main work is done:
     - write or update runbook,
     - capture lessons learned,
     - ensure monitoring / alerts are in place if relevant.

5. **Done / Archived**
   - Project is complete (or consciously parked).
   - Docs and HUB entries remain for reference.

Projects may cycle through this multiple times as they evolve; AXV is a living system.

---

## 5. Structure of a K-Module

A typical K-module doc / section should answer:

1. **Name / ID**
   - `K1 – Inventory & Data Model`,
   - `K2 – MariaDB & API`, etc.

2. **Goal**
   - What does “done” mean in this module?
   - Example:  
     *“We have a clean inventory CSV + data model ready for MariaDB, with stable `asset_id` and basic packing list support.”*

3. **Context**
   - Why this K-module exists,
   - dependencies on other modules,
   - constraints (time, infra, health, etc.).

4. **Scope**
   - What is **in** and **out** of this K-module:
     - in: core inventory sheet, data validation, basic schemas,
     - out: full-blown web UI, mobile apps, etc.

5. **Steps (Klaudiusz-style)**
   - each step:
     - action / change,
     - explanation,
     - exact commands / operations if applicable,
     - quick test / validation.

6. **Log / Decisions**
   - what we actually did (date, change, result),
   - decisions made (“we chose X instead of Y because …”).

7. **Outcome / Next**
   - summary of what was achieved,
   - which K-module naturally follows (e.g. K2).

---

## 6. Klaudiusz-Style Execution Inside K-Modules

K-modules are the natural home for the **Klaudiusz-style** work pattern:

> small step → explanation → concrete action → quick test → next step.

Aster’s role inside a K-module:

- help design the sequence of steps,
- explain implications,
- keep track of what was done,
- nudge about risk levels and human factors.

This is especially important for:

- networking changes,
- cluster operations,
- schema migrations,
- anything that might break running systems.

---

## 7. MASTER HUB Integration

MASTER HUB should reflect:

1. **Projects**
   - one line per project:
     - name,
     - owner,
     - type,
     - priority,
     - status,
     - key links (docs, repos, storage),
     - current K-module / next milestone.

2. **K-modules (at high level)**
   - per project, list K1…Kx in a compact way:
     - short name,
     - status (Planned / In Progress / Done),
     - maybe due dates for critical ones.

Detailed logic and steps of each K-module live in:

- docs repo (`axv-docs`),
- sometimes in GitHub issues,
- sometimes in Google Docs / Sheets / Canvas linked from the HUB.

MASTER HUB is the **index**, not the place for full narrative.

---

## 8. Priorities and Status

At the **project level**, AXV typically uses:

- **Priority**:
  - e.g. `High / Medium / Low` or shorthand (`W / Ś / N`),
  - indicates how much attention / energy the project should get *right now*.
- **Status**:
  - `Planning` – shaping scope and first K-modules,
  - `In Progress` – active work happening,
  - `Paused` – temporarily on hold (life, health, higher priorities),
  - `Done` – outcome achieved,
  - `Archived` – closed, only for reference.

At the **K-module level**:

- `Planned` – defined but not started,
- `In Progress` – work is actively happening,
- `Done` – goal reached,
- `Dropped` – consciously cancelled / merged into another K-module.

This separation allows you to:

- pause a project while leaving a clear structure to return to,
- see at a glance what is active vs parked.

---

## 9. Mapping to Repos and Docs

Projects and K-modules should map in a predictable way to:

- **Repos**:
  - `axv-docs` – manifests, operating modes, crew & roles, specs,
  - other repos for code and infra (`axv-gw`, `axv-web`, etc.).
- **Docs**:
  - each major project can have:
    - `AXV_<ProjectName>_Overview.md`,
    - `AXV_<ProjectName>_K1_<ShortName>.md`,
    - `AXV_<ProjectName>_K2_<ShortName>.md`, etc.,
  - or grouped sections inside a single project doc.

The precise naming can evolve, but the **logic** should remain:

> “If I know the project name and K-module, I can quickly find the relevant doc.”

---

## 10. Relation to Other Documents

This document should be read together with:

- `01_AXV_Manifest_north star.md`
  - defines **why** AXV exists and the core principles.
- `02_AXV_Operating_Modes.md`
  - defines **how** AXV behaves in different environments.
- `03_AXV_Crew_and_Roles.md`
  - defines **who** is involved (human / AI / system crew).

Future companion docs:

- `05_AXV_Cargo_Bay_Spec.md`
  - data model, flows, and requirements for the inventory system.
- `06_AXV_3way_Communications.md`
  - how AXV handles communication flows between:
    - humans,
    - AI crew,
    - external services.

Together, these documents form the **backbone of AXV documentation**:  
Why → Who → How → What → In which mode → With which building blocks.
