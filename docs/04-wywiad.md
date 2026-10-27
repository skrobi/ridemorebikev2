# 04 · Wywiad

Wywiad zastępuje rozmowę bike fittera z kolarzem przed dopasowaniem. Odpowiedzi nie zmieniają pomiaru – zmieniają
**zakresy, priorytety i treść zaleceń** (`docs/09-model-dopasowania.md`, rozdz. 5).

Wersja kwestionariusza: `questionnaire_version = "q-1"`. Każda odpowiedź jest zapisywana z kodem pytania i wersją.

## 1. Zasady

| Sytuacja | Wywiad |
|---|---|
| pierwsza analiza danego roweru | pełny (S02–S06) **albo** szybki start (rozdz. 8) |
| od ostatniego pełnego > 180 dni (`interview.full_max_age_days`) | pełny |
| użytkownik zaznaczył nowy uraz lub sygnał bezpieczeństwa | pełny |
| pozostałe | krótki (rozdz. 7) – S02 skrócony do jednego ekranu „Czy coś się zmieniło?” |

Budżet czasu (panel, `docs/20`, C-11, C-20): pełny wywiad ≤ 5 min (mediana), krótki i szybki start ≤ 1 min.
Każdy krok pokazuje szacowany czas (`docs/03`, rozdz. 1). Teksty pytań: forma bezosobowa (`docs/21`, rozdz. 1).

- Wartości trwałe (wymiary ciała) zapisują się w profilu (`fit_rider_profile`); wywiad przechowuje kopię wartości
  użytych w analizie (`fit_interview_answer`), aby wynik był odtwarzalny po zmianie profilu.
- Dolegliwości i urazy to dane o zdrowiu – wymagają wyraźnej zgody przed zapisem (`docs/15`, rozdz. 4).
- Pola liczbowe: separator dziesiętny przecinek lub kropka; przechowywanie w jednostkach z tabel.

## 2. Krok 1 – cel i profil jazdy (S02)

| Kod | Pytanie | Typ | Opcje / zakres | Wymagane |
|---|---|---|---|---|
| `ride.bike_id` | Rower | wybór | rowery użytkownika + „Dodaj rower” | tak |
| `ride.profile` | Profil jazdy | wybór | `comfort` Komfort (wyprostowany, rekreacja) · `neutral` Neutralny · `endurance` Endurance (długie trasy, 3 h+) · `sport` Sportowy (nisko, wyścigi) · `custom` Własny (tylko fitter) | tak |
| `ride.weekly_hours` | Ile jeździsz w tygodniu? | wybór | `lt3` do 3 h · `h3_6` 3–6 h · `h6_10` 6–10 h · `gt10` 10 h+ | tak |
| `ride.longest` | Najdłuższa typowa jazda | wybór | `lt1` do 1 h · `h2_3` 2–3 h · `gt4` 4 h+ | tak |
| `ride.priority` | Co jest ważniejsze? (opcjonalnie) | wybór | `comfort` komfort · `pain` mniej bólu · `speed` szybkość/aerodynamika | nie |

## 3. Krok 2 – ciało (S03)

Pytamy tylko o wartości, których brakuje w profilu. Sekcja „Siodło, kierownica i bloki” jest opcjonalna;
rozstaw kości kulszowych jest dodatkowo proponowany w kroku 4 po zaznaczeniu obszaru `saddle` lub `saddle_numb`,
jeśli brak go w profilu (pytanie wtedy, gdy jest potrzebne – C-11).

