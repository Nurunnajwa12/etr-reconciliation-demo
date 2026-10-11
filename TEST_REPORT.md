# Test report

Run: 2026-10-11 · Chromium (headless, Playwright) served from localhost, plus a `file://` smoke test · 81/81 automated checks passed.

## How it was tested

1. **Independent reference check** (`tests/reference_check.py`): the rules were re-implemented separately in Python (pandas), run on the generated Excel files and compared with `demo-data/expected_results.csv` → **82 rows, 0 mismatches**. This guards the expected file itself.
2. **End-to-end browser tests** (`tests/browser_test.py`): the real page, loading the demo and uploading the sample files through the file inputs. Every one of the 82 ETR rows is compared with `expected_results.csv` for tab, exact reason text, notice match type, matched document IDs (per portfolio) and normalised date.
3. **Original-vs-new comparison** (local, not shipped): the original engine and this engine were run on the same data. On the synthetic set, the original accepted 52 rows as Ready; this version accepts 36. All 16 differences are deliberate and listed below. Every false-positive-trap row (asset code matches, transaction type does not) was already Pending in the original: the core rule was correct before.
4. **Leak scan**: all demo values were compared with the original source files and the HTML was searched for company/vendor names, URLs and paths (see "Confidentiality").

## Test data design (82 ETR rows: 36 Ready, 46 Pending)

| Case id | Rows | What it exercises |
|---|---|---|
| RDY-INT / RDY-RATE / RDY-PAYDOWN(-SCHED) / RDY-FEE / RDY-FEE2 | 14 | Each transaction-type mapping, including scheduled-paydown labelling |
| RDY-MULTI | 5 | Legitimate multi-portfolio split (2-3 portfolios) |
| RDY-STATUS | 3 | Valid matches whose notice carries Reconciled / Undetermined / Created Manually statuses (shown as tags; do not gate matching) |
| RDY-NODOC, RDY-BLANKPATH | 3 | Valid match but the Document ID is missing from the PDF map or has a blank path |
| RDY-DATEFMT | 3 | Dates stored as text in three formats, normalised |
| RDY-DUPETR | 2 | Identical ETR line twice (not de-duplicated; both Ready) |
| RDY-WHITESPACE, RDY-MIXED, RDY-DUPFLAG | 6 | Whitespace trimming; several notice types on one asset; Duplicate-flagged notices ignored |
| PEND-NONOTICE | 6 | Valid mapping, no notice for the asset |
| PEND-TYPE | 10 | **False-positive trap**: asset code matches, transaction type does not (including a case-different type) |
| PEND-DUP, PEND-ALLDUP | 7 | Ambiguous notices (two documents / exact duplicate row / duplicate in one of two portfolios); only Duplicate-flagged notices |
| PEND-NOFC, PEND-FCCONFLICT, PEND-BLANKSV, PEND-BLANKFID | 9 | Missing, conflicting, blank-Search-Value and blank-Facility-ID mappings |
| PEND-NEARMISS | 3 | Asset code with suffix / different case / one character different |
| PEND-UNMAPPED | 2 | Transaction types with no mapping entry |
| PEND-IDBLANK, PEND-TYPEBLANK, PEND-DATEBLANK, PEND-DATEBAD | 9 | Missing identifier / type / date; invalid dates (`TBD`, `2026-02-30`, `31/13/2026`) |

`demo-data/expected_results.csv` is authoritative: one line per row, with the purpose of each.

## Deliberate differences from the original engine

These make the demo stricter than the original tool. They are the cases where the original accepted a row as Ready and this version holds it for review:

