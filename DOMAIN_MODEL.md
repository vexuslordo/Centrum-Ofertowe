# CENTRUM OFERTOWE — model domenowy

Wersja 0.3 • 2026-11-02 • model koncepcyjny do zatwierdzenia, nie schemat wdrożonej bazy.

## 1. Jak interpretować model

WYMAGANIE: system obejmuje 16 głównych bytów wskazanych poniżej. Potwierdzone są ich nazwy i rozróżnienia z sekcji 2. Szczegółowe atrybuty, kardynalności, granice agregatów, operacje i niezmienniki opisane jako **P** są PROPOZYCJAMI wymagającymi zatwierdzenia. Nie stają się regułami biznesowymi przez samo umieszczenie w tym dokumencie.

**W** oznacza wymaganie źródłowe, **P** propozycję, **N** nieustaloną decyzję. Właściciel danych w kartach oznacza proponowany kontekst zarządzający znaczeniem danych; nie konkretną osobę i nie platformę informatyczną. Lokalna baza jest magazynem, a źródło prawdy trzeba ustalić na poziomie pól.

Pojęcia dodatkowe, takie jak pozycja zamówienia, adres, kwota, referencja sekretu i identyfikator zewnętrzny, to kandydaci na obiekty wartości lub składniki agregatów. Ich opis nie rozszerza zatwierdzonej listy głównych bytów. Nie rozstrzygnięto jeszcze, czy każdy z 16 bytów ma być osobnym korzeniem agregatu.

## 2. Język wspólny i potwierdzone rozróżnienia

| Pojęcie | Znaczenie potwierdzone lub ograniczenie |
|---|---|
| Produkt | Niezależny od Allegro, PrestaShop, Subiekt GT i innych integracji |
| Oferta | Prezentacja Produktu; nie drugi egzemplarz Produktu |
| Producent / Dostawca / PodmiotGPSR | Trzy niezależne pojęcia; nie scalać ich automatycznie |
| KanałSprzedaży / KontoSprzedażowe | Różne byty; platforma nie jest kontem |
| Przewoźnik | Nie jest KanałemSprzedaży; jego model nie został ustalony |
| Środek komunikacji | Obowiązująca nazwa w obszarze komunikacji |
| Zdarzenie / Operacja | Dwa wymagane byty; szczegółowe znaczenie rozróżnienia proponowane poniżej |

Nazwy API dostawcy są tłumaczone na język modelu. `item`, `listing`, `auction` i `product` nie mogą być utożsamiane wyłącznie na podstawie brzmienia. Mapowanie musi zawierać definicję i przykład.

## 3. Proponowany podział kontekstów

| Kontekst P | Byty | Odpowiedzialność |
|---|---|---|
| Katalog | Produkt, Producent, PodmiotGPSR, Dostawca | Tożsamość i dane katalogowe |
| Sprzedaż | KanałSprzedaży, KontoSprzedażowe, Oferta | Prezentacja i publikacja |
| Obsługa sprzedaży | Klient, Zamówienie, Dokument, Przesyłka | Realizacja i dokumentowanie transakcji |
| Komunikacja | Wątek, Wiadomość | Historia kontaktu i wysyłka |
| Automatyzacja i wykonanie | Reguła, Zdarzenie, Operacja | Reakcje na fakty i monitorowanie pracy |

To granice semantyczne, a nie żądanie utworzenia osobnych procesów lub baz. Kierunki współpracy przechodzą przez przypadki użycia i identyfikatory. Pełne granice wymagają warsztatu scenariuszy i zatwierdzenia.

## 4. Wspólne zasady techniczne — P

Każdy trwały byt posiada stabilne wewnętrzne ID. Proponowane metadane to czas utworzenia, czas zmiany i wersja do kontroli współbieżności. Format ID, nazwy i obowiązkowość metadanych są N. Zewnętrzny ID jest osobną referencją ze wskazaniem systemu, konta i typu obiektu. Nie wolno zakładać, że EAN, e-mail lub numer przesyłki zawsze identyfikują byt globalnie.

Daty zdarzeń przechowywać jako jednoznaczne chwile; strefę użytkownika stosować przy prezentacji. Data biznesowa bez godziny jest innym typem. Pieniądz obejmuje wartość i walutę; nie wolno sumować różnych walut bez jawnej reguły przeliczenia. Jednostki ilości wymagają uzgodnienia. Brak wartości nie oznacza zera ani pustego ciągu.

Usunięcie, archiwizacja i anonimizacja nie są zamienne. Dopóki nie ustalono retencji i referencji, nie implementować kaskadowego usuwania danych. Przed zapisem sprawdzać wersję odczytaną przez użytkownika; konflikt zwracać do rozstrzygnięcia.

## 5. Karty 16 bytów

### 5.1. Produkt

**W — odpowiedzialność:** wspólna tożsamość produktu i dane niezależne od platform. Oferta korzysta z Produktu jako jego prezentacja.

**P — właściwości:** ID, nazwa robocza, opis katalogowy, wewnętrzny symbol, opcjonalne identyfikatory towarowe, zestaw parametrów z jednostkami, referencje multimediów, stan kompletności i wersja. EAN nie jest z góry obowiązkowy ani unikalny. Cena sprzedaży, stan magazynowy i podatki nie otrzymują w tym pakiecie właściciela.

