# 09 · Model dopasowania (warstwa E)

Model dopasowania zamienia pomiary (`docs/08`) i wywiad (`docs/04`) na statusy 4 regulacji, zalecenia i ocenę zmian.
Jest **konfiguracją** (`analyzer/configs/fit-models/fit-model-v1.json`) interpretowaną przez pakiet
`ridemore_fit.fit`. Progi i teksty nie mogą być zapisane w kodzie PHP ani w szablonach.

**Wszystkie zakresy w v1 są robocze.** Każdy ma poziom dowodu: `science` (publikacja), `practice` (praktyka bike
fittingu, do weryfikacji), `assumption` (założenie projektu). Przed produkcją zatwierdza je bike fitter (`docs/18`, D-06).

**Zasada nadrzędna:** poprawnie działająca aplikacja nie może przedstawiać niezweryfikowanej reguły jako pewnego
zalecenia. Dlatego każda regulacja ma **status reguły** (rozdz. 1.1), a zalecenie w milimetrach wolno wydać tylko
wtedy, gdy jej pomiar przeszedł walidację dokładności (`docs/16`, rozdz. 6) w danym trybie nagrania.

## 1. Regulacje

| Kod | Nazwa (UI) | Pomiar sterujący | Kolejność | Krok maks. | Poziom dowodu |
|---|---|---|---|---|---|
| `saddle_height` | Wysokość siodła | `knee_flex_bottom.med` | 1 | 10 mm | `science` |
| `saddle_fore_aft` | Siodło przód / tył | `knee_over_pedal.at_p90` | 2 | 10 mm | `practice` (KOPS – punkt wyjścia, nie norma) |
| `bar_reach` | Odległość do kierownicy | `elbow_angle.med` (pomocniczo `shoulder_angle.med`) | 3 | 10 mm | `practice` |
| `bar_height` | Wysokość kierownicy | `torso_angle.med` | 4 | 10 mm | `practice` |

Kolumna „Kolejność” to kolejność bazowa „od podstawy w górę”; ostateczną kolejność ustala plan (rozdz. 11).

### 1.1 Status reguły i karta reguły

| Status (`rule_status`) | Co widzi użytkownik | Warunek przejścia |
|---|---|---|
| `measure_only` | wartość i pasek zakresu z dopiskiem „Tylko pomiar – bez zalecenia”; brak akcji i mm | stan początkowy reguły |
| `experimental` | zalecenie kierunku i stałego małego kroku (`step_experimental_mm`, 5 mm) z plakietką „Eksperymentalne”, pewność maks. `medium`, zawsze z nagraniem kontrolnym | pomiar sterujący ma `validation_status = validated` w danym trybie (`docs/16`, rozdz. 6.1) |
| `validated` | pełne zalecenie z wielkością z wariantów (rozdz. 4) | spełnione kryterium walidacji reguły z karty (V4.1 / V5.2) i decyzja rady modelu (rozdz. 13) |

Status zapisuje plik modelu; zmiana statusu = nowa wersja modelu. **Konfiguracja v1:** `saddle_height` =
`experimental`, pozostałe = `measure_only` (etap M4, `docs/17`).

Karta reguły (obowiązkowa dla każdej regulacji, w pliku modelu i w karcie modelu):

| Element | `saddle_height` | `saddle_fore_aft` | `bar_reach` | `bar_height` |
|---|---|---|---|---|
| Wejścia | `knee_flex_bottom.med` (hip, knee, ankle) | `knee_over_pedal.at_p90` (knee, pedal, skala) | `elbow_angle.med` + kierunek `shoulder_angle.med` | `torso_angle.med` (hip, shoulder, poziom) |
| Minimalna jakość | pomiar `good`, ≥ 20 cykli, `validation_status = validated` | jw. + `scale_source = marker` | jw. + chwyt zgodny z deklarowanym | jw. + poziom kamery wyznaczony |
| Wolno wydać zalecenie, gdy | wartość poza zielonym o więcej niż `min_margin` (max(2°, SD)); poza strefą sporną; RPE ≤ 6; brak blokady i `stop` | poza zielonym o > max(5 mm, SD) | poza zielonym o > max(3°, SD) i `shoulder_angle` wskazuje ten sam kierunek | poza zielonym o > max(3°, SD) |
| Krok | 5 mm (`experimental`), ≤ 10 mm (`validated`) | 5 mm | 10 mm (mostki co 10 mm) lub klamkomanetki | 5–10 mm podkładek |
| „Nie oceniono”, gdy | jakość < `good`, `W_SEGMENT_MISMATCH` udo/podudzie, < 20 cykli | `W_SCALE_ESTIMATED`, `W_TILT_UNCERTAIN`, tryb `none` | `W_GRIP_MISMATCH`, niska pewność `wrist`/`elbow` | `W_TILT_UNCERTAIN`, `W_GRIP_MISMATCH` |
| Źródło / dowód | Ferrer-Roca 2012, Bini 2011 / `science` | praktyka (KOPS – punkt wyjścia) / `practice` | praktyka / `practice` | praktyka, `lbp_review_2017` / `practice` |
| Kryterium walidacji reguły | pomiar: kryteria z `docs/16` rozdz. 6.1; przewidywanie: \|Δ przewidziana − Δ zmierzona\| ≤ 2° w ≥ 80% zmian; skutek: ocena fittera „zasadne” ≥ 80% i „gorzej” ≤ 10% w badaniu V4.1 | jw., ± 5 mm | jw., ± 3° | jw., ± 3° |
| Status w v1 | `experimental` | `measure_only` | `measure_only` | `measure_only` |

