# 14 · Frontend

## 1. Zasady

- Szablony PHP renderują stronę; interaktywność przez **Alpine.js 3** (wersja przypięta, plik lokalny
  `public/fit/assets/js/alpine.min.js`, bez CDN – zgodność z CSP).
- Brak kroku budowania (bez npm w produkcji). Własny JS w modułach ES (`assets/js/*.js`), ładowanych `type="module"`.
- **Frontend nie liczy pomiarów, zakresów, statusów ani wariantów.** Wyświetla dane z API. Dozwolone obliczenia:
  przeliczenie współrzędnych kursora na piksele obrazu (korekta punktu), pozycje elementów rysunku, formatowanie liczb.
- Makiety `docs/makiety/` są wzorcem układu i treści; dokładne odwzorowanie odstępów nie jest wymagane, hierarchia tak.

## 2. Struktura

```
templates/
  layout.php                 nagłówek, nawigacja, kontener, stopka z wersją
  partials/stepper.php       pasek kroków analizy (S02–S10)
  partials/range-bar.php     pasek zakresu regulacji
  partials/status-chip.php   status słowny + ikona
  partials/confidence.php    poziom pewności (słowo + ikona + rozwijane „dlaczego?”)
  partials/effort.php        wysiłek zmiany (czas, narzędzia, trudność, serwis)
  partials/ride-rating.php   ocena po jeździe – 5 przycisków
  partials/keyframe.php      kadr + nakładka SVG
  pages/S01-moje-analizy.php … S14-ocena-zmiany.php, S02q-szybki-start.php, M01-telefon.php,
        rowery.php, profil.php, komfort.php
  i18n/pl.php
public/fit/assets/
  css/fit.css                tokeny + komponenty
  js/alpine.min.js
  js/api.js                  fetch z CSRF, obsługa błędów 401/422
  js/upload.js               przesyłanie fragmentami
  js/job-poller.js           odpytywanie stanu zadania
  js/keyframe-editor.js      przeciąganie punktów (S10)
  js/variant-viewer.js       wybór wariantu i rysowanie (S12)
```

## 3. Komponenty

| Komponent | Zachowanie |
|---|---|
| Pasek kroków | stany `ok` / `on` / brak; klikalne tylko ukończone; szacowany czas przy kroku (`docs/03`, rozdz. 1); czytnik ekranu: `aria-current="step"` |
| Pasek zakresu | 5 stref (poza zakresem, przejściowa, zalecana, przejściowa, poza zakresem – kolory z rozdz. 4, bez czerwieni) z szerokościami z API (`green`, `yellow`, `scale`); strefa zalecana z obramowaniem i podpisem „zalecany”; znacznik wartości (trójkąt/prostokąt, `--ink`), opcjonalnie znacznik „było” (obrys); `role="img"` z opisem słownym; etykiety końców z API (`docs/21`, rozdz. 6) |
| Status | tekst + ikona + kolor wg `docs/21`, rozdz. 2; nigdy sam kolor |
| Pewność | słowo + 3 kropki (●●●/●●○/●○○) + `<details>` „Dlaczego taka pewność?” z tekstami powodów z API |
| Ocena po jeździe | 5 przycisków (`<button>`, min. 44×44 px) w jednej linii; po wyborze opcjonalny suwak 0–10 z opisami końców |
| Kadr z nakładką | `<img>` + `<svg viewBox="0 0 W H">` absolutnie nad obrazem; odcinki sylwetki, punkty (kolor źródła), pierścienie niskiej pewności, etykiety kątów z API |
| Edytor punktów (S10) | `pointerdown/move/up` na punkcie; przeliczenie pozycji kursora na px obrazu przez `getScreenCTM().inverse()`; po puszczeniu wybór zakresu korekty; `POST corrections`; blokada edycji do końca zadania; klawiatura: strzałki przesuwają zaznaczony punkt o 1 px (Shift = 10 px), Enter zatwierdza; alternatywa bez przeciągania (WCAG 2.5.7): kliknij punkt, potem kliknij nowe miejsce |
| Kod QR (S08) | obraz QR generowany po stronie serwera (biblioteka do wyboru – D-16) + krótki adres tekstowy obok; bez tokenu w adresie |
| Przesyłanie | `Blob.slice` po `chunk_bytes`, `PUT` z `Content-Range`, 3 ponowienia z odczekaniem, wznowienie po odświeżeniu (id przesyłania w `localStorage`), pasek postępu, anulowanie |
| Odpytywanie zadania | co 5 s, przy karcie w tle co 30 s; zatrzymanie po stanie końcowym |
| Wybór wariantu (S12) | suwak/segment ustawia wartość parametru z dostępnych wartości wariantów; > 1 zmiana → `POST estimate`, oznaczenie „szacunek”, przycisk „Przelicz dokładnie” → `POST simulate` + odpytywanie |

