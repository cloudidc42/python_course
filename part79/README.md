# Part 79: Machine Learning - Clustering & Dimensionality Reduction

## บทนำ

Unsupervised Learning คือการเรียนรู้จากข้อมูลที่ไม่มี label เป้าหมายคือค้นหาโครงสร้างที่ซ่อนอยู่ในข้อมูล บทนี้ครอบคลุม:
- Clustering algorithms (K-Means, DBSCAN, Hierarchical, GMM)
- Dimensionality Reduction (PCA, t-SNE, UMAP)
- Anomaly Detection
- Association Rules

---

## 1. Unsupervised Learning Overview

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.datasets import make_blobs, make_moons, make_circles

# สร้างข้อมูลตัวอย่างสำหรับ Clustering
X_blobs, y_blobs = make_blobs(n_samples=500, centers=4, cluster_std=0.8, random_state=42)
X_moons, y_moons = make_moons(n_samples=300, noise=0.05, random_state=42)
X_circles, y_circles = make_circles(n_samples=300, noise=0.05, factor=0.5, random_state=42)

# Visualize datasets
fig, axes = plt.subplots(1, 3, figsize=(15, 5))

axes[0].scatter(X_blobs[:, 0], X_blobs[:, 1], c=y_blobs, cmap='viridis', s=30)
axes[0].set_title('Blobs (Simple Clusters)')

axes[1].scatter(X_moons[:, 0], X_moons[:, 1], c=y_moons, cmap='viridis', s=30)
axes[1].set_title('Moons (Non-convex)')

axes[2].scatter(X_circles[:, 0], X_circles[:, 1], c=y_circles, cmap='viridis', s=30)
axes[2].set_title('Circles (Concentric)')

plt.suptitle('Clustering Datasets')
plt.tight_layout()
plt.savefig('clustering_datasets.png', dpi=100)
print("Datasets saved!")

# Unsupervised vs Supervised comparison
print("\nUnsupervised Learning Use Cases:")
print("  1. Customer Segmentation")
print("  2. Document/Topic Clustering")
print("  3. Image Compression")
print("  4. Anomaly Detection")
print("  5. Feature Engineering (cluster as feature)")
print("  6. Data Exploration and Visualization")
```

---

## 2. K-Means Clustering

K-Means แบ่งข้อมูลเป็น k กลุ่ม โดยลด inertia (sum of squared distances to cluster centers)

### 2.1 Basic K-Means

```python
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import silhouette_score, adjusted_rand_score
import numpy as np
import matplotlib.pyplot as plt

# สร้างข้อมูล
from sklearn.datasets import make_blobs
X, y_true = make_blobs(n_samples=500, centers=4, cluster_std=0.8, random_state=42)

# Fit K-Means
kmeans = KMeans(n_clusters=4, random_state=42, n_init=10)
y_pred = kmeans.fit_predict(X)

print("K-Means Results:")
print(f"  Inertia: {kmeans.inertia_:.2f}")
print(f"  Iterations: {kmeans.n_iter_}")
print(f"  Cluster centers:\n{kmeans.cluster_centers_.round(2)}")
print(f"  Cluster sizes: {np.bincount(y_pred)}")

# Evaluate
silhouette = silhouette_score(X, y_pred)
ari = adjusted_rand_score(y_true, y_pred)
print(f"\n  Silhouette Score: {silhouette:.4f} (higher is better, max=1)")
print(f"  Adjusted Rand Index: {ari:.4f} (vs true labels)")

# Visualize
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# True labels
axes[0].scatter(X[:, 0], X[:, 1], c=y_true, cmap='viridis', s=30)
axes[0].set_title('True Labels')

# K-Means predictions
axes[1].scatter(X[:, 0], X[:, 1], c=y_pred, cmap='viridis', s=30)
axes[1].scatter(kmeans.cluster_centers_[:, 0], kmeans.cluster_centers_[:, 1],
                 marker='*', s=300, c='red', label='Centroids')
axes[1].set_title(f'K-Means (k=4)\nSilhouette={silhouette:.4f}')
axes[1].legend()

plt.tight_layout()
plt.savefig('kmeans_basic.png', dpi=100)
print("K-Means plot saved!")
```

### 2.2 Elbow Method (หา optimal k)

```python
from sklearn.cluster import KMeans
import matplotlib.pyplot as plt
import numpy as np

X, _ = make_blobs(n_samples=500, centers=4, cluster_std=0.8, random_state=42)

# Inertia vs k
inertias = []
silhouettes = []
K_range = range(2, 15)

for k in K_range:
    kmeans = KMeans(n_clusters=k, random_state=42, n_init=10)
    labels = kmeans.fit_predict(X)
    inertias.append(kmeans.inertia_)
    silhouettes.append(silhouette_score(X, labels))

# Plot Elbow and Silhouette
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Elbow
axes[0].plot(K_range, inertias, 'o-', color='blue')
axes[0].set_xlabel('Number of Clusters (k)')
axes[0].set_ylabel('Inertia')
axes[0].set_title('Elbow Method')
axes[0].grid(True)

# Silhouette
axes[1].plot(K_range, silhouettes, 'o-', color='green')
axes[1].axvline(x=K_range[np.argmax(silhouettes)], color='red', linestyle='--',
                 label=f'Best k={K_range[np.argmax(silhouettes)]}')
axes[1].set_xlabel('Number of Clusters (k)')
axes[1].set_ylabel('Silhouette Score')
axes[1].set_title('Silhouette Scores')
axes[1].legend()
axes[1].grid(True)

plt.tight_layout()
plt.savefig('kmeans_optimal_k.png', dpi=100)
print(f"Optimal k (Silhouette): {K_range[np.argmax(silhouettes)]}")
print(f"Best Silhouette Score: {max(silhouettes):.4f}")
```

### 2.3 K-Means++ Initialization

```python
import time
from sklearn.cluster import KMeans

X, _ = make_blobs(n_samples=1000, centers=10, random_state=42)

# Compare k-means vs k-means++
for init in ['random', 'k-means++']:
    start = time.time()
    km = KMeans(n_clusters=10, init=init, n_init=10, random_state=42)
    km.fit(X)
    elapsed = time.time() - start
    silhouette = silhouette_score(X, km.labels_)
    
    print(f"{init:12s}: Inertia={km.inertia_:.2f}, Silhouette={silhouette:.4f}, "
          f"Iter={km.n_iter_}, Time={elapsed:.2f}s")

# K-Means limitations
print("\nK-Means Limitations:")
print("  1. Must specify k beforehand")
print("  2. Sensitive to initialization")
print("  3. Assumes spherical clusters of similar sizes")
print("  4. Sensitive to outliers")
print("  5. Doesn't work well with non-convex clusters")
```

---

## 3. Hierarchical Clustering

Hierarchical Clustering สร้าง tree structure (dendrogram) โดยไม่ต้องกำหนด k ล่วงหน้า

```python
from sklearn.cluster import AgglomerativeClustering
from scipy.cluster.hierarchy import dendrogram, linkage
import matplotlib.pyplot as plt
import numpy as np

X, y_true = make_blobs(n_samples=200, centers=4, cluster_std=0.8, random_state=42)

# Agglomerative Clustering
agg_ward = AgglomerativeClustering(n_clusters=4, linkage='ward')
y_ward = agg_ward.fit_predict(X)

agg_complete = AgglomerativeClustering(n_clusters=4, linkage='complete')
y_complete = agg_complete.fit_predict(X)

agg_single = AgglomerativeClustering(n_clusters=4, linkage='single')
y_single = agg_single.fit_predict(X)

# Compare linkage methods
print("Hierarchical Clustering - Linkage Methods:")
for name, labels in [('Ward', y_ward), ('Complete', y_complete), ('Single', y_single)]:
    sil = silhouette_score(X, labels)
    ari = adjusted_rand_score(y_true, labels)
    print(f"  {name:10s}: Silhouette={sil:.4f}, ARI={ari:.4f}")

