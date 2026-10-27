# 07 · Analiza obrazu (moduł Python, warstwa D)

Pakiet `analyzer/ridemore_fit`, Python 3.9. Nie łączy się z bazą ani siecią; czyta `job.json` i pliki, zapisuje
`result.json`, kadry JPEG i `progress.json`. Kontrakty: `docs/11-kontrakt-danych.md`.

## 1. Interfejs wiersza poleceń

```
python -m ridemore_fit analyze  --storage-root <root> --job <job.json> --out <result.json> [--progress <progress.json>]
python -m ridemore_fit correct  --job <job.json> --out <result.json>     # ponowne przeliczenie po korekcie punktów
python -m ridemore_fit refit    --job <job.json> --out <result.json>     # inny profil / akceptacje, bez wideo
python -m ridemore_fit simulate --job <job.json> --out <result.json>     # kombinacja zmian roweru
python -m ridemore_fit selfcheck                                          # wersje, biblioteki, test na nagraniu wzorcowym
```
(`--storage-root` jest wymagany dla wszystkich typów zadań; pominięty w skrótach powyżej.)

**Ograniczenie ścieżek (bezpieczeństwo):** `--storage-root` przekazuje worker ze stałej konfiguracji
(`storage_root`), nie z `job.json`. Przed otwarciem jakiegokolwiek pliku Python sprawdza, że każda ścieżka
z `job.json` i z argumentów po `realpath` leży wewnątrz `storage_root`, a ścieżki wyjściowe (`out_dir`,
`keyframes_dir`, `--out`, `--progress`) – wewnątrz katalogu tego zadania; dowiązania symboliczne poza katalog
są odrzucane. Naruszenie → kod 2, `E_JOB_INVALID`, żaden plik nie jest odczytany. Nieznana wersja główna
`schema_version` → kod 2, `E_SCHEMA_VERSION`. Wynik zapisywany atomowo (`.tmp` → `rename`, `docs/02`, rozdz. 5.1).

| Kod wyjścia | Znaczenie | Reakcja workera |
|---|---|---|
| 0 | sukces (także z ostrzeżeniami) | zapis wyniku |
| 2 | błąd danych (np. `E_VIDEO_FPS_LOW`) – w `result.json` jest `error` | `failed`, bez ponawiania |
| 3 | błąd przejściowy (brak pamięci, przerwanie) | ponowienie wg `docs/02`, rozdz. 5 |
| 1 / inny | błąd wewnętrzny | ponowienie; po 3 próbach `failed` z `E_INTERNAL` |

Python co ≤ 5 s nadpisuje `progress.json`: `{"stage": "tracking", "progress": 0.46, "ts": "..."}`.
Worker przenosi to do `fit_job.progress` i `heartbeat_at`.

## 2. Etapy przetwarzania

| # | Etap techniczny (`stage`) | Udział w postępie | Opis dla użytkownika (S09) |
|---|---|---|---|
| 1 | `probe` – odczyt metadanych, walidacja | 0–3% | Sprawdzamy plik i jakość obrazu |
| 2 | `select` – wybór fragmentu o stałej kadencji | 3–8% | jw. |
| 3 | `detect` – markery + model pozy na każdej klatce | 8–60% | Znajdujemy markery i punkty ciała |
| 4 | `calibrate` – skala, poziom | 60–63% | jw. |
| 5 | `track` – fuzja, śledzenie, uzupełnianie luk, wygładzanie | 63–72% | Śledzimy ruch w każdym obrocie korby |
| 6 | `cycles` – faza korby, cykle, normalizacja 0–100% | 72–78% | jw. |
| 7 | `metrics` – pomiary i statystyki (`docs/08`) | 78–82% | Sprawdzamy, czy pomiar jest wiarygodny |
| 8 | `quality` – ocena jakości, ostrzeżenia | 82–85% | jw. |
| 9 | `keyframes` – wybór i zapis 4 kadrów | 85–90% | jw. |
| 10 | `fit` – model sylwetki, zalecenia, warianty (`docs/09`, `docs/10`) | 90–99% | Dopasowujemy zalecenia do Twojego wywiadu |
| 11 | `write` – zapis wyniku | 99–100% | – |

## 3. Odczyt wideo

