# AJIO Seller Control

An operations dashboard for AJIO **dropship (VMS)** sellers. It runs as a single static
page — no server, no build step, no install. Open `index.html`, load your own export,
and everything is computed in your browser.

**This repository contains no seller data, by design.** Pricing and catalog files are
git-ignored so the repo can be public without exposing a discounting strategy.

---

## Why this exists

AJIO publishes **no seller API**. The seller portal is download-and-upload only, so any
dashboard is necessarily a "download the report, load it here" workflow. A page served
from GitHub Pages cannot fetch `seller.ajio.com` directly — browsers block cross-site
requests — so a button claiming to sync live would simply not work.

## Getting started

1. In AJIO Seller Central, open **Products → Pricing**.
2. On the **Download** tab, click **Download Trade Discount Excel File**.
3. Wait for the batch to show **Completed**, then download it.
4. Open the dashboard and drop that file on the first screen.

Your file is parsed locally and kept in this browser's `localStorage`. Nothing is
uploaded. Use **Reports & Imports → Clear all local data** to remove it.

## What it tells you

The first thing it computes is **catalog readiness** — how many of your active SKUs can
actually sell:

- SKUs with **no trade discount set** have no selling price and are effectively invisible.
- SKUs at a **very low trade discount** (often exactly `1.00%`, the signature of a bulk
  price upload that skipped rows) list at close to full MRP and will not convert.

Trade Discount is the percentage off Listing MRP, so `selling price = MRP × (1 − TD)`.

## Tabs

| Tab | Needs |
|---|---|
| Overview | Trade Discount export |
| Pricing & Promotions | Trade Discount export |
| Catalog Health | Counts you enter from Catalog → Upload History |
| Product Performance | Export, plus order report for units and revenue |
| Orders | Dropship Order Report |
| Returns & Refunds | Dropship Rtv Report, Credit Note Report |
| Return Loss | Returns + Payment report + your cost file |
| Settlements | Dropship Payment Report, Invoice Report |
| Inventory | Products → Inventory export |
| Monthly & 6-Month | One Dropship Order Report per month |
| Action Centre | Whatever is loaded |
| Reports & Imports | — |

Tabs with no data render a named empty state saying which report unlocks them. They do
**not** show zeros — an unknown figure and a figure of zero are different things.

## The eight AJIO reports

Found under **Reporting → View Reports**. There is no inventory or advertising report.

| Report | Unlocks |
|---|---|
| Dropship Order Report | Orders, fulfilment, product performance |
| Dropship Shipment Report | Dispatch timings, carrier, AWB |
| Dropship Pendency Report | Orders awaiting dispatch |
| Dropship Rtv Report | Returns |
| Dropship Payment Report | Settlements |
| Credit Note Report | Refunds / credit notes |
| Invoice Report | Invoice values |
| Dropship GST Report | Tax entries |

Each run is capped to a short date range, so backfill month by month.

## How the numbers are defined

Stated up front so any figure can be audited:

- **Return rate (units)** = returned units ÷ delivered units × 100
- **Return rate (orders)** = returned orders ÷ delivered orders × 100
  Returns are de-duplicated to one record per return ID first, because the reports carry
  one row per status change; counting rows inflates the rate.
- **Expected net settlement** = order value − (commission + fees + shipping + other
  deductions + refunds + discount). A variance beyond ₹1 is flagged for investigation —
  it is not proof of underpayment, since timing differences and partial settlements look
  identical.
- **Return loss** is never "refund = loss". Resellable stock returns to inventory and some
  fees reverse, so loss is only computed when refund value, fee reversal, reverse
  logistics and unit cost are all present.
- **Month-on-month growth** = (current − previous) ÷ previous × 100, suppressed with a
  warning when the previous period is zero or incomplete.

## Supplying costs

AJIO never provides unit cost. Margin, profitability and true return loss stay unavailable
until you load your own file mapping SKU → unit cost.

## Tech

Plain HTML, CSS and JavaScript. [SheetJS](https://sheetjs.com/) (CDN) parses `.xlsx`;
CSV is parsed in-page. No framework, no bundler, no dependencies to install.

## Licence

MIT
