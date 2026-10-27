# 03 · Ekrany i przepływ

Makiety: `docs/makiety/NN-*.html` (otwierane w przeglądarce) i `docs/makiety/png/NN-*.png`. Makiety są wzorcem
układu, hierarchii i treści; dane na nich są przykładowe. Kolory i typografia: `docs/14-frontend.md`, rozdz. 4.
Zmiany po panelu ekspertów (`docs/20`) nie są jeszcze naniesione na makiety – obowiązuje ten dokument; lista
różnic: `docs/makiety/README.md`. Teksty i etykiety: `docs/21-komunikacja.md`.

## 1. Układ wspólny

- Nagłówek 64 px: znak „ridemore.bike” + plakietka „fit”, nawigacja: **Moje analizy**, **Rowery**, **Profil**.
- Przepływ nowej analizy (S02–S10) ma lewy pasek kroków: grupa „WYWIAD” (1–5) i „ANALIZA” (6 Markery, 7 Nagranie,
  8 Sprawdź punkty, 9 Wynik). Stany kroku: `ok` (zielone kółko), `on` (bieżący), brak (szary). Krok jest
  klikalny tylko, jeśli był już ukończony. Przy każdym kroku szacowany czas (C-20): Cel 1 min · Ciało 1–2 min
  (0, gdy profil kompletny) · Mobilność 2 min (+2 dodatkowe) · Bezpieczeństwo i dolegliwości 1 min · Wnioski
  0,5 min · Markery 5 min · Nagranie 3 min + przesłanie.
- Przyciski: główny (zielony, pełny), drugorzędny (biały z obramowaniem). Na dole kroku: „Wstecz” po lewej,
  „Dalej: …” po prawej.
- Wszystkie odpowiedzi kroków wywiadu zapisują się automatycznie (szkic) – przerwany przepływ można dokończyć;
  na S01 karta „Dokończ analizę” z nazwą kroku, na którym przerwano.

## 2. Mapa tras

| Ekran | Trasa (GET) | Makieta |
|---|---|---|
| S01 Moje analizy | `/fit` | `01-main` |
| S02 Wywiad – cel i profil | `/fit/analiza/nowa/cel` | `02-wywiad1` |
| S02q Szybki start | `/fit/analiza/nowa/szybki` | brak – rozdz. 4 |
| S03 Wywiad – ciało | `/fit/analiza/{draft}/cialo` | `03-wywiad2` |
| S04 Wywiad – mobilność | `/fit/analiza/{draft}/mobilnosc` | `04-wywiad3` |
| S05 Wywiad – bezpieczeństwo i dolegliwości | `/fit/analiza/{draft}/dolegliwosci` | `05-wywiad4` |
| S06 Wywiad – co z tego wynika | `/fit/analiza/{draft}/wnioski` | `06-wywiad5` |
| S07 Markery | `/fit/analiza/{draft}/markery` | `07-markery` |
| S08 Nagranie | `/fit/analiza/{draft}/nagranie` | `08-nagranie` |
| M01 Przesłanie z telefonu | `/fit/analiza/{draft}/telefon` | brak – rozdz. 10.1 |
| S09 Analiza w toku | `/fit/sesje/{id}/postep` | `09-analiza` |
| S10 Sprawdź punkty | `/fit/sesje/{id}/punkty` | `10-punkty` |
| S11 Wynik | `/fit/sesje/{id}` (zakładki `?tab=szczegoly`) | `11-wynik` |
| S12 Wirtualne dopasowanie | `/fit/sesje/{id}/symulacja` | `12-symulator` |
| S13 Zapis zmiany | `/fit/rowery/{bikeId}/zmiana?plan={planId}` | `13-zmiana` |
| S14 Ocena zmiany | `/fit/zmiany/{changeId}` | `14-ocenazmiany` |
| Rowery (lista, edycja) | `/fit/rowery`, `/fit/rowery/{id}` | brak – rozdz. 16 |
| Profil ciała | `/fit/profil` | brak – rozdz. 16 |
| Komfort | `/fit/sesje/{id}/komfort` | brak – rozdz. 16 |

`{draft}` = id szkicu sesji (`fit_session.status = 'draft'`). Wszystkie identyfikatory w trasach są sprawdzane
pod kątem właściciela.

---

## 3. S01 · Moje analizy (start)

**Cel:** pokazać tylko analizy i ich efekty; jedno wezwanie do działania.

