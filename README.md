# Recommender NLP Visualizer
### Interactive Algorithm Debugger for E-Commerce

AI systems in modern e-commerce platforms like Amazon and Shopify operate as complex "black boxes."  
This project breaks those systems open and visualizes their internal working step-by-step.

---

## 🎯 Purpose

Slides explain theory.  
This application demonstrates **algorithmic execution**.

It allows users to manually debug:

1. Amazon-style Item-to-Item Collaborative Filtering
2. Shopify-style NLP Review Intelligence Pipeline

---

## 🧠 Module 1: Recommendation Engine Debugger

Simulates item-to-item collaborative filtering.

### Algorithm Flow:
1. Capture user interaction
2. Retrieve co-purchase dataset
3. Construct similarity formula
4. Compute similarity scores
5. Rank items
6. Generate final recommendations

Similarity formula used:

similarity = coPurchaseCount / totalPurchases

All similarity scores are calculated dynamically and displayed in real time.

---

## 📝 Module 2: Review Intelligence Debugger

Simulates Natural Language Processing pipeline.

### NLP Flow:
1. Raw text input
2. Lowercase transformation
3. Punctuation removal
4. Stopword removal
5. Tokenization
6. Sentiment scoring
7. Aspect detection
8. Insight generation

Sentiment scoring:
+1 → positive keywords  
-1 → negative keywords  

Aspect detection is performed using keyword mapping.

---

## 🔍 What Makes This Unique?

- Manual step-by-step debugging
- Live data inspector
- Intermediate computational states
- Real numeric calculations
- Transparent algorithm visualization

This is not a mock UI.
It is an algorithm visualization tool.

---

## 🏗️ Built With

- HTML
- CSS (Grid + Flexbox)
- Vanilla JavaScript
- Font Awesome

No frameworks used.

---

## 📌 Educational Value

This tool helps students understand:

- Collaborative filtering logic
- Similarity score computation
- NLP preprocessing stages
- Sentiment analysis mechanics
- Aspect extraction

---

## 🚀 Future Enhancements

- Cosine similarity implementation
- Real TF-IDF weighting
- Word embeddings
- Model accuracy comparison

---

This project was built to demonstrate the working principles behind AI systems in e-commerce platforms.