# Dendrogram
from sklearn.preprocessing import StandardScaler
X_scaled = StandardScaler().fit_transform(X[:50])  # ใช้แค่ 50 samples สำหรับ dendrogram

Z = linkage(X_scaled, method='ward')

plt.figure(figsize=(15, 5))
dendrogram(Z, truncate_mode='lastp', p=20, leaf_rotation=45, leaf_font_size=10)
plt.title('Dendrogram (Ward Linkage)')
plt.xlabel('Sample Index')
plt.ylabel('Distance')
plt.tight_layout()
plt.savefig('dendrogram.png', dpi=100)
print("Dendrogram saved!")
```

```python
# Visualize different linkage methods
fig, axes = plt.subplots(1, 3, figsize=(15, 5))

for ax, (name, labels) in zip(axes, [('Ward', y_ward), ('Complete', y_complete), ('Single', y_single)]):
    ax.scatter(X[:, 0], X[:, 1], c=labels, cmap='viridis', s=30)
    sil = silhouette_score(X, labels)
    ax.set_title(f'{name}\nSilhouette={sil:.4f}')
    ax.grid(True, alpha=0.3)

plt.suptitle('Hierarchical Clustering - Linkage Methods')
plt.tight_layout()
plt.savefig('hierarchical_linkages.png', dpi=100)
print("Comparison saved!")
```

---

## 4. DBSCAN

DBSCAN (Density-Based Spatial Clustering of Applications with Noise) ค้นหา clusters ตาม density สามารถค้นหา arbitrary-shaped clusters และระบุ noise points

```python
from sklearn.cluster import DBSCAN
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import make_moons, make_blobs
import numpy as np
import matplotlib.pyplot as plt

# DBSCAN ดีกับ non-convex clusters
X_moons, _ = make_moons(n_samples=400, noise=0.05, random_state=42)
X_scaled = StandardScaler().fit_transform(X_moons)

# Basic DBSCAN
dbscan = DBSCAN(eps=0.3, min_samples=5)
y_dbscan = dbscan.fit_predict(X_scaled)

# -1 คือ noise points
n_clusters = len(set(y_dbscan)) - (1 if -1 in y_dbscan else 0)
n_noise = (y_dbscan == -1).sum()

print("DBSCAN Results:")
print(f"  Clusters found: {n_clusters}")
print(f"  Noise points: {n_noise}")
print(f"  Cluster distribution: {dict(zip(*np.unique(y_dbscan, return_counts=True)))}")

# Effect of eps
eps_values = [0.1, 0.2, 0.3, 0.5, 1.0]
fig, axes = plt.subplots(1, len(eps_values), figsize=(20, 4))

for ax, eps in zip(axes, eps_values):
    db = DBSCAN(eps=eps, min_samples=5)
    labels = db.fit_predict(X_scaled)
    n_clust = len(set(labels)) - (1 if -1 in labels else 0)
    n_nois = (labels == -1).sum()
    
    # Color noise points red
    colors = ['red' if l == -1 else plt.cm.viridis(l / max(labels.max(), 1)) 
               for l in labels]
    ax.scatter(X_scaled[:, 0], X_scaled[:, 1], c=colors, s=20)
    ax.set_title(f'eps={eps}\n{n_clust} clusters, {n_nois} noise')
    ax.grid(True, alpha=0.3)

plt.suptitle('DBSCAN - Effect of eps (min_samples=5)')
plt.tight_layout()
plt.savefig('dbscan_eps.png', dpi=100)
print("DBSCAN eps comparison saved!")

# DBSCAN สำหรับ Anomaly Detection
print("\nDBSCAN for Anomaly Detection:")
X_normal = np.random.randn(200, 2)
X_outliers = np.random.uniform(-4, 4, (10, 2))
X_combined = np.vstack([X_normal, X_outliers])
X_combined_scaled = StandardScaler().fit_transform(X_combined)

db_anomaly = DBSCAN(eps=0.5, min_samples=5)
labels = db_anomaly.fit_predict(X_combined_scaled)
print(f"  Normal points: {(labels != -1).sum()}")
print(f"  Anomalies detected: {(labels == -1).sum()} (true: {len(X_outliers)})")
```

---

## 5. Gaussian Mixture Models (GMM)

GMM เป็น probabilistic model ที่สมมติว่าข้อมูลมาจากหลาย Gaussian distributions

```python
from sklearn.mixture import GaussianMixture
from sklearn.datasets import make_blobs
import numpy as np
import matplotlib.pyplot as plt

X, y_true = make_blobs(n_samples=500, centers=3, cluster_std=[1.0, 1.5, 0.5], random_state=42)

# Fit GMM
gmm = GaussianMixture(n_components=3, covariance_type='full', random_state=42, n_init=10)
gmm.fit(X)
y_pred = gmm.predict(X)
y_prob = gmm.predict_proba(X)  # Soft assignments

print("GMM Results:")
print(f"  Converged: {gmm.converged_}")
print(f"  Iterations: {gmm.n_iter_}")
print(f"  Log-likelihood: {gmm.lower_bound_:.4f}")
print(f"\nCluster means:\n{gmm.means_.round(2)}")
print(f"\nMixing weights: {gmm.weights_.round(4)}")

# BIC/AIC for model selection
bic_scores = []
aic_scores = []
n_components_range = range(1, 10)

for n in n_components_range:
    gmm_temp = GaussianMixture(n_components=n, random_state=42)
    gmm_temp.fit(X)
    bic_scores.append(gmm_temp.bic(X))
    aic_scores.append(gmm_temp.aic(X))

best_bic = n_components_range[np.argmin(bic_scores)]
best_aic = n_components_range[np.argmin(aic_scores)]
print(f"\nBIC optimal components: {best_bic}")
print(f"AIC optimal components: {best_aic}")

# Plot BIC/AIC
plt.figure(figsize=(10, 5))
plt.plot(n_components_range, bic_scores, 'o-', label='BIC', color='blue')
plt.plot(n_components_range, aic_scores, 'o-', label='AIC', color='red')
plt.axvline(x=best_bic, color='blue', linestyle='--', alpha=0.5)
plt.xlabel('Number of Components')
plt.ylabel('Score (lower is better)')
plt.title('GMM - BIC/AIC Model Selection')
plt.legend()
plt.grid(True)
plt.savefig('gmm_model_selection.png', dpi=100)
print("GMM selection saved!")

# Covariance types
cov_types = ['spherical', 'tied', 'diag', 'full']
print("\nCovariance Type Comparison:")
for cov_type in cov_types:
    gmm_type = GaussianMixture(n_components=3, covariance_type=cov_type, random_state=42)
    gmm_type.fit(X)
    y_pred_type = gmm_type.predict(X)
    sil = silhouette_score(X, y_pred_type)
    print(f"  {cov_type:10s}: Silhouette={sil:.4f}, BIC={gmm_type.bic(X):.2f}")
```

---

## 6. PCA (Principal Component Analysis)

PCA ลดมิติข้อมูลโดยหา orthogonal components ที่ capture variance สูงสุด

```python
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import load_iris, load_digits
import numpy as np
import matplotlib.pyplot as plt

# ตัวอย่าง: Iris dataset
iris = load_iris()
X, y = iris.data, iris.target

# Scale ก่อน PCA เสมอ
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# PCA
pca = PCA(n_components=4)  # keep all components
pca.fit(X_scaled)

print("PCA Results - Iris:")
print(f"  Original shape: {X_scaled.shape}")
print(f"\nVariance explained by each PC:")
for i, (var, cumvar) in enumerate(zip(pca.explained_variance_ratio_,
                                         np.cumsum(pca.explained_variance_ratio_))):
    print(f"  PC{i+1}: {var:.4f} ({cumvar:.4f} cumulative)")