Elementy:
1. Nagłówek roweru aktywnego + plakietka profilu jazdy + link „Zmień profil” (otwiera S11 z wyborem profilu).
2. Karta **„Twój następny krok”**: ikona, akcja (np. „Skróć mostek o ok. 10 mm”), jedno zdanie dlaczego,
   poziom pewności (słowo + ikona, `docs/21`, rozdz. 3), wysiłek („ok. 30 min · nowy mostek · serwis”),
   przyciski: „Zaplanuj” (okno: „Kiedy to zrobisz?” – data, „Na których jazdach sprawdzisz?” – domyślnie 3
   kolejne; C-08), „Zrobione – zapisz zmianę” (S13), „Dlaczego?” (S11), „Mi tak wygodnie” (akceptacja).
   Przy `safety_level = stop` zamiast tej karty – „Najpierw konsultacja” (`docs/21`, rozdz. 5).
3. Karta **„Pasuje do Twojego profilu: X z 4 ustawień”** – 4 kafelki regulacji ze statusem słownym i ikoną.
4. **„Jak się jechało?”** – gdy od zapisanej zmiany była jazda bez oceny: 5 przycisków jednym dotknięciem
   (Dużo lepiej … Dużo gorzej), potem opcjonalnie suwak siły głównej dolegliwości 0–10 (C-12, C-13).
5. **Historia Twoich zmian** (oś czasu): data, zmiana, werdykt oceny (S14) i trend komfortu (mała linia siły
   głównej dolegliwości z kolejnych ocen – bez kątów); pozycja „analizujemy nagranie” dla zadań w toku.
6. Karta „Nowa analiza: ok. 5 minut pytań (następnym razem ok. 1 minuty)” → S02.

Dane: ostatnia bieżąca analiza aktywnego roweru (`fit_analysis.is_current=1`), `fit_recommendation`,
`fit_change` + `fit_change_evaluation`, `fit_job` w toku.

Stany: brak analiz → karta powitalna „Zrób pierwszą analizę” (opis 3 kroków) zamiast kart 2–4; brak roweru →
„Dodaj rower” przed analizą.

Kryteria akceptacji:
- Na pulpicie nie ma kątów ani wykresów.
- Następny krok jest zawsze jeden (pierwszy z listy zaleceń, pomijając zaakceptowane).
- Zadanie w toku widoczne z linkiem do S09; nieudane – z linkiem do instrukcji poprawy.

## 4. S02 · Wywiad – cel i profil jazdy

Pola (`docs/04-wywiad.md`, rozdz. 2): rower (lista + „Dodaj rower”), profil jazdy (4 karty), godziny w tygodniu,
najdłuższa jazda. Ramka „Dlaczego pytamy?”.
Akcje: „Anuluj” (usuwa szkic po potwierdzeniu), „Dalej: ciało”.
Walidacja: rower i profil wymagane.

**S02q Szybki start** (tylko pierwsza analiza roweru; link „Chcę szybciej – szybki start, ok. 1 min” na S02):
jeden ekran z polami z `docs/04`, rozdz. 8 (rower, profil, główna dolegliwość, pytania bezpieczeństwa, ewentualnie
wzrost i długość nogi) → S07. Wynik oznaczony „Dopasowanie podstawowe” (C-11).

## 5. S03 · Wywiad – ciało

- Pasek „MAMY Z PROFILU” z wartościami już zapisanymi (wzrost, waga, rok urodzenia) i link „Popraw”.
- Karta brakujących wymiarów (pokazywane tylko brakujące): długość wewnętrzna nogi, rozpiętość ramion, rozmiar buta.
- Karta „Siodło, kierownica i bloki (opcjonalnie)”: rozstaw kości kulszowych, szerokość barków, długość stopy,
  łuk stopy.
- Karta „Czy jedna noga jest krótsza?”: nie / tak (strona, mm) / nie wiem.
Akcje: „Wstecz”, „Zmierzę później” (pomija opcjonalne i brakujące, z ostrzeżeniem o niższej dokładności), „Dalej”.
Walidacja: zakresy liczbowe z `docs/04`, rozdz. 3. Zapis do `fit_rider_profile` (wartości trwałe) –
wywiad zapisuje kopię wartości użytych w danej analizie.

## 6. S04 · Wywiad – mobilność

