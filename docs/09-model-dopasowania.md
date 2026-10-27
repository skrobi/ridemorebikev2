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

Teksty statusów: `ok` „W porządku”, `near` „Blisko granicy”, `out` „Do zmiany”, `unknown` „Nie oceniono”,
`accepted` „Twoje ustawienie”. Dla `near`/`out` dodatkowo kierunek: np. „Lekko za daleko”, „Siodło za nisko”.

## 4. Kierunek i akcja

| Regulacja | Wartość < zielony | Wartość > zielony |
|---|---|---|
| `saddle_height` | siodło za wysoko → **Obniż siodło** | siodło za nisko → **Podnieś siodło** |
| `saddle_fore_aft` | kolano daleko za pedałem → **Przesuń siodło do przodu** | kolano przed pedałem → **Przesuń siodło do tyłu** |
| `bar_reach` | łokcie mocno zgięte, za blisko → **Wydłuż mostek** (alternatywa: klamkomanetki do przodu) | łokcie prawie proste, za daleko → **Skróć mostek** (alternatywa: klamkomanetki do tyłu) |
| `bar_height` | plecy za nisko → **Podnieś kierownicę** (podkładki pod mostek / odwrócenie mostka) | plecy wysoko → **Możesz obniżyć kierownicę** (tylko jeśli brak modyfikatora `M_LOW_BACK`) |

Wielkość zmiany (mm) wyznacza się z **wariantów** (`docs/10`, rozdz. 4): najmniejszy wariant danego parametru,
który przenosi pomiar do celu lub najbliżej celu; zaokrąglenie do 5 mm; ograniczenie do kroku maks. (10 mm).
`amount_total_mm` = wartość potrzebna do celu (informacja „łącznie może być potrzebne ok. X mm”).

Parametr roweru dla każdej akcji: `saddle_height_mm`, `saddle_setback_mm`, `stem_length_mm`
(alternatywnie `hoods_offset_mm`), `spacers_mm` (alternatywnie `stem_angle_deg`).

## 5. Modyfikatory z wywiadu

| Kod | Warunek | Efekt | Tekst w S06 (skrót) | Poziom dowodu |
|---|---|---|---|---|
| `M_LESS_FLEX` | `flex.less_flexible = true` | `bar_height`: zakres +5°; `saddle_height`: dolna granica zielonego 33 (nie zalecamy podnoszenia siodła do zgięcia < 33°) | „Napięty tył ud – kierownica może być wyżej; siodła nie podnosimy na siłę” | `practice` + wpływ elastyczności na preferowaną wysokość siodła (Ferrer-Roca i in.) |
| `M_LOW_BACK` | dolegliwość `lower_back` z `intensity ≥ 3` | blokada akcji „obniż kierownicę”; `bar_height` dolna granica +3° | „Ból dołu pleców – nie zalecimy obniżenia kierownicy” | `science` (asocjacja: niższa kierownica ↔ większe zgięcie lędźwiowe; przegląd 2017) |
| `M_KNEE_FRONT` | `knee_front` `intensity ≥ 3` | `saddle_height` i `saddle_fore_aft` zawsze pierwsze; przy `knee_flex_bottom > 38` status co najmniej `near` | „Ból przodu kolana – najpierw siodło” | `practice` |
| `M_KNEE_BACK` | `knee_back` `intensity ≥ 3` | przy `knee_flex_bottom < 32` status co najmniej `near` (kierunek: obniż) | „Ból tyłu kolana – sprawdzimy, czy siodło nie jest za wysoko” | `practice` |
| `M_UPPER_BODY` | `neck`, `shoulders` lub `hands_numb` `intensity ≥ 3` | blokada akcji „wydłuż mostek” i „obniż kierownicę”; `bar_reach` i `bar_height` przed `saddle_fore_aft` | „Kark, barki, dłonie – nie wydłużymy ani nie obniżymy kierownicy” | `practice` |
| `M_SADDLE` | `saddle` `intensity ≥ 3` | porada informacyjna: szerokość siodła vs `body.sit_bone_cm`, kąt siodła | „Ucisk w kroczu – sprawdź szerokość i kąt siodła” | `practice` |
| `M_LEG_DIFF` | `body.leg_length_diff = yes` | flaga dla fittera; brak automatycznych zmian | „Krótsza noga – do potwierdzenia u fittera” | `assumption` |
| `M_ANKLE_LIMITED` | `flex.ankle = very_limited` | informacja w szczegółach (kąt stawu skokowego) | – | `assumption` |
| `M_ASYMMETRY` | `flex.asymmetry = true` | informacja: zalecenie nagrania drugiej strony | – | `assumption` |
| `M_INJURY` | `injury_recent = true` lub `intensity ≥ 7` | komunikat o konsultacji; zalecenia pokazywane z ostrzeżeniem | – | `assumption` |

