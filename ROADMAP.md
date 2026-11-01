# CENTRUM OFERTOWE — plan rozwoju

Wersja 0.3 • 2026-11-02 • plan do zatwierdzenia, bez deklaracji wykonania etapów.

## 1. Zasady planu

WYMAGANIE: rozwój przebiega przez 12 wskazanych etapów, z pierwszeństwem modelu domenowego przed integracjami. Nazwy i kolejność etapów wynikają z polecenia. Zadania szczegółowe, zależności i bramki poniżej są PROPOZYCJAMI. Nie nadano terminów, budżetu ani obsady, ponieważ ich nie ustalono.

Stan początkowy: przygotowano siedem dokumentów nowego projektu. Plan nie wymaga wcześniejszego repozytorium ani dokumentacji. Nie ma implementacji, schematu bazy, działającego CI ani potwierdzonej wersji zależności. Żaden etap implementacyjny nie jest oznaczony jako ukończony.

Każdy etap ma dowody odbioru: zmienione artefakty, wyniki testów, otwarte decyzje, ryzyka i osobę zatwierdzającą. Niespełniona bramka blokuje tylko zależne prace. Dokumentacja, testowanie i bezpieczeństwo są pracą ciągłą; etap 11 nie oznacza odkładania ich do końca.

## 2. Etap 1 — przygotowanie środowiska

**Cel:** odtwarzalne środowisko Windows, zgodne wersje i wspólny standard pracy.

**Zależności:** wybór miejsca na nowe repozytorium, decyzja o wersji Pythona i sposobie zarządzania zależnościami.

**Prace P:** utworzyć repozytorium nowego projektu i umieścić w nim uzgodnioną dokumentację; ustalić wersje Windows i Pythona; utworzyć izolowane środowisko; przygotować `pyproject.toml`, przypięte zależności i grupy developerskie; skonfigurować pytest, ruff, mypy, import-linter, bandit i pip-audit; opisać CI. Nie wybierać biblioteki UI tylko dlatego, że łatwo ją zainstalować.

**Artefakty P:** instrukcja uruchamiania zgodna z rzeczywistą konfiguracją, rejestr wersji, minimalny przebieg CI i lista otwartych decyzji.

**Kryteria ukończenia P:** druga czysta instalacja środowiska odtwarza ten sam zestaw; narzędzia uruchamiają się z tego środowiska; brak sekretów i danych biznesowych w repozytorium; dokumentacja nie odwołuje się do nieistniejących punktów startu.

**Testy P:** instalacja w czystym środowisku; test wykrywania próbnego błędu formatu, typowania i importu; kontrola kompletności audytu zależności. Próbne naruszenia usunąć po weryfikacji. Pusty zbiór testów nie jest dowodem poprawności aplikacji.

## 3. Etap 2 — fundament architektoniczny

**Cel:** sześć warstw, porty i mechanizm składania bez zależności zwrotnych.

**Zależności:** etap 1; decyzja P-01 i rozstrzygnięcie K-01.

**Prace P:** zatwierdzić graf importów z konstytucji; utworzyć strukturę pakietów; zaprojektować minimalne kontrakty błędów, zegara i korelacji; umieścić porty w przypadkach użycia; określić punkt składania; zaprojektować granice testów. Nie tworzyć uniwersalnego frameworka bez scenariusza.

**Artefakty P:** kontrakty import-linter, diagram zależności, testy granic i przykład izolowanego przepływu technicznego bez danych biznesowych.

**Kryteria ukończenia P:** domena ładuje się bez Qt i bazy; GUI nie importuje implementacji infrastruktury; `kanaly` i `adaptery` pozostają niezależne; bootstrap jest jawnie wyłączonym z kontraktu punktem zewnętrznym, a nie furtką dla usług biznesowych.

**Testy P:** zakazy importów, test wymienności portu, test uruchomienia bez kont i danych użytkownika. Przegląd dynamicznych zależności uzupełnia lint.

## 4. Etap 3 — model domenowy

**Cel:** zatwierdzone znaczenie 16 bytów i najważniejszych scenariuszy.

**Zależności:** etap 2; dostęp do właściciela projektu i przegląd propozycji nowego modelu.

