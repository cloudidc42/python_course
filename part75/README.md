# Part 75 - Data Visualization: Matplotlib, Seaborn, Plotly & Bokeh

## สารบัญ
1. [Matplotlib Basics](#1-matplotlib-basics)
2. [Figure และ Axes](#2-figure-และ-axes)
3. [Common Plot Types](#3-common-plot-types)
4. [Subplots](#4-subplots)
5. [Customization](#5-customization)
6. [Saving Figures](#6-saving-figures)
7. [Seaborn Library](#7-seaborn-library)
8. [Statistical Plots](#8-statistical-plots)
9. [Plotly สำหรับ Interactive Charts](#9-plotly-สำหรับ-interactive-charts)
10. [Plotly Express](#10-plotly-express)
11. [Bokeh เบื้องต้น](#11-bokeh-เบื้องต้น)
12. [Visualization Best Practices](#12-visualization-best-practices)
13. [แบบฝึกหัด](#13-แบบฝึกหัด)

---

## 1. Matplotlib Basics

### Matplotlib คืออะไร?

**Matplotlib** เป็น library visualization หลักของ Python ecosystem
สร้างโดย John D. Hunter ในปี 2003 โดยได้รับแรงบันดาลใจจาก MATLAB

**Architecture ของ Matplotlib**:
- **Figure**: container หลัก (กระดาษทั้งใบ)
- **Axes**: area ที่ plot ข้อมูล (แกน x,y)
- **Artist**: ทุกสิ่งที่วาดบน figure (lines, text, patches)

```python
# ตัวอย่างที่ 1: Matplotlib hello world
import matplotlib
import matplotlib.pyplot as plt
import numpy as np

print(f"Matplotlib version: {matplotlib.__version__}")

# สร้าง simple plot
x = np.linspace(0, 2*np.pi, 100)
y = np.sin(x)

plt.figure(figsize=(10, 4))
plt.plot(x, y)
plt.title('Sine Wave')
plt.xlabel('x (radians)')
plt.ylabel('sin(x)')
plt.grid(True)
plt.savefig('/tmp/plot_01_sine.png', dpi=100, bbox_inches='tight')
plt.close()
print("Saved: plot_01_sine.png")
```

```python
# ตัวอย่างที่ 2: Pyplot vs Object-Oriented API
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)

# Method 1: Pyplot style (เหมือน MATLAB, เหมาะกับ quick plots)
plt.figure(figsize=(12, 4))
plt.subplot(1, 2, 1)
plt.plot(x, np.sin(x), 'b-')
plt.title('Pyplot Style')
plt.xlabel('x')
plt.ylabel('sin(x)')
plt.grid(alpha=0.3)

plt.subplot(1, 2, 2)
plt.plot(x, np.cos(x), 'r--')
plt.title('Cosine')
plt.xlabel('x')
plt.ylabel('cos(x)')
plt.grid(alpha=0.3)

plt.suptitle('Pyplot API', fontsize=14)
plt.tight_layout()
plt.savefig('/tmp/plot_02_pyplot.png', dpi=100, bbox_inches='tight')
plt.close()

# Method 2: Object-Oriented (แนะนำสำหรับ complex figures)
fig, axes = plt.subplots(1, 2, figsize=(12, 4))

ax1, ax2 = axes

ax1.plot(x, np.sin(x), 'b-')
ax1.set_title('OO Style - Sine')
ax1.set_xlabel('x')
ax1.set_ylabel('sin(x)')
ax1.grid(alpha=0.3)

ax2.plot(x, np.cos(x), 'r--')
ax2.set_title('OO Style - Cosine')
ax2.set_xlabel('x')
ax2.set_ylabel('cos(x)')
ax2.grid(alpha=0.3)

fig.suptitle('Object-Oriented API', fontsize=14)
fig.tight_layout()
fig.savefig('/tmp/plot_02_oo.png', dpi=100, bbox_inches='tight')
plt.close()
print("Saved: OO style plots")
```

---

## 2. Figure และ Axes

### เข้าใจโครงสร้างของ Matplotlib

```python
# ตัวอย่างที่ 3: Figure และ Axes components
import matplotlib.pyplot as plt
import numpy as np

fig, ax = plt.subplots(figsize=(10, 6))

x = np.linspace(0, 10, 100)
y = np.sin(x) * np.exp(-0.1 * x)

# เส้นกราฟ
line, = ax.plot(x, y, 'b-', linewidth=2, label='Damped Sine')
ax.axhline(y=0, color='gray', linestyle='--', alpha=0.5)  # horizontal line

# Labels
ax.set_title('Figure and Axes Components', fontsize=16, fontweight='bold', pad=20)
ax.set_xlabel('Time (seconds)', fontsize=12, labelpad=10)
ax.set_ylabel('Amplitude', fontsize=12, labelpad=10)

# Tick customization
ax.set_xlim(0, 10)
ax.set_ylim(-1.2, 1.2)
ax.set_xticks(np.arange(0, 11, 2))
ax.set_yticks([-1, -0.5, 0, 0.5, 1])
ax.tick_params(axis='both', labelsize=10)

# Legend
ax.legend(loc='upper right', fontsize=11)

# Grid
ax.grid(True, alpha=0.3, linestyle=':')

# Text annotation
ax.annotate('Peak', xy=(np.pi/2, 1.0), xytext=(2, 1.1),
            arrowprops=dict(arrowstyle='->', color='red'),
            fontsize=11, color='red')

# Spine customization
for spine in ax.spines.values():
    spine.set_linewidth(1.5)

fig.tight_layout()
fig.savefig('/tmp/plot_03_components.png', dpi=100, bbox_inches='tight')
plt.close()
print("Saved: components demo")
```

```python
# ตัวอย่างที่ 4: Figure properties
import matplotlib.pyplot as plt
import numpy as np

# Figure parameters
fig = plt.figure(
    figsize=(12, 8),      # width, height in inches
    dpi=100,               # dots per inch
    facecolor='white',     # background color
    edgecolor='black'      # border color
)

# เพิ่ม axes ด้วย add_subplot หรือ add_axes
ax1 = fig.add_subplot(2, 2, 1)   # 2 rows, 2 cols, position 1
ax2 = fig.add_subplot(2, 2, 2)
ax3 = fig.add_subplot(2, 1, 2)   # span full width at row 2

x = np.linspace(0, 2*np.pi, 100)

ax1.plot(x, np.sin(x), 'b-')
ax1.set_title('Top Left')
ax1.set_facecolor('#f0f0f0')   # axes background

ax2.plot(x, np.cos(x), 'r-')
ax2.set_title('Top Right')

ax3.plot(x, np.sin(x) + np.cos(x), 'g-')
ax3.set_title('Bottom (Full Width)')
ax3.fill_between(x, 0, np.sin(x) + np.cos(x), alpha=0.3, color='green')

fig.suptitle('Figure with Multiple Axes', fontsize=16)
fig.tight_layout()
fig.savefig('/tmp/plot_04_figure.png', dpi=100, bbox_inches='tight')
plt.close()
```

---

## 3. Common Plot Types

### Line Plots

```python
# ตัวอย่างที่ 5: Line plots - styles และ options
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 4*np.pi, 100)
fig, ax = plt.subplots(figsize=(12, 6))

# Line styles: '-', '--', '-.', ':'
# Colors: 'b','r','g', '#FF5733', (0.1, 0.2, 0.9)
# Markers: 'o', 's', '^', 'D', '*', '+', 'x'

lines_config = [
    (np.sin(x), 'b-o', 2, 5, 'Sine (solid, circle)'),
    (np.cos(x), 'r--s', 2, 5, 'Cosine (dashed, square)'),
    (np.sin(2*x), 'g-.^', 2, 5, 'Sin(2x) (dash-dot, triangle)'),
    (np.cos(2*x), 'm:D', 2, 5, 'Cos(2x) (dotted, diamond)')
]

for y_vals, style, lw, ms, label in lines_config:
    ax.plot(x, y_vals, style, linewidth=lw, markersize=ms,
            markevery=20, label=label, alpha=0.8)

ax.set_title('Line Plot Styles', fontsize=14)
ax.set_xlabel('x')
ax.set_ylabel('y')
ax.legend(loc='upper right')
ax.grid(True, alpha=0.3)
ax.set_xlim(0, 4*np.pi)

fig.tight_layout()
fig.savefig('/tmp/plot_05_lines.png', dpi=100, bbox_inches='tight')
plt.close()
```

### Scatter Plots

```python
# ตัวอย่างที่ 6: Scatter plots
import matplotlib.pyplot as plt
import numpy as np

rng = np.random.default_rng(42)
n = 200

# สร้างข้อมูล
x = rng.normal(0, 1, n)
y = x * 2 + rng.normal(0, 0.5, n)
colors = rng.uniform(0, 1, n)    # color ตาม value
sizes = rng.integers(20, 200, n)  # size ตาม value

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Basic scatter
axes[0].scatter(x, y, alpha=0.6, color='steelblue', edgecolors='white', linewidth=0.5)
axes[0].set_title('Basic Scatter Plot')
axes[0].set_xlabel('X')
axes[0].set_ylabel('Y')
axes[0].grid(True, alpha=0.3)

# Colored scatter กับ colorbar
scatter = axes[1].scatter(x, y,
                           c=colors,         # color ตาม value
                           s=sizes,          # size ตาม value
                           alpha=0.7,
                           cmap='viridis',
                           edgecolors='gray',
                           linewidth=0.3)
plt.colorbar(scatter, ax=axes[1], label='Color Value')
axes[1].set_title('Bubble Chart (size + color)')
axes[1].set_xlabel('X')
axes[1].set_ylabel('Y')

# Trend line
z = np.polyfit(x, y, 1)
p = np.poly1d(z)
x_sorted = np.sort(x)
axes[1].plot(x_sorted, p(x_sorted), 'r--', linewidth=2, label='Trend', alpha=0.8)
axes[1].legend()

fig.tight_layout()
fig.savefig('/tmp/plot_06_scatter.png', dpi=100, bbox_inches='tight')
plt.close()
```

### Bar Charts

```python
# ตัวอย่างที่ 7: Bar charts
import matplotlib.pyplot as plt
import numpy as np

categories = ['Electronics', 'Clothing', 'Food', 'Books', 'Sports']
values_2023 = [450, 320, 280, 190, 240]
values_2024 = [520, 380, 310, 220, 290]
x = np.arange(len(categories))
width = 0.35

fig, axes = plt.subplots(1, 3, figsize=(18, 5))

# Vertical bar chart
axes[0].bar(x, values_2023, width, label='2023', color='steelblue', alpha=0.8)
axes[0].bar(x + width, values_2024, width, label='2024', color='coral', alpha=0.8)
axes[0].set_title('Grouped Bar Chart')
axes[0].set_xlabel('Category')
axes[0].set_ylabel('Sales (K)')
axes[0].set_xticks(x + width/2)
axes[0].set_xticklabels(categories, rotation=45, ha='right')
axes[0].legend()
axes[0].grid(axis='y', alpha=0.3)

# Add value labels
for i, (v1, v2) in enumerate(zip(values_2023, values_2024)):
    axes[0].text(i, v1 + 5, str(v1), ha='center', va='bottom', fontsize=9)
    axes[0].text(i + width, v2 + 5, str(v2), ha='center', va='bottom', fontsize=9)

# Horizontal bar chart
axes[1].barh(categories, values_2024, color='steelblue', alpha=0.8)
axes[1].set_title('Horizontal Bar Chart')
axes[1].set_xlabel('Sales (K)')
axes[1].grid(axis='x', alpha=0.3)

# Stacked bar chart
axes[2].bar(x, values_2023, label='2023', color='steelblue', alpha=0.8)
axes[2].bar(x, values_2024, bottom=values_2023, label='2024', color='coral', alpha=0.8)
axes[2].set_title('Stacked Bar Chart')
axes[2].set_xlabel('Category')
axes[2].set_ylabel('Total Sales (K)')
axes[2].set_xticks(x)
axes[2].set_xticklabels(categories, rotation=45, ha='right')
axes[2].legend()
axes[2].grid(axis='y', alpha=0.3)

fig.tight_layout()
fig.savefig('/tmp/plot_07_bar.png', dpi=100, bbox_inches='tight')
plt.close()
```

### Histograms

```python
# ตัวอย่างที่ 8: Histograms
import matplotlib.pyplot as plt
import numpy as np
from scipy import stats

rng = np.random.default_rng(42)
data1 = rng.normal(60, 15, 1000)
data2 = rng.normal(80, 10, 500)

fig, axes = plt.subplots(1, 3, figsize=(18, 5))

# Basic histogram
axes[0].hist(data1, bins=30, color='steelblue', alpha=0.7, edgecolor='white')
axes[0].set_title('Basic Histogram')
axes[0].set_xlabel('Value')
axes[0].set_ylabel('Frequency')
axes[0].grid(axis='y', alpha=0.3)

# Overlapping histograms
axes[1].hist(data1, bins=30, color='steelblue', alpha=0.6, label='Group A', edgecolor='white')
axes[1].hist(data2, bins=25, color='coral', alpha=0.6, label='Group B', edgecolor='white')
axes[1].set_title('Overlapping Histograms')
axes[1].set_xlabel('Value')
axes[1].set_ylabel('Count')
axes[1].legend()
axes[1].grid(axis='y', alpha=0.3)

# Histogram with density curve
axes[2].hist(data1, bins=30, density=True, color='steelblue', alpha=0.6, edgecolor='white')
x_range = np.linspace(data1.min(), data1.max(), 200)
axes[2].plot(x_range, stats.norm.pdf(x_range, data1.mean(), data1.std()),
             'r-', linewidth=2, label='Normal PDF')
axes[2].axvline(data1.mean(), color='orange', linestyle='--', label=f'Mean={data1.mean():.1f}')
axes[2].axvline(np.median(data1), color='green', linestyle=':', label=f'Median={np.median(data1):.1f}')
axes[2].set_title('Histogram with PDF')
axes[2].set_xlabel('Value')
axes[2].set_ylabel('Density')
axes[2].legend()
axes[2].grid(axis='y', alpha=0.3)

fig.tight_layout()
fig.savefig('/tmp/plot_08_histogram.png', dpi=100, bbox_inches='tight')
plt.close()
```

---

## 4. Subplots

```python
# ตัวอย่างที่ 9: Complex subplot layouts
import matplotlib.pyplot as plt
import numpy as np
from matplotlib.gridspec import GridSpec

rng = np.random.default_rng(42)
x = np.linspace(0, 10, 100)

# GridSpec - กำหนด layout แบบ flexible
fig = plt.figure(figsize=(16, 10))
gs = GridSpec(3, 3, figure=fig, hspace=0.4, wspace=0.3)

# Plot ใน positions ต่างๆ
ax1 = fig.add_subplot(gs[0, :])     # row 0, span all cols
ax2 = fig.add_subplot(gs[1, :2])    # row 1, col 0-1
ax3 = fig.add_subplot(gs[1:, 2])    # row 1-2, col 2
ax4 = fig.add_subplot(gs[2, 0])     # row 2, col 0
ax5 = fig.add_subplot(gs[2, 1])     # row 2, col 1

# ax1: Line plot - full width
ax1.plot(x, np.sin(x), 'b-', linewidth=2)
ax1.plot(x, np.cos(x), 'r--', linewidth=2)
ax1.set_title('Line Plot (Full Width)')
ax1.grid(alpha=0.3)
ax1.legend(['sin(x)', 'cos(x)'])

# ax2: Scatter
scatter_x = rng.normal(5, 2, 200)
scatter_y = scatter_x * 1.5 + rng.normal(0, 1, 200)
ax2.scatter(scatter_x, scatter_y, alpha=0.5, c='steelblue')
ax2.set_title('Scatter Plot')

# ax3: Vertical bar (tall)
cats = ['A', 'B', 'C', 'D', 'E']
vals = rng.integers(10, 100, 5)
ax3.barh(cats, vals, color='coral')
ax3.set_title('Horizontal Bar')

# ax4: Histogram
ax4.hist(rng.normal(0, 1, 500), bins=20, color='green', alpha=0.7)
ax4.set_title('Histogram')

# ax5: Pie chart
ax5.pie([25, 35, 20, 20], labels=['A', 'B', 'C', 'D'],
        colors=['#FF6B6B', '#4ECDC4', '#45B7D1', '#FFA07A'],
        autopct='%1.1f%%', startangle=90)
ax5.set_title('Pie Chart')

fig.suptitle('Complex Layout with GridSpec', fontsize=16, y=0.98)
fig.savefig('/tmp/plot_09_gridspec.png', dpi=100, bbox_inches='tight')
plt.close()
```

---

## 5. Customization

### ปรับแต่ง Style และ Appearance

```python
# ตัวอย่างที่ 10: Color schemes และ styles
import matplotlib.pyplot as plt
import numpy as np

# ดู available styles
print("Available styles:")
print(plt.style.available[:10])

x = np.linspace(0, 2*np.pi, 100)
y_values = [np.sin(x), np.cos(x), np.sin(2*x), np.cos(2*x)]
labels = ['sin(x)', 'cos(x)', 'sin(2x)', 'cos(2x)']

# สร้าง plot ด้วย style ต่างๆ
styles_to_demo = ['default', 'seaborn-v0_8', 'ggplot', 'bmh']

fig, axes = plt.subplots(2, 2, figsize=(16, 10))
axes = axes.flatten()

for i, style in enumerate(styles_to_demo):
    with plt.style.context(style):
        for y, label in zip(y_values, labels):
            axes[i].plot(x, y, label=label)
        axes[i].set_title(f'Style: {style}', fontsize=12)
        axes[i].legend(loc='upper right', fontsize=9)
        axes[i].grid(True, alpha=0.3)

fig.suptitle('Matplotlib Style Comparison', fontsize=14)
fig.tight_layout()
fig.savefig('/tmp/plot_10_styles.png', dpi=100, bbox_inches='tight')
plt.close()
```

```python
# ตัวอย่างที่ 11: Color palettes และ colormaps
import matplotlib.pyplot as plt
import numpy as np
import matplotlib.colors as mcolors

# แสดง colormaps ต่างๆ
cmaps = ['viridis', 'plasma', 'inferno', 'coolwarm', 'RdYlGn', 'Blues']

fig, axes = plt.subplots(2, 3, figsize=(15, 6))
axes = axes.flatten()

x = np.linspace(0, 10, 100)
y = np.linspace(0, 10, 100)
X, Y = np.meshgrid(x, y)
Z = np.sin(X) * np.cos(Y)

for i, cmap in enumerate(cmaps):
    im = axes[i].imshow(Z, cmap=cmap, aspect='auto', origin='lower')
    axes[i].set_title(f'cmap={cmap}', fontsize=11)
    plt.colorbar(im, ax=axes[i])

fig.suptitle('Colormap Comparison', fontsize=14)
fig.tight_layout()
fig.savefig('/tmp/plot_11_colormaps.png', dpi=100, bbox_inches='tight')
plt.close()
```

```python
# ตัวอย่างที่ 12: Annotations และ Text
import matplotlib.pyplot as plt
import numpy as np

fig, ax = plt.subplots(figsize=(12, 7))

x = np.linspace(0, 4*np.pi, 200)
y = np.sin(x) * np.exp(-x/10)

ax.plot(x, y, 'b-', linewidth=2)
ax.fill_between(x, 0, y, alpha=0.1, color='blue')
ax.axhline(y=0, color='gray', linestyle='--', alpha=0.5)

# Find peak
peak_idx = np.argmax(y)
peak_x, peak_y = x[peak_idx], y[peak_idx]

# Annotation
ax.annotate(
    f'Peak\n({peak_x:.2f}, {peak_y:.3f})',
    xy=(peak_x, peak_y),
    xytext=(peak_x + 2, peak_y + 0.2),
    fontsize=11,
    arrowprops=dict(arrowstyle='->', color='red', lw=1.5),
    bbox=dict(boxstyle='round,pad=0.5', facecolor='yellow', alpha=0.8),
    color='darkred'
)

# Text box
textstr = 'Damped Sine Wave\n$y = \sin(x) \cdot e^{-x/10}$'
props = dict(boxstyle='round', facecolor='wheat', alpha=0.5)
ax.text(0.02, 0.95, textstr, transform=ax.transAxes,
        fontsize=12, verticalalignment='top', bbox=props)

ax.set_title('Annotations and Text', fontsize=14)
ax.set_xlabel('x')
ax.set_ylabel('y')
ax.grid(True, alpha=0.3)

fig.tight_layout()
fig.savefig('/tmp/plot_12_annotations.png', dpi=100, bbox_inches='tight')
plt.close()
```

---

## 6. Saving Figures

```python
# ตัวอย่างที่ 13: Saving in different formats
import matplotlib.pyplot as plt
import numpy as np

fig, ax = plt.subplots(figsize=(8, 5))
x = np.linspace(0, 2*np.pi, 100)
ax.plot(x, np.sin(x), 'b-', linewidth=2, label='sin(x)')
ax.plot(x, np.cos(x), 'r-', linewidth=2, label='cos(x)')
ax.set_title('Save Format Demo')
ax.legend()
ax.grid(True, alpha=0.3)

# ตัวเลือกสำหรับ savefig
formats = {
    'PNG (Raster)': ('/tmp/figure.png', {'dpi': 150, 'bbox_inches': 'tight'}),
    'SVG (Vector)': ('/tmp/figure.svg', {'bbox_inches': 'tight'}),
    'PDF': ('/tmp/figure.pdf', {'bbox_inches': 'tight'}),
    'JPEG': ('/tmp/figure.jpg', {'dpi': 150, 'quality': 95, 'bbox_inches': 'tight'}),
}

import os
for fmt_name, (path, kwargs) in formats.items():
    fig.savefig(path, **kwargs)
    size = os.path.getsize(path)
    print(f"{fmt_name}: {path} ({size/1024:.1f} KB)")

plt.close()

# สำหรับ Jupyter Notebook: %matplotlib inline หรือ plt.show()
# สำหรับ production: ใช้ plt.savefig() และ plt.close()
# สำหรับ web: ส่ง bytes buffer
import io
buf = io.BytesIO()
fig_new, ax_new = plt.subplots()
ax_new.plot([1,2,3], [4,5,6])
fig_new.savefig(buf, format='png', dpi=72, bbox_inches='tight')
buf.seek(0)
print(f"PNG bytes size: {len(buf.getvalue())} bytes")
plt.close()
```

---

## 7. Seaborn Library

### Seaborn คืออะไร?

**Seaborn** สร้างบน Matplotlib โดยเน้น:
- Statistical visualization
- Beautiful default styles
- Integration กับ Pandas DataFrames
- Complex plots ด้วย code น้อยกว่า

```python
# ตัวอย่างที่ 14: Seaborn setup และ basic plots
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

print(f"Seaborn version: {sns.__version__}")

# Default theme
sns.set_theme(style='whitegrid', palette='husl')

# สร้าง sample data
rng = np.random.default_rng(42)
n = 200
data = pd.DataFrame({
    'x': rng.normal(0, 1, n),
    'y': None,
    'category': rng.choice(['A', 'B', 'C'], n),
    'value': rng.integers(1, 100, n)
})
data['y'] = data['x'] * 1.5 + rng.normal(0, 0.8, n)

fig, axes = plt.subplots(1, 3, figsize=(18, 5))

# Scatter plot (sns.scatterplot)
sns.scatterplot(data=data, x='x', y='y', hue='category',
                alpha=0.7, ax=axes[0])
axes[0].set_title('Scatter by Category')

# Line plot
x_line = np.linspace(0, 10, 50)
line_data = pd.DataFrame({
    'x': np.tile(x_line, 3),
    'y': np.concatenate([
        np.sin(x_line) + rng.normal(0, 0.1, 50),
        np.cos(x_line) + rng.normal(0, 0.1, 50),
        np.sin(2*x_line) + rng.normal(0, 0.1, 50)
    ]),
    'series': ['sin'] * 50 + ['cos'] * 50 + ['sin2x'] * 50
})
sns.lineplot(data=line_data, x='x', y='y', hue='series', ax=axes[1])
axes[1].set_title('Line Plot by Series')

# Bar plot
sns.barplot(data=data, x='category', y='value', ax=axes[2], palette='Set2',
            errorbar='se')  # error bar = standard error
axes[2].set_title('Bar Plot with Error Bars')

fig.tight_layout()
fig.savefig('/tmp/plot_14_seaborn_basic.png', dpi=100, bbox_inches='tight')
plt.close()
```

```python
# ตัวอย่างที่ 15: Distribution plots
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

sns.set_theme(style='whitegrid')
rng = np.random.default_rng(42)

data = pd.DataFrame({
    'score': np.concatenate([
        rng.normal(65, 12, 300),
        rng.normal(80, 8, 200)
    ]),
    'group': ['A'] * 300 + ['B'] * 200
})

fig, axes = plt.subplots(2, 2, figsize=(16, 12))

# histplot กับ kde
sns.histplot(data=data, x='score', hue='group', bins=30,
             kde=True, alpha=0.6, ax=axes[0,0])
axes[0,0].set_title('Histogram with KDE')

# kdeplot
sns.kdeplot(data=data, x='score', hue='group', fill=True,
            alpha=0.5, ax=axes[0,1])
axes[0,1].set_title('KDE Plot')

# ecdfplot (Empirical CDF)
sns.ecdfplot(data=data, x='score', hue='group', ax=axes[1,0])
axes[1,0].set_title('ECDF Plot')

# rugplot
sns.histplot(data=data, x='score', hue='group', bins=20,
             kde=False, ax=axes[1,1])
sns.rugplot(data=data, x='score', hue='group', ax=axes[1,1],
            alpha=0.5, height=0.05)
axes[1,1].set_title('Histogram with Rug')

fig.tight_layout()
fig.savefig('/tmp/plot_15_distributions.png', dpi=100, bbox_inches='tight')
plt.close()
```

---

## 8. Statistical Plots

```python
# ตัวอย่างที่ 16: Boxplot และ Violin Plot
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

sns.set_theme(style='ticks')
rng = np.random.default_rng(42)

n = 300
data = pd.DataFrame({
    'value': np.concatenate([
        rng.normal(50, 15, n),
        rng.normal(65, 10, n),
        rng.normal(80, 20, n),
        rng.normal(70, 12, n),
        rng.normal(55, 18, n)
    ]),
    'department': ['HR'] * n + ['IT'] * n + ['Finance'] * n +
                  ['Marketing'] * n + ['Operations'] * n,
    'gender': rng.choice(['M', 'F'], 5*n)
})

fig, axes = plt.subplots(2, 2, figsize=(16, 12))

# Boxplot
sns.boxplot(data=data, x='department', y='value',
            hue='gender', ax=axes[0,0], palette='Set2')
axes[0,0].set_title('Box Plot by Department and Gender')
axes[0,0].tick_params(axis='x', rotation=30)

# Violin plot
sns.violinplot(data=data, x='department', y='value',
               hue='gender', ax=axes[0,1], split=True,
               palette='husl', inner='quart')
axes[0,1].set_title('Violin Plot (Split by Gender)')
axes[0,1].tick_params(axis='x', rotation=30)

# Strip plot (individual points)
sns.stripplot(data=data.sample(200, random_state=42),
              x='department', y='value', hue='gender',
              ax=axes[1,0], alpha=0.6, dodge=True, jitter=True)
axes[1,0].set_title('Strip Plot')
axes[1,0].tick_params(axis='x', rotation=30)

# Boxen plot (letter-value plot - ดีกว่า boxplot สำหรับ large data)
sns.boxenplot(data=data, x='department', y='value',
              ax=axes[1,1], palette='Set3')
axes[1,1].set_title('Boxen Plot (Letter-Value)')
axes[1,1].tick_params(axis='x', rotation=30)

fig.tight_layout()
fig.savefig('/tmp/plot_16_distribution.png', dpi=100, bbox_inches='tight')
plt.close()
```

```python
# ตัวอย่างที่ 17: Heatmap
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)

# สร้าง correlation matrix
n = 200
variables = ['Revenue', 'Profit', 'Employees', 'Customers', 'Marketing', 'R&D']
raw_data = rng.multivariate_normal(
    mean=[0] * 6,
    cov=np.array([
        [1.0, 0.8, 0.5, 0.7, 0.3, 0.4],
        [0.8, 1.0, 0.4, 0.6, 0.5, 0.3],
        [0.5, 0.4, 1.0, 0.3, 0.2, 0.1],
        [0.7, 0.6, 0.3, 1.0, 0.6, 0.4],
        [0.3, 0.5, 0.2, 0.6, 1.0, 0.7],
        [0.4, 0.3, 0.1, 0.4, 0.7, 1.0]
    ]),
    size=n
)
df = pd.DataFrame(raw_data, columns=variables)
corr = df.corr()

fig, axes = plt.subplots(1, 2, figsize=(16, 6))

# Basic heatmap
sns.heatmap(corr, annot=True, fmt='.2f', cmap='coolwarm',
            vmin=-1, vmax=1, ax=axes[0],
            linewidths=0.5, linecolor='white')
axes[0].set_title('Correlation Heatmap')

# Masked upper triangle
mask = np.triu(np.ones_like(corr, dtype=bool))
sns.heatmap(corr, mask=mask, annot=True, fmt='.2f', cmap='RdYlGn',
            vmin=-1, vmax=1, ax=axes[1],
            square=True, linewidths=0.5,
            cbar_kws={'shrink': 0.8})
axes[1].set_title('Lower Triangle Heatmap')

fig.tight_layout()
fig.savefig('/tmp/plot_17_heatmap.png', dpi=100, bbox_inches='tight')
plt.close()
```

```python
# ตัวอย่างที่ 18: Pairplot
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

sns.set_theme(style='white')
rng = np.random.default_rng(42)

# สร้าง iris-like dataset
n_per_class = 50
data = pd.DataFrame({
    'sepal_length': np.concatenate([
        rng.normal(5.0, 0.4, n_per_class),
        rng.normal(6.0, 0.5, n_per_class),
        rng.normal(6.5, 0.6, n_per_class)
    ]),
    'sepal_width': np.concatenate([
        rng.normal(3.4, 0.4, n_per_class),
        rng.normal(2.9, 0.3, n_per_class),
        rng.normal(3.0, 0.3, n_per_class)
    ]),
    'petal_length': np.concatenate([
        rng.normal(1.5, 0.2, n_per_class),
        rng.normal(4.3, 0.5, n_per_class),
        rng.normal(5.5, 0.5, n_per_class)
    ]),
    'species': ['setosa'] * n_per_class + ['versicolor'] * n_per_class + ['virginica'] * n_per_class
})

# Pairplot
g = sns.pairplot(data, hue='species', diag_kind='kde',
                 plot_kws={'alpha': 0.6},
                 palette='husl')
g.fig.suptitle('Pairplot - Species Comparison', y=1.02, fontsize=14)
g.fig.savefig('/tmp/plot_18_pairplot.png', dpi=100, bbox_inches='tight')
plt.close()
```

```python
# ตัวอย่างที่ 19: FacetGrid และ categorical plots
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

sns.set_theme(style='whitegrid')
rng = np.random.default_rng(42)

# Sales data
n = 300
sales = pd.DataFrame({
    'month': rng.choice(['Jan', 'Feb', 'Mar', 'Apr'], n),
    'region': rng.choice(['North', 'South', 'East'], n),
    'product': rng.choice(['A', 'B'], n),
    'revenue': rng.integers(1000, 10000, n)
})

# FacetGrid - หลาย plots ตาม categories
g = sns.FacetGrid(sales, col='region', row='product',
                  height=4, aspect=1.2)
g.map_dataframe(sns.barplot, x='month', y='revenue',
                order=['Jan', 'Feb', 'Mar', 'Apr'],
                palette='Blues_d', errorbar=None)
g.set_axis_labels('Month', 'Revenue')
g.set_titles(col_template='{col_name}', row_template='{row_name}')
g.figure.suptitle('Revenue by Region and Product', y=1.02)
g.figure.savefig('/tmp/plot_19_facetgrid.png', dpi=100, bbox_inches='tight')
plt.close()
```

---

## 9. Plotly สำหรับ Interactive Charts

### Plotly คืออะไร?

**Plotly** เป็น library สำหรับ interactive charts
- Pan, zoom, hover tooltips
- เหมาะสำหรับ web dashboards
- ทำงานกับ Jupyter, Streamlit, Dash

```python
# ตัวอย่างที่ 20: Plotly basic setup
try:
    import plotly.graph_objects as go
    import plotly.io as pio
    import numpy as np

    # ตั้งค่า renderer สำหรับ non-notebook environment
    pio.renderers.default = 'browser'  # or 'svg', 'png', 'json'

    x = np.linspace(0, 4*np.pi, 200)

    # สร้าง figure
    fig = go.Figure()

    # เพิ่ม traces
    fig.add_trace(go.Scatter(
        x=x, y=np.sin(x),
        mode='lines',
        name='sin(x)',
        line=dict(color='blue', width=2)
    ))

    fig.add_trace(go.Scatter(
        x=x, y=np.cos(x),
        mode='lines',
        name='cos(x)',
        line=dict(color='red', width=2, dash='dash')
    ))

    # Layout
    fig.update_layout(
        title='Interactive Line Chart with Plotly',
        xaxis_title='x',
        yaxis_title='y',
        width=900,
        height=500,
        template='plotly_white',
        legend=dict(x=0.01, y=0.99)
    )

    # บันทึกเป็น HTML (interactive)
    fig.write_html('/tmp/plotly_01_line.html')
    print("Saved: plotly_01_line.html (open in browser for interactivity)")

    # บันทึกเป็น PNG (static)
    try:
        fig.write_image('/tmp/plotly_01_line.png')
        print("Saved: plotly_01_line.png")
    except Exception as e:
        print(f"PNG export needs kaleido: pip install kaleido ({e})")

except ImportError:
    print("Plotly not installed. Install with: pip install plotly")
```

```python
# ตัวอย่างที่ 21: Plotly interactive scatter
try:
    import plotly.graph_objects as go
    import pandas as pd
    import numpy as np

    rng = np.random.default_rng(42)
    n = 200

    df = pd.DataFrame({
        'x': rng.normal(0, 1, n),
        'y': None,
        'size': rng.integers(5, 30, n),
        'category': rng.choice(['A', 'B', 'C', 'D'], n),
        'info': [f'Point {i}' for i in range(n)]
    })
    df['y'] = df['x'] * 1.5 + rng.normal(0, 0.5, n)

    fig = go.Figure()

    colors = {'A': '#FF6B6B', 'B': '#4ECDC4', 'C': '#45B7D1', 'D': '#FFA07A'}

    for cat in df['category'].unique():
        mask = df['category'] == cat
        fig.add_trace(go.Scatter(
            x=df[mask]['x'],
            y=df[mask]['y'],
            mode='markers',
            name=f'Category {cat}',
            marker=dict(
                size=df[mask]['size'],
                color=colors[cat],
                opacity=0.7,
                line=dict(color='white', width=1)
            ),
            text=df[mask]['info'],
            hovertemplate='<b>%{text}</b><br>x=%{x:.2f}, y=%{y:.2f}<extra></extra>'
        ))

    fig.update_layout(
        title='Interactive Bubble Chart',
        xaxis_title='X',
        yaxis_title='Y',
        template='plotly_white',
        hovermode='closest'
    )

    fig.write_html('/tmp/plotly_02_scatter.html')
    print("Saved: plotly_02_scatter.html")

except ImportError:
    print("Plotly not installed")
```

```python
# ตัวอย่างที่ 22: Plotly subplots
try:
    import plotly.graph_objects as go
    from plotly.subplots import make_subplots
    import numpy as np
    import pandas as pd

    rng = np.random.default_rng(42)

    # สร้าง figure กับ subplots
    fig = make_subplots(
        rows=2, cols=2,
        subplot_titles=['Line Chart', 'Bar Chart', 'Scatter', 'Pie Chart'],
        specs=[[{'type': 'scatter'}, {'type': 'bar'}],
               [{'type': 'scatter'}, {'type': 'pie'}]]
    )

    # Row 1, Col 1: Line
    x = np.linspace(0, 10, 100)
    fig.add_trace(go.Scatter(x=x, y=np.sin(x), name='sin'), row=1, col=1)
    fig.add_trace(go.Scatter(x=x, y=np.cos(x), name='cos'), row=1, col=1)

    # Row 1, Col 2: Bar
    cats = ['A', 'B', 'C', 'D', 'E']
    vals = rng.integers(10, 100, 5)
    fig.add_trace(go.Bar(x=cats, y=vals, name='Revenue', marker_color='steelblue'), row=1, col=2)

    # Row 2, Col 1: Scatter
    x_s = rng.normal(0, 1, 100)
    y_s = x_s * 2 + rng.normal(0, 0.5, 100)
    fig.add_trace(go.Scatter(x=x_s, y=y_s, mode='markers', name='Data',
                             marker=dict(color='coral', opacity=0.6)), row=2, col=1)

    # Row 2, Col 2: Pie
    fig.add_trace(go.Pie(labels=cats, values=vals, name='Share'), row=2, col=2)

    fig.update_layout(height=700, title_text='Plotly Subplots Demo',
                      template='plotly_white', showlegend=True)

    fig.write_html('/tmp/plotly_03_subplots.html')
    print("Saved: plotly_03_subplots.html")

except ImportError:
    print("Plotly not installed")
```

---

## 10. Plotly Express

### Plotly Express - Simplified High-level API

```python
# ตัวอย่างที่ 23: Plotly Express basics
try:
    import plotly.express as px
    import pandas as pd
    import numpy as np

    rng = np.random.default_rng(42)
    n = 500

    df = pd.DataFrame({
        'x': rng.normal(0, 1, n),
        'y': None,
        'category': rng.choice(['A', 'B', 'C'], n),
        'size': rng.integers(5, 30, n),
        'color_val': rng.uniform(0, 100, n)
    })
    df['y'] = df['x'] * 2 + rng.normal(0, 0.8, n)

    # Line plot
    x = np.linspace(0, 10, 100)
    line_df = pd.DataFrame({
        'x': np.tile(x, 3),
        'y': np.concatenate([np.sin(x), np.cos(x), np.sin(2*x)]),
        'function': ['sin(x)'] * 100 + ['cos(x)'] * 100 + ['sin(2x)'] * 100
    })

    fig_line = px.line(line_df, x='x', y='y', color='function',
                       title='Plotly Express Line Chart',
                       template='plotly_white')
    fig_line.write_html('/tmp/px_01_line.html')
    print("Saved: px_01_line.html")

    # Scatter
    fig_scatter = px.scatter(df, x='x', y='y', color='category',
                             size='size', hover_data=['color_val'],
                             title='Plotly Express Scatter',
                             template='plotly_white',
                             trendline='ols')  # Ordinary Least Squares trendline
    fig_scatter.write_html('/tmp/px_02_scatter.html')
    print("Saved: px_02_scatter.html")

except ImportError:
    print("Plotly not installed")
```

```python
# ตัวอย่างที่ 24: Plotly Express - Statistical plots
try:
    import plotly.express as px
    import pandas as pd
    import numpy as np

    rng = np.random.default_rng(42)
    n = 400

    df = pd.DataFrame({
        'salary': np.concatenate([
            rng.normal(50000, 10000, n//4),
            rng.normal(65000, 12000, n//4),
            rng.normal(80000, 15000, n//4),
            rng.normal(95000, 20000, n//4)
        ]),
        'department': ['HR'] * (n//4) + ['IT'] * (n//4) +
                     ['Finance'] * (n//4) + ['Marketing'] * (n//4),
        'gender': rng.choice(['M', 'F'], n),
        'city': rng.choice(['Bangkok', 'CM', 'Phuket'], n)
    })

    # Box plot
    fig_box = px.box(df, x='department', y='salary', color='gender',
                     title='Salary Distribution by Department',
                     template='plotly_white',
                     notched=True)
    fig_box.write_html('/tmp/px_03_box.html')
    print("Saved: px_03_box.html")

    # Violin
    fig_violin = px.violin(df, x='department', y='salary', color='gender',
                           title='Salary Violin Plot',
                           template='plotly_white', box=True, points='outliers')
    fig_violin.write_html('/tmp/px_04_violin.html')
    print("Saved: px_04_violin.html")

    # Histogram
    fig_hist = px.histogram(df, x='salary', color='department',
                            title='Salary Distribution', nbins=30,
                            barmode='overlay', opacity=0.7,
                            template='plotly_white')
    fig_hist.write_html('/tmp/px_05_hist.html')
    print("Saved: px_05_hist.html")

    # Heatmap (correlation)
    corr = df[['salary']].assign(
        rand1=rng.normal(0, 1, n),
        rand2=rng.normal(0, 1, n)
    ).corr()
    fig_heat = px.imshow(corr, title='Correlation Heatmap',
                         color_continuous_scale='RdBu_r', text_auto=True)
    fig_heat.write_html('/tmp/px_06_heatmap.html')
    print("Saved: px_06_heatmap.html")

except ImportError:
    print("Plotly not installed")
```

```python
# ตัวอย่างที่ 25: Plotly Express - Advanced Charts
try:
    import plotly.express as px
    import pandas as pd
    import numpy as np

    rng = np.random.default_rng(42)

    # Sunburst Chart - hierarchical
    sales_data = pd.DataFrame({
        'continent': ['Asia'] * 4 + ['Europe'] * 3 + ['Americas'] * 3,
        'country': ['Thailand', 'Japan', 'China', 'India',
                    'Germany', 'UK', 'France',
                    'USA', 'Brazil', 'Canada'],
        'revenue': rng.integers(1000, 10000, 10)
    })

    fig_sun = px.sunburst(sales_data,
                          path=['continent', 'country'],
                          values='revenue',
                          title='Hierarchical Revenue (Sunburst)')
    fig_sun.write_html('/tmp/px_07_sunburst.html')
    print("Saved: px_07_sunburst.html")

    # Animated scatter (กับ time)
    n = 200
    time_df = pd.DataFrame({
        'x': rng.normal(0, 1, n),
        'y': rng.normal(0, 1, n),
        'category': rng.choice(['A', 'B', 'C'], n),
        'size': rng.integers(5, 20, n),
        'frame': rng.choice(['2022', '2023', '2024'], n)
    })

    fig_anim = px.scatter(time_df, x='x', y='y',
                          color='category', size='size',
                          animation_frame='frame',
                          title='Animated Scatter Chart',
                          template='plotly_white')
    fig_anim.write_html('/tmp/px_08_animated.html')
    print("Saved: px_08_animated.html")

except ImportError:
    print("Plotly not installed")
```

---

## 11. Bokeh เบื้องต้น

### Bokeh สำหรับ Web-based Visualization

```python
# ตัวอย่างที่ 26: Bokeh basic setup
try:
    from bokeh.plotting import figure, output_file, save
    from bokeh.models import ColumnDataSource, HoverTool
    from bokeh.layouts import row, column
    from bokeh.transform import factor_cmap
    from bokeh.palettes import Category10
    import numpy as np
    import pandas as pd

    output_file('/tmp/bokeh_01_basic.html')

    # สร้าง data
    x = np.linspace(0, 4*np.pi, 200)
    y1 = np.sin(x)
    y2 = np.cos(x)

    # สร้าง figure
    p = figure(
        title='Bokeh Line Chart',
        width=800, height=400,
        x_axis_label='x', y_axis_label='y',
        tools='pan,wheel_zoom,box_zoom,reset,save'
    )

    # เพิ่ม lines
    p.line(x, y1, legend_label='sin(x)', line_color='blue', line_width=2)
    p.line(x, y2, legend_label='cos(x)', line_color='red', line_width=2, line_dash='dashed')

    # Legend
    p.legend.location = 'top_right'
    p.legend.click_policy = 'hide'   # click to hide/show

    # Grid
    p.grid.grid_line_alpha = 0.3

    save(p)
    print("Saved: bokeh_01_basic.html (open in browser)")

except ImportError:
    print("Bokeh not installed. Install with: pip install bokeh")
```

```python
# ตัวอย่างที่ 27: Bokeh interactive scatter กับ hover tools
try:
    from bokeh.plotting import figure, output_file, save
    from bokeh.models import ColumnDataSource, HoverTool
    from bokeh.transform import factor_cmap
    from bokeh.palettes import Category10
    import numpy as np
    import pandas as pd

    output_file('/tmp/bokeh_02_scatter.html')

    rng = np.random.default_rng(42)
    n = 300
    categories = ['Category A', 'Category B', 'Category C']

    data = dict(
        x=rng.normal(0, 1, n),
        y=rng.normal(0, 1, n),
        category=rng.choice(categories, n),
        size=rng.integers(8, 25, n),
        info=[f'Point {i}' for i in range(n)]
    )
    data['y'] = [x*1.5 + rng.normal(0, 0.5) for x in data['x']]

    source = ColumnDataSource(data)

    # Hover tool
    hover = HoverTool(tooltips=[
        ('Info', '@info'),
        ('X', '@x{0.000}'),
        ('Y', '@y{0.000}'),
        ('Category', '@category')
    ])

    p = figure(
        title='Bokeh Interactive Scatter',
        width=800, height=500,
        tools=[hover, 'pan', 'wheel_zoom', 'box_zoom', 'reset', 'save']
    )

    p.circle(
        x='x', y='y',
        size='size',
        source=source,
        fill_color=factor_cmap('category', palette=Category10[3], factors=categories),
        fill_alpha=0.7,
        line_color='white',
        line_width=0.5,
        legend_field='category'
    )

    p.legend.location = 'top_left'
    p.legend.click_policy = 'hide'

    save(p)
    print("Saved: bokeh_02_scatter.html")

except ImportError:
    print("Bokeh not installed")
```

```python
# ตัวอย่างที่ 28: Bokeh widgets
try:
    from bokeh.plotting import figure, output_file, save
    from bokeh.models import ColumnDataSource, Slider, Select
    from bokeh.models.callbacks import CustomJS
    from bokeh.layouts import column
    import numpy as np

    output_file('/tmp/bokeh_03_widgets.html')

    # สร้าง data
    x = np.linspace(0, 4*np.pi, 200)
    y = np.sin(x)

    source = ColumnDataSource({'x': x, 'y': y})
    source_original = ColumnDataSource({'x': x})

    p = figure(title='Interactive Sine Wave',
               width=800, height=400,
               x_axis_label='x', y_axis_label='y')
    line = p.line('x', 'y', source=source, line_width=2, color='blue')

    # JavaScript callback (สำหรับ standalone HTML)
    callback = CustomJS(args=dict(source=source, source_orig=source_original), code="""
        const data = source.data;
        const orig = source_orig.data;
        const freq = cb_obj.value;
        const x = orig['x'];
        const y = Array.from(x).map(xi => Math.sin(freq * xi));
        data['y'] = y;
        source.change.emit();
    """)

    slider = Slider(start=0.1, end=5, value=1, step=0.1,
                    title="Frequency")
    slider.js_on_change('value', callback)

    layout = column(slider, p)
    save(layout)
    print("Saved: bokeh_03_widgets.html (interactive slider)")

except ImportError:
    print("Bokeh not installed")
```

---

## 12. Visualization Best Practices

### หลักการออกแบบ Visualization ที่ดี

```python
# ตัวอย่างที่ 29: Chart Junk vs Clean Design
import matplotlib.pyplot as plt
import numpy as np

months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun']
sales = [42, 55, 48, 63, 71, 68]

fig, axes = plt.subplots(1, 2, figsize=(16, 6))

# BAD: Chart Junk - ยัดเยียด elements มากเกินไป
ax_bad = axes[0]
ax_bad.bar(range(len(months)), sales, color=['red', 'blue', 'green', 'yellow', 'purple', 'orange'])
ax_bad.set_title('BAD: Cluttered Chart', fontsize=14, fontweight='bold', color='red')
ax_bad.set_xticks(range(len(months)))
ax_bad.set_xticklabels(months, rotation=45, fontsize=8)
ax_bad.set_xlabel('Month', fontsize=12)
ax_bad.set_ylabel('Sales (K THB)', fontsize=12)
# ปัญหา: 3D effect, gradients, too many colors, gridlines เยอะ
for i, (m, v) in enumerate(zip(months, sales)):
    ax_bad.text(i, v + 1, f'{v}K', ha='center', va='bottom', fontsize=8,
                bbox=dict(boxstyle='round', facecolor='yellow', alpha=0.5))
ax_bad.grid(True, axis='both', linewidth=1.5, color='black')
ax_bad.set_facecolor('#d9d9d9')
ax_bad.spines['top'].set_visible(True)
ax_bad.spines['right'].set_visible(True)
ax_bad.spines['left'].set_linewidth(2)
ax_bad.spines['bottom'].set_linewidth(2)

# GOOD: Clean design
ax_good = axes[1]
ax_good.bar(range(len(months)), sales, color='#3498db', alpha=0.8, width=0.6)
ax_good.set_title('GOOD: Clean Chart', fontsize=14, fontweight='bold', color='#2c3e50')
ax_good.set_xticks(range(len(months)))
ax_good.set_xticklabels(months)
ax_good.set_xlabel('Month', fontsize=11, color='#555')
ax_good.set_ylabel('Sales (K THB)', fontsize=11, color='#555')
ax_good.set_ylim(0, 85)
ax_good.grid(axis='y', alpha=0.3, linestyle='--')
ax_good.spines['top'].set_visible(False)
ax_good.spines['right'].set_visible(False)

# เพิ่มค่าบน bar อย่างสะอาด
for i, v in enumerate(sales):
    ax_good.text(i, v + 1, f'{v}K', ha='center', va='bottom',
                fontsize=10, color='#2c3e50', fontweight='bold')

fig.suptitle('Chart Design Comparison', fontsize=16)
fig.tight_layout()
fig.savefig('/tmp/plot_29_design.png', dpi=100, bbox_inches='tight')
plt.close()
```

```python
# ตัวอย่างที่ 30: Color accessibility (colorblind-friendly)
import matplotlib.pyplot as plt
import matplotlib.patches as mpatches
import numpy as np

# Colorblind-friendly palettes
# Wong palette (ดีสำหรับ colorblindness ทุกประเภท)
WONG_COLORS = ['#000000', '#E69F00', '#56B4E9', '#009E73',
               '#F0E442', '#0072B2', '#D55E00', '#CC79A7']

# Tableau colorblind-10
TABLEAU_CB = ['#006BA4', '#FF800E', '#ABABAB', '#595959',
              '#5F9ED1', '#C85200', '#898989', '#A2C8EC']

categories = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H']
values = [45, 62, 38, 71, 55, 48, 63, 42]

fig, axes = plt.subplots(1, 2, figsize=(16, 5))

# Wong palette
axes[0].bar(categories, values, color=WONG_COLORS)
axes[0].set_title('Wong Colorblind-Friendly Palette', fontsize=12)
axes[0].set_ylabel('Value')
axes[0].grid(axis='y', alpha=0.3)
axes[0].spines['top'].set_visible(False)
axes[0].spines['right'].set_visible(False)

# Tableau colorblind
axes[1].bar(categories, values, color=TABLEAU_CB)
axes[1].set_title('Tableau Colorblind Palette', fontsize=12)
axes[1].set_ylabel('Value')
axes[1].grid(axis='y', alpha=0.3)
axes[1].spines['top'].set_visible(False)
axes[1].spines['right'].set_visible(False)

fig.suptitle('Colorblind-Friendly Color Palettes', fontsize=14)
fig.tight_layout()
fig.savefig('/tmp/plot_30_colorblind.png', dpi=100, bbox_inches='tight')
plt.close()
```

```python
# ตัวอย่างที่ 31: Choosing the right chart type
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd

# Data สำหรับแต่ละ chart type
rng = np.random.default_rng(42)

fig = plt.figure(figsize=(20, 15))
fig.suptitle('Choosing the Right Chart Type', fontsize=16, y=0.98)

# 1. Comparison (Bar Chart) - ดีสำหรับเปรียบเทียบ categories
ax1 = fig.add_subplot(3, 3, 1)
cats = ['Electronics', 'Books', 'Clothing', 'Sports']
vals = [450, 200, 320, 180]
ax1.barh(cats, vals, color='steelblue', alpha=0.8)
ax1.set_title('1. Comparison\n(Horizontal Bar)')
ax1.grid(axis='x', alpha=0.3)

# 2. Trend (Line Chart) - ดีสำหรับ time series
ax2 = fig.add_subplot(3, 3, 2)
dates = pd.date_range('2024-01', periods=12, freq='ME')
trend = 100 + np.cumsum(rng.normal(3, 5, 12))
ax2.plot(dates, trend, 'b-o', markersize=5)
ax2.set_title('2. Trend Over Time\n(Line Chart)')
ax2.tick_params(axis='x', rotation=45)
ax2.grid(alpha=0.3)

# 3. Relationship (Scatter) - ดีสำหรับ correlation
ax3 = fig.add_subplot(3, 3, 3)
x = rng.normal(0, 1, 100)
y = x * 2 + rng.normal(0, 0.5, 100)
ax3.scatter(x, y, alpha=0.6, c='coral')
ax3.set_title('3. Relationship\n(Scatter)')
ax3.grid(alpha=0.3)

# 4. Distribution (Histogram) - ดีสำหรับดู spread
ax4 = fig.add_subplot(3, 3, 4)
data = rng.normal(50, 15, 500)
ax4.hist(data, bins=25, color='mediumseagreen', alpha=0.7, edgecolor='white')
ax4.set_title('4. Distribution\n(Histogram)')
ax4.grid(axis='y', alpha=0.3)

# 5. Composition (Pie) - ดีสำหรับ part of whole
ax5 = fig.add_subplot(3, 3, 5)
pie_vals = [35, 25, 20, 15, 5]
labels = ['A', 'B', 'C', 'D', 'Other']
ax5.pie(pie_vals, labels=labels, autopct='%1.0f%%', startangle=90,
        colors=['#FF6B6B', '#4ECDC4', '#45B7D1', '#FFA07A', '#98D8C8'])
ax5.set_title('5. Composition\n(Pie Chart)')

# 6. Comparison across groups (Grouped Bar)
ax6 = fig.add_subplot(3, 3, 6)
departments = ['HR', 'IT', 'Finance']
m_vals = [55000, 72000, 68000]
f_vals = [52000, 68000, 65000]
x = np.arange(len(departments))
ax6.bar(x - 0.2, m_vals, 0.4, label='Male', color='steelblue', alpha=0.8)
ax6.bar(x + 0.2, f_vals, 0.4, label='Female', color='coral', alpha=0.8)
ax6.set_title('6. Multi-group Compare\n(Grouped Bar)')
ax6.set_xticks(x)
ax6.set_xticklabels(departments)
ax6.legend()
ax6.grid(axis='y', alpha=0.3)

# 7. Heatmap - ดีสำหรับ matrix data
ax7 = fig.add_subplot(3, 3, 7)
matrix = rng.uniform(0, 1, (5, 5))
im = ax7.imshow(matrix, cmap='YlOrRd', aspect='auto')
plt.colorbar(im, ax=ax7)
ax7.set_title('7. Matrix/Correlation\n(Heatmap)')

# 8. Time series with confidence interval
ax8 = fig.add_subplot(3, 3, 8)
t = np.linspace(0, 10, 100)
mean = np.sin(t) * np.exp(-0.1*t)
std = 0.1 * np.exp(0.05*t)
ax8.plot(t, mean, 'b-', linewidth=2)
ax8.fill_between(t, mean-std, mean+std, alpha=0.3, color='blue')
ax8.set_title('8. Uncertainty\n(Line + Confidence Interval)')
ax8.grid(alpha=0.3)

# 9. Area chart - stacked comparison
ax9 = fig.add_subplot(3, 3, 9)
x_area = np.linspace(0, 10, 50)
y1 = rng.uniform(10, 30, 50)
y2 = rng.uniform(15, 35, 50)
y3 = rng.uniform(5, 20, 50)
ax9.stackplot(x_area, y1, y2, y3,
              labels=['Product A', 'Product B', 'Product C'],
              alpha=0.7, colors=['#FF6B6B', '#4ECDC4', '#45B7D1'])
ax9.legend(loc='upper left', fontsize=8)
ax9.set_title('9. Part of Whole Over Time\n(Stacked Area)')

fig.tight_layout()
fig.savefig('/tmp/plot_31_chart_types.png', dpi=100, bbox_inches='tight')
plt.close()
print("Saved: chart_types comparison")
```

---

## 13. แบบฝึกหัด

### ข้อ 1: Matplotlib - Complete Dashboard
สร้าง 4-panel dashboard แสดงข้อมูล sales

```python
# เฉลยข้อ 1
import matplotlib.pyplot as plt
import matplotlib.gridspec as gridspec
import numpy as np
import pandas as pd

rng = np.random.default_rng(42)

# สร้างข้อมูล
months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun',
          'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
monthly_revenue = [125, 148, 132, 165, 178, 155, 190, 210, 188, 225, 248, 265]
monthly_target = [130, 140, 145, 155, 170, 165, 185, 200, 195, 220, 240, 260]
product_sales = {'Laptop': 450, 'Phone': 680, 'Tablet': 320, 'Watch': 215, 'Headphones': 390}
regions = ['North', 'South', 'East', 'West', 'Central']
region_values = rng.integers(100, 500, 5)
daily = pd.date_range('2024-01-01', '2024-12-31', freq='D')
daily_sales = pd.Series(
    100 + 20*np.sin(2*np.pi*np.arange(366)/365) + rng.normal(0, 10, 366),
    index=daily
)

# สร้าง Dashboard
fig = plt.figure(figsize=(18, 12))
fig.patch.set_facecolor('#f8f9fa')
gs = gridspec.GridSpec(2, 2, figure=fig, hspace=0.35, wspace=0.3)

colors_brand = ['#2196F3', '#4CAF50', '#FF9800', '#E91E63', '#9C27B0']

# Panel 1: Monthly Revenue vs Target
ax1 = fig.add_subplot(gs[0, 0])
x = np.arange(len(months))
width = 0.35
bars1 = ax1.bar(x - width/2, monthly_revenue, width, color='#2196F3',
                 alpha=0.85, label='Actual', zorder=3)
bars2 = ax1.bar(x + width/2, monthly_target, width, color='#90CAF9',
                 alpha=0.85, label='Target', zorder=3)
ax1.set_title('Monthly Revenue vs Target (M THB)', fontsize=12, fontweight='bold', pad=10)
ax1.set_xticks(x)
ax1.set_xticklabels([m[:3] for m in months], fontsize=9)
ax1.legend(fontsize=9)
ax1.grid(axis='y', alpha=0.3, zorder=0)
ax1.spines['top'].set_visible(False)
ax1.spines['right'].set_visible(False)

# Panel 2: Product Sales Pie
ax2 = fig.add_subplot(gs[0, 1])
wedges, texts, autotexts = ax2.pie(
    list(product_sales.values()),
    labels=list(product_sales.keys()),
    autopct='%1.1f%%',
    colors=colors_brand,
    startangle=90,
    wedgeprops=dict(linewidth=1.5, edgecolor='white')
)
for autotext in autotexts:
    autotext.set_fontsize(9)
ax2.set_title('Product Mix (Units Sold)', fontsize=12, fontweight='bold', pad=10)

# Panel 3: Region Horizontal Bar
ax3 = fig.add_subplot(gs[1, 0])
sorted_idx = np.argsort(region_values)
sorted_regions = [regions[i] for i in sorted_idx]
sorted_values = region_values[sorted_idx]
bars = ax3.barh(sorted_regions, sorted_values, color='#4CAF50', alpha=0.8, height=0.6)
for bar, val in zip(bars, sorted_values):
    ax3.text(val + 5, bar.get_y() + bar.get_height()/2,
             f'{val}K', va='center', fontsize=10, fontweight='bold')
ax3.set_title('Revenue by Region (K THB)', fontsize=12, fontweight='bold', pad=10)
ax3.set_xlabel('Revenue (K THB)')
ax3.grid(axis='x', alpha=0.3)
ax3.spines['top'].set_visible(False)
ax3.spines['right'].set_visible(False)

# Panel 4: Daily Sales Trend
ax4 = fig.add_subplot(gs[1, 1])
ma30 = daily_sales.rolling(30).mean()
ax4.plot(daily_sales.index, daily_sales.values, color='#2196F3', alpha=0.3, linewidth=0.8)
ax4.plot(ma30.index, ma30.values, color='#E91E63', linewidth=2, label='30-day MA')
ax4.fill_between(daily_sales.index, daily_sales.values, alpha=0.1, color='#2196F3')
ax4.set_title('Daily Sales Trend 2024', fontsize=12, fontweight='bold', pad=10)
ax4.set_xlabel('Month')
ax4.set_ylabel('Sales (K THB)')
ax4.legend(fontsize=9)
ax4.grid(alpha=0.2)
ax4.spines['top'].set_visible(False)
ax4.spines['right'].set_visible(False)
ax4.tick_params(axis='x', rotation=30)

# Title
fig.suptitle('Sales Analytics Dashboard 2024', fontsize=18, fontweight='bold',
             color='#2c3e50', y=0.98)

fig.savefig('/tmp/plot_ex1_dashboard.png', dpi=120, bbox_inches='tight',
            facecolor=fig.get_facecolor())
plt.close()
print("Saved: dashboard")
```

### ข้อ 2: Seaborn Statistical Analysis
วิเคราะห์ distribution ของ employee data

```python
# เฉลยข้อ 2
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

sns.set_theme(style='whitegrid', palette='husl')
rng = np.random.default_rng(42)
n = 400

df = pd.DataFrame({
    'salary': np.concatenate([
        rng.normal(45000, 8000, n//4),
        rng.normal(65000, 10000, n//4),
        rng.normal(82000, 12000, n//4),
        rng.normal(95000, 15000, n//4)
    ]),
    'department': ['HR'] * (n//4) + ['IT'] * (n//4) +
                  ['Finance'] * (n//4) + ['Marketing'] * (n//4),
    'gender': rng.choice(['M', 'F'], n),
    'experience': rng.integers(1, 20, n),
    'performance': rng.uniform(2.5, 5.0, n).round(1)
})

fig, axes = plt.subplots(2, 3, figsize=(18, 12))

# 1. Distribution of salary
sns.histplot(df['salary'], bins=30, kde=True, ax=axes[0,0], color='steelblue')
axes[0,0].set_title('Salary Distribution with KDE')
axes[0,0].set_xlabel('Salary (THB)')

# 2. Boxplot by department
sns.boxplot(data=df, x='department', y='salary', ax=axes[0,1], palette='Set2')
axes[0,1].set_title('Salary by Department (Boxplot)')
axes[0,1].tick_params(axis='x', rotation=20)

# 3. Violin by dept + gender
sns.violinplot(data=df, x='department', y='salary', hue='gender',
               ax=axes[0,2], split=True, palette='Set1', inner='quart')
axes[0,2].set_title('Salary by Dept + Gender (Violin)')
axes[0,2].tick_params(axis='x', rotation=20)

# 4. Experience vs Salary scatter
sns.scatterplot(data=df, x='experience', y='salary', hue='department',
                alpha=0.6, ax=axes[1,0])
# Add regression line
for dept, color in zip(df['department'].unique(), sns.color_palette('husl', 4)):
    subset = df[df['department'] == dept]
    z = np.polyfit(subset['experience'], subset['salary'], 1)
    p = np.poly1d(z)
    x_range = np.linspace(subset['experience'].min(), subset['experience'].max(), 50)
    axes[1,0].plot(x_range, p(x_range), '--', alpha=0.7, linewidth=1.5)
axes[1,0].set_title('Experience vs Salary by Department')

# 5. Heatmap: correlation
corr = df[['salary', 'experience', 'performance']].corr()
sns.heatmap(corr, annot=True, fmt='.3f', cmap='coolwarm',
            vmin=-1, vmax=1, ax=axes[1,1], square=True,
            linewidths=1, linecolor='white')
axes[1,1].set_title('Correlation Heatmap')

# 6. Performance distribution by department
sns.kdeplot(data=df, x='performance', hue='department',
            fill=True, alpha=0.4, ax=axes[1,2])
axes[1,2].set_title('Performance Distribution by Department')

fig.suptitle('Employee Analytics - Statistical Visualization', fontsize=16)
fig.tight_layout()
fig.savefig('/tmp/plot_ex2_seaborn.png', dpi=100, bbox_inches='tight')
plt.close()
print("Saved: seaborn analysis")
```

### ข้อ 3: Plotly Express Dashboard
สร้าง interactive dashboard ด้วย Plotly Express

```python
# เฉลยข้อ 3
try:
    import plotly.express as px
    import plotly.graph_objects as go
    from plotly.subplots import make_subplots
    import pandas as pd
    import numpy as np

    rng = np.random.default_rng(42)
    n = 500

    df = pd.DataFrame({
        'date': pd.date_range('2024-01-01', periods=n, freq='D')[:n],
        'product': rng.choice(['Laptop', 'Phone', 'Tablet'], n),
        'region': rng.choice(['North', 'South', 'East', 'West'], n),
        'revenue': rng.integers(1000, 20000, n).astype(float),
        'quantity': rng.integers(1, 20, n)
    })
    df['month'] = df['date'].dt.strftime('%Y-%m')
    df['unit_price'] = (df['revenue'] / df['quantity']).round(2)

    # สร้าง subplots
    fig = make_subplots(
        rows=2, cols=2,
        subplot_titles=['Revenue Trend', 'Revenue by Product',
                        'Revenue Distribution', 'Revenue by Region'],
        specs=[[{'type': 'scatter'}, {'type': 'bar'}],
               [{'type': 'histogram'}, {'type': 'pie'}]]
    )

    # 1. Revenue trend
    monthly = df.groupby(['month', 'product'])['revenue'].sum().reset_index()
    for product in df['product'].unique():
        prod_data = monthly[monthly['product'] == product]
        fig.add_trace(go.Scatter(x=prod_data['month'], y=prod_data['revenue'],
                                 name=product, mode='lines+markers'),
                      row=1, col=1)

    # 2. Revenue by product (bar)
    prod_total = df.groupby('product')['revenue'].sum().reset_index()
    fig.add_trace(go.Bar(x=prod_total['product'], y=prod_total['revenue'],
                         name='Revenue', showlegend=False,
                         marker_color=['#FF6B6B', '#4ECDC4', '#45B7D1']),
                  row=1, col=2)

    # 3. Distribution
    fig.add_trace(go.Histogram(x=df['revenue'], nbinsx=30, name='Revenue',
                               showlegend=False, marker_color='steelblue'),
                  row=2, col=1)

    # 4. Pie by region
    region_total = df.groupby('region')['revenue'].sum().reset_index()
    fig.add_trace(go.Pie(labels=region_total['region'],
                         values=region_total['revenue'],
                         name='Region', showlegend=True),
                  row=2, col=2)

    fig.update_layout(
        height=700,
        title_text='Interactive Sales Dashboard 2024',
        template='plotly_white',
        hovermode='x unified'
    )

    fig.write_html('/tmp/plotly_ex3_dashboard.html')
    print("Saved: interactive dashboard")

except ImportError:
    print("Plotly not installed")
```

### ข้อ 4: Time Series Visualization
Visualize stock data พร้อม technical indicators

```python
# เฉลยข้อ 4
import matplotlib.pyplot as plt
import matplotlib.gridspec as gridspec
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)
n = 252  # trading days

dates = pd.date_range('2024-01-01', periods=n, freq='B')
returns = rng.normal(0.0005, 0.015, n)
price = pd.Series(100 * (1 + returns).cumprod(), index=dates)
volume = pd.Series(rng.integers(500000, 2000000, n), index=dates)

# Indicators
ma20 = price.rolling(20).mean()
ma50 = price.rolling(50).mean()
bb_std = price.rolling(20).std()
bb_upper = ma20 + 2 * bb_std
bb_lower = ma20 - 2 * bb_std

# RSI
delta = price.diff()
gain = delta.where(delta > 0, 0).rolling(14).mean()
loss = (-delta.where(delta < 0, 0)).rolling(14).mean()
rsi = 100 - (100 / (1 + gain / loss))

# MACD
ema12 = price.ewm(span=12).mean()
ema26 = price.ewm(span=26).mean()
macd = ema12 - ema26
signal = macd.ewm(span=9).mean()

fig = plt.figure(figsize=(16, 14))
gs = gridspec.GridSpec(4, 1, figure=fig, hspace=0.05,
                       height_ratios=[3, 1, 1, 1])

# Panel 1: Price + Moving Averages + Bollinger Bands
ax1 = fig.add_subplot(gs[0])
ax1.fill_between(dates, bb_lower, bb_upper, alpha=0.1, color='gray', label='Bollinger Bands')
ax1.plot(dates, price, 'k-', linewidth=1, label='Price', alpha=0.9)
ax1.plot(dates, ma20, '#FF6B6B', linewidth=1.5, label='MA20', alpha=0.8)
ax1.plot(dates, ma50, '#4ECDC4', linewidth=1.5, label='MA50', alpha=0.8)
ax1.plot(dates, bb_upper, '--', color='gray', linewidth=0.8, alpha=0.5)
ax1.plot(dates, bb_lower, '--', color='gray', linewidth=0.8, alpha=0.5)
ax1.set_ylabel('Price (THB)', fontsize=11)
ax1.legend(loc='upper left', fontsize=9)
ax1.grid(alpha=0.2)
ax1.set_title('Stock Technical Analysis Dashboard', fontsize=14, fontweight='bold')
ax1.tick_params(labelbottom=False)

# Panel 2: Volume
ax2 = fig.add_subplot(gs[1], sharex=ax1)
colors = ['#4CAF50' if r >= 0 else '#F44336' for r in returns]
ax2.bar(dates, volume, color=colors, alpha=0.7, width=0.8)
ax2.set_ylabel('Volume', fontsize=11)
ax2.grid(alpha=0.2)
ax2.tick_params(labelbottom=False)

# Panel 3: RSI
ax3 = fig.add_subplot(gs[2], sharex=ax1)
ax3.plot(dates, rsi, '#9C27B0', linewidth=1.5)
ax3.axhline(70, color='red', linestyle='--', alpha=0.5, linewidth=1)
ax3.axhline(30, color='green', linestyle='--', alpha=0.5, linewidth=1)
ax3.fill_between(dates, 70, rsi.where(rsi > 70), alpha=0.3, color='red')
ax3.fill_between(dates, 30, rsi.where(rsi < 30), alpha=0.3, color='green')
ax3.set_ylabel('RSI(14)', fontsize=11)
ax3.set_ylim(0, 100)
ax3.grid(alpha=0.2)
ax3.tick_params(labelbottom=False)

# Panel 4: MACD
ax4 = fig.add_subplot(gs[3], sharex=ax1)
macd_hist = macd - signal
ax4.bar(dates, macd_hist, color=['#4CAF50' if v >= 0 else '#F44336' for v in macd_hist],
        alpha=0.7, width=0.8)
ax4.plot(dates, macd, '#2196F3', linewidth=1.5, label='MACD')
ax4.plot(dates, signal, '#FF9800', linewidth=1.5, label='Signal')
ax4.axhline(0, color='gray', linewidth=0.5)
ax4.set_ylabel('MACD', fontsize=11)
ax4.legend(loc='upper left', fontsize=9)
ax4.grid(alpha=0.2)
ax4.tick_params(axis='x', rotation=30)

fig.savefig('/tmp/plot_ex4_stock.png', dpi=100, bbox_inches='tight')
plt.close()
print("Saved: stock technical analysis")
```

### ข้อ 5-10: Additional exercises

```python
# ข้อ 5: Geographic heatmap (choropleth ด้วย Plotly)
# ข้อ 6: Animated chart (Plotly animation)
# ข้อ 7: Custom Seaborn theme
# ข้อ 8: Multi-variable comparison
# ข้อ 9: Bokeh dashboard
# ข้อ 10: Complete visualization report

# เฉลยข้อ 10: Complete Visualization Report
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
import pandas as pd

# สร้าง complete dataset
rng = np.random.default_rng(42)
n = 500

df = pd.DataFrame({
    'date': pd.date_range('2024-01-01', periods=n, freq='D')[:n],
    'product': rng.choice(['A', 'B', 'C'], n),
    'region': rng.choice(['North', 'South', 'East'], n),
    'revenue': rng.integers(500, 15000, n).astype(float),
    'cost': None,
    'units': rng.integers(1, 50, n),
    'customer_satisfaction': rng.uniform(2, 5, n).round(1)
})
df['cost'] = df['revenue'] * rng.uniform(0.4, 0.7, n)
df['profit'] = df['revenue'] - df['cost']
df['margin'] = df['profit'] / df['revenue']
df['month'] = df['date'].dt.strftime('%b')

# 3x3 report
fig, axes = plt.subplots(3, 3, figsize=(18, 15))
sns.set_theme(style='whitegrid')

# Row 1
# 1.1 Revenue trend
monthly_rev = df.groupby('month')['revenue'].mean()
month_order = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun',
               'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
month_order = [m for m in month_order if m in monthly_rev.index]
axes[0,0].plot(range(len(month_order)), monthly_rev[month_order].values, 'o-', color='steelblue')
axes[0,0].set_xticks(range(len(month_order)))
axes[0,0].set_xticklabels(month_order, rotation=45, fontsize=8)
axes[0,0].set_title('Avg Daily Revenue by Month')
axes[0,0].grid(alpha=0.3)

# 1.2 Revenue by product
sns.boxplot(data=df, x='product', y='revenue', ax=axes[0,1], palette='Set2')
axes[0,1].set_title('Revenue Distribution by Product')

# 1.3 Profit margin distribution
sns.histplot(df['margin'], bins=30, kde=True, ax=axes[0,2], color='coral')
axes[0,2].set_title('Profit Margin Distribution')
axes[0,2].axvline(df['margin'].mean(), color='red', linestyle='--', label=f"Mean={df['margin'].mean():.2f}")
axes[0,2].legend(fontsize=9)

# Row 2
# 2.1 Revenue vs cost scatter
sns.scatterplot(data=df, x='cost', y='revenue', hue='product', alpha=0.5, ax=axes[1,0])
axes[1,0].set_title('Revenue vs Cost by Product')

# 2.2 Regional comparison
region_stats = df.groupby('region')['revenue'].agg(['mean', 'std']).reset_index()
axes[1,1].bar(region_stats['region'], region_stats['mean'],
              yerr=region_stats['std'], capsize=5,
              color=['#FF6B6B', '#4ECDC4', '#45B7D1'], alpha=0.8)
axes[1,1].set_title('Avg Revenue by Region (with Std)')

# 2.3 Satisfaction vs Margin
sns.scatterplot(data=df, x='customer_satisfaction', y='margin',
                hue='product', alpha=0.5, ax=axes[1,2])
axes[1,2].set_title('Satisfaction vs Profit Margin')

# Row 3
# 3.1 Heatmap: revenue by product x region
pivot = df.pivot_table(values='revenue', index='product',
                       columns='region', aggfunc='mean')
sns.heatmap(pivot, annot=True, fmt='.0f', cmap='YlGnBu', ax=axes[2,0])
axes[2,0].set_title('Avg Revenue: Product × Region')

# 3.2 Units vs Revenue
sns.scatterplot(data=df, x='units', y='revenue', alpha=0.3, ax=axes[2,1])
z = np.polyfit(df['units'], df['revenue'], 1)
p = np.poly1d(z)
x_r = np.linspace(df['units'].min(), df['units'].max(), 50)
axes[2,1].plot(x_r, p(x_r), 'r--', linewidth=2, label='Trend')
axes[2,1].legend()
axes[2,1].set_title('Units vs Revenue')

# 3.3 Profit margin by product+region
pivot2 = df.pivot_table(values='margin', index='product',
                        columns='region', aggfunc='mean')
sns.heatmap(pivot2, annot=True, fmt='.3f', cmap='RdYlGn',
            vmin=0, vmax=0.7, ax=axes[2,2])
axes[2,2].set_title('Avg Margin: Product × Region')

fig.suptitle('Complete Sales Analytics Report', fontsize=18, fontweight='bold', y=1.01)
fig.tight_layout()
fig.savefig('/tmp/plot_ex10_report.png', dpi=100, bbox_inches='tight')
plt.close()
print("Saved: complete visualization report")
```

---

## สรุป Part 75

ใน Part นี้เราได้เรียนรู้ Data Visualization ด้วย 4 libraries:

| Library | จุดเด่น | Use Cases |
|---------|---------|-----------|
| **Matplotlib** | ควบคุมสูงสุด, ยืดหยุ่น | Publication-quality, custom charts |
| **Seaborn** | Statistical plots, สวยงาม | EDA, statistical analysis |
| **Plotly** | Interactive, web-friendly | Dashboards, presentations |
| **Bokeh** | Web visualization, widgets | Web apps, streaming data |

### สรุป Topics ที่สำคัญ

| หัวข้อ | สิ่งที่เรียนรู้ |
|--------|----------------|
| Matplotlib | Figure/Axes, OO API, plot types, GridSpec |
| Line/Scatter/Bar | Styles, colors, markers, value labels |
| Histograms | bins, density, KDE overlay |
| Subplots | subplots(), GridSpec, shared axes |
| Customization | styles, colors, annotations, text |
| Seaborn | histplot, scatterplot, boxplot, violin |
| Statistical | KDE, ECDF, pairplot, FacetGrid |
| Heatmaps | correlation, mask, colormap |
| Plotly | Figure, traces, interactive |
| Plotly Express | px.scatter, px.box, px.histogram |
| Bokeh | figure, tools, ColumnDataSource, widgets |
| Best Practices | chart selection, color, clarity |

### Visualization Rules of Thumb

1. **เลือก chart type ที่เหมาะสม**: comparison→bar, trend→line, distribution→histogram, relation→scatter
2. **Less is more**: ลด chart junk (gridlines เยอะ, 3D, rainbow colors)
3. **Colorblind-friendly**: ใช้ Wong palette หรือ perceptually uniform colormaps
4. **Data-ink ratio**: เพิ่ม data ลด non-data ink
5. **Label everything**: axes, title, units, legend
6. **Consistent scale**: ระวัง truncated y-axis ที่ทำให้เข้าใจผิด
7. **Interactive สำหรับ exploration, static สำหรับ publication**

---

**ก่อนหน้า**: [Part 74 - Pandas Advanced Analytics](../part74/README.md)
**ต่อไป**: [Part 76 - Machine Learning Introduction](../part76/README.md)