# 2D Projection
pca_2d = PCA(n_components=2)
X_pca = pca_2d.fit_transform(X_scaled)

plt.figure(figsize=(8, 6))
for i, name in enumerate(iris.target_names):
    mask = y == i
    plt.scatter(X_pca[mask, 0], X_pca[mask, 1], label=name, s=50)
plt.xlabel(f'PC1 ({pca_2d.explained_variance_ratio_[0]*100:.1f}% variance)')
plt.ylabel(f'PC2 ({pca_2d.explained_variance_ratio_[1]*100:.1f}% variance)')
plt.title('PCA - Iris Dataset (2D)')
plt.legend()
plt.grid(True, alpha=0.3)
plt.savefig('pca_iris.png', dpi=100)
print("PCA iris saved!")
```

```python
# PCA component loadings
pca_full = PCA(n_components=4)
pca_full.fit(X_scaled)

loadings = pca_full.components_
print("\nPCA Loadings (contribution of each feature to each PC):")
import pandas as pd
loadings_df = pd.DataFrame(
    loadings, 
    columns=iris.feature_names,
    index=[f'PC{i+1}' for i in range(4)]
)
print(loadings_df.round(4))

# Scree Plot
plt.figure(figsize=(10, 4))
plt.subplot(1, 2, 1)
plt.bar(range(1, 5), pca_full.explained_variance_ratio_, color='steelblue')
plt.xlabel('Principal Component')
plt.ylabel('Explained Variance Ratio')
plt.title('Scree Plot')

plt.subplot(1, 2, 2)
plt.plot(range(1, 5), np.cumsum(pca_full.explained_variance_ratio_), 'o-', color='green')
plt.axhline(y=0.95, color='red', linestyle='--', label='95% threshold')
plt.xlabel('Number of Components')
plt.ylabel('Cumulative Explained Variance')
plt.title('Cumulative Variance')
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.savefig('pca_scree.png', dpi=100)
print("Scree plot saved!")
```

```python
# PCA สำหรับ high-dimensional data (Digits)
from sklearn.datasets import load_digits

digits = load_digits()
X_digits, y_digits = digits.data, digits.target

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X_digits)

# Find n_components for 95% variance
pca_full = PCA()
pca_full.fit(X_scaled)
cumvar = np.cumsum(pca_full.explained_variance_ratio_)
n_95 = np.argmax(cumvar >= 0.95) + 1
print(f"\nDigits: {X_digits.shape[1]} features -> {n_95} components for 95% variance")

# PCA for compression
pca_compressed = PCA(n_components=n_95)
X_compressed = pca_compressed.fit_transform(X_scaled)
X_reconstructed = pca_compressed.inverse_transform(X_compressed)
X_reconstructed = scaler.inverse_transform(X_reconstructed)

print(f"Compression ratio: {X_digits.shape[1]/n_95:.1f}x")

# Reconstruction quality
from sklearn.metrics import mean_squared_error
mse = mean_squared_error(X_digits, X_reconstructed)
print(f"Reconstruction MSE: {mse:.4f}")
```

---

## 7. t-SNE (t-Distributed Stochastic Neighbor Embedding)

t-SNE ลดมิติเพื่อ visualization โดยรักษาความสัมพันธ์ local neighborhood

```python
from sklearn.manifold import TSNE
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import load_digits
import numpy as np
import matplotlib.pyplot as plt
import time

digits = load_digits()
X, y = digits.data, digits.target

# Scale ก่อน
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# t-SNE
print("Running t-SNE (this takes a few minutes)...")
start = time.time()

tsne = TSNE(
    n_components=2,
    perplexity=30,     # balance local/global structure (5-50)
    n_iter=1000,        # number of iterations
    learning_rate='auto',
    random_state=42,
    n_jobs=-1
)
X_tsne = tsne.fit_transform(X_scaled)
elapsed = time.time() - start
print(f"t-SNE completed in {elapsed:.1f}s")

# Plot
plt.figure(figsize=(12, 8))
scatter = plt.scatter(X_tsne[:, 0], X_tsne[:, 1], c=y, cmap='tab10', s=20, alpha=0.7)
plt.colorbar(scatter, label='Digit')
plt.title('t-SNE - Digits Dataset')
plt.xlabel('t-SNE 1')
plt.ylabel('t-SNE 2')
plt.savefig('tsne_digits.png', dpi=100)
print("t-SNE saved!")

# Effect of perplexity
perplexities = [5, 30, 50]
fig, axes = plt.subplots(1, 3, figsize=(18, 5))

for ax, perp in zip(axes, perplexities):
    tsne = TSNE(n_components=2, perplexity=perp, random_state=42, n_iter=500)
    X_tsne = tsne.fit_transform(X_scaled[:500])  # 使用 500 samples
    ax.scatter(X_tsne[:, 0], X_tsne[:, 1], c=y[:500], cmap='tab10', s=20)
    ax.set_title(f'Perplexity={perp}')

plt.suptitle('t-SNE - Effect of Perplexity')
plt.tight_layout()
plt.savefig('tsne_perplexity.png', dpi=100)
print("Perplexity comparison saved!")
```

### 7.1 PCA + t-SNE (Common Practice)

```python
# ใช้ PCA ลดมิติก่อน แล้วใช้ t-SNE
# ช่วยให้ t-SNE เร็วขึ้นและให้ผลดีขึ้น

from sklearn.decomposition import PCA
from sklearn.manifold import TSNE
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import load_digits
import time

digits = load_digits()
X, y = digits.data, digits.target

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# PCA ลดเหลือ 50 dimensions ก่อน
pca = PCA(n_components=50, random_state=42)
X_pca = pca.fit_transform(X_scaled)

print(f"After PCA: {X_pca.shape}")
print(f"Variance retained: {pca.explained_variance_ratio_.sum():.4f}")

# t-SNE on PCA output
start = time.time()
tsne = TSNE(n_components=2, perplexity=30, random_state=42, n_jobs=-1)
X_combined = tsne.fit_transform(X_pca)
print(f"PCA + t-SNE time: {time.time() - start:.1f}s")

import matplotlib.pyplot as plt
plt.figure(figsize=(10, 8))
scatter = plt.scatter(X_combined[:, 0], X_combined[:, 1], c=y, cmap='tab10', s=20)
plt.colorbar(scatter)
plt.title('PCA(50) + t-SNE - Digits')
plt.savefig('pca_tsne.png', dpi=100)
print("PCA+t-SNE saved!")
```

---

## 8. UMAP

UMAP (Uniform Manifold Approximation and Projection) เร็วกว่า t-SNE และให้ผลดีกว่าในหลายกรณี

```python
try:
    import umap
    from sklearn.preprocessing import StandardScaler
    from sklearn.datasets import load_digits
    import matplotlib.pyplot as plt
    import time
    
    digits = load_digits()
    X, y = digits.data, digits.target
    
    scaler = StandardScaler()
    X_scaled = scaler.fit_transform(X)
    
    # UMAP
    print("Running UMAP...")
    start = time.time()
    reducer = umap.UMAP(
        n_components=2,
        n_neighbors=15,    # local neighborhood size
        min_dist=0.1,      # minimum distance in embedded space
        random_state=42
    )
    X_umap = reducer.fit_transform(X_scaled)
    print(f"UMAP completed in {time.time() - start:.1f}s")
    
    # Plot
    plt.figure(figsize=(10, 8))
    scatter = plt.scatter(X_umap[:, 0], X_umap[:, 1], c=y, cmap='tab10', s=20)
    plt.colorbar(scatter, label='Digit')
    plt.title('UMAP - Digits Dataset')
    plt.savefig('umap_digits.png', dpi=100)
    print("UMAP saved!")
    
    # Compare t-SNE vs UMAP
    from sklearn.manifold import TSNE
    tsne = TSNE(n_components=2, random_state=42, n_jobs=-1)
    
    start_tsne = time.time()
    X_tsne = tsne.fit_transform(X_scaled)
    time_tsne = time.time() - start_tsne
    
    start_umap = time.time()
    X_umap = reducer.fit_transform(X_scaled)
    time_umap = time.time() - start_umap
    
    print(f"\nt-SNE time: {time_tsne:.1f}s")
    print(f"UMAP time: {time_umap:.1f}s")
    print(f"Speed improvement: {time_tsne/time_umap:.1f}x")
    
