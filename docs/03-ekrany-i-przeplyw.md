# 03 · Ekrany i przepływ

Makiety: `docs/makiety/NN-*.html` (otwierane w przeglądarce) i `docs/makiety/png/NN-*.png`. Makiety są wzorcem
układu, hierarchii i treści; dane na nich są przykładowe. Kolory i typografia: `docs/14-frontend.md`, rozdz. 4.

## 1. Układ wspólny

- Nagłówek 64 px: znak „ridemore.bike” + plakietka „fit”, nawigacja: **Moje analizy**, **Rowery**, **Profil**.
- Przepływ nowej analizy (S02–S10) ma lewy pasek kroków: grupa „WYWIAD” (1–5) i „ANALIZA” (6 Markery, 7 Nagranie,
  8 Sprawdź punkty, 9 Wynik). Stany kroku: `ok` (zielone kółko), `on` (bieżący), brak (szary). Krok jest
  klikalny tylko, jeśli był już ukończony.
- Przyciski: główny (zielony, pełny), drugorzędny (biały z obramowaniem). Na dole kroku: „Wstecz” po lewej,
  „Dalej: …” po prawej.
- Wszystkie odpowiedzi kroków wywiadu zapisują się automatycznie (szkic) – przerwany przepływ można dokończyć.

## 2. Mapa tras

| Ekran | Trasa (GET) | Makieta |
|---|---|---|
| S01 Moje analizy | `/fit` | `01-main` |
| S02 Wywiad – cel i profil | `/fit/analiza/nowa/cel` | `02-wywiad1` |
| S03 Wywiad – ciało | `/fit/analiza/{draft}/cialo` | `03-wywiad2` |
| S04 Wywiad – mobilność | `/fit/analiza/{draft}/mobilnosc` | `04-wywiad3` |
| S05 Wywiad – dolegliwości | `/fit/analiza/{draft}/dolegliwosci` | `05-wywiad4` |
| S06 Wywiad – co z tego wynika | `/fit/analiza/{draft}/wnioski` | `06-wywiad5` |
| S07 Markery | `/fit/analiza/{draft}/markery` | `07-markery` |
| S08 Nagranie | `/fit/analiza/{draft}/nagranie` | `08-nagranie` |
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
   przyciski: „Zrobione – zapisz zmianę” (S13), „Dlaczego?” (S11), „Mi tak wygodnie” (akceptacja – rozdz. 11).
3. Karta **„Pasuje do Twojego profilu: X z 4 ustawień”** – 4 kafelki regulacji ze statusem słownym i ikoną.
4. **Historia Twoich zmian** (oś czasu): data, zmiana, werdykt oceny (S14) i najważniejsza różnica komfortu;
   pozycja „analizujemy nagranie” dla zadań w toku.
5. Karta „Przed nagraniem: 3 minuty rozmowy” → S02.

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

6 kart testów (`docs/04`, rozdz. 4). Każda: rysunek, nazwa, opis, czego dotyczy, 3 odpowiedzi opisowe,
pole liczbowe (opcjonalne; dla testów obustronnych – prawa/lewa).
Uwaga pod siatką: pomiar kąta telefonem; przerwij przy bólu.
Akcje: „Wstecz”, „Pomiń” (brak modyfikatorów mobilności), „Dalej: dolegliwości”.
Reguła: jeżeli wpisano liczbę, kategoria wylicza się z liczby (pole opisowe podświetla się automatycznie).

## 7. S05 · Wywiad – dolegliwości

- Sylwetka (przód/tył) z klikalnymi obszarami + lista obszarów jako przyciski (dostępność).
- Dla każdego zaznaczonego obszaru karta: strona (lewe/prawe/oba), siła 0–10 (11 przycisków), kiedy się pojawia.
- Zaznaczone obszary zwinięte do jednej linii z „Zmień”.
- Pole „Przebyte urazy lub operacje” (tekst, opcjonalny).
- „Nic mi nie dokucza” czyści zaznaczenia.

## 8. S06 · Wywiad – co z tego wynika

Lista „odpowiedź → wpływ” generowana z reguł modyfikatorów modelu dopasowania (`docs/09`, rozdz. 5) – nie
z tekstów zapisanych na sztywno w szablonie. Karta „Twój profil do analizy” (np. „Endurance · mniej giętki”),
informacja o krótkim wywiadzie następnym razem.
Akcja główna: „Dalej: markery i nagranie”.

## 9. S07 · Markery

Rysunek sylwetki na rowerze z ponumerowanymi markerami 1–8 i znacznikiem skali; lista 8 punktów z opisem, jak je
znaleźć; ramka „Jak naklejać”; wybór: „Mam zestaw ridemore” / „Zrobię je sam” / „Nagrywam bez markerów”
(zapisywany jako `fit_session.markers_mode`: `kit` / `diy` / `none`).
Szczegóły punktów: `docs/06-nagranie-i-markery.md`, rozdz. 3.

## 10. S08 · Nagranie

- Schemat ustawienia kamery + lista kontrolna (7 pozycji; zaznaczenie nie jest wymagane, ale niezaznaczone
  pozycje są zapisywane jako `checklist` w sesji i pokazywane w jakości wyniku).
- Pole przesyłania (przeciągnij lub wybierz; MP4/MOV, 30–120 s), postęp przesyłania fragmentów, możliwość
  wznowienia.
