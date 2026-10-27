# 06 · Nagranie i markery

## 1. Protokół nagrania

| Element | Wymaganie | Dlaczego |
|---|---|---|
| Rower | na trenażerze, ustawiony poziomo (przednie koło na podstawce, jeśli trenażer podnosi tył) | poziom odniesienia dla kątów |
| Kamera | statyw, obiektyw na wysokości siodła, dokładnie prostopadle do płaszczyzny roweru, wypoziomowana | błąd perspektywy rośnie z odchyleniem od prostopadłej |
| Odległość | 2,5–4 m od roweru; cały rower i rowerzysta w kadrze z zapasem ~10% | zniekształcenia obiektywu przy brzegach kadru |
| Orientacja | pozioma (landscape) | pełna sylwetka w kadrze |
| Rozdzielczość / fps | min. 1280×720 przy 30 fps; zalecane 1920×1080 przy 60 fps | rozmycie ruchu stopy przy niskim fps |
| Długość | 30–120 s jazdy w stałej kadencji (zalecane 60 s), po 10 min rozgrzewki | min. 20 ważnych cykli |
| Kadencja | stała, 80–95 obr./min (deklarowana w S08) | porównywalność sesji |
| Wysiłek | umiarkowany: RPE 3–4 w skali Borga CR10 („mogę rozmawiać pełnymi zdaniami”); nagranie przed zmęczeniem; wartość podawana w S08 (`recording_rpe`) | przy dużym obciążeniu i zmęczeniu zmienia się kinematyka, zwłaszcza stawu skokowego (Bini i in.); RPE ≥ 7 → `W_HIGH_EFFORT` (C-10) |
| Światło | z przodu lub z boku; nie pod światło (okno za rowerzystą) | wykrywanie markerów i sylwetki |
| Strój | obcisły, kontrastowy względem markerów i tła | widoczność punktów |
| Chwyt | zadeklarowany (`grip`), ten sam w porównywanych sesjach | kąt łokcia i pochylenie zależą od chwytu |
| Strona | strona zwrócona do kamery (`side`) – mierzona tylko ta strona; w S08 wstępnie strona głównej dolegliwości, jeśli jest jednostronna | analiza 2D; druga strona – osobne nagranie (`M_ASYMMETRY`, C-22) |

## 2. Lista kontrolna (S08) – kody

| Kod | Treść |
|---|---|
| `frame_full` | Cały rower i Ty w kadrze, kamera poziomo i prostopadle |
| `light` | Światło z przodu lub z boku, nie pod okno |
| `clothing` | Obcisły strój, widoczne kolano, kostka, bark i łokieć |
| `markers` | 8 markerów naklejonych na stronie zwróconej do kamery |
| `scale` | Znacznik skali 20 cm przyklejony do rury podsiodłowej |
| `warmup` | 10 minut rozgrzewki, potem stała kadencja |
| `effort` | Jadę umiarkowanie – mogę rozmawiać pełnymi zdaniami |
| `tripod` | Telefon na statywie, nie w ręce |

Niezaznaczone pozycje nie blokują wysłania, ale trafiają do wyniku jako ostrzeżenia, jeżeli analiza potwierdzi problem.

## 3. Markery

### 3.1 Zestaw punktów