**Prace P:** przeprowadzić przegląd kart DOMAIN_MODEL; zatwierdzić pola, agregaty, liczności i właścicieli; ustalić semantykę pieniędzy, czasu i ID; rozdzielić stan zamierzony od obserwowanego; uzgodnić duplikaty, archiwizację i konflikt wersji. Nie budować domeny ze schematu API Allegro.

**Artefakty P:** zatwierdzony model, macierz własności pól, scenariusze akceptacyjne, ADR zmieniające status propozycji oraz kod domeny dopiero po tych decyzjach.

**Kryteria ukończenia P:** wszystkie 16 kart mają właściciela, źródło prawdy, granicę i operacje; nieustalone reguły mają jawnie wyłączony zakres; każdy zatwierdzony niezmiennik ma scenariusz sprawdzenia. Szkicowa obecność 16 klas nie wystarcza.

**Testy P:** pozytywne, graniczne i odrzucające dla niezmienników; odrębność ról i tożsamości; historyczne dane transakcji; bez Qt, SQL i sieci.

## 5. Etap 4 — baza danych i migracje

**Cel:** trwały zapis zgodny z zatwierdzonym modelem, bezpieczna aktualizacja i odtworzenie.

**Zależności:** etap 3; P-04 i P-08; decyzja o lokalizacji danych i topologii instalacji.

**Prace P:** wybrać mechanizm dostępu i migracji; zmapować agregaty na tabele bez utożsamiania tych pojęć; ustalić klucze i indeksy; wdrożyć repozytoria i transakcje; zaprojektować współbieżny zapis, kopie i diagnostykę awarii. Zdefiniować wspierane ścieżki aktualizacji.

**Artefakty P:** schemat, rejestr migracji, adaptery portów, procedura kopii/odtworzenia i test czystej instalacji.

**Kryteria ukończenia P:** nowa baza ma zero rekordów biznesowych; zapis nie jest częściowy po błędzie; wersja aplikacji nie otwiera nieobsługiwanego schematu; odtworzenie sprawdzone na kopii. Nie stosować demonstracyjnych seedów.

**Testy P:** prawdziwy SQLite w katalogu tymczasowym; ograniczenia, rollback, konflikty zapisu, zerwane połączenie, brak miejsca symulowany kontrolowanie, migracja ze wspieranych wersji, awaria w trakcie oraz integralność odtworzonej kopii.

## 6. Etap 5 — Design System i powłoka PySide6

**Cel:** odtworzenie powłoki i dashboardu UI-REF-01 w PySide6 oraz wspólnego Design Systemu. Układ wzorca jest wymaganiem, nie dowolną propozycją stylistyczną.

**Zależności:** etap 2 oraz zatwierdzone pojęcia z etapu 3; P-05; strategia wykonywania zadań. Można opracowywać prototyp niezależnie od finalnego repozytorium danych.

**Prace P:** zatwierdzić tokeny, motywy i komponenty; stworzyć galerię stanów; powłokę z nawigacją; modele widoków; puste, błędne i ładujące stany; mechanizm zadań i postępu. Przed QFluentWidgets zamknąć bramkę techniczno-licencyjną. Ustalić sprzęt i skalę testów.

**Artefakty P:** galeria, prototyp powłoki, testy dostępności, raport porównania biblioteki i alternatywy, pomiary pętli UI.

**Kryteria ukończenia P:** wszystkie wspólne komponenty działają w obu motywach; preferencja systemowa i ograniczony ruch są obsługiwane; żadna operacja nie blokuje GUI; nie ma demonstracyjnych klientów i kont.

**Testy P:** macierz DPI z systemu UI, Narrator, klawiatura i fokus, duży tekst, wielomonitorowość, pomiar kontrastu i opóźnień. Przekroczenie proponowanego celu wymaga oceny, nie ukrycia wyniku.

## 7. Etap 6 — podstawowe moduły biznesowe

**Cel:** przygotować spójne miejsca pracy dla zatwierdzonych obszarów.

**Zależności:** etapy 3 i 5; etap 4 dla rzeczywistego odczytu. Rozstrzygnięcie K-02 jest wymagane przed traktowaniem modułów jako funkcjonalnych.

