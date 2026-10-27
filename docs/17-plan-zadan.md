# 17 · Plan zadań

Każde zadanie jest kontraktem implementacyjnym (`CLAUDE.md`, rozdz. 1 i 8). Status: `todo` · `in_progress` ·
`done` · `blocked`. Zmiany panelu ekspertów (`docs/20`, C-01…C-23) i recenzji technicznej (ADR-21…ADR-25) są wpisane
w dokumenty i zakresy zadań; zadania interfejsu czytają też `docs/21-komunikacja.md`.

## Etapy i bramki

Realizacja w 6 etapach. **Etap zaczyna się dopiero po spełnieniu bramki poprzedniego.** Najpierw wykazujemy, że
pomiar działa i jest powtarzalny; dopiero potem budujemy wokół niego zalecenia i pełny proces użytkownika.

| Etap | Cel | Bramka (kryterium ukończenia) |
|---|---|---|
| **M0** Wykonalność techniczna | środowisko, zależności, uruchamianie Pythona, kontrakty | raport F0.2 z **faktycznie uruchomionym** `selfcheck` na hostingu: wersja Pythona, OpenCV, model pozy, czas analizy 60 s nagrania ≤ 15 min, pamięć, ffmpeg/ffprobe, `proc_open`, limity crona i procesów CLI. Jeśli hosting nie spełnia – decyzja D-02 (osobne środowisko analizy i sposób komunikacji) **przed** M1 |
| **M1** Minimalny silnik pomiarowy | wideo → markery → kalibracja → śledzenie → pomiary → jakość; bez interfejsu i zaleceń | V1.1: znane błędy każdego pomiaru (z markerami i bez) na rzeczywistych nagraniach; `knee_flex_bottom` z markerami spełnia kryteria `docs/16` rozdz. 6.1; nieudane analizy domowe ≤ 20% |
| **M2** Prototyp: weryfikacja i raport pomiarowy | przesłanie → kolejka → S09 → S10 (korekty) → raport pomiarowy z przyczynami odrzuceń; **bez zaleceń** | pełny przepływ na środowisku testowym; testy kolejki i prywatności z `docs/16` rozdz. 3; 5 osób rozumie przyczyny odrzucenia pomiaru |
| **M3** Profile, rowery, wywiad, historia | integracja z kontem ridemore.bike, wersjonowanie ustawień, wywiad, retencja | Q-01 i D-04 rozstrzygnięte; testy autoryzacji, wersji ustawień, retencji przechodzą |
| **M4** Ograniczony model dopasowania | jedna regulacja `experimental` (wysokość siodła), pozostałe „tylko pomiar”; pewność, ograniczony krok, odmowa zalecenia | V4.1; karta modelu `fit-model-v1` zatwierdzona przez radę modelu (D-06); decyzja o statusie reguły |
| **M5** Symulacja, plan i ocena efektów | warianty, plan sekwencyjny, S12–S14, przypomnienia, kolejne reguły po walidacji | V5.2; 12 scenariuszy E2E; wdrożenie produkcyjne |

Wydanie 1 (ograniczona grupa użytkowników) = M0–M4. Wydanie 2 = M5. W M1–M3 wynik `analyze` nie zawiera sekcji
`fit` ani `variants` (opcjonalne w schemacie, `docs/11`); w M2 sesja może nie mieć wywiadu (`interview_id` NULL).

## Przegląd