**P — relacje:** odwołania do Producenta, PodmiotuGPSR i Dostawców; dokładna liczność oraz historia obowiązywania są N. Relacja z Ofertą jest potwierdzona znaczeniowo, lecz liczność i obsługa wariantów/zestawów są N.

**P — agregat i niezmienniki:** kandydat na korzeń agregatu katalogowego; zmiana nazwy nie zmienia ID. Dane platformowe nie mogą stać się obowiązkowym polem każdej instancji. Kompletność do publikacji ocenia scenariusz Oferty, a nie globalny wymóg Allegro narzucony wszystkim Produktom.

**P — właściciel i źródło prawdy:** kontekst Katalog; lokalne, świadomie zatwierdzone dane katalogowe. Import z ERP lub platformy jest propozycją aktualizacji z pochodzeniem. N: czy ERP będzie właścicielem wybranych pól.

**P — operacje:** utworzenie szkicu, zmiana danych, przypisanie ról, ocena kompletności, archiwizacja. Scalanie duplikatów wymaga osobnego scenariusza i reguł przenoszenia referencji.

**Weryfikacja P:** zmiana danych Oferty nie tworzy nowego Produktu; payload platformy nie pojawia się w encji; dwie wersje edycji dają jawny konflikt.

### 5.2. Producent

**W — odpowiedzialność:** niezależne pojęcie producenta; nie utożsamiać z dostawcą ani PodmiotemGPSR.

**P — właściwości:** ID, nazwa, identyfikatory rejestrowe z rodzajem i krajem, dane adresowe i kontaktowe, status aktywności, pochodzenie danych i wersja. Obowiązkowość poszczególnych pól jest N.

**P — relacje:** Produkt może wskazywać Producenta; liczność, wymaganie przypisania i obsługa marek są N. Ta sama organizacja w świecie rzeczywistym może pełnić różne role, lecz nie oznacza to wspólnego ID ani automatycznego scalenia rekordów.

**P — agregat i niezmienniki:** samodzielny słownik biznesowy; zmiana nazwy nie zmienia powiązań. Nie usuwać rekordu z aktywnymi odwołaniami bez rozstrzygnięcia polityki. Nazwa sama w sobie nie jest pewnym kluczem deduplikacji.

**P — właściciel i źródło prawdy:** Katalog, lokalnie zatwierdzony rejestr z informacją o źródle. Zewnętrzny katalog producentów nie nadpisuje automatycznie danych kontaktowych.

**P — operacje:** utworzenie, korekta, dezaktywacja, przegląd potencjalnego duplikatu. N: kto potwierdza wiarygodność danych.

**Weryfikacja P:** dezaktywacja nie usuwa Produktów; import dwóch podobnych nazw nie powoduje automatycznego scalenia.

### 5.3. PodmiotGPSR

**W — odpowiedzialność:** odrębny byt związany z danymi GPSR, niezależny od Producenta i Dostawcy.

**P — właściwości:** ID, nazwa, rodzaj roli, dane adresowe i kontaktowe, zakres/okres obowiązywania, źródło i data weryfikacji, wersja. Definicje ról i wymaganych danych są N i wymagają osobnej analizy właściwych regulacji przed wdrożeniem.

**P — relacje:** przypisanie do Produktu i wykorzystanie danych w prezentacji Oferty. Liczność, wersjonowanie i reguła wyboru podmiotu dla rynku są N. Dokument nie przesądza, że zawsze wystarczy jeden podmiot.

**P — agregat i niezmienniki:** niezależny rejestr; techniczna obecność rekordu nie dowodzi zgodności prawnej Produktu. Dane obowiązujące historycznie nie powinny znikać po bieżącej korekcie; sposób zachowania historii jest N.

**P — właściciel i źródło prawdy:** Katalog; zatwierdzone dane od odpowiedzialnej osoby, z dowodem pochodzenia. Odpowiedzialność organizacyjna za zgodność jest N.

**P — operacje:** rejestracja, korekta, oznaczenie do przeglądu, przypisanie do Produktu, wycofanie przypisania. Brak automatycznego oświadczenia „produkt zgodny” na podstawie samej walidacji pól.

**Weryfikacja P:** Producent nie jest automatycznie PodmiotemGPSR; brak przypisania daje wynik wymagający obsługi według zatwierdzonego scenariusza, bez wymyślonych danych domyślnych.

### 5.4. Dostawca

**W — odpowiedzialność:** odrębna rola dostawcy; nie mylić z producentem ani platformą sprzedaży.

**P — właściwości:** ID, nazwa, dane identyfikacyjne i kontaktowe, aktywność, zewnętrzne referencje i wersja. Warunki handlowe, waluty rozliczeń, terminy dostaw i cenniki są N; nie dodawać ich jako domyślnych reguł.

**P — relacje:** powiązanie z Produktami może zawierać symbol dostawcy i pochodzenie informacji; liczność oraz to, czy powiązanie ma własny cykl życia, wymagają zatwierdzenia. Dane dostępności z hurtowni nie są automatycznie lokalnym stanem magazynowym.

**P — agregat i niezmienniki:** samodzielny rejestr biznesowy; dwa rekordy nie są scalane tylko dlatego, że mają ten sam e-mail. Zmiana aktywności nie kasuje historii.

