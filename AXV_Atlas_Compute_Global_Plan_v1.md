# AXV Atlas & Compute – Global Plan v1 (Hybrid C)  
**Data:** 2025-11-13  
**Status:** OFFICIAL DIRECTION (AXV)

---

## 1. Executive Summary

**Hybrid C** – produkcyjna architektura AXV:

- **Cloud EDGE (KVM2)**  
  Źródło prawdy dla danych: timeline + podstawowe indeksy + „hot” vectordb.  
  Obsługuje online RAG dla całego AXV.

- **CORE / Klaster (Ruderka)**  
  Atlas DEEP: rozszerzone wektory, deep search, analityka, lokalne LLM (Ollama).  
  Działa jako akcelerator, nie jako krytyczny dependency.

- **KVM1 (Front EDGE)**  
  UI + żywe instancje czatu (sesje), krótkoterminowy cache wątku, okresowy flush do KVM2.

- **HQ (Tomaszów)**  
  Przyszłe CORE/HQ; spięte z klastrem po WireGuard (RB5009↔RB5009).  
  Może pełnić rolę offsite/backup/ops.

**Kluczowa zasada:**  
Backend na KVM2 **musi działać** również przy niedostępnym klastrze (tryb _degraded but OK_).

---

## 2. Komponenty i Role

### 2.1. KVM1 – Front EDGE

- **Host:** VPS KVM1 (srv977231 – front)  
- **Rola:**
  - serwowanie frontendu (axv-web, strony),
  - utrzymywanie żywych instancji czatu (sesje, WebSockety),
  - trzymanie krótkoterminowego cache (ostatnie N wiadomości per wątek).
- **Dane:**
  - ephemeral (RAM/cache, ewentualnie Redis),
  - okresowy flush do KVM2 (`/threads/{thread_id}/append`).

---

### 2.2. KVM2 – Backend + Atlas EDGE

- **Host:** VPS KVM2 (srv977231 – backend)  
- **Rola:**
  - `axv-gw`, `axv-api`, integracje (n8n, webhooki, HMAC),
  - **Atlas EDGE**:
    - timeline (źródło prawdy),
    - podstawowe indeksy (FTS, tagi, relacje),
    - „hot” vectordb (ostatnie dane / summary).
  - publiczny RAG dla AXV (API + UI).
- **Dane:**
  - Postgres (timeline + indeksy),
  - mały vectordb (Qdrant/pgvector) – kolekcje „hot”.

---

### 2.3. CORE – Klaster (Ruderka)

- **Hosty:** Pi5, PN41, desktop/Legion, itd.  
- **Rola:**
  - **Atlas DEEP**:
    - duży Qdrant (pełne wektory, duże zbiory),
    - zaawansowane indeksy (graf, tematy, encje),
  - lokalne modele (Ollama, inne LLM),
  - batchowe przetwarzanie:
    - nocne indeksowanie,
    - generowanie summary,
    - deep analizy i raporty.
- **Dane:**
  - kopia / rozszerzenie timeline’u (opcjonalnie),
  - pełny wektorowy „deep store” (chunkowane dokumenty, archiwalne projekty).

---

### 2.4. HQ – Tomaszów

- **Host:** RB5009 + serwery w HQ.  
- **Rola:**
  - przyszłe centrum operacyjne (CORE/HQ),
  - offsite/backup danych (Atlas, Runbook, logi),
  - potencjalnie dodatkowy compute/storage.
- **Połączenia:**
  - WireGuard RB5009↔RB5009 (Ruderka↔HQ) – docelowy tunel CORE–HQ,
  - opcjonalnie WG ↔ KVM2 (backend).

---

## 3. Warstwy Pamięci Atlasu

### 3.1. Warstwa 1 – Timeline (źródło prawdy)

**Lokalizacja:** Postgres na KVM2.

**Tabela `threads`:**
- `id`
- `created_at`
- `meta` (JSON: typ wątku, projekt, kanał, itp.)

**Tabela `messages`:**
- `id`
- `thread_id`
- `position` (inkrementowana po kolei w wątku)
- `role` (`user`, `assistant`, `system`, `agent`)
- `author` (`voytech`, `aster`, `rezon`, `n8n`, itp.)
- `content` (tekst)
- `created_at`
- `meta` (JSON: `mentions`, `source`, `model`, `phase`, `tags`, itd.)

**Zasady:**
- Timeline to **kanoniczny zapis wydarzeń** w AXV (czaty, decyzje, eventy).
- Nic z timeline’u nie jest usuwane – archiwizacja może przenosić dane, ale nie kasować historii.

