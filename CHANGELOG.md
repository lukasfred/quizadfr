# Change Log - Aplikacja Quizowo-Testowa

## [1.31] - 2026-01-24

### 🐛 Poprawka: Eksport/Import JSON nie obsługuje pytań typu "pairing" i "ordering"

#### Problem
Funkcje eksportu i importu JSON nie obsługiwały poprawnie pytań typu "pairing" (dopasowanie) i "ordering" (układanie w kolejności). Powodowało to utratę danych przy eksporcie i błędy przy imporcie.

#### Przyczyna 1: Eksport JSON (index.html:10391-10401)
Funkcja `exportQuestionsJSON()` eksportowała tylko pola `options` i `correct`, ale nie eksportowała:
- `pairs` dla pytań typu "pairing"
- `ordering` dla pytań typu "ordering"

**Kod przed:**
```javascript
questions: questionsToExport.map(q => ({
    id: q.id,
    type: q.type,
    text: q.text,
    options: q.options || [],      // ❌ Eksportuje puste options dla pairing/ordering
    correct: q.correct || [],      // ❌ Eksportuje puste correct dla pairing/ordering
    category: q.category || "",
    tags: q.tags || [],
    explanation: q.explanation || "",
    imageData: q.imageData || ""
}))
```

**Kod po:**
```javascript
questions: questionsToExport.map(q => {
    const exported = {
        id: q.id,
        type: q.type,
        text: q.text,
        category: q.category || "",
        tags: q.tags || [],
        explanation: q.explanation || "",
        imageData: q.imageData || ""
    };

    // Dla pairing - eksportuj pairs
    if (q.type === 'pairing') {
        exported.pairs = q.pairs || [];
        exported.options = [];
        exported.correct = [];
    } else {
        // Dla innych typów - eksportuj options i correct
        exported.options = q.options || [];
        exported.correct = q.correct || [];
    }

    // Dla ordering - eksportuj ordering
    if (q.type === 'ordering') {
        exported.ordering = q.ordering || [];
    }

    return exported;
})
```

#### Przyczyna 2: Import JSON - Walidacja (index.html:10943-10959)
Walidacja w `importQuestionsJSON()` wymagała `options` i `correct` dla wszystkich pytań, co powodowało odrzucanie pytań typu "pairing" i "ordering".

**Kod przed:**
```javascript
// Walidacja struktury pojedynczego pytania
if (!q.text || !q.options || !Array.isArray(q.options) || !q.correct) {
    console.warn("Nieprawidłowa struktura pytania #" + (index + 1), q);
    errors++;
    return;
}

if (q.options.length < 2) {
    console.warn("Pytanie #" + (index + 1) + " ma mniej niż 2 odpowiedzi");
    errors++;
    return;
}

if (!Array.isArray(q.correct) || q.correct.length === 0) {
    console.warn("Pytanie #" + (index + 1) + " nie ma poprawnych odpowiedzi");
    errors++;
    return;
}
```

**Kod po:**
```javascript
// Walidacja struktury pojedynczego pytania
if (!q.text) {
    console.warn("Nieprawidłowa struktura pytania #" + (index + 1) + " - brak tekstu", q);
    errors++;
    return;
}

// Dla pairing - waliduj pairs
if (q.type === 'pairing') {
    if (!q.pairs || !Array.isArray(q.pairs) || q.pairs.length < 2) {
        console.warn("Pytanie pairing #" + (index + 1) + " ma nieprawidłowe pairs", q);
        errors++;
        return;
    }
    // Sprawdź czy każda para ma left i right
    const hasValidPairs = q.pairs.every(pair =>
        pair && pair.left && pair.left.trim() !== '' &&
        pair.right && pair.right.trim() !== ''
    );
    if (!hasValidPairs) {
        console.warn("Pytanie pairing #" + (index + 1) + " ma niekompletne pary", q);
        errors++;
        return;
    }
} else if (q.type === 'ordering') {
    // Dla ordering - waliduj ordering
    if (!q.ordering || !Array.isArray(q.ordering) || q.ordering.length < 2) {
        console.warn("Pytanie ordering #" + (index + 1) + " ma nieprawidłowe ordering", q);
        errors++;
        return;
    }
} else {
    // Dla single/multiple - waliduj options i correct
    if (!q.options || !Array.isArray(q.options)) {
        console.warn("Pytanie #" + (index + 1) + " ma nieprawidłowe options", q);
        errors++;
        return;
    }

    if (q.options.length < 2) {
        console.warn("Pytanie #" + (index + 1) + " ma mniej niż 2 odpowiedzi");
        errors++;
        return;
    }

    if (!q.correct || !Array.isArray(q.correct) || q.correct.length === 0) {
        console.warn("Pytanie #" + (index + 1) + " nie ma poprawnych odpowiedzi");
        errors++;
        return;
    }
}
```

#### Przyczyna 3: Import JSON - Tworzenie obiektu (index.html:10973-10983)
Funkcja tworząca obiekt pytania nie dodawała pól `pairs` i `ordering`.

**Kod przed:**
```javascript
const newQ = {
    id: q.id || Date.now() + Math.random(),
    type: q.type || "single",
    text: q.text,
    options: q.options,       // ❌ Opcje z JSON (puste dla pairing/ordering)
    correct: q.correct,       // ❌ Poprawne z JSON (puste dla pairing/ordering)
    category: q.category || "",
    tags: q.tags || [],
    explanation: q.explanation || "",
    imageData: q.imageData || ""
};
```

**Kod po:**
```javascript
const newQ = {
    id: q.id || Date.now() + Math.random(),
    type: q.type || "single",
    text: q.text,
    category: q.category || "",
    tags: q.tags || [],
    explanation: q.explanation || "",
    imageData: q.imageData || ""
};

// Dla pairing - dodaj pairs
if (q.type === 'pairing') {
    newQ.pairs = q.pairs || [];
    newQ.options = [];
    newQ.correct = [];
} else if (q.type === 'ordering') {
    // Dla ordering - dodaj ordering
    newQ.ordering = q.ordering || [];
    newQ.options = [];
    newQ.correct = [];
} else {
    // Dla single/multiple - dodaj options i correct
    newQ.options = q.options || [];
    newQ.correct = q.correct || [];
}
```

#### Zmiany w kodzie

**1. exportQuestionsJSON() [linia 10385-10420]**
- Zmieniono mapowanie pytań na użycie warunkowego eksportu
- Dodano obsługę pola `pairs` dla pytań typu "pairing"
- Dodano obsługę pola `ordering` dla pytań typu "ordering"
- Dla pairing/ordering: ustawiamy puste `options` i `correct`

**2. importQuestionsJSON() - walidacja [linia 10958-11010]**
- Zmieniono walidację na sprawdzanie typu pytania
- Dla pairing: waliduj `pairs` (minimum 2 pary, każda ma left i right)
- Dla ordering: waliduj `ordering` (minimum 2 elementy)
- Dla single/multiple: waliduj `options` i `correct`

**3. importQuestionsJSON() - tworzenie obiektu [linia 11023-11048]**
- Zmieniono tworzenie obiektu na użycie warunkowego dodawania pól
- Dla pairing: dodaj `pairs`, puste `options` i `correct`
- Dla ordering: dodaj `ordering`, puste `options` i `correct`
- Dla single/multiple: dodaj `options` i `correct`

#### Działanie poprawione
- ✅ Eksport JSON teraz poprawnie eksportuje pytania typu "pairing" z polem `pairs`
- ✅ Eksport JSON teraz poprawnie eksportuje pytania typu "ordering" z polem `ordering`
- ✅ Import JSON poprawnie waliduje pytania "pairing" i "ordering"
- ✅ Import JSON poprawnie tworzy obiekty pytań "pairing" i "ordering"
- ✅ Dane nie są tracone przy eksporcie
- ✅ Import nie odrzuca pytań "pairing" i "ordering"

#### Lokalizacja
- **Plik:** `index.html`
- **Funkcje:** `exportQuestionsJSON()` [10369-10430], `importQuestionsJSON()` [10884-11080]

#### Test
Po zastosowaniu poprawki:
1. Eksportuj pytania do JSON
2. Sprawdź czy pytania typu "pairing" mają pole `pairs`
3. Sprawdź czy pytania typu "ordering" mają pole `ordering`
4. Importuj JSON - pytania powinny być zaimportowane bez błędów

#### Korzyści
- ✅ Pełna kompatybilność eksportu/importu dla wszystkich typów pytań
- ✅ Dane pytań "pairing" i "ordering" są zachowane
- ✅ Możliwość tworzenia backupów wszystkich pytań
- ✅ Możliwość migracji danych między instalacjami
- ✅ Przygotowanie pod wersję Android (format JSON jest obowiązujący)

