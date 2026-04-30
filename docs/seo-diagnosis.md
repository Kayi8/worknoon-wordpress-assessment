# SEO Diagnosis: Website Not Indexing After Sitemap Submission

## Scenario
A new Worknoon website has been submitted to Google Search Console with a sitemap, but pages are not appearing in Google's index.

---

## Step 1: Crawlability Tests

**Tools:** Google Search Console → URL Inspection, Screaming Frog, Google's Rich Results Test

### Checks:
- Use **URL Inspection Tool** in Search Console on the homepage URL
  - If it says "URL is not on Google" → crawling is blocked or page has indexing issues
- Run **"Request Indexing"** on critical pages
- Use `site:worknoon.com` in Google — if zero results, indexing is fully blocked
- Check if Googlebot can reach the server:
  - Look at **Coverage Report** in Search Console for crawl errors (404s, 5xx errors, redirect loops)

---

## Step 2: Canonical Checks

A misconfigured canonical tag silently blocks indexing.

### Checks:
- View page source and look for:
  ```html
  <link rel="canonical" href="https://worknoon.com/page/" />
  ```
- Ensure canonical points to **itself** (self-referencing) on each page
- Watch for:
  - Canonical pointing to a staging URL (`https://staging.worknoon.com/...`) — a very common mistake
  - Canonical pointing to HTTP instead of HTTPS
  - Duplicate content with canonical pointing away from the intended page
- Use Screaming Frog → `SEO` tab → `Canonicals` to bulk-check all URLs

---

## Step 3: Robots.txt & No-Index Audit

### Robots.txt
Navigate to `https://worknoon.com/robots.txt` and check for:

```
# PROBLEM: This blocks all crawlers
User-agent: *
Disallow: /
```

Should be:
```
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php
Sitemap: https://worknoon.com/sitemap.xml
```

### No-Index Tags
- Check page source for:
  ```html
  <meta name="robots" content="noindex" />
  ```
- In WordPress: Check **Settings → Reading** → "Discourage search engines from indexing this site" — this is frequently left checked after development
- Check Yoast/RankMath settings per page — they may have noindex toggled on

---

## Step 4: Sitemap Structure Issues

### Checks:
- Visit `https://worknoon.com/sitemap.xml` directly — does it load?
- Ensure the sitemap:
  - Contains only **canonical, indexable URLs** (no noindex pages)
  - Has correct `<lastmod>` dates
  - Does not reference 301/302 redirect URLs — use final destination URLs only
  - Is under 50,000 URLs and 50MB (use sitemap index files if larger)
- In Search Console → **Sitemaps** → verify status is "Success" not "Couldn't fetch" or "Has errors"
- Common plugin issues:
  - Yoast SEO sitemap: Make sure it's enabled under Yoast → General → Features
  - RankMath: Check RankMath → Sitemap Settings

---

## Step 5: Page Speed Indexing Blockers

Google's crawl budget is affected by slow pages. Very slow pages may be deprioritized.

### Checks:
- Run **PageSpeed Insights** (`pagespeed.web.dev`) on the homepage
- Check **Core Web Vitals** in Search Console → Experience → Core Web Vitals
- Look for:
  - **Server response time > 600ms** → upgrade hosting or enable caching
  - **Render-blocking JavaScript** → defer non-critical JS
  - **Uncompressed images** → use WebP format, install Smush or ShortPixel
  - **No caching layer** → install WP Rocket, LiteSpeed Cache, or W3 Total Cache
- Ensure **HTTPS is properly configured** — mixed content warnings block Google trust signals

---

## Step 6: Search Console Debugging Steps

### Full Debugging Flow:

1. **Coverage Report** → Go to `Index → Coverage`
   - Check "Excluded" tab: Look for "Crawled - currently not indexed", "Discovered - currently not indexed"
   - "Crawled - currently not indexed" = Google reached the page but chose not to index it (thin content, quality issue, or duplicate)

2. **URL Inspection Tool** on affected pages:
   - Click "Test Live URL" to see what Googlebot sees right now
   - Check "Page fetch" — was it successful?
   - Check "Indexing allowed?" — Yes or No
   - Check "Canonical" — does Google agree with your declared canonical?

3. **Manual Request Indexing**:
   - For priority pages, use "Request Indexing" button in URL Inspection
   - Note: This doesn't guarantee indexing, just prioritizes crawling

4. **Check Crawl Stats**:
   - Settings → Crawl Stats → Look for crawl errors, response time spikes

5. **Verify DNS / Server Issues**:
   - If server was recently launched, DNS propagation may still be incomplete
   - Use `https://dnschecker.org` to confirm global DNS resolution

---

## Quick Fix Priority Order

| Priority | Issue to Fix                                      |
|----------|---------------------------------------------------|
| 🔴 Critical | "Discourage search engines" setting in WordPress |
| 🔴 Critical | `Disallow: /` in robots.txt                      |
| 🔴 Critical | Noindex meta tag on all pages                    |
| 🟠 High   | Canonical pointing to staging/wrong URL           |
| 🟠 High   | Sitemap not submitted or returning errors         |
| 🟡 Medium | Server response time > 1s                         |
| 🟡 Medium | Pages marked excluded in Coverage report          |
| 🟢 Low    | Missing structured data                           |