**P — właściciel i źródło prawdy:** Katalog; zatwierdzona kartoteka lokalna. Feed dostawcy pozostaje źródłem określonych obserwacji z datą, nie uniwersalnym właścicielem Produktu.

**P — operacje:** dodanie, edycja, dezaktywacja, przypisanie Produktu, przegląd importu. N: model zakupów i priorytety dostawców.

**Weryfikacja P:** import cennika nie publikuje automatycznie Oferty; pomyłka w mapowaniu symbolu trafia do przeglądu zamiast tworzyć drugi Produkt.

### 5.5. KanałSprzedaży

**W — odpowiedzialność:** byt odrębny od KontaSprzedażowego; nie reprezentuje przewoźnika ani Środka komunikacji.

**P — właściwości:** ID, nazwa prezentacyjna, typ integracji, referencja implementacji, stan dostępności technicznej i wersja konfiguracji. Lista obsługiwanych możliwości powinna pochodzić z kontraktu konektora, nie z warunków w domenie.

**P — relacje:** KontoSprzedażowe wskazuje KanałSprzedaży; propozycja liczności 1:N wymaga akceptacji. N: czy jeden rekord oznacza platformę, rynek czy konfigurację instancji sklepu.

**P — agregat i niezmienniki:** kandydat na samodzielny korzeń konfiguracji biznesowej. Nie przechowuje tokenów i nie reprezentuje jednego loginu. Brak konektora nie może usunąć historycznych Ofert.

**P — właściciel i źródło prawdy:** kontekst Sprzedaż; lokalna konfiguracja plus techniczne możliwości zainstalowanego konektora. Nie mieszać statusu platformy z aktywnością konkretnego konta.

**P — operacje:** zarejestrowanie kanału, konfiguracja, oznaczenie niedostępności, wyłączenie nowych publikacji. N: zasady usuwania i zmiany typu.

**Weryfikacja P:** dwa konta jednej platformy nie tworzą dwóch konkurencyjnych modeli kanału; czysta instalacja nie zawiera automatycznie wpisanego konta ani rekordu kanału.

### 5.6. KontoSprzedażowe

**W — odpowiedzialność:** konto prowadzenia sprzedaży, różne od KanałuSprzedaży.

**P — właściwości:** ID, nazwa lokalna, referencja kanału, zewnętrzny identyfikator konta, środowisko testowe/produkcyjne, referencja sekretu, stan połączenia i wersja. Stan uwierzytelnienia jest obserwacją techniczną; nie powinien oznaczać usunięcia konta.

**P — relacje:** jedno KontoSprzedażowe należy do jednego KanałuSprzedaży; Oferty i identyfikatory zewnętrzne są zakresowane kontem. Liczności są propozycją. N: profile organizacji, uprawnienia wielu użytkowników i obsługa wielu rynków.

**P — agregat i niezmienniki:** kandydat na korzeń; nie przechowuje jawnego tokenu w domenie. Zmiana danych logowania nie może przepiąć istniejących Ofert na innego sprzedawcę bez kontrolowanego procesu.

**P — właściciel i źródło prawdy:** Sprzedaż dla konfiguracji; dostawca tożsamości platformy dla faktycznej tożsamości i uprawnień; magazyn sekretów dla poświadczeń.

**P — operacje:** rejestracja, powiązanie autoryzacji, sprawdzenie połączenia, odłączenie, wyłączenie synchronizacji. Przechowywanie i odnawianie tokenów wykonuje adapter.

**Weryfikacja P:** odłączenie konta nie kasuje Zamówień; token konta A nie jest używany w Operacji konta B.

### 5.7. Oferta

**W — odpowiedzialność:** prezentacja Produktu, nie jego drugi egzemplarz.

**P — właściwości:** ID, referencja Produktu i konta, lokalny tytuł/prezentacja, dane specyficzne publikacji, zewnętrzna referencja, lokalny zamiar stanu, zaobserwowany stan publikacji, czas synchronizacji i wersja. Cena, stan dostępności, kategorie i parametry wymagają macierzy własności pól.

**P — relacje:** Oferta wskazuje Produkt i KontoSprzedażowe, przez konto uzyskuje kanał. N: warianty, zestawy, jedna oferta z wieloma produktami, oferty bez lokalnego Produktu podczas uzgadniania importu. Nie rozstrzygać tego na podstawie jednego API.

**P — agregat i niezmienniki:** kandydat na korzeń. Zmiana tytułu nie tworzy nowego Produktu. Brak gotowości danych blokuje żądanie publikacji według zatwierdzonego kontraktu platformy. Potwierdzenie wysłania żądania nie oznacza potwierdzenia publikacji.

**P — właściciel i źródło prawdy:** Sprzedaż dla lokalnej prezentacji i zamiaru; platforma dla obserwowanego stanu zewnętrznego. Produkt pozostaje właścicielem wspólnych danych katalogowych. N: które nadpisania są dozwolone.

**P — operacje:** szkic, edycja prezentacji, walidacja gotowości, zlecenie publikacji/aktualizacji/zakończenia, uzgodnienie stanu. Proponowane stany lokalne: szkic, gotowa, w realizacji, konflikt; dokładny automat jest N.

