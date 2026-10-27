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
- wywiad (pełny przy pierwszej analizie, krótki „czy coś się zmieniło” przy kolejnych),
- profil ciała i rowery z ustawieniami (wersjonowanymi),
- instrukcja markerów i nagrania, przesłanie pliku MP4/MOV,
- kolejka zadań i analiza w Pythonie (markery + model pozy, kalibracja, cykle, kadry kluczowe, jakość),
- weryfikacja i ręczna korekta punktów z przeliczeniem na serwerze,
- wynik: 4 regulacje, następny krok, akceptacja „Mi tak wygodnie”, szczegóły dla fittera,
- wirtualne dopasowanie na wariantach policzonych na serwerze (+ dokładne przeliczenie kombinacji),
- zapis zmiany, krótka ocena po jeździe, przypomnienie o nagraniu kontrolnym, ocena zmiany,
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
| FR-10 | Każda nowa analiza zaczyna się od wywiadu: pełnego (pierwsza analiza roweru, uraz, > 180 dni od pełnego) albo krótkiego. |
| FR-11 | Wywiad obejmuje: cel i profil jazdy, wymiary ciała, 6 testów mobilności (odpowiedź opisowa + opcjonalnie liczba), dolegliwości, urazy. |
| FR-12 | Po wywiadzie użytkownik widzi, jak odpowiedzi zmieniają zakresy i kolejność zaleceń (S06). |

### Nagranie i analiza
| ID | Wymaganie |
|---|---|
| FR-20 | Instrukcja markerów (8 punktów + znacznik skali) z opcją „nagrywam bez markerów”. |
| FR-21 | Instrukcja kamery i lista kontrolna; przesłanie MP4/MOV z walidacją po stronie przeglądarki i serwera. |
| FR-22 | Wybór polityki nagrania: usuń po analizie (domyślnie) / zachowaj tylko kadry kluczowe / zachowaj nagranie. |
| FR-23 | Utworzenie zadania analizy w kolejce; żądanie HTTP nie czeka na analizę. |
| FR-24 | Stany zadania: `queued`, `running`, `done`, `done_warnings`, `failed`; ponowienie do 3 prób; komunikat błędu z instrukcją, co poprawić. |
| FR-25 | Analiza zapisuje 4 kadry kluczowe z punktami (źródło: marker / model / interpolacja / ręcznie, pewność), pomiary, przebiegi w cyklu, cykle, jakość, zalecenia i warianty symulacji. |

### Weryfikacja i wynik
| ID | Wymaganie |
|---|---|
| FR-30 | Przed wynikiem użytkownik widzi kadry z punktami; punkty niskiej pewności są wyróżnione. |
| FR-31 | Ręczna korekta punktu wysyła tylko współrzędne; przeliczenie wykonuje serwer (zadanie `correct`); pierwotny wynik zostaje zachowany. |
| FR-32 | Wynik pokazuje następny krok (jedna zmiana), 4 regulacje z paskiem zakresu i statusem słownym, wpływ wywiadu na zakresy. |
| FR-33 | „Mi tak wygodnie”: użytkownik akceptuje ustawienie poza zakresem z powodem; system przestaje zalecać tę zmianę. |
| FR-34 | Zmiana profilu jazdy przelicza zakresy i zalecenia bez nowej analizy wideo (zadanie `refit` lub obliczenie w PHP z gotowych pomiarów – `docs/09`, rozdz. 8). |
| FR-35 | Zakładka „Szczegóły dla fittera”: kąty, definicje, przebiegi w cyklu, cykle, jakość, źródła zakresów. |

### Wirtualne dopasowanie i zmiany
| ID | Wymaganie |
|---|---|
| FR-40 | Wirtualne dopasowanie pokazuje kadr z nagrania z sylwetką „teraz” i obrysem „po zmianie” z wariantów policzonych na serwerze. |
| FR-41 | Zmiana jednego parametru wybiera gotowy wariant; kombinacja zmian pokazuje szacunek i pozwala zlecić dokładne przeliczenie (zadanie `simulate`). |
| FR-42 | Plan zmian można zapisać; zapis zmiany (S13) tworzy nową wersję ustawienia i przypomnienie o nagraniu kontrolnym. |
| FR-43 | Po każdej jeździe użytkownik może jednym kliknięciem ocenić: lepiej / bez różnicy / gorzej. |
| FR-44 | Po kolejnej analizie system ocenia zmianę według 5 kryteriów i werdyktu (`docs/09`, rozdz. 7). |

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
| NFR-05 | Dostępność: kontrast tekstu ≥ 4,5:1, obsługa klawiaturą, etykiety pól, statusy nie tylko kolorem (WCAG 2.1 AA w podstawowym zakresie). |
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
