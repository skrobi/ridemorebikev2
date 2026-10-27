# 08 · Pomiary – definicje geometryczne

Wszystkie pomiary są **rzutem 2D na płaszczyznę obrazu** (płaszczyzna strzałkowa przy prawidłowym ustawieniu
kamery) i dotyczą strony ciała zwróconej do kamery.

## 1. Układy współrzędnych

| Układ | Początek | Osie | Jednostka |
|---|---|---|---|
| obraz | lewy górny róg kadru | x w prawo, y **w dół** | px |
| rower | oś suportu (`bb`) | x w kierunku jazdy, y **w górę**; obrót o `−camera_tilt_deg` | mm (`px / scale_px_per_mm`) |

Konwersja: `x_mm = s·((x−bx)·cos α + (y−by)·sin α)·d`, `y_mm = s·((x−bx)·sin α − (y−by)·cos α)`, gdzie
`(bx, by)` = `bb` w px, `α` = `camera_tilt_deg` (w radianach), `s = 1/scale_px_per_mm`, `d = +1` gdy rower jedzie
w prawo kadru, `d = −1` gdy w lewo (wtedy też odwrócenie x). Kierunek jazdy ustala się z kierunku obrotu toru `mtp5`
(zgodnie z ruchem wskazówek zegara w obrazie ⇒ jazda w prawo). Wszystkie kąty liczone w układzie roweru.

## 2. Funkcje bazowe

- `angle_at(c, a, b)` – kąt wewnętrzny w punkcie `c` między wektorami `c→a` i `c→b`, w stopniach, zakres 0–180:
  `θ = acos( clamp( ((a−c)·(b−c)) / (|a−c|·|b−c|), −1, 1) )`.
- `angle_to_horizontal(p, q)` – kąt odcinka `p→q` względem osi x układu roweru, w stopniach, zakres −90…+90,
  dodatni gdy `q` wyżej niż `p`: `atan2(q.y − p.y, |q.x − p.x|)`.
- Wektor zerowy (pokrywające się punkty) → wartość `null` dla próbki.

## 3. Lista pomiarów

| Kod | Nazwa | Punkty | Definicja | Jedn. | Wartości raportowane |
|---|---|---|---|---|---|
| `knee_angle` | Kąt kolana | `hip`, `knee` (wierzch.), `ankle` | `angle_at(knee, hip, ankle)`; 180° = pełny wyprost | ° | `max`, `min`, `rom` |
| `knee_flex_bottom` | Zgięcie kolana na dole obrotu | jw. | `180 − max(knee_angle)` w cyklu (maks. wyprost występuje zwykle blisko DMP) | ° | `med` (mediana z cykli) |
| `hip_angle` | Kąt biodra | `shoulder`, `hip` (wierzch.), `knee` | `angle_at(hip, shoulder, knee)` | ° | `min` (biodro najbardziej zamknięte, zwykle przy GMP), `max`, `rom` |
| `ankle_angle` | Kąt stawu skokowego | `knee`, `ankle` (wierzch.), `mtp5` | `angle_at(ankle, knee, mtp5)` | ° | `min`, `max`, `rom` |
| `foot_angle` | Pozycja stopy | `heel`, `mtp5` | `−angle_to_horizontal(heel, mtp5)`; dodatnia = przód stopy niżej niż pięta (palce w dół) | ° | `max`, `min`, `rom`, `at_bdc` |
| `torso_angle` | Pochylenie pleców | `hip`, `shoulder` | `angle_to_horizontal(hip, shoulder)`; 0° = poziomo, 90° = pionowo | ° | `med` |
| `shoulder_angle` | Kąt tułów–ramię | `hip`, `shoulder` (wierzch.), `elbow` | `angle_at(shoulder, hip, elbow)` | ° | `med`, `at_p90` |
| `elbow_angle` | Kąt łokcia | `shoulder`, `elbow` (wierzch.), `wrist` | `angle_at(elbow, shoulder, wrist)`; 180° = prosty | ° | `med` |
| `knee_over_pedal` | Kolano względem osi pedału | `knee`, `pedal` | `knee.x − pedal.x` przy fazie 90° (korba poziomo z przodu); ujemne = kolano za osią | mm | `at_p90` |
| `pelvis_vertical` | Ruch miednicy w pionie | `hip` | `y_hip(t) − mean(y_hip)` | mm | `rom` (amplituda), `max`, `min` |
| `pelvis_horizontal` | Ruch miednicy w poziomie | `hip` | `x_hip(t) − mean(x_hip)` | mm | `rom` |
| `hands_load_share` | Udział ciężaru tułowia na dłoniach (poglądowy) | `hip`, `shoulder`, `wrist` | model statyczny `docs/10`, rozdz. 5 | % | `med` |
| `knee_track` | Tor kolana (płaszczyzna czołowa) | – | wymaga kamery z przodu | mm | zawsze `na` w MVP |

