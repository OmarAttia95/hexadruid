# Changelog

---

## [0.1.8] - 2025-07-09

### Major Enhancements

- **Refactored Smart Schema Detection**:  
  - Now robustly auto-casts string columns to `int`/`double` where >90% of sampled values are numeric.
  - Handles malformed values gracefully—falls back to string or double if too many nulls after casting.
  - Fully tolerant of headerless files (no error or failure on missing headers).

- **Null-Tolerant Type Coercion**:  
  - All numeric inferences are now *null-tolerant* (columns with malformed values are safely cast, no job crash).

- **Improved DRTree Logic**:  
  - Logical sharding works even on datasets with minimal or non-skewed columns.
  - DRTree gracefully falls back to a single logical shard if splits aren't viable.

- **Protected/Obfuscated Core**:  
  - Swapped out PyArmor for Nuitka to compile core logic as a `.pyd` binary module (not shipped as plaintext).
  - Obfuscated `_core.py` is never exposed in the PyPI or GitHub repo.
  - Updated `.gitignore` to strictly block all core binary artifacts and sensitive files.

### API & Usability

- `schemaVisor()`:
  - Now *never* fails on files without headers; can be safely called on any flat file.
  - User headers can be optionally injected.

- **Docs & Packaging**
  - Improved `setup.py` and packaging logic to prevent sensitive files from being published.
  - Updated `.gitignore` to reflect new obfuscation and build pipeline.
  - API Reference and Project Architecture included in documentation.

---

## [0.1.7] - 2025-07-09
### Added
- Public PyPI release
- DRTree logic for shard-wise filtering
- Smart salting with Z-score/IQR support
- Primary/composite key detection
- Auto parameter tuning advisor
- Schema inference with safe coercion

### Changed
- Obfuscated internal `_core.py` logic with pyarmor

### Removed
- Legacy CLI interface (will re-add in future)
