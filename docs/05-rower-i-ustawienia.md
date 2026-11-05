# 05 · Rower i ustawienia

Rower (`fit_bike`) ma stałą geometrię ramy, a jego ustawienie (`fit_bike_setup`) jest wersjonowane: każda zmiana
tworzy nową wersję i wpis w historii (`fit_change`). Sesja analizy wskazuje wersję ustawienia, na której nagrano jazdę.

**Dane roweru są opcjonalne (ADR-26).** Analiza, statusy i zalecenia (kierunek i wielkość zmiany) liczy się
z nagrania i wymiarów ciała (`docs/10`, rozdz. 3.1). Minimum to nazwa i typ roweru (typ wybiera zakresy).
Każde pole poniżej ma odpowiedź „Nie wiem” (wartość `NULL`, nigdy 0). Dane roweru, jeśli są, dają: wartości
bezwzględne w historii, rysunek roweru po zmianie, dokładniejszy kierunek przesuwu kierownicy (kąt główki)
i ocenę rozmiaru ramy względem bieżących części (`docs/09`, rozdz. 6.4). Brakujące dane są wskazywane jako
„uzupełnij, aby lepiej zwizualizować i dopasować” – nie blokują zaleceń. Rozdz. 7 opisuje, co serwer odczytuje z nagrania zamiast danych roweru.

Układ odniesienia (wszystkie wymiary 2D): początek w osi suportu (BB), oś X do przodu roweru, oś Y w górę,
milimetry. W obrazie oś Y jest skierowana w dół – konwersja w `docs/08`, rozdz. 1.

## 1. Rower (`fit_bike`)

| Kod | Nazwa | Jedn. | Zakres | Wymagane | Uwagi |
|---|---|---|---|---|---|
| `name` | Nazwa | – | 1–60 znaków | tak | np. „Szosa endurance” |
| `type` | Typ | – | `road` · `gravel` · `mtb` · `tt` (po MVP) | tak | wybiera zakresy profilu; „Nie wiem” → propozycja z kadru (kształt kierownicy) do potwierdzenia |
| `size_label` | Rozmiar | – | tekst ≤ 10 | nie | np. „56”, „M” |
| `model_year` | Rocznik | – | 1990–(bieżący+1) | nie | |
| `is_active` | Aktywny | – | bool | – | jeden aktywny na użytkownika (domyślny w S01) |

### Geometria ramy (stała dla roweru)

| Kod | Nazwa | Jedn. | Zakres | Używane w modelu 2D |
|---|---|---|---|---|
| `geo_stack_mm` | Stack (pion BB → środek górnej krawędzi główki) | mm | 450–750 | tak – położenie główki |
| `geo_reach_mm` | Reach (poziom BB → środek główki) | mm | 330–520 | tak |
| `geo_seat_angle_deg` | Kąt rury podsiodłowej (efektywny) | ° | 65–80 | tak – kierunek przesuwu siodła |
| `geo_head_angle_deg` | Kąt główki | ° | 60–76 | tak – kierunek podkładek |
| `geo_head_tube_mm` | Długość główki | mm | 70–250 | nie (rysunek) |
| `geo_chainstay_mm` | Długość tylnych widełek | mm | 380–480 | nie (rysunek) |
| `geo_bb_drop_mm` | Obniżenie suportu | mm | 30–90 | nie (rysunek) |
| `geo_wheelbase_mm` | Rozstaw osi | mm | 900–1300 | nie (rysunek) |
| `wheel_size` | Rozmiar kół | – | `700c` · `650b` · `29` · `27_5` · `26` | nie zmienia kątów rowerzysty (rozdz. 5); **źródło skali** `wheel` (średnica obręczy ETRTO) |
| `tyre_width_mm` | Szerokość opony | mm | 20–70 | z `wheel_size` daje średnicę zewnętrzną koła ≈ obręcz + 2 × szerokość (np. 622 + 2·28 ≈ 678 mm) – źródło skali |

Brak geometrii → model podstawowy z nagrania (`docs/10`, rozdz. 3.1); zalecenia pełne, brak danych wskazany
osobno (`docs/10`, rozdz. 7).

## 2. Ustawienie roweru (`fit_bike_setup`)

### Siodło

