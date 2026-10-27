# 21 · Komunikacja z użytkownikiem

Zasady języka i prezentacji wyników ustalone przez panel ekspertów (`docs/20`, C-06, C-07, C-15, C-19). Obowiązują
wszystkie teksty interfejsu (`templates/i18n/pl.php`) i teksty modelu dopasowania (`texts` w pliku modelu,
`docs/09`, rozdz. 10). Przy przeglądzie treści każdy nowy tekst sprawdza się z listą z rozdz. 7.

## 1. Zasady ogólne

| Zasada | Znaczenie | Uzasadnienie |
|---|---|---|
| Fakt, potem działanie | najpierw co zmierzyliśmy, potem co zrobić; bez przypisywania przyczyny dolegliwości | związki ustawienie ↔ ból są niejednoznaczne (`docs/20`, rozdz. 3.2) |
| Ujęcie pozytywne | opisujemy, co zmiana poprawi, a nie co „niszczy” obecne ustawienie | ograniczenie efektu nocebo (Barnes i in. 2019) |
| Bez etykiet osobowych | opisujemy wynik testu, nie człowieka („napięty tył ud”, nie „mało giętki”) | unikanie stygmatyzacji |
| Uczciwa niepewność | każde zalecenie ma poziom pewności; liczby i przedziały w „Szczegółach” | van der Bles i in. 2020; numeryczność – Garcia-Retamero i Cokely |
| Sprawczość | przy zaleceniu: czas, narzędzia, trudność i możliwość zlecenia w serwisie | zmniejsza barierę wykonania |
| Autonomia | użytkownik decyduje (akceptacja „Mi tak wygodnie”, przypomnienia, zachowanie kadrów); bez presji i odliczania | Ng i in. 2012 |
| Forma bezosobowa | pytania bez form rodzajowych („Czy coś się zmieniło w rowerze?”, nie „Czy zmieniałeś…”) | płeć jest opcjonalna (`docs/04`, rozdz. 3) |
| Jedno zdanie – jedna myśl | zdania do ok. 20 słów; słowa potoczne zamiast terminów (terminy w „Szczegółach” z ⓘ) | odbiorca laik |

### 1.1 Nazewnictwo (ADR-27)

| Pojęcie | Piszemy | Nie piszemy |
|---|---|---|
| całość procesu | „sesja fittingowa”, „Moje sesje”, „Nowa sesja fittingowa” | „analiza” |
| przetwarzanie nagrania (S09) | „Analizujemy nagranie” | „sesja w toku” |
| wynik | „Wynik sesji z 21 października” | „Analiza z …” |

## 2. Statusy i werdykty (etykiety w interfejsie)

| Kod | Etykieta | Ikona | Kolor (token) |
|---|---|---|---|
| `ok` | W porządku | ✓ | `--good` |
| `near` | Blisko granicy | ! | `--warn-ink` |
| `out` | Warto zmienić | ↕ / ↔ (kierunek) | `--ink` na tle `--warn-soft` |
| `unknown` | Nie oceniono | ? | `--ink-mute` |
| `accepted` | Twoje ustawienie | ♥ | `--info-ink` |
| werdykt `effective` | Pomogło | ✓ | `--good` |
| `partial` | Częściowo | ◐ | `--warn-ink` |
| `no_effect` | Bez zmian | – | `--ink-soft` |
| `worse` | Gorzej – wróć do poprzedniego | ↶ | `--bad-ink` |
| `too_early` | Za wcześnie na ocenę | ⏱ | `--ink-mute` |

Kolor czerwony (`--bad-*`) tylko dla błędów, „Wstrzymaj zmiany” i werdyktu `worse` (C-19).

## 3. Poziom pewności zalecenia (C-06)

| Kod | Etykieta | Ikona | Tekst „dlaczego?” (przykłady powodów z `confidence_reasons`) |
|---|---|---|---|
| `high` | Pewne | ●●● | „Punkty dobrze widoczne, powtarzalne obroty, zakres oparty na badaniach.” |
| `medium` | Raczej pewne | ●●○ | „Część klatek miała zasłonięty marker stopy.” |
| `low` | Do sprawdzenia | ●○○ | „Wynik blisko granicy zakresu przy dużej zmienności – nagraj ponownie przed zmianą.” |

- W podsumowaniu: słowo + ikona; liczby (wartość, SD, przedział) wyłącznie w „Szczegółach”.
- Przy `low` przycisk główny „Nagraj kontrolnie” zamiast „Zrobione – zapisz zmianę” (zalecenie pozostaje widoczne).
- Kody powodów (`conf.*`) i reguły: `docs/09`, rozdz. 6.1.

### 3.1 Status reguły, kolejność i przewidywanie

