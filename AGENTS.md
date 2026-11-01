# CENTRUM OFERTOWE — instrukcje dla agentów

Wersja pakietu: 0.3, 2026-11-02. Status: projekt do przeglądu; nie jest potwierdzeniem wdrożenia ani zatwierdzeniem nowych decyzji.

## 1. Zakres i moc instrukcji

Ten plik określa sposób pracy w przyszłym repozytorium CENTRUM OFERTOWE. W obecnym zadaniu wolno przygotować wyłącznie dokumentację. Nie tworzyć funkcjonalności, migracji, instalatorów ani konfiguracji udającej istniejącą aplikację.

Projekt jest planowany od zera. Nie szukać dawnych decyzji ani nie uzależniać pracy od poprzedniej dokumentacji. Nie traktować zdań oznaczonych jako propozycje jak zgody na zmianę modelu. Podczas przyszłej pracy w repozytorium zachowywać zmiany użytkownika.

## 2. Czytanie dokumentacji i rozstrzyganie rozbieżności

Kolejność analizy: ten plik, [README](README.md), [konstytucja](ARCHITECTURE_CONSTITUTION.md), [model](DOMAIN_MODEL.md), [rejestr decyzji](ADR_INDEX.md), [system UI](UI_UX_DESIGN_SYSTEM.md), [roadmapa](ROADMAP.md). Kolejność czytania nie nadaje nowej propozycji pierwszeństwa przed zatwierdzoną decyzją.

W całym pakiecie obowiązują statusy:

| Oznaczenie | Znaczenie | Skutek dla agenta |
|---|---|---|
| WYMAGANIE | Wynika bezpośrednio z polecenia i rozmowy źródłowej | Zachować; zgłaszać sprzeczności |
| PROPOZYCJA | Nowy szczegół projektowy, reguła lub parametr | Nie implementować jako zatwierdzony bez decyzji właściciela |
| NIEUSTALONE | Brak źródła lub potwierdzenia | Nie dopowiadać znaczenia |
| KONFLIKT | Dwie zasady wymagają rozstrzygnięcia | Opisać obie strony, wpływ i rekomendację |

Zatwierdzenie musi mieć wskazaną osobę, datę, zakres i odnośnik do decyzji. Samo wygenerowanie pliku, jego obecność w repozytorium lub zapis słowa „musi” nie stanowią dowodu zatwierdzenia propozycji. Nowy plan korzysta wyłącznie z własnego rejestru ADR-001 i kolejnych numerów; nie wymaga odtwarzania dawnych decyzji.

## 3. Obowiązkowa analiza przed kodowaniem

Przed zmianą ustalić:

1. Jaki scenariusz użytkownika ma się zmienić i po czym poznamy sukces.
2. Które z 16 pojęć domenowych biorą udział; kto jest właścicielem każdego zmienianego pola.
3. Które niezmienniki są potwierdzone, a które dopiero proponowane.
4. Jaka jest granica agregatu i transakcji; czy potrzebny jest efekt zewnętrzny.
5. Jakie porty przypadków użycia są potrzebne i jak odizolować zewnętrzne DTO.
6. Które warstwy zostaną zmienione i jakie importy pojawią się po zmianie.
7. Jakie są konsekwencje dla migracji, odtworzenia danych, testów i dostępności UI.
8. Czy zadanie wymaga decyzji z rejestru otwartych zagadnień.

Jeżeli zadanie dotyka niezatwierdzonego modelu, najpierw przygotować konkretną propozycję: scenariusz, zmieniane pojęcia, alternatywy, przykładowe dane syntetyczne, niezmienniki oraz kryteria akceptacji. Kontynuować niezależne prace, ale nie utrwalać spornej relacji w schemacie SQLite ani publicznym API.

## 4. Nienaruszalne wymagania