- `cv2.VideoCapture` (FFmpeg backend). Rotacja z metadanych kontenera; obraz obracany do orientacji wyświetlania.
- Klatki przetwarzane strumieniowo – w pamięci tylko bieżąca klatka i wyniki punktów (nie cała sekwencja).
- Skalowanie: jeżeli krótszy bok > `processing.max_short_side` (1080), klatka zmniejszana; współrzędne wynikowe
  zawsze w pikselach **oryginalnej** rozdzielczości po rotacji.
- fps > `processing.max_fps` (60) → pomijanie klatek do ≤ 60 fps.
- Fragment: najdłuższe okno o długości `select.window_s` (domyślnie 60 s, min. 30 s) z najmniejszą zmiennością
  kadencji (kadencja szacowana w szybkim przebiegu co 4. klatki na podstawie toru największego jasnego obiektu
  w dolnej części kadru lub punktu `mtp5` z modelu). Odrzucane są pierwsze `select.skip_start_s` (3 s).

## 4. Wykrywanie markerów

1. Konwersja do HSV, progowanie zakresami `markers.color_hsv_ranges` (domyślnie żółty fluorescencyjny).
2. Operacje morfologiczne (otwarcie, zamknięcie, jądro 3×3).
3. Kontury → kandydaci: pole w zakresie `markers.area_px_range` (przeliczane ze skali, gdy znana:
   średnica 20–30 mm), okrągłość `4πA/P² ≥ markers.min_circularity` (0,7), obwódka ciemna (opcjonalnie).
4. Środek kandydata = środek ciężkości maski; wynik `score` 0–1 z okrągłości, kontrastu i dopasowania rozmiaru.
5. Przypisanie kandydatów do punktów (`hip` … `wrist`):
   - pierwsza klatka / po zgubieniu: przypisanie do najbliższego punktu modelu pozy (algorytm węgierski, bramka
     `markers.assign_gate_mm` = 80 mm),
   - kolejne klatki: predykcja (stała prędkość lub filtr Kalmana) + bramka `markers.track_gate_mm` = 40 mm
     (przy 60 fps; skalowana ∝ 1/fps).
6. Brak kandydata w bramce → marker „niewidoczny” w tej klatce (przyczyna: zasłonięcie, rozmycie).

Znacznik skali: dwie ciemne kropki na jasnym pasku w obszarze rury podsiodłowej; szukane w pierwszych 30 s
nagrania, mediana odległości z ≥ 30 klatek; odrzucenie, gdy zmienność > 2%.

## 5. Estymacja pozy (model)

Interfejs (pakiet `ridemore_fit.vision.pose`):

```
PoseEstimator.name -> str, .version -> str
PoseEstimator.estimate(frame_bgr) -> {point_code: (x_px, y_px, confidence)} | None
```

Mapowanie punktów modelu na kody modułu jest częścią implementacji estymatora i jest **przybliżeniem**: model
wskazuje środek stawu, a marker – punkt na powierzchni kości (np. środek stawu biodrowego ≠ krętarz większy).
Dlatego punkt z modelu ma źródło `model` i jest używany: (a) do przypisania markerów, (b) gdy marker niewidoczny,
(c) w trybie bez markerów.

Kandydaci (wybór w zadaniu E3 na podstawie porównania – rozdz. 13):

| Model | Licencja | Uruchomienie | Zgodność z Pythonem 3.9 / CPU | Uwagi |
|---|---|---|---|---|
| MediaPipe Pose Landmarker (BlazePose) | Apache-2.0 | pakiet `mediapipe` | do sprawdzenia (koła dla systemu i architektury hostingu) | 33 punkty, m.in. pięta i czubek stopy; szybki na CPU |
| RTMPose (OpenMMLab MMPose) | Apache-2.0 | model wyeksportowany do ONNX, uruchamiany przez `onnxruntime` (bez PyTorch na serwerze) | do sprawdzenia (`onnxruntime` dla 3.9) | wysoka dokładność; wymaga detektora osoby lub stałego ROI |
| MoveNet (Thunder) | Apache-2.0 | TFLite (`tflite-runtime`) | do sprawdzenia | 17 punktów, brak punktów stopy |
| tylko markery (`markers_only`) | – | OpenCV | tak (OpenCV dla 3.9 do sprawdzenia) | tryb awaryjny przy braku modelu; wymaga markerów |

