# CENTRUM OFERTOWE — konstytucja architektoniczna

Wersja 0.3 • 2026-11-02 • dokument projektowy do przeglądu.

## 1. Pochodzenie i zakres norm

Projekt jest planowany od zera na podstawie wymagań użytkownika: Windows, Python, PySide6, SQLite, sześć warstw, 16 bytów oraz Fluent Design. Plan jest samodzielny i nie wymaga odnajdywania ani uzupełniania dawnych decyzji lub poprzedniej dokumentacji.

Pakiet jest dokumentacją nowego projektu, a nie opisem istniejącej implementacji. Wszystkie nowe rozwiązania techniczne w sekcjach 3–13 są PROPOZYCJAMI, chyba że nazwano je WYMAGANIEM. Nowe decyzje mają własne identyfikatory ADR-001 i kolejne.

Definicje statusów i procedura zatwierdzania: [AGENTS](AGENTS.md). Rejestr braków i konfliktów: [ADR_INDEX](ADR_INDEX.md).

## 2. Wymagania nadrzędne

CENTRUM OFERTOWE jest długoterminową aplikacją Windows do zarządzania procesami sprzedaży. Pierwszeństwo ma spójny model domenowy, a dopiero potem integracje. Obowiązkowy stos: Python, PySide6, SQLite, Clean Architecture, DDD, pytest, ruff, mypy, import-linter, bandit i pip-audit.

System zawiera sześć warstw: `rdzen`, `domena`, `przypadki_uzycia`, `adaptery`, `kanaly`, `gui`. Zależności prowadzą do wewnątrz. Domena nie zna UI, bazy i API. GUI nie zawiera logiki biznesowej. Model obejmuje dokładnie 16 wymaganych głównych bytów wymienionych w [DOMAIN_MODEL](DOMAIN_MODEL.md); techniczne DTO, obiekty wartości i tabele łącznikowe nie są automatycznie nowymi głównymi bytami.

## 3. Proponowany kontrakt warstw

| Warstwa | Odpowiedzialność | Dozwolone zależności wewnętrzne | Zakazy |
|---|---|---|---|
| `rdzen` | Minimalne, neutralne typy techniczne: identyfikacja korelacji, kontrakt zegara, wspólna kategoria błędu | Brak | Qt, SQL, HTTP, encje i reguły biznesowe |
| `domena` | Encje, obiekty wartości, niezmienniki, zdarzenia domenowe, usługi domenowe | `rdzen` | Pozostałe warstwy, serializacja API, zapis bazy |
| `przypadki_uzycia` | Orkiestracja, komendy, zapytania, porty, DTO wejścia i wyniku | `domena`, `rdzen` | Konkretne repozytoria i komponenty Qt |
| `adaptery` | SQLite, pliki, logowanie, magazyn sekretów, wykonanie zadań; implementacje portów | `przypadki_uzycia`, `domena`, `rdzen` | `gui`, `kanaly`; decyzje biznesowe |
| `kanaly` | Integracje sprzedażowe, transport API, mapowanie, ochrona modelu przed semantyką dostawcy | `przypadki_uzycia`, `domena`, `rdzen` | `gui`, `adaptery`; bezpośredni SQL |
| `gui` | Widoki, modele widoków, modele tabel Qt, nawigacja i Design System | `przypadki_uzycia`, `rdzen` | `adaptery`, `kanaly`, bezpośredni import encji `domena`, SQL |

To graf częściowy: trzy zewnętrzne warstwy są równorzędnymi adapterami, a nie trzema szczeblami. GUI korzysta z DTO przypadków użycia; nie przekazuje widżetów do aplikacji. Integracja otrzymuje gotowe porty, np. transportu, przez konstruktor. Nie importuje implementacji transportu z `adaptery`.

