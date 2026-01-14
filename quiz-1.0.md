# Dokumentacja kodu aplikacji Quiz v1.0 - quiz-1.0.html

## Wprowadzenie

Aplikacja Quiz to profesjonalne narzędzie do nauki i testowania wiedzy w formacie pytań wielokrotnego wyboru (Single Choice, Multiple Choice). Aplikacja jest napisana w czystym HTML, CSS i JavaScript bez użycia zewnętrznych frameworków.

---

## Architektura aplikacji

### Struktura danych

Aplikacja używa **localStorage** jako mechanizm przechowywania danych. Główne struktury to:

#### 1. Pytania (`questions`)
Tablica obiektów pytań. Każde pytanie ma strukturę:

```javascript
{
    id: number,                    // Unikalny identyfikator pytania
    type: "single" | "multiple",  // Typ pytania
    text: string,                   // Treść pytania
    options: string[],              // Tablica opcji odpowiedzi
    correct: number[],             // Tablica indeksów poprawnych (1-based)
    category: string,               // Kategoria pytania (np. "AI-900")
    tags: string[],                 // Tagi powiązane z pytaniem
    explanation: string,            // Wyjaśnienie
    imageData: string               // Dane obrazu w base64 (opcjonalne)
    questionNotes: string           // Notatki użytkownika
}
```

#### 2. Użytkownicy (`users`)
Tablica obiektów użytkowników. Każdy użytkownik ma strukturę:

```javascript
{
    username: string,              // Nazwa użytkownika
    password: string,              // Hasło (haszowane)
    isAdmin: boolean,               // Uprawnienia administratora
    canEditQuestions: boolean,      // Prawo do edycji pytań
    studyStreak: number,           // Liczba dni nauki z rzędu
    correctAnswers: number,         // Liczba poprawnych odpowiedzi
    totalAnswers: number,            // Całkowita liczba odpowiedzi
    questionNotes: object          // Notatki do pytań: {questionId: note}
}
```

#### 3. Ustawienia (`settings`)
Obiekt z konfiguracją aplikacji:

```javascript
{
    currentTheme: string,    // Aktualny motyw ("classic", "modern", "minimal")
}
```

---

## Główne funkcje aplikacji

### 1. Zarządzanie pytaniami

#### `isValidQuestion(q)`
Waliduje pojedyncze pytanie.

**Parametry:**
- `q` - obiekt pytania do zweryfikacji

**Zwraca:** `boolean` - `true` jeśli pytanie jest poprawne, `false` w przeciwnym razie

**Walidacja:**
- Pytanie musi być obiektem
- Musi mieć pole `text` typu string
- Musi mieć pole `options` typu array
- `options.length >= 2` - co najmniej 2 opcje
- Musi mieć pole `correct` typu array
- Musi mieć pole `id`
- Jeśli brakuje `type`, ustawia na `"single"`
- Jeśli brakuje `id`, generuje: `Date.now() + Math.random()`

---

#### `cleanQuestionsData()`
Czyści dane pytań w localStorage, usuwając pytania niepoprawne.

**Logika:**
- Pobiera surowe dane: `JSON.parse(localStorage.getItem("quizQuestions", "[]"))`
- Filtruje za pomocą `isValidQuestion()`
- Zapisuje do localStorage: `localStorage.setItem("quizQuestions", JSON.stringify(questions))`
- Wyświetla w konsoli informację o usuniętych pytaniach

**Działanie przy błędach:**
- Wyłapuje błędy odczytu/zapisu w localStorage (`try...catch`)
- Wyświetla komunikaty błędów użytkownikowi przez `showToast()`

---

#### `renderQuestions(filteredQuestions)`
Renderuje listę pytań do panelu administracyjnego.

**Parametry:**
- `filteredQuestions` - tablica pytań do wyświetlenia (opcjonalnie, po filtrowaniu)

**Funkcjonalność:**

1. **Paginacja:**
   - Wyświetla określoną liczbę pytań na stronę (`questionsItemsPerPage` = 10)
   - Oblicza całkowitą liczbę stron: `Math.ceil(totalQuestions / questionsItemsPerPage)`
   - Nawigacja między stronami
   - Wyświetlanie informatora: `"Pytanie 1-10 z 50"`