**Weryfikacja P:** timeout publikacji prowadzi do uzgodnienia stanu; ponowne naciśnięcie nie powoduje automatycznie drugiej publikacji.

### 5.8. Klient

**W — odpowiedzialność:** wspólny byt Klienta; integracje nie utrzymują konkurencyjnych modeli.

**P — właściwości:** ID, typ osoby/podmiotu, nazwa prezentacyjna, dane kontaktowe, referencje zewnętrzne, preferencje i wersja. Zakres danych, podstawy przetwarzania i retencja są N. Zgody nie mogą wynikać automatycznie z faktu zakupu.

**P — relacje:** Zamówienia i Wątki mogą wskazywać Klienta; anonimowe zakupy i wielu uczestników Wątku są N. Adres zamówienia powinien być historycznym zapisem transakcji, a nie dynamicznym odczytem aktualnej kartoteki — propozycja wymagająca akceptacji.

**P — agregat i niezmienniki:** kandydat na korzeń. E-mail nie jest globalnym kluczem Klienta. Automatyczne scalenie tożsamości między kontami jest zabronione w proponowanym modelu bez jawnej reguły i dowodów.

**P — właściciel i źródło prawdy:** Obsługa sprzedaży dla lokalnej kartoteki; platforma dla identyfikacji kupującego w swoim zakresie; użytkownik dla zatwierdzonych korekt. Własność pól kontaktowych wymaga ustalenia.

**P — operacje:** rejestracja/import, korekta, powiązanie referencji, przegląd duplikatu, kontrolowana anonimizacja. N: uprawnienia i warunki usuwania.

**Weryfikacja P:** zmiana adresu Klienta nie zmienia historycznego adresu wysłanej paczki; podobny e-mail nie scala automatycznie osób.

### 5.9. Zamówienie

**W — odpowiedzialność:** wspólny model Zamówienia, niezależny od struktur poszczególnych integracji.

**P — właściwości:** ID, referencja źródła/konta, numer zewnętrzny, chwila złożenia, pozycje z opisem historycznym, ilości i kwoty z walutą, dane nabywcy/odbiorcy utrwalone dla transakcji, obserwowany status zewnętrzny, lokalny stan obsługi i wersja.

**P — relacje:** Klient, Oferty/Produkty w pozycjach, Dokumenty i Przesyłki. Liczności, pozycja bez dopasowania Produktu, płatności, zwroty i korekty są N. Nie dopisywać bytu Płatność jako zatwierdzonego siedemnastego bytu.

**P — agregat i niezmienniki:** proponowany korzeń z pozycjami jako składnikami. Historyczny opis zakupu nie zmienia się wraz z katalogiem. Wartości nie sumują się między walutami. Reguły rabatów, podatków, dokładności i zaokrągleń wymagają decyzji, zanim powstanie algorytm sumowania.

**P — właściciel i źródło prawdy:** Obsługa sprzedaży dla lokalnej realizacji; system źródłowy dla otrzymanego zamówienia i jego zmian; pola konfliktowe mają kontrakt synchronizacji.

**P — operacje:** przyjęcie/import, dopasowanie pozycji, rozpoczęcie obsługi, zlecenie dokumentu lub wysyłki, uzgodnienie zmiany/anulowania. N: dozwolone przejścia i skutki anulowania po wysyłce.

**Weryfikacja P:** powtórny import tego samego zdarzenia nie dubluje zamówienia; aktualizacja Produktu nie zmienia ceny historycznej pozycji.

### 5.10. Dokument

**W — odpowiedzialność:** odrębny byt dokumentu w procesach sprzedaży. Rodzaje dokumentów nie zostały określone.

**P — właściwości:** ID, rodzaj, numer/referencja wystawcy, daty biznesowe, wersja treści, lokalizacja pliku lub referencja systemu, suma kontrolna, status i pochodzenie. Nie zakładać, że każdy Dokument jest fakturą albo plikiem PDF.

**P — relacje:** powiązanie z Zamówieniem i ewentualnie Klientem; liczności, dokument zbiorczy i korekty są N. Plik binarny może być zasobem infrastruktury, a nie polem encji ładowanym do pamięci przy każdej liście.

**P — agregat i niezmienniki:** kandydat na korzeń. Wystawionej treści nie zastępować edycją bez śladu; sposób korekty zależy od rodzaju dokumentu. Numeracja, wymogi podatkowe i uprawnienia nie są ustalone w tym pakiecie.

**P — właściciel i źródło prawdy:** Obsługa sprzedaży dla powiązań; ustalony system wystawiający dla treści i numeracji. Wybór wystawcy lokalnego lub ERP jest N.

**P — operacje:** rejestracja, pobranie, powiązanie, weryfikacja dostępności, zlecenie wystawienia/korekty po uzgodnieniu reguł. N: przechowywanie i retencja.

**Weryfikacja P:** ponowne pobranie nie nadpisuje innego pliku przy zbieżnej nazwie; brak pliku nie powoduje utraty metadanych i historii.

### 5.11. Przesyłka

**W — odpowiedzialność:** byt Przesyłki; przewoźnik nie jest KanałemSprzedaży.

**P — właściwości:** ID, referencja usługi przewozowej, numer śledzenia w zakresie przewoźnika, dane nadania i odbioru, parametry paczek z jednostkami, referencja etykiety, status zaobserwowany i czas obserwacji, wersja.