PROPOZYCJA: zewnętrzny punkt składania `bootstrap.py` w pakiecie aplikacji konstruuje adaptery, integracje i GUI. Nie jest siódmą warstwą biznesową. Jako jedyny może importować wszystkie sześć warstw; żadna z nich nie importuje bootstrapu. Jeśli właściciel wymaga absolutnie wszystkich plików wewnątrz sześciu katalogów, trzeba najpierw rozstrzygnąć K-01 w rejestrze.

Import-linter powinien egzekwować osobne zakazy dla warstw równorzędnych, zakazy frameworków w centrum oraz brak zależności do bootstrapu. Sam jeden kontrakt `layers` ustawiający wszystkie sześć katalogów w szeregu jest niewystarczający. Weryfikacja obejmuje też importy pośrednie, dynamiczne importy, rejestry pluginów i adnotacje typów. Test kontraktu powinien wykazać, że celowo dodany zabroniony import powoduje błąd, po czym usunąć próbne naruszenie.

## 4. Model i granice odpowiedzialności

Agregat chroni lokalny niezmiennik i stanowi granicę atomowego zapisu. Nie jest synonimem tabeli, modułu ani całej funkcji biznesowej. Relacje pomiędzy agregatami przechowują identyfikatory, bez kaskadowego ładowania całego grafu. Proponowane granice są w DOMAIN_MODEL i wymagają akceptacji przed schematem.

Obiekty wartości opisują znaczenie, walidują się przy tworzeniu i nie mają własnego cyklu życia: proponowane przykłady to kwota z walutą, adres, identyfikator zewnętrzny z przestrzenią nazw, zakres dat. Nie wprowadzać `float` dla kwot; propozycja reprezentacji to `Decimal` w domenie i uzgodniony tekst dziesiętny lub liczba jednostek minimalnych w SQLite. Reguły zaokrągleń i podatków są NIEUSTALONE.

Usługa domenowa jest dopuszczalna tylko dla reguły, która nie należy naturalnie do jednej encji. Przypadek użycia ustala kolejność, transakcję i obsługę błędów, ale nie kopiuje niezmienników encji. Repozytorium nie podejmuje decyzji o legalności przejścia biznesowego na podstawie własnych flag SQL.

SOLID stosować przez konkretne ograniczenia: jedna odpowiedzialność modułu; małe porty dla potrzeb klienta; wymienne implementacje przestrzegające tych samych testów kontraktowych; jawne zależności przez konstruktor; rozszerzanie integracji bez warunków `if allegro` w Produkcie. Nie tworzyć interfejsu dla każdej klasy bez rzeczywistej potrzeby izolacji.

## 5. Źródła prawdy i konflikty

WYMAGANIE: każdy byt ma jawnego właściciela danych i źródło prawdy. PROPOZYCJA: właścicielem reguł jest wskazany kontekst biznesowy, a odpowiedzialnym za zatwierdzenie reguł — właściciel produktu; osoby i role organizacyjne nie zostały wskazane.

Nie wystarcza stwierdzenie „SQLite jest źródłem prawdy”: SQLite jest miejscem przechowywania. Dla każdego synchronizowanego pola sporządzić kontrakt: właściciel, kierunek synchronizacji, identyfikator źródła, wersja, czas obserwacji, sposób rozwiązania konfliktu. Lokalny opis Produktu może być własnością katalogu, a status publikacji Oferty obserwacją platformy; nie wolno automatycznie nadpisywać jednego drugim.

PROPOZYCJA: konflikt staje się jawnym wynikiem przypadku użycia z obiema wersjami i dostępnymi akcjami. „Ostatni zapis wygrywa” nie jest domyślną regułą. Identyfikator zewnętrzny ma zakres co najmniej systemu i odpowiedniego konta; sam numer otrzymany z API nie jest globalnym ID encji.

## 6. Przypadki użycia i transakcje

Proponowany przebieg komendy:

