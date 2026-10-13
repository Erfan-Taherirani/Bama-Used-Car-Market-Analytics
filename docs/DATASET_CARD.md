![Iran Used-Car Listings — Dataset Card](../figures/dataset-card-banner.png)

# Iran Used-Car Listings — Dataset Card

**Dataset:** Iran Used-Car Listings (Bama.ir snapshot)  
**Project:** Used-Car Market Intelligence & Pricing Analytics  
**Documentation version:** 0.1 (initial, unverified against source files)  
**Card updated:** 2026-10-13  
**Source website:** [Bama.ir](https://bama.ir/)  
**Affiliation:** Independent portfolio project; **not** an official Bama.ir dataset or endorsement.

> **At a glance:** **39,000+ listing records · 18 source features · one collection campaign · individual collection timestamps available.** Counts and field descriptions are provided by the dataset collector and will be verified during the first data-quality audit. This is a **snapshot of advertised listings**, not a dataset of completed vehicle sales.

## 1. Dataset overview

This dataset contains information scraped from car advertisements visible on **Bama.ir**, an Iranian automotive listing website. It records vehicle identification, specifications, reported condition, geographic location, and advertisement-related attributes. The dataset was assembled to support exploratory data analysis (EDA), SQL-based business intelligence, asking-price comparisons, and interactive dashboards for the Iranian used-car listing market.

The collection contains **more than 39,000 observations** and exactly **18 source columns**. Each observation is intended to represent an advertisement encountered during a **single data-collection campaign**. An advertisement should not automatically be treated as a distinct physical vehicle: multiple listings or repeated records may exist. Collection timestamps were retained, but they are **separate collection metadata**, not one of the 18 source fields listed below.

### Summary metadata

| Property | Description |
|---|---|
| Dataset modality | Structured/tabular data with Persian-language categorical and text fields |
| Geographic scope | Vehicle advertisements associated with locations in Iran; actual geographic coverage requires profiling |
| Source | Public-facing vehicle listings on Bama.ir |
| Collection approach | Programmatic web scraping by the project creator |
| Collection design | Single collection campaign / observational snapshot |
| Collection date range | **To confirm from the saved collection timestamps** |
| Dataset size | **39,000+ records (collector-reported; exact count pending audit)** |
| Source feature count | **18** |
| Unit of observation | One collected advertisement record, prior to deduplication |
| Asking-price currency | **Iranian toman**, as reported by the collector |
| Target / label | No verified sale outcome; `price` is an advertised asking price, **not** a final transaction price |
| Official train/test splits | None; the collection was assembled for analytics, not as a published ML benchmark |
| License and redistribution | **Unknown / not established**; do not assume permission to republish scraped records |
| Dataset release status | Source records **not approved for public distribution** pending a rights review |

## 2. Why the dataset was created

The collection supports a portfolio project focused on **used-car market intelligence and pricing analytics**. Its intended analytical questions include:

- Which brands, models, body types, price ranges, and locations are prominent **within the collected listings**?
- How do advertised prices vary with manufacturing year, mileage, transmission, trim, location, and reported body condition?
- What are reasonable descriptive price ranges for closely comparable listed vehicles?
- Which advertisements have asking prices meaningfully above or below a clearly defined comparable-listing benchmark?
- How complete and consistent are marketplace advertisement fields, and which gaps limit analysis?

These are **supply-side and asking-price** questions. Listing counts alone do not establish consumer demand, actual market share, sales frequency, or profitability.

## 3. Source schema and feature dictionary

The table documents **all 18 original features**. Types are **expected analytical types**, not a claim that the raw data is already clean or stored in those types.

### A. Vehicle identity and variant — 4 features

| Feature | Expected type | Definition and interpretation |
|---|---|---|
| `title` | Text | Advertisement headline; usually includes the car name/model and may contain other variant details. Source for later model-name extraction, not necessarily a standardized model identifier. |
| `subtitle` | Text | Listing subtitle, typically combining the vehicle trim/variant and manufacturing year. Can help cross-check information in `trim` and `year`. |
| `trim` | Categorical text | Named vehicle trim, equipment grade, or packaged variant. Differences may be commercially significant even within the same model. |
| `brand` | Categorical text | Manufacturer or make of the advertised vehicle. Requires normalization of spelling and aliases for reliable grouping. |

### B. Age and usage — 2 features

| Feature | Expected type | Definition and interpretation |
|---|---|---|
| `year` | Year / integer after parsing | Vehicle manufacturing/model year as shown on the listing. **Calendar convention must be checked** (e.g., Solar Hijri/Jalali versus Gregorian) before deriving vehicle age. |
| `mileage` | Numeric after parsing | Seller-reported distance traveled, expected to be in **kilometers**; formatting, units, and plausible ranges must be checked. May be missing or imprecise. |

### C. Technical characteristics — 4 features

| Feature | Expected type | Definition and interpretation |
|---|---|---|
| `transmission` | Categorical text | Gearbox category, such as manual or automatic; additional values should be retained and mapped only when verified. |
| `fuel` | Categorical text | Fuel or powertrain type, such as gasoline, hybrid, or electric, subject to the categories present in the source. |
| `body_type` | Categorical text | Vehicle design/class category, e.g., passenger car, sedan, hatchback, crossover, or SUV as represented in the collected records. Categories may overlap and require a defined taxonomy. |
| `cylinder_fa` | Persian text / integer after parsing | Reported number of engine cylinders, potentially expressed using Persian characters or numerals. May be **not applicable** for fully electric vehicles. |

### D. Appearance and reported condition — 3 features

| Feature | Expected type | Definition and interpretation |
|---|---|---|
| `body_color` | Categorical text | Advertised exterior paint color of the vehicle. |
| `inside_color` | Categorical text | Advertised interior/cabin color of the vehicle. |
| `body_status` | Categorical text | Seller-reported body/paint condition, including descriptions such as `بدون رنگ` (no reported repaint), `یک لکه رنگ` (one painted area), `دو لکه رنگ` (two painted areas), `چند لکه رنگ` (multiple painted areas), and `دور رنگ` (repaint described around the vehicle). These descriptions **do not substitute for an independent inspection**. |

### E. Geography — 1 feature

| Feature | Expected type | Definition and interpretation |
|---|---|---|
| `location` | Categorical text | Geographic place associated with the advertisement, such as a city or local area. Its granularity and accuracy require validation before city/province analysis. |

### F. Advertisement details and price — 4 features

| Feature | Expected type | Definition and interpretation |
|---|---|---|
| `image_count` | Integer after parsing | Number of vehicle photographs/images associated with the advertisement. **No image files are included by this field**; it is a count only. |
| `modified_date` | Date/datetime after parsing | Last modification/update date displayed for the listing. **Not necessarily the original publication date**, collection timestamp, or sale date. The calendar and timezone must be verified. |
| `price` | Numeric after parsing | Advertised **asking price in Iranian toman**. May reflect seller preference and does **not** verify a completed sale or final transaction value. Non-numeric or negotiable prices require explicit handling. |
| `link` | URL / text | Link to the source advertisement. Useful for traceability and possible listing-ID extraction, but may later break, change, or expose seller-linked information if republished. |

**Feature count check:** 4 identity + 2 age/usage + 4 technical + 3 appearance/condition + 1 geography + 4 advertisement/pricing = **18 original features**.

## 4. How the data was collected

### Confirmed collection facts

1. The project creator used **web scraping** to collect information from existing vehicle advertisements on **[Bama.ir](https://bama.ir/)**.
2. Collection took place as **one campaign**, producing a cross-sectional dataset rather than repeated daily or weekly market snapshots.
3. For collected records, the scraper extracted the **18 fields** documented above, where they were available on the source listing.
4. **Collection timestamps were retained**, allowing the collection window to be reconstructed and each observation to be associated with the time it was scraped.
5. Each record contains a `link` field pointing to its source advertisement, providing provenance where the page remains accessible.

### Collection workflow

```text
Existing car advertisements on Bama.ir
                 |
                 v
      Single web-scraping campaign
                 |
                 v
  Extract 18 advertisement/vehicle fields
                 +
       Retain collection timestamp
                 |
                 v
          Raw listing records
                 |
                 v
  [Next phase] Profile and validate schema
                 |
                 v
  [Next phase] Clean, standardize, deduplicate
                 |
                 v
  [Next phase] SQL, EDA, and Power BI analysis
```

**Collection parameters still to document:** exact start/end dates and timezone; scraper library and execution environment; pages or filters covered; pagination and retry strategy; handling of unavailable fields; requests/errors; and whether any categories or regions were systematically excluded. These details are **not inferred** from the data description and should be filled in from actual scrape scripts/logs. No claim is made that the campaign sampled *all* listings on Bama.ir or that its sampling was random.

**Important temporal distinction:** `modified_date` is an advertisement attribute from the source. The separately saved scrape timestamp (`scraped_at` or equivalent) records when the observation was collected. Neither provides a verified listing creation or sale date.

## 5. Processing and quality assurance status

**Status at card creation: source collection completed; full data profiling, normalization, and deduplication have not yet been independently verified.** The following are planned validation steps, **not claims about processing already performed**:

| Quality dimension | Planned checks |
|---|---|
| Structural integrity | Confirm exact row count, all 18 columns, parsing/encoding, and collection timestamp availability |
| Uniqueness | Check duplicate URLs/listing IDs; separately assess likely cross-listing duplicates when feasible |
| Completeness | Report missing or placeholder values by field and by vehicle segment |
| Numeric validity | Parse Persian/Arabic digits, number separators, mileage, asking price, and image count; flag invalid values |
| Calendar correctness | Inspect Jalali/Gregorian formats and timezone assumptions for `year`, `modified_date`, and scrape timestamps |
| Category consistency | Normalize make/model/trim aliases, location names, fuel, transmission, colors, and body condition |
| Outliers | Examine prices and mileage **within comparable model/trim/age segments**, rather than using naive global cutoffs |
| Traceability | Keep raw source values, transformation rules, quality flags, and record-level provenance separately from public deliverables |

Recommended additional **derived** columns include `listing_id`, `model_normalized`, `trim_normalized`, `city`, `province`, `body_condition_group`, `price_toman`, `vehicle_age`, and `quality_flags`. They are **not part of the original 18-feature schema**. The saved scrape timestamp is also collection metadata, not an original feature in the list above.

### Quality statistics to add after profiling

- Exact raw row count: **Pending**
- Distinct listing IDs and duplicate rate: **Pending**
- Missing/invalid asking-price rate: **Pending**
- Valid mileage/year coverage: **Pending**
- Normalized model extraction coverage and audited precision: **Pending**
- Collection start/end timestamps: **Pending**
- Geographic/category coverage: **Pending**

## 6. Intended and appropriate uses

- **Exploratory data analysis:** distributions of observed listings, brands/models, specifications, age, mileage, and asking prices.
- **Business intelligence:** SQL queries, data-quality scorecards, Power BI dashboards, and segment comparisons.
- **Descriptive pricing intelligence:** peer-group median asking prices, dispersion, and asking-price deviations with sample-size checks.
- **Educational and portfolio work:** reproducible cleaning, normalization, documentation, and analytical storytelling, subject to source-data rights.
- **Optional predictive experiments:** estimating **asking prices**, using properly constructed validation splits and clear warnings about selection bias.

### Uses the dataset does **not** support on its own

- Estimating **final transaction prices**, sales revenue, actual sales volume, or dealership profit.
- Inferring **buyer demand**, market-wide share, conversion rate, time on market, or sales velocity from listing counts.
- Claiming historical price appreciation/depreciation based on this **single snapshot**.
- Treating reported damage/paint status, odometer readings, or asking prices as independently verified vehicle facts.
- Inferring that a lower-than-median asking price is inherently a bargain, fraud, or a profitable purchase.
- Re-identifying, profiling, or contacting individual sellers using source links or linked records.

## 7. Known limitations and potential biases

1. **Platform selection:** Bama.ir listings represent advertisements available through one online platform, not a verified random sample of Iran's entire car market.
2. **Snapshot timing:** Supply and asking prices may change; the dataset cannot establish longitudinal trends without repeat collections.
3. **Collection coverage unknown:** Filtering, page availability, rate limits, ranking, geography, and scrape failures may affect which listings were observed.
4. **Self-reported attributes:** Mileage, condition, and specifications may be inaccurate, missing, inconsistently described, or updated after collection.
5. **Advertised versus realized value:** `price` records asking prices rather than accepted purchase prices.
6. **Possible duplicates:** Repeated scrape observations or distinct advertisements for the same physical car may inflate counts until investigated.
7. **Text inconsistency:** Persian spelling variations, mixed digits, multiple naming conventions, and unstructured titles complicate model/trim parsing.
8. **Uneven group sizes:** Sparse models/locations may make averages and comparable-price benchmarks unstable.
9. **Link persistence/privacy:** Source links can expire and may connect to seller-identifiable information on the source website.

Analytical outputs should use descriptions such as **“share of collected listings”** and **“median advertised asking price”**, rather than “market share” or “average sale price.”

## 8. Privacy, rights, and public distribution

**Redistribution authorization has not been established.** The fact that advertisements were publicly viewable does not automatically grant permission to download, republish, sell, or provide the compiled records to others. The dataset creator should review the current platform terms and applicable law and, where needed, obtain authorization before sharing source-derived records or public dashboards.

Until distribution rights are clarified:

- **Do not publish** the original scraped dataset, a row-level cleaned derivative, full listing URLs, or seller-identifying details in the public repository.
- Keep raw and processed listing records in **private, access-controlled storage**; add their directories to `.gitignore` and verify they are absent from Git history.
- Publish code, documentation, schema definitions, and **clearly labeled synthetic demonstration data** instead, where permissible.
- Review charts, tables, screenshots, and aggregate exports before publishing; aggregated analysis is **not automatically exempt** from source terms or other rights.
- Do not label the dataset as **open-source, open-data, or licensed under MIT/CC BY** unless those rights have been established. Any license for original analysis *code* is separate from rights to source *data*.

No separate direct personal-information fields are listed among the 18 columns, but `link`, free-text titles, and associated location information may lead to identifiable sellers. Avoid publication of record-level provenance.

## 9. Maintenance, versioning, and reproducibility

- **Initial data-card version:** `0.1` — documented from the collector's description; not yet audited against source files.
- **Dataset snapshot/version:** Assign a stable snapshot ID after verifying collection metadata and calculate a checksum for the privately stored raw file.
- **Future changes:** Keep raw data immutable. Version scripts, normalized category mappings, data dictionary changes, and derived-data outputs separately.
- **Maintainer/contact:** **To be added by the project owner** before a public repository release.
- **Corrections:** Record corrections and schema changes in a changelog. Do not silently alter historical snapshot definitions.
- **Refresh policy:** No recurring refresh currently claimed or planned as part of this snapshot card. A later scrape is a **new snapshot**, not an overwrite of this one.

## 10. Documentation principles and references

This card is organized around provenance, dataset composition, collection, processing, intended uses, limitations, and stewardship, consistent with established dataset-documentation practices:

1. T. Gebru *et al.*, **“Datasheets for Datasets,”** *Communications of the ACM*, 64(12), 86–92 (2021). https://doi.org/10.1145/3458723
2. Hugging Face, **“Dataset Cards,”** documentation: https://huggingface.co/docs/hub/datasets-cards

**Source attribution:** Vehicle advertisement information originates from [Bama.ir](https://bama.ir/). This independent documentation does not imply any relationship with, approval by, or licensing from Bama.ir.

---

*This card describes a collector-reported dataset, not a completed quality audit. Any figures beyond “39,000+ records” and “18 fields” should be computed from the actual dataset before publication.*
