# Metria — parent-coach landing

Validation landing page for the Metria parent-coach app.
Hosted on GitHub Pages → **https://alicankuklaci.github.io/parent-coach-landing/**

---

## How to swap the app name

The working name **Metria** lives in ONE constant in `index.html`:

```js
const APP_NAME = 'Metria';
```

Change that string and the header/brand will update automatically via `document.querySelectorAll('.js-appname')`.

---

## How to update copy

All visible text is plain HTML inside `data-lang="tr"` / `data-lang="en"` elements in `index.html`.
Edit the text directly, commit and push — GitHub Pages deploys automatically.

---

## How to update pricing

Two price cards live in `index.html` at `id="price-a"` and `id="price-b"`.
Change the `₺` / `$` amounts inside each card.
The A/B split (50/50) is handled in JS — no code change needed.

---

## Edge function

The waitlist backend lives in the Oneiros Supabase project:

- **URL:** `https://laevrozkdndxvfecjfzf.supabase.co/functions/v1/waitlist`
- **Source:** `supabase/functions/waitlist/index.ts` in the oneiros repo
- **Migration:** `supabase/migrations/009_waitlist.sql`
- **CORS:** `https://alicankuklaci.github.io` only

---

## SQL — read counts

```sql
-- Total signups per price variant
SELECT
  price_variant,
  COUNT(*) AS signups
FROM waitlist_signups
GROUP BY price_variant
ORDER BY price_variant;

-- Event funnel per variant (view → price_click → signup)
SELECT
  price_variant,
  event,
  COUNT(DISTINCT session_id) AS sessions
FROM waitlist_events
GROUP BY price_variant, event
ORDER BY price_variant, event;

-- Price-click-to-signup conversion rate per variant
WITH views AS (
  SELECT price_variant, COUNT(DISTINCT session_id) AS views
  FROM waitlist_events WHERE event = 'view' GROUP BY price_variant
),
clicks AS (
  SELECT price_variant, COUNT(DISTINCT session_id) AS clicks
  FROM waitlist_events WHERE event = 'price_click' GROUP BY price_variant
),
signups AS (
  SELECT price_variant, COUNT(*) AS signups FROM waitlist_signups GROUP BY price_variant
)
SELECT
  v.price_variant,
  v.views,
  c.clicks,
  s.signups,
  ROUND(100.0 * s.signups / NULLIF(v.views, 0), 1) AS signup_rate_pct,
  ROUND(100.0 * c.clicks  / NULLIF(v.views, 0), 1) AS click_rate_pct
FROM views v
LEFT JOIN clicks  c USING (price_variant)
LEFT JOIN signups s USING (price_variant);
```

---

## Files

| File | Purpose |
|---|---|
| `index.html` | Landing page (TR default, `?lang=en` or toggle for EN) |
| `privacy.html` | KVKK / GDPR privacy notice (TR + EN) |
| `README.md` | This file |

---

## Decision threshold (from SYNTHESIS §6)

Stop the A/B test and pick a price when: **≥100 signups** and **≥3% price-click rate**.
