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

### 1.1 Źródła panelu ekspertów (`docs/20`)

| ID | Pozycja | Wykorzystanie | Uwagi |
|---|---|---|---|
| `clarsen_2010` | Clarsen B., Krosshaug T., Bahr R. „Overuse injuries in professional road cyclists”, Am J Sports Med 2010. <https://pubmed.ncbi.nlm.nih.gov/20847225/> | częstość bólu dolnego odcinka pleców i przedniej części kolana | uzasadnia szczegóły dolegliwości (C-13) |
| `farrar_2001` | Farrar J.T. i in. „Clinical importance of changes in chronic pain intensity measured on an 11-point numerical pain rating scale”, Pain 2001 | próg istotnej zmiany bólu ok. 2 pkt / 30% | `feels_better` (C-12); badanie w bólu przewlekłym – przeniesienie na kolarzy jest założeniem |
| `kamper_2009` | Kamper S.J. i in. „Global rating of change scales: a review of strengths and weaknesses”, J Man Manip Ther 2009. <https://pmc.ncbi.nlm.nih.gov/articles/PMC2762832> | 5-stopniowa ocena po jeździe | C-12, C-13 |
| `bennell_1998` | Bennell K. i in. „Intra-rater and inter-rater reliability of a weight-bearing lunge measure of ankle dorsiflexion”, Aust J Physiother 1998 | rzetelność testu kolano–ściana w rękach badaczy | C-01, C-02 |
| `mayorga_vega_2014` | Mayorga-Vega D. i in. „Criterion-related validity of sit-and-reach tests…: a meta-analysis”, J Sports Sci Med 2014. <https://www.jssm.org/jssm-13-1.xml-abst> | trafność siad i skłon: umiarkowana (tył ud), niska (lędźwie) | C-01 |
| `inclinometer_smartphone` | Badania rzetelności pomiaru kąta aplikacją telefonu. <https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6617898/> | pomiar kąta telefonem w testach T3, T6 | dotyczy warunków badania, nie samodzielnych testów |
| `ferrer_roca_crank` | Ferrer-Roca V. i in. – wpływ długości korby na kinematykę kończyny dolnej. <https://lida.sport-iat.de/bdr/Record/4033004> | dłuższe korby ↔ większe zgięcie kolana i biodra | `knee_flex_top`, `tip.crank_length` (C-09) |
| `reynolds_2025` | Reynolds i in. „The impact of minor crank length adjustments on lower body cycling kinematics”, 2025. <https://repository.derby.ac.uk/item/qy8v2/the-impact-of-minor-crank-length-adjustments-on-lower-body-cycling-kinematics> | krótsze korby zmniejszają zgięcie kolana i biodra | C-09 |
| `bini_fatigue` | Bini R. i in. – zmiany kinematyki przy obciążeniu i zmęczeniu. <https://pmc.ncbi.nlm.nih.gov/articles/PMC7988189/> | standaryzacja wysiłku nagrania (RPE) | C-10 |
| `borg_cr10` | Borg G. „Psychophysical bases of perceived exertion”, Med Sci Sports Exerc 1982 | skala RPE 0–10 | C-10 |
| `kinematics_2d_3d` | García-López J. i in. – porównanie pomiaru 2D i 3D w kolarstwie. <https://core.ac.uk/works/70944953>; korekta ok. +2,2°: <https://lida.sport-iat.de/bdr/Record/4032256?lng=en> | 2D zaniża kąty kolana o ok. 2–3° | `metrics.knee_2d_offset_deg = 0` do walidacji (C-21) |
| `perineal_meta_2020` | Litwinowicz K. i in. – metaanaliza: kolarstwo a drętwienie krocza i rodzaj siodła, 2020. <https://pmc.ncbi.nlm.nih.gov/articles/PMC7846539/> | `M_PERINEAL`, porada o siodle | C-05 |
| `pfp_review` | Przegląd czynników ryzyka bólu rzepkowo-udowego u kolarzy. <https://lida.sport-iat.de/bdr/Record/4005584> | niejednoznaczny związek wysokości siodła z bólem kolana | język nieprzyczynowy (C-15) |
| `msk_triage_nhs` | Ścieżki triażu MSK (NHS Borders / Highland). <https://www.rightdecisions.scot.nhs.uk/borders-ref-help-toolkit/physiotherapy/msk-physiotherapy/lower-limb> | sygnały do konsultacji (czerwone flagi) | lista `stop`/`caution` (C-03), D-15 |
| `van_der_bles_2020` | van der Bles A.M. i in. „The effects of communicating uncertainty on public trust in facts and numbers”, PNAS 2020. <https://research.rug.nl/en/publications/the-effects-of-communicating-uncertainty-on-public-trust-in-facts/> | ujawnianie niepewności | poziom pewności (C-06) |
| `barnes_2019` | Barnes K. i in. „Can positive framing reduce nocebo side effects? Current evidence and recommendation for future research”, 2019. <https://pubmed.ncbi.nlm.nih.gov/30894815/> | ujęcie pozytywne, unikanie nocebo | `docs/21` (C-15) |
| `garcia_retamero_2013` | Garcia-Retamero R., Cokely E.T. „Communicating health risks with visual aids”, Curr Dir Psychol Sci 2013. <https://journals.sagepub.com/doi/10.1177/0963721413491570> | obraz + słowo dla osób o niskiej numeryczności | C-06 |
| `gollwitzer_2006` | Gollwitzer P.M., Sheeran P. „Implementation intentions and goal achievement: a meta-analysis”, 2006 | intencja wdrożeniowa (d ≈ 0,65) | okno „Zaplanuj” (C-08) |
| `sms_reminders_meta` | Metaanaliza przypomnień tekstowych w przestrzeganiu zaleceń zdrowotnych. <https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9850787/> | przypomnienia | C-14 |
| `jachimowicz_2019` | Jachimowicz J.M. i in. „When and why defaults influence decisions: a meta-analysis of default effects”, Behavioural Public Policy 2019. <https://ideas.repec.org/a/cup/bpubpo/v3y2019i02p159-186_00.html> | siła ustawień domyślnych | uzasadnia ostrożność przy domyślnej retencji (C-16) |
| `ng_2012` | Ng J.Y.Y. i in. „Self-determination theory applied to health contexts: a meta-analysis”, Perspect Psychol Sci 2012 | wspieranie autonomii | `docs/21` (C-14, C-15) |
| `galesic_2009` | Galesic M., Bosnjak M. „Effects of questionnaire length on participation and indicators of response quality in a web survey”, Public Opinion Quarterly 2009 | dłuższy deklarowany czas → mniej rozpoczętych ankiet | budżet czasu wywiadu (C-11, C-20) |
| `cvd_prevalence` | Przegląd częstości zaburzeń widzenia barw. <https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12385717/> | do ok. 8% mężczyzn | paski bez czerwieni (C-19) |
| `wcag_22` | W3C „Web Content Accessibility Guidelines (WCAG) 2.2”. <https://www.w3.org/TR/WCAG22/> | poziom AA | NFR-05, `docs/14` |

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