| ID | Zadanie | Etap | Zależy od | Status |
|---|---|---|---|---|
| F0.1 | Szkielet repozytorium i narzędzi | M0 | – | todo |
| F0.2 | Weryfikacja hostingu (bramka M0) | M0 | F0.1 | todo |
| F0.3 | Kontrakty JSON Schema i przykłady | M0 | F0.1 | todo |
| P1.1 | Geometria i pomiary | M1 | F0.1 | todo |
| P1.2 | Generator syntetycznych nagrań | M1 | P1.1 | todo |
| P1.3 | Odczyt i walidacja wideo, wybór fragmentu | M1 | F0.2 | todo |
| P1.4 | Markery i znacznik skali | M1 | P1.2, P1.3 | todo |
| P1.5 | Estymatory pozy i porównanie modeli | M1 | P1.3, F0.2 | todo |
| P1.6 | Fuzja, śledzenie, wygładzanie | M1 | P1.4, P1.5 | todo |
| P1.7 | Kalibracja, faza korby, cykle | M1 | P1.6 | todo |
| P1.8 | Statystyki, jakość, kadry kluczowe, `analyze` E2E | M1 | P1.7, F0.3 | todo |
| Q4.3 | Szablon markerów DIY i treści instrukcji | M1 | P1.4 | todo |
| V1.1 | Walidacja pomiaru na rzeczywistych nagraniach (bramka M1) | M1 | P1.8, Q4.3 | todo |
| B2.1 | Rdzeń PHP: front controller, router, middleware, układ | M2 | F0.1 | todo |
| B2.2 | Migracje bazy (M2: sesje, pliki, kolejka, analizy; M3: pozostałe) | M2/M3 | B2.1 | todo |
| B2.5 | Markery, nagranie, przesyłanie (S07–S08, M01) | M2 | B2.2 | todo |
| B2.6 | Kolejka, worker, importer, retencja (S09) | M2 | B2.5, F0.3 | todo |
| P1.10 | `correct` – przeliczenie po korekcie punktów | M2 | P1.8 | todo |
| U3.1 | S10 Sprawdź punkty + korekty | M2 | B2.6, P1.10 | todo |
| U2.1 | Raport pomiarowy (S11 bez zaleceń) | M2 | U3.1 | todo |
| Q4.2 | Wdrożenie na hosting (M2: środowisko testowe; M5: produkcja) | M2/M5 | U2.1 | todo |
| B2.3 | Profil, zgody, rowery, ustawienia, historia | M3 | B2.2 | todo |
| B2.4 | Wywiad (S02–S06) | M3 | B2.3 | todo |
| P1.9 | Model dopasowania v1 (karty reguł, `refit`, karta modelu) | M4 | P1.8, F0.3, V1.1 | todo |
| U3.2 | S11 Wynik, profil jazdy, akceptacje | M4 | U2.1, P1.9, B2.4 | todo |
| V4.1 | Walidacja reguły wysokości siodła (bramka M4) | M4 | U3.2 | todo |
| P1.11 | Kinematyka, warianty, `simulate`, plan sekwencyjny | M5 | P1.9, V4.1 | todo |
| U3.3 | S12 Wirtualne dopasowanie | M5 | P1.11, U3.2 | todo |
| U3.4 | S13 Zapis zmiany, oceny jazd, przypomnienia, Komfort | M5 | U3.3 | todo |
| U3.5 | S14 Ocena zmiany, S01 pulpit | M5 | U3.4 | todo |
| B2.7 | Preferencje, przypomnienia, zdarzenia produktowe | M5 | B2.2 | todo |
| Q4.1 | E2E web i dostępność | M2–M5 (przyrostowo) | U3.5 | todo |
| V5.2 | Walidacja pozostałych reguł i modelu wariantów (bramka M5) | M5 | P1.11 | todo |

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
- **Zakres:** wykonanie listy E2 przez SSH; skrypt `analyzer/tools/host_check.py`; sprawdzenie: dostępnych
  interpreterów Pythona (deklaracja dostawcy: zalecany 3.6.15, maksymalny 3.9.23 – do potwierdzenia), instalacji
  kół `numpy`/`opencv-python-headless`/runtime modelu pozy dla architektury serwera, `selfcheck` na nagraniu
  wzorcowym (czas, pamięć szczytowa), ffmpeg/ffprobe, limitów PHP (`upload_max_filesize`, `post_max_size`,
  `max_execution_time` CLI), wersji MariaDB, crona (minimalny odstęp, limit czasu procesu), `proc_open`, procesów
  w tle i ich zabijania przez dostawcę.
- **Kryteria (bramka M0):** raport w `docs/18` (D-01, D-02, D-03) z faktycznymi wersjami i czasami. Jeśli
  którykolwiek warunek nie jest spełniony – decyzja D-02 (osobne środowisko analizy i sposób komunikacji z PHP)
  przed rozpoczęciem M1. **Nie deklarować zgodności bez uruchomienia.**

