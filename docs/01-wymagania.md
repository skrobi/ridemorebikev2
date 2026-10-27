# 01 · Wymagania

## 1. Cel

Pomóc rowerzyście (szosa, gravel, MTB) poprawić komfort jazdy przez:
- wiarygodny pomiar pozycji z nagrania wideo (2D, kamera z boku, trenażer),
- zalecenia zrozumiałe dla laika („co zmienić w rowerze i o ile”),
- możliwość sprawdzenia zmiany wirtualnie,
- historię zmian i ocenę, czy zmiana pomogła.

Moduł **nie** diagnozuje chorób, **nie** wyznacza jednej idealnej pozycji i **nie** zastępuje bike fittera.

## 2. Użytkownicy i role

| Rola | Opis | Etap |
|---|---|---|
| `rider` | zalogowany użytkownik ridemore.bike; widzi tylko własne dane | MVP |
| `fitter` | bike fitter, któremu rowerzysta udostępnił sesję; może weryfikować punkty, ustawiać profil „Własny” | po MVP (dane i uprawnienia projektowane od początku) |
| `admin` | zarządza wersjami modelu dopasowania, podgląda kolejkę i błędy (bez dostępu do nagrań) | MVP (minimalnie: CLI / prosty widok) |

Rola pochodzi z konta ridemore.bike lub z tabeli `fit_role` (`docs/12-baza-danych.md`).

## 3. Zakres MVP

W zakresie:
- wywiad (pełny lub szybki start przy pierwszej analizie, krótki „czy coś się zmieniło” przy kolejnych) z pytaniami
  bezpieczeństwa,
- profil ciała i rowery z ustawieniami (wersjonowanymi),
- instrukcja markerów i nagrania, przesłanie pliku MP4/MOV (z komputera lub z telefonu przez stronę M01),
- kolejka zadań i analiza w Pythonie (markery + model pozy, kalibracja, cykle, kadry kluczowe, jakość),
- weryfikacja i ręczna korekta punktów z przeliczeniem na serwerze,
- wynik: 4 regulacje, następny krok, akceptacja „Mi tak wygodnie”, szczegóły dla fittera,
- wirtualne dopasowanie na wariantach policzonych na serwerze (+ dokładne przeliczenie kombinacji),
- plan zmiany („Zaplanuj”), zapis zmiany, ocena po jeździe jednym dotknięciem, przypomnienia (pulpit, e-mail
  opt-in), ocena zmiany,
- zdarzenia produktowe do mierników sukcesu (bez danych o zdrowiu),
- retencja nagrań zgodnie z polityką i zgodą.

Poza zakresem MVP: płatności, integracje Strava/Garmin, funkcje społecznościowe i rankingi, nagrania z przodu
(tor kolana), analiza 3D, mapy nacisku, symulacja pracy mięśni, aplikacja mobilna, eksport PDF (planowany po MVP),
panel bike fittera (po MVP).

## 4. Wymagania funkcjonalne

### Profil, rowery, ustawienia
| ID | Wymaganie |
|---|---|
| FR-01 | System rozpoznaje zalogowanego użytkownika z sesji ridemore.bike; bez sesji przekierowuje do logowania ridemore.bike. |
| FR-02 | Użytkownik ma profil ciała (`docs/04-wywiad.md`, rozdz. 3); wywiad pyta tylko o brakujące wartości. |
| FR-03 | Użytkownik zarządza rowerami (typ, rozmiar, geometria) i ich ustawieniami (`docs/05`). |
| FR-04 | Każda zmiana ustawienia tworzy nową wersję ustawienia i wpis w historii zmian (parametr, przed, po, jednostka, data, kto, powód). |

### Wywiad
| ID | Wymaganie |
|---|---|
| FR-10 | Każda nowa analiza zaczyna się od wywiadu: pełnego (pierwsza analiza roweru, uraz, > 180 dni od pełnego), szybkiego startu (pierwsza analiza – wynik „dopasowanie podstawowe”) albo krótkiego. |
| FR-11 | Wywiad obejmuje: cel i profil jazdy, wymiary ciała, 3 podstawowe i 3 dodatkowe testy mobilności (odpowiedź opisowa + opcjonalnie liczba z 2 prób, kto mierzył, ból), pytania bezpieczeństwa, dolegliwości ze szczegółami (rodzaj, czas trwania, częstość, poza rowerem, główna dolegliwość), urazy i operacje z datą. |
| FR-12 | Po wywiadzie użytkownik widzi, jak odpowiedzi zmieniają zakresy i kolejność zaleceń, oraz komunikat bezpieczeństwa, jeśli dotyczy (S06). |
| FR-13 | Poziom bezpieczeństwa: `stop` – pomiar bez zaleceń zmian i komunikat o konsultacji; `caution` – zalecenia z krokiem ≤ 5 mm (`docs/09`, rozdz. 5.1). |
| FR-14 | Uzupełnienie pełnego wywiadu po szybkim starcie przelicza wynik bez nowego nagrania. |