* Duplicate/conflicting notices for one portfolio (original: silently used the first in the file) → Pending "Ambiguous notices".
* Only Duplicate-flagged notices (original: Ready) → Pending "No usable notice".
* Invalid or missing transaction date (original: not checked) → Pending.
* Facility row with blank Facility ID (original: Ready if a notice matched) → Pending "Incomplete facility mapping".
* Identifier listed twice with different Search Values (original: last row silently won) → Pending "Ambiguous facility mapping".
* The ETR file must now contain a `Transaction Date` column; fully blank ETR rows are skipped.
* Header names are trimmed of stray spaces (the original code noted this could leave the Facility Code column blank on real exports).
* The unused Scheduled-Paydown *fallback* was removed: the original described it in comments and built an index for it, but the matching function never used it. (The Scheduled Paydown *label* on exact paydown matches is kept.)

Reasons are also more specific (for example the mismatch reason lists the notice types that do exist for the asset).

## Results

| Result | Check |
|---|---|
| PASS | Page title is generic |
| PASS | Welcome screen shown on first load |
| PASS | Fictional-data banner visible |
| PASS | No banned company/vendor terms on the rendered page (4 terms supplied privately) |
| PASS | Load Demo / Upload / Reset buttons present |
| PASS | Demo loads 82 ETR records |
| PASS | Summary cards: total/ready/pending/processed |
| PASS | Source badge shows Demo |
| PASS | Welcome hidden after load |
| PASS | Every ETR row classified exactly once |
| PASS | All 82 classifications match expected_results.csv |
| PASS | Ready cases (36 rows) |
| PASS | False-positive (asset match, wrong type) cases (12 rows) |
| PASS | Duplicate/ambiguous notice cases (5 rows) |
| PASS | Only duplicate-flagged notices (2 rows) |
| PASS | Duplicate-flagged notices ignored, valid one used (3 rows) |
| PASS | Missing facility mapping (3 rows) |
| PASS | Conflicting facility mapping (2 rows) |
| PASS | Incomplete facility mapping (2 rows) |
| PASS | Near-miss asset codes (3 rows) |
| PASS | Invalid / missing dates (5 rows) |
| PASS | Missing identifier (2 rows) |
| PASS | Missing transaction type (2 rows) |
| PASS | Unmapped transaction types (2 rows) |
| PASS | False-positive trap: no asset-code-only match accepted as Ready |
| PASS | Pending row expands to show the reason and match inputs |
| PASS | Ready row shows 'Why Ready' + portfolio table |
| PASS | PDF map resolves relative demo-pdfs paths |
| PASS | Sample PDF links return real PDFs (200, %PDF) |
| PASS | 'Notice Found' button opens a bundled PDF URL in a new tab |
| PASS | Expanded row PDF link points at bundled PDF |
| PASS | Ready rows with a Document ID but no PDF path show 'No Notice' badge |
| PASS | Search filters Ready rows |
| PASS | Search with no hits shows empty state |
| PASS | Clearing search restores all rows |
| PASS | Search by reason text finds mismatch rows in Pending |
| PASS | Column filter popover opens (dropdown values listed) |
| PASS | Column filter narrows Pending rows |
| PASS | Sort ascending by Identifier |
| PASS | Sort descending by Identifier |
| PASS | Group by issuer renders groups |
| PASS | CSV export (Pending): filename marks DEMO + header + all rows |
| PASS | CSV export (Ready): row count + extra notice columns |
| PASS | Mark as Processed moves a row to Processed |
| PASS | Processed tab lists the row with 'Processed By'/time |
| PASS | CSV export (Processed) downloads |
| PASS | Undo restores the row to Ready |
| PASS | Demo seeds fictional members |
| PASS | Auto-assign letters creates assignments |
| PASS | Letter-range filter buttons appear on Ready tab |
| PASS | Letter-range filter reduces visible Ready rows |
| PASS | Auto-assign rows creates row assignments |
| PASS | Browser storage keys are namespaced (etrdemo_*) |
| PASS | Reset Demo returns to welcome and clears storage |
| PASS | Upload flow with sample files: same 82 rows |
| PASS | Uploaded results are identical to demo-loaded results |
| PASS | Upload is labelled as user data (badge + banner), not demo |
| PASS | CSV from uploaded data is NOT prefixed DEMO_ |
| PASS | Uploaded data survives refresh (this browser only) |
| PASS | Notice file with prefixed asset-code header + other sheet name still loads |
| PASS | Trailing blank ETR rows are ignored (still 82 rows) |
| PASS | Rejects: Corrupt .xlsx (random bytes) |
| PASS | Rejects: File that starts like a zip but is not a workbook |
| PASS | Rejects: Empty file |
| PASS | Rejects: Wrong extension (.txt) |
| PASS | Rejects: ETR missing 'Transaction Date' column |
| PASS | Rejects: ETR missing 'Identifier' column |
| PASS | Rejects: ETR with header only (no data rows) |
| PASS | Rejects: Notice file missing 'Asset Code' column |
| PASS | Rejects: Facility file missing 'Search Value' column |
| PASS | Rejects: Facility file uploaded in the ETR slot |
| PASS | Rejects: ETR file uploaded in the notice slot |
| PASS | App recovers and processes valid files after errors |
| PASS | No external network requests (fonts, CDNs, APIs) |
| PASS | No uncaught JS errors |
| PASS | No console errors during normal use (rejected-upload errors excluded) |
| PASS | Content-Security-Policy blocks outside connections |
| PASS | With CSP enforced: demo load, expand, CSV export, file picker all work with no violations |
| PASS | Mobile: welcome screen has no horizontal page scroll |
| PASS | Mobile: dashboard has no horizontal page scroll |
| PASS | Mobile: expanded row details visible without page overflow |

