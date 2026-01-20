# 📝 QuizApp - Aplikacja Quizowo-Testowa z Fiszkami

Aplikacja do nauki i testowania wiedzy z systemem fiszek, trybem nauki, testów i algorytmem Spaced Repetition.

![Version](https://img.shields.io/badge/version-1.30-brightgreen.svg)
![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Status](https://img.shields.io/badge/status-production-success.svg)

## 📋 Spis treści

- [Funkcjonalności](#funkcjonalności)
- [Instalacja](#instalacja)
- [Użycie](#użycie)
- [Typy pytań](#typy-pytań)
- [Tryby nauki](#tryby-nauki)
- [Spaced Repetition](#spaced-repetition)
- [Import/Export](#importexport)
- [Zarządzanie użytkownikami](#zarządzanie-użytkownikami)
- [Motywy wyglądu](#motywy-wyglądu)
- [Śledzenie postępów](#śledzenie-postępów)
- [Konfiguracja](#konfiguracja)
- [Wymagania](#wymagania)
- [Zmiany w wersjach](#zmiany-w-wersjach)
- [Licencja](#licencja)

---

## ✨ Funkcjonalności

### Pytania
- **Jedna poprawna odpowiedź** (single choice)
- **Wiele poprawnych odpowiedzi** (multiple choice)
- **Ułóż w kolejności** (ordering)
- **Dopasowanie** (pairing) - nowe w v1.26! Połącz elementy (np. flagi → kraje, definicje → pojęcia)
- **Zdjęcia** w pytaniach (z kompresją)
- **Kategorie** i **tagi**
- **Wyjaśnienia** dla pytań
- **Oznaczanie pytań do powtórki** (⭐)
- **Notatki** do pytań

### Tryby nauki
- **Tryb testu** - sprawdzenie wiedzy pod presją czasu
- **Tryb nauki** - uczenie się bez limitu czasowego z natychmiastym feedbackiem
- **Tryb fiszek** - nauka z kartami (przód: pytanie, tył: odpowiedź)
- **Filtrowanie po:**
  - Kategorii
  - Typie pytania
  - Tagu
  - Oznaczeniu do powtórki
  - Trybie: wszystkie / oznaczone do powtórki / "Nie umiem" / SRS (due for review)

### System Fiszek
- **Przód karty:** Treść pytania + odpowiedzi (bez oznaczenia)
- **Tył karty:** Odpowiedź z podkreśleniem poprawnych
- **Ocena:** Umiem / Nie umiem
- **SRS Rating:** Poważenie (1 - Znowo, 2 - Trudne, 3 - Średnie, 4 - Łatwe, 5 - Bardzo łatwe)
- **Gestures:** Przeciągnięcie na mobile (swipe)

### Spaced Repetition (SRS)
- Algorytm SuperMemo-2 do optymalizacji powtórek
- Automatyczne obliczanie kolejnego terminu powtórki
- Inteligentne podejście: trudne pytania pojawiają się częściej
- Oszczędność czasu nauki

### Śledzenie postępów
- **Dashboard** z wizualizacją postępów
- **Wyniki** w trybie testu i nauki
- **Szczegółowe wyniki** - analiza odpowiedzi na każde pytanie
- **Wykresy** pokazujące trendy uczenia się
- **Statystyki SRS** - pytań do powtórki, next review date, retention

### Import/Export
- **JSON** - zachowuje wszystkie dane i formatowanie
- **CSV** - kompatybilny z Excel
- **PDF** - gotowy do druku z kolorowaniem
- **Excel** - tabela z kolorowaniem wyników
- **Pełny backup** - wszystkich pytań, użytkowników, wyników

### Zarządzanie użytkownikami
- **Zalogowanie / Rejestracja**
- **Uprawnienia:**
  - **Administrator** - pełny dostęp
  - **Redaktor** - edycja pytań
  - **Czytelnik** - tylko rozwiązywanie quizów
- **Blokowanie** użytkowników
- **Usuwanie** użytkowników

### Motywy wyglądu
- **Classic** - jasny, klasyczny wygląd
- **Dark** - ciemny motyw dla komfortu oczu
- **Modern (Neon)** - nowoczesny, efektowny motyw

### Responsywność
- **Desktop** - optymalizacja dla dużych ekranów
- **Tablet** - dostosowane interfejsy
- **Mobile** - kompaktowy interfejs, bottom navigation
- **Touch gestures** - swipe dla fiszek

---

## 📥 Instalacja

### Metoda 1: Bezpośrednie otwarcie (PWA)

1. Pobierz plik `index.html`
2. Otwórz go w przeglądarce (Chrome, Firefox, Edge, Safari)
3. Gotowe! Aplikacja działa offline po pierwszym załadowaniu.

### Metoda 2: Lokalny serwer (development)

```bash
# Użyj Python
python -m http.server 8000

# Lub Node.js
npx serve

# Lub PHP
php -S localhost:8000
```

Otwórz `http://localhost:8000` w przeglądarce.

### Metoda 3: Hosting statyczny

1. Wrzuć plik `index.html` do dowolnego hostingu statycznego:
   - **GitHub Pages**
   - **Netlify** (przeciągnij i upuść)
   - **Vercel**
   - **Firebase Hosting**
   - **Surge.sh**
   - **GitHub Release** (download)

2. Gotowe!

---

## 🚀 Użycie

### Pierwsze kroki

1. **Zarejestruj się**
   - Kliknij "Rejestracja" w głównym menu
   - Wypełnij formularz (username, hasło)
   - Przejdź do logowania

2. **Dodaj pytania**
   - Sekcja "Dodaj/edytuj pytanie"
   - Wypełnij:
     - Treść pytania (z podstawowym formatowaniem)
     - Typ pytania (jedna, wiele, ordering, pairing)
     - Odpowiedzi / kroki / pary
     - Zaznacz poprawne odpowiedzi
     - (Opcjonalnie) Kategoria, tagi, wyjaśnienie, obraz
   - Kliknij "Zapisz pytanie"

3. **Rozpocznij naukę**
   - Wybierz tryb:
     - **Test** - sprawdź się pod presją czasu
     - **Nauka** - uczenie się bez limitu czasowego
     - **Fiszki** - nauka z kartami

### Tryb Testu

1. Wybierz **Start testu** z głównego menu
2. Ustaw opcje:
   - Kategoria (lub wszystkie)
   - Tryb: wszystkie / oznaczone do powtórki / "Nie umiem" / SRS
   - Ilość pytań (lub wszystkie)
3. Kliknij "Rozpocznij test"
4. Odpowiadaj na pytania
5. Przeglądaj szczegółowe wyniki po zakończeniu

### Tryb Nauki

1. Wybierz **Tryb nauki** z głównego menu
2. Ustaw opcje:
   - Kategoria (lub wszystkie)
   - Tryb: wszystkie / oznaczone do powtórki / "Nie umiem" / SRS
   - Ilość pytań (lub wszystkie)
   - Losowanie odpowiedzi
3. Kliknij "Rozpocznij naukę"
4. Odpowiadaj na pytanie
5. Kliknij "Sprawdź odpowiedź"
6. Oznacz pytanie jako "Oznacz do powtórki" (opcjonalne)
7. Dodaj notatki do pytania (opcjonalne)
8. Przejdź do następnego pytania

### Tryb Fiszek

1. Wybierz **Fiszki** z głównego menu
2. Ustaw opcje:
   - Kategoria (lub wszystkie)
   - Tryb: wszystkie / oznaczone do powtórki / "Nie umiem" / SRS
   - Ilość pytań (lub wszystkie)
   - SRS Rating (tylko dla pytań do powtórki)
3. Kliknij "Rozpocznij"
4. Zobacz pytanie na przód karty
5. Kliknij kartę aby zobaczyć odpowiedź
6. Oceń: **Umiem** / **Nie umiem** lub wybierz rating SRS (1-5)
7. Przejdź do następnej karty

---

## 📚 Typy pytań

### 1. Jedna poprawna odpowiedź (Single)
Wybierz jedną poprawną odpowiedź z listy.

**Przykład:**
```
Pytanie: Stolica Polski?
Odpowiedzi:
○ Berlin
○ Paryż
○ Warszawa
○ Praga
```

### 2. Wiele poprawnych odpowiedzi (Multiple)
Wybierz wszystkie poprawne odpowiedzi.

**Przykład:**
```
Pytanie: Które z poniższych są kolorowe? (zaznacz wszystkie)
☐ Czerwony
☑ Zielony
☑ Niebieski
☐ Pomarańczowy
```

### 3. Ułóż w kolejności (Ordering)
Ułóż kroki lub elementy w prawidłowej kolejności.

**Przykład:**
```
Pytanie: Kolejność budowy DNA:
1. Woda deszczowa
2. ...
3. ...
4. ...
```

### 4. Dopasowanie (Pairing) - NOWE w v1.26!
Połącz dwa powiązane elementy (np. kraj → flaga, definicja → pojęcie).

**Przykład:**
```
Pytanie: Dopasuj flagi do krajów:
Anglia    → [wybierz] ↓
Niemcy    → [wybierz] ↓
Francja   → [wybierz] ↓
```

**Dostępne pary:**
- Kraj → Flaga
- Definicja → Pojęcie
- Format danych → Opis
- Data → Typ
- I wiele więcej!

---

## 🎓 Tryby nauki

### Tryb Testu
- **Cel:** Sprawdzenie wiedzy pod presją czasu
- **Timer:** Podejmuje czas odpowiedzi na każde pytanie
- **Wynik:** Procent poprawnych odpowiedzi
- **Szczegółowe wyniki:** Analiza każdej odpowiedzi

### Tryb Nauki
- **Cel:** Uczenie się bez limitu czasowego
- **Natychmiasty feedback:** Sprawdź po każdej odpowiedzi
- **Bookmarking:** Oznacz trudne pytania do powtórki
- **Notatki:** Dodawaj własne notatki do pytań

### Tryb Fiszek
- **Cel:** Długotrwała pamięć
- **Przód karty:** Pytanie + odpowiedzi (bez oznaczenia)
- **Tył karty:** Odpowiedź z podkreśleniem
- **Ocena:** Umiem / Nie umiem lub SRS rating (1-5)

---

## 🧠 Spaced Repetition

Aplikacja używa algorytmu SuperMemo-2 do optymalizacji powtórek.

### Jak to działa:

1. **Ocena po każdej karcie:**
   - **1 - Znowo** - Początek nauki
   - **2 - Trudne** - Niepamiętasz
   - **3 - Średnio** - Pamiętasz częściowo
   - **4 - Łatwe** - Pamiętasz dobrze
   - 5 - Bardzo łatwe** - Pamiętasz bardzo dobrze

2. **Algorytm oblicza:**
   - Kiedy następnia powtórka (next review date)
   - Jak długo zapamiętać
   - Optymalne powtórki dla maksymalnej efektywności

3. **Wynik:**
   - Trudne pytania powtarzają się często
   - Łatwe pytania rzadziej
   - Oszczędność czasu nauki

### Filtry SRS:
- **Dopasowanie (Dopasowanie):** Pytania do dopasowania
- **Wszystkie:** Wszystkie pytania (oznaczone na początku)
- **Oznaczone do powtórki:** Tylko oznaczone ⭐
- **„Nie umiem”:** Tylko z rating = 0 (ponownie)
- **SRS:** Tylko due for review (przygotowane przez algorytm)

---

## 📤 Import/Export

### Formaty eksportu:

| Format | Zastosowanie | Zachowuje |
|--------|--------------|-------------|
| **JSON** | Pełny backup / transfer | Wszystko |
| **CSV** | Excel / arkusze kalkulacyjne | Pytania |
| **PDF** | Druk / udostępnianie | Formatowanie |
| **Excel** | Analiza w Excel | Kolorowanie wyników |

### Zrzut ekranu wyników:

![Dashboard](https://i.imgur.com/example.png)
*Dashboard z wizualizacją postępów*

---

## 👥 Zarządzanie użytkownikami

### Uprawnienia:

| Rola | Dodawanie pytań | Edycja pytań | Rozwiązywanie testów | Administracja |
|------|----------------|---------------|-----------------|--------------|
| **Administrator** | ✅ | ✅ | ✅ | ✅ |
| **Redaktor** | ✅ | ✅ | ✅ | ❌ |
| **Czytelnik** | ❌ | ❌ | ✅ | ❌ |

### Akcje administratora:
- Blokowanie użytkowników
- Odblokowywanie użytkowników
- Nadawanie uprawnień edycji
- Odbieranie uprawnień edycji
- Usuwanie użytkowników
- Nadawanie uprawnień administratora

---

## 🎨 Motywy wyglądu

### Classic (Domyślny)
- Jasny, biało-szary interfejs
- Tradycyjny wygląd

### Dark
- Ciemny motyw dla komfortu oczu
- Odpowiedni na nocną naukę

### Modern (Neon)
- Nowoczesny, efektowny design
- Gradienty, cienie, nowoczesne kolory

---

## 📊 Śledzenie postępów

### Dashboard zawiera:

1. **Statystyki ogólne:**
   - Liczba pytań w bazie
   - Liczba użytkowników
   - Średni wynik

2. **Wyniki:**
   - Historia sesji testowych
   - Historia sesji nauki
   - Szczegółowa analiza pytań

3. **Wykresy:**
   - Tendy wyników w czasie
   - Retention wiedzy
   - Distribucja wyników

4. **SRS Analytics:**
   - Pytania do powtórki
   - Next review dates
   - Retention rate

---

## ⚙️ Konfiguracja

### Wszystkie dane są zapisywane w:
- **localStorage** (przeglądarka)
- **Brak backendu** - aplikacja działa offline

### Dane są usuwane gdy:
- Wyczyścisz dane przeglądarki
- Użyjesz trybu "Inkognito" / "Private Browsing"
- Usuniesz cookies i dane strony

### Wskazówki:
- **Regularny backup:** Eksportuj do JSON periodicznie
- **Zdjęcia:** Używaj kompresji (automatyczna w aplikacji)
- **Import/Export:** Używaj formatu JSON dla pełnego backupu

---

## 💡 Wskazówki efektywnej nauki

### Tryb SRS:
1. Zacznij od pytań oznaczonych jako „Nie umiem"
2. Oceniaj szczerze:
   - 1: Zupełnie nie pamiętam
   - 2: Prawie nie pamiętam
   - 3: Pamiętam częściowo
   - 4: Pamiętam dobrze
   - 5: Pamiętam bardzo dobrze
3. Regularnie powtarzaj w trybie SRS

### Tryb nauki:
1. Zawsze sprawdź szczegółowe wyniki
2. Dodawaj notatki do trudnych pytań
3. Oznaczaj pytania do powtórki
4. Regularnie powtarzaj oznaczone pytania

### Tryb fiszek:
1. Przejrzyj pytanie na przód karty
2. Spróbuj odpowiedzieć przed obrotem
3. Oceń szczerze
4. Regularne sesje (codziennie) są najlepsze

---

## 📋 Wymagania

### Przeglądarka:
- **Minimalna:** Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- **Zalecana:** Najnowsza wersja (Chrome, Firefox, Edge)

### Urządzenia:
- Desktop: Windows 10+, macOS 10.14+, Linux
- Tablet: iPadOS 13+, Android 10+
- Mobile: iOS 13+, Android 10+

### Inne:
- Włączony JavaScript
- ~10-20 MB wolnej przestrzeni w localStorage (zależy od ilości pytań)
- Połączenie internetowe (tylko do pierwszego załadowania, potem offline)

---

## 📈 Zmiany w wersjach

Szczegółowa lista zmian w pliku [CHANGELOG.md](CHANGELOG.md)

### Główne wersje:
- **v1.30** - Poprawka: exitPractice() wywoływał sekcję wyników
- **v1.29** - Poprawka UI: Fiszki pokazują obie strony pary
- **v1.28** - Poprawki: Walidacja w trybie nauki, UI fiszek, kolorystyka
- **v1.27** - Poprawka: Walidacja pytań pairing
- **v1.26** - Nowy typ pytania: Dopasowanie (Pairing)
- **v1.25** - Poprawka: Selektory w getPairsData()
- **v1.24** - Poprawka: Filtr typu "pairing" w wyszukiwarce
- **v1.23** - Nowy typ pytania: Dopasowanie (Select-based)
- **v1.22** - Poprawka UI: Skompaktowanie interfejsu fiszek
- **i wiele więcej...**

---

## 👨‍💻 Rozwój

### Technologie:
- **HTML5** - semantyczny HTML
- **CSS3** - Flexbox, Grid, Animacje, Media Queries
- **Vanilla JavaScript** - bez frameworków
- **PWA** - Progressive Web App

### Brak zależności:
- Brak frameworków (React, Vue, Angular, etc.)
- Brak bibliotek zewnętrznych (jQuery, Lodash, etc.)
- Czysty vanilla JS + CSS

### Kod:
- **Jednolikowy plik:** `index.html` (~15,000 linii)
- **CSS:** ~4,000 linii
- **JS:** ~11,000 linii
- **Wielojęzyczny:** Polski, angielski, francuski

---

## 📝 Licencja

MIT License

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
DEALINGS IN THE SOFTWARE.

---

## 🤝 Współpraca

Chętnie przyjmę:
- 🐛 **Bug reports** - zgłaszanie błędów
- 💡 **Feature requests** - propozycje nowych funkcji
- 📝 **Pull requests** - poprawki kodu
- 🌍 **Tłumaczenia** - na inne języki

### Jak zgłosić problemy:
1. Otwórz issue w GitHub
2. Opisz problem:
   - Jakie kroki doprowadziły do problemu?
   - Jaki jest oczekiwany wynik?
   - Screenshots (jeśli dotyczy interfejsu)
3. Dodaj informacje:
   - Wersja aplikacji
   - Przeglądarka i wersja
   - System operacyjny

---

## 📞 Kontakt

- **GitHub Issues:** https://github.com/twoj-repo/quizapp/issues
- **License:** MIT

---

## 🎉 Dziękujemy za użycie!

Dziękujemy za używanie QuizApp! Mam nadzieję, że pomoże Ci w efektywnej nauce i śledzeniu postępów.

**Happy Learning! 📚**
