# Shopify headless hosting: Is Oxygen enough, what does self-hosting require, and when is a VPS the wrong buy? (Oxygen's hard limits, token rules and a plan table)

Hosting a headless Shopify storefront comes down to two questions: does your frontend need a server at runtime, and do you want Shopify to run it? Build with Hydrogen and deploy to Oxygen, and hosting adds nothing to a paid Shopify plan. Run Next.js or your own stack, and you rent a machine that can execute Node, then own the tokens, environment variables, TLS and uptime yourself.

Those two paths differ by more than a bill. Oxygen removes credential plumbing but caps what your build can weigh and how long a request may run. A self-managed box removes the caps but hands you every operational task Shopify was doing. The sections below cover the limits, the token rules that catch people out, and the VPS catalog you'd be choosing from if you go the self-hosted route.

> **Quick verdict:** if you're on Hydrogen, use Oxygen — it's included on Starter through Plus and saves you the token work. If your storefront is Next.js on infrastructure you already run, a self-managed VPS is the cheapest option that still runs server-side rendering, provided you accept the sysadmin work. 👉 [See SixtyNet's current VPS plans and pricing](https://sixtynet.net/aff.php?aff=39)

## What a headless Shopify storefront actually needs from a host

The rendering mode decides the answer. A statically generated storefront is a folder of HTML, JavaScript and images built once at deploy time, so any CDN can serve it and object storage is enough. A server-rendered storefront runs your framework's server process on every request, which means the host has to run Node and keep that process alive, with static assets ideally served through a CDN.

Three things have to work on either setup:

- **Environment variables.** Your Storefront API tokens and Customer Account API credentials get injected at build or boot time, and they need a place to live that isn't your repository.
- **A build and deploy pipeline.** Push to GitHub or your own CI, build, ship, and be able to roll back.
- **A domain and TLS.** Checkout redirects, cart permalinks and product URLs all assume HTTPS on your own hostname.

Worth ruling out early: if the goal is product cards and a buy button on a site you already run, Storefront Web Components or Buy Button JS drop commerce into an existing page and you never pick a host at all. Headless hosting only becomes a decision once you're building a separate storefront.

## Oxygen: included with your plan, but bounded by hard limits

Oxygen is Shopify's global serverless layer for Hydrogen storefronts. It handles deployment environments, environment variables, caching and CDN integration, and it runs on Cloudflare's open-source `workerd` runtime rather than a full Node process, so some Node.js APIs simply aren't available.

The reason most Hydrogen users start here is credential handling. When you create a storefront in the Hydrogen channel, Shopify injects `PRIVATE_STOREFRONT_API_TOKEN`, `PUBLIC_STOREFRONT_ID`, `PUBLIC_CUSTOMER_ACCOUNT_API_CLIENT_ID` and `SHOP_ID` for you, and you pull the same values locally with `npx shopify hydrogen link` and `npx shopify hydrogen env pull`. Nothing gets copy-pasted by hand.

What Oxygen will not do is scale past its limits, and these are the numbers that decide whether your build fits:

| Limit | Value |
| --- | --- |
| Worker bundle size | 10 MB or less |
| Worker startup time | 400 ms or less |
| Entry file name | `index.js` (source map `index.js.map`) |
| CPU time per request | 30 seconds |
| Memory per worker | 128 MB (exceeding it can drop requests) |
| Custom environment variables | 110 |
| Outbound API requests | Must complete within 2 minutes |
| Static assets | Images 20 MB, video 1 GB, 3D models 500 MB, other files 20 MB |
| Proxy in front of the deployment | Not supported |

Two of those matter more than the rest. The 10 MB bundle ceiling punishes dependency sprawl, so a storefront that pulls in a heavy UI kit or several analytics SDKs can hit the wall before launch. And no proxy means you can't park a third-party WAF or CDN layer in front of Oxygen; Shopify's bot mitigation conflicts with it and the result shows up as SEO problems.

Oxygen is also not a general-purpose host. It's available on development stores, Plus Partner Sandbox stores and trial plans, but development and trial stores get no public environment, so deployment URLs always require a store login. It doesn't apply to Shopify's Agentic plans either.

## Running the storefront on your own VPS: what SixtyNet sells

If you want root on a Linux box, a Node runtime you control and ports you can open yourself, SixtyNet sells KVM virtual servers out of Los Angeles. Its catalog splits into two families, and the split is the single most important thing to understand before buying for a storefront.

The residential-IP plans carry AT&T Mobility residential addresses, aimed at ad verification, scraping and social account work. Their product pages state plainly that ports 80 and 443 are closed for IP risk-control reasons. That rules them out for a public storefront: no standard HTTP or HTTPS, so no shopper can reach your Next.js or Hydrogen app. At the time of writing those plans also displayed zero availability.

The LAX series is the web-capable side of the catalog, with DDoS protection and no port restriction mentioned.

| Plan | Core specs | Price and billing | Best for | Order |
| --- | --- | --- | --- | --- |
| Residential-STARTER | 2 cores, 2 GB RAM, 40 GB NVMe, 5 TB traffic, residential IP | $25/mo, monthly | IP-sensitive tasks, not public sites | [View plan](https://sixtynet.net/aff.php?aff=39) |
| Residential-MICRO | 4 cores, 4 GB RAM, 80 GB NVMe, 10 TB traffic, residential IP | $45/mo, monthly | Scraping and multi-account work | [View plan](https://sixtynet.net/aff.php?aff=39) |
| Residential-MEDIUM | 8 cores, 8 GB RAM, 160 GB NVMe, 20 TB traffic, residential IP | $85/mo, monthly | Heavier residential workloads | [View plan](https://sixtynet.net/aff.php?aff=39) |
| Residential-LARGE | 16 cores, 16 GB RAM, 320 GB NVMe, 40 TB traffic, residential IP | $165/mo, monthly | Large residential fleets | [View plan](https://sixtynet.net/aff.php?aff=39) |
| LAX-STARTER | 2 cores, 2 GB RAM, 40 GB NVMe, 5 TB traffic, DDoS protection | $200/year (list $240, 17% off annual) | Small SSR storefront | [View plan](https://sixtynet.net/aff.php?aff=39) |
| LAX-MICRO | 4 cores, 4 GB RAM, 80 GB NVMe, 10 TB traffic, DDoS protection | $400/year (list $480) | Storefront plus CMS or database on one box | [View plan](https://sixtynet.net/aff.php?aff=39) |
| LAX-MEDIUM | 8 cores, 8 GB RAM, 160 GB NVMe, 20 TB traffic, DDoS protection | $800/year (list $960) | Multiple storefronts or heavy rendering | [View plan](https://sixtynet.net/aff.php?aff=39) |
| LAX-LARGE | 8 cores, 16 GB RAM, 320 GB NVMe, 40 TB traffic, DDoS protection | $1,600/year (list $1,920) | Highest sustained load in the lineup | [View plan](https://sixtynet.net/aff.php?aff=39) |

The catalog also lists a China Unicom premium-network option at $2,500 a year, down from $3,000, built around the AS9929 route with 95th-percentile billing, plus a storage-heavy plan with 300 GB and a free 200 GB RAID 10 volume. Read those listings directly for their full spec sheets.

Here's the arithmetic worth doing before you buy. A residential STARTER at $25 a month costs $300 a year and can't serve a web page. The LAX-STARTER covers the same 2 cores and 2 GB, allows ports 80 and 443, adds DDoS protection, and runs $200 a year on the annual price. Choosing the residential plan for a storefront is both more expensive and less capable.

Sizing follows the app, not the catalog. A single Hydrogen or Next.js server process for a small catalog fits comfortably in 2 GB. Move to 4 GB if you're also running a CMS, a database or a build step on the same machine. The 8-core plans make sense when one box serves several storefronts or you're rendering pages with heavy data fetching. 👉 [Check the LAX plan specs and current availability](https://sixtynet.net/aff.php?aff=39)

## Self-hosting checklist: the three things that break first

**The private token ends up in the browser.** Shopify issues two Storefront API tokens. The public one is visible to buyers and belongs in browser or mobile code. The private token is a secret and must never appear in client-side code, a mobile app, or behind a browser prefix like `NEXT_PUBLIC_`. If it leaks, rotating creates new credentials but the old ones stay valid until you explicitly revoke them.

**Tokenless access gets mistaken for full access.** You can query products, collections, search and the cart without any token, subject to a query complexity limit of 1,000. Customer accounts are outside that scope and need Customer Account API credentials, with a client ID and a secret for server-side clients.

**Product URLs and cart links break.** Shopify expects product pages at `/products/:handle`. If your storefront uses another structure, serve a 3XX redirect from the expected path to the real page. Your storefront should also handle cart permalinks so shared links and checkout handoffs work; Hydrogen covers this with request handlers, other frameworks need a route.

One channel detail catches people out too. Self-hosted storefronts get keys from the Headless channel, which covers mobile apps and Hydrogen storefronts deployed anywhere other than Oxygen, including Vercel. The Hydrogen channel is for Hydrogen on Oxygen. Within the Headless channel, each storefront has its own tokens but every storefront shares the same API permissions, so a permission change affects all of them.

If you're following a tutorial, check its version. Shopify's legacy Hydrogen docs cover versions 2026-04 and earlier, describe Hydrogen as a React Router app, and point readers on 2026-10 and later to newer documentation with a migration guide. Current docs describe Hydrogen as a toolkit for building a full storefront in a JavaScript framework such as Next.js or React Router. Old setup instructions and current ones don't match.

## What headless really costs to run

Shopify states it directly: a headless storefront costs more to build and run than a theme. The extra spend isn't the hosting line. It's the developer who maintains the frontend after launch, the upgrade work when your framework or Hydrogen changes versions, and the debugging time when a checkout handoff misbehaves on a Friday.

Put the options on the same footing. Oxygen adds no hosting charge to a paid Shopify plan, but it doesn't exist on Agentic plans and gives development and trial stores no public environment. A VPS runs $200 to $1,600 a year at SixtyNet on annual billing, and that number excludes your time on patching, TLS certificates, backups and monitoring. A managed platform bills monthly against usage and takes the server maintenance away, at a higher per-unit cost than a VPS.

## The other hosting options, and who they suit

The choice mostly follows your framework and how much ops you want:

- **Vercel** is built around Next.js and optimizes its native features, so it fits a Next storefront with little configuration.
- **Netlify** handles Next, Astro and Remix well and works as a general-purpose option when you're not committed to one framework.
- **Cloudflare Pages** suits statically generated storefronts.
- **AWS** works either as a purpose-built environment or, for pure static generation with client-side rendering, as S3 storage.
- **DigitalOcean** offers low-cost cloud servers that handle both production and development fine.
- **Railway** is newer and still a little scrappy, better for prototyping than for a storefront you intend to leave alone.
- **Self-hosting on your own server** is viable with the right configuration, but hard to recommend without in-house resources or an agency partner who'll maintain the environment long term.

## FAQ

**Is Oxygen free?**
It adds no charge on top of a paid Shopify plan, covering Starter, Basic, Grow, Advanced, Plus and Pause and build. It isn't available on Agentic plans, and development stores and trial plans use it without a public environment.

**Can I deploy a Hydrogen storefront somewhere other than Oxygen?**
Yes. Shopify supports self-hosting Hydrogen, and storefronts deployed to platforms like Vercel get their keys from the Headless channel rather than the Hydrogen channel.

**Do I need a Storefront API token?**
Not for products, collections, search and the cart. Tokenless access covers those with a query complexity limit of 1,000. Customer accounts and higher query volumes need tokens.

**Can a residential-IP VPS host my storefront?**
Not at SixtyNet. Those plans close ports 80 and 443 by policy, so the site can't be reached over standard HTTP or HTTPS. Use the LAX DDoS-protected plans instead.

**Do I have to use Hydrogen to go headless?**
No. You can call the Storefront API and Customer Account API directly from any language or stack, build a native mobile app with Checkout Kit, or add commerce to an existing site with Storefront Web Components.

## Which option fits your setup

Use the rules in order:

- **Hydrogen is your stack and you'd rather not run servers:** Oxygen. It's in your plan, tokens are injected for you, and the limits only bite if your bundle or request times grow.
- **You already have a Next.js app or your own stack:** a Node-capable host. A VPS at $200 to $400 a year covers a small storefront; a managed platform costs more and removes maintenance.
- **You want a storefront plus CMS or database on one machine:** the 4-core, 4 GB tier is the sensible starting point, not the 2 GB one.
- **You only need buy buttons on an existing site:** Storefront Web Components or Buy Button JS. No headless build, no host decision.
- **You need residential IPs for scraping or account work:** that's a different purchase from storefront hosting, and SixtyNet's residential line is built for it.

Before you pay, confirm four things: that the plan you picked allows inbound traffic on 80 and 443, whether the price you're looking at is monthly or annual (SixtyNet's LAX annual prices carry a 17% discount over the listed rate), whether stock is actually available, and what the refund terms on the checkout page say. Then check that whoever maintains the frontend after launch is still around, because that's the cost that outlives the server bill.

👉 [Start with SixtyNet's VPS catalog and pick a plan that can actually serve a storefront](https://sixtynet.net/aff.php?aff=39)
