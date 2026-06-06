# Detector Verification Report

This document verifies the implementation of the SEO detectors against the sample export dataset and the provided rulebook.

## Summary Table

| Detector | Severity | Count | Logic | Example URL | CSV Match? |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `missing_title` | High | 0 | `idx200` and no title | N/A | N/A |
| `duplicate_title` | High | 12 | `idx200` sharing identical title | `https://nmgtechnologies.com/` | Yes (12) |
| `title_too_long` | Medium | 63 | `idx200` with length > 60 or width > 561px | `https://nmgtechnologies.com/` | Close (62) |
| `broken_link` | High | 6 | Any URL with status 4xx | `https://nmgtechnologies.com/hs-fs/hubfs/B-W_icon.png.png...` | Yes (6) |
| `server_error` | High | 0 | Any URL with status 5xx | N/A | Yes (0) |
| `redirect` | Medium | 7 | Any URL with status 3xx | `https://nmgtechnologies.com/company/about-us/` | Yes (7) |
| `orphan_page` | Medium | 0 | `idx200` with 0 inlinks | N/A | N/A |
| `title_too_short` | Low | 21 | `idx200` with length < 30 and not empty | `https://nmgtechnologies.com/blog` | Yes (21) |
| `missing_meta_description` | Medium | 0 | `idx200` and no meta description | N/A | Yes (0) |
| `duplicate_meta_description` | Medium | 16 | `idx200` sharing identical meta description | `https://nmgtechnologies.com/blog` | Yes (16) |
| `meta_description_too_long` | Low | 42 | `idx200` with length > 155 | `https://nmgtechnologies.com/` | Yes (42) |
| `missing_h1` | Medium | 2 | `200 html` and no H1 | `https://nmgtechnologies.com/company/process/` | Yes (2) |
| `duplicate_h1` | Low | 19 | `idx200` sharing identical H1 | `https://nmgtechnologies.com/blog` | Close (18) |
| `redirect_chain` | High | 0 | 3xx URL where target is also 3xx | N/A | Yes (0) |
| `thin_content` | Low | 10 | `idx200` with word count < 200 | `https://nmgtechnologies.com/blog/tag/events/page/2` | Yes (10) |
| `non_indexable_but_linked` | Medium | 2 | `200 html`, not indexable, inlinks > 0 | `https://nmgtechnologies.com/industry/edtech` | Yes (2) |
| `slow_page` | Low | 11 | `200 html` with response time > 1.0s | `https://nmgtechnologies.com/blog/best-practices...` | N/A |

**Note:** CSV Match is based on `sample-export/issues_reports/issues_overview_report.csv`.

## Schema Validation Result

**File:** `outputs/report.json`  
**Schema:** `report.schema.json`  
**Status:** ✅ VALID

No missing fields, wrong types, or schema violations were found.
