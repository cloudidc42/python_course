# Part 77: Machine Learning - Regression

## บทนำ

Regression คือ task ใน Supervised Learning ที่ทำนายค่าตัวเลขต่อเนื่อง (continuous values) เช่น ราคาบ้าน, อุณหภูมิ, ยอดขาย บทนี้จะครอบคลุม regression algorithms ตั้งแต่ Linear Regression พื้นฐานจนถึง Gradient Boosting ขั้นสูง

---

## 1. Linear Regression

### 1.1 Simple Linear Regression

Linear Regression หาเส้นตรงที่ fit กับข้อมูลดีที่สุด โดยลด MSE (Mean Squared Error)

$$y = \beta_0 + \beta_1 x + \epsilon$$

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error, r2_score

# สร้างข้อมูลตัวอย่าง
np.random.seed(42)
X = np.random.rand(100, 1) * 10  # features
y = 3.5 * X.flatten() + 7 + np.random.randn(100) * 2  # y = 3.5x + 7 + noise

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# สร้างและ train model
model = LinearRegression()
model.fit(X_train, y_train)

print(f"Coefficient (slope): {model.coef_[0]:.4f}")
print(f"Intercept: {model.intercept_:.4f}")
print(f"Equation: y = {model.coef_[0]:.4f}x + {model.intercept_:.4f}")

y_pred = model.predict(X_test)
mse = mean_squared_error(y_test, y_pred)
r2 = r2_score(y_test, y_pred)

print(f"\nMSE: {mse:.4f}")
print(f"RMSE: {np.sqrt(mse):.4f}")
print(f"R² Score: {r2:.4f}")

# Plot
plt.figure(figsize=(10, 6))
plt.scatter(X_train, y_train, color='blue', alpha=0.5, label='Training data')
plt.scatter(X_test, y_test, color='green', alpha=0.5, label='Test data')
plt.plot(X_test, y_pred, color='red', linewidth=2, label=f'Regression line (R²={r2:.4f})')
plt.xlabel('X')
plt.ylabel('y')
plt.title('Simple Linear Regression')
plt.legend()
plt.grid(True)
plt.savefig('simple_linear_regression.png', dpi=100)
print("Plot saved!")
```

### 1.2 Gradient Descent (Manual Implementation)

```python
class LinearRegressionGD:
    """Linear Regression ด้วย Gradient Descent"""
    
    def __init__(self, learning_rate=0.01, n_iterations=1000):
        self.lr = learning_rate
        self.n_iter = n_iterations
        self.weights = None
        self.bias = None
        self.loss_history = []
    
    def fit(self, X, y):
        n_samples, n_features = X.shape
        self.weights = np.zeros(n_features)
        self.bias = 0
        
        for i in range(self.n_iter):
            # Forward pass
            y_pred = np.dot(X, self.weights) + self.bias
            
            # Compute MSE loss
            loss = np.mean((y_pred - y) ** 2)
            self.loss_history.append(loss)
            
            # Compute gradients
            dw = (2/n_samples) * np.dot(X.T, (y_pred - y))
            db = (2/n_samples) * np.sum(y_pred - y)
            
            # Update parameters
            self.weights -= self.lr * dw
            self.bias -= self.lr * db
            
            if i % 100 == 0:
                print(f"Iteration {i:4d}: Loss = {loss:.6f}")
    
    def predict(self, X):
        return np.dot(X, self.weights) + self.bias

# ทดสอบ
X_train_s = (X_train - X_train.mean()) / X_train.std()
X_test_s = (X_test - X_train.mean()) / X_train.std()

custom_model = LinearRegressionGD(learning_rate=0.1, n_iterations=500)
custom_model.fit(X_train_s, y_train)
y_pred_custom = custom_model.predict(X_test_s)
print(f"\nCustom GD MSE: {mean_squared_error(y_test, y_pred_custom):.4f}")
print(f"Custom GD R²: {r2_score(y_test, y_pred_custom):.4f}")
```

---

## 2. Multiple Linear Regression

Multiple Linear Regression มีหลาย features:

$$y = \beta_0 + \beta_1 x_1 + \beta_2 x_2 + ... + \beta_p x_p + \epsilon$$

```python
import pandas as pd
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error, r2_score
import numpy as np

# สร้าง dataset: ราคาบ้าน
np.random.seed(42)
n = 500

data = pd.DataFrame({
    'area': np.random.randint(50, 300, n),
    'bedrooms': np.random.randint(1, 6, n),
    'bathrooms': np.random.randint(1, 4, n),
    'age': np.random.randint(0, 50, n),
    'distance_to_center': np.random.uniform(0.5, 30, n)
})

# สร้าง target ที่มีความสัมพันธ์จริง
data['price'] = (
    2000 * data['area'] + 
    150000 * data['bedrooms'] + 
    100000 * data['bathrooms'] - 
    3000 * data['age'] - 
    15000 * data['distance_to_center'] + 
    np.random.randn(n) * 50000
)
data['price'] = np.maximum(data['price'], 100000)  # ราคาขั้นต่ำ

X = data.drop('price', axis=1)
y = data['price']

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Train model
model = LinearRegression()
model.fit(X_train, y_train)

print("Multiple Linear Regression - House Price")
print("=" * 50)
for feat, coef in zip(X.columns, model.coef_):
    print(f"  {feat:25s}: {coef:10.2f}")
print(f"  Intercept: {model.intercept_:.2f}")

y_pred = model.predict(X_test)
rmse = np.sqrt(mean_squared_error(y_test, y_pred))
r2 = r2_score(y_test, y_pred)

print(f"\nRMSE: {rmse:,.2f}")
print(f"R² Score: {r2:.4f}")
print(f"Average actual price: {y_test.mean():,.2f}")
print(f"Average predicted price: {y_pred.mean():,.2f}")
```

```python
# Residual Analysis
y_pred = model.predict(X_test)
residuals = y_test.values - y_pred

import matplotlib.pyplot as plt

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Residuals vs Fitted
axes[0].scatter(y_pred, residuals, alpha=0.5, color='blue')
axes[0].axhline(y=0, color='red', linestyle='--')
axes[0].set_xlabel('Fitted Values')
axes[0].set_ylabel('Residuals')
axes[0].set_title('Residuals vs Fitted')
axes[0].grid(True)

# Q-Q Plot (Normal distribution of residuals)
from scipy import stats
stats.probplot(residuals, dist="norm", plot=axes[1])
axes[1].set_title('Normal Q-Q Plot of Residuals')

plt.tight_layout()
plt.savefig('residual_analysis.png', dpi=100)
print("Residual analysis saved!")
```

---

## 3. Polynomial Regression

Polynomial Regression ขยาย Linear Regression โดยเพิ่ม polynomial features

$$y = \beta_0 + \beta_1 x + \beta_2 x^2 + ... + \beta_d x^d$$

```python
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression
from sklearn.pipeline import Pipeline
from sklearn.metrics import mean_squared_error, r2_score
import numpy as np
import matplotlib.pyplot as plt

# Non-linear data
np.random.seed(42)
X = np.sort(np.random.rand(100, 1) * 6 - 3)  # range [-3, 3]
y = 0.5 * X.flatten()**3 - 2 * X.flatten()**2 + X.flatten() + 3 + np.random.randn(100) * 0.5

