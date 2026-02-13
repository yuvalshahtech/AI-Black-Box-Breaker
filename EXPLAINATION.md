# AI Black Box Breaker – Technical Architecture & Working Explanation

This document provides a comprehensive technical explanation of the application's architecture, execution flow, and debugging methodology.

---

## Table of Contents

1. [Application Overview](#application-overview)
2. [Architecture Design](#architecture-design)
3. [Recommendation Engine Debugger](#recommendation-engine-debugger)
4. [NLP Review Intelligence Debugger](#nlp-review-intelligence-debugger)
5. [UI Architecture](#ui-architecture)
6. [Design Philosophy](#design-philosophy)

---

## Application Overview

**AI Black Box Breaker** is an interactive algorithm debugger that visualizes the internal workings of two e-commerce AI systems:

1. **Recommendation Engine Debugger** – Item-to-item collaborative filtering (6 steps)
2. **NLP Review Intelligence Debugger** – Natural language sentiment analysis pipeline (8 steps)

### Core Concept

The application implements a **manual step-by-step execution model** where:
- Step 1 auto-executes upon user interaction
- Steps 2–N require manual progression via "Next Step" button
- Each step displays intermediate algorithm state in real-time
- Live data inspector shows internal variables and transformations

---

## Architecture Design

### Event-Driven Initialization

Both debuggers follow a consistent event-driven architecture:

```
User Interaction → initializeDebugger() → Execute Step 1 → Enable "Next Step" → Manual Progression
```

#### Core Methods

Each debugger class implements:

**`initializeDebugger(input)`**
- Resets internal state
- Sets `currentStep = 1`
- Stores user input (product or review text)
- Immediately executes Step 1
- Enables "Next Step" button
- Updates progress indicator
- Renders live data inspector

**`nextStep()`**
- Increments `currentStep`
- Calls `showStep(currentStep - 1)` to render execution state
- Updates progress indicator
- Disables "Next Step" button when reaching final step

**`reset()`**
- Clears all internal state variables
- Resets `currentStep = 0`
- Clears UI containers (steps and inspector)
- Resets user input (deselects products or clears textarea)
- Disables "Next Step" button
- Re-renders empty state in inspector

**`showStep(stepIndex)`**
- Uses switch-case to route to specific step renderer
- Calls `renderStepN()` methods
- Updates step visualization container
- Triggers inspector update

**`renderInspector()` / `renderResults()`**
- Displays current algorithm state
- Shows intermediate data structures
- Updates dynamically after each step

### Tab Switching Behavior

The application uses a tab container system:

```javascript
// Tab click handler
button.addEventListener('click', () => {
    // Deactivate all tabs
    document.querySelectorAll('.tab-container').forEach(container => {
        container.classList.remove('active');
    });
    
    // Activate selected tab
    document.getElementById(tabName).classList.add('active');
    
    // Auto-initialize NLP if text exists
    if (tabName === 'review') {
        const text = reviewInput.value.trim();
        if (text.length > 0 && !nlpDebugger.isActive) {
            nlpDebugger.initializeDebugger(text);
        }
    }
});
```

**Tab Indicator Animation:**
- Animated underline indicator tracks active tab
- Uses `cubic-bezier(0.4, 0, 0.2, 1)` transition
- Updates position based on button `offsetLeft` and `offsetWidth`
- Responds to window resize events

---

## Recommendation Engine Debugger

### Initialization Trigger

Product selection immediately triggers Step 1:

```javascript
card.addEventListener('click', () => {
    document.querySelectorAll('.product-card').forEach(c => c.classList.remove('selected'));
    card.classList.add('selected');
    
    // Auto-initialize debugger and execute Step 1
    recDebugger.initializeDebugger(product);
});
```

### Data Structures

**Co-Purchase Dataset:**
```javascript
const coPurchaseData = {
    'Laptop': {
        'Mouse': 45,        // 45 users bought Laptop + Mouse
        'Keyboard': 37,
        'USB Cable': 28,
        'Monitor': 22,
        'Phone Case': 5
    },
    // ... other products
};
```

**Total Purchases (Denominator):**
```javascript
const totalPurchasesPerProduct = {
    'Laptop': 165,   // Total users who purchased Laptop
    'Phone': 274,
    'Shoes': 95,
    'Headphones': 131,
    'Watch': 145
};
```

### 6-Step Execution Flow

#### **Step 1: Capture User Interaction**
- Records product selection event
- Displays selected product name
- Creates initial debug context

**Inspector State:**
```javascript
{
    product: "Laptop",
    coItems: {...},
    totalPurchases: 165,
    similarities: {},
    ranked: {}
}
```

#### **Step 2: Retrieve Co-Purchase Dataset**
- Fetches co-purchase relationships for selected product
- Displays table of co-purchased items with counts
- Example: `Laptop → Mouse: 45 users`

**Visualization:**
- Table showing all co-purchased items
- Count values for each relationship

#### **Step 3: Construct Similarity Formula & Calculate Scores**
- Applies similarity calculation for each co-purchased item
- **Formula:** `similarity = coCount / totalPurchases`

**Example Calculation:**
```
Mouse similarity = 45 / 165 = 0.2727
Keyboard similarity = 37 / 165 = 0.2242
USB Cable similarity = 28 / 165 = 0.1697
```

**Inspector State (Updated):**
```javascript
similarities: {
    'Mouse': '0.27',
    'Keyboard': '0.22',
    'USB Cable': '0.17',
    'Monitor': '0.13',
    'Phone Case': '0.03'
}
```

#### **Step 4: Compute Similarity Scores**
- Displays calculated similarity scores in table format
- Shows decimal precision (0.XXXX format)

#### **Step 5: Rank Items by Similarity Score**
- Sorts items in descending order by similarity
- Creates ranked object with ordered entries
- Displays rank position (1, 2, 3, ...)

**Inspector State (Updated):**
```javascript
ranked: {
    'Mouse': '0.27',       // Rank 1
    'Keyboard': '0.22',    // Rank 2
    'USB Cable': '0.17',   // Rank 3
    'Monitor': '0.13',     // Rank 4
    'Phone Case': '0.03'   // Rank 5
}
```

#### **Step 6: Final Recommendation Output**
- Selects top 2 items from ranked list
- Displays final recommendations with similarity scores
- Example output:
  ```
  [1] RECOMMEND: Mouse (similarity: 0.27)
  [2] RECOMMEND: Keyboard (similarity: 0.22)
  ```

### Live Data Inspector

The inspector panel updates dynamically after each step, showing:

- **Selected Product** (Step 1+)
- **Total Purchases** (Step 1+)
- **Co-Purchase Data** (Step 2+)
- **Similarity Scores** (Step 3+)
- **Ranked Items** (Step 5+)

All data displayed in JSON format with syntax highlighting via `<pre>` tags.

---

## NLP Review Intelligence Debugger

### Initialization Trigger

Two methods trigger Step 1 auto-execution:

**1. Debounced Text Input (800ms delay):**
```javascript
reviewInput.addEventListener('input', () => {
    clearTimeout(debounceTimer);
    
    debounceTimer = setTimeout(() => {
        const text = reviewInput.value.trim();
        if (text.length > 0) {
            nlpDebugger.initializeDebugger(text);
        }
    }, 800);
});
```

**2. Enter Key (Immediate execution):**
```javascript
reviewInput.addEventListener('keydown', (e) => {
    if (e.key === 'Enter' && !e.shiftKey) {
        e.preventDefault();
        const text = reviewInput.value.trim();
        if (text.length > 0) {
            nlpDebugger.initializeDebugger(text);
        }
    }
});
```

### Data Structures

**Stopwords Set:**
```javascript
const stopwords = new Set([
    'the', 'a', 'an', 'and', 'or', 'but', 'in', 'on', 'at', 'to', 'for',
    'of', 'with', 'by', 'from', 'is', 'was', 'are', 'been', 'be', 'have',
    // ... ~50 common English stopwords
]);
```

**Sentiment Lexicon:**
```javascript
const sentimentLexicon = {
    positive: ['good', 'great', 'excellent', 'amazing', 'fantastic', 'wonderful',
              'awesome', 'fast', 'quick', 'reliable', 'efficient', 'beautiful',
              'love', 'perfect', 'best', 'satisfied', 'happy', 'impressed'],
    negative: ['bad', 'terrible', 'awful', 'horrible', 'poor', 'slow',
              'broken', 'useless', 'waste', 'disappointed', 'worse', 'worst',
              'late', 'damaged', 'cheap', 'fragile', 'unhappy', 'regret']
};
```

**Aspect Keywords:**
```javascript
const aspectKeywords = {
    'Quality': ['quality', 'material', 'durability', 'build', 'solid', 'sturdy'],
    'Delivery': ['delivery', 'shipping', 'arrived', 'late', 'fast', 'slow', 'package'],
    'Price': ['price', 'expensive', 'cheap', 'cost', 'value', 'worth', 'affordable'],
    'Packaging': ['packaging', 'box', 'wrap', 'presentation', 'unbox']
};
```

### 8-Step Execution Flow

#### **Step 1: Raw Input Text**
- Captures original review text exactly as entered
- Displays character count
- Stores in `debugData.originalText`

**Example:**
```
Input: "The product quality is EXCELLENT! Fast delivery."
Length: 50 characters
```

#### **Step 2: Lowercase Conversion**
- Applies `.toLowerCase()` transformation
- Normalizes text for consistent matching

**Transformation:**
```
Before: "The product quality is EXCELLENT! Fast delivery."
After:  "the product quality is excellent! fast delivery."
```

**State Update:**
```javascript
debugData.lowercase = "the product quality is excellent! fast delivery."
```

#### **Step 3: Remove Punctuation**
- Uses regex: `/[^\w\s]/g` to remove non-alphanumeric characters
- Preserves spaces and word characters

**Transformation:**
```
Before: "the product quality is excellent! fast delivery."
After:  "the product quality is excellent fast delivery"
```

**State Update:**
```javascript
debugData.noPunctuation = "the product quality is excellent fast delivery"
```

#### **Step 4: Remove Stopwords**
- Splits text into tokens: `split(/\s+/)`
- Filters tokens against stopwords set
- Displays removed stopwords count

**Transformation:**
```
Original tokens: ["the", "product", "quality", "is", "excellent", "fast", "delivery"]
Stopwords removed: ["the", "is"] (2 words)
Remaining tokens: ["product", "quality", "excellent", "fast", "delivery"]
```

**State Update:**
```javascript
debugData.tokensAfterStopwords = ["product", "quality", "excellent", "fast", "delivery"]
```

#### **Step 5: Tokenization**
- Displays final token array
- Shows total token count
- Prepares tokens for sentiment analysis

**Output:**
```javascript
tokens = [
    "product",
    "quality",
    "excellent",
    "fast",
    "delivery"
]
Total Tokens: 5
```

#### **Step 6: Sentiment Scoring**
- Iterates through tokens
- Applies lexicon-based scoring:
  - **+1** for each positive word
  - **-1** for each negative word
- Displays step-by-step score accumulation

**Algorithm:**
```javascript
score = 0
// Found: "excellent" (positive)
score += 1  // score = 1

// Found: "fast" (positive)
score += 1  // score = 2

FINAL SCORE = 2
```

**State Update:**
```javascript
debugData.sentimentScore = 2
debugData.positiveWords = ["excellent", "fast"]
debugData.negativeWords = []
```

#### **Step 7: Aspect Detection**
- Maps tokens to aspect categories
- Identifies which product aspects are mentioned
- Shows keywords found for each aspect

**Detection Logic:**
```javascript
for (const aspect in aspectKeywords) {
    const keywords = aspectKeywords[aspect];
    const found = tokens.filter(t => keywords.includes(t));
    if (found.length > 0) {
        detectedAspects[aspect] = found;
    }
}
```

**Example Output:**
```javascript
Quality: ["quality"]
Delivery: ["delivery", "fast"]
```

**State Update:**
```javascript
debugData.detectedAspects = {
    'Quality': ["quality"],
    'Delivery': ["delivery", "fast"]
}
```

#### **Step 8: Insight Generation**
- Classifies overall sentiment (Positive/Negative/Neutral)
- Generates human-readable summary based on aspects and sentiment
- Combines aspect-level and overall sentiment analysis

**Classification Logic:**
```javascript
if (score > 0) classification = 'Positive';
if (score < 0) classification = 'Negative';
if (score === 0) classification = 'Neutral';
```

**Insight Generation:**
```javascript
// Identify positive and negative aspects
for (const aspect in detectedAspects) {
    const keywords = detectedAspects[aspect];
    let aspectScore = 0;
    
    keywords.forEach(kw => {
        if (sentimentLexicon.positive.includes(kw)) aspectScore += 1;
        if (sentimentLexicon.negative.includes(kw)) aspectScore -= 1;
    });
    
    if (aspectScore > 0) positiveAspects.push(aspect);
    if (aspectScore < 0) negativeAspects.push(aspect);
}

// Generate insight
insight = 'Customers appreciate ' + positiveAspects.join(' and ');
```

**Example Output:**
```
Classification: Positive
Score: 2
Generated Insight: "Customers appreciate Quality and Delivery."
```

### Results Panel

Displays structured analysis in right column:

**Classification Badge:**
- Positive (green)
- Negative (red)
- Neutral (gray)

**Score Display:**
- Final sentiment score
- Positive word count
- Negative word count

**Aspects Section:**
- Lists detected aspects
- Shows keywords found for each
- Color-coded by aspect sentiment

**Insight Display:**
- Human-readable summary
- Generated from aspect analysis

---

## UI Architecture

### Three-Column Grid Layout

The application uses CSS Grid for main content area:

```css
.three-column {
    display: grid;
    grid-template-columns: 1fr 2fr 1.2fr;  /* Left : Center : Right */
    gap: 2rem;
    padding: 2rem;
    height: 100%;
    width: 100%;
    flex: 1;
    overflow-y: auto;
}
```

**Column Responsibilities:**

1. **Left Column (1fr)** – Controls & Input
   - Product selection cards (Recommendation)
   - Review textarea (NLP)
   - Control buttons (Next Step, Reset)
   - Progress indicator

2. **Center Column (2fr)** – Algorithm Execution Pipeline
   - Step-by-step visualization
   - Step cards with highlighting
   - Algorithm state rendering
   - Animated step transitions

3. **Right Column (1.2fr)** – Live Data Inspector
   - Real-time state display
   - Intermediate variables
   - Data structure visualization
   - Results panel (NLP)

### Flex + Grid Interaction

The layout combines flexbox containers with grid children:

```css
/* Flex parent */
.tab-container.active {
    display: flex;
}

/* Grid child */
.three-column {
    flex: 1;           /* Flex item property */
    display: grid;     /* Grid container property */
}
```

This allows:
- Tab containers to switch via `display: flex` / `display: none`
- Grid layout to expand within flex parent
- Full viewport utilization

### Responsive Breakpoints

**Desktop (> 1400px):**
```css
grid-template-columns: 1fr 2fr 1.2fr;
```

**Tablet (1000px – 1400px):**
```css
@media (max-width: 1400px) {
    .three-column {
        grid-template-columns: 1fr 1.5fr 1.5fr;
        gap: 1.5rem;
        padding: 1.5rem;
    }
}
```

**Mobile (< 1000px):**
```css
@media (max-width: 1000px) {
    .three-column {
        grid-template-columns: 1fr;  /* Single column stack */
        gap: 1.5rem;
    }
}
```

### Step Card Highlighting

Active step cards use consistent styling:

```css
.step-card.active {
    border-color: var(--accent);
    background: linear-gradient(135deg, #f8fbff 0%, #ffffff 100%);
    box-shadow: var(--shadow-md);
    animation: slideIn 0.4s ease-out;
}

@keyframes slideIn {
    from {
        opacity: 0;
        transform: translateY(20px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}
```

### Tab Indicator Animation

Smooth animated underline for active tab:

```css
.tab-indicator {
    position: absolute;
    bottom: -2px;
    height: 3px;
    background-color: var(--accent);
    transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}
```

JavaScript dynamically updates position:

```javascript
function updateIndicator(activeButton) {
    const { offsetLeft, offsetWidth } = activeButton;
    indicator.style.left = offsetLeft + 'px';
    indicator.style.width = offsetWidth + 'px';
}
```

---

## Design Philosophy

### Manual Step-by-Step Debugging

The application prioritizes **educational transparency** over convenience:

- **No auto-play:** Users must manually click "Next Step"
- **Intermediate state visibility:** Every transformation is shown
- **Deterministic execution:** Same input always produces same output
- **Pausable flow:** Users can study each step before proceeding

### Visible Intermediate State

Traditional AI systems hide internal operations. This debugger exposes:

- Raw data structures (JSON objects)
- Calculation formulas with actual values
- Accumulation logic (score += 1, score -= 1)
- Filtering operations (stopword removal counts)
- Sorting/ranking transformations

### Live Data Inspector

The inspector panel updates immediately after each step, showing:

- **Current algorithm state** (all internal variables)
- **Data transformations** (before/after comparisons)
- **Intermediate results** (similarity scores, tokens, aspects)
- **Final outputs** (recommendations, insights)

### Event-Driven Architecture

Prevention of duplicate initialization:

```javascript
initializeDebugger(reviewText) {
    // Prevent re-initialization with same text
    if (this.isActive && this.reviewText === reviewText) return;
    
    // Continue with initialization...
}
```

Debouncing prevents excessive execution:

```javascript
let debounceTimer = null;

reviewInput.addEventListener('input', () => {
    clearTimeout(debounceTimer);
    debounceTimer = setTimeout(() => {
        nlpDebugger.initializeDebugger(text);
    }, 800);  // 800ms delay after user stops typing
});
```

### State Management

Each debugger maintains isolated state:

```javascript
class RecommendationDebugger {
    constructor() {
        this.selectedProduct = null;
        this.currentStep = 0;
        this.maxSteps = 6;
        this.isActive = false;
        this.debugData = {};
    }
}
```

State resets cleanly on user action:

```javascript
reset() {
    this.selectedProduct = null;
    this.currentStep = 0;
    this.isActive = false;
    this.debugData = {};
    // Clear UI elements
    // Re-render empty state
}
```

### No Side Effects

All operations are pure transformations:

- **Text preprocessing** does not modify original input
- **Sentiment scoring** creates new score variable
- **Ranking** creates new sorted object (does not mutate)
- **Reset** recreates initial state (does not leave artifacts)

---

## Summary

**AI Black Box Breaker** is an educational debugging tool that converts opaque AI algorithms into transparent, step-by-step visualizations. By implementing manual execution, live state inspection, and intermediate variable display, it allows users to understand exactly how recommendation systems and NLP pipelines operate internally.

The application demonstrates that AI is not magic—it is deterministic computation that can be understood, debugged, and demystified through proper visualization and educational tooling.