## 2. Zakresy bazowe

**Metoda pomiaru i konsensus.** Każdy zakres ma pole `method` (w v1: `dynamic_2d_lateral_markers` – wideo z boku,
w ruchu, markery na punktach z `docs/06`). Źródła mierzone inną metodą (np. statycznie goniometrem w dolnym
położeniu korby, 3D, inne punkty anatomiczne) są przeliczane na tę metodę albo opisane jako niezgodne – wiele
pozornych sprzeczności między fizjoterapeutami i fitterami wynika z różnych metod (np. ok. 25–35° statycznie
odpowiada ok. 30–40° w ruchu – Ferrer-Roca i in. 2012). Gdy źródła tej samej metody się różnią, obszar różnicy jest
**strefą sporną** (`disputed`): status `near`, brak akcji, dopisek „specjaliści różnią się w ocenie”. Zmianę
zalecamy tylko wtedy, gdy wartość jest poza zakresem akceptowanym przez wszystkie uwzględnione źródła.

Zielony = zakres zalecany; żółty = strefa przejściowa (`near`); poza żółtym = `out`.

### `saddle_height` – zgięcie kolana na dole obrotu (°)

| Profil | Zielony | Żółty | Cel | Źródło |
|---|---|---|---|---|
| wszystkie | 30–40 | 25–45 | 35 | Ferrer-Roca i in. 2012 (30–40° w pomiarze dynamicznym); przegląd Bini i in. 2011 (25–30° dla ekonomii – rozbieżność odnotowana) |

### `saddle_fore_aft` – kolano względem osi pedału przy korbie poziomo (mm, marker na nadkłykciu bocznym)

| Profil | Zielony | Żółty | Cel | Źródło |
|---|---|---|---|---|
| wszystkie | −45…−15 | −55…−5 | −30 | `practice`; wartości dla nadkłykcia bocznego (nie guzowatości piszczeli) – do kalibracji z fitterem |

### `bar_reach` – kąt łokcia (°)

| Profil | Zielony | Żółty | Cel |
|---|---|---|---|
| comfort, neutral, endurance | 148–165 | 140–170 | 156 |
| sport | 145–162 | 135–168 | 153 |

### `bar_height` – pochylenie pleców (°)

| Typ roweru | comfort | neutral | endurance | sport |
|---|---|---|---|---|
| `road` | 50–65 | 42–52 | 40–50 | 30–42 |
| `gravel` | 55–70 | 47–57 | 45–55 | 35–47 |
| `mtb` | 55–70 | 48–60 | 45–58 | 40–52 |

Żółty = zielony ± 5°. Cel = środek zielonego. Poziom dowodu: `practice`.

### Pomiar pomocniczy `hands_load_share` (tylko symulator i opis obciążenia)
Zielony 20–34%, żółty 15–40% – `assumption` (model poglądowy, `docs/10`, rozdz. 5).

## 3. Status regulacji

| Jakość pomiaru (`docs/07`, rozdz. 9) | Wartość w zielonym | w żółtym | poza żółtym |
|---|---|---|---|
| `good` | `ok` | `near` | `out` |
| `warn` | `ok` | `near` + flaga „sprawdź kolejnym nagraniem” | `out` + flaga |
| `bad` / `na` | `unknown` | `unknown` | `unknown` |
| warunek „nie oceniono” z karty reguły (rozdz. 1.1) | `unknown` | `unknown` | `unknown` |
| zaakceptowana przez użytkownika | `accepted` (niezależnie od wartości) | | |

Teksty statusów: `ok` „W porządku”, `near` „Blisko granicy”, `out` „Warto zmienić”, `unknown` „Nie oceniono”,
`accepted` „Twoje ustawienie”. Dla `near`/`out` dodatkowo kierunek: np. „Lekko za daleko”, „Siodło nisko”.
Zasady języka i ikony: `docs/21`.

