# 20 · Przegląd dokumentacji przez panel ekspertów

**Charakter dokumentu.** Panel jest **symulowany**: stanowiska zostały opracowane na podstawie literatury naukowej
i dobrych praktyk dla każdej z ról, aby uporządkować założenia przed implementacją. Nie zastępuje opinii
rzeczywistych specjalistów – uzgodnione zmiany oznaczone jako wymagające potwierdzenia trafiają do rejestru
decyzji (`docs/18`, D-06, D-07, D-15, D-16).

## 1. Skład panelu

| Rola | Niezależny konsultant | Perspektywa eksperta | Perspektywa konsultanta |
|---|---|---|---|
| Fizjoterapeuta sportowy (FZ) | fizjoterapeuta – metodolog pomiarów (FZ-K) | pozycja ma wynikać z ruchomości i bólu; więcej testów i liczb | testy wykonywane samodzielnie w domu są mało wiarygodne; mniej, ale pewniej |
| Ortopeda (OR) | lekarz medycyny sportowej (OR-K) | bezpieczeństwo: urazy, operacje, czerwone flagi | unikać medykalizacji i fałszywych alarmów; nie twierdzić przyczynowo |
| Psycholog zdrowia (PS) | badaczka komunikacji ryzyka i numeryczności (PS-K) | uczciwa komunikacja niepewności, unikanie nocebo, poczucie sprawczości | laicy gubią się w liczbach; obraz i prostota przed szczegółem |
| Behawiorysta (BH) | etyk projektowania cyfrowego (BH-K) | doprowadzić użytkownika od zalecenia do wykonania i sprawdzenia | bez manipulacji: prywatność i autonomia, uwaga na zmęczenie powiadomieniami |
| Product owner (PO) | konsultant lean / growth (PO-K) | spójny zakres MVP, mierniki sukcesu, długość wywiadu | jeszcze krótsza droga do pierwszej wartości |
| Projektant UX/UI (UX) | specjalista dostępności (UX-K) | przekazanie nagrania z telefonu, stopniowe ujawnianie, czytelne paski | WCAG 2.2, daltonizm, realna obsługa kamery w przeglądarkach |

## 2. Metoda

1. Każdy ekspert przeczytał `docs/01–19` i makiety; spisał tezy i propozycje.
2. Każdy konsultant przygotował stanowisko odmienne, oparte na innych źródłach.
3. Debata według sporów (rozdz. 4); rozstrzygnięcie = zmiana komplementarna (nie usuwa spójności założeń)
   z identyfikatorem `C-NN`.
4. Zmiany wprowadzono do dokumentów wskazanych w rozdz. 5; nowe zasady komunikacji w `docs/21-komunikacja.md`.

## 3. Stanowiska

### 3.1 Fizjoterapeuta (FZ) vs konsultant (FZ-K)

**FZ – teza:** ból i ruchomość są głównym powodem, dla którego rowerzysta szuka dopasowania. W badaniu 109
zawodowych kolarzy 58% zgłaszało ból dolnego odcinka pleców, a 36% ból przedniej części kolana w ciągu roku
(Clarsen i in. 2010). Dlatego aplikacja powinna:
- mierzyć ból skalą NRS 0–10 z rozróżnieniem rodzaju (ból / drętwienie / mrowienie / ucisk), czasu trwania po jeździe,
  częstości i tego, czy występuje także poza rowerem,
- oceniać zmianę bólu progiem klinicznie istotnym: ok. 2 punkty lub ok. 30% (Farrar i in. 2001), a nie 1 punktem,
- zbierać wyniki testów liczbowo i obustronnie oraz rejestrować intensywność jazdy w trakcie nagrania (przy dużym
  obciążeniu zmienia się kinematyka stawu skokowego – Bini i in.),
- dodać pomiar zgięcia kolana w górnym martwym punkcie: dłuższe korby zwiększają zgięcie kolana i biodra
  (Ferrer-Roca i in.), krótsze je zmniejszają (Reynolds i in. 2025), a duże zgięcie kolana wiąże się z obciążeniem
  stawu rzepkowo-udowego.

**FZ-K – kontrteza:** brak badań, które potwierdzają rzetelność testów ruchomości wykonywanych **samodzielnie
w domu**; dobra rzetelność dotyczy pomiarów wykonywanych przez przeszkolonych badaczy (np. test kolano–ściana:
ICC 0,97–0,99 u Bennell i in. 1998; aplikacja poziomicy iPhone ICC 0,85 w warunkach badania). Test „siad
i skłon” ma tylko umiarkowaną trafność wobec rozciągliwości tylnej taśmy uda i niską wobec odcinka lędźwiowego
(metaanaliza Mayorga-Vega i in. 2014). Wniosek: testy traktować jako orientacyjne, ograniczyć ich wpływ na zakresy
i nie mnożyć pytań.

