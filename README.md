# HexaDruid 🧠⚡

[![PyPI version](https://badge.fury.io/py/hexadruid.svg)](https://badge.fury.io/py/hexadruid)
[![Python Version](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**HexaDruid** is an intelligent Spark optimizer designed to tackle value skew, schema bloat, and ambiguous key detection using decision-tree logic and adaptive tuning. It's a plug-and-play tool to make your PySpark pipelines faster, smarter, and more efficient.

---

## 🚀 Installation

```bash
pip install hexadruid
```

---

## 🔍 Features

- Smart Salting for skew mitigation via adaptive percentile binning  
- `drTree()`: Decision-rule tree that simulates shard-aware logic  
- Primary & Composite Key Detection  
- Schema Inference with type safety & memory optimization  
- Auto-Parameter Advisor for skew and groupBy tuning  
- Z-score visualizations & partition size logging  
- Spark-native (DataFrame API only) with no CLI or RDDs  

---

## 🧠 Quickstart

```python
from hexadruid import HexaDruid

# Initialize
hd = HexaDruid(df)

# Step 1: Balance skew in a column
salted_df = hd.apply_smart_salting("column_name")

# Step 2: Detect primary or composite keys
key_info = hd.detect_keys()

# Step 3: Run schema optimizer
typed_df, inferred_schema, dr_tree = HexaDruid.schemaVisor(df)
```

---

## 📊 CLI-Based Balancing (Optional)

```python
from hexadruid import balance_skew

df_balanced = balance_skew(df)
```

---

## 📈 Example Output

`Z-Score Distribution Before vs After Salting`  
Place an image here if you wish:

```
docs/zscore_example.png
```

---

## 🧠 For Beginners – What’s Going On Here?

If you’re not a Spark wizard — no problem!  
**HexaDruid** helps Spark run faster and smarter by fixing one big issue: **data skew**.

---

### 🤔 What’s Data Skew?

Imagine Spark workers like kitchen chefs:

```
              🍱🍱🍱🍱🍱          🧠
Worker 1: [ 100 tasks ]
Worker 2: [  5 tasks  ]
Worker 3: [  2 tasks  ]
```

One worker gets overwhelmed while others do nothing = slow performance.

HexaDruid fixes this with a smart trick: it "salts" the data to rebalance it:

```
         Salted Keys → Even Tasks
         ------------------------
         value_0, value_1, ..., value_9

         Each worker gets ~equal load!
```

---

### 🪄 What Does HexaDruid Do?

```python
from hexadruid import HexaDruid

hd = HexaDruid(df)
df_balanced = hd.apply_smart_salting("sales_amount")
```

Behind the scenes:

```
Step 1: Check if the column is skewed (using z-score / IQR)
Step 2: Split values into balanced buckets (percentiles)
Step 3: Add a "salt" number to each row to create uniqueness
Step 4: Repartition the data using the salted keys
```

✅ Now your `groupBy()` or `join()` runs way faster!

---

### 🧠 How to Use (Summary)

```python
# Rebalance a skewed column
hd = HexaDruid(df)
df2 = hd.apply_smart_salting("column_name")

# Detect primary or composite keys
key_info = hd.detect_keys()

# Optimize column types
typed_df, schema, dr_tree = HexaDruid.schemaVisor(df)
```

Simple. Fast. No rocket science. 🧃

---

## 📚 For Developers & Scientists – Under the Hood

### 🔬 Core Algorithm 1: Smart Salting

Salting is based on value distribution percentiles:

Let `x` be a skewed column. Compute cut points:

```
P = percentile_approx(x, [0%, 10%, 20%, ..., 100%])
```

For each range `[P_i, P_{i+1})`, assign a salt ID.

```python
salted_key = concat_ws("_", col("x").cast("string"), col("salt").cast("string"))
df.repartition(num_partitions, col("salted_key"))
```

---

### 📉 Z-score Logic (Before/After)

```text
Z = (x - mean(x)) / std(x)
```

HexaDruid plots this before and after salting to show improvement.

---

### 🌲 Core Algorithm 2: DRTree (Decision Rule Tree)

The `drTree()` engine is a custom recursive decision tree used for:

- Splitting data into shards  
- Evaluating column confidence scores  
- Handling CDC / missing schema drift  

```
         Root
          │
   ┌──────┴───────┐
[ x <= 50 ]   [ x > 50 ]
     │             │
 Leaf A         Leaf B
(shard_1)     (shard_2)
```

Each leaf = filter predicate applied to data for isolated analysis.

---

### 🔑 Key Detection Logic

A column is marked as a primary key if:

```
Score = distinct_ratio - null_ratio

If Score ≥ 0.99 → confident key
```

For composite keys:

```python
combo_key = concat_ws("_", col1, col2, ...)
score = approx_count_distinct(combo_key) / total_rows - null_ratio

If score ≥ 0.99 → valid composite key
```

---

## 🧪 Testing

```bash
pytest tests/
```

Tests are coming soon. Mocked SparkSession will be included.

---

## 🧱 Project Structure (Suggested)

```
hexadruid/
├── core.py                # HexaDruid entry point
├── skew_balancer.py       # Smart salting logic
├── key_detection.py       # Primary/composite key detection
├── schema_optimizer.py    # schemaVisor logic
├── drtree.py              # DRTree recursive logic
├── advisor.py             # AutoParameterAdvisor
└── utils.py               # Plotting, logging, etc.
```

---

## 🔧 Roadmap

- [ ] CLI support  
- [ ] Delta Lake + Iceberg support  
- [ ] REST API / JupyterLab extension  
- [ ] Export DRTree JSON for audit logging  

---

## 📄 License

MIT License

---

## 🤝 Contributing

Pull requests, issues, and stars are welcome!  
This is just the beginning for intelligent Spark tools.

---
