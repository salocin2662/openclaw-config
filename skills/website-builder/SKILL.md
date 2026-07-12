---
name: "website-builder"
description: "Design, build, and deploy modern websites with orders, email, and payment processing — from static generators to full-stack e-commerce."
---

# website-builder Skill

## Purpose

Build and deploy modern websites with commerce features: orders, email automation, and payment processing. Covers full-stack frameworks, e-commerce platforms, payment APIs, transactional email, and order management.

## When to Use

- User wants to build or improve a website with e-commerce features
- User asks about web frameworks, hosting, or deployment
- User needs payment integration (Stripe, PayPal, etc.)
- User needs transactional email setup (receipts, order confirmations)
- User needs order management or inventory tracking
- User wants to add a cart, checkout, or digital products to an existing site

## Files to Load

1. `https://github.com/salocin2662/website-builder/blob/main/MASTER_PROMPT.md`
2. `https://github.com/salocin2662/website-builder/blob/main/reference-docs/FRONTEND_FRAMEWORKS.md`
3. `https://github.com/salocin2662/website-builder/blob/main/reference-docs/ECOMMERCE_PLATFORMS.md`
4. `https://github.com/salocin2662/website-builder/blob/main/reference-docs/PAYMENTS.md`
5. `https://github.com/salocin2662/website-builder/blob/main/reference-docs/EMAIL.md`
6. `https://github.com/salocin2662/website-builder/blob/main/reference-docs/ORDER_MANAGEMENT.md`
7. `https://github.com/salocin2662/website-builder/blob/main/reference-docs/HOSTING.md`
8. `https://github.com/salocin2662/website-builder/blob/main/templates/QUICK_START_NEXTJS_STRIPE.md`
9. `https://github.com/salocin2662/website-builder/blob/main/templates/QUICK_START_SNIPCART.md`

## Decision Tree

```
What does Nicolas want to build?
├── A simple brochure / portfolio / blog site
│   → Hugo or Astro + GitHub Pages or Cloudflare Pages
├── A small business store (physical or digital products)
│   ├── Has existing site → Add Snipcart (3 lines of JS)
│   └── New build
│       ├── Fastest path → Hugo + Snipcart
│       └── Full control → Next.js + Stripe + Resend
├── A custom e-commerce / marketplace
│   ├── Full platform → Spree or nopCommerce
│   ├── Headless/composable → Saleor or Medusa
│   └── Custom build → Next.js + Stripe Connect + InvenTree
├── A SaaS / subscription product
│   → Next.js + Stripe Billing + Resend
└── An API-only backend
    → FastAPI or Node.js/Express + Railway/Render
```

## Architecture Patterns

### Pattern 1: Static + Snipcart (Fastest)
Hugo/Astro/11ty + Snipcart (3 lines JS) + Stripe/PayPal. Time to store: 1–4 hours.

### Pattern 2: Next.js + Stripe (Full Custom)
Next.js + Stripe Checkout + webhook → DB (orders/inventory) + Resend email + InvenTree tracking.

### Pattern 3: E-Commerce Platform (All-In-One)
Spree (Ruby) / nopCommerce (.NET) / OpenCart (PHP) with built-in payments, orders, inventory.

### Pattern 4: Hyperswitch (Multi-Provider)
Hyperswitch API routes to Stripe OR Adyen OR Braintree for best fee routing.

## Key Repos

| Category | Top Repo |
|---|---|
| Stripe examples | `stripe-samples/accept-a-payment` |
| Marketplace + Connect | `michaelokolo/marketplace` |
| Open-source payment router | `juspay/hyperswitch` |
| E-commerce platform | `spree/spree`, `nopcommerce/nopcommerce` |
| Headless commerce | `saleor/saleor`, `medusajs/medusa` |
| Cart embed | `snipcart/snipcart` |
| Inventory | `inventree/InvenTree` |
| Transactional email | `resend/resend` (3K free/mo) |

## Hard Rules

1. **Never recommend WooCommerce** for new custom builds
2. **Always use Stripe Checkout or Payment Element** — never custom payment forms
3. **Always use a transactional email API** (Resend, Postmark, Mailgun) — never send from app server
4. **Always implement webhooks** for payment confirmation
5. **Always use HTTPS**

## Output Format

1. **Recommended stack** (1 sentence)
2. **Architecture pattern**
3. **Key GitHub repos**
4. **Quick-start template** if applicable
5. **Next steps**

## Skill Version

v1.0 — 2026-07-10 — Initial release