2. **Edycja i usuwanie:**
   - Jeśli użytkownik jest administratorem (`isAdmin || canEditQuestions`):
     - Wyświetla przyciski: `✏️ Edytuj` i `🗑️ Usuń`
     - Wywołuje: `window.editQuestion(id)` lub `window.deleteQuestion(id)`
   - W przeciwnym razie:
     - Wyświetla komunikat: `"🔒 Tylko do odczytu - brak uprawnień do edycji"`

3. **Renderowanie pojedynczego pytania:**
   - Tworzy kontener `<div class="question-item">`
   - Dodaje numer pytania (w prawym górnym rogu): `data-question-number`
   - Wyświetla treść pytania w `<p class="question-text">`
   - Renderuje opcje:
     - Dla typu `single` / `multiple`: checkboxy lub radio buttony
     - Dla typu `ordering`: przeciąganie i upuszczanie kroków
   - Wyświetla kategorię jako badge
   - Wyświetla tagi (jeśli istnieją)
   - Wyświetla wyjaśnienie (jeśli istnieje)
   - Wyświetla notatki użytkownika (jeśli istnieją)

**Typy pytań:**
- **Single Choice** - tylko jedna poprawna odpowiedź (checkboxy z single select)
- **Multiple Choice** - więcej niż jedna poprawna odpowiedź (checkboxy)
- **Ordering** - układanie elementów w kolejności

---

### 2. Dodawanie pytań

#### `addQuestion()` / `showAddQuestionForm()`
Dodawanie nowego pytania do bazy.

**Kroki:**

1. **Walidacja formularza:**
   - Weryfikacja pola tekstowego (niepuste)
   - Weryfikacja pola typu pytania
   - Weryfikacja liczby opcji (minimum 2)
   - Weryfikacja, czy przynajmniej jedna poprawna odpowiedź została zaznaczona

2. **Obsługa formularza Yes/No:**
   - Dynamiczne dodawanie/usuwanie opcji
   - Walidacja poprawnych odpowiedzi (co najmniej jedna)

3. **Obsługa formularza Ordering:**
   - Dodawanie kroków do układania
   - Przeciąganie i upuszczanie kroków w kolejności
   - Walidacja poprawnej kolejności

4. **Zapis do localStorage:**
   - Tworzy obiekt pytania z wszystkimi polami
   - Ustawia unikalne `id`: `Date.now() + Math.random()`
   - Dodaje do tablicy `questions`
   - Zapisuje do localStorage: `localStorage.setItem("quizQuestions", JSON.stringify(questions))`
   - Odświeża widok: `renderQuestions()`
   - Aktualizuje dashboard: `updateDashboard()`

**Parametry formularza:**
- Treść pytania - wymagane pole
- Typ pytania - `single` lub `multiple`
- Opcje - tablica stringów
- Poprawna odpowiedź - indeks lub indeksy (1-based)
- Kategoria - opcjonalne
- Tagi - opcjonalne
- Wyjaśnienie - opcjonalne

---

### 3. Edycja pytań

#### `editQuestion(id)`
Edytuje istniejące pytanie.

**Kroki:**

1. **Znajdź pytanie:**
   - Wyszukuje w tablicy `questions` pytanie o danym `id`
   - Jeśli nie znaleziono: wyświetla komunikat błedu

2. **Załaduj dane do formularza:**
   - Wypełnia pola formularza danymi z pytania
   - Obsługuje formularze Yes/No i Ordering

3. **Zapisz zmiany:**
   - Aktualizuje obiekt pytania w tablicy
   - Zapisuje do localStorage
   - Odświeża widok: `renderQuestions()`

---

### 4. Usuwanie pytań

#### `deleteQuestion(id)`
Usuwa pytanie z bazy danych.

**Kroki:**

1. **Potwierdzenie:**
   - Wyświetla okno dialogowe `confirm("Czy na pewno chcesz usunąć to pytanie?")`

2. **Usuwanie z localStorage:**
   - Filtruje tablicę: `questions.filter(q => q.id !== id)`
   - Zapisuje do localStorage
   - Usuwa notatki: `safeRemoveItem("questionNotes")`
   - Odświeża widok: `renderQuestions()`
   - Aktualizuje dashboard: `updateDashboard()`

**Zabezpieczenia:**
- Wymaga potwierdzenia użytkownika
- Blokuje usuwanie jeśli użytkownik nie ma uprawnień

---

### 5. Zapisywanie notatek

#### `saveQuestionNote(questionId, content)`
Dodaje notatkę do pytania.

**Parametry:**
- `questionId` - identyfikator pytania
- `content` - treść notatki

