# Detector Analysis: seo/detector.py

## 1. Currently Implemented Detectors
The current implementation includes a "starter set" of 7 deterministic detectors:

| Issue Type | Logic | Severity |
| :--- | :--- | :--- |
| `missing_title` | `Title 1` is empty on indexable 200 pages | High |
| `duplicate_title` | 2+ indexable URLs sharing the same `Title 1` | High |
| `title_too_long` | `Title 1 Pixel Width` > 561 OR `Title 1 Length` > 60 | Medium |
| `broken_link` | `Status Code` between 400 and 499 | High |
| `server_error` | `Status Code` between 500 and 599 | High |
| `redirect` | `Status Code` between 300 and 399 | Medium |
| `orphan_page` | `Inlinks` = 0 on indexable 200 pages | Medium |

## 2. Available Helper Functions
The detector uses several utilities to handle the messy nature of CSV data:

- **Data Loading**:
    - `load_rows(export_dir)`: Loads `internal_all.csv` using `csv.DictReader`.
- **Type Casting**:
    - `_int(v, default=0)`: Safely converts strings to integers (handles floats and whitespace).
    - `_float(v, default=0.0)`: Safely converts strings to floats.
- **Filtering**:
    - `is_html(r)`: Checks if the `Content Type` contains `text/html`.
    - `is_200(r)`: Validates if the `Status Code` is exactly 200.
    - `indexable(r)`: Validates if the `Indexability` column is exactly "Indexable".
- **Aggregation**:
    - `summarize(issues)`: Counts total issues and breaks them down by severity (`High`, `Medium`, `Low`).

## 3. Missing Detectors (from `rulebook.md`)
The following rules are defined in the rulebook but not yet implemented in `detector.py`:

- **Titles/Metas**: `title_too_short`, `missing_meta_description`, `duplicate_meta_description`, `meta_description_too_long`.
- **Headers**: `missing_h1`, `duplicate_h1`.
- **Links/Redirects**: `redirect_chain`.
- **Content/Performance**: `thin_content`, `non_indexable_but_linked`, `slow_page`.

## 4. Current Detection Limitations
- **No Relational Analysis**: The current logic processes rows independently or via simple grouping. It cannot detect `redirect_chain` because that requires building a map of redirects and tracing the path.
- **Partial Rulebook Coverage**: Only ~40% of the required rules are implemented.
- **Basic Filtering**: While it has `idx200` and `html` filters, it does not yet implement more complex combined filters for all rule types (e.g., some rules might apply to all 200s, others only to indexable ones).

## 5. Sample Export Reporting
If the sample export only reports 4 issue types despite 7 being implemented, it is because the **sample data specifically only contains those 4 types of errors**. The `detector.py` only adds an issue to the list if `urls` is non-empty (Line 50). Therefore, the output is a direct reflection of the flaws present in the `internal_all.csv` provided in the sample.

## Recommended Implementation Order

I recommend implementing the missing detectors in three waves to manage complexity:

### Wave 1: Simple Column Checks (Quick Wins)
These only require a single `if` statement over a column for the `idx200` or `html` sets.
1. `title_too_short`
2. `missing_meta_description`
3. `meta_description_too_long`
4. `missing_h1`
5. `thin_content`
6. `slow_page`
7. `non_indexable_but_linked`

### Wave 2: Grouping/Deduplication
These require using `defaultdict` to find duplicates, similar to the existing `duplicate_title` logic.
1. `duplicate_meta_description`
2. `duplicate_h1`

### Wave 3: Graph/Map Logic (High Complexity)
This requires building a state map of the entire crawl before detection.
1. `redirect_chain` (Build `{Address: RedirectURL}` map $\rightarrow$ trace targets)
