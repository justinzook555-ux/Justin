# Shipping & Overhang Cost Calculator

A single self-contained HTML page (`index.html`) that quotes delivery cost
for portable buildings from five dispatch locations, by destination ZIP
code, building width, and building length — with an optional overhang
add-on priced and totaled in the same quote.

Open `index.html` directly in any browser — no build step, no server, no
external services required at runtime.

## Locations

| Location | ZIP | Rate tier |
|---|---|---|
| Greenwood, SC | 29649 | Standard |
| Clarkson, KY | 42726 | Standard |
| Powell, WY | 82435 | Standard |
| Mill Hall, PA | 17751 | Standard (same as Kentucky) |
| Atglen, PA | 19310 | Atglen |

## Rate rules

**Standard tier** (Greenwood SC, Clarkson KY, Powell WY, Mill Hall PA):
- $10/mile for 10′-wide, $12/mile for 12′-wide, $14/mile for 14′-wide
- +60 mile buffer added to the estimated distance
- $500 flat drop fee

**Atglen tier** — rate depends on building length (load size):
- **30′ and under (half load)**: $5/mile (10′-wide), $6/mile (12′-wide), $7/mile (14′-wide)
- **31′–42′ (three-quarter load)**: $7/mile (10′-wide), $8.50/mile (12′-wide), $10/mile (14′-wide)
- **43′ and over (full load)**: same per-mile rates as the Standard tier ($10/$12/$14)
- +50 mile buffer added to the estimated distance (all length classes)
- $250 flat drop fee — **$750** if the destination is Long Island, NY (Nassau or Suffolk County)

`total = max(rate × billed miles, minimum mileage cost) + drop fee`

**Minimum mileage cost** (a floor for short-haul deliveries): if the
calculated mileage cost falls below this amount, the minimum is billed
instead — the drop fee is still added on top as normal.
- **$750** for Greenwood SC, Clarkson KY, Powell WY, and Mill Hall PA
- **$250** for Atglen, PA

## Distance estimate

The page cannot call a live routing API at runtime (browser sandbox
restrictions), so mileage is estimated from an embedded ZIP-code
latitude/longitude table:

`estimated driving miles = straight-line distance × road-circuity multiplier`

The circuity multiplier defaults to **1.25** (a reasonable rural/interstate
approximation) and is adjustable per-quote under "Advanced". For an exact
number, use the **manual mileage** field in Advanced to type in a distance
looked up from a mapping tool — the location's mile buffer is still added on
top of a manual entry.

Long Island detection is automatic (destination ZIP falls in Nassau or
Suffolk County, NY) with a manual override checkbox for edge cases.

## Assumptions worth double-checking

- "Clarks, KY" and "Ackland, PA" from the original request were confirmed as
  **Clarkson, KY (42726)** and **Atglen, PA (19310)**.
- The Atglen $250/$750 drop fee is flat regardless of building length; only
  the per-mile rate changes across the half / three-quarter / full load
  length brackets.
- 30′ exactly falls into the half-load bracket (30′ and under); the
  three-quarter bracket is 31′–42′, and full load starts at 43′.
- The minimum mileage cost applies to the *mileage cost* line only (rate ×
  billed miles), not the total — the drop fee (and Long Island surcharge)
  is always added on top after the minimum is applied.
- ZIP-to-coordinate data covers ~42,000 US ZIP codes; a destination ZIP not
  in the table will prompt for manual mileage entry instead of blocking the
  quote.

## Overhang add-on

A single **"Add an overhang to this quote"** checkbox on the main form
reveals an Overhang Size picker (4′ / 8′ / 10′ / 12′). It reuses the same
form fields as the shipping quote — no separate shop, ZIP, or length
fields:

- **Building Width** (already selected above) decides 4′ Fixed vs. 4′
  Hinged: a 10′-wide barn always gets 4′ Fixed; every other width gets 4′
  Hinged.
- **Building Length** (already entered above) looks up the overhang's
  price. It must be 12′–60′ in 2′ steps to match the price sheet — the
  calculator shows an error and won't quote the add-on otherwise.
- **Ship From** and **Destination ZIP** (already selected above) set the
  route, with one exception: see Greenwood, SC below.

Prices pulled from the "Shed Row Overhang Pricing" sheet (revised
07/24/2025).

**Overhang types and per-mile add-ons:**

| Overhang | Base price | Per-mile add-on |
|---|---|---|
| 4′ Fixed | flat installed price | none |
| 4′ Hinged | flat installed price | +$12/mile (crew) |
| 8′ Overhang (7′ clearance) | installed, includes roof upgrades | +$14/mile (onsite labor) |
| 10′ Overhang (7′ clearance) | installed, includes roof upgrades | +$14/mile (onsite labor) |
| 12′ Overhang (7′ clearance) | installed, includes roof upgrades | +$14/mile (onsite labor) |

- **4′ Hinged is not offered at 50′ barn length and over** (N/A on the
  sheet) — the calculator blocks the quote and says to call the shop.
- 8′/10′/12′ overhangs on 50′–60′ barns include forklift onsite (already
  baked into the table price).
- 8′, 10′, and 12′ overhangs include roof upgrades (taller front wall,
  2×6 rafters on 16″ centers — may vary by local snow load).
- The sheet's **"overhangs out of Corral Shop"** surcharge (**+$500**
  flat) is applied automatically, not as a checkbox — see below.
- The full price table and these rules are reproduced below the quote form,
  collapsed by default behind a "Show full overhang price table & rules"
  toggle.

`overhang total = (billed miles × per-mile rate) + base price + $500 out-of-shop fee (if applicable)`
`grand total = shipping total + overhang total`

**Overhang mileage — shared with the shipping leg, except out of
Greenwood, SC**:

- **Clarkson KY, Atglen PA, Powell WY, and Mill Hall PA**: the overhang
  travels with the same trip as the building, so it reuses the exact same
  distance (and manual mileage override, if entered) as the shipping quote
  above, plus its own **40-mile buffer** and per-mile rate.
- **Greenwood, SC** builds its overhangs out of **Mooresville, NC
  (28115)** instead, so overhang mileage is always auto-estimated from
  there — the shipping leg's manual mileage override does not apply to it
  — and the **$500 out-of-shop fee is added automatically** (no checkbox).
  If the destination ZIP isn't in the lookup table, the overhang add-on
  can't be priced this way and the calculator shows an error.
