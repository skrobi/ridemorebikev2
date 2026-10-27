# 16 · Testy i walidacja

## 1. Poziomy testów

| Poziom | Narzędzie | Zakres | Uruchomienie |
|---|---|---|---|
| Jednostkowe Python | `pytest` | geometria, filtry, cykle, markery, model dopasowania, kinematyka | `pytest analyzer/tests` |
| Jednostkowe/integracyjne PHP | `phpunit` | usługi domenowe, repozytoria (MariaDB testowa), middleware, importer | `vendor/bin/phpunit` |
| Kontrakt | JSON Schema | przykłady `schemas/examples/*.json`, wyniki testów E2E Pythona | `pytest analyzer/tests/contract` + `phpunit --group contract` |
| E2E Python | syntetyczne wideo | cały potok `analyze` na nagraniach generowanych z modelu (znana prawda) | `pytest analyzer/tests/e2e -m slow` |
| E2E web | Playwright | przepływ S01→S14 z workerem zastępczym (zwraca przygotowany `result.json`) | `npx playwright test` (tylko środowisko deweloperskie) |
| Dostępność | axe-core w Playwright | S01–S14 | jw. |
| Statyczna analiza | `phpstan` (poziom 6), `ruff` / `mypy` (Python) | cały kod | CI |

## 2. Testy Python – wymagane przypadki

| Obszar | Przypadki |
|---|---|
| Geometria | wektory z `docs/08`, rozdz. 6; konwersja obraz ↔ rower z przechyłem ±5°; kierunek jazdy w lewo |
| Okrąg toru | dopasowanie do punktów na okręgu z szumem 2 mm → środek ±2 mm, promień ±2 mm |
| Faza i cykle | syntetyczna trajektoria 88 obr./min z jednym zgubionym cyklem → poprawna liczba cykli i `reject_reason` |
| Filtr | sygnał 1,5 Hz + szum 15 Hz → po filtrze amplituda szumu < 10%, opóźnienie fazy 0 |
| Markery | obraz syntetyczny z 8 kołami na tle i stroju; zasłonięcie 1 markera → `source = model` / `missing` |
| Fuzja | reguły z `docs/07`, rozdz. 6 – tabela przypadków |
| Jakość | progi `docs/07`, rozdz. 9 – przypadki graniczne (0,849 vs 0,85) |
| Model dopasowania | tabelaryczne: dla każdej regulacji wartości < zielony, w żółtym, w zielonym, > zielony; każdy modyfikator; blokady akcji (modyfikatory mobilności nigdy nie blokują); akceptacja; kolejność następnego kroku; `safety_level` `caution` (krok ≤ 5 mm) i `stop` (brak zaleceń); `fit_level = basic`; każda reguła pewności z `docs/09`, rozdz. 6.1 osobno i łącznie (minimum `low`, maksimum `high`); `effort` dla każdej akcji; porada o korbie i sygnał wahania miednicy na progach |
| Reguły i plan | `measure_only` → brak zalecenia; `experimental` → krok 5 mm, pewność ≤ `medium`; pomiar `unvalidated` → brak zalecenia; wartość poza zielonym o mniej niż `min_margin` → brak akcji; strefa sporna → `near` bez akcji; koperta literatury przycina sumę modyfikatorów; łączenie przesunięć przez maksimum; pusty zakres → `unknown` + flaga; każdy warunek „nie oceniono” z kart reguł; ślad uzasadnienia kompletny; plan sekwencyjny: przykład z `docs/09` rozdz. 11 (pominięty mostek), odrzucenie kroku wypychającego inną regulację z `ok` do `out`, maks. 4 kroki |
| Ścieżki i wersje | `--storage-root` + ścieżka spoza → `E_JOB_INVALID`; nieznana wersja główna → `E_SCHEMA_VERSION`; zapis atomowy `result.json` (przerwanie w trakcie nie zostawia pliku końcowego) |
| Granice regulacji | z danymi: mostek 80 mm, potrzebne −20 mm → `tip.frame_size` (za duża); bez danych: potrzebne −35 mm → zalecenie „skróć o 35 mm” + `tip.frame_size`, −20 mm → samo zalecenie; brak danych → `missing_data`, pewność bez zmian |
| Bez danych roweru | rower tylko z typem: analiza, statusy i wielkość zmiany siodła identyczne (± 1 mm) jak z pełną geometrią na nagraniu syntetycznym; przykład z `docs/10` rozdz. 3.1 (≈ 15 mm); skala `body` vs `marker` różnica ≤ 5%; `scale_source = none` → tylko kierunek i krok 5 mm; korba z promienia toru ± 5 mm |
| Kinematyka | rozdzielenie `geometry` / `prediction`; kierunki: siodło +10 mm ⇒ `knee_flex_bottom` maleje; mostek −10 mm ⇒ `elbow_angle` maleje; podkładki +10 mm ⇒ `torso_angle` rośnie; zmiana kół ⇒ bez zmian kątów; wariant nieosiągalny ⇒ flaga |
| Ocena zmiany | każdy werdykt z `docs/09`, rozdz. 9 i kombinacje kryteriów; progi odczuć: spadek 5→3 (yes), 5→4 (partial), 3→2 (33% – yes), 4→6 (worse); bez dolegliwości – tylko oceny 5-stopniowe; RPE różne o 3 → `comparable = no` |
| CLI | kody wyjścia 0/2/3; `progress.json` aktualizowany; brak dostępu do sieci (test z blokadą gniazd) |