except ImportError:
    print("UMAP not installed. Run: pip install umap-learn")
    print("t-SNE is used as alternative (already demonstrated above)")
```

---

## 9. Anomaly Detection

### 9.1 Isolation Forest

```python
from sklearn.ensemble import IsolationForest
from sklearn.preprocessing import StandardScaler
import numpy as np
import matplotlib.pyplot as plt

# สร้างข้อมูลที่มี anomalies
np.random.seed(42)
n_normal = 400
n_outliers = 20

X_normal = np.random.randn(n_normal, 2)
X_outliers = np.random.uniform(-4, 4, (n_outliers, 2))
X = np.vstack([X_normal, X_outliers])
y_true = np.hstack([np.ones(n_normal), -np.ones(n_outliers)])

# Isolation Forest
iso_forest = IsolationForest(
    n_estimators=100,
    contamination=0.05,  # expected proportion of outliers
    random_state=42
)
y_pred = iso_forest.fit_predict(X)
# 1 = normal, -1 = anomaly

# Evaluate
true_positives = ((y_pred == -1) & (y_true == -1)).sum()
false_positives = ((y_pred == -1) & (y_true == 1)).sum()
print(f"Isolation Forest:")
print(f"  Anomalies detected: {(y_pred == -1).sum()}")
print(f"  True positives: {true_positives}/{n_outliers}")
print(f"  False positives: {false_positives}")

# Plot
plt.figure(figsize=(12, 5))
plt.subplot(1, 2, 1)
plt.scatter(X[:, 0], X[:, 1], c=y_true, cmap='RdYlGn', s=30)
plt.title('True Labels (green=normal, red=anomaly)')

plt.subplot(1, 2, 2)
plt.scatter(X[:, 0], X[:, 1], c=y_pred, cmap='RdYlGn', s=30)
plt.title('Isolation Forest Predictions')

plt.tight_layout()
plt.savefig('isolation_forest.png', dpi=100)
print("Isolation Forest plot saved!")

# Anomaly scores
anomaly_scores = iso_forest.decision_function(X)
print(f"\nAnomaly scores range: [{anomaly_scores.min():.4f}, {anomaly_scores.max():.4f}]")
print(f"Negative scores = anomalies, positive = normal")
```

### 9.2 Local Outlier Factor (LOF)

```python
from sklearn.neighbors import LocalOutlierFactor
import numpy as np
import matplotlib.pyplot as plt

np.random.seed(42)
X_normal = np.random.randn(300, 2)
X_outliers = np.array([[4, 4], [-4, -4], [4, -4], [-4, 4], [0, 5]])
X = np.vstack([X_normal, X_outliers])
y_true = np.hstack([np.ones(300), -np.ones(5)])

# LOF
lof = LocalOutlierFactor(n_neighbors=20, contamination=0.02)
y_pred = lof.fit_predict(X)
# 1 = inlier, -1 = outlier

print("Local Outlier Factor:")
print(f"  Outliers detected: {(y_pred == -1).sum()}")
print(f"  True positives: {((y_pred == -1) & (y_true == -1)).sum()}/5")

# LOF scores
lof_scores = lof.negative_outlier_factor_
print(f"  Score range: [{lof_scores.min():.4f}, {lof_scores.max():.4f}]")
print(f"  More negative = more outlier")

# One-Class SVM
from sklearn.svm import OneClassSVM
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

oc_svm = OneClassSVM(kernel='rbf', nu=0.05, gamma='scale')
y_svm = oc_svm.fit_predict(X_scaled)

print(f"\nOne-Class SVM:")
print(f"  Outliers detected: {(y_svm == -1).sum()}")
print(f"  True positives: {((y_svm == -1) & (y_true == -1)).sum()}/5")
```

### 9.3 Autoencoder-based Anomaly Detection (with sklearn)

```python
from sklearn.neural_network import MLPRegressor
from sklearn.preprocessing import StandardScaler
import numpy as np

# Autoencoder: เรียน reconstruct normal data
# ข้อมูล anomaly จะมี reconstruction error สูง

np.random.seed(42)
X_normal = np.random.randn(1000, 10)

scaler = StandardScaler()
X_train = scaler.fit_transform(X_normal)

# Train on normal data only
autoencoder = MLPRegressor(
    hidden_layer_sizes=(8, 4, 8),  # bottleneck architecture
    activation='relu',
    max_iter=500,
    random_state=42
)
autoencoder.fit(X_train, X_train)  # input = target (reconstruction)

# Test with anomalies
X_normal_test = np.random.randn(100, 10)
X_anomaly = np.random.randn(20, 10) * 5  # anomalies

X_all = np.vstack([X_normal_test, X_anomaly])
y_true = np.hstack([np.ones(100), -np.ones(20)])

X_all_scaled = scaler.transform(X_all)

# Reconstruction error
X_reconstructed = autoencoder.predict(X_all_scaled)
reconstruction_errors = np.mean((X_all_scaled - X_reconstructed) ** 2, axis=1)

threshold = np.percentile(reconstruction_errors[:100], 95)
y_pred = np.where(reconstruction_errors > threshold, -1, 1)

print("Autoencoder Anomaly Detection:")
print(f"  Threshold: {threshold:.4f}")
print(f"  Detected anomalies: {(y_pred == -1).sum()}")
print(f"  True positives: {((y_pred == -1) & (y_true == -1)).sum()}/20")
```

---

## 10. Association Rules (Apriori)

```python
try:
    from mlxtend.frequent_patterns import apriori, association_rules
    from mlxtend.preprocessing import TransactionEncoder
    import pandas as pd
    
    # Supermarket Transaction Data
    transactions = [
        ['bread', 'milk', 'eggs'],
        ['bread', 'butter'],
        ['milk', 'eggs', 'cheese'],
        ['bread', 'milk', 'butter', 'eggs'],
        ['bread', 'milk'],
        ['butter', 'cheese'],
        ['milk', 'eggs'],
        ['bread', 'milk', 'eggs', 'butter'],
        ['bread', 'eggs'],
        ['milk', 'cheese', 'eggs'],
        ['bread', 'milk', 'cheese'],
        ['butter', 'milk'],
        ['bread', 'butter', 'cheese'],
        ['milk', 'eggs', 'bread'],
        ['cheese', 'eggs', 'butter']
    ] * 20  # repeat to get more data
    
    # Encode transactions
    te = TransactionEncoder()
    te_array = te.fit_transform(transactions)
    df_trans = pd.DataFrame(te_array, columns=te.columns_)
    
    print(f"Transactions: {len(df_trans)}")
    print(f"Items: {list(te.columns_)}")
    print(f"\nItem frequencies:")
    print(df_trans.sum().sort_values(ascending=False))
    
    # Apriori - find frequent itemsets
    frequent_itemsets = apriori(df_trans, min_support=0.3, use_colnames=True)
    print(f"\nFrequent itemsets (min_support=0.3):")
    print(frequent_itemsets.sort_values('support', ascending=False).head(10))
    
    # Association Rules
    rules = association_rules(frequent_itemsets, metric='confidence', min_threshold=0.6)
    rules['lift'] = rules['lift'].round(4)
    rules['confidence'] = rules['confidence'].round(4)
    rules['support'] = rules['support'].round(4)
    
    print(f"\nAssociation Rules (confidence >= 0.6):")
    print(rules[['antecedents', 'consequents', 'support', 'confidence', 'lift']]
           .sort_values('lift', ascending=False).head(10))
    
    # Interpret rules
    print("\nTop rule interpretation:")
    top_rule = rules.sort_values('lift', ascending=False).iloc[0]
    print(f"  If customer buys: {list(top_rule['antecedents'])}")
    print(f"  Then likely to buy: {list(top_rule['consequents'])}")
    print(f"  Confidence: {top_rule['confidence']:.2%}")
    print(f"  Lift: {top_rule['lift']:.2f} (>1 means positive association)")

