# selenium scraping: dynamic pages, explicit waits and anti-bot proxies — a practical guide with real numbers

Want to scrape pages that plain HTTP requests never see? Here is the short answer up front: use Selenium when the data only appears after JavaScript renders, or when you need clicks, logins and pagination. For static HTML, Requests plus BeautifulSoup wins on speed and simplicity. And once your Selenium scraper starts getting blocked by Cloudflare or served CAPTCHAs, the problem shifts from "how do I write the code" to "where do my requests exit from" — that is the part this guide covers in detail, with a pricing breakdown most reviews skip.

> **Quick pick**: for most individuals and small teams, ScraperAPI's free plan is the right starting point — 1,000 API credits per month, 5 concurrent connections, plus 5,000 extra requests during your first 7 days, no credit card required. When you outgrow that, Hobby starts at $49/month with 10% off on annual billing. 👉 [Start your 7-day free trial and test your target site first](https://www.scraperapi.com/signup?fp_ref=coupons)

## What Selenium actually solves

Selenium drives a real browser — Chrome or Firefox — and loads pages, runs JavaScript and clicks buttons the way a person would. Many modern pages return an almost empty HTML shell, with the data filled in by scripts; Requests and BeautifulSoup only see the shell, while Selenium sees the rendered page. The contrast is easy to measure: on the JavaScript build of quotes.toscrape.com, Requests finds 0 quote elements while Selenium finds 10.

The trade-off is speed. Selenium is heavier than Requests plus BeautifulSoup: every page load spins up a browser, renders, and executes scripts. So the selection rule is straightforward. If the data is already in the HTML, use lighter tools. Reach for Selenium when the page renders with JavaScript or needs interaction. Playwright also runs JavaScript and has built-in auto-waiting, which saves boilerplate on new projects; if your team already runs Selenium in its test suite, reusing that knowledge outweighs the syntax savings.

| Factor | Selenium | Playwright |
| --- | --- | --- |
| Auto-waiting | Manual (explicit waits) | Built in |
| Language bindings | Python, Java, C#, JS, Ruby, more | Python, Java, C#, JS |
| Driver setup | Auto since 4.6 | Auto (bundled browsers) |
| Best fit | Existing test suites, cross-language teams | New JS-scraping projects |

## Getting started: three mistakes to avoid

Good news first: since Selenium 4.6, driver management is automatic. Run `pip install selenium`, point it at Chrome, and Selenium Manager detects your installed browser, downloads the matching driver and caches it. The old ritual of manually downloading ChromeDriver and matching version numbers is over.

Three pitfalls account for most beginner failures:

1. **Use `find_elements` (plural) instead of `find_element`.** The singular version throws an exception when nothing matches; the plural returns a list, and an empty list simply skips the loop.
2. **Wait for elements instead of using `time.sleep()`.** After `get()` returns, the page keeps loading asynchronously, so grabbing elements immediately often fails. The fix is an explicit wait: `WebDriverWait(driver, 10).until(EC.presence_of_element_located(...))` resumes the moment the element appears. Useful conditions include `presence_of_element_located`, `visibility_of_element_located`, `element_to_be_clickable`, and `text_to_be_present_in_element`.
3. **Call `driver.quit()` when done.** Headless browsers do not clean up after themselves. Skip this and dead Chrome processes pile up on your server fast.

For headless mode, the common flags are `--headless=new`, `--no-sandbox` and `--disable-dev-shm-usage`. Pagination follows a simple loop: scrape the current page, find the "Next" link (`li.next > a` on the practice site), click it, repeat. When the next link disappears on the last page, that absence is your natural stop signal.

## When Selenium meets anti-bot: proxies are the real dividing line

A correct script is only half the battle. What actually kills Selenium scrapers at scale is the IP address. Once a site sits behind Cloudflare or Datadome, your home or datacenter IP gets flagged quickly, and rotating user agents will not save you. What you need is rotating proxy pools combined with browser rendering. ScraperAPI fits that slot: it handles JS rendering, rotates residential and mobile IPs, and bypasses CAPTchas and anti-bot checks, and Selenium can sit directly on top of it.

How you wire it up matters. The official documentation calls out a common mistake that most tutorials never mention:

**The recommended method: Proxy Port mode.** Install `selenium-wire` first — it makes proxy configuration far easier than vanilla Selenium — then point your traffic at ScraperAPI's proxy port:

python
API_KEY = 'YOUR_API_KEY'
proxy_options = {
    'proxy': {
        'http': f'http://scraperapi:{API_KEY}@proxy-server.scraperapi.com:8001',
        'https': f'http://scraperapi:{API_KEY}@proxy-server.scraperapi.com:8001',
        'no_proxy': 'localhost,127.0.0.1'
    }
}
driver = webdriver.Chrome(options=options, seleniumwire_options=proxy_options)


With this in place, every request Selenium makes exits through the proxy port. CSS, images and relative links all load normally, and the rest of your code looks exactly like ordinary Selenium.

**The method the docs explicitly warn against: hitting the API endpoint.** The tempting shortcut is `driver.get("http://api.scraperapi.com/?api_key=xxx&url=xxx")`. The official documentation names two concrete failures. First, relative-path assets get requested from the ScraperAPI endpoint instead of the target site: scraping quotes.toscrape.com turns a request for `/static/main.css` into `api.scraperapi.com/static/main.css`, which errors out, leaving Selenium with only the first HTML response. In the docs' own words, this "kinda defeats the purpose of using a headless browser." Second, clicking relative links to navigate breaks for the same reason. Their conclusion is blunt: they highly recommend not using the API endpoint for Selenium integration. If you want a proxy, use the proxy port. 👉 [Sign up, grab your API key and wire up the proxy port](https://www.scraperapi.com/signup?fp_ref=coupons)

