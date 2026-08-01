---
name: perk-export-expenses
description: Export a company's Perk expenses (invoice lines) for a date range for finance reconciliation.
api: Perk Travel & Spend API
base_url: https://api.perk.com
operations:
  - list-invoice-profiles      # GET /invoices/profiles
  - list-invoice               # GET /invoices
  - list-invoice-lines-1       # GET /invoices/lines
  - download-pdf               # GET /invoices/{serial}/pdf
mcp_tools:
  - invoices_list_invoice_profiles
  - invoices_list_invoices
  - invoices_list_invoice_lines
method: generated
source: https://developers.perk.com/reference
---

# Export Perk expenses for a date range

Pull all booked-service expense lines for a period so finance can reconcile them.

## Auth
Send the account API key on every request:
`Authorization: apikey <your_api_key>` and `Api-Version: 1`.
(Partners use OAuth 2.0 Authorization Code instead — see `authentication/perk-authentication.yml`.)

## Steps
1. `GET /invoices/profiles` (**list-invoice-profiles**) to find the invoice profile(s) — the legal entity trips are invoiced to.
2. `GET /invoices/lines?expense_date_gte=<from>&expense_date_lte=<to>` (**list-invoice-lines-1**) to get every expense line. Each line is one booked service (flight/hotel/car) with trip metadata, traveler, cost center, tax, and amount.
3. Page through results using `offset`/`limit`; stop when `offset + returned >= total`.
4. Optionally `GET /invoices` (**list-invoice**) for invoice-level totals and `GET /invoices/{serial}/pdf` (**download-pdf**) for the source document.

## Rules
- Only invoices issued on or after 2019-01-01 are returned.
- On `401` re-check the API key; on `429` back off and retry after the seconds in the message (see `errors/perk-problem-types.yml`).
- For a conversational/agent flow, prefer the MCP tools above against `https://mcp.perk.com/api/mcp/mcp`.
