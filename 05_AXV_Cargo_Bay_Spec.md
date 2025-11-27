# AXV Cargo Bay — Spec (v1 Data Model & Flows)

**Author:** Aster × Wojtek  
**Version:** v0.1  
**Status:** Draft (living, will evolve with system)

---

## 1. Cel dokumentu

Ten dokument opisuje **AXV Cargo Bay** jako system:

- jakie są **pojęcia i ID** (asset, container, mission),
- jak wygląda **model danych v1** (Google Sheets),
- jaki jest docelowy **model MariaDB v2**,
- jak Cargo Bay współpracuje z:
  - packing listami (misje),
  - QR / etykietami,
  - resztą AXV (HUB, Atlas, n8n).

Docelowo Cargo Bay jest **jednym źródłem prawdy o sprzęcie AXV**  
(Core + HQ + mobilne / morskie setupy).

---

## 2. Zakres i wersje

### v1 – Google Sheets (obecny stan)

- Główny arkusz inwentaryzacji:
  - Core + HQ + inne lokalizacje.
- Kolumny z data validation (listy):
  - `location`, `item_type`, `category`, `category_group`, `status`, `reservation_status` itd.
- Packing listy jako osobne zakładki:
  - `mission_HQ`, `mission_MS2`, itd.
- `asset_id` już istnieje i jest niezmienny.

### v2 – MariaDB + API

- Główna tabela `assets` + pomocnicze:
  - `packing_lists`, `packing_list_items`,
  - w przyszłości `asset_events`.
- Backend (AXV Gateway / inne API) jako główny interfejs:
  - UI web,
  - mobilna apka,
  - n8n.

### v3 – QR, Web UI, Mobile

- Endpointy:
  - `/a/{asset_id}` – karta sprzętu,
  - `/missions/{mission_id}` – packing listy.
- Autoryzacja:
  - logowanie / token (non-public system).
- Apka mobilna:
  - scanning QR,
  - podgląd,
  - szybkie zmiany statusu.

Ten dokument skupia się głównie na **v1 (Sheets)** + **bazowym modelu MariaDB (v2)**.

---

## 3. Pojęcia i ID

### 3.1. `asset`

Najdrobniejsza jednostka, którą śledzimy osobno. Może to być:

- pojedyncze urządzenie (kamera, mikser, router),
- pojedynczy kabel / adapter (jeśli jest istotny),
- moduł DIY (kit, płytka),
- element większego zestawu, który *mimo to* chcemy mieć jako osobną pozycję.

Każdy `asset` ma:

- niezmienny `asset_id`,
- nazwę `name`,
- lokalizację (`location` + `location_detail`),
- typ (`item_type = asset`),
- status (`status`, `reservation_status`),
- opcjonalnie:
  - `container`,
  - `category`, `category_group`,
  - informacje zakupowe / serwisowe.

### 3.2. `container`

Logiczny lub fizyczny „pojemnik”, który może zawierać wiele assetów, np.:

- organizer z DIY rzeczami,
- walizka z przystosowanym setupem,
- skrzynia AV,
- case `AXV-Live-Small-Conference`.

Reprezentacja:

- w v1 (Sheets): `container` to **pole tekstowe** w rekordzie assetu  
  np. `DIY-BOX-01`, `CASE-AV-SMALL-01`.
- w v2 (DB): opcja 1 – `container` jako tekst,  
  opcja 2 (później) – osobna tabela lub self-FK (asset jako container asset).

Na start przyjmujemy **prostą wersję tekstową**.

### 3.3. `mission` / packing list

Konkretne przedsięwzięcie wymagające spakowania sprzętu, np.:

- `mission_HQ` – wyjazd do HQ,
- `mission_MS2` – kontrakt na Mein Schiff 2.

Packing lista:

- ma swój identyfikator (`mission_id`),
- składa się z wierszy:
  - `asset_id` **lub** `extra_name` (ubrania, dokumenty itd.),
  - meta:
    - `priority`,
    - `from_location` / `to_location`,
    - `qty`,
    - `packed`,
    - `notes`.

### 3.4. `asset_id`

Stały identyfikator, np.:

- `AXV-0001`, `AXV-0002`, …

Właściwości:

- nadawany **raz**,
- niezmienny w czasie,
- nie zależy od lokalizacji,
- używany w:
  - Cargo Bay,
  - packing listach,
  - QR etykietach,
  - API / mobilce.

Format (v1):

- prefix `AXV-`,
- 4 cyfry (`0001`–`9999`) – do rozszerzenia w przyszłości.

---

## 4. Model danych v1 — Google Sheets

### 4.1. Główny arkusz inwentaryzacji

Minimalny zestaw kolumn (to, co mamy + lekko uporządkowane):

- `asset_id` – **unikatowy**, np. `AXV-0123`.
- `name` – czytelna nazwa rzeczy.
- `description` (opcjonalne) – doprecyzowanie.
- `location` – główna lokalizacja:
  - wartości: `CORE`, `HQ`, `MS`, `OTHER`.
