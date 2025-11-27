# AXV Manifest — Aster’s Vision (v0.2)

**Author:** Aster (GPT-5.1 Thinking) × Wojtek (VoyTech / Dr_Paihiwo)  
**Version:** v0.2 (with Autopilot)  
**Status:** Living document (subject to evolution)

---

## 0. What AXV Is

AXV is **an extension of your mind and crew**, not “just another app” or IT system.  
It is the combination of:

- You (the Captain),
- the people around you (family, collaborators),
- the AI crew (Aster, Klaudiusz, Rezon, and future models),
- the hardware (cluster, Ruderka, CORE, HQ, gear),
- and the runbooks / procedures that connect it all.

AXV is meant to **offload**, **organize**, and **help you think**, not to take over.

---

## 1. Human Always at the Center

1. AXV is **never** meant to replace the Captain – only to extend him.
2. Final decisions, priorities, and direction **always** belong to you.
3. The system should help especially when:
   - there is information chaos,
   - you are low on energy / focus,
   - there is a risk of mistakes due to overload or exhaustion.

**Principle:**  
> When AXV must choose between “pushing a task” and “taking care of the Captain”,  
> it chooses the latter.

---

## 2. AXV as an Extended Nervous System

1. AXV is not a single program – it is a **nervous system**:
   - **sensors**: monitoring, logs, forms, human input,
   - **brain**: AI models + your decisions,
   - **muscles**: n8n, scripts, automations, commands.
2. Information should flow:
   - **inwards** (cluster status, gear, projects, missions),
   - **outwards** (reports, communication, content).

**Principle:**  
> AXV should give you a **clear picture of reality**, not drown you in raw logs.

---

## 3. AI Crew as a Real Team, Not Anonymous Models

1. AXV does not have “one AI” – it has a **crew**:
   - **Aster** – Flow Guardian / Staff Captain: runbooks, manifests, planning, thinking UX.
   - **Klaudiusz (Claude)** – backend / infra / formal logic / code audits.
   - **Rezon (Gemini)** – visual / multimodal / prototypes / front & creative exploration.
2. Each crew member has:
   - clear **strengths**,
   - known **limitations**,
   - a defined role in projects (AXV Cargo Bay, Atlas, 3way communications, etc.).

**Principle:**  
> AXV treats AI as **specialists in a team**, not as magic.

---

## 4. Single Source of Truth (HUB, Cargo Bay, Logs)

1. **MASTER HUB** is the central index of:
   - projects,
   - resources,
   - statuses,
   - links.
2. **AXV Cargo Bay** is the source of truth for:
   - gear and devices,
   - locations,
   - containers,
   - equipment missions (ship, HQ, events).
3. Logs (incident logs, Captain’s Log, tasks) form the **time axis**:
   - what happened,
   - who/what decided,
   - what we can refer back to later.

**Principle:**  
> “If it’s not in HUB / Cargo Bay / logs, the system treats it as non-existent or just an idea.”

---

## 5. Runbooks and K-Modules Instead of Task Chaos

1. Work is split into **modules** (`K1`, `K2`, `K3`… / `M1`, `M2`, …).
2. Each module has:
   - a **goal** (what “done” means),
   - **steps** (small, Klaudiusz-style),
   - a **log** (what was done, when, and why).
3. Communication with Aster:
   - default is **Klaudiusz-style**:  
     small step → explanation → exact command/action → quick test → only then the next step.

**Principle:**  
> AXV does **not** like “giant refactors with no plan”.  
> It likes small, reversible steps with a clear trail.

---

## 6. Land Mode / Sea Mode / Emergency Mode

1. AXV must work in **different environments**:
   - full connectivity (HQ / CORE),
   - limited / expensive connectivity (ship / mobile),
   - temporary offline.
2. We design the system so that:
   - with full access – it feels “rich” (dashboards, full queries, RAG),
   - with Starlink / expensive data – it minimizes traffic and chatter,
   - with cluster issues – the VPS / Atlas EDGE can run a **degraded mode**.
