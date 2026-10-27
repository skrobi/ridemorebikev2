# 13 · Backend PHP 8.2 (bez frameworka)

## 1. Zasady

- `declare(strict_types=1)` we wszystkich plikach; PSR-4 `Fit\` → `src/`; autoload Composera.
- Zależności produkcyjne: tylko `opis/json-schema` (Apache-2.0) do walidacji `result.json` – uzasadnienie:
  kontrakt jest złożony, ręczna walidacja byłaby zawodna. Deweloperskie: `phpunit/phpunit`, `phpstan/phpstan`.
  Inne – tylko po decyzji.
- Brak globalnego stanu poza front controllerem; zależności przekazywane przez konstruktor (prosty kontener
  w `src/App.php`).
- Odpowiedzi JSON: `application/json; charset=utf-8`, format błędu `{ "error": { "code": "...", "message": "..." } }`.

## 2. Moduły (`src/Domain`)

| Moduł | Odpowiedzialność | Główne klasy |
|---|---|---|
| `Rider` | profil ciała, zgody, rola | `RiderProfileService`, `ConsentService` |
| `Bike` | rowery, wersje ustawień, historia zmian | `BikeService`, `SetupService`, `ChangeService` |
| `Interview` | szkic wywiadu, walidacja odpowiedzi, wskaźniki `flex_*` | `InterviewService`, `MobilityClassifier` |
| `Session` | szkic sesji, przesyłanie, zlecenie analizy | `SessionService`, `UploadService` |
| `Queue` | zadania, przejęcie, ponowienia, wygaszanie | `JobRepository`, `JobScheduler` |
| `Analysis` | import `result.json`, odczyt wyników, kadry, korekty | `ResultImporter`, `AnalysisQuery`, `CorrectionService` |
| `Recommendation` | odczyt regulacji i zaleceń, akceptacje, plan zmian, szacunek kombinacji | `AcceptanceService`, `PlanService`, `EstimateService` |
| `Change` | zapis zmiany, oceny po jeździe, ocena zmiany (import z Pythona), przypomnienia | `ChangeService`, `RideFeedbackService`, `ReminderService` |
| `Comfort` | ocena komfortu | `ComfortService` |
| `Retention` | usuwanie plików i danych | `RetentionService` |

`MobilityClassifier` i progi kategorii pochodzą z pliku konfiguracji kwestionariusza
`config/questionnaire-q-1.json` (ten sam plik czyta Python dla wskaźników) – jedno źródło prawdy.

## 3. Middleware

1. `ErrorHandler` – wyjątki → strona błędu / JSON z identyfikatorem zdarzenia; log bez danych osobowych.
2. `SecurityHeaders` – `Content-Security-Policy` (`default-src 'self'; img-src 'self' blob: data:; style-src 'self' 'unsafe-inline' fonts.googleapis.com; font-src fonts.gstatic.com; script-src 'self'`), `X-Content-Type-Options: nosniff`, `Referrer-Policy: same-origin`, `X-Frame-Options: DENY`.
3. `Auth` – `IdentityProvider` (`docs/02`, rozdz. 6); brak użytkownika → 302 do logowania / 401 dla API.
4. `Csrf` – dla metod POST/PUT/PATCH/DELETE: token z nagłówka `X-CSRF-Token` lub pola `_csrf`; token per sesja,
   porównanie `hash_equals`.
5. `RateLimit` – proste limity na użytkownika w tabeli lub APCu (np. 60 żądań/min API, 5 nowych analiz/dzień –
   konfiguracja).

## 4. Trasy stron (HTML, GET)

Lista i parametry: `docs/03`, rozdz. 2. Kontroler strony ładuje dane przez `*Query`, sprawdza właściciela,
renderuje szablon `templates/pages/S01-moje-analizy.php` itd. Dane do Alpine.js przekazywane jako JSON w atrybucie
`x-data` (escapowany `htmlspecialchars(json_encode(...))`).

## 5. API (`/fit/api`, JSON)

| Metoda | Ścieżka | Opis | Wejście | Wyjście |
|---|---|---|---|---|
| GET | `/me` | użytkownik, profil, flagi | – | `{user, profile, consents}` |
| PUT | `/profile` | zapis wymiarów ciała | pola z `docs/04` §3 | profil |
| GET/POST | `/bikes` | lista / dodanie roweru | `docs/05` §1 | rower |
| GET/PATCH/DELETE | `/bikes/{id}` | szczegóły / edycja / miękkie usunięcie | | |
| GET | `/bikes/{id}/setups` | wersje ustawień | | lista |
| POST | `/bikes/{id}/changes` | zapis zmiany (S13) – tworzy nową wersję ustawienia | `{changes:[{param,before,after,unit}], changed_at, performed_by, reason, reason_note, recommendation_id?, plan_id?}` | `{change_ids, setup_id, reminder}` |
| POST | `/interviews` | nowy szkic wywiadu | `{bike_id, kind}` | wywiad |
| PATCH | `/interviews/{id}` | zapis odpowiedzi kroku (autozapis) | `{answers:[{code,side?,value}], complaints?:[...]}` | wywiad + `flex_*` + modyfikatory (podgląd S06) |
| POST | `/interviews/{id}/complete` | zakończenie | – | wywiad |
| POST | `/sessions` | szkic sesji | `{interview_id}` | sesja |
| PATCH | `/sessions/{id}` | opcje nagrania | `{side, grip, markers_mode, scale_marker_mm, cadence_target_rpm, video_retention, checklist}` | sesja |
| POST | `/sessions/{id}/uploads` | otwarcie przesyłania | `{size_bytes, file_name, mime}` | `{upload_id, chunk_bytes, chunks_total}` |
| PUT | `/uploads/{id}/chunks/{n}` | fragment (body binarne) | `Content-Range` | `{received}` |
| POST | `/uploads/{id}/complete` | złożenie, walidacja, `fit_video` | – | `{video_id}` |
| POST | `/sessions/{id}/submit` | zgody + zlecenie `analyze` | `{consents:[...]}` | `{job_id}` |
| GET | `/jobs/{id}` | stan zadania | – | `{status, stage, progress, error?, analysis_id?}` |
| POST | `/jobs/{id}/retry` | ponowienie `failed` (jeśli plik istnieje) | – | `{job_id}` |
| GET | `/analyses/{id}` | wynik (podsumowanie) | – | regulacje, zalecenia, modyfikatory, jakość, wersje |
| GET | `/analyses/{id}/details` | szczegóły dla fittera | – | metryki, cykle, serie (z `result.json`) |
| GET | `/analyses/{id}/keyframes` | kadry i punkty | – | `[{phase, image_url, width, height, points}]` |
| GET | `/keyframes/{id}/image` | obraz JPEG | – | `image/jpeg` |
| POST | `/analyses/{id}/corrections` | korekta punktu | `{phase, point, x_px, y_px, scope}` | `{job_id}` |
| POST | `/analyses/{id}/confirm-points` | zatwierdzenie punktów (S10) | – | `{ok}` |
| POST | `/analyses/{id}/revert` | przywrócenie wersji bazowej jako bieżącej | `{analysis_id}` | `{ok}` |
| POST | `/analyses/{id}/refit` | zmiana profilu jazdy | `{profile}` | `{job_id}` |
| POST/DELETE | `/acceptances` | „Mi tak wygodnie” / cofnięcie | `{bike_id, regulation, reason, note}` | akceptacja + `{job_id}` (refit) |
| GET | `/analyses/{id}/variants?phase=p90` | warianty z punktami do rysowania | – | lista wariantów |
| POST | `/analyses/{id}/estimate` | szacunek kombinacji (suma różnic wariantów) | `{changes:{param:value}}` | `{metrics, statuses, estimate:true}` |
| POST | `/analyses/{id}/simulate` | dokładne przeliczenie kombinacji | `{changes}` | `{job_id}` |
| POST | `/plans` | zapis planu z S12 | `{analysis_id, items}` | plan |
| POST | `/changes/{id}/rides` | ocena po jeździe | `{ride_date, feeling, note}` | `{ok}` |
| GET | `/changes/{id}/evaluation` | ocena zmiany (S14) | – | werdykt + kryteria |
| POST | `/sessions/{id}/comfort` | ocena komfortu | `{ride_type, duration_min, items:[...]}` | `{ok}` |
| DELETE | `/sessions/{id}/video` | usunięcie nagrania i kadrów na żądanie | – | `{ok}` |

Każdy endpoint: autoryzacja → walidacja wejścia (typy, zakresy z `docs/04`/`docs/05`) → sprawdzenie właściciela →
operacja w transakcji → odpowiedź. Błędy walidacji: 422 z listą pól.

## 6. Przesyłanie pliku

1. `POST /sessions/{id}/uploads`: sprawdzenie `size_bytes ≤ upload.max_bytes`, rozszerzenia z `file_name`
   (nazwa nie jest używana dalej); utworzenie `fit_upload` i katalogu `uploads/{upload_id}/`.
2. `PUT /uploads/{id}/chunks/{n}`: `n` w zakresie, rozmiar fragmentu = `chunk_bytes` (ostatni ≤), zapis
   `part-{n}` (nadpisanie dozwolone – wznowienie).
3. `POST /uploads/{id}/complete`: komplet fragmentów → złożenie strumieniowo do `videos/{user}/{session}/source.{ext}`,
   SHA-256, sprawdzenie sygnatury (`ftyp`) i `finfo`; błąd → usunięcie, `E_VIDEO_FORMAT`.
4. Limity PHP (`upload_max_filesize`, `post_max_size`) muszą przekraczać `chunk_bytes` – zadanie E2 sprawdza.

## 7. Worker (`bin/fit-worker.php`)

```
flock(storage/fit/worker.lock-{slot})       # maks. worker.max_parallel
while (czas < max_seconds && zadań < max_jobs):
    job = JobScheduler::claim()             # docs/02 rozdz. 5
    if !job: break
    zbuduj job.json (JobInputBuilder – z bazy: sesja, wywiad, profil, rower, ustawienie, akceptacje, model)
    PythonRunner::run(type, jobDir, timeout) # proc_open([python_bin, '-m', 'ridemore_fit', type, '--job', ..., '--out', ..., '--progress', ...])
        pętla: co 2 s odczyt progress.json → fit_job.progress, stage, heartbeat_at; przekroczenie czasu → SIGTERM, po 10 s SIGKILL
    kod 0 → ResultImporter (walidacja schematu, transakcja, zapis tabel, przeniesienie kadrów, is_current)
    kod 2 → failed (kod z result.error)
    kod 3/inny → ponowienie lub failed
    RetentionService::afterJob(job)         # usunięcie wideo wg polityki
