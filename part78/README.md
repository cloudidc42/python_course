# Part 78: Machine Learning - Classification

## บทนำ

Classification คือ task ใน Supervised Learning ที่ทำนาย class หรือ category ของข้อมูล เช่น spam/not spam, โรค/ไม่เป็นโรค, ประเภทสัตว์ บทนี้ครอบคลุม classification algorithms ทั้งหมดตั้งแต่ Logistic Regression ไปถึง Neural Networks

---

## 1. Logistic Regression

แม้ชื่อจะมีคำว่า Regression แต่ Logistic Regression ใช้สำหรับ Classification โดยใช้ sigmoid function แปลง linear combination เป็น probability

$$P(y=1|x) = \sigma(\beta_0 + \beta_1 x_1 + ... + \beta_p x_p) = \frac{1}{1 + e^{-z}}$$

```python
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import classification_report, roc_auc_score
import numpy as np

bc = load_breast_cancer()
X, y = bc.data, bc.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

# Logistic Regression
lr = LogisticRegression(
    C=1.0,          # inverse regularization strength
    penalty='l2',   # L2 regularization
    solver='lbfgs',
    max_iter=10000,
    random_state=42
)
lr.fit(X_train_s, y_train)

y_pred = lr.predict(X_test_s)
y_prob = lr.predict_proba(X_test_s)[:, 1]

print("Logistic Regression Results:")
print(f"  Accuracy: {lr.score(X_test_s, y_test):.4f}")
print(f"  ROC-AUC: {roc_auc_score(y_test, y_prob):.4f}")
print(f"\nClassification Report:")
print(classification_report(y_test, y_pred, target_names=bc.target_names))

# Coefficients (feature importance)
import pandas as pd
coef_df = pd.DataFrame({
    'feature': bc.feature_names,
    'coefficient': lr.coef_[0]
}).sort_values('coefficient', key=abs, ascending=False)
print(f"\nTop 5 most important features:")
print(coef_df.head())
```

```python
# Effect of regularization strength C
import matplotlib.pyplot as plt
C_values = np.logspace(-4, 4, 20)
accuracies = []

for C in C_values:
    lr = LogisticRegression(C=C, max_iter=10000, random_state=42)
    scores = cross_val_score(
        lr, X_train_s, y_train, cv=5, scoring='accuracy'
    )
    accuracies.append(scores.mean())

plt.figure(figsize=(10, 6))
plt.semilogx(C_values, accuracies, 'o-', color='blue')
plt.xlabel('C (inverse regularization)')
plt.ylabel('CV Accuracy')
plt.title('Logistic Regression - Effect of C')
plt.axvline(C_values[np.argmax(accuracies)], color='red', linestyle='--',
            label=f'Best C={C_values[np.argmax(accuracies)]:.4f}')
plt.legend()
plt.grid(True)
plt.savefig('lr_regularization.png', dpi=100)
print("Plot saved!")
```

### 1.1 Multi-class Logistic Regression

```python
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import classification_report

iris = load_iris()
X, y = iris.data, iris.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

# Multinomial Logistic Regression
lr_multi = LogisticRegression(
    multi_class='multinomial',  # หรือ 'ovr' (one-vs-rest)
    solver='lbfgs',
    max_iter=10000,
    C=1.0,
    random_state=42
)
lr_multi.fit(X_train_s, y_train)

print("Multinomial Logistic Regression - Iris:")
print(classification_report(y_test, lr_multi.predict(X_test_s), 
                              target_names=iris.target_names))
print(f"\nCoefficients shape: {lr_multi.coef_.shape}")  # (3 classes, 4 features)
```

---

## 2. Decision Trees

Decision Trees แบ่งข้อมูลด้วย if-then rules โดยใช้ impurity measures (Gini, Entropy)

```python
from sklearn.tree import DecisionTreeClassifier, export_text, plot_tree
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split, cross_val_score, GridSearchCV
from sklearn.metrics import classification_report
import matplotlib.pyplot as plt
import numpy as np

iris = load_iris()
X, y = iris.data, iris.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Decision Tree
dt = DecisionTreeClassifier(
    max_depth=3,
    min_samples_split=5,
    min_samples_leaf=3,
    criterion='gini',  # หรือ 'entropy'
    random_state=42
)
dt.fit(X_train, y_train)

print("Decision Tree Results:")
print(f"  Train Accuracy: {dt.score(X_train, y_train):.4f}")
print(f"  Test Accuracy: {dt.score(X_test, y_test):.4f}")
print(f"  Tree depth: {dt.get_depth()}")
print(f"  Leaves: {dt.get_n_leaves()}")

# แสดง rules
print("\nDecision Tree Rules:")
print(export_text(dt, feature_names=iris.feature_names))

# Plot tree
plt.figure(figsize=(15, 8))
plot_tree(dt, feature_names=iris.feature_names, 
          class_names=iris.target_names, filled=True, rounded=True)
plt.title('Decision Tree - Iris')
plt.tight_layout()
plt.savefig('decision_tree.png', dpi=100, bbox_inches='tight')
print("Tree saved!")

# Tune max_depth
depths = range(1, 20)
train_accs, test_accs = [], []
for d in depths:
    dt = DecisionTreeClassifier(max_depth=d, random_state=42)
    dt.fit(X_train, y_train)
    train_accs.append(dt.score(X_train, y_train))
    test_accs.append(dt.score(X_test, y_test))

best_depth = depths[np.argmax(test_accs)]
print(f"\nBest depth: {best_depth}, Best test accuracy: {max(test_accs):.4f}")
```

### 2.1 Cost Complexity Pruning

```python
from sklearn.tree import DecisionTreeClassifier
from sklearn.model_selection import cross_val_score
import matplotlib.pyplot as plt
import numpy as np

from sklearn.datasets import load_breast_cancer
bc = load_breast_cancer()
X, y = bc.data, bc.target
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Find optimal ccp_alpha
dt = DecisionTreeClassifier(random_state=42)
path = dt.cost_complexity_pruning_path(X_train, y_train)
ccp_alphas = path.ccp_alphas

cv_scores = []
for alpha in ccp_alphas:
    dt_pruned = DecisionTreeClassifier(ccp_alpha=alpha, random_state=42)
    scores = cross_val_score(dt_pruned, X_train, y_train, cv=5)
    cv_scores.append(scores.mean())

best_alpha = ccp_alphas[np.argmax(cv_scores)]
print(f"Best ccp_alpha: {best_alpha:.6f}")

# Pruned tree
dt_best = DecisionTreeClassifier(ccp_alpha=best_alpha, random_state=42)
dt_best.fit(X_train, y_train)
print(f"Test accuracy: {dt_best.score(X_test, y_test):.4f}")
print(f"Tree depth: {dt_best.get_depth()}, Leaves: {dt_best.get_n_leaves()}")
```

