# 17 · Plan zadań

Każde zadanie jest kontraktem implementacyjnym (`CLAUDE.md`, rozdz. 1 i 8). Kolejność jest zalecana; kolumna
„Zależy od” wskazuje twarde zależności. Status: `todo` · `in_progress` · `done` · `blocked`.

## Przegląd

| ID | Zadanie | Zależy od | Status |
|---|---|---|---|
| F0.1 | Szkielet repozytorium i narzędzi | – | todo |
| F0.2 | Weryfikacja hostingu | F0.1 | todo |
| F0.3 | Kontrakty JSON Schema i przykłady | F0.1 | todo |
| P1.1 | Geometria i pomiary | F0.1 | todo |
| P1.2 | Generator syntetycznych nagrań | P1.1 | todo |
| P1.3 | Odczyt i walidacja wideo, wybór fragmentu | F0.2 | todo |
| P1.4 | Markery i znacznik skali | P1.2, P1.3 | todo |
| P1.5 | Estymatory pozy i porównanie modeli | P1.3, F0.2 | todo |
| P1.6 | Fuzja, śledzenie, wygładzanie | P1.4, P1.5 | todo |
| P1.7 | Kalibracja, faza korby, cykle | P1.6 | todo |
| P1.8 | Statystyki, jakość, kadry kluczowe, `analyze` E2E | P1.7, F0.3 | todo |
| P1.9 | Model dopasowania v1 | P1.1, F0.3 | todo |
| P1.10 | Kinematyka, warianty, `correct` / `refit` / `simulate` | P1.8, P1.9 | todo |
| B2.1 | Rdzeń PHP: front controller, router, middleware, układ | F0.1 | todo |
| B2.2 | Migracje bazy | B2.1 | todo |
| B2.3 | Profil, zgody, rowery, ustawienia, historia | B2.2 | todo |
| B2.4 | Wywiad (S02–S06) | B2.3 | todo |
| B2.5 | Markery, nagranie, przesyłanie (S07–S08) | B2.4 | todo |
| B2.6 | Kolejka, worker, importer, retencja (S09) | B2.5, F0.3 | todo |
| U3.1 | S10 Sprawdź punkty + korekty | B2.6, P1.10 | todo |
| U3.2 | S11 Wynik, profil jazdy, akceptacje | U3.1 | todo |
| U3.3 | S12 Wirtualne dopasowanie | U3.2 | todo |
| U3.4 | S13 Zapis zmiany, oceny jazd, przypomnienia, Komfort | U3.3 | todo |
| U3.5 | S14 Ocena zmiany, S01 pulpit | U3.4 | todo |
| Q4.1 | E2E web i dostępność | U3.5 | todo |
| Q4.2 | Wdrożenie na hosting | Q4.1 | todo |
| Q4.3 | Szablon markerów DIY i treści instrukcji | B2.5 | todo |
| V5.1 | Walidacja: powtarzalność | Q4.2 | todo |
| V5.2 | Walidacja: zgodność z fitterem, kalibracja modelu dopasowania | V5.1 | todo |

---

## F0.1 · Szkielet repozytorium i narzędzi
- **Czytaj:** `docs/02` rozdz. 3 i 8, `docs/16` rozdz. 1 i 5.
- **Zakres:** struktura katalogów; `composer.json` (PSR-4, phpunit, phpstan, opis/json-schema); `analyzer/` z
  `pyproject`/`requirements.txt` (numpy, scipy, opencv-python-headless – wersje po F0.2 przypięte), `pytest`;
  `config/config.example.php`; `.gitignore` (config.php, storage, venv, vendor); skrypt `bin/check.sh`
  (lint + testy obu języków); `public/fit/index.php` zwracający stronę „moduł w budowie” tylko dla zalogowanych.
- **Kryteria:** `bin/check.sh` przechodzi na czystym klonie; brak sekretów w repo; instrukcja uruchomienia
  lokalnego (Windows: WSL2/Docker) w README.

## F0.2 · Weryfikacja hostingu
- **Czytaj:** `docs/07` rozdz. 13 (E2), `docs/01` rozdz. 6.
- **Zakres:** wykonanie listy E2 przez SSH; skrypt `analyzer/tools/host_check.py`; sprawdzenie limitów PHP
  (`upload_max_filesize`, `post_max_size`, `max_execution_time` CLI), wersji MariaDB, crona, `proc_open`.
- **Kryteria:** raport w `docs/18` (D-01, D-02, D-03) z faktycznymi wersjami i czasami; decyzja: analiza na hostingu
  albo osobne środowisko. **Nie deklarować zgodności bez uruchomienia.**

## F0.3 · Kontrakty JSON Schema
- **Czytaj:** `docs/11`, `docs/09` rozdz. 10.
- **Zakres:** `schemas/job.schema.json`, `result.schema.json`, `fit-model.schema.json`, przykłady w
  `schemas/examples/`; walidator w Pythonie (`jsonschema`) i PHP (`opis/json-schema`); testy kontraktu.
