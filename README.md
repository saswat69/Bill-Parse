# bill-upload

Personal reimbursement claim workspace. Open this folder as its own Cursor project (not `finalyzer-ui`).

## Setup

1. Place blank HR templates in `templates/`:
   - `Claim Statement-2026.xlsx`
   - `Employee claim Form -2026.docx`
2. Drop PDFs into `Travel/` or `Food/`.
3. Edit `config.yaml` for the quarter (period, employee details).
4. In Cursor, run the **`/process-claim`** command.
5. Upload renamed PDFs and filled forms to Drive yourself.

## Folders

| Folder | Purpose |
|--------|---------|
| `templates/` | Read-only blank forms from HR |
| `Travel/` | Uber, petrol, flight PDFs |
| `Food/` | Zomato / Swiggy PDFs |
| `output/` | Generated markdown + filled claim files (gitignored) |

## Command

`.cursor/commands/process-claim.md` — guides the AI: read/rename PDFs → markdown → fill Excel & Word in `output/`.
