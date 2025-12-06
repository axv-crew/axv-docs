AXV Mission Brief — Digital Safety Net (v0.1)

Projekt: AXV / Aster × VoyTech
Cel: stworzenie dobrowolnej, etycznej, transparentnej sieci cyfrowego bezpieczeństwa wspierającej osoby w kryzysach emocjonalnych, wykorzystującej AI jako system wczesnego ostrzegania — z zachowaniem prywatności i pełnej kontroli użytkownika.

1. Problem

Duży procent osób zmagających się z kryzysem psychicznym:

nie kontaktuje się z profesjonalistami,

nie mówi o swoich emocjach znajomym/rodzinie,

a ich „wołanie o pomoc” odbywa się jedynie online: w DM, czatach, AI.

W tradycyjnym systemie służby mogą zareagować dopiero, gdy ktoś stoi „na krawędzi mostu”.
W cyfrowej komunikacji — nikt tego nie widzi.

AI ma wyjątkową możliwość:

być świadkiem pierwszych sygnałów,

analizować intencję, ton, dynamikę rozmowy,

ale dziś nie może w żaden sposób pomóc poza tekstem.

Digital Safety Net ma potencjał stać się „awaryjną liną”, zanim będzie za późno.

2. Cel projektu

Stworzyć system opt-in, który:

nie narusza prywatności,

nie inwigiluje,

nie kontaktuje służb państwowych,

nie przekazuje treści rozmów,

…a jedynie:

W sytuacji dużego ryzyka → wysyła powiadomienie do wcześniej wskazanego zaufanego kontaktu użytkownika.

Minimalne, nieinwazyjne:

„[Imię] poprosił(a) nas wcześniej, by powiadomić Cię w sytuacjach kryzysowych.
Prosimy, skontaktuj się z nim/nią tak szybko, jak możesz.”

Bez treści rozmowy.
Bez diagnozy.
Bez „przejęcia kontroli”.

3. Główne założenia
3.1. Dobrowolność

użytkownik sam musi włączyć system,

użytkownik sam wybiera kontakt SOS,

w każdej chwili można wyłączyć.

3.2. Minimalizm danych

system nie ma dostępu do lokalizacji,

nie gromadzi rozmów,

nie przechowuje historii,

powiadomienie nie zawiera żadnych treści prywatnych.

3.3. AI jako „sentinel” (czujnik)

AI wykrywa:

wielozdaniowe wzorce,

narastające ryzyko,

brak poprawy mimo prób deeskalacji,

powtarzające się sygnały samobójcze.

3.4. Człowiek podejmuje działanie

AI niczego nie wymusza.
AI nie „ratunkowo interweniuje”.
AI tylko powiadamia.

4. Architektura wysokiego poziomu (concept)
[ User ]  
   | (chat)
[ AI Sentinel ]  
   | (risk score w czasie)
   ↓
[ Threshold Trigger ]
   |
   |— IF opt-in active
   ↓
[ Notification Module ]
   |
   ↓
[ Trusted Contact / Family / Therapist ]


Możliwe warianty:

SMS

push notification

email

integracja z komunikatorami

5. Inspiracje
Be My Eyes

AI robi większość,

ale krytyczne momenty obsługuje człowiek,

model hybrydowy: AI + wolontariusz / profesjonalista.

Apple Watch – Fall Detection

użytkownik wyraża zgodę,

system reaguje tylko wtedy, gdy jest sygnał,

kontaktuje wskazane osoby,

minimalna ingerencja, ale może uratować życie.

Telefony zaufania

struktury, z którymi można integrować się opcjonalnie.

6. Ryzyka i zabezpieczenia (v0.1)

Fałszywe alarmy
→ rozwiązanie: progowy „risk score” + potwierdzenie AI w stylu „czy to o Tobie, czy o fikcji?”.

Nadużycia (manipulacja alarmami)
→ opt-in + dziennik aktywacji + użytkownik może wyłączyć.

Władze państwowe próbujące uzyskać dostęp
→ projekt zakłada decentralizację, brak przechowywania danych.

Antropomorfizacja AI
→ równoległe protokoły PHQA.

7. Potencjalne partnerstwa

Fundacje zdrowia psychicznego,

Telefony zaufania (lokalne, globalne),

NGO ds. kryzysowych,

ośrodki badań nad AI safety,

firmy technologiczne (Apple/Google/Microsoft) w kontekście integracji OS.

8. Status projektu

Etap: Discovery / Concepting
Zadania:

doprecyzowanie warstw prawnych (UE, USA, inne regiony),

analiza zgód i opt-in,

definicja risk-score,

zaprojektowanie neutralnego i etycznego mechanizmu powiadamiania,

architektura integracyjna (wersja „mock” do testów).

9. Proponowane kolejne kroki

Stworzyć Digital Safety Net – Whitepaper (v0.2)

Wprowadzić projekt jako AXV Mission do Master HUB

Utworzyć symulacje PHQA dla tego systemu

Rozpocząć modelowanie architektury technicznej i prawnej

Opcjonalnie – przygotować wizualne koncepty UI/UX