**Wynik debaty:** testy podzielono na **podstawowe** (siad i skłon, prosta noga w górę, kolano do ściany) i
**dodatkowe** (skłon stojąc, kolano do klatki, skręt tułowia). Każdy wynik liczbowy podaje się jako średnią z 2 prób,
z informacją „z pomocą drugiej osoby / sam”. Wpływ mobilności na zakresy ograniczono do przesunięć (nigdy blokady
zalecenia) i oznaczono jako „orientacyjny” (C-01, C-02). Próg istotnej zmiany bólu: ≥ 2 pkt lub ≥ 30% (C-12).
Dodano zgięcie kolana w górze obrotu i intensywność nagrania (C-09, C-10).

### 3.2 Ortopeda (OR) vs konsultant (OR-K)

**OR – teza:** aplikacja, która mówi „zmień pozycję”, musi najpierw wykluczyć sytuacje, w których nie powinna
nic zalecać. Standardowe ścieżki triażu wskazują m.in. ból nocny niezwiązany z wysiłkiem, obrzęk, blokowanie lub
uciekanie stawu, utrzymujące się drętwienie lub osłabienie jako sygnały do konsultacji (ścieżki NHS Highland,
NHS Borders). Potrzebna jest ustrukturyzowana historia operacji (więzadło krzyżowe, łąkotka, endoproteza biodra lub
kolana, kręgosłup) z datą i informacją o zaleceniach lekarza. Drętwienie krocza należy rozdzielić od bólu siedzenia:
kolarstwo wiąże się z częstszym drętwieniem krocza, a zmiana siodła (np. bez nosa) zmniejsza nacisk
(metaanaliza Litwinowicz i in. 2020).

**OR-K – kontrteza:** czerwone flagi zgłaszane samodzielnie mają niską swoistość – zbyt szeroka lista wystraszy
użytkowników i zablokuje większość analiz. Ponadto związek wysokości siodła z bólem rzepkowo-udowym jest
niejednoznaczny: przegląd z 2001 r. wymienia wiele czynników, siły w stawie zmieniają się o kilka procent
(badanie z 2012 r.), a praca z 2020 r. wskazuje raczej na korzyść wyższego siodła. Aplikacja nie może sugerować
przyczyny bólu.

**Wynik debaty:** dwa poziomy bezpieczeństwa (C-03):
- **„Wstrzymaj zmiany”** – wąska lista silnych sygnałów (operacja < 6 mies. lub obowiązujące ograniczenia od
  lekarza, obrzęk z blokowaniem/uciekaniem stawu, drętwienie lub osłabienie utrzymujące się > 24 h, ból nocny
  niezwiązany z jazdą, upadek < 6 tyg. z utrzymującym się bólem, ból ≥ 8/10; drętwienie krocza z zaburzeniami
  oddawania moczu lub stolca – dodatkowo komunikat o pilnej konsultacji): pomiar jest pokazany, zalecenia zmian –
  nie; komunikat o konsultacji.
- **„Ostrożnie”** – ból 5–7/10, ból także poza rowerem, operacja dawniej niż 6 mies.: zalecenia z krokiem maks. 5 mm
  i sugestią konsultacji.
Teksty nie przypisują przyczyny bólu (C-15, `docs/21`). Dodano historię operacji (C-04) i osobny obszar
„drętwienie krocza” z modyfikatorem (C-05).

### 3.3 Psycholog zdrowia (PS) vs konsultantka (PS-K)

**PS – teza:** ujawnianie niepewności tylko nieznacznie obniża zaufanie, zwłaszcza gdy podaje się ją jako zakres
liczbowy, a nie słowne zastrzeżenie (van der Bles i in. 2020). Każde zalecenie powinno mieć **poziom pewności**.
Sformułowania typu „przeciąża kolano” mogą wywoływać efekt nocebo; pozytywne ujęcie informacji może go ograniczać
(przegląd Barnes i in. 2019). Etykiety jak „mniej giętki” mogą stygmatyzować. Poczucie sprawczości rośnie,
gdy wiadomo, ile czasu i jakich narzędzi wymaga zmiana.

**PS-K – kontrteza:** osoby o niskiej numeryczności lepiej rozumieją ryzyko i informacje zdrowotne z pomocą dobrze
zaprojektowanych obrazów (np. tablice ikon) niż z liczb (przeglądy Garcia-Retamero i Cokely). Dodatkowe liczby
i przedziały w podsumowaniu zwiększą obciążenie. Wiele pytań o ból kieruje uwagę na ból – zaczynać od jednego
ogólnego pytania o komfort.