## 4. Kierunek i akcja

| Regulacja | Wartość < zielony | Wartość > zielony |
|---|---|---|
| `saddle_height` | siodło za wysoko → **Obniż siodło** | siodło za nisko → **Podnieś siodło** |
| `saddle_fore_aft` | kolano daleko za pedałem → **Przesuń siodło do przodu** | kolano przed pedałem → **Przesuń siodło do tyłu** |
| `bar_reach` | łokcie mocno zgięte, za blisko → **Wydłuż mostek** (alternatywa: klamkomanetki do przodu) | łokcie prawie proste, za daleko → **Skróć mostek** (alternatywa: klamkomanetki do tyłu) |
| `bar_height` | plecy za nisko → **Podnieś kierownicę** (podkładki pod mostek / odwrócenie mostka) | plecy wysoko → **Możesz obniżyć kierownicę** (tylko jeśli brak modyfikatora `M_LOW_BACK`) |

Wielkość zmiany (mm) dla reguł `validated` wyznacza się z **wariantów** (`docs/10`, rozdz. 4); dla reguł
`experimental` jest to stały krok `step_experimental_mm` (rozdz. 1.1): najmniejszy wariant danego parametru,
który przenosi pomiar do celu lub najbliżej celu; zaokrąglenie do 5 mm; ograniczenie do kroku maks. (10 mm;
5 mm przy `safety_level = caution` – rozdz. 5.1).
`amount_total_mm` = wartość potrzebna do celu (informacja „łącznie może być potrzebne ok. X mm”).

Parametr roweru dla każdej akcji: `saddle_height_mm`, `saddle_setback_mm`, `stem_length_mm`
(alternatywnie `hoods_offset_mm`), `spacers_mm` (alternatywnie `stem_angle_deg`).

## 5. Modyfikatory z wywiadu

| Kod | Warunek | Efekt | Tekst w S06 (skrót) | Poziom dowodu |
|---|---|---|---|---|
| `M_LESS_FLEX` | `flex.less_flexible = true` | `bar_height`: zakres +5°; `saddle_height`: dolna granica zielonego 33 (nie zalecamy podnoszenia siodła do zgięcia < 33°) | „Napięty tył ud w testach – kierownica może być wyżej; siodła nie podnosimy na siłę” | `practice` + wpływ elastyczności na preferowaną wysokość siodła (Ferrer-Roca i in.); testy domowe – orientacyjne |
| `M_LOW_BACK` | dolegliwość `lower_back` z `intensity ≥ 3` | blokada akcji „obniż kierownicę”; `bar_height` dolna granica +3° | „Ból dołu pleców – nie zalecimy obniżenia kierownicy” | `science` (asocjacja: niższa kierownica ↔ większe zgięcie lędźwiowe; przegląd 2017) |
| `M_KNEE_FRONT` | `knee_front` `intensity ≥ 3` | `saddle_height` i `saddle_fore_aft` zawsze pierwsze; przy `knee_flex_bottom > 38` status co najmniej `near` | „Ból przodu kolana – najpierw siodło” | `practice` |
| `M_KNEE_BACK` | `knee_back` `intensity ≥ 3` | przy `knee_flex_bottom < 32` status co najmniej `near` (kierunek: obniż) | „Ból tyłu kolana – sprawdzimy, czy siodło nie jest za wysoko” | `practice` |
| `M_UPPER_BODY` | `neck`, `shoulders` lub `hands_numb` `intensity ≥ 3` | blokada akcji „wydłuż mostek” i „obniż kierownicę”; `bar_reach` i `bar_height` przed `saddle_fore_aft` | „Kark, barki, dłonie – nie wydłużymy ani nie obniżymy kierownicy” | `practice` |
| `M_SADDLE` | `saddle` `intensity ≥ 3` | porada informacyjna: szerokość siodła vs `body.sit_bone_cm`, kąt siodła | „Ucisk na siodle – sprawdź szerokość i kąt siodła” | `practice` |
| `M_PERINEAL` | `saddle_numb` `intensity ≥ 3` lub `frequency` ∈ {`half`, `always`} | blokada akcji „obniż kierownicę”; porada `tip.perineal` (kąt siodła 0…−3°, szerokość vs kości kulszowe, siodło z wycięciem lub skróconym noskiem, wstawanie z siodła co 10–15 min) | „Drętwienie krocza – nie obniżymy kierownicy; sprawdź siodło” | `science` (siodło bez noska/z wycięciem zmniejsza nacisk i drętwienie – metaanaliza 2020) + `practice` |
| `M_LEG_DIFF` | `body.leg_length_diff = yes` | flaga dla fittera; brak automatycznych zmian | „Krótsza noga – do potwierdzenia u fittera” | `assumption` |
| `M_ANKLE_LIMITED` | `flex.ankle = very_limited` | informacja w szczegółach (kąt stawu skokowego) | – | `assumption` |
| `M_ASYMMETRY` | `flex.asymmetry = true` lub główna dolegliwość jednostronna po stronie innej niż `session.side` | porada `tip.record_other_side`: „Nagraj też drugą stronę” (C-22) | – | `assumption` |
| `M_JOINT_REPLACEMENT` | `surgery[].type` ∈ {`knee_replacement`, `hip_replacement`} | blokada akcji „obniż kierownicę”; porada o długości korby przy niższym progu (rozdz. 6.3); flaga dla fittera | „Endoproteza – nie zamykamy bardziej kąta biodra” | `practice` |

