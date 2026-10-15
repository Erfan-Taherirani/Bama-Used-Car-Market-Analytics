# Data Profiling & Quality Audit — Execution Checklist

**Project:** Iran Used-Car Market Intelligence & Pricing Analytics  
**Dataset:** Bama.ir used-car listings · **39,000+ collected records** · **18 original features**  
**Collection design:** One scraping campaign, with collection timestamps available  
**Tools:** Python, pandas, Jupyter Notebook  
**Status:** Planning checklist — **no audit results have been measured yet**  
**Companion notebook:** `data_quality_audit.ipynb`

> **Goal:** Assess and document data quality *before* changing the source; apply traceable cleaning rules in a separate pass; rerun the same checks; and report what improved, what was excluded, and what remains uncertain.
>
> **Analytical boundary:** This is a single snapshot of **advertised listings**. `price` is an *asking price*, not a sale price; `modified_date` is a last-update field, not a listing creation/sale date. Listing counts are not transaction volumes or demand.

## 0. Audit contract and workflow

**Recommended two-pass process**

```text
Freeze original file + collection metadata
       ↓
PASS A — Profile and diagnose (read-only)
   Baseline → Uniqueness → Completeness → Validity → Consistency
   → Numeric formats → Temporal consistency → Categorical consistency
   → Outlier screening
       ↓
Prioritized issue register + approved cleaning rules
       ↓
PASS B — Remediate in a separate working copy
   Normalize → Resolve duplicates → Handle missing/invalid values
   → Standardize categories → Flag or investigate outliers
       ↓
Rerun the SAME audit + reconcile all records
       ↓
Quality report + notebook + updated DATASET_CARD.md
```

**Rules of engagement**

- [ ] Store the raw source **unchanged**; never overwrite it or silently edit values in the notebook.
- [ ] Confirm exactly **18 source columns**: `title`, `subtitle`, `trim`, `year`, `mileage`, `location`, `transmission`, `fuel`, `body_color`, `inside_color`, `body_status`, `body_type`, `cylinder_fa`, `image_count`, `modified_date`, `brand`, `price`, `link`. Treat `scraped_at` and any batch ID as **collection metadata**, not part of the original 18.
- [ ] Assign stable internal row IDs (`raw_row_id`) so every original record remains traceable without publicly exposing listing URLs.
- [ ] Capture original filename, file size, SHA-256 checksum, import settings, collection window/timezone, Python/pandas versions, and audit date.
- [ ] Keep **raw**, **audited**, **cleaned**, and **quarantined** records distinct; track why any record is excluded.
- [ ] Record baseline statistics before remediation. During Pass A, temporary *diagnostic* normalized columns are allowed; do **not** change the original fields.
- [ ] Define the comparison unit: **one distinct advertisement per snapshot**; a listing URL does **not** prove a distinct physical car.

**Key artifacts:** `data_quality_audit.ipynb`, `audit_baseline.csv`, `issue_register.csv`, `cleaning_decisions.md`, `audit_after_cleaning.csv`, `record_reconciliation.csv`, `DATA_QUALITY_REPORT.md`; all source-derived row-level files stay private.

---

## Pass A — Baseline profiling and issue discovery

### 1. Dataset inventory (before the quality tests)

- [x] Load with explicit encoding and source format; verify row/column counts (`df.shape`) and expected column names/order. Check duplicate column names, whitespace in headers, and incorrectly split fields.
- [x] Review first/last rows *privately*, `df.info()`, `df.dtypes`, `df.describe(include="all")`, and `df.nunique(dropna=False)`; note objects that should become numeric/datetime.
- [x] Check all-null rows, unexpectedly constant columns, suspicious text encoding, and possible import artifacts.
- [x] Verify `scraped_at` values are available and determine the actual minimum/maximum collection timestamps; record how timestamps link to source rows.
- [x] Save **baseline:** raw row count, 18-column schema check, distinct-count summary, recorded collection window, and import warnings.

### 2. Uniqueness — `link` as the candidate key