**Prace P:** ustalić kolejność modułów; rekomendacja: Katalog → konfiguracja sprzedaży → Oferty → obsługa sprzedaży → komunikacja. Zaprojektować listy, szczegóły, formularze, filtry i kontrakty zapytań. W tym etapie przygotować układ i adaptery prezentacji; nie pisać reguł biznesowych w GUI dla ominięcia etapu 7.

**Artefakty P:** mapy ekranów, makiety, modele prezentacji, scenariusze wejścia/wyjścia i zachowania bez danych.

**Kryteria ukończenia P:** każdy ekran ma spójne stany, zakres i akcję; pole „Dostawca” nie zastępuje „Producent”; aktywne konto jest jawne; brak pozornie działających przycisków. Operacje niewdrożone nie są przedstawiane jako ukończone.

**Testy P:** nawigacja, zachowanie filtrów i fokusu, formularz po błędzie, brak wyników vs pusta instalacja, brak zależności GUI od SQL. Zestawy danych wyłącznie syntetyczne, izolowane od produkcji.

## 8. Etap 7 — przypadki użycia

**Cel:** działające pionowe scenariusze od UI przez aplikację i domenę do portów.

**Zależności:** etapy 3–6; P-06 dla trwałej pracy. Etapy 6 i 7 realizować iteracyjnie w obrębie małego scenariusza; numeracja nie uzasadnia logiki w GUI.

**Prace P:** dla każdego przypadku opisać wejście, wynik, warunki, transakcję, błędy i uprawnienia; podłączyć porty; kontrolować wersje i duplikaty. Rekomendowane pierwsze scenariusze do akceptacji: utworzenie Produktu, zmiana danych, utworzenie szkicu Oferty i odczyt listy. Nie traktować ich jako zgody na publikację zewnętrzną.

**Artefakty P:** kontrakty komend/zapytań, implementacje przypadków i testy przepływu, dokumentacja błędów.

**Kryteria ukończenia P:** GUI nie zna repozytorium; błąd nie gubi formularza; zapis chroni niezmienniki; działania wymagające decyzji pozostają wyłączone. Każdy scenariusz ma potwierdzony wynik biznesowy.

**Testy P:** sukces, błędne wejście, odmowa, konflikt wersji, awaria portu, rollback i ponowne uruchomienie, jeśli dotyczy. Test integracyjny pełnego lokalnego przepływu bez aktywnych API.

## 9. Etap 8 — integracje sprzedażowe

**Cel:** pierwsza wymienna integracja bez podporządkowania jej modelu domenowego.

**Zależności:** etap 7; P-03, P-06 i P-07; kolejność wdrożenia Allegro i PrestaShop, sandbox, autoryzacja oraz kompletne mapowanie pól. Allegro i PrestaShop są wymaganymi integracjami wzorca. Ich kolejność pozostaje otwarta. Subiekt GT pozostaje przyszłym kierunkiem, niewymaganym przez ten ekran.

**Prace P:** kontrakty konektora, DTO, mapowanie statusów i identyfikatorów, paginacja, limity, timeouty, deduplikacja i uzgadnianie wyniku. Osobno testować odczyt/import i skutki zewnętrzne. Utworzyć ścieżkę obsługi konfliktu i niewspieranego statusu.

**Artefakty P:** tabela mapowania, kontrakty integracji, raport autoryzacji i zakresów, procedura awarii oraz testy sandbox.

**Kryteria ukończenia P:** dwa konta są izolowane; import jest powtarzalny bez duplikatów; restart nie gubi pracy; wynik nieznany jest widoczny; odłączenie konta nie kasuje danych. Pierwsze rzeczywiste publikacje wymagają jawnego zakresu pilotażu.

**Testy P:** 401/403, limity, timeout przed i po skutku, błędny JSON, nowy status, duplikaty i zmiana kolejności zdarzeń, cofnięcie autoryzacji. Testy domyślnie nie wykonują prawdziwej sprzedaży ani wysyłki.

## 10. Etap 9 — automatyzacje

**Cel:** kontrolowane reakcje Reguł na Zdarzenia z widocznymi Operacjami.