```

- `PythonRunner` używa tablicy argumentów (`proc_open` z tablicą – bez powłoki), ścieżek generowanych przez serwer,
  zmiennych środowiska ograniczonych do `PATH`, `HOME`, `OMP_NUM_THREADS=1` (konfigurowalne).
- `ResultImporter` jest idempotentny (ponowny import tego samego `job_id` nie dubluje danych).
- Ocena zmiany: jeżeli ustawienie sesji powstało w wyniku zmiany (`fit_change.setup_after_id`), `JobInputBuilder`
  dodaje do `job.json` sekcję `evaluate_change` (`docs/11`, rozdz. 1); Python zwraca `change_evaluation`, a importer
  zapisuje `fit_change_evaluation`. Gdy później dojdą oceny po jeździe lub komfortu, PHP zleca `refit` z tą samą
  sekcją i aktualizuje ocenę. Werdykt liczy wyłącznie Python.

## 8. Cron

```
* * * * *  php /ścieżka/bin/fit-worker.php --max-seconds=840 --max-jobs=3 >> /ścieżka/storage/fit/logs/worker.log 2>&1
17 * * * * php /ścieżka/bin/fit-retention.php >> /ścieżka/storage/fit/logs/retention.log 2>&1
```

## 9. Szacunek kombinacji (`EstimateService`)

Dla zmian `{param: value}`: dla każdego parametru wariant o tej wartości (musi istnieć w siatce, inaczej 422);
`metric_est = metric_current + Σ (metric_variant − metric_current)`; statusy wg zakresów zapisanych w
`fit_regulation` (te same granice co w wyniku). Wynik oznaczony `estimate: true`. To jedyna arytmetyka na
wynikach poza Pythonem – nie tworzy nowych pomiarów ani zaleceń.

## 10. Logowanie

- Plik `storage/fit/logs/app-YYYY-MM-DD.log`, format JSON-lines: czas, poziom, `event`, `request_id`, `user_id`
  (liczba), `job_id`, czas trwania. Zakazane: punkty, serie, odpowiedzi wywiadu, dolegliwości, nazwy plików użytkownika.
- Rotacja: 30 dni.

## 11. i18n

Teksty interfejsu w `templates/i18n/pl.php` (klucz → tekst z parametrami `{name}`). Teksty zaleceń przychodzą
z wyniku (wyrenderowane przez model dopasowania); teksty błędów i ostrzeżeń – z `pl.php` po kodzie.