### Nagranie i analiza
| ID | Wymaganie |
|---|---|
| FR-20 | Instrukcja markerów (8 punktów + znacznik skali) z opcją „nagrywam bez markerów”. |
| FR-21 | Instrukcja kamery i lista kontrolna; przesłanie MP4/MOV z walidacją po stronie przeglądarki i serwera. |
| FR-22 | Wybór polityki nagrania: usuń po analizie (domyślnie) / zachowaj tylko kadry kluczowe / zachowaj nagranie; opisy neutralne. |
| FR-26 | Przesłanie nagrania z telefonu: kod QR i krótki adres do strony M01 (wymaga zalogowania); komputer widzi stan przesyłania. |
| FR-27 | Sesja zapisuje wysiłek podczas nagrania (RPE 0–10, opcjonalnie moc); RPE ≥ 7 daje ostrzeżenie i obniża pewność zaleceń. |
| FR-23 | Utworzenie zadania analizy w kolejce; żądanie HTTP nie czeka na analizę. |
| FR-24 | Stany zadania: `queued`, `running`, `done`, `done_warnings`, `failed`; ponowienie do 3 prób; komunikat błędu z instrukcją, co poprawić. |
| FR-25 | Analiza zapisuje 4 kadry kluczowe z punktami (źródło: marker / model / interpolacja / ręcznie, pewność), pomiary, przebiegi w cyklu, cykle, jakość, zalecenia i warianty symulacji. |

### Weryfikacja i wynik
| ID | Wymaganie |
|---|---|
| FR-30 | Przed wynikiem użytkownik widzi kadry z punktami; punkty niskiej pewności są wyróżnione. |
| FR-31 | Ręczna korekta punktu wysyła tylko współrzędne; przeliczenie wykonuje serwer (zadanie `correct`); pierwotny wynik zostaje zachowany. |
| FR-32 | Wynik pokazuje następny krok (jedna zmiana), 4 regulacje z paskiem zakresu i statusem słownym, wpływ wywiadu na zakresy. |
| FR-36 | Każde zalecenie ma poziom pewności (wysoka / średnia / niska) z wyjaśnieniem oraz wysiłek zmiany (czas, narzędzia, trudność, serwis). |
| FR-33 | „Mi tak wygodnie”: użytkownik akceptuje ustawienie poza zakresem z powodem; system przestaje zalecać tę zmianę. |
| FR-34 | Zmiana profilu jazdy przelicza zakresy i zalecenia bez nowej analizy wideo (zadanie `refit` lub obliczenie w PHP z gotowych pomiarów – `docs/09`, rozdz. 8). |
| FR-35 | Zakładka „Szczegóły dla fittera”: kąty, definicje, przebiegi w cyklu, cykle, jakość, źródła zakresów. |

### Wirtualne dopasowanie i zmiany
| ID | Wymaganie |
|---|---|
| FR-40 | Wirtualne dopasowanie pokazuje kadr z nagrania z sylwetką „teraz” i obrysem „po zmianie” z wariantów policzonych na serwerze. |
| FR-41 | Zmiana jednego parametru wybiera gotowy wariant; kombinacja zmian pokazuje szacunek i pozwala zlecić dokładne przeliczenie (zadanie `simulate`). |
| FR-42 | Plan zmian można zapisać (także z okna „Zaplanuj”: kiedy, na których jazdach); zapis zmiany (S13) tworzy nową wersję ustawienia i przypomnienie o nagraniu kontrolnym. |
| FR-43 | Po każdej jeździe użytkownik może jednym kliknięciem ocenić w 5 stopniach (dużo lepiej … dużo gorzej) i opcjonalnie podać siłę głównej dolegliwości 0–10. |
| FR-44 | Po kolejnej analizie system ocenia zmianę według 5 kryteriów i werdyktu (`docs/09`, rozdz. 9); istotna poprawa bólu = spadek o ≥ 2 pkt lub ≥ 30%. |
| FR-45 | Przypomnienia: na pulpicie domyślnie, e-mail tylko po włączeniu; najwyżej 2 na zmianę; można je wyłączyć. |