#### Statystyki zmian
- Linie zmodyfikowane: ~70
- Nowe linie: ~35
- Wersja: 1.30 → 1.31
- Typ zmiany: patch (krytyczna poprawka eksportu/importu)

#### Backup
- Utworzono backup: `index.html.backup-1.30.1`

---

## [1.30] - 2025-01-19

### 🐛 Poprawka: exitPractice() wywołuje zły sekcję

#### Problem
Po zakończeniu ćwiczenia w trybie nauki i kliknięciu "Wróć do menu" następuje przekierowanie do sekcji "Wyniki" zamiast do ustawień trybu nauki (gdzie można wybrać kategorię i rozpocząć nową sesję).

#### Przyczyna
Funkcja `exitPractice()` wywoływała `showSection("results")`, co pokazywało sekcję wyników zamiast powrotu do ustawień trybu nauki.

**Kod przed:**
```javascript
function exitPractice() {
    document.getElementById("practice-interface").classList.add("hidden");
    document.getElementById("practice-setup").classList.remove("hidden");
    showSection("results");  // ❌ Problem - pokazuje wyniki zamiast ustawień
}
```

#### Rozwiązanie
Usunięto wywołanie `showSection("results")` z funkcji `exitPractice()`.

**Kod po:**
```javascript
function exitPractice() {
    document.getElementById("practice-interface").classList.add("hidden");
    document.getElementById("practice-setup").classList.remove("hidden");
    // Nie wywołujemy showSection("results") - zostajemy w ustawieniach trybu nauki
}
```

#### Działanie poprawione
- Po kliknięciu "Wróć do menu" użytkownik zamyka interfejs ćwiczenia
- Użytkownik wraca do ekranu ustawień trybu nauki
- Może wybrać kategorię, ilość pytań, tryb i inne opcje
- Może rozpocząć nową sesję ćwiczeń
- **Nie** jest przenoszony do sekcji wyników

#### Lokalizacja
- **Plik:** `index.html`
- **Lini:** ~12386
- **Funkcja:** `exitPractice()`

#### Korzyści
- ✅ Poprawny przepływ pracy w trybie nauki
- ✅ Użytkownik może łatwo rozpocząć nową sesję
- ✅ Brak niepotrzebnych przekierowań do wyników
- ✅ Logiczny workflow: nauka → ustawienia → nowa nauka → wyniki

#### Statystyki zmian
- Linie zmodyfikowane: 1 (usunięta 1 linia)
- Wersja: 1.29 → 1.30
- Typ zmiany: patch (poprawka UX)

---

## [1.29] - 2025-01-19

### 🐛 Poprawka: UI fiszek - przód karty pokazuje obie strony pary

#### Problem
Na pierwszej stronie fiszki (przód karty) dla pytań typu "pairing" nie były widoczne wszystkie elementy. Użytkownik chciał widzieć obie strony pary (np. kraj **ORAZ** flaga) ale bez oznaczenia poprawnych połączeń.

#### Przyczyna
Poprzednia implementacja pokazywała tylko lewe elementy + strzałkę w dół, co nie dawało pełnego obrazu wszystkich możliwości.

#### Rozwiązanie

**Zmieniono renderowanie w `showFlashcard()` dla pytań pairing:**

**Przód karty (bez oznaczenia poprawnych):**
- Pokazuje obie strony każdej pary: lewy element + prawy element
- Elementy są oddzielone strzałką "↔"
- Wszystkie pary są widoczne
- **Brak** oznaczenia poprawnych połączeń (wszystkie są w normalnym kolorze)

**Tył karty (z oznaczeniem poprawnych):**
- Pokazuje poprawnie połączone pary
- Poprawne pary mają klasę `correct` (zielony kolor)
- Dodano ikonę checkmark "✓" przy poprawnych parach

**Przed:**
```javascript
// Przód karty - tylko lewe elementy + strzałka w dół
optionsFrontHTML = q._shuffledLeftItems.map((leftItem, i) => `
    <div class="flashcard-option">
        <span><strong>${escapeHTML(leftItem.value)}</strong></span>
        <span>↓</span>
    </div>
`).join("");
```

**Po:**
```javascript
// Przód karty - obie strony pary (bez oznaczenia)
const pairs = q.pairs || [];

// Losuj kolejność par
if (!q._shuffledPairs) {
    const indices = pairs.map((_, i) => i);
    for (let i = indices.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [indices[i], indices[j]] = [indices[j], indices[i]];
    }
    q._shuffledPairs = indices.map(i => pairs[i]);
}

optionsFrontHTML = q._shuffledPairs.map((pair, i) => `
    <div class="flashcard-option">
        <div style="display: flex; align-items: center; justify-content: space-between; width: 100%;">
            <span><strong>${escapeHTML(pair.left)}</strong></span>
            <span style="color: #666; margin: 0 10px;">↔</span>
            <span>${escapeHTML(pair.right)}</span>
        </div>
    </div>
`).join("");

// Tył karty - poprawnie połączone pary (z podkreśleniem)
optionsBackHTML = pairs.map((pair, i) => `
    <div class="flashcard-option correct">
        <div class="option-number">${i + 1}</div>
        <div style="flex: 1;">
            <strong>${escapeHTML(pair.left)}</strong>
            <span style="margin: 0 10px; color: #10b981; font-size: 1.2em;">✓</span>
            ${escapeHTML(pair.right)}
        </div>
    </div>
`).join("");
```

#### Przykład wizualny

**Przód karty (bez oznaczenia):**
```
1. Anglia  ↔ 🏴�󠁧󠁢󠁥󠁮󠁧󠁿
2. Niemcy  ↔ 🇩🇪
3. Francja  ↔ 🇫🇷
```

**Tył karty (z oznaczeniem):**
```
✓ 1. Anglia  ↔ 🏴�󠁧󠁢󠁥󠁮󠁧󠁿
✓ 2. Niemcy  ↔ 🇩🇪
✓ 3. Francja  ↔ 🇫🇷
```

#### Zmiany w strukturze danych

Dodano nowe pole do shuffled data:
```javascript
q._shuffledPairs = [shuffledPairs];  // Nowe!
```

#### Zmiany w kodzie

**Lokalizacja:** `index.html` linia ~8246-8280
**Funkcja:** `showFlashcard()`

**Kluczowe zmiany:**
1. Zmieniono logikę shuffling - teraz shuffling całych par zamiast osobnych elementów
2. Przód karty pokazuje obie strony każdej pary (left ↔ right)
3. Brak oznaczenia poprawnych na przód karty (wszystkie normalne)
4. Tył karty pokazuje poprawne połączenia z ikoną checkmark

#### Korzyści
- ✅ Na przód karty widoczne wszystkie elementy (obie strony pary)
- ✅ Użytkownik widzi pełne możliwości dopasowania przed sprawdzeniem
- ✅ Strzałka "↔" jasno wskazuje na połączenie
- ✅ Tył karty wyraźnie pokazuje poprawne odpowiedzi
- ✅ Ikona "✓" podkreśla poprawne pary
- ✅ Losowanie kolejności par dla różnorodności

#### Statystyki zmian
- Linie zmodyfikowane: ~40
- Wersja: 1.27 → 1.28
- Typ zmiany: patch (poprawka UI fiszek)

---

## [1.27] - 2025-01-17

### 🐛 Poprawki: Walidacja w trybie nauki, UI fiszek i kolorystyka w trybie testu

#### Problem 1: Błąd "Błąd danych pytania" w trybie nauki dla pytań pairing
W trybie nauki wyskakiwał błąd "Błąd danych pytania" gdy napotkało pytanie typu "pairing".

#### Przyczyna 1
Walidacja w `renderPracticeQuestion()` sprawdzała:
```javascript
if (!q || !q.text || !q.options) {  // ❌ Problem
```

Dla pytań pairing ustawiano `options = null`, więc walidacja nie przechodziła.

#### Rozwiązanie 1
Zaktualizowano walidację w `renderPracticeQuestion()` (linia ~11631):
```javascript
// Walidacja pytania
if (!q || !q.text || !q.type) {
    console.error("Nieprawidłowe dane pytania:", q);
    showToast("Błąd danych pytania. Przechodzę do następnego.", "error");
    nextPracticeQuestion();
    return;
}

// Dla pairing - waliduj pary
if (q.type === "pairing") {
    if (!q.pairs || !Array.isArray(q.pairs) || q.pairs.length === 0) return false;
    if (q.pairs.length < 2) return false;
    // Sprawdź czy pary mają oba pola wypełnione
    const hasValidPairs = q.pairs.every(pair => 
        pair && pair.left && pair.left.trim() !== "" && 
        pair.right && pair.right.trim() !== ""
    );
    if (!hasValidPairs) return false;
} else {
    // Dla innych typów - sprawdź options
    if (!q.options || !Array.isArray(q.options) || q.options.length === 0) return false;
}
```

