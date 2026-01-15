# Change Log - Aplikacja Quizowo-Testowa

## [1.13] - 2025-01-15

### 🎴 Poprawki wyświetlania fiszek na mobile

#### Problem główny
1. **Formatowanie pytań inne niż w innych trybach** - tekst był wyśrodkowany (`text-align: center`), podczas gdy w test/practice był wyrównany do lewej
2. **Odpowiedzi trzeba było przewijać po obróceniu karty** - max-height był za mały (250px/200px)
3. **Okno było pomniejszone** - brakowało dynamicznego dopasowania do dłuższej części (pytanie lub odpowiedź)

#### Rozwiązania wdrożone

**1. Zmiana wyrównania tekstu z wyśrodkowanego na lewo**
- Zmieniono `.flashcard-content { text-align: center; }` na `text-align: left;`
- Teraz tekst jest wyrównany tak jak w trybach test/practice

**2. Usunięcie wyśrodkowania pionowego na mobile**
- Dodano `justify-content: flex-start` do `.flashcard-face`
- Treść zaczyna się teraz od góry karty, a nie jest wyśrodkowana wertykalnie
- Ułatwia czytanie długich pytań i odpowiedzi

**3. Dynamiczne dopasowanie wysokości do ekranu (viewport-based sizing)**
- Mobile (max-width: 768px): `max-height: calc(100vh - 320px)`
- Small mobile (max-width: 480px): `max-height: calc(100vh - 340px)`
- Okno fiszki automatycznie dopasowuje się do dostępnej przestrzeni ekranu

**4. Poprawa paddingu na mobile**
- Mobile: `padding: 36px 24px`
- Small mobile: `padding: 20px`
- Lepsze wykorzystanie dostępnej przestrzeni

**5. Utrzymanie minimalnej wysokości dla lepszej czytelności**
- Mobile: `min-height: 380px` (dla `.flashcard-face`)
- Small mobile: ten sam `min-height: 380px`

#### Zmiany w CSS

**Zmienione reguły CSS:**
```css
/* Przed */
.flashcard-content {
    text-align: center;
}

.flashcard-face {
    justify-content: center;
    padding: 40px;
}

/* Po - mobile */
.flashcard-content {
    text-align: left;
    overflow-y: auto;
    max-height: calc(100vh - 320px);
}

.flashcard-face {
    justify-content: flex-start;
    padding: 36px 24px;
    min-height: 380px;
}
```

**Nowe reguły CSS (small mobile):**
```css
.flashcard-content {
    max-height: calc(100vh - 340px);
}

.flashcard-face {
    justify-content: flex-start;
    padding: 20px;
}
```

#### Statystyki zmian
- Linie zmienione: ~10
- Wersja: 1.12 → 1.13

---

## [1.12] - 2025-01-15

### 🔧 Poprawka: Przywracanie bottom navigation po zakończeniu sesji

#### Problem
Po zakończeniu quizu, fiszek lub trybu nauki menu dolne (bottom navigation) nie było wyświetlane na urządzeniach mobilnych.

#### Przyczyna
- CSS `display: none !important` miał wyższy priorytet niż `display: flex`
- Po usunięciu klas `.in-test`, `.in-practice`, `.in-flashcards` bottom navigation nie było przywracane
- Brak priorytetu dla wyświetlania bottom navigation gdy użytkownik jest zalogowany

#### Rozwiązanie
1. **Usunięto `!important` z CSS ukrywania bottom navigation**
   - Zmieniono `display: none !important` na `display: none`

2. **Dodano priorytet dla wyświetlania bottom navigation**
   - Dodano CSS: `body.logged-in:not(.in-test):not(.in-practice):not(.in-flashcards) .bottom-nav { display: flex !important; }`
   - To CSS ma wyższą specyficzność (5 selektorów) i priorytet `!important`

3. **Dodano klasę `has-bottom-nav` do body**
   - Klasa jest dodawana przy logowaniu i przywracaniu sesji
   - Klasa jest usuwana przy wylogowaniu

#### Zmiany w JavaScript

**Funkcje zmodyfikowane:**
- `restoreSession()`: Dodano `document.body.classList.add('has-bottom-nav')`
- Event listener login form: Dodano `document.body.classList.add('has-bottom-nav')`
- `logout()`: Dodano `document.body.classList.remove('has-bottom-nav')`

#### Zmiany w CSS

**Nowe reguły CSS:**
```css
/* Ukrywanie - bez !important */
body.in-test .bottom-nav,
body.in-practice .bottom-nav,
body.in-flashcards .bottom-nav {
    display: none;
}

/* Priorytet - pokaż bottom navigation dla zalogowanych użytkowników */
body.logged-in:not(.in-test):not(.in-practice):not(.in-flashcards) .bottom-nav {
    display: flex !important;
}
```

**Zmienione reguły CSS:**
- Usunięto `!important` z ukrywania bottom navigation
- Zwiększono specyficzność z `.logged-in .bottom-nav` na `body.logged-in .bottom-nav`

#### Statystyki zmian
- Linie zmienione: ~5
- Wersja: 1.11 → 1.12

---

## [1.11] - 2025-01-15

### 📱 Poprawki responsywności mobile (duże zmiany)