**P — relacje:** Zamówienie lub jego zakres realizacji; liczności, wiele paczek, konsolidacja zamówień i przesyłki zwrotne są N. Przewoźnik/usługa to na razie referencja lub obiekt wartości do rozstrzygnięcia.

**P — agregat i niezmienniki:** kandydat na korzeń. Wysłanie żądania etykiety nie dowodzi fizycznego nadania. Numer śledzenia bez wskazania przewoźnika nie jest pewnym globalnym identyfikatorem. Dane historycznie użyte do nadania nie zmieniają się automatycznie z kartoteką Klienta.

**P — właściciel i źródło prawdy:** Obsługa sprzedaży dla zamiaru wysyłki i powiązań; przewoźnik dla obserwowanego śledzenia; system generujący dla etykiety.

**P — operacje:** przygotowanie, walidacja, zlecenie utworzenia, pobranie etykiety, odświeżenie śledzenia, zlecenie anulowania. N: granica, po której anulowanie nie jest możliwe.

**Weryfikacja P:** timeout generowania etykiety wymusza sprawdzenie wyniku przed ponowieniem; anulowanie lokalnego oczekiwania nie oznacza anulowania przewozu.

### 5.12. Wątek

**W — odpowiedzialność:** byt grupujący komunikację; stosować pojęcie Środek komunikacji.

**P — właściwości:** ID, temat, referencja źródła, Środek komunikacji, uczestnicy jako referencje, stan obsługi, czas ostatniej aktywności i wersja. Przeczytanie i biznesowe rozwiązanie sprawy są różnymi stanami.

**P — relacje:** Wiadomości, opcjonalne powiązanie z Klientem i Zamówieniem. Liczności, wątki wieloosobowe, łączenie wątków oraz wielu Środków komunikacji w jednym Wątku są N.

**P — agregat i niezmienniki:** propozycja: korzeń metadanych z osobno przechowywanymi Wiadomościami, by nie ładować całej historii do jednej transakcji. Alternatywa: Wiadomość jako element agregatu; trzeba rozstrzygnąć przed implementacją. Nie grupować automatycznie różnych rozmów tylko po tytule.

**P — właściciel i źródło prawdy:** Komunikacja dla lokalnego stanu obsługi; system źródłowy dla identyfikatora rozmowy i zewnętrznej historii.

**P — operacje:** utworzenie/import, powiązanie, zmiana lokalnego stanu, oznaczenie odczytu. N: przypisywanie operatora i eskalacja.

**Weryfikacja P:** import starszej Wiadomości nie usuwa nowszego lokalnego stanu obsługi; zmiana tematu nie tworzy automatycznie nowego Wątku.

### 5.13. Wiadomość

**W — odpowiedzialność:** pojedyncza Wiadomość w obszarze komunikacji, odrębna od Wątku.

**P — właściwości:** ID, referencja Wątku, kierunek, nadawca i adresaci w zakresie źródła, treść z formatem, czas powstania/wysłania/odbioru, referencje załączników, zewnętrzny ID, stan dostarczenia i wersja.

**P — relacje:** Wątek; załączniki jako zasoby, nie automatycznie Dokumenty. N: czy wiadomości systemowe bez Wątku są dopuszczalne, czy edycje zewnętrzne tworzą wersje.

**P — agregat i niezmienniki:** granica zależy od decyzji dla Wątku. Wiadomości przychodzącej nie edytuje się jak lokalnego szkicu. „Przyjęta przez API” nie oznacza „dostarczona”. HTML i załączniki wymagają bezpiecznej prezentacji, bez wykonywania treści.

**P — właściciel i źródło prawdy:** Komunikacja dla szkicu i zamiaru wysyłki; system źródłowy dla otrzymanej treści i potwierdzeń transportu; lokalny zapis przechowuje pochodzenie.

**P — operacje:** utworzenie szkicu, korekta przed wysyłką, zlecenie wysłania, import, uzgodnienie dostarczenia. N: retencja, ponowienie i polityka usuwania.

**Weryfikacja P:** timeout wysyłki nie uruchamia ślepego powtórzenia; dwa odbiory tego samego zewnętrznego ID nie tworzą dwóch Wiadomości.

### 5.14. Reguła

**W — odpowiedzialność:** jeden z wymaganych bytów; szczegółowa semantyka automatyzacji jest proponowana.

**P — właściwości:** ID, nazwa, wersja definicji, warunki, akcje, zakres kont/obiektów, stan aktywacji, priorytet i ograniczenia wykonania. Język wyrażeń, harmonogram i kolejność konfliktujących reguł są N.

**P — relacje:** reaguje na Zdarzenia i zleca przypadki użycia reprezentowane Operacjami. Nie modyfikuje encji bezpośrednio przez SQL. Referencja użytej wersji reguły jest zapisywana przy wykonaniu.

**P — agregat i niezmienniki:** kandydat na korzeń konfiguracji automatyzacji. Definicja nie jest dowolnym kodem Python, `eval` ani skryptem powłoki. Przed aktywacją przechodzi walidację, symulację i kontrolę zakresu skutków. Ochrona przed pętlą wymaga identyfikacji pochodzenia i limitu reakcji; konkretne limity są N.

