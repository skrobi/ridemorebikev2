# Makiety ekranów

Statyczne makiety przepływu (dane przykładowe, jeden spójny scenariusz: brak uwag bezpieczeństwa, ból przodu kolana 4/10, siodło przód/tył zaakceptowane, wysokość kierownicy „tylko pomiar”, rower bez długości mostka). Pliki `NN-*.html` otwierają się bezpośrednio w przeglądarce
(linki między ekranami działają); `png/NN-*.png` to zrzuty do szybkiego podglądu.

Makiety są wzorcem układu, hierarchii i treści. Specyfikacja zachowania: `docs/03-ekrany-i-przeplyw.md`.
Kod makiet **nie** jest kodem aplikacji – nie kopiować go do `templates/` (używa styli inline i danych na sztywno).

| Ekran | Makieta | Zrzut | Specyfikacja |
|---|---|---|---|
| S01 Moje sesje | [01-main.html](01-main.html) | ![S01](png/01-main.png) | `docs/03` rozdz. 3 |
| S02 Wywiad – cel i profil jazdy | [02-wywiad1.html](02-wywiad1.html) | ![S02](png/02-wywiad1.png) | rozdz. 4 |
| S02q Szybki start | [02q-szybki-start.html](02q-szybki-start.html) | ![S02q](png/02q-szybki-start.png) | rozdz. 4 |
| S03 Wywiad – ciało | [03-wywiad2.html](03-wywiad2.html) | ![S03](png/03-wywiad2.png) | rozdz. 5 |
| S03a Pomiar ciała krok po kroku | [03a-pomiar-krok.html](03a-pomiar-krok.html) | ![S03a](png/03a-pomiar-krok.png) | rozdz. 5, `docs/04` rozdz. 3.1 |
| S04 Wywiad – mobilność | [04-wywiad3.html](04-wywiad3.html) | ![S04](png/04-wywiad3.png) | rozdz. 6 |
| S05 Wywiad – bezpieczeństwo i dolegliwości | [05-wywiad4.html](05-wywiad4.html) | ![S05](png/05-wywiad4.png) | rozdz. 7 |
| S06 Wywiad – co z tego wynika | [06-wywiad5.html](06-wywiad5.html) | ![S06](png/06-wywiad5.png) | rozdz. 8 |
| S07 Markery – przygotowanie | [07-markery.html](07-markery.html) | ![S07](png/07-markery.png) | rozdz. 9 |
| S07a Marker krok po kroku | [07a-marker-krok.html](07a-marker-krok.html) | ![S07a](png/07a-marker-krok.png) | rozdz. 9, `docs/06` rozdz. 3.4 |
| S07b Markery – podsumowanie | [07b-markery-gotowe.html](07b-markery-gotowe.html) | ![S07b](png/07b-markery-gotowe.png) | rozdz. 9 |
| S08 Nagranie | [08-nagranie.html](08-nagranie.html) | ![S08](png/08-nagranie.png) | rozdz. 10 |
| M01 Przesłanie z telefonu (390 px) | [08m-telefon.html](08m-telefon.html) | ![M01](png/08m-telefon.png) | rozdz. 10.1 |
| S09 Analiza w toku | [09-analiza.html](09-analiza.html) | ![S09](png/09-analiza.png) | rozdz. 11 |
| S10 Sprawdź punkty | [10-punkty.html](10-punkty.html) | ![S10](png/10-punkty.png) | rozdz. 12 |
| S11 Wynik sesji | [11-wynik.html](11-wynik.html) | ![S11](png/11-wynik.png) | rozdz. 13 |
| S11 tryb „raport pomiarowy” (etap M2) | [11r-raport.html](11r-raport.html) | ![S11r](png/11r-raport.png) | rozdz. 13, `docs/07` rozdz. 9.1 |
| S12 Wirtualne dopasowanie | [12-symulator.html](12-symulator.html) | ![S12](png/12-symulator.png) | rozdz. 14 |
| S13 Zapis zmiany | [13-zmiana.html](13-zmiana.html) | ![S13](png/13-zmiana.png) | rozdz. 15 |
| S14 Ocena zmiany | [14-ocenazmiany.html](14-ocenazmiany.html) | ![S14](png/14-ocenazmiany.png) | rozdz. 16 |

Znane uproszczenia makiet (obowiązuje specyfikacja, nie makieta):
- S10 i S12 pokazują sylwetkę na ciemnym tle w miejscu zdjęcia z nagrania.
- S11 nie pokazuje zakładki „Szczegóły dla fittera” (zawartość: `docs/03` rozdz. 13).
- S12: suwaki są statyczne; wartości „było/będzie” przykładowe.
- Stany „Małymi krokami”, „Najpierw konsultacja”, „Dopasowanie podstawowe” i zalecenie „Eksperymentalne” są opisane w `docs/03` i `docs/21`, bez osobnych makiet.
- S03a i S07a pokazują po jednym przykładowym kroku (długość nogi, kolano); pozostałe kroki według tabel w `docs/04` rozdz. 3.1 i `docs/06` rozdz. 3.4.
- Kod QR w S08 jest wzorem graficznym, nie działającym kodem.
- Ekrany Rowery, Profil i Komfort nie mają makiet w tym zestawie (`docs/03` rozdz. 17).

Makiety pokazują pełny produkt (wydanie 2). Wydanie 1 (etapy M0–M4, `docs/17`) używa S01–S11 z regulacjami
„Tylko pomiar” i jedną regulacją eksperymentalną; S12–S14 – wydanie 2.

Źródło makiet: kanwa projektowa „ridemore Fit – wywiad i profil jazdy” (artefakt claude.ai), eksport 2026-10-27; zaktualizowane w repozytorium o zmiany z `docs/20`, recenzji technicznej i ADR-26 (dane roweru opcjonalne).
