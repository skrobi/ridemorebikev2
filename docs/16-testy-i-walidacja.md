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
| Model dopasowania | tabelaryczne: dla każdej regulacji wartości < zielony, w żółtym, w zielonym, > zielony; każdy modyfikator; blokady akcji; akceptacja; kolejność następnego kroku |
| Kinematyka | kierunki: siodło +10 mm ⇒ `knee_flex_bottom` maleje; mostek −10 mm ⇒ `elbow_angle` maleje; podkładki +10 mm ⇒ `torso_angle` rośnie; zmiana kół ⇒ bez zmian kątów; wariant nieosiągalny ⇒ flaga |
| Ocena zmiany | każdy werdykt z `docs/09`, rozdz. 9 i kombinacje kryteriów |
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
| Przesyłanie | fragmenty w złej kolejności, wznowienie, za duży plik, zła sygnatura, porzucone przesyłanie → retencja |
| Kolejka | jednoczesne przejęcie przez 2 workery → jedno przejęcie; ponowienie z wykładniczym opóźnieniem; zawieszone zadanie; limit czasu (zastępczy skrypt Pythona z `sleep`) |
| Importer | poprawny `result.json` → komplet tabel; niezgodny ze schematem → `failed` `E_INTERNAL`; ponowny import idempotentny |
| Retencja | każda reguła z `docs/12`, rozdz. 4; wpis w audycie |
| Szacunek | suma różnic wariantów; brak wariantu → 422 |
| Wersje ustawień | zapis zmiany tworzy nową wersję; poprzednia niezmieniona |

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

## 5. Środowisko testowe

- MariaDB w kontenerze (wersja zgodna z hostingiem), baza `fit_test` z migracjami.
- Python 3.9 venv z `requirements.txt` + `requirements-dev.txt`.
- Na Windows (środowisko deweloperskie): WSL2 lub Docker Desktop; instrukcja w `README` po E1.

## 6. Walidacja pomiarów (etap 6)

| Badanie | Protokół | Kryterium (`docs/01`, rozdz. 7) |
|---|---|---|
| Powtarzalność | ≥ 10 osób × 2 nagrania tego samego dnia bez zmian ustawień, markery zdjęte i naklejone ponownie | różnica `knee_flex_bottom`, `torso_angle` ≤ 3° (95% przypadków); SD różnic raportowane |
| Zgodność z fitterem | ≥ 10 osób; fitter mierzy goniometrem / własnym systemem w tej samej sesji | Bland–Altman; rozbieżność ≤ 5° w ≥ 80% |
| Markery vs bez markerów | te same nagrania analizowane w trybie `none` | różnica raportowana; komunikat jakości w trybie `none` dostosowany |
| Wrażliwość na ustawienie kamery | obrót kamery 5° i 10° od prostopadłej, wysokość ±30 cm | wpływ raportowany; progi ostrzeżeń kalibrowane |
| Model wariantów | rzeczywiste zmiany siodła ±10 mm i mostka ±10 mm z nagraniem przed/po | |Δ przewidziana − Δ zmierzona| ≤ 2° dla kolana |
| Zrozumiałość | 5 laików przechodzi S01→S11 bez pomocy | ≥ 4/5 poprawnie opisuje, co ma zmienić |

Nagrania walidacyjne – wyłącznie za pisemną zgodą uczestników, przechowywane poza repozytorium.
