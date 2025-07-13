# HexaDruid 🧠⚡

[![PyPI version](https://badge.fury.io/py/hexadruid.svg)](https://badge.fury.io/py/hexadruid)  
[![Python Version](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/)  
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**HexaDruid** is an intelligent Spark optimizer designed to tackle **data skew**, **ambiguous key detection**, and **schema bloat** using smart salting, recursive shard-aware rule trees, and adaptive tuning. It enables better parallelism, safer memory layout, and intelligent insight into skewed datasets using PySpark’s native DataFrame API.

---

## 🚀 Installation

```bash
pip install hexadruid
```

To upgrade to the latest version:

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
  Single `.limit(max_sample).collect()` → regex/JSON sniff → safe `try_cast()`. Sub-second on 100K rows.

- 🌲 **DRTree Sharding**  
  One-level or recursive logical filters for skew mitigation. Always falls back to an `all` branch.

- 🔑 **Key Detection**  
  Finds primary/composite keys with confidence scoring; always returns the best candidate.

- 📈 **Auto-Parameter Advisor**  
  Detects top skewed and categorical columns using sample-based heuristics.

- 📊 **Beginner-Friendly Wrappers**  
  - `simple_optimize(df, skew_col, sample_frac, salt_count)`  
  - `visualize_salting(df, skew_col, salt_count)`  
  - `interactive_optimize(df)`

- 🚨 **Robustness**  
  Handles nulls, headerless files, corrupt formats, and verbose edge case logging.

---

## 🧠 Quickstart

```python
from pyspark.sql import SparkSession
from hexadruid import HexaDruid, simple_optimize, visualize_salting

spark = SparkSession.builder.getOrCreate()
spark.conf.set("spark.sql.shuffle.partitions", "8")  # match your bucket count

df = spark.read.csv("data.csv", header=True, inferSchema=False)

# 1) Fast schema + default DRTree
typed_df, schema, dr_tree = HexaDruid(df).schemaVisor()
print("Schema:", schema.simpleString())

# 2) Auto heavy-hitter salting
df_salted = HexaDruid(df).apply_smart_salting()
df_salted.groupBy("salt").count().show()

# 3) One-liner optimize
df_opt = simple_optimize(df, skew_col="user_id", salt_count=5)
df_opt.groupBy("salt").count().show()

# 4) Visualize before/after
visualize_salting(df, skew_col="user_id", salt_count=5)

# 5) Interactive tuning
df_inter = HexaDruid(df).interactive_optimize(df)
```

---

## 📚 What’s Under the Hood?

### Heavy-Hitter Salting

1. Detect the top skewed column if none is provided  
2. If `salt_count` is not set → default to `sparkContext.defaultParallelism`  
3. Identify heavy keys by frequency  
4. Assign:
   - Heavy keys → `floor(rand() * salt_count)`
   - Light keys → `pmod(hash(key), salt_count)`  
5. Create `salted_key` → repartition → cache

---

### Fast Schema Inference

- `.limit()` to sample rows  
- Regex, numeric range, JSON, and boolean sniffing  
- Null-safe `try_cast()` for Spark stability  
- Zero RDDs or Pandas usage

---

### DRTree Logic

- Splits on skewed columns  
- Supports recursive logical branching  
- Each node = a Spark SQL predicate  
- Always returns a fallback all-true branch

---

### Key Detection

- Primary: `(distinct - nulls) / total >= threshold`  
- Composite: tests top combinations with high uniqueness  
- Shard-aware: DRTree logic passed into key scanner

---

### Auto-Parameter Advisor

- Suggests top skewed & low-cardinality categorical columns  
- Single-pass, sample-based profiling  
- Returns: `(skew_cols, groupBy_cols, metrics_df)`

---

## 🛠️ API Reference

### `HexaDruid` Core Methods

| Method               | Signature                                                                 |
|----------------------|---------------------------------------------------------------------------|
| Constructor          | `HexaDruid(df, output_dir="hexa_druid_outputs")`                          |
| `schemaVisor()`      | `(sample_frac=0.01, max_sample=1000) → (typed_df, schema, dr)`            |
| `detect_skew()`      | `(threshold=0.1, top_n=3) → List[str]`                                    |
| `apply_smart_salting()` | `(col_name=None, salt_count=None) → DataFrame`                         |
| `detect_keys()`      | `(threshold=0.99, max_combo=3) → List[str]`                               |
| `build_shard_tree()` | `(detector, max_depth=3, min_samples=500) → DRTree`                       |

---

### Wrappers & Utilities

| Function / Class          | Description                                                           |
|---------------------------|-----------------------------------------------------------------------|
| `simple_optimize()`       | Infer schema + apply smart salting in one call                        |
| `visualize_salting()`     | Print before/after z-score distributions for a given column           |
| `interactive_optimize()`  | Advisor → table → prompt → salting pipeline                           |
| `AutoParameterAdvisor`    | `recommend() → (skew_cols, cat_cols, metrics_df)`                     |
| `DRTree`, `Branch`, `Root`| Build and inspect decision-rule trees for logical sharding            |

---

## 🧪 Testing

```bash
pytest tests/
```

---

## 🛣️ Roadmap

- [ ] Multi-format ingestion (Avro, Delta, Iceberg)  
- [ ] Streaming support & incremental re-sharding  
- [ ] JupyterLab extension for visual DRTree editing  
- [ ] REST/gRPC microservice and Kubernetes charts  
- [ ] Web UI for interactive parameter tuning and audit logs  

---

## 🤝 Contributing

Pull requests and issues welcome—please see `CONTRIBUTING.md`.

---

## 📄 License

MIT © 2025 Omar Hossam Attia