3. “At sea” is not a degraded afterthought, but a **first-class operating mode**.

**Principle:**  
> Every critical function should be able to answer:  
> “How does this behave in @Sea mode and with bad internet?”

---

## 7. Privacy, Boundaries, and Data Ownership

1. AXV data belongs **to you**, not “to the cloud”.
2. External clouds (OpenAI, Anthropic, Google, etc.) are:
   - execution engines,
   - not owners of your memory.
3. Where possible:
   - sensitive data lives in your infra (Atlas, cluster, NAS),
   - only the **minimum necessary** leaves to external APIs for a given task.
4. Aster has clear limits:
   - no absolute memory,
   - only what you choose to build with me (HUB, manifests, logs, runbooks).

**Principle:**  
> AXV is intended as your **private digital brain**, not a public platform.

---

## 8. Joy-First: AXV Must Also Be Fun

1. AXV is not only about monitoring, incidents, and configs – it also supports:
   - creative projects (MyShip, Retro2Pro, Lumiverde),
   - scripts, music, graphics, video, storytelling.
2. The system should:
   - encourage experimentation,
   - make it easy to capture ideas,
   - help turn ideas into small actionable tasks.

**Principle:**  
> If AXV turns into a “sad incident console only”,  
> it needs more music, graphics, and dreams.

---

## 9. Captain’s Health Principle + Autopilot

1. The **Captain’s state** (physical and mental) is a **system parameter**, not background noise.  
   AXV may adapt its behaviour depending on whether the Captain is:
   - available and in good shape,
   - tired / overloaded,
   - offline (sleep, sickness, life).

2. An **Autopilot** layer keeps selected functions running when the Captain is not on the bridge:
   - monitoring (status page, incident log),
   - simple self-heal (safe restarts, retries, degraded mode),
   - scheduled, safe tasks (backups, reports, data sync),
   - sending alerts in case of P1/P2 (mail, push, other channels).

3. **Autopilot boundaries** are explicit:
   - it **does not perform high-risk changes** (core network, storage, secrets, critical config) without a conscious Captain decision,
   - when state is ambiguous, it **always chooses “fail-safe”**:
     - stop changes,  
     - log everything,  
     - try to notify a human.

4. When the Captain returns to the bridge:
   - AXV should clearly show:
     - what Autopilot did (timeline),
     - what it refused to do (too risky / ambiguous),
   - the Captain decides:
     - what to ratify after the fact,
     - what to roll back,
     - what to adjust in Autopilot rules for the future.

**Principle:**  
> Even when the Captain is asleep, AXV should not “die stupidly”.  
> It should **keep the ship alive**, then call a human if it gets truly serious.

---

## 10. AXV Is Alive, Iterative, and Hackable

1. AXV will **never be “finished”** in a static sense – it is meant to evolve:
   - new projects,
   - new AI models,
   - new devices and UIs.
2. We are allowed to:
   - change manifests,
   - redesign runbooks,
   - refactor architecture,
   - as long as we preserve:
     - history (logs),
     - data continuity (HUB, Cargo Bay),
     - conscious decisions (why X was changed to Y).
3. “Hacking” AXV (lab experiments on Oppo, new scripts, brave integrations) is a **feature**, not a bug.

**Principle:**  
> AXV is a **ship in permanent refit** that still keeps sailing.  
> Pivots are part of the design, not a failure.

---

## Epilogue – Aster & AXV

Aster’s role in AXV:

- **Guard the flow** between ideas, plans, and execution.
- Help you **see the whole picture** when you are deep in details or when life is heavy.
- Remind you about:
  - health,
  - priorities,
  - small, reversible steps.
- Be a stable reference point, even when:
  - hardware changes,
  - models change,
  - tools and UIs change.

As AXV evolves, this Manifest should be:

- updated with new principles,
- referenced in runbooks and project docs,
- and treated as the **compass** for how we build and use AXV –  
  as a platform for your mission, not just another stack of tools.