---

## 3. Random Forest Classifier

Random Forest คือ ensemble ของ Decision Trees โดยแต่ละ tree ใช้ random subset ของข้อมูลและ features

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.metrics import classification_report, roc_auc_score
import numpy as np
import pandas as pd

bc = load_breast_cancer()
X, y = bc.data, bc.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)

# Random Forest
rf = RandomForestClassifier(
    n_estimators=200,
    max_depth=None,
    min_samples_split=5,
    min_samples_leaf=2,
    max_features='sqrt',  # จำนวน features ที่ random เลือก
    bootstrap=True,
    oob_score=True,       # Out-of-bag score
    n_jobs=-1,
    random_state=42
)
rf.fit(X_train, y_train)

y_pred = rf.predict(X_test)
y_prob = rf.predict_proba(X_test)[:, 1]

print("Random Forest Results:")
print(f"  Train Accuracy: {rf.score(X_train, y_train):.4f}")
print(f"  Test Accuracy: {rf.score(X_test, y_test):.4f}")
print(f"  OOB Score: {rf.oob_score_:.4f}")
print(f"  ROC-AUC: {roc_auc_score(y_test, y_prob):.4f}")
print(f"\n{classification_report(y_test, y_pred, target_names=bc.target_names)}")

# Feature Importances
feature_imp = pd.Series(rf.feature_importances_, index=bc.feature_names).sort_values(ascending=False)
print(f"\nTop 10 Features:")
print(feature_imp.head(10))

# Effect of n_estimators
n_trees = [10, 50, 100, 200, 500]
oob_scores = []
for n in n_trees:
    rf_temp = RandomForestClassifier(n_estimators=n, oob_score=True, n_jobs=-1, random_state=42)
    rf_temp.fit(X_train, y_train)
    oob_scores.append(rf_temp.oob_score_)
    print(f"  n_estimators={n:4d}: OOB={rf_temp.oob_score_:.4f}")
```

---

## 4. Gradient Boosting Classifiers

### 4.1 Sklearn GradientBoosting

```python
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report, roc_auc_score
import numpy as np

bc = load_breast_cancer()
X, y = bc.data, bc.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

gb = GradientBoostingClassifier(
    n_estimators=300,
    max_depth=4,
    learning_rate=0.05,
    subsample=0.8,
    min_samples_leaf=20,
    random_state=42
)
gb.fit(X_train, y_train)

y_pred = gb.predict(X_test)
y_prob = gb.predict_proba(X_test)[:, 1]

print("Gradient Boosting:")
print(f"  Accuracy: {gb.score(X_test, y_test):.4f}")
print(f"  ROC-AUC: {roc_auc_score(y_test, y_prob):.4f}")
print(classification_report(y_test, y_pred))
```

### 4.2 XGBoost Classifier

```python
try:
    import xgboost as xgb
    from sklearn.datasets import load_breast_cancer
    from sklearn.model_selection import train_test_split, cross_val_score
    from sklearn.metrics import roc_auc_score, classification_report
    
    bc = load_breast_cancer()
    X, y = bc.data, bc.target
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
    
    xgb_clf = xgb.XGBClassifier(
        n_estimators=300,
        max_depth=5,
        learning_rate=0.05,
        subsample=0.8,
        colsample_bytree=0.8,
        scale_pos_weight=1,   # สำหรับ class imbalance
        use_label_encoder=False,
        eval_metric='logloss',
        random_state=42,
        n_jobs=-1
    )
    
    xgb_clf.fit(
        X_train, y_train,
        eval_set=[(X_test, y_test)],
        verbose=100
    )
    
    y_prob = xgb_clf.predict_proba(X_test)[:, 1]
    print(f"XGBoost ROC-AUC: {roc_auc_score(y_test, y_prob):.4f}")
    print(f"XGBoost Accuracy: {xgb_clf.score(X_test, y_test):.4f}")
    
except ImportError:
    print("Install XGBoost: pip install xgboost")
```

### 4.3 LightGBM Classifier

```python
try:
    import lightgbm as lgb
    from sklearn.datasets import load_breast_cancer
    from sklearn.model_selection import train_test_split
    from sklearn.metrics import roc_auc_score
    
    bc = load_breast_cancer()
    X, y = bc.data, bc.target
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
    
    lgb_clf = lgb.LGBMClassifier(
        n_estimators=300,
        max_depth=6,
        learning_rate=0.05,
        num_leaves=31,
        subsample=0.8,
        colsample_bytree=0.8,
        is_unbalance=False,
        random_state=42,
        n_jobs=-1
    )
    
    lgb_clf.fit(
        X_train, y_train,
        eval_set=[(X_test, y_test)]
    )
    
    y_prob = lgb_clf.predict_proba(X_test)[:, 1]
    print(f"LightGBM ROC-AUC: {roc_auc_score(y_test, y_prob):.4f}")
    print(f"LightGBM Accuracy: {lgb_clf.score(X_test, y_test):.4f}")
    
except ImportError:
    print("Install LightGBM: pip install lightgbm")
```

### 4.4 CatBoost Classifier

```python
try:
    from catboost import CatBoostClassifier
    from sklearn.datasets import load_breast_cancer
    from sklearn.model_selection import train_test_split
    from sklearn.metrics import roc_auc_score
    
    bc = load_breast_cancer()
    X, y = bc.data, bc.target
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
    
    # CatBoost จัดการ categorical features ได้โดยตรง
    cat_model = CatBoostClassifier(
        iterations=300,
        depth=6,
        learning_rate=0.05,
        random_seed=42,
        verbose=100
    )
    
    cat_model.fit(X_train, y_train, eval_set=(X_test, y_test))
    
    y_prob = cat_model.predict_proba(X_test)[:, 1]
    print(f"CatBoost ROC-AUC: {roc_auc_score(y_test, y_prob):.4f}")
    
except ImportError:
    print("Install CatBoost: pip install catboost")
```

---

## 5. Support Vector Machine (SVM)

SVM หา hyperplane ที่แบ่ง classes โดยมี maximum margin

```python
from sklearn.svm import SVC
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.datasets import load_iris, load_breast_cancer
from sklearn.model_selection import train_test_split, GridSearchCV, cross_val_score
from sklearn.metrics import classification_report, roc_auc_score
import numpy as np

bc = load_breast_cancer()
X, y = bc.data, bc.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# SVM ต้องการ scaling เสมอ
svm_pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('svm', SVC(kernel='rbf', C=1.0, gamma='scale', probability=True, random_state=42))
])

svm_pipeline.fit(X_train, y_train)
y_pred = svm_pipeline.predict(X_test)
y_prob = svm_pipeline.predict_proba(X_test)[:, 1]

print("SVM Results:")
print(f"  Accuracy: {svm_pipeline.score(X_test, y_test):.4f}")
print(f"  ROC-AUC: {roc_auc_score(y_test, y_prob):.4f}")
print(classification_report(y_test, y_pred))

