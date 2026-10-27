# 10 · Model sylwetki i wirtualne dopasowanie

Model kinematyczny 2D liczony **wyłącznie na serwerze** (pakiet `ridemore_fit.fit.kinematics`). Przeglądarka
dostaje gotowe współrzędne do narysowania i gotowe wartości; niczego nie przelicza.

## 1. Cel

- Przewidzieć pozycję rowerzysty po zmianie jednego parametru roweru (warianty).
- Wyznaczyć wielkość zalecanej zmiany w mm (`docs/09`, rozdz. 4).
- Dostarczyć do S12 sylwetkę „teraz” (zmierzoną) i „po zmianie” (przewidzianą) w pikselach kadrów kluczowych.
- Dostarczyć przewidywaną zmianę pomiaru do oceny zmiany (kryterium „wykonano zgodnie z zaleceniem”).

### 1.1 Trzy rodzaje wyniku – nigdy nie mieszane

| Rodzaj (`kind`) | Co to jest | Pewność | Jak pokazujemy |
|---|---|---|---|
| `geometry` – geometria roweru | nowe położenie punktów kontaktu po zmianie (np. siodło +10 mm wzdłuż rury o kącie 73,5° = +9,6 mm w pionie i 2,8 mm w tył; zmiana drop, odległość siodło–kierownica) | dokładne przy znanej geometrii ramy i poprawnym pomiarze roweru | „Rower po zmianie” – liczby w mm, bez zastrzeżeń poza `approximate` (rozdz. 7) |
| `prediction` – przewidywana pozycja ciała | sylwetka i kąty po zmianie z modelu z jawnymi założeniami (rozdz. 4.1) | przybliżenie; człowiek może zareagować inaczej (przesunąć miednicę, zmienić pracę stopy, ułożenie tułowia) | „Przewidywanie modelu – nie pomiar i nie dowód, że zmiana pomoże”; obrys przerywany |
| `measured` – pomiar po zmianie | wynik nagrania kontrolnego po wykonaniu zmiany | jak każdy pomiar (`docs/07`, rozdz. 9) | „Zmierzone po zmianie” w S14; jedyna podstawa oceny zmiany |

Przewidywanie (`prediction`) służy do: wyboru wielkości kroku dla reguł `validated`, planu sekwencyjnego
(`docs/09`, rozdz. 11) i wizualizacji. Nigdy nie jest przedstawiane jako potwierdzenie korzyści biomechanicznej;
ocena zmiany opiera się wyłącznie na `measured` i odczuciach (`docs/09`, rozdz. 9). Trafność przewidywania jest
mierzona (rozdz. 10) i raportowana w karcie modelu.

Parametry `sim.*` (np. `torso_follow_deg_per_mm`, `reach_margin_mm`, `min_reach_mm`, `default_hip_offset_mm`,
`default_hand_offset_mm`, `stem_height_mm`, siatka wariantów) są w pliku modelu dopasowania, sekcja `simulation`
(`docs/09`, rozdz. 10) – zmiana = nowa wersja modelu.

## 2. Dane wejściowe

| Element | Źródło |
|---|---|
| Długości segmentów: udo `hip–knee`, podudzie `knee–ankle`, stopa `ankle–mtp5`, `heel–mtp5`, tułów `hip–shoulder`, ramię `shoulder–elbow`, przedramię `elbow–wrist` | mediany z ważnych cykli (mm), po korekcie punktów |
| Krzywa kąta stopy w cyklu | `foot_angle.mean[k]` |
| Pochylenie pleców | `torso_angle.med` |
| Położenie biodra i dłoni względem roweru | punkty zmierzone w układzie roweru (mediana w cyklu) |
| Ustawienie roweru | `fit_bike_setup` sesji (`docs/05`) |
| Geometria ramy | `fit_bike` (gdy brak – położenia z obrazu, rozdz. 7) |

## 3. Model roweru i punkty kontaktu

1. Punkt siodła `T` (miejsce podparcia na górnej powierzchni) = `bb + saddle_height_mm · (−cos β, sin β)`
   przesunięty poziomo o różnicę cofnięcia, gdzie `β = geo_seat_angle_deg`.
2. Główka: górny punkt `HT = (geo_reach_mm, geo_stack_mm)`.
3. Oś kierownicy `C` = `HT + (spacers_mm + h_stem/2) · u_steerer + stem_length_mm · u_stem`, gdzie `u_steerer`
   – jednostkowy wzdłuż sterownika (kąt główki), `u_stem` – obrócony o `stem_angle_deg` od prostopadłej do
   sterownika, `h_stem` = 40 mm (stała konfiguracji).