Modyfikatory z testów mobilności (`M_LESS_FLEX`, `M_ANKLE_LIMITED`, `M_ASYMMETRY`) tylko przesuwają zakresy lub
dodają informację – **nigdy nie blokują akcji** (C-02). Dawny modyfikator `M_INJURY` zastąpił poziom
bezpieczeństwa (rozdz. 5.1).

### 5.1 Poziom bezpieczeństwa (C-03)

`safety_level` wylicza PHP z wywiadu (`docs/04`, rozdz. 5.1) i przekazuje w `job.json` (`rider.safety`). Python
stosuje skutki z sekcji `safety` pliku modelu:

| Poziom | Statusy i pomiary | Zalecenia | Następny krok | S12 |
|---|---|---|---|---|
| `none` | tak | tak | pierwsze zalecenie | pełny |
| `caution` | tak | tak, `amount_mm` ≤ `safety.caution_step_mm` (5); tekst „Małymi krokami” | pierwsze zalecenie + komunikat | pełny |
| `stop` | tak | **brak** (`recommendations = []`) | karta „Najpierw konsultacja” (`docs/21`, rozdz. 5) | tylko podgląd, bez „Zapisz plan” |

Zmiana odpowiedzi bezpieczeństwa (nowy wywiad krótki/pełny) zleca `refit` z nowym poziomem. Poziom dowodu:
`practice` (ścieżki triażu MSK); lista i progi do zatwierdzenia przez lekarza (D-15).

### 5.2 Dopasowanie podstawowe (C-11)

Przy `fit_level = basic` (szybki start, brak zgody `health_data`) modyfikatory mobilności są nieaktywne,
a pewność każdego zalecenia obniża się o jeden poziom (`conf.basic_fit`).

Porady informacyjne (bez statusu regulacji, sekcja „Inne wskazówki”):
- szerokość siodła ≈ `body.sit_bone_cm` + 20–30 mm (szosa) lub + 30–40 mm (pozycja wyprostowana) – `practice`,
- szerokość kierownicy ≈ `body.shoulder_width_cm` ± 2 cm – `practice`,
- długość korby – rozdz. 6.3; drętwienie krocza – `M_PERINEAL`; nagranie drugiej strony – `M_ASYMMETRY`.

### 5.3 Kolejność stosowania warstw i ich łączenie

Zakres i akcje regulacji powstają zawsze w tej kolejności; każdy krok zapisuje się w śladzie uzasadnienia (rozdz. 12):

1. zakres bazowy (`ranges[bike_type][profile]`, z `method` i strefą sporną);
2. **wymiary ciała nie zmieniają zakresów kątowych** – kąty mierzone w ruchu już zawierają proporcje ciała;
   wymiary służą do kontroli skali i segmentów, porad (siodło, kierownica) i modelu sylwetki (`docs/10`);
3. modyfikatory mobilności – tylko przesunięcia zakresu;
4. modyfikatory dolegliwości – blokady kierunku, zawężenia, priorytet;
5. **koperta literatury:** wynikowy zielony zakres przycina się do żółtego zakresu bazowego (modyfikatory nie
   wyprowadzą zakresu poza literaturę); przesunięcia tego samego zakresu łączą się przez **maksimum**, nie sumę;
6. **pusty zakres** (np. `M_KNEE_FRONT` wymaga ≤ 38°, `M_KNEE_BACK` ≥ 32° i dolna granica przekracza górną) →
   status `unknown`, brak zalecenia, flaga `conflicting_requirements` „do oceny przez fittera”;
7. akceptacja „Mi tak wygodnie”; 8. poziom bezpieczeństwa (rozdz. 5.1); 9. status reguły (rozdz. 1.1).

## 6. Następny krok i lista zaleceń

