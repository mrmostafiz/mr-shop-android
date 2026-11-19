# MR Shop Android — v0.3.0 Offline Sync Foundation

## Included
- Yellow / white / blue visual direction specified by the supplied design references (UI redesign is staged; existing app screens remain a prototype shell).
- Room local database and persistent sync outbox table.
- WorkManager worker constrained to network connectivity, retry/backoff, and batch delivery.
- Google Apps Script receiver that creates Products, Stock, Sales, Expenses, Customers and Transactions sheets.
- Idempotent event handling by unique Event ID in Google Sheets.
- Product create and stock adjustment enqueue events for cloud sync.
- Google Sheets connection profile card component (`GoogleSheetConnectionCard`).
- Google ML Kit Google Code Scanner helper for common product barcode formats.

## Connect Google Sheets
1. Create a Google Spreadsheet.
2. Open Extensions → Apps Script.
3. Replace Code.gs with `google-apps-script/Code.gs`.
4. Change `SHARED_TOKEN` to a long random secret (at least 20 characters).
5. Deploy → New deployment → Web app.
6. Execute as: Me. Access: Anyone (or your approved organization policy).
7. Copy the Web App URL ending `/exec`.
8. In the app Profile screen, enter the URL and same token.

## Important integration notes
- `GoogleSheetConnectionCard(context)` is provided as a composable for the Profile screen; it must be added to the app's Settings/Profile route.
- Call `SheetSync.schedule(context)` after a local write is committed. The WorkManager worker will wait for network availability and retry.
- Use `scanProductBarcode(activity, onResult = { scanned -> ... })` from a product form. The helper uses Google Play services' code scanner; first use may download the scanner module.
- The current sample app does not yet include a full Sales/POS workflow. When sales are implemented, enqueue one immutable SALE transaction event per sale (plus payment/stock movement events) in the same Room transaction as the sale.
- Existing product and stock repository methods enqueue events, but the caller must trigger `SheetSync.schedule(context)` after those writes. Wire this from the UI after a successful save.
- Sync endpoint currently stores each event as JSON in a row. This preserves the original event data and gives one row per unique event; it is not yet a normalized accounting workbook with formulas/reports.
- This is an implementation foundation, not a production release. Test on multiple devices and poor networks before using for real accounts.

## Security and multi-user
- Shared token is a simple receiver authentication mechanism, not a full user identity system. Anyone with the deployed endpoint and token can submit events.
- For production, use Firebase Authentication or a server-side identity layer and enforce authorization server-side. Do not rely on app-only role checks.
- Google Apps Script Web Apps configured as "Anyone" expose the endpoint publicly; keep the token private and rotate it if exposed.
- Google Sheets is a reporting/backup destination, not a transactional database. Room remains the immediate source of truth for offline operations.