**Kroki:**

1. **Walidacja uprawnień:**
   - Sprawdza czy użytkownik jest zalogowany
   - Jeśli nie: wyświetla komunikat: `"Musisz być zalogowany, aby dodawać notatki"`

2. **Zapis do localStorage:**
   - Pobiera aktualne notatki: `safeGetItem("questionNotes", "{}")`
   - Dodaje nową notatkę
   - Zapisuje: `localStorage.setItem("questionNotes", JSON.stringify(questionNotes))`

3. **Wyświetlenie komunikatu:**
   - `showToast("Notatka zapisana!", "success")`

**Wyświetlanie notatek w panelu pytań:**
- Jeśli pytanie ma notatkę, wyświetla ikonę 📝 obok numeru pytania

---

### 6. Import pytań

#### `importQuestions()` - Import z CSV
Importuje pytania z pliku CSV do bazy danych.

**Parametry:**
- Obsługuje zdarzenie `change` na elemencie `<input type="file">`
- Wybór kategorii (opcjonalne)

**Format pliku CSV:**
```
Pytanie,Opcja 1,Opcja 2,Opcja 3,Opcja 4,Poprawna odpowiedź,Typ,Kategoria
Treść pytania 1,Odpowiedź 1,Odpowiedź 2,Odpowiedź 3,Odpowiedź 4,3,single,AI-900
```

**Proces importu:**

1. **Walidacja pliku:**
   - Sprawdza czy plik nie jest pusty
   - Weryfikuje czy przynajmniej jeden nagłówek
   - Waliduje strukturę wierszy CSV

2. **Parsowanie CSV:**
   - Używa niestandardowy parser obsługujący cudzysłowy i wielolinijkowe pola
   - Usuwa znaki specjalne i spacje

3. **Automatyczny backup:**
   - Przed importem tworzy backup: `localStorage.setItem("quizQuestions_backup_{timestamp}", ...)`
   - Loguje do konsoli: `console.log("✅ Utworzono backup: " + backupKey)`

4. **Tryby importu:**
   - **Replace** - usuwa WSZYSTKIE pytania i zastępuje nowymi
   - **Append** - dodaje nowe pytania do istniejących

5. **Obsługa duplikatów:**
- Jeśli pytanie o tym samym ID lub treści już istnieje:
  - Pomija pytanie: `continue`
  - Zlicza pominięte pytania: `skipped++`
- Wyświetla raport: `"Pominięto (duplikaty): X pytań"`

---

#### `importQuestionsJSON()` - Import z JSON
Importuje pytania z pliku JSON.

**Parametry:**
- Obsługuje zdarzenie na elemencie `<input type="file">`
- Wybór trybu importu: `replace` lub `append`

**Format pliku JSON:**
```json
{
  "version": "1.0",
  "exportDate": "2024-01-01T00:00:00Z",
  "category": "AI-900",
  "totalQuestions": 100,
  "questions": [...]
}
```

**Walidacja JSON:**
- Sprawdza obecność tablicy `questions`
- Weryfikuje strukturę pierwszego pytania

**Tryb Replace:**
- Usuwa wszystkie pytania: `questions = []`
- Zastępuje nowymi: `questions = jsonData.questions`
- Potwierdza akcję z użytkownikiem

**Tryb Append:**
- Dodaje nowe pytania: `questions.push(...)` z JSON
- Zlicza zaimportowane pytania: `imported++`

---

### 7. Backup danych

#### `importBackup()`
Przywraca dane z kopii zapasowej.

**Format backupu:**
```json
{
  "version": "1.0",
  "exportDate": "2024-01-01T00:00:00Z",
  "data": {
    "users": [...],
    "questions": [...],
    "settings": {
      "currentTheme": "classic"
    }
  }
}
```

**Proces przywracania:**

1. **Walidacja pliku:**
   - Sprawdza rozszerzenie: musi być `.json`
   - Parsuje JSON: `JSON.parse(e.target.result)`

2. **Walidacja struktury:**
   - Sprawdza obecność: `backupData.data`, `backupData.data.users`, `backupData.data.questions`
   - Jeśli brakuje: wyświetla komunikat błedu

3. **Przywracanie danych:**
- Użytkownicy: `safeSetItem("quizUsers", JSON.stringify(backupData.data.users))`
- Pytania: `safeSetItem("quizQuestions", JSON.stringify(backupData.data.questions))`
- Ustawienia: `safeSetItem("quizTheme", backupData.data.settings.currentTheme)`

