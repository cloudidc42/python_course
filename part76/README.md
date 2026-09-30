# Part 76: Scikit-learn - Machine Learning Fundamentals

## บทนำ

Machine Learning (การเรียนรู้ของเครื่อง) คือสาขาหนึ่งของ Artificial Intelligence ที่ให้คอมพิวเตอร์สามารถเรียนรู้จากข้อมูลได้โดยไม่ต้องเขียนโปรแกรมให้ทำงานโดยตรง Scikit-learn เป็น library Python ที่ครอบคลุมที่สุดสำหรับ Machine Learning แบบ Classical ใช้งานง่าย มีประสิทธิภาพสูง และมีเอกสารที่ครบถ้วน

ในบทนี้เราจะเรียนรู้:
- ประเภทของ Machine Learning
- Scikit-learn ecosystem และการติดตั้ง
- การ preprocessing ข้อมูล
- การแบ่งข้อมูล train/test
- Cross-validation
- Pipeline
- การประเมินผลโมเดล
- Feature selection
- Hyperparameter tuning

---

## 1. ประเภทของ Machine Learning

### 1.1 Supervised Learning (การเรียนรู้แบบมีผู้สอน)

Supervised Learning คือการเรียนรู้จากข้อมูลที่มี label (คำตอบ) แล้ว โมเดลจะเรียนรู้ความสัมพันธ์ระหว่าง input (X) และ output (y)

**ประเภทย่อย:**
- **Regression**: ทำนายค่าตัวเลขต่อเนื่อง เช่น ราคาบ้าน, อุณหภูมิ
- **Classification**: จัดประเภทข้อมูล เช่น spam/not spam, โรค/ไม่เป็นโรค

```python
# ตัวอย่าง Supervised Learning - Classification
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.neighbors import KNeighborsClassifier
from sklearn.metrics import accuracy_score

# โหลด dataset
iris = load_iris()
X, y = iris.data, iris.target

# แบ่งข้อมูล
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# สร้างโมเดล
model = KNeighborsClassifier(n_neighbors=3)
model.fit(X_train, y_train)

# ทำนาย
y_pred = model.predict(X_test)
print(f"Accuracy: {accuracy_score(y_test, y_pred):.4f}")
```

### 1.2 Unsupervised Learning (การเรียนรู้แบบไม่มีผู้สอน)

Unsupervised Learning คือการเรียนรู้จากข้อมูลที่ไม่มี label โมเดลจะค้นหาโครงสร้างหรือรูปแบบที่ซ่อนอยู่ในข้อมูล

**ประเภทย่อย:**
- **Clustering**: จัดกลุ่มข้อมูลที่คล้ายกัน
- **Dimensionality Reduction**: ลดมิติข้อมูล
- **Association Rules**: ค้นหาความสัมพันธ์

```python
# ตัวอย่าง Unsupervised Learning - Clustering
from sklearn.datasets import make_blobs
from sklearn.cluster import KMeans
import matplotlib.pyplot as plt
import numpy as np

# สร้างข้อมูลตัวอย่าง
X, y_true = make_blobs(n_samples=300, centers=4, cluster_std=0.60, random_state=42)

# K-Means Clustering
kmeans = KMeans(n_clusters=4, random_state=42)
y_pred = kmeans.fit_predict(X)

print(f"Cluster centers:\n{kmeans.cluster_centers_}")
print(f"Inertia: {kmeans.inertia_:.2f}")
```

### 1.3 Reinforcement Learning (การเรียนรู้แบบเสริมแรง)

Reinforcement Learning คือการเรียนรู้จากการลองผิดลองถูก agent จะเรียนรู้ policy ที่ดีที่สุดจากการได้รับ reward และ penalty

```python
# Reinforcement Learning แบบ Simple Q-Learning
import numpy as np

# Simple Grid World Environment
class GridWorld:
    def __init__(self, size=4):
        self.size = size
        self.state = 0
        self.goal = size * size - 1
        
    def reset(self):
        self.state = 0
        return self.state
    
    def step(self, action):
        # Actions: 0=up, 1=down, 2=left, 3=right
        row, col = self.state // self.size, self.state % self.size
        
        if action == 0 and row > 0: row -= 1
        elif action == 1 and row < self.size-1: row += 1
        elif action == 2 and col > 0: col -= 1
        elif action == 3 and col < self.size-1: col += 1
        
        self.state = row * self.size + col
        reward = 1 if self.state == self.goal else -0.01
        done = self.state == self.goal
        return self.state, reward, done

# Q-Learning
env = GridWorld(size=4)
Q = np.zeros((16, 4))
alpha, gamma, epsilon = 0.1, 0.9, 0.1

for episode in range(1000):
    state = env.reset()
    done = False
    while not done:
        if np.random.random() < epsilon:
            action = np.random.randint(4)
        else:
            action = np.argmax(Q[state])
        
        next_state, reward, done = env.step(action)
        Q[state, action] += alpha * (reward + gamma * np.max(Q[next_state]) - Q[state, action])
        state = next_state

print("Q-Table trained successfully!")
print(f"Optimal policy example (state 0): {np.argmax(Q[0])}")
```

---

## 2. Scikit-learn Ecosystem

### 2.1 การติดตั้งและ Import

```python
# ติดตั้ง scikit-learn
# pip install scikit-learn

# ตรวจสอบ version
import sklearn
print(f"Scikit-learn version: {sklearn.__version__}")

# Import หลัก
from sklearn import datasets, preprocessing, model_selection
from sklearn import linear_model, tree, ensemble, svm, neighbors
from sklearn import metrics, pipeline, feature_selection
```

### 2.2 โครงสร้างหลักของ Scikit-learn

```python
# Scikit-learn API แบบ Consistent
# 1. สร้าง Estimator
# 2. fit() - เรียนรู้จากข้อมูล
# 3. predict() - ทำนาย
# 4. transform() - แปลงข้อมูล

from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import StandardScaler
import numpy as np

# ตัวอย่าง Basic Workflow
X = np.array([[1, 2], [3, 4], [5, 6], [7, 8]])
y = np.array([2, 4, 6, 8])

# Scaler - มีทั้ง fit() และ transform()
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
print(f"Scaled X:\n{X_scaled}")

# Model - มี fit() และ predict()
model = LinearRegression()
model.fit(X_scaled, y)
print(f"Coefficients: {model.coef_}")
print(f"Intercept: {model.intercept_:.4f}")
```

### 2.3 Datasets ใน Scikit-learn

```python
from sklearn import datasets
import pandas as pd

# Toy Datasets
iris = datasets.load_iris()
boston = datasets.fetch_california_housing()  # แทน Boston Housing
digits = datasets.load_digits()
wine = datasets.load_wine()
breast_cancer = datasets.load_breast_cancer()

print(f"Iris features: {iris.feature_names}")
print(f"Iris classes: {iris.target_names}")
print(f"Iris data shape: {iris.data.shape}")

# แปลงเป็น DataFrame
df = pd.DataFrame(iris.data, columns=iris.feature_names)
df['target'] = iris.target
df['species'] = df['target'].map({0: 'setosa', 1: 'versicolor', 2: 'virginica'})
print(f"\nIris DataFrame:\n{df.head()}")
```

```python
# สร้าง Synthetic Datasets
from sklearn.datasets import make_classification, make_regression, make_blobs

# Classification dataset
X_cls, y_cls = make_classification(
    n_samples=1000, n_features=20, n_informative=10,
    n_redundant=5, random_state=42
)
print(f"Classification: X={X_cls.shape}, y={y_cls.shape}")

# Regression dataset  
X_reg, y_reg = make_regression(
    n_samples=500, n_features=10, noise=0.1, random_state=42
)
print(f"Regression: X={X_reg.shape}, y={y_reg.shape}")

# Clustering dataset
X_blob, y_blob = make_blobs(
    n_samples=300, centers=5, cluster_std=1.0, random_state=42
)
print(f"Blobs: X={X_blob.shape}, y={y_blob.shape}")
```

