# ridemore.bike Fit – moduł analizy i dopasowania pozycji na rowerze

Moduł platformy **ridemore.bike**, który na podstawie nagrania jazdy na trenażerze (kamera z boku) i krótkiego
wywiadu z rowerzystą:

1. mierzy pozycję (kąty i odległości wyznaczone z punktów anatomicznych, wspomaganych markerami na ciele),
2. ocenia jakość pomiaru i pozwala ręcznie poprawić punkty,
3. podaje **proste zalecenie, co zmienić w rowerze** (np. „Podnieś siodło o ok. 10 mm”),
4. pozwala **wirtualnie sprawdzić zmianę** na własnej sylwetce, zanim cokolwiek zostanie przykręcone,
5. zapisuje zmiany ustawień i **ocenia, czy zmiana pomogła** (pomiar + odczucia).

Moduł nie diagnozuje chorób i nie wyznacza jednej „idealnej pozycji”. Zalecenia są szacunkiem opartym na
publikacjach i praktyce bike fittingu, dopasowanym do profilu jazdy i wywiadu.

**Status:** dokumentacja projektowa (bez kodu). Realizacja etapami z bramkami M0–M5 – najpierw wykonalność
i zwalidowany pomiar, potem zalecenia (`docs/17`). Repozytorium zawiera wyłącznie moduł Fit; platforma ridemore.bike
(konta, sesje, strona) działa osobno i jest źródłem tożsamości użytkownika.

---

## Mapa dokumentacji

| Dokument | Zawartość |
|---|---|
| `CLAUDE.md` | zasady pracy dla Claude Code (kolejność czytania, zakres, bezpieczeństwo, weryfikacja) |
| `docs/01-wymagania.md` | cel, użytkownicy, zakres MVP, wymagania funkcjonalne (FR) i niefunkcjonalne (NFR), ograniczenia środowiska |
| `docs/02-architektura.md` | warstwy A–E, struktura katalogów, przepływ danych, kolejka zadań, integracja z kontami ridemore.bike |
| `docs/03-ekrany-i-przeplyw.md` | ekrany S01–S14, S02q i M01 z makiet: zawartość, dane, stany, akcje, kryteria akceptacji |
| `docs/04-wywiad.md` | wszystkie pytania wywiadu: typy, opcje, jednostki, walidacja, sposób pomiaru, wpływ na analizę |
| `docs/05-rower-i-ustawienia.md` | parametry roweru: geometria ramy, komponenty, punkty kontaktu, bloki, koła; definicje i jednostki |
| `docs/06-nagranie-i-markery.md` | protokół nagrania, ustawienie kamery, markery na ciele, znacznik skali, limity plików |
| `docs/07-analiza-obrazu.md` | potok przetwarzania wideo (Python): markery, estymacja pozy, fuzja, śledzenie, kalibracja, cykle, kadry kluczowe, jakość |
| `docs/08-pomiary.md` | definicje geometryczne wszystkich pomiarów, wzory, konwencje znaków, wartości raportowane |
| `docs/09-model-dopasowania.md` | model dopasowania (moduł E): profile jazdy, zakresy, modyfikatory z wywiadu, reguły zaleceń, „Mi tak wygodnie”, ocena zmiany |
| `docs/10-model-sylwetki-i-symulacja.md` | model kinematyczny 2D sylwetki i roweru, warianty wirtualnego dopasowania liczone na serwerze |
| `docs/11-kontrakt-danych.md` | wersjonowane kontrakty JSON: zlecenie analizy, wynik, korekta punktów, symulacja, kody błędów |
| `docs/12-baza-danych.md` | schemat MariaDB (tabele `fit_*`), relacje, retencja |
| `docs/13-backend-php.md` | backend PHP 8.2 bez frameworka: moduły, trasy HTTP/API, przesyłanie plików, worker kolejki, cron |
| `docs/14-frontend.md` | frontend: szablony PHP + Alpine.js, nakładki SVG, korekta punktów, styl ridemore.bike |
| `docs/15-bezpieczenstwo-i-prywatnosc.md` | autoryzacja, CSRF, IDOR, pliki poza webroot, RODO (wizerunek, dane o zdrowiu), retencja |
| `docs/16-testy-i-walidacja.md` | strategia testów (PHP, Python, kontrakty, E2E) i walidacja pomiarów |
| `docs/17-plan-zadan.md` | etapy i zadania (kontrakty implementacyjne) w kolejności realizacji |
| `docs/18-decyzje-i-otwarte-kwestie.md` | rejestr decyzji (ADR) i kwestii do potwierdzenia |
| `docs/19-zrodla.md` | źródła naukowe i przegląd podobnych aplikacji |
| `docs/20-panel-ekspertow.md` | przegląd dokumentacji przez panel ekspertów: stanowiska, debata, uzgodnione zmiany C-01…C-23 |
| `docs/21-komunikacja.md` | zasady języka i prezentacji: statusy, poziom pewności, wysiłek zmiany, komunikaty bezpieczeństwa, paski zakresu |
| `docs/makiety/` | makiety 17 widoków (S01–S14, S02q, M01, raport pomiarowy; HTML + PNG) – wzorzec wyglądu i przepływu |