Porady informacyjne (bez statusu regulacji, sekcja „Inne wskazówki”):
- szerokość siodła ≈ `body.sit_bone_cm` + 20–30 mm (szosa) lub + 30–40 mm (pozycja wyprostowana) – `practice`,
- szerokość kierownicy ≈ `body.shoulder_width_cm` ± 2 cm – `practice`.

## 6. Następny krok i lista zaleceń

1. Odrzuć regulacje `ok`, `unknown`, `accepted`.
2. Posortuj: najpierw `out`, potem `near`; w grupie wg kolejności z rozdz. 1, zmienionej przez modyfikatory.
3. Usuń akcje zablokowane przez modyfikatory (wtedy regulacja pokazuje status, ale zamiast akcji tekst
   „Nie zalecamy zmiany z powodu …”).
4. Pierwsza pozycja = **następny krok** (zawsze jedna). Pozostałe: „Kolejne kroki – po nagraniu kontrolnym”.

Teksty (`text_key` + parametry): np. `rec.saddle_height.raise` → „Podnieś siodło o ok. {mm} mm”, uzasadnienie
`why.saddle_height.high` → „Na dole obrotu noga jest za mocno zgięta. Zwykle przeciąża to przód kolana…”,
sposób `how.saddle_height` → „Poluzuj obejmę sztycy, przesuń sztycę i zaznacz nową wysokość taśmą. Zmieniaj
najwyżej o 5–10 mm naraz.” Pełna lista tekstów w pliku modelu (`texts`), język PL.

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
| Odczucia | `feels_better` | intensywność dolegliwości docelowej spadła o ≥ 1 lub ocena „Lepiej”; „Gorzej” lub wzrost ≥ 1 → no | yes / partial / no |
| Nagrania porównywalne | `comparable` | ta sama strona i chwyt; skala ±10%; kadencja ±10 obr./min; jakość obu ≥ `warn` | yes / no |
| Jedna zmiana naraz | `single_change` | jeden parametr w grupie zmian | yes / no |

Werdykt (pierwsza pasująca reguła):
1. `too_early` „Za wcześnie” – `rides_since_change < 3` (z wywiadu krótkiego lub ocen po jeździe).
2. `worse` „Pogorszenie” – `feels_better = no` z oceną „Gorzej” lub wzrostem dolegliwości, albo pomiar oddalił się od zakresu.
3. `effective` „Skuteczna” – `in_range = yes` i `feels_better = yes`.
4. `partial` „Częściowo” – dokładnie jedno z `in_range`, `feels_better` = yes (lub oba partial).
5. `no_effect` „Bez efektu” – pozostałe.
Gdy `comparable = no` – werdykt z dopiskiem „ocena orientacyjna” i propozycja ponownego nagrania.
Przy `worse` – zalecenie „Wróć do poprzedniego ustawienia” (wartość z `setup_before`).

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
  "change_evaluation": { "min_rides": 3, "angle_tolerance_deg": 2, "scale_tolerance": 0.10, "cadence_tolerance_rpm": 10 },
  "texts": { "rec.saddle_height.raise": "Podnieś siodło o ok. {mm} mm" },
  "sources": { "ferrer_roca_2012": "Ferrer-Roca V. i in., J Strength Cond Res 2012; 26(11): 3025–3029" }
}
```

Klucze `ranges`: `ranges[bike_type][profile]`, `*` = dowolny. Plik jest walidowany schematem
`schemas/fit-model.schema.json`. Zmiana pliku = nowa `model_version`; stare wyniki zachowują swoją wersję.