| Nr | Kod punktu | Punkt anatomiczny | Jak znaleźć (tekst w S07) | Na czym |
|---|---|---|---|---|
| 1 | `hip` | krętarz większy kości udowej | twarda wypukłość na boku uda, ok. dłoni poniżej talii; przy obrocie stopy do środka czuć jej ruch | skóra / obcisły strój |
| 2 | `knee` | nadkłykieć boczny kości udowej | wypukłość na zewnątrz kolana, tuż nad szczeliną stawu, w połowie jego głębokości | skóra |
| 3 | `ankle` | kostka boczna (wierzchołek) | najbardziej wystający punkt kostki zewnętrznej | skóra / skarpeta |
| 4 | `heel` | pięta (tył buta) | na bucie, z tyłu, na wysokości podeszwy | but |
| 5 | `mtp5` | głowa V kości śródstopia | na bucie, w najszerszym miejscu stopy od strony małego palca | but |
| 6 | `shoulder` | wyrostek barkowy (akromion) | kostny „daszek” na szczycie barku | skóra / obcisły strój |
| 7 | `elbow` | nadkłykieć boczny kości ramiennej | wypukłość po zewnętrznej stronie łokcia | skóra |
| 8 | `wrist` | wyrostek rylcowaty kości łokciowej | guzek na nadgarstku od strony małego palca | skóra |
| S | `scale_a`, `scale_b` | znacznik skali | dwie kropki w odstępie `scale_marker_mm` (domyślnie 200 mm) na pasku przyklejonym wzdłuż rury podsiodłowej | rama |

Dodatkowo wykrywane bez markera (model pozy / geometria): `head` (orientacyjnie), `bb` (oś suportu – środek okręgu
toru pedału), `pedal` (oś pedału – szacowana z toru `mtp5`, `docs/07`, rozdz. 7).

### 3.2 Specyfikacja fizyczna markera

| Cecha | Wartość | Uwagi |
|---|---|---|
| Kształt | koło Ø 25 mm (zakres 20–30 mm) | |
| Wzór | jednolite koło w kolorze sygnałowym z czarną obwódką 3 mm | obwódka poprawia wykrywanie na jasnym stroju |
| Kolor | domyślnie żółty fluorescencyjny; alternatywnie pomarańczowy | konfigurowany w `pipeline.json` (`markers.color_hsv_ranges`) |
| Powierzchnia | matowa | błyszcząca daje odblaski |
| Znacznik skali | pasek 40×250 mm, dwie czarne kropki Ø 12 mm w odstępie 200,0 mm, na białym tle | szablon do druku w wersji DIY (zadanie F7) |

Tryb `diy`: szablon PDF do wydruku (koła + znacznik skali) i instrukcja przyklejenia taśmą dwustronną.
Tryb `kit`: zestaw ridemore (decyzja biznesowa – `docs/18`, D-09).
Tryb `none`: analiza tylko modelem pozy; wszystkie punkty mają źródło `model` i niższą pewność.

### 3.3 Ograniczenia markerów

- Marker na luźnym materiale przesuwa się względem kości – instrukcja wymaga skóry lub obcisłego stroju.
- Marker może być zasłonięty (korba, but, ręka) – wtedy punkt pochodzi z modelu pozy lub interpolacji, z niższą
  pewnością (`docs/07`, rozdz. 6).
- Błędne naklejenie (np. na rzepce zamiast nadkłykcia) daje systematyczny błąd – S10 pozwala go poprawić ręcznie.

### 3.4 Kreator naklejania krok po kroku (S07)

Kreator działa na komputerze i na telefonie (M01, kod QR) – przy rowerze zwykle jest telefon. Postęp zapisuje się
w szkicu sesji. Kolejność: przygotowanie → 8 punktów → znacznik skali → podsumowanie.

**Przygotowanie (S07):** co potrzebne (8 naklejek, znacznik skali, obcisły strój, pomocnik – zalecany przy 1 i 6),
wybór `kit` / `diy` / `none` (`none` → od razu nagranie), **strona ciała** (`side` – wybierana tutaj, nie w S08;
naklejamy na stronie zwróconej do kamery), czas ok. 5 min.

**Krok punktu (S07a, „N z 8”):** zbliżenie miejsca na rysunku sylwetki, pozycja, jak znaleźć, typowy błąd z
rysunkiem „tak / nie tak”, przyciski „Naklejone” / „Nie mogę znaleźć – pomiń” (punkt wyznaczy model pozy,
z niższą pewnością, zapis `markers_skipped`) / „Wstecz”.