---

### 3.2. Warstwa 2 – Indeksy (tekst, tematy, graf)

**Lokalizacja:** KVM2 (Postgres + FTS i/lub dodatkowy engine, np. Meilisearch).

**Przykładowe struktury:**

`message_topics`:
- `message_id`
- `topic` (np. `AXV/Cluster/LibreNMS`, `n8n/Pulse`, `Governance`)

`message_entities`:
- `message_id`
- `entity` (np. `srv977231`, `RB5009`, `KVM2`, `Pi5`)
- `entity_type` (`host`, `service`, `person`, `project`, itd.)

`edges` (relacje/graf):
- `from_id`
- `to_id`
- `kind` (`refers_to`, `answers`, `depends_on`, `duplicates`, itd.)

**Źródła indeksów:**
- batch AI (cron / n8n / worker) na KVM2 lub CORE:
  - pobiera nowe wiadomości z timeline’u,
  - wywołuje LLM do:
    - klasyfikacji tematów,
    - ekstrakcji encji,
    - tworzenia relacji,
  - zapisuje wynik do tabel indeksowych + meta.

---

### 3.3. Warstwa 3 – Wektory (semantic search / RAG)

**EDGE vectordb (KVM2):**

- trzyma:
  - „hot data” (ostatnie dni/miesiące, aktywne projekty),
  - summary chunków (skondensowane dane z dłuższych wątków),
- rola:
  - obsługa online zapytań RAG dla UI/n8n (bez zależności od CORE),
  - fallback, gdy CORE jest offline.

**DEEP vectordb (CORE / Qdrant):**

- trzyma:
  - pełne, szczegółowe wektory całej historii (projekty, captain’s logi, runbooki),
  - archiwalne dane, duże zbiory dokumentów,
- rola:
  - deep search („znajdź rzeczy z całej historii AXV”),
  - ciężkie analizy, globalne raporty.

**Synchronizacja (high-level):**

- KVM2:
  - przy nowych wiadomościach generuje embedding (bezpośrednio przez API LLM lub delegując do CORE),
  - zapisuje wektory do EDGE vectordb.
- CORE:
  - subskrybuje eventy z KVM2 (np. kolejka, webhook, cron),
  - ściąga nowe wiadomości / embeddingi,
  - aktualizuje DEEP Qdrant,
  - opcjonalnie generuje summary/compact embeddingi i odsyła je na KVM2 (do EDGE).

---

## 4. Kluczowe Przepływy (Flows)

### 4.1. Flow – Interaktywny czat z frontu

1. Użytkownik pisze w przeglądarce → request trafia na **KVM1**.
2. KVM1:
   - utrzymuje stan sesji (ostatnie N wiadomości),
   - wysyła event do KVM2, np.:

   ```jsonc
   POST /ai/query
   {
     "thread_id": "chat-123",
     "sender": "voytech",
     "text": "@Aster, co z n8n?",
     "mentions": ["@Aster"],
     "source": "web",
     "ts": "2025-11-13T..."
   }
   ```

3. **KVM2:**
   - zapisuje event do `threads` / `messages`,
   - **Context Builder**:
     - FTS + vectordb EDGE → wybór ~5–10 istotnych fragmentów,
     - jeśli CORE online i zapytanie wymaga deep context → opcjonalny call do Atlas DEEP (Qdrant na klastrze),
   - składa prompt i woła model (OpenAI/Gemini/Moonshot/...).

4. Odpowiedź LLM:
   - zostaje zapisana jako kolejna wiadomość w timeline,
   - otrzymuje embedding (i/lub trafia w kolejkę „do embedowania”),
   - jest równolegle odesłana do KVM1 i wyświetlona użytkownikowi.

---

### 4.2. Flow – Flush z KVM1 do KVM2

- Okresowo (co X wiadomości lub Y sekund) KVM1 wykonuje:

  ```http
  POST /threads/{thread_id}/append
  {
    "since_position": 120,
    "messages": [ ...nowe wiadomości... ]
  }
  ```

- KVM2:
  - dopisuje wiadomości do timeline’u,
  - dorzuca je do pipeline’u indeksowania (tematy, encje, wektory).

Po flushu KVM1 może zwolnić starsze wiadomości z RAM/cache (zachowując np. tylko lokalne summary).

---

### 4.3. Flow – Deep indeksowanie na klastrze (Atlas DEEP)

- **Trigger:**
  - cron (np. co 5 min),
  - lub event typu „new_messages_batch” z KVM2.