Wykluczone bez osobnej decyzji: OpenPose (licencja niekomercyjna), Ultralytics YOLO-pose (AGPL-3.0 / licencja
komercyjna), modele ciała SMPL (licencja niekomercyjna).

ROI: po wykryciu sylwetki w pierwszych klatkach przetwarzany jest prostokąt otaczający z marginesem 15%
(przyspieszenie i stabilność).

## 6. Fuzja źródeł punktu

Dla każdego punktu i klatki:

| Warunek | Źródło (`source`) | Pewność (`confidence`) |
|---|---|---|
| marker wykryty, `score ≥ markers.min_score` (0,6), w bramce względem modelu (gdy model dostępny) | `marker` | `0,7 + 0,3·score` |
| marker niewidoczny, model z `conf ≥ pose.min_conf` (0,5) | `model` | `0,9·conf_model` (dla punktów z markerem oczekiwanym: × 0,85) |
| luka ≤ `tracking.max_gap_s` (0,1 s) między wiarygodnymi wartościami | `interpolated` | min(pewności sąsiadów) × 0,8 |
| brak danych | `missing` | 0 |
| korekta użytkownika (kadry kluczowe) | `manual` | 1,0 (oznaczana osobno) |

Tryb `markers_mode = none`: tylko `model`/`interpolated`/`missing`.

## 7. Śledzenie, wygładzanie, punkty pochodne

- Wygładzanie współrzędnych: filtr Butterwortha dolnoprzepustowy 4. rzędu, zerofazowy (`filtfilt`),
  częstotliwość odcięcia `tracking.lowpass_hz` = 6 Hz (wartość domyślna do weryfikacji w E4). Stosowany na
  odcinkach bez luk dłuższych niż `tracking.max_gap_s`.
- Odrzucanie skoków: przesunięcie między klatkami > `tracking.max_jump_mm_per_s` (3000 mm/s) → wartość `missing`
  i ponowna interpolacja.
- `bb` (oś suportu) i promień toru: dopasowanie okręgu (metoda Kåsy, następnie Gaussa–Newtona) do toru `mtp5`
  z całego fragmentu; środek okręgu ≈ oś suportu (przesunięcie stopy względem osi pedału jest w przybliżeniu
  stałe w cyklu; błąd raportowany jako `bb_fit_rmse_mm`).
- `pedal` (oś pedału): punkt na okręgu o promieniu `crank_length_mm` wokół `bb` pod kątem fazy korby (rozdz. 8).
- Kontrola długości segmentów: mediany długości udo (`hip–knee`), podudzie (`knee–ankle`), ramię, przedramię;
  klatki, w których długość odbiega o > `quality.segment_tolerance` (8%) od mediany, obniżają pewność punktów
  tego segmentu × 0,7. Porównanie z wywiadem (np. `inseam`) daje ostrzeżenie `W_SEGMENT_MISMATCH`, gdy różnica > 12%.

## 8. Faza korby i cykle

- Faza `θ(t)` = kąt wektora `bb → mtp5` w układzie roweru (po korekcie poziomu), mierzony zgodnie z kierunkiem
  pedałowania, 0° = góra. Przesunięcie fazy między `mtp5` a osią pedału: stała kalibrowana jako różnica kąta
  w chwili najwyższego położenia toru `mtp5` (≈ GMP). Dokładność ±5° – opisana w szczegółach jakości.
- Cykl: od przejścia `θ` przez 0° do następnego przejścia. Kadencja cyklu = 60 / czas cyklu.
- Normalizacja: każda seria w cyklu jest próbkowana na 101 punktów względem **fazy korby** (0–100%), interpolacja
  liniowa.
- Odrzucenie cyklu (`reject_reason`):
  | Kod | Warunek |
  |---|---|
  | `incomplete` | pierwszy/ostatni niepełny cykl fragmentu |
  | `missing_points` | któryś punkt potrzebny do pomiarów `missing` w > 10% próbek cyklu |
  | `low_confidence` | średnia pewność punktów kluczowych < `quality.cycle_min_conf` (0,6) |
  | `jump` | wykryty skok punktu w cyklu |
  | `cadence_outlier` | czas cyklu poza medianą ±15% |
- Minimalna liczba ważnych cykli: `quality.min_valid_cycles` = 10 (poniżej → `E_CYCLES_TOO_FEW`); < 20 → ostrzeżenie `W_FEW_CYCLES`.