| Sytuacja | Tekst |
|---|---|
| `measure_only` | „Tylko pomiar – bez zalecenia. Tę regulację jeszcze weryfikujemy, dlatego nie podpowiadamy zmiany.” |
| `experimental` | plakietka „Eksperymentalne” + „Zalecenie w fazie sprawdzania. Zmień o 5 mm i nagraj kontrolnie.” |
| pomiar `unvalidated` (Szczegóły) | plakietka „pomiar niezweryfikowany” |
| strefa sporna | „Specjaliści różnią się w ocenie tego ustawienia – nie zalecamy zmiany.” |
| pusty zakres | „Twoje odpowiedzi wskazują sprzeczne potrzeby – to ustawienie warto ocenić z fitterem.” |
| „Dlaczego teraz to?” | „Najpierw {regulacja}, bo od niej zależy reszta pozycji.” |
| „Wpływ na inne ustawienia” | „Kierownica będzie o ok. {mm} mm dalej – nadal w zakresie.” / „… – sprawdzimy to w kolejnym kroku.” |
| przewidywanie (S12) | „Przewidywanie modelu – nie pomiar i nie dowód, że zmiana pomoże. Sprawdzimy to nagraniem po zmianie.” |
| brak danych roweru | „Uzupełnij {pole} – dokładniej pokażemy rower po zmianie i dopasujemy część.” (nie osłabia zalecenia) |
| zmiana ponad zakres regulacji | „Tak duża zmiana sugeruje, że rama może być za duża / za mała – warto sprawdzić rozmiar ramy.” |
| kolejne kroki planu | „Wstępnie – po nagraniu kontrolnym przeliczymy plan.” |

## 4. Wysiłek zmiany (C-07)

Na karcie zalecenia linia: „ok. {minutes} min · {tools} · {difficulty}”, np. „ok. 10 min · klucz imbusowy 4–5 mm ·
łatwe”. Przy `shop_recommended = true` dopisek „Możesz to zlecić w serwisie rowerowym”. Wartości pochodzą z pliku
modelu (`effort`, `docs/09`, rozdz. 6.2).

## 5. Bezpieczeństwo (C-03)

| Poziom | Komunikat (tytuł / treść) |
|---|---|
| `stop` | „Najpierw konsultacja” / „Z Twoich odpowiedzi wynika, że przed zmianami pozycji warto porozmawiać z lekarzem lub fizjoterapeutą. Pomiar zapisaliśmy – wrócisz do niego po konsultacji.” |
| `stop` (drętwienie krocza z zaburzeniami oddawania moczu lub stolca) | „Skontaktuj się pilnie z lekarzem” / „Te objawy wymagają pilnej oceny lekarskiej, niezależnie od roweru.” |
| `caution` | „Małymi krokami” / „Zalecamy zmiany najwyżej o 5 mm i obserwowanie, jak reaguje ciało. Przy nasileniu dolegliwości przerwij i skonsultuj się ze specjalistą.” |

Komunikaty nie stawiają diagnozy i nie łączą objawu z konkretną regulacją.

## 6. Paski zakresu i wykresy (C-19)

- Strefy: zalecana (zielona, z obramowaniem i podpisem „zalecany”), przejściowa (jasna piaskowa), poza zakresem
  (jasna szara). Bez czerwieni na paskach.
- Znacznik wartości ciemny (`--ink`), kontrast ≥ 3:1 względem każdej strefy; znacznik „było” – obrys.
- Etykiety końców słowne z API („siodło nisko” / „siodło wysoko”), nie ocenne.
- Każdy pasek ma opis tekstowy dla czytnika ekranu: „Zgięcie kolana 38°, zalecany zakres 33–40°, w porządku”.

## 7. Słownik zamian (przykłady obowiązujące)

| Zamiast | Piszemy |
|---|---|
| „Siodło przeciąża kolano” | „Na dole obrotu kolano zgina się bardziej niż w zalecanym zakresie.” |
| „Zła pozycja”, „błąd” | „Poza zalecanym zakresem” |
| „Do zmiany” | „Warto zmienić” |
| „Mniej giętki” | „Napięty tył ud” / „ograniczona ruchomość w teście” |
| „Ryzyko urazu” | „Możesz odczuć większe obciążenie {obszaru}” (tylko przy wskaźnikach poglądowych) |
| „Musisz” | „Zalecamy” / „Spróbuj” |
| „Czy zmieniałeś…”, „Ile razy jeździłeś…” | „Czy coś się zmieniło w rowerze…”, „Ile jazd było po zmianie?” |
| „Ból zniknie” | „Wielu osobom taka zmiana poprawia komfort; sprawdzimy to po kilku jazdach.” |

## 8. Ankiety

- Po jeździe: jedno pytanie ogólne (5 stopni: „Dużo lepiej / Lepiej / Bez różnicy / Gorzej / Dużo gorzej”),
  dopiero potem opcjonalnie siła głównej dolegliwości 0–10 (C-12, C-13).
- Skale opisane na końcach (0 = „nic”, 10 = „nie da się jechać”); ten sam opis wszędzie.
- Szacowany czas kroku widoczny przed rozpoczęciem (C-20).
