# Web Scraping API for Node.js: Which One Actually Works at Scale — ScraperAPI Setup Guide, Plan Breakdown, and How to Stop Getting Blocked (Includes Axios, Puppeteer & Playwright Integration)

If you've ever tried to scrape a real website with vanilla Node.js and hit a wall of 403s, CAPTCHAs, and mysteriously empty responses within the first ten minutes, you already know the problem. Building a web scraper in Node.js is easy. *Keeping* it running is something else entirely.

This guide walks through what a web scraping API actually solves for Node.js developers, how to integrate ScraperAPI with the three most popular Node.js scraping approaches (Axios, Puppeteer, and Playwright), and which pricing plan makes sense for what kind of project — with real numbers, not marketing language.

---

**Why Node.js Developers Need a Scraping API (Not Just Libraries)**

Node.js has a genuinely strong ecosystem for web scraping. Axios fetches HTML in a few lines. Cheerio parses it like jQuery. Puppeteer and Playwright spin up real browser environments that execute JavaScript. For a lot of tasks, that stack works great.

The problem shows up at scale or when your targets get serious about protecting their data.

When you hit a site that uses Cloudflare, DataDome, or PerimeterX bot protection, a plain Axios request gets rejected before it ever returns a response. Puppeteer can handle browser fingerprinting to a degree, but maintaining a working headless browser configuration against actively-updated bot detection is a part-time job. Residential proxy rotation, CAPTCHA solving, retry logic, geotargeting — all of that has to be built and maintained if you're rolling your own infrastructure.

A web scraping API for Node.js handles that infrastructure layer so you don't have to. You send a URL, the API routes it through rotating residential proxies, renders JavaScript if needed, solves CAPTCHAs, and returns clean HTML. Your Node.js code stays simple and doesn't break every time a target site updates its bot detection.

**ScraperAPI** is one of the most widely used options in this category. It routes requests through a pool of 40+ million IPs across 50+ countries, integrates directly with Axios as a simple URL parameter swap, and works as a proxy server for Puppeteer and Playwright with no library rewrites. That's the practical draw for Node.js developers specifically — your existing scraping code stays largely intact.

---

**How ScraperAPI Works with Node.js: The Actual Integration**

The integration model is straightforward, and that's what most developers mention first when recommending it. There's no new SDK to learn, no restructuring your existing code. You're adding one parameter (or pointing at a proxy server) and the infrastructure complexity moves off your plate.

**Integrating with Axios**

If you're using Axios to fetch pages, the integration is a one-line change. Instead of hitting the target URL directly, you send your request to ScraperAPI's endpoint with your target URL and API key as parameters:

javascript
const axios = require('axios');
const cheerio = require('cheerio');

const API_KEY = 'YOUR_API_KEY';
const targetUrl = 'https://example.com/products';

axios('http://api.scraperapi.com/', {
  params: {
    api_key: API_KEY,
    url: targetUrl,
    render: true  // Set to true for JavaScript-rendered pages
  }
}).then(response => {
  const html = response.data;
  const $ = cheerio.load(html);
  // Your Cheerio parsing logic here
  console.log($('h1').text());
}).catch(console.error);


The `render: true` parameter tells ScraperAPI to run a headless Chromium browser and execute the page's JavaScript before returning the HTML. This costs extra credits (more on that in the pricing section), but it means you can scrape JavaScript-heavy SPAs with an HTTP client instead of spinning up Puppeteer.

**Integrating with Puppeteer**

When you need browser interactions — scrolling, clicking buttons, waiting for elements — you still need Puppeteer or Playwright. In that case, ScraperAPI works as a proxy server that handles IP rotation and bot detection while you control the browser:

javascript
const cheerio = require('cheerio');
const puppeteer = require('puppeteer');

const PROXY_USERNAME = 'scraperapi';
const PROXY_PASSWORD = 'YOUR_API_KEY';
const PROXY_SERVER = 'proxy-server.scraperapi.com';
const PROXY_PORT = '8001';

const scrapedData = [];

(async () => {
  const browser = await puppeteer.launch({
    ignoreHTTPSErrors: true,
    args: [`--proxy-server=http://${PROXY_SERVER}:${PROXY_PORT}`]
  });
  const page = await browser.newPage();
  await page.authenticate({
    username: PROXY_USERNAME,
    password: PROXY_PASSWORD,
  });

  try {
    await page.goto('https://example.com', { timeout: 180000 });
    const bodyHTML = await page.evaluate(() => document.body.innerHTML);
    const $ = cheerio.load(bodyHTML);
    // Your parsing logic here
  } catch (err) {
    console.log(err);
  }

  await browser.close();
  console.log(scrapedData);
})();