4. Przesunięcia stałe wyznaczone z pomiaru dla bieżącego ustawienia:
   - `o_hip = H_zmierzone − T_bieżące` (biodro względem siodła),
   - `o_hand = W_zmierzone − C_bieżące` (nadgarstek względem osi kierownicy dla zadeklarowanego chwytu),
   - przesunięcie `hoods_offset_mm` przesuwa `o_hand` wzdłuż łuku (przybliżenie: poziomo).
5. Oś pedału: `P(θ) = bb + crank_length_mm · (sin θ, cos θ)` (θ = 0 u góry, układ roweru, y w górę).

## 4. Rozwiązanie wariantu

Dla nowego ustawienia (zmieniony jeden lub kilka parametrów):
1. `T'`, `C'` z rozdz. 3; `H' = T' + o_hip`; `W' = C' + o_hand`.
2. Noga dla każdej fazy θ (krok 5°): `MTP' = P'(θ) + o_foot(θ)` (przesunięcie stopy względem osi pedału
   zmierzone w cyklu), kostka `A' = MTP' − L_foot · (cos φ(θ), −sin φ(θ))` z kątem stopy φ z krzywej zmierzonej;
   kolano `K'` = przecięcie okręgów (`H'`, udo) i (`A'`, podudzie), rozwiązanie z kolanem z przodu.
   Brak przecięcia (noga za krótka) → noga wyprostowana, flaga `overextended` (wariant oznaczony jako nierealny).
3. Tułów i ręce:
   - wyjściowy kąt pleców `τ' = τ + k_follow · Δy_hand`, gdzie `Δy_hand` – zmiana wysokości dłoni (mm),
     `k_follow = sim.torso_follow_deg_per_mm` (domyślnie 0,10°/mm – **założenie**, do kalibracji na danych przed/po),
   - bark `S' = H' + L_torso · (cos τ', sin τ')`,
   - jeśli `|S'−W'| > L_upper + L_fore − sim.reach_margin_mm` (15 mm) → zmniejszaj `τ'` co 0,5° aż do osiągnięcia
     (rowerzysta pochyla się, żeby sięgnąć); jeśli `|S'−W'| < sim.min_reach_mm` (łokieć < 90°) → zwiększaj `τ'`,
   - łokieć `E'` = przecięcie okręgów (`S'`, ramię) i (`W'`, przedramię), rozwiązanie z łokciem poniżej odcinka `S'W'`.
4. Pomiary wariantu liczone tymi samymi funkcjami co dla nagrania (`docs/08`): `knee_flex_bottom`, `knee_flex_top`, `hip_angle.min`,
   `knee_over_pedal`, `elbow_angle`, `shoulder_angle`, `torso_angle`, `hands_load_share`.
5. **Korekta przesunięcia modelu:** wartość wariantu = `zmierzona + (model(nowe) − model(bieżące))` – model
   przewiduje różnicę, a nie wartość bezwzględną; błąd modelu dla stanu bieżącego nie przenosi się do wyniku.
6. Statusy regulacji wariantu według bieżącego profilu i modyfikatorów (`docs/09`).

### 4.1 Jawne założenia przewidywania i kiedy zawodzą

| Założenie | Kiedy zawodzi | Skutek w interfejsie |
|---|---|---|
| biodro w stałym miejscu względem siodła (`o_hip`) | rowerzysta po zmianie przesuwa się na siodle (częste przy dużej zmianie wysokości lub kąta siodła) | krok ograniczony do 10 mm; przewidywanie dla zmian > 10 mm oznaczone `low` |
| dłoń w stałym miejscu względem kierownicy (`o_hand`) | zmiana chwytu, inne klamkomanetki | `W_GRIP_MISMATCH` → brak przewidywania dla górnej części ciała |
| krzywa kąta stopy jak w nagraniu | zmiana pracy stopy po zmianie wysokości siodła (kompensacja w stawie skokowym) | przewidywana zmiana zgięcia kolana może być przeszacowana – opisane w „Szczegółach” |
| stałe pochylenie pleców, dopóki dłonie sięgają (`torso_follow`) | zmiana nawyku ułożenia tułowia | wskaźniki tułowia i obciążeń jako poglądowe |
| ruch w płaszczyźnie strzałkowej | rotacja miednicy, tor kolana w płaszczyźnie czołowej | niewidoczne w 2D – stałe zastrzeżenie |

## 5. Wskaźniki obciążenia (poglądowe)

| Kod | Wzór | Poziomy (niskie / średnie / wysokie) | Poziom dowodu |
|---|---|---|---|
| `hands_load_share` | `0,55 · (S.x − H.x) / (W.x − H.x) · 100` – udział ciężaru górnej części ciała przenoszony na dłonie w modelu statycznym (moment wokół biodra, środek ciężkości w 55% odcinka biodro–bark) | < 30% / 30–38% / > 38% | `assumption` |
| `neck_load` | z `torso_angle`: im niżej plecy, tym większe odgięcie głowy | > 45° / 35–45° / < 35° | `assumption` |
| `low_back_load` | gorszy z: `torso_angle` (> 42° / 33–42° / < 33°) i `hip_angle.min` (> 56° / 50–56° / < 50°) | jw. | `assumption` + asocjacja z przeglądu 2017 |