## F0.3 · Kontrakty JSON Schema
- **Czytaj:** `docs/11`, `docs/09` rozdz. 10.
- **Zakres:** `schemas/job.schema.json`, `result.schema.json`, `fit-model.schema.json`, przykłady w
  `schemas/examples/`; walidator w Pythonie (`jsonschema`) i PHP (`opis/json-schema`); testy kontraktu.
- **Kryteria:** przykłady przechodzą w obu językach; przykład z błędnym typem odrzucany w obu.

## P1.1 · Geometria i pomiary
- **Czytaj:** `docs/08`.
- **Zakres:** `ridemore_fit.metrics`: funkcje bazowe, konwersje układów, wszystkie pomiary z rozdz. 3 na seriach,
  statystyki cykli (rozdz. 4).
- **Kryteria:** wszystkie wektory testowe z rozdz. 6 (w tym `knee_flex_top`); testy wartości `null`.

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

## P1.9 · Model dopasowania v1 (etap M4)
- **Czytaj:** `docs/09` (w tym rozdz. 1.1, 5.3, 12, 13), `docs/04` rozdz. 4–5 i 8, `docs/21`.
- **Zakres:** `configs/fit-models/fit-model-v1.json` (wszystkie zakresy z `method` i strefami spornymi, karty reguł,
  `rule_status` – `saddle_height` `experimental`, pozostałe `measure_only`; modyfikatory, sekcje `safety`,
  `confidence`, `signals`, `effort`, teksty PL zgodne z `docs/21`, źródła), interpreter reguł z kolejnością warstw,
  kopertą literatury i pustym zakresem, ślad uzasadnienia, poziom pewności, skutki `safety_level` i `fit_level`,
  porady, `refit`; ocena zmiany (używana od M5); karta modelu `docs/model-cards/fit-model-v1.md`.
- **Kryteria:** testy tabelaryczne z `docs/16` rozdz. 2; reguła nie wydaje zalecenia, gdy pomiar ma
  `validation_status ≠ validated` lub nie spełnia karty reguły; brak progów w kodzie (tylko w pliku); teksty bez
  przypisywania przyczyny dolegliwości.

## P1.10 · `correct` – przeliczenie po korekcie punktów (etap M2)
- **Czytaj:** `docs/11` rozdz. 1, `docs/07` rozdz. 6–9.
- **Kryteria:** korekta `all_frames` przelicza wszystkie zależne pomiary i jakość, `keyframe` zmienia tylko kadr;
  nowa analiza z `parent_id`; testy z `docs/16` rozdz. 3 (korekty).

## P1.11 · Kinematyka, warianty, `simulate`, plan sekwencyjny (etap M5)
- **Czytaj:** `docs/10`, `docs/09` rozdz. 11, `docs/11` rozdz. 1–2.
- **Kryteria:** testy kierunków z `docs/16`; siatka ≤ 38 wariantów < 5 s; wyniki rozdzielone na `geometry`
  i `prediction`; macierz wpływu; plan sekwencyjny odtwarza przykład z `docs/09` rozdz. 11 (pominięty mostek);
  szkielety w px kadrów.

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
- **Czytaj:** `docs/04`, `docs/03` rozdz. 4–8, `docs/21`, makiety 02–06 (z różnicami z `docs/makiety/README.md`).
- **Zakres:** szkic i autozapis, pełny/krótki/szybki start (S02q), testy podstawowe i dodatkowe z metadanymi
  (próby, „sam / z pomocą”, ból), klasyfikacja mobilności i poziomu bezpieczeństwa (`MobilityClassifier`,
  `SafetyClassifier`) z `config/questionnaire-q-1.json`, pytania bezpieczeństwa, szczegóły dolegliwości i główna
  dolegliwość, historia operacji (`fit_surgery`), podgląd modyfikatorów i komunikatu bezpieczeństwa w S06 (z pliku
  modelu dopasowania – odczyt tekstów i warunków, bez wyliczania zakresów), uzupełnienie wywiadu po szybkim starcie
  → `refit`, szacowany czas przy krokach.