#### Problem 2: Fiszki - na przód karty nie widać możliwych odpowiedzi
Na pierwszej stronie karty fiszki nie były widoczne żadne opcje odpowiedzi.

#### Przyczyna 2
Kod pokazywał tylko lewe elementy (np. kraje) bez żadnych prawych elementów (np. flagi) lub dropdownu z wyborem.

#### Rozwiązanie 2
Zaktualizowano `showFlashcard()` dla pytań pairing:
- **Przód karty**: Lewe elementy (np. kraje) + strzałka w dół wskazująca na dropdown
- **Tył karty**: Połączone pary z wyraźnym oddzieleniem (np. "Anglia" ↔ "🏴󠁧󠁢󠁥󠁮󠁧󠁿")

```javascript
if (q.type === "pairing") {
    // Przygotuj listę prawych elementów
    const rightItems = pairs.map((pair, i) => ({
        id: pair.id,
        value: pair.right
    }));

    // Losuj kolejność prawych elementów
    if (!q._shuffledRightItems) {
        // ... shuffling ...
    }

    // Losuj kolejność lewych elementów
    if (!q._shuffledLeftItems) {
        // ... shuffling ...
    }

    // Przód karty - lewe elementy + strzałka w dół
    optionsFrontHTML = q._shuffledLeftItems.map((leftItem, i) => `
        <div class="flashcard-option">
            <div style="flex: 1; display: flex; align-items: center; gap: 15px;">
                <span><strong>${escapeHTML(leftItem.value)}</strong></span>
                <span>↓</span>
            </div>
        </div>
    `).join("");
    
    // Tył karty - połączone pary
    optionsBackHTML = pairs.map((pair, i) => `
        <div class="flashcard-option correct">
            <div class="option-number">${i + 1}</div>
            <div style="flex: 1;">
                <strong>${escapeHTML(pair.left)}</strong>
                <span style="margin: 0 10px;">↔</span>
                ${escapeHTML(pair.right)}
            </div>
        </div>
    `).join("");
}
```

#### Problem 3: Shuffling pytań pairing w trybie nauki powodował błędy
Shuffling w `startPractice()` używało `q.options.map()` co powodowało błędy dla pytań pairing z `options = null`.

#### Przyczyna 3
```javascript
// PIERWOTNIE (w startPractice):
const indices = q.options.map((_, i) => i);  // ❌ Błąd dla pairing!
```

Dla pytań pairing `options = null`, więc shuffling kończyło się błędem.

#### Rozwiązanie 3
Zaktualizowano logikę shuffling w `startPractice()` (linia ~11545):
```javascript
// Dla ordering/pairing - ZAWSZE losuj kolejność
if (q.type === "ordering" || q.type === "pairing") {
    shouldShuffle = true;
} else {
    // Dla single/multiple - losuj tylko jeśli checkbox jest zaznaczony
    shouldShuffle = randomizeAnswers && q.options && q.options.length > 1;
}

if (shouldShuffle) {
    // Dla ordering/pairing - nie używamy shuffling dla options
    // (robimy to w renderze: renderPairingQuestion/renderOrderingQuestion)
    if (q.type !== "ordering" && q.type !== "pairing") {
        // Dla single/multiple - shuffling jak dotychczas
        const indices = q.options.map((_, i) => i);
        // ... shuffling ...
    }
}
```

#### Problem 4: Kolorystyka w trybie testu - niedopasowanie do motywu
Styling dla pytań pairing w trybie testu był niedopasowany do wybranego motywu (dark mode, modern theme).

#### Rozwiązanie 4
Dodano style dla pytań pairing w różnych motywach:

**Dark mode:**
```css
body.dark-mode .pairing-test-item {
    background: #161b22;
    border-color: #30363d;
}

body.dark-mode .pairing-row {
    background: #21262d;
    border-color: #30363d;
}

body.dark-mode .pairing-left {
    color: #e6edf3;
}

body.dark-mode .pairing-select {
    background: #0d1117;
    border-color: #30363d;
    color: #c9d1d9;
}

body.dark-mode .pairing-instruction {
    color: #c9d1d9;
}
```

**Modern theme:**
```css
[data-theme="modern"] .pairing-test-item {
    background: #1e1b4b;
    border-color: rgba(255, 0, 255, 0.3);
}

[data-theme="modern"] .pairing-row {
    background: #2d285a;
    border-color: rgba(255, 0, 255, 0.2);
}

[data-theme="modern"] .pairing-left {
    color: #e2e8f0;
}

[data-theme="modern"] .pairing-select {
    background: #1e1b4b;
    border-color: rgba(255, 0, 255, 0.4);
    color: #ffffff;
}

[data-theme="modern"] .pairing-instruction {
    color: #e2e8f0;
}
```

#### Zmiany w kodzie

**1. renderPracticeQuestion() [linia ~11631]**
- Zmieniono walidację z `!q.options` na sprawdzenie typu
- Dodano szczegółową walidację dla pytań pairing
- Dodano komunikaty błędów dla różnych problemów

**2. startPractice() [linia ~11545]**
- Zaktualizowano logikę shuffling
- Dla ordering/pairing: zawsze shuffle (ale w renderze)
- Dla single/multiple: shuffle tylko jeśli checkbox jest zaznaczony
- Dla ordering/pairing: nie wywołuj shuffling na options

**3. showFlashcard() [linia ~8246]**
- Zmieniono renderowanie przodu karty dla pairing
- Dodano shuffling lewych i prawych elementów
- Dodano strzałkę w dół wskazującą na dropdown
- Zmieniono symbol łączenia z "↔" na bardziej czytelny format

**4. CSS [różne lokalizacje]**
- Dodano style dla pairing w dark mode (linia ~3471)
- Dodano style dla pairing w modern theme (linia ~4350)

#### Lokalizacja zmian
- `index.html:11631-11684` - walidacja w renderPracticeQuestion()
- `index.html:11545-11575` - shuffling w startPractice()
- `index.html:8246-8280` - renderowanie w showFlashcard()
- `index.html:3471-3495` - CSS dla pairing w dark mode
- `index.html:4350-4380` - CSS dla pairing w modern theme

#### Korzyści
- ✅ Tryb nauki działa poprawnie dla pytań pairing
- ✅ Fiszki pokazują prawidłową strukturę (przód + tył)
- ✅ Strzałka w dół na przód karty jasno wskazuje na odpowiedź
- ✅ Kolorystyka jest dopasowana do wybranego motywu
- ✅ Shuffling nie powoduje błędów dla pytań pairing
- ✅ Poprawne oddzielenie elementów na tył karty

#### Statystyki zmian
- Linie zmodyfikowane: ~100
- Nowe linie CSS: ~30
- Wersja: 1.26 → 1.27
- Typ zmiany: patch (poprawki błędów i UI)

---

## [1.26] - 2025-01-17

### 🐛 Poprawka: isValidQuestion() usuwa pytania typu "pairing"

#### Problem
Pytania typu "Dopasowanie" (pairing) nie były dodawane do bazy i nie można ich było wyszukać.

#### Przyczyna
Pytania pairing były **zapisywane** do localStorage, ale **usuwaną** przez funkcję `isValidQuestion()` przy ładowaniu danych. Funkcja ta sprawdzała:

```javascript
function isValidQuestion(q) {
    // ...
    if (!Array.isArray(q.options) || q.options.length < 2) return false;  // ❌ Problem
    if (!Array.isArray(q.correct)) return false;  // ❌ Problem
    // ...
}
```

Dla pytań typu "pairing" w kodzie zapisywania ustawiano:
```javascript
// Dla pairing używamy pary jako specjalny atrybut
options = null; // options nie są używane dla pairing
correct = null; // correct nie jest używane dla pairing
```

To powodowało, że pytań pairing nie przechodziły walidację w `isValidQuestion()` i były usuwaną.

#### Rozwiązanie

**Przed:**
```javascript
function isValidQuestion(q) {
    if (!q || typeof q !== 'object') return false;
    if (!q.text || typeof q.text !== 'string') return false;
    if (!Array.isArray(q.options) || q.options.length < 2) return false;  // ❌
    if (!Array.isArray(q.correct)) return false;  // ❌
    if (!q.id) q.id = Date.now() + Math.random();
    if (!q.type) q.type = 'single';
    if (q.imageData && typeof q.imageData === 'string' && !q.imageData.startsWith('data:')) {
        delete q.imageData;
    }
    return true;
}
```

