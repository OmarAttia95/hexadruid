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
To upgrade to the latest version?

```bash
pip install --upgrade hexadruid
```
---

## 🔍 Features

 - 🎯 **Heavy-Hitter Salting:**
    Auto-detects the hottest keys and spreads them randomly, hashes the rest—two–stage shuffle with O(N) runtime.

- ⚙️ **Dynamic API:**

```python
hd.apply_smart_salting(col_name=None, salt_count=None)
```
  - Auto-detects `col_name` and `salt_count` if omitted.

- 🧠 **Fast Schema Inference:**

  - Single `.limit(max_sample).collect()` → regex/JSON sniff → safe `try_cast()`. Sub-second on 100 K rows.

- 🌲 **DRTree Sharding**:

  - 1-level or recursive shard splits for logical filtering and lineage. Always falls back to “all” when no skew.

- 🔑 **Key Detection**:

  - Finds primary/composite keys with confidence scoring; always returns the best candidate.

- 📈 Auto-Parameter Advisor:

  - Sample-based recommender for skewed numeric and low-cardinality categorical columns, with a metrics table.

- 📊 **Beginner-Friendly Wrappers**:

  - `simple_optimize(df, skew_col, sample_frac, salt_count)`

  - `visualize_salting(df, skew_col, salt_count)`

  - `interactive_optimize(df)`

- 🚨 **Robustness**:
  - Null-tolerant casts, headerless file handling, highly-null column drop, verbose logging.

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

# 2) Auto heavy-hitter salting (auto col + buckets)
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

**Heavy-Hitter Salting**

1. Detect the top skewed column if none given.

2. Compute `salt_count = sparkContext.defaultParallelism` if none given.

3. Aggregate counts by key → identify “heavy” keys with count > total/salt_count.

4. Assign
  - heavy keys → `floor(rand()*salt_count)`
  - others → `pmod(hash(key), salt_count)`

5. Repartition on `"salted_key"` and cache.


**Fast Schema Inference**

- `df.limit(max_sample).collect()` → small list of Rows

- Regex/JSON sniff to pick `IntegerType`, `DoubleType`, `BooleanType`, `TimestampType`, or `StringType`

- Cast with `expr("try_cast(col AS type)")` to avoid crashes

- O(1) Spark jobs, O(max_sample) driver work

**DRTree**

- 1-level: splits on median of top skew column

- Recursive: up to `max_depth`, each branch re-applies skew detector

- Always provides at least one branch `(all → true)`

**Key Detection**

- Single-column: `(distinct–nulls)/total >= threshold`

- Composite: tests top N by distinctness in combinations

- Fallback: returns the best single if none meet threshold

**Auto-Parameter Advisor**

- Samples `sample_frac` of data, up to `max_sample` rows

- In one pass: computes distinct & null counts for all cols

- Computes IQR skew on numeric sample → picks top‐N skewed

- Picks top‐N low-cardinality strings

**🛠️ API Reference**

`**HexaDruid**`

```
| Method | Signature | 
|-------------|-----------|
| Constructor     | `HexaDruid(df, output_dir="hexa_druid_outputs")`    | 
| schemaVisor      | 	`(sample_frac=0.01, max_sample=1000) → (typed_df, schema, dr)` | 
| detect_skew  | `(threshold=0.1, top_n=3) → List[str]` |
| apply_smart_salting  | `(col_name=None, salt_count=None) → DataFrame` |
| detect_keys  | `(threshold=0.99, max_combo=3) → List[str]` |
| build_shard_tree  | `(detector, max_depth=3, min_samples=500) → DRTree` |
```
**Wrappers & Utilities**

```
| Function / Class | 	Description | 
|-------------|-----------|
| `simple_optimize`     | infer_schema + apply_smart_salting in one call    | 
| `visualize_salting`      | 		Print before/after distributions for a given column | 
| `interactive_optimize`  | Advisor → table → prompt → salting |
| `AutoParameterAdvisor`  | `recommend() → (skew_cols, cat_cols, metrics_df)` |
| `DRTree`, `Branch`, `Root`  | 	Build and inspect decision-rule trees for logical sharding |
```

🧪 **Testing**

```bash
pytest tests/
```

🛣️ **Roadmap**

- Multi-format ingestion (Avro, Delta, Iceberg)

-  Streaming support & incremental re-sharding

-  JupyterLab extension for visual DRTree editing

-  REST/gRPC microservice and Kubernetes charts

-  Web UI for interactive parameter tuning and audit logs

🤝 **Contributing**

We welcome PRs, issues, and ideas. Please read [CONTRIBUTING.md] for guidelines.

📄 **License**
MIT © 2025 Omar Hossam Attia