- **Kryteria:** walidacja zakresów; zgoda `health_data` przed S04/S05 i pytaniami bezpieczeństwa (odmowa →
  `basic` + `caution`); testy klasyfikatorów z `docs/16` rozdz. 3; zgodność ekranów z makietami i `docs/03`.

## B2.5 · Markery, nagranie, przesyłanie
- **Czytaj:** `docs/06` (w tym rozdz. 7), `docs/03` rozdz. 9–10.1, `docs/13` rozdz. 6, makiety 07–08.
- **Zakres:** S07, S08 (RPE, moc, domyślna strona z dolegliwości, neutralny opis polityk nagrania, karta QR),
  M01 (przesłanie z telefonu po zalogowaniu), `GET /sessions/{id}/upload-status`.
- **Kryteria:** przesyłanie 500 MB fragmentami z wznowieniem; walidacja sygnatury; zgody; zapis opcji sesji;
  M01 działa na 390 px; adres w kodzie QR bez tokenu i z kontrolą właściciela.

## B2.6 · Kolejka, worker, importer, retencja
- **Czytaj:** `docs/02` rozdz. 4–5, `docs/13` rozdz. 7–8, `docs/12` rozdz. 4, `docs/11` rozdz. 3, makieta 09.
- **Kryteria:** testy kolejki z `docs/16` rozdz. 3; worker z zastępczym skryptem i z prawdziwym `analyze`;
  S09 z odpytywaniem i błędami.

## B2.7 · Preferencje, przypomnienia, zdarzenia produktowe
- **Czytaj:** `docs/12` (`fit_user_pref`, `fit_reminder`, `fit_event`), `docs/13` rozdz. 2 i 5, `docs/15` rozdz. 1,
  `docs/01` rozdz. 7, `docs/20` (C-14, C-17).
- **Zakres:** `PreferenceService`, `GET/PUT /preferences`, limit przypomnień, kanał e-mail za flagą
  `notifications.email_available`, `EventLogger` z białą listą zdarzeń i kluczy, retencja `fit_event`, prosty
  raport mierników (CLI `bin/fit-kpi.php` – agregaty bez `user_id`).
- **Kryteria:** testy z `docs/16` rozdz. 3 (analityka, przypomnienia); brak danych o zdrowiu w `fit_event`.

## U3.1 · S10 Sprawdź punkty
- **Czytaj:** `docs/03` rozdz. 12, `docs/14` rozdz. 3, `docs/10` rozdz. 9, makieta 10.
- **Kryteria:** korekta → zadanie `correct` → nowa wersja; przywrócenie; obsługa klawiatury i korekta bez
  przeciągania (kliknij punkt → kliknij miejsce); brak obliczeń w JS.

## U2.1 · Raport pomiarowy (S11 bez zaleceń, etap M2)
- **Czytaj:** `docs/03` rozdz. 13 (zakładka szczegółów), `docs/07` rozdz. 9–9.1, `docs/08`, `docs/21`.
- **Zakres:** S11 w trybie „raport pomiarowy”: wartości pomiarów z jakością i statusem walidacji, przyczyny
  odrzucenia lub „nie oceniono” prostym językiem, „Nagraj ponownie”, kadry; brak zakresów-zaleceń i akcji.
- **Kryteria:** każda przyczyna z `docs/07` rozdz. 9.1 ma tekst; pomiary `unvalidated` z plakietką, `hidden`
  niewidoczne; test zrozumiałości przyczyn odrzucenia (5 osób).

## U3.2 · S11 Wynik
- **Czytaj:** `docs/03` rozdz. 13, `docs/09` rozdz. 1.1 i 3–8, `docs/21`, makieta 11.
- **Kryteria:** statusy i paski z API (strefy bez czerwieni, `docs/21` rozdz. 6); poziom pewności z powodami
  i wysiłek zmiany na karcie następnego kroku; stany `caution`, `stop` i `basic`; zmiana profilu → `refit`;
  akceptacja i cofnięcie; zakładka szczegółów z wykresem w cyklu (SVG), definicjami i śladem uzasadnienia;
  regulacje `measure_only` z dopiskiem „Tylko pomiar – bez zalecenia”, zalecenie `experimental` z plakietką.