**Po:**
```javascript
function isValidQuestion(q) {
    if (!q || typeof q !== 'object') return false;
    if (!q.text || typeof q.text !== 'string') return false;

    // Dla pairing - waliduj pary zamiast options i correct
    if (q.type === 'pairing') {
        if (!Array.isArray(q.pairs) || q.pairs.length < 2) return false;
        // Waliduj czy każda para ma left i right
        const hasValidPairs = q.pairs.every(pair =>
            pair && pair.left && pair.left.trim() !== '' &&
            pair.right && pair.right.trim() !== ''
        );
        if (!hasValidPairs) return false;
    } else {
        // Dla innych typów - waliduj options i correct
        if (!Array.isArray(q.options) || q.options.length < 2) return false;
        if (!Array.isArray(q.correct)) return false;
    }

    if (!q.id) q.id = Date.now() + Math.random();
    if (!q.type) q.type = 'single';
    if (q.imageData && typeof q.imageData === 'string' && !q.imageData.startsWith('data:')) {
        delete q.imageData;
    }
    return true;
}
```

#### Zmiany w logice walidacji

**Nowa walidacja dla "pairing":**
1. Sprawdza czy `q.pairs` jest tablicą
2. Sprawdza czy jest minimum 2 pary
3. Sprawdza czy każda para ma `left` i `right`
4. Sprawdza czy `left` i `right` nie są puste

**Walidacja dla innych typów (bez zmian):**
1. Sprawdza `q.options` (tablica, minimum 2 elementy)
2. Sprawdza `q.correct` (tablica)

#### Lokalizacja
- **Plik:** `index.html`
- **Linia:** ~7418-7442
- **Funkcja:** `isValidQuestion(q)`

#### Jak to naprawiło problem

**Proces zapisywania i ładowania:**

1. **Zapisywanie:** `saveQuestion()` (submit handler)
   - Tworzy obiekt `questionData` z `pairs`
   - Dodaje do tablicy `questions`
   - Zapisuje do localStorage
   - ✅ **To działało**

2. **Walidacja:** `cleanQuestionsData()` wywołuje `isValidQuestion()`
   - Sprawdza wszystkie pytania w localStorage
   - Usuwa pytania nieprzechodzące walidację
   - ❌ **To usuwało pytania pairing**

3. **Wyświetlanie:** `renderQuestions()` używa walidowanych pytań
   - Pytania pairing już usunięte
   - ❌ **To nic nie pokazywało**

**Po naprawie:**
- Pytania pairing przechodzą walidację
- Są dostępne w `renderQuestions()`
- Można je wyszukać

#### Korzyści
- ✅ Pytania pairing są poprawnie zapisywane i wyświetlane
- ✅ Można wyszukać pytania pairing przez filtr typów
- ✅ Walidacja pytań pairing (minimum 2 pary)
- ✅ Sprawdzanie czy każda para ma oba pola wypełnione

#### Statystyki zmian
- Linie zmodyfikowane: ~15
- Wersja: 1.25 → 1.26
- Typ zmiany: patch (krytyczna poprawka błędu walidacji)

---

## [1.25] - 2025-01-17

### 🐛 Poprawka: Błędny selektor w funkcji getPairsData()

#### Problem
Dodawanie pytań typu "Dopasowanie" (pairing) nie działało poprawnie - pytania nie trafiały do bazy i nie było ich można wyszukać.

#### Przyczyna
W funkcji `getPairsData()` był błędny selektor CSS:
```javascript
const leftInput = row.querySelector(".pair-input:first-child");
const rightInput = row.querySelector(".pair-input:nth-child(2)");  // ❌ BŁĘD
```

Selektor `:nth-child(2)` wybiera drugi element w danym wierszu, niezależnie od typu. W strukturze HTML:
```html
<div class="pair-row">
    <input type="text" class="pair-input">  <!-- 1 -->
    <input type="text" class="pair-input">  <!-- 2 -->
    <div class="pair-actions">  <!-- 3 -->
        <button></button>  <!-- 4 -->
    </div>
</div>
```

Selektor `:nth-child(2)` prawidłowo wybiera drugi element (drugie input), ale jest to nietypowy selektor. Należało użyć bardziej precyzyjnego selektora opartego na typie elementu.

#### Rozwiązanie

**Przed:**
```javascript
const leftInput = row.querySelector(".pair-input:first-child");
const rightInput = row.querySelector(".pair-input:nth-child(2)");
```

**Po:**
```javascript
const leftInput = row.querySelector(".pair-input:nth-of-type(1)");
const rightInput = row.querySelector(".pair-input:nth-of-type(2)");
```

**Alternatywna poprawka:**
```javascript
// Mogłoby być też tak:
const leftInput = row.querySelector(".pair-input:first-child");
const rightInput = row.querySelector(".pair-input:last-child");
```

#### Dlaczego to był problem?

**Wyjaśnienie działania selektorów:**
- `:nth-child(2)` - drugi element w wierszu (niezależnie od typu)
- `:nth-of-type(2)` - drugi element tego samego typu (input)

**Różnica na przykładzie:**
```html
<div>
    <p>Pierwszy</p>        <!-- nth-child(1), nth-of-type(1) -->
    <span>Drugi</span>    <!-- nth-child(2), nth-of-type(1) -->
    <p>Trzeci</p>        <!-- nth-child(3), nth-of-type(2) -->
</div>
```

W przypadku `pair-row`, oba selektory działają tak samo, ponieważ:
- `input type="text" class="pair-input"` - pierwszy
- `input type="text" class="pair-input"` - drugi

Ale `:nth-of-type` jest bardziej precyzyjny i zalecany.

#### Zmiany
- **Lokalizacja:** `index.html` (linia ~9785-9786)
- **Element:** Funkcja `getPairsData()`
- **Zmiana:** Selektory z `:first-child`/`:nth-child(2)` na `:nth-of-type(1)`/`:nth-of-type(2)`

#### Korzyści
- ✅ Poprawne pobieranie danych z formularza par
- ✅ Pytania typu "pairing" trafiają do bazy
- ✅ Pytania typu "pairing" można wyszukać
- ✅ Bardziej precyzyjne selektory CSS
- ✅ Lepsze zrozumienie kodu

#### Statystyki zmian
- Linie zmodyfikowane: 2
- Wersja: 1.24 → 1.25
- Typ zmiany: patch (poprawka błędu)

---

## [1.24] - 2025-01-17

### 🔍 Poprawka: Dodano filtr typu "pairing" w wyszukiwarce pytań

#### Problem
W wyszukiwarce pytań w sekcji "Pytania" brakowało filtra dla nowego typu pytania "pairing" (Dopasowanie), który został dodany w wersji 1.23.

#### Rozwiązanie
Dodano opcję "Dopasowanie (Pairing)" do selecta `#filterType` w wyszukiwarce pytań:

**Przed:**
```html
<select id="filterType" onchange="window.filterQuestions()">
    <option value="all">Wszystkie typy</option>
    <option value="single">Jedna poprawna</option>
    <option value="multiple">Wiele poprawnych</option>
    <option value="ordering">Ułóż w kolejności</option>
</select>
```

**Po:**
```html
<select id="filterType" onchange="window.filterQuestions()">
    <option value="all">Wszystkie typy</option>
    <option value="single">Jedna poprawna</option>
    <option value="multiple">Wiele poprawnych</option>
    <option value="ordering">Ułóż w kolejności</option>
    <option value="pairing">Dopasowanie (Pairing)</option>
</select>
```

#### Zmiany
- **Lokalizacja:** `index.html` (linia ~6576-6582)
- **Element:** `<select id="filterType">` w sekcji wyszukiwarki pytań
- **Zmiana:** Dodano `<option value="pairing">Dopasowanie (Pairing)</option>`

#### Korzyści
- ✅ Użytkownik może filtrować pytania według typu "pairing"
- ✅ Pełna zgodność filtrowania z nowym typem pytania
- ✅ Możliwość szybkiego znalezienia wszystkich pytań typu "dopasowanie"

#### Statystyki zmian
- Linie zmodyfikowane: 1
- Wersja: 1.23 → 1.24
- Typ zmiany: patch (poprawka braku w funkcjonalności)

---

## [1.22] - 2025-01-17

### 📐 Poprawki UI: Skompaktowanie interfejsu fiszek na mobile