### Komfort
| ID | Wymaganie |
|---|---|
| FR-50 | Ocena komfortu 0–10 dla obszarów ciała, strona ciała, lokalizacja, moment pojawienia się; powiązana z sesją i ustawieniem. |

## 5. Wymagania niefunkcjonalne

| ID | Wymaganie |
|---|---|
| NFR-01 | Analiza 60 s nagrania 1080p/60 fps kończy się w ≤ 15 min na CPU hostingu (do potwierdzenia w zadaniu E2; jeśli nie – osobne środowisko analizy). |
| NFR-02 | Żadne żądanie HTTP nie trwa dłużej niż 10 s (poza przesyłaniem fragmentów pliku). |
| NFR-03 | Przesyłanie dużych plików fragmentami (domyślnie 8 MB), odporne na przerwanie; limit pliku konfigurowalny (domyślnie 500 MB). |
| NFR-04 | Interfejs po polsku, priorytet: komputer (≥ 1280 px); działa w wąskim widoku (≥ 360 px) bez poziomego przewijania strony. |
| NFR-05 | Dostępność: WCAG 2.2 AA – kontrast tekstu ≥ 4,5:1, elementów graficznych ≥ 3:1, obsługa klawiaturą, alternatywa dla przeciągania, etykiety pól, statusy nie tylko kolorem, paski zakresu bez par czerwony–zielony (`docs/14`, `docs/21`). |
| NFR-11 | Budżet czasu: pełny wywiad ≤ 5 min (mediana), krótki i szybki start ≤ 1 min; szacowany czas widoczny przy krokach. |
| NFR-06 | Przeglądarki: 2 ostatnie wersje Chrome, Edge, Firefox, Safari. |
| NFR-07 | Wyniki odtwarzalne: każdy wynik ma wersję schematu, algorytmu, modelu pozy i modelu dopasowania; ponowna analiza nową wersją nie nadpisuje starej. |
| NFR-08 | Prywatność: nagrania i kadry poza webroot, dostęp tylko właściciela, retencja automatyczna (`docs/15`). |
| NFR-09 | Logi bez danych osobowych o zdrowiu i bez tablic punktów; czas przetwarzania i kody błędów w logach. |
| NFR-10 | Moduł Python bez zależności od schematu bazy; wymiana przez wersjonowany JSON. |

## 6. Ograniczenia środowiska

| Element | Wartość |
|---|---|
| Serwer WWW | Apache |
| PHP | 8.2 |
| Baza | MariaDB (wersja do potwierdzenia – wpływa na `SKIP LOCKED`, `JSON`) |
| Python | maks. 3.9.23 (dostawca zaleca 3.6.15 – nieużywane, brak wsparcia bibliotek) |
| Dostęp | SSH, cron |
| Procesy stałe | niegwarantowane – worker uruchamiany z crona, ograniczony czasem |
| GPU | brak |
| Środowisko deweloperskie | Windows 10/11 |
| ffmpeg / ffprobe | dostępność do sprawdzenia (zadanie E2); fallback: OpenCV w Pythonie |

## 7. Kryteria sukcesu MVP

- Powtarzalność: dwa nagrania tego samego rowerzysty bez zmian ustawień → różnica zgięcia kolana na dole ≤ 3°,
  pochylenia pleców ≤ 3° (z markerami). Weryfikacja: `docs/16`, rozdz. 6.
- Zgodność z bike fitterem: rozbieżność kątów z pomiarem fittera (goniometr / jego system) ≤ 5° w ≥ 80% przypadków.
- Laik bez pomocy przechodzi przepływ S01→S11 (test z 5 osobami) i potrafi powiedzieć, co ma zmienić.

### 7.1 Mierniki produktu (C-17, wartości docelowe robocze)

Liczone z `fit_event` i tabel wyników, bez danych o zdrowiu (`docs/12`, `docs/15`):

| Miernik | Definicja | Cel roboczy |
|---|---|---|
| Ukończenie pierwszej analizy | odsetek rozpoczętych wywiadów zakończonych wynikiem S11 | ≥ 60% |
| Czas wywiadu | mediana pełnego / krótkiego i szybkiego startu | ≤ 5 min / ≤ 1 min |
| Nieudane analizy | odsetek zadań `analyze` zakończonych `failed` | ≤ 15% |
| Wykonanie zalecenia | zapis zmiany ≤ 30 dni od wyniku z zaleceniem | ≥ 40% |
| Domknięcie pętli | zmiany z oceną (nagranie kontrolne lub ≥ 3 oceny po jeździe) | ≥ 25% |
| Pogorszenia | werdykty `worse` wśród ocenionych zmian | ≤ 10% (wyższy – przegląd modelu) |