**Zależności:** etapy 7–8 dla automatyzowanych scenariuszy; zatwierdzona semantyka Reguły, Zdarzenia i Operacji oraz ograniczenia skutków.

**Prace P:** wersjonowanie definicji; walidacja; symulacja bez skutków; aktywacja/pauza; deduplikacja; wykrywanie cykli; ograniczenia pracy i historia przyczyny. Nie udostępniać dowolnego `eval` jako edytora reguł.

**Artefakty P:** specyfikacja języka warunków, macierz przejść wykonania, historia użytej wersji, panel Operacji.

**Kryteria ukończenia P:** każda automatyczna czynność ma przyczynę i wersję reguły; pauza ma określony wpływ na pracę już rozpoczętą; błąd jednej Reguły nie uruchamia nieskończonych ponowień.

**Testy P:** duplikat zdarzenia, pętla, konflikt Reguł, restart, równoległe wykonanie, błędna definicja i nieznana wersja payloadu. Symulacja nie wysyła nic do systemu produkcyjnego.

## 11. Etap 10 — analityka

**Cel:** wiarygodne wskaźniki o jasno opisanym znaczeniu.

**Zależności:** stabilne dane z etapów 7–9 i zatwierdzone definicje metryk. Nie obliczać przychodu bez rozstrzygnięcia walut, anulowań, podatków i zakresu dat.

**Prace P:** słownik metryk; modele odczytu/projekcje; filtrowanie po kontekście; czas ostatniej aktualizacji; uzgadnianie wyników z danymi źródłowymi; eksport z kontrolą zakresu danych osobowych.

**Artefakty P:** definicje liczników i mianowników, źródła pól, reguły czasu i walut, pulpity z pustymi stanami.

**Kryteria ukończenia P:** wskaźnik jest odtwarzalny; brak danych różni się od zera; waluty nie są mieszane; projekcja ma opis aktualności. Nie wstawiać fikcyjnych wyników dla nowej instalacji.

**Testy P:** granice dat i stref, anulowania, korekty, różne waluty, porównanie projekcji z wyliczeniem referencyjnym oraz koszt zapytań na zatwierdzonym wolumenie.

## 12. Etap 11 — testy wydajności i bezpieczeństwa

**Cel:** kwalifikacja całości i usunięcie ryzyk przed dystrybucją; kontrole narzędziowe działają już od etapu 1.

**Zależności:** wdrożony zakres wydania, sprzęt i dane referencyjne, model zagrożeń, polityka retencji i kopii.

**Prace P:** pomiary list, wyszukiwania, importu i długich Operacji; analiza pamięci; blokady SQLite; awarie i odtworzenie; przegląd tokenów, logów i załączników; audyt zależności i licencji; kontrola granic architektonicznych.

**Artefakty P:** raport z warunkami pomiaru, wyniki wszystkich siedmiu narzędzi jakości, rejestr ryzyk i wyjątków, raport przywrócenia danych.

**Kryteria ukończenia P:** spełnione zatwierdzone cele; żadna znana krytyczna luka bez rozstrzygnięcia; wyjątki mają właściciela i termin; brak sekretów w diagnostyce; rzeczywiście sprawdzone odtworzenie. Priorytety ryzyka i zasady blokowania wydania wymagają zatwierdzenia.

**Testy P:** obciążenie i długotrwała praca, przerwanie zasilania symulowane na kopii, pełny dysk, uszkodzony import, niebezpieczna ścieżka załącznika, odmowa dostępu do magazynu sekretów, pomiary GUI podczas synchronizacji.

## 13. Etap 12 — dystrybucja Windows

**Cel:** instalowalne, aktualizowalne i obsługiwalne wydanie.

**Zależności:** etap 11, P-09, komplet licencji i wspierana macierz Windows. Narzędzie pakowania i sposób aktualizacji są NIEUSTALONE.

**Prace P:** reproducowalny pakiet, metadane wersji, podpisywanie według przyjętej polityki, zasoby i licencje, instalacja bez narzędzi developerskich, aktualizacja schematu, diagnostyka i instrukcja wsparcia. Ustalić zachowanie deinstalacji względem danych użytkownika.

