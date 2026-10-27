# 09 · Model dopasowania (warstwa E)

Model dopasowania zamienia pomiary (`docs/08`) i wywiad (`docs/04`) na statusy 4 regulacji, zalecenia i ocenę zmian.
Jest **konfiguracją** (`analyzer/configs/fit-models/fit-model-v1.json`) interpretowaną przez pakiet
`ridemore_fit.fit`. Progi i teksty nie mogą być zapisane w kodzie PHP ani w szablonach.

**Wszystkie zakresy w v1 są robocze.** Każdy ma poziom dowodu: `science` (publikacja), `practice` (praktyka bike
fittingu, do weryfikacji), `assumption` (założenie projektu). Przed produkcją zatwierdza je bike fitter (`docs/18`, D-06).

## 1. Regulacje

| Kod | Nazwa (UI) | Pomiar sterujący | Kolejność | Krok maks. | Poziom dowodu |
|---|---|---|---|---|---|
| `saddle_height` | Wysokość siodła | `knee_flex_bottom.med` | 1 | 10 mm | `science` |
| `saddle_fore_aft` | Siodło przód / tył | `knee_over_pedal.at_p90` | 2 | 10 mm | `practice` (KOPS – punkt wyjścia, nie norma) |
| `bar_reach` | Odległość do kierownicy | `elbow_angle.med` (pomocniczo `shoulder_angle.med`) | 3 | 10 mm | `practice` |
| `bar_height` | Wysokość kierownicy | `torso_angle.med` | 4 | 10 mm | `practice` |

## 2. Zakresy bazowe

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

Wielkość zmiany (mm) wyznacza się z **wariantów** (`docs/10`, rozdz. 4): najmniejszy wariant danego parametru,
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

## 6. Następny krok i lista zaleceń

1. Odrzuć regulacje `ok`, `unknown`, `accepted`.
2. Posortuj: najpierw `out`, potem `near`; w grupie wg kolejności z rozdz. 1, zmienionej przez modyfikatory.
3. Usuń akcje zablokowane przez modyfikatory (wtedy regulacja pokazuje status, ale zamiast akcji tekst
   „Nie zalecamy zmiany z powodu …”).
4. Pierwsza pozycja = **następny krok** (zawsze jedna). Pozostałe: „Kolejne kroki – po nagraniu kontrolnym”.

Teksty (`text_key` + parametry): np. `rec.saddle_height.raise` → „Podnieś siodło o ok. {mm} mm”, uzasadnienie
`why.saddle_height.high` → „Na dole obrotu kolano zgina się bardziej niż w zalecanym zakresie. Wyższe siodło
zmniejszy to zgięcie.”, sposób `how.saddle_height` → „Poluzuj obejmę sztycy, przesuń sztycę i zaznacz nową
wysokość taśmą. Zmieniaj najwyżej o 5–10 mm naraz.” Pełna lista tekstów w pliku modelu (`texts`), język PL,
zgodnie z `docs/21` (bez przypisywania przyczyny dolegliwości).

Każde zalecenie zawiera dodatkowo `confidence` z powodami (rozdz. 6.1) i `effort` (rozdz. 6.2).

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
      "metric": "knee_flex_bottom", "rep": "med", "order": 1, "step_max_mm": 10,
      "param": "saddle_height_mm", "evidence": "science", "source_ids": ["ferrer_roca_2012", "bini_2011"],
      "ranges": { "*": { "*": { "green": [30, 40], "yellow": [25, 45], "target": 35 } } },
      "actions": { "below": "rec.saddle_height.lower", "above": "rec.saddle_height.raise" }
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

Klucze `ranges`: `ranges[bike_type][profile]`, `*` = dowolny. Plik jest walidowany schematem
`schemas/fit-model.schema.json`. Zmiana pliku = nowa `model_version`; stare wyniki zachowują swoją wersję.