1. Odrzuć regulacje `ok`, `unknown`, `accepted` i reguły `measure_only`.
2. Kolejność ustala **plan sekwencyjny** (rozdz. 11); przy jednej regulacji `experimental` (etap M4) – ta
   regulacja.
3. Akcje zablokowane przez modyfikatory nie wchodzą do planu (regulacja pokazuje status i tekst
   „Nie zalecamy zmiany z powodu …”).
4. Pierwszy krok planu = **następny krok** (zawsze jeden). Pozostałe: „Kolejne kroki – wstępnie, po nagraniu
   kontrolnym przeliczymy”.

Teksty (`text_key` + parametry): np. `rec.saddle_height.raise` → „Podnieś siodło o ok. {mm} mm”, uzasadnienie
`why.saddle_height.high` → „Na dole obrotu kolano zgina się bardziej niż w zalecanym zakresie. Wyższe siodło
zmniejszy to zgięcie.”, sposób `how.saddle_height` → „Poluzuj obejmę sztycy, przesuń sztycę i zaznacz nową
wysokość taśmą. Zmieniaj najwyżej o 5–10 mm naraz.” Pełna lista tekstów w pliku modelu (`texts`), język PL,
zgodnie z `docs/21` (bez przypisywania przyczyny dolegliwości).

Każde zalecenie zawiera dodatkowo `confidence` z powodami (rozdz. 6.1), `effort` (rozdz. 6.2), `rule_status`,
skutki uboczne dla pozostałych regulacji (rozdz. 11) i ślad uzasadnienia (rozdz. 12). Przy `experimental`
wielkość = `step_experimental_mm` (bez `amount_total_mm`), pewność maks. `medium`.

### 6.1 Poziom pewności (C-06)

Start: `high`. Każdy spełniony warunek obniża o jeden poziom (minimum `low`); wsparcie podnosi o jeden (maksimum
`high`). Powody zapisywane jako lista kodów `confidence_reasons` (teksty `conf.*` w pliku modelu):

| Kod | Warunek | Wpływ |
|---|---|---|
| `conf.quality_warn` | pomiar sterujący ma jakość `warn` | −1 |
| `conf.no_markers` | `markers_mode = none` | −1 |
| `conf.near_boundary` | odległość wartości od najbliższej granicy zielonego < 2 × SD pomiaru | −1 |
| `conf.evidence_assumption` | zakres regulacji ma poziom dowodu `assumption` | −1 |
| `conf.approximate_geometry` | warianty `approximate` (brak geometrii ramy) | −1 |
| `conf.scale_estimated` | `W_SCALE_ESTIMATED` i regulacja mierzona w mm (`saddle_fore_aft`) | −1 |
| `conf.basic_fit` | `fit_level = basic` | −1 |
| `conf.high_effort` | `W_HIGH_EFFORT` (nagranie przy RPE ≥ 7) | −1 |
| `conf.pelvis_rock_support` | „obniż siodło” i `pelvis_vertical.rom > signals.pelvis_rock_mm` (C-23) | +1 |
| `conf.rule_experimental` | `rule_status = experimental` | pewność maks. `medium` (ograniczenie, nie odjęcie) |

Przy `low` interfejs proponuje nagranie kontrolne przed zmianą (`docs/21`, rozdz. 3). Progi `signals.*` robocze
(D-15).

### 6.2 Wysiłek zmiany (C-07)

Sekcja `effort` pliku modelu, dla każdej akcji: `minutes`, `tools` (lista kodów `tool.*` z tekstami),
`difficulty` (`easy`/`medium`/`hard`), `shop_recommended`.

| Akcja | Czas | Narzędzia | Trudność | Serwis |
|---|---|---|---|---|
| wysokość siodła | 10 min | klucz imbusowy 4–5 mm, taśma, miarka | `easy` | nie |
| siodło przód/tył | 10 min | klucz imbusowy 4–6 mm, miarka | `easy` | nie |
| długość mostka | 30 min | nowy mostek, klucz imbusowy, klucz dynamometryczny | `medium` | tak |
| klamkomanetki przód/tył | 45 min | nowa owijka, klucz imbusowy 5 mm | `hard` | tak |
| podkładki / odwrócenie mostka | 20 min | klucz imbusowy, klucz dynamometryczny | `medium` | tak |
| długość korby | 60 min | nowe korby, narzędzia do suportu | `hard` | tak |

Wartości robocze (`practice`).

### 6.3 Długość korby i sygnały pomocnicze (C-09, C-23)