| Kod | Nazwa | Jedn. | Zakres | Definicja pomiaru | Wariant symulacji |
|---|---|---|---|---|---|
| `saddle_height_mm` | Wysokość siodła | mm | 550–950 | od osi suportu do górnej powierzchni siodła wzdłuż osi rury podsiodłowej, w miejscu podparcia kości kulszowych (ok. 70–80 mm od tyłu siodła) | −20…+20 co 5 |
| `saddle_setback_mm` | Cofnięcie siodła | mm | 0–150 | poziomo: pion przez oś suportu → czubek siodła (lub środek – zapisać metodę `setback_ref`: `nose`/`center`) | −20…+20 co 5 |
| `saddle_tilt_deg` | Kąt siodła | ° | −10…+5 | poziomica na siodle; ujemny = nos w dół | informacyjnie |
| `saddle_model` | Model siodła | – | tekst | – | – |
| `saddle_width_mm` | Szerokość siodła | mm | 120–180 | najszerszy punkt | dobór z `body.sit_bone_cm` |
| `saddle_length_mm` | Długość siodła | mm | 200–300 | – | – |

### Kierownica i mostek

| Kod | Nazwa | Jedn. | Zakres | Definicja | Wariant symulacji |
|---|---|---|---|---|---|
| `stem_length_mm` | Długość mostka | mm | 35–150 | oś sterownika → oś kierownicy | −30…+20 co 10 |
| `stem_angle_deg` | Kąt mostka | ° | −25…+25 | względem prostopadłej do sterownika; ujemny = w dół | {−6, 0, +6, +17} |
| `spacers_mm` | Podkładki pod mostkiem | mm | 0–60 | suma wysokości podkładek | 0…40 co 5 (w granicy dostępnych) |
| `bar_width_mm` | Szerokość kierownicy | mm | 340–820 | szosa: środek–środek na klamkach; MTB: całkowita | informacyjnie (dobór z `body.shoulder_width_cm`) |
| `bar_reach_mm` | Reach kierownicy | mm | 60–110 | szosa: oś kierownicy → najdalszy punkt łuku | nie (stałe) |
| `bar_drop_mm` | Drop kierownicy | mm | 100–150 | szosa | nie (stałe) |
| `hoods_offset_mm` | Przesunięcie klamkomanetek | mm | −20…+20 | wzdłuż łuku względem pozycji fabrycznej (+ do przodu) | −10, +10 |
| `grip_default` | Typowy chwyt | – | `hoods` · `drops` · `tops` · `flat` | – | – |

### Korby, pedały, bloki

| Kod | Nazwa | Jedn. | Zakres | Definicja | Wariant symulacji |
|---|---|---|---|---|---|
| `crank_length_mm` | Długość korby | mm | 150–180 | oś suportu → oś pedału (nadruk na korbie) | {165; 167,5; 170; 172,5; 175} |
| `pedal_system` | System pedałów | – | `spd_sl` · `look_keo` · `spd` · `time` · `speedplay` · `flat` · `other` | – | – |
| `pedal_stack_mm` | Wysokość pedału + bloku | mm | 5–25 | oś pedału → podeszwa | nie (wpływa na wysokość siodła – informacyjnie) |
| `cleat_fore_aft_mm` | Blok przód/tył | mm | −20…+20 | położenie osi pedału względem głowy I kości śródstopia; + = oś pedału za głową | informacyjnie (2D nie modeluje stopy szczegółowo) |
| `cleat_rotation_deg` | Obrót bloku | ° | −10…+10 | + = pięta do środka | informacyjnie |
| `cleat_shims` | Podkładki pod blok | – | lista: `{side, type: wedge/lift, mm|deg}` | – | flaga przy różnicy długości nóg |
| `q_factor_mm` | Rozstaw pedałów (Q-factor) | mm | 140–190 | – | informacyjnie |

## 3. Wymiary punktów kontaktu (pochodne, na schemacie roweru)

Wyliczane z ustawienia i geometrii (lub zmierzone przez użytkownika), pokazywane na schemacie w ekranie „Rowery”:

| Litera | Kod | Nazwa |
|---|---|---|
| A | `saddle_height_mm` | wysokość siodła |
| B | `saddle_setback_mm` | cofnięcie siodła |
| C | `saddle_to_bar_mm` | odległość siodło (czubek) → kierownica (oś) |
| D | `saddle_bar_drop_mm` | różnica wysokości siodło → kierownica (+ = kierownica niżej) |