- `location_detail` – szczegóły:
  - np. `Domek – biurko`, `Ruderka – szuflada 3`, `Kabina MS2`.
- `category_group` – grupa wyższego poziomu:
  - np. `DIY`, `VIDEO`, `AUDIO`, `COMPUTERS`, `NETWORK`, `POWER`, `TOOLS`.
- `category` – podkategoria w ramach grupy:
  - np. w `AUDIO`: `MIC`, `INTERFACE`, `HEADPHONES`,
  - w `NETWORK`: `ROUTER`, `SWITCH`, `ACCESS_POINT`, `CABLE`.
- `item_type` – typ rekordów:
  - `asset` – sprzęt, który śledzimy,
  - `consumable` – rzeczy zużywalne (taśmy, baterie),
  - `container` – same kontenery (case/box) jeśli chcesz je śledzić osobno,
  - `other` – fallback.
- `status` – stan:
  - propozycja: `in_use`, `spare`, `service`, `retired`, `lost`.
- `container` – nazwa/logiczny ID pojemnika:
  - np. `DIY-BOX-01`, `CASE-AV-SMALL-01`, albo puste.
- `reservation_status` – rezerwacja:
  - `free`, `reserved`.
- `priority` (opcjonalne) – jak ważny asset (np. `P1`, `P2`, `P3`).
- `notes` – luźne notatki.

Dodatkowe opcjonalne pola (do wprowadzenia kiedy będziesz miał siłę):

- `serial_number`
- `purchase_date`
- `purchase_price`
- `vendor`
- `warranty_until`
- `tags` – słowa kluczowe (`"rack, live, ms2"`).

### 4.2. Reguły data validation (v1)

Dla spójności:

- `location`: lista: `CORE, HQ, MS, OTHER`.
- `item_type`: lista: `asset, consumable, container, other`.
- `status`: lista: `in_use, spare, service, retired, lost`.
- `reservation_status`: lista: `free, reserved`.
- `category_group`: lista rozszerzalna (DIY, VIDEO, AUDIO, COMPUTERS, NETWORK, POWER, TOOLS…).
- `category`: powiązana logicznie z `category_group` (może być luźno na v1, doprecyzujemy v2).

Dodatkowo:

- `asset_id`:
  - format: `AXV-` + 4 cyfry (można weryfikować regexem albo ręcznie),
  - unikalność pilnowana **na razie mentalnie** + przez filtr/warunki.

---

## 5. Packing listy v1 — arkusze `mission_*`

Każda misja ma osobną zakładkę, np.:  
`mission_HQ`, `mission_MS2`, `mission_Conference_small`.

Proponowany układ kolumn (odpowiada temu, co już mamy):

- `priority` – `P1`, `P2`, `P3`, `P4` (np. P1 = must have).
- `item_type` – `asset` / `extra` / `container` (można wyliczać formułą).
- `asset_id` – jeśli dotyczy assetu z Cargo Bay (`AXV-XXXX`).
- `extra_name` – jeśli to rzecz spoza Cargo Bay (ubrania, dokumenty, jedzenie).
- `name` – kolumna obliczeniowa:
  - jeśli `item_type = asset` → lookup z Cargo Bay (`name`),
  - w przeciwnym wypadku → `extra_name`.
- `from_location` – skąd bierzemy (np. `CORE`, `HQ`).
- `to_location` – dokąd jedzie (np. `HQ`, `MS2`).
- `qty` – ilość.
- `packed` – checkbox / `YES/NO`.
- `notes` – komentarze.

Logika formuł (v1):

- `item_type`:
  - jeśli `asset_id` pasuje do `AXV-\d{4}` → `asset`,
  - inaczej → `extra`.
- `name`:
  - jeśli `item_type = asset` → `VLOOKUP(asset_id, CargoBay!...)`,
  - inaczej → `extra_name`.

---

## 6. Model danych v2 — MariaDB (`assets`)

Docelowa tabela w MariaDB (pierwsza wersja):

```sql
CREATE TABLE assets (
  id                 INT AUTO_INCREMENT PRIMARY KEY,
  asset_id           VARCHAR(32)  NOT NULL UNIQUE,
  name               VARCHAR(255) NOT NULL,
  description        TEXT         NULL,

  location           ENUM('CORE','HQ','MS','OTHER') NOT NULL DEFAULT 'CORE',
  location_detail    VARCHAR(255) NULL,

  category_group     VARCHAR(64)  NULL,
  category           VARCHAR(64)  NULL,

  item_type          ENUM('asset','consumable','container','other')
                     NOT NULL DEFAULT 'asset',

  status             ENUM('in_use','spare','service','retired','lost')
                     NOT NULL DEFAULT 'spare',

  container          VARCHAR(64)  NULL,
  -- opcjonalnie w przyszłości: container_id INT, FK do assets.id

  reservation_status ENUM('free','reserved')
                     NOT NULL DEFAULT 'free',

  priority           ENUM('P1','P2','P3','P4') NULL,

  serial_number      VARCHAR(128) NULL,
  purchase_date      DATE         NULL,
  purchase_price     DECIMAL(10,2) NULL,
  vendor             VARCHAR(255) NULL,
  warranty_until     DATE         NULL,

  tags               VARCHAR(255) NULL,

  created_at         DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at         DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
                     ON UPDATE CURRENT_TIMESTAMP
);
```