| Kod | Nazwa | Jedn. | Zakres walidacji | Jak zmierzyć | Do czego służy |
|---|---|---|---|---|---|
| `body.height_cm` | Wzrost | cm | 120–220 | boso, przy ścianie | kontrola skali, proporcje |
| `body.mass_kg` | Masa ciała | kg | 35–200 | waga łazienkowa | opisy obciążenia (poglądowe) |
| `body.birth_year` | Rok urodzenia | – | 1920–(bieżący−10) | – | statystyki, nie zmienia zaleceń |
| `body.sex` | Płeć (opcjonalnie) | – | `female` · `male` · `undisclosed` | – | nie zmienia zaleceń w v1 |
| `body.inseam_cm` | Długość wewnętrzna nogi | cm | 60–110 | książka między nogami, mocno w górę; od podłogi do grzbietu książki | **skala zapasowa** bez znacznika (z wzrostem i rozpiętością ramion), kontrola punktów z nagrania, przeliczenie mm – zastępuje dane roweru (`docs/10`, rozdz. 7) |
| `body.arm_span_cm` | Rozpiętość ramion | cm | 120–230 | ramiona w bok, od końca palców do końca palców | kontrola długości ramienia + przedramienia |
| `body.shoe_eu` | Rozmiar buta rowerowego | EU | 34–50 | – | ustawienie bloków (gdy brak długości stopy) |
| `body.sit_bone_cm` | Rozstaw kości kulszowych | cm | 8,0–20,0 | siad na tekturze falistej na schodku, stopy w górze; odległość między środkami 2 wgnieceń | dobór szerokości siodła (`docs/09`, rozdz. 6) |
| `body.shoulder_width_cm` | Szerokość barków | cm | 28–55 | z tyłu, między wyrostkami barkowymi (kostne wypustki na szczycie barków) | dobór szerokości kierownicy |
| `body.foot_length_cm` | Długość stopy | cm | 20,0–33,0 | obrys stopy na kartce, od pięty do najdłuższego palca | ustawienie bloków przód/tył |
| `body.foot_arch` | Łuk stopy | – | `flat` płaski · `medium` średni · `high` wysoki | mokry ślad stopy na tekturze | wkładka, podkładki pod bloki (informacyjnie) |
| `body.leg_length_diff` | Czy jedna noga jest krótsza? | – | `no` · `yes` · `unknown` | stwierdzone przez lekarza / fizjoterapeutę | flaga dla fittera |
| `body.leg_length_diff_side` | Która krótsza | – | `left` · `right` | – | jw. |
| `body.leg_length_diff_mm` | O ile | mm | 1–30 | wg dokumentacji medycznej | jw.; **nie** zmienia zaleceń automatycznie |

### 3.1 Pomiar ciała krok po kroku (S03a)

Przy każdym wymiarze w S03 jest „Zmierz krok po kroku”, a na górze „Zmierz wszystko krok po kroku (ok. 3 min,
opcjonalne +5 min)”. Kreator pokazuje jeden wymiar na ekranie (np. „2 z 4 · Długość nogi”): rysunek, potrzebne
rzeczy, ponumerowane kroki, typowy błąd, pole wyniku, „Nie mam czym zmierzyć – pomiń”. Wynik od razu sprawdzany
z innymi wymiarami (kontrola wiarygodności – ostrzeżenie, nie blokada). Teksty w i18n, rysunki w zadaniu Q4.3.