## 4. Historia zmian (`fit_change`)

Każdy wpis: `parameter` (kod z rozdz. 2), `delta` (zmiana względna, np. +15 mm – **zawsze wymagana**),
`value_before`, `value_after` (opcjonalne – tylko gdy znana wartość wyjściowa), `unit`, `changed_at`, `performed_by`
(`self`/`shop`/`fitter`), `reason` (`recommendation`/`discomfort`/`new_part`/`fitter_advice`/`other`),
`reason_note`, `recommendation_id` (jeśli z zalecenia), `setup_before_id`, `setup_after_id`.
Kilka parametrów zmienionych tego samego dnia = kilka wpisów z tym samym `setup_after_id` (ocena zmiany oznaczy
kryterium „jedna zmiana naraz” jako niespełnione).

Zmiana wpisywana jest względnie („Podniosłem siodło o 15 mm”, z instrukcją: zaznacz sztycę taśmą przed zmianą,
zmierz przesunięcie linijką). Gdy wartość wyjściowa jest znana, serwer liczy `value_after`; gdy nie – nowa wersja
ustawienia przechowuje tylko przesunięcie. Ocena zmiany porównuje kąty zmierzone przed i po, więc nie wymaga
wartości bezwzględnych.

## 5. Koła i rozmiar ramy

Zmiana kół lub opony podnosi lub obniża cały rower razem z rowerzystą; położenie siodła, kierownicy i pedałów
względem suportu się nie zmienia, więc **kąty rowerzysty pozostają takie same**. Te parametry są w grupie
„Rama i koła” symulatora wyłącznie informacyjnie (wysokość nad ziemią, prześwit) i do przyszłego porównania ram
(po MVP). Zmiana rozmiaru ramy w symulacji wymaga podania geometrii nowej ramy i jest liczona jako zadanie `simulate`.

## 6. Parametry nagrania zapisywane w sesji (`fit_session`)

| Kod | Nazwa | Wartości |
|---|---|---|
| `side` | Strona ciała do kamery | `right` · `left` |
| `grip` | Chwyt podczas nagrania | `hoods` · `drops` · `tops` · `flat` |
| `markers_mode` | Markery | `kit` · `diy` · `none` |
| `scale_marker_mm` | Długość znacznika skali | 100–400, domyślnie 200 |
| `cadence_target_rpm` | Deklarowana kadencja | 60–110 (opcjonalnie) |
| `video_retention` | Polityka nagrania | `delete_after` (domyślnie) · `keyframes_only` · `keep` |
| `recording_rpe` | Wysiłek podczas nagrania (skala Borga CR10) | 0–10, wymagane; zalecane 3–4 (`docs/06`, rozdz. 1) |
| `recording_power_w` | Moc podczas nagrania (jeśli trenażer pokazuje) | 30–1500 W, opcjonalnie |
| `upload_device` | Skąd przesłano nagranie | `desktop` · `phone` (`docs/06`, rozdz. 7) |
| `checklist` | Lista kontrolna z S08 | mapa `kod → bool` (`docs/06`, rozdz. 2) |

## 7. Dane roweru odczytywane z nagrania

| Wielkość | Jak | Dokładność | Pokazywane jako |
|---|---|---|---|
| oś suportu, okrąg pedału | dopasowanie okręgu do toru `mtp5` (`docs/07`, rozdz. 8) | ± kilka mm | – |
| długość korby (przybliżona) | promień toru stopy ze skalą | ± 5 mm | „ok. 172,5 mm – zmierzone z nagrania, popraw, jeśli znasz” |
| wysokość siodła, cofnięcie, drop siodło–kierownica | odległości `bb` → punkt podparcia (biodro − typowe przesunięcie) i → nadgarstek, ze skalą | ± 10 mm (zależnie od skali) | „zmierzone z nagrania – możesz poprawić” |
| kierunek przesuwu siodła (kąt sztycy) | linia `bb` → siodło na kadrze kluczowym; brak → 73,5° | ± 2° (wpływ < 0,1 mm na 10 mm zmiany) | w „Szczegółach” |

Wartości z nagrania zapisywane są w wyniku (`body_model.bike_from_video`), a w ustawieniu roweru tylko po
potwierdzeniu przez użytkownika (`fit_bike_setup.sources_json`: `user` / `video` / `default`).