# Kernel Comparison
kernels = ['linear', 'poly', 'rbf', 'sigmoid']
for kernel in kernels:
    svm = Pipeline([
        ('scaler', StandardScaler()),
        ('svm', SVC(kernel=kernel, probability=True))
    ])
    scores = cross_val_score(svm, X_train, y_train, cv=5, scoring='roc_auc')
    print(f"  SVM ({kernel}): ROC-AUC = {scores.mean():.4f}")

# Grid Search
param_grid = {
    'svm__C': [0.01, 0.1, 1, 10, 100],
    'svm__gamma': ['scale', 'auto', 0.001, 0.01, 0.1]
}

grid_search = GridSearchCV(
    Pipeline([('scaler', StandardScaler()), ('svm', SVC(kernel='rbf', probability=True))]),
    param_grid, cv=5, scoring='roc_auc', n_jobs=-1
)
grid_search.fit(X_train, y_train)
print(f"\nBest SVM params: {grid_search.best_params_}")
print(f"Best ROC-AUC: {grid_search.best_score_:.4f}")
```

---

## 6. K-Nearest Neighbors (KNN)

KNN จัดประเภทข้อมูลใหม่โดยดูจาก k เพื่อนบ้านที่ใกล้ที่สุด

```python
from sklearn.neighbors import KNeighborsClassifier
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split, cross_val_score
import matplotlib.pyplot as plt
import numpy as np

iris = load_iris()
X, y = iris.data, iris.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# KNN ต้องการ scaling เสมอ
knn_pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('knn', KNeighborsClassifier(n_neighbors=5))
])

knn_pipeline.fit(X_train, y_train)
print(f"KNN Accuracy: {knn_pipeline.score(X_test, y_test):.4f}")

# หา optimal k
k_values = range(1, 51)
cv_scores = []

for k in k_values:
    knn = Pipeline([
        ('scaler', StandardScaler()),
        ('knn', KNeighborsClassifier(n_neighbors=k))
    ])
    scores = cross_val_score(knn, X_train, y_train, cv=5, scoring='accuracy')
    cv_scores.append(scores.mean())

best_k = k_values[np.argmax(cv_scores)]
print(f"Best k: {best_k}, Best CV Accuracy: {max(cv_scores):.4f}")

plt.figure(figsize=(10, 6))
plt.plot(k_values, cv_scores, 'o-', color='blue')
plt.axvline(x=best_k, color='red', linestyle='--', label=f'Best k={best_k}')
plt.xlabel('Number of Neighbors (k)')
plt.ylabel('CV Accuracy')
plt.title('KNN - Effect of k')
plt.legend()
plt.grid(True)
plt.savefig('knn_k_effect.png', dpi=100)
print("Plot saved!")

# Distance metrics comparison
for metric in ['euclidean', 'manhattan', 'minkowski', 'cosine']:
    knn = Pipeline([
        ('scaler', StandardScaler()),
        ('knn', KNeighborsClassifier(n_neighbors=5, metric=metric))
    ])
    scores = cross_val_score(knn, X_train, y_train, cv=5)
    print(f"  KNN ({metric}): {scores.mean():.4f}")
```

---

## 7. Naive Bayes

Naive Bayes ใช้ Bayes' theorem โดยสมมติว่า features เป็น independent กัน

```python
from sklearn.naive_bayes import GaussianNB, MultinomialNB, BernoulliNB, ComplementNB
from sklearn.datasets import load_iris, load_digits
from sklearn.model_selection import train_test_split, cross_val_score
import numpy as np

# Gaussian NB - สำหรับ continuous features
iris = load_iris()
X, y = iris.data, iris.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

gnb = GaussianNB()
gnb.fit(X_train, y_train)
print(f"Gaussian NB Accuracy: {gnb.score(X_test, y_test):.4f}")

# ตัวอย่าง Text Classification (Spam Detection)
from sklearn.datasets import fetch_20newsgroups
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.pipeline import Pipeline
from sklearn.metrics import classification_report

# โหลดเฉพาะบางหมวด
categories = ['alt.atheism', 'soc.religion.christian', 'comp.graphics', 'sci.med']
newsgroups = fetch_20newsgroups(subset='train', categories=categories)

# Pipeline: TF-IDF + Naive Bayes
text_pipeline = Pipeline([
    ('tfidf', TfidfVectorizer(max_features=10000, stop_words='english')),
    ('clf', MultinomialNB(alpha=0.1))
])

text_pipeline.fit(newsgroups.data, newsgroups.target)

# Test
newsgroups_test = fetch_20newsgroups(subset='test', categories=categories)
y_pred = text_pipeline.predict(newsgroups_test.data)
print("\nMultinomial NB - Newsgroups:")
print(classification_report(newsgroups_test.target, y_pred, 
                              target_names=categories))

# Complement NB (ดีกว่า Multinomial สำหรับ imbalanced text)
comp_pipeline = Pipeline([
    ('tfidf', TfidfVectorizer(max_features=10000, stop_words='english')),
    ('clf', ComplementNB(alpha=0.1))
])
comp_pipeline.fit(newsgroups.data, newsgroups.target)
print(f"Complement NB Accuracy: {comp_pipeline.score(newsgroups_test.data, newsgroups_test.target):.4f}")
```

---

## 8. Neural Networks (MLP)

Multi-Layer Perceptron เป็น neural network แบบ feedforward

```python
from sklearn.neural_network import MLPClassifier
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split, GridSearchCV
from sklearn.metrics import classification_report, roc_auc_score
import numpy as np

bc = load_breast_cancer()
X, y = bc.data, bc.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# MLP Classifier
mlp_pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('mlp', MLPClassifier(
        hidden_layer_sizes=(128, 64, 32),  # 3 hidden layers
        activation='relu',
        solver='adam',
        alpha=0.001,          # L2 regularization
        batch_size=32,
        learning_rate='adaptive',
        max_iter=500,
        early_stopping=True,  # ป้องกัน overfitting
        validation_fraction=0.1,
        n_iter_no_change=20,
        random_state=42
    ))
])

mlp_pipeline.fit(X_train, y_train)

y_pred = mlp_pipeline.predict(X_test)
y_prob = mlp_pipeline.predict_proba(X_test)[:, 1]

print("MLP Results:")
print(f"  Accuracy: {mlp_pipeline.score(X_test, y_test):.4f}")
print(f"  ROC-AUC: {roc_auc_score(y_test, y_prob):.4f}")
print(f"  Iterations: {mlp_pipeline.named_steps['mlp'].n_iter_}")
print(classification_report(y_test, y_pred, target_names=bc.target_names))

# Compare architectures
architectures = [
    (64,),
    (128, 64),
    (256, 128, 64),
    (64, 64, 64),
]

