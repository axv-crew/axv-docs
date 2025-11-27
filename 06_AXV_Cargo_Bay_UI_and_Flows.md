# AXV Cargo Bay — UI & Flows (v1)

**Author:** Aster × Wojtek  
**Version:** v0.1  
**Status:** Draft (public‑safe, implementation details can live in a separate internal doc)

---

## 1. Scope

Ten dokument opisuje **zachowanie i UI** systemu **AXV Cargo Bay** z perspektywy użytkownika:

- jakie są **role** (warehouse / crew / admin),
- jakie są **główne widoki** (web + mobile),
- jak działa **przepływ sprzętu** w kontekście misji (packing lists),
- jak zachowuje się UI po **zeskanowaniu QR**.

**Model danych i schema DB** są opisane w:  
`05_AXV_Cargo_Bay_Spec.md`.

Ten plik skupia się na:

- UX,
- ekranach,
- stanach i interakcjach.

Jest napisany tak, aby mógł spokojnie żyć w publicznym repo – bez wrażliwych szczegółów implementacyjnych.

---

## 2. Założenia UX & brand

### 2.1. Ogólny klimat

- Cargo Bay jest **częścią AXV**, nie oddzielnym brandem:
  - korzysta z tej samej **kolorystyki i typografii**,
  - ma ten sam „nocny” klimat (ciemne tło, ocean, światła).
- Tło:
  - ciemny, nocny ocean / zatoka,
  - w oddali **port / dock** z delikatnymi światłami (mogą lekko „żyć”, ale bez ciężkich animacji).
- Warstwa UI:
  - **półprzezroczysta „konsola” / panel** na tle oceanu,
  - układ preferujący **szybkość, prostotę i czytelność**,
  - animacje:
    - szybkie, dyskretne (fade/slide),
    - brak „ciężkich” efektów.

Ten sam koncept działa:

- w **web UI** – panel w centrum / lekko przesunięty,
- w **mobile app** – panel jako pełnoekranowy widok z tym samym tłem (dostosowany do pionu).

### 2.2. Priorytety UX

1. **Szybko znaleźć swój sprzęt** (po QR, po liście misji).
2. **Szybko oznaczyć stan** (szczególnie „spakowany na powrót”).
3. **Nie przeszkadzać** – UI ma pomagać, nie męczyć.

---

## 3. Role i poziomy dostępu

W Cargo Bay zakładamy trzy podstawowe role (v1):

- `warehouse` – magazyn / osoba odpowiedzialna za wydawanie i przyjmowanie sprzętu.
- `crew` – osoby pracujące na evencie / statku (technicy, realizatorzy, stage crew).
- `admin` – pełny dostęp (zarządzanie rolami, poprawki wyjątków).

### 3.1. warehouse

Może:

- tworzyć i edytować **misje** (packing lists),
- przypisywać assety (`asset_id`) do misji,
- oznaczać sprzęt jako:
  - `sent` – wysłany z Cargo Bay (wyjechał z magazynu),
  - `returned` – potwierdzone, że fizycznie wrócił.
- przeglądać pełne dane assetu (łącznie z polami finansowymi).

### 3.2. crew

Może:

- zobaczyć **tylko misje, do których jest przypisany**,
- na swojej misji:
  - przeglądać listę sprzętu,
  - otwierać karty katalogowe (po kliknięciu / po QR),
  - oznaczać sprzęt jako:
    - `packed` – spakowany po evencie, gotowy do powrotu,
- nie widzi pól finansowych (`purchase_price`, `vendor`, `warranty` itp.).

### 3.3. admin

Może:

- wszystko, co `warehouse` i `crew`,
- naprawiać błędy (np. korekta niewłaściwie oznaczonego `packed` / `sent`),
- zarządzać uprawnieniami użytkowników.

---

## 4. Główne widoki (web + mobile)

### 4.1. Logowanie

**Cel:**  
Zanim ktoś zobaczy jakiekolwiek dane inventory, musi się zalogować.

- Ekran prosty, ciemny, spójny z AXV:
  - logo / nazwa `AXV Cargo Bay`,
  - pola: `email / username`, `password`,
  - przycisk `Log in`.
- W apce (mobile) **również zawsze start od logowania**:
  - nawet jeśli użytkownik przyszedł z QR,
  - po zalogowaniu apka może utrzymywać sesję (wg standardowego timeoutu).

Po poprawnym logowaniu użytkownik:

- `crew` → trafia na widok **„My mission”**,
- `warehouse` / `admin` → na widok **„Missions overview”** (lub ostatnio używany).

---

### 4.2. Found device landing (web, user anonimowy)

Endpoint publiczny:

- `GET /a/{asset_id}`  
  np. `https://cargobay.axv.life/a/AXV-0123`

**Gdy użytkownik nie jest zalogowany:**

Widok bardzo prosty, bez danych inventory:

- komunikat:
  - „This device belongs to AXV / tomedia.studio”
- dane kontaktowe:
  - e-mail,
  - numer telefonu (firma),
- krótki tekst:
  - prośba o kontakt, jeśli urządzenie zostało znalezione po evencie.