except ImportError:
    print("mlxtend not installed. Run: pip install mlxtend")
    
    # Manual Association Rules
    from itertools import combinations
    
    transactions = [
        ['bread', 'milk'], ['bread', 'butter'], ['milk', 'eggs'],
        ['bread', 'milk', 'butter'], ['milk', 'eggs', 'cheese']
    ]
    
    # Count item frequencies
    from collections import Counter
    item_counts = Counter()
    for trans in transactions:
        for item in trans:
            item_counts[item] += 1
    
    n_trans = len(transactions)
    print("Item frequencies:")
    for item, count in sorted(item_counts.items(), key=lambda x: x[1], reverse=True):
        print(f"  {item}: {count}/{n_trans} ({count/n_trans:.2%})")
```

---

## 11. Evaluation Metrics for Clustering

### 11.1 Silhouette Score

```python
from sklearn.metrics import silhouette_score, silhouette_samples
from sklearn.cluster import KMeans
import numpy as np
import matplotlib.pyplot as plt

X, y_true = make_blobs(n_samples=300, centers=4, cluster_std=0.8, random_state=42)

# Silhouette score
kmeans = KMeans(n_clusters=4, random_state=42)
labels = kmeans.fit_predict(X)
sil_score = silhouette_score(X, labels)
sil_samples = silhouette_samples(X, labels)

print(f"Overall Silhouette Score: {sil_score:.4f}")
print("\nPer-cluster silhouette scores:")
for cluster in range(4):
    cluster_sil = sil_samples[labels == cluster].mean()
    print(f"  Cluster {cluster}: {cluster_sil:.4f}")

# Silhouette diagram
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 5))

# Silhouette Plot
y_lower = 10
colors = plt.cm.nipy_spectral(np.linspace(0, 1, 4))

for i, color in enumerate(colors):
    ith_cluster_sil = sil_samples[labels == i]
    ith_cluster_sil.sort()
    size = ith_cluster_sil.shape[0]
    y_upper = y_lower + size
    ax1.fill_betweenx(np.arange(y_lower, y_upper), 0, ith_cluster_sil, 
                        facecolor=color, edgecolor=color, alpha=0.7)
    ax1.text(-0.05, y_lower + 0.5 * size, str(i))
    y_lower = y_upper + 10

ax1.axvline(x=sil_score, color='red', linestyle='--', label=f'Mean={sil_score:.4f}')
ax1.set_title('Silhouette Plot')
ax1.set_xlabel('Silhouette Coefficient')
ax1.legend()

# Data visualization
ax2.scatter(X[:, 0], X[:, 1], c=labels, cmap='nipy_spectral', s=30)
ax2.scatter(kmeans.cluster_centers_[:, 0], kmeans.cluster_centers_[:, 1], 
             marker='*', s=300, c='red')
ax2.set_title('K-Means Clustering')

plt.tight_layout()
plt.savefig('silhouette_analysis.png', dpi=100)
print("Silhouette analysis saved!")
```

### 11.2 Calinski-Harabasz and Davies-Bouldin

```python
from sklearn.metrics import calinski_harabasz_score, davies_bouldin_score
from sklearn.cluster import KMeans
import numpy as np

X, y_true = make_blobs(n_samples=300, centers=4, cluster_std=0.8, random_state=42)

# Compare metrics across different k
k_values = range(2, 10)
metrics = {'silhouette': [], 'calinski_harabasz': [], 'davies_bouldin': []}

for k in k_values:
    km = KMeans(n_clusters=k, random_state=42)
    labels = km.fit_predict(X)
    
    metrics['silhouette'].append(silhouette_score(X, labels))
    metrics['calinski_harabasz'].append(calinski_harabasz_score(X, labels))
    metrics['davies_bouldin'].append(davies_bouldin_score(X, labels))

print("Clustering Metrics:")
print(f"{'k':>4} {'Silhouette':>12} {'Calinski':>12} {'Davies-Bouldin':>15}")
print("-" * 45)
for i, k in enumerate(k_values):
    print(f"{k:>4} {metrics['silhouette'][i]:>12.4f} "
          f"{metrics['calinski_harabasz'][i]:>12.2f} "
          f"{metrics['davies_bouldin'][i]:>15.4f}")

print(f"\nBest k by Silhouette: {list(k_values)[np.argmax(metrics['silhouette'])]}")
print(f"Best k by Calinski-Harabasz: {list(k_values)[np.argmax(metrics['calinski_harabasz'])]}")
print(f"Best k by Davies-Bouldin: {list(k_values)[np.argmin(metrics['davies_bouldin'])]}")
print("\nNote: Silhouette (higher=better), Calinski (higher=better), Davies (lower=better)")
```

---

## 12. Customer Segmentation Project

```python
"""
โปรแกรมจริง: Customer Segmentation
ใช้ RFM Analysis + K-Means
"""
import numpy as np
import pandas as pd
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA
from sklearn.metrics import silhouette_score
import matplotlib.pyplot as plt
from datetime import datetime, timedelta
import warnings
warnings.filterwarnings('ignore')

# สร้าง Synthetic Customer Data
np.random.seed(42)
n_customers = 1000
reference_date = datetime(2024, 1, 1)

# RFM: Recency, Frequency, Monetary
data = {
    'customer_id': range(1, n_customers + 1),
    'last_purchase_days': np.concatenate([
        np.random.exponential(30, 200),   # VIP: recent buyers
        np.random.exponential(90, 300),   # Regular
        np.random.exponential(200, 300),  # At-risk
        np.random.exponential(365, 200)   # Churned
    ]),
    'purchase_frequency': np.concatenate([
        np.random.poisson(20, 200),       # VIP: frequent
        np.random.poisson(8, 300),        # Regular
        np.random.poisson(3, 300),        # Occasional
        np.random.poisson(1, 200)         # Very low
    ]),
    'total_spent': np.concatenate([
        np.random.exponential(5000, 200) + 5000,  # VIP: high value
        np.random.exponential(2000, 300) + 1000,  # Regular
        np.random.exponential(500, 300) + 200,    # Low value
        np.random.exponential(200, 200) + 50      # Very low
    ]),
    'age': np.random.randint(18, 70, n_customers),
    'region': np.random.choice(['North', 'South', 'East', 'West'], n_customers)
}

df = pd.DataFrame(data)
df['recency'] = df['last_purchase_days']
df['frequency'] = df['purchase_frequency']
df['monetary'] = df['total_spent']

print("=== Customer Dataset ===")
print(f"Customers: {len(df)}")
print(f"\nRFM Statistics:")
print(df[['recency', 'frequency', 'monetary']].describe().round(2))

# RFM Scoring
def rfm_score(df, col, reverse=False):
    """สร้าง quantile score 1-5"""
    labels = [1, 2, 3, 4, 5] if not reverse else [5, 4, 3, 2, 1]
    return pd.qcut(df[col], q=5, labels=labels, duplicates='drop').astype(int)

df['r_score'] = rfm_score(df, 'recency', reverse=True)  # เร็ว = ดี
df['f_score'] = rfm_score(df, 'frequency')
df['m_score'] = rfm_score(df, 'monetary')
df['rfm_score'] = df['r_score'] + df['f_score'] + df['m_score']

# Prepare features for clustering
rfm_features = df[['recency', 'frequency', 'monetary']].copy()
rfm_features['log_recency'] = np.log1p(rfm_features['recency'])
rfm_features['log_frequency'] = np.log1p(rfm_features['frequency'])
rfm_features['log_monetary'] = np.log1p(rfm_features['monetary'])