## 9. Ocena jakości

| Poziom | Punkt w kadrze kluczowym | Pomiar | Analiza ogółem |
|---|---|---|---|
| `good` | `confidence ≥ quality.point_good` (0,85) | wszystkie punkty pomiaru `good` w ≥ 90% próbek ważnych cykli i SD między cyklami ≤ `metric.sd_good` | ≥ 85% klatek z kompletem punktów, ≥ 20 cykli, wszystkie 4 regulacje mają pomiar `good` |
| `warn` | 0,6–0,85 | w przeciwnym razie, jeśli ≥ 60% próbek | wynik zakończony z ostrzeżeniami (`done_warnings`) |
| `bad` | < 0,6 | < 60% próbek → wartość ukryta w interfejsie | – |
| `na` | – | pomiar niemożliwy z tego nagrania (np. tor kolana z boku) | – |

Ostrzeżenia (kody): `W_SCALE_ESTIMATED`, `W_TILT_UNCERTAIN`, `W_MARKER_OCCLUDED:{point}`, `W_LOW_CONFIDENCE:{point}`,
`W_FEW_CYCLES`, `W_CADENCE_UNSTABLE`, `W_LOW_FPS`, `W_VIDEO_TRIMMED`, `W_SEGMENT_MISMATCH:{segment}`,
`W_GRIP_MISMATCH` (chwyt wykryty ≠ zadeklarowany), `W_CHECKLIST:{code}`, `W_HIGH_EFFORT` (`session.recording_rpe`
≥ `quality.high_effort_rpe`, C-10). Komunikaty dla użytkownika: `docs/11`, rozdz. 7.

### 9.1 Decyzja: przyjąć, odrzucić czy nagrać ponownie

| Sytuacja | Wynik | Komunikat |
|---|---|---|
| błąd danych `E_*` (`docs/11`, rozdz. 7) | analiza odrzucona | przyczyna + co poprawić, przycisk „Nagraj ponownie” |
| `quality.overall = bad` | wynik zapisany, regulacje `unknown`, brak zaleceń | „Pomiar niewiarygodny” + przyczyny z ostrzeżeń, „Nagraj ponownie” |
| `warn`, ale pomiar sterujący regulacji < `good` | ta regulacja `unknown` | „Nie oceniliśmy {regulacja}: {przyczyna}. Nagraj ponownie, aby ją ocenić” |
| `good` / `warn` z kompletem pomiarów | wynik przyjęty | ostrzeżenia w „Szczegółach” |

Każda przyczyna odrzucenia lub „nie oceniono” jest pokazywana użytkownikowi w raporcie pomiarowym (S11, etap M2).

## 10. Kadry kluczowe

Dla faz 0° (`tdc`), 90° (`p90`), 180° (`bdc`), 270° (`p270`):
1. Kandydaci: klatki ważnych cykli z `|θ − faza| ≤ 4°`.
2. Wybór: najwyższa średnia pewność punktów, przy remisie – klatka, której pomiary są najbliższe medianie.
3. Zapis JPEG (jakość 85, krótszy bok ≤ 1080 px) + współrzędne wszystkich punktów (px w rozdzielczości obrazu kadru
   i mm w układzie roweru), źródło, pewność.

## 11. Konfiguracja przetwarzania (`analyzer/configs/pipeline.json`)

