Process a quarterly expense reimbursement claim from PDFs in this project.

Read `config.yaml` first for employee details, claim period, and paths.

Scope: **local files only** — read PDFs, write markdown, fill Excel and Word into `output/`. Do not upload to Google Drive or use browser automation; the user handles bill folders and uploads separately.

## Inputs

| Path | Contents |
|------|----------|
| `Travel/` | Uber receipts, petrol bills, flight invoices (PDF) |
| `Food/` | Zomato / Swiggy invoices (PDF) |
| `templates/` | Blank `Claim Statement-*.xlsx` and `Employee claim Form -*.docx` — **never edit in place** |

## Outputs (write to `output/` only)

- `travel-receipts-{date}.md` — columns: ID \| Date \| Amount (₹)
- `food-invoices-{date}.md` — columns: Invoice Number \| Date \| Amount (₹)
- `Claim Statement-{year}.xlsx` — copied from template, then filled
- `Employee claim Form -{year}.docx` — copied from template, then filled

---

## Step 1 — Read PDFs and rename

Use Python (`pypdf`) locally. Pipe terminal output to `.ai/terminal/`.

For each PDF in `Travel/` and `Food/`:

1. Extract invoice/receipt ID, date, and amount.
2. **Rename the file** to `{id}.pdf` if it is not already named that way (e.g. Uber UUID, `IXIFT00002261483`, Swiggy invoice number). Rename in place within the same folder.

**Amount rules (use final paid total, not subtotal):**

| Vendor | Field to match |
|--------|----------------|
| Uber | `Total ₹` (not `subtotal ₹`) |
| Zomato tax invoice | `Total Value` |
| Zomato order summary | `Total ₹` |
| Swiggy | `Invoice Total` |
| Cleartrip / flight | `Net Payable` / `Total Payable` |
| Shell petrol | `TOTAL` line on receipt |
| Scanned petrol memos (OMM etc.) | Manual read from image; no PDF text |

**Dates:** extract from PDF text where possible; use manual dates for scanned PDFs only.

---

## Step 2 — Create markdown summaries

Write both markdown files to `output/`.

Include **all** extracted rows in the markdown (with dates). When filling Excel/Word, use only rows within `claim.period_start` … `claim.period_end` from `config.yaml`.

Drop out-of-period rows from claim totals only — e.g. a March Cleartrip flight in `Travel/` stays in the markdown but is excluded from Apr–Jun claim totals.

Each file ends with a **Total** row (all rows in that markdown).

---

## Step 3 — Fill Excel and Word

Copy templates → `output/`. Never edit `templates/`.

### Excel (`openpyxl`)

Fill **Travel** and **Food** sheets only; leave Phone/Broadband blank unless asked.

| Column | Value |
|--------|--------|
| Serial No. | 1, 2, 3… |
| Date | From markdown (in-period rows only) |
| Description | Invoice / receipt ID |
| Amount | From markdown |
| Bill | `Yes` |

Set employee name and emp ID on all sheets from `config.yaml`.

**Do not corrupt sheet layout:** if `insert_rows` breaks footer merges, rebuild footer from the Food sheet pattern rather than leaving orphaned merges mid-table.

### Word (`python-docx`)

**Critical:** the form uses merged table cells. **Never set `cell.text` on merged outer-table cells** — that duplicates content across columns and destroys layout.

Safe approach:

1. Edit **row 1** merged employee cell only (`rows[1].cells[0]`): name, emp ID, expense period from `config.yaml`.
2. Edit the **nested 6×6 itemized table** inside `rows[2].cells[0]` for Travel, Food, Driver, Phone amounts and TOTAL (in-period totals only).
3. Replace rupees-in-words and claim date via paragraph/run text replace in `rows[2].cells[0]`.
4. Leave header row (logo / PV-HR-04) untouched.

| Description | Amount |
|-------------|--------|
| Travel/Conveyance | In-period travel total |
| Food expenses | Food total |
| Driver receipt | 0 unless user provides |
| Telephone/Landline | 0 unless user provides |

Fill rupees in words (Indian format) and claim date (today unless user specifies).

---

## Final checklist (report to user)

| Category | Row count | Total (₹) |
|----------|-----------|-----------|
| Travel (in-period) | | |
| Food | | |
| **Grand total** | | |

List any PDFs renamed. Note any out-of-period items excluded from Excel/Word. Remind user to upload bills and filled forms to Drive themselves.

## Do not

- Edit files in `templates/`
- Upload to Google Drive or use Chrome MCP
- Mix Food PDFs into Travel processing or vice versa
- Include out-of-period invoices in Excel/Word totals