- [x] Measure **exact duplicate rows** (`df.duplicated(keep=False)`) separately from **duplicate listing links**.
- [x] Audit `link` for missing/blank values, malformed URLs, inconsistent scheme/host, trailing slashes, fragments, and tracking parameters.
- [ ] Build a **diagnostic canonical URL/listing ID**, removing only demonstrably non-identity-changing fragments/parameters. **Do not** discard unknown query parameters or merge different listing paths without evidence.
- [ ] For **nonmissing** canonical links/IDs, calculate distinct IDs, duplicate groups, affected rows, and **extra duplicate observations** (group size minus one). Exclude missing IDs from duplicate-key counts.
- [ ] Compare duplicate records' `price`, `mileage`, `modified_date`, `title`, and other fields to identify **identical repeats vs. conflicting repeats**.
- [ ] Investigate candidate different-link/same-vehicle matches separately; **do not automatically merge them**.
- [ ] Document the resolution rule for each duplicate category. Retain enough provenance to explain why a representative record was selected; do not automatically prefer the newest `modified_date` without verifying its meaning.

**Report:** exact duplicate rows; missing/invalid links; distinct valid listing IDs; duplicate-link groups; extra observations; number of conflicting groups; unresolved identity cases.

### 3. Completeness — missing values and missingness patterns

- [ ] Run `df.isna().sum()` and percentage missing per column; record the numerator and denominator.
- [ ] Separately profile **blank strings, whitespace, `-`, `نامشخص`, `ناموجود`, `None`/`null` strings**, and other observed placeholders. pandas `isna()` alone does **not** count empty strings as missing.
- [ ] Distinguish **true missing**, **not applicable**, **unknown**, and **not parseable**. For `price`, distinguish a missing/negotiable price from a numeric asking price; never interpret zero as a free car without verification.
- [ ] Measure missingness of business-critical fields (`link`, `brand`, `title`, `year`, `mileage`, `price`, `body_status`, `location`) and combinations such as *price missing + model missing*.
- [ ] Compare missingness across large brands, title groups, locations, and other credible segments to identify systematic patterns; do not assume missingness is random.
- [ ] Document which analyses require each field rather than choosing a single blanket drop/imputation rule.

**Report:** per-field missing count/rate, placeholder counts, joint missingness patterns, usable-record counts for key analyses.

### 4. Validity — schema, domain, and cross-field checks

- [ ] Verify expected broad types: text (`title`, `subtitle`, `trim`, `location`, `brand`, etc.), URL (`link`), numeric-compatible (`mileage`, `price`, `image_count`, `cylinder_fa`), and calendar-aware date/year (`modified_date`, `year`).
- [ ] Check `link` is a plausible listing URL and, where expected, belongs to the documented source domain. A syntactically valid URL need not still resolve.
- [ ] Identify negative `price`/`mileage`/`image_count`, fractional image counts, unexpected cylinder values, implausible manufacturing years, and unusual zero values. **Flag** questionable cases; do not delete them automatically.
- [ ] Validate manufacturing year against the **correct calendar** and collection date. A model-year designation can occasionally differ from manufacture/registration year; investigate before exclusion.
- [ ] Check logical combinations, e.g., source `subtitle` year vs. `year`, `brand` vs. `title`, and electric fuel vs. cylinder count (where cylinders may be not applicable).
- [ ] Record all tests that need parsed numeric/date *diagnostic columns*; re-evaluate them after Steps 6–7 to avoid falsely marking Persian-formatted values invalid.

**Report:** rules tested, count/rate failing each rule, severity, and verified exceptions.

### 5. General consistency — conflicting information

- [ ] Identify inconsistent spelling, casing, whitespace, Persian/Arabic characters (`ی/ي`, `ک/ك`), punctuation, digit scripts, and unit suffixes across records.
- [ ] Examine mismatches among `title`, `subtitle`, `brand`, `trim`, and `year`; note records requiring manual review before automated model extraction.
- [ ] Check comparable fields for contradictory representations of condition, fuel, transmission, or body type.
- [ ] Inspect records with identical canonical listing IDs but inconsistent descriptive fields; keep conflict counts separate from duplicate counts.
- [ ] Document standardized mappings and **unmapped** values; never guess unknown model/trim assignments.

**Report:** inconsistent patterns, affected fields/rows, examples with sensitive identifiers redacted, and proposed mapping rules.

### 6. Numeric format audit — parsing reliability