for arch in architectures:
    mlp = Pipeline([
        ('scaler', StandardScaler()),
        ('mlp', MLPClassifier(hidden_layer_sizes=arch, max_iter=500, random_state=42))
    ])
    mlp.fit(X_train, y_train)
    acc = mlp.score(X_test, y_test)
    print(f"  MLP {arch}: {acc:.4f}")
```

---

## 9. Handling Imbalanced Datasets

### 9.1 Class Weights

```python
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report, roc_auc_score
import numpy as np

# สร้าง imbalanced dataset
X, y = make_classification(
    n_samples=1000, n_features=20, n_informative=10,
    weights=[0.95, 0.05],  # 95% negative, 5% positive
    random_state=42
)

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)

print(f"Class distribution (train):")
from collections import Counter
print(f"  {Counter(y_train)}")

# Without class_weight (biased toward majority)
lr_no_weight = LogisticRegression(random_state=42, max_iter=1000)
lr_no_weight.fit(X_train, y_train)
y_pred = lr_no_weight.predict(X_test)
print("\nWithout class_weight:")
print(classification_report(y_test, y_pred, target_names=['Negative', 'Positive']))

# With class_weight='balanced'
lr_balanced = LogisticRegression(class_weight='balanced', random_state=42, max_iter=1000)
lr_balanced.fit(X_train, y_train)
y_pred_bal = lr_balanced.predict(X_test)
print("\nWith class_weight='balanced':")
print(classification_report(y_test, y_pred_bal, target_names=['Negative', 'Positive']))

# Custom weights
custom_weights = {0: 1, 1: 20}  # ให้ความสำคัญ class 1 มากขึ้น
lr_custom = LogisticRegression(class_weight=custom_weights, random_state=42, max_iter=1000)
lr_custom.fit(X_train, y_train)
y_pred_cust = lr_custom.predict(X_test)
print("\nWith custom class_weight {0:1, 1:20}:")
print(classification_report(y_test, y_pred_cust, target_names=['Negative', 'Positive']))
```

### 9.2 SMOTE (Synthetic Minority Over-sampling Technique)

```python
try:
    from imblearn.over_sampling import SMOTE, ADASYN
    from imblearn.pipeline import Pipeline as ImbPipeline
    from imblearn.under_sampling import RandomUnderSampler
    from imblearn.combine import SMOTETomek
    from sklearn.linear_model import LogisticRegression
    from sklearn.preprocessing import StandardScaler
    from sklearn.model_selection import cross_val_score, StratifiedKFold
    from sklearn.metrics import classification_report
    from collections import Counter
    import numpy as np
    
    from sklearn.datasets import make_classification
    X, y = make_classification(
        n_samples=1000, n_features=20, weights=[0.95, 0.05], random_state=42
    )
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)
    
    print(f"Before SMOTE: {Counter(y_train)}")
    
    # SMOTE
    smote = SMOTE(random_state=42)
    X_resampled, y_resampled = smote.fit_resample(X_train, y_train)
    print(f"After SMOTE: {Counter(y_resampled)}")
    
    # Pipeline with SMOTE
    smote_pipeline = ImbPipeline([
        ('scaler', StandardScaler()),
        ('smote', SMOTE(random_state=42)),
        ('clf', LogisticRegression(max_iter=1000, random_state=42))
    ])
    
    # Note: ใช้ imblearn Pipeline ซึ่งรองรับ sampler
    cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
    scores = cross_val_score(smote_pipeline, X_train, y_train, cv=cv, scoring='roc_auc')
    print(f"\nSMOTE Pipeline ROC-AUC: {scores.mean():.4f}")
    
    # Compare methods
    methods = {
        'SMOTE': SMOTE(random_state=42),
        'ADASYN': ADASYN(random_state=42),
        'SMOTETomek': SMOTETomek(random_state=42)
    }
    
    from sklearn.preprocessing import StandardScaler
    scaler = StandardScaler()
    X_train_s = scaler.fit_transform(X_train)
    X_test_s = scaler.transform(X_test)
    
    for name, sampler in methods.items():
        try:
            X_res, y_res = sampler.fit_resample(X_train_s, y_train)
            lr = LogisticRegression(max_iter=1000, random_state=42)
            lr.fit(X_res, y_res)
            from sklearn.metrics import roc_auc_score
            y_prob = lr.predict_proba(X_test_s)[:, 1]
            auc = roc_auc_score(y_test, y_prob)
            print(f"  {name}: ROC-AUC={auc:.4f}, Resampled={Counter(y_res)}")
        except Exception as e:
            print(f"  {name}: Error - {e}")

except ImportError:
    print("Install imbalanced-learn: pip install imbalanced-learn")
    print("Using class_weight='balanced' as alternative...")
    
    from sklearn.ensemble import RandomForestClassifier
    rf = RandomForestClassifier(n_estimators=100, class_weight='balanced', random_state=42)
    rf.fit(X_train, y_train)
    print(f"RF with balanced weights: {rf.score(X_test, y_test):.4f}")
```

---

## 10. Multi-class Classification

```python
from sklearn.datasets import load_digits
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.metrics import classification_report, confusion_matrix
from sklearn.multiclass import OneVsRestClassifier, OneVsOneClassifier
from sklearn.svm import SVC
import numpy as np
import matplotlib.pyplot as plt

# Digits Dataset - 10 classes
digits = load_digits()
X, y = digits.data, digits.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Random Forest - รองรับ multi-class โดยตรง
rf = RandomForestClassifier(n_estimators=200, random_state=42, n_jobs=-1)
rf.fit(X_train, y_train)
y_pred = rf.predict(X_test)

print("Random Forest - Digits:")
print(f"  Accuracy: {rf.score(X_test, y_test):.4f}")
print(classification_report(y_test, y_pred))

# One-vs-Rest (OvR) Strategy
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline

ovr_svm = Pipeline([
    ('scaler', StandardScaler()),
    ('clf', OneVsRestClassifier(SVC(kernel='rbf', probability=True), n_jobs=-1))
])
ovr_svm.fit(X_train, y_train)
print(f"\nOvR SVM Accuracy: {ovr_svm.score(X_test, y_test):.4f}")

# One-vs-One (OvO) Strategy  
ovo_svm = Pipeline([
    ('scaler', StandardScaler()),
    ('clf', OneVsOneClassifier(SVC(kernel='rbf'), n_jobs=-1))
])
ovo_svm.fit(X_train, y_train)
print(f"OvO SVM Accuracy: {ovo_svm.score(X_test, y_test):.4f}")

# Confusion Matrix
cm = confusion_matrix(y_test, rf.predict(X_test))
plt.figure(figsize=(10, 8))
plt.imshow(cm, cmap='Blues')
plt.colorbar()
plt.xticks(range(10))
plt.yticks(range(10))
plt.xlabel('Predicted')
plt.ylabel('Actual')
plt.title('Random Forest - Digits Confusion Matrix')
for i in range(10):
    for j in range(10):
        plt.text(j, i, cm[i, j], ha='center', va='center', 
                 color='white' if cm[i, j] > cm.max()/2 else 'black')