- `tip.crank_length` – gdy `knee_flex_top.med ≥ signals.knee_flex_top_high_deg` (roboczo 115°; przy
  `M_JOINT_REPLACEMENT` – `signals.knee_flex_top_replacement_deg`, roboczo 110°) albo
  `hip_angle.min < signals.hip_closed_deg` (roboczo 45°), oraz dolegliwość `knee_front` lub `hip` z
  `intensity ≥ 3` albo `M_JOINT_REPLACEMENT`. Tekst: „Rozważ krótsze korby (o 2,5–5 mm) – zmniejszają zgięcie
  kolana i biodra w górze obrotu; zwykle trzeba wtedy podnieść siodło o tyle samo. Sprawdź wirtualnie.” Link do
  wariantu `crank_length_mm` (`docs/10`, rozdz. 6). Poziom dowodu: `science` dla kierunku zmian kinematyki
  (Ferrer-Roca i in.; Reynolds i in. 2025), `assumption` dla progów.
- `pelvis_rock` – `pelvis_vertical.rom > signals.pelvis_rock_mm` (roboczo 20 mm): podnosi pewność „obniż siodło”;
  przy statusie `ok` wysokości siodła – tylko flaga w szczegółach („duży ruch miednicy w pionie”).

## 7. Akceptacja „Mi tak wygodnie”

- Zapis: `fit_acceptance` (użytkownik, rower, regulacja, powód, notatka, data, `revoked_at`).
- Powody: `anatomy` budowa ciała / proporcje · `injury` przebyty uraz lub operacja · `leg_length` różnica długości
  nóg · `mobility` ograniczona ruchomość · `comfortable` po prostu wygodnie.
- Skutek: status `accepted`, brak zalecenia, regulacja nie liczy się do „X z 4” jako błąd (liczy się jako
  „Twoje ustawienie”).
- Wygaśnięcie: użytkownik cofa ręcznie albo w nowym wywiadzie pojawia się dolegliwość związana z regulacją
  (`saddle_height`/`saddle_fore_aft` ↔ kolano, biodro; `bar_*` ↔ kark, barki, dłonie, dół pleców) z
  `intensity ≥ 3` → zapytanie „Czy nadal jest Ci wygodnie?” zamiast automatycznego cofnięcia.

## 8. Zmiana profilu jazdy bez nowego nagrania

Zmiana profilu lub akceptacji tworzy zadanie `refit` (Python, ~1 s): te same pomiary i warianty, nowe zakresy,
statusy i zalecenia; nowy rekord `fit_analysis` z `parent_id` i `kind = 'refit'`. PHP niczego nie przelicza.

## 9. Ocena zmiany (S14)

Dla zmiany `fit_change` (lub grupy z tym samym `setup_after_id`) i pary analiz: przed (`before`, ostatnia bieżąca
na `setup_before_id`) i po (`after`, pierwsza na `setup_after_id`):

| Kryterium | Kod | Reguła | Wynik |
|---|---|---|---|
| Zmiana wykonana zgodnie z zaleceniem | `executed_as_planned` | `|Δ_zmierzona − Δ_przewidziana| ≤ max(2°, SD_before)` dla pomiaru sterującego regulacji (Δ przewidziana z wariantu) | yes / partial (≤ 2× tolerancji) / no |
| Ustawienie w zakresie | `in_range` | status `after` = `ok` (lub `near` → partial) | yes / partial / no |
| Odczucia | `feels_better` | dolegliwość docelowa = główna (`is_primary`) sprzed zmiany. **yes**: spadek siły o ≥ 2 pkt lub ≥ 30% (Farrar i in. 2001), a bez dolegliwości – ≥ 50% ocen po jeździe „Lepiej”/„Dużo lepiej”; **partial**: spadek o 1 pkt (< 30%) lub więcej ocen pozytywnych niż negatywnych; **no**: pozostałe | yes / partial / no |
| Nagrania porównywalne | `comparable` | ta sama strona i chwyt; skala ±10%; kadencja ±10 obr./min; RPE ±2 (jeśli podano w obu); jakość obu ≥ `warn` | yes / no |
| Jedna zmiana naraz | `single_change` | jeden parametr w grupie zmian | yes / no |

Werdykt (pierwsza pasująca reguła):
1. `too_early` „Za wcześnie na ocenę” – `rides_since_change < 3` (z wywiadu krótkiego lub ocen po jeździe).
2. `worse` „Gorzej – wróć do poprzedniego” – wzrost siły dolegliwości docelowej o ≥ 2 pkt lub ≥ 30%, nowa dolegliwość o sile ≥ 3,
   większość ocen po jeździe „Gorzej”/„Dużo gorzej”, albo pomiar oddalił się od zakresu.