| Wymiar | Potrzebne | Kroki | Typowy błąd | Kontrola |
|---|---|---|---|---|
| Wzrost | ściana, książka, ołówek, miarka | 1. Stań boso plecami do ściany: pięty, pośladki i łopatki dotykają ściany. 2. Połóż książkę płasko na głowie, dociśnij do ściany. 3. Zaznacz dolną krawędź książki. 4. Zmierz od podłogi do znaku. | w butach; książka przekrzywiona | 120–220 cm |
| Długość wewnętrzna nogi | twarda książka, ściana, ołówek, miarka; pomocnik ułatwia | 1. Stań boso plecami do ściany, stopy ok. 15–20 cm od siebie (jak na pedałach). 2. Wsuń książkę grzbietem do góry między nogi i unieś **mocno**, jak siodło. 3. Zaznacz górę grzbietu na ścianie. 4. Zmierz od podłogi do znaku. 5. Powtórz i wpisz średnią. | książka za nisko (wynik zaniżony o 2–3 cm); miękka książka | 0,43–0,50 wzrostu |
| Rozpiętość ramion | ściana, miarka; pomocnik | 1. Stań przodem lub tyłem do ściany, ramiona poziomo w bok, palce wyprostowane. 2. Pomocnik zaznacza końce środkowych palców. 3. Zmierz między znakami. | ramiona opuszczone; zgięte palce | 0,95–1,06 wzrostu |
| Rozmiar buta | – | wybierz rozmiar butów rowerowych (EU) | rozmiar butów codziennych | 34–50 |
| Kości kulszowe (opcjonalnie) | tektura falista lub folia aluminiowa na złożonym ręczniku, twardy stołek, ołówek, linijka | 1. Połóż tekturę na stołku. 2. Usiądź, unieś stopy i pochyl się lekko do przodu jak na rowerze – 10 s. 3. Wstań, zaznacz **środki** dwóch wgnieceń. 4. Zmierz odległość między środkami. | miękkie podłoże; mierzenie zewnętrznych krawędzi wgnieceń | 8–20 cm |
| Szerokość barków (opcjonalnie) | pomocnik, linijka lub miarka | 1. Pomocnik stoi za Tobą. 2. Przesuwa palce wzdłuż obojczyka na zewnątrz aż do kostnego „daszka” na szczycie barku – po obu stronach. 3. Mierzy w linii prostej między najbardziej zewnętrznymi punktami. | mierzenie po łuku lub do mięśni | 28–55 cm, 0,20–0,27 wzrostu |
| Długość stopy (opcjonalnie) | kartka, ściana, ołówek | 1. Połóż kartkę przy ścianie. 2. Stań piętą przy ścianie, ciężar na stopie. 3. Zaznacz najdłuższy palec. 4. Zmierz od ściany do znaku. | pomiar w siadzie (stopa krótsza) | 20–33 cm |
| Łuk stopy (opcjonalnie) | ciemny papier lub tektura, woda | 1. Zwilż stopę. 2. Stań na papierze ciężarem ciała. 3. Porównaj ślad z 3 wzorami na ekranie (płaski / średni / wysoki). | ślad w siadzie | wybór z 3 |

## 4. Krok 3 – mobilność (S04)

Każdy test ma odpowiedź opisową (wymagana, jeśli test nie pominięty) i opcjonalną wartość liczbową. Jeżeli podano
liczbę, kategoria jest wyliczana z niej (progi poniżej). **Progi kategorii są robocze – do zatwierdzenia przez
fizjoterapeutę / bike fittera** (`docs/18`, D-07).

**Zestaw (C-01):** testy **podstawowe** – T2 skłon w siadzie, T3 prosta noga w górę, T4 kolano do ściany (zawsze
pokazywane, ok. 2 min); testy **dodatkowe** – T1, T5, T6 (sekcja zwinięta „Dodatkowe testy, ok. 2 min”). Testy
domowe są orientacyjne: rzetelność potwierdzono dla pomiarów wykonywanych przez przeszkolone osoby (Bennell i in.
1998), a test siad i skłon ma umiarkowaną trafność wobec tylnej taśmy uda i niską wobec odcinka lędźwiowego
(Mayorga-Vega i in. 2014). Dlatego mobilność tylko przesuwa zakresy, nigdy nie blokuje zalecenia (`docs/09`, rozdz. 5).

**Metadane każdego testu liczbowego (C-02):**

| Kod (sufiks do kodu testu) | Pytanie | Wartości |
|---|---|---|
| `.trials` | Wynik 1. i 2. próby | dwie liczby; zapisywana średnia (`value_num`) i obie próby (`trials_json`) |
| `.helper` | Kto mierzył? | `self` sam · `helper` z pomocą drugiej osoby |
| `.pain` | Czy test wywołał ból? | `no` · `yes` → test przerwany, kategoria `unknown`, obszar proponowany w kroku 4 |

Druga próba jest opcjonalna; przy jednej próbie zapisuje się jedną wartość.