**Wynik debaty:** poziom pewności jako **słowo + ikona** (wysoka / średnia / niska) z rozwijanym „dlaczego?”;
liczby i przedziały – w „Szczegółach” (C-06). Język pozytywny i nieprzyczynowy, bez etykiet osobowych (C-15).
Na każdym zaleceniu: czas, narzędzia, trudność, „możesz to zlecić w serwisie” (C-07). Ankieta po jeździe zaczyna się
od jednego pytania ogólnego (C-13).

### 3.4 Behawiorysta (BH) vs konsultant (BH-K)

**BH – teza:** największa strata następuje między zaleceniem a wykonaniem. Intencje wdrożeniowe („kiedy i gdzie
to zrobię”) mają w metaanalizie średni efekt d ≈ 0,65 (Gollwitzer i Sheeran 2006). Przypomnienia tekstowe
zwiększają przestrzeganie zaleceń zdrowotnych (metaanalizy przypomnień SMS). Ustawienia domyślne mają silny wpływ
na decyzje (metaanaliza Jachimowicz i in. 2019, d ≈ 0,68) – domyślnie proponować zachowanie kadrów, bo poprawia
to weryfikację i symulację.

**BH-K – kontrteza:** długoterminowy efekt przypomnień jest niepewny, a ich nadmiar zniechęca. Ustawienie domyślne
dotyczące wizerunku musi chronić prywatność (RODO art. 25 – ochrona danych w fazie projektowania i domyślna).
Wspieranie autonomii wiąże się z lepszą motywacją i wynikami zdrowotnymi (metaanaliza Ng i in. 2012, teoria
samostanowienia) – użytkownik sam decyduje o przypomnieniach; bez serii i grywalizacji w kontekście bólu.

**Wynik debaty:** przy następnym kroku (S01, S11) okno „Zaplanuj”: „Kiedy to zrobisz?” i „Na których jazdach
sprawdzisz?”, przenoszone do S13 (C-08); przypomnienia w
aplikacji domyślnie, e-mail tylko po włączeniu, maks. 2 przypomnienia na zmianę (C-14); domyślna polityka nagrania
pozostaje **`delete_after`** (rozstrzyga D-05), a S08 opisuje korzyść `keyframes_only` neutralnie, bez
wstępnego zaznaczenia (C-16). Ankieta po jeździe jednym dotknięciem na pulpicie (C-13).

### 3.5 Product owner (PO) vs konsultant (PO-K)

**PO – teza:** zakres jest spójny, ale brakuje mierników sukcesu i budżetu czasu. Dłuższy deklarowany czas ankiety
zmniejsza liczbę osób, które ją zaczynają (Galesic i Bosnjak 2009). Pierwszy pełny wywiad ≤ 5 min, krótki ≤ 1 min;
wymiary opcjonalne pytać wtedy, gdy są potrzebne (np. kości kulszowe – gdy zgłoszono dolegliwości siodła). Nagranie
powstaje na telefonie – przenoszenie 300 MB na komputer to największe tarcie.

**PO-K – kontrteza:** pierwsza wartość powinna przyjść jeszcze szybciej: „szybki start” bez wywiadu, a markery
domyślnie wyłączone.

**Wynik debaty:** „Szybki start” (ok. 1 min: rower, profil, główna dolegliwość, pytania bezpieczeństwa) jest
dostępny, ale wynik jest oznaczony jako „dopasowanie podstawowe”, a pełny wywiad odblokowuje „dopasowanie pełne”
(C-11). Pytania bezpieczeństwa zostają obowiązkowe (OR). Markery pozostają zalecane, nie wymagane (bez zmian).
Mierniki sukcesu i zdarzenia analityczne bez danych o zdrowiu (C-17). Przekazanie nagrania z telefonu (C-18).

### 3.6 Projektant UX/UI (UX) vs konsultant (UX-K)

**UX – teza:** proces wymaga stopniowego ujawniania informacji, oszacowania czasu kroków, zapisywania postępu
i przekazania nagrania z telefonu przez kod QR z prowadzeniem kadru na żywo (podgląd kamery z sylwetką).

**UX-K – kontrteza:** zaburzenia rozróżniania barw czerwono-zielonych dotyczą do 8% mężczyzn pochodzenia
północnoeuropejskiego – paski czerwono-zielone są ryzykowne. Obowiązuje WCAG 2.2 (m.in. minimalny rozmiar celu
24×24 px). Podgląd kamery w przeglądarce telefonu działa różnie na iOS i Androidzie – w MVP prostsze rozwiązanie.