X = rfm_features[['log_recency', 'log_frequency', 'log_monetary']].values

# Scale
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# Find optimal k
k_scores = {}
for k in range(2, 9):
    km = KMeans(n_clusters=k, random_state=42, n_init=10)
    labels = km.fit_predict(X_scaled)
    k_scores[k] = silhouette_score(X_scaled, labels)

optimal_k = max(k_scores, key=k_scores.get)
print(f"\nOptimal k: {optimal_k} (silhouette={k_scores[optimal_k]:.4f})")

# Final clustering
kmeans = KMeans(n_clusters=optimal_k, random_state=42, n_init=10)
df['cluster'] = kmeans.fit_predict(X_scaled)

# Analyze clusters
print(f"\n=== Cluster Analysis ===")
cluster_analysis = df.groupby('cluster').agg({
    'recency': 'mean',
    'frequency': 'mean',
    'monetary': 'mean',
    'rfm_score': 'mean',
    'customer_id': 'count'
}).round(2)
cluster_analysis.columns = ['Avg_Recency', 'Avg_Frequency', 'Avg_Monetary', 'Avg_RFM', 'Count']
print(cluster_analysis)

# Name clusters based on RFM
def name_cluster(row):
    if row['Avg_RFM'] >= 11:
        return 'Champions'
    elif row['Avg_RFM'] >= 8:
        return 'Loyal Customers'
    elif row['Avg_RFM'] >= 5:
        return 'At Risk'
    else:
        return 'Churned'

cluster_analysis['Segment'] = cluster_analysis.apply(name_cluster, axis=1)
print(f"\nCluster Names:")
print(cluster_analysis[['Count', 'Segment', 'Avg_RFM']])

# PCA visualization
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X_scaled)

plt.figure(figsize=(10, 7))
colors = ['#1f77b4', '#ff7f0e', '#2ca02c', '#d62728', '#9467bd']
for cluster_id in sorted(df['cluster'].unique()):
    mask = df['cluster'] == cluster_id
    segment_name = cluster_analysis.loc[cluster_id, 'Segment'] if cluster_id in cluster_analysis.index else f'Cluster {cluster_id}'
    count = mask.sum()
    plt.scatter(X_pca[mask, 0], X_pca[mask, 1], 
                 label=f'{segment_name} (n={count})',
                 s=30, alpha=0.6, color=colors[cluster_id % len(colors)])

plt.xlabel(f'PC1 ({pca.explained_variance_ratio_[0]*100:.1f}%)')
plt.ylabel(f'PC2 ({pca.explained_variance_ratio_[1]*100:.1f}%)')
plt.title('Customer Segmentation (K-Means + PCA)')
plt.legend(loc='upper right')
plt.grid(True, alpha=0.3)
plt.savefig('customer_segmentation.png', dpi=100)
print("\nCustomer segmentation plot saved!")

# Business recommendations
print("\n=== Business Recommendations ===")
segment_recommendations = {
    'Champions': 'Reward them. Can be early adopters for new products',
    'Loyal Customers': 'Offer membership/loyalty programs',
    'At Risk': 'Send personalized reactivation campaigns',
    'Churned': 'Win-back campaigns, surveys to understand why they left'
}
for segment, rec in segment_recommendations.items():
    print(f"\n{segment}:")
    print(f"  Action: {rec}")
```

---

## 13. Document Clustering Project

```python
"""
โปรแกรมจริง: Document Clustering
"""
import numpy as np
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.cluster import KMeans, AgglomerativeClustering
from sklearn.decomposition import TruncatedSVD, LatentDirichletAllocation
from sklearn.preprocessing import Normalizer
from sklearn.pipeline import Pipeline
from sklearn.metrics import silhouette_score
import matplotlib.pyplot as plt

# สร้าง Synthetic Documents
topics = {
    'technology': ['python', 'machine learning', 'artificial intelligence', 'computer', 
                    'software', 'algorithm', 'data science', 'neural network', 'deep learning'],
    'sports': ['football', 'basketball', 'tennis', 'player', 'team', 'score', 
                'match', 'championship', 'athlete', 'game'],
    'cooking': ['recipe', 'ingredients', 'cook', 'bake', 'kitchen', 'dish', 
                 'flavor', 'spice', 'meal', 'restaurant'],
    'travel': ['destination', 'hotel', 'flight', 'tourism', 'vacation', 'passport', 
                'culture', 'explore', 'adventure', 'trip']
}

np.random.seed(42)
documents = []
doc_topics = []