| Kod | Test | Co ocenia | Instrukcja | Odpowiedzi opisowe → kategoria | Wartość liczbowa | Progi z liczby |
|---|---|---|---|---|---|---|
| `mob.t1_standing_reach` | Skłon stojąc | plecy + tył ud | stojąc, proste nogi, powolny skłon; ile brakuje palcom do podłogi | `floor` dotykam podłogi → `good` · `ankles` kostki → `limited` · `shins` piszczele → `very_limited` | `t1_cm` – brak do podłogi w cm (0 = dotyka; ujemna = dłonie poniżej poziomu stóp na podwyższeniu) | ≤ 0 good · 1–10 limited · > 10 very_limited |
| `mob.t2_sit_reach` | Skłon w siadzie | plecy + tył ud | siad prosty, stopy o ścianę/pudełko; sięgnij do przodu; wynik względem stóp | `beyond` za stopy → good · `toes` do stóp → limited · `short` przed stopami → very_limited | `t2_cm` – plus za stopy, minus przed stopami | ≥ +2 good · −2…+2 limited · < −2 very_limited |
| `mob.t3_slr` | Prosta noga w górę | tył ud | leżąc na plecach, druga osoba unosi prostą nogę do pierwszego oporu; telefon (poziomica) na piszczeli | `gt80` ponad 80° → good · `h60_80` 60–80° → limited · `lt60` poniżej 60° → very_limited | `t3_deg_r`, `t3_deg_l` | > 80 good · 60–80 limited · < 60 very_limited (gorsza strona) |
| `mob.t4_knee_wall` | Kolano do ściany | kostka (zgięcie grzbietowe) | wykrok przodem do ściany, pięta na ziemi, kolano dotyka ściany; maksymalna odległość palców od ściany | `gt10` ponad 10 cm → good · `cm5_10` 5–10 cm → limited · `lt5` mniej → very_limited | `t4_cm_r`, `t4_cm_l` | > 10 good · 5–10 limited · < 5 very_limited |
| `mob.t5_hip_flexor` | Kolano do klatki (test Thomasa uproszczony) | zginacze biodra | leżąc na brzegu łóżka, przyciągnij jedno kolano; obserwuj drugą nogę | `flat` płasko → good · `slight` lekko w górę → limited · `clear` wyraźnie → very_limited | `t5_deg` (opcjonalnie) – uniesienie uda nad poziom | ≤ 0 good · 1–10 limited · > 10 very_limited |
| `mob.t6_thoracic_rot` | Skręt tułowia | odcinek piersiowy | siad na krześle, kij na barkach, obrót w każdą stronę | `free` swobodnie → good · `resisted` z oporem → limited · `limited` mocno ograniczony → very_limited | `t6_deg` (opcjonalnie) | ≥ 45 good · 30–44 limited · < 30 very_limited |

Wskaźniki pochodne (zapisywane w wywiadzie, używane przez model dopasowania):

| Wskaźnik | Wyliczenie |
|---|---|
| `flex.posterior_chain` | najgorsza kategoria z T1, T2, T3 (z podanych) |
| `flex.ankle` | kategoria T4 (gorsza strona) |
| `flex.hip_flexor` | kategoria T5 |
| `flex.thoracic` | kategoria T6 |
| `flex.asymmetry` | `true`, jeśli `|t3_deg_r − t3_deg_l| ≥ 10` lub `|t4_cm_r − t4_cm_l| ≥ 3` |
| `flex.less_flexible` | `true`, jeśli `posterior_chain` = `very_limited` lub co najmniej 2 z (posterior_chain, hip_flexor, thoracic) = `limited`/`very_limited` |

Pominięty test → kategoria `unknown` (brak modyfikatora). Ból podczas testu → przerwij, zaznacz w kroku 4.
Etykiety kategorii w interfejsie: `good` „swobodnie”, `limited` „z lekkim ograniczeniem”, `very_limited`
„z wyraźnym ograniczeniem” (bez etykiet osobowych – `docs/21`).

## 5. Krok 4 – bezpieczeństwo i dolegliwości (S05)

### 5.1 Pytania bezpieczeństwa (C-03)

Wymagane (tak/nie), na początku kroku, przed wyborem obszarów. Odpowiedź „tak” nie przerywa wywiadu.

| Kod | Pytanie | Poziom przy „tak” |
|---|---|---|
| `safety.surgery_recent` | Czy w ciągu ostatnich 6 miesięcy była operacja kolana, biodra, kręgosłupa, barku lub stopy? | `stop` |
| `safety.doctor_restrictions` | Czy lekarz lub fizjoterapeuta zalecił obecnie ograniczenie jazdy lub ruchu? | `stop` |
| `safety.joint_swelling_locking` | Czy staw puchnie i jednocześnie się blokuje lub „ucieka”? | `stop` |
| `safety.numbness_persistent` | Czy drętwienie lub osłabienie ręki albo nogi utrzymuje się dłużej niż dobę po jeździe? | `stop` |
| `safety.night_pain` | Czy ból budzi w nocy lub występuje w spoczynku, niezależnie od jazdy? | `stop` |
| `safety.recent_fall` | Czy w ostatnich 6 tygodniach był upadek lub uraz, po którym ból się utrzymuje? | `stop` |
| `safety.saddle_numb_bladder` | Czy drętwieniu krocza towarzyszą kłopoty z oddawaniem moczu lub stolca? | `stop` + komunikat pilny |

