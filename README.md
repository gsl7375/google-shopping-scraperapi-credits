# Google Shopping scraper: what ScraperAPI returns, why every request costs 25 credits, and how many keywords your plan really covers (full plan table)

ScraperAPI handles Google Shopping through a dedicated structured-data endpoint that returns JSON, and it bills those calls at the Google/Bing rate of **25 credits per request**. The free tier's 1,000 credits therefore buy 40 Shopping requests, enough to inspect a payload and not much else. Everything below is aimed at the two numbers that decide whether this works for you: how many requests your keyword list turns into, and what each of those requests brings back.

This is a google shopping scraper question with a billing question hiding inside it, so the article covers both: every plan with its allowance, the credit table that sets your monthly cost, the documented response fields, the four Google Shopping behaviours no vendor can code around, and the point at which a different kind of tool is the better buy.

> **Quick pick.** Still checking whether the JSON fits your pipeline? Start on the free plan: 1,000 credits, no credit card, 40 Shopping requests. Running keyword checks every day? Startup's 1,000,000 credits cover 40,000 requests a month, about 1,300 a day.
> 👉 [Check current plan pricing and start on the free tier](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Every ScraperAPI plan, with the Google Shopping allowance converted

ScraperAPI sells seven tiers plus a free plan. The "Shopping requests" column is monthly credits divided by 25, which is the only conversion that matters if Google Shopping is all you run.

| Plan | Credits / month | Shopping requests | Threads | Price | Start |
| --- | --- | --- | --- | --- | --- |
| Free | 1,000 | 40 | 5 | $0, no card required | [Free plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | 100,000 | 4,000 | 20 | Listed on the pricing page | [Hobby](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | 1,000,000 | 40,000 | 50 | Listed on the pricing page | [Startup](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | 3,000,000 | 120,000 | 100 | Listed on the pricing page | [Business](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | 5,000,000 | 200,000 | 200 | $475/mo, or $427.50/mo billed annually ($5,130/yr) | [Scaling](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | 10,500,000 | 420,000 | 300 | Listed on the pricing page | [Professional](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | 21,500,000 | 860,000 | 500 | $1,975/mo, or $1,777.50/mo billed annually ($21,330/yr) | [Advanced](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 880,000+ | 500+ | Custom quote | [Enterprise](https://www.scraperapi.com/pricing?fp_ref=coupons) |

On the two tiers with published headline figures, annual billing takes exactly 10% off the monthly rate. Scaling is the one ScraperAPI flags as most popular; Advanced currently carries a limited-offer bonus of 500,000 extra credits, and priority routing.

Plan differences that aren't about volume:

- **Geotargeting.** Hobby and Startup are limited to US and EU. Country-level targeting starts at Business and continues up. For Google Shopping that matters more than it sounds, since prices, merchants and availability change by market.
- **Analytics history.** Last 30 days on Hobby and Startup, unlimited from Business upward.
- **Included on every plan**, including free: JS rendering, premium proxies, rotating proxy pools, and JSON auto parsing. Those are the four things you'd otherwise assemble yourself.
- **Ultra Premium proxies** are available on paid plans only.

## The billing unit that decides your monthly cost

ScraperAPI bills per request, not per product and not per page of results. The same credit table applies to everything you send.

| Request type | Credits | Notes |
| --- | --- | --- |
| Standard page | 1 |  |
| Amazon, Walmart, eBay | 5 |  |
| Google, Bing (subdomains included) | 25 | The Google Shopping endpoint sits in this bucket |
| LinkedIn | 30 |  |
| Bot-protection bypass | +10 | Cloudflare, DataDome, PerimeterX/HUMAN, Akamai |
| JS rendering (`render=true`) | +10 |  |
| Premium proxy (`premium=true`) | +10 |  |
| Screenshot (`screenshot=true`) | +10 |  |
| Premium proxy + JS rendering | 25 total |  |
| Ultra Premium (`ultra_premium=true`) | 30 | Paid plans only |
| Ultra Premium + JS rendering | 75 | Paid plans only |
| Geotargeting, sessions, device type, output format, auto parsing | 0 | No surcharge |

At the two published price points, a Google Shopping request costs about **$0.0024** on Scaling ($475 ÷ 200,000) and about **$0.0023** on Advanced. That sounds trivially cheap until you notice what it buys: one request, whatever the row count turns out to be.

Two practical consequences.

The `num` parameter sets how many results come back per page. A request that returns 20 rows costs the same 25 credits as one that returns 80, so a default `num` you never thought about can double your effective cost per product. Set it deliberately.

Because billing is per request, pagination is where budgets leak. Walking `start` offsets through ten pages of a keyword costs ten times one page, and Google Shopping stops serving fresh products well before that (see below). A Hobby plan is 4,000 requests. Ten pages across 400 keywords already fills it.

> Before committing to a tier, run one request from the API Playground. The Summary panel shows the job's credit cost before you execute, the Domain Cost Estimator in the dashboard prices any URL in advance, and every response returns its own cost in the `sa-credit-cost` header.
> 👉 [Open the pricing page to compare tiers against your request count](https://www.scraperapi.com/pricing?fp_ref=coupons)

## How a Google Shopping call is built

The endpoint lives under **Structured Data Endpoints** in the API Playground. Pick Google Shopping, enter a query and a country code, and the Playground generates the request and shows the response inline.

The documented parameters:

- `API_KEY` (required) and `QUERY` (required).
- `COUNTRY_CODE`, two-letter country codes for the markets ScraperAPI geotargets.
- `TLD`, the Google domain to hit: `com`, `co.uk`, `ca`, `de`, `es`, `fr`, `it`, `co.jp`, `in`, `com.sg`, `com.mx`, `com.br`, `nl`, `com.au`, `com.tr`, `sa`, `se`, `pl`, `cn`, `ae`. Defaults to `com`. When scraping google.com from another country, set both `TLD` and `COUNTRY_CODE`.
- `OUTPUT_FORMAT`, `json` (default) or `csv`. CSV is the fastest path to a spreadsheet.
- `UULE` for city-level search origin, `NUM` for results per page, `HL` and `GL` for language and country, `START` for the result offset, plus `IE` and `OE` for query and result encoding.

Batch work runs through the Async method: paste full structured endpoint URLs one per line, get a `statusUrl` per job, poll until `status` reads `finished`, then read `response.body`. Daily monitoring is a crontab entry against that batch. Nothing in the documented parameter list covers sorting or price-range filtering, so order and filter on `extracted_price` in your own code after the fetch. That costs nothing extra and saves the credits a re-query would burn.

## What actually comes back

Each entry in `shopping_results` carries:

| Field | Type | What it's for |
| --- | --- | --- |
| `position` | integer | Where the card sat on the page |
| `title` | string | Product name |
| `price` | string | Formatted price, for display |
| `extracted_price` | float | Numeric price for comparisons, sorting and history |
| `source` | string | The retailer |
| `delivery_options` | string | Delivery text from the card |

Use `extracted_price`, not `price`. String prices in different markets use different separators, and a comparison on "$1,299.00" versus "1299" is a bug waiting for its first European keyword.

The documented list stops at those six fields. Google's own Shopping cards also show star ratings, review counts, strikethrough "was" prices, thumbnails and product IDs, and what your queries actually receive is worth verifying in the Playground before you pick a tier. The one you should check first is a stable product ID: if you plan to track a product's price across runs, you need an identifier that survives a new session. `position` will not do it.

## Four Google Shopping behaviours that hit any scraper

These come from the data source, not from ScraperAPI's implementation.

**Proxies are mandatory.** Google answers an unproxied request with a "JavaScript required" shell or an "unusual traffic" check, neither of which contains products. A scraper that reports success while returning an empty dataset is failing quietly. ScraperAPI's rotating proxy pools come with every plan including free, which removes the part you'd otherwise have to build, rent and keep alive.

**Thumbnails arrive empty more often than you'd expect.** Google lazy-loads product images and publishes only some of their URLs in the page itself. The rest need a second browser request that an HTTP scraper never makes, and top-of-page products are consistently among the missing. Budget for a null image field on a share of rows. Title, price, source and delivery are unaffected.

**Results are personalised and unstable.** Google Shopping varies by location, time and session. Two runs minutes apart can return different products, different prices and different ordering. Join runs on the product ID, never on position.

**Page depth runs out before your credits do.** Vendor measurements of a single results page differ: roughly 50 to 70 products in one count, about 100 unique products (up to 134 observed) in another, depending on whether carousel and shelf placements are counted. Either way, a keyword yields a finite set. Ask for more than that and you're paying 25 credits for repeats. Broad keywords are worse: when Google finds no real inventory for a query, it fills the page with unrelated popular products. Adding a product word to the keyword fixes more of these cases than switching tools does.

## What your keyword list actually costs

Convert your schedule into requests first, credits second. Monthly request counts assume 30 runs.

| What you run | Requests / month | Credits / month | Tier that covers it |
| --- | --- | --- | --- |
| Weekly check, 100 keywords | ~430 | ~11,000 | Hobby |
| Daily price check, 20 products | 600 | 15,000 | Hobby |
| Daily, 100 keywords | 3,000 | 75,000 | Hobby |
| Daily, 500 keywords | 15,000 | 375,000 | Startup |
| Daily, 1,000 keywords × 2 pages | 60,000 | 1,500,000 | Business |
| Hourly, 500 keywords | 360,000 | 9,000,000 | Professional |

Two things fall out of that table. The free plan's 40 requests are a payload test, not a monitoring budget: a weekly check of even 20 keywords needs 88 requests a month. And Hobby's 4,000 requests absorb a surprising amount of daily work, which is why the jump straight to Scaling is usually the wrong instinct.

Before you hit a ceiling, know which ceiling it is. Exceed your concurrency and you get a 429 asking you to dial back; the request is turned away and you aren't charged for it. Exhaust your credits and scraping pauses until renewal, unless you've switched on auto-upgrade (Hobby through Professional) or pay-as-you-go (Scaling, Professional, Advanced, Enterprise), the latter at a fixed per-credit rate up to a monthly cap you set. Annual plans can also renew early for a fresh batch of credits.

> If your daily volume sits under 1,300 requests, Startup's 1,000,000 credits leave headroom for retries and pagination. Past that, Business at 3,000,000 is the tier that stops the pause-and-wait conversation.
> 👉 [Compare Startup and Business credit allowances](https://www.scraperapi.com/pricing?fp_ref=coupons)

## When this is the wrong purchase

Occasional lookups. If you want the price of ten specific products this afternoon, 40 free requests will cover it once. A monthly plan is the wrong shape for a task you'll repeat twice a year.

One-off exports. If the deliverable is a spreadsheet you'll hand to someone and never refresh, a per-result tool or a downloaded dataset costs less than a subscription with a 4,000-request floor.

Zero budget plus proxy appetite. Open-source Google Shopping scrapers exist and some are decent. Running one means owning the proxy pool, the retry logic and the failure alerting, which is precisely what the 25-credit rate pays for. That trade is fine if your time is cheap and your volume is low.

Very sparse keywords. Per-request billing at 25 credits is efficient when a query returns 60 cards and poor when it returns 12. Several managed scrapers bill per product returned instead, and for a list of niche keywords that model can undercut per-request pricing. Run your own keyword sample through a cost estimate before assuming either model wins.

## Picking a tier in one pass

- **Still validating the payload** → free plan. 1,000 credits, no card, 40 Shopping requests.
- **Up to ~130 Shopping requests a day** → Hobby, 100,000 credits, 20 threads.
- **130 to 1,300 a day** → Startup, 1,000,000 credits, 50 threads.
- **1,300 to 6,500 a day** → Business, 3,000,000 credits, 100 threads.
- **Need 200 concurrent threads more than you need credits** → Scaling at $475/mo, $427.50/mo annually.
- **Continuous multi-market pipelines** → Professional or Advanced, where country-level geotargeting, unlimited analytics history and priority routing are all in place.

Three things to confirm before you enter a card. Run a single request in the Playground and check the response carries the fields you need, particularly a stable product ID for cross-run joins. Check what the free 1,000 credits cover against your real query, since 40 requests is 40 keywords, not 40 pages of one keyword. And know your overage path: 429 at the concurrency limit, a pause at credit exhaustion, and whether your tier qualifies for pay-as-you-go with a cap.

> If you only need the payload check, the free tier answers it today. If you already know your request volume, take the tier that covers it with room for retries rather than the one that fits exactly.
> 👉 [Start on the free plan or pick a paid tier](https://www.scraperapi.com/pricing?fp_ref=coupons)
