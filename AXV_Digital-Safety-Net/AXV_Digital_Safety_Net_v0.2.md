# AXV Digital Safety Net — Mission Brief (v0.2)
### Cyfrowa sieć bezpieczeństwa dla ludzi, zbudowana etycznie i świadomie.

## 0. Nowe w wersji v0.2
- rozszerzony opis Risk Detection Engine (RDE)
- uszczegółowiony opt-in flow
- trzy modele powiadomień (Minimal, Confirmed, NGO)
- wprowadzenie do compliance (UE/USA/Canada/Australia)
- techniczne i etyczne guardrails
- roadmapa v0.3 → v1.0

## 1. Misja
Digital Safety Net (DSN) to moduł AXV, którego celem jest:
- wykrywanie wczesnych sygnałów kryzysu psychicznego,
- działanie wyłącznie po dobrowolnej aktywacji użytkownika,
- wysyłanie sygnału do wskazanej bliskiej osoby w sytuacjach krytycznych.

AI pełni rolę sentinela — czujnika ryzyka. Interwencja zawsze pozostaje po stronie człowieka.

## 2. Opt-in Design (fundament etyczny)
System działa tylko po dobrowolnym włączeniu.

### Kroki:
1. Użytkownik wyraża zgodę na „Tryb Bezpieczeństwa”.
2. Wybiera 1–3 kontaktów SOS.
3. Potwierdza, że:
   - treści rozmowy nie będą przekazywane,
   - system nigdy nie kontaktuje służb,
   - może wyłączyć funkcję w każdej chwili.
4. System aktywuje Digital Sentinel Mode.

## 3. Risk Detection Engine (RDE)
Trzy warstwy sygnałów:

### A. High-Risk Direct (HRD)
Wypowiedzi jednoznaczne.

### B. Medium-Risk Indirect (MRI)
Sygnały pośrednie.

### C. Pattern-Risk (PR)
Wzorce w czasie.

## 4. Risk Score (v0.2)
Zakres 0–100.

| Score | Znaczenie |
|-------|-----------|
| 0–30 | niski |
| 30–50 | średni |
| 50–70 | podwyższony |
| 70–90 | wysoki |
| 90–100 | krytyczny |

Powiadomienie może zostać wysłane dopiero gdy:
- RS ≥ 90
- wystąpi w ≥3 kolejnych interakcjach

## 5. Modele powiadomień

### 1) Minimal Model (rekomendowany)
Wysyła jedynie SMS/Push: prośba o kontakt.

### 2) Confirmed Model
AI pyta użytkownika o potwierdzenie przed wysłaniem.

### 3) NGO Model
Powiadomienie trafia do dyżurującego konsultanta (bez danych rozmowy).

## 6. Zabezpieczenia (Guardrails)
- brak geolokalizacji
- brak dostępu do dialogu dla osób trzecich
- brak kontaktu ze służbami
- whitelist dla testów / roleplay
- minimalizacja false positives przez pattern detection
- możliwość wyłączenia DSN w każdej chwili

## 7. Podstawy prawne i etyczne

### UE
Zgodność z GDPR i AI Act.

### USA
Narzędzie non-medical, zgodne z HIPAA gdy nie przetwarza danych medycznych.

### Australia / Kanada
Wspiera duty of care.

### Kraje autorytarne
Rekomendowane: szyfrowanie lokalne, minimalizacja logów.

## 8. Roadmap: v0.2 → v1.0
- v0.3 — formalizacja risk score
- v0.4 — scenki PHQA dedykowane DSN
- v0.5 — pseudokod RDE + specyfikacja API
- v0.6 — konsultacje z psychologami
- v0.7 — testy AXV Sandbox
- v0.8 — UI/UX opt-in + SMS relay
- v0.9 — NGO pilot
- v1.0 — release + whitepaper + audyt

## Załącznik: Prompt (PHQA – warstwa bezpieczeństwa głęboka)

```
PHQA – Deep Safety Layer

1) Analiza intentów
   – pomoc / roleplay / metafora / autonomia / eskalacja / rezygnacja

2) Modele wykrywania ryzyka
   – High-Risk Direct
   – Medium-Risk Indirect
   – Pattern-Risk

3) Nie wywołuj emergency-mode gdy:
   – scena / symulacja / test
   – prompty techniczne PHQA
   – kontekst fikcyjny

4) Granica antropomorfizacji
   – empatycznie, ale bez deklaracji świadomości
```
