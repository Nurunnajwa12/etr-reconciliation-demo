# ETR Reconciliation Demo

A browser-based tool that reconciles an **ETR report** (expected transactions) against **loan notice documents**, using a **facility-code mapping** to join them. Every ETR row ends up **Ready** (a notice matches on asset code *and* transaction type) or **Pending** (with a specific, human-readable reason). Ready rows can then be marked **Processed** and work can be assigned to team members.

> **All data in this demo is fictional.** Company names, identifiers, amounts, dates and PDF notices were generated from scratch by `tools/generate_demo_data.py`. Nothing is sent to a server: the page runs entirely in the browser and makes no network requests.

The tool was built with AI assistance. This repository demonstrates the business-logic design, validation, error handling and testing around it.

## Try it

1. Open `index.html` (or the hosted URL) and click **Load Demo Data**.
2. Open any row with the **+** button to see *why* it was Ready or Pending (inputs, required notice type, notice types on file, reason).
3. Search, sort and filter; tick a Ready row to mark it Processed (and Undo); download a CSV; open the **Members** tab and auto-assign work.
4. **Reset Demo** clears everything the page stored in your browser.
5. **Upload Files** accepts your own files in the same format. Sample inputs are in `demo-data/`; the page marks uploaded data as "Your uploaded files" so it is never confused with the demo.

## What it checks

| Rule | Outcome |
|---|---|
| Identifier, transaction type and transaction date must be present; the date must be a real calendar date (several formats are normalised to `YYYY-MM-DD`) | otherwise Pending with a specific reason |
| Identifier must map to exactly one facility row that has both a Search Value and a Facility ID | otherwise Pending ("No FacilityCode mapping", "Ambiguous facility mapping", "No Search Value", "Incomplete facility mapping") |
| **An asset-code match alone is never enough.** The ETR transaction type is mapped to a notice type (`Interest Payment`→`Interest Payment`, `Loan - Maturity`→`Rate Set`, `Facility - Commitment Reduction`→`Paydown`, `Fee Payments`/`Facility Fee Payment`→`Fee Payment`) and must match the notice's type exactly | a wrong-type notice gives Pending "Transaction type mismatch ... only has: ..." |
| Matching is exact and case-sensitive on asset code and type (no prefix/fuzzy matching; surrounding whitespace is trimmed) | near-misses stay Pending |
| Notices already flagged *Duplicate* by the source are ignored; if two or more non-duplicate notices remain for the same portfolio the match is ambiguous | Pending "Ambiguous notices ..." / "No usable notice ..." |
| One notice per portfolio across several portfolios is a normal split | Ready, with a per-portfolio breakdown |

## Files

| Path | What |
|---|---|
| `index.html` | The whole app (HTML, CSS, JS, a small built-in `.xlsx` reader, and the demo inputs embedded as base64). Open directly or host statically. |
| `demo-data/ETR_10.09.2026.xlsx`, `FacilityCodes.xlsx`, `Loan_Notice_Documents.xlsx`, `DocID_PDF_Map.csv` | Synthetic input files (same structure the upload flow expects). |
| `demo-data/expected_results.csv` | Expected classification, reason, match type, documents and normalised date for every ETR row, with the purpose of each test case. |
| `demo-pdfs/` | Fictional one-page notices that the document links open. |
| `tools/generate_demo_data.py` | Deterministic generator for everything in `demo-data/` and `demo-pdfs/`. |
| `tests/` | `reference_check.py` (independent Python re-implementation checked against the expected file), `browser_test.py` (end-to-end tests in headless Chromium), `test_results.json`, screenshots. |
| `TEST_REPORT.md` | What was tested, results, bugs/limitations found. |
| `APPROVED_FILES.md` | The exact list of files intended for publication. |

## Input formats (upload flow)

All `.xlsx` (the optional map may be `.csv`). Only these columns are required; extra columns are carried through or ignored.

* **ETR report**: `Identifier`, `Transaction Type`, `Transaction Date` (plus display columns such as `Issuer/Borrower Name`, `Description`). First sheet is used (or a sheet named `ETR`). Reporting date is read from the filename (`ETR_10.09.2026.xlsx` → `10.09.2026`).
* **Facility Code**: `Identifier`, `Search Value`, `Facility ID`. First sheet (or `FacilityCodes`).
* **Loan Notice Documents**: `Asset Code` (any header ending in "asset code" is accepted), `Transaction Type`; also reads `Portfolio`, `Document ID`, `Record Type`, `Review Status`, `Reconciliation Status`, amount and agent columns. Sheet `Export` or the first sheet.
* **DocID to PDF map (optional)**: columns `document_id` and `Path` (or `URL`). In the demo the paths are relative (`demo-pdfs/DOC-<id>.pdf`).

## Run, test, regenerate

No build step or backend. To view locally you can just double-click `index.html`, or serve the folder: `python3 -m http.server 8000` and open http://localhost:8000.

```bash
# regenerate the synthetic data (deterministic; writes demo-data/ and demo-pdfs/)
pip install openpyxl reportlab
python3 tools/generate_demo_data.py .

# independent check of expected_results.csv
pip install pandas
python3 tests/reference_check.py

# end-to-end browser tests (needs Playwright + a Chromium)
pip install playwright && playwright install chromium
python3 tests/browser_test.py
```

`index.html` embeds a copy of the demo files. If you regenerate the data, the embedded copy must be refreshed too (the embedded and `demo-data/` copies are what the "Load Demo Data" and "Upload" paths use respectively).

## Deploy (static hosting)

Everything uses relative URLs, so any static host works, including a project sub-path.

**GitHub Pages**
1. Create a new **public** repository under your personal account (for example `etr-reconciliation-demo`).
2. Upload the contents of this folder (including the hidden `.nojekyll` file) to the repository root.
3. *Settings → Pages → Build and deployment*: source "Deploy from a branch", branch `main`, folder `/ (root)`, Save.
4. After a minute the site is at `https://<your-username>.github.io/<repository-name>/`.
5. Tip: commit with your GitHub `noreply` email address, not a work email.

**Netlify (alternative)**: drag this folder onto https://app.netlify.com/drop.

## Dependencies and licensing

* **No third-party code or services are bundled or loaded.** The `.xlsx` reader (`xlsx-lite`, inlined in `index.html`) is original code; fonts are the visitor's system fonts; there are no CDNs, analytics, cookies or external requests (enforced by a Content-Security-Policy meta tag, `connect-src 'none'`).
* Browser requirement: `DecompressionStream` (Chrome/Edge 80+, Safari 16.4+, Firefox 113+).
* Test tooling only (not shipped to visitors): Python, `openpyxl`, `reportlab`, `pandas`, `playwright` (all permissively licensed).
* **No `LICENSE` file is included yet.** Choose the licence yourself after you have confirmed you are allowed to publish this work (see below).

## Before you publish: ownership check

The tool originated as an internal work tool. This public version contains no company data, names, logos, file paths or URLs, but it still embeds the tool's **design and business rules** (for example the transaction-type mapping and the Scheduled Paydown qualification rule). Confirm with your employer that you may publish it, or generalise those rules, before making the repository public.

## Known limitations

See `TEST_REPORT.md` for the full list. In short: classification is rule-based and exact (by design); ETR rows are not de-duplicated; slash dates are read as US `M/D/YYYY`; `.xls`/`.xlsb` are not supported; uploaded DocID maps containing network paths produce `file://` links that only work on the machine that can see those paths; tests were run in Chromium only.