**P — właściciel i źródło prawdy:** Automatyzacja i wykonanie; lokalna zatwierdzona definicja wersjonowana. Zewnętrzna konfiguracja nie aktywuje samoczynnie lokalnej Reguły.

**P — operacje:** szkic, walidacja, symulacja bez efektów, aktywacja, pauza, utworzenie nowej wersji, przegląd historii.

**Weryfikacja P:** duplikat Zdarzenia nie powiela skutku; Reguła generująca następne Zdarzenie nie tworzy nieograniczonej pętli.

### 5.15. Zdarzenie

**W — odpowiedzialność:** wymagany byt. **P:** opis faktu, który już zaszedł, w odróżnieniu od polecenia wykonania pracy.

**P — właściwości:** ID, typ, wersja schematu, czas zajścia i czas rejestracji, źródło, referencja obiektu, identyfikatory korelacji/przyczyny, klucz deduplikacji i minimalny payload. Struktura trwałych zdarzeń jest N.

**P — relacje:** może odnosić się do dowolnego zatwierdzonego bytu; Reguła może na nie reagować; Operacja może je wytworzyć. To nie oznacza dowolnych, niewalidowanych referencji w jednej tabeli bez kontraktu.

**P — agregat i niezmienniki:** zapis faktu jest niemutowalny; sprostowanie jest kolejnym faktem. Zdarzenie domenowe, zewnętrzne powiadomienie i log diagnostyczny mają różne kontrakty. Nie każdy sygnał Qt i wpis logu staje się Zdarzeniem biznesowym. Kolejność i dostarczenie dokładnie raz nie są gwarantowane.

**P — właściciel i źródło prawdy:** emitujący kontekst dla znaczenia faktu; Automatyzacja i wykonanie dla jego rejestracji/dostarczenia. Zewnętrzne powiadomienie wymaga uwierzytelnienia i uzgodnienia, jeśli samo nie dowodzi aktualnego stanu.

**P — operacje:** rejestracja, odczyt, oznaczenie przetworzenia w osobnym stanie technicznym, ponowne przetworzenie z deduplikacją. N: retencja i migracja payloadów.

**Weryfikacja P:** drugi odbiór faktu nie powoduje drugiego skutku; nieznana wersja schematu nie trafia bez walidacji do Reguły.

### 5.16. Operacja

**W — odpowiedzialność:** wymagany byt. **P:** trwałe śledzenie zleconej pracy i jej wyniku, szczególnie długotrwałej i zewnętrznej.

**P — właściwości:** ID, typ, zakres obiektu i konta, klucz idempotencji, stan, czasy, liczba prób, postęp, bezpieczny kod błędu, korelacja, referencja wyniku i wersja. Dane wejściowe nie mogą zawierać jawnych sekretów.

**P — relacje:** inicjator jako użytkownik lub wersja Reguły; obiekt docelowy przez typowaną referencję; Zdarzenia skutków. N: tożsamość użytkownika i model uprawnień.

**P — agregat i niezmienniki:** kandydat na korzeń wykonania. Proponowane stany: oczekuje → wykonywana → zakończona / błąd / wynik nieznany; dodatkowo żądanie anulowania i anulowana, tylko gdy potwierdzono brak dalszego wykonania. Pełna macierz przejść wymaga zatwierdzenia. Ukończonej Operacji nie uruchamia się od nowa przez wyzerowanie pól; ponowienie ma ślad próby lub odrębnej operacji według decyzji.

**P — właściciel i źródło prawdy:** Automatyzacja i wykonanie dla lokalnego przebiegu; zewnętrzny system dla faktycznego skutku. Procent postępu nie dowodzi sukcesu biznesowego.

**P — operacje:** zlecenie, rozpoczęcie, aktualizacja postępu, zapis wyniku, żądanie anulowania, kontrolowane ponowienie, uzgodnienie nieznanego wyniku po restarcie.

**Weryfikacja P:** awaria po sukcesie API, ale przed lokalnym zapisem, kończy się uzgodnieniem; dwaj workerzy nie wykonują jednocześnie tej samej pracy bez kontroli.

## 6. Proponowane relacje — lista do zatwierdzenia

| Referencja | Potwierdzone | Nadal nieustalone |
|---|---|---|
| Oferta → Produkt | Prezentacja Produktu | Liczność, warianty, zestawy, import niedopasowanych ofert |
| Produkt → Producent/PodmiotGPSR/Dostawca | Rozdzielność pojęć | Obowiązkowość, liczności, okresy obowiązywania |
| KontoSprzedażowe → KanałSprzedaży | Rozdzielność pojęć | 1:N, rynki i instancje platformy |
| Oferta → KontoSprzedażowe | Brak potwierdzonej liczności | Zakres publikacji i zmiana właściciela |
| Zamówienie → Klient/pozycje | Brak szczegółowego zatwierdzenia | Anonimowość, dopasowanie, snapshoty, sumy |
| Dokument/Przesyłka → Zamówienie | Brak szczegółowego zatwierdzenia | Zbiorczość i częściowa realizacja |
| Wiadomość → Wątek | Pojęcia wymagane oddzielnie | Granica agregatu, liczność i historia |
| Reguła → Zdarzenie → Operacja | Brak szczegółowego zatwierdzenia | Kolejność, deduplikacja, pętle i retencja |