3 karty testów podstawowych (T2, T3, T4) i zwinięta sekcja „Dodatkowe testy (ok. 2 min)” z T1, T5, T6
(`docs/04`, rozdz. 4, C-01). Każda karta: rysunek, nazwa, opis, czego dotyczy, 3 odpowiedzi opisowe, pole liczbowe
(opcjonalne; dla testów obustronnych – prawa/lewa) z polem „2. próba”, wybór „Mierzę sam / z pomocą” i pole
„Test wywołał ból” (C-02).
Uwaga pod siatką: pomiar kąta telefonem; przerwij przy bólu; wyniki domowe są orientacyjne.
Akcje: „Wstecz”, „Pomiń” (brak modyfikatorów mobilności), „Dalej: dolegliwości”.
Reguła: jeżeli wpisano liczbę, kategoria wylicza się z liczby (pole opisowe podświetla się automatycznie).

## 7. S05 · Wywiad – bezpieczeństwo i dolegliwości

- Karta **„Zanim zaczniemy: 7 krótkich pytań”** (`docs/04`, rozdz. 5.1) – tak/nie, wymagane; po „tak” dopisek, że
  pomiar będzie możliwy, a zalecenia mogą zostać wstrzymane (bez straszenia – `docs/21`, rozdz. 5).
- Sylwetka (przód/tył) z klikalnymi obszarami + lista obszarów jako przyciski (dostępność); osobny obszar
  „Drętwienie krocza” (C-05).
- Dla każdego zaznaczonego obszaru karta: strona (lewe/prawe/oba), siła 0–10 (11 przycisków), rodzaj, kiedy się
  pojawia, jak długo trwa po jeździe, jak często, czy także poza rowerem (C-13); wybór „To mi dokucza najbardziej”
  przy > 1 obszarze.
- Zaznaczone obszary zwinięte do jednej linii z „Zmień”.
- Karta „Urazy i operacje”: lista operacji (rodzaj, strona, rok i miesiąc, zalecenia lekarza – C-04), uraz w
  ostatnich 6 miesiącach, pole tekstowe dla fittera (opcjonalne).
- Przy obszarze `saddle`/`saddle_numb` i braku w profilu – pytanie o rozstaw kości kulszowych.
- „Nic mi nie dokucza” czyści zaznaczenia.

## 8. S06 · Wywiad – co z tego wynika

Lista „odpowiedź → wpływ” generowana z reguł modyfikatorów modelu dopasowania (`docs/09`, rozdz. 5) – nie
z tekstów zapisanych na sztywno w szablonie. Karta „Twój profil do analizy” (np. „Endurance · z uwzględnieniem ruchomości”),
informacja o krótkim wywiadzie następnym razem. Przy `safety_level` ≠ `none` – komunikat z `docs/21`, rozdz. 5
(nie blokuje przejścia dalej).
Akcja główna: „Dalej: markery i nagranie”.

## 9. S07 · Markery

Rysunek sylwetki na rowerze z ponumerowanymi markerami 1–8 i znacznikiem skali; lista 8 punktów z opisem, jak je
znaleźć; ramka „Jak naklejać”; wybór: „Mam zestaw ridemore” / „Zrobię je sam” / „Nagrywam bez markerów”
(zapisywany jako `fit_session.markers_mode`: `kit` / `diy` / `none`).
Szczegóły punktów: `docs/06-nagranie-i-markery.md`, rozdz. 3.

## 10. S08 · Nagranie

- Schemat ustawienia kamery + lista kontrolna (8 pozycji, `docs/06`, rozdz. 2; zaznaczenie nie jest wymagane, ale
  niezaznaczone pozycje są zapisywane jako `checklist` w sesji i pokazywane w jakości wyniku).
- Pole przesyłania (przeciągnij lub wybierz; MP4/MOV, 30–120 s), postęp przesyłania fragmentów, możliwość
  wznowienia; karta **„Nagrywasz telefonem?”** z kodem QR i krótkim adresem do M01 (C-18).
- Strona ciała do kamery: prawa / lewa (wstępnie strona głównej dolegliwości, jeśli jednostronna – C-22).
- „Jak ciężko się jechało podczas nagrania?” – skala 0–10 z opisami (Borg CR10), wymagane; opcjonalnie moc w W (C-10).
- Polityka nagrania: **usuń po analizie** (domyślnie) / **zachowaj tylko kadry kluczowe** / **zachowaj nagranie** –
  każda z neutralnym opisem, co daje i co przechowujemy (C-16).
- Zgoda na przetwarzanie nagrania (wizerunek) – wymagana; zgoda na dane o zdrowiu – zebrana w S05 (`docs/15`).
Akcja: „Wyślij do analizy” – aktywna po przesłaniu pliku i zgodzie; tworzy zadanie `analyze` i przechodzi do S09.
Walidacja w przeglądarce: rozszerzenie, rozmiar; w PHP: rozmiar, rozszerzenie, sygnatura kontenera (`ftyp`);
w Pythonie: czas trwania, fps, rozdzielczość (`docs/06`, rozdz. 5).

