# Part 74 - Pandas: Advanced Analytics (การวิเคราะห์ขั้นสูงด้วย Pandas)

## สารบัญ
1. [Time Series Analysis](#1-time-series-analysis)
2. [Resampling and Rolling Windows](#2-resampling-and-rolling-windows)
3. [MultiIndex DataFrames](#3-multiindex-dataframes)
4. [Advanced Groupby](#4-advanced-groupby)
5. [Window Functions](#5-window-functions)
6. [Cross-tabulations](#6-cross-tabulations)
7. [Memory Optimization](#7-memory-optimization)
8. [Chunked Processing](#8-chunked-processing)
9. [Pandas with Databases](#9-pandas-with-databases)
10. [Apply with Progress (tqdm)](#10-apply-with-progress-tqdm)
11. [Performance Profiling](#11-performance-profiling)
12. [ตัวอย่างโปรแกรมจริง](#12-ตัวอย่างโปรแกรมจริง)
13. [แบบฝึกหัด](#13-แบบฝึกหัด)

---

## 1. Time Series Analysis

### Time Series ใน Pandas

Pandas มี support ที่แข็งแกร่งสำหรับ time series data:
- `DatetimeIndex`: index ที่เป็น timestamps
- `Period`: ช่วงเวลา (month, quarter, year)
- `Timedelta`: ระยะเวลา
- resample, rolling, shift operations

```python
# ตัวอย่างที่ 1: DatetimeIndex และ datetime operations
import pandas as pd
import numpy as np

# สร้าง time series
rng = np.random.default_rng(42)
dates = pd.date_range('2024-01-01', periods=365, freq='D')

ts = pd.Series(
    100 + 20 * np.sin(2 * np.pi * np.arange(365) / 365) +  # seasonal
    0.1 * np.arange(365) +                                   # trend
    rng.normal(0, 3, 365),                                   # noise
    index=dates,
    name='daily_sales'
)

print("Time Series (first 10 days):")
print(ts.head(10))

# Properties ของ DatetimeIndex
print(f"\nStart: {ts.index[0]}")
print(f"End: {ts.index[-1]}")
print(f"Frequency: {ts.index.freq}")

# เข้าถึงด้วย datetime
print(f"\nJanuary 15:", ts['2024-01-15'])
print(f"All January:", ts['2024-01'].head())
print(f"Q1 (Jan-Mar):", ts['2024-01':'2024-03-31'].count(), "days")
```

```python
# ตัวอย่างที่ 2: Datetime properties
import pandas as pd
import numpy as np

# สร้าง DataFrame กับ datetime column
df = pd.DataFrame({
    'timestamp': pd.date_range('2024-01-01', periods=100, freq='6H'),
    'value': np.random.default_rng(42).normal(100, 15, 100)
})

# Datetime accessor (.dt)
df['year'] = df['timestamp'].dt.year
df['month'] = df['timestamp'].dt.month
df['day'] = df['timestamp'].dt.day
df['hour'] = df['timestamp'].dt.hour
df['day_of_week'] = df['timestamp'].dt.day_of_week  # 0=Monday
df['day_name'] = df['timestamp'].dt.day_name()
df['is_weekend'] = df['timestamp'].dt.day_of_week >= 5
df['quarter'] = df['timestamp'].dt.quarter
df['week'] = df['timestamp'].dt.isocalendar().week

print("Datetime properties:")
print(df[['timestamp', 'year', 'month', 'day', 'hour',
          'day_name', 'is_weekend', 'quarter']].head(8))
```

```python
# ตัวอย่างที่ 3: Timezone handling
import pandas as pd

# Create timezone-aware datetime
ts_utc = pd.date_range('2024-01-01', periods=5, freq='D', tz='UTC')
print("UTC:", ts_utc)

# Convert timezone
ts_bkk = ts_utc.tz_convert('Asia/Bangkok')
print("\nBangkok:", ts_bkk)

ts_ny = ts_utc.tz_convert('America/New_York')
print("New York:", ts_ny)

# Localize naive datetime
ts_naive = pd.date_range('2024-01-01', periods=5, freq='D')
ts_localized = ts_naive.tz_localize('Asia/Bangkok')
print("\nLocalized:", ts_localized)
```

```python
# ตัวอย่างที่ 4: Date arithmetic
import pandas as pd
import numpy as np

# Timedelta operations
df = pd.DataFrame({
    'start_date': pd.to_datetime(['2024-01-01', '2024-02-15', '2024-03-10']),
    'end_date': pd.to_datetime(['2024-06-30', '2024-09-20', '2024-12-25'])
})

# คำนวณ duration
df['duration_days'] = (df['end_date'] - df['start_date']).dt.days
df['duration_weeks'] = df['duration_days'] / 7
df['duration_months'] = df['duration_days'] / 30.44  # approximate

print("Date arithmetic:")
print(df)

# เพิ่ม/ลด dates
df['one_month_later'] = df['start_date'] + pd.DateOffset(months=1)
df['two_weeks_later'] = df['start_date'] + pd.Timedelta(weeks=2)
print("\nDate offsets:")
print(df[['start_date', 'one_month_later', 'two_weeks_later']])

# BusinessDay offset
from pandas.tseries.offsets import BDay
df['next_business_day'] = df['start_date'] + BDay(1)
print("\nNext business day:")
print(df[['start_date', 'next_business_day']])
```

---

## 2. Resampling and Rolling Windows

### Resampling - เปลี่ยน Frequency ของ Time Series

```python
# ตัวอย่างที่ 5: resample - รวมข้อมูลตาม time period
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)
# Hourly data for 30 days
dates = pd.date_range('2024-01-01', periods=720, freq='H')  # 30 days * 24 hours
hourly_sales = pd.Series(
    rng.integers(10, 100, 720),
    index=dates,
    name='hourly_sales'
)

# Resample to different frequencies
daily = hourly_sales.resample('D').sum()
print("Daily totals:")
print(daily.head(7))

weekly = hourly_sales.resample('W').sum()
print("\nWeekly totals:")
print(weekly.head())

monthly = hourly_sales.resample('ME').agg({
    'hourly_sales': ['sum', 'mean', 'max', 'min']
})
print("\nMonthly summary:")
print(monthly)
```

```python
# ตัวอย่างที่ 6: Upsampling - เพิ่ม frequency (interpolation)
import pandas as pd
import numpy as np

# Monthly data
monthly_data = pd.Series(
    [100, 120, 110, 130, 140, 125, 150, 160, 145, 170, 180, 165],
    index=pd.date_range('2024-01-01', periods=12, freq='MS'),  # month start
    name='monthly_revenue'
)

# Upsample to daily (fills with NaN by default)
daily = monthly_data.resample('D').asfreq()
print(f"Monthly -> Daily (first 10): {daily.head(10).tolist()}")

# Fill methods
daily_ffill = monthly_data.resample('D').ffill()   # forward fill
daily_bfill = monthly_data.resample('D').bfill()   # backward fill
daily_interp = monthly_data.resample('D').interpolate('linear')  # interpolate

print(f"\nForward fill (first 5 Feb days): {daily_ffill['2024-02-01':'2024-02-05'].tolist()}")
print(f"Interpolated Feb: {daily_interp['2024-02-01':'2024-02-05'].round(2).tolist()}")
```

```python
# ตัวอย่างที่ 7: Rolling windows
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)
dates = pd.date_range('2024-01-01', periods=100, freq='D')
prices = pd.Series(
    100 + np.cumsum(rng.normal(0, 1, 100)),
    index=dates,
    name='price'
)

# Rolling mean (moving average)
ma7 = prices.rolling(window=7).mean()    # 7-day MA
ma30 = prices.rolling(window=30).mean()  # 30-day MA

print("Price with Moving Averages:")
df = pd.DataFrame({'price': prices, 'MA7': ma7, 'MA30': ma30})
print(df.tail(10).round(2))

# Rolling statistics
df['rolling_std'] = prices.rolling(window=7).std()
df['rolling_max'] = prices.rolling(window=7).max()
df['rolling_min'] = prices.rolling(window=7).min()
df['bollinger_upper'] = ma7 + 2 * df['rolling_std']
df['bollinger_lower'] = ma7 - 2 * df['rolling_std']

print("\nBollinger Bands (last 5 days):")
print(df[['price', 'MA7', 'bollinger_upper', 'bollinger_lower']].tail(5).round(2))
```

```python
# ตัวอย่างที่ 8: Expanding windows
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)
daily_returns = pd.Series(
    rng.normal(0.001, 0.02, 252),  # daily stock returns
    index=pd.date_range('2024-01-01', periods=252, freq='B'),
    name='daily_return'
)

# Expanding mean/std (all-time running stats)
running_mean = daily_returns.expanding().mean()
running_std = daily_returns.expanding().std()
running_sharpe = running_mean / running_std * np.sqrt(252)  # annualized Sharpe

# Cumulative product (total return)
cumulative_return = (1 + daily_returns).cumprod() - 1

print("Portfolio Performance:")
df = pd.DataFrame({
    'daily_return': daily_returns,
    'cumulative_return': cumulative_return,
    'running_sharpe': running_sharpe
})
print(df.iloc[::50].round(4))  # ทุก 50 วัน
print(f"\nFinal cumulative return: {cumulative_return.iloc[-1]*100:.2f}%")
print(f"Final Sharpe ratio: {running_sharpe.iloc[-1]:.2f}")
```

---

## 3. MultiIndex DataFrames

### MultiIndex - หลาย levels ของ Index

```python
# ตัวอย่างที่ 9: สร้าง MultiIndex DataFrame
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)

# สร้าง MultiIndex จาก tuples
index = pd.MultiIndex.from_product(
    [['2023', '2024'],
     ['Q1', 'Q2', 'Q3', 'Q4'],
     ['North', 'South', 'East']],
    names=['year', 'quarter', 'region']
)

data = pd.DataFrame({
    'revenue': rng.integers(10000, 100000, len(index)),
    'cost': rng.integers(5000, 50000, len(index)),
    'units': rng.integers(10, 500, len(index))
}, index=index)

print("MultiIndex DataFrame:")
print(data.head(12))
print(f"\nIndex levels: {data.index.names}")
```

```python
# ตัวอย่างที่ 10: MultiIndex - accessing data
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)
index = pd.MultiIndex.from_product(
    [['2023', '2024'], ['Q1', 'Q2', 'Q3', 'Q4'], ['North', 'South']],
    names=['year', 'quarter', 'region']
)
data = pd.DataFrame({
    'revenue': rng.integers(10000, 100000, len(index)),
    'units': rng.integers(10, 500, len(index))
}, index=index)

# เข้าถึงด้วย loc (levels จากซ้ายไปขวา)
print("2024 data:")
print(data.loc['2024'].head(8))

print("\n2024 Q1 data:")
print(data.loc[('2024', 'Q1')])

print("\n2024 Q1 North:")
print(data.loc[('2024', 'Q1', 'North')])

# xs - cross-section (เลือก level ที่ไม่ใช่ outermost)
print("\nAll Q2 data (year=any, region=any):")
print(data.xs('Q2', level='quarter'))

print("\nAll South data:")
print(data.xs('South', level='region').head(8))
```

```python
# ตัวอย่างที่ 11: swaplevel, sortlevel, reset_index
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)
index = pd.MultiIndex.from_product(
    [['2023', '2024'], ['Q1', 'Q2', 'Q3', 'Q4']],
    names=['year', 'quarter']
)
data = pd.Series(rng.integers(100, 1000, 8), index=index, name='sales')

print("Original:")
print(data)

# swaplevel - สลับลำดับ levels
swapped = data.swaplevel()
print("\nAfter swaplevel:")
print(swapped)

# sortlevel
sorted_data = swapped.sort_index(level=0)
print("\nAfter sort by quarter:")
print(sorted_data)

# reset_index - แปลง MultiIndex กลับเป็น columns
df_flat = data.reset_index()
print("\nAfter reset_index:")
print(df_flat)

# set_index - สร้าง MultiIndex จาก columns
df = pd.DataFrame({
    'year': ['2023', '2023', '2024', '2024'],
    'quarter': ['Q1', 'Q2', 'Q1', 'Q2'],
    'sales': [100, 200, 150, 250]
})
df_mi = df.set_index(['year', 'quarter'])
print("\nMultiIndex from columns:")
print(df_mi)
```

---

## 4. Advanced Groupby

```python
# ตัวอย่างที่ 12: filter - กรอง groups ตาม condition
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)
df = pd.DataFrame({
    'department': rng.choice(['IT', 'HR', 'Finance', 'Marketing'], 50),
    'salary': rng.integers(30000, 100000, 50),
    'performance': rng.uniform(2.0, 5.0, 50).round(1)
})

# filter groups ที่ avg salary > 60000
high_salary_depts = df.groupby('department').filter(
    lambda x: x['salary'].mean() > 60000
)
print("Departments with avg salary > 60000:")
print(high_salary_depts['department'].unique())
print(f"Total employees: {len(high_salary_depts)}")

# filter groups ที่มีมากกว่า 10 employees
large_depts = df.groupby('department').filter(
    lambda x: len(x) > 10
)
print(f"\nDepartments with > 10 employees: {large_depts['department'].unique()}")
```

```python
# ตัวอย่างที่ 13: apply ใน groupby - custom aggregation
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)
df = pd.DataFrame({
    'department': rng.choice(['IT', 'HR', 'Finance'], 60),
    'name': [f'Emp_{i}' for i in range(60)],
    'salary': rng.integers(30000, 100000, 60),
    'performance': rng.uniform(2.0, 5.0, 60).round(1)
})

# apply ที่ return DataFrame
def dept_summary(group):
    """Return custom summary per department"""
    return pd.Series({
        'headcount': len(group),
        'avg_salary': group['salary'].mean(),
        'salary_range': group['salary'].max() - group['salary'].min(),
        'salary_p90': group['salary'].quantile(0.9),
        'high_performers': (group['performance'] >= 4.0).sum(),
        'high_performer_pct': (group['performance'] >= 4.0).mean() * 100
    })

summary = df.groupby('department').apply(dept_summary, include_groups=False)
print("Department Summary (custom apply):")
print(summary.round(2))

# apply ที่ return DataFrame (เพิ่มหลาย columns)
def add_dept_stats(group):
    """Add department-level stats to each row"""
    result = group.copy()
    result['dept_avg_salary'] = group['salary'].mean()
    result['salary_vs_avg'] = group['salary'] - group['salary'].mean()
    result['salary_pct_rank'] = group['salary'].rank(pct=True)
    return result

df_enriched = df.groupby('department').apply(add_dept_stats, include_groups=False).reset_index(drop=True)
print("\nEnriched with dept stats:")
print(df_enriched.head(8).round(2))
```

```python
# ตัวอย่างที่ 14: Aggregation functions
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)
df = pd.DataFrame({
    'group': rng.choice(['A', 'B', 'C'], 100),
    'value': rng.normal(100, 20, 100)
})

# Named aggregation ด้วย custom functions
def cv(x):
    """Coefficient of Variation"""
    return x.std() / x.mean()

def iqr(x):
    """Interquartile Range"""
    return x.quantile(0.75) - x.quantile(0.25)

result = df.groupby('group')['value'].agg([
    'mean', 'std', 'median',
    ('cv', cv),
    ('iqr', iqr),
    ('skewness', pd.Series.skew)
])
print("Custom aggregations:")
print(result.round(4))
```

---

## 5. Window Functions

### Window Functions ใน Pandas

```python
# ตัวอย่างที่ 15: Rolling with custom window functions
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)
dates = pd.date_range('2024-01-01', periods=50, freq='D')
prices = pd.Series(
    100 + np.cumsum(rng.normal(0, 1.5, 50)),
    index=dates
)

# Rolling Sharpe ratio (ใช้ apply ใน rolling)
def rolling_sharpe(returns, risk_free=0.02/252):
    excess = returns - risk_free
    if excess.std() == 0:
        return np.nan
    return excess.mean() / excess.std() * np.sqrt(252)

# Convert price to returns
returns = prices.pct_change()

# 20-day rolling Sharpe
rolling_sharpe_20 = returns.rolling(20).apply(rolling_sharpe, raw=True)
print("Rolling 20-day Sharpe ratio:")
print(rolling_sharpe_20.dropna().head(10).round(4))

# Rolling correlation
rng2 = np.random.default_rng(43)
returns2 = pd.Series(
    rng2.normal(0, 1.5, 50) * 0.01,
    index=dates
)
rolling_corr = returns.rolling(20).corr(returns2)
print("\nRolling 20-day correlation with second series:")
print(rolling_corr.dropna().head(10).round(4))
```

```python
# ตัวอย่างที่ 16: Exponential Weighted Moving Average (EWMA)
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)
prices = pd.Series(
    100 + np.cumsum(rng.normal(0, 1, 100)),
    name='price'
)

# EWMA - ให้น้ำหนักมากกับข้อมูลล่าสุด
ewm_fast = prices.ewm(span=7).mean()    # span=7 (เปลี่ยนเร็ว)
ewm_slow = prices.ewm(span=30).mean()   # span=30 (เปลี่ยนช้า)

# EWM statistics
ewm_std = prices.ewm(span=7).std()
ewm_var = prices.ewm(span=7).var()

df = pd.DataFrame({
    'price': prices,
    'EWM_7': ewm_fast,
    'EWM_30': ewm_slow,
    'EWM_std': ewm_std
})

print("EWMA (Exponential Weighted Moving Average):")
print(df.tail(10).round(4))

# MACD (Moving Average Convergence Divergence)
ema12 = prices.ewm(span=12).mean()
ema26 = prices.ewm(span=26).mean()
macd = ema12 - ema26
signal = macd.ewm(span=9).mean()
histogram = macd - signal

macd_df = pd.DataFrame({'MACD': macd, 'Signal': signal, 'Histogram': histogram})
print("\nMACD (last 10 periods):")
print(macd_df.tail(10).round(4))
```

---

## 6. Cross-tabulations

```python
# ตัวอย่างที่ 17: Advanced crosstab
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)
n = 300
survey = pd.DataFrame({
    'age_group': rng.choice(['18-25', '26-35', '36-45', '46+'], n),
    'gender': rng.choice(['M', 'F'], n),
    'product': rng.choice(['A', 'B', 'C', 'D'], n),
    'satisfaction': rng.choice([1, 2, 3, 4, 5], n),
    'purchase': rng.choice([True, False], n)
})

# Simple crosstab
ct = pd.crosstab(survey['age_group'], survey['product'])
print("Age Group × Product:")
print(ct)

# Crosstab with values (aggregate)
ct_values = pd.crosstab(
    survey['age_group'],
    survey['product'],
    values=survey['satisfaction'],
    aggfunc='mean'
)
print("\nAvg Satisfaction by Age × Product:")
print(ct_values.round(2))

# Multiple index/column
ct_multi = pd.crosstab(
    [survey['age_group'], survey['gender']],
    survey['product'],
    margins=True
)
print("\nAge+Gender × Product:")
print(ct_multi)
```

---

## 7. Memory Optimization

### ประหยัด Memory ใน Pandas

```python
# ตัวอย่างที่ 18: Memory profiling
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)
n = 100_000

# สร้าง DataFrame ขนาดกลาง
df = pd.DataFrame({
    'id': rng.integers(1, 10001, n),
    'category': rng.choice(['Electronics', 'Clothing', 'Food', 'Books', 'Sports'], n),
    'sub_category': rng.choice([f'Sub_{i}' for i in range(20)], n),
    'price': rng.uniform(10, 10000, n),
    'quantity': rng.integers(1, 100, n),
    'discount': rng.choice([0, 0.05, 0.10, 0.15, 0.20], n),
    'city': rng.choice(['Bangkok', 'CM', 'Phuket', 'Korat'], n),
    'year': rng.integers(2020, 2025, n),
    'month': rng.integers(1, 13, n)
})

print("Memory usage (default dtypes):")
print(df.dtypes)
print(f"\nTotal memory: {df.memory_usage(deep=True).sum() / 1024**2:.2f} MB")
```

```python
# ตัวอย่างที่ 19: Optimizing dtypes
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)
n = 100_000

df = pd.DataFrame({
    'id': rng.integers(1, 10001, n),
    'category': rng.choice(['Electronics', 'Clothing', 'Food', 'Books', 'Sports'], n),
    'price': rng.uniform(10, 10000, n),
    'quantity': rng.integers(1, 100, n),
    'year': rng.integers(2020, 2025, n),
    'month': rng.integers(1, 13, n)
})

mem_before = df.memory_usage(deep=True).sum() / 1024**2

# Optimize
def optimize_dataframe(df):
    df = df.copy()

    for col in df.columns:
        col_type = df[col].dtype

        if col_type == 'object':
            # ถ้า cardinality ต่ำ (unique values < 50% of total) ใช้ category
            if df[col].nunique() / len(df) < 0.5:
                df[col] = df[col].astype('category')

        elif col_type in ['int64', 'int32']:
            col_min = df[col].min()
            col_max = df[col].max()

            if col_min >= 0:
                if col_max < 256:
                    df[col] = df[col].astype(np.uint8)
                elif col_max < 65536:
                    df[col] = df[col].astype(np.uint16)
                elif col_max < 4294967296:
                    df[col] = df[col].astype(np.uint32)
            else:
                if col_min > -128 and col_max < 128:
                    df[col] = df[col].astype(np.int8)
                elif col_min > -32768 and col_max < 32768:
                    df[col] = df[col].astype(np.int16)
                elif col_min > -2147483648 and col_max < 2147483648:
                    df[col] = df[col].astype(np.int32)

        elif col_type == 'float64':
            # ลด precision ถ้ายังรองรับค่าได้
            df[col] = df[col].astype(np.float32)

    return df

df_optimized = optimize_dataframe(df)
mem_after = df_optimized.memory_usage(deep=True).sum() / 1024**2

print(f"Before: {mem_before:.2f} MB")
print(f"After:  {mem_after:.2f} MB")
print(f"Saved:  {(1 - mem_after/mem_before)*100:.1f}%")
print("\nOptimized dtypes:")
print(df_optimized.dtypes)
```

---

## 8. Chunked Processing

### ประมวลผลไฟล์ขนาดใหญ่แบบ Chunked

```python
# ตัวอย่างที่ 20: สร้างไฟล์ขนาดใหญ่และอ่านแบบ chunk
import pandas as pd
import numpy as np

# สร้าง large CSV ก่อน
rng = np.random.default_rng(42)
n_rows = 1_000_000

print("Creating large CSV...")
# เขียนทีละ chunk
chunk_size = 100_000
header_written = False

for i in range(n_rows // chunk_size):
    chunk_df = pd.DataFrame({
        'id': np.arange(i * chunk_size, (i+1) * chunk_size),
        'category': rng.choice(['A', 'B', 'C', 'D'], chunk_size),
        'value': rng.normal(100, 20, chunk_size).round(2),
        'quantity': rng.integers(1, 100, chunk_size)
    })
    mode = 'w' if not header_written else 'a'
    chunk_df.to_csv('/tmp/large_data.csv', mode=mode, index=False,
                    header=not header_written)
    header_written = True

print("CSV created!")

# อ่านแบบ chunk
chunk_stats = []
total_rows = 0

for chunk in pd.read_csv('/tmp/large_data.csv', chunksize=100_000):
    # ประมวลผลทีละ chunk
    chunk_stats.append({
        'rows': len(chunk),
        'sum_value': chunk['value'].sum(),
        'sum_quantity': chunk['quantity'].sum(),
        'cat_counts': chunk['category'].value_counts().to_dict()
    })
    total_rows += len(chunk)
    print(f"  Processed chunk: {total_rows:,} rows so far...")

# รวมผลลัพธ์
import functools

total_value_sum = sum(s['sum_value'] for s in chunk_stats)
total_qty_sum = sum(s['sum_quantity'] for s in chunk_stats)
print(f"\nTotal rows: {total_rows:,}")
print(f"Total value sum: {total_value_sum:,.2f}")
print(f"Avg value: {total_value_sum/total_rows:.2f}")
```

```python
# ตัวอย่างที่ 21: Generator-based chunked processing
import pandas as pd
import numpy as np

def process_large_file(filepath, chunk_size=50_000, filters=None):
    """
    Generator ที่อ่านและประมวลผล CSV แบบ chunked
    ประหยัด memory โดยใช้ generator pattern
    """
    for i, chunk in enumerate(pd.read_csv(filepath, chunksize=chunk_size)):
        # Apply filters ถ้ามี
        if filters:
            for col, value in filters.items():
                chunk = chunk[chunk[col] == value]

        if len(chunk) > 0:
            yield i, chunk

# ใช้งาน
total_a = 0
count_a = 0

for chunk_idx, chunk in process_large_file(
    '/tmp/large_data.csv',
    chunk_size=100_000,
    filters={'category': 'A'}
):
    total_a += chunk['value'].sum()
    count_a += len(chunk)

print(f"Category A: {count_a:,} rows, avg value: {total_a/count_a:.2f}")
```

---

## 9. Pandas with Databases

### เชื่อมต่อ Pandas กับ Databases

```python
# ตัวอย่างที่ 22: SQLite (ไม่ต้อง install)
import pandas as pd
import sqlite3
import numpy as np

# สร้าง SQLite database ใน memory
conn = sqlite3.connect('/tmp/example.db')

# สร้าง sample data
rng = np.random.default_rng(42)
n = 1000

df = pd.DataFrame({
    'id': range(1, n+1),
    'name': [f'Customer_{i}' for i in range(1, n+1)],
    'age': rng.integers(18, 65, n),
    'city': rng.choice(['Bangkok', 'CM', 'Phuket'], n),
    'purchase_amount': rng.integers(100, 10000, n).astype(float)
})

# เขียน DataFrame ลง SQL
df.to_sql('customers', conn, if_exists='replace', index=False)
print("Wrote DataFrame to SQLite")

# อ่านกลับด้วย SQL query
df_from_sql = pd.read_sql(
    "SELECT * FROM customers WHERE city = 'Bangkok' ORDER BY purchase_amount DESC LIMIT 10",
    conn
)
print("\nTop Bangkok customers:")
print(df_from_sql)
```

```python
# ตัวอย่างที่ 23: Complex SQL queries ผ่าน Pandas
import pandas as pd
import sqlite3
import numpy as np

conn = sqlite3.connect('/tmp/example.db')

# สร้าง tables หลาย tables
rng = np.random.default_rng(42)

orders = pd.DataFrame({
    'order_id': range(1, 501),
    'customer_id': rng.integers(1, 101, 500),
    'product_id': rng.integers(1, 21, 500),
    'quantity': rng.integers(1, 10, 500),
    'order_date': pd.date_range('2024-01-01', periods=500, freq='12H').strftime('%Y-%m-%d')
})

products = pd.DataFrame({
    'product_id': range(1, 21),
    'name': [f'Product_{i}' for i in range(1, 21)],
    'category': rng.choice(['A', 'B', 'C'], 20),
    'price': rng.integers(100, 5000, 20).astype(float)
})

orders.to_sql('orders', conn, if_exists='replace', index=False)
products.to_sql('products', conn, if_exists='replace', index=False)

# SQL JOIN ผ่าน pandas
query = """
    SELECT
        p.category,
        COUNT(o.order_id) as num_orders,
        SUM(o.quantity * p.price) as total_revenue,
        AVG(o.quantity * p.price) as avg_order_value
    FROM orders o
    JOIN products p ON o.product_id = p.product_id
    GROUP BY p.category
    ORDER BY total_revenue DESC
"""

result = pd.read_sql(query, conn)
print("Revenue by Category (SQL):")
print(result)

conn.close()
```

---

## 10. Apply with Progress (tqdm)

```python
# ตัวอย่างที่ 24: tqdm กับ Pandas
try:
    from tqdm import tqdm
    import pandas as pd
    import numpy as np
    import time

    # สร้าง DataFrame ขนาดกลาง
    df = pd.DataFrame({
        'text': [f'Sample text {i} with content' for i in range(10000)],
        'value': np.random.randint(1, 100, 10000)
    })

    # ลงทะเบียน tqdm กับ pandas
    tqdm.pandas(desc="Processing")

    # simulate slow function
    def slow_transform(text):
        # จำลองงานที่ใช้เวลา
        words = text.split()
        return len(words)

    # ใช้ progress_apply แทน apply
    print("Processing with progress bar:")
    df['word_count'] = df['text'].progress_apply(slow_transform)

    print(f"\nProcessed {len(df):,} rows")
    print(f"Avg word count: {df['word_count'].mean():.1f}")

except ImportError:
    print("tqdm not installed. Install with: pip install tqdm")
    print("\nWithout tqdm (standard apply):")
    import pandas as pd
    import numpy as np

    df = pd.DataFrame({
        'text': [f'Sample text {i}' for i in range(1000)],
        'value': np.random.randint(1, 100, 1000)
    })
    df['word_count'] = df['text'].apply(lambda x: len(x.split()))
    print(f"Processed {len(df):,} rows, avg word count: {df['word_count'].mean():.1f}")
```

---

## 11. Performance Profiling

### วัดประสิทธิภาพของ Pandas Operations

```python
# ตัวอย่างที่ 25: Profiling different approaches
import pandas as pd
import numpy as np
import time

rng = np.random.default_rng(42)
n = 100_000

df = pd.DataFrame({
    'a': rng.normal(0, 1, n),
    'b': rng.normal(0, 1, n),
    'category': rng.choice(['X', 'Y', 'Z'], n)
})

def benchmark(name, func, *args, **kwargs):
    """วัดเวลาและ memory"""
    import tracemalloc

    tracemalloc.start()
    start = time.perf_counter()
    result = func(*args, **kwargs)
    elapsed = time.perf_counter() - start
    current, peak = tracemalloc.get_traced_memory()
    tracemalloc.stop()

    print(f"{name:40s}: {elapsed*1000:6.2f}ms, peak memory: {peak/1024:.1f}KB")
    return result

# Method 1: Pure Python loop
def method_loop(df):
    result = []
    for _, row in df.iterrows():
        result.append(row['a'] + row['b'])
    return result

# Method 2: itertuples (เร็วกว่า iterrows)
def method_itertuples(df):
    result = []
    for row in df.itertuples():
        result.append(row.a + row.b)
    return result

# Method 3: apply
def method_apply(df):
    return df.apply(lambda row: row['a'] + row['b'], axis=1)

# Method 4: Vectorized
def method_vectorized(df):
    return df['a'] + df['b']

# Test ด้วย n=1000 (loop methods ช้ามาก)
df_small = df.head(1000)
print("Performance comparison (1000 rows):")
r1 = benchmark("1. iterrows (loop)", method_loop, df_small)
r2 = benchmark("2. itertuples", method_itertuples, df_small)
r3 = benchmark("3. apply", method_apply, df_small)
r4 = benchmark("4. vectorized", method_vectorized, df_small)

print(f"\nFull dataset (100,000 rows) - vectorized only:")
r5 = benchmark("4. vectorized (100k)", method_vectorized, df)
```

```python
# ตัวอย่างที่ 26: Profiling groupby operations
import pandas as pd
import numpy as np
import time

rng = np.random.default_rng(42)
n = 500_000

df = pd.DataFrame({
    'category': rng.choice(['A', 'B', 'C', 'D', 'E'], n),
    'value': rng.normal(100, 20, n)
})

# Method 1: groupby + agg
t0 = time.perf_counter()
r1 = df.groupby('category')['value'].agg(['mean', 'std', 'count'])
t1 = time.perf_counter()
print(f"groupby+agg: {(t1-t0)*1000:.2f}ms")

# Method 2: groupby + transform
t0 = time.perf_counter()
df['group_mean'] = df.groupby('category')['value'].transform('mean')
t1 = time.perf_counter()
print(f"groupby+transform: {(t1-t0)*1000:.2f}ms")

# Method 3: merge back
t0 = time.perf_counter()
group_means = df.groupby('category')['value'].mean().reset_index()
group_means.columns = ['category', 'group_mean2']
df_merged = df.merge(group_means, on='category')
t1 = time.perf_counter()
print(f"merge back: {(t1-t0)*1000:.2f}ms")

print("\nTransform result sample:")
print(df[['category', 'value', 'group_mean']].head(6).round(2))
```

---

## 12. ตัวอย่างโปรแกรมจริง

### โปรแกรมจริงที่ 1: Sales Analysis Dashboard

```python
# ตัวอย่างที่ 27: Complete Sales Analysis
import pandas as pd
import numpy as np
from datetime import datetime

class SalesAnalyzer:
    """
    Complete sales analysis class ที่ทำงานด้วย Pandas
    """

    def __init__(self, data: pd.DataFrame):
        self.data = data.copy()
        self._prepare_data()

    def _prepare_data(self):
        """Prepare and validate data"""
        required_cols = ['date', 'product', 'region', 'quantity', 'unit_price']
        missing = [c for c in required_cols if c not in self.data.columns]
        if missing:
            raise ValueError(f"Missing columns: {missing}")

        self.data['date'] = pd.to_datetime(self.data['date'])
        self.data['revenue'] = self.data['quantity'] * self.data['unit_price']
        self.data['month'] = self.data['date'].dt.to_period('M')
        self.data['quarter'] = self.data['date'].dt.to_period('Q')
        self.data['year'] = self.data['date'].dt.year
        self.data['day_of_week'] = self.data['date'].dt.day_name()

    def summary(self):
        """Executive summary"""
        print("=" * 60)
        print("SALES EXECUTIVE SUMMARY")
        print("=" * 60)
        print(f"Date Range: {self.data['date'].min().date()} to {self.data['date'].max().date()}")
        print(f"Total Revenue: {self.data['revenue'].sum():,.0f}")
        print(f"Total Transactions: {len(self.data):,}")
        print(f"Avg Order Value: {self.data['revenue'].mean():.2f}")
        print(f"Products: {self.data['product'].nunique()}")
        print(f"Regions: {self.data['region'].nunique()}")

    def monthly_trend(self):
        """Monthly revenue trend"""
        monthly = self.data.groupby('month').agg(
            revenue=('revenue', 'sum'),
            transactions=('quantity', 'count'),
            avg_order=('revenue', 'mean')
        )
        monthly['revenue_growth'] = monthly['revenue'].pct_change() * 100
        print("\nMONTHLY TREND:")
        print(monthly.round(2).to_string())
        return monthly

    def product_analysis(self, top_n=5):
        """Product performance analysis"""
        product_stats = self.data.groupby('product').agg(
            total_revenue=('revenue', 'sum'),
            total_quantity=('quantity', 'sum'),
            avg_price=('unit_price', 'mean'),
            transaction_count=('revenue', 'count')
        ).sort_values('total_revenue', ascending=False)

        product_stats['revenue_share'] = (
            product_stats['total_revenue'] /
            product_stats['total_revenue'].sum() * 100
        )

        print(f"\nTOP {top_n} PRODUCTS BY REVENUE:")
        print(product_stats.head(top_n).round(2).to_string())
        return product_stats

    def regional_analysis(self):
        """Regional performance"""
        regional = self.data.pivot_table(
            values='revenue',
            index='region',
            columns=self.data['date'].dt.quarter.map({1:'Q1',2:'Q2',3:'Q3',4:'Q4'}),
            aggfunc='sum',
            fill_value=0
        )
        regional['total'] = regional.sum(axis=1)
        print("\nREGIONAL ANALYSIS BY QUARTER:")
        print(regional.round(0).to_string())
        return regional

    def day_of_week_analysis(self):
        """Sales pattern by day of week"""
        dow_order = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday']
        dow_stats = (self.data.groupby('day_of_week')['revenue']
                         .agg(['mean', 'sum', 'count'])
                         .reindex(dow_order))
        print("\nDAY OF WEEK PATTERN:")
        print(dow_stats.round(2).to_string())
        return dow_stats

# สร้าง data
rng = np.random.default_rng(42)
n = 2000
products = ['Laptop', 'Phone', 'Tablet', 'Headphones', 'Smartwatch']
prices = {'Laptop': 35000, 'Phone': 15000, 'Tablet': 12000,
          'Headphones': 3000, 'Smartwatch': 8000}

data = pd.DataFrame({
    'date': pd.date_range('2024-01-01', '2024-12-31', periods=n),
    'product': rng.choice(products, n),
    'region': rng.choice(['North', 'South', 'East', 'West', 'Central'], n),
    'quantity': rng.integers(1, 10, n)
})
data['unit_price'] = data['product'].map(prices)
data['unit_price'] *= rng.uniform(0.9, 1.1, n)  # price variation

# วิเคราะห์
analyzer = SalesAnalyzer(data)
analyzer.summary()
monthly_trend = analyzer.monthly_trend()
product_stats = analyzer.product_analysis()
```

### โปรแกรมจริงที่ 2: Customer Segmentation (RFM Analysis)

```python
# ตัวอย่างที่ 28: RFM Customer Segmentation
import pandas as pd
import numpy as np

class RFMAnalyzer:
    """
    RFM (Recency, Frequency, Monetary) Analysis
    วิธีการ segment ลูกค้าที่นิยมใน marketing
    """

    def __init__(self, transactions: pd.DataFrame,
                 reference_date=None):
        self.transactions = transactions.copy()
        self.transactions['date'] = pd.to_datetime(self.transactions['date'])
        self.reference_date = reference_date or self.transactions['date'].max()

    def compute_rfm(self):
        """คำนวณ RFM metrics สำหรับแต่ละ customer"""
        rfm = self.transactions.groupby('customer_id').agg(
            # Recency: กี่วันที่แล้วที่ซื้อล่าสุด (ต่ำ = ดี)
            recency=('date', lambda x: (self.reference_date - x.max()).days),
            # Frequency: จำนวนครั้งที่ซื้อ (สูง = ดี)
            frequency=('date', 'count'),
            # Monetary: ยอดซื้อรวม (สูง = ดี)
            monetary=('amount', 'sum')
        )
        self.rfm = rfm
        return rfm

    def score_rfm(self, n_quintiles=5):
        """ให้คะแนน R, F, M (1-5 โดย 5 = ดีที่สุด)"""
        rfm = self.rfm.copy()

        # Recency: ยิ่งน้อยวันยิ่งดี (score สูง)
        rfm['R_score'] = pd.qcut(rfm['recency'],
                                  q=n_quintiles,
                                  labels=range(n_quintiles, 0, -1))

        # Frequency: ยิ่งมากครั้งยิ่งดี (score สูง)
        rfm['F_score'] = pd.qcut(rfm['frequency'].rank(method='first'),
                                  q=n_quintiles,
                                  labels=range(1, n_quintiles+1))

        # Monetary: ยิ่งสูงยิ่งดี (score สูง)
        rfm['M_score'] = pd.qcut(rfm['monetary'],
                                  q=n_quintiles,
                                  labels=range(1, n_quintiles+1))

        rfm['RFM_score'] = (rfm['R_score'].astype(int) +
                            rfm['F_score'].astype(int) +
                            rfm['M_score'].astype(int))
        self.rfm_scored = rfm
        return rfm

    def segment_customers(self):
        """แบ่ง customers เป็น segments"""
        rfm = self.rfm_scored.copy()

        def segment(row):
            r = int(row['R_score'])
            f = int(row['F_score'])
            m = int(row['M_score'])
            rfm_sum = r + f + m

            if r >= 4 and f >= 4 and m >= 4:
                return 'Champions'
            elif r >= 3 and f >= 3:
                return 'Loyal Customers'
            elif r >= 4 and f <= 2:
                return 'Recent Customers'
            elif r <= 2 and f >= 4:
                return 'At Risk'
            elif r <= 2 and f <= 2:
                return 'Lost'
            elif r >= 3 and m >= 4:
                return 'Potential Loyalist'
            else:
                return 'Others'

        rfm['segment'] = rfm.apply(segment, axis=1)
        self.rfm_segmented = rfm
        return rfm

    def print_report(self):
        """Print analysis report"""
        print("=" * 60)
        print("RFM CUSTOMER SEGMENTATION REPORT")
        print("=" * 60)
        print(f"\nTotal Customers: {len(self.rfm_segmented):,}")
        print(f"Total Revenue: {self.rfm_segmented['monetary'].sum():,.0f}")
        print(f"\nRFM Statistics:")
        print(self.rfm_segmented[['recency', 'frequency', 'monetary']].describe().round(2))

        print("\nCustomer Segments:")
        seg_stats = self.rfm_segmented.groupby('segment').agg(
            count=('monetary', 'count'),
            pct=('monetary', lambda x: len(x)/len(self.rfm_segmented)*100),
            avg_revenue=('monetary', 'mean'),
            total_revenue=('monetary', 'sum')
        ).sort_values('total_revenue', ascending=False)
        print(seg_stats.round(2))

# สร้างข้อมูล
rng = np.random.default_rng(42)
n_customers = 500
n_transactions = 5000

transactions = pd.DataFrame({
    'customer_id': rng.integers(1001, 1001+n_customers, n_transactions),
    'date': pd.date_range('2023-01-01', '2024-12-31', periods=n_transactions),
    'amount': rng.exponential(500, n_transactions)
})

# วิเคราะห์
rfm_analyzer = RFMAnalyzer(transactions)
rfm_analyzer.compute_rfm()
rfm_analyzer.score_rfm()
rfm_analyzer.segment_customers()
rfm_analyzer.print_report()
```

### โปรแกรมจริงที่ 3: Time Series Forecasting Preparation

```python
# ตัวอย่างที่ 29: Time Series Feature Engineering
import pandas as pd
import numpy as np

def create_ts_features(df, target_col, date_col='date'):
    """
    สร้าง features สำหรับ time series forecasting
    จาก raw time series data
    """
    df = df.copy().sort_values(date_col)
    df[date_col] = pd.to_datetime(df[date_col])
    df = df.set_index(date_col)

    target = df[target_col]

    # Calendar features
    df['year'] = df.index.year
    df['month'] = df.index.month
    df['day_of_year'] = df.index.day_of_year
    df['day_of_week'] = df.index.day_of_week
    df['quarter'] = df.index.quarter
    df['is_weekend'] = df.index.day_of_week >= 5
    df['is_month_start'] = df.index.is_month_start
    df['is_month_end'] = df.index.is_month_end
    df['is_quarter_start'] = df.index.is_quarter_start

    # Lag features
    for lag in [1, 7, 14, 30]:
        df[f'lag_{lag}'] = target.shift(lag)

    # Rolling statistics
    for window in [7, 14, 30]:
        df[f'rolling_mean_{window}'] = target.shift(1).rolling(window).mean()
        df[f'rolling_std_{window}'] = target.shift(1).rolling(window).std()
        df[f'rolling_max_{window}'] = target.shift(1).rolling(window).max()
        df[f'rolling_min_{window}'] = target.shift(1).rolling(window).min()

    # Exponential moving averages
    for span in [7, 14, 30]:
        df[f'ewma_{span}'] = target.shift(1).ewm(span=span).mean()

    # Year-over-year comparison (ถ้ามีข้อมูล)
    df['yoy_change'] = target.pct_change(periods=365)
    df['mom_change'] = target.pct_change(periods=30)  # month-over-month

    # Fourier features (seasonality)
    t = np.arange(len(df))
    for k in [1, 2, 3]:
        df[f'sin_{k}'] = np.sin(2 * np.pi * k * t / 365)
        df[f'cos_{k}'] = np.cos(2 * np.pi * k * t / 365)

    return df

# สร้าง daily sales data
rng = np.random.default_rng(42)
dates = pd.date_range('2022-01-01', '2024-12-31', freq='D')
n = len(dates)
t = np.arange(n)

# Realistic time series: trend + seasonality + noise
trend = 1000 + 0.5 * t
yearly_seasonal = 200 * np.sin(2 * np.pi * t / 365 - np.pi/2)
weekly_seasonal = 50 * np.sin(2 * np.pi * t / 7)
noise = rng.normal(0, 30, n)

sales_values = trend + yearly_seasonal + weekly_seasonal + noise
sales_values = np.maximum(sales_values, 0)  # no negative sales

daily_sales = pd.DataFrame({
    'date': dates,
    'sales': sales_values
})

# สร้าง features
features = create_ts_features(daily_sales, 'sales')

print(f"Original shape: {daily_sales.shape}")
print(f"With features shape: {features.shape}")
print(f"\nFeature columns ({len(features.columns)}):")
for col in features.columns:
    print(f"  - {col}")

# ตัดข้อมูลที่มี NaN (เกิดจาก lag features)
features_clean = features.dropna()
print(f"\nAfter dropna: {features_clean.shape}")
print(f"\nLast 5 rows:")
print(features_clean[['sales', 'lag_1', 'rolling_mean_7', 'ewma_7']].tail(5).round(2))
```

---

## 13. แบบฝึกหัด

### ข้อ 1: Time Series Analysis
สร้าง time series ของ website traffic (hourly) และวิเคราะห์:
- Daily, weekly, monthly aggregations
- Peak hours และ peak days
- Month-over-month growth
- Rolling 7-day average

```python
# เฉลยข้อ 1
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)

# สร้าง hourly traffic data 6 เดือน
hours = pd.date_range('2024-01-01', '2024-06-30', freq='H')
n = len(hours)

# Traffic มีรูปแบบ: ชั่วโมง, วัน, seasonal
hour_of_day = hours.hour
day_of_week = hours.day_of_week

hourly_pattern = 1000 + 500 * np.sin(np.pi * (hour_of_day - 6) / 12)
hourly_pattern *= np.where(day_of_week < 5, 1.2, 0.7)  # weekday higher
monthly_trend = 1 + 0.05 * (hours.month - 1)  # 5% monthly growth
noise = rng.normal(1, 0.15, n)

traffic = pd.Series(
    (hourly_pattern * monthly_trend * noise).clip(min=0).astype(int),
    index=hours,
    name='visits'
)

print(f"Hourly traffic: {n:,} data points")

# Daily aggregation
daily = traffic.resample('D').sum()
print(f"\nDaily total range: {daily.min():,} to {daily.max():,} visits")

# Weekly aggregation
weekly = traffic.resample('W').sum()
print(f"Weekly total avg: {weekly.mean():,.0f} visits")

# Peak hours (average by hour)
hourly_avg = traffic.groupby(traffic.index.hour).mean()
peak_hour = hourly_avg.idxmax()
print(f"\nPeak hour: {peak_hour}:00 ({hourly_avg[peak_hour]:,.0f} avg visits)")

# Peak day of week
dow_avg = traffic.groupby(traffic.index.day_of_week).sum()
dow_names = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
peak_dow = dow_avg.idxmax()
print(f"Peak day: {dow_names[peak_dow]} ({dow_avg[peak_dow]:,.0f} total visits)")

# Month-over-month growth
monthly = traffic.resample('ME').sum()
mom_growth = monthly.pct_change() * 100
print("\nMonth-over-month growth:")
for month, growth in mom_growth.dropna().items():
    print(f"  {month.strftime('%b %Y')}: {growth:+.1f}%")

# Rolling 7-day average
rolling_7d = daily.rolling(7).mean()
print(f"\nRolling 7-day avg (last 7 days): {rolling_7d.tail(7).round(0).tolist()}")
```

### ข้อ 2: MultiIndex Operations
สร้าง MultiIndex DataFrame ของ sales data และทำ queries

```python
# เฉลยข้อ 2
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)

index = pd.MultiIndex.from_product(
    [range(2022, 2025), ['Q1', 'Q2', 'Q3', 'Q4'],
     ['Electronics', 'Clothing', 'Food']],
    names=['year', 'quarter', 'category']
)

data = pd.DataFrame({
    'revenue': rng.integers(10000, 200000, len(index)),
    'units': rng.integers(50, 2000, len(index)),
    'returns': rng.integers(0, 100, len(index))
}, index=index)

data['return_rate'] = data['returns'] / data['units']
data['avg_price'] = data['revenue'] / data['units']

print("MultiIndex DataFrame:")
print(data.head(12))

# Query 1: 2024 Electronics
q1 = data.loc[(2024, slice(None), 'Electronics')]
print("\n2024 Electronics:")
print(q1)

# Query 2: All Q4 data (cross-section)
q2 = data.xs('Q4', level='quarter')
print(f"\nAll Q4 data: {len(q2)} rows")
print(q2.groupby(level=['year', 'category'])['revenue'].sum().unstack())

# Query 3: Year-over-year growth
yearly_cat = data.groupby(['year', 'category'])['revenue'].sum().unstack()
yoy_growth = yearly_cat.pct_change() * 100
print("\nYear-over-year growth:")
print(yoy_growth.round(1))
```

### ข้อ 3: Advanced Groupby with Transform
วิเคราะห์ salary data และคำนวณ metrics ที่ต้องใช้ transform

```python
# เฉลยข้อ 3
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)
n = 500

df = pd.DataFrame({
    'employee_id': range(1, n+1),
    'department': rng.choice(['IT', 'HR', 'Finance', 'Marketing', 'Ops'], n),
    'location': rng.choice(['HQ', 'Branch_A', 'Branch_B'], n),
    'salary': rng.integers(30000, 150000, n).astype(float),
    'performance': rng.uniform(2.0, 5.0, n).round(1)
})

# Transform operations
df['dept_avg_salary'] = df.groupby('department')['salary'].transform('mean')
df['dept_median_salary'] = df.groupby('department')['salary'].transform('median')
df['salary_vs_dept_mean_pct'] = ((df['salary'] - df['dept_avg_salary']) /
                                  df['dept_avg_salary'] * 100)

df['dept_loc_avg'] = df.groupby(['department', 'location'])['salary'].transform('mean')
df['salary_percentile'] = df.groupby('department')['salary'].transform(
    lambda x: x.rank(pct=True)
)
df['is_above_dept_avg'] = df['salary'] > df['dept_avg_salary']
df['is_top_quartile'] = df['salary_percentile'] >= 0.75

print("Transform results:")
print(df[['department', 'location', 'salary', 'dept_avg_salary',
          'salary_vs_dept_mean_pct', 'salary_percentile', 'is_top_quartile']].head(10).round(2))

print("\nTop quartile count by department:")
print(df.groupby('department')['is_top_quartile'].sum())
```

### ข้อ 4: Rolling Window Analysis
วิเคราะห์ stock price data ด้วย rolling windows

```python
# เฉลยข้อ 4
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)
n_days = 500

# สร้าง stock data
dates = pd.date_range('2023-01-01', periods=n_days, freq='B')  # business days
returns = rng.normal(0.0005, 0.015, n_days)
price = 100 * (1 + returns).cumprod()

df = pd.DataFrame({'close': price}, index=dates)
df['volume'] = rng.integers(1000000, 5000000, n_days)

# Technical indicators
df['daily_return'] = df['close'].pct_change()
df['MA20'] = df['close'].rolling(20).mean()
df['MA50'] = df['close'].rolling(50).mean()
df['MA200'] = df['close'].rolling(200).mean()

# Bollinger Bands
df['BB_std'] = df['close'].rolling(20).std()
df['BB_upper'] = df['MA20'] + 2 * df['BB_std']
df['BB_lower'] = df['MA20'] - 2 * df['BB_std']
df['BB_width'] = (df['BB_upper'] - df['BB_lower']) / df['MA20']

# RSI (Relative Strength Index)
delta = df['close'].diff()
gain = delta.where(delta > 0, 0).rolling(14).mean()
loss = (-delta.where(delta < 0, 0)).rolling(14).mean()
rs = gain / loss
df['RSI'] = 100 - (100 / (1 + rs))

# Trading signals
df['golden_cross'] = (df['MA20'] > df['MA50']) & (df['MA20'].shift(1) <= df['MA50'].shift(1))
df['death_cross'] = (df['MA20'] < df['MA50']) & (df['MA20'].shift(1) >= df['MA50'].shift(1))

print("Stock Analysis (last 10 trading days):")
print(df[['close', 'MA20', 'MA50', 'RSI', 'BB_width',
          'golden_cross', 'death_cross']].tail(10).round(2))

print(f"\nGolden crosses: {df['golden_cross'].sum()}")
print(f"Death crosses: {df['death_cross'].sum()}")
print(f"Days RSI > 70 (overbought): {(df['RSI'] > 70).sum()}")
print(f"Days RSI < 30 (oversold): {(df['RSI'] < 30).sum()}")
```

### ข้อ 5: Memory Optimization Challenge
Optimize DataFrame ขนาด 1M rows ให้ใช้ memory ลดลงมากที่สุด

```python
# เฉลยข้อ 5
import pandas as pd
import numpy as np
import sys

rng = np.random.default_rng(42)
n = 1_000_000

# สร้าง original DataFrame
df_original = pd.DataFrame({
    'user_id': rng.integers(1, 100001, n),
    'product_id': rng.integers(1, 10001, n),
    'category': rng.choice(['Electronics', 'Books', 'Clothing', 'Sports', 'Food',
                            'Beauty', 'Home', 'Toys', 'Automotive', 'Garden'], n),
    'sub_category': rng.choice([f'Sub_{i}' for i in range(50)], n),
    'rating': rng.uniform(1.0, 5.0, n).astype(np.float64),
    'review_count': rng.integers(0, 1000, n).astype(np.int64),
    'price': rng.uniform(10.0, 10000.0, n).astype(np.float64),
    'discount_pct': rng.choice([0, 5, 10, 15, 20, 25, 30, 40, 50], n).astype(np.int64),
    'in_stock': rng.choice([True, False], n),
    'shipping_days': rng.integers(1, 15, n).astype(np.int64),
    'year': rng.integers(2020, 2025, n).astype(np.int64),
    'month': rng.integers(1, 13, n).astype(np.int64)
})

mem_original = df_original.memory_usage(deep=True).sum() / 1024**2
print(f"Original memory: {mem_original:.1f} MB")
print(f"Original dtypes:")
print(df_original.dtypes)

# Optimize
df_opt = df_original.copy()

# Integer columns
df_opt['user_id'] = df_opt['user_id'].astype(np.uint32)        # 1-100000
df_opt['product_id'] = df_opt['product_id'].astype(np.uint16)  # 1-10000
df_opt['review_count'] = df_opt['review_count'].astype(np.uint16) # 0-1000
df_opt['discount_pct'] = df_opt['discount_pct'].astype(np.uint8)  # 0-50
df_opt['shipping_days'] = df_opt['shipping_days'].astype(np.uint8) # 1-15
df_opt['year'] = df_opt['year'].astype(np.uint16)
df_opt['month'] = df_opt['month'].astype(np.uint8)

# Float columns
df_opt['rating'] = df_opt['rating'].astype(np.float32)
df_opt['price'] = df_opt['price'].astype(np.float32)

# String columns -> category
df_opt['category'] = df_opt['category'].astype('category')
df_opt['sub_category'] = df_opt['sub_category'].astype('category')

mem_optimized = df_opt.memory_usage(deep=True).sum() / 1024**2
print(f"\nOptimized memory: {mem_optimized:.1f} MB")
print(f"Memory reduction: {(1 - mem_optimized/mem_original)*100:.1f}%")
print(f"\nOptimized dtypes:")
print(df_opt.dtypes)

# Verify data integrity
assert df_original['user_id'].max() == df_opt['user_id'].max()
assert abs(df_original['price'].mean() - df_opt['price'].mean()) < 0.1
print("\nData integrity: OK")
```

### ข้อ 6-10: (Advanced exercises)

```python
# ข้อ 6: Chunked processing สำหรับ large CSV
# ข้อ 7: SQLite database operations
# ข้อ 8: RFM Analysis implementation
# ข้อ 9: Time series feature engineering
# ข้อ 10: Complete analytics pipeline

# เฉลยข้อ 10: Complete Analytics Pipeline
import pandas as pd
import numpy as np

def run_analytics_pipeline(n_customers=1000, n_transactions=10000):
    """Complete analytics pipeline"""
    rng = np.random.default_rng(42)

    # 1. สร้างข้อมูล
    customers = pd.DataFrame({
        'customer_id': range(1001, 1001+n_customers),
        'name': [f'Customer_{i}' for i in range(1, n_customers+1)],
        'tier': rng.choice(['Bronze', 'Silver', 'Gold', 'Platinum'],
                           n_customers, p=[0.5, 0.3, 0.15, 0.05]),
        'city': rng.choice(['Bangkok', 'CM', 'Phuket', 'Korat'], n_customers),
        'join_date': pd.to_datetime('2020-01-01') +
                     pd.to_timedelta(rng.integers(0, 1460, n_customers), unit='D')
    })

    transactions = pd.DataFrame({
        'txn_id': range(1, n_transactions+1),
        'customer_id': rng.integers(1001, 1001+n_customers, n_transactions),
        'amount': rng.exponential(500, n_transactions).round(2),
        'date': pd.to_datetime('2024-01-01') +
                pd.to_timedelta(rng.integers(0, 365, n_transactions), unit='D'),
        'category': rng.choice(['A', 'B', 'C', 'D', 'E'], n_transactions)
    })

    # 2. Merge
    df = transactions.merge(
        customers[['customer_id', 'tier', 'city']],
        on='customer_id', how='left'
    )

    # 3. Feature Engineering
    df['month'] = df['date'].dt.month
    df['quarter'] = df['date'].dt.quarter
    df['day_of_week'] = df['date'].dt.day_name()

    # 4. Analytics
    results = {}

    # Revenue by tier
    results['tier_revenue'] = df.groupby('tier').agg(
        total_revenue=('amount', 'sum'),
        avg_txn=('amount', 'mean'),
        count=('txn_id', 'count')
    ).round(2)

    # Monthly trend
    results['monthly_trend'] = df.groupby('month')['amount'].sum()

    # Top categories by tier
    results['cat_by_tier'] = pd.pivot_table(
        df, values='amount', index='tier',
        columns='category', aggfunc='sum', fill_value=0
    )

    # Customer cohort (by join month)
    customers['join_month'] = customers['join_date'].dt.to_period('M')
    customer_txns = transactions.merge(
        customers[['customer_id', 'join_month']], on='customer_id'
    )
    results['cohort'] = customer_txns.groupby(
        ['join_month', transactions['date'].dt.to_period('M').rename('txn_month')]
    )['amount'].sum().unstack()

    print("Analytics Pipeline Complete!")
    print(f"\nTier Revenue:")
    print(results['tier_revenue'])
    print(f"\nTop Month: {results['monthly_trend'].idxmax()} (revenue: {results['monthly_trend'].max():,.0f})")

    return results

results = run_analytics_pipeline()
```

---

## สรุป Part 74

ใน Part นี้เราได้เรียนรู้ Pandas ขั้นสูง:

| หัวข้อ | สิ่งที่ได้เรียน |
|--------|----------------|
| **Time Series** | DatetimeIndex, dt accessor, timezone, offsets |
| **Resample** | downsample, upsample, interpolation |
| **Rolling** | moving average, std, custom functions, EWM |
| **MultiIndex** | creation, access, xs, swaplevel, stack/unstack |
| **Advanced Groupby** | filter, apply, custom agg, transform |
| **Window Functions** | rolling Sharpe, EWMA, MACD |
| **Crosstab** | frequency tables, value aggregation, normalize |
| **Memory Optimization** | dtype selection, category, float32 |
| **Chunked Processing** | read_csv chunksize, generator pattern |
| **SQL Integration** | read_sql, to_sql, complex queries |
| **tqdm** | progress bars กับ apply |
| **Performance** | profiling, vectorized vs loop, groupby compare |
| **Real Programs** | Sales Analysis, RFM, Time Series Feature Eng |

### Key Advanced Concepts

1. **Resample frequencies**: 'D'=day, 'W'=week, 'ME'=month-end, 'QE'=quarter-end, 'YE'=year-end, 'H'=hour, 'T'=minute
2. **Rolling vs EWM**: Rolling ให้ weight เท่ากัน, EWM ให้ weight มากกับข้อมูลล่าสุด
3. **Transform vs Agg**: transform คืน Series ขนาดเดิม, agg คืน reduced Series
4. **Chunked processing**: ใช้เมื่อข้อมูลใหญ่กว่า RAM
5. **Category dtype**: บันทึก strings เป็น integer codes + lookup table

---

**ก่อนหน้า**: [Part 73 - Pandas Data Manipulation](../part73/README.md)
**ต่อไป**: [Part 75 - Data Visualization](../part75/README.md)
