# 18 · Decyzje i otwarte kwestie

## 1. Decyzje przyjęte (ADR)

| ID | Decyzja | Uzasadnienie / źródło |
|---|---|---|
| ADR-01 | Repozytorium zawiera tylko moduł Fit; platforma ridemore.bike pozostaje osobno | decyzja właściciela |
| ADR-02 | Backend: czysty PHP 8.2 bez frameworka; frontend: szablony PHP + Alpine.js, bez kroku budowania | zgodność z obecną platformą i hostingiem |
| ADR-03 | Tożsamość z wspólnej sesji i bazy MariaDB ridemore.bike | decyzja właściciela |
| ADR-04 | Analiza w Pythonie 3.9 jako CLI; bez dostępu do bazy; wymiana przez wersjonowany JSON; worker PHP z crona | ograniczenia hostingu, niezależność modułu |
| ADR-05 | Wszystkie obliczenia (pomiary, zalecenia, warianty, ocena zmian) po stronie serwera w Pythonie; przeglądarka tylko wyświetla, weryfikuje i przyjmuje ręczne korekty; jedyna arytmetyka w PHP to szacunek kombinacji wariantów (oznaczony) | jedno źródło prawdy, brak rozbieżności kątów |
| ADR-06 | Markery na 8 punktach anatomicznych + znacznik skali, fuzja z modelem pozy; tryb bez markerów dopuszczony z niższą pewnością | dokładność i powtarzalność |
| ADR-07 | MVP: analiza 2D z kamery bocznej; 3D (np. Three.js) i nagrania z przodu po MVP | ograniczenia pomiaru i zasobów |
| ADR-08 | Dwie warstwy interfejsu: podsumowanie dla laika (4 regulacje, następny krok, jedna zmiana naraz) i szczegóły dla fittera | wymaganie właściciela |
| ADR-09 | Każda analiza zaczyna się od wywiadu: pełnego lub krótkiego | wymaganie właściciela |
| ADR-10 | Profile jazdy: Komfort, Neutralny, Endurance, Sportowy, Własny (fitter) | wymaganie właściciela |
| ADR-11 | Akceptacja „Mi tak wygodnie” wyłącza zalecenie dla regulacji | budowa ciała / urazy mogą uzasadniać ustawienie poza zakresem |
| ADR-12 | Wirtualne dopasowanie na wariantach liczonych po analizie; kombinacje – szacunek + dokładne przeliczenie zadaniem | brak obliczeń w przeglądarce, szybka interakcja |
| ADR-13 | Tylko aplikacja webowa; aplikacja mobilna później | wymaganie właściciela |
| ADR-14 | Bez symulacji pracy mięśni w MVP; wskaźniki obciążenia tylko poglądowe | brak walidowanego modelu mięśniowo-szkieletowego |

## 2. Decyzje do podjęcia

| ID | Temat | Opcje / co jest potrzebne | Kiedy |
|---|---|---|---|
| D-01 | Zgodność bibliotek na hostingu | wynik zadania F0.2 (wersje, czasy, RAM) | przed P1.3 |
| D-02 | Miejsce analizy | hosting / osobne środowisko (VPS) z tym samym CLI | po F0.2 |
| D-03 | Wersja MariaDB | wpływa na `SKIP LOCKED`, `JSON`; obecnie rozwiązanie zgodne wstecz | F0.2 |
| D-04 | Klucz obcy do `users` i własny klucz `fit_*` w sesji platformy | zgoda administratora platformy; alternatywa: ciasteczko CSRF podpisane HMAC | przed B2.1 |
| D-05 | Domyślna polityka nagrania | `delete_after` (domyślna wg pierwotnych wymagań) albo `keyframes_only` (lepsze UX w S10/S12) | przed B2.5 |
| D-06 | Zatwierdzenie zakresów modelu dopasowania | kto: bike fitter współpracujący z ridemore; wynik: `fit-model-v1` → `v2` | przed produkcją, V5.2 |
| D-07 | Progi kategorii testów mobilności | fizjoterapeuta / fitter | przed B2.4 (można startować z roboczymi) |
| D-08 | Model pozy | wynik porównania E3 (P1.5) | P1.5 |
| D-09 | Zestaw markerów ridemore (`kit`) | produkt fizyczny czy tylko DIY | przed Q4.3 |
| D-10 | Symulacja pracy mięśni | osobny etap (np. OpenSim, licencja Apache-2.0) – wymaga walidacji | po MVP |
| D-11 | Kanał przypomnień | tylko pulpit / e-mail z platformy | przed U3.4 |
| D-12 | Analiza prawna (RODO, art. 9) | podstawa, rejestr, umowa z hostingiem, ewentualna ocena skutków | przed produkcją |
| D-13 | Rola fittera i udostępnianie sesji | zakres uprawnień, panel | po MVP |
| D-14 | Siódmy test mobilności „głęboki przysiad” | dodać / nie (obecnie skłon w siadzie prostym = test 2) | przed B2.4 |

## 3. Pytania do właściciela platformy

| ID | Pytanie | Wpływ |
|---|---|---|
| Q-01 | Nazwa tabeli kont, typ kolumny `id`, nazwa ciasteczka sesji, klucz z id w `$_SESSION`, adres logowania | `docs/02` rozdz. 6, `docs/12` rozdz. 1 |
| Q-02 | Czy konta mają role (np. fitter, admin)? | `fit_role` lub odczyt z platformy |
| Q-03 | Hosting: system, architektura CPU, limity czasu procesów cron i CLI, liczba rdzeni | F0.2, NFR-01 |
| Q-04 | Adres modułu: `/fit` w domenie ridemore.bike czy subdomena | ciasteczka sesji, CSP |
| Q-05 | Ustawienia ciasteczka sesji (`SameSite`, `Secure`, domena) | CSRF, subdomena |
| Q-06 | Czy platforma ma wspólny szablon nagłówka/stopki do użycia w module? | `templates/layout.php` |
