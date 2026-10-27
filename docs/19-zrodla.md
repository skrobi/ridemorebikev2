# 19 · Źródła

Identyfikatory (`source_ids`) używane w pliku modelu dopasowania (`docs/09`, rozdz. 10). Przed produkcją każde
źródło należy zweryfikować w pełnym tekście (część pozycji sprawdzono tylko na podstawie streszczeń i omówień).

## 1. Publikacje

| ID | Pozycja | Wykorzystanie | Uwagi |
|---|---|---|---|
| `ferrer_roca_2012` | Ferrer-Roca V. i in. „Static versus dynamic evaluation in bike fitting: influence of saddle height on lower limb kinematics”, J Strength Cond Res 2012; 26(11): 3025–3029. Opis: <https://lida.sport-iat.de/bdr/Record/4022398?lng=en> | zakres zgięcia kolana 30–40° przy maksymalnym wyproście (pomiar dynamiczny) | pełna lista autorów do uzupełnienia; zakres przytaczany przez późniejsze prace – potwierdzić w pełnym tekście |
| `bini_2011` | Bini R., Hume P., Croft J. „Effects of bicycle saddle height on knee injury risk and cycling performance”, Sports Medicine 2011. Repozytorium: <https://openrepository.aut.ac.nz/handle/10292/2636> | rozbieżność zaleceń (25–30° dla ekonomii), wrażliwość kinematyki kolana na wysokość siodła | przegląd narracyjny/systematyczny |
| `hamstring_saddle_2014` | „The influence of hamstring extensibility on preselected saddle height within experienced competitive cyclists”. Opis: <https://lida.sport-iat.de/bdr/Record/4033015> | modyfikator `M_LESS_FLEX` (wpływ elastyczności tylnej taśmy na wysokość siodła) | autorzy do uzupełnienia |
| `lbp_review_2017` | Przegląd systematyczny: pozycja ciała, aktywność mięśni i kinematyka kręgosłupa u kolarzy z bólem i bez bólu dolnego odcinka pleców (2017). <https://pmc.ncbi.nlm.nih.gov/articles/PMC5315261> | modyfikator `M_LOW_BACK`: niższa kierownica ↔ większe zgięcie lędźwiowe | badania przekrojowe – asocjacja, nie przyczynowość |
| `spine_review_2023` | „Effects of cycling on the morphology and spinal posture in professional and recreational cyclists: a systematic review”. Opis: <https://lida.sport-iat.de/ta/Record/4081343> | niższa kierownica ↔ większe pochylenie miednicy i kifoza lędźwiowa | wspiera `M_LOW_BACK` |

## 2. Praktyka bike fittingu i podobne aplikacje

| Pozycja | Co przejęto | Odnośnik |
|---|---|---|
| MyVeloFit | 4 regulacje (wysokość i przód/tył siodła, wysokość i odległość kierownicy), dla każdej zakres + zalecenie; zaczynać od wysokości siodła, małe kroki | <https://www.myvelofit.com/fit-academy/myvelofit-s-bike-fit-report-explained/> |
| Bike Fast Fit (EZ / Elite / Pro) | kadr z opisanymi pomiarami, paski zakresów, raport przed/po | <https://dcrainmaker.com/2013/10/automated-measurement-review.html>, <https://apps.apple.com/app/id6502648219> |
| BikeViewer | wskazówki prostym językiem, profile jazdy z opcją „mniej giętki”, nakładanie dwóch pozycji, wpływ na kark/dłonie (poglądowo), kolejność dopasowania; zastrzeżenie, że to model geometrii, a nie symulacja biomechaniczna | <https://bikeviewer.com/> (obserwacja publicznej strony) |
| Retül | wywiad przed dopasowaniem, markery na 8 punktach, zapis wymiarów roweru | <https://www.retul.com/retul-fit>, <https://support.specialized.com/retul/en/retuel-fit/what-to-expect-during-a-retuel-fit> |
| idmatch | wywiad o typie roweru, dolegliwościach i nawykach przed pomiarem | <https://thesweetcyclists.com/idmatch-bikefit-review/> |
| Velogic Studio | nakładanie danych 3D na wideo, baza geometrii rowerów | <https://bikerumor.com/velogic-doubles-up-on-3d-motion-capture-bike-fits> |
| GebioMized | mapy nacisku (poza zakresem MVP) | <https://www.cyclefit.co.uk/saddle-fitting> |
| Shimano bikefitting.com | ocena giętkości jako podstawa decyzji o pozycji | <https://bikerumor.com/2013/08/27/shimano-looks-to-fit-in-partners-with-bikefitting-com-on-new-fitting-program/> |
| Formularze przed dopasowaniem | zakres wywiadu: cel, urazy, giętkość, różnica długości nóg, ustawienie stopy | <https://form.jotform.com/203495752702154>, <https://landiscyclery.com/about/specialized-body-geometry-fit-pg349.htm> |

## 3. Oprogramowanie (licencje do potwierdzenia przy F0.2 / P1.5)

| Komponent | Licencja | Status |
|---|---|---|
| OpenCV | Apache-2.0 | kandydat (wymagany) |
| MediaPipe | Apache-2.0 | kandydat modelu pozy |
| MMPose / RTMPose | Apache-2.0 | kandydat (przez ONNX Runtime) |
| ONNX Runtime | MIT | kandydat |
| MoveNet | Apache-2.0 | kandydat |
| Alpine.js | MIT | wybrany |
| opis/json-schema | Apache-2.0 | wybrany (PHP) |
| OpenPose | licencja niekomercyjna | wykluczony |
| Ultralytics YOLO | AGPL-3.0 / komercyjna | wykluczony bez decyzji |
| SMPL | niekomercyjna | wykluczony (3D po MVP – osobna decyzja) |
| OpenSim | Apache-2.0 | ewentualnie po MVP (D-10) |