plt.tight_layout()
plt.savefig('multiclass_confusion_matrix.png', dpi=100)
print("Confusion matrix saved!")
```

---

## 11. Spam Detection Project

```python
"""
โปรแกรมจริง: Spam Detection
ใช้ Text Classification
"""
import numpy as np
import pandas as pd
from sklearn.feature_extraction.text import TfidfVectorizer, CountVectorizer
from sklearn.naive_bayes import MultinomialNB, ComplementNB
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.pipeline import Pipeline
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.metrics import classification_report, roc_auc_score, confusion_matrix
import re
import warnings
warnings.filterwarnings('ignore')

# สร้าง Synthetic Spam Dataset
def create_spam_dataset(n_samples=2000):
    """สร้าง synthetic email dataset"""
    np.random.seed(42)
    
    spam_words = ['free', 'win', 'winner', 'cash', 'prize', 'offer', 'click', 
                   'buy now', 'limited time', 'congratulations', 'discount', 
                   'money', 'urgent', 'act now', 'guaranteed', 'million']
    
    ham_words = ['meeting', 'project', 'report', 'team', 'please', 'attached',
                  'schedule', 'update', 'review', 'discussion', 'deadline',
                  'presentation', 'document', 'analysis', 'proposal']
    
    emails = []
    labels = []
    
    for i in range(n_samples):
        is_spam = np.random.random() < 0.4  # 40% spam
        
        if is_spam:
            n_words = np.random.randint(20, 80)
            spam_count = np.random.randint(3, 8)
            text_words = np.random.choice(spam_words, spam_count).tolist()
            filler = ['the', 'a', 'is', 'to', 'and', 'for', 'you', 'your', 'this']
            text_words += np.random.choice(filler, n_words - spam_count).tolist()
            np.random.shuffle(text_words)
            emails.append(' '.join(text_words))
            labels.append(1)  # spam
        else:
            n_words = np.random.randint(30, 100)
            ham_count = np.random.randint(3, 8)
            text_words = np.random.choice(ham_words, ham_count).tolist()
            filler = ['the', 'a', 'please', 'we', 'our', 'i', 'have', 'will', 'be']
            text_words += np.random.choice(filler, n_words - ham_count).tolist()
            np.random.shuffle(text_words)
            emails.append(' '.join(text_words))
            labels.append(0)  # ham
    
    return emails, labels

emails, labels = create_spam_dataset(2000)
print(f"Dataset: {len(emails)} emails")
print(f"Spam: {sum(labels)}, Ham: {len(labels) - sum(labels)}")

# Split
X_train, X_test, y_train, y_test = train_test_split(
    emails, labels, test_size=0.2, random_state=42, stratify=labels
)

# Text Preprocessing Function
def preprocess_text(text):
    text = text.lower()
    text = re.sub(r'[^a-z\s]', '', text)
    return text

X_train_clean = [preprocess_text(t) for t in X_train]
X_test_clean = [preprocess_text(t) for t in X_test]

# Models
models = {
    'Multinomial NB (Count)': Pipeline([
        ('vectorizer', CountVectorizer(max_features=5000, stop_words='english')),
        ('clf', MultinomialNB(alpha=0.1))
    ]),
    'Complement NB (TF-IDF)': Pipeline([
        ('vectorizer', TfidfVectorizer(max_features=5000, stop_words='english')),
        ('clf', ComplementNB(alpha=0.1))
    ]),
    'Logistic Reg (TF-IDF)': Pipeline([
        ('vectorizer', TfidfVectorizer(max_features=5000, ngram_range=(1, 2), stop_words='english')),
        ('clf', LogisticRegression(C=1.0, max_iter=1000, random_state=42))
    ]),
    'Random Forest (TF-IDF)': Pipeline([
        ('vectorizer', TfidfVectorizer(max_features=3000, stop_words='english')),
        ('clf', RandomForestClassifier(n_estimators=100, random_state=42, n_jobs=-1))
    ])
}

print("\n=== Model Comparison ===")
for name, model in models.items():
    scores = cross_val_score(model, X_train_clean, y_train, cv=5, scoring='roc_auc')
    print(f"{name:35s}: ROC-AUC = {scores.mean():.4f} (+/- {scores.std()*2:.4f})")

# Best Model: Logistic Regression
best_model = models['Logistic Reg (TF-IDF)']
best_model.fit(X_train_clean, y_train)
y_pred = best_model.predict(X_test_clean)
y_prob = best_model.predict_proba(X_test_clean)[:, 1]

print(f"\n=== Best Model Results ===")
print(f"ROC-AUC: {roc_auc_score(y_test, y_prob):.4f}")
print(classification_report(y_test, y_pred, target_names=['Ham', 'Spam']))

# Test with custom examples
test_emails = [
    "win free money now click here limited time offer",
    "please review the attached report for tomorrow meeting",
    "congratulations you have been selected for cash prize",
    "team meeting scheduled for monday project update"
]

for email in test_emails:
    clean = preprocess_text(email)
    pred = best_model.predict([clean])[0]
    prob = best_model.predict_proba([clean])[0][1]
    print(f"\n  Email: {email[:50]}...")
    print(f"  Prediction: {'SPAM' if pred == 1 else 'HAM'} (spam probability: {prob:.2%})")
```

---

## 12. Sentiment Analysis Project

```python
"""
โปรแกรมจริง: Sentiment Analysis
"""
import numpy as np
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.svm import LinearSVC
from sklearn.pipeline import Pipeline
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.metrics import classification_report, accuracy_score
import re

# สร้าง Synthetic Review Dataset
def create_sentiment_dataset(n=1000):
    np.random.seed(42)
    
    positive_words = ['excellent', 'amazing', 'great', 'wonderful', 'fantastic',
                       'love', 'best', 'perfect', 'highly recommend', 'outstanding',
                       'impressive', 'superb', 'brilliant', 'awesome', 'incredible']
    
    negative_words = ['terrible', 'awful', 'horrible', 'worst', 'hate',
                       'disappointed', 'poor', 'bad', 'waste', 'useless',
                       'broken', 'defective', 'failure', 'regret', 'never again']
    
    neutral_filler = ['product', 'service', 'quality', 'price', 'delivery',
                       'the', 'a', 'is', 'was', 'this', 'very', 'quite']
    
    reviews = []
    sentiments = []
    
    for _ in range(n):
        sentiment = np.random.choice(['positive', 'negative', 'neutral'], 
                                       p=[0.4, 0.4, 0.2])
        
        if sentiment == 'positive':
            words = np.random.choice(positive_words, np.random.randint(2, 5)).tolist()
            words += np.random.choice(neutral_filler, np.random.randint(5, 10)).tolist()
            label = 1
        elif sentiment == 'negative':
            words = np.random.choice(negative_words, np.random.randint(2, 5)).tolist()
            words += np.random.choice(neutral_filler, np.random.randint(5, 10)).tolist()
            label = 0
        else:  # neutral -> classify as negative for binary
            words = np.random.choice(neutral_filler, np.random.randint(8, 15)).tolist()
            label = np.random.choice([0, 1])
        
        np.random.shuffle(words)
        reviews.append(' '.join(words))
        sentiments.append(label)
    
    return reviews, sentiments