Pomiary roweru z obrazu (kontrolne, nie zastępują ustawień wpisanych przez użytkownika): `img_saddle_height_mm`
(odległość `bb` → punkt siodła wskazany na kadrze przez użytkownika, opcjonalnie), `img_crank_radius_mm`
(promień toru `mtp5`, kontrola skali względem `crank_length_mm`, ostrzeżenie przy różnicy > 10%).

## 4. Statystyki

Dla każdego pomiaru `m` i każdego ważnego cyklu `c`:
1. seria `m_c[k]`, k = 0…100 (faza korby 0–100%),
2. wartości cyklu: `max_c`, `min_c`, `rom_c = max_c − min_c`, `med_c` (mediana serii), `at_p90_c` (`k = 25`),
   `at_bdc_c` (`k = 50`),
3. wartość reprezentatywna = **mediana** wartości cykli; zmienność = **odchylenie standardowe** wartości cykli
   (z poprawką n−1),
4. przebieg średni `mean[k]` i `sd[k]` po cyklach (do wykresu w szczegółach).

Liczba cykli `n_cycles` zapisywana przy każdym pomiarze.

## 5. Zależność jakości pomiaru od punktów

| Pomiar | Punkty wymagane | Uwagi o wiarygodności |
|---|---|---|
| `knee_flex_bottom`, `knee_angle` | hip, knee, ankle | najbardziej wiarygodny; czuły na błąd punktu `hip` |
| `hip_angle` | shoulder, hip, knee | czuły na rotację miednicy (niewidoczna w 2D) |
| `ankle_angle`, `foot_angle` | knee, ankle, heel, mtp5 | punkty na bucie; częste zasłonięcia przez korbę |
| `torso_angle` | hip, shoulder | czuły na przechył kamery (`camera_tilt_deg`) |
| `shoulder_angle`, `elbow_angle` | hip/shoulder/elbow/wrist | zależne od chwytu (`grip`) |
| `knee_over_pedal` | knee, pedal (bb + faza) | zależny od skali i dokładności fazy |
| `pelvis_*` | hip | zależny od skali; przechyłu/rotacji miednicy nie da się zmierzyć z jednego punktu |

## 6. Wektory testowe (testy jednostkowe Python i PHP)

| Test | Dane | Oczekiwany wynik |
|---|---|---|
| kąt prosty | `angle_at((0,0),(1,0),(0,1))` | 90,0 |
| wyprost | `angle_at((0,0),(1,0),(−1,0))` | 180,0 |
| 45° do poziomu | `angle_to_horizontal((0,0),(1,1))` | 45,0 |
| odcinek w dół | `angle_to_horizontal((0,0),(1,−1))` | −45,0 |
| kolano | hip (−200, 765), knee (121, 494), ankle (22, 25) [mm, y w górę] | `knee_angle` = 118,25 ± 0,01 |
| tułów | hip (−200, 765), shoulder (248, 1134) | `torso_angle` = 39,48 ± 0,01 |
| KOPS | knee.x = 121, pedal.x = 172,5 | `knee_over_pedal` = −51,5 |
| konwersja obrazu | bb = (500, 800) px, scale = 2 px/mm, tilt = 0, jazda w prawo, punkt (700, 400) px | (100, 200) mm |
| wektor zerowy | `angle_at((0,0),(0,0),(1,0))` | `null` |

Wartości „kolano” i „tułów” policzone z definicji – implementacja ma je odtworzyć w obu językach (PHP potrzebuje ich
tylko do testów kontraktu i ewentualnego przeliczenia profilu – `docs/09`, rozdz. 8).