Strzałka oznacza referencję, nie zależność importów ani kaskadę usuwania. Żadnej niepotwierdzonej liczności nie utrwalać w kluczach obcych przed przeglądem.

## 7. Macierz źródeł prawdy — wymagane uzupełnienie przed integracją

Dla każdego pola ustalić: nazwę i semantykę, kontekst właściciela, system źródłowy, dopuszczalną edycję lokalną, kierunek synchronizacji, rozstrzygnięcie konfliktu i historię. Przykład P: `Oferta.stan_obserwowany` należy do obserwacji platformy, a `Oferta.zamiar_publikacji` do aplikacji; ich rozbieżność jest stanem procesu, nie błędem naprawianym przez wzajemne nadpisanie.

Minimalne scenariusze akceptacyjne P: edycja Produktu przy aktywnych Ofertach; dwa konta jednej platformy; ten sam klient z dwóch źródeł; import Zamówienia bez dopasowanego Produktu; częściowa wysyłka; timeout publikacji; powtórny odbiór Wiadomości; restart Operacji; korekta danych PodmiotuGPSR z zachowaniem historii. Każdy scenariusz musi dostać wynik oczekiwany zatwierdzony przez właściciela, zanim stanie się testem normatywnym.

## 8. Granice niewchodzące automatycznie do zakresu

Nie ustalono modelu magazynu, rezerwacji, płatności, fakturowania, podatków, zwrotów, reklamacji, wielofirmowości ani wielostanowiskowości. Nie wynika z tego zakaz ich przyszłego rozwoju; wymagają osobnej analizy i zatwierdzenia. Nie wolno ukrywać nowych bytów i reguł tych obszarów w polu JSON istniejącej encji.

Model jest nowym, samodzielnym projektem; nie zależy od wcześniejszych decyzji. Nowy rejestr pytań i ścieżka decyzji: [ADR_INDEX](ADR_INDEX.md).

## 9. Rozszerzenia zakresu wynikające z UI-REF-01

WYMAGANIE: aplikacja ma udostępniać opcje z [referencji użytkownika](assets/UI_REFERENCE.png). Obraz określa funkcje i ich miejsce w interfejsie, ale nie definiuje kompletnego modelu danych. Poniższy podział doprecyzowuje wpływ tych opcji. Pierwotna lista 16 głównych bytów pozostaje zachowana; nowe kandydatury na agregaty są jawnie oznaczone i wymagają decyzji przed schematem.

| Funkcja wymagana | Powiązanie z 16 bytami | Propozycja modelu / granica niepewności |
|---|---|---|
| Cenniki i import z cennika | Produkt, Dostawca, Oferta, Operacja | Cennik i pozycja cennika to kandydaci na dodatkowe pojęcia; nie zostały zatwierdzone jako główne byty |
| Zdjęcia i media | Produkt, Oferta, Operacja | Referencja zasobu, metadane, kolejność i rola; osobny byt MediaZasób pozostaje propozycją |
| Kreator opisów (AI) | Produkt/Oferta i Operacja | Propozycja treści → akceptacja → zmiana wskazanego opisu; AI nie jest właścicielem danych |
| Masowa edycja | Wiele Produktów lub Ofert i Operacja | Zbiorcza orkiestracja tych samych metod domeny, nie alternatywny model |
| Zmiana/synchronizacja cen | Oferta, Produkt jako referencja, KontoSprzedażowe, Operacja | Cena z walutą i źródłem; właściciel i kierunek synchronizacji do ustalenia |
| Szablony i układy ofert | Oferta, Produkt przez powiązania danych, Operacja | Wersjonowana definicja prezentacji; kandydat SzablonUkładu, nie zatwierdzony siedemnasty byt |
| Sprawdź błędy | Wszystkie obiekty podlegające walidacji, Operacja | Problem walidacyjny jako wynik odczytu: kod, obiekt, pole, waga; trwały model problemów N |
| Kolejka / historia | Operacja, Zdarzenie, Reguła jeśli inicjuje zadanie | Kolejka to uporządkowany widok pracy, nie nowy model sprzedaży |
| Statystyki / raporty | Zamówienie, Oferta, Operacja | Projekcje danych z zatwierdzoną metryką; brak własnych mutowalnych kartotek |
| Globalne szukanie | Produkt, Oferta | Wyszukiwanie SKU/EAN/nazwy z nawigacją do obiektu; nie nowe źródło prawdy |

### 9.1. Ceny i cenniki — propozycja do ADR-013

Wymagane funkcje: lista cenników, import, podgląd zmian, zmiana cen i synchronizacja. P: pozycja źródłowa zawiera referencję Produktu lub niedopasowany symbol, kwotę z walutą, źródło, czas obowiązywania/obserwacji i pochodzenie importu. Czy Cennik ma własny ID, wersję i transakcyjną granicę, jest do rozstrzygnięcia.

Nieustalone: cena zakupu czy sprzedaży; netto/brutto; stawki podatku; promocje; źródło nadrzędne; mnożniki i zaokrąglenia; wiele walut; harmonogram; częściowe obowiązywanie. Nie tworzyć domyślnej marży, kursu i cennika z nazwą ze screenu. Ceny widoczne w tabeli są projekcją jawnie wybranego kontekstu. Ekran ma wskazywać źródło, jeśli nie jest oczywiste.