### Syntetyczne nagrania (prawda znana)

Generator `analyzer/tests/synth/render.py`: model sylwetki z `docs/10` + rower z geometrii → klatki 1920×1080 przy
60 fps z kolorowymi markerami (Ø 25 mm w skali), znacznikiem skali, szumem i rozmyciem ruchu; opcje: przechył
kamery, brak znacznika, zasłonięcie stopy, zmiana kadencji. Oczekiwania: błąd `knee_flex_bottom` ≤ 1°,
`torso_angle` ≤ 1°, `knee_over_pedal` ≤ 5 mm przy markerach i braku zakłóceń.

## 3. Testy PHP – wymagane przypadki

| Obszar | Przypadki |
|---|---|
| Autoryzacja | brak sesji → 302/401; zasób innego użytkownika → 404 dla każdego endpointu z `docs/13`, rozdz. 5 |
| CSRF | żądanie zmieniające stan bez tokenu → 403 |
| Walidacja | zakresy pól `docs/04`/`docs/05`; 422 z listą błędów |
| Klasyfikatory wywiadu | `MobilityClassifier` (średnia prób, kategorie z progów); `SafetyClassifier`: każde pytanie z `docs/04` rozdz. 5.1 → `stop`; operacja < 6 mies. z daty → `stop`; ≥ 6 mies. → `caution`; siła 8 → `stop`, 5–7 → `caution`; brak zgody `health_data` → `caution` + `basic` |
| Analityka | `EventLogger` odrzuca zdarzenia i klucze spoza białej listy; brak zapisu przy `analytics_opt_out`; retencja 12 mies. |
| Przypomnienia | maks. 2 na zmianę; e-mail tylko przy `notify_email = 1` |
| Przesyłanie | fragmenty w złej kolejności, wznowienie, za duży plik, zła sygnatura, porzucone przesyłanie → retencja |
| Kolejka – stany | każde przejście z tabeli `docs/02` rozdz. 5.1 dozwolone, każde inne odrzucone (wyjątek, bez zmiany stanu) |
| Kolejka – współbieżność | jednoczesne przejęcie przez 2 workery → jedno przejęcie; ponowienie z wykładniczym opóźnieniem; zawieszone zadanie (brak `heartbeat_at`) wraca do kolejki; limit czasu (zastępczy skrypt Pythona z `sleep`) |
| Kolejka – ponowne żądania | dwukrotne `POST /sessions/{id}/submit` (także równoległe) → jedno zadanie i ten sam `job_id` (`active_key`); korekta przy aktywnym zadaniu sesji → 409; ponowienie `failed` bez pliku → 409 z komunikatem „Nagraj ponownie” |
| Kolejka – przerwanie | zabicie workera w trakcie Pythona i w trakcie importu → zadanie wraca do kolejki, brak częściowych danych w tabelach, kadry przeniesione tylko po zatwierdzeniu transakcji; niekompletny `result.json` (zapis przerwany) nie jest importowany |
| Importer | poprawny `result.json` → komplet tabel; niezgodny ze schematem lub nieznana wersja główna → `failed` `E_INTERNAL` + wpis alertu; ponowny import idempotentny |
| Kontrakt – wersje | `job.json` z nieznaną wersją główną → Python `E_SCHEMA_VERSION` (kod 2); nieznane pole w `job.json` → odrzucenie; poprzednia wersja główna obsługiwana |
| Korekty punktów | korekta `hip` zmienia wszystkie zależne pomiary (kolano, biodro, tułów, ramię) i warianty; nowa analiza z `parent_id`, poprzednia bez zmian; wpis `fit_audit_log`; cofnięcie przywraca poprzednią jako bieżącą |
| Prywatność – separacja | dwa konta: każdy endpoint, obraz kadru i wynik innego użytkownika → 404; adres M01 cudzego szkicu → 404 |
| Prywatność – ścieżki | `job.json` ze ścieżką spoza `storage_root`, z `..` lub przez dowiązanie symboliczne → Python `E_JOB_INVALID` (kod 2), żaden plik nie jest otwarty; PHP nie przyjmuje ścieżek od użytkownika |
| Prywatność – usuwanie | usunięcie nagrania na żądanie i przez retencję usuwa: plik, kadry, fragmenty `uploads/`, katalog `jobs/{id}`, `tracks.npz`; wpis w audycie; dane po nieudanej analizie usuwane po `retention.failed_video_days` |
| Prywatność – logi i uprawnienia | po scenariuszach E2E skan logów (`app-*.log`, `worker.log`) na kody obszarów dolegliwości, odpowiedzi `safety.*`, współrzędne i nazwy plików – brak trafień; pliki 0640, katalogi 0750; proces Pythona bez dostępu do sieci (test z blokadą gniazd) |
| Retencja | każda reguła z `docs/12`, rozdz. 4; wpis w audycie |
| Szacunek | suma różnic wariantów; brak wariantu → 422 |
| Zmiana względna | zapis „+15 mm” bez znanej wartości wyjściowej tworzy wersję z `delta`; ze znaną – liczy `value_after` |
| Wersje ustawień | zapis zmiany tworzy nową wersję; poprzednia niezmieniona; analiza wskazuje `setup_id` z chwili wysłania – późniejsza edycja ustawień nie zmienia tego powiązania |