**Wynik debaty:** strefy pasków: zielona + neutralne (piaskowa/szara), czerwień tylko dla błędów, „Wstrzymaj zmiany” i werdyktu „Gorzej”;
status zawsze ikona + tekst (C-19). Oszacowanie czasu przy krokach (C-20). Przekazanie z telefonu w MVP: kod QR →
prosta strona przesyłania na telefonie ze statyczną instrukcją kadru; prowadzenie kadru na żywo po MVP (C-18, D-16).

## 4. Główne spory i rozstrzygnięcia

| Spór | Strona A | Strona B | Rozstrzygnięcie |
|---|---|---|---|
| Liczba i waga testów mobilności | FZ: więcej, liczbowo, obustronnie | FZ-K: testy domowe niepotwierdzone | 3 podstawowe + 3 dodatkowe, średnia z 2 prób, wpływ tylko jako przesunięcie zakresu (C-01, C-02) |
| Siódmy test „głęboki przysiad” (D-14) | FZ: ocena łączna biodro–kostka | FZ-K, PO: dubluje kolano–ściana i Thomasa, wydłuża wywiad | nie dodajemy (D-14 zamknięta) |
| Czerwone flagi | OR: szeroki triaż | OR-K: niska swoistość | 2 poziomy, wąska lista „Wstrzymaj” (C-03) |
| Przyczynowość ból ↔ ustawienie | FZ: wiązać dolegliwości z regulacjami | OR-K: dowody niejednoznaczne | dolegliwości zmieniają kolejność i pewność, teksty nieprzyczynowe (C-15) |
| Korekta zaniżenia kąta kolana w 2D | FZ: dodać ok. 2,2° (praca z 2014 r.) | FZ-K: zależy od ustawienia kamery | brak korekty, bo zakres Ferrer-Roca także pochodzi z pomiaru 2D; parametr konfiguracyjny `knee_2d_offset_deg = 0` do walidacji (C-21) |
| Liczby vs obraz | PS: przedziały liczbowe budują zaufanie | PS-K: niska numeryczność | podsumowanie: obraz + słowo; liczby w szczegółach (C-06) |
| Domyślna retencja | BH: `keyframes_only` | BH-K: prywatność domyślna | `delete_after` domyślnie, neutralna informacja o korzyści (C-16) |
| Przypomnienia | BH: przypominać | BH-K: zmęczenie, autonomia | w aplikacji, e-mail opt-in, maks. 2 na zmianę (C-14) |
| Szybki start | PO-K: bez wywiadu | OR/FZ: bezpieczeństwo, dopasowanie | 1-minutowy szybki start z pytaniami bezpieczeństwa, wynik „podstawowy” (C-11) |
| Kamera na żywo | UX: prowadzenie kadru | UX-K: zgodność przeglądarek | QR + przesyłanie w MVP, prowadzenie na żywo po MVP (C-18) |
| Kolory marki vs dostępność | UX: paleta ridemore | UX-K: daltonizm | zielony + neutralne strefy, ikony obowiązkowe (C-19) |

## 5. Uzgodnione zmiany