# Polynomial degrees to compare
degrees = [1, 2, 3, 5, 10]

fig, axes = plt.subplots(1, len(degrees), figsize=(20, 4))
X_plot = np.linspace(-3, 3, 300).reshape(-1, 1)

for ax, degree in zip(axes, degrees):
    poly_pipe = Pipeline([
        ('poly', PolynomialFeatures(degree=degree)),
        ('linear', LinearRegression())
    ])
    poly_pipe.fit(X, y)
    y_plot = poly_pipe.predict(X_plot)
    y_pred = poly_pipe.predict(X)
    r2 = r2_score(y, y_pred)
    
    ax.scatter(X, y, color='blue', alpha=0.5, s=10, label='Data')
    ax.plot(X_plot, y_plot, color='red', linewidth=2, label=f'Degree {degree}')
    ax.set_title(f'Degree {degree}\nR²={r2:.4f}')
    ax.legend()
    ax.set_ylim(-10, 10)
    ax.grid(True)

plt.tight_layout()
plt.savefig('polynomial_regression.png', dpi=100)
print("Polynomial regression plot saved!")

# เลือก degree ที่เหมาะสมด้วย CV
from sklearn.model_selection import cross_val_score

cv_scores = {}
for degree in range(1, 15):
    pipe = Pipeline([
        ('poly', PolynomialFeatures(degree=degree)),
        ('scaler', StandardScaler()),
        ('linear', LinearRegression())
    ])
    scores = cross_val_score(pipe, X, y, cv=5, scoring='r2')
    cv_scores[degree] = scores.mean()
    print(f"Degree {degree:2d}: CV R² = {scores.mean():.4f}")

best_degree = max(cv_scores, key=cv_scores.get)
print(f"\nBest degree: {best_degree}")
```

---

## 4. Ridge Regression (L2 Regularization)

Ridge Regression เพิ่ม L2 penalty เพื่อป้องกัน overfitting:

$$\text{Loss} = MSE + \alpha \sum_{j=1}^{p} \beta_j^2$$

```python
from sklearn.linear_model import Ridge, RidgeCV
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error, r2_score
import numpy as np
import matplotlib.pyplot as plt

# Dataset ที่มี many features (multicollinearity)
np.random.seed(42)
n_samples, n_features = 200, 50

X = np.random.randn(n_samples, n_features)
# เพิ่ม multicollinearity
X[:, 1] = X[:, 0] + np.random.randn(n_samples) * 0.1
X[:, 2] = X[:, 0] - X[:, 1] + np.random.randn(n_samples) * 0.1

# Only 10 features are truly important
true_coef = np.zeros(n_features)
true_coef[:10] = np.random.randn(10) * 2
y = X.dot(true_coef) + np.random.randn(n_samples) * 0.5

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

# Compare Linear vs Ridge
from sklearn.linear_model import LinearRegression
lr = LinearRegression()
lr.fit(X_train_s, y_train)
lr_r2 = r2_score(y_test, lr.predict(X_test_s))

# Ridge กับ alpha ต่างๆ
alphas = [0.01, 0.1, 1, 10, 100, 1000]
ridge_results = []

for alpha in alphas:
    ridge = Ridge(alpha=alpha)
    ridge.fit(X_train_s, y_train)
    r2 = r2_score(y_test, ridge.predict(X_test_s))
    rmse = np.sqrt(mean_squared_error(y_test, ridge.predict(X_test_s)))
    ridge_results.append((alpha, r2, rmse))
    print(f"Ridge alpha={alpha:7.2f}: R²={r2:.4f}, RMSE={rmse:.4f}")

print(f"\nLinear Regression: R²={lr_r2:.4f}")

# RidgeCV - หา alpha อัตโนมัติ
ridge_cv = RidgeCV(alphas=np.logspace(-3, 3, 100), cv=5)
ridge_cv.fit(X_train_s, y_train)
print(f"\nBest alpha (CV): {ridge_cv.alpha_:.4f}")
print(f"Ridge CV R²: {r2_score(y_test, ridge_cv.predict(X_test_s)):.4f}")
```

```python
# Coefficient Comparison: Linear vs Ridge
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Coefficients comparison
axes[0].bar(range(n_features), lr.coef_, color='blue', alpha=0.7, label='Linear')
axes[0].bar(range(n_features), ridge_cv.coef_, color='red', alpha=0.7, label='Ridge')
axes[0].set_xlabel('Feature Index')
axes[0].set_ylabel('Coefficient Value')
axes[0].set_title('Linear vs Ridge Coefficients')
axes[0].legend()
axes[0].grid(True, alpha=0.3)

# Regularization path
coefs = []
for alpha in np.logspace(-3, 3, 100):
    r = Ridge(alpha=alpha)
    r.fit(X_train_s, y_train)
    coefs.append(r.coef_)

axes[1].plot(np.logspace(-3, 3, 100), coefs)
axes[1].set_xscale('log')
axes[1].set_xlabel('Alpha (regularization strength)')
axes[1].set_ylabel('Coefficients')
axes[1].set_title('Ridge Regularization Path')
axes[1].axvline(ridge_cv.alpha_, color='red', linestyle='--', label=f'Best alpha={ridge_cv.alpha_:.2f}')
axes[1].legend()
axes[1].grid(True)

plt.tight_layout()
plt.savefig('ridge_regression.png', dpi=100)
print("Ridge regression plots saved!")
```

---

## 5. Lasso Regression (L1 Regularization)

Lasso เพิ่ม L1 penalty ซึ่งทำให้ coefficients บางตัวเป็น 0 (feature selection อัตโนมัติ):

$$\text{Loss} = MSE + \alpha \sum_{j=1}^{p} |\beta_j|$$

```python
from sklearn.linear_model import Lasso, LassoCV
from sklearn.preprocessing import StandardScaler
import numpy as np
import matplotlib.pyplot as plt

np.random.seed(42)
n_samples, n_features = 200, 50
X = np.random.randn(n_samples, n_features)
true_coef = np.zeros(n_features)
true_coef[:5] = [3, -2, 1.5, -1, 0.5]  # เฉพาะ 5 features ที่สำคัญ
y = X.dot(true_coef) + np.random.randn(n_samples) * 0.5

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# LassoCV
lasso_cv = LassoCV(cv=5, max_iter=10000)
lasso_cv.fit(X_scaled, y)

print(f"Best alpha: {lasso_cv.alpha_:.6f}")
print(f"\nNon-zero coefficients: {(lasso_cv.coef_ != 0).sum()}/{n_features}")
print("\nCoefficients (non-zero):")
for i, coef in enumerate(lasso_cv.coef_):
    if abs(coef) > 0.001:
        print(f"  Feature {i:2d}: {coef:8.4f} (True: {true_coef[i]:.4f})")

# Lasso Path
from sklearn.linear_model import lasso_path

alphas, coefs, _ = lasso_path(X_scaled, y, eps=1e-5)

plt.figure(figsize=(10, 6))
for i, coef_path in enumerate(coefs):
    if max(abs(coef_path)) > 0.1:  # แสดงเฉพาะ features สำคัญ
        plt.plot(np.log10(alphas[::-1]), coef_path[::-1], label=f'Feature {i}')