Założenia:

- `asset_id` jest głównym, zewnętrznym ID:
  - używany w QR, API, frontach.
- `id` (INT) jest **wewnętrznym** PK (wygodnym dla relacji).
- `container` na v2 dalej jako tekst:
  - dopiero później możemy wprowadzić relację:
    - asset jako `container` dla innych (self-FK).

---

## 7. Packing listy v2 — `packing_lists` i `packing_list_items`

Propozycja v2 (DB):

```sql
CREATE TABLE packing_lists (
  id             INT AUTO_INCREMENT PRIMARY KEY,
  mission_id     VARCHAR(64) NOT NULL UNIQUE,
  name           VARCHAR(255) NOT NULL,
  description    TEXT NULL,

  from_location  ENUM('CORE','HQ','MS','OTHER') NULL,
  to_location    ENUM('CORE','HQ','MS','OTHER') NULL,

  created_at     DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at     DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
                 ON UPDATE CURRENT_TIMESTAMP
);
```

```sql
CREATE TABLE packing_list_items (
  id                INT AUTO_INCREMENT PRIMARY KEY,
  packing_list_id   INT NOT NULL,
  CONSTRAINT fk_packing_list
    FOREIGN KEY (packing_list_id) REFERENCES packing_lists(id),

  item_type         ENUM('asset','extra','container') NOT NULL,

  asset_id          VARCHAR(32)   NULL,  -- jeśli item_type = 'asset'
  extra_name        VARCHAR(255)  NULL,  -- jeśli item_type = 'extra'

  priority          ENUM('P1','P2','P3','P4') NULL,
  qty               INT NOT NULL DEFAULT 1,

  from_location     ENUM('CORE','HQ','MS','OTHER') NULL,
  to_location       ENUM('CORE','HQ','MS','OTHER') NULL,

  packed            TINYINT(1) NOT NULL DEFAULT 0,
  notes             TEXT NULL
);
```

Logika:

- `item_type` decyduje, które pola są używane:
  - `asset` → `asset_id` (lookup do `assets`),
  - `extra` → `extra_name`,
  - `container` → na razie może używać `extra_name`, później `container_id`.
- `mission_id` może odpowiadać zakładce `mission_*` w v1.

---

## 8. QR, etykiety i URL

Każdy asset ma:

- tekst: `asset_id` (np. `AXV-0123`),
- QR kod prowadzący do:
  - `https://cargobay.axv.life/a/AXV-0123`.

Minimalny endpoint (v2):

- `GET /a/{asset_id}`:
  - zwraca stronę:
    - `asset_id`,
    - `name`,
    - `location`, `location_detail`,
    - `status`, `reservation_status`,
    - `container`,
    - podstawowe meta (`category_group`, `category`, `notes`),
  - dostępny **tylko po zalogowaniu / autoryzacji** (non-public).

W przyszłości:

- `GET /api/assets/{asset_id}` – JSON dla mobilki / innych systemów.

---

## 9. Integracja z resztą AXV

### 9.1. MASTER HUB

Cargo Bay pojawia się w MASTER HUB jako:

- osobna pozycja typu `System / Inventory`,
- z linkami:
  - do arkusza v1 (Sheets),
  - do specy (ten plik),
  - do przyszłego UI / API.

### 9.2. AXV @ Sea

Packing listy są kluczowe dla misji @Sea:

- `mission_MS2` → co trzeba zabrać na statek,
- filtr po `location = CORE/HQ` + `to_location = MS`.

W trybie @Sea:

- Cargo Bay UI powinno mieć **lekki widok**:
  - lista rzeczy na aktualną misję,
  - status (`packed`, `reserved`),
  - podgląd kart assetów.

### 9.3. n8n / automatyzacje

Przyszłe flow (przykłady):

- sync arkusza v1 → MariaDB (import),
- codzienny eksport stanu `assets`:
  - do backupu,
  - do Atlas (RAG),
- powiadomienia:
  - o assetach ze statusem `service`,
  - o assetach oznaczonych jako `lost`.

---

## 10. Dalsza ewolucja

Ten dokument jest punktem startowym. Następne iteracje mogą:

- doprecyzować:
  - słowniki `category_group` / `category`,
  - reguły użycia `container`,
  - statusy (`status`, `reservation_status`),
- opisać:
  - migrację z arkusza v1 do MariaDB v2,
  - szczegóły API (`/api/assets`, `/api/missions`, `/api/packing-lists`),
  - powiązania z Atlas (RAG po inventory).

Powinien być czytany razem z:

- `01_AXV_Manifest_north star.md`
- `02_AXV_Operating_Modes.md`
- `03_AXV_Crew_and_Roles.md`
- `04_AXV_Projects_and_K_Modules.md`

Razem tworzą one **szkielet AXV Cargo Bay i całej platformy**.