- **CORE:**
  - pobiera listę nowych/zmodyfikowanych `messages` z KVM2,
  - liczy embeddingi przy użyciu lokalnych modeli (Ollama) lub zewnętrznych API,
  - zapisuje wektory do Qdrant (Atlas DEEP),
  - opcjonalnie generuje summary / „compact embeddings” i odsyła je na KVM2.

---

## 5. Tryb Degraded (CORE Offline)

**Założenie:** CORE jest akceleratorem, nie warunkiem działania AXV.

- RPC do CORE (np. `http://core-api.lan/atlas/search`) mają:
  - krótki timeout (np. 200–500 ms),
  - obsługę błędów: jeśli brak odpowiedzi → log + fallback.
- **Fallback:**
  - RAG wyłącznie z EDGE vectordb (KVM2) + FTS,
  - jeśli vectordb EDGE też niedostępny → model pracuje bez RAG, a użytkownik dostaje
    informację, że działa w trybie ograniczonego kontekstu.

**Żaden endpoint publiczny** nie może przestać działać tylko dlatego, że klaster (CORE) jest offline.

---

## 6. Snippet Do Runbooka (High-Level)

```markdown
## AXV Atlas & Compute – Global Plan v1 (Hybrid C) — 2025-11-13

- Hybrid C = produkcyjny standard AXV:
  - Atlas EDGE (KVM2) – timeline + podstawowe indeksy/FTS + hot vectordb.
  - Atlas DEEP (Klaster) – rozszerzone wektory, deep search, analityka.
  - Front (KVM1) – UI + żywy stream czatu (flush do KVM2).
  - HQ – spięte z CORE po WireGuard (RB5009↔RB5009).

- KVM2 = źródło prawdy (timeline).  
  CORE jest akceleratorem, a nie krytycznym dependency:
  - brak klastra → backend działa w trybie „degraded RAG” na EDGE.
  - CORE online → dodatkowo deep search / globalne analizy.

- Wszystkie nowe prace dot. pamięci/AI (AXV, Atlas, Runbook, logi, governance)
  muszą być zgodne z tym modelem (Hybrid C).
```

---

## 7. Roadmap v1 (Tasklist)

### A. Atlas EDGE (KVM2)

1. [ ] Zaprojektować i utworzyć schemat `threads` + `messages` w Postgresie.  
2. [ ] Dodać prosty FTS (Postgres lub osobny search engine).  
3. [ ] Postawić mały vectordb (pgvector/Qdrant) na KVM2:
   - baza `atlas_edge`,
   - kolekcja `messages_edge` (hot data).
4. [ ] Zaimplementować endpoint `POST /ai/query` w `axv-gw`:
   - przyjmujący `thread_id`, `text`, `sender`, `mentions`, `source`,
   - zapis do timeline,
   - budowanie kontekstu (FTS + vectordb EDGE),
   - wywołanie modelu LLM,
   - zapis odpowiedzi do timeline + kolejka do embeddingu.

### B. Front (KVM1)

5. [ ] W `axv-web` dodać obsługę `thread_id` i mechanizm flushowania:
   - flush co X wiadomości / Y sekund do `POST /threads/{thread_id}/append`.  
6. [ ] Ewentualnie dodać cache (RAM/Redis) i politykę zwalniania starych wiadomości po flushu.

### C. CORE / Atlas DEEP (Ruderka)

7. [ ] Zaprojektować API `core-api` (np. `/atlas/deep/index`, `/atlas/deep/search`).  
8. [ ] Skonfigurować Qdrant na klastrze jako DEEP store (pełne wektory).  
9. [ ] Dodać proces synchronizacji:
   - KVM2 → CORE: nowe wiadomości / embeddingi,
   - CORE → Qdrant DEEP: aktualizacja indeksów,
   - opcjonalnie CORE → KVM2: summary/compact embeddings.

### D. Resilience / Networking

10. [ ] Zdefiniować health-check CORE (`/healthz`) i timeouty w `axv-gw`.  
11. [ ] Zaprojektować i wdrożyć tunel WireGuard RB5009↔RB5009 (Ruderka↔HQ).  
12. [ ] Dodać monitoring (Prometheus/Grafana) dla:
    - CPU/RAM/dysk KVM2,
    - latency/availability CORE (API, Qdrant),
    - zajętość vectordb EDGE/DEEP.

---

**Koniec dokumentu – AXV Atlas & Compute – Global Plan v1 (Hybrid C)**