---

## 3. Data Preprocessing

### 3.1 StandardScaler

StandardScaler ทำให้ข้อมูลมี mean=0 และ std=1 (Z-score normalization)

$$z = \frac{x - \mu}{\sigma}$$

```python
from sklearn.preprocessing import StandardScaler
import numpy as np

# สร้างข้อมูลตัวอย่าง
X = np.array([[1, 1000], [2, 2000], [3, 3000], [4, 4000], [5, 5000]], dtype=float)
print(f"Original data:\n{X}")
print(f"Mean: {X.mean(axis=0)}")
print(f"Std: {X.std(axis=0)}")

# ใช้ StandardScaler
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
print(f"\nScaled data:\n{X_scaled}")
print(f"Mean after scaling: {X_scaled.mean(axis=0)}")
print(f"Std after scaling: {X_scaled.std(axis=0)}")

# เก็บ parameters
print(f"\nScaler mean: {scaler.mean_}")
print(f"Scaler scale: {scaler.scale_}")

# Transform ข้อมูลใหม่
X_new = np.array([[6, 6000]])
X_new_scaled = scaler.transform(X_new)
print(f"\nNew data scaled: {X_new_scaled}")
```

### 3.2 MinMaxScaler

MinMaxScaler ปรับให้ข้อมูลอยู่ในช่วง [0, 1] หรือช่วงที่กำหนด

$$x_{scaled} = \frac{x - x_{min}}{x_{max} - x_{min}}$$

```python
from sklearn.preprocessing import MinMaxScaler

X = np.array([[1, 200], [5, 800], [10, 1500], [3, 500]], dtype=float)
print(f"Original:\n{X}")

# Default range [0, 1]
scaler_01 = MinMaxScaler()
X_scaled_01 = scaler_01.fit_transform(X)
print(f"\nScaled [0,1]:\n{X_scaled_01}")

# Custom range [-1, 1]
scaler_custom = MinMaxScaler(feature_range=(-1, 1))
X_scaled_custom = scaler_custom.fit_transform(X)
print(f"\nScaled [-1,1]:\n{X_scaled_custom}")

# เมื่อไหร่ควรใช้ MinMaxScaler?
# - เมื่อต้องการ range ที่แน่นอน
# - Neural Networks มักต้องการ [0,1]
# - เมื่อข้อมูลไม่มี outliers มาก
```

### 3.3 RobustScaler

RobustScaler ใช้ median และ IQR ทนทานต่อ outliers กว่า StandardScaler

```python
from sklearn.preprocessing import RobustScaler

# ข้อมูลที่มี outliers
X_with_outliers = np.array([[1], [2], [3], [4], [100]], dtype=float)

scaler_std = StandardScaler()
scaler_robust = RobustScaler()

X_std = scaler_std.fit_transform(X_with_outliers)
X_robust = scaler_robust.fit_transform(X_with_outliers)

print("Original:", X_with_outliers.flatten())
print("StandardScaler:", X_std.flatten().round(2))
print("RobustScaler:", X_robust.flatten().round(2))

# RobustScaler: x_scaled = (x - median) / IQR
print(f"\nMedian: {scaler_robust.center_}")
print(f"IQR: {scaler_robust.scale_}")
```

### 3.4 LabelEncoder

LabelEncoder แปลง categorical labels เป็นตัวเลข

```python
from sklearn.preprocessing import LabelEncoder

# แปลง string labels เป็นตัวเลข
le = LabelEncoder()
labels = ['cat', 'dog', 'bird', 'cat', 'dog', 'fish']
encoded = le.fit_transform(labels)
print(f"Original: {labels}")
print(f"Encoded: {encoded}")
print(f"Classes: {le.classes_}")

# Decode กลับ
decoded = le.inverse_transform(encoded)
print(f"Decoded: {decoded}")

# ใช้กับ DataFrame
import pandas as pd
df = pd.DataFrame({
    'color': ['red', 'blue', 'green', 'red', 'blue'],
    'size': ['S', 'M', 'L', 'XL', 'M']
})

for col in df.columns:
    le_col = LabelEncoder()
    df[f'{col}_encoded'] = le_col.fit_transform(df[col])
    
print(f"\nDataFrame with encoded columns:\n{df}")
```

### 3.5 OneHotEncoder

OneHotEncoder สร้าง binary columns สำหรับแต่ละ category (แนะนำมากกว่า LabelEncoder สำหรับ nominal data)

```python
from sklearn.preprocessing import OneHotEncoder
import numpy as np

# ข้อมูล categorical
colors = np.array([['red'], ['blue'], ['green'], ['red'], ['blue']])

# OneHotEncoder
ohe = OneHotEncoder(sparse_output=False)
encoded = ohe.fit_transform(colors)
print(f"Original:\n{colors.flatten()}")
print(f"OneHot encoded:\n{encoded}")
print(f"Feature names: {ohe.get_feature_names_out()}")

# จัดการหลาย columns พร้อมกัน
from sklearn.compose import ColumnTransformer

data = pd.DataFrame({
    'color': ['red', 'blue', 'green'],
    'size': ['S', 'M', 'L'],
    'price': [100, 200, 300]
})

# ColumnTransformer - apply transformations to specific columns
ct = ColumnTransformer(transformers=[
    ('onehot', OneHotEncoder(), ['color', 'size']),
    ('passthrough', 'passthrough', ['price'])
])

transformed = ct.fit_transform(data)
print(f"\nTransformed shape: {transformed.shape}")
print(f"Transformed:\n{transformed}")
```

### 3.6 การจัดการกับ Missing Values

```python
from sklearn.impute import SimpleImputer
import numpy as np
import pandas as pd

# ข้อมูลที่มี missing values
X = np.array([[1, 2, np.nan],
              [4, np.nan, 6],
              [7, 8, 9],
              [np.nan, 11, 12]])

# Mean imputation
imputer_mean = SimpleImputer(strategy='mean')
X_mean = imputer_mean.fit_transform(X)
print("Mean imputation:\n", X_mean)

# Median imputation
imputer_median = SimpleImputer(strategy='median')
X_median = imputer_median.fit_transform(X)
print("\nMedian imputation:\n", X_median)

# Most frequent (mode)
imputer_mode = SimpleImputer(strategy='most_frequent')

# Constant value
imputer_const = SimpleImputer(strategy='constant', fill_value=0)
X_const = imputer_const.fit_transform(X)
print("\nConstant (0) imputation:\n", X_const)
```

```python
# KNN Imputation (advanced)
from sklearn.impute import KNNImputer

X_missing = np.array([[1, 2, np.nan],
                       [3, 4, 3],
                       [np.nan, 6, 5],
                       [8, 8, 7]])

knn_imputer = KNNImputer(n_neighbors=2)
X_knn = knn_imputer.fit_transform(X_missing)
print("KNN Imputation:\n", X_knn)
```

---

## 4. Train/Test Split

### 4.1 Basic Train/Test Split

```python
from sklearn.model_selection import train_test_split
from sklearn.datasets import load_iris
import numpy as np

iris = load_iris()
X, y = iris.data, iris.target

# Split แบบ basic
X_train, X_test, y_train, y_test = train_test_split(
    X, y, 
    test_size=0.2,      # 20% สำหรับ test
    random_state=42,     # reproducibility
    stratify=y           # รักษาสัดส่วน class
)

print(f"Total samples: {len(X)}")
print(f"Train: {len(X_train)} ({len(X_train)/len(X)*100:.0f}%)")
print(f"Test: {len(X_test)} ({len(X_test)/len(X)*100:.0f}%)")

# ตรวจสอบ class distribution
from collections import Counter
print(f"\nTrain distribution: {Counter(y_train)}")
print(f"Test distribution: {Counter(y_test)}")
```

### 4.2 Train/Validation/Test Split