3. `effective` „Pomogło” – `in_range = yes` i `feels_better = yes`.
4. `partial` „Częściowo” – dokładnie jedno z `in_range`, `feels_better` = yes (lub oba partial).
5. `no_effect` „Bez zmian” – pozostałe (etykiety: `docs/21`, rozdz. 2).
Gdy `comparable = no` – werdykt z dopiskiem „ocena orientacyjna” i propozycja ponownego nagrania.
Przy `worse` – zalecenie „Wróć do poprzedniego ustawienia” (wartość z `setup_before`).
Ocena po jeździe ma 5 stopni (`much_better`, `better`, `same`, `worse`, `much_worse` – globalna ocena zmiany,
Kamper i in. 2009) i opcjonalnie siłę dolegliwości docelowej 0–10 (`docs/03`, rozdz. 3 i 16).

## 10. Format pliku modelu

```json
{
  "model_version": "fit-model-v1",
  "valid_from": "2027-01-01",
  "regulations": {
    "saddle_height": {
      "metric": "knee_flex_bottom", "rep": "med", "order": 1, "step_max_mm": 10, "step_experimental_mm": 5,
      "rule_status": "experimental", "min_quality": "good", "min_cycles": 20,
      "min_margin": { "abs": 2, "sd_factor": 1 }, "unknown_if": ["W_SEGMENT_MISMATCH:thigh", "W_SEGMENT_MISMATCH:shank"],
      "param": "saddle_height_mm", "evidence": "science", "source_ids": ["ferrer_roca_2012", "bini_2011"],
      "method": "dynamic_2d_lateral_markers",
      "ranges": { "*": { "*": { "green": [30, 40], "yellow": [25, 45], "target": 35, "disputed": [] } } },
      "actions": { "below": "rec.saddle_height.lower", "above": "rec.saddle_height.raise" },
      "validation": { "metric_tolerance_deg": 2, "prediction_tolerance_deg": 2, "fitter_agreement_min": 0.8,
                      "worse_max": 0.10 }
    }
  },
  "modifiers": [
    { "code": "M_LESS_FLEX", "when": { "flex.less_flexible": true },
      "effects": [ { "regulation": "bar_height", "shift_range_deg": 5 },
                   { "regulation": "saddle_height", "green_low": 33 } ],
      "text_key": "mod.less_flex" }
  ],
  "safety": { "caution_step_mm": 5, "stop_blocks_recommendations": true },
  "confidence": { "near_boundary_sd": 2, "rules": ["conf.quality_warn", "conf.no_markers", "conf.near_boundary",
                  "conf.evidence_assumption", "conf.approximate_geometry", "conf.scale_estimated", "conf.basic_fit",
                  "conf.high_effort", "conf.pelvis_rock_support"] },
  "signals": { "knee_flex_top_high_deg": 115, "knee_flex_top_replacement_deg": 110, "hip_closed_deg": 45,
               "pelvis_rock_mm": 20 },
  "effort": { "rec.saddle_height.raise": { "minutes": 10, "tools": ["tool.hex_4_5", "tool.tape"], "difficulty": "easy",
                                           "shop_recommended": false } },
  "change_evaluation": { "min_rides": 3, "angle_tolerance_deg": 2, "scale_tolerance": 0.10, "cadence_tolerance_rpm": 10,
                         "rpe_tolerance": 2, "pain_mcid_points": 2, "pain_mcid_pct": 30, "new_complaint_min": 3 },
  "texts": { "rec.saddle_height.raise": "Podnieś siodło o ok. {mm} mm" },
  "sources": { "ferrer_roca_2012": "Ferrer-Roca V. i in., J Strength Cond Res 2012; 26(11): 3025–3029" }
}
```

Sekcja `plan`: `{ "max_steps": 4, "base_order": ["saddle_height", "saddle_fore_aft", "bar_reach", "bar_height"] }`.

Klucze `ranges`: `ranges[bike_type][profile]`, `*` = dowolny. Plik jest walidowany schematem
`schemas/fit-model.schema.json`. Zmiana pliku = nowa `model_version`; stare wyniki zachowują swoją wersję.

## 11. Zależności między regulacjami i plan zmian

Zmiana jednego parametru wpływa na inne regulacje. Przykład z modelu 2D (wartości indywidualne liczy serwer
z wariantów, `docs/10`, rozdz. 6):

| Zmiana o 10 mm | Zgięcie kolana | Kolano względem pedału | Łokieć | Plecy |
|---|---|---|---|---|
| siodło w górę (wzdłuż rury, kąt 73,5°) | −3…−5° | −3 mm (siodło cofa się o ok. 2,8 mm) | +2…+4° | ~0* |
| siodło do przodu | +0,5…1° | +10 mm | −5…−10° | ~0* |
| mostek krótszy | 0 | 0 | −3…−5° | +0,5…1° |
| kierownica wyżej | 0 | 0 | ok. −1° | ok. +1° (założenie `torso_follow`) |

