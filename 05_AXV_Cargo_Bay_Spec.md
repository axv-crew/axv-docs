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
- case „AXV-Live-Small-Conference”.

Reprezentacja:

- na v1 (Sheets): `container` to **pole tekstowe** w rekordzie assetu:
  - np. `DIY-BOX-01`, `CASE-AV-SMALL-01`.
- w v2 (DB): opcja 1:
  - `container` jako tekst,  
  opcja 2 (później):
  - osobna tabela lub self-FK (asset jako container asset).

Na start przyjmujemy **prostą wersję tekstową**.

### 3.3. `mission` / packing list

Konkretne przedsięwzięcie wymagające spakowania sprzętu, np.:

- `mission_HQ` – wyjazd do HQ,
- `mission_MS2` – kontrakt na Mein Schiff 2.

Packing lista:

- ma swój identyfikator (`mission_id`),
- składa się z wierszy:
  - `asset_id` **lub** `extra_name` (ubernia, dokumenty itd.),
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

Formuły (v1, logika):

- `item_type`:
  - jeśli `asset_id` pasuje do `AXV-\d{4}` → `asset`,
  - inaczej → `extra` (na razie wystarczy).
- `name`:
  - jeśli `item_type = asset` → `VLOOKUP(asset_id, CargoBay!...)`,
  - inaczej → `extra_name`.

To już w praktyce mamy zaczęte.

---

## 6. Model danych v2 — MariaDB (`assets`)

Docelowa tabela w MariaDB (pierwsza wersja):

```text
Table: assets

id                  INT AUTO_INCREMENT PRIMARY KEY
asset_id            VARCHAR(32)  NOT NULL UNIQUE
name                VARCHAR(255) NOT NULL
description         TEXT         NULL

location            ENUM('CORE','HQ','MS','OTHER') NOT NULL DEFAULT 'CORE'
location_detail     VARCHAR(255) NULL

category_group      VARCHAR(64)  NULL
category            VARCHAR(64)  NULL

item_type           ENUM('asset','consumable','container','other')
                    NOT NULL DEFAULT 'asset'

status              ENUM('in_use','spare','service','retired','lost')
                    NOT NULL DEFAULT 'spare'

container           VARCHAR(64)  NULL
-- (opcjonalnie w przyszłości: container_id INT, FK do assets.id)

reservation_status  ENUM('free','reserved')
                    NOT NULL DEFAULT 'free'

priority            ENUM('P1','P2','P3','P4') NULL

serial_number       VARCHAR(128) NULL
purchase_date       DATE         NULL
purchase_price      DECIMAL(10,2) NULL
vendor              VARCHAR(255) NULL
warranty_until      DATE         NULL

tags                VARCHAR(255) NULL

created_at          DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
updated_at          DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
                    ON UPDATE CURRENT_TIMESTAMP