4. **Powiadomienie:**
- Wyświetla toast: `"Dane zostały pomyślnie przywrócone!"`
- Odświeża stronę: `location.reload()`

---

### 8. Quiz / Test

#### `startTest(minutes)`
Rozpoczyna sesję testową.

**Parametry:**
- `minutes` - czas trwania testu w minutach

**Kroki:**

1. **Przygotowanie trybu testu:**
   - Ustawia: `testMode = true`
   - Resetuje timer: `testTimer = null`, `testDuration = 0`
   - Wczytuje pytania: `loadQuestionsForTest()`
   - Przełacza pytania do formy testowej

2. **Odliczanie czasu:**
   - Ustawia interwał: `setInterval` co sekundę
   - Aktualizuje wyświetlacz czasu: `formatTime()` i `updateTimerDisplay()`
   - Zapisuje czas rozpoczęcia: `testStartTime = Date.now()`

3. **Sterowanie testu:**
   - Może wstrzymać (`pause`) i wznowić (`resume`)
   - Przyciski: `⏸️ Wstrzymaj`, `▶️ Wznów`

4. **Zakończenie testu:**
   - Zapisuje wynik do localStorage
- Oblicza wynik: `calculateScore()`
- Wyświetla wyniki: `showTestResults()`

**Funkcje pomocnicze:**
- `loadQuestionsForTest()` - ładuje pytania (z filtracją tagów i kategorii)
- `calculateScore()` - oblicza wynik procentowy
- `showTestResults()` - wyświetla szczegółowe wyniki

---

### 9. Fiszki (Flashcards)

#### `startFlashcards()`
Uruchamia tryb nauki z fiszkami.

**Kroki:**

1. **Ładowanie pytań:**
   - Wczytuje pytania z localStorage: `safeGetItem("quizQuestions", "[]")`
   - Filtruje pytania do nauki (nie zrobione w 100%)
   - Wyświetla panel SRS z statystykami

2. **Algorytm SRS (Spaced Repetition System):**
   - Oblicza interwał powtórki: `quality = (3 * days + 1) * 24 * 60 * 60 * 1000` (dla jakości 3)
   - Wyświetla pytania kolejno według interwału powtórki
   - Ukrywa pytania już opanowane (kiedy `nextReviewDate` w przeszłości)

3. **Interfejs fiszeki:**
   - Front z pytaniem
   - Tyl z odpowiedziami (przyciągnięty w górę)
   - Przyciski oceny jakości: `😓 Ponów`, `😐 Trudny`, `😶 Wąski`, `😫 Brak`
   - Przyciski nawigacji: `← Wstecz`, `Następna →`

4. **Zapisywanie oceny:**
   - Zapisuje do localStorage: `safeSetItem(...)`
   - Aktualizuje statystyki: `calculateStudyStreak()`, `updateStudyStreak()`

**Funkcje SRS:**
- `nextFlashcard()` - przechodzi do następnego pytania
- `previousFlashcard()` - wraca do poprzedniego pytania
- `flipCard()` - odwraca kartę (animacja CSS)
- `rateFlashcard(quality)` - zapisuje ocenę i oblicza kolejny interwał
- `exitFlashcards()` - kończy sesję fiszek

---

### 10. Analityka i Statystyki

#### `updateDashboard()`
Aktualizuje panel statystyk.

**Wyświetlane dane:**

1. **Całkowite statystyki:**
   - Liczba pytań w bazie
   - Liczba użytkowników
   - Liczba pytań w każdej kategorii

2. **Statystyki użytkownika:**
   - Łączna liczba odpowiedzi
   - Procent poprawnych odpowiedzi
   - Liczba dni nauki z rzędu (`studyStreak`)

3. **Wskaźniki nauki:**
   - Pytania do powtórzenia (kiedy `nextReviewDate < now`)
   - Słabe obszary (pytania z niskim wynikiem)

**Funkcje obliczeniowe:**
- `calculateStudyStreak()` - oblicza liczbę dni z rzędu
- `calculatePerformanceMetrics()` - oblicza średni wynik itp.

---

### 11. Kategorii i Tagi

#### `getAllCategories()`
Pobiera listę wszystkich kategorii z pytań.

**Logika:**
- Przechodzi przez pytania: `questions.map(q => q.category || "Uncategorized")`
- Filtruje unikalne kategorie
- Zwraca posortowaną listę