\* model zakłada stałe pochylenie pleców, dopóki dłonie sięgają kierownicy (`docs/10`, rozdz. 4 – założenie).

**Macierz wpływu** `M[param][metric]` = różnica pomiaru wariantu i stanu bieżącego, liczona dla każdej analizy.
Jest prawie trójkątna: siodło wpływa na wszystkie regulacje, kierownica tylko na górę ciała. To uzasadnia kolejność
„od podstawy w górę” – najpierw to, od czego zależy reszta, aby nie poprawiać dwa razy tego samego.

**Plan sekwencyjny** (etap M5; liczony w `analyze`, `correct`, `refit`):
1. Stan `S0` = zmierzony.
2. Kandydaci: akcje regulacji `out`/`near` o statusie reguły ≥ `experimental`, niezablokowane, dozwolone przy
   poziomie bezpieczeństwa.
3. Odrzuć akcje, które przewidywanie wypycha inną regulację ze statusu `ok` do `out`.
4. Wybierz regulację najbliżej podstawy (kolejność bazowa) ze statusem `out`, potem `near`. Dolegliwość podnosi
   priorytet regulacji z nią związanej tylko w obrębie tego samego poziomu podstawy (nie wyprzedza `out` niższej
   regulacji, chyba że jej akcja jest zablokowana).
5. Spośród akcji tej regulacji wybierz kolejno: poprawiającą przy okazji inną regulację `out`/`near`, z najmniejszymi
   skutkami ubocznymi, z najmniejszym wysiłkiem (rozdz. 6.2).
6. `S(k+1)` = przewidywanie po kroku (wariant; dla kolejnych kroków – `simulate` łącznie z poprzednimi); powtórz
   od 2. Maks. 4 kroki.
7. Jedna zmiana na krok; korekta skutku ubocznego jest **osobnym** krokiem. Pewny jest tylko krok 1 – kolejne
   oznaczone „wstępnie”; po nagraniu kontrolnym plan liczy się od nowa ze zmierzonego stanu.

Przykład (kolano 42°, kolano 50 mm za pedałem, łokieć 168°, ból dołu pleców): lista statyczna dałaby siodło +10 →
siodło do przodu → **krótszy mostek** → kierownica wyżej; plan sekwencyjny po kroku 2 (siodło do przodu) przewiduje
łokieć ok. 161° w zakresie, więc **pomija zakup mostka**.

## 12. Ślad uzasadnienia (`rationale`)

Każde zalecenie i każda regulacja `unknown` zapisuje uporządkowany ślad (pokazywany w „Szczegółach”, skrót w „Dlaczego?”):

| Pole | Przykład |
|---|---|
| `measurement` | zgięcie kolana 42° (SD 1,4; jakość dobra; 41 cykli; pomiar zwalidowany – markery) |
| `base_range` | 30–40°, `method` dynamic_2d_lateral_markers, źródła `ferrer_roca_2012`, `bini_2011`, `science` |
| `layers` | profil: bez zmian → `M_LESS_FLEX`: dolna granica 33 → koperta: bez przycięcia |
| `decision` | poza zakresem o 2° (> `min_margin` 1,4°) → siodło nisko |
| `side_effects` | kolano względem pedału −3 mm (nadal w zakresie), łokieć +3° |
| `amount` | +5 mm (`experimental`) / +10 mm z wariantu (`validated`) |
| `order` | krok 1 – siodło wpływa na pozostałe regulacje |
| `confidence` | `medium` – reguła eksperymentalna |
| `model` | `fit-model-v1`, `rule_status` experimental |

## 13. Karta modelu i rada modelu

Każda wersja modelu ma **kartę modelu** (wzorzec: Mitchell i in. 2019) w `docs/model-cards/{model_version}.md`,
tworzoną w P1.9 i aktualizowaną po każdej walidacji:
- zakresy, metody i źródła; strefy sporne; zmiany względem poprzedniej wersji;
- status każdej reguły i dane, na których go nadano;
- wyniki trzech poziomów poprawności: **pomiar** (kryteria `docs/16` rozdz. 6.1, z markerami i bez),
  **przewidywanie** (różnice przewidziane − zmierzone), **zalecenie** (zasadność wg fittera, odsetek „pomogło” /
  „gorzej” dla każdej reguły);
- znane ograniczenia (2D, jedna strona, założenia modelu sylwetki, testy domowe);
- kto zatwierdził (rada modelu: bike fitter + fizjoterapeuta, D-06) i kiedy.
Wynik zawiera `fit.model_version`, więc każde zalecenie odsyła do karty (link w „Szczegółach”).