## Confidentiality

* No source file, row, name, identifier, asset code, document ID, e-mail, path or URL from the original data is used. A comparison of every text value in the demo files and PDFs against the original source workbooks found overlap only in generic vocabulary the schema requires (transaction types such as "Interest Payment", status names such as "Reconciled", facility types such as "Term Loan B").
* No company name, logo, vendor product name or internal path remains in `index.html`, the data files or the PDFs. Excel files carry no author metadata; PDFs say "anonymous".
* The page loads nothing from the internet (no fonts, CDNs, analytics) and sets a restrictive Content-Security-Policy.
* **Not removed:** the tool's business rules and the transaction-type mapping are the original author's design. Confirm you may publish them.

## Bugs and limitations found (not claimed fixed unless stated)

1. *Original, dormant code* (fixed by removal): Scheduled-Paydown fallback described but never executed.
2. *Original, pre-existing* (fixed in this version): header whitespace could blank the Facility Code column; duplicate facility identifiers resolved silently.
3. *Limitation:* "Processed" is a user action in the session, not something derived from the data. The notice's own Reconciliation/Processing statuses are shown but do not affect classification.
4. *Limitation:* ETR rows are not de-duplicated; two identical ETR lines both appear.
5. *Limitation:* when several portfolios match, the first notice in file order supplies the top-level loan fields; per-portfolio amounts and documents are all shown.
6. *Limitation:* slash dates are US `M/D/YYYY`; an impossible month is rejected, never guessed.
7. *Limitation:* `.xls` and `.xlsb` are not supported (the original also accepted `.xls` for the optional map). The built-in reader was cross-checked cell-for-cell against pandas/openpyxl on the demo files and on larger real exports (~35,000 rows × 72 columns read in ~3 s in Node); no mismatches were found. Dates are shown as `YYYY-MM-DD`, which can differ from Excel's display format.
8. *Limitation:* uploaded DocID maps with network paths create `file://` links that browsers only open on a machine that can reach them. The demo uses relative links.
9. *Limitation:* browser storage holds uploaded data, members, assignments and an audit log on the visitor's own device only (key prefix `etrdemo_`); very large uploads may exceed the storage quota (the page warns).
10. *Not tested:* Firefox and Safari (Chromium only), real touch devices (mobile layout was checked at a 390 px viewport only), and screen-reader behaviour.