```python
# แบ่ง 3 ส่วน: 60/20/20
from sklearn.model_selection import train_test_split

X, y = iris.data, iris.target

# Split ครั้งแรก: 80/20
X_temp, X_test, y_temp, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# Split ครั้งที่สอง: 75/25 จาก 80% = 60/20
X_train, X_val, y_train, y_val = train_test_split(
    X_temp, y_temp, test_size=0.25, random_state=42, stratify=y_temp
)

print(f"Train: {len(X_train)} ({len(X_train)/len(X)*100:.0f}%)")
print(f"Validation: {len(X_val)} ({len(X_val)/len(X)*100:.0f}%)")
print(f"Test: {len(X_test)} ({len(X_test)/len(X)*100:.0f}%)")
```

---

## 5. Cross-Validation

Cross-validation ช่วยประเมินโมเดลได้แม่นยำกว่าการแบ่ง train/test ครั้งเดียว โดยใช้ข้อมูลทุกส่วนทั้งสำหรับ training และ testing

### 5.1 K-Fold Cross-Validation

```python
from sklearn.model_selection import KFold, cross_val_score
from sklearn.datasets import load_iris
from sklearn.neighbors import KNeighborsClassifier
import numpy as np

iris = load_iris()
X, y = iris.data, iris.target
model = KNeighborsClassifier(n_neighbors=5)

# Basic cross_val_score
scores = cross_val_score(model, X, y, cv=5, scoring='accuracy')
print(f"CV Scores: {scores}")
print(f"Mean: {scores.mean():.4f} (+/- {scores.std() * 2:.4f})")

# Manual K-Fold
kf = KFold(n_splits=5, shuffle=True, random_state=42)
manual_scores = []

for fold, (train_idx, val_idx) in enumerate(kf.split(X)):
    X_train, X_val = X[train_idx], X[val_idx]
    y_train, y_val = y[train_idx], y[val_idx]
    
    model.fit(X_train, y_train)
    score = model.score(X_val, y_val)
    manual_scores.append(score)
    print(f"Fold {fold+1}: {score:.4f}")

print(f"\nMean: {np.mean(manual_scores):.4f}")
```

### 5.2 Stratified K-Fold

```python
from sklearn.model_selection import StratifiedKFold, cross_val_score

# StratifiedKFold รักษาสัดส่วน class ในแต่ละ fold
skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(model, X, y, cv=skf, scoring='accuracy')
print(f"Stratified CV Scores: {scores}")
print(f"Mean: {scores.mean():.4f}")

# ตรวจสอบ class distribution ในแต่ละ fold
for fold, (train_idx, val_idx) in enumerate(skf.split(X, y)):
    train_dist = Counter(y[train_idx])
    val_dist = Counter(y[val_idx])
    print(f"Fold {fold+1} - Train: {dict(train_dist)}, Val: {dict(val_dist)}")
```

### 5.3 Cross-Validate (หลาย metrics พร้อมกัน)

```python
from sklearn.model_selection import cross_validate

# cross_validate ช่วยได้หลาย scoring พร้อมกัน
results = cross_validate(
    model, X, y,
    cv=5,
    scoring=['accuracy', 'f1_macro', 'precision_macro'],
    return_train_score=True
)

for key, values in results.items():
    if 'test' in key or 'train' in key:
        print(f"{key}: {values.mean():.4f} (+/- {values.std():.4f})")
```

### 5.4 Leave-One-Out Cross-Validation (LOOCV)

```python
from sklearn.model_selection import LeaveOneOut, cross_val_score

loo = LeaveOneOut()
# ใช้เฉพาะกับ dataset เล็กๆ เพราะ expensive มาก
X_small, y_small = X[:50], y[:50]  # ใช้แค่ 50 samples

scores = cross_val_score(model, X_small, y_small, cv=loo)
print(f"LOO CV: {scores.mean():.4f} (+/- {scores.std():.4f})")
print(f"Number of splits: {loo.get_n_splits(X_small)}")
```

---

## 6. Pipeline

Pipeline รวม preprocessing steps และ model เข้าด้วยกัน ช่วยป้องกัน data leakage และทำให้ code สะอาดขึ้น

### 6.1 Basic Pipeline

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.svm import SVC
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

iris = load_iris()
X, y = iris.data, iris.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# สร้าง Pipeline
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('classifier', SVC(kernel='rbf', C=1.0))
])

# Fit และ Predict เหมือน model ปกติ
pipe.fit(X_train, y_train)
y_pred = pipe.predict(X_test)
print(f"Pipeline Accuracy: {accuracy_score(y_test, y_pred):.4f}")

# Access individual steps
print(f"\nScaler mean: {pipe.named_steps['scaler'].mean_}")
print(f"SVC params: {pipe.named_steps['classifier'].get_params()}")
```

### 6.2 Pipeline กับ Cross-Validation

```python
from sklearn.model_selection import cross_val_score

# Pipeline + Cross-Validation ป้องกัน data leakage ได้
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('classifier', SVC())
])

scores = cross_val_score(pipe, X, y, cv=5, scoring='accuracy')
print(f"Pipeline CV Scores: {scores}")
print(f"Mean: {scores.mean():.4f}")

# ข้อผิดพลาดที่พบบ่อย: scaling ก่อน split (data leakage!)
# WRONG:
# scaler = StandardScaler()
# X_scaled = scaler.fit_transform(X)  # ใช้ข้อมูล test ด้วย!
# cross_val_score(model, X_scaled, y, cv=5)

# CORRECT: ใช้ Pipeline
# cross_val_score(pipe, X, y, cv=5)
```

### 6.3 Pipeline กับ ColumnTransformer

```python
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.ensemble import RandomForestClassifier
from sklearn.pipeline import Pipeline
import pandas as pd
import numpy as np

# สร้าง dataset ที่มีทั้ง numerical และ categorical features
np.random.seed(42)
n_samples = 200

data = pd.DataFrame({
    'age': np.random.randint(18, 80, n_samples),
    'income': np.random.randint(20000, 200000, n_samples),
    'gender': np.random.choice(['M', 'F'], n_samples),
    'education': np.random.choice(['High School', 'Bachelor', 'Master', 'PhD'], n_samples)
})
target = (data['income'] > 60000).astype(int)

X = data
y = target

# กำหนด columns
numerical_features = ['age', 'income']
categorical_features = ['gender', 'education']

# สร้าง preprocessor
preprocessor = ColumnTransformer(transformers=[
    ('num', StandardScaler(), numerical_features),
    ('cat', OneHotEncoder(handle_unknown='ignore'), categorical_features)
])

# สร้าง Pipeline
clf_pipeline = Pipeline([
    ('preprocessor', preprocessor),
    ('classifier', RandomForestClassifier(n_estimators=100, random_state=42))
])

# Train
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
clf_pipeline.fit(X_train, y_train)

print(f"Accuracy: {clf_pipeline.score(X_test, y_test):.4f}")
```

---

## 7. Model Evaluation Metrics

### 7.1 Accuracy

```python
from sklearn.metrics import accuracy_score
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier

iris = load_iris()
X, y = iris.data, iris.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)
y_pred = model.predict(X_test)

accuracy = accuracy_score(y_test, y_pred)
print(f"Accuracy: {accuracy:.4f}")

# Accuracy = TP + TN / (TP + TN + FP + FN)
# เหมาะกับ balanced dataset
# ไม่เหมาะกับ imbalanced dataset
```

### 7.2 Precision, Recall, F1-Score

```python
from sklearn.metrics import precision_score, recall_score, f1_score, classification_report

# Binary Classification
from sklearn.datasets import load_breast_cancer
bc = load_breast_cancer()
X, y = bc.data, bc.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)
y_pred = model.predict(X_test)

print(f"Precision: {precision_score(y_test, y_pred):.4f}")
print(f"Recall: {recall_score(y_test, y_pred):.4f}")
print(f"F1-Score: {f1_score(y_test, y_pred):.4f}")

