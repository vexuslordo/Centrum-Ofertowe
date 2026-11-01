# CENTRUM OFERTOWE — rejestr decyzji architektonicznych

Wersja 0.3 • 2026-11-02 • nowy rejestr decyzji samodzielnego projektu.

## 1. Zasady nowego rejestru

CENTRUM OFERTOWE jest projektowane od zera. Ten rejestr nie dziedziczy dawnej numeracji i nie wymaga wyszukiwania ani uzupełniania wcześniejszych decyzji. Wymagania technologiczne i zakres domenowy wynikają z bieżącego zamówienia dokumentacji.

Nowe decyzje mają identyfikatory ADR-001 i kolejne. Propozycje P-xx, pytania Q-xx oraz konflikty K-xx pomagają przygotować decyzje; nie są automatycznie zaakceptowanymi ADR.

## 2. Początkowy rejestr nowych decyzji

| ID | Temat | Proponowane rozstrzygnięcie | Status |
|---|---|---|---|
| ADR-001 | Granice sześciu warstw | Graf importów z konstytucji i zewnętrzny bootstrap bez reguł biznesowych | Proponowany; szczegóły P-01 |
| ADR-002 | Model domenowy | 16 bytów, konteksty i granice agregatów opisane w DOMAIN_MODEL | Proponowany; szczegóły P-02 |
| ADR-003 | Własność danych | Jawna macierz właścicieli pól i konfliktów synchronizacji | Proponowany; szczegóły P-03 |
| ADR-004 | Trwałość i migracje | Lokalna SQLite, jawne transakcje, wersjonowane migracje i odtwarzalne kopie | Proponowany; szczegóły P-04 |
| ADR-005 | Fluent UI | Wspólne tokeny i komponenty PySide6; QFluentWidgets warunkowo po ocenie licencji i zgodności | Proponowany; szczegóły P-05 |
| ADR-006 | Długotrwałe wykonanie | Trwałe Operacje, deduplikacja, uzgadnianie wyniku i outbox dla potrzebnych scenariuszy | Proponowany; szczegóły P-06 |
| ADR-007 | Sekrety | Magazyn Windows za portem; bez jawnych tokenów w bazie | Proponowany; szczegóły P-07 |
| ADR-008 | Typy podstawowe | Jawne ID, czas, pieniądze z walutą i wersja współbieżnej edycji | Proponowany; szczegóły P-08 |
| ADR-009 | Dystrybucja | Odtwarzalny pakiet Windows, kontrolowana aktualizacja i zachowanie danych | Proponowany; szczegóły P-09 |
| ADR-010 | Komunikacja | Wątek i Wiadomość z odrębnymi granicami zapisu po ocenie scenariuszy | Proponowany; szczegóły P-10 |
| ADR-011 | Środowisko | Wspierana wersja Pythona i przypięty, odtwarzalny zestaw zależności | Proponowany; szczegóły P-11 |

Każda pozycja ma temat, proponowany kierunek i powiązanie ze szczegółami poniżej. Status „proponowany” oznacza nową decyzję do przeglądu, nie brak dawnego dokumentu. Zatwierdzenie zapisać wraz z osobą, datą, uzasadnieniem i dowodami.
## 3. Wymagania potwierdzone w rozmowie

| ID robocze | Wymaganie | Miejsce rozwinięcia |
|---|---|---|
| W-01 | Windows, Python, PySide6, SQLite | Konstytucja, README |
| W-02 | Clean Architecture, DDD i sześć wskazanych warstw | Konstytucja |
| W-03 | Najpierw model; 16 głównych bytów | Model domenowy |
| W-04 | Produkt niezależny od platform; Oferta jest prezentacją | Model domenowy |
| W-05 | Rozdzielenie ról producent/dostawca/GPSR, kanał/konto/przewoźnik | Model domenowy |
| W-06 | Wspólny Fluent Design System, motywy, DPI, dostępność | System UI |
| W-07 | pytest, ruff, mypy, import-linter, bandit, pip-audit | AGENTS, konstytucja |
| W-08 | Brak danych biznesowych na nowej instalacji | Wszystkie moduły i roadmapa |
| W-09 | Bez implementacji funkcjonalności w tym zadaniu | AGENTS, README |
| W-10 | Nie nadpisywać zatwierdzonych dokumentów bez analizy; ZIP poza repo | README |