## ScraperAPI plans and pricing, in full

ScraperAPI bills by the number of API requests you make each month — there is no bandwidth-based pricing. Here is the complete plan lineup currently shown on the official pricing page:

| Plan | Credits/month | Concurrent threads | Monthly price | Annual (per month) | Link |
| --- | --- | --- | --- | --- | --- |
| Free plan | 1,000 | 5 | $0 | $0 | [ Sign up free](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Hobby | 100,000 | 20 | $49 | $44.10 | [ See Hobby pricing](https://scraperapi.com/pricing?fp_ref=coupons) |
| Startup | 1,000,000 | 50 | $149 | $134.10 | [ See Startup pricing](https://scraperapi.com/pricing?fp_ref=coupons) |
| Business | 3,000,000 | 100 | $299 | $269.10 | [ See Business pricing](https://scraperapi.com/pricing?fp_ref=coupons) |
| Scaling (most popular) | 5,000,000 | 200 | $475 | $427.50 | [ See Scaling pricing](https://scraperapi.com/pricing?fp_ref=coupons) |
| Professional | 10,500,000 | 300 | $975 | $877.50 | [ See Professional pricing](https://scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | 21,500,000 | 500 | $1,975 | $1,777.50 | [ See Advanced pricing](https://scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 500+ | Custom | Custom | [ Contact sales](https://www.scraperapi.com/?fp_ref=coupons) |

Annual billing takes 10% off across the board — on Hobby that is $4.90 less per month. Every paid plan includes JS rendering, residential and mobile IPs, automatic retries, unlimited bandwidth and a 99.9% uptime guarantee. One limit worth knowing: geotargeting on Hobby and Startup covers US and EU only; country-level geotargeting unlocks from Business upward. 👉 [Check the full plan configurations and current prices](https://scraperapi.com/pricing?fp_ref=coupons)

## The credit consumption rules you must check before buying

This is the part most comparison posts gloss over: one request does not equal one credit. ScraperAPI prices harder targets separately:

- Standard page: 1 credit
- Amazon: 5 credits
- Google and Bing (including subdomains): 25 credits
- LinkedIn: 30 credits
- Sites behind Cloudflare, Datadome or PerimeterX: +10 credits per request when bypassed

Do the math on Hobby's 100,000 credits: pointed entirely at Google search results, that is only 4,000 requests. The per-request cost of a SERP scrape is dramatically higher than a regular page. ScraperAPI puts a Domain Cost Estimator in the dashboard so you can check the exact cost of any URL before committing, and you can set a `max_cost` per scrape to cap any single request. Estimate your target site's cost per request first — it tells you more than the plan price tags do.

One more operational detail: exceeding your concurrent thread limit returns a 429 error. You are not charged for it, but requests are rejected outright rather than queued, so retry-and-throttle logic belongs in your code.

## What happens when credits run out, and the refund safety net

When credits hit zero, scraping pauses by default until renewal. If you need continuity, there are two opt-in paths. On monthly Hobby through Professional plans, auto-upgrade moves you to the next tier automatically when credits run out — which, per the official docs, often provides a better price-per-credit. On Scaling and above, Pay-As-You-Go lets you keep scraping at a fixed per-credit rate up to a monthly spending cap you set yourself; it is a spending limit, not a pre-charge, so no surprise bills.

The risk-free part is refreshingly concrete: your first 7 days after signing up include 5,000 free requests for large-scale testing, and after purchase there is a 7-day no-questions-asked refund policy. You can cancel at any time without a cancellation fee. Testing costs essentially nothing, and the exit door stays open. 👉 [Use 5,000 free requests to validate your target site first](https://www.scraperapi.com/signup?fp_ref=coupons)

## FAQ

**Is Selenium scraping slow, and can I speed it up?**
Yes, it is slower than pure HTTP requests, because browser rendering cannot be skipped. You can block unnecessary resource loading (images, fonts, media) and run headless with `--headless=new`. But for static pages, go back to Requests or Scrapy — that is where the real speed lives.

**Will I still get blocked if I use Selenium?**
Yes. Selenium solves rendering, not IP reputation. The flag `--disable-blink-features=AutomationControlled` removes one automation signal that anti-bot systems check, and rendering at a desktop resolution helps, but IP-level bans need rotating proxies. That is exactly what ScraperAPI's proxy port mode is for: Selenium handles rendering, the proxy layer handles IPs and CAPTCHAs.

**Is the free plan enough?**
For validating an idea and scraping a few hundred pages, 1,000 monthly credits plus the 7-day 5,000-request trial are plenty. Just remember Google costs 25 credits per request — the free allowance covers 40 Google queries. If your target is search engine data, budget carefully from day one.

## Final decision guide

Match your situation: if you are learning selenium scraping or pulling a few semi-dynamic pages, the free plan plus a plain Selenium script is all you need — spend nothing. If you scrape several thousand standard pages a week, Hobby's 100,000 credits is the entry point. If your targets are Google, Amazon or protected sites, run the Domain Cost Estimator before choosing a tier, because those credits burn fast and plan prices alone will mislead you. Two things to confirm before you commit: the per-request credit cost of your target domain, and which monthly request volume your project actually lands in. 👉 [Start free, prove the pipeline works, then pick a tier](https://www.scraperapi.com/signup?fp_ref=coupons)