- Najpierw model domenowy, potem integracje. Produkt jest niezależny od Allegro, PrestaShop, Subiekt GT i innych dostawców.
- Oferta prezentuje Produkt. Nie jest drugim, konkurencyjnym katalogiem produktów.
- Producent, Dostawca i PodmiotGPSR są odrębnymi rolami. KanałSprzedaży i KontoSprzedażowe są odrębnymi pojęciami. Przewoźnik nie jest KanałemSprzedaży.
- W dokumentach i komunikatach stosować określenie „Środek komunikacji”.
- Python, PySide6, SQLite; Clean Architecture i DDD. Warstwy: `rdzen`, `domena`, `przypadki_uzycia`, `adaptery`, `kanaly`, `gui`.
- Domena nie importuje PySide6, SQLite, klientów HTTP ani modeli zewnętrznych API. GUI nie zawiera reguł biznesowych i nie wykonuje SQL.
- Integracja tłumaczy modele zewnętrzne na wspólny model; nie tworzy własnego Produktu, Klienta ani Zamówienia jako konkurencyjnego źródła prawdy.
- Nowa instalacja ma zero rekordów biznesowych. Testowe podmioty, konta i cenniki występują wyłącznie w izolowanych fixture'ach; żaden mechanizm startu nie zasila nimi produkcji.

## 5. Granice kodu i styl pracy

Dokładny kontrakt importów opisuje konstytucja; jest PROPOZYCJĄ do zatwierdzenia. Nie spłaszczać sześciu warstw do jednego łańcucha i nie dodawać importu infrastruktury do GUI dla wygody. Połączenie implementacji portów wykonuje wyłącznie zewnętrzny punkt składania aplikacji.

Nazwy modułów i identyfikatorów w kodzie: proponowane polskie nazwy bez znaków diakrytycznych, np. `KontoSprzedazowe`; terminy ekranowe i dokumentacyjne zachowują polską pisownię. Format identyfikatorów, wersja Pythona, biblioteka dostępu do bazy i narzędzie pakowania pozostają otwarte. Nie wprowadzać ich pod pretekstem formatowania.

Stosować małe, typowane funkcje i jawne zależności. Nie przechwytywać wszystkich wyjątków bez klasyfikacji. Nie używać globalnego połączenia SQLite, globalnego kontenera usług ani mutowalnego singletonu domeny. Nie przemycać reguł domenowych do `rdzen`, dekoratorów, sygnałów Qt lub mapperów ORM.

Zmiana powinna mieć jeden cel. Nie dołączać masowego formatowania niezwiązanych plików, wymiany stosu ani refaktoryzacji całego projektu. Przed edycją sprawdzić zastane zmiany użytkownika; zachować je. Nie usuwać plików i danych, by uzyskać zielone testy.

## 6. Standard testowania i kontroli

Docelowe komendy, dopiero po utworzeniu i zatwierdzeniu konfiguracji projektu:

```powershell
python -m pytest
python -m ruff check .
python -m ruff format --check .
python -m mypy src
lint-imports
python -m bandit -r src
python -m pip_audit
```

Nie przedstawiać tych komend jako działających w samym pakiecie dokumentacji. `lint-imports` jest poleceniem narzędzia import-linter; wymaga zainstalowanego środowiska i kontraktów. Audyt środowiska musi odpowiadać zależnościom dystrybuowanego produktu; osobno sprawdzać zależności developerskie.

| Rodzaj zmiany | Wymagane dowody |
|---|---|
| Reguła domenowa | Test pozytywny, graniczny, odrzucenie niepoprawnego stanu; bez Qt i bazy |
| Przypadek użycia | Sukces, błąd portu, brak częściowego zapisu, współbieżna edycja, jeśli dotyczy |
| Repozytorium SQLite | Test na rzeczywistej tymczasowej bazie: mapowanie, transakcja, ograniczenia |
| Migracja | Przejście ze wspieranych wersji, przerwanie, kopia i odtworzenie |
| Integracja | Mapowanie, timeout, duplikat, limit wywołań, odmowa autoryzacji, nieznany wynik |
| GUI | Klawiatura, fokus, stany błędu i pustki, DPI i oba motywy |
| Granice warstw | Kontrakty import-linter oraz przegląd dynamicznych zależności |

Nie dodawać testów wyłącznie powtarzających implementację. Nie wyłączać globalnie ostrzeżeń ruff/mypy/bandit. Wyjątek wymaga lokalnego uzasadnienia, właściciela, zakresu i terminu ponownej oceny. Brak sieci dla pip-audit oznacza brak wyniku, a nie brak podatności.

## 7. Bezpieczeństwo i praca z danymi