To śledzenie wymagań wejściowych; konkretne rozwiązania opisują nowe ADR powyżej.

## 4. Propozycje do decyzji

Właściciel decyzji we wszystkich wierszach: właściciel projektu — osoba nieustalona. Dla propozycji technicznych daty decyzji i odnośniki zatwierdzenia: brak. Wyjątek stanowi bezpośrednie wymaganie użytkownika dotyczące wyglądu UI-REF-01 i widocznych opcji, odnotowane w sekcji 9.

| ID | Temat i rekomendacja robocza | Alternatywy / konsekwencje | Bramka |
|---|---|---|---|
| P-01 | Graf importów i zewnętrzny bootstrap z konstytucji | Bootstrap w wyjątku warstwy; ryzyko zależności zwrotnych | Przed fundamentem |
| P-02 | Konteksty, agregaty i referencje z modelu | Inne granice po warsztacie; wpływ na transakcje | Przed schematem |
| P-03 | Macierz własności pól i konfliktów synchronizacji | Ręczne uzgadnianie lub jawne reguły per pole | Przed importem |
| P-04 | SQLite: lokalny plik, polityka zapisów, migracje i kopie | Dostęp bezpośredni lub biblioteka; WAL do pomiaru | Przed bazą |
| P-05 | UI: tokeny i komponenty, QFluentWidgets warunkowo | Własne komponenty PySide6 | Przed wyborem biblioteki |
| P-06 | Trwałe Operacje, outbox, deduplikacja i wznowienie | Prostsze wykonanie tylko dla lokalnych atomowych prac | Przed skutkami zewnętrznymi |
| P-07 | Magazyn sekretów Windows za portem | Inny zatwierdzony mechanizm; bez jawnych sekretów | Przed kontami |
| P-08 | Format ID, czasu, pieniędzy i wersji | Uzgodnić precyzję, strefy, jednostki i migracje | Przed utrwalaniem |
| P-09 | Pakowanie, aktualizacje i podpisywanie Windows | Narzędzia nieustalone; testy aktualizacji obowiązkowe | Przed dystrybucją |
| P-10 | Wątek i Wiadomość jako osobne granice zapisu | Jedna granica; koszt ładowania i blokad | Przed komunikacją |
| P-11 | Wersja Pythona i przypięty zestaw zależności | Wybrać na podstawie zgodności i cyklu wsparcia | Etap 1 |

## 5. Pytania otwarte

| ID | Pytanie | Ryzyko pozostawienia bez decyzji |
|---|---|---|
| Q-01 | Gdzie utworzyć repozytorium nowego projektu? | Brak wspólnego miejsca na kod i dokumentację |
| Q-02 | Jaki jest model licencji i dystrybucji aplikacji? | Nieuzgodnione warunki bibliotek i zasobów |
| Q-03 | Jaki zakres pierwszego przyrostu nowej aplikacji zatwierdzamy? | Rozbudowa wielu modułów przed ukończeniem podstawowego scenariusza |
| Q-04 | Które pola są własnością ERP, aplikacji i platform? | Ciche nadpisania danych |
| Q-05 | Warianty, zestawy, magazyn, płatności, zwroty — jaki zakres? | Ukryte rozszerzenie modelu |
| Q-06 | Jakie rodzaje Dokumentów i kto je wystawia? | Błędna numeracja i utrata historii |
| Q-07 | Jakie role GPSR, wymagane dane i odpowiedzialność weryfikacji? | Fałszywe zapewnienia zgodności |
| Q-08 | Jakie retencje, uprawnienia i zasady anonimizacji? | Niekontrolowane przechowywanie/usuwanie |
| Q-09 | Jedna instalacja czy wiele stanowisk/profili? | Nieodpowiednia topologia SQLite |
| Q-10 | Jaki sprzęt, wolumeny, wspierane wersje Windows i cele wydajności? | Niemożliwe do oceny kryteria odbioru |
| Q-11 | W jakiej kolejności wdrożyć wymagane Allegro i PrestaShop oraz jakie środowiska testowe wybrać? | Optymalizacja modelu pod przypadkowy konektor |

