# Part 82: NLP - Natural Language Processing

## สารบัญ
1. [NLP Pipeline Overview](#nlp-pipeline)
2. [Text Preprocessing](#text-preprocessing)
3. [NLTK Library](#nltk)
4. [spaCy Library](#spacy)
5. [Word Embeddings](#word-embeddings)
6. [TF-IDF](#tf-idf)
7. [Transformers & Hugging Face](#transformers)
8. [BERT และ Sentence Transformers](#bert)
9. [Text Classification](#text-classification)
10. [Named Entity Recognition (NER)](#ner)
11. [Sentiment Analysis](#sentiment)
12. [Text Generation](#text-generation)
13. [ตัวอย่างโปรแกรมจริง](#real-examples)
14. [แบบฝึกหัด](#exercises)

---

## 1. NLP Pipeline Overview {#nlp-pipeline}

### NLP คืออะไร

Natural Language Processing (NLP) คือสาขาหนึ่งของ AI ที่เกี่ยวข้องกับการให้คอมพิวเตอร์เข้าใจ ตีความ และสร้างภาษามนุษย์

### NLP Pipeline ทั่วไป

```
Raw Text
   ↓
Text Cleaning (remove HTML, special chars)
   ↓
Tokenization (split into tokens)
   ↓
Normalization (lowercase, stemming/lemmatization)
   ↓
Stop Word Removal
   ↓
Feature Extraction (TF-IDF, embeddings)
   ↓
Model Training/Inference
   ↓
Post-processing (decode predictions)
```

### งานหลักใน NLP

| งาน | ตัวอย่าง |
|-----|---------|
| Text Classification | Spam detection, Sentiment |
| NER | Extract names, places, dates |
| Translation | Thai ↔ English |
| Summarization | News articles |
| Question Answering | Reading comprehension |
| Text Generation | GPT-style generation |
| Relation Extraction | "X is founded by Y" |

```python
# ตัวอย่างที่ 1: Basic NLP pipeline overview
# Simple text processing example

text = """
Natural Language Processing (NLP) is a subfield of linguistics, 
computer science, and artificial intelligence concerned with the 
interactions between computers and human language.
"""

# Step 1: Basic cleaning
import re
cleaned = text.strip().lower()
cleaned = re.sub(r'[^\w\s]', '', cleaned)  # remove punctuation
print("Cleaned:", cleaned[:100])

# Step 2: Simple tokenization
tokens = cleaned.split()
print(f"Token count: {len(tokens)}")
print(f"First 10 tokens: {tokens[:10]}")

# Step 3: Remove stop words (manual)
stop_words = {'is', 'a', 'of', 'the', 'and', 'in', 'between', 'with'}
filtered = [t for t in tokens if t not in stop_words]
print(f"After filtering: {len(filtered)} tokens")
print(f"First 10: {filtered[:10]}")
```

---

## 2. Text Preprocessing {#text-preprocessing}

### Tokenization

```python
# ตัวอย่างที่ 2: Different tokenization methods
import re

text = "Hello, world! It's a beautiful day. Let's test tokenization."

# Word tokenization (simple)
words = text.split()
print("Word split:", words[:5])

# Character tokenization
chars = list(text)
print("Chars:", chars[:10])

# Subword tokenization (manual BPE-like)
def basic_tokenize(text):
    """Basic tokenizer with punctuation handling"""
    # Insert spaces around punctuation
    text = re.sub(r'([.,!?;:])', r' \1 ', text)
    # Handle contractions
    text = re.sub(r"'s", " 's", text)
    text = re.sub(r"n't", " n't", text)
    # Normalize whitespace
    text = re.sub(r'\s+', ' ', text).strip()
    return text.split()

tokens = basic_tokenize(text)
print("Tokenized:", tokens)
```

```python
# ตัวอย่างที่ 3: Stemming algorithms
class PorterStemmer:
    """Simplified Porter Stemmer"""
    
    def __init__(self):
        self.suffixes = {
            'ies': 'y',
            'ied': 'y', 
            'ing': '',
            'ed': '',
            'ly': '',
            'er': '',
            'est': '',
            'ness': '',
            'tion': '',
            'ation': '',
        }
    
    def stem(self, word):
        word = word.lower()
        for suffix, replacement in sorted(self.suffixes.items(), 
                                         key=lambda x: -len(x[0])):
            if word.endswith(suffix) and len(word) - len(suffix) >= 3:
                return word[:-len(suffix)] + replacement
        return word
    
    def stem_tokens(self, tokens):
        return [self.stem(t) for t in tokens]

stemmer = PorterStemmer()
words = ['running', 'happily', 'information', 'beautiful', 'stopped']
stemmed = stemmer.stem_tokens(words)
for w, s in zip(words, stemmed):
    print(f"{w} -> {s}")
```

```python
# ตัวอย่างที่ 4: Text cleaning pipeline
import re
import unicodedata

class TextCleaner:
    """Comprehensive text cleaning"""
    
    def __init__(self, lowercase=True, remove_html=True,
                 remove_urls=True, remove_special_chars=True):
        self.lowercase = lowercase
        self.remove_html = remove_html
        self.remove_urls = remove_urls
        self.remove_special_chars = remove_special_chars
    
    def clean(self, text: str) -> str:
        # Remove HTML tags
        if self.remove_html:
            text = re.sub(r'<[^>]+>', ' ', text)
        
        # Remove URLs
        if self.remove_urls:
            text = re.sub(r'http[s]?://\S+', ' ', text)
            text = re.sub(r'www\.\S+', ' ', text)
        
        # Normalize unicode
        text = unicodedata.normalize('NFKC', text)
        
        # Remove special characters
        if self.remove_special_chars:
            text = re.sub(r'[^\w\s\.,!?]', ' ', text)
        
        # Lowercase
        if self.lowercase:
            text = text.lower()
        
        # Fix whitespace
        text = re.sub(r'\s+', ' ', text).strip()
        
        return text
    
    def clean_batch(self, texts):
        return [self.clean(t) for t in texts]

# Test
cleaner = TextCleaner()
test_texts = [
    "<p>Hello <b>World</b>!</p>",
    "Visit https://example.com for more info.",
    "It's a great day! 😊 #python",
]

for text in test_texts:
    cleaned = cleaner.clean(text)
    print(f"Original: {text}")
    print(f"Cleaned:  {cleaned}")
    print()
```

---

## 3. NLTK Library {#nltk}

```python
# ตัวอย่างที่ 5: NLTK basics
# pip install nltk

import nltk
# ดาวน์โหลด resources (ครั้งแรก)
# nltk.download('punkt')
# nltk.download('stopwords')
# nltk.download('wordnet')
# nltk.download('averaged_perceptron_tagger')
# nltk.download('vader_lexicon')

from nltk.tokenize import word_tokenize, sent_tokenize, TweetTokenizer
from nltk.corpus import stopwords
from nltk.stem import PorterStemmer, WordNetLemmatizer
from nltk.tag import pos_tag
from nltk.chunk import ne_chunk

text = """John Smith works at Google in Mountain View, California.
He has been there since 2015 and loves working on NLP projects."""

# Sentence tokenization
sentences = sent_tokenize(text)
print("Sentences:")
for i, sent in enumerate(sentences):
    print(f"  {i+1}. {sent}")

# Word tokenization
words = word_tokenize(text)
print(f"\nWords: {words[:10]}")

# Stop word removal
stop_words = set(stopwords.words('english'))
filtered = [w for w in words if w.lower() not in stop_words and w.isalpha()]
print(f"\nFiltered: {filtered[:10]}")

# Stemming
stemmer = PorterStemmer()
stemmed = [stemmer.stem(w) for w in filtered]
print(f"\nStemmed: {stemmed[:10]}")

# Lemmatization (better than stemming)
lemmatizer = WordNetLemmatizer()
lemmatized = [lemmatizer.lemmatize(w, pos='v') for w in filtered]
print(f"\nLemmatized: {lemmatized[:10]}")
```

```python
# ตัวอย่างที่ 6: NLTK POS Tagging and Named Entities
from nltk import pos_tag, ne_chunk
from nltk.tokenize import word_tokenize

text = "Apple CEO Tim Cook announced new iPhone in Cupertino yesterday."
tokens = word_tokenize(text)

# POS Tagging
pos_tags = pos_tag(tokens)
print("POS Tags:")
for word, tag in pos_tags:
    print(f"  {word:15} {tag}")

# POS tag meanings
pos_meanings = {
    'NN': 'Noun', 'NNP': 'Proper Noun', 'NNS': 'Plural Noun',
    'VB': 'Verb', 'VBD': 'Verb past', 'JJ': 'Adjective',
    'IN': 'Preposition', 'DT': 'Determiner', 'CD': 'Cardinal',
    'RB': 'Adverb', 'PRP': 'Pronoun'
}

print("\nWith meanings:")
for word, tag in pos_tags:
    meaning = pos_meanings.get(tag, tag)
    print(f"  {word:15} {meaning}")
```

```python
# ตัวอย่างที่ 7: NLTK Frequency Distribution
from nltk.probability import FreqDist
from nltk.tokenize import word_tokenize
from nltk.corpus import stopwords
import re

# Sample corpus
corpus = """
Machine learning is a method of data analysis that automates analytical model building.
It is based on the idea that systems can learn from data, identify patterns and make decisions.
Deep learning is a subset of machine learning that uses neural networks with multiple layers.
Natural language processing combines linguistics and machine learning to help computers understand text.
"""

# Preprocess
tokens = word_tokenize(corpus.lower())
stop_words = set(stopwords.words('english'))
clean_tokens = [t for t in tokens if t.isalpha() and t not in stop_words]

# Frequency distribution
fdist = FreqDist(clean_tokens)
print("Top 15 words:")
for word, freq in fdist.most_common(15):
    print(f"  {word:20} {freq}")

# N-grams
from nltk.util import ngrams
bigrams = list(ngrams(clean_tokens, 2))
trigrams = list(ngrams(clean_tokens, 3))

bigram_fdist = FreqDist(bigrams)
print("\nTop 10 bigrams:")
for bigram, freq in bigram_fdist.most_common(10):
    print(f"  {' '.join(bigram):30} {freq}")
```

---

## 4. spaCy Library {#spacy}

```python
# ตัวอย่างที่ 8: spaCy basics
# pip install spacy
# python -m spacy download en_core_web_sm

import spacy

# Load model
# nlp = spacy.load("en_core_web_sm")

# Mock spaCy usage (for demonstration)
class MockToken:
    def __init__(self, text, pos, dep, lemma, is_stop=False, ent_type=''):
        self.text = text
        self.pos_ = pos
        self.dep_ = dep
        self.lemma_ = lemma
        self.is_stop = is_stop
        self.ent_type_ = ent_type
        self.ent_iob_ = 'O' if not ent_type else 'B'

text = "Apple was founded by Steve Jobs and Steve Wozniak in Cupertino."
print(f"Processing: {text}")
print("\nToken analysis:")
print(f"{'Token':20} {'POS':10} {'Lemma':15} {'Stop'}")
print("-" * 60)

# ใน practice จะใช้:
# doc = nlp(text)
# for token in doc:
#     print(f"{token.text:20} {token.pos_:10} {token.lemma_:15} {token.is_stop}")

# Demonstration output
sample_tokens = [
    ("Apple", "PROPN", "apple", False),
    ("was", "AUX", "be", True),
    ("founded", "VERB", "found", False),
    ("by", "ADP", "by", True),
    ("Steve", "PROPN", "Steve", False),
    ("Jobs", "PROPN", "Jobs", False),
    ("in", "ADP", "in", True),
    ("Cupertino", "PROPN", "Cupertino", False),
]

for text, pos, lemma, is_stop in sample_tokens:
    print(f"{text:20} {pos:10} {lemma:15} {is_stop}")
```

```python
# ตัวอย่างที่ 9: spaCy NER and Dependency Parsing
# Simulating spaCy NER output

class NERExample:
    """Demonstrating NER concepts"""
    
    def __init__(self):
        self.entity_colors = {
            'PERSON': '\033[92m',    # Green
            'ORG': '\033[94m',       # Blue
            'GPE': '\033[93m',       # Yellow
            'DATE': '\033[95m',      # Purple
            'MONEY': '\033[96m',     # Cyan
        }
        self.RESET = '\033[0m'
    
    def display_entities(self, text, entities):
        """Display entities with highlighting"""
        print(f"Text: {text}\n")
        print("Entities found:")
        for entity_text, label, start, end in entities:
            color = self.entity_colors.get(label, '')
            print(f"  {color}{entity_text}{self.RESET} → [{label}]")
    
    def extract_info(self, entities):
        """Extract structured info from entities"""
        info = {
            'persons': [],
            'organizations': [],
            'locations': [],
            'dates': []
        }
        
        label_map = {
            'PERSON': 'persons',
            'ORG': 'organizations', 
            'GPE': 'locations',
            'DATE': 'dates'
        }
        
        for text, label, _, _ in entities:
            key = label_map.get(label)
            if key:
                info[key].append(text)
        
        return info

# Example usage
ner = NERExample()
text = "Elon Musk founded SpaceX in 2002 and Tesla in Palo Alto."
entities = [
    ("Elon Musk", "PERSON", 0, 9),
    ("SpaceX", "ORG", 18, 24),
    ("2002", "DATE", 28, 32),
    ("Tesla", "ORG", 37, 42),
    ("Palo Alto", "GPE", 46, 55),
]

ner.display_entities(text, entities)
info = ner.extract_info(entities)
print(f"\nExtracted info:")
for key, vals in info.items():
    if vals:
        print(f"  {key}: {', '.join(vals)}")
```

```python
# ตัวอย่างที่ 10: spaCy text similarity
# spaCy ใช้ word vectors ในการคำนวณ similarity

def cosine_similarity(vec1, vec2):
    """Compute cosine similarity between two vectors"""
    import math
    dot = sum(a * b for a, b in zip(vec1, vec2))
    norm1 = math.sqrt(sum(a**2 for a in vec1))
    norm2 = math.sqrt(sum(b**2 for b in vec2))
    if norm1 == 0 or norm2 == 0:
        return 0
    return dot / (norm1 * norm2)

# Demonstration with random vectors (in practice: use spaCy vectors)
import random
random.seed(42)

def get_mock_vector(word):
    """Generate deterministic mock vector"""
    random.seed(hash(word) % 10000)
    return [random.gauss(0, 1) for _ in range(300)]

word_pairs = [
    ("king", "queen"),
    ("king", "man"),
    ("cat", "dog"),
    ("cat", "car"),
    ("apple", "orange"),
    ("python", "programming"),
]

print("Word Similarity (mock vectors):")
print("-" * 40)
for w1, w2 in word_pairs:
    v1 = get_mock_vector(w1)
    v2 = get_mock_vector(w2)
    sim = cosine_similarity(v1, v2)
    print(f"  {w1:15} ↔ {w2:15}: {sim:.4f}")

# In practice with spaCy:
# nlp = spacy.load("en_core_web_md")  # needs md or lg model
# doc1 = nlp("I like cats")
# doc2 = nlp("I love dogs")
# print(f"Similarity: {doc1.similarity(doc2):.4f}")
```

---

## 5. Word Embeddings {#word-embeddings}

### Word2Vec

```python
# ตัวอย่างที่ 11: Word2Vec implementation concept
import numpy as np
from collections import defaultdict

class Word2VecSimple:
    """
    Simplified Word2Vec (Skip-gram) implementation
    เพื่อทำความเข้าใจ concept
    """
    
    def __init__(self, vocab_size, embedding_dim, window_size=2):
        self.vocab_size = vocab_size
        self.embedding_dim = embedding_dim
        self.window_size = window_size
        
        # Initialize weight matrices
        self.W1 = np.random.randn(vocab_size, embedding_dim) * 0.01  # Input
        self.W2 = np.random.randn(embedding_dim, vocab_size) * 0.01  # Output
    
    def softmax(self, x):
        exp_x = np.exp(x - np.max(x))
        return exp_x / exp_x.sum()
    
    def forward(self, center_idx):
        """Forward pass"""
        # Get embedding for center word
        h = self.W1[center_idx]  # (embedding_dim,)
        
        # Compute output scores
        u = self.W2.T @ h  # (vocab_size,)
        
        # Probabilities
        y_hat = self.softmax(u)
        return h, y_hat
    
    def loss(self, y_hat, true_idx):
        """Cross-entropy loss"""
        return -np.log(y_hat[true_idx] + 1e-10)
    
    def train_step(self, center_idx, context_idx, lr=0.01):
        """One training step"""
        h, y_hat = self.forward(center_idx)
        
        total_loss = 0
        dL_dW1 = np.zeros_like(self.W1)
        dL_dW2 = np.zeros_like(self.W2)
        
        for ctx_idx in context_idx:
            total_loss += self.loss(y_hat, ctx_idx)
            
            # Gradient
            dL_du = y_hat.copy()
            dL_du[ctx_idx] -= 1  # Gradient of cross-entropy + softmax
            
            # Update W2
            dL_dW2 += np.outer(h, dL_du)
            
            # Gradient for h
            dL_dh = self.W2 @ dL_du
            
            # Update W1
            dL_dW1[center_idx] += dL_dh
        
        self.W1 -= lr * dL_dW1
        self.W2 -= lr * dL_dW2
        
        return total_loss
    
    def get_embedding(self, word_idx):
        return self.W1[word_idx]
    
    def most_similar(self, word_idx, top_n=5):
        """Find most similar words"""
        query = self.W1[word_idx]
        
        similarities = []
        for i in range(self.vocab_size):
            if i == word_idx:
                continue
            sim = np.dot(query, self.W1[i]) / (
                np.linalg.norm(query) * np.linalg.norm(self.W1[i]) + 1e-10
            )
            similarities.append((i, sim))
        
        return sorted(similarities, key=lambda x: -x[1])[:top_n]

# Demonstration
w2v = Word2VecSimple(vocab_size=100, embedding_dim=50)

# Simulate training
for _ in range(100):
    center = np.random.randint(100)
    context = [np.random.randint(100) for _ in range(3)]
    w2v.train_step(center, context)

# Get embedding
emb = w2v.get_embedding(0)
print(f"Embedding shape: {emb.shape}")
print(f"Embedding (first 5): {emb[:5]}")

similar = w2v.most_similar(0, top_n=3)
print(f"Most similar to word 0: {similar}")
```

```python
# ตัวอย่างที่ 12: Using Gensim Word2Vec
# pip install gensim

from gensim.models import Word2Vec, KeyedVectors
import numpy as np

# Sample corpus
corpus = [
    ["i", "love", "machine", "learning"],
    ["deep", "learning", "is", "powerful"],
    ["natural", "language", "processing", "nlp"],
    ["word", "embeddings", "capture", "semantics"],
    ["python", "is", "great", "for", "nlp"],
    ["transformer", "models", "revolutionized", "nlp"],
    ["bert", "gpt", "are", "transformer", "models"],
    ["machine", "learning", "powers", "ai"],
    ["language", "models", "understand", "text"],
    ["deep", "neural", "networks", "learn", "representations"],
]

# Train Word2Vec
model = Word2Vec(
    sentences=corpus,
    vector_size=100,    # Embedding dimensions
    window=5,           # Context window
    min_count=1,        # Minimum word frequency
    workers=4,          # Parallel training
    sg=1,               # 1=Skip-gram, 0=CBOW
    epochs=100
)

# Use the model
wv = model.wv  # Word vectors

print("Vocabulary:", list(wv.vocab.keys())[:10] if hasattr(wv, 'vocab') else list(wv.key_to_index.keys())[:10])

# Get vector
print(f"\nVector for 'learning' (first 5): {wv['learning'][:5]}")

# Most similar
try:
    similar = wv.most_similar('learning', topn=5)
    print("\nMost similar to 'learning':")
    for word, sim in similar:
        print(f"  {word:20} {sim:.4f}")
    
    # Word analogies
    # king - man + woman ≈ queen
    result = wv.most_similar(
        positive=['nlp', 'deep'],
        negative=['machine'],
        topn=3
    )
    print("\nWord analogy result:")
    for word, sim in result:
        print(f"  {word:20} {sim:.4f}")
except Exception as e:
    print(f"Note: {e}")

# Save and load
model.save("word2vec_model.bin")
loaded = Word2Vec.load("word2vec_model.bin")
print(f"\nModel loaded, vocab size: {len(loaded.wv)}")
```

```python
# ตัวอย่างที่ 13: GloVe and FastText
# สาธิต concept ของ GloVe

class GloVeDemo:
    """Demonstrating GloVe concept"""
    
    def __init__(self, texts, embedding_dim=50):
        self.embedding_dim = embedding_dim
        # Build vocab
        all_words = [w for text in texts for w in text.split()]
        self.vocab = {w: i for i, w in enumerate(set(all_words))}
        self.vocab_size = len(self.vocab)
        
        # Build co-occurrence matrix
        self.cooc = self._build_cooccurrence(texts)
        
        # Initialize embeddings
        self.W = np.random.randn(self.vocab_size, embedding_dim) * 0.01
        self.W_context = np.random.randn(self.vocab_size, embedding_dim) * 0.01
        self.b = np.zeros(self.vocab_size)
        self.b_context = np.zeros(self.vocab_size)
    
    def _build_cooccurrence(self, texts, window=3):
        """Build co-occurrence matrix"""
        cooc = defaultdict(float)
        for text in texts:
            words = text.split()
            for i, word in enumerate(words):
                for j in range(max(0, i-window), min(len(words), i+window+1)):
                    if i != j:
                        dist = abs(i - j)
                        cooc[(word, words[j])] += 1.0 / dist
        return cooc
    
    def train(self, epochs=50, lr=0.05, x_max=100, alpha=0.75):
        """Train GloVe"""
        total_loss = 0
        pairs = [(i, j, v) for (w_i, w_j), v in self.cooc.items()
                 if w_i in self.vocab and w_j in self.vocab
                 for i, j in [(self.vocab[w_i], self.vocab[w_j])]]
        
        for epoch in range(epochs):
            epoch_loss = 0
            for i, j, x_ij in pairs:
                # Weighting function
                f = min(1.0, (x_ij / x_max) ** alpha) if x_ij < x_max else 1.0
                
                # Forward
                dot = np.dot(self.W[i], self.W_context[j])
                diff = dot + self.b[i] + self.b_context[j] - np.log(x_ij + 1)
                loss = f * diff ** 2
                epoch_loss += loss
                
                # Backward
                grad = 2 * f * diff
                self.W[i] -= lr * grad * self.W_context[j]
                self.W_context[j] -= lr * grad * self.W[i]
                self.b[i] -= lr * grad
                self.b_context[j] -= lr * grad
            
            if (epoch + 1) % 20 == 0:
                print(f"  Epoch {epoch+1}: loss = {epoch_loss:.4f}")
        
        return epoch_loss
    
    def get_embedding(self, word):
        """Get final embedding (average of W and W_context)"""
        if word not in self.vocab:
            return np.zeros(self.embedding_dim)
        idx = self.vocab[word]
        return (self.W[idx] + self.W_context[idx]) / 2

texts = [
    "machine learning is a type of artificial intelligence",
    "deep learning uses neural networks",
    "natural language processing is part of ai",
    "python is popular for machine learning"
]

glove = GloVeDemo(texts, embedding_dim=20)
print("Training GloVe...")
glove.train(epochs=60)

emb_ml = glove.get_embedding("machine")
emb_dl = glove.get_embedding("learning")
emb_ai = glove.get_embedding("intelligence")

print(f"\nEmbedding for 'machine' (first 5): {emb_ml[:5]}")
```

---

## 6. TF-IDF {#tf-idf}

```python
# ตัวอย่างที่ 14: TF-IDF from scratch
import math
from collections import Counter

class TFIDF:
    """Term Frequency - Inverse Document Frequency"""
    
    def __init__(self, max_features=None, min_df=1, max_df=0.95):
        self.max_features = max_features
        self.min_df = min_df
        self.max_df = max_df
        self.vocabulary = {}
        self.idf_values = {}
    
    def _tokenize(self, text):
        return text.lower().split()
    
    def fit(self, corpus):
        """Build vocabulary and IDF values"""
        n_docs = len(corpus)
        
        # Count document frequency for each word
        df = Counter()
        for doc in corpus:
            unique_words = set(self._tokenize(doc))
            df.update(unique_words)
        
        # Filter by min/max df
        vocab = []
        for word, count in df.items():
            doc_freq = count / n_docs
            if self.min_df <= count and doc_freq <= self.max_df:
                vocab.append(word)
        
        # Limit vocabulary size
        if self.max_features:
            vocab = sorted(vocab, key=lambda w: df[w], reverse=True)
            vocab = vocab[:self.max_features]
        
        self.vocabulary = {w: i for i, w in enumerate(sorted(vocab))}
        
        # Compute IDF
        for word in self.vocabulary:
            count = df.get(word, 0)
            self.idf_values[word] = math.log((n_docs + 1) / (count + 1)) + 1
        
        return self
    
    def transform(self, corpus):
        """Transform documents to TF-IDF matrix"""
        n_docs = len(corpus)
        n_features = len(self.vocabulary)
        matrix = [[0.0] * n_features for _ in range(n_docs)]
        
        for doc_idx, doc in enumerate(corpus):
            words = self._tokenize(doc)
            word_counts = Counter(words)
            
            for word, count in word_counts.items():
                if word in self.vocabulary:
                    feat_idx = self.vocabulary[word]
                    tf = count / len(words)
                    idf = self.idf_values[word]
                    matrix[doc_idx][feat_idx] = tf * idf
            
            # L2 normalize
            row = matrix[doc_idx]
            norm = math.sqrt(sum(x**2 for x in row))
            if norm > 0:
                matrix[doc_idx] = [x / norm for x in row]
        
        return matrix
    
    def fit_transform(self, corpus):
        return self.fit(corpus).transform(corpus)
    
    def get_feature_names(self):
        return sorted(self.vocabulary.keys(), key=lambda w: self.vocabulary[w])

# Example usage
corpus = [
    "machine learning is awesome",
    "deep learning uses neural networks",
    "machine learning and deep learning are both AI",
    "natural language processing is NLP",
    "NLP uses machine learning techniques",
]

tfidf = TFIDF(max_features=20)
matrix = tfidf.fit_transform(corpus)
features = tfidf.get_feature_names()

print("TF-IDF Matrix:")
print(f"Shape: {len(matrix)} × {len(matrix[0])}")
print("\nTop features:", features[:10])
print("\nDocument 0 (top 5 weights):")
doc0 = list(zip(features, matrix[0]))
doc0_sorted = sorted(doc0, key=lambda x: -x[1])[:5]
for feat, weight in doc0_sorted:
    print(f"  {feat:20} {weight:.4f}")
```

```python
# ตัวอย่างที่ 15: Sklearn TF-IDF and document similarity
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np

corpus = [
    "Python is a great programming language",
    "Machine learning with Python is powerful",
    "Deep learning requires lots of data",
    "Natural language processing uses ML",
    "Python libraries for data science are great",
    "NLP and deep learning combined are very powerful",
]

# TF-IDF with sklearn
vectorizer = TfidfVectorizer(
    max_features=100,
    stop_words='english',
    ngram_range=(1, 2),   # unigrams and bigrams
    min_df=1,
    max_df=0.9,
    sublinear_tf=True     # Apply log normalization to TF
)

tfidf_matrix = vectorizer.fit_transform(corpus)
print(f"TF-IDF matrix shape: {tfidf_matrix.shape}")
print(f"Features: {vectorizer.get_feature_names_out()[:15]}")

# Document similarity
sim_matrix = cosine_similarity(tfidf_matrix)
print("\nDocument Similarity Matrix:")
print(np.round(sim_matrix, 2))

# Find most similar documents to a query
query = "Python programming for machine learning"
query_vec = vectorizer.transform([query])
similarities = cosine_similarity(query_vec, tfidf_matrix)[0]

print(f"\nQuery: '{query}'")
print("Most similar documents:")
for idx in np.argsort(similarities)[::-1][:3]:
    print(f"  Score: {similarities[idx]:.4f} | {corpus[idx]}")
```

---

## 7. Transformers & Hugging Face {#transformers}

```python
# ตัวอย่างที่ 16: Self-Attention Mechanism
import torch
import torch.nn as nn
import math

class SelfAttention(nn.Module):
    """Multi-head self-attention"""
    
    def __init__(self, d_model, num_heads):
        super().__init__()
        assert d_model % num_heads == 0
        
        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads
        
        self.W_q = nn.Linear(d_model, d_model)
        self.W_k = nn.Linear(d_model, d_model)
        self.W_v = nn.Linear(d_model, d_model)
        self.W_o = nn.Linear(d_model, d_model)
        
    def split_heads(self, x):
        """Split d_model into num_heads × d_k"""
        batch, seq, d_model = x.shape
        x = x.view(batch, seq, self.num_heads, self.d_k)
        return x.permute(0, 2, 1, 3)  # (batch, heads, seq, d_k)
    
    def forward(self, x, mask=None):
        batch, seq, _ = x.shape
        
        # Project queries, keys, values
        Q = self.split_heads(self.W_q(x))  # (batch, heads, seq, d_k)
        K = self.split_heads(self.W_k(x))
        V = self.split_heads(self.W_v(x))
        
        # Scaled dot-product attention
        scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(self.d_k)
        
        if mask is not None:
            scores = scores.masked_fill(mask == 0, -1e9)
        
        attn_weights = torch.softmax(scores, dim=-1)
        
        # Apply attention to values
        context = torch.matmul(attn_weights, V)  # (batch, heads, seq, d_k)
        context = context.permute(0, 2, 1, 3).contiguous()
        context = context.view(batch, seq, self.d_model)
        
        return self.W_o(context), attn_weights

# Test attention
attn = SelfAttention(d_model=256, num_heads=8)
x = torch.randn(4, 10, 256)  # (batch, seq, d_model)
output, weights = attn(x)
print(f"Attention output: {output.shape}")
print(f"Attention weights: {weights.shape}")
```

```python
# ตัวอย่างที่ 17: Hugging Face Transformers
# pip install transformers

from transformers import (
    AutoTokenizer, 
    AutoModelForSequenceClassification,
    AutoModelForTokenClassification,
    AutoModelForCausalLM,
    pipeline
)
import torch

# Text Classification pipeline (sentiment)
print("=== Sentiment Analysis Pipeline ===")
# classifier = pipeline("sentiment-analysis")
# result = classifier("I love this product! It's amazing.")
# print(result)  # [{'label': 'POSITIVE', 'score': 0.9998}]

# Named Entity Recognition pipeline
print("\n=== NER Pipeline ===")
# ner = pipeline("ner", grouped_entities=True)
# result = ner("Elon Musk founded SpaceX in Hawthorne, California in 2002.")
# for entity in result:
#     print(f"  {entity['entity_group']}: {entity['word']} ({entity['score']:.3f})")

# Question Answering
print("\n=== Question Answering Pipeline ===")
# qa = pipeline("question-answering")
# context = "PyTorch was developed by Facebook AI Research. It was released in 2016."
# question = "Who developed PyTorch?"
# result = qa(question=question, context=context)
# print(f"Answer: {result['answer']} (score: {result['score']:.3f})")

# Demonstrate tokenizer usage
from transformers import AutoTokenizer
# tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
# text = "Hello, how are you today?"
# tokens = tokenizer(text, return_tensors="pt")
# print(f"Input IDs: {tokens['input_ids']}")
# print(f"Token count: {tokens['input_ids'].shape[1]}")

print("Hugging Face Transformers - see code above for usage")
print("Install: pip install transformers")
```

---

## 8. BERT และ Sentence Transformers {#bert}

```python
# ตัวอย่างที่ 18: BERT tokenization and features
# from transformers import BertTokenizer, BertModel
# import torch

# tokenizer = BertTokenizer.from_pretrained('bert-base-uncased')
# model = BertModel.from_pretrained('bert-base-uncased')

def demonstrate_bert_tokenization():
    """Demonstrate BERT tokenization concept"""
    
    # BERT special tokens
    # [CLS] = Classification token (start of sequence)
    # [SEP] = Separator token (end of sequence)
    # [PAD] = Padding token
    # [MASK] = Mask token (for MLM training)
    
    text = "The quick brown fox jumps"
    
    # BERT uses WordPiece tokenization
    # Words not in vocab are split into subwords
    example_tokenization = {
        "The": ["the"],
        "quick": ["quick"],
        "brown": ["brown"],
        "fox": ["fox"],
        "jumps": ["jump", "##s"],  # WordPiece subwords
        "unbelievable": ["un", "##belie", "##vable"],
        "tokenization": ["token", "##ization"],
    }
    
    print("BERT WordPiece Tokenization Examples:")
    print("-" * 40)
    for word, subwords in example_tokenization.items():
        print(f"  {word:20} → {' | '.join(subwords)}")
    
    # BERT input format
    text1 = "Hello, how are you?"
    text2 = "I am fine, thanks!"
    
    print(f"\nSingle sentence input:")
    print(f"  [CLS] {text1} [SEP]")
    
    print(f"\nPair input (sentence pair tasks):")
    print(f"  [CLS] {text1} [SEP] {text2} [SEP]")

demonstrate_bert_tokenization()

# BERT use cases
print("\n=== BERT Use Cases ===")
bert_tasks = {
    "Sentence Classification": "Use [CLS] token embedding",
    "Token Classification (NER)": "Use all token embeddings",
    "Question Answering": "Find start/end positions",
    "Sentence Similarity": "Compare [CLS] embeddings",
    "Text Extraction": "Extract relevant spans",
}

for task, method in bert_tasks.items():
    print(f"  {task:35} → {method}")
```

```python
# ตัวอย่างที่ 19: Sentence Transformers
# pip install sentence-transformers

# from sentence_transformers import SentenceTransformer
# import numpy as np

def demonstrate_sentence_transformers():
    """
    Sentence Transformers สร้าง dense sentence embeddings
    ที่จับ semantic meaning ได้ดีกว่า average word embeddings
    """
    
    # Example sentences
    sentences = [
        "The quick brown fox",
        "A fast orange fox",
        "Python is a programming language",
        "Python is used for data science",
        "Machine learning transforms industries",
        "Deep learning is a subset of ML",
    ]
    
    # model = SentenceTransformer('all-MiniLM-L6-v2')
    # embeddings = model.encode(sentences)
    
    # Simulate embeddings (for demonstration)
    import numpy as np
    np.random.seed(42)
    embeddings = np.random.randn(len(sentences), 384)
    
    # Compute similarities
    def cosine_sim(a, b):
        return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b) + 1e-10)
    
    # Similarity matrix
    print("Sentence Similarity Matrix (simulated):")
    print("Sentences:")
    for i, s in enumerate(sentences):
        print(f"  {i}: {s[:50]}")
    
    print("\nSimilarity pairs:")
    pairs = [(0,1), (2,3), (4,5), (0,2), (1,4)]
    for i, j in pairs:
        sim = cosine_sim(embeddings[i], embeddings[j])
        print(f"  [{i}] vs [{j}]: {sim:.4f}")
    
    # Semantic search
    query = "fox in nature"
    query_emb = np.random.randn(384)  # would be model.encode(query)
    
    print(f"\nSemantic search for: '{query}'")
    sims = [cosine_sim(query_emb, emb) for emb in embeddings]
    for idx in np.argsort(sims)[::-1][:3]:
        print(f"  Score {sims[idx]:.4f}: {sentences[idx]}")

demonstrate_sentence_transformers()
```

---

## 9. Text Classification {#text-classification}

```python
# ตัวอย่างที่ 20: Full text classification pipeline
import torch
import torch.nn as nn
from torch.utils.data import Dataset, DataLoader
import numpy as np
from collections import Counter

class Vocabulary:
    """Build vocabulary from corpus"""
    
    def __init__(self, min_freq=1):
        self.min_freq = min_freq
        self.token2idx = {'<PAD>': 0, '<UNK>': 1}
        self.idx2token = {0: '<PAD>', 1: '<UNK>'}
    
    def build(self, texts):
        counter = Counter()
        for text in texts:
            counter.update(text.lower().split())
        
        for word, freq in counter.items():
            if freq >= self.min_freq:
                idx = len(self.token2idx)
                self.token2idx[word] = idx
                self.idx2token[idx] = word
        
        return self
    
    def encode(self, text, max_len=100):
        tokens = text.lower().split()[:max_len]
        ids = [self.token2idx.get(t, 1) for t in tokens]
        # Pad
        ids += [0] * (max_len - len(ids))
        return ids
    
    def __len__(self):
        return len(self.token2idx)

class TextClassificationDataset(Dataset):
    def __init__(self, texts, labels, vocab, max_len=100):
        self.vocab = vocab
        self.max_len = max_len
        self.inputs = [vocab.encode(t, max_len) for t in texts]
        self.labels = labels
    
    def __len__(self):
        return len(self.labels)
    
    def __getitem__(self, idx):
        return (
            torch.tensor(self.inputs[idx], dtype=torch.long),
            torch.tensor(self.labels[idx], dtype=torch.long)
        )

class TextCNN(nn.Module):
    """CNN for text classification"""
    
    def __init__(self, vocab_size, embed_dim, num_classes, 
                 filter_sizes=[2, 3, 4], num_filters=100):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim, padding_idx=0)
        
        self.convs = nn.ModuleList([
            nn.Conv1d(embed_dim, num_filters, fs)
            for fs in filter_sizes
        ])
        
        self.dropout = nn.Dropout(0.5)
        self.fc = nn.Linear(len(filter_sizes) * num_filters, num_classes)
    
    def forward(self, x):
        # x: (batch, seq)
        embedded = self.embedding(x).permute(0, 2, 1)  # (batch, embed, seq)
        
        # Apply convolutions
        pooled = []
        for conv in self.convs:
            activated = torch.relu(conv(embedded))  # (batch, filters, seq-k+1)
            pooled.append(activated.max(dim=2)[0])   # Max pooling
        
        # Concatenate
        cat = torch.cat(pooled, dim=1)  # (batch, 3*filters)
        cat = self.dropout(cat)
        
        return self.fc(cat)

class BiLSTMClassifier(nn.Module):
    """Bidirectional LSTM for classification"""
    
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_classes):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim, padding_idx=0)
        self.lstm = nn.LSTM(
            embed_dim, hidden_dim, num_layers=2,
            batch_first=True, bidirectional=True, dropout=0.3
        )
        self.attention = nn.Linear(hidden_dim * 2, 1)
        self.dropout = nn.Dropout(0.3)
        self.fc = nn.Linear(hidden_dim * 2, num_classes)
    
    def forward(self, x):
        embedded = self.dropout(self.embedding(x))
        
        # LSTM
        output, (hidden, _) = self.lstm(embedded)
        
        # Attention over time steps
        attn_weights = torch.softmax(self.attention(output), dim=1)
        context = (attn_weights * output).sum(dim=1)
        
        return self.fc(self.dropout(context))

# Test
texts = [
    "This is a great movie", "Terrible experience waste of time",
    "Amazing product love it", "Worst purchase ever broken",
    "Highly recommend excellent", "Do not buy poor quality",
    "Five stars outstanding", "One star horrible service",
]
labels = [1, 0, 1, 0, 1, 0, 1, 0]  # 1=positive, 0=negative

vocab = Vocabulary()
vocab.build(texts)

dataset = TextClassificationDataset(texts, labels, vocab, max_len=20)
loader = DataLoader(dataset, batch_size=4, shuffle=True)

# TextCNN
model = TextCNN(len(vocab), embed_dim=50, num_classes=2)
criterion = nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(model.parameters())

for epoch in range(20):
    for X, y in loader:
        optimizer.zero_grad()
        pred = model(X)
        loss = criterion(pred, y)
        loss.backward()
        optimizer.step()

# Test prediction
test_text = "This product is absolutely wonderful"
x = torch.tensor([vocab.encode(test_text)], dtype=torch.long)
with torch.no_grad():
    pred = model(x)
    prob = torch.softmax(pred, dim=1)
    sentiment = "Positive" if prob[0, 1] > 0.5 else "Negative"
    print(f"Text: {test_text}")
    print(f"Prediction: {sentiment} (confidence: {prob[0, 1]:.4f})")
```

---

## 10. Named Entity Recognition {#ner}

```python
# ตัวอย่างที่ 21: BiLSTM-CRF for NER
import torch
import torch.nn as nn

class BiLSTM_NER(nn.Module):
    """Bidirectional LSTM for NER with IOB tagging"""
    
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_tags):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim, padding_idx=0)
        self.lstm = nn.LSTM(
            embed_dim, hidden_dim, 
            batch_first=True, bidirectional=True
        )
        self.dropout = nn.Dropout(0.3)
        self.hidden2tag = nn.Linear(hidden_dim * 2, num_tags)
    
    def forward(self, x):
        embedded = self.dropout(self.embedding(x))
        lstm_out, _ = self.lstm(embedded)
        lstm_out = self.dropout(lstm_out)
        logits = self.hidden2tag(lstm_out)
        return logits

# IOB2 tagging scheme
# B-XX = Beginning of entity type XX
# I-XX = Inside entity type XX
# O = Outside (not an entity)
TAG_TO_IDX = {
    'O': 0,
    'B-PER': 1, 'I-PER': 2,   # Person
    'B-ORG': 3, 'I-ORG': 4,   # Organization
    'B-LOC': 5, 'I-LOC': 6,   # Location
    'B-DATE': 7, 'I-DATE': 8, # Date
}
IDX_TO_TAG = {v: k for k, v in TAG_TO_IDX.items()}

# Example sequence
sentence = ["Steve", "Jobs", "founded", "Apple", "in", "Cupertino", "."]
tags =      ["B-PER", "I-PER", "O", "B-ORG", "O", "B-LOC", "O"]

print("NER Example:")
print("Word         | Tag")
print("-" * 30)
for word, tag in zip(sentence, tags):
    print(f"{word:12} | {tag}")

# Extract entities from IOB tags
def extract_entities(words, tags):
    """Extract entity spans from IOB tags"""
    entities = []
    current_entity = None
    current_type = None
    start = 0
    
    for i, (word, tag) in enumerate(zip(words, tags)):
        if tag.startswith('B-'):
            if current_entity:
                entities.append({
                    'text': ' '.join(current_entity),
                    'type': current_type,
                    'start': start,
                    'end': i - 1
                })
            current_entity = [word]
            current_type = tag[2:]
            start = i
        elif tag.startswith('I-') and current_entity:
            current_entity.append(word)
        else:  # O tag
            if current_entity:
                entities.append({
                    'text': ' '.join(current_entity),
                    'type': current_type,
                    'start': start,
                    'end': i - 1
                })
            current_entity = None
            current_type = None
    
    # Don't forget last entity
    if current_entity:
        entities.append({
            'text': ' '.join(current_entity),
            'type': current_type,
            'start': start,
            'end': len(words) - 1
        })
    
    return entities

entities = extract_entities(sentence, tags)
print("\nExtracted Entities:")
for ent in entities:
    print(f"  '{ent['text']}' → {ent['type']}")
```

---

## 11. Sentiment Analysis {#sentiment}

```python
# ตัวอย่างที่ 22: Lexicon-based sentiment analysis
class LexiconSentimentAnalyzer:
    """Rule-based sentiment analyzer using lexicon"""
    
    def __init__(self):
        # Positive and negative word lists
        self.positive_words = {
            'good', 'great', 'excellent', 'wonderful', 'amazing', 'love',
            'best', 'fantastic', 'outstanding', 'perfect', 'brilliant',
            'awesome', 'superb', 'magnificent', 'splendid', 'marvelous',
            'happy', 'pleased', 'satisfied', 'delighted', 'thrilled'
        }
        
        self.negative_words = {
            'bad', 'terrible', 'awful', 'horrible', 'poor', 'hate',
            'worst', 'disgusting', 'dreadful', 'atrocious', 'disappointing',
            'pathetic', 'useless', 'broken', 'waste', 'garbage',
            'sad', 'angry', 'frustrated', 'disappointed', 'upset'
        }
        
        self.negators = {'not', "n't", 'never', 'no', 'neither', 'nor'}
        
        self.intensifiers = {
            'very': 1.5, 'really': 1.5, 'extremely': 2.0,
            'incredibly': 2.0, 'absolutely': 1.8, 'quite': 1.2,
            'somewhat': 0.7, 'barely': 0.5, 'slightly': 0.6
        }
    
    def analyze(self, text):
        words = text.lower().split()
        score = 0
        negated = False
        intensifier = 1.0
        
        for i, word in enumerate(words):
            # Check negation
            if word in self.negators:
                negated = True
                intensifier = 1.0
                continue
            
            # Check intensifier
            if word in self.intensifiers:
                intensifier = self.intensifiers[word]
                continue
            
            # Compute sentiment
            if word in self.positive_words:
                word_score = intensifier
                if negated:
                    word_score = -word_score * 0.5
                score += word_score
            elif word in self.negative_words:
                word_score = -intensifier
                if negated:
                    word_score = -word_score * 0.5
                score += word_score
            else:
                # Reset negation after non-sentiment word
                if i > 0 and words[i-1] not in self.negators:
                    negated = False
                intensifier = 1.0
        
        # Classify
        if score > 0.5:
            label = 'POSITIVE'
        elif score < -0.5:
            label = 'NEGATIVE'
        else:
            label = 'NEUTRAL'
        
        return {
            'score': score,
            'label': label,
            'magnitude': abs(score)
        }

# Test
analyzer = LexiconSentimentAnalyzer()

test_texts = [
    "This is an excellent product! I absolutely love it.",
    "The service was terrible and the food was awful.",
    "The movie was not bad, actually quite good.",
    "It was okay, nothing special.",
    "This is the worst experience I've ever had.",
    "Very happy with my purchase, brilliant quality!"
]

print("Sentiment Analysis Results:")
print("-" * 60)
for text in test_texts:
    result = analyzer.analyze(text)
    print(f"Text: {text[:50]}")
    print(f"  → {result['label']} (score: {result['score']:+.2f})")
    print()
```

```python
# ตัวอย่างที่ 23: BERT-based sentiment analysis (fine-tuning concept)
import torch
import torch.nn as nn

class BERTSentimentClassifier(nn.Module):
    """
    Fine-tuned BERT for sentiment classification
    Demonstrates the architecture concept
    """
    
    def __init__(self, bert_hidden_dim=768, num_classes=3):
        super().__init__()
        # In practice: self.bert = BertModel.from_pretrained('bert-base-uncased')
        
        # Simulate BERT encoder
        self.bert = nn.Sequential(
            nn.Linear(768, 768),
            nn.GELU(),
            nn.Linear(768, 768)
        )
        
        # Classification head
        self.classifier = nn.Sequential(
            nn.Dropout(0.3),
            nn.Linear(bert_hidden_dim, 256),
            nn.GELU(),
            nn.Dropout(0.1),
            nn.Linear(256, num_classes)
        )
    
    def forward(self, input_ids=None, attention_mask=None):
        # BERT would give us (batch, seq, 768)
        # We use the [CLS] token (first token) for classification
        
        if input_ids is not None:
            # Simulate BERT embedding
            batch_size = input_ids.shape[0]
            cls_representation = torch.randn(batch_size, 768)
        else:
            cls_representation = torch.randn(8, 768)
        
        cls_output = self.bert(cls_representation)
        logits = self.classifier(cls_output)
        return logits

model = BERTSentimentClassifier(num_classes=3)  # neg, neutral, pos

# Test
x = torch.randint(0, 10000, (4, 128))  # (batch, seq_len)
output = model(x)
print(f"BERT Classifier output: {output.shape}")

# Prediction
probs = torch.softmax(output, dim=1)
labels = ['Negative', 'Neutral', 'Positive']
for i in range(4):
    pred_class = probs[i].argmax().item()
    print(f"Sample {i}: {labels[pred_class]} ({probs[i, pred_class]:.4f})")
```

---

## 12. Text Generation {#text-generation}

```python
# ตัวอย่างที่ 24: Language Model for text generation
import torch
import torch.nn as nn
import torch.nn.functional as F

class CharLM(nn.Module):
    """Character-level language model"""
    
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_layers=2):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.lstm = nn.LSTM(
            embed_dim, hidden_dim, num_layers,
            batch_first=True, dropout=0.2
        )
        self.fc = nn.Linear(hidden_dim, vocab_size)
        self.hidden_dim = hidden_dim
        self.num_layers = num_layers
    
    def forward(self, x, hidden=None):
        embedded = self.embedding(x)
        out, hidden = self.lstm(embedded, hidden)
        logits = self.fc(out)
        return logits, hidden
    
    def generate(self, seed_text, vocab, max_len=200, temperature=1.0, 
                 top_k=0, top_p=0.9):
        """Generate text using various sampling strategies"""
        self.eval()
        
        # Encode seed
        input_ids = [vocab.get(c, 0) for c in seed_text]
        input_tensor = torch.tensor([input_ids])
        
        generated = list(seed_text)
        hidden = None
        
        with torch.no_grad():
            for _ in range(max_len):
                logits, hidden = self(input_tensor, hidden)
                logits = logits[0, -1, :] / temperature
                
                # Top-k filtering
                if top_k > 0:
                    top_k_values, _ = torch.topk(logits, top_k)
                    logits[logits < top_k_values[-1]] = float('-inf')
                
                # Top-p (nucleus) filtering
                if top_p < 1.0:
                    sorted_logits, sorted_indices = torch.sort(logits, descending=True)
                    cumulative_probs = torch.cumsum(F.softmax(sorted_logits, dim=-1), dim=-1)
                    sorted_indices_to_remove = cumulative_probs > top_p
                    sorted_indices_to_remove[1:] = sorted_indices_to_remove[:-1].clone()
                    sorted_indices_to_remove[0] = 0
                    indices_to_remove = sorted_indices[sorted_indices_to_remove]
                    logits[indices_to_remove] = float('-inf')
                
                probs = F.softmax(logits, dim=-1)
                next_char_idx = torch.multinomial(probs, 1).item()
                
                # Decode
                idx_to_char = {v: k for k, v in vocab.items()}
                next_char = idx_to_char.get(next_char_idx, '?')
                generated.append(next_char)
                
                # Update input
                input_tensor = torch.tensor([[next_char_idx]])
                hidden = (hidden[0].detach(), hidden[1].detach())
        
        return ''.join(generated)

# Create small char-level model
text = "The quick brown fox jumps over the lazy dog. " * 10
chars = sorted(set(text))
vocab = {c: i for i, c in enumerate(chars)}

model = CharLM(len(vocab), embed_dim=32, hidden_dim=128)
print(f"Vocabulary: {chars}")
print(f"Model: CharLM with {sum(p.numel() for p in model.parameters())} params")

# Generate (before training - will be random)
generated = model.generate("The ", vocab, max_len=50, temperature=1.0)
print(f"\nGenerated: {generated}")
```

---

## 13. ตัวอย่างโปรแกรมจริง {#real-examples}

### Example 1: Sentiment Analyzer with REST API

```python
# ตัวอย่างที่ 25: Production sentiment analyzer
import torch
import torch.nn as nn
from dataclasses import dataclass
from typing import List, Dict, Optional
import re

@dataclass
class SentimentResult:
    text: str
    label: str
    score: float
    confidence: float
    explanation: Optional[Dict] = None

class ProductionSentimentAnalyzer:
    """
    Production-ready sentiment analyzer combining
    lexicon and neural approaches
    """
    
    def __init__(self, model=None, vocab=None):
        self.lexicon_analyzer = LexiconSentimentAnalyzer()
        self.model = model
        self.vocab = vocab
        self.max_len = 128
    
    def preprocess(self, text: str) -> str:
        """Clean and normalize text"""
        text = re.sub(r'http\S+', '', text)
        text = re.sub(r'@\w+', '', text)
        text = re.sub(r'#(\w+)', r'\1', text)
        text = text.strip()
        return text
    
    def analyze(self, text: str) -> SentimentResult:
        """Analyze sentiment of text"""
        cleaned = self.preprocess(text)
        
        # Lexicon analysis
        lex_result = self.lexicon_analyzer.analyze(cleaned)
        
        # If neural model available, use it
        if self.model and self.vocab:
            neural_label = self._neural_predict(cleaned)
            # Ensemble
            final_label = neural_label
        else:
            final_label = lex_result['label']
        
        return SentimentResult(
            text=text,
            label=final_label,
            score=lex_result['score'],
            confidence=min(abs(lex_result['score']) / 3, 1.0),
            explanation={
                'lexicon_score': lex_result['score'],
                'word_count': len(cleaned.split()),
                'cleaned_text': cleaned
            }
        )
    
    def analyze_batch(self, texts: List[str]) -> List[SentimentResult]:
        """Analyze multiple texts"""
        return [self.analyze(t) for t in texts]
    
    def _neural_predict(self, text: str) -> str:
        """Neural model prediction"""
        # Encode text
        tokens = text.lower().split()[:self.max_len]
        ids = [self.vocab.get(t, 1) for t in tokens]
        ids += [0] * (self.max_len - len(ids))
        
        x = torch.tensor([ids], dtype=torch.long)
        
        self.model.eval()
        with torch.no_grad():
            logits = self.model(x)
            probs = torch.softmax(logits, dim=1)
            pred = probs.argmax().item()
        
        return ['NEGATIVE', 'NEUTRAL', 'POSITIVE'][pred]
    
    def get_summary_stats(self, results: List[SentimentResult]) -> Dict:
        """Get summary statistics"""
        total = len(results)
        pos = sum(1 for r in results if r.label == 'POSITIVE')
        neg = sum(1 for r in results if r.label == 'NEGATIVE')
        neu = sum(1 for r in results if r.label == 'NEUTRAL')
        avg_score = sum(r.score for r in results) / total if total > 0 else 0
        
        return {
            'total': total,
            'positive': pos,
            'negative': neg,
            'neutral': neu,
            'positive_ratio': pos / total if total > 0 else 0,
            'negative_ratio': neg / total if total > 0 else 0,
            'average_score': avg_score,
            'overall_sentiment': 'Positive' if avg_score > 0.2 
                               else 'Negative' if avg_score < -0.2 
                               else 'Neutral'
        }

# Test the analyzer
analyzer = ProductionSentimentAnalyzer()

reviews = [
    "This restaurant was absolutely amazing! Best food ever.",
    "Terrible service, waited 45 minutes and the food was cold.",
    "It was okay, nothing special. Average experience.",
    "I really love this book, couldn't put it down!",
    "Disappointed with the quality, not worth the money.",
    "Great value for money, highly recommend to everyone!",
    "The hotel was noisy and the room was dirty.",
    "Fantastic customer service, very helpful staff.",
]

print("=== Sentiment Analysis Results ===\n")
results = analyzer.analyze_batch(reviews)

for result in results:
    print(f"Text: {result.text[:60]}")
    print(f"  Sentiment: {result.label} | Score: {result.score:+.2f} | Confidence: {result.confidence:.2f}")
    print()

# Summary stats
stats = analyzer.get_summary_stats(results)
print("=== Summary Statistics ===")
print(f"Total reviews: {stats['total']}")
print(f"Positive: {stats['positive']} ({stats['positive_ratio']:.1%})")
print(f"Negative: {stats['negative']} ({stats['negative_ratio']:.1%})")
print(f"Neutral: {stats['neutral']}")
print(f"Average score: {stats['average_score']:+.2f}")
print(f"Overall: {stats['overall_sentiment']}")
```

### Example 2: Text Summarizer

```python
# ตัวอย่างที่ 26: Extractive text summarizer
import re
import math
from collections import Counter, defaultdict
from typing import List, Tuple

class ExtractiveSummarizer:
    """
    Extractive text summarizer using:
    1. TF-IDF sentence scoring
    2. Sentence position
    3. Named entity density
    """
    
    def __init__(self, compression_ratio=0.3):
        self.compression_ratio = compression_ratio
    
    def split_sentences(self, text: str) -> List[str]:
        """Split text into sentences"""
        sentences = re.split(r'(?<=[.!?])\s+', text.strip())
        return [s.strip() for s in sentences if len(s.split()) > 3]
    
    def get_word_freq(self, sentences: List[str]) -> Counter:
        """Get word frequencies across all sentences"""
        all_words = []
        stop_words = {'the', 'a', 'an', 'in', 'is', 'are', 'was', 'were',
                     'to', 'of', 'and', 'or', 'but', 'for', 'with', 'it',
                     'this', 'that', 'these', 'those', 'i', 'we', 'they'}
        
        for sent in sentences:
            words = re.findall(r'\b[a-zA-Z]+\b', sent.lower())
            all_words.extend([w for w in words if w not in stop_words])
        
        return Counter(all_words)
    
    def score_sentences(self, sentences: List[str], word_freq: Counter) -> List[float]:
        """Score each sentence"""
        scores = []
        max_freq = max(word_freq.values()) if word_freq else 1
        
        for i, sent in enumerate(sentences):
            words = re.findall(r'\b[a-zA-Z]+\b', sent.lower())
            
            # TF-IDF-like score
            content_score = sum(
                word_freq.get(w, 0) / max_freq 
                for w in words
            ) / max(len(words), 1)
            
            # Position score (first/last sentences more important)
            n = len(sentences)
            if i < n * 0.2:  # First 20%
                position_score = 1.0
            elif i >= n * 0.8:  # Last 20%
                position_score = 0.7
            else:
                position_score = 0.5
            
            # Length score (penalize too short/long)
            word_count = len(words)
            if 10 <= word_count <= 30:
                length_score = 1.0
            elif word_count < 10:
                length_score = 0.6
            else:
                length_score = 0.8
            
            # Combined score
            final_score = (
                0.5 * content_score + 
                0.3 * position_score + 
                0.2 * length_score
            )
            scores.append(final_score)
        
        return scores
    
    def summarize(self, text: str, max_sentences: int = None) -> dict:
        """Generate extractive summary"""
        sentences = self.split_sentences(text)
        
        if len(sentences) <= 3:
            return {'summary': text, 'sentences': sentences, 'scores': [1.0] * len(sentences)}
        
        word_freq = self.get_word_freq(sentences)
        scores = self.score_sentences(sentences, word_freq)
        
        # Select top sentences
        if max_sentences is None:
            max_sentences = max(1, int(len(sentences) * self.compression_ratio))
        
        # Get indices of top-scored sentences
        ranked = sorted(enumerate(scores), key=lambda x: -x[1])
        selected_indices = sorted([idx for idx, _ in ranked[:max_sentences]])
        
        # Build summary maintaining original order
        summary_sentences = [sentences[i] for i in selected_indices]
        summary = ' '.join(summary_sentences)
        
        return {
            'original_length': len(text.split()),
            'summary': summary,
            'summary_length': len(summary.split()),
            'compression': len(summary.split()) / len(text.split()),
            'selected_indices': selected_indices,
            'scores': scores
        }

# Test
article = """
Artificial intelligence (AI) has transformed numerous industries over the past decade.
Machine learning algorithms can now diagnose diseases with unprecedented accuracy.
In healthcare, AI systems analyze medical images and detect cancer at early stages.
Financial institutions use AI to detect fraud and manage risk effectively.
The technology has also revolutionized how we interact with devices through natural language processing.
Virtual assistants like Siri and Alexa have become household names.
Self-driving cars represent another breakthrough, though challenges remain.
The automotive industry is investing billions in autonomous vehicle technology.
AI is also changing education, with personalized learning platforms adapting to each student.
Despite these advances, concerns about job displacement and ethical implications persist.
Many experts argue that AI will create more jobs than it eliminates in the long run.
Regulation and governance frameworks are being developed globally to ensure responsible AI deployment.
The future of AI holds immense promise for solving humanity's greatest challenges.
"""

summarizer = ExtractiveSummarizer(compression_ratio=0.4)
result = summarizer.summarize(article)

print("=== Text Summarization ===")
print(f"Original: {result['original_length']} words")
print(f"Summary: {result['summary_length']} words")
print(f"Compression: {result['compression']:.1%}")
print(f"\nSummary:")
print(result['summary'])
```

### Example 3: Simple Chatbot

```python
# ตัวอย่างที่ 27: Rule-based chatbot
import re
import random
from dataclasses import dataclass, field
from typing import List, Dict, Callable, Optional

@dataclass
class Intent:
    name: str
    patterns: List[str]
    responses: List[str]
    action: Optional[Callable] = None

class ChatBot:
    """Simple rule-based chatbot with intent recognition"""
    
    def __init__(self, name="PyBot"):
        self.name = name
        self.conversation_history = []
        self.context = {}
        self.intents = self._define_intents()
    
    def _define_intents(self) -> List[Intent]:
        return [
            Intent(
                name="greeting",
                patterns=[
                    r'\b(hello|hi|hey|good morning|good afternoon|howdy)\b',
                    r'^hi+$', r'^hello+$'
                ],
                responses=[
                    f"Hello! I'm {self.name}. How can I help you?",
                    "Hi there! What can I do for you?",
                    f"Hey! {self.name} here. What's on your mind?"
                ]
            ),
            Intent(
                name="farewell",
                patterns=[
                    r'\b(bye|goodbye|see you|farewell|take care)\b',
                    r'see you (later|soon|tomorrow)'
                ],
                responses=[
                    "Goodbye! Have a great day!",
                    "See you later! Take care!",
                    "Bye! Feel free to chat anytime!"
                ]
            ),
            Intent(
                name="about_python",
                patterns=[
                    r'(what is|tell me about|explain) python',
                    r'python (programming|language)',
                    r'learn python'
                ],
                responses=[
                    "Python is a versatile, high-level programming language known for its simplicity and readability.",
                    "Python is great for data science, web development, automation, and AI!",
                    "Python is one of the most popular programming languages. It's beginner-friendly yet powerful."
                ]
            ),
            Intent(
                name="about_nlp",
                patterns=[
                    r'(what is|explain|tell me about) nlp',
                    r'natural language processing',
                    r'how does nlp work'
                ],
                responses=[
                    "NLP (Natural Language Processing) is a field combining linguistics and AI to help computers understand human language.",
                    "NLP enables computers to read, understand, and generate human language. It powers chatbots, translation, and sentiment analysis.",
                    "Great topic! NLP uses techniques like tokenization, embeddings, and transformers to process text."
                ]
            ),
            Intent(
                name="weather",
                patterns=[
                    r'(what.s|how.s|what is) the weather',
                    r'weather (today|tomorrow|forecast)',
                    r'is it (raining|sunny|cold|hot)'
                ],
                responses=[
                    "I don't have access to real-time weather data. Please check a weather service!",
                    "For current weather, I'd recommend checking weather.com or your local service.",
                ]
            ),
            Intent(
                name="thanks",
                patterns=[
                    r'\b(thanks|thank you|thx|appreciate it)\b',
                    r'that.s (helpful|great|awesome|perfect)'
                ],
                responses=[
                    "You're welcome! Glad I could help!",
                    "Anytime! Let me know if you need anything else.",
                    "Happy to help! Is there anything else?"
                ]
            ),
            Intent(
                name="help",
                patterns=[
                    r'\b(help|assist|support)\b',
                    r'what can you do',
                    r'your (capabilities|features|functions)'
                ],
                responses=[
                    "I can answer questions about Python, NLP, and general topics. What do you need?",
                    "I'm here to chat! Ask me about Python programming, NLP concepts, or just have a conversation.",
                ]
            )
        ]
    
    def detect_intent(self, text: str) -> Optional[Intent]:
        """Detect intent from user input"""
        text_lower = text.lower()
        
        for intent in self.intents:
            for pattern in intent.patterns:
                if re.search(pattern, text_lower):
                    return intent
        
        return None
    
    def generate_response(self, intent: Optional[Intent], user_input: str) -> str:
        """Generate response based on intent"""
        if intent:
            response = random.choice(intent.responses)
            
            # Execute action if defined
            if intent.action:
                action_result = intent.action(user_input)
                if action_result:
                    response = f"{response}\n{action_result}"
        else:
            fallback_responses = [
                "Interesting! Tell me more.",
                "I'm not sure I understand. Could you rephrase that?",
                "That's a topic I'm still learning about. What else can I help with?",
                "Hmm, I don't have a specific answer for that. Ask me about Python or NLP!",
            ]
            response = random.choice(fallback_responses)
        
        return response
    
    def chat(self, user_input: str) -> str:
        """Process user input and return response"""
        # Add to history
        self.conversation_history.append({
            'role': 'user',
            'content': user_input
        })
        
        # Detect intent
        intent = self.detect_intent(user_input)
        
        # Generate response
        response = self.generate_response(intent, user_input)
        
        # Add response to history
        self.conversation_history.append({
            'role': 'bot',
            'content': response
        })
        
        return response
    
    def run_interactive(self):
        """Run interactive chat session"""
        print(f"\n=== {self.name} Chat ===")
        print("Type 'quit' to exit\n")
        
        while True:
            user_input = input("You: ").strip()
            
            if not user_input:
                continue
            
            if user_input.lower() in ['quit', 'exit', 'q']:
                print(f"{self.name}: Goodbye!")
                break
            
            response = self.chat(user_input)
            print(f"{self.name}: {response}\n")

# Demo
bot = ChatBot("PyBot")

test_inputs = [
    "Hello!",
    "What is Python?",
    "Tell me about NLP",
    "Thanks for the info!",
    "Goodbye!"
]

print("=== Chatbot Demo ===\n")
for user_input in test_inputs:
    print(f"User: {user_input}")
    response = bot.chat(user_input)
    print(f"Bot: {response}\n")

print(f"Conversation history: {len(bot.conversation_history)} messages")
```

```python
# ตัวอย่างที่ 28: Neural machine translation (concept)
import torch
import torch.nn as nn

class Seq2Seq(nn.Module):
    """Sequence-to-Sequence model for translation"""
    
    class Encoder(nn.Module):
        def __init__(self, vocab_size, embed_dim, hidden_dim):
            super().__init__()
            self.embedding = nn.Embedding(vocab_size, embed_dim, padding_idx=0)
            self.rnn = nn.GRU(embed_dim, hidden_dim, batch_first=True, 
                              bidirectional=True)
            self.fc = nn.Linear(hidden_dim * 2, hidden_dim)
        
        def forward(self, x):
            embedded = self.embedding(x)
            outputs, hidden = self.rnn(embedded)
            # Combine bidirectional
            hidden = torch.tanh(self.fc(
                torch.cat([hidden[-2], hidden[-1]], dim=1)
            ))
            return outputs, hidden
    
    class Attention(nn.Module):
        def __init__(self, encoder_dim, decoder_dim):
            super().__init__()
            self.attn = nn.Linear(encoder_dim + decoder_dim, decoder_dim)
            self.v = nn.Linear(decoder_dim, 1, bias=False)
        
        def forward(self, hidden, encoder_outputs):
            # hidden: (batch, dec_dim)
            # encoder_outputs: (batch, src_len, enc_dim)
            src_len = encoder_outputs.shape[1]
            hidden = hidden.unsqueeze(1).repeat(1, src_len, 1)
            
            energy = torch.tanh(self.attn(
                torch.cat([hidden, encoder_outputs], dim=2)
            ))
            attention = self.v(energy).squeeze(2)
            return torch.softmax(attention, dim=1)
    
    class Decoder(nn.Module):
        def __init__(self, vocab_size, embed_dim, hidden_dim, encoder_dim):
            super().__init__()
            self.embedding = nn.Embedding(vocab_size, embed_dim, padding_idx=0)
            self.attention = Seq2Seq.Attention(encoder_dim, hidden_dim)
            self.rnn = nn.GRU(embed_dim + encoder_dim, hidden_dim, batch_first=True)
            self.fc_out = nn.Linear(hidden_dim + encoder_dim + embed_dim, vocab_size)
        
        def forward(self, x, hidden, encoder_outputs):
            x = x.unsqueeze(1)  # (batch, 1)
            embedded = self.embedding(x)  # (batch, 1, embed)
            
            # Attention
            attn_weights = self.attention(hidden, encoder_outputs)
            attn_weights = attn_weights.unsqueeze(1)  # (batch, 1, src)
            context = torch.bmm(attn_weights, encoder_outputs)  # (batch, 1, enc_dim)
            
            # RNN
            rnn_input = torch.cat([embedded, context], dim=2)
            output, hidden = self.rnn(rnn_input, hidden.unsqueeze(0))
            hidden = hidden.squeeze(0)
            
            # Predict
            pred = self.fc_out(
                torch.cat([output.squeeze(1), context.squeeze(1), embedded.squeeze(1)], dim=1)
            )
            return pred, hidden
    
    def __init__(self, src_vocab_size, tgt_vocab_size, 
                 embed_dim=128, hidden_dim=256):
        super().__init__()
        self.encoder = self.Encoder(src_vocab_size, embed_dim, hidden_dim)
        self.decoder = self.Decoder(
            tgt_vocab_size, embed_dim, hidden_dim, 
            encoder_dim=hidden_dim * 2
        )
    
    def forward(self, src, tgt, teacher_forcing_ratio=0.5):
        batch, tgt_len = tgt.shape
        tgt_vocab_size = self.decoder.embedding.num_embeddings
        
        encoder_outputs, hidden = self.encoder(src)
        
        outputs = torch.zeros(batch, tgt_len, tgt_vocab_size)
        x = tgt[:, 0]  # <SOS> token
        
        for t in range(1, tgt_len):
            pred, hidden = self.decoder(x, hidden, encoder_outputs)
            outputs[:, t] = pred
            
            # Teacher forcing
            teacher_force = torch.rand(1).item() < teacher_forcing_ratio
            x = tgt[:, t] if teacher_force else pred.argmax(1)
        
        return outputs

# Test
src_vocab_size = 5000
tgt_vocab_size = 6000

model = Seq2Seq(src_vocab_size, tgt_vocab_size)
src = torch.randint(1, src_vocab_size, (4, 20))  # (batch, src_len)
tgt = torch.randint(1, tgt_vocab_size, (4, 15))  # (batch, tgt_len)

output = model(src, tgt)
print(f"Seq2Seq output: {output.shape}")  # (4, 15, 6000)
```

```python
# ตัวอย่างที่ 29: Document embeddings and clustering
import numpy as np
from sklearn.cluster import KMeans
from sklearn.decomposition import TruncatedSVD
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import TfidfVectorizer

def cluster_documents(texts, n_clusters=3, n_components=50):
    """Cluster documents using TF-IDF + SVD + K-Means"""
    
    # Build TF-IDF
    vectorizer = TfidfVectorizer(
        max_features=1000,
        stop_words='english',
        ngram_range=(1, 2)
    )
    
    # Dimensionality reduction (LSA)
    svd = TruncatedSVD(n_components=n_components, random_state=42)
    
    # Clustering
    kmeans = KMeans(n_clusters=n_clusters, random_state=42, n_init=10)
    
    # Pipeline
    pipeline = Pipeline([
        ('tfidf', vectorizer),
        ('svd', svd),
    ])
    
    # Fit and transform
    doc_vectors = pipeline.fit_transform(texts)
    
    # Cluster
    labels = kmeans.fit_predict(doc_vectors)
    
    # Analyze clusters
    clusters = {}
    for i in range(n_clusters):
        cluster_docs = [texts[j] for j in range(len(texts)) if labels[j] == i]
        
        # Get top TF-IDF terms for cluster
        cluster_tfidf = vectorizer.transform(cluster_docs)
        top_terms_idx = cluster_tfidf.mean(axis=0).A1.argsort()[-10:][::-1]
        top_terms = [vectorizer.get_feature_names_out()[i] for i in top_terms_idx]
        
        clusters[i] = {
            'size': len(cluster_docs),
            'top_terms': top_terms,
            'sample': cluster_docs[0][:100] if cluster_docs else ""
        }
    
    return labels, clusters

# Test with sample documents
documents = [
    "Machine learning is a subset of artificial intelligence",
    "Deep learning uses neural networks with multiple layers",
    "Python is popular for data science and machine learning",
    "Natural language processing analyzes text and speech",
    "Computer vision processes and understands images",
    "Sports teams compete in various tournaments and leagues",
    "Football season starts in September every year",
    "Basketball players practice dribbling and shooting",
    "Olympic games include athletics, swimming, and gymnastics",
    "Tennis tournaments include Wimbledon and US Open",
    "Cooking requires ingredients, heat, and technique",
    "Baking bread at home is a relaxing hobby",
    "Restaurants serve various cuisines from around the world",
    "Food photography has become popular on social media",
    "Healthy eating includes fruits and vegetables daily",
]

labels, clusters = cluster_documents(documents, n_clusters=3)

print("=== Document Clustering ===")
for cluster_id, info in clusters.items():
    print(f"\nCluster {cluster_id} ({info['size']} documents):")
    print(f"  Top terms: {', '.join(info['top_terms'][:5])}")
    print(f"  Sample: {info['sample']}")

print("\nDocument assignments:")
for i, (doc, label) in enumerate(zip(documents, labels)):
    print(f"  [{label}] {doc[:60]}")
```

```python
# ตัวอย่างที่ 30: Information retrieval system
from typing import List, Tuple
import re
import math
from collections import defaultdict

class InformationRetrieval:
    """Simple TF-IDF based information retrieval system"""
    
    def __init__(self):
        self.documents = []
        self.doc_ids = []
        self.index = defaultdict(dict)  # term -> {doc_id: tf}
        self.df = defaultdict(int)
        self.vocab = set()
    
    def _tokenize(self, text: str) -> List[str]:
        """Tokenize and normalize text"""
        tokens = re.findall(r'\b[a-zA-Z]+\b', text.lower())
        stop_words = {'the', 'a', 'an', 'in', 'is', 'are', 'was', 'were',
                     'to', 'of', 'and', 'or', 'it', 'this', 'that', 'be'}
        return [t for t in tokens if t not in stop_words and len(t) > 1]
    
    def add_document(self, doc_id: str, text: str):
        """Add document to index"""
        tokens = self._tokenize(text)
        self.documents.append(text)
        self.doc_ids.append(doc_id)
        
        # TF for this document
        term_count = defaultdict(int)
        for token in tokens:
            term_count[token] += 1
        
        # Normalize TF
        max_count = max(term_count.values()) if term_count else 1
        for term, count in term_count.items():
            self.index[term][doc_id] = count / max_count
            self.df[term] += 1
            self.vocab.add(term)
    
    def search(self, query: str, top_k: int = 5) -> List[Tuple[str, float, str]]:
        """Search documents using cosine similarity"""
        n_docs = len(self.documents)
        if n_docs == 0:
            return []
        
        query_tokens = self._tokenize(query)
        
        # Query TF-IDF
        query_tf = defaultdict(float)
        for token in query_tokens:
            query_tf[token] += 1
        
        # Score documents
        doc_scores = defaultdict(float)
        query_norm = 0
        
        for term, q_tf in query_tf.items():
            if term not in self.index:
                continue
            
            # IDF
            idf = math.log((n_docs + 1) / (self.df.get(term, 0) + 1)) + 1
            q_weight = q_tf * idf
            query_norm += q_weight ** 2
            
            # Add to doc scores
            for doc_id, d_tf in self.index[term].items():
                d_weight = d_tf * idf
                doc_scores[doc_id] += q_weight * d_weight
        
        if query_norm == 0:
            return []
        
        query_norm = math.sqrt(query_norm)
        
        # Normalize scores
        results = []
        for doc_id, score in doc_scores.items():
            doc_idx = self.doc_ids.index(doc_id)
            doc_text = self.documents[doc_idx]
            normalized_score = score / query_norm
            results.append((doc_id, normalized_score, doc_text))
        
        return sorted(results, key=lambda x: -x[1])[:top_k]
    
    def highlight_query(self, text: str, query: str) -> str:
        """Highlight query terms in text"""
        query_tokens = self._tokenize(query)
        words = text.split()
        highlighted = []
        for word in words:
            clean = re.sub(r'[^\w]', '', word.lower())
            if clean in query_tokens:
                highlighted.append(f"**{word}**")
            else:
                highlighted.append(word)
        return ' '.join(highlighted)

# Build index
ir = InformationRetrieval()

corpus = {
    "doc1": "Python is a high-level programming language for general-purpose software development.",
    "doc2": "Machine learning algorithms learn patterns from data to make predictions.",
    "doc3": "Natural language processing enables computers to understand human text and speech.",
    "doc4": "Deep learning uses neural networks with many layers for complex tasks.",
    "doc5": "Python has many libraries like NumPy, Pandas, and Scikit-learn for data science.",
    "doc6": "Transformers and BERT revolutionized NLP tasks like translation and summarization.",
    "doc7": "Data science combines statistics, programming, and domain expertise.",
    "doc8": "Computer vision processes images and videos using deep learning models.",
}

for doc_id, text in corpus.items():
    ir.add_document(doc_id, text)

print("=== Information Retrieval System ===")
print(f"Indexed {len(ir.documents)} documents, {len(ir.vocab)} unique terms\n")

queries = [
    "Python programming language libraries",
    "machine learning deep neural networks",
    "NLP text processing BERT transformers",
]

for query in queries:
    print(f"Query: '{query}'")
    results = ir.search(query, top_k=3)
    for doc_id, score, text in results:
        print(f"  [{doc_id}] Score: {score:.4f}")
        highlighted = ir.highlight_query(text, query)
        print(f"    {highlighted[:100]}")
    print()
```

---

## 14. แบบฝึกหัด {#exercises}

### แบบฝึกหัดที่ 1: Text Preprocessing Pipeline

```python
# เฉลย
import re
from typing import List

class NLPPipeline:
    """Complete NLP preprocessing pipeline"""
    
    def __init__(self):
        self.stop_words = {
            'a', 'an', 'the', 'in', 'on', 'at', 'to', 'for', 'of', 'and',
            'is', 'are', 'was', 'were', 'be', 'been', 'being', 'have', 'has',
            'had', 'do', 'does', 'did', 'will', 'would', 'could', 'should',
            'may', 'might', 'must', 'can', 'it', 'this', 'that', 'these',
            'those', 'i', 'we', 'you', 'they', 'he', 'she', 'or', 'but',
            'not', 'no', 'nor', 'so', 'yet', 'both', 'either', 'neither'
        }
    
    def clean(self, text: str) -> str:
        text = re.sub(r'<[^>]+>', ' ', text)
        text = re.sub(r'http\S+', ' ', text)
        text = re.sub(r'[^\w\s]', ' ', text)
        text = text.lower()
        text = re.sub(r'\s+', ' ', text).strip()
        return text
    
    def tokenize(self, text: str) -> List[str]:
        return text.split()
    
    def remove_stopwords(self, tokens: List[str]) -> List[str]:
        return [t for t in tokens if t not in self.stop_words]
    
    def simple_stem(self, word: str) -> str:
        suffixes = ['ing', 'tion', 'ness', 'er', 'ly', 'ed', 'est', 'ies', 'y']
        for suffix in suffixes:
            if word.endswith(suffix) and len(word) - len(suffix) >= 3:
                return word[:-len(suffix)]
        return word
    
    def process(self, text: str, stem=True, remove_stops=True) -> List[str]:
        text = self.clean(text)
        tokens = self.tokenize(text)
        if remove_stops:
            tokens = self.remove_stopwords(tokens)
        if stem:
            tokens = [self.simple_stem(t) for t in tokens]
        return tokens
    
    def process_batch(self, texts, **kwargs):
        return [self.process(t, **kwargs) for t in texts]

# Test
pipeline = NLPPipeline()
texts = [
    "<p>Machine learning is transforming how we process information!</p>",
    "Natural language processing enables computers to understand human speech.",
    "Deep learning models are being trained on massive datasets.",
]

for text in texts:
    processed = pipeline.process(text)
    print(f"Original: {text[:60]}")
    print(f"Processed: {processed}")
    print()
```

### แบบฝึกหัดที่ 2: Implement Byte-Pair Encoding (BPE)

```python
# เฉลย
from collections import Counter, defaultdict

class BPETokenizer:
    """Byte-Pair Encoding tokenizer"""
    
    def __init__(self, vocab_size=500):
        self.vocab_size = vocab_size
        self.merges = []
        self.vocab = {}
    
    def _get_pairs(self, vocab):
        """Count pairs of symbols"""
        pairs = Counter()
        for word, freq in vocab.items():
            symbols = word.split()
            for i in range(len(symbols) - 1):
                pairs[(symbols[i], symbols[i+1])] += freq
        return pairs
    
    def _merge_vocab(self, pair, vocab):
        """Merge most common pair"""
        v_out = {}
        bigram = ' '.join(pair)
        replacement = ''.join(pair)
        
        for word, freq in vocab.items():
            w_out = word.replace(bigram, replacement)
            v_out[w_out] = freq
        
        return v_out
    
    def fit(self, texts):
        """Learn BPE merges from corpus"""
        # Initialize vocab with character-level
        word_freq = Counter()
        for text in texts:
            for word in text.split():
                # Add end-of-word marker
                word_freq[' '.join(list(word)) + ' </w>'] += 1
        
        vocab = dict(word_freq)
        
        # Build character vocab
        for word in vocab:
            for char in word.split():
                self.vocab[char] = len(self.vocab)
        
        # BPE iterations
        num_merges = self.vocab_size - len(self.vocab)
        
        for i in range(max(0, num_merges)):
            pairs = self._get_pairs(vocab)
            if not pairs:
                break
            
            # Get most common pair
            best_pair = max(pairs, key=pairs.get)
            vocab = self._merge_vocab(best_pair, vocab)
            
            # Save merge
            new_symbol = ''.join(best_pair)
            self.merges.append(best_pair)
            self.vocab[new_symbol] = len(self.vocab)
        
        self.word_vocab = vocab
        return self
    
    def tokenize(self, text):
        """Tokenize using learned merges"""
        tokens = []
        for word in text.split():
            # Start with characters
            word_tokens = list(word) + ['</w>']
            
            # Apply merges
            for merge_pair in self.merges:
                i = 0
                while i < len(word_tokens) - 1:
                    if word_tokens[i] == merge_pair[0] and word_tokens[i+1] == merge_pair[1]:
                        word_tokens = (word_tokens[:i] + 
                                     [''.join(merge_pair)] + 
                                     word_tokens[i+2:])
                    else:
                        i += 1
            
            tokens.extend(word_tokens)
        
        return tokens

# Test
corpus = [
    "lower newer older elder",
    "lowest newest oldest eldest",
    "running jumping walking swimming",
    "runner jumper walker swimmer",
]

tokenizer = BPETokenizer(vocab_size=50)
tokenizer.fit(corpus)

print(f"Vocabulary size: {len(tokenizer.vocab)}")
print(f"Number of merges: {len(tokenizer.merges)}")
print(f"\nTop 10 merges: {tokenizer.merges[:10]}")

test_word = "running"
tokens = tokenizer.tokenize(test_word)
print(f"\nTokenized '{test_word}': {tokens}")
```

### แบบฝึกหัดที่ 3: N-gram Language Model

```python
# เฉลย
from collections import defaultdict, Counter
import random
import math

class NGramLM:
    """N-gram language model with Laplace smoothing"""
    
    def __init__(self, n=2):
        self.n = n
        self.ngrams = defaultdict(Counter)
        self.vocab = set()
        self.total_tokens = 0
    
    def tokenize(self, text):
        return ['<s>'] * (self.n - 1) + text.lower().split() + ['</s>']
    
    def train(self, texts):
        """Train on corpus"""
        for text in texts:
            tokens = self.tokenize(text)
            self.vocab.update(tokens)
            self.total_tokens += len(tokens) - (self.n - 1)
            
            for i in range(len(tokens) - self.n + 1):
                context = tuple(tokens[i:i+self.n-1])
                next_word = tokens[i+self.n-1]
                self.ngrams[context][next_word] += 1
        
        return self
    
    def probability(self, context, word, smoothing=True):
        """Compute P(word | context)"""
        context = tuple(context[-(self.n-1):])
        
        if context not in self.ngrams:
            return 1 / len(self.vocab) if smoothing else 0
        
        context_count = sum(self.ngrams[context].values())
        word_count = self.ngrams[context].get(word, 0)
        
        if smoothing:
            # Laplace smoothing
            return (word_count + 1) / (context_count + len(self.vocab))
        else:
            return word_count / context_count if context_count > 0 else 0
    
    def perplexity(self, text):
        """Compute perplexity of text"""
        tokens = self.tokenize(text)
        log_prob = 0
        n = 0
        
        for i in range(self.n - 1, len(tokens)):
            context = tokens[i-self.n+1:i]
            word = tokens[i]
            prob = self.probability(context, word)
            log_prob += math.log(prob + 1e-10)
            n += 1
        
        if n == 0:
            return float('inf')
        return math.exp(-log_prob / n)
    
    def generate(self, seed=None, max_len=20, temperature=1.0):
        """Generate text"""
        if seed:
            tokens = ['<s>'] * (self.n - 1) + seed.split()
        else:
            tokens = ['<s>'] * (self.n - 1)
        
        for _ in range(max_len):
            context = tuple(tokens[-(self.n-1):])
            
            if context not in self.ngrams:
                # Random fallback
                next_word = random.choice(list(self.vocab - {'<s>', '</s>'}))
            else:
                # Sample from distribution
                candidates = list(self.ngrams[context].keys())
                weights = [self.ngrams[context][w] ** (1/temperature) 
                          for w in candidates]
                total = sum(weights)
                weights = [w/total for w in weights]
                
                next_word = random.choices(candidates, weights=weights)[0]
            
            if next_word == '</s>':
                break
            
            tokens.append(next_word)
        
        return ' '.join(t for t in tokens if t not in ['<s>', '</s>'])

# Train bigram model
corpus = [
    "the cat sat on the mat",
    "the dog ran in the park",
    "a cat and a dog played",
    "the quick brown fox jumped",
    "the lazy dog slept all day",
    "machines learn from data",
    "neural networks process information",
    "language models predict next words",
]

bigram_lm = NGramLM(n=2)
bigram_lm.train(corpus)

trigram_lm = NGramLM(n=3)
trigram_lm.train(corpus)

print("N-gram Language Model")
print(f"Vocabulary: {len(bigram_lm.vocab)} words")

# Test probabilities
print(f"\nP(cat | the) = {bigram_lm.probability(['the'], 'cat'):.4f}")
print(f"P(dog | the) = {bigram_lm.probability(['the'], 'dog'):.4f}")

# Perplexity
test = "the cat sat in the park"
print(f"\nPerplexity (bigram): {bigram_lm.perplexity(test):.2f}")

# Generate
print("\nGenerated text:")
for _ in range(3):
    print(f"  {bigram_lm.generate(max_len=10)}")
```

### แบบฝึกหัดที่ 4-8: Additional Exercises

```python
# แบบฝึกหัดที่ 4: Text Similarity
def text_similarity_demo():
    """Multiple similarity measures"""
    
    def jaccard(text1, text2):
        """Jaccard similarity"""
        s1 = set(text1.lower().split())
        s2 = set(text2.lower().split())
        if not s1 and not s2:
            return 1.0
        return len(s1 & s2) / len(s1 | s2)
    
    def overlap(text1, text2):
        """Overlap coefficient"""
        s1 = set(text1.lower().split())
        s2 = set(text2.lower().split())
        if not s1 or not s2:
            return 0.0
        return len(s1 & s2) / min(len(s1), len(s2))
    
    def edit_distance(s1, s2):
        """Levenshtein edit distance"""
        m, n = len(s1), len(s2)
        dp = [[0] * (n + 1) for _ in range(m + 1)]
        
        for i in range(m + 1):
            dp[i][0] = i
        for j in range(n + 1):
            dp[0][j] = j
        
        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if s1[i-1] == s2[j-1]:
                    dp[i][j] = dp[i-1][j-1]
                else:
                    dp[i][j] = 1 + min(dp[i-1][j],    # delete
                                       dp[i][j-1],    # insert
                                       dp[i-1][j-1])  # replace
        
        return dp[m][n]
    
    def normalized_edit_distance(s1, s2):
        dist = edit_distance(s1, s2)
        return 1 - dist / max(len(s1), len(s2), 1)
    
    # Test
    pairs = [
        ("Python programming", "Python coding"),
        ("machine learning", "deep learning"),
        ("hello world", "bye world"),
        ("cats are great", "dogs are great"),
    ]
    
    print("Text Similarity Measures:")
    print(f"{'Text 1':30} {'Text 2':30} {'Jaccard':10} {'Overlap':10} {'EditDist':10}")
    print("-" * 90)
    for t1, t2 in pairs:
        j = jaccard(t1, t2)
        o = overlap(t1, t2)
        e = normalized_edit_distance(t1, t2)
        print(f"{t1:30} {t2:30} {j:10.4f} {o:10.4f} {e:10.4f}")

text_similarity_demo()
```

```python
# แบบฝึกหัดที่ 5: Named Entity Recognition with rules
import re

class RuleBasedNER:
    """Rule-based NER system"""
    
    def __init__(self):
        self.patterns = [
            # Dates
            (r'\b\d{4}\b', 'DATE'),
            (r'\b(January|February|March|April|May|June|July|August|'
             r'September|October|November|December)\s+\d{1,2},?\s+\d{4}\b', 'DATE'),
            # Money
            (r'\$[\d,]+(?:\.\d+)?(?:\s*(?:million|billion))?', 'MONEY'),
            (r'\d+\s*(?:million|billion)\s*dollars?', 'MONEY'),
            # Percentages
            (r'\d+(?:\.\d+)?\s*%', 'PERCENT'),
            # Organizations (simple heuristic)
            (r'\b[A-Z][a-z]+(?:\s+[A-Z][a-z]+)*\s+(?:Inc|Corp|Ltd|LLC|Co)\b', 'ORG'),
            # URLs
            (r'https?://\S+', 'URL'),
            # Email
            (r'\b[\w.+-]+@[\w-]+\.[a-z]{2,}\b', 'EMAIL'),
        ]
        
        # Known entities
        self.known_persons = {
            'Elon Musk', 'Tim Cook', 'Sundar Pichai', 'Jeff Bezos',
            'Steve Jobs', 'Bill Gates', 'Mark Zuckerberg'
        }
        self.known_orgs = {
            'Apple', 'Google', 'Microsoft', 'Amazon', 'Tesla',
            'SpaceX', 'Meta', 'Facebook', 'Twitter', 'OpenAI'
        }
        self.known_places = {
            'California', 'New York', 'Texas', 'Silicon Valley',
            'San Francisco', 'Seattle', 'Boston', 'Chicago'
        }
    
    def extract(self, text):
        entities = []
        
        # Rule-based patterns
        for pattern, entity_type in self.patterns:
            for match in re.finditer(pattern, text, re.IGNORECASE):
                entities.append({
                    'text': match.group(),
                    'type': entity_type,
                    'start': match.start(),
                    'end': match.end(),
                    'method': 'rule'
                })
        
        # Known entity lookup
        for entity in self.known_persons:
            if entity in text:
                start = text.index(entity)
                entities.append({
                    'text': entity,
                    'type': 'PERSON',
                    'start': start,
                    'end': start + len(entity),
                    'method': 'lookup'
                })
        
        for entity in self.known_orgs:
            if entity in text:
                start = text.index(entity)
                entities.append({
                    'text': entity,
                    'type': 'ORG',
                    'start': start,
                    'end': start + len(entity),
                    'method': 'lookup'
                })
        
        for entity in self.known_places:
            if entity in text:
                start = text.index(entity)
                entities.append({
                    'text': entity,
                    'type': 'GPE',
                    'start': start,
                    'end': start + len(entity),
                    'method': 'lookup'
                })
        
        # Remove duplicates and sort by position
        seen = set()
        unique_entities = []
        for e in sorted(entities, key=lambda x: x['start']):
            key = (e['start'], e['end'])
            if key not in seen:
                seen.add(key)
                unique_entities.append(e)
        
        return unique_entities

# Test
ner = RuleBasedNER()
texts = [
    "Elon Musk founded Tesla in 2003 and SpaceX in California.",
    "Apple's revenue reached $383 billion in 2023, up 12.5%.",
    "Tim Cook sent email to developers@apple.com from San Francisco.",
]

for text in texts:
    print(f"\nText: {text}")
    entities = ner.extract(text)
    for ent in entities:
        print(f"  [{ent['type']:8}] {ent['text']:30} ({ent['method']})")
```

---

## สรุป

ใน Part 82 นี้ เราได้เรียนรู้:

1. **NLP Pipeline** - ขั้นตอนการประมวลผลภาษา
2. **Text Preprocessing** - tokenization, cleaning, stemming, lemmatization
3. **NLTK** - library พื้นฐาน: POS tagging, NER, frequency distribution
4. **spaCy** - library ที่ทันสมัยสำหรับ NLP
5. **Word Embeddings** - Word2Vec, GloVe สำหรับ semantic representation
6. **TF-IDF** - feature extraction สำหรับ text
7. **Transformers** - BERT architecture และ Hugging Face
8. **Text Classification** - CNN, BiLSTM สำหรับ classification
9. **NER** - Named Entity Recognition ด้วย IOB tagging
10. **Sentiment Analysis** - Lexicon-based และ neural approaches
11. **Text Generation** - Language models และ sampling strategies

### ขั้นต่อไป
- Part 83: Computer Vision with OpenCV & PIL
- ศึกษาเพิ่มเติม: https://huggingface.co/

---
*Part 82 - NLP: Natural Language Processing | Python Course*
