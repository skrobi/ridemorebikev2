# 04 · Wywiad

Wywiad zastępuje rozmowę bike fittera z kolarzem przed dopasowaniem. Odpowiedzi nie zmieniają pomiaru – zmieniają
**zakresy, priorytety i treść zaleceń** (`docs/09-model-dopasowania.md`, rozdz. 5).

Wersja kwestionariusza: `questionnaire_version = "q-1"`. Każda odpowiedź jest zapisywana z kodem pytania i wersją.

## 1. Zasady

| Sytuacja | Wywiad |
|---|---|
| pierwsza analiza danego roweru | pełny (S02–S06) |
| od ostatniego pełnego > 180 dni (`interview.full_max_age_days`) | pełny |
| użytkownik zaznaczył nowy uraz | pełny |
| pozostałe | krótki (rozdz. 7) – S02 skrócony do jednego ekranu „Czy coś się zmieniło?” |

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

Pytamy tylko o wartości, których brakuje w profilu. Sekcja „Siodło, kierownica i bloki” jest opcjonalna.

| Kod | Nazwa | Jedn. | Zakres walidacji | Jak zmierzyć | Do czego służy |
|---|---|---|---|---|---|
| `body.height_cm` | Wzrost | cm | 120–220 | boso, przy ścianie | kontrola skali, proporcje |
| `body.mass_kg` | Masa ciała | kg | 35–200 | waga łazienkowa | opisy obciążenia (poglądowe) |
| `body.birth_year` | Rok urodzenia | – | 1920–(bieżący−10) | – | statystyki, nie zmienia zaleceń |
| `body.sex` | Płeć (opcjonalnie) | – | `female` · `male` · `undisclosed` | – | nie zmienia zaleceń w v1 |
| `body.inseam_cm` | Długość wewnętrzna nogi | cm | 60–110 | książka między nogami, mocno w górę; od podłogi do grzbietu książki | kontrola punktów z nagrania, przeliczenie mm, start wysokości siodła |
| `body.arm_span_cm` | Rozpiętość ramion | cm | 120–230 | ramiona w bok, od końca palców do końca palców | kontrola długości ramienia + przedramienia |
| `body.shoe_eu` | Rozmiar buta rowerowego | EU | 34–50 | – | ustawienie bloków (gdy brak długości stopy) |
| `body.sit_bone_cm` | Rozstaw kości kulszowych | cm | 8,0–20,0 | siad na tekturze falistej na schodku, stopy w górze; odległość między środkami 2 wgnieceń | dobór szerokości siodła (`docs/09`, rozdz. 6) |
| `body.shoulder_width_cm` | Szerokość barków | cm | 28–55 | z tyłu, między wyrostkami barkowymi (kostne wypustki na szczycie barków) | dobór szerokości kierownicy |
| `body.foot_length_cm` | Długość stopy | cm | 20,0–33,0 | obrys stopy na kartce, od pięty do najdłuższego palca | ustawienie bloków przód/tył |
| `body.foot_arch` | Łuk stopy | – | `flat` płaski · `medium` średni · `high` wysoki | mokry ślad stopy na tekturze | wkładka, podkładki pod bloki (informacyjnie) |
| `body.leg_length_diff` | Czy jedna noga jest krótsza? | – | `no` · `yes` · `unknown` | stwierdzone przez lekarza / fizjoterapeutę | flaga dla fittera |
| `body.leg_length_diff_side` | Która krótsza | – | `left` · `right` | – | jw. |
| `body.leg_length_diff_mm` | O ile | mm | 1–30 | wg dokumentacji medycznej | jw.; **nie** zmienia zaleceń automatycznie |

## 4. Krok 3 – mobilność (S04)

Każdy test ma odpowiedź opisową (wymagana, jeśli test nie pominięty) i opcjonalną wartość liczbową. Jeżeli podano
liczbę, kategoria jest wyliczana z niej (progi poniżej). **Progi kategorii są robocze – do zatwierdzenia przez
fizjoterapeutę / bike fittera** (`docs/18`, D-07).

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

## 5. Krok 4 – dolegliwości (S05)

| Kod obszaru | Etykieta | Strona |
|---|---|---|
| `neck` | Kark | – |
| `shoulders` | Barki | L/P/obie |
| `hands_numb` | Drętwienie dłoni | L/P/obie |
| `wrists` | Nadgarstki | L/P/obie |
| `lower_back` | Dół pleców | – |
| `saddle` | Krocze / siodło | – |
| `hip` | Biodro | L/P/obie |
| `knee_front` | Przód kolana | L/P/obie |
| `knee_back` | Tył kolana | L/P/obie |
| `knee_medial` | Wewnętrzna strona kolana | L/P/obie |
| `knee_lateral` | Zewnętrzna strona kolana | L/P/obie |
| `calf_achilles` | Łydka / ścięgno Achillesa | L/P/obie |
| `feet_numb` | Drętwienie stóp | L/P/obie |
| `none` | Nic mi nie dokucza | – |

Dla każdego obszaru: `intensity` 0–10 (0 = nic, 10 = nie da się jechać), `onset`:
`start` od początku · `m30` po 30 min · `m60` po 60 min · `h2` po 2 h · `after` dopiero po jeździe.

Pole `injuries_text` (do 500 znaków, opcjonalne) i flaga `injury_recent` (uraz w ostatnich 6 miesiącach).

Komunikat bezpieczeństwa (nie blokuje): jeśli `intensity ≥ 7` w dowolnym obszarze lub `injury_recent = true` –
„Jeśli coś boli, a nie tylko męczy, skonsultuj się z lekarzem lub fizjoterapeutą przed zmianami pozycji.”

## 6. Krok 5 – co z tego wynika (S06)

Ekran wyświetla listę aktywnych modyfikatorów (`docs/09`, rozdz. 5) w postaci „odpowiedź → wpływ” oraz wynikowy
profil analizy, np. `endurance` + `less_flexible`. Teksty pochodzą z konfiguracji modelu dopasowania.

## 7. Krótki wywiad (kolejne analizy)

| Kod | Pytanie | Typ |
|---|---|---|
| `short.setup_changed` | Czy zmieniałeś coś w rowerze od ostatniej analizy? | tak/nie; „tak” → przejście do zapisu zmiany (S13), jeśli nie zapisano |
| `short.profile_same` | Czy nadal jeździsz w profilu „X”? | tak / zmień |
| `short.complaints` | Czy zmieniły się dolegliwości? | bez zmian / lepiej / gorzej / nowe (→ krok 4) |
| `short.injury` | Czy był uraz od ostatniej analizy? | nie / tak (→ pełny wywiad) |
| `short.rides_since_change` | Ile razy jeździłeś po ostatniej zmianie? | liczba 0–50 (dla oceny zmiany „za wcześnie”) |

## 8. Zapis

| Tabela | Zawartość |
|---|---|
| `fit_interview` | nagłówek: użytkownik, rower, rodzaj (`full`/`short`), `questionnaire_version`, profil, wskaźniki pochodne (`flex_*`), data |
| `fit_interview_answer` | kod pytania, wartość tekstowa/liczbowa, jednostka, strona (dla testów obustronnych) |
| `fit_complaint` | obszar, strona, intensywność, moment pojawienia się (dla wywiadu i dla ocen komfortu) |
| `fit_rider_profile` | aktualne wymiary ciała (nadpisywane, z `updated_at`) |

Szczegóły kolumn: `docs/12-baza-danych.md`.
