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
| ADR-15 | Domyślna polityka nagrania `delete_after`; S08 opisuje opcje neutralnie (zamyka D-05) | prywatność domyślna (RODO art. 25); panel `docs/20`, C-16 |
| ADR-16 | Bez siódmego testu „głęboki przysiad”; 3 testy podstawowe + 3 dodatkowe (zamyka D-14) | dubluje T4 i T5, wydłuża wywiad; testy domowe orientacyjne – `docs/20`, C-01 |
| ADR-17 | Dwa poziomy bezpieczeństwa: `stop` (pomiar bez zaleceń) i `caution` (krok ≤ 5 mm); poziom liczy PHP z wywiadu, skutki stosuje Python | `docs/20`, C-03; ścieżki triażu MSK |
| ADR-18 | Każde zalecenie ma poziom pewności i wysiłek zmiany; zasady języka w `docs/21` | `docs/20`, C-06, C-07, C-15 |
| ADR-19 | Szybki start (~1 min) z wynikiem „dopasowanie podstawowe”; pytania bezpieczeństwa zawsze obowiązkowe | `docs/20`, C-11 |
| ADR-20 | Przesyłanie z telefonu przez stronę M01 po normalnym zalogowaniu (kod QR bez tokenu); prowadzenie kadru na żywo po MVP | `docs/20`, C-18; bez osłabiania uwierzytelnienia |

## 2. Decyzje do podjęcia

| ID | Temat | Opcje / co jest potrzebne | Kiedy |
|---|---|---|---|
| D-01 | Zgodność bibliotek na hostingu | wynik zadania F0.2 (wersje, czasy, RAM) | przed P1.3 |
| D-02 | Miejsce analizy | hosting / osobne środowisko (VPS) z tym samym CLI | po F0.2 |
| D-03 | Wersja MariaDB | wpływa na `SKIP LOCKED`, `JSON`; obecnie rozwiązanie zgodne wstecz | F0.2 |
| D-04 | Klucz obcy do `users` i własny klucz `fit_*` w sesji platformy | zgoda administratora platformy; alternatywa: ciasteczko CSRF podpisane HMAC | przed B2.1 |
| D-05 | ~~Domyślna polityka nagrania~~ | **zamknięta** – ADR-15 | – |
| D-06 | Zatwierdzenie zakresów modelu dopasowania | kto: bike fitter współpracujący z ridemore; wynik: `fit-model-v1` → `v2` | przed produkcją, V5.2 |
| D-07 | Progi kategorii testów mobilności | fizjoterapeuta / fitter; dane z badania testów domowych (`docs/16`, rozdz. 6) | przed B2.4 (można startować z roboczymi) |
| D-08 | Model pozy | wynik porównania E3 (P1.5) | P1.5 |
| D-09 | Zestaw markerów ridemore (`kit`) | produkt fizyczny czy tylko DIY | przed Q4.3 |
| D-10 | Symulacja pracy mięśni | osobny etap (np. OpenSim, licencja Apache-2.0) – wymaga walidacji | po MVP |
| D-11 | Kanał przypomnień | ustalono: pulpit domyślnie, e-mail tylko po włączeniu (C-14); do potwierdzenia – czy platforma udostępnia wysyłkę e-mail (`notifications.email_available`) | przed U3.4 |
| D-12 | Analiza prawna (RODO, art. 9) | podstawa, rejestr, umowa z hostingiem, ewentualna ocena skutków; także podstawa i retencja zdarzeń produktowych `fit_event` | przed produkcją |
| D-13 | Rola fittera i udostępnianie sesji | zakres uprawnień, panel | po MVP |
| D-14 | ~~Siódmy test mobilności~~ | **zamknięta** – ADR-16 | – |
| D-15 | Zatwierdzenie listy i progów bezpieczeństwa (`stop`/`caution`) oraz progów sygnałów (`knee_flex_top`, `hip_closed`, `pelvis_rock`) | lekarz medycyny sportowej / fizjoterapeuta; robocze wartości w `docs/04` rozdz. 5.1 i `docs/09` rozdz. 6.3 | przed produkcją (można startować z roboczymi) |
| D-16 | Kod QR i prowadzenie kadru | biblioteka QR po stronie serwera (licencja MIT/Apache do potwierdzenia, uzasadnienie zależności) albo sam krótki adres; podgląd kamery na żywo – po MVP, po teście iOS/Android | przed B2.5 / po MVP |

## 3. Pytania do właściciela platformy

| ID | Pytanie | Wpływ |
|---|---|---|
| Q-01 | Nazwa tabeli kont, typ kolumny `id`, nazwa ciasteczka sesji, klucz z id w `$_SESSION`, adres logowania | `docs/02` rozdz. 6, `docs/12` rozdz. 1 |
| Q-02 | Czy konta mają role (np. fitter, admin)? | `fit_role` lub odczyt z platformy |
| Q-03 | Hosting: system, architektura CPU, limity czasu procesów cron i CLI, liczba rdzeni | F0.2, NFR-01 |
| Q-04 | Adres modułu: `/fit` w domenie ridemore.bike czy subdomena | ciasteczka sesji, CSP |
| Q-05 | Ustawienia ciasteczka sesji (`SameSite`, `Secure`, domena) | CSRF, subdomena |
| Q-06 | Czy platforma ma wspólny szablon nagłówka/stopki do użycia w module? | `templates/layout.php` |