## 4. Testy E2E web (Playwright)

Scenariusze na danych testowych (worker zastępczy zwraca pliki z `tests/fixtures/results/`):
1. Pierwsza analiza: S01 (pusty) → dodanie roweru → S02–S06 (pełny wywiad) → S07 → S08 (plik testowy MP4 2 MB) → S09 → S10 → S11.
2. Korekta punktu w S10 → nowa wersja wyniku, przywrócenie.
3. Zmiana profilu jazdy w S11 → `refit` → inne statusy.
4. „Mi tak wygodnie” → status „Twoje ustawienie”, następny krok zmienia się.
5. S12: wybór wariantu, kombinacja (szacunek), „Przelicz dokładnie”, zapis planu → S13 → przypomnienie.
6. Druga analiza (krótki wywiad) → S14 z werdyktem.
7. Nieudana analiza (`E_VIDEO_FPS_LOW`) → komunikat i „Nagraj ponownie”.
8. Widok 1440 px i 390 px: brak poziomego przewijania; brak błędów JS; axe bez naruszeń krytycznych.
9. Szybki start → wynik „Dopasowanie podstawowe” → uzupełnienie wywiadu → `refit` → `fit_level = full`.
10. Pytanie bezpieczeństwa „tak” → S06 z komunikatem → S11 bez zaleceń, karta „Najpierw konsultacja”, S12 tylko podgląd.
11. M01 w widoku 390 px (zalogowany): przesłanie pliku; S08 na komputerze pokazuje „Nagranie dotarło z telefonu”.
12. Okno „Zaplanuj” → przypomnienie `planned_change` na pulpicie; ocena po jeździe jednym dotknięciem (5 stopni).

## 5. Środowisko testowe

- MariaDB w kontenerze (wersja zgodna z hostingiem), baza `fit_test` z migracjami.
- Python 3.9 venv z `requirements.txt` + `requirements-dev.txt`.
- Na Windows (środowisko deweloperskie): WSL2 lub Docker Desktop; instrukcja w `README` po E1.

## 6. Walidacja pomiarów i reguł

Testy na nagraniach syntetycznych (rozdz. 2) sprawdzają oprogramowanie, ale **nie zastępują** porównania
z pomiarem referencyjnym na rzeczywistych nagraniach. Walidacja ma trzy poziomy, zgodne z kartą modelu
(`docs/09`, rozdz. 13): pomiar → przewidywanie → zalecenie.

### 6.1 Kryteria dokładności pomiaru (bramka etapu M1)

Referencja (D-17): (a) **wykrywanie punktów** – te same klatki oznaczone ręcznie przez 2 przeszkolone osoby
(błąd oprogramowania); (b) **trafność biomechaniczna** – laboratoryjny system 3D lub system fittera z markerami
aktywnymi w tej samej sesji (ograniczenia 2D); (c) **powtarzalność** – 2 sesje tego samego dnia, markery zdjęte
i naklejone ponownie. Próba: ≥ 20 osób (różne sylwetki, szosa i gravel/MTB), plus ≥ 10 nagrań wykonanych przez
uczestników samodzielnie w domu według instrukcji (odsetek nieudanych analiz, rozkład jakości).

Kryteria robocze (do zatwierdzenia przez radę modelu, D-17): bias = średnia różnica do referencji (b),
LoA = granice zgodności Blanda–Altmana 95%, MDC = najmniejsza wykrywalna zmiana z (c).

