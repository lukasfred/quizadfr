# Change Log - Aplikacja Quizowo-Testowa

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