**Artefakty P:** instalator/pakiet, sumy kontrolne, wykaz zależności i licencji, release notes, instrukcja instalacji/aktualizacji/odtworzenia.

**Kryteria ukończenia P:** działa na czystej wspieranej maszynie; nowa instalacja ma pusty katalog biznesowy; aktualizacja zachowuje dane i preferencje; niezgodna wersja nie uszkadza schematu; deinstalacja nie usuwa danych bez świadomej decyzji. Procedura powrotu uwzględnia zgodność schematu, nie tylko starszy plik EXE.

**Testy P:** czysta instalacja, aktualizacja każdej wspieranej wersji, brak uprawnień administratora w normalnej pracy, ścieżki ze spacjami i polskimi znakami, Defender według polityki organizacji, DPI, start offline, przerwana aktualizacja, odtworzenie kopii i deinstalacja.

## 14. Zależności i zakres pierwszego wydania

Ścieżka bazowa: 1 → 2 → 3 → 4; etap 5 po 2 i przeglądzie 3; następnie iteracja 6↔7 → 8 → 9 → 10 → 11 → 12. Zakres 9–10 może trafić do późniejszych wydań wyłącznie po jawnej decyzji; nie jest to automatyczna zmiana wymaganego planu.

Nie zatwierdzono MVP. Propozycja pierwszego przyrostu do rozmowy: lokalny Katalog, szkic Oferty, pusta instalacja, kopia/odtworzenie i wspólna powłoka, bez skutków zewnętrznych. Kolejny przyrost może wprowadzić jedną integrację. Decyzję o zakresie powiązać z P/Q/K w [ADR_INDEX](ADR_INDEX.md), nie z domyślnym wyborem platformy.

## 15. Definicja ukończenia przyrostu — propozycja

Przyrost ma zatwierdzony scenariusz, zgodny model, właściwe źródła prawdy, testy i kontrole jakości, obsługę błędów i awarii, dostępne UI oraz aktualną dokumentację. Wykonane sprawdzenia mają dowód; niewykonane są wymienione. Nie przenosić niejawnie długu do późniejszego etapu. Ukończenie wymaga przeglądu, a nie wyłącznie działającej demonstracji.

## 16. Aktualizacja zakresu według UI-REF-01

WYMAGANIE: docelowy interfejs i wszystkie widoczne opcje mają odpowiadać [ekranowi użytkownika](assets/UI_REFERENCE.png). Dokładna specyfikacja jest w UI_UX_DESIGN_SYSTEM, sekcje 15–23. Układ nie wymaga ponownego wyboru kierunku stylistycznego. Zadania i szczegóły techniczne poniżej są P, a wymagany zakres funkcji wynika z polecenia aktualizacji.

### 16.1. Włączenie do 12 istniejących etapów