1. Zweryfikować format wejścia i kontekst operacji.
2. Wczytać potrzebne agregaty przez porty.
3. Wykonać metodę domenową z potwierdzonymi niezmiennikami.
4. Zapisać stan i wymagane dane o pracy do wykonania w jednej lokalnej transakcji.
5. Zatwierdzić transakcję; zwrócić wynik bez obiektów Qt i połączeń bazy.
6. Uruchomić efekt zewnętrzny poza transakcją, jeżeli scenariusz go wymaga.

Nie utrzymywać blokady SQLite podczas żądania sieciowego ani oczekiwania na dialog użytkownika. Dla zmian lokalnych i zewnętrznych nie obiecywać jednej atomowej transakcji. PROPOZYCJA: trwała kolejka/outbox oraz Operacja reprezentująca stan pracy, z deduplikacją i wznowieniem. To rozwiązanie wymaga odrębnej decyzji; nie zakłada wdrożenia pełnego event sourcingu.

Zapytania mogą używać wyspecjalizowanych modeli odczytu przez porty aplikacyjne. GUI nie musi odtwarzać agregatu, by wyświetlić listę. Projekcje są odtwarzalnymi widokami, nie konkurencyjnym źródłem reguł. Nowa projekcja wymaga opisu opóźnienia aktualizacji.

## 7. SQLite i migracje

SQLite jest WYMAGANIEM. PROPOZYCJE operacyjne:

- Baza w lokalnym katalogu danych użytkownika, poza katalogiem instalacji i repozytorium; dokładna ścieżka oraz obsługa wielu profili pozostają otwarte.
- Połączenie ma jednego właściciela wątku. Tworzenie i zwalnianie odbywa się w tym samym kontekście wykonania; widżet nigdy nie przechowuje połączenia.
- Włączać i testować egzekwowanie kluczy obcych na każdym połączeniu. Parametryzować wartości SQL; nazwy sortowania wybierać z jawnej listy.
- Jawne transakcje zapisu i krótki czas blokady. Strategię WAL, timeout blokad, kolejkę zapisów i poziom trwałości zatwierdzić na podstawie testów, nie domyślnie.
- Nie trzymać aktywnej bazy na współdzielonym dysku sieciowym ani traktować synchronizacji pliku przez chmurę jako rozwiązania wielostanowiskowego. Ewentualna obsługa wielu stanowisk wymaga osobnej architektury.
- Wersja schematu, monotoniczny rejestr migracji, suma kontrolna skryptu, informacja o wyniku i wspieranych wersjach aplikacji. Zastosowanej migracji nie edytować po publikacji.

Procedura aktualizacji: ustalić wersję → wykluczyć równoległy zapis → sprawdzić wolne miejsce i możliwość kopii → wykonać spójną kopię wspieranym mechanizmem → zweryfikować możliwość odczytu kopii → zastosować migrację → sprawdzić integralność i niezmienniki → dopiero wtedy uruchomić UI biznesowe. Szczegóły zależą od wybranego narzędzia migracji, które jest NIEUSTALONE.

W razie błędu zatrzymać otwieranie danych, zachować diagnostykę bez danych osobowych i zaproponować odtworzenie. Nie usuwać automatycznie bazy. Przywrócenie starej kopii może utracić zmiany wykonane po kopii — procedura musi to jasno komunikować. Kopiowanie jednego aktywnego pliku przy WAL nie stanowi zatwierdzonego sposobu kopii.

WYMAGANIE pustej instalacji dotyczy danych biznesowych. Schemat, rejestr migracji i techniczne preferencje mogą istnieć; nie wolno dodawać producentów, dostawców, klientów, kont, podmiotów GPSR, ofert i cenników. Klasy integracji i ich techniczne możliwości nie oznaczają automatycznego utworzenia rekordów KanałSprzedaży.

## 8. Integracje i odporność

Każdy konektor ma warstwę tłumaczenia: zewnętrzne DTO → walidacja → mapowanie → komenda aplikacyjna. Surowy JSON może być materiałem diagnostycznym jedynie po określeniu retencji i ochrony danych. Nie staje się modelem Produktu.