| ID | Zmiana | Dokumenty | Etap |
|---|---|---|---|
| C-01 | Testy mobilności: 3 podstawowe (T2 siad i skłon, T3 prosta noga, T4 kolano–ściana) + 3 dodatkowe (T1, T5, T6) | 04, 03 | MVP |
| C-02 | Metadane testów: średnia z 2 prób, „sam / z pomocą”, ból podczas testu; wpływ mobilności tylko jako przesunięcie zakresu, poziom „orientacyjny” | 04, 09, 12 | MVP |
| C-03 | Bezpieczeństwo: poziomy `stop` i `caution`, lista sygnałów, zachowanie zaleceń | 04, 09, 03, 11, 12, 15 | MVP |
| C-04 | Historia operacji (rodzaj, data, zalecenia lekarza) | 04, 09, 12 | MVP |
| C-05 | Obszar „drętwienie krocza” + modyfikator `M_PERINEAL` | 04, 09 | MVP |
| C-06 | Poziom pewności zalecenia (wysoka/średnia/niska) z wyjaśnieniem | 09, 11, 03, 14, 21 | MVP |
| C-07 | Wysiłek zmiany: czas, narzędzia, trudność, „zrób w serwisie” | 09, 03, 14 | MVP |
| C-08 | Intencja wdrożeniowa: okno „Zaplanuj” przy następnym kroku (kiedy, na których jazdach) → S13 | 03, 12, 13 | MVP |
| C-09 | Nowy pomiar: zgięcie kolana w górze obrotu (`knee_flex_top`) + porada o długości korby | 08, 09 | MVP |
| C-10 | Intensywność nagrania (RPE 0–10, opcjonalnie moc) i ostrzeżenie przy RPE ≥ 7 | 05, 06, 07, 11, 12 | MVP |
| C-11 | Szybki start (~1 min) z wynikiem „dopasowanie podstawowe” | 01, 03, 04, 09 | MVP |
| C-12 | Ocena zmiany: istotna poprawa bólu ≥ 2 pkt lub ≥ 30%; 5-stopniowa ocena globalna | 09, 03, 04 | MVP |
| C-13 | Szczegóły dolegliwości (rodzaj, czas trwania po jeździe, częstość, poza rowerem); ankieta po jeździe jednym dotknięciem; trend komfortu na pulpicie | 04, 03, 12 | MVP |
| C-14 | Preferencje powiadomień: w aplikacji domyślnie, e-mail opt-in, maks. 2 na zmianę | 03, 12, 13 | MVP |
| C-15 | Zasady języka: nieprzyczynowo, pozytywnie, bez etykiet osobowych; nowe nazwy statusów | 21, 09, 03 | MVP |
| C-16 | Domyślna polityka nagrania `delete_after` (D-05 zamknięta) | 06, 18, 03 | MVP |
| C-17 | Mierniki sukcesu i zdarzenia analityczne bez danych o zdrowiu | 01, 12, 15 | MVP |
| C-18 | Przekazanie nagrania z telefonu (kod QR, strona M01); prowadzenie kadru na żywo po MVP | 03, 06, 13, 15 | MVP / po MVP |
| C-19 | Paski: strefa zielona + neutralne; czerwień tylko dla błędów; ikony obowiązkowe; WCAG 2.2 AA | 14, 03 | MVP |
| C-20 | Szacowany czas przy krokach przepływu i zapis postępu | 03, 14 | MVP |
| C-21 | Parametr `metrics.knee_2d_offset_deg` (domyślnie 0) i badanie walidacyjne | 07, 16 | MVP / walidacja |
| C-22 | Zalecenie drugiego nagrania (druga strona) przy asymetrii lub jednostronnych dolegliwościach po stronie niefilmowanej | 06, 09 | MVP (informacja) |
| C-23 | Wahanie miednicy w pionie jako sygnał wspierający dla wysokości siodła (podnosi pewność „obniż siodło”) | 08, 09 | MVP |

## 6. Nowe parametry pomiaru i wywiadu (zestawienie)

| Parametr | Typ | Źródło zmiany |
|---|---|---|
| `knee_flex_top` – zgięcie kolana w górze obrotu | pomiar z nagrania | C-09 |
| `signals.pelvis_rock` – nadmierne wahanie miednicy w pionie (`pelvis_vertical.rom`) | pochodny pomiaru | C-23 |
| `session.recording_rpe`, `session.recording_power_w` | parametry nagrania | C-10 |
| `mob.*.trials`, `mob.*.helper`, `mob.*.pain` | metadane testów | C-02 |
| `complaint.type`, `complaint.duration_after`, `complaint.frequency`, `complaint.off_bike` | wywiad | C-13 |
| `safety.*` (7 pytań), `safety_level`, `fit_level` | wywiad / model | C-03, C-11 |
| `complaint.is_primary` – główna dolegliwość (dolegliwość docelowa oceny zmiany) | wywiad | C-12 |
| `surgery[]` (rodzaj, data, ograniczenia) | wywiad | C-04 |
| `ride.global_comfort` (5 stopni) | ocena po jeździe / zmiany | C-12, C-13 |
| `plan.planned_date`, `plan.test_rides` | plan zmiany | C-08 |
| `recommendation.confidence`, `recommendation.effort` | wynik | C-06, C-07 |

## 7. Czego panel świadomie nie zmienił

- 4 regulacje i zasada „jedna zmiana naraz” – zgoda wszystkich ról.
- Obliczenia wyłącznie po stronie serwera (ADR-05).
- Zakresy bazowe z `docs/09` – zmiany zakresów wymagają rzeczywistego bike fittera (D-06).
- Brak symulacji pracy mięśni (ADR-14).

## 8. Ograniczenia panelu

- Panel symulowany; część źródeł sprawdzono na podstawie streszczeń (`docs/19`).
- Progi (lista „Wstrzymaj”, kategorie mobilności, wahanie miednicy) są robocze – do zatwierdzenia przez
  fizjoterapeutę i lekarza (D-07, D-15).