- Strona ciała do kamery: prawa / lewa.
- Polityka nagrania: **usuń po analizie** (domyślnie) / **zachowaj tylko kadry kluczowe** / **zachowaj nagranie**.
- Zgoda na przetwarzanie nagrania (wizerunek) – wymagana; zgoda na dane o zdrowiu – zebrana w S05 (`docs/15`).
Akcja: „Wyślij do analizy” – aktywna po przesłaniu pliku i zgodzie; tworzy zadanie `analyze` i przechodzi do S09.
Walidacja w przeglądarce: rozszerzenie, rozmiar; w PHP: rozmiar, rozszerzenie, sygnatura kontenera (`ftyp`);
w Pythonie: czas trwania, fps, rozdzielczość (`docs/06`, rozdz. 5).

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

Podsumowanie:
1. Wybór profilu jazdy (Komfort / Neutralny / Endurance / Sportowy / Własny-nieaktywny dla rowerzysty) –
   zmiana przelicza zakresy (FR-34) bez nowej analizy.
2. **Twoje 4 ustawienia**: każda regulacja – nazwa, opis pomiaru prostym językiem, pasek zakresu
   (czerwony–żółty–zielony–żółty–czerwony, ciemniejsze pole = zakres dopasowany do wywiadu), znacznik wartości,
   etykiety końców (np. „za wysoko” / „za nisko”), status słowny; adnotacja, gdy zakres przesunięto z powodu wywiadu.
3. Regulacja zaakceptowana („Mi tak wygodnie”) – status „Twoje ustawienie”, powód, „Cofnij”.
4. Sekcja „Coś jest poza zakresem, a jest Ci wygodnie?” z powodami do wyboru.
5. Karta **Twój następny krok** (akcja z liczbą mm, uzasadnienie, „Jak:”, przyciski „Sprawdź wirtualnie” (S12),
   „Zrobione – zapisz zmianę” (S13), „Mi tak wygodnie”).
6. Karta „Dopasowane do Ciebie z wywiadu” (modyfikatory) + link do odpowiedzi.
7. Zastrzeżenie (szacunek z nagrania 2D, nie zastępuje fittera, ból → fizjoterapeuta).

Szczegóły dla fittera: jakość (klatki z wiarygodnymi punktami, cykle zaakceptowane/odrzucone z powodami,
skala px/mm, przechył kamery, wersje modeli), tabela wszystkich pomiarów (`docs/08`) z wartością reprezentatywną,
zakresem w cyklu, zmiennością (SD), jakością, definicją (ⓘ), wykres wybranego pomiaru w cyklu 0–100% (średnia ± SD),
reguły i źródła zaleceń, porównanie z poprzednią analizą.

## 14. S12 · Wirtualne dopasowanie

- Kadr z nagrania (faza wybierana: góra / przód / dół / tył) z sylwetką **„teraz”** (linia ciągła, żółta) i obrysem
  **„po zmianie”** (zielona przerywana) z wybranego wariantu; etykiety kątów z różnicą (np. „152° (−4)”).
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
uchwytu suwaka); wartości dla kombinacji oznaczone „szacunek” do czasu zakończenia `simulate`.

## 15. S13 · Zapis zmiany

Formularz: element (lista parametrów `docs/05`), było, jest (wstępnie z planu), „+ Dodaj kolejną zmianę”,
ostrzeżenie „Zmieniaj jedną rzecz naraz” przy > 1 pozycji; data zmiany, wykonał (ja / serwis / bike fitter),
powód (z analizy / ból / nowa część / porada fittera). Karta „Co dalej”: 3–5 jazd, ocena po każdej jeździe,
przypomnienie o nagraniu kontrolnym (data = data zmiany + `fit.followup_days`, domyślnie 14), ocena zmiany.
Zapis tworzy: nową wersję `fit_bike_setup`, rekord(y) `fit_change`, przypomnienie `fit_reminder`.

## 16. S14 · Ocena zmiany

- Werdykt (Skuteczna / Częściowo / Bez efektu / Pogorszenie / Za wcześnie) z ikoną i jednym zdaniem.
- „Na czym opieramy ocenę”: 5 kryteriów z wynikiem tak / częściowo / nie i wyjaśnieniem (`docs/09`, rozdz. 7).
- Legenda werdyktów; „Jak się jeździ po zmianie?” (Lepiej / Bez różnicy / Gorzej) i pole „Czy pojawiło się coś
  nowego?”; karta „Dalej” z następnym krokiem.

## 17. Ekrany pomocnicze (bez makiet w tym zestawie)

Wzorzec wizualny: prototyp z PR skrobi/ahd#26 (`ridemore-fit/prototyp`) oraz styl S01–S14.

| Ekran | Zawartość |
|---|---|
| Rowery | lista rowerów (aktywny), szczegóły: geometria, komponenty, punkty kontaktu ze schematem (A wysokość siodła, B cofnięcie, C siodło–kierownica, D drop), historia ustawień; formularz zmiany |
| Profil | dane ciała z `docs/04`, rozdz. 3, wyniki testów mobilności w czasie, dolegliwości; informacja o danych wrażliwych |
| Komfort | ocena 0–10 dla 11 obszarów, strona, lokalizacja, moment; kontekst jazdy (czas, rodzaj) |

## 18. Stany globalne

| Sytuacja | Zachowanie |
|---|---|
| brak sesji ridemore.bike | przekierowanie do logowania z powrotem na bieżący adres |
| zasób innego użytkownika | 404 (bez ujawniania istnienia) |
| błąd serwera | strona błędu z identyfikatorem zdarzenia (bez szczegółów technicznych) |
| wynik o niskiej jakości | ostrzeżenie w S10/S11; regulacje z jakością `unknown` mają status „Nie oceniono” i brak zalecenia |