Nie umieszczać tokenów, haseł, danych klientów i pełnych odpowiedzi API w kodzie, fixture'ach, logach, zrzutach ekranu i zgłoszeniach. Sekrety mają być dostarczane przez port magazynu sekretów; jego implementacja Windows pozostaje propozycją. Baza zawiera co najwyżej odwołanie do sekretu, nigdy domyślny token.

Przed zmianą schematu opisać kompatybilność, procedurę kopii i przywracania oraz zachowanie przy awarii. Nie zakładać, że migracja „w dół” zawsze jest możliwa bez utraty danych. Nie uruchamiać testów na bazie użytkownika. Nie wykonywać rzeczywistych publikacji ofert i wysyłki wiadomości w testach automatycznych.

## 8. Zakończenie zadania

Raport podaje: co się zmieniło, jakie wymagania spełniono, jakie sprawdzenia wykonano, czego nie udało się sprawdzić i jakie decyzje nadal blokują dalszy krok. Dokumentacja musi być aktualizowana razem z zachowaniem aplikacji. Nie deklarować „gotowe do produkcji” wyłącznie na podstawie lintowania.

Przegląd zmiany ma potwierdzić: brak nowego konkurencyjnego modelu, właściwego właściciela danych, kierunek zależności, test niezmiennika, bezpieczeństwo danych oraz zgodność z centralnym Design Systemem. Szczegóły, które nie mają potwierdzenia, pozostają jawnie oznaczone.

## 9. Obowiązujący wzorzec wyglądu UI-REF-01

WYMAGANIE użytkownika: program ma wyglądać jak [załączony ekran](assets/UI_REFERENCE.png). Dokładny kontrakt: [UI_UX_DESIGN_SYSTEM, sekcje 15–23](UI_UX_DESIGN_SYSTEM.md). Wzorzec określa ciemną granatową powłokę, niebieskie akcje, boczne menu, cztery KPI, tabelę produktów, ostatnie operacje, statystyki, szybkie akcje, dwie karty integracji i pasek stanu. Nie zastępować go dowolnym dashboardem ani klasycznym formularzem Qt.

Przed implementacją ekranu sporządzić mapę widocznych opcji do przypadków użycia i modeli odczytu. Zachować nazwy i kolejność menu. Wszystkie wymienione na wzorcu funkcje są zakresem planu: Cenniki, media, opisy AI, tworzenie i szkice ofert, masowa edycja, zmiana/synchronizacja cen, układy/szablony, Allegro, PrestaShop, synchronizacja, kolejka, raporty i historia.

Zrzut nie ustala nowych niezmienników domeny. Nie przypisywać ceny i stanu bezpośrednio Produktowi tylko dlatego, że są kolumnami tabeli. Nie zmieniać listy 16 głównych bytów bez osobnej decyzji. Profile, logotypy i wersje z obrazu nie ustanawiają tożsamości firmy, wersji Pythona, limitu API ani modelu licencji.

Wymagania kontroli:

- Ctrl+K otwiera globalne wyszukiwanie; wszystkie ikony mają nazwy dostępności.
- „Masowa edycja” i „Edycja masowa” uruchamiają wspólny proces; kreatory układu i szablonów współdzielą edytor.
- Import, ceny, publikacja, AI i synchronizacja nie wykonują się w wątku GUI; ich przebieg jest widoczny jako Operacja.
- AI generuje propozycję, którą użytkownik zatwierdza. Nie publikuje samodzielnie i nie otrzymuje sekretów ani danych klientów przez przypadek.
- Dane ekranowe, nazwy producentów, `vexlabs`, domena sklepu, statystyki i daty nie trafiają do produkcyjnych seedów. Dane demonstracyjne wyłącznie w testach wizualnych.
- Widoczne „Połączono” i sukces Operacji wymagają rzeczywistego dowodu stanu. Testować częściowe połączenie i nieznany wynik.
- Odbiór ma obejmować porównanie z UI-REF-01 przy 1536×1024, a następnie testy responsywności, obu motywów, DPI i pustej bazy. Sama obecność przycisków nie jest ukończeniem funkcjonalności.

Zakres tego zadania pozostaje dokumentacyjny; aktualizacja wzorca nie jest poleceniem implementowania aplikacji.