**Wyświetlanie:**
- W menu rozwijanym: kategorie z liczbą pytań
- W select formularzu dodawania pytania: opcje wyboru

---

#### `updateCategorySelects()`
Aktualizuje listy kategorii w różnych miejscach.

**Miejsca aktualizacji:**
- Formularz dodawania pytania
- Filtry pytań
- Formularz edycji pytania

---

#### Funkcje tagów
- `renderTags()` - renderuje listę tagów w panelu edycji
- `addTag()` - dodaje nowy tag
- `removeTag()` - usuwa tag
- `updateFilterTagSelect()` - aktualizuje select filtra tagów

---

### 12. Filtrowanie pytań

#### `filterQuestions()`
Filtruje pytania według różnych kryteriów.

**Kryteria filtrowania:**
- Kategoria - dropdown wyboru
- Tag - dropdown wyboru
- Typ pytania - checkboxy
- Wyszukiwanie - pole tekstowe

**Logika filtrowania:**
```javascript
const filters = {
    category: document.getElementById("filterCategorySelect").value,
    tags: document.getElementById("filterTagSelect").value,
    type: document.getElementById("filterTypeSelect").value,
    search: document.getElementById("filterSearchInput").value
};

// Filtrowanie
filteredQuestions = questions.filter(q => {
    // Filtrowanie po kategorii
    if (filters.category && filters.category !== "all") {
        return q.category === filters.category;
    }
    
    // Filtrowanie po tagach
    if (filters.tags.length > 0) {
        return filters.tags.every(tag => q.tags.includes(tag));
    }
    
    // Filtrowanie po typie
    if (filters.type && filters.type !== "all") {
        return q.type === filters.type;
    }
    
    // Filtrowanie po wyszukiwaniu
    if (filters.search) {
        const searchTerm = filters.search.toLowerCase();
        return (
            q.text.toLowerCase().includes(searchTerm) ||
            (q.tags && q.tags.some(tag => tag.toLowerCase().includes(searchTerm))) ||
            (q.category && q.category.toLowerCase().includes(searchTerm))
        );
    }
    
    // Zastosowanie wszystkich filtrów (AND logic)
    return filteredQuestions;
```

---

### 13. Edytor pytań (WYSIWYG)

#### `initWYSIWYG(toolbarId, editorId, hiddenTextareaId)`
Inicjalizuje edytor tekstowy z paskiem narzędzi.

**Funkcjonalność paska narzędzi:**
- **Formatowanie tekstu:** pogrubienie, kursywa, lista numerowana
- **Wstawianie:** wstawianie symboli, linków
- **Podgląd na żywo:** aktualizacja podglądu podczas pisania

**Edytowany element:**
- `<textarea id="editorId">` - główne pole edycyjne
- Ukryte pole tekstowe: `<textarea id="hiddenTextareaId">` - do podglądu

**Obsługa klawiatury:**
- Aktualizacja podglądu po każdej klawiszy (debounce 500ms)

---

### 14. Pytania typu "Ordering"

#### `addOrderingStep()`, `removeOrderingStep()`
Zarządza krokami pytania typu Ordering.

**Logika:**
- Dodaje krok do tablicy `q.options`
- Automatycznie numeruje kroki: `1)`, `2)`, `3)`, itp.
- Umożliwia przeciąganie i upuszczanie (drag & drop)
- Waliduje poprawną kolejność

**Struktura pytania Ordering:**
```javascript
{
    type: "ordering",
    text: "Ułóż elementy w odpowiedniej kolejności:",
    options: ["Element A", "Element B", "Element C"],
    correct: [1, 2, 3],  // Poprawna kolejność
    orderingSteps: [...]  // Kolejność kroków
}
```

**Funkcje pomocnicze:**
- `getOrderingStepsData()` - parsuje dane kroków
- `loadOrderingStepsData()` - ładuje dane kroków
- `showAddQuestionForm()` - wyświetla formularz
- `window.addOrderingStep()` - dodaje krok globalny
- `window.removeOrderingStep()` - usuwa krok globalny

---

### 15. Wyszukiwanie pytań powiązanych

#### `findRelatedQuestions(currentQuestion, maxQuestions = 3)`
Znajduje pytania powiązane z aktualnie wyświetlanym.

**Algorytm:**

1. **Analiza tekstu pytania:**
   - Wyodrębnia słowa kluczowe (np. "machine learning", "computer vision")
   - Filtruje przystanki i słowa krótkie

