# Shipping & Overhang Cost Calculator

A single self-contained HTML page (`index.html`) with two tools:

1. A shipping calculator that quotes delivery cost for portable buildings
   from five dispatch locations, by destination ZIP code, building width,
   and building length.
2. An overhang calculator, at the bottom of the page, that prices shed row
   / run-in overhangs by barn length and overhang size.

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

**Atglen tier**:
- Buildings **under 30′ long**: $5/mile (10′-wide), $6/mile (12′-wide), $7/mile (14′-wide)
- Buildings **30′ and over**: same per-mile rates as the Standard tier ($10/$12/$14)
- +50 mile buffer added to the estimated distance (both length classes)
- $250 flat drop fee — **$750** if the destination is Long Island, NY (Nassau or Suffolk County)

`total = (estimated driving miles + buffer) × per-mile rate + drop fee`

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
  the per-mile rate changes at the 30′ length split.
- ZIP-to-coordinate data covers ~42,000 US ZIP codes; a destination ZIP not
  in the table will prompt for manual mileage entry instead of blocking the
  quote.

## Overhang calculator

Prices pulled from the "Shed Row Overhang Pricing" sheet (revised
07/24/2025), for barn lengths 12′–60′ in 2′ steps.

**Overhang types and per-mile add-ons:**

| Overhang | Base price | Per-mile add-on |
|---|---|---|
| 4′ Fixed | flat installed price | none |
| 4′ Hinged | flat installed price | +$12/mile (crew) |
| 8′ Overhang (7′ clearance) | installed, includes roof upgrades | +$14/mile (onsite labor) |
| 10′ Overhang (7′ clearance) | installed, includes roof upgrades | +$14/mile (onsite labor) |
| 12′ Overhang (7′ clearance) | installed, includes roof upgrades | +$14/mile (onsite labor) |

- **4′ Fixed vs. 4′ Hinged is automatic**: a 10′-wide barn always gets 4′
  Fixed; every other barn width gets 4′ Hinged.
- **4′ Hinged is not offered at 50′ barn length and over** (N/A on the
  sheet) — the calculator blocks the quote and says to call the shop.
- 8′/10′/12′ overhangs on 50′–60′ barns include forklift onsite (already
  baked into the table price).
- 8′, 10′, and 12′ overhangs include roof upgrades (taller front wall,
  2×6 rafters on 16″ centers — may vary by local snow load).
- The sheet's **"overhangs out of Corral Shop"** surcharge (**+$500**
  flat) is applied automatically, not as a checkbox — see below.
- The full price table is reproduced under the calculator for reference.

`total = (estimated driving miles + 40 mi buffer) × per-mile rate + $500 out-of-shop fee (if applicable)`

**Mileage is ZIP-based, like the shipping calculator**: pick which shop is
building the overhang and type the destination ZIP code — driving distance
is estimated the same way (great-circle distance × road-circuity
multiplier, with a manual override available), plus a flat **40-mile
buffer** added before the per-mile rate is applied. The mileage *origin*
point depends on the shop:

- **Greenwood, SC** builds its overhangs out of **Wytheville, VA (24382)**,
  so mileage is measured from there instead of Greenwood, and the **$500
  out-of-shop fee is added automatically** for every Greenwood quote (no
  checkbox — it's just part of shipping out of South Carolina).
- **Clarkson KY, Atglen PA, Powell WY, and Mill Hall PA** each bill mileage
  from their own location directly, with no out-of-shop fee.
