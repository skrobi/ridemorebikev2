# Makiety ekranów

Statyczne makiety przepływu (dane przykładowe). Pliki `NN-*.html` otwierają się bezpośrednio w przeglądarce
(linki między ekranami działają); `png/NN-*.png` to zrzuty do szybkiego podglądu.

Makiety są wzorcem układu, hierarchii i treści. Specyfikacja zachowania: `docs/03-ekrany-i-przeplyw.md`.
Kod makiet **nie** jest kodem aplikacji – nie kopiować go do `templates/` (używa styli inline i danych na sztywno).

| Ekran | Makieta | Zrzut | Specyfikacja |
|---|---|---|---|
| S01 Moje analizy | [01-main.html](01-main.html) | ![S01](png/01-main.png) | `docs/03` rozdz. 3 |
| S02 Wywiad – cel i profil jazdy | [02-wywiad1.html](02-wywiad1.html) | ![S02](png/02-wywiad1.png) | rozdz. 4 |
| S03 Wywiad – ciało | [03-wywiad2.html](03-wywiad2.html) | ![S03](png/03-wywiad2.png) | rozdz. 5 |
| S04 Wywiad – mobilność | [04-wywiad3.html](04-wywiad3.html) | ![S04](png/04-wywiad3.png) | rozdz. 6 |
| S05 Wywiad – dolegliwości | [05-wywiad4.html](05-wywiad4.html) | ![S05](png/05-wywiad4.png) | rozdz. 7 |
| S06 Wywiad – co z tego wynika | [06-wywiad5.html](06-wywiad5.html) | ![S06](png/06-wywiad5.png) | rozdz. 8 |
| S07 Markery | [07-markery.html](07-markery.html) | ![S07](png/07-markery.png) | rozdz. 9 |
| S08 Nagranie | [08-nagranie.html](08-nagranie.html) | ![S08](png/08-nagranie.png) | rozdz. 10 |
| S09 Analiza w toku | [09-analiza.html](09-analiza.html) | ![S09](png/09-analiza.png) | rozdz. 11 |
| S10 Sprawdź punkty | [10-punkty.html](10-punkty.html) | ![S10](png/10-punkty.png) | rozdz. 12 |
| S11 Wynik analizy | [11-wynik.html](11-wynik.html) | ![S11](png/11-wynik.png) | rozdz. 13 |
| S12 Wirtualne dopasowanie | [12-symulator.html](12-symulator.html) | ![S12](png/12-symulator.png) | rozdz. 14 |
| S13 Zapis zmiany | [13-zmiana.html](13-zmiana.html) | ![S13](png/13-zmiana.png) | rozdz. 15 |
| S14 Ocena zmiany | [14-ocenazmiany.html](14-ocenazmiany.html) | ![S14](png/14-ocenazmiany.png) | rozdz. 16 |

Znane uproszczenia makiet (obowiązuje specyfikacja, nie makieta):
- S10 i S12 pokazują sylwetkę na ciemnym tle w miejscu zdjęcia z nagrania.
- S10 nie pokazuje wyboru zakresu korekty („we wszystkich klatkach / tylko w tym kadrze”).
- S11 nie pokazuje zakładki „Szczegóły dla fittera” (zawartość: `docs/03` rozdz. 13).
- S12: suwaki są statyczne; wartości „było/będzie” przykładowe.
- Ekrany Rowery, Profil i Komfort nie mają makiet w tym zestawie (`docs/03` rozdz. 17).

## Różnice po panelu ekspertów (do naniesienia na makiety)

Makiety powstały przed przeglądem `docs/20`. Do czasu ich aktualizacji obowiązują poniższe zmiany (specyfikacja:
`docs/03`, teksty: `docs/21`).

| Ekran | Zmiana | Zmiana panelu |
|---|---|---|
| wszystkie z paskiem kroków | szacowany czas przy krokach; krok 4 „Bezpieczeństwo i dolegliwości” | C-20, C-03 |
| S01 | pewność i wysiłek na karcie następnego kroku; „Zaplanuj”; karta „Jak się jechało?” (5 przycisków); trend komfortu w historii; karta „ok. 5 minut pytań”; etykiety werdyktów z `docs/21`; stan „Najpierw konsultacja” | C-06, C-07, C-08, C-12, C-13 |
| S02 | link „Szybki start (ok. 1 min)”; nowy ekran S02q | C-11 |
| S04 | 3 testy podstawowe + zwinięte „Dodatkowe testy”; pola 2. próby, „sam / z pomocą”, „test wywołał ból”; etykiety kategorii bez ocen osobowych | C-01, C-02, C-15 |
| S05 | karta 7 pytań bezpieczeństwa; obszar „Drętwienie krocza”; szczegóły dolegliwości (rodzaj, czas trwania, częstość, poza rowerem, główna); lista operacji z datą | C-03, C-04, C-05, C-13 |
| S06 | komunikat bezpieczeństwa; profil „z uwzględnieniem ruchomości” zamiast „mniej giętki” | C-03, C-15 |
| S08 | RPE 0–10 i moc; 8. pozycja listy kontrolnej („jadę umiarkowanie”); karta QR „Nagrywasz telefonem?”; neutralne opisy polityk nagrania; domyślna strona z dolegliwości; nowy ekran M01 | C-10, C-16, C-18, C-22 |
| S11 | paski bez czerwieni z obramowaniem strefy zalecanej; status „Warto zmienić”; pewność z „dlaczego?”; wysiłek; „Zaplanuj”; stany „Małymi krokami”, „Najpierw konsultacja”, „Dopasowanie podstawowe” | C-03, C-06, C-07, C-11, C-15, C-19 |
| S12 | paski bez czerwieni; tryb tylko do podglądu przy `stop` | C-03, C-19 |
| S13 | karta „Przypomnienia” (pulpit / e-mail opt-in, limit 2); wejście z planu | C-08, C-14 |
| S14 | etykiety werdyktów z `docs/21`; ocena 5-stopniowa i siła dolegliwości 0–10; opis progu istotnej poprawy | C-12, C-15 |

Źródło makiet: kanwa projektowa „ridemore Fit – wywiad i profil jazdy” (artefakt claude.ai), eksport 2026-10-27.