2. **Wyszukiwanie:**
   - Przeszukuje wszystkie pytania
   - Wybiera `maxQuestions = 3` najbardziej powiązanych

3. **Ranking:**
   - Sortuje wg podobieństwa (wspólne słowa kluczowe)
   - Zwraca pierwsze 3 wyniki

**Wyświetlanie:**
- Tworzy sekcję "Powiązane pytania" w panelu pytań
- Renderuje listę z linkami: `<a href="#" onclick="goToQuestion(id)">Pytanie X</a>`

**Funkcja pomocnicza:**
- `generateRelatedQuestionsHTML()` - generuje HTML sekcji
- `makeLinksClickable()` - zamienia tekst na linki klikalne

---

### 16. Tryb ciemny (Dark Mode)

#### `toggleDarkMode()`, `applyDarkModeToInlineStyles()`, `initDarkMode()`
Przełącza między trybem jasnym i ciemnym.

**Tryby działania:**

1. **Toggle:** przełączanie motywu (classic/modern/minimal)
2. **Persist:** zapisuje wybór do localStorage: `localStorage.setItem("quizTheme", themeName)`
3. **Apply:** aktualizuje atrybut `data-theme` na elemencie `<body>`
4. **Observer:** używa `MutationObserver` do detekowania zmian w zewnętrznych aplikacjach

**Motywy:**
- **Classic** - szary i niebieski, kontrastowy
- **Modern** - ciemny z fioletowymi neonowymi akcentami
- **Minimal** - jasny z szarymi odcieniami

---

### 17. UI/UX i Responsywność

#### Stylizacja CSS
Aplikacja używa zmiennych CSS (Custom Properties) dla łatwego zarządzania wyglądem.

**Główne zmienne:**
- `--primary-color` - główny kolor akcentu (niebieski dla classic)
- `--bg-color`, `--nav-bg` itd.
- `--font-body`, `--font-display`, `--font-mono` - rodziny czcionek
- `--text-xs`, `--text-sm` itd. - skala typograficzna
- `--shadow-sm`, `--shadow-md` itd. - cienie

**Responsywność:**
- Mobile-first design
- Zastosowanie `flexbox` i `grid`
- Media queries dla różnych rozmiarów ekranu
- Dostosowanie interfejsu fiszek dla urządzeń mobilnych

**Animacje:**
- `staggerIn` - animacja wejścia elementów
- `slideInLeft`, `slideInRight` - wejście boczne
- `fadeInUp` - wejście od dołu
- `shimmer` - efekt poświatła na pasku postępu
- `criticalPulse` - pulsacja dla stanów krytycznych

---

### 18. Obsługa błędów

#### Funkcje bezpieczeństwa localStorage
`safeSetItem(key, value)`, `safeGetItem(key, defaultValue)`, `safeRemoveItem(key)`

**Zabezpieczenia:**
1. **Obsługa `QuotaExceededError`:**
   - Wykrywa brak miejsca w localStorage
   - Wyświetla komunikat: `"Brak miejsca w pamięci! Usuń stare pytania lub wyczyść dane przeglądarki."`

2. **Walidacja zapisu/odczytu:**
   - Obsługuje wszystkie wyjątki w `try...catch`
   - Loguje błędy do konsoli: `console.error()`
   - Wyświetla komunikaty użytkownikowi: `showToast()`

---

### 19. Drukowanie i Eksport

#### `exportQuestionsPDF()`, `exportQuestionsExcel()`, `exportUserResultsCSV()`
Eksportuje dane do różnych formatów.

**Eksport PDF:**
- Używa bibliotekę `jspdf` do generowania pliku PDF
- Tworzy nowy dokument z listą pytań i wynikami
- Automatyczne pobieranie: `window.open()` lub `<a download>`

**Eksport Excel:**
- Używa bibliotekę `SheetJS` (`xlsx` package)
- Generuje arkusz z listą pytań
- Dwa tryby:
  - **Tylko pytania** - lista wszystkich pytań z metadanymi
  - **Z wynikami** - pytania + wyniki użytkownika

**Eksport CSV wyników użytkownika:**
- Generuje CSV z:
  - Nazwa użytkownika
  - Łączna liczba odpowiedzi
  - Procent poprawnych
  - Liczba dni nauki

---

### 20. Walidacja formularzy