Przed pierwszą integracją uzgodnić: wersję API, zakresy uprawnień, limity, politykę paginacji, mapowanie statusów, identyfikację duplikatów, synchronizację usunięć, zachowanie przy nieznanej wartości i właściciela każdego pola. Nieznany status nie może być po cichu mapowany na sukces.

PROPOZYCJA klasyfikacji błędów: wejście niepoprawne, brak uwierzytelnienia, brak uprawnień, konflikt wersji, limit, błąd przejściowy, błąd trwały, wynik nieznany. Automatyczne ponowienie wymaga oceny idempotencji. Po timeout publikacji najpierw uzgodnić stan po stronie platformy; ślepe ponowienie może utworzyć duplikat.

Poświadczenia izolować per konto. Ograniczać uprawnienia do potrzeb scenariusza. Środowisko testowe musi być wyraźnie odróżnione od produkcyjnego; zadania testowe nie używają aktywnych kont użytkownika.

## 9. GUI i współbieżność

WYMAGANIE: PySide6, wspólne komponenty, Fluent Design, jasny/ciemny/systemowy motyw, dostępność i DPI. Szczegóły: [UI_UX_DESIGN_SYSTEM](UI_UX_DESIGN_SYSTEM.md).

PROPOZYCJA: widok obsługuje układ, model widoku stan prezentacji, przypadek użycia reguły procesu, domena niezmienniki. Operacje sieciowe, migracje i ciężkie zapytania wykonują się poza pętlą UI. Wynik dociera przez bezpieczną kolejkę/sygnał do wątku GUI. Worker nie modyfikuje widżetów i modeli Qt należących do GUI.

Anulowanie jest żądaniem, nie gwarancją cofnięcia. Oddzielić „przerwano oczekiwanie”, „nie wykonano” i „wykonano zewnętrznie”. Przy zamykaniu aplikacji ustalić politykę aktywnych Operacji i zapisu postępu; zakaz gwałtownego ubijania wątku jako zwykłego sposobu anulowania.

## 10. Bezpieczeństwo i obserwowalność

PROPOZYCJE wymagające doprecyzowania przed danymi rzeczywistymi:

- Model zagrożeń obejmuje kradzież plików lokalnych, wyciek tokenu, niebezpieczny załącznik, błędne ponowienie publikacji i manipulację plikiem importu.
- Magazyn sekretów Windows za portem; wybór mechanizmu jest otwarty. Standardowa baza SQLite nie oznacza automatycznego szyfrowania.
- Log strukturalny: czas, poziom, identyfikator korelacji i Operacji, typ błędu, wersja aplikacji. Bez treści wiadomości, tokenów i pełnych adresów klientów.
- Załączniki: limity wielkości, bezpieczna nazwa, kontrola ścieżki docelowej, brak automatycznego wykonania. Format pliku nie jest potwierdzony samym rozszerzeniem.
- Retencja i usuwanie danych osobowych, dokumentów i audytu wymagają decyzji właściciela; ten pakiet nie ustanawia okresów prawnych.
- Audyt zmian biznesowych jest odrębny od logu technicznego. Dostęp do audytu i możliwość korekty muszą być ustalone.

Każda podatność z bandit/pip-audit wymaga oceny zastosowania. Wyjątek zawiera powód, zakres wersji, właściciela, zabezpieczenie i termin. Brak podatności raportowanych przez narzędzia nie dowodzi bezpieczeństwa produktu.

## 11. Jakość i dowody

Testy domeny nie uruchamiają Qt, SQL i sieci. Testy aplikacyjne używają kontrolowanych portów i zegara. Repozytoria sprawdza się także na rzeczywistym SQLite; mock nie potwierdza ograniczeń bazy. Integracje mają testy kontraktowe mapperów i kontrolowane odpowiedzi transportu. GUI wymaga automatycznych testów krytycznych interakcji oraz ręcznej oceny dostępności i układu.