#### Problem
Interfejs fiszek na urządzeniach mobilnych zajmował za dużo miejsca w pionie:
1. Sekcja "Fiszka X z Y" miała za duży padding i marginesy
2. Przyciski oceny "Umiem/Nie umiem" były ustawione w kolumnie (pod sobą) zamiast w wierszu (obok siebie)
3. Cały interfejs wymuszał przewijanie strony na mobile

#### Przyczyna
1. `.flashcard-progress` miał duży padding (12px) i margin-top (12px)
2. `.flashcard-rating` na mobile (max-width: 480px) miał `flex-direction: column` co powodowało ułożenie przycisków pod sobą
3. Przyciski miały duże padding (14px 20px) i font-size (16px)
4. Pasek postępu miał wysokość 8px, co również zajmowało miejsce

#### Rozwiązanie

**1. Zmniejszenie sekcji "Fiszka X z Y" (flashcard-progress)**

**Desktop (base styles):**
- Padding: `12px` → `8px 10px`
- Border-radius: `12px` → `8px`
- Margin-top: `12px` → `8px`
- Pasek postępu height: `8px` → `6px`
- Pasek postępu margin: `8px 0` → `6px 0`
- Pasek postępu border-radius: `6px` → `4px`

**Mobile (max-width: 768px):**
- Maksymalna wysokość treści: `calc(100vh - 320px)` → `calc(100vh - 280px)`

**Small mobile (max-width: 480px):**
- Maksymalna wysokość treści: `calc(100vh - 340px)` → `calc(100vh - 300px)`

**2. Naprawa układu przycisków oceny (flashcard-rating)**

**Usunięto:** `flex-direction: column` na small mobile
```css
/* Przed */
.flashcard-rating {
    flex-direction: column;  /* To powodowało ułożenie pod sobą */
    gap: 10px;
}

/* Po */
.flashcard-rating {
    /* Brak flex-direction - dziedziczy flex (obok siebie) */
    gap: 8px;
    margin-top: 16px;
}
```

**3. Zmniejszenie przycisków oceny**

**Desktop (base styles):**
- Padding: `14px 20px` → `10px 16px`
- Border-radius: `12px` → `10px`
- Font-size: `16px` → `14px`
- Margin-top: `24px` → `16px`

**Mobile (max-width: 768px):**
- Font-size: `var(--text-sm)` → `12px`
- Padding: `12px 16px` → `8px 12px`

**Small mobile (max-width: 480px):**
- Padding: `10px 12px` → `8px 10px`
- Font-size: `var(--text-xs)` → `11px`

**4. Dalsze skompaktowanie licznika "Umiem/Nie umiem"**

**Mobile (max-width: 768px):**
- Margin-top: `8px` → `6px`
- Gap: `6px` → `4px`
- Padding stat: `4px 6px` → `3px 6px`
- Flex stat: `1 1 calc(50% - 3px)` → `1 1 calc(50% - 2px)`
- Font-size value: `13px` → `12px`
- Font-size label: `9px` → `8px`
- Margin-top label: `0px` → `0px`

**Small mobile (max-width: 480px):**
- Margin-top: `6px` → `4px`
- Gap: `4px` → `3px`
- Padding stat: `4px 6px` → `3px 5px`
- Font-size value: `var(--text-base)` → `11px`
- Font-size label: `9px` → `8px`
- Margin button "wróć": `6px` → `5px`
- Padding flashcard-face: `20px` → `18px`

#### Zmiany w CSS

**Before (base styles):**
```css
.flashcard-progress {
    background: var(--card-bg);
    padding: 12px;
    border-radius: 12px;
    margin-top: 12px;
}

.flashcard-progress-bar {
    height: 8px;
    margin: 8px 0;
    border-radius: 6px;
}

.flashcard-rating {
    margin-top: 24px;
}

.flashcard-rating button {
    padding: 14px 20px;
    font-size: 16px;
    border-radius: 12px;
}
```

**After (base styles):**
```css
.flashcard-progress {
    background: var(--card-bg);
    padding: 8px 10px;
    border-radius: 8px;
    margin-top: 8px;
}

.flashcard-progress-bar {
    height: 6px;
    margin: 6px 0;
    border-radius: 4px;
}

.flashcard-rating {
    margin-top: 16px;
}

.flashcard-rating button {
    padding: 10px 16px;
    font-size: 14px;
    border-radius: 10px;
}
```

**Before (small mobile - key issue):**
```css
@media (max-width: 480px) {
    .flashcard-rating {
        flex-direction: column;  /* PROBLEM */
        gap: 10px;
    }
}
```

**After (small mobile):**
```css
@media (max-width: 480px) {
    .flashcard-rating {
        /* Brak flex-direction - dziedziczy flex z base */
        gap: 8px;
        margin-top: 16px;
    }
}
```

#### Korzyści
- ✅ Sekcja "Fiszka X z Y" zajmuje o ~40% mniej miejsca w wysokości
- ✅ Przyciski "Umiem/Nie umiem" są obok siebie (nie pod sobą) na mobile
- ✅ Mniejsza konieczność przewijania strony na mobile
- ✅ Bardziej kompaktowy interfejs
- ✅ Lepsze wykorzystanie dostępnej przestrzeni ekranu
- ✅ Zachowany responsywny układ (desktop vs mobile)
- ✅ Zwiększona czytelność dzięki optymalnemu spacingowi

#### Statystyki zmian
- Linie zmienione: ~40
- Wersja: 1.21 → 1.22
- Typ zmiany: patch (poprawki UI/UX)

---

## [1.21] - 2025-01-17

### 🎯 Dodanie filtra "Tryb" do trybu nauki i testu

#### Problem
W trybie nauki (practice) oraz trybie testu brakowało filtra "Tryb", który był dostępny w trybie fiszek. Użytkownicy nie mogli wybierać pytań oznaczonych do powtórki, pytań oznaczonych jako "Nie umiem" ani pytań do powtórki w systemie Spaced Repetition.

#### Przyczyna
Filtr "Tryb" został zaimplementowany tylko dla trybu fiszek, ale nie dla trybu nauki i testu, mimo że te funkcje korzystają z tego samego mechanizmu filtrowania pytań.

#### Rozwiązanie

**1. Dodanie select "Tryb" do HTML**

Dodano select do ustawień trybu testu (po kategorii):
```html
<div class="form-group">
    <label>Tryb:</label>
    <select id="testMode">
        <option value="all">Wszystkie pytania (oznaczone na początku)</option>
        <option value="markedForReview">⭐ Tylko oznaczone do powtórki</option>
        <option value="difficult">🔄 Tylko powtórki (pytania oznaczone "Nie umiem")</option>
        <option value="srs">🧠 Spaced Repetition (due for review)</option>
    </select>
</div>
```

Dodano select do ustawień trybu nauki (po kategorii):
```html
<div class="form-group">
    <label>Tryb:</label>
    <select id="practiceMode">
        <option value="all">Wszystkie pytania (oznaczone na początku)</option>
        <option value="markedForReview">⭐ Tylko oznaczone do powtórki</option>
        <option value="difficult">🔄 Tylko powtórki (pytania oznaczone "Nie umiem")</option>
        <option value="srs">🧠 Spaced Repetition (due for review)</option>
    </select>
</div>
```

**2. Zmodyfikowanie funkcji startTest()**

Przed filtrowaniem po kategorii dodano filtrowanie po trybie:
```javascript
const mode = document.getElementById("testMode").value;

if (mode === "markedForReview") {
    filteredQuestions = filteredQuestions.filter(q => q.markedForReview);
    // sprawdzenie czy są pytania...
} else if (mode === "difficult") {
    filteredQuestions = filteredQuestions.filter(q => {
        const srsData = getQuestionSRSData(q.id);
        return srsData && srsData.rating === 0;
    });
    // sprawdzenie czy są pytania...
} else if (mode === "srs") {
    filteredQuestions = filteredQuestions.filter(q => isQuestionDueForReview(q.id));
    // sortowanie po nextReviewDate
    // sprawdzenie czy są pytania...
} else {
    // Tryb "all" - priorytetowe pokazywanie oznaczonych do powtórki
    const markedQuestions = filteredQuestions.filter(q => q.markedForReview);
    const otherQuestions = filteredQuestions.filter(q => !q.markedForReview);
    filteredQuestions = [...markedQuestions, ...otherQuestions];
}
```

**3. Zmodyfikowanie funkcji startPractice()**

