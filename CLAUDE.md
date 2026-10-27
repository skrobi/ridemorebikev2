# CLAUDE.md – ridemore.bike Fit

Zasady pracy nad modułem. To nie jest dokumentacja aplikacji – ta jest w `README.md` i `docs/`.

## 1. Zanim zmienisz kod

1. Przeczytaj `README.md` (mapa dokumentacji, pojęcia).
2. Znajdź zadanie w `docs/17-plan-zadan.md` – zadanie wskazuje dokumenty i kryteria akceptacji.
3. Przeczytaj tylko wskazane dokumenty `docs/*.md` i odpowiednie makiety `docs/makiety/png/*.png`.
4. Zlokalizuj istniejące pliki i dopiero wtedy implementuj.

Dokumentacja jest kontraktem. Jeżeli kod musi od niej odejść, zatrzymaj się i zapytaj albo – po decyzji – zaktualizuj
właściwy dokument w tym samym commicie.

## 2. Zakres

- Repozytorium zawiera **wyłącznie moduł Fit**. Nie implementuj kont, logowania ani rejestracji – tożsamość
  użytkownika pochodzi z istniejącej sesji ridemore.bike (`docs/02-architektura.md`, rozdz. 6).
- Wykonuj tylko zmiany potrzebne do bieżącego zadania. Bez refaktoryzacji przy okazji, bez zmian API, schematu bazy
  ani bibliotek spoza zadania.
- Problem spoza zadania zgłoś na końcu jako uwagę; nie naprawiaj go automatycznie.

## 3. Stos i ograniczenia (nie zmieniaj bez decyzji)

| Warstwa | Technologia |
|---|---|
| Backend | PHP 8.2, **bez frameworka**: front controller, własny router, PDO (MariaDB), szablony PHP |
| Frontend | HTML + CSS + **Alpine.js 3** (strona ridemore.bike już go używa), SVG; bez kroku budowania |
| Baza | MariaDB, tabele z prefiksem `fit_`, migracje SQL `migrations/NNN_opis.sql` |
| Analiza | Python **3.9** (maksimum na hostingu: 3.9.23), pakiet `analyzer/ridemore_fit`, uruchamiany z CLI |
| Kolejka | tabela `fit_job` w MariaDB, worker PHP CLI uruchamiany z crona co minutę |
| Serwer | Apache, SSH, cron; brak gwarancji stałych procesów i GPU |

- Python nie łączy się z bazą. Wymiana danych tylko przez pliki JSON zgodne z `docs/11-kontrakt-danych.md`.
- Nie dodawaj zależności bez uzasadnienia w opisie zmiany (licencja, rozmiar, zgodność z Pythonem 3.9 / PHP 8.2).
  Zakazane licencje dla modeli i bibliotek: niekomercyjne (np. OpenPose, SMPL bez licencji), AGPL bez decyzji
  właściciela (np. Ultralytics YOLO).
- Nie deklaruj zgodności bibliotek z hostingiem bez sprawdzenia na hostingu (zadanie E2 w planie).

## 4. Konwencje

- Dokumentacja, interfejs i komunikaty: **język polski**. Identyfikatory w kodzie, bazie i JSON: **angielski**
  (`snake_case` w bazie i JSON, `camelCase` w PHP/JS, `snake_case` w Pythonie).
- Jednostki w danych: milimetry (`_mm`), stopnie (`_deg`), sekundy (`_s`), piksele (`_px`). Kąty zawsze w stopniach.
- Kody pomiarów, regulacji, punktów i błędów – tylko z list w `docs/08`, `docs/09`, `docs/07`, `docs/11`.
- Każdy wynik analizy zapisuje: wersję schematu, wersję algorytmu, model pozy i jego wersję, wersję modelu
  dopasowania, czas przetwarzania, ostrzeżenia.

## 5. Bezpieczeństwo (nie osłabiaj)

- Każde zapytanie o zasób (sesja, rower, kadr, plik) filtruj po `user_id` zalogowanego użytkownika (IDOR).
- Każde żądanie zmieniające stan wymaga tokenu CSRF.
- Pliki wideo i kadry przechowuj **poza katalogiem publicznym**; wydawaj je tylko przez PHP po sprawdzeniu właściciela.
- Nie uruchamiaj poleceń systemowych z danymi od użytkownika. Wywołanie Pythona: stała ścieżka, stałe argumenty,
  ścieżki plików generowane przez serwer.
- Nie loguj tablic punktów ani danych o zdrowiu. Nie zapisuj sekretów w repozytorium (`config/config.php` jest
  w `.gitignore`, w repo tylko `config/config.example.php`).
- Szczegóły: `docs/15-bezpieczenstwo-i-prywatnosc.md`.

## 6. Dane demonstracyjne i uczciwość wyników

- Nie twórz atrap udających działającą analizę. Dane przykładowe muszą być jawnie oznaczone w interfejsie.
- Gdy jakość pomiaru jest niewystarczająca, interfejs pokazuje ostrzeżenie lub ukrywa wartość – nigdy nie pokazuje
  niewiarygodnego pomiaru jako pewnego.
- Zakresy i reguły zaleceń biorą się wyłącznie z konfiguracji modelu dopasowania (`docs/09`) – nie wpisuj progów
  w kod.

## 7. Weryfikacja po implementacji

1. Sprawdź wszystkie zmienione pliki.
2. Uruchom testy: `vendor/bin/phpunit` (PHP), `pytest analyzer/tests` (Python), testy kontraktu JSON,
   a dla zmian interfejsu – testy E2E (`docs/16-testy-i-walidacja.md`). Brakujące testy dopisz.
3. Porównaj ekran z makietą (`docs/makiety/png`).
4. Sprawdź skutki uboczne i to, czy nie zmieniono niczego poza zakresem.
5. Nie deklaruj, że coś działa, jeżeli nie zostało to sprawdzone.

## 8. Po zakończeniu zadania

Nie streszczaj zmian. Napisz tylko: czy zadanie zakończone powodzeniem; jeśli nie – co pozostało; problemy,
ograniczenia i kwestie spoza zakresu. Zaktualizuj status zadania w `docs/17-plan-zadan.md`.