Wyliczenie poziomu `safety_level` (`none` · `caution` · `stop`, `SafetyClassifier`, progi w
`config/questionnaire-q-1.json`):
- `stop` – dowolne „tak” powyżej, operacja z historii (rozdz. 5.3) z datą < 6 mies., albo dolegliwość
  o `intensity ≥ 8`;
- `caution` – dolegliwość o `intensity` 5–7, dolegliwość z `off_bike = true`, operacja ≥ 6 mies. temu,
  `injury_recent = true`;
- `none` – pozostałe.
Skutki poziomu dla zaleceń: `docs/09`, rozdz. 5.1; teksty: `docs/21`, rozdz. 5. Progi robocze – D-15.
Bez zgody `health_data` pytania nie są zadawane ani zapisywane; analiza otrzymuje `safety_level = caution`
i `fit_level = basic` (`docs/15`, rozdz. 4).

### 5.2 Obszary dolegliwości

| Kod obszaru | Etykieta | Strona |
|---|---|---|
| `neck` | Kark | – |
| `shoulders` | Barki | L/P/obie |
| `hands_numb` | Drętwienie dłoni | L/P/obie |
| `wrists` | Nadgarstki | L/P/obie |
| `lower_back` | Dół pleców | – |
| `saddle` | Siedzenie – ucisk lub ból na siodle | – |
| `saddle_numb` | Drętwienie krocza | – |
| `hip` | Biodro | L/P/obie |
| `knee_front` | Przód kolana | L/P/obie |
| `knee_back` | Tył kolana | L/P/obie |
| `knee_medial` | Wewnętrzna strona kolana | L/P/obie |
| `knee_lateral` | Zewnętrzna strona kolana | L/P/obie |
| `calf_achilles` | Łydka / ścięgno Achillesa | L/P/obie |
| `feet_numb` | Drętwienie stóp | L/P/obie |
| `none` | Nic mi nie dokucza | – |

Dla każdego obszaru (C-13):

| Pole | Pytanie | Wartości |
|---|---|---|
| `intensity` | Jak silne? | 0–10 (0 = nic, 10 = nie da się jechać) |
| `type` | Jak to odczuwasz? | `pain` ból · `numb` drętwienie · `tingling` mrowienie · `pressure` ucisk / dyskomfort (dla `hands_numb`, `feet_numb`, `saddle_numb` ustawiane automatycznie na `numb`) |
| `onset` | Kiedy się pojawia? | `start` od początku · `m30` po 30 min · `m60` po 60 min · `h2` po 2 h · `after` dopiero po jeździe |
| `duration_after` | Jak długo trwa po jeździe? | `none` mija od razu · `lt2h` do 2 h · `same_day` do końca dnia · `next_day` następnego dnia też |
| `frequency` | Jak często? | `rare` rzadko · `half` co druga jazda · `always` prawie zawsze |
| `off_bike` | Czy pojawia się także poza rowerem? | `no` · `yes` |

Jeden obszar użytkownik oznacza jako **główną dolegliwość** (`is_primary`) – to ona jest „dolegliwością docelową”
w ocenie zmiany (`docs/09`, rozdz. 9). Przy jednym obszarze ustawiane automatycznie.

### 5.3 Urazy i operacje (C-04)

| Pole | Wartości |
|---|---|
| `surgery[].type` | `acl` więzadło krzyżowe · `meniscus` łąkotka · `knee_replacement` endoproteza kolana · `hip_replacement` endoproteza biodra · `spine` kręgosłup · `shoulder` bark · `foot_ankle` stopa / staw skokowy · `other` inna |
| `surgery[].side` | `left` · `right` · `both` · brak |
| `surgery[].performed_ym` | rok i miesiąc (RRRR-MM) |
| `surgery[].restrictions` | czy obowiązują zalecenia lekarza dotyczące ruchu: `no` · `yes` |