#### Problem główny
- Bottom navigation zasłaniało przyciski "Zakończ test", "Zakończ naukę" i przyciski oceny w trybie fiszek na urządzeniach mobilnych
- Fiszki zajmowały za dużo miejsca i nie miały scrollowania dla długich treści
- Tryb nauki (practice) miał słabą czytelność na małych ekranach

#### Rozwiązania wdrożone

**1. Automatyczne ukrywanie bottom navigation podczas sesji**
- Dodano klasy CSS: `.in-test`, `.in-practice`, `.in-flashcards`
- Bottom navigation jest teraz automatycznie ukrywane podczas:
  - Trwania testu
  - Sesji nauki (tryb practice)
  - Pracy z fiszkami
- Przywracanie bottom navigation po zakończeniu sesji

**2. Sticky positioning dla przycisków "Zakończ"**
- Przyciski "Zakończ test" i "Zakończ naukę" mają teraz `position: sticky`
- Zawsze widoczne na dole ekranu nad bottom navigation
- Cień (box-shadow) dla lepszej widoczności

**3. Poprawa wyświetlania fiszek na mobile**
- Dodano scrollowanie dla długich treści w fiszkach:
  - Mobile (max-width: 768px): max-height: 250px
  - Small mobile (max-width: 480px): max-height: 200px
- Mniejsze przyciski oceny na małych ekranach:
  - Mobile: font-size: var(--text-sm), padding: 12px 16px
  - Small mobile: font-size: var(--text-xs), padding: 10px 12px
- Dodano padding (margin-bottom: 80px) dla kontenera fiszek gdy bottom navigation jest widoczne

**4. Przeniesienie opisu klawiszy w trybie fiszek**
- Opis klawiszy (`.flashcard-keyboard-hint`) został przeniesiony:
  - Z: przed fiszką
  - Na: dół ekranu (po przyciskach oceny)
- Mniejszy rozmiar tekstu na małych ekranach:
  - Mobile: font-size: 11px
  - Small mobile: font-size: 10px
- Mniejsze elementy kbd na small mobile:
  - Mobile: padding: 3px 6px, font-size: 10px
  - Small mobile: padding: 2px 5px, font-size: 9px

**5. Poprawa trybu nauki (practice mode)**
- Przycisk "Zakończ naukę" przeniesiony poza kontener `.practice-stats`
- Sticky positioning dla przycisku "Zakończ naukę"
- Dodano marginesy dla lepszej czytelności:
  - `.practice-stats`: margin-bottom: 20px (mobile), 15px (small mobile)
  - `#practice-question-container`: margin-bottom: 15px (mobile), 12px (small mobile)
  - `#practice-feedback`: margin-bottom: 15px (mobile), 12px (small mobile)

#### Zmiany w JavaScript

**Funkcje zmodyfikowane:**
- `startTest()`: Dodano `document.body.classList.add('in-test')`
- `finishTest()`: Dodano `document.body.classList.remove('in-test')`
- `startPractice()`: Dodano `document.body.classList.add('in-practice')`
- `finishPractice()`: Dodano `document.body.classList.remove('in-practice')`
- `startFlashcards()`: Dodano `document.body.classList.add('in-flashcards')`
- `exitFlashcards()`: Dodano `document.body.classList.remove('in-flashcards')`

#### Zmiany w CSS

**Nowe reguły CSS:**
```css
/* Ukrywanie bottom navigation */
body.in-test .bottom-nav,
body.in-practice .bottom-nav,
body.in-flashcards .bottom-nav {
    display: none !important;
}

/* Sticky positioning */
@media (max-width: 768px) {
    #test-interface button[onclick="window.finishTest()"],
    #practice-interface button[onclick="window.endPracticeEarly()"] {
        position: sticky;
        bottom: 20px;
        z-index: 9998;
        margin-top: 20px;
        margin-bottom: 20px;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
    }
}

/* Scroll dla treści fiszek */
.flashcard-content {
    overflow-y: auto;
    max-height: 250px;
}
```

#### Statystyki zmian
- Linie dodane: ~135
- Plik oryginalny: 492 KB
- Plik zmodyfikowany: 496 KB
- Wersja: 1.1 → 1.11

---

## [1.1] - Wersja bazowa
- Pierwsza wersja z zapisanymi backupami
- Dostępne motywy: klasyczny, neon (cyberpunk), minimalistyczny
- Tryby: test, nauka (practice), fiszki
- System użytkowników z uprawnieniami admin
- Import/Export pytań (CSV, JSON)
- Statystyki i wyniki
- Notatki do pytań
- Oznaczanie pytań do powtórki

---

## Konwencje wersjonowania

### Minor (druga cyfra) - np. 1.1 → 1.2
Nowe funkcjonalności lub znaczące zmiany:
- Nowe tryby nauki
- Nowe rodzaje pytań
- Zmiana architektury
- Duże ulepszenia UI/UX

### Patch (trzecia cyfra) - np. 1.11 → 1.12
Drobne korekty i poprawki:
- Poprawki responsywności
- Poprawki błędów (bug fixes)
- Zmiany w CSS
- Optymalizacje wydajności
- Poprawki dostępności

### Major (pierwsza cyfra) - np. 1.x → 2.0
Największe zmiany:
- Całkowita zmiana interfejsu
- Nowa architektura aplikacji
- Zmiana technologii
- Migracja danych
- Zmiana modelu danych
