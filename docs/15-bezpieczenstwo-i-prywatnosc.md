# 15 · Bezpieczeństwo i prywatność

## 1. Dane wrażliwe w module

| Dane | Kategoria | Przechowywanie |
|---|---|---|
| Nagranie wideo, kadry kluczowe | wizerunek (dane osobowe) | pliki poza webroot, retencja wg wyboru (`docs/06`, rozdz. 6) |
| Dolegliwości, urazy, wyniki testów mobilności, różnica długości nóg | **dane o zdrowiu** (szczególna kategoria – art. 9 RODO) | tabele `fit_complaint`, `fit_interview*`, `fit_rider_profile`; tylko za wyraźną zgodą |
| Wymiary ciała, masa | dane osobowe | `fit_rider_profile` |
| Wyniki pomiarów, ustawienia roweru | dane osobowe | tabele `fit_*` |

Analiza prawna (podstawa przetwarzania, rejestr czynności, umowa powierzenia z hostingiem, ewentualna ocena skutków)
– po stronie właściciela ridemore.bike (`docs/18`, D-12). Dokumentacja definiuje mechanizmy techniczne.

## 2. Uwierzytelnienie i autoryzacja

- Tożsamość wyłącznie z sesji ridemore.bike (`docs/02`, rozdz. 6); moduł nie ma własnego logowania.
- Każde zapytanie o zasób zawiera warunek `user_id = :current_user` (lub rola `fitter` z udostępnieniem – po MVP,
  tabela `fit_share`). Zasób innego użytkownika → 404.
- Identyfikatory w adresach są liczbowe; ochrona przed IDOR nie polega na ich nieprzewidywalności, tylko na
  sprawdzeniu właściciela w każdym repozytorium (`findForUser($id, $userId)`).
- Rola `admin`: dostęp do kolejki i statystyk technicznych, **bez** dostępu do nagrań, kadrów i danych o zdrowiu.

## 3. Ochrona żądań

| Zagrożenie | Mechanizm |
|---|---|
| CSRF | token per sesja w nagłówku `X-CSRF-Token` / polu `_csrf`; `SameSite=Lax` ciasteczek platformy (do potwierdzenia) |
| XSS | `htmlspecialchars` przy każdym wyjściu w szablonach; dane do Alpine jako escapowany JSON; CSP bez `unsafe-eval` |
| SQL injection | wyłącznie PDO z parametrami; nazwy kolumn do sortowania z białej listy |
| Wstrzyknięcie poleceń | Python uruchamiany przez `proc_open` z tablicą argumentów, bez powłoki; argumenty to stałe i ścieżki generowane przez serwer |
| Przesyłanie plików | limit rozmiaru, biała lista rozszerzeń, sygnatura kontenera, nazwa pliku generowana przez serwer, katalog poza webroot, brak wykonywania (`.htaccess` nie dotyczy – pliki poza `public/`) |
| Path traversal | ścieżki składane wyłącznie z identyfikatorów liczbowych; `realpath` + sprawdzenie prefiksu `storage_root` |
| Nadużycia | limity: 5 analiz/dzień, 60 żądań API/min, 30 korekt/godz. na użytkownika (konfiguracja) |
| Wycieki w logach | brak danych o zdrowiu, punktów, nazw plików; identyfikator zdarzenia zamiast szczegółów w odpowiedzi |

## 4. Zgody

| Kod (`fit_consent.kind`) | Kiedy | Treść (skrót) | Skutek odmowy / cofnięcia |
|---|---|---|---|
| `health_data` | przed zapisem kroku „Dolegliwości” (S05) i testów mobilności (S04) | zgoda na przetwarzanie informacji o dolegliwościach i ruchomości w celu dopasowania zaleceń | kroki S04–S05 pomijane; brak modyfikatorów; cofnięcie → usunięcie odpowiedzi tych kroków |
| `video_processing` | S08, przed wysłaniem | zgoda na przetwarzanie nagrania (wizerunek) w celu analizy pozycji | brak analizy |
| `keyframes_storage` | S08, przy wyborze `keyframes_only` / `keep` | zgoda na przechowywanie kadrów / nagrania | obowiązuje `delete_after` |
| `model_feedback` | opcjonalnie, w profilu | anonimowe dane do ulepszania modelu (`fit_model_feedback`) | brak zapisu |

Treść zgód wersjonowana (`version`); zmiana treści = ponowne zapytanie.

## 5. Pliki

- `storage_root` poza `DocumentRoot`; uprawnienia katalogów 0750, plików 0640, właściciel – użytkownik PHP.
- Kadry wydawane przez `GET /fit/api/keyframes/{id}/image` po sprawdzeniu właściciela; nagłówki
  `Cache-Control: private, max-age=300`, `Content-Disposition: inline`, `X-Content-Type-Options: nosniff`.
- Nagranie nigdy nie jest udostępniane przez URL (brak podglądu wideo w MVP).
- Usunięcie na żądanie użytkownika (S11/profil): nagranie, kadry, wszystkie dane modułu – natychmiast, z wpisem
  w `fit_audit_log`.

## 6. Sekrety i konfiguracja

- `config/config.php` poza repozytorium (w repo `config/config.example.php` bez wartości).
- Hasło bazy z uprawnieniami minimalnymi: `SELECT, INSERT, UPDATE, DELETE` na `fit_*`, `SELECT` na kolumnach
  `users` potrzebnych do weryfikacji konta.
- Python nie dostaje żadnych sekretów.

## 7. Ograniczenia i komunikaty o rzetelności

- Pomiar 2D z jednej kamery – zawsze komunikat o ograniczeniach (`docs/14`, rozdz. 6).
- Wartości o niewystarczającej jakości ukryte lub oznaczone (`docs/09`, rozdz. 3).
- Moduł nie stawia diagnoz; przy silnym bólu lub świeżym urazie – zalecenie konsultacji (`docs/04`, rozdz. 5).