**Integrating with Playwright**

Playwright's integration follows the same proxy pattern, with the authentication step moved into the browser context:

javascript
const cheerio = require('cheerio');
const { chromium } = require('playwright');

const PROXY_SERVER = 'proxy-server.scraperapi.com';
const PROXY_PORT = '8001';
const PROXY_USERNAME = 'scraperapi';
const PROXY_PASSWORD = 'YOUR_API_KEY';

const scrapedData = [];

(async () => {
  const browser = await chromium.launch({
    args: [
      `--proxy-server=http://${PROXY_SERVER}:${PROXY_PORT}`,
      '--ignore-certificate-errors'
    ]
  });

  const context = await browser.newContext({
    httpCredentials: {
      username: PROXY_USERNAME,
      password: PROXY_PASSWORD
    }
  });

  const page = await context.newPage();

  try {
    await page.goto('https://example.com');
    const bodyHTML = await page.content();
    const $ = cheerio.load(bodyHTML);
    // Your parsing logic here
  } catch (err) {
    console.log(err);
  }

  await browser.close();
  console.log(scrapedData);
})();


Both Puppeteer and Playwright setups route all browser traffic through ScraperAPI's proxy network, which means IP rotation, geotargeting, and bot detection handling happen at the infrastructure level while you keep full browser control for the interaction logic.

---

**Understanding the Credit System Before You Pick a Plan**

Before getting into the plan comparison table, one thing is worth understanding clearly: **not all requests cost the same number of credits**, and this is where a lot of people end up surprised by their actual costs.

Every plan gives you a monthly credit bucket, but what you can actually do with those credits depends heavily on what sites you're scraping:

| Request Type | Credits Per Request |
|---|---|
| Standard unprotected page | 1 credit |
| E-commerce sites (Amazon, eBay, etc.) | 5 credits |
| Search engines (Google, Bing) | 25 credits |
| LinkedIn | 30 credits |
| JavaScript rendering (`render=true`) | +10 credits |
| Premium proxies (`premium=true`) | +10 credits |
| Ultra-premium proxies (`ultra_premium=true`) | +30 credits |
| Screenshot capture | +10 credits |

So if you're scraping Amazon product pages with JavaScript rendering enabled, each successful request costs 15 credits (5 for Amazon + 10 for rendering). On the Hobby plan's 100,000 monthly credits, that gets you roughly 6,600 successful Amazon scrapes — not 100,000. Plan accordingly.

The credit system does have one genuinely fair feature: **you're only charged for successful requests**. Failed attempts, blocked responses (anything other than 200 or 404), and timeouts don't burn your credits. That's not universal in this category.

> **Tip**: Before committing to a plan, use the Domain Cost Estimator in the ScraperAPI dashboard to check what your actual per-request cost will be on your specific targets.

---

**ScraperAPI Plan Comparison: Full Breakdown**

ScraperAPI offers a free tier, a 7-day trial, and eight paid tiers. Here's the complete picture — every plan that's currently available, including the newer Growth plans added in mid-2026:

| Plan | Monthly Price | Annual Price (10% off) | Credits/Month | Concurrent Threads | Geotargeting | Purchase |
|---|---|---|---|---|---|---|
| **Free** | $0 (permanent) | — | 1,000 | 5 | Limited |  [Start Free](https://www.scraperapi.com/?fp_ref=coupons) |
| **Free Trial** | $0 (7 days) | — | 5,000 (one-time) | 5 | Limited |  [Start Free Trial](https://www.scraperapi.com/?fp_ref=coupons) |
| **Hobby** | $49/mo | $44.10/mo | 100,000 | 20 | US & EU only |  [Get Hobby Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Startup** | $149/mo | $134.10/mo | 1,000,000 | 50 | US & EU only |  [Get Startup Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Business** | $299/mo | $269.10/mo | 3,000,000 | 100 | Global |  [Get Business Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Scaling** *(Most Popular)* | $475/mo | $427.50/mo | 5,000,000 | 200 | Global + PAYG |  [Get Scaling Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Professional** *(Growth)* | $975/mo | $877.50/mo | 10,500,000 (+250K bonus) | 300 | Global + PAYG |  [Get Professional Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Advanced** *(Growth)* | $1,975/mo | $1,777.50/mo | 21,500,000 (+500K bonus) | 500 | Global + PAYG |  [Get Advanced Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Enterprise** | Custom | Custom | 22M+ | 500+ | Global + PAYG |  [Contact Sales](https://www.scraperapi.com/?fp_ref=coupons) |

**A few things that aren't obvious from the table:**

- **Geotargeting is gated.** Hobby and Startup plans can only use US and EU proxies. If your project needs country-level targeting anywhere else in the world — Southeast Asia, Latin America, specific EU countries — you need Business tier or above.
- **Pay-as-you-go only starts at Scaling.** On Hobby, Startup, and Business, hitting your credit limit mid-month means you're either upgrading to the next tier or talking to support. There's no overflow option below Scaling.
- **Credits don't roll over.** Your credit balance resets at renewal. There's no carrying forward unused credits, so it's worth matching your plan size to your actual monthly volume.
- **The Growth plans (Professional and Advanced) include limited-time bonus credits** — 250K for Professional and 500K for Advanced — on top of the standard monthly allocation. These bonus credits were introduced in May 2026 when ScraperAPI launched the Growth pricing tier for teams who had outgrown the standard Scaling plan.
- **Annual billing saves 10%** automatically — no coupon code needed. Some third-party coupon sites also list active promo codes (such as DATALOVER or ANWAR10 for 10% off) that can apply to monthly billing instead.

---

**Which Plan Should You Actually Pick?**

The honest answer is: it depends on what you're scraping, not just how much. Here's a practical breakdown based on typical Node.js scraping project types:

**Hobby ($49/mo)** makes sense if you're building a personal project, testing an idea, or running a small-scale price monitor on a handful of standard websites. If your targets are mostly plain HTML pages without aggressive bot protection, 100,000 credits goes a long way. If you're hitting Amazon or Google regularly, do the math first with the credit multipliers in mind.

**Startup ($149/mo)** is the right tier for a small product or agency running consistent scraping jobs across several clients. The jump from 100K to 1M credits is significant, and 50 concurrent threads lets you parallelize properly. Still limited to US/EU geotargeting, though.

**Business ($299/mo)** is where you land when you need global geotargeting, unlimited analytics history in the dashboard, or a higher thread count for larger parallel jobs. This is also the first tier with enough headroom to run production infrastructure that other parts of a business depend on.

**Scaling ($475/mo)** adds pay-as-you-go overflow, which is valuable if your scraping volume spikes unpredictably. Not being hard-capped mid-month matters a lot once you're running production data pipelines.

**Professional and Advanced Growth plans ($975–$1,975/mo)** are designed for teams that have already maxed out the Scaling plan and need a self-service upgrade path without negotiating an enterprise contract. The May 2026 addition of these tiers filled a real gap for customers who needed more than 5M credits per month but weren't ready for fully custom enterprise pricing.

**Enterprise (custom)** is for organizations with requirements that don't fit standard tiers — custom credit volumes, SLA guarantees, dedicated support, or specific compliance needs.

---

**What the Benchmarks Actually Say**

ScraperAPI has been independently benchmarked by Scrapeway, which runs automated weekly tests across 12 target websites and 8 competing APIs. In the most recent results (July 2026):

- **Overall success rate**: 62% (vs. industry average of ~60%)
- **Average response time**: 4.9 seconds (well below the 11.2-second industry average)
- **Cost per 1,000 requests**: ~$3.24 (slightly below the $3.40 industry average)
- **Ranking among 8 tested APIs**: #4 overall

The per-target breakdown is more useful than the aggregate number. ScraperAPI performs strongest on mainstream e-commerce and job sites: 94% success on Amazon, 94% on LinkedIn, 97% on Etsy, 96% on StockX. Performance drops significantly on certain targets — booking.com, Twitter, and Instagram returned 0% success in the same benchmark period.

On review platforms, ScraperAPI holds a 4.5/5 on Trustpilot and 4.4/5 on G2. The consistent praise in user reviews focuses on how little code change is required to integrate it, responsive support, and clean documentation. The consistent criticism is about the credit multiplier system being easy to underestimate before you've run real tests on your actual targets.

The practical takeaway: ScraperAPI is a strong default choice for scraping mainstream e-commerce, real estate, and job sites from Node.js. It's not the answer for every target — no single scraping API is — but for the use cases that come up most often in Node.js projects, it performs reliably and integrates cleanly.

---

**Features Worth Knowing About Beyond the Basics**

Most guides stop at "it rotates proxies and handles CAPTCHAs," but a few ScraperAPI features are specifically relevant for Node.js workflows:

**Async Scraper Mode** — For large batch jobs, you can submit thousands of URLs asynchronously and poll for results instead of waiting synchronously for each request. This makes a significant difference when you're processing large URL lists and need to manage concurrency without blocking.

**DataPipeline** — ScraperAPI's built-in scheduler lets you set up recurring scraping jobs that run on a defined schedule and deliver results to an endpoint you specify. For Node.js projects where you'd otherwise be managing cron jobs and result storage yourself, this removes a chunk of operational overhead.

**Structured Data / Autoparse** — For a set of well-supported domains (Amazon, Google, Walmart), ScraperAPI can return structured JSON directly instead of raw HTML. This means less Cheerio parsing on your end and cleaner data straight out of the response.

**Sessions** — For scraping workflows where you need to maintain a consistent IP across multiple requests (logging in, navigating paginated results as a single user), ScraperAPI supports persistent sessions that hold the same IP for up to 15 minutes.

**Webhook support** — Results from async jobs can be delivered to a webhook endpoint, which integrates naturally with Node.js API servers.

---

**The Free Trial: A Practical Starting Point**

ScraperAPI offers a 7-day free trial with 5,000 credits — no credit card required. That's enough to run meaningful tests against your actual scraping targets, not just toy examples.

The permanent free plan gives you 1,000 credits per month with 5 concurrent connections, which is genuinely usable for small-scale monitoring or low-volume projects.

The most valuable use of the free trial is running your actual target URLs through the API and watching what your real credit consumption looks like in the dashboard before choosing a paid plan. The difference between "I need 100K credits a month" and "I actually need 500K credits a month" becomes clear pretty quickly once you factor in the multipliers for your specific targets.

👉 [Start your ScraperAPI free trial — 5,000 credits, no credit card needed](https://www.scraperapi.com/?fp_ref=coupons)

---

**How ScraperAPI Compares to Other Options for Node.js**

The web scraping API market in 2026 has a few clear positioning differences worth knowing:

**ScrapingBee** ($49/mo entry) — Similar developer experience and entry price, slightly more beginner-friendly documentation. The credit multiplier system is comparable to ScraperAPI's, making costs similarly hard to predict at scale. No official Node.js SDK.

**Bright Data** ($499/mo entry) — The enterprise-grade option with 150+ million residential IPs and pre-built scrapers for 120+ platforms. Best success rates on difficult targets, but the price point puts it out of reach for most individual projects.

**Scrape.do** ($29/mo) — The cheapest entry point among established options. Worth considering if budget is the primary constraint and your targets are simpler, though the feature set is lighter.

**ZenRows** ($69.99/mo) — Offers native Puppeteer/Playwright browser automation on cloud infrastructure, which is interesting for complex interaction scripts. Independent benchmarks show below-average success rates in current testing.

**Firecrawl** ($16/mo) — Best option if your pipeline feeds into an LLM and you need Markdown output ready for AI processing, rather than raw HTML for traditional parsing.

For Node.js developers specifically, ScraperAPI's proxy-based integration model means almost no code change from your existing Puppeteer or Playwright scripts. That frictionless integration is usually what tips the choice for developers who have existing scrapers and need to make them more resilient rather than rebuild them.

---

**Getting Started: The Short Version**

1. Sign up at ScraperAPI and grab your API key (5,000 free trial credits, no card required)
2. Run your actual scraping targets through the trial and check credit consumption in the dashboard
3. Pick a plan based on your real monthly credit needs, not the headline credit number
4. Integrate with Axios (URL parameter swap), Puppeteer, or Playwright (proxy server configuration) — all three patterns are covered in the code examples above

The infrastructure problem of web scraping at scale is genuinely boring to solve from scratch. Proxy management, CAPTCHA handling, retry logic, browser fingerprint evasion — none of it is the interesting part of your project. Offloading it to a web scraping API for Node.js lets you spend your time on the actual data logic instead.

👉 [Try ScraperAPI free — 5,000 trial credits, no credit card required](https://www.scraperapi.com/?fp_ref=coupons)

---

**Frequently Asked Questions**

**Does one API request always consume one credit?**
No. The base rate is 1 credit for a standard page, but the actual cost depends on the target domain and parameters you use. Amazon costs 5x, Google costs 25x, and adding JavaScript rendering adds another 10 credits per request. Check your specific targets with the dashboard's cost estimator before choosing a plan.

**Can I use ScraperAPI with my existing Puppeteer scripts without rewriting them?**
Yes. The integration uses ScraperAPI as a proxy server — you add proxy configuration and authentication to your Puppeteer launch options, and your page interaction scripts stay unchanged.

**What happens if I run out of credits mid-month?**
On Hobby, Startup, and Business plans, you can upgrade to the next tier or contact support. On Scaling, Professional, Advanced, and Enterprise plans, pay-as-you-go kicks in automatically (with an optional spending cap) so you're never hard-stopped.

**Is there a refund policy?**
ScraperAPI offers a 7-day no-questions-asked refund. Contact support and it's handled.

**Do unused credits carry over?**
No. Credits reset at the start of each billing cycle.

**Is the 10% annual discount automatic?**
Yes — choosing annual billing at checkout applies the discount automatically. No coupon code needed.
