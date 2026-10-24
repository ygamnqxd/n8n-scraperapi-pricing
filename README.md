# web scraping automation n8n: how to build scraping workflows that don't break, with real pricing and credit math

Setting up web scraping automation in n8n usually starts the same way: you add an HTTP Request node, point it at a URL, and watch it return a CAPTCHA page, half-loaded HTML, or nothing at all. The fix is to let a scraping API handle proxies, bot walls, and JavaScript rendering while n8n handles triggers, logic, and destinations. ScraperAPI is one of the few scraping services that ships a native n8n node plus a hosted MCP server, so the integration takes minutes — and its pricing page is unusually honest about what a request actually costs, which matters more than the sticker price once you're scraping at volume.

Here's the short version before the details:

> **Most readers:** start with the 7-day free trial (5,000 API credits, no credit card), then pick **Hobby ($49/mo)** for small projects or **Business ($299/mo)** if you're monitoring e-commerce or SERPs in production. Annual billing takes 10% off every plan. Failed requests don't cost credits, so testing a workflow is essentially free.
>
> 👉 [See ScraperAPI's full pricing and start the free trial](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## Why your n8n HTTP Request node keeps failing on real websites

The HTTP Request node sends a plain request, and bot-protected sites recognize it instantly. What comes back is a block page, a CAPTCHA, or an empty response — and here's the part that bites people in production: the failure is silent. Your downstream nodes keep running on empty data, so your Google Sheet fills with blanks and your dashboard quietly goes stale. ScraperAPI's own docs describe this exact failure mode, and it's the reason "works on example.com, dies on Amazon" is such a common n8n scraping experience.

A scraping API node changes where the hard work happens. The ScraperAPI node routes each request through a rotating pool of 40M+ proxies, handles CAPTCHA and anti-bot bypass, and renders JavaScript when needed — all before the data reaches your workflow. And when a page genuinely can't be reached, it returns a clear error instead of empty data, so your error branch actually triggers. ScraperAPI only charges for successful requests, which matters a lot when you're iterating on a new workflow.

There's a workflow-structure lesson worth stealing from other scraping tutorials (ScrapingBot's n8n guide documents it well): batch 5–10 URLs at a time with a short wait between groups, check the real HTTP status code instead of assuming success, and route persistent failures to a logging branch rather than letting them vanish. Those patterns apply regardless of which scraping API you attach.

## How to connect ScraperAPI to n8n (three ways)

The official route takes about two minutes:

1. Sign up and copy your API key from the ScraperAPI dashboard.
2. In n8n, click **+** on the canvas, search for **ScraperAPI**, and add the node.
3. Create a new credential, paste the API key, and save — n8n tests it automatically.
4. Select the **API** resource, enter your target URL, set optional parameters, and execute.

That's the whole setup. The node works on both n8n Cloud and self-hosted instances, and there's no proxy list, retry logic, or headless browser to maintain on your side. If you'd rather not build from scratch, ScraperAPI publishes pre-built n8n templates, including daily price-drop digests for Amazon, Walmart, and Google that you can import and adapt.

The second option is the AI Agent path, which is genuinely useful for chat-driven scraping: add a Chat Message Received trigger, connect an AI Agent node, wire an AI Chat Model (OpenAI or similar) into the Chat Model input, add Simple Memory, and connect the ScraperAPI node as the Agent's tool. With a system prompt describing the task, the agent decides when and what to scrape.

The third option arrived in early 2026: ScraperAPI runs a hosted MCP server that plugs into n8n's MCP Client Tool. You point it at the endpoint `https://mcp.scraperapi.com/mcp`, set the transport to HTTP Streamable, and use your ScraperAPI key as the Bearer token. Any MCP-compatible client can then pull Amazon and Walmart prices, SERP results, or full page content. There's also a self-hosted MCP server on GitHub if you want it on your own infrastructure.

### Parameters worth knowing before your first workflow

Most requests need nothing beyond the URL, but a few parameters solve specific problems:

- `COUNTRY_CODE` — two-letter ISO code (US, GB, DE) for geo-targeted results.
- `RENDER` — enables JavaScript rendering. It slows requests down, so only set it when the target needs it.
- `PREMIUM` / `ULTRA_PREMIUM` — higher-grade residential proxies for stubborn sites. They're mutually exclusive, and both cost more credits.
- `AUTOPARSE` — auto-returns parsed JSON for supported sites at no extra credit cost.
- `SESSION_NUMBER` — keeps the same proxy session across requests (can't combine with Premium options).
- `KEEP_HEADERS` — sends your own headers/cookies, currently only through the HTTP Request node, and not compatible with Ultra Premium.

One detail that trips people up: custom headers only work through a plain HTTP Request node pointed at ScraperAPI's endpoint, not through the dedicated node. If your target requires cookies or specific headers, plan for that.

## ScraperAPI plans and pricing: the full table

Every paid plan includes the same core feature set — JavaScript rendering, premium residential and mobile IPs, CAPTCHA and anti-bot bypass, structured data endpoints, DataPipeline, crawler access, custom headers and sessions, automatic retries, unlimited bandwidth, a 99.9% uptime guarantee, and Slack support. What changes is volume, concurrency, and geo-targeting depth.

Annual billing knocks 10% off across the board. The table shows both prices per month.

| Plan | Credits/mo | Concurrent requests | Monthly | Annual (per mo) | Get started |
| --- | --- | --- | --- | --- | --- |
| Hobby | 100,000 | 20 | $49 | $44.10 | [Start Hobby](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | 1,000,000 | 50 | $149 | $134.10 | [Start Startup](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | 3,000,000 | 100 | $299 | $269.10 | [Start Business](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | 5,000,000 | 200 | $475 | $427.50 | [Start Scaling](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | 10,500,000 | 300 | $975 | $877.50 | [Start Professional](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | 21,500,000 | 500 | $1,975 | $1,777.50 | [Start Advanced](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 500+ | Custom | Custom | [Talk to sales](https://www.scraperapi.com/?fp_ref=coupons) |

Hobby and Startup are limited to US & EU geo-targeting and a 30-day analytics window; Business and up get country-level targeting and unlimited analytics history. Exceed your concurrency limit and you get a 429 — requests are rejected, not queued, and you're not charged for the rejected ones. Just slow your workflow down to your plan's concurrency and retry.

If you want to try before paying, there are two doors: sign up and get **1,000 free credits** immediately (capped at 5 concurrent connections), or start the **7-day trial with 5,000 credits** and no credit card required. Either is enough to build and test a real n8n workflow end to end.

👉 [Compare every plan and claim the free trial](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## What one credit actually buys — the number most reviews skip

Here's the math that determines whether a plan fits your workload, and it's buried in the pricing FAQ rather than on the plan cards. Credits are per request, but not all requests cost the same:

| Target | Credits per request |
| --- | --- |
| Standard page | 1 |
| Amazon | 5 |
| Google or Bing (incl. subdomains) | 25 |
| LinkedIn | 30 |
| Sites behind Cloudflare, Datadome, or PerimeterX (when bypassed) | +10 on top |

So a 100,000-credit Hobby plan is 100,000 generic pages — but only 20,000 Amazon product pages, or 4,000 Google searches, before premium proxy surcharges. That difference completely changes which plan you need, and it's the single most common source of "why did my credits vanish" surprises. ScraperAPI gives you a Domain Cost Estimator in the dashboard so you can check any URL's exact cost, plus a `max_cost` parameter to cap what a single request can burn. Set it. One runaway scrape against a heavily protected site shouldn't eat a week of quota.

Competitors handle this differently. ScrapingBot charges 1 credit for a standard request, 10 for headless Chrome rendering, 10 for premium proxies, and 20 when you combine both — so on that service, a JS-rendered residential-IP scrape of an Amazon page runs 20–25 credits. The structures differ enough that comparing plans purely on headline credit counts will mislead you; always map the credit cost onto your actual targets.

## When credits run out: three ways it can go

By default, scraping pauses when your credits hit zero until the next renewal. That's fine for batch jobs, annoying for continuous monitoring. ScraperAPI offers two opt-in continuations, and knowing which applies to your plan matters:

- **Auto-upgrade** — on Hobby through Professional, your account can automatically move to the next plan when credits run out. It's opt-in, and upgrades generally mean a better per-credit price.
- **Pay-as-you-go** — on Scaling and above, you can keep scraping past your limit at a fixed per-credit rate, capped by a monthly spending limit you set yourself.

Annual plans can also renew early for a fresh batch of credits. If you're running a scheduled price monitor that must not skip days, decide which of these you want before it matters — auto-upgrade on a low plan can quietly triple your bill if your volume estimates were off.

## How ScraperAPI compares with other n8n scraping options

Since you're probably comparing several before committing, here's what the verified landscape looks like — links deliberately omitted for competitors, information only.

**Firecrawl** also has a native n8n integration with a sync/async agent model: the sync mode waits up to 600 seconds for a crawl job, while async mode returns a job ID you poll with a second node. It returns markdown or structured output and gives free signup credits. It's a solid pick when your goal is LLM-ready content extraction from arbitrary pages. Where it differs: ScraperAPI's strength is high-volume structured e-commerce and SERP data (Amazon, Walmart, Google, Redfin come back as pre-parsed JSON fields), plus granular credit-cost control.

**ScrapingBot** works through n8n's generic HTTP Request node with Basic Auth — no dedicated node — and its free tier gives 1,000 credits monthly, no card required. The tutorial trade-off is real: you must set Response Format to Text, remember that `proxyCountry` silently does nothing without `premiumProxy=true`, and write your own retry logic. More assembly required, lower entry price.

**Browse AI** takes a different shape entirely: point-and-click robots that run on Browse AI's own platform and schedule. If your automation already lives in n8n, that split is awkward — your triggers and logic sit in one tool while the scraping runs elsewhere. The ScraperAPI node keeps the whole workflow in n8n, which is exactly why the native node exists.

None of these is universally "better." If you want markdown for AI pipelines, Firecrawl fits well. If you want the cheapest possible entry to test an idea, ScrapingBot's free monthly credits are hard to beat. If your workflows are e-commerce prices, SERPs, and real-estate data flowing into Sheets, Slack, Airtable, or a database, the pre-parsed JSON is where ScraperAPI earns its keep — your next n8n node reads fields directly instead of untangling HTML.

## A concrete starter workflow: scheduled price monitoring

To make this tangible, here's the shape of the workflow most people build first:

1. **Schedule Trigger** — run daily at a fixed time.
2. **Read target URLs** from Google Sheets or a database node.
3. **Loop Over Items** (Split in Batches), 5–10 URLs per batch.
4. **ScraperAPI node** with the structured data endpoint for Amazon or Walmart — prices and product fields arrive as JSON, no HTML parsing step.
5. **Write results** to Sheets, Airtable, or Postgres, and **Slack yourself** when a price drops below a threshold.

Total credits per run: 5 per Amazon URL, so 100 products daily is 500 credits/day — roughly 15,000 a month, comfortably inside Hobby. The same skeleton works for SERP tracking (at 25 credits per search, budget accordingly) or job-listing collection fed into a CRM.

## FAQ

**Does the n8n HTTP Request node work for scraping at all?**
For unprotected, static pages, yes. For anything behind Cloudflare or requiring JavaScript, it returns blocks or empty HTML and fails silently. Use a scraping API node or point an HTTP Request node at a scraping API endpoint instead.

**How much does n8n web scraping cost with ScraperAPI?**
The free tier gives 1,000 credits (5 concurrent), the 7-day trial gives 5,000, and paid plans start at $49/month for 100,000 credits. Remember the multipliers: Amazon pages cost 5 credits, Google searches 25, and bypassed anti-bot pages add 10.

**Do failed requests cost credits?**
No — ScraperAPI charges only for successful requests and returns a clear error when a page can't be reached. Combined with the free trial, iterating on a workflow costs nothing.

**n8n Cloud or self-hosted — does it matter for the ScraperAPI node?**
Both work identically. Cloud is the fastest start; self-hosted via Docker gives you full control and keeps execution data on your own server.

## Picking a plan before you click buy

Match the plan to your credit math, not your ambitions: casual projects and testing → Hobby at $49 ($44.10 annual); steady e-commerce or SERP monitoring in production → Business at $299, which is where country-level geo-targeting and unlimited analytics kick in; continuous high-volume pipelines → Scaling or above, where pay-as-you-go and the bigger concurrency pools justify the jump. Skip annual billing until you've run the trial and confirmed your real per-day credit burn with the Domain Cost Estimator — then switch, since the 10% discount applies site-wide.

One last check before subscribing: list your actual target domains, multiply each by its credit cost, and confirm the resulting monthly total fits the plan with 30–50% headroom. If the numbers work, 👉 [head to ScraperAPI's pricing page and start your free trial](https://www.scraperapi.com/pricing/?fp_ref=coupons).