| Klucz | Domyślnie | Opis |
|---|---|---|
| `video.min_s` / `video.max_s` | 30 / 120 | czas nagrania |
| `video.min_fps` | 25 | minimalne fps |
| `video.min_short_side_px` | 720 | minimalna rozdzielczość |
| `processing.max_short_side` | 1080 | skalowanie do detekcji |
| `processing.max_fps` | 60 | maks. fps przetwarzania |
| `select.window_s` / `select.skip_start_s` | 60 / 3 | fragment analizy |
| `markers.color_hsv_ranges` | `[[20,100,100],[40,255,255]]` | żółty fluorescencyjny (OpenCV HSV) |
| `markers.diameter_mm_range` | [18, 32] | rozmiar markera |
| `markers.min_circularity` | 0.7 | okrągłość |
| `markers.min_score` | 0.6 | minimalna jakość kandydata |
| `markers.assign_gate_mm` / `markers.track_gate_mm` | 80 / 40 | bramki przypisania i śledzenia |
| `pose.model` | `mediapipe_full` | wybrany estymator (po E3) |
| `pose.min_conf` | 0.5 | minimalna pewność modelu |
| `pose.roi_margin` | 0.15 | margines ROI |
| `tracking.max_gap_s` | 0.1 | maks. luka do interpolacji |
| `tracking.lowpass_hz` | 6.0 | odcięcie filtra |
| `tracking.max_jump_mm_per_s` | 3000 | detekcja skoków |
| `quality.point_good` / `quality.point_warn` | 0.85 / 0.6 | progi punktu |
| `quality.cycle_min_conf` | 0.6 | próg cyklu |
| `quality.min_valid_cycles` / `quality.good_valid_cycles` | 10 / 20 | liczba cykli |
| `quality.segment_tolerance` | 0.08 | zmienność długości segmentu |
| `metric.sd_good` | `{"deg": 3.0, "mm": 8.0}` | maks. SD między cyklami dla `good` |
| `keyframes.phase_tolerance_deg` | 4 | dobór kadru |
| `keyframes.jpeg_quality` / `keyframes.max_short_side` | 85 / 1080 | zapis kadru |
| `quality.high_effort_rpe` | 7 | próg ostrzeżenia `W_HIGH_EFFORT` |
| `metrics.knee_2d_offset_deg` | 0 | korekta kątów kolana 2D (C-21); 0, bo zakres `saddle_height` (Ferrer-Roca) także pochodzi z pomiaru 2D; zmiana tylko po badaniu walidacyjnym (`docs/16`, rozdz. 6) |

| `metrics.validation_status` | wszystkie `unvalidated` | status walidacji każdego pomiaru osobno dla `markers_mode` (`kit`/`diy` = markery, `none`) – wynik badania V1.1 (`docs/16`, rozdz. 6.1) |

Wszystkie progi są **technicznymi parametrami jakości pomiaru**, nie normami biomechanicznymi. Zmiana progów lub
statusów walidacji = nowa wersja algorytmu. Każdy pomiar w `result.json` ma `validation_status`.

## 12. Zasoby i wydajność

- Pamięć: brak przechowywania klatek; seria punktów ~ 3600 klatek × 13 punktów × 4 liczby – pomijalna.
- Kadry kluczowe w drugim przebiegu (odczyt wybranych klatek po indeksie).
- Cel: ≤ 15 min dla 60 s 1080p60 na CPU hostingu (NFR-01). Pomiar czasu w `result.algorithm.processing_ms`
  z podziałem na etapy.

## 13. Weryfikacja środowiska (zadanie E2) i porównanie modeli (zadanie E3)

E2 – na hostingu przez SSH, wynik zapisany w `docs/18` (D-01, D-02):
1. `python3.9 -V`, system i architektura (`uname -m`), glibc (`ldd --version`), RAM, limity procesów (`ulimit -a`).
2. Utworzenie venv; instalacja i import: `numpy`, `scipy`, `opencv-python-headless`, kandydaci modeli
   (`mediapipe`, `onnxruntime`, `tflite-runtime`); zapis wersji, które się instalują.
3. Dekodowanie nagrań testowych H.264 i HEVC (iPhone).
4. Czas analizy nagrania wzorcowego (60 s, 1080p60) na jednym rdzeniu; zużycie pamięci.
5. Dostępność `ffprobe` (opcjonalnie).
Jeżeli biblioteki nie działają: (a) inny zgodny model, (b) osobne środowisko analizy (VPS/serwer z tym samym CLI,
pobierające `job.json` i odsyłające `result.json` – kontrakt bez zmian), (c) przeniesienie później.

E3 – co najmniej 2 modele na tym samym zestawie ≥ 10 nagrań (z markerami):
| Kryterium | Miara |
|---|---|
| Dokładność punktu | mediana i 90. percentyl odległości punktu modelu od markera (mm) per punkt |
| Stabilność | SD drugiej różnicy współrzędnych (drgania) po wygładzeniu |
| Braki | % klatek bez punktu przy `conf ≥ 0,5` |
| Wpływ na pomiar | różnica zgięcia kolana na dole i pochylenia pleców względem wersji z markerami |
| Zasoby | ms/klatkę, RAM |
| Licencja | zgodność z użyciem komercyjnym |
Wynik: wybór `pose.model` (ADR w `docs/18`).