## U3.3 · S12 Wirtualne dopasowanie
- **Czytaj:** `docs/03` rozdz. 14, `docs/10`, `docs/13` rozdz. 9, makieta 12.
- **Kryteria:** wybór wariantu; szacunek kombinacji; `simulate`; zapis planu; parametry niemodelowane jako
  informacja; przy `stop` tylko podgląd.

## U3.4 · S13, oceny jazd, przypomnienia, Komfort
- **Czytaj:** `docs/03` rozdz. 3 i 15 i 17, `docs/05` rozdz. 4, `docs/21` rozdz. 8, makieta 13.
- **Kryteria:** okno „Zaplanuj” (data, jazdy testowe) → `fit_plan` + przypomnienie; nowa wersja ustawienia +
  zmiany; ostrzeżenie przy > 1 zmianie; ocena po jeździe 5-stopniowa z opcjonalną siłą dolegliwości;
  przypomnienia zgodne z preferencjami (maks. 2 na zmianę).

## U3.5 · S14 i pulpit
- **Czytaj:** `docs/03` rozdz. 3 i 16, `docs/09` rozdz. 9, makiety 01 i 14.
- **Kryteria:** werdykt z wyniku Pythona (etykiety z `docs/21`); pulpit bez kątów; ocena jazdy jednym
  dotknięciem i trend komfortu na S01; karta „Dokończ analizę”; stany puste.

## Q4.1 · E2E web i dostępność
- **Czytaj:** `docs/16` rozdz. 4.
- **Kryteria:** 12 scenariuszy przechodzi; axe (reguły WCAG 2.2 AA) bez naruszeń krytycznych; 1440 i 390 px.

## Q4.2 · Wdrożenie na hosting
- **Czytaj:** `docs/02` rozdz. 7–8, `docs/13` rozdz. 8, `docs/15` rozdz. 5–6.
- **Zakres:** instrukcja krok po kroku dla osoby nietechnicznej (`docs/wdrozenie.md`): katalogi, uprawnienia,
  venv, konfiguracja, migracje, cron, podpięcie `/fit` w Apache, test dymny.
- **Kryteria:** wdrożenie na kopii testowej wykonane według instrukcji bez pomocy (M2); produkcja po bramce M5
  (wydanie 1 dla ograniczonej grupy – po M4).

## Q4.3 · Szablon markerów DIY i treści
- **Czytaj:** `docs/06` rozdz. 3.
- **Kryteria:** PDF A4 z 8 markerami i znacznikiem skali 200,0 mm (sprawdzony po wydruku linijką); teksty
  instrukcji w i18n.

## V1.1 · Walidacja pomiaru (bramka M1)
- **Czytaj:** `docs/16` rozdz. 6.1–6.2, `docs/18` (D-17).
- **Kryteria:** raport dokładności każdego pomiaru (bias, LoA, MDC) z markerami i bez, na rzeczywistych
  nagraniach; `metrics.validation_status` w `pipeline.json`; korekta 2D kolana (C-21); nagrania domowe.

## V4.1 · Walidacja reguły wysokości siodła (bramka M4)
- **Czytaj:** `docs/16` rozdz. 6.2, `docs/09` rozdz. 1.1 i 13, `docs/18` (D-06).
- **Kryteria:** wyniki względem karty reguły; karta modelu uzupełniona; decyzja rady modelu: `validated` albo
  pozostaje `experimental` (nowa wersja modelu, jeśli zmiana).

## V5.2 · Walidacja pozostałych reguł i modelu wariantów (bramka M5)
- **Czytaj:** `docs/16` rozdz. 6, `docs/09` (D-06), `docs/18` (D-07, D-15).
- **Kryteria:** raporty względem kart reguł i kryteriów z `docs/01` rozdz. 7 (także testy mobilności w domu,
  czas wywiadu, zrozumiałość poziomów pewności); decyzje o statusach reguł, zakresach i progach jako nowa wersja
  modelu dopasowania lub kwestionariusza.
