# AXV Cargo Bay — UI & Flows (v1)

**Author:** Aster × Wojtek  
**Version:** v0.1  
**Status:** Draft (public-safe, implementation details can live in a separate internal doc)

---

## 1. Scope

This document describes the **behaviour and UI** of **AXV Cargo Bay** from the user’s perspective:

- what **roles** exist (warehouse / crew / admin),
- what the **main views** are (web + mobile),
- how **gear flows** in the context of missions (packing lists),
- how the UI behaves after **scanning a QR code**.

**Data model and DB schema** are defined in:  
`05_AXV_Cargo_Bay_Spec.md`.

This file focuses on:

- UX,
- screens,
- states and interactions.

It is written so that it can safely live in a public repo – without sensitive implementation details.

---

## 2. UX & brand assumptions

### 2.1. Overall mood

- Cargo Bay is **part of AXV**, not a separate brand:
  - it reuses the same **colour palette and typography**,
  - it has the same “night” mood (dark background, ocean, lights).
- Background:
  - dark, night ocean / bay,
  - in the distance a **port / dock** with subtle lights (optionally slightly animated, but nothing heavy).
- UI layer:
  - a **semi‑transparent “console” / panel** on top of the ocean,
  - layout optimised for **speed, simplicity and readability**,
  - animations:
    - fast, subtle (fade/slide),
    - no heavy visual effects.

The same concept works for:

- **web UI** – panel centred or slightly offset,
- **mobile app** – panel as full‑screen view, with the same background adapted to portrait.

### 2.2. UX priorities

1. **Find your gear fast** (via QR or mission list).
2. **Mark state fast** (especially “packed for return”).
3. **Stay out of the way** – UI should help, not fight the user.

---

## 3. Roles and access levels

Cargo Bay v1 assumes three basic roles:

- `warehouse` – person responsible for issuing and receiving gear.
- `crew` – people working on events / on board (technicians, operators, stage crew).
- `admin` – full access (roles, overrides, exceptional fixes).

### 3.1. warehouse

Can:

- create and edit **missions** (packing lists),
- assign assets (`asset_id`) to missions,
- mark gear as:
  - `sent` – left the warehouse / Cargo Bay,
  - `returned` – physically confirmed back in storage,
- view full asset details (including financial fields).

### 3.2. crew

Can:

- see **only missions they are assigned to**,
- within their mission:
  - browse the gear list,
  - open asset catalog cards (via click or QR),
  - mark gear as:
    - `packed` – packed after the event, ready to return,
- does **not** see financial fields (`purchase_price`, `vendor`, `warranty`, etc.).

### 3.3. admin

Can:

- do everything `warehouse` and `crew` can do,
- fix mistakes (e.g. wrong `packed` / `sent` state),
- manage user permissions.

---

## 4. Main views (web + mobile)

### 4.1. Login

**Goal:**  
No one sees any inventory data before logging in.

- Simple, dark screen aligned with AXV:
  - logo / name: `AXV Cargo Bay`,
  - fields: `email / username`, `password`,
  - button: `Log in`.
- In the mobile app we **always start from login**:
  - even if the user came from a QR scan,
  - after logging in, the app may keep a session (according to standard timeout rules).

After successful login:

- `crew` → lands on **“My mission”**,
- `warehouse` / `admin` → lands on **“Missions overview”** (or the last used screen).

---

### 4.2. Found device landing (web, anonymous user)

Public endpoint:

- `GET /a/{asset_id}`  
  e.g. `https://cargobay.axv.life/a/AXV-0123`

**When the user is not logged in:**

A very simple view, no inventory data:

- message:
  - “This device belongs to AXV / tomedia.studio”
- contact details:
  - email,
  - phone number (company),
- short text:
  - asking to contact if the device was found after an event.
- button:
  - `Log in to AXV Cargo Bay` (for crew),
  - after login the user is redirected back to `/a/{asset_id}` as the **asset card**.

No:

- price,
- location,
- any sensitive details.

---