- **Kryteria:** przykłady przechodzą w obu językach; przykład z błędnym typem odrzucany w obu.

## P1.1 · Geometria i pomiary
- **Czytaj:** `docs/08`.
- **Zakres:** `ridemore_fit.metrics`: funkcje bazowe, konwersje układów, wszystkie pomiary z rozdz. 3 na seriach,
  statystyki cykli (rozdz. 4).
- **Kryteria:** wszystkie wektory testowe z rozdz. 6; testy wartości `null`.

## P1.2 · Generator syntetycznych nagrań
- **Czytaj:** `docs/16` rozdz. 2, `docs/10` rozdz. 3–4.
- **Zakres:** `analyzer/tests/synth/render.py` + plik prawdy (kąty na klatkę).
- **Kryteria:** wygenerowane MP4 (H.264) 20 s odtwarzalne przez OpenCV; prawda zgodna z `ridemore_fit.metrics`.

## P1.3 · Odczyt i walidacja wideo, wybór fragmentu
- **Czytaj:** `docs/06` rozdz. 5, `docs/07` rozdz. 3.
- **Zakres:** `vision.io`: metadane, rotacja, skalowanie, pomijanie klatek, kody `E_VIDEO_*`, `W_*`; `select`.
- **Kryteria:** testy na nagraniach syntetycznych i plikach uszkodzonych; strumieniowanie bez trzymania klatek.

## P1.4 · Markery i znacznik skali
- **Czytaj:** `docs/06` rozdz. 3–4, `docs/07` rozdz. 4.
- **Kryteria:** na syntetycznych nagraniach: wykrycie ≥ 98% widocznych markerów, błąd środka ≤ 2 px; skala ±1%.

## P1.5 · Estymatory pozy i porównanie modeli
- **Czytaj:** `docs/07` rozdz. 5 i 13 (E3).
- **Zakres:** interfejs `PoseEstimator`, ≥ 2 implementacje dostępne na hostingu (wg F0.2), mapowanie punktów,
  ROI; raport porównania na ≥ 10 nagraniach z markerami.
- **Kryteria:** raport w `docs/18` (ADR wyboru `pose.model`) z tabelą kryteriów E3; licencje potwierdzone.

## P1.6 · Fuzja, śledzenie, wygładzanie
- **Czytaj:** `docs/07` rozdz. 6–7.
- **Kryteria:** testy tabelaryczne fuzji; filtr bez przesunięcia fazy; zasłonięcie markera stopy przez 0,08 s →
  interpolacja; 0,3 s → `missing`.

## P1.7 · Kalibracja, faza korby, cykle
- **Czytaj:** `docs/06` rozdz. 4, `docs/07` rozdz. 7–8, `docs/08` rozdz. 1.
- **Kryteria:** syntetyczne: środek suportu ±3 mm, faza ±5°, liczba cykli zgodna, odrzucenia z poprawnym kodem.

## P1.8 · Statystyki, jakość, kadry kluczowe, `analyze` E2E
- **Czytaj:** `docs/07` rozdz. 9–12, `docs/11` rozdz. 2.
- **Kryteria:** `python -m ridemore_fit analyze` na syntetycznym nagraniu daje `result.json` zgodny ze schematem;
  błędy pomiarów w granicach `docs/16` rozdz. 2; `progress.json`; kody wyjścia.

## P1.9 · Model dopasowania v1
- **Czytaj:** `docs/09`, `docs/04` rozdz. 4–5.
- **Zakres:** `configs/fit-models/fit-model-v1.json` (wszystkie zakresy, modyfikatory, teksty PL, źródła),
  interpreter reguł, ocena zmiany, porady informacyjne.
- **Kryteria:** testy tabelaryczne z `docs/16` rozdz. 2; brak progów w kodzie (tylko w pliku).

## P1.10 · Kinematyka, warianty, `correct` / `refit` / `simulate`
- **Czytaj:** `docs/10`, `docs/11` rozdz. 1.
- **Kryteria:** testy kierunków z `docs/16`; siatka ≤ 38 wariantów < 5 s; korekta `all_frames` zmienia pomiary,
  `keyframe` nie zmienia statystyk; szkielety w px kadrów.

## B2.1 · Rdzeń PHP
- **Czytaj:** `docs/13` rozdz. 1, 3, 10, 11; `docs/02` rozdz. 6; `docs/15` rozdz. 2–3; `docs/14` rozdz. 2 i 4.
- **Zakres:** front controller, router, Request/Response, middleware (ErrorHandler, SecurityHeaders, Auth –
  `SharedSessionIdentityProvider`, Csrf, RateLimit), layout + nawigacja, `fit.css` z tokenami, i18n, logger.
- **Kryteria:** testy auth (brak sesji, sesja ważna), CSRF, nagłówki; strona pusta S01 w układzie makiety.