- [ ] Inventory actual raw patterns for `price`, `mileage`, `image_count`, `cylinder_fa`, and numeric `year`: Persian/Arabic/Latin digits, commas (`٬`, `,`), decimals, spaces, text labels, units, and abbreviations.
- [ ] Confirm that `price` is denominated in **toman** in the collected representation; do not silently mix rial/toman or million/billion units.
- [ ] Develop **diagnostic** standardized strings and numeric parses; keep originals untouched. Use `pd.to_numeric(..., errors="coerce")` after documented normalization.
- [ ] Count **conversion failures among previously populated values** separately from truly missing values; explicitly record unsupported formats.
- [ ] Check nonnegative/integer constraints after parsing and assess whether large numbers retain sufficient precision.
- [ ] Measure valid parse coverage per numeric field; record representative failed patterns for targeted fixes.

**Report:** numeric parse success, failure reasons, zero/negative cases, currency/unit assumptions, and source-vs-parsed examples.

### 7. Temporal consistency — year, modified date, collection time

- [ ] Detect Jalali/Solar Hijri vs. Gregorian formats in `year` and `modified_date`; do **not** blindly parse Jalali dates with `pd.to_datetime` as Gregorian.
- [ ] Document date format(s), timezone of source strings and `scraped_at`, conversion method, and any ambiguity in date-only or relative labels.
- [ ] Check invalid dates, unparseable values, and dates in the future **relative to actual collection time**, allowing for documented timezone/clock differences.
- [ ] Where meaningful, check `modified_date <= scraped_at` with an explicit tolerance/exception policy; verify whether `modified_date` can be date-only.
- [ ] Derive vehicle age only after normalizing manufacturing/model year to a consistent calendar; flag inconsistent or negative derived ages for review.
- [ ] Record the actual collection window. **Do not infer** listing age, time-on-market, transaction date, or time-series price trends from `modified_date`.

**Report:** date parse rates, calendar distribution, chronological violations, collection range, unresolved format/timezone questions.

### 8. Categorical consistency — controlled vocabularies

- [ ] Produce raw frequency tables (`value_counts(dropna=False)`) for `brand`, `trim`, `transmission`, `fuel`, `body_color`, `inside_color`, `body_status`, `body_type`, `location`, and common `title` patterns.
- [ ] Create version-controlled alias mappings; normalize Persian spelling before grouping, but preserve original values alongside standardized versions.
- [ ] Separate `brand`, **extracted model**, and `trim`: model extraction from `title` is a later, explicit transformation, not an original feature.
- [ ] Review `body_status` carefully: `بدون رنگ`, `یک لکه رنگ`, `دو لکه رنگ`, `چند لکه رنگ`, and `دور رنگ` have different meanings; structural damage must not be collapsed into cosmetic repaint categories.
- [ ] Verify city/province extraction from `location` only when supported; keep unknown or ambiguous geographic labels unresolved.
- [ ] Quantify **mapping coverage**, residual `Other`/`Unknown`, rare categories, and collisions where different original values were merged.
- [ ] Manually review a stratified sample of extracted vehicle models, including less common brands/titles; report classification precision and extraction coverage separately.

**Report:** raw vs. standardized distinct counts, mapping coverage, unmapped values, and model-extraction validation results (if implemented).

### 9. Outlier screening — detection, not automatic deletion

- [ ] Examine numeric distributions: `price`, `mileage`, `image_count`, `year`, and derived age where valid. Report min/median/p95/p99/max, histograms, and box plots as appropriate.
- [ ] Use **domain checks** first (negative mileage, impossible year, malformed amount), then statistical flags: **IQR fences** and **median absolute deviation (MAD)** for suitable fields; note special handling if MAD is zero.
- [ ] Compare pricing **within sufficiently populated, comparable vehicle segments** (brand/model/trim/year band/mileage/condition), not only across the whole market. Use explicit sample-size requirements; mark sparse peer groups as inconclusive.
- [ ] Inspect price–mileage and price–year scatterplots by major models; examine the most extreme cases manually, using source records privately.
- [ ] Classify each flagged observation as *confirmed error*, *plausible extreme*, or *unresolved*. A luxury vehicle is not erroneous solely for being expensive.
- [ ] Do **not** winsorize, cap, replace, or drop outliers in Pass A; propose treatment only after reviewing their likely causes and downstream impact.

**Report:** number flagged by method/field, overlap among methods, sample sizes, reviewed outcomes, and whether any proposed exclusion is defensible.