reviews, sentiments = create_sentiment_dataset(2000)
X_train, X_test, y_train, y_test = train_test_split(
    reviews, sentiments, test_size=0.2, random_state=42
)

# Models comparison
models = {
    'Logistic Regression': Pipeline([
        ('tfidf', TfidfVectorizer(max_features=5000, ngram_range=(1, 2))),
        ('clf', LogisticRegression(C=1.0, max_iter=1000))
    ]),
    'Linear SVM': Pipeline([
        ('tfidf', TfidfVectorizer(max_features=5000, ngram_range=(1, 2))),
        ('clf', LinearSVC(C=1.0, max_iter=5000))
    ])
}

print("=== Sentiment Analysis ===")
for name, model in models.items():
    model.fit(X_train, y_train)
    acc = accuracy_score(y_test, model.predict(X_test))
    cv_scores = cross_val_score(model, X_train, y_train, cv=5, scoring='accuracy')
    print(f"{name:20s}: Test={acc:.4f}, CV={cv_scores.mean():.4f}")

# Feature importance (most predictive words)
lr_model = models['Logistic Regression']
vectorizer = lr_model.named_steps['tfidf']
clf = lr_model.named_steps['clf']

feature_names = vectorizer.get_feature_names_out()
coefs = clf.coef_[0]

# Top positive words
top_positive = [(feature_names[i], coefs[i]) for i in np.argsort(coefs)[-15:]]
top_negative = [(feature_names[i], coefs[i]) for i in np.argsort(coefs)[:15]]

print("\nTop Positive Words:")
for word, coef in sorted(top_positive, key=lambda x: x[1], reverse=True):
    print(f"  {word:20s}: {coef:.4f}")

print("\nTop Negative Words:")
for word, coef in sorted(top_negative, key=lambda x: x[1]):
    print(f"  {word:20s}: {coef:.4f}")
```

---

## 13. Image Classification (Conceptual)

```python
"""
โปรแกรมจริง: Image Classification ด้วย Sklearn
ใช้ Digits Dataset เป็นตัวอย่าง
"""
from sklearn.datasets import load_digits
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.svm import SVC
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.metrics import classification_report, confusion_matrix
from sklearn.decomposition import PCA
import numpy as np
import matplotlib.pyplot as plt

digits = load_digits()
X, y = digits.data, digits.target

print(f"Digits Dataset:")
print(f"  Shape: {X.shape}")
print(f"  Classes: {digits.target_names}")
print(f"  Image size: 8x8 pixels")

# Visualize samples
fig, axes = plt.subplots(2, 5, figsize=(12, 5))
for i, ax in enumerate(axes.flatten()):
    ax.imshow(digits.images[i], cmap='gray')
    ax.set_title(f'Label: {y[i]}')
    ax.axis('off')
plt.suptitle('Sample Digits')
plt.tight_layout()
plt.savefig('digits_samples.png', dpi=100)
print("Samples saved!")

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# PCA for visualization and speed
pca = PCA(n_components=50)
X_train_pca = pca.fit_transform(X_train)
X_test_pca = pca.transform(X_test)
print(f"\nPCA: {X_train.shape[1]} -> {X_train_pca.shape[1]} features")
print(f"Explained variance: {pca.explained_variance_ratio_.sum():.4f}")

# Models
models = {
    'SVM (RBF)': Pipeline([('scaler', StandardScaler()), ('clf', SVC(kernel='rbf', C=10))]),
    'Random Forest': RandomForestClassifier(n_estimators=200, random_state=42, n_jobs=-1),
    'SVM + PCA': Pipeline([('scaler', StandardScaler()), ('pca', PCA(n_components=50)),
                             ('clf', SVC(kernel='rbf', C=10))])
}

print("\n=== Digit Classification Results ===")
for name, model in models.items():
    scores = cross_val_score(model, X_train, y_train, cv=5, scoring='accuracy', n_jobs=-1)
    print(f"{name:20s}: {scores.mean():.4f} (+/- {scores.std()*2:.4f})")

# Best: SVM
svm = Pipeline([('scaler', StandardScaler()), ('clf', SVC(kernel='rbf', C=10))])
svm.fit(X_train, y_train)
y_pred = svm.predict(X_test)
print(f"\nSVM Test Accuracy: {svm.score(X_test, y_test):.4f}")
print(classification_report(y_test, y_pred))
```

---

## 14. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Logistic Regression Tuning

```python
# เฉลย
from sklearn.linear_model import LogisticRegression
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.model_selection import GridSearchCV, cross_val_score
from sklearn.datasets import load_breast_cancer
from sklearn.metrics import roc_auc_score, classification_report
from sklearn.model_selection import train_test_split

bc = load_breast_cancer()
X, y = bc.data, bc.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

param_grid = {
    'lr__C': [0.001, 0.01, 0.1, 1, 10, 100],
    'lr__penalty': ['l1', 'l2'],
    'lr__solver': ['liblinear']
}

pipe = Pipeline([('scaler', StandardScaler()), ('lr', LogisticRegression(max_iter=1000))])
grid = GridSearchCV(pipe, param_grid, cv=5, scoring='roc_auc', n_jobs=-1)
grid.fit(X_train, y_train)

print(f"Best params: {grid.best_params_}")
print(f"Best CV AUC: {grid.best_score_:.4f}")
best = grid.best_estimator_
best.fit(X_train, y_train)
y_prob = best.predict_proba(X_test)[:, 1]
print(f"Test AUC: {roc_auc_score(y_test, y_prob):.4f}")
print(classification_report(y_test, best.predict(X_test), target_names=bc.target_names))
```

### แบบฝึกหัดที่ 2: Random Forest vs GradientBoosting

```python
# เฉลย
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.model_selection import cross_val_score, RandomizedSearchCV
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.metrics import roc_auc_score
from scipy.stats import randint, uniform
import numpy as np