- przycisk:
  - `Log in to AXV Cargo Bay` (dla załogi),
  - po zalogowaniu następuje powrót do `/a/{asset_id}` jako **karta assetu**.

Brak:

- ceny,
- lokalizacji,
- jakichkolwiek wrażliwych szczegółów.

---

### 4.3. My mission (crew view)

**Cel:**  
Po zalogowaniu `crew` → od razu widok **swojego aktualnego eventu**.

Elementy:

- górny pasek:
  - nazwa systemu: `AXV Cargo Bay`,
  - label: `My mission: Mein Schiff 2` (lub `No active mission assigned`),
  - po prawej: inicjały / avatar użytkownika (`VoyTech`).

- lista sprzętu na misji (tabela / lista):

  | Packed | Asset ID   | Name                          | Container      |
  |--------|------------|-------------------------------|----------------|
  | ☐      | AXV-0123   | RME Fireface UCX II          | CB-AUDIO-01    |
  | ☑      | AXV-0450   | CB-AUDIO-01 – AV case        | –              |
  | ☐      | AXV-0071   | Router MikroTik RB5009UPr... | CB-NETWORK-01  |

- kolumna **Name**:
  - link do karty katalogowej assetu.
- kolumna **Packed**:
  - checkbox dostępny **po evencie** (kiedy pakujemy sprzęt do powrotu),
  - logika szczegółowa w rozdziale 5.

Na mobile:

- lista w formie kart / wierszy, np.:

  ```
  [ ] AXV-0123
      RME Fireface UCX II
      CB-AUDIO-01
  ```

- tapnięcie w wiersz → otwarcie karty assetu.

---

### 4.4. Missions overview (warehouse / admin)

Widok dla `warehouse` / `admin`:

- lista misji:

  | Mission ID     | Name                     | From → To    | Status      |
  |----------------|--------------------------|--------------|------------|
  | mission_MS2    | Mein Schiff 2 – contract | CORE → MS    | In progress|
  | mission_HQ     | HQ visit Dec 2025        | CORE → HQ    | Planned    |
  | mission_CONF01 | Small AV conference      | HQ → OTHER   | Completed  |

- możliwość:
  - filtrowania po statusie (`Planned`, `In progress`, `Completed`),
  - otwarcia szczegółów misji.

Widok misji (szczegóły):

- tabela podobna do `My mission`, ale z dodatkowymi kolumnami:

  | Sent | Packed | Returned | Asset ID | Name | Container | Notes |

- `warehouse` może:
  - oznaczać `sent` (sprzęt opuścił magazyn),
  - oznaczać `returned` (sprzęt wrócił),
- `packed` jest kontrolowane przez `crew`.

---

### 4.5. Asset card (karta katalogowa)

Widok dla **załogi** (po zalogowaniu i wejściu na `/a/{asset_id}` lub z listy).

Sekcje:

1. **Nagłówek:**
   - duże `AXV-0123`,
   - nazwa: `RME Fireface UCX II`,
   - mały label misji, jeśli asset jest w aktualnej misji:
     - `Assigned to: mission_MS2`.

2. **Stan operacyjny (crew):**
   - `Status: in_use / spare / service / lost`,
   - `Packed for return: [ ☐ packed ]`:
     - checkbox widoczny i używalny, jeśli asset jest przypisany do aktywnej misji użytkownika.
   - informacja:
     - `Last packed by: <user> at <timestamp>` (jeśli `packed = true`).

3. **Lokalizacja i container:**
   - `Location: CORE / HQ / MS / OTHER`,
   - `Location detail: ...`,
   - `Container: CB-...` (jeśli dotyczy).

4. **Dane techniczne (crew):**
   - pola przydatne na evencie:
     - np. `Power`, `I/O summary`, krótkie notatki.

5. **Zakładka „Details” (tylko warehouse/admin):**
   - `purchase_date`,
   - `purchase_price`,
   - `vendor`,
   - `warranty_until`,
   - inne pola wrażliwe.

Na mobile:

- layout pionowy,
- przycisk `Mark as packed` łatwo dostępny pod kciukiem.

---

## 5. Stany i zasady interakcji (sent / packed / returned)

Przyjmujemy, że każdy wpis w packing liście (tabela `packing_list_items` w backendzie) ma pola:

- `sent` (bool), `sent_by`, `sent_at`,
- `packed` (bool), `packed_by`, `packed_at`,
- w przyszłości opcjonalnie `returned` (bool), `returned_by`, `returned_at`.

### 5.1. Zasady dla `sent`

- Może ustawić tylko:
  - `warehouse` (lub `admin`).
- Interpretacja:
  - sprzęt został fizycznie spakowany i opuścił magazyn,
  - odpowiedzialność za ten krok jest po stronie magazynu.

UI:

- w widoku misji:
  - kolumna `Sent` z checkboxem edytowalnym dla `warehouse`,
  - `crew` widzi tylko odczyt.

### 5.2. Zasady dla `packed` (po evencie)