plt.axvline(x=np.log10(lasso_cv.alpha_), color='red', linestyle='--', label='Best alpha')
plt.xlabel('log(alpha)')
plt.ylabel('Coefficients')
plt.title('Lasso Regularization Path')
plt.legend()
plt.grid(True)
plt.savefig('lasso_path.png', dpi=100)
print("Lasso path saved!")
```

---

## 6. ElasticNet

ElasticNet รวม L1 และ L2 regularization:

$$\text{Loss} = MSE + \alpha \left( \rho \sum|\beta_j| + \frac{1-\rho}{2} \sum \beta_j^2 \right)$$

```python
from sklearn.linear_model import ElasticNet, ElasticNetCV
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
from sklearn.metrics import r2_score
import numpy as np

np.random.seed(42)
n = 300
X = np.random.randn(n, 20)
true_coef = np.zeros(20)
true_coef[:8] = np.random.randn(8) * 2
y = X.dot(true_coef) + np.random.randn(n) * 0.5

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

# ElasticNet CV
enet_cv = ElasticNetCV(
    l1_ratio=[0.1, 0.5, 0.7, 0.9, 0.95, 1.0],  # l1_ratio=1: Lasso, l1_ratio=0: Ridge
    cv=5, max_iter=10000
)
enet_cv.fit(X_train_s, y_train)

print(f"Best alpha: {enet_cv.alpha_:.6f}")
print(f"Best l1_ratio: {enet_cv.l1_ratio_:.2f}")
print(f"Non-zero coefs: {(enet_cv.coef_ != 0).sum()}")
print(f"R²: {r2_score(y_test, enet_cv.predict(X_test_s)):.4f}")

# เปรียบเทียบ Ridge, Lasso, ElasticNet
from sklearn.linear_model import Ridge, Lasso
models_compare = {
    'Ridge': Ridge(alpha=1.0),
    'Lasso': Lasso(alpha=0.01, max_iter=10000),
    'ElasticNet': ElasticNet(alpha=0.01, l1_ratio=0.5, max_iter=10000)
}

print("\nComparison:")
for name, m in models_compare.items():
    m.fit(X_train_s, y_train)
    r2 = r2_score(y_test, m.predict(X_test_s))
    nonzero = (m.coef_ != 0).sum() if hasattr(m, 'coef_') else 'N/A'
    print(f"  {name:12s}: R²={r2:.4f}, Non-zero coefs={nonzero}")
```

---

## 7. Decision Tree Regressor

```python
from sklearn.tree import DecisionTreeRegressor, export_text
from sklearn.model_selection import cross_val_score
import numpy as np

# ตัวอย่าง: ทำนายราคาบ้าน
from sklearn.datasets import fetch_california_housing
housing = fetch_california_housing()
X, y = housing.data, housing.target

from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Basic Decision Tree
dt = DecisionTreeRegressor(max_depth=5, min_samples_leaf=20, random_state=42)
dt.fit(X_train, y_train)

from sklearn.metrics import mean_squared_error, r2_score
y_pred = dt.predict(X_test)
print("Decision Tree Regressor:")
print(f"  RMSE: {np.sqrt(mean_squared_error(y_test, y_pred)):.4f}")
print(f"  R²: {r2_score(y_test, y_pred):.4f}")

# Effect of max_depth
depths = range(1, 20)
train_scores, test_scores = [], []

for depth in depths:
    dt = DecisionTreeRegressor(max_depth=depth, random_state=42)
    dt.fit(X_train, y_train)
    train_scores.append(r2_score(y_train, dt.predict(X_train)))
    test_scores.append(r2_score(y_test, dt.predict(X_test)))

best_depth = depths[np.argmax(test_scores)]
print(f"\nBest max_depth: {best_depth}")
print(f"Best test R²: {max(test_scores):.4f}")

import matplotlib.pyplot as plt
plt.figure(figsize=(10, 6))
plt.plot(depths, train_scores, 'o-', label='Train R²', color='blue')
plt.plot(depths, test_scores, 'o-', label='Test R²', color='red')
plt.axvline(x=best_depth, color='green', linestyle='--', label=f'Best depth={best_depth}')
plt.xlabel('Max Depth')
plt.ylabel('R² Score')
plt.title('Decision Tree - Effect of Max Depth')
plt.legend()
plt.grid(True)
plt.savefig('dt_depth_effect.png', dpi=100)
```

---

## 8. Random Forest Regressor

```python
from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import randint
import numpy as np

from sklearn.datasets import fetch_california_housing
housing = fetch_california_housing()
X, y = housing.data, housing.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Basic Random Forest
rf = RandomForestRegressor(n_estimators=100, random_state=42, n_jobs=-1)
rf.fit(X_train, y_train)

y_pred = rf.predict(X_test)
rmse = np.sqrt(mean_squared_error(y_test, y_pred))
r2 = r2_score(y_test, y_pred)
print(f"Random Forest:")
print(f"  RMSE: {rmse:.4f}")
print(f"  R²: {r2:.4f}")

# Feature Importances
import pandas as pd
importances = pd.Series(rf.feature_importances_, index=housing.feature_names)
importances = importances.sort_values(ascending=False)
print(f"\nFeature Importances:")
print(importances)

# Hyperparameter Tuning
param_dist = {
    'n_estimators': randint(100, 500),
    'max_depth': [None] + list(range(5, 25, 5)),
    'min_samples_split': randint(2, 20),
    'min_samples_leaf': randint(1, 10),
    'max_features': [0.3, 0.5, 0.7, 'sqrt', 'log2']
}

rand_search = RandomizedSearchCV(
    RandomForestRegressor(random_state=42, n_jobs=-1),
    param_dist, n_iter=30, cv=5,
    scoring='neg_root_mean_squared_error',
    n_jobs=-1, random_state=42
)
rand_search.fit(X_train, y_train)

best_rf = rand_search.best_estimator_
y_pred_best = best_rf.predict(X_test)
print(f"\nBest RF after tuning:")
print(f"  RMSE: {np.sqrt(mean_squared_error(y_test, y_pred_best)):.4f}")
print(f"  R²: {r2_score(y_test, y_pred_best):.4f}")
print(f"  Best params: {rand_search.best_params_}")
```

---

## 9. Gradient Boosting Regression

### 9.1 sklearn GradientBoosting

```python
from sklearn.ensemble import GradientBoostingRegressor
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error, r2_score
import numpy as np

from sklearn.datasets import fetch_california_housing
housing = fetch_california_housing()
X, y = housing.data, housing.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Gradient Boosting
gb = GradientBoostingRegressor(
    n_estimators=300,
    max_depth=5,
    learning_rate=0.05,
    min_samples_leaf=20,
    random_state=42
)
gb.fit(X_train, y_train)

y_pred = gb.predict(X_test)
print(f"Gradient Boosting:")
print(f"  RMSE: {np.sqrt(mean_squared_error(y_test, y_pred)):.4f}")
print(f"  R²: {r2_score(y_test, y_pred):.4f}")

# Training vs Validation Error over iterations
test_score = np.zeros(gb.n_estimators)
for i, y_pred_staged in enumerate(gb.staged_predict(X_test)):
    test_score[i] = mean_squared_error(y_test, y_pred_staged)

import matplotlib.pyplot as plt
plt.figure(figsize=(10, 6))
plt.plot(range(1, gb.n_estimators + 1), test_score, color='red', label='Test')
plt.plot(range(1, gb.n_estimators + 1), 
         gb.train_score_, color='blue', label='Train')