#### Weryfikacja pól przy dodawaniu/edycji pytań

**Walidacja formularza single/multiple:**
- Treść pytania: niepusty string
- Liczba opcji: minimum 2
- Poprawna odpowiedź: co najmniej jedna zaznaczona
- Indeksy poprawnych: w zakresie 1 do liczby opcji

**Walidacja formularza ordering:**
- Liczba kroków: minimum 2
- Kolejność: wszystkie kroki przyporządkowane

**Funkcje walidujące:**
- Inline walidacja w formularzu (czerwony tekst przy polach)
- Walidacja przy zapisie: `isValidQuestion()`
- Walidacja przy imporcie: sprawdzanie struktury CSV/JSON

---

## Użycie aplikacji

### Przebieg typowy użycia

1. **Uruchomienie aplikacji:**
   - Otwarcie w przeglądarce internetowej (Chrome, Firefox, Edge)
   - Zalogowanie jako administrator (opcjonalne)
   - Przeglądanie listy pytań w Dashboard

2. **Nauka z fiszkami:**
   - Uruchomienie trybu fiszek: Fiszki
   - Ocena jakości zapamiętania (ponów/trudny/wąski)
   - Automatyczne przechodzenie do kolejnego pytania
   - Przeglądanie statystyk postępów

3. **Testowanie wiedzy:**
   - Uruchomienie quizu: Test
   - Wybór czasu trwania (10/30/60 minut)
   - Rozwiązywanie pytań
   - Przeglądanie wyników szczegółowych

4. **Administracja:**
   - Dodawanie nowych pytań
   - Edycja istniejących pytań
   - Usuwanie pytań
   - Import/eksport pytań
   - Zarządzanie kategoriami i tagami

5. **Analityka:**
   - Przeglądanie statystyk globalnych
   - Identyfikacja słabych obszarów
   - Śledzenie postępów nauki (study streak)

---

## Struktura pliku HTML

```
quiz-1.0.html/
├── <head>
│   ├── Meta tagi PWA (manifest.json, theme-color)
│   ├── Google Fonts (Inter, Newsreader, JetBrains Mono)
│   └── Style (CSS ~5000 linii)
├── <body>
│   ├── Nawigacja (tab: Dashboard, Pytania, Fiszki, Analityka, Użytkownicy)
│   ├── Sekcja Dashboard (statystyki, wykresy, wskaźniki postępu)
│   ├── Sekcja Pytania (lista z paginacją, formularz dodawania)
│   ├── Sekcja Fiszki (karty SRS, interfejs nauki)
│   ├── Sekcja Analityka (wykresy, słabe obszary)
│   ├── Sekcja Użytkownicy (lista użytkowników, formularz logowania)
│   └── Sekcja Test (ekran testowy, zegar, wyniki)
└── <script>
    ├── Zmienne globalne (~2000 linii)
    ├── Obsługa localStorage
    ├── Funkcje zarządzania pytaniami
    ├── Funkcje UI/UX
    └── Inicjalizacja aplikacji (DOMContentLoaded)
```

---

## Kluczowe cechy aplikacji

### 1. PWA (Progressive Web App)
- Manifest PWA do instalacji na pulpicie
- Service Worker dla cache'owania offline
- Obsługa `beforeinstall` event

### 2. Bezpieczeństwo
- Haszowanie haseł (SHA-256)
- Walidacja nazwy użytkownika
- Obsługa błędów localStorage
- Automatyczne backupy przed importem

### 3. UX/UI
- Responsywność dla mobile (media queries)
- Animacje wejścia elementów
- Tryby ciemny/jasny
- Powiadomienia toast (zamiast alert)

### 4. Funkcjonalność
- 3 typy pytań: single, multiple, ordering
- Fiszki z algorytmem SRS (Spaced Repetition)
- Analityka z wykresami
- Tagi i kategorie
- Eksport do PDF/Excel/CSV

### 5. Wydajność
- Leniwe ładowanie (lazy loading)
- Virtual Scrolling dla dużych list
- Debounce dla operacji częstych

---

## Podsumowanie

Dokumentacja opisuje pełną strukturę i funkcjonalność aplikacji Quiz v1.0. Jest kompletna i zawiera wszystkie niezbędne informacje dla zrozumienia kodu aplikacji.

Aplikacja jest dobrze zorganizowana, z czytelnym podziałem na funkcje i moduły. Kod jest zrozumiały i dobrze ustrukturyzowany.