for topic, words in topics.items():
    for _ in range(50):  # 50 documents per topic
        n_words = np.random.randint(20, 50)
        doc_words = np.random.choice(words, n_words, replace=True).tolist()
        filler = ['the', 'a', 'is', 'are', 'was', 'and', 'or', 'in', 'on', 'at', 'to']
        doc_words += np.random.choice(filler, n_words // 2, replace=True).tolist()
        np.random.shuffle(doc_words)
        documents.append(' '.join(doc_words))
        doc_topics.append(topic)

print(f"Documents: {len(documents)}")
print(f"Topics: {list(topics.keys())}")

# TF-IDF Vectorization
tfidf = TfidfVectorizer(max_features=500, stop_words='english', min_df=2)
X_tfidf = tfidf.fit_transform(documents)
print(f"\nTF-IDF shape: {X_tfidf.shape}")

# Dimensionality Reduction (LSA/SVD)
svd = TruncatedSVD(n_components=50, random_state=42)
X_lsa = svd.fit_transform(X_tfidf)
print(f"After SVD: {X_lsa.shape}")
print(f"Explained variance: {svd.explained_variance_ratio_.sum():.4f}")

# Normalize
X_normalized = Normalizer().fit_transform(X_lsa)

# K-Means on documents
kmeans = KMeans(n_clusters=4, random_state=42, n_init=10)
cluster_labels = kmeans.fit_predict(X_normalized)

# Evaluate
from sklearn.preprocessing import LabelEncoder
le = LabelEncoder()
true_labels = le.fit_transform(doc_topics)

from sklearn.metrics import adjusted_rand_score, normalized_mutual_info_score
ari = adjusted_rand_score(true_labels, cluster_labels)
nmi = normalized_mutual_info_score(true_labels, cluster_labels)
sil = silhouette_score(X_normalized, cluster_labels)

print(f"\nClustering Quality:")
print(f"  Adjusted Rand Index: {ari:.4f}")
print(f"  Normalized Mutual Info: {nmi:.4f}")
print(f"  Silhouette Score: {sil:.4f}")

# Top words per cluster
feature_names = tfidf.get_feature_names_out()
print("\nTop Words per Cluster:")
order_centroids = kmeans.cluster_centers_.argsort()[:, ::-1]

# Reconstruct original TF-IDF centroids
tfidf_centroids = kmeans.cluster_centers_.dot(svd.components_)
for i in range(4):
    top_indices = tfidf_centroids[i].argsort()[-10:][::-1]
    top_words = [feature_names[j] for j in top_indices]
    doc_count = (cluster_labels == i).sum()
    print(f"\nCluster {i} ({doc_count} docs): {', '.join(top_words)}")
```

---

## 14. Clustering Comparison

```python
"""
เปรียบเทียบ Clustering Algorithms บน Different Datasets
"""
from sklearn.cluster import KMeans, DBSCAN, AgglomerativeClustering
from sklearn.mixture import GaussianMixture
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import make_blobs, make_moons, make_circles
from sklearn.metrics import adjusted_rand_score
import numpy as np
import matplotlib.pyplot as plt

# Datasets
datasets = {
    'Blobs (Easy)': make_blobs(n_samples=300, centers=4, cluster_std=0.8, random_state=42),
    'Moons (Non-convex)': make_moons(n_samples=300, noise=0.05, random_state=42),
    'Circles (Concentric)': make_circles(n_samples=300, noise=0.05, factor=0.5, random_state=42)
}

# Algorithms
algorithms = {
    'K-Means': KMeans(n_clusters=2, random_state=42),
    'DBSCAN': DBSCAN(eps=0.3, min_samples=5),
    'Agglomerative': AgglomerativeClustering(n_clusters=2, linkage='ward'),
    'GMM': GaussianMixture(n_components=2, random_state=42)
}

print("Clustering Algorithm Comparison (ARI scores):")
print(f"{'Dataset':25s}", end='')
for algo_name in algorithms.keys():
    print(f"{algo_name:15s}", end='')
print()
print("-" * 85)

fig, axes = plt.subplots(len(datasets), len(algorithms), figsize=(20, 12))

for row, (ds_name, (X, y_true)) in enumerate(datasets.items()):
    X_scaled = StandardScaler().fit_transform(X)
    print(f"\n{ds_name:25s}", end='')
    
    for col, (algo_name, algo) in enumerate(algorithms.items()):
        ax = axes[row, col]
        
        try:
            if algo_name == 'GMM':
                labels = algo.fit_predict(X_scaled)
            else:
                labels = algo.fit_predict(X_scaled)
            
            n_clusters = len(set(labels)) - (1 if -1 in labels else 0)
            ari = adjusted_rand_score(y_true, labels)
            print(f"{ari:15.4f}", end='')
            
            ax.scatter(X[:, 0], X[:, 1], c=labels, cmap='viridis', s=20)
            ax.set_title(f'{algo_name}\nARI={ari:.3f}, k={n_clusters}')
            
        except Exception as e:
            print(f"{'Error':15s}", end='')
            ax.set_title(f'{algo_name}\nError')
        
        if row == 0:
            ax.set_title(f'{algo_name}\nARI={ari if "ari" in dir() else "N/A":.3f}')
        
        ax.set_xticks([])
        ax.set_yticks([])
        if col == 0:
            ax.set_ylabel(ds_name[:20])
    
print()

plt.suptitle('Clustering Algorithms Comparison', fontsize=14)
plt.tight_layout()
plt.savefig('clustering_comparison.png', dpi=80)
print("\nComparison plot saved!")
```

---

## 15. แบบฝึกหัด

### แบบฝึกหัดที่ 1: K-Means Customer Segmentation

```python
# เฉลย
import numpy as np
import pandas as pd
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import silhouette_score
import matplotlib.pyplot as plt

np.random.seed(42)
n = 500
customers = pd.DataFrame({
    'annual_income': np.random.exponential(50000, n) + 20000,
    'spending_score': np.random.uniform(0, 100, n),
    'age': np.random.randint(18, 70, n),
    'years_customer': np.random.exponential(5, n) + 0.5
})

# Elbow method
scaler = StandardScaler()
X = scaler.fit_transform(customers)

sse = []
sil = []
for k in range(2, 11):
    km = KMeans(n_clusters=k, random_state=42, n_init=10)
    labels = km.fit_predict(X)
    sse.append(km.inertia_)
    sil.append(silhouette_score(X, labels))

best_k = range(2, 11)[np.argmax(sil)]
print(f"Best k: {best_k}, Silhouette: {max(sil):.4f}")

# Final clustering
km_final = KMeans(n_clusters=best_k, random_state=42)
customers['segment'] = km_final.fit_predict(X)

print("\nSegment Profiles:")
print(customers.groupby('segment')[['annual_income', 'spending_score']].mean().round(2))
```

### แบบฝึกหัดที่ 2: DBSCAN Anomaly Detection

```python
# เฉลย
import numpy as np
from sklearn.cluster import DBSCAN
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import precision_score, recall_score

np.random.seed(42)
n_normal = 500
n_anomaly = 25

X_normal = np.random.randn(n_normal, 3)
X_anomaly = np.random.uniform(-5, 5, (n_anomaly, 3))
X = np.vstack([X_normal, X_anomaly])
y_true = np.hstack([np.zeros(n_normal), np.ones(n_anomaly)])

X_scaled = StandardScaler().fit_transform(X)

best_f1 = 0
best_params = {}
for eps in [0.3, 0.5, 0.7, 1.0]:
    for min_samples in [3, 5, 10]:
        db = DBSCAN(eps=eps, min_samples=min_samples)
        labels = db.fit_predict(X_scaled)
        y_pred = (labels == -1).astype(int)
        
        if y_pred.sum() > 0:
            from sklearn.metrics import f1_score
            f1 = f1_score(y_true, y_pred)
            if f1 > best_f1:
                best_f1 = f1
                best_params = {'eps': eps, 'min_samples': min_samples}

print(f"Best DBSCAN params: {best_params}, F1={best_f1:.4f}")

db_best = DBSCAN(**best_params)
labels = db_best.fit_predict(X_scaled)
y_pred = (labels == -1).astype(int)
print(f"Detected anomalies: {y_pred.sum()}/{n_anomaly}")
print(f"Precision: {precision_score(y_true, y_pred):.4f}")
print(f"Recall: {recall_score(y_true, y_pred):.4f}")
```

### แบบฝึกหัดที่ 3: PCA Compression and Reconstruction

```python
# เฉลย
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import load_digits
from sklearn.metrics import mean_squared_error
import numpy as np
import matplotlib.pyplot as plt

digits = load_digits()
X, y = digits.data, digits.target

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# Find components for different variance levels
pca_full = PCA()
pca_full.fit(X_scaled)
cumvar = np.cumsum(pca_full.explained_variance_ratio_)

for threshold in [0.50, 0.70, 0.90, 0.95, 0.99]:
    n_components = np.argmax(cumvar >= threshold) + 1
    
    pca = PCA(n_components=n_components)
    X_compressed = pca.fit_transform(X_scaled)
    X_reconstructed = scaler.inverse_transform(pca.inverse_transform(X_compressed))
    
    mse = mean_squared_error(X, X_reconstructed)
    compression = X.shape[1] / n_components
    
    print(f"Variance={threshold:.0%}: {X.shape[1]}->{n_components} dims, "
          f"compression={compression:.1f}x, MSE={mse:.4f}")
```

### แบบฝึกหัดที่ 4: t-SNE Visualization

```python
# เฉลย
from sklearn.manifold import TSNE
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import load_wine
import matplotlib.pyplot as plt

wine = load_wine()
X, y = wine.data, wine.target

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

fig, axes = plt.subplots(1, 3, figsize=(18, 5))
perplexities = [5, 30, 50]
colors = ['blue', 'red', 'green']

for ax, perp in zip(axes, perplexities):
    tsne = TSNE(n_components=2, perplexity=perp, random_state=42, n_iter=1000)
    X_tsne = tsne.fit_transform(X_scaled)
    
    for i, (name, color) in enumerate(zip(wine.target_names, colors)):
        mask = y == i
        ax.scatter(X_tsne[mask, 0], X_tsne[mask, 1], label=name, color=color, s=30)
    
    ax.set_title(f'Perplexity={perp}')
    ax.legend()

plt.suptitle('t-SNE - Wine Dataset')
plt.tight_layout()
plt.savefig('tsne_wine.png', dpi=100)
print("t-SNE wine saved!")
```

### แบบฝึกหัดที่ 5: GMM vs K-Means

```python
# เฉลย
from sklearn.cluster import KMeans
from sklearn.mixture import GaussianMixture
from sklearn.metrics import silhouette_score, adjusted_rand_score
from sklearn.datasets import make_blobs
import numpy as np

np.random.seed(42)
X, y_true = make_blobs(n_samples=500, centers=4, cluster_std=[1.0, 1.5, 0.5, 2.0], random_state=42)

# K-Means
km = KMeans(n_clusters=4, random_state=42)
labels_km = km.fit_predict(X)

# GMM
gmm = GaussianMixture(n_components=4, random_state=42)
labels_gmm = gmm.fit_predict(X)

for name, labels in [('K-Means', labels_km), ('GMM', labels_gmm)]:
    sil = silhouette_score(X, labels)
    ari = adjusted_rand_score(y_true, labels)
    print(f"{name:10s}: Silhouette={sil:.4f}, ARI={ari:.4f}")

# BIC for GMM
bics = []
for k in range(2, 8):
    g = GaussianMixture(n_components=k, random_state=42)
    g.fit(X)
    bics.append(g.bic(X))

best_k = range(2, 8)[np.argmin(bics)]
print(f"\nGMM optimal k (BIC): {best_k}")
```

### แบบฝึกหัดที่ 6: Association Rules for Market Basket

```python
# เฉลย (ไม่ต้องการ mlxtend)
from collections import defaultdict
from itertools import combinations
import numpy as np

np.random.seed(42)

# สร้าง transactions
items = ['bread', 'milk', 'eggs', 'butter', 'cheese', 'jam', 'coffee', 'tea']

transactions = []
for _ in range(1000):
    n_items = np.random.randint(2, 6)
    trans = list(np.random.choice(items, n_items, replace=False))
    transactions.append(trans)

n_trans = len(transactions)
print(f"Transactions: {n_trans}")

# Count frequencies
item_counts = defaultdict(int)
pair_counts = defaultdict(int)

for trans in transactions:
    for item in trans:
        item_counts[item] += 1
    for pair in combinations(sorted(trans), 2):
        pair_counts[pair] += 1

# Calculate support and confidence
min_support = 0.1
min_confidence = 0.4

print("\nFrequent Items (support >= 0.1):")
for item, count in sorted(item_counts.items(), key=lambda x: x[1], reverse=True):
    support = count / n_trans
    if support >= min_support:
        print(f"  {item:10s}: support={support:.3f}")

print("\nAssociation Rules (confidence >= 0.4):")
for (a, b), ab_count in sorted(pair_counts.items(), key=lambda x: x[1], reverse=True):
    support_ab = ab_count / n_trans
    if support_ab >= min_support:
        confidence = ab_count / item_counts[a]
        lift = confidence / (item_counts[b] / n_trans)
        if confidence >= min_confidence:
            print(f"  {a} -> {b}: support={support_ab:.3f}, "
                  f"confidence={confidence:.3f}, lift={lift:.2f}")
```

### แบบฝึกหัดที่ 7: Hierarchical Clustering Dendrogram

```python
# เฉลย
from sklearn.cluster import AgglomerativeClustering
from scipy.cluster.hierarchy import dendrogram, linkage, fcluster
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import load_iris
from sklearn.metrics import silhouette_score, adjusted_rand_score
import matplotlib.pyplot as plt
import numpy as np

iris = load_iris()
X, y = iris.data, iris.target

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# Dendrogram
Z = linkage(X_scaled, method='ward')

plt.figure(figsize=(15, 6))
plt.subplot(1, 2, 1)
dendrogram(Z, truncate_mode='lastp', p=30, leaf_rotation=45, leaf_font_size=8)
plt.title('Ward Linkage Dendrogram')
plt.xlabel('Samples')
plt.ylabel('Distance')

# Cut at different heights
plt.subplot(1, 2, 2)
thresholds = [1, 2, 3, 5, 8]
for t in thresholds:
    labels = fcluster(Z, t=t, criterion='distance')
    n_clust = len(set(labels))
    if n_clust >= 2:
        sil = silhouette_score(X_scaled, labels)
        ari = adjusted_rand_score(y, labels - 1)
        print(f"Threshold={t}: k={n_clust}, Sil={sil:.4f}, ARI={ari:.4f}")

plt.tight_layout()
plt.savefig('hierarchical_iris.png', dpi=100)
print("Hierarchical clustering saved!")
```

### แบบฝึกหัดที่ 8: Complete Unsupervised Pipeline

```python
# เฉลย
import numpy as np
import pandas as pd
from sklearn.cluster import KMeans
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import silhouette_score
from sklearn.ensemble import IsolationForest
import matplotlib.pyplot as plt

np.random.seed(42)
n = 1000
features = pd.DataFrame({
    'age': np.random.randint(20, 70, n),
    'income': np.random.lognormal(10.5, 0.7, n),
    'web_visits': np.random.poisson(15, n),
    'purchase_amount': np.random.exponential(500, n),
    'tenure_months': np.random.randint(1, 120, n)
})

# 1. Anomaly Detection
scaler = StandardScaler()
X_scaled = scaler.fit_transform(features)

iso = IsolationForest(contamination=0.05, random_state=42)
anomaly_labels = iso.fit_predict(X_scaled)
print(f"Anomalies detected: {(anomaly_labels == -1).sum()}")

# Remove anomalies
features_clean = features[anomaly_labels == 1].copy()
X_clean = X_scaled[anomaly_labels == 1]

# 2. Find optimal k
sil_scores = {}
for k in range(2, 9):
    km = KMeans(n_clusters=k, random_state=42)
    labels = km.fit_predict(X_clean)
    sil_scores[k] = silhouette_score(X_clean, labels)

best_k = max(sil_scores, key=sil_scores.get)
print(f"Optimal k: {best_k}")

# 3. Clustering
km_final = KMeans(n_clusters=best_k, random_state=42)
features_clean['cluster'] = km_final.fit_predict(X_clean)

# 4. PCA Visualization
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X_clean)