### 10.1 M01 · Przesłanie z telefonu

Strona dla wąskiego ekranu (≥ 360 px), wymaga zalogowania: nazwa roweru i data szkicu, rysunek kadru, 4 punkty
listy kontrolnej, wybór strony, przycisk „Wybierz lub nagraj film”, postęp przesyłania, komunikat „Gotowe – wróć
do komputera albo wyślij do analizy tutaj” (zgody i RPE jak w S08). Szczegóły: `docs/06`, rozdz. 7.

## 11. S09 · Analiza w toku

Pasek postępu (procent z `fit_job.progress`), 5 etapów opisanych prostym językiem (`docs/07`, rozdz. 2 – mapowanie
etapów technicznych na 5 opisów), ramka „Jeśli coś pójdzie nie tak”, skrót do wirtualnego dopasowania poprzedniej
analizy (jeśli istnieje). Odświeżanie stanu co 5 s (`GET /fit/api/jobs/{id}`).
Stany: `queued` („Czekamy w kolejce”), `running`, `failed` (komunikat z `docs/11`, rozdz. 7, przyciski
„Ponów analizę tego samego pliku” – jeśli plik jest przechowywany – i „Nagraj ponownie”), `done*` → przekierowanie do S10.

## 12. S10 · Sprawdź punkty

- Duży kadr (obraz JPEG z serwera) z nałożoną sylwetką (SVG): odcinki, punkty z kolorem źródła
  (marker – zielony, model – niebieski, ręcznie – fioletowy, niska pewność – żółty pierścień przerywany).
- 4 miniatury kadrów (korba w górze, z przodu, na dole, z tyłu) ze statusem „OK” / „N do sprawdzenia”.
- Tabela punktów: nazwa, źródło, pewność; punkty z pewnością poniżej progu (`docs/07`, rozdz. 9) wyróżnione z opisem
  przyczyny (np. „marker zasłonięty przez korbę”).
- Przeciągnięcie punktu → po puszczeniu krótki wybór zakresu: **„we wszystkich klatkach”** (domyślnie – np. marker
  naklejony obok właściwego punktu) albo **„tylko w tym kadrze”** (np. punkt chwilowo zasłonięty) → 
  `POST /fit/api/analyses/{id}/corrections` → zadanie `correct` → po zakończeniu kadr i tabela odświeżają się;
  komunikat „przeliczono hh:mm”. Pierwotna analiza zostaje.
- Akcje: „Później” (S01), „Poproś bike fittera o sprawdzenie” (po MVP – ukryte flagą), „Punkty się zgadzają – pokaż wynik”.
Kryteria: przeglądarka nie liczy kątów; po korekcie widoczna nowa wersja wyniku; cofnięcie korekty możliwe
(przywrócenie poprzedniej wersji jako bieżącej).

## 13. S11 · Wynik analizy

Zakładki: **Podsumowanie** (domyślna) i **Szczegóły dla fittera**.

**Tryb „raport pomiarowy” (etap M2, zadanie U2.1):** zamiast Podsumowania – wartości pomiarów z jakością
i statusem walidacji, przyczyny odrzucenia lub „nie oceniono” (`docs/07`, rozdz. 9.1), „Nagraj ponownie”; bez
zakresów-zaleceń i akcji. **Statusy reguł (od M4):** regulacja `measure_only` – kafelek „Tylko pomiar – bez
zalecenia” (wartość i pasek w „Szczegółach”); zalecenie `experimental` – plakietka „Eksperymentalne”, krok 5 mm
i nagranie kontrolne (`docs/09`, rozdz. 1.1, `docs/21`, rozdz. 3.1).

Podsumowanie:
1. Wybór profilu jazdy (Komfort / Neutralny / Endurance / Sportowy / Własny-nieaktywny dla rowerzysty) –
   zmiana przelicza zakresy (FR-34) bez nowej analizy. Przy `fit_level = basic` plakietka „Dopasowanie
   podstawowe” i karta „Uzupełnij wywiad (ok. 4 min) – dokładniejsze zalecenia” (C-11).