---

## Pass B — Remediation, retesting, and documentation

### 10. Approve a cleaning policy **before** applying changes

- [ ] Give every issue a unique ID, severity (`critical` / `major` / `minor`), affected fields, and measured impact.
- [ ] Choose an explicit action: `normalize`, `parse`, `standardize`, `deduplicate`, `flag`, `set missing`, `quarantine`, `exclude from a specific metric`, or `leave unchanged`.
- [ ] Explain why; keep unchanged originals and transformation code. Do not impute asking prices, reported body condition, or ambiguous trims as though verified.
- [ ] Use deterministic duplicate-resolution criteria, independent of row order where possible; log conflicting values and retained record IDs.
- [ ] Keep records with valid data for analyses they still support; avoid global deletion because a nonessential field is missing.
- [ ] Preserve `raw_row_id`, original link privately, original numeric/text values, parsed values, and relevant quality flags.

**Issue register template** — populate one row per issue/rule:

| Issue ID | Dimension | Rule/field | Baseline affected (n, %) | Severity | Evidence | Decision / rationale | After affected (n, %) | Status |
|---|---|---|---|---|---|---|---|---|
| DQ-001 | Uniqueness | Duplicate canonical `link` | Pending | Pending | Pending | Pending | Pending | Open |
| DQ-002 | Completeness | Missing or negotiable `price` | Pending | Pending | Pending | Pending | Pending | Open |

### 11. Apply changes in a separate, reproducible pipeline

- [ ] Create `df_clean = df_raw.copy()` (or an equivalent staged transform). Avoid hidden manual edits and blanket `dropna()`/`drop_duplicates()` calls.
- [ ] Normalize known text/number/date patterns; write alias tables, parsing exceptions, and source-to-output mappings.
- [ ] Resolve confirmed duplicate observations **after** conflict investigation; preserve a private exclusion/reconciliation log.
- [ ] For missing or invalid fields, choose field-specific treatment; keep analysis eligibility indicators (e.g., `has_valid_price`, `has_valid_mileage`, `model_parse_status`).
- [ ] Store outlier **flags** by default. Exclude only confirmed errors or clearly documented analysis-specific cases.
- [ ] Produce a cleaned dataset and an audit trail without overwriting the raw snapshot.

### 12. Rerun the **same** checks; compare before and after

- [ ] Recalculate all nine quality sections using consistent definitions and appropriate denominators.
- [ ] Reconcile record counts using **mutually exclusive** disposition categories: `raw rows = retained clean rows + excluded duplicate observations + quarantined/excluded rows` (if those are the only dispositions). Explain anything not covered.
- [ ] Verify uniqueness of retained listing IDs, expected data types, valid numeric/date conversions, domain rules, accepted category mappings, and absence of unexplained row loss.
- [ ] Report **baseline → after → delta** for duplicate extra rows, missingness, parse failures, invalid values, unresolved categories, suspicious outliers, and key analysis eligibility.
- [ ] Note that dropping incomplete records may mechanically reduce missing-value rates; show **row retention and usable-record counts** alongside percentages.
- [ ] List unresolved issues, known biases, potential errors introduced by cleaning, and tests needed before using specific KPIs.

**Before/after scorecard template:**

| Metric | Before | After | Interpretation |
|---|---:|---:|---|
| Raw / retained rows | Pending | Pending | Show disposition and retention |
| Extra duplicate observations (`link`) | Pending | Pending | Valid canonical IDs only |
| Missing/negotiable asking price | Pending | Pending | Keep statuses separate |
| Numeric parsing failures | Pending | Pending | Count nonmissing but unparseable |
| Invalid year/mileage/image count | Pending | Pending | Rule-specific failures |
| Unparseable `modified_date` | Pending | Pending | Calendar-aware check |
| Unmapped brand/model/condition | Pending | Pending | Include coverage and sample-size notes |
| Price outlier flags | Pending | Pending | Flags are **not** automatically defects |
| Records eligible for pricing analysis | Pending | Pending | Denominator and rules documented |

### 13. Write the professional audit report and update the dataset card