## B2.2 · Migracje bazy
- **Czytaj:** `docs/12`.
- **Kryteria:** `php bin/fit-migrate.php` tworzy wszystkie tabele na czystej bazie; ponowne uruchomienie nic nie
  zmienia; test zgodności kolumn z dokumentem.

## B2.3 · Profil, zgody, rowery, ustawienia, historia
- **Czytaj:** `docs/04` rozdz. 3, `docs/05`, `docs/15` rozdz. 4, `docs/03` rozdz. 17.
- **Kryteria:** API z `docs/13`; wersjonowanie ustawień; IDOR → 404; ekrany Rowery i Profil.

## B2.4 · Wywiad
- **Czytaj:** `docs/04`, `docs/03` rozdz. 4–8, makiety 02–06.
- **Zakres:** szkic i autozapis, pełny/krótki, klasyfikacja mobilności z `config/questionnaire-q-1.json`,
  podgląd modyfikatorów w S06 (z pliku modelu dopasowania – odczyt tekstów i warunków, bez wyliczania zakresów).
- **Kryteria:** walidacja zakresów; zgoda `health_data` przed S04/S05; zgodność ekranów z makietami.

## B2.5 · Markery, nagranie, przesyłanie
- **Czytaj:** `docs/06`, `docs/03` rozdz. 9–10, `docs/13` rozdz. 6, makiety 07–08.
- **Kryteria:** przesyłanie 500 MB fragmentami z wznowieniem; walidacja sygnatury; zgody; zapis opcji sesji.

## B2.6 · Kolejka, worker, importer, retencja
- **Czytaj:** `docs/02` rozdz. 4–5, `docs/13` rozdz. 7–8, `docs/12` rozdz. 4, `docs/11` rozdz. 3, makieta 09.
- **Kryteria:** testy kolejki z `docs/16` rozdz. 3; worker z zastępczym skryptem i z prawdziwym `analyze`;
  S09 z odpytywaniem i błędami.

## U3.1 · S10 Sprawdź punkty
- **Czytaj:** `docs/03` rozdz. 12, `docs/14` rozdz. 3, `docs/10` rozdz. 9, makieta 10.
- **Kryteria:** korekta → zadanie `correct` → nowa wersja; przywrócenie; obsługa klawiatury; brak obliczeń w JS.

## U3.2 · S11 Wynik
- **Czytaj:** `docs/03` rozdz. 13, `docs/09` rozdz. 3–8, makieta 11.
- **Kryteria:** statusy i paski z API; zmiana profilu → `refit`; akceptacja i cofnięcie; zakładka szczegółów
  z wykresem w cyklu (SVG) i definicjami.

## U3.3 · S12 Wirtualne dopasowanie
- **Czytaj:** `docs/03` rozdz. 14, `docs/10`, `docs/13` rozdz. 9, makieta 12.
- **Kryteria:** wybór wariantu; szacunek kombinacji; `simulate`; zapis planu; parametry niemodelowane jako informacja.

## U3.4 · S13, oceny jazd, przypomnienia, Komfort
- **Czytaj:** `docs/03` rozdz. 15 i 17, `docs/05` rozdz. 4, makieta 13.
- **Kryteria:** nowa wersja ustawienia + zmiany; ostrzeżenie przy > 1 zmianie; przypomnienie na pulpicie.

## U3.5 · S14 i pulpit
- **Czytaj:** `docs/03` rozdz. 3 i 16, `docs/09` rozdz. 9, makiety 01 i 14.
- **Kryteria:** werdykt z wyniku Pythona; pulpit bez kątów; stany puste.

## Q4.1 · E2E web i dostępność
- **Czytaj:** `docs/16` rozdz. 4.
- **Kryteria:** 8 scenariuszy przechodzi; axe bez naruszeń krytycznych; 1440 i 390 px.

## Q4.2 · Wdrożenie na hosting
- **Czytaj:** `docs/02` rozdz. 7–8, `docs/13` rozdz. 8, `docs/15` rozdz. 5–6.
- **Zakres:** instrukcja krok po kroku dla osoby nietechnicznej (`docs/wdrozenie.md`): katalogi, uprawnienia,
  venv, konfiguracja, migracje, cron, podpięcie `/fit` w Apache, test dymny.
- **Kryteria:** wdrożenie na kopii testowej wykonane według instrukcji bez pomocy.

## Q4.3 · Szablon markerów DIY i treści
- **Czytaj:** `docs/06` rozdz. 3.
- **Kryteria:** PDF A4 z 8 markerami i znacznikiem skali 200,0 mm (sprawdzony po wydruku linijką); teksty
  instrukcji w i18n.

## V5.1 / V5.2 · Walidacja
- **Czytaj:** `docs/16` rozdz. 6, `docs/09` (D-06).
- **Kryteria:** raporty z wynikami względem kryteriów z `docs/01` rozdz. 7; decyzje o zmianie zakresów jako nowa
  wersja modelu dopasowania.