2. **Twoje 4 ustawienia**: każda regulacja – nazwa, opis pomiaru prostym językiem, pasek zakresu (strefy: poza
   zakresem – przejściowa – zalecana – przejściowa – poza zakresem, bez czerwieni; ciemniejsze pole = zakres
   dopasowany do wywiadu; `docs/21`, rozdz. 6), znacznik wartości, etykiety końców (np. „siodło nisko” / „siodło
   wysoko”), status słowny z ikoną; adnotacja, gdy zakres przesunięto z powodu wywiadu.
3. Regulacja zaakceptowana („Mi tak wygodnie”) – status „Twoje ustawienie”, powód, „Cofnij”.
4. Sekcja „Coś jest poza zakresem, a jest Ci wygodnie?” z powodami do wyboru.
5. Karta **Twój następny krok** (akcja z liczbą mm, uzasadnienie, „Jak:”, poziom pewności z rozwijanym
   „Dlaczego taka pewność?”, „Dlaczego teraz to?” (kolejność z planu), „Wpływ na inne ustawienia” (skutki uboczne),
   wysiłek zmiany, przyciski „Sprawdź wirtualnie” (S12), „Zaplanuj”, „Zrobione – zapisz
   zmianę” (S13), „Mi tak wygodnie”; przy pewności `low` przycisk główny „Nagraj kontrolnie”). Przy
   `safety_level = caution` dopisek „Małymi krokami”, przy `stop` – karta „Najpierw konsultacja” zamiast zalecenia.
6. Karta „Dopasowane do Ciebie z wywiadu” (modyfikatory) + link do odpowiedzi.
7. Zastrzeżenie (szacunek z nagrania 2D, nie zastępuje fittera, ból → fizjoterapeuta).

Szczegóły dla fittera: jakość (klatki z wiarygodnymi punktami, cykle zaakceptowane/odrzucone z powodami,
skala px/mm, przechył kamery, wysiłek nagrania RPE, wersje modeli), powody poziomu pewności każdego zalecenia,
odpowiedzi bezpieczeństwa i historia operacji (dla fittera), tabela wszystkich pomiarów (`docs/08`) z wartością reprezentatywną,
zakresem w cyklu, zmiennością (SD), jakością, definicją (ⓘ), wykres wybranego pomiaru w cyklu 0–100% (średnia ± SD),
reguły i źródła zaleceń, porównanie z poprzednią analizą.

## 14. S12 · Wirtualne dopasowanie

- Kadr z nagrania (faza wybierana: góra / przód / dół / tył) z sylwetką **„teraz”** (linia ciągła, żółta) i obrysem
  **„przewidywanie po zmianie”** (zielona przerywana) z wybranego wariantu; etykiety kątów z różnicą (np. „152° (−4)”).
- Trzy rodzaje wyniku rozdzielone wizualnie (`docs/10`, rozdz. 1.1): **„Rower po zmianie”** (geometria, mm –
  dokładne), **„Przewidywana pozycja”** (z plakietką „przewidywanie modelu – nie pomiar i nie dowód, że zmiana
  pomoże”; „niezweryfikowane”, dopóki `prediction_validated = false`), **„Zmierzone”** tylko w S14 po nagraniu
  kontrolnym. Rozwijane „Założenia modelu” (`docs/10`, rozdz. 4.1).
- Panel „Zmień ustawienie” – grupy rozwijane: **Siodło** (wysokość, przód/tył, kąt*), **Kierownica i mostek**
  (długość mostka, kąt mostka −6°/0°/+6°/+17°, podkładki, szerokość kierownicy*, klamkomanetki*), **Korby, pedały
  i bloki** (długość korby, bloki*), **Rama i koła** (rozmiar ramy*, koła i opony*). Parametry oznaczone * nie
  zmieniają kątów w modelu 2D albo nie są modelowane – pokazują tylko informację i wpływ opisowy (`docs/10`, rozdz. 6).
- Panel „Jak pasuje”: 5 pasków (4 regulacje + ciężar na dłoniach) ze znacznikiem „było” (szary) i „będzie” (czarny);
  licznik „X z 5 w zakresie (było Y)”.
- Karty wpływu na ciało: dłonie, kark, dół pleców (poziom niskie/średnie/wysokie, opis; oznaczone jako poglądowe).
- Karta „Twój plan zmian”: lista zmian; przy > 1 zmianie informacja o szacunku i przycisk „Przelicz dokładnie”
  (zadanie `simulate`); „Zapisz plan” (S13), „Zaproponuj za mnie” (wybiera wariant wskazany przez zalecenie).