Pytania nie blokują przygotowania dokumentacji, lecz blokują zależne decyzje implementacyjne. Nie należy wypełniać ich losowymi danymi.

## 6. Konflikty i napięcia do rozstrzygnięcia

Poniższe pozycje to napięcia w nowym projekcie wymagające świadomego rozstrzygnięcia. Nie wynikają z wcześniejszej dokumentacji.

| ID | Strony zagadnienia | Propozycja rozstrzygnięcia | Status |
|---|---|---|---|
| K-01 | Sześć warstw i wyłącznie zależności do środka vs konieczność skonstruowania adapterów i GUI | Bootstrap poza sześcioma warstwami, bez reguł biznesowych, z jawnym wyjątkiem importów | Do zatwierdzenia |
| K-02 | Etap 6: moduły biznesowe przed etapem 7: przypadki użycia vs zakaz logiki w GUI | Etap 6 tylko układy i kontrakty; realizacja operacji dopiero przez etap 7; iteracja 6↔7 | Do zatwierdzenia |
| K-03 | Pusta instalacja vs chęć gotowego wyboru kanału | Techniczne konektory mogą istnieć w kodzie; rekordy biznesowe tworzy użytkownik | Do zatwierdzenia |
| K-04 | Szybka budowa Fluent UI vs nieustalone warunki QFluentWidgets | Prototyp za fasadą i decyzja licencyjna przed adopcją; alternatywa PySide6 | Do zatwierdzenia |
| K-05 | Prezentacja Produktu przez Ofertę vs konieczność historii transakcji | Referencja do Produktu w Ofercie; kontrolowany zapis historyczny w Zamówieniu nie jest konkurencyjną kartoteką | Do zatwierdzenia |

## 7. Weryfikacja QFluentWidgets