X, y = make_classification(n_samples=2000, n_features=20, n_informative=12,
                             weights=[0.7, 0.3], random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Tune RF
rf_params = {'n_estimators': randint(100, 500), 'max_depth': [None]+list(range(5, 25, 5)),
              'min_samples_leaf': randint(1, 10)}
rf_search = RandomizedSearchCV(RandomForestClassifier(random_state=42), rf_params,
                                 n_iter=30, cv=5, scoring='roc_auc', random_state=42, n_jobs=-1)
rf_search.fit(X_train, y_train)

# Tune GB
gb_params = {'n_estimators': randint(100, 500), 'max_depth': randint(3, 8),
              'learning_rate': uniform(0.01, 0.2)}
gb_search = RandomizedSearchCV(GradientBoostingClassifier(random_state=42), gb_params,
                                 n_iter=30, cv=5, scoring='roc_auc', random_state=42, n_jobs=-1)
gb_search.fit(X_train, y_train)

for name, search in [('Random Forest', rf_search), ('Gradient Boosting', gb_search)]:
    y_prob = search.best_estimator_.predict_proba(X_test)[:, 1]
    auc = roc_auc_score(y_test, y_prob)
    print(f"{name}: CV AUC={search.best_score_:.4f}, Test AUC={auc:.4f}")
    print(f"  Best params: {search.best_params_}")
```

### แบบฝึกหัดที่ 3: Imbalanced Classification

```python
# เฉลย
from sklearn.datasets import make_classification
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split, StratifiedKFold, cross_val_score
from sklearn.metrics import classification_report, roc_auc_score, f1_score
from sklearn.preprocessing import StandardScaler
from collections import Counter
import numpy as np

X, y = make_classification(n_samples=5000, n_features=20, n_informative=10,
                             weights=[0.97, 0.03], random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)

print(f"Train distribution: {Counter(y_train)}")

# Strategies comparison
strategies = {
    'No adjustment': LogisticRegression(max_iter=1000),
    'Balanced LR': LogisticRegression(class_weight='balanced', max_iter=1000),
    'RF balanced': RandomForestClassifier(n_estimators=200, class_weight='balanced', random_state=42),
    'GB': GradientBoostingClassifier(n_estimators=200, random_state=42)
}

from sklearn.pipeline import Pipeline
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

print("\nStrategy Comparison:")
for name, model in strategies.items():
    model.fit(X_train_s, y_train)
    y_pred = model.predict(X_test_s)
    y_prob = model.predict_proba(X_test_s)[:, 1]
    f1 = f1_score(y_test, y_pred)
    auc = roc_auc_score(y_test, y_prob)
    print(f"  {name:20s}: F1={f1:.4f}, AUC={auc:.4f}")
```

### แบบฝึกหัดที่ 4: Text Classification Pipeline

```python
# เฉลย
from sklearn.datasets import fetch_20newsgroups
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.naive_bayes import ComplementNB
from sklearn.linear_model import LogisticRegression
from sklearn.svm import LinearSVC
from sklearn.model_selection import cross_val_score
from sklearn.metrics import classification_report
import numpy as np

# Load newsgroups - 4 categories
categories = ['sci.space', 'sci.med', 'comp.graphics', 'talk.politics.misc']
train_data = fetch_20newsgroups(subset='train', categories=categories, remove=('headers', 'footers', 'quotes'))
test_data = fetch_20newsgroups(subset='test', categories=categories, remove=('headers', 'footers', 'quotes'))

print(f"Train: {len(train_data.data)}, Test: {len(test_data.data)}")

models = {
    'Complement NB': Pipeline([
        ('tfidf', TfidfVectorizer(max_features=20000, ngram_range=(1,2), stop_words='english')),
        ('clf', ComplementNB(alpha=0.1))
    ]),
    'Logistic Reg': Pipeline([
        ('tfidf', TfidfVectorizer(max_features=20000, ngram_range=(1,2), stop_words='english')),
        ('clf', LogisticRegression(C=1.0, max_iter=1000, random_state=42))
    ]),
    'Linear SVM': Pipeline([
        ('tfidf', TfidfVectorizer(max_features=20000, ngram_range=(1,2), stop_words='english')),
        ('clf', LinearSVC(C=1.0, max_iter=5000))
    ])
}

for name, model in models.items():
    model.fit(train_data.data, train_data.target)
    acc = model.score(test_data.data, test_data.target)
    print(f"{name:15s}: {acc:.4f}")

print("\nBest model - Linear SVM:")
best = models['Linear SVM']
print(classification_report(test_data.target, best.predict(test_data.data),
                              target_names=categories))
```

### แบบฝึกหัดที่ 5: SVM Kernel Comparison

```python
# เฉลย
from sklearn.svm import SVC
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.model_selection import GridSearchCV, cross_val_score
from sklearn.datasets import make_moons, make_circles, make_classification
from sklearn.metrics import accuracy_score
import numpy as np

datasets = {
    'Moons': make_moons(n_samples=500, noise=0.15, random_state=42),
    'Circles': make_circles(n_samples=500, noise=0.1, factor=0.5, random_state=42),
    'Linear': make_classification(n_samples=500, n_features=20, random_state=42)
}

kernels = ['linear', 'poly', 'rbf']

from sklearn.model_selection import train_test_split
print("SVM Kernel Comparison:")
for ds_name, (X, y) in datasets.items():
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
    print(f"\n{ds_name}:")
    for kernel in kernels:
        pipe = Pipeline([('scaler', StandardScaler()), ('svm', SVC(kernel=kernel))])
        scores = cross_val_score(pipe, X_train, y_train, cv=5, scoring='accuracy')
        print(f"  {kernel:8s}: {scores.mean():.4f} (+/- {scores.std()*2:.4f})")
```

### แบบฝึกหัดที่ 6: KNN with Feature Engineering

```python
# เฉลย
from sklearn.neighbors import KNeighborsClassifier
from sklearn.preprocessing import StandardScaler, PolynomialFeatures
from sklearn.pipeline import Pipeline
from sklearn.model_selection import cross_val_score, GridSearchCV
from sklearn.datasets import load_wine
import numpy as np

wine = load_wine()
X, y = wine.data, wine.target

# Basic KNN
knn_basic = Pipeline([
    ('scaler', StandardScaler()),
    ('knn', KNeighborsClassifier())
])
basic_scores = cross_val_score(knn_basic, X, y, cv=5, scoring='accuracy')
print(f"Basic KNN: {basic_scores.mean():.4f}")

# KNN with Polynomial Features
knn_poly = Pipeline([
    ('scaler', StandardScaler()),
    ('poly', PolynomialFeatures(degree=2, interaction_only=True, include_bias=False)),
    ('scaler2', StandardScaler()),
    ('knn', KNeighborsClassifier())
])
poly_scores = cross_val_score(knn_poly, X, y, cv=5, scoring='accuracy')
print(f"KNN + Poly Features: {poly_scores.mean():.4f}")

# Grid Search for k
param_grid = {
    'knn__n_neighbors': range(1, 31),
    'knn__weights': ['uniform', 'distance'],
    'knn__p': [1, 2]  # 1=manhattan, 2=euclidean
}
grid = GridSearchCV(Pipeline([('scaler', StandardScaler()), ('knn', KNeighborsClassifier())]),
                    param_grid, cv=5, scoring='accuracy', n_jobs=-1)
grid.fit(X, y)
print(f"\nBest KNN params: {grid.best_params_}")
print(f"Best CV accuracy: {grid.best_score_:.4f}")
```

### แบบฝึกหัดที่ 7: Multi-class Classification with Probability Calibration

```python
# เฉลย
from sklearn.datasets import load_digits
from sklearn.ensemble import RandomForestClassifier
from sklearn.calibration import CalibratedClassifierCV
from sklearn.model_selection import train_test_split
from sklearn.metrics import log_loss, accuracy_score
import numpy as np

digits = load_digits()
X, y = digits.data, digits.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Uncalibrated
rf = RandomForestClassifier(n_estimators=200, random_state=42, n_jobs=-1)
rf.fit(X_train, y_train)
y_prob_uncal = rf.predict_proba(X_test)
print(f"Uncalibrated - Accuracy: {accuracy_score(y_test, rf.predict(X_test)):.4f}, Log-loss: {log_loss(y_test, y_prob_uncal):.4f}")

# Calibrated (Platt scaling)
rf_cal_sigmoid = CalibratedClassifierCV(rf, cv=5, method='sigmoid')
rf_cal_sigmoid.fit(X_train, y_train)
y_prob_sig = rf_cal_sigmoid.predict_proba(X_test)
print(f"Sigmoid calibrated - Log-loss: {log_loss(y_test, y_prob_sig):.4f}")

# Calibrated (Isotonic regression)
rf_cal_iso = CalibratedClassifierCV(rf, cv=5, method='isotonic')
rf_cal_iso.fit(X_train, y_train)
y_prob_iso = rf_cal_iso.predict_proba(X_test)
print(f"Isotonic calibrated - Log-loss: {log_loss(y_test, y_prob_iso):.4f}")
```

### แบบฝึกหัดที่ 8: Complete Classification Project (Credit Risk)

```python
# เฉลย
import numpy as np
import pandas as pd
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split, StratifiedKFold, cross_val_score
from sklearn.preprocessing import StandardScaler, LabelEncoder
from sklearn.pipeline import Pipeline
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report, roc_auc_score, confusion_matrix
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import randint
import warnings
warnings.filterwarnings('ignore')

# สร้าง Credit Risk Dataset
np.random.seed(42)
n = 2000

# Imbalanced: 20% default
X, y = make_classification(
    n_samples=n, n_features=15, n_informative=10,
    weights=[0.80, 0.20], random_state=42
)

feature_names = ['age', 'income', 'credit_score', 'debt_ratio', 'loan_amount',
                  'employment_length', 'num_accounts', 'late_payments', 
                  'credit_utilization', 'home_ownership', 'loan_purpose',
                  'annual_income', 'dti', 'pub_rec', 'revol_util']
df = pd.DataFrame(X, columns=feature_names)
df['default'] = y

print(f"Default rate: {y.mean()*100:.1f}%")

X_data = df.drop('default', axis=1)
y_data = df['default']

X_train, X_test, y_train, y_test = train_test_split(
    X_data, y_data, test_size=0.2, random_state=42, stratify=y_data
)

models = {
    'LR (Balanced)': Pipeline([
        ('scaler', StandardScaler()),
        ('clf', LogisticRegression(class_weight='balanced', max_iter=1000, C=0.1))
    ]),
    'RF (Balanced)': RandomForestClassifier(
        n_estimators=200, class_weight='balanced', random_state=42, n_jobs=-1
    ),
    'GB': GradientBoostingClassifier(n_estimators=200, max_depth=4, random_state=42)
}

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)

