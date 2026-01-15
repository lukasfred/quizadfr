# Change Log - Aplikacja Quizowo-Testowa

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
