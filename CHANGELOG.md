# Change Log - Aplikacja Quizowo-Testowa

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