# Classification Report - แสดงทุก metric
print("\nClassification Report:")
print(classification_report(y_test, y_pred, target_names=bc.target_names))
```

```python
# Precision, Recall, F1 สำหรับ Multi-class
iris = load_iris()
X, y = iris.data, iris.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)
model.fit(X_train, y_train)
y_pred = model.predict(X_test)

# averaging methods:
# 'macro': mean ของแต่ละ class (ไม่คำนึงถึง imbalance)
# 'weighted': weighted mean ตามจำนวน samples
# 'micro': คำนวณ globally

print("Macro F1:", f1_score(y_test, y_pred, average='macro'))
print("Weighted F1:", f1_score(y_test, y_pred, average='weighted'))
print("Micro F1:", f1_score(y_test, y_pred, average='micro'))
```

### 7.3 Confusion Matrix

```python
from sklearn.metrics import confusion_matrix, ConfusionMatrixDisplay
import matplotlib.pyplot as plt

iris = load_iris()
X, y = iris.data, iris.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)
y_pred = model.predict(X_test)

cm = confusion_matrix(y_test, y_pred)
print("Confusion Matrix:")
print(cm)

# Plot Confusion Matrix
disp = ConfusionMatrixDisplay(confusion_matrix=cm, display_labels=iris.target_names)
fig, ax = plt.subplots(figsize=(8, 6))
disp.plot(ax=ax, cmap='Blues')
plt.title('Confusion Matrix - Iris Classification')
plt.tight_layout()
plt.savefig('confusion_matrix.png', dpi=100)
print("\nConfusion matrix saved!")

# คำอธิบาย:
# TP (True Positive): ทำนายถูก class ที่ถูกต้อง
# TN (True Negative): ทำนายถูกว่าไม่ใช่ class นั้น
# FP (False Positive): ทำนายว่าเป็น class นั้น แต่จริงๆ ไม่ใช่
# FN (False Negative): ทำนายว่าไม่ใช่ class นั้น แต่จริงๆ เป็น
```

### 7.4 ROC-AUC

ROC (Receiver Operating Characteristic) curve แสดงความสัมพันธ์ระหว่าง True Positive Rate และ False Positive Rate

```python
from sklearn.metrics import roc_curve, roc_auc_score, auc
from sklearn.datasets import load_breast_cancer
import matplotlib.pyplot as plt

bc = load_breast_cancer()
X, y = bc.data, bc.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

# ต้องการ probability
model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)
y_prob = model.predict_proba(X_test)[:, 1]  # probability สำหรับ class 1

# ROC Curve
fpr, tpr, thresholds = roc_curve(y_test, y_prob)
auc_score = roc_auc_score(y_test, y_prob)

print(f"ROC-AUC Score: {auc_score:.4f}")

# Plot ROC
plt.figure(figsize=(8, 6))
plt.plot(fpr, tpr, color='blue', linewidth=2, label=f'ROC Curve (AUC = {auc_score:.4f})')
plt.plot([0, 1], [0, 1], color='gray', linestyle='--', label='Random Classifier')
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('ROC Curve')
plt.legend(loc='lower right')
plt.grid(True)
plt.savefig('roc_curve.png', dpi=100)
print("ROC curve saved!")
```

### 7.5 Regression Metrics

```python
from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score
from sklearn.linear_model import LinearRegression
from sklearn.datasets import make_regression
import numpy as np