| Etap | Dodatkowe prace | Dodatkowa bramka i dowód |
|---|---|---|
| 1 — Środowisko | Dodać referencję do dokumentacji; ustalić narzędzie porównań wizualnych i bibliotekę wykresów; źródła ikon/logo | Zapis źródła UI-REF-01; brak narzucenia Pythona 3.10.12 i limitów z makiety |
| 2 — Fundament | Porty modeli odczytu, generatora AI, importu pliku, magazynu mediów i pracy w tle | GUI nie importuje parsera, klienta AI, SQLite ani konektora platformy |
| 3 — Model | Cenniki, źródła cen i ilości, media, układy, akceptacja AI, KPI i problemy | Rozstrzygnięte ADR-013–ADR-016 dla implementowanego przyrostu; brak ukrytego siedemnastego bytu |
| 4 — Baza | Projekcje dashboardu, trwałość Operacji, dane zatwierdzonych rozszerzeń | Pusta instalacja; migracja i odtworzenie; brak nazw kont i producentów z obrazu |
| 5 — Design System | Odtworzyć nagłówek, menu, KPI, pasek działań, panele, stopkę, miniatury i stany | Porównanie 1536×1024 z referencją, DPI, jasny/ciemny, klawiatura i Narrator |
| 6 — Moduły | Cenniki, media, AI, edycja masowa, ceny, szablony, podstrony platform, kolejka, raporty i pomoc | Każda pozycja wzorca ma poprawną trasę i spójny widok; brak fałszywych sukcesów |
| 7 — Przypadki użycia | Podłączyć wyszukiwanie, kreatory, import, podgląd zmian, walidację, akceptację opisów i wyniki | Test pełnego lokalnego scenariusza, konfliktu wersji i zachowania formularza po błędzie |
| 8 — Integracje | Allegro i PrestaShop; pobieranie, publikacja, synchronizacja cen, stany połączenia i limitów | Dwa konektory w planie; sandboxy, wiele kont, częściowe połączenie, wynik nieznany |
| 9 — Automatyzacja | Kolejka dla importu, cen, AI, mediów, publikacji i walidacji, gdzie praca jest długotrwała | Restart, deduplikacja, bezpieczne ponowienie; brak autopublikacji AI |
| 10 — Analityka | Cztery KPI, historia, wykres dwóch serii, okresy, tooltip i tabela danych | Definicje metryk, pusta baza, brak porównania, anulowania, strefy i waluty |
| 11 — Kwalifikacja | Testy dużej galerii, masowych zmian, złośliwych plików/HTML i danych wejścia AI | Brak wycieku sekretów; UI reaguje podczas pracy; obraz/AI traktowane jako dane |
| 12 — Dystrybucja | Spakować zasoby, ikony, wykresy i licencje; sprawdzić świeży profil | Wygląd po instalacji zgodny z wzorcem bez obecności demonstracyjnych rekordów |

### 16.2. Proponowane pionowe przyrosty

**A — wierna powłoka i dashboard lokalny.** Zależności: zatwierdzony graf importów, tokeny i modele odczytu. Zakres: menu ze wzorca, globalne szukanie, cztery KPI, ostatnie Produkty, Operacje, dolne panele i stopka; stany puste i niepołączone. Dane demonstracyjne tylko do odrębnego testu wizualnego. Odbiór: porównanie wizualne i brak fałszywych aktywnych integracji.

**B — przygotowanie katalogu i ofert.** Zależności: model cen/mediów/układów i lokalne przypadki użycia. Zakres: Cenniki/import, galeria, nowe Oferty i szkice, masowa edycja, zmiana cen, kreator układu i kopiowanie, sprawdzenie błędów. Odbiór: podgląd, walidacja, zapis i raport per obiekt; brak skutków zewnętrznych bez potwierdzenia.

**C — integracje i kolejka.** Zależności: sandboxy, macierz źródeł, bezpieczeństwo kont i trwałe Operacje. Zakres: Allegro i PrestaShop, pobieranie, publikacja, uzgadnianie cen, karty integracji i historia. Odbiór: timeout po skutku, wiele kont, duplikat, przerwanie/restart i częściowy sukces.

**D — AI i analityka.** Zależności: port generatora, wybrany dostawca i warunki danych; definicje metryk. Zakres: kreator opisów i kafel AI, świadoma akceptacja, raporty, wykres i delty KPI. Odbiór: brak automatycznej publikacji, odrzucenie wyniku bez zmiany Produktu, metryki zgodne z danymi, obsługa braku porównania.

Przyrosty są propozycją kolejności, nie usunięciem części funkcji. Wcześniejszy pomysł MVP z sekcji 14 nie definiuje kompletności nowego zakresu. Pełna zgodność z ekranem wymaga funkcjonalnych wejść wszystkich jego opcji. Harmonogram i granice wydań pozostają otwarte.

### 16.3. Rejestr odbioru wzorca

Dla każdej opcji prowadzić wpis: identyfikator funkcji, element ekranu, przypadek użycia, model/port, stan implementacji, test, wynik i różnice. Obowiązkowe przypadki: nowa baza; podłączone dwa systemy; tylko jeden dostępny; wiele kont; brak sieci; błędny import; częściowa edycja masowa; nieznany wynik publikacji; niedostępny AI; brak pliku zdjęcia; 200% DPI. Wydanie nie może deklarować pełnej zgodności, gdy kafle jedynie wyświetlają „gotowe” bez wykonania.
