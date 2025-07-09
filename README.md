# HexaDruid 🧠⚡

[![PyPI version](https://badge.fury.io/py/hexadruid.svg)](https://badge.fury.io/py/hexadruid)
[![Python Version](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**HexaDruid** is an intelligent Spark optimizer designed to tackle value skew, schema bloat, and ambiguous key detection using recursive decision-rule trees and adaptive tuning. It enables smart salting, distributed key detection, and shard-aware schema inference — all with PySpark's native DataFrame API.

---

## 📦 Installation

```bash
pip install hexadruid
```

---

## 🔍 Features

- 📊 Smart salting using IQR or z-score binning
- 🌲 Recursive DRTree-based shard splitting (non-linear DAG logic)
- 🔑 Primary & Composite Key Detection (UUIDs, strings, hexadecimals)
- 🧠 Schema Inference with memory-safe type coercion and metadata hints
- 🧪 Z-score visualizations and partition diagnostics
- ⚙️ Auto tuning of salt count and shuffle partitions
- ✅ 100% Spark-native — no RDDs, no CLI, no external dependencies

---

## 🚀 Quickstart

```python
from hexadruid import HexaDruid

hd = HexaDruid(df)

# Balance skew in a numeric column
df_salted = hd.apply_smart_salting("sales_amount")

# Detect keys (UUIDs, composite, etc.)
key = hd.detect_keys()

# Infer schema and shard tree
typed_df, inferred_schema, dr_tree = HexaDruid.schemaVisor(df)
```

---

## 📚 What Does It Do?

Imagine this DataFrame:

| order_id (UUID) | amount |
|-----------------|--------|
| a12e...         | 500.0  |
| b98c...         | 5000.0 |
| ...             | ...    |

You want to `groupBy("amount")`, but most rows have the same few values → Spark chokes on skew.

### 🔥 With HexaDruid:

```python
df2 = hd.apply_smart_salting("amount")
```

It:

1. Detects skew using IQR asymmetry or z-score
2. Creates a `salt` column using percentile splits
3. Builds `salted_key = concat(amount, salt)`
4. Repartitions the DataFrame using `salted_key`

✅ Result: Balanced shuffle across executors.

---

## 🌳 DRTree (Decision Rule Tree) — Deep Dive

HexaDruid’s `drTree()` is not a classifier. It's a recursive shard explainer that builds SQL-compatible predicates based on imbalanced features.

Each node evaluates data distribution and splits the dataset using decision logic on the best skewed column.

### ASCII Tree

```
                [Root: sales_amount]
                        |
             ┌──────────┴────────────┐
     [amount <= 500]         [amount > 500]
           |                         |
       [Leaf A]                  [Leaf B]
    (shard_1234)              (shard_5678)
```

- **Root**: Skewed feature selected for first split
- **Branches**: Logical split conditions (e.g., amount <= 500)
- **Leaves**: Filtered DataFrames with shard metadata
- **Output**: SQL predicates reusable for schemaVisor, key detection, or isolation

---

## 🔬 Smart Salting Algorithm

**Input**: Skewed numeric column  
**Goal**: Equalize shuffle load in `groupBy`, `join`

### Steps:

1. **Skew Check**:
   ```python
   z = (x - mean(x)) / std(x)   # or use IQR-based rule
   ```
2. **Percentile Splitting**:
   ```python
   bounds = percentile_approx(x, [0%, 10%, ..., 100%])
   ```
3. **Salt Assignment**:
   ```python
   salt = when((x >= b0) & (x < b1), 0).when(...)...
   ```
4. **Salted Key & Repartition**:
   ```python
   salted_key = concat_ws("_", col("x"), col("salt"))
   df.repartition(N, salted_key)
   ```

🎯 Auto-tuned salt count if not provided.

---

## 🔑 Key Detection (UUIDs, Alphanumerics, Composite)

```python
key_info = hd.detect_keys()
```

### Primary Key Scoring:

```python
score = approx_count_distinct(col) / total_rows - null_ratio
if score ≥ 0.99:
    → confident primary key
```

### Composite Key:

```python
combo = concat_ws("_", col1, col2, ...)
score = approx_count_distinct(combo) / total_rows - null_ratio
if score ≥ 0.99:
    → confident composite key
```

💡 Supports detection of:
- UUIDs
- Alphanumeric IDs
- Hexadecimal fields
- Concatenated business keys

---

## 🧠 schemaVisor (Schema + Tree Inference)

```python
typed_df, inferred_schema, tree = HexaDruid.schemaVisor(df)
```

### Features:

- Optimizes column types: string → varchar(x), float → int if safe
- Adds logical metadata (nullable, min_len, max_len)
- Connects schema + tree for shard-specific schema enforcement

---

## 🧪 Testing

```bash
pytest tests/
```

Mocked SparkSession and sample DataFrames are included.

---

## 🧱 Project Structure

```
hexadruid/
├── core.py                # Main interface
├── skew_balancer.py       # Salting + distribution logic
├── key_detection.py       # Primary + composite key analysis
├── schema_optimizer.py    # schemaVisor and type casting
├── drtree.py              # DRTree recursive engine
├── advisor.py             # Parameter tuning module
└── utils.py               # Plotting, IQR, logging
```

---

## 🛣️ Roadmap

- [ ] Delta Lake & Apache Iceberg compatibility
- [ ] CLI & REST API
- [ ] Auto JSON export of tree branches for audit
- [ ] JupyterLab plugin for visual shard insight

---

## 📄 License

MIT License

---

## 🤝 Contributing

Pull requests, issues, and ideas are welcome!  
HexaDruid is evolving — your insights shape its intelligence.