- [ ] Prepare `DATA_QUALITY_REPORT.md` with **Executive summary → Dataset/scope → Method and rules → Findings by dimension → Remediation and effects → Residual risks → Fitness for intended use → Reproducibility**.
- [ ] For each significant problem, show **what failed, how many records were affected, the decision, before/after result, and remaining limitation**. Prefer compact tables and a few high-value charts to lengthy notebook screenshots.
- [ ] Add exact verified counts and collection window to `DATASET_CARD.md` **§1**; verified scraping metadata to **§4**; audit figures and actions to **§5**; observed biases to **§7**; audit date/version and reproducibility details to **§9**.
- [ ] Retain the distinction between **18 original features** and derived columns/collection metadata. Never describe an audit or model parser as completed unless the notebook verifies it.
- [ ] Review public outputs for licensing/privacy restrictions; until source redistribution rights are verified, publish **code, rules, documentation, and clearly labeled synthetic examples**, not source-derived row-level records or unreviewed public aggregates.

---

## Minimum notebook structure (`data_quality_audit.ipynb`)

1. **Scope and imports:** paths, pandas version, frozen raw-file checksum, original 18-column schema, timestamp provenance.
2. **Baseline:** `shape`, `info`, `dtypes`, descriptive statistics, field cardinalities, collection window.
3. **Diagnostics:** sections 2–9 above in the same order, each with *purpose → rule → measure → examples → finding → issue ID*.
4. **Issue register:** prioritized summary of problems identified **without modifying raw data**.
5. **Remediation:** explicit transformation functions and decision logs operating on a working copy.
6. **Post-clean audit:** rerun diagnostics, produce the before/after scorecard and row reconciliation.
7. **Conclusions:** reliability for market EDA and price benchmarking, exclusions, unresolved risks, recommended next steps.

### Small pandas starter reference

```python
import pandas as pd

# df_raw: read once; preserve independently of cleaned/diagnostic copies.
print(df_raw.shape)
df_raw.info(memory_usage="deep")

# Exact-row duplicate participation (both copies count as affected).
exact_duplicate_rows = df_raw.duplicated(keep=False).sum()

# Raw-link diagnostic only; also inspect URL variants before deduplication.
link = df_raw["link"].astype("string").str.strip()
nonblank = link.notna() & link.ne("")
raw_link_duplicate_rows = (
    link[nonblank].duplicated(keep=False).sum()
)

# Built-in missingness; report placeholders separately.
missing = pd.DataFrame({
    "missing_n": df_raw.isna().sum(),
    "missing_pct": 100 * df_raw.isna().mean(),
})

# Example numeric *diagnostic* parse: normalize scripts/units first.
# Preserve source; compare failed parses only where raw value was populated.
# parsed_price = pd.to_numeric(normalized_price, errors="coerce")
# parse_failed = normalized_price.notna() & parsed_price.isna()
```

**Notes:** `duplicated(keep=False)` counts **every row involved in a duplicate group**, which is not the same as the number of **extra rows to remove**. pandas `isna()` does not count empty strings as missing. `to_numeric(..., errors="coerce")` produces missing results for failed conversions, so report those *separately* from pre-existing missingness. `pd.to_datetime` is appropriate for verified Gregorian formats; use an explicit Jalali conversion when necessary.

## Completion criteria

- [ ] All **18 original fields** and collection metadata have baseline quality measurements.
- [ ] Every consequential problem has an issue ID, severity, evidence, and documented decision.
- [ ] No original records were silently modified; all removals/quarantines reconcile with the input row count.
- [ ] The same tests ran before and after cleaning and their results are comparable.
- [ ] Eligibility and limitations for EDA, pricing, and dashboard KPIs are explicit.
- [ ] Reports and the dataset card distinguish **confirmed findings** from **pending/unverified facts**.

**Next handoff:** Once `data_quality_audit.ipynb` (and, if needed, the generated summary tables) is provided, use its *actual measured results* to update `DATASET_CARD.md` and write the concise, evidence-based `DATA_QUALITY_REPORT.md`. **Do not invent statistics in advance.**

### References

- [pandas — `DataFrame.duplicated`](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.duplicated.html)
- [pandas — `DataFrame.isna`](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.isna.html)
- [pandas — `to_numeric`](https://pandas.pydata.org/docs/reference/api/pandas.to_numeric.html)
- [pandas — `to_datetime`](https://pandas.pydata.org/docs/reference/api/pandas.to_datetime.html)
