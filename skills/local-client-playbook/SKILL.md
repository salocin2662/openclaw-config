---
name: "local-client-playbook"
description: "Run the full Local Client Playbook: scrape Google Maps → filter leads → build landing pages → cold outreach → close at $800/client."
---

# Skill: local-client-playbook

> **Turn Google Maps into a $10k/month agent business.**
> Find local businesses with no website → build them a landing page → charge **$800 per client**.
> Based on [Raytar's viral thread](https://x.com/Raytar/status/2062812847428481488) by way of [tylerdotai/local-client-playbook](https://github.com/tylerdotai/local-client-playbook).

---

## Overview

This skill runs the full **Local Client Playbook** pipeline — from prospect scraping to closing:

1. **Scrape** Google Maps for local service businesses
2. **Filter** to businesses with no website, 3+ stars, 3+ reviews, a phone number
3. **Build** a clean landing page for each qualified lead
4. **Cold outreach** via text and phone — offer the free preview
5. **Close** at $800 (hosting included year 1, then $30/month)

**Metrics to track:**
| Milestone | Target |
|---|---|
| Raw leads scraped | 50–100 |
| Qualified leads after filter | 20–30 |
| Pages built (first batch) | 5–10 |
| Outreach sent | 10–15 |
| Replies | 3–5 |
| Deals closed (first batch) | 1–2 |

---

## Step 1 — Scrape Google Maps

**Time:** 30–60 minutes
**Goal:** 50–100 raw leads in `leads_raw.csv`

### Setup

```bash
git clone https://github.com/tylerdotai/Google-Maps-Scrapper
cd Google-Maps-Scrapper
uv venv .venv --python 3.11
source .venv/bin/activate
uv pip install -r requirements.txt
playwright install chromium
```

### Run Scapes

Target one city, multiple categories:

```bash
python main.py -s "plumber in [CITY]" -t 50 -o leads_plumber.csv
python main.py -s "hvac in [CITY]" -t 50 -o leads_hvac.csv
python main.py -s "electrician in [CITY]" -t 50 -o leads_electrician.csv
python main.py -s "roofing in [CITY]" -t 50 -o leads_roofing.csv
python main.py -s "pest control in [CITY]" -t 50 -o leads_pest.csv
python main.py -s "landscaping in [CITY]" -t 50 -o leads_landscape.csv
python main.py -s "cleaning service in [CITY]" -t 50 -o leads_cleaning.csv
```

Combine:
```bash
cat leads_*.csv > leads_raw.csv
```

### Lead Quality Tiers

| Tier | Criteria | Priority |
|---|---|---|
| **Gold** | No website, 3.5+ stars, 10+ reviews, phone present | Build pages first |
| **Good** | No website, 3.0–3.4 stars, 5–9 reviews, phone present | Build pages next |
| **Marginal** | No website, under 3.0 stars, under 5 reviews | Skip unless desperate |

### Fields Extracted

`name`, `address`, `phone_number`, `website`, `reviews_count`, `reviews_average`, `place_type`

---

## Step 2 — Filter Leads

**Time:** 10–15 minutes
**Goal:** 20–30 qualified leads in `leads-filtered.csv`

### Filter Rules

Remove any business where:
- **Website** is not empty (they already have a site)
- **Name** contains a national chain (Home Depot, Servpro, Stanley Steemer, Molly Maid, Terminix, Orkin, etc.)
- **Stars** under 3.0
- **Reviews** under 3
- **Phone** is empty
- **Address** is outside target city
- Business is permanently closed

### Python Filter Script

```python
#!/usr/bin/env python3
"""Filter raw leads from Google Maps scraper."""

import csv
import os

CHAINS = [
    "home depot", "servpro", "stanley steemer", "molly maid",
    "servicemaster", "merry maids", "copper state", "franchise",
    "terminix", "orkin", "rentokil", "atkins", "maids", "maid brigade",
]

def filter_leads(input_file="leads_raw.csv", output_file="leads-filtered.csv"):
    if not os.path.exists(input_file):
        print(f"Error: {input_file} not found.")
        return

    with open(input_file, newline="", encoding="utf-8") as f:
        reader = csv.DictReader(f)
        rows = list(reader)

    print(f"Loaded {len(rows)} raw leads...")

    def good(row):
        website = str(row.get("website", "")).strip()
        if website and website.lower() not in ("nan", "none", ""):
            return False
        name = str(row.get("name", "")).lower()
        for chain in CHAINS:
            if chain in name:
                return False
        try:
            stars = float(row.get("reviews_average", 0))
            if stars < 3.0:
                return False
        except (ValueError, TypeError):
            pass
        try:
            reviews = int(float(row.get("reviews_count", 0)))
            if reviews < 3:
                return False
        except (ValueError, TypeError):
            pass
        phone = str(row.get("phone_number", "")).strip()
        if phone in ("nan", "none", "", "0"):
            return False
        return True

    filtered = [r for r in rows if good(r)]

    def sort_key(r):
        try:
            stars = float(r.get("reviews_average", 0))
        except (ValueError, TypeError):
            stars = 0
        try:
            reviews = int(float(r.get("reviews_count", 0)))
        except (ValueError, TypeError):
            reviews = 0
        return (stars, reviews)

    filtered.sort(key=sort_key, reverse=True)

    with open(output_file, "w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=rows[0].keys())
        writer.writeheader()
        writer.writerows(filtered)

    print(f"Filtered to {len(filtered)} qualified leads -> {output_file}")

if __name__ == "__main__":
    import sys
    filter_leads(sys.argv[1] if len(sys.argv) > 1 else "leads_raw.csv")
```

Run: `python scripts/filter_leads.py`

---

## Step 3 — Build Landing Pages

**Time:** 2–4 hours
**Goal:** 5–10 pages ready for outreach

### Master Prompt for Claude

Use this exact prompt. Replace all bracketed variables with data from `leads-filtered.csv`:

```
Use the business info below to build a single-page landing page for [BUSINESS NAME].

[BUSINESS NAME] is a [CATEGORY] serving [CITY/AREA].
Phone: [PHONE]
Address: [ADDRESS]
Rating: [X] stars on Google ([X] reviews)

Build a complete HTML file with:
- Hero: business name, what they do, service area
- Services: 4–6 realistic services for a [CATEGORY]
- Social proof: "Proudly serving [CITY] for X years" (estimate 8–15 years if unknown)
- Why Choose Us: 3 short bullets
- Contact section: phone, address, hours
- Footer: address and phone repeated
- A "Get a Free Quote" CTA button

Design: clean, professional, trustworthy. Navy blue and white with gold (#C8A84B) accents. No stock photos. No Lorem Ipsum. No filler text.

Output the complete HTML. Include a <style> block with embedded CSS. Make it mobile-responsive.
```

### Service Type Reference

| Business Type | Services to Include |
|---|---|
| Plumber | Drain cleaning, water heater repair, leak detection, pipe replacement, fixture installation, sump pump |
| HVAC | AC repair, heating repair, AC installation, maintenance plans, indoor air quality, ductwork |
| Electrician | Panel upgrades, outlet installation, lighting installation, wiring repair, safety inspection, surge protection |
| Roofer | Shingle replacement, storm damage repair, roof inspection, gutter cleaning, skylight installation, roof maintenance |
| Landscaper | Lawn maintenance, landscape design, tree trimming, irrigation installation, mulch installation, outdoor lighting |
| Pest Control | Termite treatment, bed bug treatment, rodent control, ant control, preventive treatments, spider removal |
| Cleaner | Residential cleaning, deep cleaning, move-in/move-out cleaning, commercial cleaning, window cleaning, carpet cleaning |

### Output Structure

```
local-client-playbook/
├── leads-filtered.csv
└── pages/
    ├── acme-plumbing-landing.html
    ├── jones-hvac-landing.html
    └── ...
```

Create: `mkdir -p pages`

### Before Sending — Quality Checklist

- [ ] Business name is correct
- [ ] Phone number is right
- [ ] Address is right
- [ ] Services look realistic for that trade
- [ ] Page loads without errors
- [ ] CTA button is visible above the fold

### Hosting (Vercel or Netlify)

When a client agrees, get the page live in 60 seconds:

**Netlify (fastest):**
1. Go to [app.netlify.com/drop](https://app.netlify.com/drop)
2. Drag the HTML file onto the window
3. Get a live URL instantly

**Vercel:**
```bash
npm i -g vercel
cd pages
vercel --prod --yes
```

---

## Step 4 — Cold Outreach

**Time:** 60–90 minutes
**Goal:** 5–10 replies, 2–3 conversations booked

### The Golden Rule

**Send outreach the SAME DAY you build the page.** Energy is highest right after building. Strike while it's fresh.

### Text Template (Day 1)

```
Hi [FIRST NAME], I noticed [BUSINESS NAME] — really solid reviews on Google.
I actually built a custom landing page for [CITY] [TRADE]s last week and one of them said it brought in 6 new calls in a month.
Wanted to offer the same thing to you — no cost, no obligation.
If you're curious, I can send you a preview link. Takes 2 min to look at.
-[YOUR NAME]
```

**Variables:** `[FIRST NAME]`, `[BUSINESS NAME]`, `[CITY]`, `[TRADE]`, `[YOUR NAME]`

### Follow-Up Sequence

| Day | Action |
|---|---|
| Day 1 | Send initial text with live URL |
| Day 3 | No reply → follow-up text |
| Day 7 | No reply → phone call |
| Day 10 | No reply → final text with link |

### Follow-Up Text (Day 3)

```
Hey [NAME], just following up.
Here's that preview link: [URL]
Still no charge. Still no catch.
Let me know if you have any questions — [PHONE]
```

### Phone Script (Day 7)

```
Hi [NAME], this is [YOU] — I'm a local web developer.
I was searching [CITY] [TRADE]s on Google and saw you don't have a website yet.
I built one for [OTHER BUSINESS IN SAME CITY] last week and they liked it.
I want to build you one — no charge — just to show you what it looks like.
Worst case, you see something you like and it's free. Best case, it brings in new customers.
You got 2 minutes?
```

### Tracking Spreadsheet

| Business | Contact | Date | Method | Reply? | Status |
|---|---|---|---|---|---|
| Acme Plumbing | John Smith | 7/8 | Text | Yes | Sent proposal |
| Jones HVAC | — | 7/8 | Text | No | Following up 7/11 |

---

## Step 5 — Close

**Pricing:**
| Package | Price |
|---|---|
| Landing page + year 1 hosting | **$800** |
| Annual hosting after year 1 | $30/month |
| Custom domain | $15/year (optional) |

### The Close Line

> "The page is live at [URL]. For $800, I set this up, host it for the first year, and connect it to your Google Business Profile. After a year, it's $30/month to keep it hosted. You can cancel any time."

### Objection Handling

**"I already have a website."**
> "Cool — how's it working for you? This would be an additional page specifically designed to convert the people who find you on Google. Different purpose."

**"No budget right now."**
> "Totally understand. Can I check back in 60 days?"

**"Too expensive."**
> "What would make this feel like the right price for you?"

**"Can you send me a proposal?"**
> "Sure. Give me their email."

### After Closing

1. Invoice via Venmo, PayPal, Cash App, or Stripe
2. Transfer hosting to their domain or keep it running
3. Give them access to the page
4. **Ask for referrals:** "Do you know any other [TRADE]s in [CITY] who could use this?"

### Scaling

Once you've done **5 clients**:
- Raise to $800/page
- Add "Google Business Profile optimization" as an upsell ($200)
- Offer monthly retainer for ongoing updates

Once you've done **15+ clients**:
- Systematize with an AI agent that builds the pages
- Hire a VA to do outreach while you build
- Raise prices to $1,200+/page

---

## Landing Page Template (Static HTML)

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{BUSINESS_NAME}} | {{CITY}} {{CATEGORY}}</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { font-family: 'Segoe UI', system-ui, sans-serif; color: #1a1a2e; background: #fff; line-height: 1.6; }
    header { background: #0a1628; color: #fff; padding: 60px 20px; text-align: center; }
    header h1 { font-size: 2.4rem; margin-bottom: 10px; color: #fff; }
    header p { font-size: 1.2rem; color: #c8a84b; margin-bottom: 20px; }
    .stars { color: #f5a623; font-size: 1.1rem; margin-bottom: 10px; }
    .cta-btn { display: inline-block; background: #c8a84b; color: #0a1628; padding: 16px 36px; font-size: 1.1rem; font-weight: 700; text-decoration: none; border-radius: 4px; transition: background 0.2s; }
    .cta-btn:hover { background: #e0bc5a; }
    section { padding: 50px 20px; max-width: 900px; margin: 0 auto; }
    .services { background: #f8f9fa; }
    .services h2 { text-align: center; font-size: 1.8rem; margin-bottom: 30px; color: #0a1628; }
    .service-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); gap: 20px; }
    .service-card { background: #fff; border: 1px solid #e0e0e0; border-radius: 6px; padding: 24px; }
    .service-card h3 { color: #0a1628; margin-bottom: 8px; font-size: 1.1rem; }
    .service-card p { color: #555; font-size: 0.95rem; }
    .why-us { background: #0a1628; color: #fff; }
    .why-us h2 { color: #c8a84b; text-align: center; font-size: 1.8rem; margin-bottom: 30px; }
    .why-list { display: grid; grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); gap: 24px; max-width: 900px; margin: 0 auto; }
    .why-item { text-align: center; }
    .why-item .icon { font-size: 2rem; margin-bottom: 10px; }
    .why-item h3 { margin-bottom: 6px; font-size: 1.1rem; }
    .why-item p { color: #b0b8c8; font-size: 0.9rem; }
    .contact { background: #fff; text-align: center; }
    .contact h2 { font-size: 1.8rem; margin-bottom: 20px; color: #0a1628; }
    .contact-info { font-size: 1.1rem; margin-bottom: 30px; }
    .contact-info p { margin: 8px 0; }
    .contact-info strong { color: #0a1628; }
    footer { background: #061018; color: #607080; text-align: center; padding: 24px 20px; font-size: 0.9rem; }
    @media (max-width: 600px) {
      header h1 { font-size: 1.8rem; }
      section { padding: 36px 16px; }
    }
  </style>
</head>
<body>
<header>
  <h1>{{BUSINESS_NAME}}</h1>
  <p>Trusted {{CATEGORY}} Services in {{CITY}} and Surrounding Areas</p>
  <div class="stars">★ ★ ★ ★ ☆ <span style="color:#fff; font-size:0.9rem;">{{STAR_RATING}} ({{REVIEW_COUNT}} Google Reviews)</span></div>
  <a href="tel:{{PHONE}}" class="cta-btn">Call for a Free Quote</a>
</header>
<section class="services">
  <h2>Our Services</h2>
  <div class="service-grid">
    <!-- 4–6 service cards -->
  </div>
</section>
<section class="why-us">
  <h2>Why Choose {{BUSINESS_NAME}}</h2>
  <div class="why-list">
    <div class="why-item"><div class="icon">★</div><h3>Licensed & Insured</h3><p>Full liability coverage. Peace of mind on every job.</p></div>
    <div class="why-item"><div class="icon">★</div><h3>Free Estimates</h3><p>Upfront pricing. No surprises when the bill comes.</p></div>
    <div class="why-item"><div class="icon">★</div><h3>Local & Reliable</h3><p>Serving {{CITY}} for over {{YEARS_IN_BUSINESS}} years.</p></div>
  </div>
</section>
<section class="contact">
  <h2>Get in Touch</h2>
  <div class="contact-info">
    <p><strong>Phone:</strong> <a href="tel:{{PHONE}}">{{PHONE}}</a></p>
    <p><strong>Address:</strong> {{ADDRESS}}</p>
    <p><strong>Hours:</strong> Mon–Sat 7:00 AM – 6:00 PM</p>
  </div>
  <a href="tel:{{PHONE}}" class="cta-btn">Call Now</a>
</section>
<footer>
  <p>{{BUSINESS_NAME}} &mdash; {{ADDRESS}} &mdash; {{PHONE}}</p>
</footer>
</body>
</html>
```

**Brand colors:** Navy `#0a1628` + White `#fff` + Gold `#c8a84b`
**Fonts:** Segoe UI / system-ui / sans-serif

---

## Key Principles

1. **Send outreach the same day you build the page.** Energy fades.
2. **Tier 1 leads first.** 4+ stars, 10+ reviews, no website.
3. **Send the live URL, not the HTML file.** Make it effortless to view.
4. **Price is $800.** Do not apologize. Do not discount early.
5. **Ask for referrals after every close.** "Who else in [CITY] could use this?"
6. **No stock photos. No Lorem Ipsum.** Ever.