plt.xlabel('Number of Trees')
plt.ylabel('MSE')
plt.title('Gradient Boosting - Training Progress')
plt.legend()
plt.grid(True)
plt.savefig('gb_training.png', dpi=100)
print("GB training plot saved!")
```

### 9.2 XGBoost

```python
try:
    import xgboost as xgb
    from sklearn.model_selection import train_test_split
    from sklearn.metrics import mean_squared_error, r2_score
    import numpy as np
    
    from sklearn.datasets import fetch_california_housing
    housing = fetch_california_housing()
    X, y = housing.data, housing.target
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
    
    # XGBoost Regressor
    xgb_model = xgb.XGBRegressor(
        n_estimators=500,
        max_depth=6,
        learning_rate=0.05,
        subsample=0.8,
        colsample_bytree=0.8,
        reg_alpha=0.1,   # L1
        reg_lambda=1.0,  # L2
        random_state=42,
        n_jobs=-1
    )
    
    xgb_model.fit(
        X_train, y_train,
        eval_set=[(X_test, y_test)],
        verbose=100
    )
    
    y_pred = xgb_model.predict(X_test)
    print(f"XGBoost:")
    print(f"  RMSE: {np.sqrt(mean_squared_error(y_test, y_pred)):.4f}")
    print(f"  R²: {r2_score(y_test, y_pred):.4f}")
    
    # Feature importance
    import pandas as pd
    importance_df = pd.DataFrame({
        'feature': housing.feature_names,
        'importance': xgb_model.feature_importances_
    }).sort_values('importance', ascending=False)
    print("\nXGBoost Feature Importances:")
    print(importance_df)
    
except ImportError:
    print("XGBoost not installed. Run: pip install xgboost")
    print("Using sklearn GradientBoosting as fallback...")
```

### 9.3 LightGBM

```python
try:
    import lightgbm as lgb
    from sklearn.model_selection import train_test_split
    from sklearn.metrics import mean_squared_error, r2_score
    import numpy as np
    
    from sklearn.datasets import fetch_california_housing
    housing = fetch_california_housing()
    X, y = housing.data, housing.target
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
    
    # LightGBM - เร็วกว่า XGBoost มากสำหรับข้อมูลใหญ่
    lgb_model = lgb.LGBMRegressor(
        n_estimators=500,
        max_depth=6,
        learning_rate=0.05,
        num_leaves=31,
        subsample=0.8,
        colsample_bytree=0.8,
        reg_alpha=0.1,
        reg_lambda=1.0,
        random_state=42,
        n_jobs=-1
    )
    
    lgb_model.fit(
        X_train, y_train,
        eval_set=[(X_test, y_test)]
    )
    
    y_pred = lgb_model.predict(X_test)
    print(f"LightGBM:")
    print(f"  RMSE: {np.sqrt(mean_squared_error(y_test, y_pred)):.4f}")
    print(f"  R²: {r2_score(y_test, y_pred):.4f}")
    
except ImportError:
    print("LightGBM not installed. Run: pip install lightgbm")
```

---

## 10. Support Vector Regression (SVR)

```python
from sklearn.svm import SVR
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.model_selection import GridSearchCV, train_test_split
from sklearn.metrics import mean_squared_error, r2_score
import numpy as np

# SVR ต้องการการ scale ข้อมูลเสมอ
np.random.seed(42)
X = np.sort(np.random.rand(200, 1) * 10, axis=0)
y = np.sin(X.flatten()) + np.random.randn(200) * 0.2

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# SVR Pipeline
svr_pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('svr', SVR(kernel='rbf', C=100, epsilon=0.1, gamma='scale'))
])

svr_pipeline.fit(X_train, y_train)
y_pred = svr_pipeline.predict(X_test)

print(f"SVR:")
print(f"  RMSE: {np.sqrt(mean_squared_error(y_test, y_pred)):.4f}")
print(f"  R²: {r2_score(y_test, y_pred):.4f}")

# SVR with different kernels
kernels = ['linear', 'poly', 'rbf', 'sigmoid']
for kernel in kernels:
    svr = Pipeline([
        ('scaler', StandardScaler()),
        ('svr', SVR(kernel=kernel))
    ])
    svr.fit(X_train, y_train)
    r2 = r2_score(y_test, svr.predict(X_test))
    print(f"  SVR ({kernel}): R²={r2:.4f}")

# Grid Search
param_grid = {
    'svr__C': [0.1, 1, 10, 100],
    'svr__epsilon': [0.01, 0.1, 1],
    'svr__gamma': ['scale', 'auto', 0.01, 0.1]
}

grid_search = GridSearchCV(
    Pipeline([('scaler', StandardScaler()), ('svr', SVR(kernel='rbf'))]),
    param_grid, cv=5, scoring='neg_root_mean_squared_error', n_jobs=-1
)
grid_search.fit(X_train, y_train)
print(f"\nBest SVR params: {grid_search.best_params_}")
print(f"Best RMSE: {-grid_search.best_score_:.4f}")
```

---

## 11. Evaluation Metrics for Regression

```python
import numpy as np
from sklearn.metrics import (
    mean_squared_error, mean_absolute_error, r2_score,
    mean_absolute_percentage_error, explained_variance_score
)

def evaluate_regression(y_true, y_pred, model_name="Model"):
    """คำนวณ metrics ทั้งหมดสำหรับ regression"""
    mse = mean_squared_error(y_true, y_pred)
    rmse = np.sqrt(mse)
    mae = mean_absolute_error(y_true, y_pred)
    r2 = r2_score(y_true, y_pred)
    mape = mean_absolute_percentage_error(y_true, y_pred)
    evs = explained_variance_score(y_true, y_pred)
    
    # Adjusted R²
    n = len(y_true)
    p = 1  # ปรับตาม number of features
    adj_r2 = 1 - (1 - r2) * (n - 1) / (n - p - 1)
    
    print(f"\n{'='*40}")
    print(f"Model: {model_name}")
    print(f"{'='*40}")
    print(f"MSE:   {mse:.4f}")
    print(f"RMSE:  {rmse:.4f}")
    print(f"MAE:   {mae:.4f}")
    print(f"MAPE:  {mape*100:.2f}%")
    print(f"R²:    {r2:.4f}")
    print(f"Adj R²:{adj_r2:.4f}")
    print(f"EVS:   {evs:.4f}")
    
    return {'mse': mse, 'rmse': rmse, 'mae': mae, 'r2': r2}

# ตัวอย่างการใช้
np.random.seed(42)
y_true = np.random.rand(100) * 100
y_pred = y_true + np.random.randn(100) * 10

metrics = evaluate_regression(y_true, y_pred, "Sample Model")

print("\n--- Metric Explanations ---")
print("MSE:  Mean Squared Error - ลงโทษ errors ใหญ่มาก (squared)")
print("RMSE: Root MSE - อยู่ใน same unit กับ target")
print("MAE:  Mean Absolute Error - robust to outliers")
print("MAPE: Mean Abs Percentage Error - interpretable (%)")
print("R²:   Proportion of variance explained (0-1)")
print("Adj R²: R² adjusted for number of features")
```

---

## 12. Feature Engineering for Regression

```python
import pandas as pd
import numpy as np
from sklearn.preprocessing import PolynomialFeatures
from sklearn.pipeline import Pipeline
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import cross_val_score

