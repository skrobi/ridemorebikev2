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

Źródło makiet: kanwa projektowa „ridemore Fit – wywiad i profil jazdy” (artefakt claude.ai), eksport 2026-10-27.
