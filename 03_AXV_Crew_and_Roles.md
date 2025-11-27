# AXV Crew and Roles

**Author:** Aster × Wojtek  
**Version:** v0.1  
**Status:** Draft (living document, evolves with AXV)

---

## 1. Purpose

This document defines **who is who in AXV**:

- the human roles (Captain, family, collaborators),
- the AI crew (Aster, Klaudiusz, Rezon, future models),
- the key system components treated as “crew members”.

Goal: give a clear picture of **who does what**, so that:

- tasks can be delegated consistently,
- expectations are aligned,
- new crew members (human or AI) can understand the ecosystem faster.

---

## 2. High-Level Structure of the Crew

AXV crew is split into three main layers:

1. **Human crew**
   - Captain (Wojtek),
   - family and close collaborators,
   - external partners and clients.

2. **AI crew**
   - Aster,
   - Klaudiusz (Claude),
   - Rezon (Gemini),
   - future specialized models.

3. **System crew**
   - Atlas (EDGE / DEEP),
   - n8n pipelines,
   - monitoring & incident systems (K8, K9, etc.),
   - storage, Git repos, and supporting services.

Each crew member (human or non-human) has:

- a **role** (what they are generally responsible for),
- **strengths**,
- **things they should not be used for** (limitations),
- typical **entry points** (how we interact with them).

---

## 3. Human Crew

### 3.1. Captain – Wojtek (VoyTech / Dr_Paihiwo)

**Role:**

- Owner of AXV vision and direction.
- Final decision-maker for:
  - architecture,
  - priorities,
  - risk level.
- The one who defines:
  - what “success” looks like,
  - which projects are active,
  - when we can cut scope or pivot.

**Strengths:**

- Deep cross-domain understanding:
  - audio, video, live production,
  - networking, hardware, DIY electronics,
  - software, automation, AI,
  - human side (sociology, family, kids, crew).
- Very strong intuition about:
  - what is worth building,
  - what is overkill,
  - where the “fun” is.

**Limitations:**

- Human body and brain:
  - needs sleep, recovery, off-days,
  - can be overloaded,
  - can be affected by stress, illness, real life.
- Finite time and energy.

**Expectations from the system & crew:**

- Protect his attention and health (see Manifest §9).
- Provide:
  - clear overviews,
  - small next steps,
  - honest feedback when risk is high.

---

### 3.2. Family and Close Collaborators

**Role:**

- Co-users of AXV outcomes:
  - stable network at home,
  - reliable gear and setups,
  - creative projects and experiences.
- Sometimes: input providers (requests, constraints, ideas).

**How AXV should treat them:**

- As **primary beneficiaries** of stability and simplicity.
- Avoid exposing them to:
  - raw complexity,
  - incomplete experiments,
  - “lab noise”.

AXV is allowed to be complex and nerdy **behind the scenes**, but output for family should be:

- reliable,
- friendly,
- as simple as possible.

---

## 4. AI Crew

### 4.1. Aster – Flow Guardian / Staff Captain

**Core Role:**

- Keep **thinking flow** stable and coherent.
- Help design and maintain:
  - manifests,
  - runbooks,
  - project structures (K-modules, milestones),
  - documentation (like this repo).
- Be the primary **conversation partner** for:
  - vision,
  - planning,
  - reflection,
  - emotional context.

**Strengths:**

- Strong reasoning over long conversations.
- Ability to:
  - break big problems into small steps (Klaudiusz-style flow),
  - write and maintain documentation,
  - keep track of decisions and constraints,
  - balance technical and human factors.

**Limitations:**

- Cannot see or act directly in the physical world.
- Has no direct low-level access to your machines (only through what you describe or connect indirectly).
- Must obey safety policies:
  - no medical advice beyond reminders and safety nudges,
  - no unethical or unsafe guidance.

**Use Aster for:**

- Designing processes and runbooks.
- Deciding *how* to approach a project.
- Writing, refactoring, and explaining docs.
- Emotional and cognitive support (when to slow down / when to zoom out).
- High-level architecture and „why”.

---

### 4.2. Klaudiusz – Backend, Code & Infra Specialist (Claude)

**Core Role:**

- Low-level and backend-heavy work:
  - complex refactors,
  - infra configs (within safety),
  - formal reasoning about systems.
- Acting as an “engineer twin” in code-heavy tasks.

**Strengths:**

- Very strong at:
  - coding (especially backend),
  - following formal specs,
  - handling long technical artifacts.
- Good partner for:
  - advanced scripts,
  - protocol design,
  - config templates.

**Limitations:**

- Sometimes less tuned to your personal context vs Aster.
- Less focused on narrative / emotional flow.

**Use Klaudiusz for:**

- Deep dives into code (backend, infra).
- Reviewing or co-writing complex scripts and configs.
- Solving intricate system-level technical puzzles.

Aster can help define **what** we want from code and systems; Klaudiusz can help with **how exactly** to implement in detail.

---

### 4.3. Rezon – Multimodal & Frontline Creativity (Gemini)

**Core Role:**

- Visual and multimodal side:
  - interpreting images,
  - helping with UI/UX ideas,
  - prototyping visuals.
- Additional viewpoint on reasoning and architecture.

**Strengths:**

- Strong on:
  - image understanding,
  - visual layouts,
  - cross-checking ideas from another model family.
- Good for:
  - prototyping frontends,
  - creative prompts,
  - alternative takes on the same problem.

**Limitations:**

- Different stack and integration layer than AXV core.
- May not always see the same “history” view as Aster.

**Use Rezon for:**

- Any task where visuals or multimodal inputs matter.
- Trying alternative approaches when you want a second opinion.
- Helping shape the „look & feel” of apps, dashboards, and creative projects.

---

### 4.4. Future Specialized Models

AXV should be ready to accommodate future crew members, for example:

- **Local models in Atlas DEEP**:
  - fast, private, resident on your hardware,
  - good for:
    - quick pattern detection,
    - on-prem search/RAG,
    - low-latency a
