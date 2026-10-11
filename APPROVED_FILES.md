# Files proposed for publication

Nothing has been published. Approve (or amend) this list before deployment. Total size is about 2.4 MB.

| File | Purpose | Notes |
|---|---|---|
| `index.html` | The demo app | Derived from your internal tool: see the ownership check in `README.md` |
| `.nojekyll` | Tells GitHub Pages not to process the site | empty file |
| `README.md`, `TEST_REPORT.md`, `APPROVED_FILES.md` | Documentation | |
| `demo-data/ETR_10.09.2026.xlsx` | Synthetic ETR report (82 rows) | generated |
| `demo-data/FacilityCodes.xlsx` | Synthetic facility mapping (128 rows) | generated |
| `demo-data/Loan_Notice_Documents.xlsx` | Synthetic notices (184 rows, 72 columns) | generated |
| `demo-data/DocID_PDF_Map.csv` | Synthetic document-id to PDF map | relative paths only |
| `demo-data/expected_results.csv` | Expected result per ETR row | |
| `demo-pdfs/DOC-*.pdf` (180 files) | Fictional notices | marked "FICTIONAL" on the page and in the title |
| `tools/generate_demo_data.py` | Deterministic data generator | contains no source data |
| `tests/browser_test.py`, `tests/reference_check.py`, `tests/test_results.json` | Tests and last results | |
| `tests/screenshot-*.png` (5) | Desktop and mobile screenshots of the demo | fictional data only |

**Deliberately excluded (never publish):** the original `ETR_Dashboard_ V2.html`, `ETR 5.29.26.xlsx`, `FacilityCode 1.xlsx`, `Loan Notice Documents ... .xlsx`, `Notices YTD 2026 ... .xlsx`, any internal PDFs or paths, and the private build/regression scripts used to compare against the original.