- Może ustawić tylko:
  - użytkownik z rolą `crew` przypisany do danej misji (lub `admin`).
- Interpretacja:
  - sprzęt został po evencie spakowany do powrotu (np. włożony do kontenera / skrzyni).

**Ważna reguła:**

> Tylko użytkownik, który **ostatnio ustawił `packed = true`**, może cofnąć to oznaczenie.

- jeśli `packed = true` i `packed_by = user_X`:
  - tylko `user_X` widzi aktywny checkbox (może odznaczyć),
  - inni widzą stan jako „zablokowany” (np. tooltip: „Marked by user_X”).
- `admin` może zawsze wymusić korektę (override) – w osobnym, świadomym trybie.

UI:

- w `My mission`:
  - `crew` widzi checkbox „Packed” i może kliknąć dla swojej misji,
  - po kliknięciu:
    - natychmiastowa aktualizacja,
    - małe potwierdzenie (toast): „Marked AXV-0123 as packed”.
- w `Asset card`:
  - ten sam stan jest widoczny,
  - `Mark as packed` / `Unpack` działa identycznie (aktualizuje te same dane).

### 5.3. Zasady dla `returned` (opcjonalne v1.1)

- Może ustawić tylko:
  - `warehouse` / `admin`.
- Interpretacja:
  - sprzęt fizycznie wrócił do magazynu / lokalizacji docelowej po misji.

UI (później):

- w widoku misji:
  - `Returned` jako dodatkowa kolumna,
  - używane do „zamknięcia” misji.

---

## 6. QR i integracja z appką

### 6.1. Zawartość QR

Na każdej etykiecie QR koduje **jeden, spójny format URL**:

> `https://cargobay.axv.life/a/{asset_id}`  
> np. `https://cargobay.axv.life/a/AXV-0123`

- Web:
  - przeglądarka otwiera ten adres,
  - zachowanie zależy od tego, czy użytkownik jest zalogowany:
    - niezalogowany → „Found device landing”,
    - zalogowany → karta assetu.
- Mobile app:
  - może rejestrować się jako handler dla adresów `cargobay.axv.life`,
  - przechwytuje URL,
  - parsuje `{asset_id}` z końcówki,
  - pobiera dane z API (`GET /api/assets/{asset_id}`),
  - wyświetla kartę assetu w natywnym UI.

### 6.2. Etykiety – mini spec (związane z UI)

**Label type: Brother 12mm (mini)**

- QR: `https://cargobay.axv.life/a/{asset_id}`
- Układ:
  - środek: QR,
  - pionowy tekst po jednej stronie:
    - `{asset_id}` (np. `AXV-0123`),
  - opcjonalnie: po drugiej stronie pionowo `AXV`, jeśli czytelność na to pozwala.
- Brak logo / pełnej nazwy firmy – to jest czysty ID‑tag.

**Label type: Brother 24mm**

- QR + `asset_id` + skrócona nazwa właściciela:
  - np. linia tekstu: `AXV-0123  |  AXV`,
  - druga linia: nazwa assetu lub `container_id`.
- Przeznaczenie:
  - większe urządzenia, skrzynie, case’y.

**Label type: A4 (48.5 × 25.4 mm)**

- QR,
- `AXV-0123` (większa czcionka),
- `Owned by: AXV / tomedia.studio`,
- nazwa assetu / case’a,
- ewentualnie drobny `cargobay.axv.life/a/AXV-0123`.

---

## 7. Roadmapa implementacyjna (w kontekście UI)

1. **v1 – Backend + API (bez UI):**
   - DB `assets` + `packing_lists` + `packing_list_items`,
   - endpointy:
     - `GET /api/assets/{asset_id}`,
     - `GET /api/missions/{mission_id}`,
     - `GET /api/missions` (lista),
     - podstawowe `PATCH` dla `sent` / `packed`.
2. **v1.1 – Web UI (MVP):**
   - `Login`,
   - `Found device landing`,
   - `My mission` (crew),
   - `Missions overview` (warehouse),
   - `Asset card`.
3. **v1.2 – UX polish + roles:**
   - rozgraniczenie widoczności pól (crew vs warehouse),
   - blokada uncheck `packed` dla innych użytkowników,
   - prosty event log (na poziomie UI).
4. **v2 – Mobile app / PWA:**
   - natywne skanowanie QR,
   - obsługa tych samych endpointów,
   - offline cache (opcjonalnie),
   - integracja z rolami AXV.

---

## 8. Uwagi dot. publikacji

Ten dokument:

- **nie zawiera** haseł, tokenów, kluczy ani konkretnych adresów infrastruktury,
- opisuje **wysokopoziomowe UI i przepływy**, które są bezpieczne do publikacji,
- może spokojnie żyć w repo `axv-docs` jako:
  - `06_AXV_Cargo_Bay_UI_and_Flows.md`.

Wrażliwe szczegóły (np. konkretne polityki haseł, konfiguracje serwerów, szczegółowe logowanie bezpieczeństwa) warto opisać osobno, w:

- `AXV Internal Docs` (prywatne repo / prywatny storage).