print("\nCredit Risk Model Comparison:")
for name, model in models.items():
    scores = cross_val_score(model, X_train, y_train, cv=cv, scoring='roc_auc', n_jobs=-1)
    print(f"  {name:20s}: ROC-AUC = {scores.mean():.4f} (+/- {scores.std()*2:.4f})")

# Final: GB
gb = models['GB']
gb.fit(X_train, y_train)
y_pred = gb.predict(X_test)
y_prob = gb.predict_proba(X_test)[:, 1]

print(f"\nFinal Test Results (Gradient Boosting):")
print(f"ROC-AUC: {roc_auc_score(y_test, y_prob):.4f}")
print(classification_report(y_test, y_pred, target_names=['Good', 'Default']))

# Feature Importance
feat_imp = pd.Series(gb.feature_importances_, index=feature_names).sort_values(ascending=False)
print(f"\nTop Features:")
print(feat_imp.head(5))

# Business Decision: set threshold
thresholds = [0.3, 0.4, 0.5, 0.6, 0.7]
print(f"\nThreshold Analysis:")
for thresh in thresholds:
    y_pred_thresh = (y_prob >= thresh).astype(int)
    from sklearn.metrics import precision_score, recall_score
    prec = precision_score(y_test, y_pred_thresh)
    rec = recall_score(y_test, y_pred_thresh)
    print(f"  Threshold={thresh:.1f}: Precision={prec:.4f}, Recall={rec:.4f}")
```

---

## สรุปบทที่ 78

| Algorithm | เหมาะกับ | ข้อดี | ข้อเสีย |
|-----------|---------|-------|---------|
| Logistic Regression | Binary/Multi, Linear | ตีความง่าย, เร็ว, probability | ต้องการ scaling |
| Decision Tree | ทุกประเภท | ตีความง่าย, ไม่ต้องการ scaling | Overfitting |
| Random Forest | ทุกประเภท | Robust, OOB validation | ตีความยาก |
| Gradient Boosting | ทุกประเภท | Accuracy สูง | Overfitting, ช้า |
| SVM | Small-medium | Kernel trick, Margin | ช้ากับ large data |
| KNN | Simple patterns | ไม่ต้องการ training | Memory-intensive, ช้า |
| Naive Bayes | Text, high-dim | เร็วมาก, text classification | Independence assumption |
| MLP | Complex patterns | Non-linear, flexible | Hyperparameter tuning |

**Key Takeaways:**
1. Scale ข้อมูลเสมอสำหรับ LR, SVM, KNN, MLP
2. Tree-based models ไม่ต้องการ scaling
3. ใช้ `class_weight='balanced'` หรือ SMOTE สำหรับ imbalanced data
4. F1-score สำหรับ imbalanced, ROC-AUC สำหรับ probability ranking
5. Gradient Boosting (XGBoost/LightGBM) มักชนะ competition
6. Naive Bayes ดีมากสำหรับ text classification
7. ใช้ Calibration เมื่อต้องการ well-calibrated probabilities

**Part 79** จะเรียนเรื่อง Clustering & Dimensionality Reduction