| Nr | Punkt | Pozycja przy naklejaniu | Jak znaleźć | Typowy błąd |
|---|---|---|---|---|
| 1 | Biodro – krętarz większy | siedząc na rowerze, korba na dole | dłoń na boku uda ok. szerokość dłoni poniżej talii; obracaj stopę do środka i na zewnątrz – pod palcami przesuwa się kostna wypukłość; naklej na jej środku | za wysoko na kości miednicy; na luźnych spodenkach |
| 2 | Kolano – nadkłykieć boczny | korba poziomo z przodu (kolano zgięte ok. 90°) | z boku kolana znajdź miękkie zagłębienie szczeliny stawu; przesuń palec 1–2 cm w górę na kostną wypukłość uda, w połowie głębokości kolana | na rzepce (z przodu); na główce kości strzałkowej (niżej) |
| 3 | Kostka boczna | korba na dole | najbardziej wystający punkt kostki zewnętrznej | na bucie powyżej kostki |
| 4 | Pięta | stopa w bucie, korba na dole | na bucie z tyłu, na wysokości podeszwy, w środku pięty | za wysoko na zapiętku |
| 5 | Stopa – głowa V kości śródstopia | stopa w bucie | na bucie od strony małego palca, w najszerszym miejscu stopy (zwykle nad osią pedału) | za blisko palców |
| 6 | Bark – wyrostek barkowy | dłonie na kierownicy w zwykłym chwycie | przesuwaj palce wzdłuż obojczyka na zewnątrz aż do kostnego „daszka”; naklej na jego bocznej krawędzi | na mięśniu poniżej barku |
| 7 | Łokieć – nadkłykieć boczny | jw., łokieć lekko zgięty | kostna wypukłość po zewnętrznej stronie łokcia | na mięśniu przedramienia |
| 8 | Nadgarstek – wyrostek rylcowaty kości łokciowej | jw. | guzek na nadgarstku od strony małego palca | na luźnym mankiecie lub rękawiczce |
| S | Znacznik skali | – | pasek płasko na rurze podsiodłowej, wzdłuż rury, kropki widoczne z boku | na zakrzywionej części ramy; przekrzywiony |

**Podsumowanie (S07b):** lista punktów (naklejone / pominięte), przypomnienie o stronie, „Dalej: nagranie”.

**Po MVP – zdjęcie kontrolne (D-18):** zdjęcie z boku przed nagraniem; serwer wykrywa markery i sprawdza kolejność
i odległości (np. biodro–kolano względem długości nogi), wskazuje punkt do poprawy. Wymaga nowego typu zadania.

## 4. Kalibracja

| Element | Metoda | Wynik |
|---|---|---|
| Skala | odległość w pikselach między `scale_a` i `scale_b` / `scale_marker_mm` | `scale_px_per_mm` |
| Poziom | linia podłogi (wykrywanie linii) lub linia osi kół; fallback: założenie poziomej kamery + ostrzeżenie `W_TILT_UNCERTAIN` | `camera_tilt_deg` |
| Ograniczenie | skala obowiązuje tylko w płaszczyźnie znacznika; ciało rowerzysty jest ~10–15 cm bliżej kamery niż rama → błąd skali kilka % | komunikat w szczegółach |

Brak znacznika skali → wartości w mm (KOPS, ruch miednicy, warianty w mm) liczone ze skali z wymiarów ciała:
długość nogi (`body.inseam_cm`), wzrost i rozpiętość ramion przez proporcje segmentów (wzajemna kontrola –
rozbieżność > 8% → ostrzeżenie), oznaczone jako mniej pewne (`W_SCALE_ESTIMATED`). Brak znacznika i wymiarów →
tylko kąty i kierunek zmiany (`W_NO_SCALE`, `docs/10`, rozdz. 7). Dane roweru nie są potrzebne do skali.

## 5. Plik wideo – walidacja