### 4.3. My mission (crew view)

**Goal:**  
After login, `crew` immediately sees their **current mission**.

Elements:

- top bar:
  - system name: `AXV Cargo Bay`,
  - label: `My mission: Mein Schiff 2` (or `No active mission assigned`),
  - right side: user avatar / initials (`VoyTech`).

- gear list for the mission (table / list):

  | Packed | Asset ID   | Name                          | Container      |
  |--------|------------|-------------------------------|----------------|
  | ☐      | AXV-0123   | RME Fireface UCX II          | CB-AUDIO-01    |
  | ☑      | AXV-0450   | CB-AUDIO-01 – AV case        | –              |
  | ☐      | AXV-0071   | Router MikroTik RB5009UPr... | CB-NETWORK-01  |

- **Name** column:
  - link to the asset catalog card.
- **Packed** column:
  - checkbox used **after the event**, when packing gear for return,
  - detailed behaviour in section 5.

On mobile:

- list as stacked rows / cards, for example:

  ```
  [ ] AXV-0123
      RME Fireface UCX II
      CB-AUDIO-01
  ```

- tapping a row opens the asset card.

---

### 4.4. Missions overview (warehouse / admin)

View for `warehouse` / `admin`:

- list of missions:

  | Mission ID     | Name                     | From → To    | Status       |
  |----------------|--------------------------|--------------|-------------|
  | mission_MS2    | Mein Schiff 2 – contract | CORE → MS    | In progress |
  | mission_HQ     | HQ visit Dec 2025        | CORE → HQ    | Planned     |
  | mission_CONF01 | Small AV conference      | HQ → OTHER   | Completed   |

- capabilities:
  - filter by status (`Planned`, `In progress`, `Completed`),
  - open mission details.

Mission detail view:

- table similar to `My mission`, but with extra columns:

  | Sent | Packed | Returned | Asset ID | Name | Container | Notes |

- `warehouse` can:
  - set `Sent` (gear left the warehouse),
  - set `Returned` (gear is back),
- `Packed` is controlled by `crew`.

---

### 4.5. Asset card (catalog card)

View for **crew** (after login and coming from `/a/{asset_id}` or from a list).

Sections:

1. **Header:**
   - large `AXV-0123`,
   - name: `RME Fireface UCX II`,
   - small mission label if the asset belongs to the user’s current mission:
     - `Assigned to: mission_MS2`.

2. **Operational state (crew):**
   - `Status: in_use / spare / service / lost`,
   - `Packed for return: [ ☐ packed ]`:
     - checkbox visible and active if the asset is assigned to an active mission for this user.
   - info:
     - `Last packed by: <user> at <timestamp>` (if `packed = true`).

3. **Location and container:**
   - `Location: CORE / HQ / MS / OTHER`,
   - `Location detail: ...`,
   - `Container: CB-...` (if applicable).

4. **Technical data (crew):**
   - fields useful on site:
     - e.g. `Power`, `I/O summary`, short notes.

5. **“Details” tab (warehouse/admin only):**
   - `purchase_date`,
   - `purchase_price`,
   - `vendor`,
   - `warranty_until`,
   - other sensitive fields.

On mobile:

- vertical layout,
- `Mark as packed` button placed conveniently under the thumb.

---

## 5. States and interaction rules (sent / packed / returned)

Each entry in a packing list (backend table `packing_list_items`) has at least:

- `sent` (bool), `sent_by`, `sent_at`,
- `packed` (bool), `packed_by`, `packed_at`,
- optionally later: `returned` (bool), `returned_by`, `returned_at`.

### 5.1. Rules for `sent`

- Can only be set by:
  - `warehouse` (or `admin`).
- Meaning:
  - gear has been physically packed and left the warehouse / Cargo Bay,
  - responsibility for this step is on the warehouse side.

UI:

- in mission view:
  - `Sent` column with an editable checkbox for `warehouse`,
  - `crew` sees it read‑only.

### 5.2. Rules for `packed` (after the event)

