# HexaDruid: Intelligent Data Skew & Schema Optimizer for PySpark  
**Inspired by Ancient Druid Wisdom — Optimized for Modern Spark Pipelines**

[![PyPI version](https://badge.fury.io/py/hexadruid.svg)](https://badge.fury.io/py/hexadruid)  
[![Python Version](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/)  
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)  
[![LinkedIn: Omar Hossam Attia](https://img.shields.io/badge/LinkedIn-Connect-blue?logo=linkedin)](http://linkedin.com/in/omar-h-attia-pmp®-855a091a4)  
[![Medium: Read the Article](https://img.shields.io/badge/Medium-HexaDruid_Story-black?logo=medium)](https://medium.com/@wrbyspdkf/introducing-hexadruid-using-ancient-druid-wisdom-to-optimize-spark-pipelines-c2ee09dc0b38)

**HexaDruid** is an intelligent Spark optimizer designed to tackle **data skew**, **ambiguous key detection**, and **schema bloat** using smart salting, recursive shard-aware rule trees (DRTree), and adaptive tuning.  
It enables better parallelism, safer memory layout, and intelligent insight into skewed datasets using **pure PySpark DataFrame API**.

---

## 🚀 Installation

```bash
pip install hexadruid
```

To upgrade:

```bash
pip install --upgrade hexadruid
```

---

## 🔍 Features

- 🎯 **Heavy-Hitter Salting**  
  Auto-detects the hottest keys and spreads them randomly; hashes the rest. Two–stage shuffle with O(N) performance.

- ⚙️ **Dynamic API**  
  ```python
  hd.apply_smart_salting(col_name=None, salt_count=None)
  ```
  Auto-detects `col_name` and `salt_count` if omitted.

- 🧠 **Fast Schema Inference**  
  One-pass `.limit().collect()` → regex/JSON/type sniffing → safe `try_cast()` for robust schema guessing.

- 🌲 **DRTree Sharding**  
  Recursive logical splits based on SQL predicate logic. Always returns fallback `[TRUE]` node if no pattern detected.

- 🔑 **Key Detection**  
  Confidence-based scoring for primary/composite keys:
  ```
  key_score = (distinct_non_null / total_count)
  composite_score = ∑ (individual_uniqueness × weights)
  ```

- 📈 **Auto-Parameter Advisor**  
  Detects top skewed and categorical fields using sample-based heuristics + IQR-based analysis.

- 🧪 **Pandas-Free Architecture**  
  No `.toPandas()` or Arrow dependency. Operates entirely on Spark-native DataFrames.

---

## 🧠 Quickstart

```python
from pyspark.sql import SparkSession
from hexadruid import HexaDruid, simple_optimize, visualize_salting

spark = SparkSession.builder.getOrCreate()
spark.conf.set("spark.sql.shuffle.partitions", "8")

df = spark.read.csv("data.csv", header=True, inferSchema=False)

# 1. Schema inference with DRTree
typed_df, schema, dr_tree = HexaDruid(df).schemaVisor()

# 2. Apply salting
df_salted = HexaDruid(df).apply_smart_salting()

# 3. One-liner optimization
df_opt = simple_optimize(df, skew_col="user_id", salt_count=5)

# 4. Visualize
visualize_salting(df, skew_col="user_id", salt_count=5)

# 5. Interactive auto advisor
df_inter = HexaDruid(df).interactive_optimize(df)
```

---

## 🌳 DRTree Logic Architecture

```
                 ╔═══════════════════════╗
                 ║      ROOT: amount     ║
                 ╚═══════════════════════╝
                           |
                ┌──────────┴──────────┐
        IF SPLIT ≤500        IF SPLIT >500
                |                   |
       ╔════════╝         ╔═════════╧═════════╗
       ║ DecisionNode     ║ DecisionNode      ║
       ╚════════════════  ╚═══════════════════╝
            |                       |
    IF SPLIT ≤100          IF SPLIT >1000
        |                       |
     [Leaf A]                [Leaf B]

Legend:
- ROOT ➜ Initial split column (usually top skewed)
- IF SPLIT ➜ Predicate-based routing (e.g., col ≤ val)
- DecisionNode ➜ A branch that leads to further splits
- Leaf ➜ Terminal node with defined predicate path
```

---

## 📚 Internals & Algorithms

### 🧮 Heavy-Hitter Salting

```text
salt = CASE
  WHEN key IN (top_heavy_keys) THEN floor(rand() * salt_count)
  ELSE pmod(hash(key), salt_count)
END
```

### 🧬 Schema Inference

- `.limit(max_sample)` rows
- Detects:
  - numeric types: `regex`, `range`
  - bools: `["true", "false", "1", "0"]`
  - JSON fields: `is_json(col)`
- Applies:
  ```python
  expr(f"try_cast({col} AS {target_type})")
  ```

### 🧩 Key Detection

```python
key_score = (approx_distinct(col) - null_count) / total_count
```

- Composite:
  ```python
  for combo in combinations(columns, max_combo):
      if key_score(combo) > threshold: return combo
  ```

---

## 🛠️ API Reference

### 🔧 Core Class: `HexaDruid`

| Method               | Signature                                                                 |
|----------------------|---------------------------------------------------------------------------|
| `__init__`           | `HexaDruid(df, output_dir="hexa_druid_outputs")`                          |
| `schemaVisor()`      | `(sample_frac=0.01, max_sample=1000)` → `(typed_df, schema, dr_tree)`    |
| `detect_skew()`      | `(threshold=0.1, top_n=3)` → `List[str]`                                  |
| `apply_smart_salting()`| `(col_name=None, salt_count=None)` → `DataFrame`                        |
| `detect_keys()`      | `(threshold=0.99, max_combo=3)` → `List[str]`                             |
| `build_shard_tree()` | `(detector, max_depth=3, min_samples=500)` → `DRTree`                     |

---

### 🎯 Wrappers & Utilities

| Function / Class         | Description                                                            |
|--------------------------|------------------------------------------------------------------------|
| `simple_optimize()`      | Combines `schemaVisor + apply_smart_salting`                           |
| `visualize_salting()`    | Shows z-score distribution before/after salting                        |
| `interactive_optimize()` | Auto advisor → config prompt → salting pipeline                        |
| `AutoParameterAdvisor`   | `.recommend()` → `(skew_cols, cat_cols, metrics_df)`                   |
| `DRTree`, `Root`, `Branch`, `DecisionNode`, `Leaf` | Classes representing logical rule tree for splits |

---

## 🧪 Testing

```bash
pytest tests/
```

---

## 🛣️ Roadmap

- [ ] Delta / Iceberg / Avro support  
- [ ] Streaming DRTree inference  
- [ ] Visual DRTree editor (Jupyter extension)  
- [ ] Kubernetes-native microservice deployment  
- [ ] Prescriptive analytics & advisor explainability

---

## 🤝 Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md)  
You can also open issues or pull requests for bugs, improvements, or features.

---

## 📄 License

MIT © 2025 Omar Hossam Attia  
Version: **v0.2.2**