| Kontrola | Gdzie | Limit (konfiguracja) | Kod błędu |
|---|---|---|---|
| Rozszerzenie | przeglądarka, PHP | `mp4`, `mov` | `E_VIDEO_FORMAT` |
| Rozmiar | przeglądarka, PHP | ≤ `upload.max_bytes` (500 MB) | `E_VIDEO_TOO_LARGE` |
| Sygnatura kontenera | PHP (`ftyp` w pierwszych 12 bajtach; `finfo` MIME `video/mp4`/`video/quicktime`) | – | `E_VIDEO_FORMAT` |
| Dekodowanie | Python | kodeki H.264, HEVC (HEVC z iPhone – zgodność do sprawdzenia, zadanie E2) | `E_VIDEO_CODEC` |
| Czas trwania | Python | 30–120 s (`video.min_s`, `video.max_s`); dłuższe – analizowany najlepszy fragment 60 s | `E_VIDEO_TOO_SHORT` / ostrzeżenie `W_VIDEO_TRIMMED` |
| fps | Python | ≥ 25 (`video.min_fps`); < 50 → ostrzeżenie `W_LOW_FPS` | `E_VIDEO_FPS_LOW` |
| Rozdzielczość | Python | krótszy bok ≥ 720 px | `E_VIDEO_RES_LOW` |
| Orientacja | Python (metadane rotacji) | obrót wg metadanych; pion → ostrzeżenie | `W_PORTRAIT` |

## 6. Retencja nagrania

| Wybór użytkownika | Po udanej analizie | Po nieudanej analizie |
|---|---|---|
| `delete_after` (domyślnie) | wideo usunięte zaraz po analizie; kadry JPEG usunięte po zatwierdzeniu punktów w S10 lub po `retention.keyframes_grace_days` (7); zostają wyniki i współrzędne punktów (sylwetka rysowana bez zdjęcia) | wideo przechowywane `retention.failed_video_days` (7) dla ponowienia, potem usunięte |
| `keyframes_only` | wideo usunięte; 4 kadry kluczowe JPEG zostają (do S10, S12) | jw. |
| `keep` | wideo i kadry zostają do czasu usunięcia przez użytkownika | jw. |

Uwaga: przy `delete_after` weryfikacja punktów (S10) działa na zdjęciu, ale po jej zatwierdzeniu wirtualne
dopasowanie (S12) i kolejne korekty pokazują sylwetkę na neutralnym tle. S08 opisuje wszystkie trzy opcje
neutralnie – co daje każda i co jest przechowywane – bez wstępnego zaznaczenia innej niż domyślna.
Domyślna opcja `delete_after` – decyzja ADR-15 (`docs/18`, C-16).

## 7. Przesłanie nagrania z telefonu (C-18)

Nagranie zwykle powstaje telefonem, a przepływ odbywa się na komputerze. W S08 karta „Nagrywasz telefonem?”:

1. Kod QR i krótki adres prowadzący do strony **M01** `/fit/sesja/{draft}/telefon` (prosta strona przesyłania
   na telefonie, część aplikacji webowej).
2. Telefon wymaga normalnego zalogowania do ridemore.bike (ta sama sesja platformy, `docs/15`, rozdz. 2) – kod QR
   nie zawiera tokenu dostępu.
3. M01: statyczna instrukcja kadru (rysunek: rower w poziomie, kamera na wysokości siodła, 2,5–4 m), wybór strony,
   pole pliku `accept="video/mp4,video/quicktime"` (na telefonie także nagranie aparatem), przesyłanie fragmentami
   tym samym API (`docs/13`, rozdz. 6), pasek postępu.
4. S08 na komputerze odpytuje stan przesyłania szkicu i po zakończeniu pokazuje „Nagranie dotarło z telefonu”;
   zgody, RPE i „Wyślij do analizy” – na komputerze lub na M01.

Po MVP (D-16): prowadzenie kadru na żywo w przeglądarce telefonu (podgląd kamery z obrysem sylwetki) – wymaga
sprawdzenia zgodności przeglądarek iOS i Android.