Dodatkowo `injury_recent` (uraz bez operacji w ostatnich 6 miesiącach) i `injuries_text` (do 500 znaków,
opcjonalne – tylko dla fittera, nie wpływa na reguły). Wpływ na model: `docs/09`, rozdz. 5 i 5.1.

## 6. Krok 5 – co z tego wynika (S06)

Ekran wyświetla listę aktywnych modyfikatorów (`docs/09`, rozdz. 5) w postaci „odpowiedź → wpływ” oraz wynikowy
profil analizy, np. „Endurance · z uwzględnieniem ruchomości”. Teksty pochodzą z konfiguracji modelu dopasowania.
Przy `safety_level` ≠ `none` na górze ekranu komunikat z `docs/21`, rozdz. 5 (bez blokowania dalszych kroków).

## 7. Krótki wywiad (kolejne analizy)

| Kod | Pytanie | Typ |
|---|---|---|
| `short.setup_changed` | Czy coś się zmieniło w rowerze od ostatniej analizy? | tak/nie; „tak” → przejście do zapisu zmiany (S13), jeśli nie zapisano |
| `short.profile_same` | Czy profil jazdy „X” jest nadal aktualny? | tak / zmień |
| `short.complaints` | Czy zmieniły się dolegliwości? | bez zmian / lepiej / gorzej / nowe (→ krok 4); przy „lepiej”/„gorzej” – nowa siła głównej dolegliwości 0–10 |
| `short.injury` | Czy był uraz, operacja albo coś z listy bezpieczeństwa (rozdz. 5.1)? | nie / tak (→ pełny wywiad) |
| `short.rides_since_change` | Ile jazd było po ostatniej zmianie? | liczba 0–50 (dla oceny zmiany „za wcześnie”; wstępnie z ocen po jeździe) |

## 8. Szybki start (C-11)

Dla pierwszej analizy roweru użytkownik może wybrać „Szybki start (ok. 1 min)” zamiast pełnego wywiadu.

| Zawartość | Kody |
|---|---|
| rower i profil jazdy | `ride.bike_id`, `ride.profile` |
| główna dolegliwość (jedna) z siłą | obszar z rozdz. 5.2, `intensity` |
| pytania bezpieczeństwa | wszystkie z rozdz. 5.1 (obowiązkowe) |
| wzrost i długość nogi, jeśli brak w profilu | `body.height_cm`, `body.inseam_cm` (można pominąć) |

Wynik ma `fit_level = basic` („dopasowanie podstawowe”): brak modyfikatorów mobilności, pewność zaleceń obniżona
o jeden poziom (`docs/09`, rozdz. 6.1). W S11 karta „Uzupełnij wywiad (ok. 4 min) – dokładniejsze zalecenia”;
dokończenie pełnego wywiadu (`kind = full`, `upgrade_of_analysis_id`) zleca `refit` bez nowego nagrania i daje
`fit_level = full`. Kolejna analiza tego roweru wymaga pełnego wywiadu, jeśli nie został uzupełniony.

## 9. Zapis

| Tabela | Zawartość |
|---|---|
| `fit_interview` | nagłówek: użytkownik, rower, rodzaj (`full`/`short`/`quick`), `questionnaire_version`, profil, wskaźniki pochodne (`flex_*`), `safety_level` i powody, data |
| `fit_interview_answer` | kod pytania, wartość tekstowa/liczbowa, próby testu (`trials_json`), jednostka, strona (dla testów obustronnych); także odpowiedzi `safety.*` |
| `fit_complaint` | obszar, strona, intensywność, rodzaj, moment pojawienia się, czas trwania po jeździe, częstość, poza rowerem, `is_primary` (dla wywiadu i dla ocen komfortu) |
| `fit_surgery` | rodzaj, strona, rok i miesiąc, zalecenia lekarza |
| `fit_rider_profile` | aktualne wymiary ciała (nadpisywane, z `updated_at`) |

Szczegóły kolumn: `docs/12-baza-danych.md`.
