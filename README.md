# Shipping & Overhang Cost Calculator

A single self-contained HTML page (`index.html`) that quotes delivery cost
for portable buildings from five dispatch locations, by destination ZIP
code, building width, and building length — with an optional overhang
add-on priced and totaled in the same quote.

Open `index.html` directly in any browser — no build step, no server
required. Mileage uses live OpenRouteService driving distance when an API
key is configured (see "OpenRouteService setup" below); otherwise
everything still works using a built-in straight-line distance estimate.

## OpenRouteService setup (optional, free, no billing account needed)

By default mileage is estimated from ZIP-code coordinates (great-circle
distance × a 1.25 road-circuity multiplier) — no external service, works
everywhere. To use exact driving distance instead:

1. Go to [openrouteservice.org/dev/#/signup](https://openrouteservice.org/dev/#/signup)
   and create a free account — no credit card required.
2. In the dashboard, **Request a token** → **Standard (free)** → name it
   anything (e.g. "shipping-calculator") → create.
3. Copy the token.
4. Open `index.html`, find `var ORS_API_KEY = "";` near the top of the
   `<script>` block, paste the token between the quotes, and push.

Free tier: 2,000 requests/day, 40/minute — comfortably enough for a
shipping calculator doing quotes.

**If the key is missing, invalid, over quota, or a lookup fails for any
reason**, the calculator silently falls back to the straight-line
estimate for that quote — it never blocks or errors out. The quote
breakdown always shows which method was used ("OpenRouteService driving
distance" vs. "Estimated driving distance").

**Note on the Claude Artifact preview link**: the Artifact viewer's
content security policy blocks outbound network calls to any external
service, so OpenRouteService mileage only works on the real hosted page
(GitHub Pages) or the standalone file — the Artifact preview will always
show the estimate, even with a key configured.

## Locations

| Location | ZIP | Rate tier |
|---|---|---|
| Greenwood, SC | 29649 | Standard |
| Clarkson, KY | 42726 | Standard |
| Powell, WY | 82435 | Standard |
| Mill Hall, PA | 17751 | Standard (same as Kentucky) |
| Atglen, PA | 19310 | Atglen |

## Rate rules

Each shop ships a different set of building widths, shown as the Building
Width options once that shop is selected:
- **Greenwood SC, Clarkson KY, Powell WY, Mill Hall PA**: 6′, 8′, 10′, 12′, 14′-wide
- **Atglen, PA**: 6′, 8′, 10′, 12′-wide (no 14′)

**Standard tier** (Greenwood SC, Clarkson KY, Powell WY, Mill Hall PA):
- 10′/12′/14′-wide are a flat per-mile rate: $10/mile, $12/mile, $14/mile
- 6′/8′-wide are length-bracket priced, like Atglen — see the table below
  — but at **$2/mile more per bracket** than Atglen's 6′/8′ rates
- +60 mile buffer added to the estimated distance
- $500 flat drop fee

**Standard tier 6′/8′-wide length brackets:**

| Building length | 6′ / 8′ wide |
|---|---|
| 12 ft and under | $6/mile |
| 13–24 ft | $8/mile |
| 25–40 ft | $9/mile |
| 41 ft and over | $10/mile |

**Atglen tier** — Atglen does not split loads; every shipment prices off
the per-mile rate for its width and building-length bracket:

| Building length | 6′ / 8′ wide | 10′ wide | 12′ wide |
|---|---|---|---|
| 12 ft and under | $4/mile | $5/mile (14 ft and under) | $8/mile (14 ft and under) |
| 13–24 ft | $6/mile | $7/mile (15–24 ft) | $9/mile (15–24 ft) |
| 25–40 ft | $7/mile | $8/mile | $10/mile |
| 41 ft and over | $8/mile | $9/mile | $12/mile |

(10′ and 12′-wide use 14 ft, not 12 ft, as the first bracket's ceiling —
see Assumptions below.)

**Atglen mileage bands** (based on driving distance, not straight-line):
- **Under 50 miles**: flat **$250** minimum shipping charge — no separate
  mileage cost, mile buffer, or drop fee on top of it.
- **50–100 miles**: normal rate × miles, **no mile buffer**, plus the $250
  drop fee ($750 to Long Island, NY).
- **Over 100 miles**: normal rate × miles **plus a 50-mile buffer**, plus
  the $250 drop fee ($750 to Long Island, NY).

`total = 250                                   if driving miles < 50
       = (rate × miles) + drop fee             if 50 ≤ driving miles ≤ 100
       = (rate × (miles + 50)) + drop fee       if driving miles > 100`

**Standard tier minimum mileage cost** (a floor for short-haul
deliveries out of Greenwood SC, Clarkson KY, Powell WY, and Mill Hall
PA): **$750** — if the calculated mileage cost (rate × billed miles)
falls below this, the minimum is billed instead and the $500 drop fee
is still added on top as normal. Atglen no longer uses this mechanism;
its short-haul floor is the flat $250 charge under 50 miles above.

## Distance calculation

Mileage resolves in this order, for both the shipping leg and an added
overhang's leg:

1. **Manual mileage** (Advanced), if entered — always wins.
2. **OpenRouteService driving distance**, if an API key is configured (see
   "OpenRouteService setup" above) and the destination ZIP resolved.
3. **Straight-line estimate** from an embedded ZIP-code latitude/longitude
   table, as a fallback: `estimated driving miles = straight-line distance
   × road-circuity multiplier`. The circuity multiplier defaults to
   **1.25** (a reasonable rural/interstate approximation) and is
   adjustable under "Advanced".

The quote breakdown always states which method was actually used for
that quote.

Long Island detection is fully automatic — the destination ZIP's county is
checked against Nassau/Suffolk, NY — with no manual checkbox or override.

## Assumptions worth double-checking

- "Clarks, KY" and "Ackland, PA" from the original request were confirmed as
  **Clarkson, KY (42726)** and **Atglen, PA (19310)**.
- The Atglen $250/$750 drop fee is flat regardless of building length; only
  the per-mile rate changes across the length brackets.
- The Atglen length brackets as given had gaps or overlaps at their edges
  (e.g. "0-12" then "14-24" for 6′/8′-wide; "0-14" then "14-24" for
  10′/12′-wide). Since buildings are built in 2 ft steps, each bracket is
  implemented as "up to and including its stated ceiling," so the next
  bracket effectively starts 2 ft later — no length is double-priced or
  unpriced. This is why 10′/12′-wide's first bracket ceiling is 14 ft
  while 6′/8′-wide's is 12 ft: that's what was explicitly stated for each.
  A length above the last bracket's ceiling (56 ft) still bills at that
  top bracket's rate.
- 14′-wide is no longer offered from Atglen — the new pricing only defined
  6′, 8′, 10′, and 12′-wide, so the Building Width choices for Atglen
  became 6′/8′/10′/12′ instead of 10′/12′/14′.
- 6′/8′-wide from the other four shops uses the exact same bracket
  *breakpoints* as Atglen's 6′/8′ table (≤12, 13–24, 25–40, 41+ ft) with
  every rate $2/mile higher — it does not use Atglen's own mile buffer or
  drop fee, only its bracket structure.
- Building Length is required whenever the rate is bracket-priced: Atglen
  at any width, or 6′/8′-wide from any shop. It's unused (and optional)
  for 10′/12′/14′-wide outside Atglen.
- The standard-tier minimum mileage cost applies to the *mileage cost*
  line only (rate × billed miles), not the total — the drop fee (and
  Long Island surcharge) is always added on top after the minimum is
  applied. Atglen's under-50-mile flat fee is different: it replaces the
  *entire* total (mileage cost and drop fee both), since it's a flat
  minimum shipping charge rather than a floor on one line item.
- The "over 100 miles" buffer condition is strict: exactly 100 miles gets
  no buffer, 100.1+ miles does.
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

The quote shows two separate totals — **Barn Total** and **Overhang
Total** — with no combined grand total.

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
