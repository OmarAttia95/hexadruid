# HexaDruid 🧠⚡

[![PyPI version](https://badge.fury.io/py/hexadruid.svg)](https://badge.fury.io/py/hexadruid)  
[![Python Version](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/)  
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**HexaDruid** is a Spark-native optimizer that tackles **data skew**, **ambiguous key detection**, and **schema bloat** using advanced salting, rule-based decision trees (DRTree), and adaptive partition tuning.  
It operates in **two Spark stages**, with no Pandas, no UDFs, and no external engines.

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
  Auto-detects the hottest keys and spreads them randomly. Hashes remaining keys. Two-stage shuffle with O(N) performance.

- ⚙️ **Dynamic API**  
  ```python
  hd.apply_smart_salting(col_name=None, salt_count=None)
  ```
  Auto-selects column and salt count if omitted.

- 🧠 **Fast Schema Inference**  
  Single `.limit().collect()` pass + safe `try_cast()` logic. Sub-second on 100K+ rows.

- 🌲 **DRTree Sharding**  
  Recursive decision rule trees for logical partitioning and filtering. Always yields a fallback `all → true` branch.

- 🔑 **Key Detection**  
  Detects both primary and composite keys using uniqueness ratios.

- 📈 **Auto-Parameter Advisor**  
  Recommends skewed numeric and low-cardinality string columns using IQR and cardinality metrics.

- 📊 **Beginner-Friendly Wrappers**  
  - `simple_optimize(df, skew_col, sample_frac, salt_count)`  
  - `visualize_salting(df, skew_col, salt_count)`  
  - `interactive_optimize(df)`

- 🚨 **Robustness**  
  Handles nulls, malformed types, no-header files, corrupt values, all-null columns, and more.

---

## 🧠 Quickstart

```python
from pyspark.sql import SparkSession
from hexadruid import HexaDruid, simple_optimize, visualize_salting

spark = SparkSession.builder.getOrCreate()
spark.conf.set("spark.sql.shuffle.partitions", "8")  # match salt count

df = spark.read.csv("data.csv", header=True, inferSchema=False)

# 1) Fast schema + DRTree
typed_df, schema, dr_tree = HexaDruid(df).schemaVisor()
print("Schema:", schema.simpleString())

# 2) Auto heavy-hitter salting
df_salted = HexaDruid(df).apply_smart_salting()
df_salted.groupBy("salt").count().show()

# 3) One-liner optimize
df_opt = simple_optimize(df, skew_col="user_id", salt_count=5)

# 4) Visualize skew impact
visualize_salting(df, skew_col="user_id", salt_count=5)

# 5) Interactive optimization with smart prompts
df_inter = HexaDruid(df).interactive_optimize(df)
```

---

## 📚 What’s Under the Hood?

### Heavy-Hitter Salting

1. If `col_name` not provided, detect skewed column.  
2. If `salt_count` is None, use `sc.defaultParallelism`.  
3. Group and count → identify heavy keys (count > total/salt_count).  
4. Assign:
   - Heavy keys → `floor(rand() * salt_count)`
   - Light keys → `pmod(hash(col), salt_count)`
5. Repartition and cache.

---

### Fast Schema Inference

- Uses `.limit()` to collect `max_sample` records  
- Applies regex, JSON, numeric sniffing  
- Avoids type errors via `try_cast()`  
- Falls back to `StringType` for undecidable fields

---

### DRTree Logic

- Root splits on most skewed column  
- Recursive decision tree branching  
- Every node stores a Spark SQL predicate  
- Tree always includes a fallback: `all → true`

---

### DRTree ASCII Diagram

```text
                      [Root Node: amount]
                             |
                 ┌──────────┴──────────┐
                 |                     |
     [DecisionNode: amount ≤ 500]   [DecisionNode: amount > 500]
                 |                     |
           ┌─────┴─────┐         ┌─────┴─────┐
           |           |         |           |
 [LeafNode: ≤100] [LeafNode: 101–500] [LeafNode: 501–1000] [LeafNode: >1000]

Legend:
- Root Node → First entry node
- DecisionNode → Conditional split (e.g. amount ≤ 500)
- LeafNode → Logical partition / shard for query pushdown
```

---

### Key Detection

- Single column: `(distinct - nulls) / total ≥ threshold`  
- Composite key: top N combos tested via distinctiveness  
- Fallback to best-available candidate

---

### Auto-Parameter Advisor

- Samples `sample_frac` up to `max_sample` rows  
- Detects:
  - Highly skewed numeric columns (via IQR)
  - Low-cardinality string columns  
- Returns a clean `metrics_df` and top-N candidates

---

## 🛠️ API Reference

### Core Methods

| Method               | Signature                                                                 |
|----------------------|---------------------------------------------------------------------------|
| Constructor          | `HexaDruid(df, output_dir="hexa_druid_outputs")`                          |
| schemaVisor()        | `(sample_frac=0.01, max_sample=1000) → (typed_df, schema, dr)`            |
| detect_skew()        | `(threshold=0.1, top_n=3) → List[str]`                                    |
| apply_smart_salting()| `(col_name=None, salt_count=None) → DataFrame`                            |
| detect_keys()        | `(threshold=0.99, max_combo=3) → List[str]`                               |
| build_shard_tree()   | `(detector, max_depth=3, min_samples=500) → DRTree`                       |

---

### Wrappers & Utilities

| Function / Class         | Description                                                           |
|--------------------------|-----------------------------------------------------------------------|
| `simple_optimize()`      | Infer schema + apply smart salting in one line                        |
| `visualize_salting()`    | Show before/after z-score distribution plots                          |
| `interactive_optimize()` | Advisor → table → prompt → salting                                   |
| `AutoParameterAdvisor`   | `.recommend() → (skew_cols, cat_cols, metrics_df)`                    |
| `DRTree`, `Branch`, `Root` | Build & introspect logical trees for partition sharding             |

---

## 🧪 Testing

```bash
pytest tests/
```

---

## 🛣️ Roadmap

- [ ] Multi-format support: Avro, Delta, Iceberg  
- [ ] Streaming + incremental DRTree updates  
- [ ] JupyterLab plugin for tree visual editing  
- [ ] REST/gRPC microservice + Kubernetes helm charts  
- [ ] Web UI for audit trails & optimization logs  

---

## 🤝 Contributing

Pull requests, feature ideas, and bug reports are welcome.  
Please see [`CONTRIBUTING.md`](CONTRIBUTING.md) for guidelines.

---

## 📄 License

MIT © 2025 Omar Hossam Attia  
Current Version: **v0.2.2**