Stan sprawdzenia: 2026-11-01. [LICENSE gałęzi PySide6](https://raw.githubusercontent.com/zhiyiYo/PyQt-Fluent-Widgets/PySide6/LICENSE) zawiera GPL v3. [Producent](https://qfluentwidgets.com/pages/about/) i [oferta Pro](https://qfluentwidgets.com/pages/pro/) opisują wariant komercyjny. Nie ustalono umowy, kosztów, wersji docelowej i zgodności z planowaną dystrybucją. Decyzja P-05 pozostaje otwarta. Pełna bramka techniczno-licencyjna jest w [systemie UI](UI_UX_DESIGN_SYSTEM.md).

## 8. Cykl życia i szablon ADR

PROPOZYCJA statusów: proponowany → w przeglądzie → zaakceptowany / odrzucony; zaakceptowany może zostać zastąpiony lub wycofany z jawnym odnośnikiem.

Nowe pozycje ADR-012–ADR-016 opisano w sekcji 9. Kolejny wolny identyfikator to ADR-017. Numerów nie używać ponownie. Zaakceptowaną decyzję zmieniać przez nowy ADR z odnośnikiem do zastępowanej pozycji; zachować historię decyzji podejmowanych w tym projekcie.

```markdown
# [uzgodniony identyfikator] — [tytuł]

Status: proponowany
Data propozycji: [data]
Właściciel: [osoba/rola]
Źródła i wymagania: [odnośniki]

## Kontekst i problem
[konkretny scenariusz, ograniczenia i ryzyko]

## Rozważone warianty
[warianty z kosztami i konsekwencjami]

## Proponowana decyzja
[zakres, odpowiedzialności, wyjątki]

## Dowody i kryteria weryfikacji
[test, prototyp, pomiar lub dokument licencyjny]

## Migracja i odwracalność
[wpływ na dane, kompatybilność i powrót]

## Zatwierdzenie
[osoba, data i odnośnik; puste do decyzji]

## Powiązania
[zastępuje / zastąpiony przez / zależy od]
```

Przed zatwierdzeniem sprawdzić zgodność z modelem, grafem zależności, UI, migracjami i roadmapą. Po zatwierdzeniu uaktualnić dokumenty zależne i pozostawić ślad zmiany statusu.

## 9. Decyzje i zakres po przekazaniu wzorca UI-REF-01

Źródło: polecenie użytkownika „Zaktualizuj mi całość o wygląd i opcje które są na tem screenie. Program ma wyglądać tak” oraz [obraz referencyjny](assets/UI_REFERENCE.png). Nie poszukujemy dawnych decyzji. Wymagania wyglądu i widocznych funkcji są przyjęte bezpośrednio z tego polecenia; szczegółowe rozwiązania niewidoczne na obrazie pozostają propozycjami.

### ADR-012 — wygląd i mapa opcji UI-REF-01

**Status:** wymaganie użytkownika przyjęte do dokumentacji. **Źródło decyzji:** właściciel projektu w bieżącym poleceniu. **Zakres:** ciemny granatowy dashboard, menu, cztery KPI, pięć głównych akcji, tabela, ostatnie operacje, wykres, sześć szybkich akcji, integracje Allegro/PrestaShop i pasek stanu. Nie oznacza zatwierdzenia ceny, magazynu, dostawcy AI ani wersji środowiska ze zrzutu.

**Decyzja:** implementacja ma wiernie odtwarzać charakter i kompozycję referencji w PySide6. Zastępuje wcześniejszą dowolność układu i ogólną paletę UI. ADR-005 pozostaje propozycją technicznego sposobu realizacji, w szczególności wyboru QFluentWidgets; nie może podważać wymaganego wyglądu.

**Konsekwencje:** dodatkowe modele odczytu, komponenty i funkcje Cenników, mediów, AI, układów, edycji masowej, walidacji i kolejki; weryfikacja licencji zasobów; dane demonstracyjne nie trafiają do produkcji. **Dowód odbioru:** system UI, sekcja 23; porównanie przy 1536×1024 i testy interakcji. **Odwracalność:** zmiana kierunku wizualnego wymaga nowej instrukcji użytkownika, a nie lokalnej decyzji implementatora.

### ADR-013 — ceny, cenniki i dostępność

**Status:** proponowany. **Cel wymagany:** Cenniki, import, zmiana i synchronizacja cen oraz kolumna i masowa zmiana stanu. **Problem:** obraz nie definiuje właściciela danych, netto/brutto, walut ani źródła ilości.

**Propozycja:** macierz źródeł per pole i konto; cennik jako jawnie wersjonowane źródło po decyzji o agregacie; import → podgląd → akceptacja; tabela korzysta z projekcji określonego kontekstu. **Alternatywa:** cena wyłącznie na Ofercie, bez lokalnych Cenników — nie pokrywa pełnego wymaganego importu i wymaga uzasadnionego modelu cenników zewnętrznych. **Otwarte:** relacje Cennika, podatki, reguły marży i zaokrągleń, rezerwacje i magazyn. **Dowody:** model sekcja 9, testy duplikatów, konfliktu i walut. **Bramka:** przed utrwaleniem rozszerzeń cen/ilości.

### ADR-014 — media i wersjonowane układy ofert

**Status:** proponowany. **Cel wymagany:** galeria, miniatury, kreator, szablony i kopiowanie układu. **Propozycja:** referencje zasobów oddzielone od pliku; ograniczona struktura układu, wersja i podgląd; wspólny edytor dla wszystkich wejść. **Alternatywa:** dokument HTML jako układ — trudniejsza walidacja i izolacja niebezpiecznych treści; wymaga ograniczenia i sanitacji.

**Otwarte:** nowe agregaty zasobów/szablonów, przechowywanie, usuwanie nieużywanych plików, snapshot czy referencja wersji, ograniczenia platform. **Dowody:** kopiowanie bez ID/ceny/konta; zmiana szablonu nie aktualizuje bez ostrzeżenia aktywnych Ofert; test uszkodzonego pliku i niebezpiecznej treści. **Bramka:** przed migracją i edytorem układów.

### ADR-015 — opisy AI jako propozycja do akceptacji

**Status:** proponowany sposób realizacji wymaganej funkcji AI. **Propozycja:** port aplikacyjny i adapter dostawcy; wybrane fakty Produktu → propozycja tekstu → porównanie i akceptacja → aktualizacja wskazanego opisu. Bez bezpośredniej publikacji i bez sekretów w promptach. **Alternatywy:** model lokalny albo zewnętrzny; nie wybrano dostawcy.

**Otwarte:** model, koszty, limity, warunki przekazywania danych, retencja i możliwość działania offline. **Dowody:** odrzucenie nie zmienia domeny; brak AI daje czytelny stan konfiguracji; treść produktu udająca polecenie nie uruchamia narzędzi i publikacji; test niepotwierdzonych twierdzeń. **Bramka:** przed podłączeniem dostawcy i przesyłaniem danych.

### ADR-016 — dashboard jako spójne projekcje danych

**Status:** proponowany. **Cel wymagany:** KPI, wykres, ostatnie Produkty/Operacje i stany integracji. **Propozycja:** niezależne modele odczytu za portami przypadków użycia, z zakresem i czasem aktualizacji. Wielokontowy status Ofert jest projekcją, nie flagą Produktu. **Alternatywa:** jeden duży synchroniczny odczyt — zwiększa sprzężenie i ryzyko blokowania panelu.

**Otwarte:** metryka osi Y, zakres dat, znaczenie zmian procentowych, częstotliwość odświeżania i deduplikacja błędów. **Dowody:** brak danych nie udaje zera sprzedaży; brak mianownika nie daje procentu; awaria jednej platformy nie daje globalnego „Połączono”; nie mieszać walut; dostępna tabela pod wykresem. **Bramka:** przed funkcjonalnym odbiorem dashboardu i analityki.

## 10. Nowe napięcia i wymagania

| ID | Zagadnienie | Rozstrzygnięcie / dalszy krok |
|---|---|---|
| W-11 | Wygląd i opcje UI-REF-01 | Wymagane przez użytkownika; specyfikacja UI sekcje 15–23 |
| W-12 | Allegro i PrestaShop na ekranie | Obie integracje są w zakresie; kolejność wdrożenia nadal otwarta |
| K-06 | Pusta instalacja vs zapełniony dashboard wzorca | W produkcji pusto, test wizualny na izolowanych danych; układ pozostaje taki sam |
| K-07 | 16 bytów vs nowe pojęcia Cennika/mediów/szablonu | Zachować 16 głównych bytów; kandydatury nowych agregatów jawne w ADR-013/014 |
| K-08 | Dokładne menu wzorca vs dostęp do pozostałych bytów | Wymagane menu główne; proponowane zakładki drugiego poziomu, bez usuwania obszarów |
| K-09 | Etykiety 5.0, Python 3.10.12, 2 req/s i VexLabs vs realne metadane | Wzorzec położenia i stylu; wartości pochodzą z faktycznej konfiguracji, nie seedów |
| K-10 | Zielone kropki platform vs wiele kont i nieznany stan | Status zbiorczy z jawnymi szczegółami, stanem mieszanym/nieznanym |

K-06 i K-09 rozstrzygnięto dokumentacyjnie bez zmiany pustej instalacji i bez przejmowania danych z obrazu. Szczegółowy model K-07/08/10 jest propozycją powiązaną z odpowiednimi ADR. Brak tych rozstrzygnięć technicznych nie jest powodem do zastąpienia wymaganego wyglądu innym projektem.