W interfejsie zawsze z dopiskiem „poglądowe – z uproszczonego modelu, nie z pomiaru”. Nie są to symulacje pracy
mięśni (te poza zakresem – `docs/18`, D-10).

## 6. Siatka wariantów (liczona po każdej analizie, korekcie i `refit`)

| Parametr | Kroki względem bieżącego | Liczba |
|---|---|---|
| `saddle_height_mm` | −20, −15, −10, −5, +5, +10, +15, +20 | 8 |
| `saddle_setback_mm` | −20 … +20 co 5 | 8 |
| `stem_length_mm` | −30, −20, −10, +10, +20 (w zakresie 35–150) | 5 |
| `stem_angle_deg` | z {−6, 0, +6, +17} poza bieżącym | 3 |
| `spacers_mm` | −20 … +20 co 5 (w granicach 0–60) | ≤ 8 |
| `crank_length_mm` | z {165; 167,5; 170; 172,5; 175} poza bieżącym | 4 |
| `hoods_offset_mm` | −10, +10 | 2 |
| **Razem** | | ≤ 38 |

Każdy wariant zapisuje: parametr, wartość, geometrię roweru po zmianie (`geometry`: położenie siodła, osi
kierownicy, drop, odległość siodło–kierownica), przewidywane pomiary (`prediction`), statusy regulacji, wskaźniki
obciążenia, flagi (`overextended`, `unreachable`), punkty sylwetki dla 4 faz (mm oraz px w kadrze danej fazy).
Siatka wariantów daje też **macierz wpływu** (`docs/09`, rozdz. 11).

Parametry niemodelowane (tylko informacja w UI): `saddle_tilt_deg`, `bar_width_mm`, `cleat_*`, `q_factor_mm`,
`wheel_size`, `tyre_width_mm`, zmiana ramy (ta – przez zadanie `simulate` z nową geometrią, po MVP).

## 7. Brak geometrii ramy

Jeśli rower nie ma `geo_*`, model używa położeń z obrazu: `T` z punktu wskazanego przez użytkownika na kadrze
(opcjonalnie) lub z `H − o_hip_typowe` (`sim.default_hip_offset_mm` = (13, 46)), `C` z `W − o_hand_typowe`
(`sim.default_hand_offset_mm` = (130, 35)). Warianty są wtedy oznaczone `approximate = true` i UI pokazuje
„dokładność ograniczona – uzupełnij geometrię roweru”.

## 8. Kombinacje zmian

- Jeden parametr → gotowy wariant.
- Kilka parametrów → **szacunek liniowy liczony w PHP** (suma różnic pojedynczych wariantów dla każdego pomiaru,
  bez geometrii) oznaczony `estimate = true` + przycisk „Przelicz dokładnie” → zadanie `simulate` (Python,
  pełne rozwiązanie z rozdz. 4) → wynik zastępuje szacunek.
- Przeglądarka tylko wyświetla wartości otrzymane z API.
- Plan sekwencyjny (`docs/09`, rozdz. 11): krok 1 z wariantu, kolejne kroki – pełne rozwiązanie łącznie
  z poprzednimi krokami (ta sama funkcja co `simulate`, wywoływana w `analyze`/`correct`/`refit`, ≤ 4 rozwiązania).

## 9. Rysowanie w S12 i S10

API zwraca dla wybranej fazy: URL kadru, rozmiar obrazu, listę odcinków „teraz” i „po zmianie” w px obrazu,
pozycje etykiet kątów z wartościami i różnicami. Frontend rysuje SVG na wymiarach obrazu (`viewBox` = rozmiar
kadru). Przy `video_retention = delete_after` po wygaśnięciu kadrów – to samo na neutralnym tle z rysunkiem roweru
z geometrii.

## 10. Walidacja modelu

- Po każdej rzeczywistej zmianie z nagraniem kontrolnym: porównanie Δ przewidzianej z Δ zmierzoną (kryterium
  `executed_as_planned`) – dane zbierane anonimowo (`fit_model_feedback`) do kalibracji `k_follow` i przesunięć.
- Cel: |Δ przewidziana − Δ zmierzona| ≤ 2° dla `knee_flex_bottom` przy zmianie wysokości siodła o 5–10 mm
  w ≥ 80% zmian; analogiczne cele dla pozostałych regulacji w kartach reguł (`docs/09`, rozdz. 1.1).
- Dopóki cel nie jest potwierdzony (V4.1, V5.2), wielkość kroku nie pochodzi z przewidywania (reguły
  `experimental` – stały krok), a S12 pokazuje przewidywanie z plakietką „niezweryfikowane”.