plt.figure(figsize=(10, 7))
for c in range(best_k):
    mask = features_clean['cluster'] == c
    plt.scatter(X_pca[mask, 0], X_pca[mask, 1], 
                 label=f'Segment {c} (n={mask.sum()})', s=30, alpha=0.6)

plt.xlabel('PC1')
plt.ylabel('PC2')
plt.title('Customer Segments (PCA + K-Means)')
plt.legend()
plt.grid(True, alpha=0.3)
plt.savefig('complete_pipeline.png', dpi=100)

print("\nSegment Profiles:")
print(features_clean.groupby('cluster').mean().round(2))
```

---

## สรุปบทที่ 79

| Algorithm | ประเภท | ข้อดี | ข้อเสีย | ใช้เมื่อ |
|-----------|--------|-------|---------|---------|
| K-Means | Clustering | เร็ว, scalable | ต้องกำหนด k, spherical only | Large data, convex clusters |
| Hierarchical | Clustering | ไม่ต้องกำหนด k, dendrogram | ช้า O(n³) | Small data, สำรวจโครงสร้าง |
| DBSCAN | Clustering | Arbitrary shapes, noise detection | ต้องกำหนด eps | Non-convex, anomaly detection |
| GMM | Clustering | Probabilistic, soft assignments | ช้า, covariance assumptions | Overlapping clusters |
| PCA | Dim. Reduction | Fast, linear, interpretable | Linear only | Preprocessing, visualization |
| t-SNE | Visualization | Non-linear, great visualization | ช้า, not for new data | 2D/3D visualization |
| UMAP | Visualization | เร็วกว่า t-SNE, preserves structure | Less interpretable | Large-scale visualization |
| Isolation Forest | Anomaly | Fast, no assumptions | Contamination parameter | High-dim anomaly |
| LOF | Anomaly | Local density, no global assumption | O(n²) | Local anomalies |

**Key Takeaways:**
1. Scale ข้อมูลก่อนใช้ distance-based algorithms เสมอ
2. K-Means ใช้ Elbow/Silhouette หา optimal k
3. DBSCAN เหมาะกับ non-convex clusters และ anomaly detection
4. PCA ลด dimension ก่อน t-SNE เพื่อประสิทธิภาพ
5. Silhouette > 0.5 = good, > 0.7 = strong clustering
6. ใช้หลาย metrics ในการประเมิน clustering
7. Customer segmentation + RFM เป็น use case ที่ใช้บ่อยมาก

**Part 80** จะเรียนเรื่อง Deep Learning ด้วย TensorFlow และ Keras
