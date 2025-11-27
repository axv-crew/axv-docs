# AXV Cargo Bay — Data Model & ID Spec (v1)

**Author:** Aster × Wojtek  
**Version:** v0.2 (EN)  
**Status:** Draft (public-safe)

---

## 1. Purpose & Scope

AXV Cargo Bay is the **central, non-public inventory system** for all AXV gear:

- CORE (home lab / core infrastructure),
- HQ (office / studio),
- mobile / on‑board missions (ships, remote events).

This document defines:

- the **canonical ID schemes** (`asset_id`, `container_id`),
- the **logical data model** (tables and key fields),
- constraints and assumptions for future API / UI work.

It is intentionally **backend‑oriented** and meant to be read together with:

- `06_AXV_Cargo_Bay_UI_and_Flows.md` – UX & flows,
- `AXV_Asset_Inventory_QR_Spec_v1.md` – QR / labelling spec.

---

## 2. ID Schemes

### 2.1. Asset ID (`asset_id`)

Each physical asset gets a **stable identifier**:

- Format: `AXV-XXXX`
  - `AXV` – constant prefix,
  - `XXXX` – zero‑padded integer, starting from `0001` (e.g. `AXV-0001`, `AXV-0123`).
- The identifier is:
  - **globally unique** within AXV,
  - **stable over time** (never reused, never reassigned),
  - independent of:
    - location,
    - owner / project,
    - mission.

Rules:

- Once assigned, an `asset_id` must **never** be reused, even if:
  - the asset is sold,
  - retired,
  - or lost.
- Retired assets keep their IDs for historical purposes (logs, missions, etc.).

Implementation notes:

- The numeric part can be implemented as:
  - an auto‑increment primary key stored separately and formatted as `AXV-%04d`,
  - or an integer field plus a generated `asset_id` string.
- For future‑proofing, the system **must not** assume a fixed 4‑digit length:
  - UI formatting may keep 4 digits initially,
  - internally we treat the numeric ID as an integer `>= 1`.

---

### 2.2. Container ID (`container_id`)

Containers are physical or logical units used to **group assets**:

- crates / flight cases,
- organiser boxes,
- cable organisers, etc.

Format (recommended):

- `CB-{GROUP}-{NN}`

Where:

- `CB` – prefix for “Cargo Box / Container”,
- `{GROUP}` – short code (2–5 letters) for the container family / purpose:
  - `DIY`, `CABLE`, `AUDIO`, `NET`, etc.
- `{NN}` – 2‑digit, zero‑padded index within that group:
  - `01`, `02`, `03`, …

Examples:

- `CB-DIY-01` – DIY box #1,
- `CB-DIY-02` – DIY box #2,
- `CB-CABLE-01` – cable organiser #1,
- `CB-AUDIO-01` – small AV conference case.

Rules:

- Container IDs are **stable** and **unique** within AXV.
- A container can be:
  - a first‑class record in the `containers` table,
  - referenced from multiple assets via the `container_id` field.

---

### 2.3. Mission ID (`mission_id`)

Missions represent **trips / events / jobs** which require a packing list.

Format (suggested, not strictly enforced):

- `mission_{short_code}`

Examples:

- `mission_MS2` – Mein Schiff 2 contract,
- `mission_HQ_2025_12` – HQ visit Dec 2025,
- `mission_CONF01` – small AV conference #1.

Semantics and flows for missions are described in  
`06_AXV_Cargo_Bay_UI_and_Flows.md`.

---

## 3. Core Concepts

### 3.1. Asset

A **physical item** tracked by Cargo Bay:

- can be packed into containers,
- can move between locations (CORE / HQ / MS / OTHER),
- can participate in missions (packing lists),
- may have financial / vendor / warranty metadata.

Examples:

- network gear (routers, APs),
- audio interfaces, microphones,
- lights, stands, cables,
- DIY electronics (dev boards, sensors), where it makes sense.

### 3.2. Container

A **physical grouping unit** for assets:

- boxes, cases, organisers,
- may itself be an asset (e.g. expensive flight case),
- but is also referenced via `container_id` so that:
  - UI can show “what’s inside CB-AUDIO-01”.

### 3.3. Mission

A **context** in which gear moves together:

- defined by a time frame, locations and purpose,
- has a packing list of assets (and optionally non‑AXV items),
- tracks states like `sent`, `packed`, `returned` per item.

---

## 4. Data Model Overview

v1 focuses on **four primary tables**:

1. `assets` – inventory of all AXV gear.
2. `containers` – registry of named containers / boxes (optional but recommended).
3. `missions` – missions / trips / events.
4. `mission_items` – packing list entries for missions.

v2+ may add:

- `asset_events` – timeline of state changes per asset,
- `mission_events` – timeline for missions.

Below is a logical schema for v1.

---

## 5. Table: `assets`

Canonical inventory of all assets.

**Recommended fields (logical model):**

- `id` – integer primary key (surrogate).
- `asset_id` – string, unique, e.g. `AXV-0123`.
- `name` – human‑readable name, e.g. `RME Fireface UCX II`.
- `model` – model code / short model name (optional).
- `brand` – manufacturer / brand (optional).
- `category_group` – high‑level grouping, e.g. `AUDIO`, `VIDEO`, `NETWORK`, `DIY`, `COMPUTER`, `POWER`, etc.
- `category` – more specific category, e.g. `audio_interface`, `router`, `led_light`, `tool`, `sensor`.
- `item_type` – e.g. `asset`, `container`, `consumable` (for future differentiation if needed).
- `location` – canonical location: `CORE`, `HQ`, `MS`, `OTHER`.
- `location_detail` – free‑form detail, e.g. `Domek – rack`, `Ruderka – desk`, `MS2 – cabin`.
- `container_id` – optional, e.g. `CB-DIY-01`, `CB-AUDIO-01`.
- `status` – operational status (see below).
- `serial_number` – optional serial / unique hardware identifier.
- `notes` – free text.

Optional financial / vendor fields:

- `purchase_date` – date.
- `purchase_price` – decimal.
- `currency` – ISO 4217 code, e.g. `PLN`, `EUR`.
- `vendor` – where it was bought (shop / supplier).
- `warranty_until` – date.

System metadata:

- `created_at` – timestamp.
- `updated_at` – timestamp.
- `retired_at` – nullable timestamp, if the asset is retired.

### 5.1. Status values

`status` should be an enum or constrained string, e.g.:

- `in_use` – currently deployed / actively used.
- `spare` – available in storage.
- `service` – under repair / in service.
- `lost` – missing, presumed lost.
- `retired` – formally retired (sold / recycled).

Retired assets should keep their records and IDs.

### 5.2. Example DDL (MariaDB)

```sql
CREATE TABLE assets (
  id              INT UNSIGNED NOT NULL AUTO_INCREMENT,
  asset_id        VARCHAR(16) NOT NULL,
  name            VARCHAR(255) NOT NULL,
  model           VARCHAR(128),
  brand           VARCHAR(128),
  category_group  VARCHAR(64),
  category        VARCHAR(64),
  item_type       VARCHAR(32) DEFAULT 'asset',
  location        VARCHAR(16) NOT NULL,
  location_detail VARCHAR(255),
  container_id    VARCHAR(32),
  status          VARCHAR(16) NOT NULL DEFAULT 'spare',
  serial_number   VARCHAR(128),
  notes           TEXT,
  purchase_date   DATE,
  purchase_price  DECIMAL(12,2),
  currency        CHAR(3),
  vendor          VARCHAR(255),
  warranty_until  DATE,
  created_at      DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at      DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  retired_at      DATETIME NULL,
  PRIMARY KEY (id),
  UNIQUE KEY uq_asset_id (asset_id),
  KEY idx_location (location),
  KEY idx_container (container_id),
  KEY idx_category (category_group, category)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

## 6. Table: `containers` (optional v1, recommended)

Registers all named containers / boxes.

Fields:

- `id` – integer primary key.
- `container_id` – string, e.g. `CB-DIY-01`, unique.
- `name` – human name, e.g. `DIY box #1`, `Cable organiser #1`.
- `description` – optional longer description (contents / purpose).
- `location` – `CORE`, `HQ`, `MS`, `OTHER`.
- `location_detail` – free text.
- `notes` – free text.
- `created_at`, `updated_at`.

Example DDL:

```sql
CREATE TABLE containers (
  id              INT UNSIGNED NOT NULL AUTO_INCREMENT,
  container_id    VARCHAR(32) NOT NULL,
  name            VARCHAR(255) NOT NULL,
  description     TEXT,
  location        VARCHAR(16) NOT NULL,
  location_detail VARCHAR(255),
  notes           TEXT,
  created_at      DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at      DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (id),
  UNIQUE KEY uq_container_id (container_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

`assets.container_id` refers logically to `containers.container_id` (no strict FK needed at start, but recommended in later phases).

---

## 7. Table: `missions`

Represents a mission / event / trip that needs gear.

Fields:

- `id` – integer primary key.
- `mission_id` – string, e.g. `mission_MS2`, unique.
- `name` – human-readable name, e.g. `Mein Schiff 2 – contract`.
- `description` – optional description.
- `origin_location` – e.g. `CORE`, `HQ`.
- `target_location` – e.g. `MS`, `OTHER`.
- `start_date` – optional.
- `end_date` – optional.
- `status` – e.g. `planned`, `in_progress`, `completed`, `cancelled`.
- `created_at`, `updated_at`.

Example DDL:

```sql
CREATE TABLE missions (
  id              INT UNSIGNED NOT NULL AUTO_INCREMENT,
  mission_id      VARCHAR(64) NOT NULL,
  name            VARCHAR(255) NOT NULL,
  description     TEXT,
  origin_location VARCHAR(16),
  target_location VARCHAR(16),
  start_date      DATE,
  end_date        DATE,
  status          VARCHAR(16) NOT NULL DEFAULT 'planned',
  created_at      DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at      DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (id),
  UNIQUE KEY uq_mission_id (mission_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

## 8. Table: `mission_items` (packing list entries)

Links missions with assets (and optionally non‑AXV items).

Fields:

- `id` – integer primary key.
- `mission_id` – FK to `missions.mission_id` (or integer FK to `missions.id`).
- `asset_id` – FK to `assets.asset_id` (nullable for non‑AXV items).
- `external_label` – optional free text for non‑AXV items (e.g. “personal clothes”, “documents”).
- `container_id` – optional hint container for packing (e.g. `CB-AUDIO-01`).
- `quantity` – integer, default 1.
- `sent` – bool, gear left the warehouse.
- `sent_by`, `sent_at`.
- `packed` – bool, packed after event for return.
- `packed_by`, `packed_at`.
- `returned` – bool, returned to storage (optional v1.1).
- `returned_by`, `returned_at`.
- `notes` – free text.

Example DDL:

```sql
CREATE TABLE mission_items (
  id            INT UNSIGNED NOT NULL AUTO_INCREMENT,
  mission_id    VARCHAR(64) NOT NULL,
  asset_id      VARCHAR(16),
  external_label VARCHAR(255),
  container_id  VARCHAR(32),
  quantity      INT NOT NULL DEFAULT 1,
  sent          TINYINT(1) NOT NULL DEFAULT 0,
  sent_by       VARCHAR(64),
  sent_at       DATETIME,
  packed        TINYINT(1) NOT NULL DEFAULT 0,
  packed_by     VARCHAR(64),
  packed_at     DATETIME,
  returned      TINYINT(1) NOT NULL DEFAULT 0,
  returned_by   VARCHAR(64),
  returned_at   DATETIME,
  notes         TEXT,
  created_at    DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at    DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (id),
  KEY idx_mission (mission_id),
  KEY idx_asset (asset_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

Notes:

- For non‑AXV items, `asset_id` may be `NULL` and only `external_label` is used.
- Behaviour and UI rules for `sent` / `packed` / `returned` are described in `06_AXV_Cargo_Bay_UI_and_Flows.md`.

---

## 9. Future: `asset_events` (v2+)

Not required for v1, but useful later for history / auditing.

Possible fields:

- `id` – integer primary key.
- `asset_id` – FK to `assets.asset_id`.
- `event_type` – e.g. `status_change`, `location_change`, `mission_assignment`, `mission_unassignment`.
- `payload` – JSON with event details.
- `created_at` – timestamp.
- `created_by` – user identifier.

This table would allow reconstructing the lifecycle of each asset over time.

---

## 10. Mapping from Google Sheets (current inventory)

The current Google Sheets inventory for CORE + HQ should map roughly as:

- `asset_id` → `assets.asset_id`
- `name` → `assets.name`
- `model` → `assets.model`
- `brand` → `assets.brand`
- `category_group` → `assets.category_group`
- `category` → `assets.category`
- `item_type` → `assets.item_type`
- `location` → `assets.location`
- `location_detail` → `assets.location_detail`
- `container` → `assets.container_id`
- `status` → `assets.status`
- `notes` → `assets.notes`
- `vendor` → `assets.vendor`
- `purchase_price` → `assets.purchase_price`
- `purchase_date` → `assets.purchase_date`
- `warranty_until` → `assets.warranty_until`
- any existing serial numbers → `assets.serial_number`

This mapping can be implemented by:

1. Exporting the sheet to CSV.
2. Using a small import script (Python / Node / etc.) to:
   - normalise values (e.g. `CORE` vs `Core`),
   - validate enums (`location`, `status`),
   - insert rows into `assets`.

---

## 11. Publication & security notes

This spec:

- contains **no passwords, tokens, IP addresses or secrets**,
- defines logical structure and example DDL only,
- can safely live in the public `axv-docs` repository as:

  `05_AXV_Cargo_Bay_Spec.md`

Operational security (network topology, DB credentials, backups, hardening) should be documented separately in private, internal documentation.