## Główny przepływ użytkownika

```
S01 Moje analizy
  → S02–S06 Wywiad (cel i profil jazdy, ciało, mobilność, bezpieczeństwo i dolegliwości, wnioski)
     albo S02q Szybki start (ok. 1 min, wynik „dopasowanie podstawowe”)
  → S07 Markery → S08 Nagranie (lub M01 – przesłanie z telefonu) → S09 Analiza w toku
  → S10 Sprawdź punkty (weryfikacja / ręczna korekta)
  → S11 Wynik (następny krok, 4 regulacje, „Mi tak wygodnie”)
  → S12 Wirtualne dopasowanie → „Zaplanuj” → S13 Zapis zmiany
  → (jazdy z oceną jednym dotknięciem, nagranie kontrolne) → S14 Ocena zmiany
```

## Pojęcia

| Pojęcie | Znaczenie |
|---|---|
| Sesja (analiza) | jedno nagranie + wywiad + wynik dla jednego roweru i jednego ustawienia |
| Ustawienie roweru | zapisany stan punktów kontaktu (siodło, kierownica, korby, bloki) w danym czasie |
| Regulacja | jedna z 4 ocenianych regulacji: wysokość siodła, siodło przód/tył, odległość do kierownicy, wysokość kierownicy |
| Profil jazdy | Komfort, Neutralny, Endurance, Sportowy lub Własny – przesuwa zalecane zakresy |
| Kadr kluczowy | klatka nagrania w fazie korby: 0° (góra, GMP), 90°, 180° (dół, DMP), 270° |
| Marker | naklejka na punkcie anatomicznym, wykrywana na obrazie |
| Wariant | pozycja przeliczona na serwerze dla zmiany jednego parametru roweru (wirtualne dopasowanie) |
| Akceptacja („Mi tak wygodnie”) | świadoma decyzja użytkownika, że ustawienie poza zakresem jest dla niego właściwe |
| Ocena zmiany | werdykt po kolejnej analizie: pomogło / częściowo / bez zmian / gorzej / za wcześnie |
| Poziom bezpieczeństwa | wynik pytań bezpieczeństwa: brak uwag / „ostrożnie” (małe kroki) / „wstrzymaj zmiany” (pomiar bez zaleceń) |
| Poziom pewności | jak bardzo ufamy zaleceniu: pewne / raczej pewne / do sprawdzenia – z wyjaśnieniem |
| Status reguły | tylko pomiar / eksperymentalna / zwalidowana – czy i jak regulacja może dawać zalecenia |
| Status walidacji pomiaru | zwalidowany / niezweryfikowany / ukryty – wynik porównania z pomiarem referencyjnym |
| Karta modelu | opis wersji modelu: zakresy, źródła, wyniki walidacji, ograniczenia, kto zatwierdził |
| Dopasowanie podstawowe / pełne | wynik po szybkim starcie / po pełnym wywiadzie |

## Dla Claude Code

Zacznij od `CLAUDE.md`, a następnie `docs/17-plan-zadan.md`. Każde zadanie wskazuje dokumenty, które trzeba
przeczytać przed implementacją.