Zastąpiono sekcję "Priorytetowe pokazywanie oznaczonych do powtórki" logiką obsługującą wszystkie tryby (all, markedForReview, difficult, srs):
```javascript
const mode = document.getElementById("practiceMode").value;

if (mode === "markedForReview") {
    filteredQuestions = filteredQuestions.filter(q => q.markedForReview);
    // sprawdzenie czy są pytania...
} else if (mode === "difficult") {
    filteredQuestions = filteredQuestions.filter(q => {
        const srsData = getQuestionSRSData(q.id);
        return srsData && srsData.rating === 0;
    });
    // sprawdzenie czy są pytania...
} else if (mode === "srs") {
    filteredQuestions = filteredQuestions.filter(q => isQuestionDueForReview(q.id));
    // sortowanie po nextReviewDate
    // sprawdzenie czy są pytania...
}

const count = document.getElementById("practiceQuestionCount").value;

if (mode === "all") {
    // Priorytetowe pokazywanie oznaczonych do powtórki
    // ... logika jak wcześniej ...
} else {
    // Inne tryby - ogranicz liczbę
    if (count !== "all") {
        practiceQuestions = filteredQuestions.slice(0, limit);
    } else {
        practiceQuestions = filteredQuestions;
    }
}
```

**4. Dodanie eksportów funkcji SRS**

Dodano funkcje do exports window, aby mogły być używane w startTest i startPractice:
```javascript
window.getQuestionSRSData = getQuestionSRSData;
window.isQuestionDueForReview = isQuestionDueForReview;
```

#### Opisy trybów

- **Wszystkie pytania (oznaczone na początku)** - Pokazuje wszystkie pytania, z oznaczonymi do powtórki na początku listy
- **⭐ Tylko oznaczone do powtórki** - Pokazuje tylko pytania oznaczone gwiazdką (markedForReview)
- **🔄 Tylko powtórki (pytania oznaczone "Nie umiem")** - Pokazuje tylko pytania z SRS rating = 0 (ponownie)
- **🧠 Spaced Repetition (due for review)** - Pokazuje tylko pytania, które są do powtórki według algorytmu SRS

#### Komunikaty użytkownika

Dla każdego trybu (jeśli brak pytań):
- `markedForReview`: "Nie masz jeszcze pytań oznaczonych do powtórki! Oznacz trudne pytania w szczegółach wyników."
- `difficult`: "Brak pytań oznaczonych jako 'Nie umiem'! Rozwiązuj testy i oznaczaj trudne pytania."
- `srs`: "Brak pytań do powtórki! Wszystkie są odłożone na przyszłość. 🎉"

#### Korzyści
- ✅ Ujednolicony interfejs wszystkich trybów (test, nauka, fiszki)
- ✅ Możliwość skupienia się na trudnych pytaniach w trybie nauki i testie
- ✅ Pełna integracja z systemem Spaced Repetition
- ✅ Ochrona przed uruchomieniem pustego quizu (komunikaty toast)
- ✅ Logiczne sortowanie pytań SRS po dacie powtórki
- ✅ Priorytetowe pokazywanie oznaczonych pytań w trybie "all"

#### Statystyki zmian
- Linie dodane: ~90
- Linie zmodyfikowane: ~50
- Wersja: 1.20 → 1.21
- Typ zmiany: minor (nowa funkcjonalność - filtr trybu)

---

## [1.20] - 2025-01-17

### 📐 Poprawki UI: Kompaktowy licznik i lepsza kontrastowość numeracji w fiszkach

#### Problem 1
Element flashcard-stats (licznik "Fiszka X z Y" i przyciski "Umiałem/Nie umiałem") zajmował zbyt dużo miejsca, szczególnie na urządzeniach mobilnych, co wymuszało przewijanie strony.