P: synchronizacja nie znaczy zawsze „wyślij lokalne ceny”. Wybór kierunku i konta, porównanie wartości, rozwiązanie konfliktu, potwierdzenie i raport są oddzielnymi krokami. Testować identyczną wartość, brak waluty, duplikat pozycji, brak dopasowania i cenę zmienioną po podglądzie.

### 9.2. Dostępność / kolumna Stan

WYMAGANIE UI: kolumna Stan oraz możliwość masowej zmiany dostępności. Z obrazu nie wynika magazyn ani rezerwacje. P: wyświetlać obserwację ilości z jednostką, źródłem i czasem. Źródłem może być zatwierdzony system zewnętrzny lub przyszły obszar lokalny; nie wybieramy tego automatycznie. Nie synchronizować stanu dostawcy do każdej Oferty bez reguły. Próg czerwonego ostrzeżenia i znaczenie ilości są N. Brak danych nie jest zerem.

### 9.3. Media — propozycja do ADR-014

WYMAGANIE: galeria, miniatury tabeli i odświeżanie zdjęć. P: metadane zasobu obejmują identyfikację, pochodzenie, MIME po weryfikacji, rozmiar, sumę kontrolną, lokalizację, rolę i kolejność przypisania. Prawa użycia i autorstwo są danymi wymagającymi ustalenia. Materiał Produktu może być wykorzystany w Ofercie, lecz zmiana kolejności na jednej Ofercie nie musi zmieniać katalogu — wymaga decyzji.

Nie utożsamiać każdej fotografii z Dokumentem. Nie usuwać wspólnego pliku po odłączeniu jednej Oferty. Potrzebna polityka referencji i nieużywanych zasobów. Cache miniatur jest odtwarzalny. Testować brak pliku, powtórne dodanie, uszkodzone dane i próbę wyjścia poza katalog zasobów.

### 9.4. Opisy AI — propozycja do ADR-015

WYMAGANIE: Kreator opisów (AI) i kafel generowania opisów. P: wejściem są wybrane, potwierdzone fakty Produktu; wyjściem szkic tekstu, nie automatyczna korekta encji. Użytkownik porównuje i przyjmuje tekst jako opis katalogowy albo nadpisanie prezentacji Oferty. Zakres musi być wskazany, aby nie utracić rozdzielenia Produkt/Oferta.

Wynik może zawierać błędy. Nie tworzyć domyślnie zapewnień o zgodności GPSR, właściwościach technicznych i kompatybilności bez źródła. Przechowywać ślad akceptacji i docelowej wersji według zatwierdzonej retencji. Dane klientów, tokeny i treści komunikacji nie należą do domyślnego wejścia generatora.

### 9.5. Układy i szablony — propozycja do ADR-014

WYMAGANIE: kreator, lista szablonów, kopiowanie układu do wielu ofert. P: układ składa się z ograniczonych sekcji tekst/media/powiązanie z danymi, ma wersję, podgląd i walidację możliwości platformy. Nie zawiera wykonującego się kodu ani poświadczeń. Decyzji wymaga, czy Oferta zachowuje snapshot układu, czy wskazuje wersję; zmiana szablonu nie może bez ostrzeżenia aktualizować aktywnych ofert.

Kopiowanie układu nie kopiuje tożsamości Produktu, zewnętrznego ID Oferty, ceny, ilości ani KontaSprzedażowego. Zastosowanie układu do wielu Ofert jest Operacją z podglądem i wynikiem per obiekt.

### 9.6. Dashboard i problemy — propozycja do ADR-016

P: zakres dashboardu to zbiór wybranych kont i lokalnych danych, prezentowany jawnie. KPI Produktów, aktywnych Ofert, szkiców i błędów korzystają z definicji w systemie UI. Okres zmiany procentowej, znaczenie „ostatnie” i metryka wykresu wymagają decyzji. Dane sprzedaży pochodzą z Zamówień, nie z dowolnej sumy opublikowanych Ofert.

Problem ma stabilny kod, typ i ID obiektu, pole/obszar, wagę oraz status rozwiązania, jeśli będzie utrwalany. Deduplikacja zapobiega liczeniu jednego błędu z każdej próby jako osobnego problemu. Walidacja obowiązków platformy nie może narzucać warunków Allegro całej domenie Produktu. Bezpieczne szczegóły błędu są widoczne w Operacji, a nie w postaci pełnego tracebacku w panelu.

### 9.7. Właściciele i źródła danych rozszerzeń

P: opisy przyjęte przez użytkownika — Katalog lub Sprzedaż zależnie od celu; parametry i media katalogowe — Katalog; układ i nadpisania Oferty — Sprzedaż; wykonanie importu/AI/synchronizacji — Automatyzacja i wykonanie; potwierdzony status publikacji — obserwacja platformy. Własność cen i dostępności pozostaje N. Dostawca AI nie staje się źródłem prawdy o Produkcie.

Rozszerzenia przeglądać w etapie 3, zanim powstaną nowe tabele. Oznaczenie szczegółów jako P nie usuwa wymaganych opcji ze zakresu; oddziela zatwierdzony cel od jeszcze nieustalonego sposobu modelowania.