# สร้าง dataset
np.random.seed(42)
n = 500
df = pd.DataFrame({
    'date': pd.date_range('2020-01-01', periods=n, freq='D'),
    'temperature': 20 + 10 * np.sin(np.arange(n) * 2*np.pi/365) + np.random.randn(n),
    'humidity': 60 + 20 * np.random.randn(n),
    'day_of_week': np.tile(range(7), n//7 + 1)[:n],
    'is_holiday': np.random.choice([0, 1], n, p=[0.9, 0.1])
})

# Feature Engineering
df['month'] = df['date'].dt.month
df['day_of_year'] = df['date'].dt.dayofyear
df['week_of_year'] = df['date'].dt.isocalendar().week.astype(int)
df['is_weekend'] = (df['day_of_week'] >= 5).astype(int)

# Cyclical features (สำหรับ periodic features)
df['month_sin'] = np.sin(2 * np.pi * df['month'] / 12)
df['month_cos'] = np.cos(2 * np.pi * df['month'] / 12)
df['day_sin'] = np.sin(2 * np.pi * df['day_of_year'] / 365)
df['day_cos'] = np.cos(2 * np.pi * df['day_of_year'] / 365)

# Interaction features
df['temp_humidity'] = df['temperature'] * df['humidity'] / 100
df['temp_squared'] = df['temperature'] ** 2

# Target
df['energy_consumption'] = (
    100 + 
    2 * df['temperature'] + 
    0.5 * df['humidity'] + 
    50 * df['is_weekend'] + 
    30 * df['is_holiday'] + 
    np.random.randn(n) * 5
)

feature_cols = ['temperature', 'humidity', 'month_sin', 'month_cos', 
                 'day_sin', 'day_cos', 'is_weekend', 'is_holiday',
                 'temp_humidity', 'temp_squared']

X = df[feature_cols].values
y = df['energy_consumption'].values

# Compare with/without engineered features
X_basic = df[['temperature', 'humidity']].values

from sklearn.preprocessing import StandardScaler
model = Pipeline([('scaler', StandardScaler()), ('lr', LinearRegression())])

basic_scores = cross_val_score(model, X_basic, y, cv=5, scoring='r2')
full_scores = cross_val_score(model, X, y, cv=5, scoring='r2')

print(f"Basic features (2): R² = {basic_scores.mean():.4f}")
print(f"Engineered features ({len(feature_cols)}): R² = {full_scores.mean():.4f}")
print(f"Improvement: +{(full_scores.mean() - basic_scores.mean()):.4f}")
```

---

## 13. House Price Prediction Project

```python
"""
โปรแกรมจริง: House Price Prediction
ใช้ California Housing Dataset
"""
import numpy as np
import pandas as pd
from sklearn.datasets import fetch_california_housing
from sklearn.model_selection import train_test_split, cross_val_score, RandomizedSearchCV
from sklearn.preprocessing import StandardScaler, PolynomialFeatures
from sklearn.pipeline import Pipeline
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor
from sklearn.linear_model import Ridge
from sklearn.metrics import mean_squared_error, r2_score, mean_absolute_error
import matplotlib.pyplot as plt
import warnings
warnings.filterwarnings('ignore')

# 1. โหลดข้อมูล
housing = fetch_california_housing()
X = pd.DataFrame(housing.data, columns=housing.feature_names)
y = pd.Series(housing.target, name='price')

print("=== California Housing Dataset ===")
print(f"Samples: {X.shape[0]}, Features: {X.shape[1]}")
print(f"\nFeatures: {list(X.columns)}")
print(f"\nTarget statistics:")
print(f"  Min: ${y.min()*100000:,.0f}")
print(f"  Max: ${y.max()*100000:,.0f}")
print(f"  Mean: ${y.mean()*100000:,.0f}")
print(f"\nData overview:")
print(X.describe().round(2))

# 2. Feature Engineering
X_eng = X.copy()
X_eng['rooms_per_household'] = X['AveRooms'] / X['HouseAge'].clip(lower=1)
X_eng['bedrooms_ratio'] = X['AveBedrms'] / X['AveRooms'].clip(lower=1)
X_eng['population_per_household'] = X['Population'] / X['AveOccup'].clip(lower=1)
X_eng['income_per_household'] = X['MedInc'] / X['AveOccup'].clip(lower=1)

# Log transform income (right-skewed)
X_eng['log_income'] = np.log1p(X['MedInc'])

print(f"\nAfter engineering: {X_eng.shape[1]} features")

# 3. Split
X_train, X_test, y_train, y_test = train_test_split(
    X_eng, y, test_size=0.2, random_state=42
)

# 4. Compare Models
models = {
    'Ridge': Pipeline([('scaler', StandardScaler()), ('model', Ridge(alpha=1.0))]),
    'Random Forest': RandomForestRegressor(n_estimators=100, random_state=42, n_jobs=-1),
    'Gradient Boosting': GradientBoostingRegressor(n_estimators=200, max_depth=5, 
                                                    learning_rate=0.1, random_state=42)
}

print("\n=== Model Comparison (5-fold CV) ===")
results = {}
for name, model in models.items():
    scores = cross_val_score(model, X_train, y_train, cv=5, 
                              scoring='neg_root_mean_squared_error', n_jobs=-1)
    rmse_scores = -scores
    results[name] = rmse_scores
    print(f"{name:20s}: RMSE = {rmse_scores.mean():.4f} (+/- {rmse_scores.std()*2:.4f})")

# 5. Final Evaluation  
print("\n=== Final Test Set Results ===")
for name, model in models.items():
    model.fit(X_train, y_train)
    y_pred = model.predict(X_test)
    rmse = np.sqrt(mean_squared_error(y_test, y_pred))
    r2 = r2_score(y_test, y_pred)
    mae = mean_absolute_error(y_test, y_pred)
    print(f"{name:20s}: RMSE={rmse:.4f}, R²={r2:.4f}, MAE={mae:.4f}")

# 6. Best Model Analysis
best_model = models['Gradient Boosting']
best_model.fit(X_train, y_train)
y_pred = best_model.predict(X_test)

# Prediction vs Actual plot
plt.figure(figsize=(10, 6))
plt.scatter(y_test, y_pred, alpha=0.3, color='blue')
plt.plot([y_test.min(), y_test.max()], [y_test.min(), y_test.max()], 'r--', lw=2)
plt.xlabel('Actual Price (x$100k)')
plt.ylabel('Predicted Price (x$100k)')
plt.title('Gradient Boosting - Actual vs Predicted')
plt.grid(True)
r2 = r2_score(y_test, y_pred)
plt.text(0.05, 0.95, f'R² = {r2:.4f}', transform=plt.gca().transAxes, 
         bbox=dict(boxstyle='round', facecolor='wheat'))
plt.savefig('house_price_prediction.png', dpi=100)
print("\nHouse price prediction plot saved!")

# Prediction example
sample_house = X_test.iloc[0]
actual_price = y_test.iloc[0] * 100000
predicted_price = best_model.predict(sample_house.values.reshape(1, -1))[0] * 100000
print(f"\nSample Prediction:")
print(f"  Actual: ${actual_price:,.0f}")
print(f"  Predicted: ${predicted_price:,.0f}")
print(f"  Difference: ${abs(actual_price - predicted_price):,.0f}")
```

---

## 14. Stock Price Prediction (Time Series Regression)

```python
"""
โปรแกรมจริง: Stock Price Prediction
ใช้ Technical Indicators เป็น Features
"""
import numpy as np
import pandas as pd
from sklearn.ensemble import GradientBoostingRegressor
from sklearn.model_selection import TimeSeriesSplit
from sklearn.metrics import mean_squared_error, r2_score
import warnings
warnings.filterwarnings('ignore')

# สร้าง simulated stock data
np.random.seed(42)
n_days = 1000
dates = pd.date_range('2020-01-01', periods=n_days)

# Simulate stock price with trends and seasonality
price = 100 * np.cumprod(1 + np.random.randn(n_days) * 0.02)
volume = np.abs(np.random.randn(n_days) * 1e6 + 5e6)

df = pd.DataFrame({'date': dates, 'close': price, 'volume': volume})

# Technical Indicators
def add_technical_indicators(df, n=df if False else None):
    """เพิ่ม technical indicators"""
    df = df.copy()
    
    # Moving Averages
    for window in [5, 10, 20, 50]:
        df[f'ma_{window}'] = df['close'].rolling(window).mean()
        df[f'ma_{window}_ratio'] = df['close'] / df[f'ma_{window}']
    
    # RSI
    delta = df['close'].diff()
    gain = delta.where(delta > 0, 0).rolling(14).mean()
    loss = (-delta.where(delta < 0, 0)).rolling(14).mean()
    rs = gain / loss
    df['rsi'] = 100 - (100 / (1 + rs))
    
    # MACD
    ema12 = df['close'].ewm(span=12).mean()
    ema26 = df['close'].ewm(span=26).mean()
    df['macd'] = ema12 - ema26
    df['macd_signal'] = df['macd'].ewm(span=9).mean()
    
    # Bollinger Bands
    ma20 = df['close'].rolling(20).mean()
    std20 = df['close'].rolling(20).std()
    df['bb_upper'] = ma20 + 2 * std20
    df['bb_lower'] = ma20 - 2 * std20
    df['bb_position'] = (df['close'] - df['bb_lower']) / (df['bb_upper'] - df['bb_lower'])
    
    # Returns
    for lag in [1, 2, 3, 5, 10]:
        df[f'return_{lag}d'] = df['close'].pct_change(lag)
    
    # Volatility
    df['volatility_10'] = df['close'].pct_change().rolling(10).std()
    df['volatility_20'] = df['close'].pct_change().rolling(20).std()
    
    # Volume
    df['volume_ratio'] = df['volume'] / df['volume'].rolling(20).mean()
    
    return df

df = add_technical_indicators(df)

# Target: next day return (1-day ahead prediction)
df['target'] = df['close'].shift(-1) / df['close'] - 1

# Drop NaN
df = df.dropna()
print(f"Dataset after feature engineering: {df.shape}")

# Features
feature_cols = [col for col in df.columns 
                if col not in ['date', 'close', 'volume', 'target']]

X = df[feature_cols].values
y = df['target'].values

# Time Series Split (ห้ามใช้ random split กับ time series!)
tscv = TimeSeriesSplit(n_splits=5)

print(f"\nFeatures used ({len(feature_cols)}):")
for col in feature_cols:
    print(f"  - {col}")

# Train with Time Series CV
model = GradientBoostingRegressor(
    n_estimators=200, max_depth=4, learning_rate=0.05,
    subsample=0.8, random_state=42
)

cv_rmse = []
for fold, (train_idx, val_idx) in enumerate(tscv.split(X)):
    X_train, X_val = X[train_idx], X[val_idx]
    y_train, y_val = y[train_idx], y[val_idx]
    
    model.fit(X_train, y_train)
    y_pred = model.predict(X_val)
    rmse = np.sqrt(mean_squared_error(y_val, y_pred))
    cv_rmse.append(rmse)
    print(f"Fold {fold+1}: RMSE = {rmse:.6f}")

print(f"\nMean RMSE: {np.mean(cv_rmse):.6f}")
print(f"Note: Predicting daily returns is difficult!")
print(f"Baseline (predict 0): RMSE = {np.sqrt(mean_squared_error(y, np.zeros_like(y))):.6f}")
```

---

## 15. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Linear Regression with Diagnostics

**โจทย์**: สร้าง multiple linear regression และทำการ residual analysis

```python
# เฉลย
import numpy as np
import pandas as pd
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error, r2_score
from sklearn.preprocessing import StandardScaler
import matplotlib.pyplot as plt
from scipy import stats

np.random.seed(42)
n = 300
X = pd.DataFrame({
    'x1': np.random.randn(n),
    'x2': np.random.randn(n) * 2,
    'x3': np.random.randn(n) * 0.5,
    'x4': np.random.randn(n)  # irrelevant
})
y = 3*X['x1'] + 2*X['x2'] - X['x3'] + np.random.randn(n) * 0.5

X_arr = X.values
X_train, X_test, y_train, y_test = train_test_split(X_arr, y, test_size=0.2, random_state=42)

model = LinearRegression()
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
residuals = y_test - y_pred

print("Coefficients:", dict(zip(X.columns, model.coef_)))
print(f"R²: {r2_score(y_test, y_pred):.4f}")

# Diagnostics
fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# 1. Residuals vs Fitted
axes[0,0].scatter(y_pred, residuals, alpha=0.5)
axes[0,0].axhline(y=0, color='r', linestyle='--')
axes[0,0].set_xlabel('Fitted')
axes[0,0].set_ylabel('Residuals')
axes[0,0].set_title('Residuals vs Fitted')

# 2. QQ Plot
stats.probplot(residuals, dist="norm", plot=axes[0,1])
axes[0,1].set_title('Normal Q-Q')

# 3. Scale-Location
axes[1,0].scatter(y_pred, np.sqrt(np.abs(residuals)), alpha=0.5)
axes[1,0].set_xlabel('Fitted')
axes[1,0].set_ylabel('√|Residuals|')
axes[1,0].set_title('Scale-Location')

# 4. Actual vs Predicted
axes[1,1].scatter(y_test, y_pred, alpha=0.5)
axes[1,1].plot([y_test.min(), y_test.max()], [y_test.min(), y_test.max()], 'r--')
axes[1,1].set_xlabel('Actual')
axes[1,1].set_ylabel('Predicted')
axes[1,1].set_title('Actual vs Predicted')

plt.tight_layout()
plt.savefig('regression_diagnostics.png', dpi=100)
print("Diagnostics saved!")
```

### แบบฝึกหัดที่ 2: Ridge vs Lasso vs ElasticNet

**โจทย์**: เปรียบเทียบ regularized regression บน high-dimensional dataset

```python
# เฉลย
import numpy as np
from sklearn.linear_model import RidgeCV, LassoCV, ElasticNetCV, LinearRegression
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
from sklearn.metrics import r2_score, mean_squared_error
import matplotlib.pyplot as plt

np.random.seed(42)
n, p = 200, 100
X = np.random.randn(n, p)
true_coef = np.zeros(p)
true_coef[:15] = np.random.randn(15) * 3
y = X.dot(true_coef) + np.random.randn(n)

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

models = {
    'Linear': LinearRegression(),
    'Ridge (CV)': RidgeCV(alphas=np.logspace(-3, 3, 100), cv=5),
    'Lasso (CV)': LassoCV(cv=5, max_iter=10000),
    'ElasticNet (CV)': ElasticNetCV(l1_ratio=[0.1, 0.5, 0.9], cv=5, max_iter=10000)
}

results = {}
for name, model in models.items():
    model.fit(X_train_s, y_train)
    y_pred = model.predict(X_test_s)
    r2 = r2_score(y_test, y_pred)
    rmse = np.sqrt(mean_squared_error(y_test, y_pred))
    coef = model.coef_
    nonzero = (np.abs(coef) > 0.01).sum()
    results[name] = {'r2': r2, 'rmse': rmse, 'nonzero': nonzero, 'coef': coef}
    
    alpha_str = f", alpha={model.alpha_:.4f}" if hasattr(model, 'alpha_') else ""
    print(f"{name:20s}: R²={r2:.4f}, RMSE={rmse:.4f}, Non-zero={nonzero}{alpha_str}")

# Coefficient Comparison
fig, axes = plt.subplots(2, 2, figsize=(14, 10))
for ax, (name, res) in zip(axes.flatten(), results.items()):
    bars = ax.bar(range(p), res['coef'], color=['red' if abs(c) > 0.01 else 'lightblue' 
                                                  for c in res['coef']])
    ax.plot(range(p), true_coef, 'go--', markersize=3, label='True coef')
    ax.set_title(f"{name} (R²={res['r2']:.4f})")
    ax.legend()
    ax.set_xlabel('Feature Index')

plt.suptitle('Coefficient Comparison: Regularized Regression')
plt.tight_layout()
plt.savefig('regularization_comparison.png', dpi=100)
print("Comparison saved!")
```

### แบบฝึกหัดที่ 3: Polynomial Regression with Regularization

```python
# เฉลย
import numpy as np
from sklearn.preprocessing import PolynomialFeatures, StandardScaler
from sklearn.linear_model import LinearRegression, Ridge
from sklearn.pipeline import Pipeline
from sklearn.model_selection import cross_val_score, train_test_split
from sklearn.metrics import r2_score
import matplotlib.pyplot as plt

np.random.seed(42)
n = 100
X = np.linspace(-3, 3, n).reshape(-1, 1)
y = 2*X.flatten()**3 - X.flatten()**2 + 0.5*X.flatten() + np.random.randn(n)

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

# Try different degrees with and without regularization
degrees = [2, 3, 5, 8, 12]
X_plot = np.linspace(-3.5, 3.5, 300).reshape(-1, 1)

fig, axes = plt.subplots(2, len(degrees), figsize=(20, 8))

for i, degree in enumerate(degrees):
    # Without regularization
    pipe_linear = Pipeline([
        ('poly', PolynomialFeatures(degree=degree)),
        ('scaler', StandardScaler()),
        ('lr', LinearRegression())
    ])
    pipe_linear.fit(X_train, y_train)
    
    # With Ridge regularization
    pipe_ridge = Pipeline([
        ('poly', PolynomialFeatures(degree=degree)),
        ('scaler', StandardScaler()),
        ('ridge', Ridge(alpha=1.0))
    ])
    pipe_ridge.fit(X_train, y_train)
    
    # CV Scores
    cv_linear = cross_val_score(pipe_linear, X, y, cv=5).mean()
    cv_ridge = cross_val_score(pipe_ridge, X, y, cv=5).mean()
    
    for ax, pipe, title in [(axes[0,i], pipe_linear, 'No Reg'),
                              (axes[1,i], pipe_ridge, 'Ridge')]:
        y_plot = pipe.predict(X_plot)
        r2 = r2_score(y_test, pipe.predict(X_test))
        ax.scatter(X_train, y_train, s=10, color='blue', alpha=0.5)
        ax.scatter(X_test, y_test, s=10, color='green', alpha=0.5)
        ax.plot(X_plot, y_plot, 'r-', lw=2)
        ax.set_ylim(-30, 30)
        ax.set_title(f"Deg={degree}, {title}\nR²={r2:.2f}")

plt.tight_layout()
plt.savefig('poly_ridge_comparison.png', dpi=100)
print("Plot saved!")
```

### แบบฝึกหัดที่ 4: Model Selection with Cross-Validation

```python
# เฉลย
from sklearn.datasets import fetch_california_housing
from sklearn.model_selection import cross_val_score, train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.linear_model import Ridge, Lasso, ElasticNet
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor
from sklearn.svm import SVR
import numpy as np

housing = fetch_california_housing()
X, y = housing.data, housing.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

models = {
    'Ridge': Pipeline([('s', StandardScaler()), ('m', Ridge())]),
    'Lasso': Pipeline([('s', StandardScaler()), ('m', Lasso(max_iter=5000))]),
    'ElasticNet': Pipeline([('s', StandardScaler()), ('m', ElasticNet(max_iter=5000))]),
    'SVR': Pipeline([('s', StandardScaler()), ('m', SVR(C=10))]),
    'Random Forest': RandomForestRegressor(n_estimators=100, random_state=42, n_jobs=-1),
    'Gradient Boosting': GradientBoostingRegressor(n_estimators=100, random_state=42)
}

print("Model Comparison (RMSE, 5-fold CV):")
print("="*55)
for name, model in models.items():
    scores = cross_val_score(model, X_train, y_train, cv=5, 
                              scoring='neg_root_mean_squared_error', n_jobs=-1)
    rmse = -scores
    print(f"{name:20s}: {rmse.mean():.4f} (+/- {rmse.std()*2:.4f})")
```

### แบบฝึกหัดที่ 5: XGBoost/GBM Hyperparameter Tuning

```python
# เฉลย
from sklearn.ensemble import GradientBoostingRegressor
from sklearn.model_selection import RandomizedSearchCV, train_test_split
from sklearn.metrics import mean_squared_error, r2_score
from sklearn.datasets import fetch_california_housing
from scipy.stats import randint, uniform
import numpy as np

housing = fetch_california_housing()
X, y = housing.data, housing.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

param_dist = {
    'n_estimators': randint(100, 500),
    'max_depth': randint(3, 10),
    'learning_rate': uniform(0.01, 0.2),
    'min_samples_leaf': randint(5, 50),
    'subsample': uniform(0.6, 0.4),
    'max_features': uniform(0.5, 0.5)
}

# Baseline
baseline = GradientBoostingRegressor(n_estimators=100, random_state=42)
baseline.fit(X_train, y_train)
baseline_rmse = np.sqrt(mean_squared_error(y_test, baseline.predict(X_test)))
print(f"Baseline RMSE: {baseline_rmse:.4f}")

# Tuned
random_search = RandomizedSearchCV(
    GradientBoostingRegressor(random_state=42),
    param_dist, n_iter=50, cv=5,
    scoring='neg_root_mean_squared_error',
    n_jobs=-1, random_state=42, verbose=1
)
random_search.fit(X_train, y_train)
best_rmse = np.sqrt(mean_squared_error(y_test, random_search.best_estimator_.predict(X_test)))
print(f"\nTuned RMSE: {best_rmse:.4f}")
print(f"Improvement: {(baseline_rmse - best_rmse)/baseline_rmse*100:.2f}%")
print(f"Best params: {random_search.best_params_}")
```

### แบบฝึกหัดที่ 6: Regression with Categorical Features

```python
# เฉลย
import pandas as pd
import numpy as np
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.pipeline import Pipeline
from sklearn.ensemble import GradientBoostingRegressor
from sklearn.model_selection import cross_val_score, train_test_split
from sklearn.metrics import r2_score, mean_squared_error

np.random.seed(42)
n = 1000

# Dataset: Car Price Prediction
df = pd.DataFrame({
    'brand': np.random.choice(['Toyota', 'Honda', 'BMW', 'Mercedes', 'Ford'], n),
    'year': np.random.randint(2000, 2024, n),
    'mileage': np.random.randint(0, 300000, n),
    'engine_size': np.random.choice([1.0, 1.6, 2.0, 2.5, 3.0, 4.0], n),
    'fuel_type': np.random.choice(['Petrol', 'Diesel', 'Electric', 'Hybrid'], n),
    'transmission': np.random.choice(['Manual', 'Automatic'], n)
})

brand_premiums = {'BMW': 50000, 'Mercedes': 60000, 'Toyota': 10000, 'Honda': 8000, 'Ford': 5000}
df['price'] = (
    df['brand'].map(brand_premiums) +
    2000 * (df['year'] - 2000) +
    -0.1 * df['mileage'] +
    10000 * df['engine_size'] +
    np.random.randn(n) * 5000
).clip(lower=1000)

X = df.drop('price', axis=1)
y = df['price']

numerical_cols = ['year', 'mileage', 'engine_size']
categorical_cols = ['brand', 'fuel_type', 'transmission']

preprocessor = ColumnTransformer([
    ('num', StandardScaler(), numerical_cols),
    ('cat', OneHotEncoder(drop='first'), categorical_cols)
])

pipeline = Pipeline([
    ('prep', preprocessor),
    ('model', GradientBoostingRegressor(n_estimators=200, random_state=42))
])

scores = cross_val_score(pipeline, X, y, cv=5, scoring='r2')
print(f"Car Price R²: {scores.mean():.4f} (+/- {scores.std()*2:.4f})")

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
pipeline.fit(X_train, y_train)
print(f"Test R²: {r2_score(y_test, pipeline.predict(X_test)):.4f}")
```

### แบบฝึกหัดที่ 7: Time Series Regression (Advanced)

```python
# เฉลย
import numpy as np
import pandas as pd
from sklearn.ensemble import GradientBoostingRegressor
from sklearn.model_selection import TimeSeriesSplit
from sklearn.metrics import mean_squared_error, r2_score
from sklearn.preprocessing import StandardScaler

np.random.seed(42)
n = 500
t = np.arange(n)
# Seasonal + trend + noise
y = (100 + 0.5*t + 
     20*np.sin(2*np.pi*t/12) + 
     10*np.sin(2*np.pi*t/52) + 
     np.random.randn(n) * 5)

def create_lag_features(y, lags=[1, 2, 3, 7, 14, 30]):
    df = pd.DataFrame({'y': y})
    for lag in lags:
        df[f'lag_{lag}'] = df['y'].shift(lag)
    for window in [7, 14, 30]:
        df[f'ma_{window}'] = df['y'].rolling(window).mean()
        df[f'std_{window}'] = df['y'].rolling(window).std()
    df['trend'] = np.arange(len(df))
    df['month'] = np.arange(len(df)) % 12
    df['month_sin'] = np.sin(2*np.pi*df['month']/12)
    df['month_cos'] = np.cos(2*np.pi*df['month']/12)
    return df.dropna()

df = create_lag_features(y)
X = df.drop('y', axis=1).values
y_target = df['y'].values

tscv = TimeSeriesSplit(n_splits=5)
model = GradientBoostingRegressor(n_estimators=200, max_depth=4, 
                                    learning_rate=0.05, random_state=42)

cv_rmse = []
for train_idx, val_idx in tscv.split(X):
    model.fit(X[train_idx], y_target[train_idx])
    y_pred = model.predict(X[val_idx])
    cv_rmse.append(np.sqrt(mean_squared_error(y_target[val_idx], y_pred)))

print(f"Time Series CV RMSE: {np.mean(cv_rmse):.4f} (+/- {np.std(cv_rmse)*2:.4f})")
```

### แบบฝึกหัดที่ 8: Ensemble Regression (Stacking)

```python
# เฉลย
from sklearn.ensemble import (RandomForestRegressor, GradientBoostingRegressor, 
                                StackingRegressor)
from sklearn.linear_model import Ridge
from sklearn.svm import SVR
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline, make_pipeline
from sklearn.model_selection import cross_val_score, train_test_split
from sklearn.datasets import fetch_california_housing
from sklearn.metrics import r2_score
import numpy as np

housing = fetch_california_housing()
X, y = housing.data, housing.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Base models
estimators = [
    ('rf', RandomForestRegressor(n_estimators=100, random_state=42, n_jobs=-1)),
    ('gb', GradientBoostingRegressor(n_estimators=100, random_state=42)),
    ('svr', make_pipeline(StandardScaler(), SVR(C=10)))
]

# Stacking
stacking = StackingRegressor(
    estimators=estimators,
    final_estimator=Ridge(alpha=1.0),
    cv=5,
    n_jobs=-1
)

# Compare
print("Individual Models:")
for name, model in estimators:
    if name == 'svr':
        scores = cross_val_score(model, X_train, y_train, cv=3, scoring='r2', n_jobs=-1)
    else:
        scores = cross_val_score(model, X_train, y_train, cv=5, scoring='r2', n_jobs=-1)
    print(f"  {name}: {scores.mean():.4f}")

print("Stacking (fitting on full train):")
stacking.fit(X_train, y_train)
y_pred = stacking.predict(X_test)
r2 = r2_score(y_test, y_pred)
print(f"  Stacking R²: {r2:.4f}")
```

---

## สรุปบทที่ 77

| Algorithm | ข้อดี | ข้อเสีย | เมื่อใช้ |
|-----------|-------|---------|---------|
| Linear Regression | ตีความง่าย, เร็ว | ต้องการ linear relationship | ข้อมูล linear, baseline |
| Ridge | ป้องกัน overfitting | ไม่ทำ feature selection | Multicollinearity |
| Lasso | Feature selection อัตโนมัติ | Sparse data | High-dim data |
| ElasticNet | Balance ระหว่าง Ridge/Lasso | มี 2 hyperparams | กรณีทั่วไป |
| Random Forest | Robust, handles non-linear | ช้า, memory-intensive | Large datasets |
| Gradient Boosting | Accuracy สูง | Overfitting ง่าย | Competitions |
| SVR | Kernel trick | ช้ากับ large data | Medium-size, non-linear |
| Decision Tree | Interpretable | Overfitting มาก | Baseline, explainability |

**Key Takeaways:**
1. เสมอ scale ข้อมูลก่อนใช้ Linear, SVR, KNN
2. Tree-based models ไม่ต้องการ scaling
3. L1 (Lasso) ทำ feature selection, L2 (Ridge) แค่ shrink coefficients
4. ใช้ TimeSeriesSplit กับ time series data เสมอ
5. Polynomial features ต้องระวัง overfitting -> ใช้ regularization
6. Gradient Boosting มักให้ผลดีที่สุด แต่ต้องการ tuning

**Part 78** จะเรียนเรื่อง Machine Learning Classification algorithms ในเชิงลึก
