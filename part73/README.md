# Part 73 - Pandas: Data Manipulation (การจัดการข้อมูลด้วย Pandas)

## สารบัญ
1. [Pandas Series and DataFrame](#1-pandas-series-and-dataframe)
2. [Creating DataFrames](#2-creating-dataframes)
3. [Data Inspection](#3-data-inspection)
4. [Indexing: loc vs iloc](#4-indexing-loc-vs-iloc)
5. [Boolean Selection](#5-boolean-selection)
6. [Data Cleaning](#6-data-cleaning)
7. [Data Transformation](#7-data-transformation)
8. [Sorting and Ranking](#8-sorting-and-ranking)
9. [Groupby Operations](#9-groupby-operations)
10. [Merge, Join, Concat](#10-merge-join-concat)
11. [Pivot Tables](#11-pivot-tables)
12. [แบบฝึกหัด](#12-แบบฝึกหัด)

---

## 1. Pandas Series and DataFrame

### Pandas คืออะไร?

**Pandas** เป็น library สำหรับ data manipulation และ analysis ใน Python
สร้างโดย Wes McKinney ในปี 2008 โดย build บน NumPy

โครงสร้างข้อมูลหลักใน Pandas:
1. **Series**: 1D labeled array (column เดียว)
2. **DataFrame**: 2D labeled table (หลาย columns)
3. **Index**: label สำหรับ rows และ columns

```python
# ตัวอย่างที่ 1: Pandas Series พื้นฐาน
import pandas as pd
import numpy as np

# สร้าง Series จาก list
s1 = pd.Series([10, 20, 30, 40, 50])
print("Default index Series:")
print(s1)
print(f"dtype: {s1.dtype}, shape: {s1.shape}")

# Series ที่มี custom index
s2 = pd.Series([100, 200, 300],
               index=['a', 'b', 'c'],
               name='values')
print("\nNamed index Series:")
print(s2)

# Series จาก dict (keys เป็น index)
data = {'Jan': 1500, 'Feb': 1800, 'Mar': 2100, 'Apr': 1900}
sales = pd.Series(data)
print("\nSales Series:")
print(sales)

# การเข้าถึง Series
print("\ns2['b']:", s2['b'])            # by label
print("sales['Mar']:", sales['Mar'])    # 2100
print("sales[1]:", sales[1])            # by position (1800)
print("sales[['Jan', 'Mar']]:")         # multiple labels
print(sales[['Jan', 'Mar']])
```

```python
# ตัวอย่างที่ 2: Series operations
import pandas as pd
import numpy as np

s = pd.Series([10, 20, 30, 40, 50], index=['a', 'b', 'c', 'd', 'e'])

# Arithmetic operations
print("s + 5:")
print(s + 5)

print("\ns * 2:")
print(s * 2)

# Alignment by index (automatic)
s1 = pd.Series([1, 2, 3], index=['a', 'b', 'c'])
s2 = pd.Series([10, 20, 30], index=['b', 'c', 'd'])
result = s1 + s2
print("\ns1 + s2 (with alignment):")
print(result)  # 'a' และ 'd' เป็น NaN เพราะไม่มีใน series อีกอัน

# Boolean operations
print("\nValues > 25:")
print(s[s > 25])

# Statistics
print(f"\nMean: {s.mean():.2f}")
print(f"Std: {s.std():.2f}")
print(f"Sum: {s.sum()}")
```

```python
# ตัวอย่างที่ 3: DataFrame พื้นฐาน
import pandas as pd

# สร้าง DataFrame จาก dict
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve'],
    'age': [25, 30, 35, 28, 32],
    'salary': [50000, 65000, 80000, 55000, 70000],
    'department': ['HR', 'IT', 'Finance', 'IT', 'HR'],
    'active': [True, True, False, True, True]
})

print("DataFrame:")
print(df)
print(f"\nShape: {df.shape}")        # (5, 5) - rows, cols
print(f"Columns: {df.columns.tolist()}")
print(f"Index: {df.index.tolist()}")
print(f"dtypes:\n{df.dtypes}")
```

---

## 2. Creating DataFrames

### วิธีสร้าง DataFrames ต่างๆ

```python
# ตัวอย่างที่ 4: สร้าง DataFrame จากหลายแหล่ง
import pandas as pd
import numpy as np

# Method 1: จาก list of dicts
records = [
    {'name': 'Alice', 'score': 85},
    {'name': 'Bob', 'score': 92},
    {'name': 'Charlie', 'score': 78}
]
df1 = pd.DataFrame(records)
print("From list of dicts:")
print(df1)

# Method 2: จาก NumPy array
arr = np.random.default_rng(42).integers(60, 100, (4, 3))
df2 = pd.DataFrame(arr,
                   columns=['Math', 'Science', 'English'],
                   index=['Alice', 'Bob', 'Charlie', 'Diana'])
print("\nFrom NumPy array:")
print(df2)

# Method 3: Series เป็น columns
df3 = pd.DataFrame({
    'month': pd.Series(['Jan', 'Feb', 'Mar']),
    'revenue': pd.Series([1000, 1200, 900]),
    'expenses': pd.Series([800, 850, 750])
})
print("\nFrom Series:")
print(df3)
```

```python
# ตัวอย่างที่ 5: อ่านจาก CSV
import pandas as pd
import io

# จำลอง CSV content
csv_content = """id,name,age,city,salary,join_date
1,Alice Johnson,28,Bangkok,55000,2021-03-15
2,Bob Smith,35,Chiang Mai,72000,2019-07-22
3,Charlie Brown,42,Bangkok,95000,2015-11-08
4,Diana Prince,29,Phuket,48000,2022-01-30
5,Eve Davis,31,Bangkok,63000,2020-09-12
6,Frank Miller,38,Chiang Mai,85000,2018-05-20
7,Grace Lee,26,Bangkok,42000,2023-02-14
8,Henry Wilson,45,Phuket,110000,2012-08-03
"""

df = pd.read_csv(io.StringIO(csv_content))

# ตรวจสอบ dtypes อัตโนมัติ
print("DataFrame from CSV:")
print(df)
print("\nDtypes:")
print(df.dtypes)

# แปลง join_date เป็น datetime
df['join_date'] = pd.to_datetime(df['join_date'])
print("\nAfter datetime conversion:")
print(df.dtypes)
```

```python
# ตัวอย่างที่ 6: read_csv parameters
import pandas as pd
import io

# CSV ที่มี options ต่างๆ
csv_content = """# Company Sales Data
; Product; Q1; Q2; Q3; Q4
; Laptop; 150; 200; 180; 250
; Phone; 300; 350; 320; 400
; Tablet; 120; 140; 110; 180
"""

# options หลักๆ ของ read_csv
df = pd.read_csv(io.StringIO(csv_content),
                 sep=';',           # separator
                 skiprows=1,        # skip comment line
                 index_col=1,       # column 1 เป็น index (Product)
                 skipinitialspace=True,  # ตัด spaces หลัง separator
                 na_values=['N/A', '-', ''])  # ค่าที่ถือว่าเป็น NaN

print(df)

# write to CSV
df.to_csv('/tmp/sales.csv', index=True)
print("\nSaved to CSV")

# อ่านกลับ
df_back = pd.read_csv('/tmp/sales.csv', index_col=0)
print(df_back)
```

```python
# ตัวอย่างที่ 7: สร้าง sample data สำหรับ workshop
import pandas as pd
import numpy as np

def create_employee_data(n=100, seed=42):
    """สร้าง employee dataset จำลอง"""
    rng = np.random.default_rng(seed)

    names = ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve', 'Frank',
             'Grace', 'Henry', 'Iris', 'Jack', 'Karen', 'Leo']
    departments = ['HR', 'IT', 'Finance', 'Marketing', 'Operations']
    cities = ['Bangkok', 'Chiang Mai', 'Phuket', 'Korat', 'Hat Yai']

    # สร้าง department-based salary range
    dept_salary = {
        'HR': (30000, 70000),
        'IT': (50000, 120000),
        'Finance': (45000, 100000),
        'Marketing': (35000, 80000),
        'Operations': (30000, 65000)
    }

    dept_arr = rng.choice(departments, n)
    salaries = np.array([
        rng.integers(*dept_salary[d])
        for d in dept_arr
    ])

    df = pd.DataFrame({
        'employee_id': [f'EMP{i:04d}' for i in range(1, n+1)],
        'name': [f'{rng.choice(names)} {rng.choice(["Smith","Johnson","Lee","Brown","Davis"])}'
                 for _ in range(n)],
        'age': rng.integers(22, 58, n),
        'gender': rng.choice(['M', 'F'], n),
        'department': dept_arr,
        'city': rng.choice(cities, n),
        'salary': salaries,
        'years_experience': rng.integers(1, 20, n),
        'performance_score': rng.uniform(2.0, 5.0, n).round(1),
        'join_date': pd.date_range('2015-01-01', periods=n, freq='3D')[:n]
    })

    # เพิ่ม missing values บางตัว
    df.loc[rng.choice(n, 5, replace=False), 'salary'] = np.nan
    df.loc[rng.choice(n, 3, replace=False), 'performance_score'] = np.nan

    return df

# สร้างข้อมูล
emp_df = create_employee_data(100)
print("Employee Dataset:")
print(emp_df.head())
print(f"\nShape: {emp_df.shape}")
```

---

## 3. Data Inspection

### ตรวจสอบและสำรวจข้อมูล

```python
# ตัวอย่างที่ 8: Basic inspection methods
import pandas as pd
import numpy as np

df = create_employee_data(100)  # จาก function ข้างบน

# head/tail - ดูข้อมูลต้น/ท้าย
print("head(5):")
print(df.head(5))

print("\ntail(3):")
print(df.tail(3))

# sample - สุ่มดูบางส่วน
print("\nsample(3):")
print(df.sample(3, random_state=42))
```

```python
# ตัวอย่างที่ 9: info() - ข้อมูลโครงสร้าง
import pandas as pd

df = create_employee_data(100)

# info() แสดง dtype, non-null count, memory usage
print("df.info():")
df.info()

# shape, ndim, size
print(f"\nshape: {df.shape}")
print(f"ndim: {df.ndim}")
print(f"size: {df.size}")

# dtypes
print("\ndtypes:")
print(df.dtypes)

# columns และ index
print("\ncolumns:", df.columns.tolist())
print("index:", df.index[:5].tolist(), "...")
```

```python
# ตัวอย่างที่ 10: describe() - สถิติพื้นฐาน
import pandas as pd

df = create_employee_data(100)

# describe() สำหรับ numeric columns
print("Numeric stats:")
print(df.describe())

# describe() สำหรับ categorical columns
print("\nCategorical stats:")
print(df.describe(include=['object']))

# describe() ทุก columns
print("\nAll columns:")
print(df.describe(include='all'))

# ข้อมูลเพิ่มเติม
print("\nValue counts - Department:")
print(df['department'].value_counts())

print("\nUnique values in department:", df['department'].unique())
print("Number of unique:", df['department'].nunique())
```

```python
# ตัวอย่างที่ 11: Missing data overview
import pandas as pd
import numpy as np

df = create_employee_data(100)

# ตรวจสอบ missing values
print("Missing values per column:")
print(df.isnull().sum())

print("\nMissing percentage:")
print((df.isnull().sum() / len(df) * 100).round(2))

# Heatmap ของ missing values (เป็น boolean)
print("\nMissing pattern (first 10 rows):")
print(df.isnull().head(10))

# หา rows ที่มี missing values
rows_with_na = df[df.isnull().any(axis=1)]
print(f"\nRows with any missing: {len(rows_with_na)}")
```

---

## 4. Indexing: loc vs iloc

### ความแตกต่างระหว่าง loc และ iloc

| | `loc` | `iloc` |
|--|-------|--------|
| Type | Label-based | Integer position-based |
| Row select | ใช้ index label | ใช้ integer position |
| Col select | ใช้ column name | ใช้ integer position |
| Slicing | **inclusive** endpoint | **exclusive** endpoint |

```python
# ตัวอย่างที่ 12: loc - label-based indexing
import pandas as pd

df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana'],
    'age': [25, 30, 35, 28],
    'salary': [50000, 65000, 80000, 55000]
}, index=['emp01', 'emp02', 'emp03', 'emp04'])

print("DataFrame with custom index:")
print(df)

# loc[row_label, col_label]
print("\ndf.loc['emp01']:")
print(df.loc['emp01'])

print("\ndf.loc['emp01', 'name']:", df.loc['emp01', 'name'])

print("\ndf.loc['emp01':'emp03']:")          # inclusive!
print(df.loc['emp01':'emp03'])

print("\ndf.loc[['emp01', 'emp03']]:")       # multiple rows
print(df.loc[['emp01', 'emp03']])

print("\ndf.loc['emp01':'emp03', 'name':'age']:")  # rows and cols
print(df.loc['emp01':'emp03', 'name':'age'])
```

```python
# ตัวอย่างที่ 13: iloc - integer position indexing
import pandas as pd

df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve'],
    'age': [25, 30, 35, 28, 32],
    'salary': [50000, 65000, 80000, 55000, 70000],
    'dept': ['HR', 'IT', 'Finance', 'IT', 'HR']
}, index=['emp01', 'emp02', 'emp03', 'emp04', 'emp05'])

# iloc[row_int, col_int]
print("iloc[0]:")
print(df.iloc[0])

print("\niloc[2, 1]:", df.iloc[2, 1])   # row 2, col 1

print("\niloc[0:3]:")                     # exclusive endpoint
print(df.iloc[0:3])

print("\niloc[[0, 2, 4]]:")
print(df.iloc[[0, 2, 4]])

print("\niloc[1:4, 0:2]:")
print(df.iloc[1:4, 0:2])

print("\niloc[-1]:")               # last row
print(df.iloc[-1])

# สำคัญ: ถ้า index เป็น integer ระวังความสับสนระหว่าง loc และ iloc
df_int_idx = df.reset_index(drop=True)
print("\nWith integer index:")
print(df_int_idx.head(3))
print("loc[1] vs iloc[1] ต่างกันเมื่อ index ถูก reorder")
```

```python
# ตัวอย่างที่ 14: การแก้ไขค่าด้วย loc/iloc
import pandas as pd
import numpy as np

df = create_employee_data(10)
print("Before modification:")
print(df[['name', 'salary', 'performance_score']].head(3))

# แก้ไข single value
df.loc[0, 'salary'] = 60000
print("\nAfter loc[0, 'salary'] = 60000")

# แก้ไข column ทั้งหมด
df.loc[:, 'salary'] = df['salary'] * 1.1  # เพิ่ม 10%
print("After 10% salary increase")

# แก้ไขเฉพาะ rows ที่ตรงเงื่อนไข
df.loc[df['department'] == 'IT', 'performance_score'] = 4.5
print("\nAfter setting IT performance_score to 4.5")

print(df[['name', 'department', 'salary', 'performance_score']].head(6))
```

---

## 5. Boolean Selection

### การกรองข้อมูลด้วยเงื่อนไข

```python
# ตัวอย่างที่ 15: Boolean indexing พื้นฐาน
import pandas as pd

df = create_employee_data(50)

# Simple conditions
print("Employees in IT:")
it_employees = df[df['department'] == 'IT']
print(it_employees[['name', 'department', 'salary']].head())

print(f"\nEmployees with salary > 80000: {(df['salary'] > 80000).sum()}")

# Multiple conditions (&, |, ~)
high_it = df[(df['department'] == 'IT') & (df['salary'] > 70000)]
print(f"\nIT employees with salary > 70000: {len(high_it)}")

either_dept = df[(df['department'] == 'IT') | (df['department'] == 'Finance')]
print(f"IT or Finance employees: {len(either_dept)}")

not_bangkok = df[~(df['city'] == 'Bangkok')]
print(f"Employees NOT in Bangkok: {len(not_bangkok)}")
```

```python
# ตัวอย่างที่ 16: isin, between, str methods
import pandas as pd

df = create_employee_data(50)

# isin - เลือก rows ที่ column มีค่าใน list
selected_depts = df[df['department'].isin(['IT', 'Finance', 'HR'])]
print(f"Employees in IT/Finance/HR: {len(selected_depts)}")

# between - เลือกค่าในช่วง (inclusive ทั้งสองด้าน)
age_range = df[df['age'].between(25, 35)]
print(f"Employees aged 25-35: {len(age_range)}")

# String methods
df_with_names = df[df['name'].str.contains('a', case=False)]  # ชื่อมี 'a'
print(f"Names containing 'a': {len(df_with_names)}")

df_starts_with_b = df[df['name'].str.startswith('B')]
print(f"Names starting with 'B': {len(df_starts_with_b)}")
```

```python
# ตัวอย่างที่ 17: query() method
import pandas as pd

df = create_employee_data(50)

# query() - เขียน filter เป็น string expression (อ่านง่ายกว่า)
result1 = df.query("department == 'IT' and salary > 70000")
print("IT employees with salary > 70000:")
print(result1[['name', 'department', 'salary']])

# ใช้ variable ใน query ด้วย @
min_age = 30
max_salary = 80000
result2 = df.query("age >= @min_age and salary <= @max_salary")
print(f"\nAge >= {min_age} and salary <= {max_salary}: {len(result2)} employees")

# Chained queries
result3 = (df.query("city == 'Bangkok'")
             .query("performance_score >= 4.0"))
print(f"\nBangkok employees with high performance: {len(result3)}")
```

---

## 6. Data Cleaning

### ทำความสะอาดข้อมูล

```python
# ตัวอย่างที่ 18: Handling Missing Values
import pandas as pd
import numpy as np

df = create_employee_data(50)

print("Missing values before cleaning:")
print(df.isnull().sum())

# dropna - ลบ rows/cols ที่มี NaN
df_drop_rows = df.dropna()  # ลบทุก row ที่มี NaN ใด
df_drop_cols = df.dropna(axis=1)  # ลบ columns ที่มี NaN ใด

# subset - ดูเฉพาะบาง columns
df_drop_salary_na = df.dropna(subset=['salary'])  # ลบเฉพาะ row ที่ salary เป็น NaN

print(f"\nOriginal: {len(df)} rows")
print(f"After dropna(): {len(df_drop_rows)} rows")
print(f"After dropna(subset=['salary']): {len(df_drop_salary_na)} rows")

# how parameter
df_any = df.dropna(how='any')   # ลบถ้ามี NaN ใดๆ (default)
df_all = df.dropna(how='all')   # ลบเฉพาะถ้าทุก value เป็น NaN
print(f"After dropna(how='all'): {len(df_all)} rows")
```

```python
# ตัวอย่างที่ 19: fillna - เติมค่า Missing
import pandas as pd
import numpy as np

df = create_employee_data(50)

# fillna ด้วยค่าคงที่
df_filled_0 = df.fillna(0)
df_filled_unknown = df.copy()
df_filled_unknown['salary'] = df['salary'].fillna(0)

# fillna ด้วย statistics
df_filled_mean = df.copy()
df_filled_mean['salary'] = df['salary'].fillna(df['salary'].mean())
df_filled_median = df.copy()
df_filled_median['salary'] = df['salary'].fillna(df['salary'].median())

print(f"Original salary NaN count: {df['salary'].isnull().sum()}")
print(f"After fill with mean: {df_filled_mean['salary'].isnull().sum()}")
print(f"After fill with median: {df_filled_median['salary'].isnull().sum()}")

# Forward/backward fill (สำหรับ time series)
s = pd.Series([1, np.nan, np.nan, 4, np.nan, 6])
print("\nTime series with NaN:")
print(s.values)
print("Forward fill:", s.ffill().values)
print("Backward fill:", s.bfill().values)

# fillna ด้วย groupby mean
df_group_fill = df.copy()
dept_mean_salary = df.groupby('department')['salary'].transform('mean')
df_group_fill['salary'] = df['salary'].fillna(dept_mean_salary)
print("\nFilled salary with department mean")
```

```python
# ตัวอย่างที่ 20: Duplicates
import pandas as pd

# สร้างข้อมูลที่มี duplicates
data = pd.DataFrame({
    'id': [1, 2, 2, 3, 4, 4, 4, 5],
    'name': ['Alice', 'Bob', 'Bob', 'Charlie', 'Diana', 'Diana', 'Diana', 'Eve'],
    'score': [85, 90, 90, 78, 92, 92, 95, 88]
})

print("Data with duplicates:")
print(data)

# ตรวจสอบ duplicates
print("\nDuplicate rows:")
print(data.duplicated())

print("\nDuplicate rows (any column):")
print(data[data.duplicated(keep=False)])  # keep=False แสดงทุก copies

# drop_duplicates
df_no_dup = data.drop_duplicates()
print(f"\nAfter drop_duplicates: {len(df_no_dup)} rows")

# ลบ duplicates ตาม subset ของ columns
df_unique_id = data.drop_duplicates(subset=['id'])
print(f"After drop_duplicates(subset=['id']): {len(df_unique_id)} rows")

# เก็บ duplicate ตัวไหน
df_keep_last = data.drop_duplicates(subset=['id'], keep='last')
print("\nKeeping last occurrence:")
print(df_keep_last)
```

```python
# ตัวอย่างที่ 21: Data type conversion และ cleaning
import pandas as pd
import numpy as np

# ข้อมูล "สกปรก" จาก real world
dirty_data = pd.DataFrame({
    'price': ['$1,234.50', '$999', '1500.00', 'N/A', '$2,100'],
    'date': ['2024-01-15', '01/02/2024', '2024-3-5', 'Mar 10 2024', None],
    'rating': ['4.5/5', '3/5', '5/5', '2.5/5', '4/5'],
    'phone': ['0891234567', '+66 89-123-4567', '089 123 4567', '089-1234567', None]
})

print("Original dirty data:")
print(dirty_data)

# ทำความสะอาด price
dirty_data['price_clean'] = (dirty_data['price']
    .str.replace('$', '', regex=False)
    .str.replace(',', '', regex=False)
    .replace('N/A', np.nan)
    .astype(float))

# ทำความสะอาด date
dirty_data['date_clean'] = pd.to_datetime(dirty_data['date'], errors='coerce')

# ทำความสะอาด rating
dirty_data['rating_clean'] = (dirty_data['rating']
    .str.extract(r'(\d+\.?\d*)')[0]  # เอาเฉพาะตัวเลข
    .astype(float))

print("\nCleaned data:")
print(dirty_data[['price_clean', 'date_clean', 'rating_clean']])
```

---

## 7. Data Transformation

### การแปลงข้อมูล

```python
# ตัวอย่างที่ 22: rename - เปลี่ยนชื่อ columns/index
import pandas as pd

df = create_employee_data(10)

# rename ด้วย dict
df_renamed = df.rename(columns={
    'employee_id': 'id',
    'years_experience': 'experience',
    'performance_score': 'perf_score'
})
print("Renamed columns:", df_renamed.columns.tolist())

# rename ด้วย function
df_upper = df.rename(columns=str.upper)
print("Uppercase columns:", df_upper.columns.tolist())

# inplace
df.rename(columns={'salary': 'base_salary'}, inplace=True)
print("\nAfter inplace rename:", 'base_salary' in df.columns)
df.rename(columns={'base_salary': 'salary'}, inplace=True)  # revert
```

```python
# ตัวอย่างที่ 23: astype - เปลี่ยน dtype
import pandas as pd
import numpy as np

df = create_employee_data(20)

print("Before astype:")
print(df[['age', 'salary', 'performance_score']].dtypes)

# แปลง dtypes
df['age'] = df['age'].astype(np.int8)          # ประหยัด memory
df['department'] = df['department'].astype('category')  # categorical = ประหยัด memory + เร็วกว่า
df['gender'] = df['gender'].astype('category')

print("\nAfter astype:")
print(df[['age', 'salary', 'department', 'gender']].dtypes)

# memory usage
print("\nMemory usage before/after:")
print(f"  String 'department': {pd.Series(['HR','IT','Finance']*100).memory_usage()}")
print(f"  Category 'department': {pd.Series(['HR','IT','Finance']*100, dtype='category').memory_usage()}")
```

```python
# ตัวอย่างที่ 24: map - แปลง Series ด้วย dict/function
import pandas as pd

df = create_employee_data(20)

# map ด้วย dict
dept_code = {
    'HR': 'D01',
    'IT': 'D02',
    'Finance': 'D03',
    'Marketing': 'D04',
    'Operations': 'D05'
}
df['dept_code'] = df['department'].map(dept_code)
print("Department codes:")
print(df[['department', 'dept_code']].head(8))

# map ด้วย function
df['grade'] = df['performance_score'].map(
    lambda x: 'A' if x >= 4.5 else ('B' if x >= 3.5 else 'C') if pd.notna(x) else 'N/A'
)
print("\nPerformance grades:")
print(df[['performance_score', 'grade']].head(8))
```

```python
# ตัวอย่างที่ 25: apply - apply function ต่อ Series หรือ DataFrame
import pandas as pd
import numpy as np

df = create_employee_data(20)

# apply ต่อ Series (column)
df['salary_band'] = df['salary'].apply(
    lambda x: 'High' if x > 80000 else ('Mid' if x > 50000 else 'Low')
    if pd.notna(x) else 'Unknown'
)
print("Salary bands:")
print(df[['salary', 'salary_band']].head(8))

# apply ต่อ DataFrame (ต่อ row หรือ column)
def compute_bonus(row):
    """คำนวณ bonus ตาม performance และ department"""
    if pd.isna(row['salary']) or pd.isna(row['performance_score']):
        return np.nan
    base = row['salary']
    perf = row['performance_score']
    dept_mult = 1.2 if row['department'] == 'IT' else 1.0
    return base * (perf / 5.0) * 0.15 * dept_mult

df['bonus'] = df.apply(compute_bonus, axis=1)
print("\nBonus (first 8):")
print(df[['name', 'department', 'salary', 'performance_score', 'bonus']].head(8))
```

```python
# ตัวอย่างที่ 26: assign - เพิ่ม columns แบบ functional
import pandas as pd
import numpy as np

df = create_employee_data(20)

# assign ช่วยให้เขียน pipeline ได้ (method chaining)
df_processed = (df
    .assign(salary_k=lambda x: x['salary'] / 1000)           # salary ใน หน่วย K
    .assign(age_group=lambda x: pd.cut(x['age'],
                                        bins=[20, 30, 40, 50, 60],
                                        labels=['20s', '30s', '40s', '50s']))
    .assign(is_senior=lambda x: x['years_experience'] >= 10)
    .assign(total_comp=lambda x: x['salary'] + x['salary'] * 0.1)  # 10% bonus
)

print("Processed DataFrame:")
print(df_processed[['name', 'age', 'age_group', 'salary_k',
                     'years_experience', 'is_senior', 'total_comp']].head(8))
```

```python
# ตัวอย่างที่ 27: pd.cut และ pd.qcut - Binning
import pandas as pd
import numpy as np

df = create_employee_data(50)

# cut - กำหนด bins เอง (equal width)
salary_bins = [0, 40000, 60000, 80000, 100000, float('inf')]
salary_labels = ['Very Low', 'Low', 'Medium', 'High', 'Very High']

df['salary_cat'] = pd.cut(df['salary'],
                           bins=salary_bins,
                           labels=salary_labels)
print("Salary distribution:")
print(df['salary_cat'].value_counts().sort_index())

# qcut - แบ่งตาม quantiles (equal frequency)
df['salary_quartile'] = pd.qcut(df['salary'],
                                 q=4,
                                 labels=['Q1', 'Q2', 'Q3', 'Q4'])
print("\nSalary quartiles:")
print(df['salary_quartile'].value_counts().sort_index())
```

---

## 8. Sorting and Ranking

```python
# ตัวอย่างที่ 28: sort_values
import pandas as pd

df = create_employee_data(20)

# Sort by single column
df_sorted = df.sort_values('salary', ascending=False)
print("Sorted by salary (desc):")
print(df_sorted[['name', 'salary', 'department']].head(6))

# Sort by multiple columns
df_multi_sort = df.sort_values(
    ['department', 'salary'],
    ascending=[True, False]  # dept ASC, salary DESC
)
print("\nSorted by dept ASC, salary DESC:")
print(df_multi_sort[['name', 'department', 'salary']].head(10))

# Na values position
df_na_first = df.sort_values('salary', na_position='first')  # NaN ก่อน
df_na_last = df.sort_values('salary', na_position='last')    # NaN ท้าย
print("\nNaN handling:")
print(df_na_first[['name', 'salary']].head(3))

# sort_index
df_reset = df.sort_values('salary').reset_index()
df_reindexed = df_reset.sort_index()
```

```python
# ตัวอย่างที่ 29: rank - จัดอันดับ
import pandas as pd

df = create_employee_data(20)

# rank() - rank ของแต่ละ element
df['salary_rank'] = df['salary'].rank(ascending=False)  # 1 = highest
df['perf_rank'] = df['performance_score'].rank(ascending=False)

print("Rankings:")
print(df[['name', 'salary', 'salary_rank', 'performance_score', 'perf_rank']].head(8))

# method ต่างๆ สำหรับ ties
scores = pd.Series([90, 85, 90, 78, 85])
print("\nRank methods for ties:")
print("average:", scores.rank(method='average').values)  # default, ค่าเฉลี่ยของ tied ranks
print("min:    ", scores.rank(method='min').values)      # เอา rank ต่ำสุด
print("max:    ", scores.rank(method='max').values)      # เอา rank สูงสุด
print("first:  ", scores.rank(method='first').values)   # เอาลำดับการปรากฏ
print("dense:  ", scores.rank(method='dense').values)   # ไม่เว้นว่าง

# Rank แบ่งกลุ่ม (rank within group)
df['dept_salary_rank'] = df.groupby('department')['salary'].rank(ascending=False)
print("\nRank within department:")
print(df[['name', 'department', 'salary', 'dept_salary_rank']].head(10))
```

---

## 9. Groupby Operations

### การจัดกลุ่มและ Aggregate

```python
# ตัวอย่างที่ 30: groupby พื้นฐาน
import pandas as pd

df = create_employee_data(100)

# groupby single column
dept_groups = df.groupby('department')

# Aggregate functions
print("Mean salary by department:")
print(dept_groups['salary'].mean().round(2))

print("\nCount by department:")
print(dept_groups['name'].count())

print("\nMultiple aggregations:")
print(dept_groups['salary'].agg(['mean', 'median', 'std', 'min', 'max']).round(2))
```

```python
# ตัวอย่างที่ 31: agg - multiple aggregations
import pandas as pd

df = create_employee_data(100)

# agg ด้วย dict: {column: [functions]}
result = df.groupby('department').agg({
    'salary': ['mean', 'median', 'std', 'count'],
    'age': ['mean', 'min', 'max'],
    'performance_score': ['mean', 'max']
}).round(2)

print("Aggregated stats:")
print(result)

# Named aggregations (pandas 0.25+)
result2 = df.groupby('department').agg(
    avg_salary=('salary', 'mean'),
    median_salary=('salary', 'median'),
    employee_count=('name', 'count'),
    avg_performance=('performance_score', 'mean'),
    max_age=('age', 'max')
).round(2)

print("\nNamed aggregations:")
print(result2)
```

```python
# ตัวอย่างที่ 32: groupby หลาย columns
import pandas as pd

df = create_employee_data(100)

# Group by multiple columns
result = df.groupby(['department', 'city'])['salary'].mean().round(2)
print("Mean salary by dept + city:")
print(result)

print("\nAs DataFrame:")
print(result.reset_index())

# groupby หลาย columns + หลาย aggregations
result2 = df.groupby(['department', 'gender']).agg(
    count=('name', 'count'),
    avg_salary=('salary', 'mean')
).round(2)

print("\nBy dept + gender:")
print(result2)
```

```python
# ตัวอย่างที่ 33: transform - เพิ่ม aggregated values กลับไปใน DataFrame
import pandas as pd

df = create_employee_data(30)

# transform คืน Series ที่มีขนาดเท่ากับ original (ไม่ reduce)
df['dept_avg_salary'] = df.groupby('department')['salary'].transform('mean')
df['salary_vs_dept_avg'] = df['salary'] - df['dept_avg_salary']
df['salary_percentile_in_dept'] = df.groupby('department')['salary'].transform(
    lambda x: x.rank(pct=True)
)

print("Employee salary vs department average:")
print(df[['name', 'department', 'salary', 'dept_avg_salary',
          'salary_vs_dept_avg', 'salary_percentile_in_dept']].head(10).round(2))
```

---

## 10. Merge, Join, Concat

### การรวม DataFrames

```python
# ตัวอย่างที่ 34: pd.concat - เชื่อม DataFrames
import pandas as pd

# สร้างข้อมูลตัวอย่าง
df1 = pd.DataFrame({
    'id': [1, 2, 3],
    'name': ['Alice', 'Bob', 'Charlie'],
    'score': [85, 90, 78]
})

df2 = pd.DataFrame({
    'id': [4, 5],
    'name': ['Diana', 'Eve'],
    'score': [92, 88]
})

# Vertical concat (เพิ่ม rows)
df_concat = pd.concat([df1, df2], ignore_index=True)
print("Vertical concat:")
print(df_concat)

# Horizontal concat (เพิ่ม columns)
df_grades = pd.DataFrame({'grade': ['B', 'A', 'C', 'A', 'B']})
df_horizontal = pd.concat([df_concat, df_grades], axis=1)
print("\nHorizontal concat:")
print(df_horizontal)

# concat ที่ index ไม่ตรงกัน
df_a = pd.DataFrame({'x': [1, 2, 3]}, index=[0, 1, 2])
df_b = pd.DataFrame({'x': [4, 5, 6]}, index=[2, 3, 4])
concat_outer = pd.concat([df_a, df_b])            # outer (default)
print("\nConcat with overlapping index:")
print(concat_outer)
```

```python
# ตัวอย่างที่ 35: pd.merge - SQL-style joins
import pandas as pd

# สร้าง relational tables
employees = pd.DataFrame({
    'emp_id': [1, 2, 3, 4, 5],
    'name': ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve'],
    'dept_id': [10, 20, 10, 30, 20]
})

departments = pd.DataFrame({
    'dept_id': [10, 20, 30, 40],
    'dept_name': ['Engineering', 'Marketing', 'Finance', 'HR'],
    'budget': [5000000, 3000000, 4000000, 2000000]
})

projects = pd.DataFrame({
    'project_id': [101, 102, 103, 104],
    'emp_id': [1, 2, 1, 6],  # emp_id 6 ไม่มีใน employees
    'project_name': ['Alpha', 'Beta', 'Gamma', 'Delta']
})

print("Employees:")
print(employees)
print("\nDepartments:")
print(departments)

# INNER JOIN (default)
inner = pd.merge(employees, departments, on='dept_id', how='inner')
print("\nInner join (employees with departments):")
print(inner)
```

```python
# ตัวอย่างที่ 36: join types
import pandas as pd

employees = pd.DataFrame({
    'emp_id': [1, 2, 3, 4, 5],
    'name': ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve'],
    'dept_id': [10, 20, 10, 30, 20]
})

departments = pd.DataFrame({
    'dept_id': [10, 20, 30, 40],
    'dept_name': ['Engineering', 'Marketing', 'Finance', 'HR'],
})

# LEFT JOIN - เก็บทุก row จาก left table
left = pd.merge(employees, departments, on='dept_id', how='left')
print("Left join:")
print(left)

# RIGHT JOIN - เก็บทุก row จาก right table
right = pd.merge(employees, departments, on='dept_id', how='right')
print("\nRight join (dept 40 = HR has no employees):")
print(right)

# OUTER JOIN - เก็บทุก row จากทั้งสองตาราง
outer = pd.merge(employees, departments, on='dept_id', how='outer')
print("\nOuter join:")
print(outer)
```

```python
# ตัวอย่างที่ 37: merge กับ column names ต่างกัน
import pandas as pd

orders = pd.DataFrame({
    'order_id': [1001, 1002, 1003, 1004],
    'customer_id': [101, 102, 101, 103],
    'amount': [500, 750, 300, 1200]
})

customers = pd.DataFrame({
    'cust_id': [101, 102, 103, 104],  # ชื่อต่างกัน!
    'name': ['Alice', 'Bob', 'Charlie', 'Diana'],
    'city': ['Bangkok', 'CM', 'Bangkok', 'Phuket']
})

# ใช้ left_on และ right_on
merged = pd.merge(orders, customers,
                  left_on='customer_id',
                  right_on='cust_id')
print("Merge with different column names:")
print(merged)

# ลบ column ที่ซ้ำซ้อน
merged = merged.drop('cust_id', axis=1)
print("\nAfter dropping cust_id:")
print(merged)
```

---

## 11. Pivot Tables

### Pivot Table คือการ Reshape ข้อมูล

```python
# ตัวอย่างที่ 38: pivot_table
import pandas as pd
import numpy as np

# สร้าง sales data
np.random.seed(42)
n = 200
sales_data = pd.DataFrame({
    'month': np.random.choice(['Jan', 'Feb', 'Mar', 'Apr'], n),
    'product': np.random.choice(['Laptop', 'Phone', 'Tablet'], n),
    'region': np.random.choice(['North', 'South', 'East'], n),
    'sales': np.random.randint(1000, 10000, n),
    'quantity': np.random.randint(1, 20, n)
})

# Basic pivot table
pivot = pd.pivot_table(
    sales_data,
    values='sales',
    index='month',
    columns='product',
    aggfunc='sum',
    fill_value=0
)
print("Sales pivot table (month × product):")
print(pivot)

# Multiple aggregations
pivot2 = pd.pivot_table(
    sales_data,
    values=['sales', 'quantity'],
    index='region',
    columns='product',
    aggfunc={'sales': 'sum', 'quantity': 'mean'},
    fill_value=0
)
print("\nMulti-aggregation pivot:")
print(pivot2)
```

```python
# ตัวอย่างที่ 39: pivot vs pivot_table
import pandas as pd

# pivot - ไม่ aggregate (ต้องมีค่า unique)
df = pd.DataFrame({
    'date': ['2024-01', '2024-01', '2024-02', '2024-02'],
    'product': ['A', 'B', 'A', 'B'],
    'sales': [100, 200, 150, 250]
})

pivoted = df.pivot(index='date', columns='product', values='sales')
print("Simple pivot:")
print(pivoted)

# melt - reverse ของ pivot (wide to long)
melted = pivoted.reset_index().melt(
    id_vars='date',
    var_name='product',
    value_name='sales'
)
print("\nAfter melt (back to original format):")
print(melted)
```

```python
# ตัวอย่างที่ 40: crosstab - frequency table
import pandas as pd

df = create_employee_data(100)

# crosstab - นับ frequency
ct = pd.crosstab(df['department'], df['gender'])
print("Crosstab: Department × Gender")
print(ct)

# พร้อม margins (totals)
ct_margins = pd.crosstab(df['department'], df['gender'], margins=True)
print("\nWith margins:")
print(ct_margins)

# normalize - เป็น percentage
ct_norm = pd.crosstab(df['department'], df['gender'], normalize='index')
print("\nNormalized (row percentage):")
print(ct_norm.round(3))
```

```python
# ตัวอย่างที่ 41: stack และ unstack - reshape MultiIndex
import pandas as pd
import numpy as np

# สร้าง MultiIndex DataFrame
df = pd.DataFrame(
    np.random.randint(100, 1000, (4, 6)),
    index=pd.MultiIndex.from_tuples([
        ('2024', 'Q1'), ('2024', 'Q2'),
        ('2025', 'Q1'), ('2025', 'Q2')
    ], names=['year', 'quarter']),
    columns=['Laptop', 'Phone', 'Tablet', 'Mouse', 'Keyboard', 'Monitor']
)

print("MultiIndex DataFrame:")
print(df)

# stack - columns ไปเป็น index level (wide to long)
stacked = df.stack()
print("\nAfter stack:")
print(stacked.head(10))
print(f"Shape: {stacked.shape}")  # (24,) หรือ (24, 1)

# unstack - index level ไปเป็น columns (long to wide)
unstacked = stacked.unstack()
print("\nAfter unstack (same as original):")
print(unstacked)
```

---

## 12. แบบฝึกหัด

### ข้อ 1: DataFrame Creation และ Inspection
สร้าง DataFrame ของนักเรียน 20 คนที่มีคอลัมน์: student_id, name, math, science, english, grade
- คำนวณ total_score และ average_score
- หา students ที่ average >= 75 (pass)
- แสดง statistics ของแต่ละวิชา

```python
# เฉลยข้อ 1
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)
n_students = 20

students = pd.DataFrame({
    'student_id': [f'STD{i:03d}' for i in range(1, n_students+1)],
    'name': [f'Student_{i}' for i in range(1, n_students+1)],
    'math': rng.integers(40, 100, n_students),
    'science': rng.integers(40, 100, n_students),
    'english': rng.integers(40, 100, n_students)
})

# คำนวณ total และ average
students['total_score'] = students[['math', 'science', 'english']].sum(axis=1)
students['average_score'] = students['total_score'] / 3

# Grade ตาม average
def assign_grade(avg):
    if avg >= 90: return 'A'
    elif avg >= 80: return 'B'
    elif avg >= 70: return 'C'
    elif avg >= 60: return 'D'
    else: return 'F'

students['grade'] = students['average_score'].apply(assign_grade)

print("Student DataFrame:")
print(students.head(10))
print(f"\nTotal students: {len(students)}")
print(f"Passing students (avg >= 75): {(students['average_score'] >= 75).sum()}")
print(f"\nSubject statistics:")
print(students[['math', 'science', 'english']].describe())
```

### ข้อ 2: Data Cleaning Pipeline
สร้าง pipeline ทำความสะอาดข้อมูลที่มี: missing values, duplicates, wrong dtypes, outliers

```python
# เฉลยข้อ 2
import pandas as pd
import numpy as np

# ข้อมูลสกปรก
dirty_df = pd.DataFrame({
    'id': [1, 2, 2, 3, 4, 5, 5, 6, 7, 8],
    'name': ['Alice', 'Bob', 'Bob', 'Charlie', None, 'Eve',
             'Eve', 'Frank', 'Grace', 'Henry'],
    'age': [25, 200, 200, -5, 30, 35, 35, 28, 999, 32],  # invalid ages
    'salary': ['50000', '65k', '65k', '80000', '55000',
               None, None, '72000', 'N/A', '63000'],
    'email': ['alice@test.com', 'bob@test.com', 'bob@test.com',
              'charlie', None, 'eve@test.com', 'eve@test.com',
              'frank@test.com', 'grace@test.com', 'henry@test.com']
})

print("Before cleaning:")
print(dirty_df)
print(f"\nShape: {dirty_df.shape}")
print(f"Missing values:\n{dirty_df.isnull().sum()}")

# Cleaning pipeline
def clean_employee_data(df):
    df = df.copy()

    # 1. ลบ duplicates
    df = df.drop_duplicates(subset=['id'], keep='first')
    print(f"\nAfter dedup: {len(df)} rows")

    # 2. ทำความสะอาด salary
    df['salary'] = (df['salary']
        .str.replace('k', '000', regex=False)
        .replace('N/A', np.nan)
        .astype(float))

    # 3. ลบ/แก้ไข invalid ages
    df.loc[(df['age'] < 0) | (df['age'] > 120), 'age'] = np.nan

    # 4. ลบ rows ที่ name เป็น None
    df = df.dropna(subset=['name'])
    print(f"After name dropna: {len(df)} rows")

    # 5. validate email
    df['email_valid'] = df['email'].str.contains(r'@.*\.', na=False, regex=True)

    # 6. fill missing salary ด้วย median
    df['salary'] = df['salary'].fillna(df['salary'].median())

    # 7. fill missing age ด้วย mean
    df['age'] = df['age'].fillna(df['age'].mean()).round()

    return df

cleaned = clean_employee_data(dirty_df)
print("\nAfter cleaning:")
print(cleaned)
print(f"\nMissing values:\n{cleaned.isnull().sum()}")
```

### ข้อ 3: GroupBy Analysis
วิเคราะห์ข้อมูลพนักงานตาม department และ city

```python
# เฉลยข้อ 3
import pandas as pd
import numpy as np

df = create_employee_data(200)

# 1. Statistics ต่อ department
dept_stats = df.groupby('department').agg(
    headcount=('name', 'count'),
    avg_salary=('salary', 'mean'),
    median_salary=('salary', 'median'),
    avg_experience=('years_experience', 'mean'),
    avg_performance=('performance_score', 'mean')
).round(2)
print("Department Statistics:")
print(dept_stats)

# 2. ระบุ top performer ของแต่ละ department
top_performers = (df.sort_values('performance_score', ascending=False)
                    .drop_duplicates('department')
                    [['name', 'department', 'performance_score']])
print("\nTop performer per department:")
print(top_performers)

# 3. Salary percentile ใน department
df['dept_salary_pct'] = df.groupby('department')['salary'].transform(
    lambda x: x.rank(pct=True)
).round(3)

# 4. หา departments ที่มีความเหลื่อมล้ำสูง (high salary variance)
salary_gini = df.groupby('department')['salary'].agg(
    lambda x: np.sqrt(np.var(x)) / np.mean(x)  # Coefficient of Variation
).round(4)
print("\nSalary Coefficient of Variation by Department:")
print(salary_gini.sort_values(ascending=False))
```

### ข้อ 4: Merge สร้าง Complete Employee Report
รวมข้อมูลจาก 3 tables: employees, departments, performance_reviews

```python
# เฉลยข้อ 4
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)

# Table 1: Employees
employees = pd.DataFrame({
    'emp_id': range(1, 21),
    'name': [f'Employee_{i}' for i in range(1, 21)],
    'dept_id': rng.integers(1, 6, 20),
    'hire_date': pd.date_range('2018-01-01', periods=20, freq='90D')
})

# Table 2: Departments
departments = pd.DataFrame({
    'dept_id': [1, 2, 3, 4, 5],
    'dept_name': ['Engineering', 'Marketing', 'Finance', 'HR', 'Operations'],
    'dept_head': ['Dr. Smith', 'Ms. Johnson', 'Mr. Brown', 'Mrs. Davis', 'Mr. Wilson'],
    'budget': [10000000, 5000000, 8000000, 3000000, 6000000]
})

# Table 3: Performance Reviews
reviews = pd.DataFrame({
    'emp_id': list(range(1, 21)) + [1, 5, 10],  # some have multiple reviews
    'review_year': [2023] * 20 + [2022, 2022, 2022],
    'score': rng.uniform(2.5, 5.0, 23).round(1),
    'promotion_eligible': rng.choice([True, False], 23)
})

# เอาเฉพาะ latest review ของแต่ละ employee
latest_reviews = reviews.sort_values('review_year', ascending=False).drop_duplicates('emp_id')

# Merge ทั้งหมด
full_report = (employees
    .merge(departments, on='dept_id', how='left')
    .merge(latest_reviews, on='emp_id', how='left')
)

# คำนวณ tenure
full_report['tenure_years'] = (
    (pd.Timestamp('2024-01-01') - full_report['hire_date']).dt.days / 365
).round(1)

print("Complete Employee Report:")
print(full_report[['name', 'dept_name', 'hire_date', 'tenure_years',
                    'score', 'promotion_eligible']].head(10))
print(f"\nTotal records: {len(full_report)}")
print(f"Promotion eligible: {full_report['promotion_eligible'].sum()}")
```

### ข้อ 5: Pivot Analysis - Sales Dashboard
ทำ Pivot Table สำหรับ sales data และ compute insights

```python
# เฉลยข้อ 5
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)
n = 500

# สร้าง sales data
sales = pd.DataFrame({
    'date': pd.date_range('2024-01-01', periods=n, freq='D')[:n],
    'product': rng.choice(['Laptop', 'Phone', 'Tablet', 'Watch'], n),
    'region': rng.choice(['North', 'South', 'East', 'West'], n),
    'salesperson': rng.choice([f'SP{i:02d}' for i in range(1, 11)], n),
    'quantity': rng.integers(1, 20, n),
    'unit_price': rng.choice([999, 1299, 499, 799], n)
})
sales['revenue'] = sales['quantity'] * sales['unit_price']
sales['month'] = sales['date'].dt.strftime('%Y-%m')

# Pivot 1: Revenue by Product × Month
rev_pivot = pd.pivot_table(
    sales,
    values='revenue',
    index='product',
    columns='month',
    aggfunc='sum',
    fill_value=0
)
print("Revenue by Product × Month:")
print(rev_pivot)

# Pivot 2: Quantity by Region × Product
qty_pivot = pd.pivot_table(
    sales,
    values='quantity',
    index='region',
    columns='product',
    aggfunc='sum',
    margins=True,  # รวม row/col totals
    margins_name='Total'
)
print("\nQuantity by Region × Product:")
print(qty_pivot)

# Top 3 salesperson ต่อ region
top_sp = (sales.groupby(['region', 'salesperson'])['revenue']
               .sum()
               .reset_index()
               .sort_values('revenue', ascending=False)
               .groupby('region')
               .head(3))
print("\nTop 3 salesperson per region:")
print(top_sp)
```

### ข้อ 6: Boolean Filtering Complex Queries
เขียน queries สำหรับ employee data:

```python
# เฉลยข้อ 6
import pandas as pd
import numpy as np

df = create_employee_data(200)

# Q1: Senior IT employees (experience >= 8) with high performance (>= 4.0)
q1 = df[(df['department'] == 'IT') &
        (df['years_experience'] >= 8) &
        (df['performance_score'] >= 4.0)]
print(f"Q1 - Senior high-performing IT: {len(q1)} employees")
print(q1[['name', 'years_experience', 'salary', 'performance_score']].head(5))

# Q2: ใน Bangkok หรือ Chiang Mai ที่ salary > median
median_salary = df['salary'].median()
q2 = df[(df['city'].isin(['Bangkok', 'Chiang Mai'])) & (df['salary'] > median_salary)]
print(f"\nQ2 - BKK/CM employees above median salary: {len(q2)}")

# Q3: ไม่ใช่ HR และ Marketing ที่ age < 30
q3 = df[~(df['department'].isin(['HR', 'Marketing'])) & (df['age'] < 30)]
print(f"\nQ3 - Non-HR/Marketing under 30: {len(q3)}")

# Q4: Employees ที่ salary ต่ำกว่า 60% ของ department average (underpaid)
dept_avg = df.groupby('department')['salary'].transform('mean')
q4 = df[df['salary'] < dept_avg * 0.6]
print(f"\nQ4 - Employees paid < 60% of dept avg (underpaid): {len(q4)}")
if len(q4) > 0:
    print(q4[['name', 'department', 'salary']].head())
```

### ข้อ 7: Data Transformation Pipeline
ทำ complete transformation pipeline สำหรับ raw data

```python
# เฉลยข้อ 7
import pandas as pd
import numpy as np

raw = create_employee_data(100)

def transform_pipeline(df):
    """Complete transformation pipeline"""
    return (df
        # เพิ่ม computed columns
        .assign(
            salary_k=lambda x: (x['salary'] / 1000).round(1),
            age_group=lambda x: pd.cut(
                x['age'], bins=[0, 30, 40, 50, 100],
                labels=['Young', 'Mid', 'Senior', 'Veteran']
            ),
            exp_group=lambda x: pd.cut(
                x['years_experience'], bins=[0, 3, 7, 15, 30],
                labels=['Junior', 'Mid', 'Senior', 'Principal']
            ),
            is_high_performer=lambda x: x['performance_score'] >= 4.0,
            salary_rank_in_dept=lambda x: x.groupby('department')['salary']
                                            .rank(ascending=False, method='min'),
            tenure_category=lambda x: np.where(
                x['years_experience'] < 3, 'New',
                np.where(x['years_experience'] < 7, 'Regular', 'Veteran')
            )
        )
        # Drop original columns ที่แทนด้วยที่ใหม่
        .drop(columns=['employee_id'])
        # Rename
        .rename(columns={'years_experience': 'exp_years'})
        # Reorder columns
        [['name', 'age', 'age_group', 'gender', 'department',
          'city', 'salary', 'salary_k', 'exp_years', 'exp_group',
          'tenure_category', 'performance_score', 'is_high_performer',
          'salary_rank_in_dept', 'join_date']]
    )

processed = transform_pipeline(raw)
print("Transformed DataFrame:")
print(processed.head(8))
print(f"\nShape: {processed.shape}")
print(f"\nDtypes:\n{processed.dtypes}")
```

### ข้อ 8: Comprehensive Groupby
วิเคราะห์ข้อมูลลูกค้าและ transactions

```python
# เฉลยข้อ 8
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)
n_transactions = 500

# สร้าง transaction data
transactions = pd.DataFrame({
    'transaction_id': [f'TXN{i:05d}' for i in range(1, n_transactions+1)],
    'customer_id': rng.integers(1001, 1051, n_transactions),  # 50 customers
    'product_category': rng.choice(['Electronics', 'Clothing', 'Food', 'Books', 'Sports'], n_transactions),
    'amount': rng.integers(100, 10000, n_transactions).astype(float),
    'date': pd.date_range('2024-01-01', periods=n_transactions, freq='H')[:n_transactions]
})

# Customer analysis
customer_stats = transactions.groupby('customer_id').agg(
    total_spent=('amount', 'sum'),
    avg_transaction=('amount', 'mean'),
    transaction_count=('transaction_id', 'count'),
    favorite_category=('product_category', lambda x: x.mode()[0]),
    first_purchase=('date', 'min'),
    last_purchase=('date', 'max')
).round(2)

customer_stats['days_since_last'] = (
    pd.Timestamp('2024-12-31') - customer_stats['last_purchase']
).dt.days

# Customer segments (RFM-lite)
customer_stats['segment'] = pd.cut(
    customer_stats['total_spent'],
    bins=[0, 5000, 20000, 50000, float('inf')],
    labels=['Low', 'Medium', 'High', 'VIP']
)

print("Customer Analysis:")
print(customer_stats.head(10))
print(f"\nCustomer segments:")
print(customer_stats['segment'].value_counts())
print(f"\nAvg spending by segment:")
print(customer_stats.groupby('segment')['total_spent'].mean().round(2))
```

### ข้อ 9: Merge Complex
รวม 4 tables เพื่อสร้าง analytical report

```python
# เฉลยข้อ 9
import pandas as pd
import numpy as np

rng = np.random.default_rng(42)

# 4 tables
orders = pd.DataFrame({
    'order_id': range(1001, 1051),
    'customer_id': rng.integers(101, 121, 50),
    'product_id': rng.integers(201, 211, 50),
    'order_date': pd.date_range('2024-01-01', periods=50, freq='7D'),
    'quantity': rng.integers(1, 10, 50),
    'discount': rng.choice([0, 0.05, 0.10, 0.15], 50)
})

customers = pd.DataFrame({
    'customer_id': range(101, 121),
    'customer_name': [f'Customer_{i}' for i in range(101, 121)],
    'tier': rng.choice(['Bronze', 'Silver', 'Gold', 'Platinum'], 20)
})

products = pd.DataFrame({
    'product_id': range(201, 211),
    'product_name': [f'Product_{i}' for i in range(201, 211)],
    'category': rng.choice(['A', 'B', 'C'], 10),
    'unit_price': rng.integers(500, 5000, 10)
})

# Build complete order report
report = (orders
    .merge(customers, on='customer_id', how='left')
    .merge(products, on='product_id', how='left')
)

report['revenue'] = (report['quantity'] * report['unit_price'] *
                      (1 - report['discount']))

print("Complete Order Report:")
print(report[['order_id', 'customer_name', 'product_name',
              'quantity', 'discount', 'revenue']].head(10))

# Summary by customer tier
tier_summary = report.groupby('tier').agg(
    total_orders=('order_id', 'count'),
    total_revenue=('revenue', 'sum'),
    avg_order_value=('revenue', 'mean')
).round(2)
print("\nRevenue by Customer Tier:")
print(tier_summary)
```

### ข้อ 10: End-to-End Analysis
วิเคราะห์ข้อมูลทั้งหมดและสร้าง report

```python
# เฉลยข้อ 10
import pandas as pd
import numpy as np

df = create_employee_data(200)

print("=" * 60)
print("EMPLOYEE ANALYTICS REPORT")
print("=" * 60)

# 1. Executive Summary
print("\n1. EXECUTIVE SUMMARY")
print(f"   Total Employees: {len(df)}")
print(f"   Departments: {df['department'].nunique()}")
print(f"   Cities: {df['city'].nunique()}")
print(f"   Avg Salary: {df['salary'].mean():,.0f} THB")
print(f"   Avg Performance: {df['performance_score'].mean():.2f}/5.0")

# 2. Department Overview
print("\n2. DEPARTMENT BREAKDOWN")
dept_overview = df.groupby('department').agg(
    headcount=('name', 'count'),
    avg_salary=('salary', 'mean'),
    avg_performance=('performance_score', 'mean'),
    high_performers=('performance_score', lambda x: (x >= 4.0).sum())
).round(2)
print(dept_overview)

# 3. Geographic Distribution
print("\n3. GEOGRAPHIC DISTRIBUTION")
city_dist = df.groupby('city').agg(
    count=('name', 'count'),
    avg_salary=('salary', 'mean'),
    total_salary_cost=('salary', 'sum')
).round(2)
print(city_dist)

# 4. Salary Analysis
print("\n4. SALARY ANALYSIS")
salary_quartiles = df['salary'].quantile([0.25, 0.5, 0.75, 0.9]).round(2)
print(f"   Q1 (25th): {salary_quartiles[0.25]:,.0f} THB")
print(f"   Median:    {salary_quartiles[0.5]:,.0f} THB")
print(f"   Q3 (75th): {salary_quartiles[0.75]:,.0f} THB")
print(f"   P90:       {salary_quartiles[0.9]:,.0f} THB")

# 5. Performance Distribution
print("\n5. PERFORMANCE DISTRIBUTION")
df['grade'] = pd.cut(df['performance_score'],
                     bins=[0, 2.5, 3.5, 4.0, 4.5, 5.0],
                     labels=['Poor', 'Fair', 'Good', 'Excellent', 'Outstanding'])
print(df['grade'].value_counts().sort_index())

# 6. Correlation Analysis
print("\n6. CORRELATION")
corr = df[['age', 'salary', 'years_experience', 'performance_score']].corr().round(3)
print(corr)

# 7. Save report
df.to_csv('/tmp/employee_report.csv', index=False)
print("\n7. Saved to employee_report.csv")
print(f"   File size: ~{len(df) * 10} bytes (estimated)")
```

---

## สรุป Part 73

ใน Part นี้เราได้เรียนรู้:

| หัวข้อ | สิ่งที่ได้เรียน |
|--------|----------------|
| **Series & DataFrame** | สร้าง, attributes, operations |
| **Creating DataFrames** | dict, array, CSV, Excel, SQL |
| **Inspection** | head, tail, info, describe, isnull |
| **loc vs iloc** | label-based vs position-based |
| **Boolean Selection** | conditions, isin, between, query |
| **Data Cleaning** | dropna, fillna, duplicates, type conversion |
| **Transformation** | rename, astype, map, apply, assign, cut |
| **Sorting & Ranking** | sort_values, rank, argsort |
| **Groupby** | agg, named agg, transform, filter |
| **Merge/Join** | inner, left, right, outer join |
| **Concat** | vertical, horizontal, handling duplicates |
| **Pivot Tables** | pivot_table, pivot, melt, crosstab, stack/unstack |

### Key Best Practices

1. **loc vs iloc**: ใช้ `loc` สำหรับ label-based (ชัดเจนกว่า), `iloc` สำหรับ position-based
2. **Method chaining**: ใช้ `assign()`, `query()` เพื่อ chain operations (อ่านง่าย)
3. **ใช้ `query()` แทน complex boolean**: `df.query("age > 30 and salary > 50000")`
4. **Vectorized operations**: หลีกเลี่ยง `apply` ถ้าทำ vectorized ได้
5. **Category dtype**: ใช้ `'category'` สำหรับ columns ที่มี cardinality ต่ำ ประหยัด memory

---

**ก่อนหน้า**: [Part 72 - NumPy Advanced](../part72/README.md)
**ต่อไป**: [Part 74 - Pandas Advanced Analytics](../part74/README.md)