## 4. Styl (zgodny z ridemore.bike)

Tokeny (CSS custom properties):

| Token | Wartość | Użycie |
|---|---|---|
| `--bg` | `#F4F5F1` | tło strony |
| `--paper` | `#FFFFFF` | karty |
| `--tint` | `#EDF1EC` | tła pomocnicze |
| `--hair` / `--hair-strong` | `#E2E5DD` / `#D3D8CE` | linie, obramowania |
| `--ink` / `--ink-soft` / `--ink-mute` | `#15201A` / `#5E6A62` / `#8A948C` | tekst |
| `--accent` / `--accent-dark` / `--accent-soft` | `#2C6B4F` / `#1E4B38` / `#E4EEE8` | przyciski, wybór |
| `--blaze` | `#FFC93F` | akcent na ciemnym tle, sylwetka „teraz”, markery |
| `--good` / `--good-soft` | `#2E7D4F` / `#E3F0E7` | status OK |
| `--warn-ink` / `--warn-soft` | `#6B4E00` / `#FDF3D6` | blisko granicy |
| `--bad-ink` / `--bad-soft` | `#8E2B24` / `#F9E8E6` | do zmiany / błąd |
| `--info-ink` / `--info-soft` | `#1E3F94` / `#E3EAFA` | „Twoje ustawienie”, źródło „model” |
| `--manual` | `#4C2F86` / `#EDE6F7` | źródło „ręcznie” |
| Paski zakresu | poza zakresem `#E6E7E1` (jasna szara), przejściowa `#F4EBCF` (jasna piaskowa), zalecana `#8FC3A3` z obramowaniem `--accent-dark`, zalecana „dla Ciebie” `#5FA27D` | bez czerwieni (C-19); zaburzenia widzenia barw czerwono-zielonych dotyczą do ok. 8% mężczyzn |
| Tło kadru | `#2B3530` | gdy brak zdjęcia |

Typografia: nagłówki **Archivo** 700–800, tekst **Instrument Sans** 400–600, liczby i etykiety techniczne
**IBM Plex Mono** 600, znak „ridemore.bike” **Big Shoulders Display** 900 (Google Fonts – dopuszczone w CSP; do
rozważenia hostowanie lokalne). Promień kart 16 px, przycisków 999 px. Bez gradientów i cieni.

## 5. Responsywność i dostępność

- Priorytet ≥ 1280 px. Do 860 px: pasek kroków nad treścią (poziomo przewijany), siatki 1 kolumna; tabele w
  kontenerze z poziomym przewijaniem; brak poziomego przewijania strony.
- Poziom: **WCAG 2.2 AA** (C-19), m.in. 1.4.11 (kontrast elementów graficznych ≥ 3:1 – znacznik wartości,
  obramowanie strefy zalecanej), 2.5.7 (alternatywa dla przeciągania), 2.5.8 (cel ≥ 24×24 px; w module ≥ 44 px),
  3.3.7 (bez ponownego pytania o dane już podane – profil ciała).
- Kontrast tekstu ≥ 4,5:1; cele dotykowe ≥ 44 px; wszystkie kontrolki to `<button>`, `<a href>`, `<input>` z `<label>`.
- M01 projektowana od 360 px (telefon); pozostałe ekrany wg priorytetu ≥ 1280 px.
- Sylwetka i paski mają alternatywę tekstową (lista wartości) dla czytników ekranu.

## 6. Komunikaty i treści

- Teksty z `templates/i18n/pl.php`; teksty zaleceń z API; zasady języka – `docs/21`.
- Zawsze widoczne zastrzeżenie w S11/S12: szacunek z nagrania 2D, nie zastępuje fittera, ból → fizjoterapeuta.
- Wartości liczbowe w formacie polskim (przecinek dziesiętny, „−” jako minus, spacja przed jednostką „mm”, bez
  spacji przed „°” i „%”).