- Can only be set by:
  - a user with `crew` role assigned to the mission (or `admin`).
- Meaning:
  - gear has been packed after the event for return (e.g. back in its case / container).

**Important rule:**

> Only the user who **last set `packed = true`** may undo this.

- If `packed = true` and `packed_by = user_X`:
  - only `user_X` sees an active checkbox and can uncheck it,
  - others see it as locked (e.g. tooltip: “Marked by user_X”).
- `admin` may always override in a dedicated, explicit flow.

UI:

- in `My mission`:
  - `crew` sees the “Packed” checkbox and can click it for their mission,
  - after clicking:
    - we update state immediately,
    - show a small toast: “Marked AXV-0123 as packed”.
- in the asset card:
  - the same state is visible,
  - `Mark as packed` / `Unpack` acts on the same fields.

### 5.3. Rules for `returned` (optional v1.1)

- Can only be set by:
  - `warehouse` / `admin`.
- Meaning:
  - gear physically returned to target storage after the mission.

UI (later):

- mission view:
  - `Returned` as an extra column,
  - used to “close” the mission.

---

## 6. QR and app integration

### 6.1. QR contents

Every label’s QR code encodes **one, consistent URL format**:

> `https://cargobay.axv.life/a/{asset_id}`  
> e.g. `https://cargobay.axv.life/a/AXV-0123`

- Web:
  - the browser opens this URL,
  - behaviour depends on login state:
    - not logged in → “Found device landing”,
    - logged in → asset card.
- Mobile app:
  - can register as a handler for `cargobay.axv.life` URLs,
  - intercepts the URL,
  - parses `{asset_id}` from the end,
  - fetches data from API (`GET /api/assets/{asset_id}`),
  - shows the asset card in native UI.

### 6.2. Labels – mini spec (UI‑related parts)

**Label type: Brother 12mm (mini)**

- QR: `https://cargobay.axv.life/a/{asset_id}`
- Layout:
  - centre: QR,
  - vertical text on one side:
    - `{asset_id}` (e.g. `AXV-0123`),
  - optionally vertical `AXV` on the other side, if legible.
- No logo / full company name – this is a pure ID tag.

**Label type: Brother 24mm**

- QR + `asset_id` + short owner text:
  - e.g. first line: `AXV-0123  |  AXV`,
  - second line: asset name or `container_id`.
- Intended for:
  - larger devices, cases, racks.

**Label type: A4 (48.5 × 25.4 mm)**

- QR,
- `AXV-0123` (larger font),
- `Owned by: AXV / tomedia.studio`,
- asset / case name,
- optional small `cargobay.axv.life/a/AXV-0123`.

---

## 7. Implementation roadmap (UI‑related)

1. **v1 – Backend + API (no UI yet):**
   - DB: `assets` + `packing_lists` + `packing_list_items`,
   - endpoints:
     - `GET /api/assets/{asset_id}`,
     - `GET /api/missions/{mission_id}`,
     - `GET /api/missions` (list),
     - basic `PATCH` for `sent` / `packed`.
2. **v1.1 – Web UI (MVP):**
   - `Login`,
   - `Found device landing`,
   - `My mission` (crew),
   - `Missions overview` (warehouse),
   - `Asset card`.
3. **v1.2 – UX polish + roles:**
   - field visibility (crew vs warehouse),
   - block uncheck of `packed` for other users,
   - simple event log (surfaced in the UI).
4. **v2 – Mobile app / PWA:**
   - native QR scanning,
   - same endpoints as web,
   - optional offline cache,
   - integration with AXV roles.

---

## 8. Publication notes

This document:

- **does not contain** passwords, tokens, keys or specific infrastructure addresses,
- describes **high‑level UX and flows** which are safe to publish,
- can safely live in `axv-docs` as:
  - `06_AXV_Cargo_Bay_UI_and_Flows.md`.

Sensitive details (password policies, server configs, detailed security logging) should be documented separately in a private:

- `AXV Internal Docs` repo / storage.