Proponowane katalogi testów: `unit`, `application`, `integration`, `contracts`, `gui`, `migration`, `performance`. Ostateczny podział zatwierdzić w etapie 1. Przebieg CI ma obejmować pytest, ruff check/format, mypy, import-linter, bandit i pip-audit. Próg pokrycia nie został ustalony; ważniejsze są testy każdego zatwierdzonego niezmiennika i scenariusza awarii niż arbitralny procent.

Każdy miernik wydajności musi mieć sprzęt, skalę danych, scenariusz, rozgrzanie i statystykę pomiaru. Celów UI z Design Systemu nie traktować jak wyników pomiaru.

## 12. Dług techniczny i ewolucja

PROPOZYCJA rejestru długu: identyfikator, ograniczenie, przyczyna, ryzyko dla użytkownika, właściciel, zakres dotknięty, kryterium usunięcia i termin przeglądu. Komentarz TODO bez właściciela nie jest planem.

Zmiana granic agregatu, właściciela danych, źródła prawdy, strategii migracji, biblioteki UI, sposobu uwierzytelniania lub dystrybucji wymaga propozycji ADR przed implementacją. Refaktoryzacja bez zmiany znaczenia może być zatwierdzona przeglądem kodu. Nie zamieniać każdej drobnej korekty w osobny ADR.

## 13. Kryterium zgodności architektonicznej

Zmiana jest zgodna, gdy ma powiązanie z wymaganiem, nie narusza grafu zależności, nie tworzy drugiego źródła prawdy, chroni zatwierdzone niezmienniki i ma odpowiedni dowód testowy. Gdy wymaga odstępstwa, musi istnieć jawna decyzja z konsekwencjami i planem wyjścia. Nie wolno „tymczasowo” obejść granic architektury bez odnotowania odstępstwa.

## 14. Rozwinięcie architektury o dashboard UI-REF-01

WYMAGANIE: [wzorzec użytkownika](assets/UI_REFERENCE.png) jest docelowym wyglądem i mapą widocznych opcji. Dokładny układ i kontrakty interakcji zawiera system UI. Dotychczasowa ogólna stylistyka Fluent zostaje uszczegółowiona przez ten obraz; sześć warstw i niezależność domeny pozostają wymagane.

PROPOZYCJA podziału odpowiedzialności nowych obszarów:

| Obszar | `domena` | `przypadki_uzycia` | Warstwy zewnętrzne |
|---|---|---|---|
| Dashboard | Znaczenie danych i zatwierdzone reguły statusów | Zapytania KPI, produktów, operacji, wykresu i integracji | `adaptery`: modele odczytu SQLite; `gui`: kafle i wykres |
| Cenniki i ceny | Semantyka kwoty, reguły zmiany po zatwierdzeniu | Import, podgląd, walidacja, zastosowanie i synchronizacja | `adaptery`: parser pliku; `kanaly`: API cen; `gui`: różnice i wynik |
| Media | Referencje i zasady użycia zasobu po zatwierdzeniu | Dodanie, przypisanie, kolejność, usunięcie referencji | `adaptery`: pliki/miniatury; `gui`: galeria |
| Opisy AI | Akceptowana treść Produktu/Oferty | Żądanie propozycji, porównanie, akceptacja | `adaptery`: dostawca AI za portem; `gui`: edytor |
| Układy/szablony | Znaczenie i wersja układu po decyzji | Zapis, kopia, podgląd, przypisanie | `kanaly`: ograniczenia formatu; `gui`: kreator |
| Masowa edycja | Niezmiennik każdego obiektu | Podgląd zmian, kontrola wersji, wykonanie, raport częściowy | `gui`: wybór zakresu; `adaptery`: trwały zapis Operacji |
| Walidacja błędów | Zatwierdzone niezmienniki | Zbiorczy wynik z kodem, polem, obiektem i wagą | `gui`: lista problemów i wejście do korekty |