X, y = make_regression(n_samples=200, n_features=5, noise=20, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

model = LinearRegression()
model.fit(X_train, y_train)
y_pred = model.predict(X_test)

mse = mean_squared_error(y_test, y_pred)
rmse = np.sqrt(mse)
mae = mean_absolute_error(y_test, y_pred)
r2 = r2_score(y_test, y_pred)

print(f"MSE: {mse:.4f}")
print(f"RMSE: {rmse:.4f}")
print(f"MAE: {mae:.4f}")
print(f"R² Score: {r2:.4f}")

# R² = 1 - SS_res/SS_tot
# R² = 1: perfect fit
# R² = 0: model ไม่ดีกว่า mean
# R² < 0: model แย่กว่า mean
```

---

## 8. Confusion Matrix เชิงลึก

```python
from sklearn.metrics import confusion_matrix
import numpy as np

def detailed_confusion_matrix(y_true, y_pred, labels=None):
    """คำนวณ metrics ทั้งหมดจาก confusion matrix"""
    cm = confusion_matrix(y_true, y_pred)
    
    # For binary classification
    if cm.shape == (2, 2):
        tn, fp, fn, tp = cm.ravel()
        
        accuracy = (tp + tn) / (tp + tn + fp + fn)
        precision = tp / (tp + fp) if (tp + fp) > 0 else 0
        recall = tp / (tp + fn) if (tp + fn) > 0 else 0
        specificity = tn / (tn + fp) if (tn + fp) > 0 else 0
        f1 = 2 * precision * recall / (precision + recall) if (precision + recall) > 0 else 0
        
        print(f"Confusion Matrix:")
        print(f"         Predicted")
        print(f"         Neg  Pos")
        print(f"Actual Neg [{tn:4d} {fp:4d}]")
        print(f"       Pos [{fn:4d} {tp:4d}]")
        print(f"\nMetrics:")
        print(f"  TP={tp}, TN={tn}, FP={fp}, FN={fn}")
        print(f"  Accuracy:    {accuracy:.4f}")
        print(f"  Precision:   {precision:.4f} (TP/(TP+FP))")
        print(f"  Recall:      {recall:.4f} (TP/(TP+FN))")
        print(f"  Specificity: {specificity:.4f} (TN/(TN+FP))")
        print(f"  F1-Score:    {f1:.4f}")
    
    return cm

# ทดสอบ
from sklearn.datasets import load_breast_cancer
bc = load_breast_cancer()
X, y = bc.data, bc.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

from sklearn.svm import SVC
svm_model = SVC(kernel='rbf', random_state=42)
svm_model.fit(X_train, y_train)
y_pred = svm_model.predict(X_test)

cm = detailed_confusion_matrix(y_test, y_pred)
```

---

## 9. Learning Curves

Learning curves แสดงว่าโมเดล overfitting หรือ underfitting และต้องการข้อมูลมากแค่ไหน

```python
from sklearn.model_selection import learning_curve
from sklearn.svm import SVC
from sklearn.datasets import load_digits
import matplotlib.pyplot as plt
import numpy as np

digits = load_digits()
X, y = digits.data, digits.target

# คำนวณ learning curve
train_sizes, train_scores, val_scores = learning_curve(
    SVC(kernel='rbf', C=1.0), X, y,
    cv=5,
    n_jobs=-1,
    train_sizes=np.linspace(0.1, 1.0, 10),
    scoring='accuracy'
)

# คำนวณ mean และ std
train_mean = train_scores.mean(axis=1)
train_std = train_scores.std(axis=1)
val_mean = val_scores.mean(axis=1)
val_std = val_scores.std(axis=1)

# Plot
plt.figure(figsize=(10, 6))
plt.plot(train_sizes, train_mean, 'o-', color='blue', label='Training Score')
plt.fill_between(train_sizes, train_mean - train_std, train_mean + train_std, alpha=0.1, color='blue')
plt.plot(train_sizes, val_mean, 'o-', color='red', label='Validation Score')
plt.fill_between(train_sizes, val_mean - val_std, val_mean + val_std, alpha=0.1, color='red')

plt.xlabel('Training Set Size')
plt.ylabel('Accuracy')
plt.title('Learning Curves - SVM')
plt.legend(loc='lower right')
plt.grid(True)
plt.savefig('learning_curves.png', dpi=100)
print("Learning curves saved!")
print(f"Final train score: {train_mean[-1]:.4f}")
print(f"Final val score: {val_mean[-1]:.4f}")
```

```python
# วิเคราะห์ Bias-Variance Tradeoff จาก Learning Curves
def analyze_learning_curve(train_mean, val_mean, threshold=0.05):
    """
    วิเคราะห์ว่าโมเดลเป็น underfitting หรือ overfitting
    """
    final_gap = train_mean[-1] - val_mean[-1]
    final_val = val_mean[-1]
    
    if final_val < 0.7:
        diagnosis = "Underfitting (High Bias)"
        suggestion = "เพิ่ม model complexity, เพิ่ม features"
    elif final_gap > threshold:
        diagnosis = "Overfitting (High Variance)"
        suggestion = "เพิ่มข้อมูล, ใช้ regularization, ลด features"
    else:
        diagnosis = "Good Fit"
        suggestion = "โมเดลเหมาะสม"
    
    print(f"Diagnosis: {diagnosis}")
    print(f"Suggestion: {suggestion}")
    print(f"Final gap (train-val): {final_gap:.4f}")
    print(f"Final validation score: {final_val:.4f}")

analyze_learning_curve(train_mean, val_mean)
```

---

## 10. Feature Selection

Feature selection ช่วยลด features ที่ไม่จำเป็น ป้องกัน overfitting และเร่งความเร็วในการ train

### 10.1 Variance Threshold

```python
from sklearn.feature_selection import VarianceThreshold
import numpy as np

# ลบ features ที่มี variance ต่ำ
X = np.array([[0, 2, 0.1],
              [0, 3, 0.2],
              [0, 4, 0.1],
              [0, 5, 0.3]])

# Feature 0 มี variance = 0 (ค่าเดียวกันทุก sample)
sel = VarianceThreshold(threshold=0.01)
X_selected = sel.fit_transform(X)
print(f"Original shape: {X.shape}")
print(f"Selected shape: {X_selected.shape}")
print(f"Features selected: {sel.get_support()}")
```

### 10.2 SelectKBest

```python
from sklearn.feature_selection import SelectKBest, f_classif, chi2, mutual_info_classif
from sklearn.datasets import load_iris

iris = load_iris()
X, y = iris.data, iris.target

# เลือก 2 features ที่ดีที่สุด
selector = SelectKBest(score_func=f_classif, k=2)
X_selected = selector.fit_transform(X, y)

print(f"Original features: {iris.feature_names}")
print(f"Scores: {selector.scores_}")
print(f"Selected features: {[iris.feature_names[i] for i in selector.get_support(indices=True)]}")
print(f"X shape: {X.shape} -> {X_selected.shape}")
```

### 10.3 Recursive Feature Elimination (RFE)

```python
from sklearn.feature_selection import RFE, RFECV
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_breast_cancer

bc = load_breast_cancer()
X, y = bc.data, bc.target

# RFE - กำหนดจำนวน features
model = LogisticRegression(max_iter=10000)
rfe = RFE(estimator=model, n_features_to_select=10)
rfe.fit(X, y)

selected_features = [bc.feature_names[i] for i in rfe.get_support(indices=True)]
print(f"Selected {rfe.n_features_to_select_} features:")
for feat, rank in zip(bc.feature_names, rfe.ranking_):
    selected = "✓" if rank == 1 else " "
    print(f"  {selected} [{rank:2d}] {feat}")
```

```python
# RFECV - หาจำนวน features ที่เหมาะสมอัตโนมัติ
from sklearn.feature_selection import RFECV
from sklearn.model_selection import StratifiedKFold

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
rfecv = RFECV(
    estimator=LogisticRegression(max_iter=10000),
    cv=cv,
    scoring='accuracy',
    min_features_to_select=5
)
rfecv.fit(X, y)

print(f"Optimal number of features: {rfecv.n_features_}")
print(f"Best CV score: {rfecv.cv_results_['mean_test_score'].max():.4f}")
```

### 10.4 Feature Importance จาก Tree Models

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_breast_cancer
import numpy as np
import matplotlib.pyplot as plt

bc = load_breast_cancer()
X, y = bc.data, bc.target

rf = RandomForestClassifier(n_estimators=100, random_state=42)
rf.fit(X, y)

# Feature importances
importances = rf.feature_importances_
indices = np.argsort(importances)[::-1]

print("Feature Importances:")
for i, idx in enumerate(indices[:10]):  # Top 10
    print(f"  {i+1:2d}. {bc.feature_names[idx]:30s}: {importances[idx]:.4f}")

# Plot
plt.figure(figsize=(10, 6))
plt.bar(range(len(importances)), importances[indices])
plt.xticks(range(len(importances)), [bc.feature_names[i] for i in indices], rotation=90)
plt.title('Feature Importances - Random Forest')
plt.ylabel('Importance Score')
plt.tight_layout()
plt.savefig('feature_importance.png', dpi=100)
```

---

## 11. Hyperparameter Tuning

### 11.1 GridSearchCV

GridSearchCV ทดสอบทุก combination ของ hyperparameters ที่กำหนด

```python
from sklearn.model_selection import GridSearchCV
from sklearn.svm import SVC
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
import time

iris = load_iris()
X, y = iris.data, iris.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# กำหนด parameter grid
param_grid = {
    'C': [0.1, 1, 10, 100],
    'gamma': ['scale', 'auto', 0.001, 0.01],
    'kernel': ['rbf', 'poly', 'sigmoid']
}

start = time.time()
grid_search = GridSearchCV(
    SVC(), param_grid,
    cv=5,
    scoring='accuracy',
    n_jobs=-1,
    verbose=1
)
grid_search.fit(X_train, y_train)
elapsed = time.time() - start

print(f"\nTime: {elapsed:.2f}s")
print(f"Best parameters: {grid_search.best_params_}")
print(f"Best CV score: {grid_search.best_score_:.4f}")
print(f"Test accuracy: {grid_search.score(X_test, y_test):.4f}")
print(f"Total combinations tested: {len(grid_search.cv_results_['params'])}")
```

### 11.2 RandomizedSearchCV

RandomizedSearchCV สุ่มทดสอบ hyperparameters เร็วกว่า GridSearchCV มาก

```python
from sklearn.model_selection import RandomizedSearchCV
from sklearn.ensemble import RandomForestClassifier
from scipy.stats import randint, uniform
import time

rf = RandomForestClassifier(random_state=42)

# กำหนด parameter distributions (ไม่ใช่ grid)
param_distributions = {
    'n_estimators': randint(50, 500),
    'max_depth': randint(3, 20),
    'min_samples_split': randint(2, 20),
    'min_samples_leaf': randint(1, 10),
    'max_features': uniform(0.1, 0.9)
}

start = time.time()
random_search = RandomizedSearchCV(
    rf, param_distributions,
    n_iter=50,          # ทดสอบ 50 combinations
    cv=5,
    scoring='accuracy',
    n_jobs=-1,
    random_state=42,
    verbose=1
)
random_search.fit(X_train, y_train)
elapsed = time.time() - start

print(f"\nTime: {elapsed:.2f}s")
print(f"Best parameters: {random_search.best_params_}")
print(f"Best CV score: {random_search.best_score_:.4f}")
print(f"Test accuracy: {random_search.score(X_test, y_test):.4f}")
```

### 11.3 Bayesian Optimization (ด้วย scikit-optimize)

```python
# pip install scikit-optimize
try:
    from skopt import BayesSearchCV
    from skopt.space import Real, Integer, Categorical
    
    search_spaces = {
        'n_estimators': Integer(50, 500),
        'max_depth': Integer(3, 20),
        'min_samples_split': Integer(2, 20),
        'max_features': Real(0.1, 0.9)
    }
    
    bayes_search = BayesSearchCV(
        RandomForestClassifier(random_state=42),
        search_spaces,
        n_iter=30,
        cv=5,
        random_state=42,
        n_jobs=-1
    )
    bayes_search.fit(X_train, y_train)
    print(f"Bayesian Optimization Best: {bayes_search.best_params_}")
    print(f"Best score: {bayes_search.best_score_:.4f}")
    
except ImportError:
    print("scikit-optimize not installed. Run: pip install scikit-optimize")
```

### 11.4 Nested Cross-Validation

```python
from sklearn.model_selection import cross_val_score, GridSearchCV
from sklearn.svm import SVC
from sklearn.datasets import load_iris

iris = load_iris()
X, y = iris.data, iris.target

# Inner CV: hyperparameter tuning
param_grid = {'C': [0.1, 1, 10], 'kernel': ['rbf', 'linear']}
inner_cv = GridSearchCV(SVC(), param_grid, cv=3, scoring='accuracy')

# Outer CV: model evaluation
outer_scores = cross_val_score(inner_cv, X, y, cv=5, scoring='accuracy')

print("Nested CV Scores:", outer_scores)
print(f"Mean: {outer_scores.mean():.4f} (+/- {outer_scores.std()*2:.4f})")
# Nested CV ให้ผลที่ไม่ bias ในการประเมิน
```

---

## 12. Complete ML Workflow Example

```python
"""
ตัวอย่าง Complete ML Workflow ที่สมบูรณ์
ใช้ Wine Quality Dataset
"""
import numpy as np
import pandas as pd
from sklearn.datasets import load_wine
from sklearn.model_selection import train_test_split, cross_val_score, GridSearchCV
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.ensemble import RandomForestClassifier
from sklearn.svm import SVC
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report, confusion_matrix, accuracy_score
import warnings
warnings.filterwarnings('ignore')

# 1. โหลดและสำรวจข้อมูล
wine = load_wine()
X, y = wine.data, wine.target
df = pd.DataFrame(X, columns=wine.feature_names)
df['target'] = y

print("=== Wine Quality Dataset ===")
print(f"Shape: {X.shape}")
print(f"Classes: {wine.target_names}")
print(f"\nClass distribution:")
for i, name in enumerate(wine.target_names):
    print(f"  {name}: {(y == i).sum()}")

print(f"\nFeature statistics:")
print(df.describe().round(2))

# 2. แบ่งข้อมูล
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# 3. สร้าง Pipelines
models = {
    'Logistic Regression': Pipeline([
        ('scaler', StandardScaler()),
        ('clf', LogisticRegression(max_iter=10000, random_state=42))
    ]),
    'SVM': Pipeline([
        ('scaler', StandardScaler()),
        ('clf', SVC(random_state=42))
    ]),
    'Random Forest': Pipeline([
        ('clf', RandomForestClassifier(n_estimators=100, random_state=42))
    ])
}

# 4. Cross-Validation
print("\n=== Cross-Validation Results ===")
cv_results = {}
for name, model in models.items():
    scores = cross_val_score(model, X_train, y_train, cv=5, scoring='accuracy')
    cv_results[name] = scores
    print(f"{name:20s}: {scores.mean():.4f} (+/- {scores.std()*2:.4f})")

# 5. เลือก Best Model และ Tune Hyperparameters
best_model_name = max(cv_results, key=lambda k: cv_results[k].mean())
print(f"\nBest model: {best_model_name}")

# Tune Random Forest
param_grid = {
    'clf__n_estimators': [50, 100, 200],
    'clf__max_depth': [None, 5, 10],
    'clf__min_samples_split': [2, 5]
}

rf_pipeline = models['Random Forest']
grid_search = GridSearchCV(rf_pipeline, param_grid, cv=5, scoring='accuracy', n_jobs=-1)
grid_search.fit(X_train, y_train)

print(f"\nBest RF parameters: {grid_search.best_params_}")
print(f"Best CV score: {grid_search.best_score_:.4f}")

# 6. Final Evaluation
print("\n=== Final Test Set Evaluation ===")
for name, model in models.items():
    model.fit(X_train, y_train)
    y_pred = model.predict(X_test)
    acc = accuracy_score(y_test, y_pred)
    print(f"{name}: {acc:.4f}")

# Best tuned model
best_model = grid_search.best_estimator_
y_pred_best = best_model.predict(X_test)
print(f"\nBest RF (tuned): {accuracy_score(y_test, y_pred_best):.4f}")
print("\nClassification Report:")
print(classification_report(y_test, y_pred_best, target_names=wine.target_names))
```

---

## 13. Validation Curves

```python
from sklearn.model_selection import validation_curve
import matplotlib.pyplot as plt
import numpy as np
from sklearn.svm import SVC
from sklearn.datasets import load_iris

iris = load_iris()
X, y = iris.data, iris.target

# หาค่า C ที่เหมาะสม
param_range = np.logspace(-3, 3, 20)

train_scores, val_scores = validation_curve(
    SVC(kernel='rbf'), X, y,
    param_name='C',
    param_range=param_range,
    cv=5, scoring='accuracy',
    n_jobs=-1
)

train_mean = train_scores.mean(axis=1)
train_std = train_scores.std(axis=1)
val_mean = val_scores.mean(axis=1)
val_std = val_scores.std(axis=1)

plt.figure(figsize=(10, 6))
plt.semilogx(param_range, train_mean, 'o-', color='blue', label='Training Score')
plt.fill_between(param_range, train_mean - train_std, train_mean + train_std, alpha=0.1, color='blue')
plt.semilogx(param_range, val_mean, 'o-', color='red', label='Validation Score')
plt.fill_between(param_range, val_mean - val_std, val_mean + val_std, alpha=0.1, color='red')

plt.xlabel('C (log scale)')
plt.ylabel('Accuracy')
plt.title('Validation Curve - SVM (C parameter)')
plt.legend(loc='lower right')
plt.grid(True)
plt.savefig('validation_curve.png', dpi=100)
print("Validation curve saved!")

optimal_C = param_range[np.argmax(val_mean)]
print(f"Optimal C: {optimal_C:.4f}")
print(f"Best validation score: {val_mean.max():.4f}")
```

---

## 14. Model Persistence

```python
import joblib
import pickle
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_iris

iris = load_iris()
X, y = iris.data, iris.target

model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X, y)

# บันทึกด้วย joblib (แนะนำสำหรับ scikit-learn)
joblib.dump(model, 'iris_rf_model.pkl')
print("Model saved with joblib!")

# โหลดกลับ
loaded_model = joblib.load('iris_rf_model.pkl')
print(f"Loaded model score: {loaded_model.score(X, y):.4f}")

# บันทึกด้วย pickle
with open('iris_model_pickle.pkl', 'wb') as f:
    pickle.dump(model, f)

with open('iris_model_pickle.pkl', 'rb') as f:
    pickle_model = pickle.load(f)

print(f"Pickle model score: {pickle_model.score(X, y):.4f}")

# บันทึก Pipeline ทั้งหมด
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler

pipe = Pipeline([('scaler', StandardScaler()), ('clf', RandomForestClassifier())])
pipe.fit(X, y)
joblib.dump(pipe, 'iris_pipeline.pkl')
print("Pipeline saved!")
```

---

## 15. Practical Tips and Best Practices

```python
"""
Best Practices สำหรับ Scikit-learn
"""

# 1. ป้องกัน Data Leakage
# BAD:
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
import numpy as np

iris = load_iris()
X, y = iris.data, iris.target

# Wrong: fit scaler ก่อน split
# scaler = StandardScaler()
# X_scaled = scaler.fit_transform(X)  # ข้อมูล test ถูกใช้ใน fit!
# X_train, X_test = train_test_split(X_scaled, ...)

# GOOD: ใช้ Pipeline หรือ fit เฉพาะ train set
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)  # fit เฉพาะ train
X_test_scaled = scaler.transform(X_test)  # transform เท่านั้น

# 2. ตั้ง random_state เสมอ
from sklearn.ensemble import RandomForestClassifier
model = RandomForestClassifier(n_estimators=100, random_state=42)
# random_state ทำให้ผลลัพธ์ reproducible

# 3. ใช้ stratify ใน classification
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# 4. ตรวจสอบ class imbalance
from collections import Counter
print(f"Class distribution: {Counter(y)}")

# 5. ใช้ appropriate metrics
# - Imbalanced: F1, ROC-AUC ดีกว่า Accuracy
# - Regression: RMSE สำหรับ interpretability, MAE สำหรับ robustness to outliers

# 6. Feature scaling สำคัญสำหรับ:
# - SVM, KNN, Linear Models, Neural Networks
# - ไม่จำเป็นสำหรับ Tree-based models (RF, GBM)
print("Best practices demonstrated!")
```

---

## 16. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Data Preprocessing Pipeline

**โจทย์**: สร้าง preprocessing pipeline สำหรับ Titanic dataset (หรือ dataset ที่มี mixed types)

```python
# เฉลย
import pandas as pd
import numpy as np
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split, cross_val_score

# สร้าง dataset จำลอง Titanic
np.random.seed(42)
n = 500
data = pd.DataFrame({
    'age': np.random.choice([np.nan if np.random.random() < 0.2 else np.random.randint(1, 80)], n).astype(float),
    'fare': np.random.exponential(50, n),
    'pclass': np.random.choice([1, 2, 3], n),
    'sex': np.random.choice(['male', 'female'], n),
    'embarked': np.random.choice(['S', 'C', 'Q', np.nan], n),
})
target = (data['fare'] > 50).astype(int)

X = data
y = target

# แก้ปัญหา NaN ใน embarked
X['embarked'] = X['embarked'].fillna('S')
X['age'] = X['age'].apply(lambda x: np.random.randint(20, 60) if pd.isna(x) else x)

numerical_features = ['age', 'fare']
categorical_features = ['sex', 'embarked']

preprocessor = ColumnTransformer([
    ('num', Pipeline([
        ('imputer', SimpleImputer(strategy='median')),
        ('scaler', StandardScaler())
    ]), numerical_features),
    ('cat', Pipeline([
        ('imputer', SimpleImputer(strategy='most_frequent')),
        ('onehot', OneHotEncoder(handle_unknown='ignore'))
    ]), categorical_features),
    ('passthrough', 'passthrough', ['pclass'])
])

full_pipeline = Pipeline([
    ('preprocessor', preprocessor),
    ('classifier', RandomForestClassifier(n_estimators=100, random_state=42))
])

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
scores = cross_val_score(full_pipeline, X_train, y_train, cv=5, scoring='accuracy')
print(f"CV Score: {scores.mean():.4f} (+/- {scores.std()*2:.4f})")
full_pipeline.fit(X_train, y_train)
print(f"Test Score: {full_pipeline.score(X_test, y_test):.4f}")
```

### แบบฝึกหัดที่ 2: Cross-Validation Comparison

**โจทย์**: เปรียบเทียบ 5 algorithms ด้วย cross-validation และแสดง boxplot

```python
# เฉลย
from sklearn.datasets import load_wine
from sklearn.model_selection import cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVC
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.neighbors import KNeighborsClassifier
import matplotlib.pyplot as plt
import numpy as np

wine = load_wine()
X, y = wine.data, wine.target

models = {
    'Logistic Reg': Pipeline([('scaler', StandardScaler()), ('clf', LogisticRegression(max_iter=1000))]),
    'SVM': Pipeline([('scaler', StandardScaler()), ('clf', SVC())]),
    'KNN': Pipeline([('scaler', StandardScaler()), ('clf', KNeighborsClassifier())]),
    'Random Forest': RandomForestClassifier(n_estimators=100, random_state=42),
    'Gradient Boosting': GradientBoostingClassifier(n_estimators=100, random_state=42)
}

results = {}
for name, model in models.items():
    scores = cross_val_score(model, X, y, cv=10, scoring='accuracy')
    results[name] = scores
    print(f"{name:20s}: {scores.mean():.4f} (+/- {scores.std()*2:.4f})")

# Boxplot
fig, ax = plt.subplots(figsize=(12, 6))
ax.boxplot(results.values(), labels=results.keys())
ax.set_ylabel('Accuracy')
ax.set_title('Algorithm Comparison - Wine Dataset')
ax.yaxis.grid(True)
plt.xticks(rotation=15)
plt.tight_layout()
plt.savefig('algorithm_comparison.png', dpi=100)
print("Comparison plot saved!")
```

### แบบฝึกหัดที่ 3: Hyperparameter Tuning

**โจทย์**: Tune hyperparameters ของ Random Forest ด้วย RandomizedSearchCV

```python
# เฉลย
from sklearn.model_selection import RandomizedSearchCV
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from scipy.stats import randint

bc = load_breast_cancer()
X, y = bc.data, bc.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)

# Baseline
baseline = RandomForestClassifier(n_estimators=100, random_state=42)
baseline.fit(X_train, y_train)
print(f"Baseline accuracy: {baseline.score(X_test, y_test):.4f}")

# Tuned
param_dist = {
    'n_estimators': randint(100, 500),
    'max_depth': [None] + list(range(5, 30, 5)),
    'min_samples_split': randint(2, 20),
    'min_samples_leaf': randint(1, 10),
    'max_features': ['sqrt', 'log2', 0.3, 0.5, 0.7]
}

random_search = RandomizedSearchCV(
    RandomForestClassifier(random_state=42),
    param_dist, n_iter=100, cv=5,
    scoring='roc_auc', n_jobs=-1, random_state=42
)
random_search.fit(X_train, y_train)

tuned_model = random_search.best_estimator_
print(f"Tuned accuracy: {tuned_model.score(X_test, y_test):.4f}")
print(f"Best params: {random_search.best_params_}")
```

### แบบฝึกหัดที่ 4: Feature Selection Pipeline

**โจทย์**: สร้าง pipeline ที่รวม feature selection เข้าไปด้วย

```python
# เฉลย
from sklearn.feature_selection import SelectFromModel, RFECV
from sklearn.ensemble import RandomForestClassifier
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import cross_val_score, train_test_split
from sklearn.metrics import classification_report

bc = load_breast_cancer()
X, y = bc.data, bc.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Pipeline with Feature Selection
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('feature_selection', SelectFromModel(
        RandomForestClassifier(n_estimators=100, random_state=42),
        threshold='mean'
    )),
    ('classifier', LogisticRegression(max_iter=10000, random_state=42))
])

pipe.fit(X_train, y_train)
print(f"Original features: {X.shape[1]}")
print(f"Selected features: {pipe.named_steps['feature_selection'].get_support().sum()}")
print(f"Test accuracy: {pipe.score(X_test, y_test):.4f}")
print(classification_report(y_test, pipe.predict(X_test)))
```

### แบบฝึกหัดที่ 5: Custom Scoring Function

**โจทย์**: สร้าง custom scoring function และใช้กับ cross-validation

```python
# เฉลย
from sklearn.metrics import make_scorer, fbeta_score
from sklearn.model_selection import cross_val_score
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_breast_cancer

bc = load_breast_cancer()
X, y = bc.data, bc.target

# F2 score (recall สำคัญกว่า precision 2 เท่า)
f2_scorer = make_scorer(fbeta_score, beta=2, average='binary')

model = RandomForestClassifier(n_estimators=100, random_state=42)
f2_scores = cross_val_score(model, X, y, cv=5, scoring=f2_scorer)
accuracy_scores = cross_val_score(model, X, y, cv=5, scoring='accuracy')

print(f"F2 Score: {f2_scores.mean():.4f} (+/- {f2_scores.std()*2:.4f})")
print(f"Accuracy: {accuracy_scores.mean():.4f} (+/- {accuracy_scores.std()*2:.4f})")

# Custom scorer ที่ซับซ้อนขึ้น
def custom_business_score(y_true, y_pred):
    """คำนึงถึง business cost: FP cost 1, FN cost 5"""
    from sklearn.metrics import confusion_matrix
    cm = confusion_matrix(y_true, y_pred)
    if cm.shape == (2, 2):
        tn, fp, fn, tp = cm.ravel()
        cost = fp * 1 + fn * 5
        return -cost  # ลบเพราะ higher is better
    return 0

business_scorer = make_scorer(custom_business_score)
business_scores = cross_val_score(model, X, y, cv=5, scoring=business_scorer)
print(f"Business Score (negative cost): {business_scores.mean():.2f}")
```

### แบบฝึกหัดที่ 6: Complete Wine Classification Project

**โจทย์**: สร้างโปรแกรมจัดประเภทไวน์แบบสมบูรณ์ พร้อม visualization

```python
# เฉลย (สรุป)
from sklearn.datasets import load_wine
from sklearn.model_selection import train_test_split, cross_val_score, GridSearchCV
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.svm import SVC
from sklearn.metrics import classification_report, confusion_matrix
import numpy as np

wine = load_wine()
X, y = wine.data, wine.target

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# Compare models
models = {
    'SVM': Pipeline([('scaler', StandardScaler()), ('clf', SVC(kernel='rbf'))]),
    'RF': RandomForestClassifier(n_estimators=200, random_state=42),
    'GB': GradientBoostingClassifier(n_estimators=100, random_state=42)
}

best_score = 0
best_name = ''
for name, model in models.items():
    scores = cross_val_score(model, X_train, y_train, cv=5)
    print(f"{name}: {scores.mean():.4f}")
    if scores.mean() > best_score:
        best_score = scores.mean()
        best_name = name

# Tune best model
print(f"\nTuning {best_name}...")
best_model = models[best_name]
best_model.fit(X_train, y_train)
y_pred = best_model.predict(X_test)

print(f"\nFinal Test Results ({best_name}):")
print(classification_report(y_test, y_pred, target_names=wine.target_names))
```

### แบบฝึกหัดที่ 7: Learning Curve Analysis

**โจทย์**: วิเคราะห์ learning curves ของ 3 models และสรุปว่าแต่ละ model มีปัญหาอะไร

```python
# เฉลย
from sklearn.model_selection import learning_curve
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVC
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_breast_cancer
import numpy as np
import matplotlib.pyplot as plt

bc = load_breast_cancer()
X, y = bc.data, bc.target

# StandardScaler for LR and SVM
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline

models = {
    'Logistic Regression': Pipeline([('scaler', StandardScaler()), 
                                       ('clf', LogisticRegression(max_iter=1000))]),
    'SVM (RBF)': Pipeline([('scaler', StandardScaler()), 
                             ('clf', SVC(kernel='rbf'))]),
    'Random Forest': RandomForestClassifier(n_estimators=100, random_state=42)
}

fig, axes = plt.subplots(1, 3, figsize=(18, 5))

for ax, (name, model) in zip(axes, models.items()):
    train_sizes, train_scores, val_scores = learning_curve(
        model, X, y, cv=5, n_jobs=-1,
        train_sizes=np.linspace(0.1, 1.0, 10),
        scoring='accuracy'
    )
    
    train_mean = train_scores.mean(axis=1)
    val_mean = val_scores.mean(axis=1)
    
    ax.plot(train_sizes, train_mean, 'o-', label='Train', color='blue')
    ax.plot(train_sizes, val_mean, 'o-', label='Validation', color='red')
    ax.set_title(name)
    ax.set_xlabel('Training Size')
    ax.set_ylabel('Accuracy')
    ax.legend()
    ax.grid(True)
    
    gap = train_mean[-1] - val_mean[-1]
    analysis = "Overfitting" if gap > 0.05 else ("Underfitting" if val_mean[-1] < 0.85 else "Good")
    ax.set_title(f"{name}\n({analysis})")

plt.tight_layout()
plt.savefig('learning_curves_comparison.png', dpi=100)
print("Learning curves comparison saved!")
```

### แบบฝึกหัดที่ 8: End-to-End ML Project

**โจทย์**: สร้างโปรแกรม ML แบบสมบูรณ์สำหรับ Heart Disease Prediction

```python
# เฉลย
import numpy as np
import pandas as pd
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split, GridSearchCV, cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report, roc_auc_score
import warnings
warnings.filterwarnings('ignore')

# จำลอง Heart Disease Dataset
np.random.seed(42)
n = 1000

# สร้างข้อมูลจำลอง
X, y = make_classification(
    n_samples=n, n_features=13, n_informative=8,
    n_redundant=3, n_classes=2, weights=[0.55, 0.45],
    random_state=42
)

feature_names = ['age', 'sex', 'cp', 'trestbps', 'chol', 'fbs', 
                  'restecg', 'thalach', 'exang', 'oldpeak', 'slope', 'ca', 'thal']
df = pd.DataFrame(X, columns=feature_names)
df['target'] = y

print("Heart Disease Dataset:")
print(f"  Shape: {df.shape}")
print(f"  Positive cases: {y.sum()} ({y.mean()*100:.1f}%)")

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# Model comparison
results = {}
for name, model in [
    ('LR', Pipeline([('scaler', StandardScaler()), ('clf', LogisticRegression(max_iter=1000))])),
    ('RF', RandomForestClassifier(n_estimators=100, random_state=42)),
    ('GB', GradientBoostingClassifier(n_estimators=100, random_state=42))
]:
    scores = cross_val_score(model, X_train, y_train, cv=5, scoring='roc_auc')
    results[name] = scores
    print(f"{name}: ROC-AUC = {scores.mean():.4f}")

# Best model: GB
gb_param_grid = {
    'n_estimators': [100, 200],
    'max_depth': [3, 5],
    'learning_rate': [0.05, 0.1]
}

grid_search = GridSearchCV(
    GradientBoostingClassifier(random_state=42),
    gb_param_grid, cv=5, scoring='roc_auc', n_jobs=-1
)
grid_search.fit(X_train, y_train)
best_model = grid_search.best_estimator_

y_pred = best_model.predict(X_test)
y_prob = best_model.predict_proba(X_test)[:, 1]

print(f"\nFinal Results:")
print(f"ROC-AUC: {roc_auc_score(y_test, y_prob):.4f}")
print(classification_report(y_test, y_pred, target_names=['No Disease', 'Disease']))
print(f"Best params: {grid_search.best_params_}")
```

---

## สรุปบทที่ 76

ในบทนี้เราได้เรียนรู้:

| หัวข้อ | สิ่งที่เรียนรู้ |
|--------|----------------|
| ML Types | Supervised, Unsupervised, RL |
| Preprocessing | StandardScaler, MinMaxScaler, RobustScaler, LabelEncoder, OneHotEncoder |
| Train/Test Split | Basic split, Stratified split, 3-way split |
| Cross-Validation | KFold, StratifiedKFold, LOOCV, cross_val_score |
| Pipeline | Basic pipeline, ColumnTransformer pipeline |
| Model Evaluation | Accuracy, Precision, Recall, F1, ROC-AUC, Confusion Matrix |
| Learning Curves | Diagnosis of overfitting/underfitting |
| Feature Selection | VarianceThreshold, SelectKBest, RFE, Feature Importance |
| Hyperparameter Tuning | GridSearchCV, RandomizedSearchCV, Bayesian Optimization |
| Model Persistence | joblib, pickle |

**Key Takeaways:**
1. ใช้ Pipeline เสมอเพื่อป้องกัน data leakage
2. ใช้ stratify ใน train/test split สำหรับ classification
3. Cross-validation ดีกว่า single train/test split
4. เลือก metric ให้เหมาะกับ problem (imbalanced -> F1/AUC)
5. Feature scaling จำเป็นสำหรับ distance-based และ gradient-based models
6. Regularization ช่วยลด overfitting

**Part 77** จะเรียนเรื่อง Machine Learning Regression algorithms ในเชิงลึก