| Pomiar | Z markerami: bias / LoA / MDC | Bez markerów: bias / LoA / MDC |
|---|---|---|
| `knee_flex_bottom`, `knee_flex_top` | ≤ 2° / ± 5° / ≤ 3° | ≤ 3° / ± 8° / ≤ 5° |
| `hip_angle` (min) | ≤ 3° / ± 6° / ≤ 4° | ≤ 4° / ± 10° / ≤ 6° |
| `torso_angle` | ≤ 2° / ± 4° / ≤ 3° | ≤ 3° / ± 6° / ≤ 4° |
| `elbow_angle`, `shoulder_angle` | ≤ 3° / ± 6° / ≤ 4° | ≤ 4° / ± 10° / ≤ 6° |
| `ankle_angle`, `foot_angle` | ≤ 3° / ± 8° / ≤ 5° | nie walidujemy – pomiar ukryty |
| `knee_over_pedal` | ≤ 5 mm / ± 12 mm / ≤ 8 mm | nie walidujemy – wymaga skali z markera |
| `pelvis_vertical` (rom) | ≤ 3 mm / ± 6 mm / ≤ 4 mm | nie walidujemy |

Wynik zapisuje się w `pipeline.json` jako `metrics.validation_status[markers_mode][metric]` (`docs/07`,
rozdz. 11) – zmiana = nowa wersja algorytmu:
- `validated` – spełnia kryteria; może sterować regułą (`docs/09`, rozdz. 1.1);
- `unvalidated` – nie zbadano lub nie spełnia; widoczny tylko w „Szczegółach” z plakietką „pomiar
  niezweryfikowany”, nigdy nie steruje zaleceniem;
- `hidden` – bias lub LoA > 2× kryterium; nie pokazujemy.
Przed walidacją wszystkie pomiary mają `unvalidated`, więc żadna reguła nie wyda zalecenia.

### 6.2 Badania

| Badanie | Protokół | Kryterium (`docs/01`, rozdz. 7) |
|---|---|---|
| V1.1 Dokładność i powtarzalność pomiaru | rozdz. 6.1 (a)–(c), z markerami i bez | kryteria z tabeli rozdz. 6.1 dla każdego pomiaru i trybu |
| V1.1 Nagrania domowe | ≥ 10 nagrań uczestników wg instrukcji S07–S08 | nieudane analizy ≤ 20%; przyczyny odrzuceń zrozumiałe dla uczestnika (ankieta) |
| Markery vs bez markerów | te same nagrania analizowane w trybie `none` | różnica raportowana; komunikat jakości w trybie `none` dostosowany |
| Wrażliwość na ustawienie kamery | obrót kamery 5° i 10° od prostopadłej, wysokość ±30 cm | wpływ raportowany; progi ostrzeżeń kalibrowane |
| V4.1 Reguła wysokości siodła | ≥ 20 osób; fitter ocenia niezależnie (bez wglądu w wynik aplikacji), czy zalecenie aplikacji jest zasadne; zmiana o krok eksperymentalny, nagranie przed/po, odczucia po ≥ 3 jazdach | zasadność wg fittera ≥ 80%; „gorzej” ≤ 10%; \|Δ przewidziana − Δ zmierzona\| ≤ 2° w ≥ 80% → przejście `experimental` → `validated` |
| V5.2 Pozostałe reguły i model wariantów | jw. dla `saddle_fore_aft`, `bar_reach`, `bar_height`; rzeczywiste zmiany siodła ±10 mm i mostka ±10 mm | kryteria z kart reguł (`docs/09`, rozdz. 1.1); kalibracja `torso_follow` |
| Zrozumiałość | 5 laików przechodzi S01→S11 bez pomocy | ≥ 4/5 poprawnie opisuje, co ma zmienić; ≥ 4/5 poprawnie wyjaśnia znaczenie poziomu pewności i paska zakresu (w tym osoba z zaburzeniem widzenia barw, jeśli możliwe) |
| Korekta 2D kolana (C-21) | ≥ 10 osób: kąt kolana z nagrania 2D vs pomiar 3D lub goniometr fittera w DMP | średnia różnica i granice zgodności; decyzja o `metrics.knee_2d_offset_deg` (zmiana = nowa wersja algorytmu) |
| Testy mobilności w domu (C-02) | ≥ 15 osób wykonuje T2–T4 samodzielnie według instrukcji z S04; fizjoterapeuta mierzy te same testy | ICC samodzielny vs fizjoterapeuta, zgodność kategorii; decyzja o progach (D-07) |
| Czas wywiadu (C-20) | zdarzenia `interview.step_completed` z `duration_ms` (`fit_event`) | mediana pełnego ≤ 5 min, krótkiego i szybkiego startu ≤ 1 min |

Nagrania walidacyjne – wyłącznie za pisemną zgodą uczestników, przechowywane poza repozytorium.