To przypisanie jest P i nie oznacza automatycznego utworzenia nowych agregatów. Konektor AI nie jest KanałemSprzedaży i należy do infrastruktury za portem. Konektory Allegro i PrestaShop pozostają w `kanaly`. Przewoźnicy nie są kanałami sprzedaży.

### 14.1. Modele odczytu zamiast reguł w widżetach

PROPOZYCJA: oddzielne zapytania aplikacyjne dla KPI, listy ostatnich Produktów, ostatnich Operacji, serii wykresu, stanu integracji i wyników wyszukiwania. Wynik ma zakres kont, filtry, czas obserwacji i informację o błędzie/nieaktualności. GUI jedynie renderuje DTO. Nazwy konkretnych portów pozostają do opracowania.

Status Produktu na Allegro/PrestaShop jest projekcją powiązanych Ofert, nie flagą biznesową `produkt.jest_na_allegro`. Cena i dostępność to projekcje określonego źródła, nie pola dodawane bez analizy. Łączny licznik błędów nie jest nowym agregatem — proponowany model odczytu zbiera problemy walidacji, integracji i Operacji, z deduplikacją.

Dashboard może odświeżać panele niezależnie. Awaria wykresu nie blokuje katalogu. Wyniki mają wersję żądania; spóźnione zapytanie nie zastępuje nowszego zakresu. P: aktualizacje po lokalnym zapisie i kontrolowany polling integracji; interwały i cache są otwarte. Nie wykonywać osobnego żądania API dla każdej komórki tabeli.

### 14.2. Edycja masowa i skutki zewnętrzne

P: sekwencja wybór zakresu → odczyt wersji → podgląd zmian → walidacja → potwierdzenie → wykonanie → raport per obiekt. Zmiana filtra po podglądzie nie może rozszerzyć potajemnie zakresu wykonania. Snapshot zakresu i wersji chroni intencję użytkownika. Atomowość wielu lokalnych rekordów jest decyzją scenariusza; atomowość wielu publikacji API nie jest gwarantowana.

Importer cennika najpierw parsuje i waliduje dane bez nadpisania katalogu; następnie przedstawia mapowanie i różnice. Parser jest adapterem. Wymagana obsługa błędnego pliku i przerwania; nie zakładać formatu `products.json` na podstawie jednego komunikatu na screenie.

### 14.3. AI i bezpieczna treść

P: port generatora otrzymuje wybrane dane produktu oraz instrukcję redakcyjną, zwraca propozycję i metadane przebiegu. Dostawca, model, koszty, przechowywanie danych i limity pozostają otwarte. Przed zewnętrznym wysłaniem pokazać zakres danych i wymaganą konfigurację. Brak dostawcy oznacza widoczne „Skonfiguruj AI”, nie fikcyjny opis.

Treść produktu, pliku, wyniku AI i HTML jest danymi, nie poleceniami sterującymi aplikacją. Nie może uruchamiać narzędzi, zmieniać konta, pobierać sekretu ani publikować. Propozycję akceptuje użytkownik. Zapisać, jaka wersja opisu została przyjęta, bez logowania wrażliwego promptu. Edytor szablonów korzysta z ograniczonej deklaratywnej struktury; nie wykonuje dowolnego Python/JavaScript z szablonu.

### 14.4. Testy architektury rozszerzeń

Wymagane scenariusze P: KPI z pustej bazy; mieszane statusy wielu kont; API jednego kanału niedostępne; nieaktualna projekcja; błędny import bez częściowego nadpisania; zmiana wersji obiektu po podglądzie masowym; AI odrzucone przez użytkownika; prompt z treścią udającą instrukcję; niebezpieczny HTML w szablonie; przerwanie i restart kolejki. Zatwierdzone definicje metryk mają osobne testy, a nie tylko zrzuty wizualne.