Kryteria: suwak wybiera gotowy wariant z `fit_variant` (bez obliczeń w przeglądarce poza interpolacją pozycji
uchwytu suwaka); wartości dla kombinacji oznaczone „szacunek” do czasu zakończenia `simulate`. Przy
`safety_level = stop` ekran tylko do podglądu (bez „Zapisz plan” i „Zaproponuj za mnie”).

## 15. S13 · Zapis zmiany

Formularz: element (lista parametrów `docs/05`), **o ile zmieniono** (np. „+15 mm”, wymagane; wstępnie z planu)
z instrukcją „zaznacz taśmą przed zmianą, zmierz linijką”, opcjonalnie było/jest, jeśli znane, „+ Dodaj kolejną zmianę”,
ostrzeżenie „Zmieniaj jedną rzecz naraz” przy > 1 pozycji; data zmiany, wykonał (ja / serwis / bike fitter),
powód (z analizy / ból / nowa część / porada fittera). Karta „Co dalej”: 3–5 jazd, ocena po każdej jeździe
(jednym dotknięciem na S01), przypomnienie o nagraniu kontrolnym (data = data zmiany + `fit.followup_days`,
domyślnie 14), ocena zmiany. Karta „Przypomnienia” (C-14): na pulpicie – zawsze; e-mail – tylko po włączeniu
(domyślnie wyłączony); najwyżej `notifications.max_per_change` (2) przypomnienia na zmianę; „Nie przypominaj”.
Wejście z okna „Zaplanuj” (S01/S11) wypełnia formularz z planu i zapamiętaną datą (`fit_plan.planned_date`).
Zapis tworzy: nową wersję `fit_bike_setup`, rekord(y) `fit_change`, przypomnienie `fit_reminder`.

## 16. S14 · Ocena zmiany

- Werdykt (etykiety z `docs/21`, rozdz. 2: Pomogło / Częściowo / Bez zmian / Gorzej – wróć do poprzedniego / Za
  wcześnie na ocenę) z ikoną i jednym zdaniem.
- „Na czym opieramy ocenę”: 5 kryteriów z wynikiem tak / częściowo / nie i wyjaśnieniem (`docs/09`, rozdz. 9);
  przy odczuciach: „Za istotną poprawę uznajemy spadek o co najmniej 2 punkty lub o 30%”.
- Legenda werdyktów; „Jak się jeździ po zmianie?” (5 stopni: Dużo lepiej / Lepiej / Bez różnicy / Gorzej / Dużo
  gorzej), siła głównej dolegliwości 0–10 i pole „Czy pojawiło się coś nowego?”; karta „Dalej” z następnym krokiem.

## 17. Ekrany pomocnicze (bez makiet w tym zestawie)

Wzorzec wizualny: prototyp z PR skrobi/ahd#26 (`ridemore-fit/prototyp`) oraz styl S01–S14.

| Ekran | Zawartość |
|---|---|
| Rowery | dodanie roweru: tylko nazwa i typ (reszta „Nie wiem”); karta „Uzupełnij 3 wymiary (2 min)”; wartości zmierzone z nagrania do potwierdzenia (`docs/05`, rozdz. 7); lista rowerów (aktywny), szczegóły: geometria, komponenty, punkty kontaktu ze schematem (A wysokość siodła, B cofnięcie, C siodło–kierownica, D drop), historia ustawień; formularz zmiany |
| Profil | dane ciała z `docs/04`, rozdz. 3, wyniki testów mobilności w czasie, dolegliwości; informacja o danych wrażliwych |
| Komfort | ocena 0–10 dla obszarów z `docs/04`, rozdz. 5.2, strona, rodzaj, moment, czas trwania po jeździe; kontekst jazdy (czas, rodzaj) |
| Preferencje (w Profilu) | przypomnienia e-mail (wł./wył.), limit przypomnień, rezygnacja z analityki produktowej (C-14, C-17) |

## 18. Stany globalne

| Sytuacja | Zachowanie |
|---|---|
| brak sesji ridemore.bike | przekierowanie do logowania z powrotem na bieżący adres |
| zasób innego użytkownika | 404 (bez ujawniania istnienia) |
| błąd serwera | strona błędu z identyfikatorem zdarzenia (bez szczegółów technicznych) |
| wynik o niskiej jakości | ostrzeżenie w S10/S11; regulacje z jakością `unknown` mają status „Nie oceniono” i brak zalecenia |
| `safety_level = stop` | pomiar i statusy widoczne; zalecenia, plan i „Zaplanuj” ukryte; komunikat z `docs/21`, rozdz. 5 |
