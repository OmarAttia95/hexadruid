# Changelog

## [2.2] – 2025-07-13
### Added
- **Heavy‐Hitter Salting** strategy in `apply_smart_salting`:  
  • Auto-detects the top skew column if none provided  
  • Auto-chooses `salt_count` = Spark default parallelism if none provided  
  • Spreads “heavy” keys (> total/salt_count) randomly; hashes the rest → two-stage shuffle for O(N) performance  
- **Fast Schema Inference** in `schemaVisor`/`infer_schema`:  
  • Single `.limit(max_sample).collect()` driver-side sample (no per-column RDD jobs)  
  • Regex/JSON sniffing on the small sample → under 1 s on 100 K rows  
- **Interactive Optimize** improvements (`interactive_optimize`):  
  • Lets you choose from both numeric *and* categorical candidates  
  • Prompts with combined list of skew+categorical columns  
- **Visualization Wrapper** (`visualize_salting`):  
  • Prints before/after distributions in one call for easy sanity checks  
- **Beginner One-Liner** (`simple_optimize`): infer schema + DRTree + heavy-hitter salting in a single call  
- **AutoParameterAdvisor** sampling overhaul:  
  • Samples only `sample_frac` (default 1%) + `max_sample` rows  
  • Computes distinct & null counts in one `.agg()` → sub-second recommendations  
- **KeyFeatureDetector** fallback logic: always returns the best uniqueness candidate (even if below threshold)

### Changed
- Core `apply_smart_salting` signature simplified to `(col_name=None, salt_count=None)`  
- `schemaVisor` no longer builds a quantile-based DRTree by default—now trivial “all” split for speed  
- Reduced default `sample_frac` in advisors to 1% with `max_sample=1 000`  
- Moved all pandas/Arrow dependencies out in favor of pure Spark collects and regex  

### Fixed
- Removed multi-candidate quantile loops in salting to eliminate minutes-long runs  
- Eliminated multiple full-table scans in `AutoParameterAdvisor` to fix 5+ min delays  
- Corrected `interactive_optimize` to include categorical choices as well as numeric  
- Resolved `NameError: Tuple` by importing missing types from `typing`  
- Ensured `detect_keys` returns a sensible fallback rather than an empty list  

### Notes
- This version is production-ready for SaaS or enterprise packaging—two Spark stages, no multi-pass quantiles, and instant sub-second schema inference.
- Next release will focus on multi-format ingestion (Avro/Delta/Iceberg) and streaming support.

---

## [2.1] – 2025-07-12
### Added
- Full dynamic schema inference with null threshold and optional column protection.  
- Enhanced DRTree to provide concise or detailed shard predicate descriptions.  
- Smart salting with auto parameter tuning based on data distribution and Spark parallelism.  
- Key detection improved to find primary and composite keys with confidence scoring.  
- Adaptive shuffle tuning to optimize Spark partition count based on data size.  
- Interactive advisor class for guided skew and grouping column selection.  
- Detailed profiling output including null fractions, cardinality, and data types.  
- Support for robust null handling and optional auto-drop of highly null columns.  
- Visualization of original vs salted data z-scores saved as PNG files.  
- Logging improvements and time measurement utilities for performance tracking.

### Fixed
- Fixed bug in `schemaVisor` where typed DataFrame was not properly initialized before casting.  
- Fixed DRTree `to_dict` method to accept concise output flag without breaking compatibility.  
- Corrected skew detection to avoid exceptions on malformed or missing numeric columns.  
- Fixed headerless CSV detection logic to correctly drop header rows after renaming.  
- Improved error handling during casting operations to use `try_cast` and avoid job failures.

### Removed
- Deprecated earlier static methods replaced by better instance method implementations.  
- Removed hard-coded default partitions in favor of dynamic tuning.

---

## [1.9] – 2025-07-12
*(legacy binary-protected release)*  
**Changed**  
- Core logic shipped as compiled binary for IP protection.  
- Robust null-tolerant coercion, headerless file handling, smarter column dropping.  

---

## [0.1.8] – 2025-07-09
**Major Enhancements**  
- Refactored schema sniffing for null-tolerant numeric coercion.  
- Improved DRTree fallback on non-skewed data.  
- Core obfuscated via Nuitka; safe for SaaS distribution.  

---

## [0.1.7] – 2025-07-09
- Initial public PyPI release: smart salting, DRTree, key detection, advisor, schema inference.  