#### Problem 2
W motywie neon (modern) kolor numeracji odpowiedzi (#00ffff - cyan) zlewał się z tłem, przez co był słabo widoczny.

#### Rozwiązanie 1: Zmniejszenie elementu flashcard-stats o połowę

**Desktop:**
- `.flashcard-stat` padding: `8px 12px` → `4px 8px`
- `.flashcard-stat-value` font-size: `18px` → `14px`
- `.flashcard-stat-label` font-size: `11px` → `9px`
- `.flashcard-stat` border-radius: `8px` → `6px`
- `.flashcard-stat-label` margin-top: `2px` → `1px`

**Mobile:**
- `.flashcard-stat` padding: `6px 8px` → `4px 6px`
- `.flashcard-stat-value` font-size: `var(--text-lg)` → `13px`
- `.flashcard-stat-label` font-size: `10px` → `9px`
- `.flashcard-stat-label` margin-top: `1px` → `0px`

#### Rozwiązanie 2: Poprawa kontrastowości numeracji w motywie neon

Zmieniono kolor numeracji w motywie modern/neon:
- **Przed**: numeracja `#00ffff` (cyan) bez tła
- **Po**: numeracja z białym tłem `#ffffff` i fioletowym tekstem `#7c3aed`

**Zmienione elementy:**
```css
/* Przed */
[data-theme="modern"] .flashcard-option .option-number {
    color: #00ffff !important;
}

/* Po */
[data-theme="modern"] .flashcard-option .option-number {
    background: #ffffff !important;
    color: #7c3aed !important;
}
```

**Poprawne odpowiedzi:**
```css
/* Przed */
[data-theme="modern"] .flashcard-option.correct .option-number {
    color: #10b981 !important;
}

/* Po */
[data-theme="modern"] .flashcard-option.correct .option-number {
    background: #ffffff !important;
    color: #10b981 !important;
}
```

**Opcje na przód karty:**
```css
/* Przed */
[data-theme="modern"] .flashcard-options-front .flashcard-option .option-number {
    color: #00ffff !important;
}

/* Po */
[data-theme="modern"] .flashcard-options-front .flashcard-option .option-number {
    background: #ffffff !important;
    color: #7c3aed !important;
}
```

#### Korzyści
- ✅ Licznik fiszek i przyciski oceny zajmują o połowę mniej miejsca
- ✅ Brak konieczności przewijania strony na mobile przy widocznym interfejsie
- ✅ Numeracja odpowiedzi w motywie neon jest wyraźnie widoczna dzięki białemu tłu
- ✅ Lepsza kontrastowość poprawia czytelność i dostępność
- ✅ Bardziej kompaktowy interfejs na małych ekranach

#### Statystyki zmian
- Linie zmienione: ~30
- Wersja: 1.19 → 1.20
- Typ zmiany: patch (poprawki UI/UX)

---

## [1.19] - 2025-01-17

### 🎴 Poprawka: Wyświetlanie odpowiedzi na przód karty fiszki

#### Problem
W trybie fiszek przód karty pokazywał samo pytanie bez odpowiedzi, co uniemożliwiało użytkownikowi zapoznanie się z opcjami przed obrotem karty i sprawdzeniem poprawnej odpowiedzi.

#### Przyczyna
Domyślny tryb pracy fiszek zakładał, że przód karty zawiera pytanie, a tył karty - odpowiedź. Jednak w przypadku pytań wielokrotnego wyboru (single/multiple) lepiej jest pokazać wszystkie opcje od razu.

#### Rozwiązanie

**1. Dodanie opcji odpowiedzi na przód karty**
- Dodano nowy kontener `.flashcard-options-front` w HTML (przed przyciskiem "odkryj odpowiedź")
- Przód karty teraz pokazuje:
  - Treść pytania
  - Wszystkie opcje odpowiedzi (bez oznaczenia poprawnej)
  - Hint: "Kliknij aby odkryć odpowiedź"

**2. Zachowanie poprawnej odpowiedzi na tyle karty**
- Tył karty dalej pokazuje:
  - Treść pytania
  - Wszystkie opcje odpowiedzi **z oznaczeniem poprawnej** (kolor zielony)
  - Przyciski oceny (Umiem to / Nie umiem lub SRS buttons)

**3. Aktualizacja JavaScript**
Zmodyfikowano funkcję `showFlashcard()`:
- Przód karty: renderuje opcje bez oznaczenia poprawnej (wszystkie mają klasę `.flashcard-option`)
- Tył karty: renderuje opcje z oznaczeniem poprawnej (poprawne mają klasę `.flashcard-option.correct`)

**Przed:**
```javascript
// Przód karty - tylko pytanie
document.getElementById("flashcard-question").innerHTML = sanitizeHTML(q.text);

// Tył karty - pytanie + opcje z oznaczeniem poprawnej
const optionsHTML = (q.options || []).map((opt, i) => {
    const isCorrect = (q.correct || []).includes(i + 1);
    const optionClass = isCorrect ? "flashcard-option correct" : "flashcard-option";
    return `<div class="${optionClass}">...</div>`;
}).join("");
document.getElementById("flashcard-options").innerHTML = optionsHTML;
```

**Po:**
```javascript
// Przód karty - pytanie + opcje (bez oznaczenia)
const optionsFrontHTML = (q.options || []).map((opt, i) => {
    return `<div class="flashcard-option">...</div>`;
}).join("");
document.getElementById("flashcard-options-front").innerHTML = optionsFrontHTML;

// Tył karty - pytanie + opcje (z oznaczeniem)
const optionsBackHTML = (q.options || []).map((opt, i) => {
    const isCorrect = (q.correct || []).includes(i + 1);
    const optionClass = isCorrect ? "flashcard-option correct" : "flashcard-option";
    return `<div class="${optionClass}">...</div>`;
}).join("");
document.getElementById("flashcard-options").innerHTML = optionsBackHTML;
```

#### Zmiany w HTML

**Dodany element:**
```html
<div class="flashcard-face flashcard-front">
    <div class="flashcard-content">
        <div id="flashcard-question" class="question-text"></div>
        <div class="flashcard-options-front" id="flashcard-options-front"></div>
        <div class="flashcard-hint">👆 Kliknij aby odkryć odpowiedź</div>
    </div>
</div>
```

#### Zmiany w CSS

**Nowe style dla przodu karty (kompaktowe):**
```css
.flashcard-options-front {
    margin: 20px 0;
}

.flashcard-options-front .flashcard-option {
    background: rgba(255, 255, 255, 0.15);
    backdrop-filter: blur(10px);
    padding: 10px 14px;
    margin: 6px 0;
    border-radius: 8px;
    border: 1px solid rgba(255, 255, 255, 0.2);
}

.flashcard-options-front .flashcard-option .option-number {
    width: 28px;
    height: 28px;
    background: rgba(255, 255, 255, 0.8);
    font-size: 14px;
}
```

**Dark mode:**
```css
body.dark-mode .flashcard-options-front .flashcard-option {
    background: #0d1117;
    border: 1px solid #30363d;
}
```

**Modern theme:**
```css
[data-theme="modern"] .flashcard-options-front .flashcard-option {
    background: rgba(45, 40, 90, 0.6) !important;
    border: 1px solid rgba(255, 0, 255, 0.3) !important;
}
```

**Mobile (responsive):**
```css
.flashcard-options-front .flashcard-option {
    padding: 10px 12px;
    margin: 6px 0;
}

.flashcard-options-front .flashcard-option .option-number {
    width: 24px;
    height: 24px;
    font-size: var(--text-xs);
}
```

#### Korzyści
- ✅ Użytkownik widzi wszystkie opcje odpowiedzi przed obrotem karty
- ✅ Może przemyśleć odpowiedź przed sprawdzeniem poprawnej
- ✅ Poprawa UX - bardziej naturalny flow nauki
- ✅ Kompaktowy rozmiar opcji na przód karty (nie zajmują za dużo miejsca)
- ✅ Zgodne style dla wszystkich motywów (classic, dark, modern)
- ✅ Responsywne na wszystkich urządzeniach (desktop, tablet, mobile)

#### Statystyki zmian
- Linie dodane: ~80
- Wersja: 1.18 → 1.19
- Typ zmiany: patch (poprawka UX)

---

## [1.18] - 2025-01-17

### 🐛 Poprawka: Błąd przy oznaczaniu pytań do powtórki

#### Problem
Podczas sesji nauki (tryb practice), gdy użytkownik chciał oznaczyć pytanie do powtórki, pojawiał się błąd uniemożliwiający działanie tej funkcji.

#### Przyczyna
1. **Niedopasowanie typów ID** - ID pytań są liczbami (np. `1737136628000.54321`), ale przycisk w HTML przekazywał je jako stringi (np. `'1737136628000.54321'`)
   - Kod używał ścisłego porównania `===` co zwracało `false` dla różnych typów
   - Funkcja `toggleBookmarkInPractice` nie mogła znaleźć pytania w bazie

2. **Brak obsługi błędów** - funkcje oznaczania nie miały bloków try-catch

3. **Błędne odświeżanie widoku** - funkcja `renderPracticeQuestion()` była wywoływana po każdym oznaczeniu, co resetowało stan odpowiedzi

#### Rozwiązanie

**1. Poprawka typu przy porównywaniu ID (=== na ==)**
Zmieniono porównania w funkcjach:
- `toggleBookmarkInPractice()` - index.html:11499
- `toggleBookmarkInQuestions()` - index.html:11564
- `isQuestionMarkedForReview()` - index.html:12419
- `openNotesModal()` - index.html:11687
- `toggleMarkForReview()` - index.html:12441

**Przed:**
```javascript
const qIdx = questions.findIndex(q => q.id === questionId);
```

**Po:**
```javascript
const qIdx = questions.findIndex(q => q.id == questionId);
```

**2. Dodanie obsługi błędów**
Dodano bloki try-catch w funkcjach:
- `toggleBookmarkInPractice()` - szczegółowa walidacja stanu
- `toggleMarkForReview()` - walidacja `currentUser`, `testResults`, `currentDetailedResultIndex`
- `toggleBookmarkInQuestions()` - walidacja tablicy `questions`
- `isQuestionMarkedForReview()` - walidacja tablicy `questions`
- `openNotesModal()` - poprawka porównania ID
- `getMarkedQuestionsCount()` - walidacja tablicy `questions`

**3. Optymalizacja odświeżania widoku**
Zamiast wywoływać `renderPracticeQuestion()` (co resetuje stan), funkcja `toggleBookmarkInPractice` teraz:
- Znajduje przycisk bookmarku w DOM
- Aktualizuje tylko tekst i styl przycisku
- **Nie resetuje** stanu odpowiedzi (`practiceAnswered`)
- **Nie resetuje** wybranych odpowiedzi (`practiceSelectedAnswers`)

#### Korzyści
- ✅ Oznaczanie pytań do powtórki działa teraz poprawnie w trybie nauki
- ✅ Stan odpowiedzi jest zachowany po oznaczeniu pytania
- ✅ Jasne komunikaty o błędach dla użytkownika
- ✅ Lepsze bezpieczeństwo dzięki walidacji i obsłudze błędów
- ✅ Poprawa UX - brak utraty postępu podczas sesji nauki

#### Statystyki zmian
- Linie zmienione: ~120
- Wersja: 1.17 → 1.18
- Typ zmiany: patch (poprawki błędów)

---

## [1.17] - 2025-01-16

### 🏷️ Dodanie tagów do pytań AI-900 (automatyczne)

#### Opis
Automatyczne uzupełnienie tagów w pliku `pytania_2026-01-16.json` na podstawie analizy treści pytań i odpowiedzi.

#### Problem
Plik `pytania_2026-01-16.json` zawierał 294 pytań z kategorią AI-900, ale wszystkie miały puste pole `"tags": []`, co utrudniało wyszukiwanie i organizację pytań w aplikacji.

#### Rozwiązanie
Stworzono i uruchomiono skrypt Python, który automatycznie przeanalizował treść pytań i dodał odpowiednie tagi na podstawie słów kluczowych dla poszczególnych kategorii tematycznych.

#### Kategorie tagów używane
- **Computer Vision**: Object Detection, Face Detection, Image Classification, OCR, Tagging, Semantic Segmentation, Bounding Box
- **Natural Language Processing**: Text Analytics, Translation, Sentiment Analysis, Key Phrase Extraction, Named Entity Recognition, Entity Recognition, Language Detection, Language Understanding (LUIS)
- **Speech Services**: Speech Recognition, Speech Synthesis, Speech Translation, Text-to-Speech, Speech-to-Text, Voice Recognition, Speaker Recognition, Language Identification
- **Machine Learning**: Classification, Regression, Clustering, Supervised/Unsupervised, Training, Validation, Evaluation, Feature Engineering, Data Ingestion, Data Transformation
- **Generative AI**: GPT Models, DALL-E, Image Generation, System Messages, Copilots, Plugins, Safety System, Content Filters, Prompts
- **Responsible AI**: Fairness, Inclusiveness, Transparency, Privacy, Security, Accountability, Reliability, Safety, NIST Framework, Ethics, Bias
- **Azure AI Services**: Azure AI Services, Azure AI Language, Azure AI Vision, Azure AI Speech, Azure AI Translator, Custom Vision, Form Recognizer, Azure AI Bot Service, Document Intelligence, Azure Machine Learning, Azure ML Designer, Azure ML Studio, Custom Vision Training, Object Detection Training
- **Azure ML Components**: Workspace, Compute, Container, Kubernetes, Pipeline, Module, Dataset, Job, Endpoint
- **Computer Vision Workloads**: Object Detection, Face Detection, Image Classification, Tagging, Semantic Segmentation, Scene Segmentation, Image Analysis, Optical Character Recognition, Face Recognition, Custom Vision
- **NLP Workloads**: Sentiment, Translation, Text Analysis, Key Phrase Extraction, Entity Extraction, Transcription, Language Detection, Language Understanding (LUIS), Entity Linking
- **Conversational AI**: Chatbot, Web Chatbot, FAQ, Knowledge Base, Question Answering, Smart Device, Assistant, Voice Assistant
- **Azure ML Designer Components**: Dataset, Compute, Pipeline, Module
- **Azure ML Metrics**: Accuracy, Confidence, Root Mean Square Error, Precision, Recall, F1, RMSE, R2, Coefficient of Determination

#### Zmiany w pliku
- Wszystkie 294 pytania w pliku `pytania_2026-01-16.json` otrzymały tagi
- Tagi są w formacie tablicy stringów: `["tag1", "tag2", "tag3"]`
- Format jest zgodny z systemem importu aplikacji
- Tagi zostały dodane automatycznie na podstawie słów kluczowych w tekście pytań i odpowiedziach

#### Statystyki zmian
- Liczba pytań przetworzonych: 294
- Liczba tagów dodanych: ~1200 (średnio ~4 tagi na pytanie)
- Format: JSON (tablica stringów)
- Plik wyjściowy: `pytania_2026-01-16.json`

#### Wersja aplikacji
- Wersja: 1.16 → 1.17
- Typ zmiany: minor (nowa funkcjonalność - automatyczne tagowanie pytań)

#### Uwagi
- Tagi są w języku angielskim (ponieważ pytania są po angielsku)
- Tagi odzwierciają się do treści pytań i prawidłowych odpowiedzi
- System jest w pełni automatyczny - nie wymaga ręcznego edytowania każdego pytania
- Możliwość dalszego ulepszania słów kluczowych

---

## [1.16] - 2025-01-16

### 🎴 Poprawka: Kompaktowy licznik "umiałem/nie umiałem" w trybie fiszek na mobile

#### Problem
W trybie mobilnym licznik "umiałem/nie umiałem" zajmował zdecydowanie za dużo miejsca. Istniała też spora przestrzeń między przyciskiem "wróć do ustawień" a ramką z licznikami.

#### Przyczyna
1. Liczniki miały za duży padding, font-size i marginesy na mobile
2. Przycisk "wróć do ustawień" miał `margin-bottom: 20px`, co tworzyło zbyt dużo przestrzeni

#### Rozwiązanie
Zmniejszono wielkość elementów i ułożono liczniki w jednej linii:

**Mobile (max-width: 768px):**
- `.flashcard-stats`: `margin-top: 8px`, `gap: 6px` (z 8px)
- `.flashcard-stat`: `padding: 6px 8px`, `flex: 1 1 calc(50% - 3px)`
- `.flashcard-stat-value`: `font-size: var(--text-lg)` (z var(--text-2xl))
- `.flashcard-stat-label`: `font-size: 10px`, `margin-top: 1px`
- Przycisk "wróć do ustawień": `margin-bottom: 10px` (z 20px)

**Small mobile (max-width: 480px):**
- `.flashcard-stats`: `margin-top: 6px`, `gap: 4px`
- `.flashcard-stat`: `padding: 4px 6px`
- `.flashcard-stat-value`: `font-size: var(--text-base)` (jeszcze mniejsze)
- `.flashcard-stat-label`: `font-size: 9px`, `margin-top: 0`
- Przycisk "wróć do ustawień": `margin-bottom: 6px` (jeszcze mniejsze)

Korzyści:
- ✅ Liczniki są w jednej linii i zajmują o połowę mniej miejsca
- ✅ Mniejsza przestrzeń między przyciskiem "wróć do ustawień" a licznikami
- ✅ Bardziej kompaktowy interfejs na małych ekranach
- ✅ Więcej miejsca na treść fiszki

#### Zmiany w CSS

**Nowe reguły CSS (mobile):**
```css
/* Flashcard stats - compact layout */
.flashcard-stats {
    margin-top: 8px;
    gap: 6px;
}

.flashcard-stat {
    padding: 6px 8px;
    flex: 1 1 calc(50% - 3px);
}

.flashcard-stat-value {
    font-size: var(--text-lg);
}

.flashcard-stat-label {
    font-size: 10px;
    margin-top: 1px;
}

/* Reduce margin on "wróć do ustawień" button */
#flashcard-active button[onclick="window.exitFlashcards()"] {
    margin-bottom: 10px !important;
}
```

**Nowe reguły CSS (small mobile):**
```css
.flashcard-stats {
    margin-top: 6px;
    gap: 4px;
}

.flashcard-stat {
    padding: 4px 6px;
}

.flashcard-stat-value {
    font-size: var(--text-base);
}

.flashcard-stat-label {
    font-size: 9px;
    margin-top: 0;
}

#flashcard-active button[onclick="window.exitFlashcards()"] {
    margin-bottom: 6px !important;
}
```

#### Statystyki zmian
- Linie zmienione: ~30
- Wersja: 1.15 → 1.16

---

## [1.15] - 2025-01-15

### 🔤 Poprawka: Skrócenie tytułu aplikacji na mobile

#### Problem
Na telefonach przycisk przełącznika trybu ciemnego nachodził na górny tytuł aplikacji "Aplikacja Quizowo-Testowa", przez co numer wersji nie był widoczny.

#### Przyczyna
Tytuł był zbyt długi (24 znaki) i nie mieścił się w jednym wierszu na małych ekranach przy zachowaniu przycisku przełącznika motywu.

#### Rozwiązanie
Skrócono tytuł aplikacji z "Aplikacja Quizowo-Testowa" na "QuizApp":
- **Przed:** "📝 Aplikacja Quizowo-Testowa" (24 znaki)
- **Po:** "📝 QuizApp" (8 znaków)

Korzyści:
- ✅ Tytuł mieści się w jednym wierszu na małych ekranach
- ✅ Numer wersji jest teraz widoczny
- ✅ Brak nakładania się przycisku przełącznika motywu na tytuł
- ✅ Lepsza czytelność na urządzeniach mobilnych

#### Zmiany w HTML

**Zmieniony element:**
```html
<!-- Przed -->
<h1>📝 Aplikacja Quizowo-Testowa <small style="font-size: 0.5em; color: #666;">v1.14</small></h1>

<!-- Po -->
<h1>📝 QuizApp <small style="font-size: 0.5em; color: #666;">v1.15</small></h1>
```

#### Statystyki zmian
- Linie zmienione: 1
- Wersja: 1.14 → 1.15

---

## [1.14] - 2025-01-15

### 🎨 Poprawka: Zmiana koloru karty odpowiedzi w trybie fiszek

#### Problem
Karta odpowiedzi (tył karty w trybie fiszek) miała kolor zielony, który był bardzo podobny do koloru prawidłowej odpowiedzi, co powodowało słabą czytelność i mylenie się z kolorami statusu odpowiedzi.

#### Przyczyna
`.flashcard-back` używał koloru `var(--success-color)` (złty/zielony), który był identyczny lub bardzo podobny do koloru używanego do oznaczania prawidłowych odpowiedzi w innych częściach aplikacji.

#### Rozwiązanie
Zmieniono gradient karty odpowiedzi z zielonego na fioletowy:
- **Przed:** `linear-gradient(135deg, var(--success-color) 0%, #059669 100%)` (zielony)
- **Po:** `linear-gradient(135deg, #7c3aed 0%, #6d28d9 100%)` (fioletowy)

Korzyści:
- ✅ Lepszy kontrast z białym tekstem
- ✅ Wyraźne odróżnienie od karty pytania (niebieskiej)
- ✅ Wyraźne odróżnienie od kolorów odpowiedzi (zielony/czerwony)
- ✅ Lepsza czytelność i estetyka

#### Zmiany w CSS

**Zmieniona reguła CSS:**
```css
/* Przed */
.flashcard-back {
    background: linear-gradient(135deg, var(--success-color) 0%, #059669 100%);
    transform: rotateY(180deg);
    -webkit-transform: rotateY(180deg);
}

/* Po */
.flashcard-back {
    background: linear-gradient(135deg, #7c3aed 0%, #6d28d9 100%);
    transform: rotateY(180deg);
    -webkit-transform: rotateY(180deg);
}
```

#### Statystyki zmian
- Linie zmienione: 1
- Wersja: 1.13 → 1.14

---

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
