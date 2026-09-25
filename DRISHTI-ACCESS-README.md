# Drishti — Real Customer Access, Wired

## What changed (v2)
- Fake "click to unlock" demo toggle — **removed entirely**.
- Real backend: `public.drishti_access` table in Supabase, RLS-locked, service-role only.
- Two ways in for a real customer, both live:
  - **Magic link**: `https://taragni.in/?drishti=THEIR-CODE` — auto-verifies the moment they open the widget, no typing.
  - **Manual**: type `/unlock THEIR-CODE` into the normal chat box.
- Neither path burns a free question. Once verified, the 5-question limit disappears entirely for that browser session.

## Onboarding a real paying customer — 2 steps

**Step 1 — Generate their chart JSON.** Same shape every time:

```json
{
  "lagna": "Leo", "lagnaLord": "Sun",
  "rasi": "Pisces", "rasiLord": "Jupiter",
  "nakshatra": "Revati", "nakshatraLord": "Mercury",
  "planets": {
    "Sun": "Leo, 1st house (own sign, own Lagna)",
    "Moon": "Pisces, 8th house"
  },
  "currentMahadasha": "Rahu Mahadasha (started 2 years ago)",
  "notes": ["Any 1-3 line observations you want Drishti to lean on, same voice as your report templates."]
}
```

**Step 2 — Insert the row.** Run this in Supabase → SQL Editor for every new customer:

```sql
insert into public.drishti_access (access_code, customer_name, report_type, chart_data)
values (
  'GENERATE-A-CODE-HERE',   -- e.g. their order ID, or a short random token
  'Customer Full Name',
  'sampurna_jeevan',        -- or: vivaha_kundali | karma_jeevika | graha_shanti | varsha_phala | vastu_tara
  '{ ...chart JSON from Step 1... }'::jsonb
);
```

Then email them:
```
Your personal Drishti reading is ready: https://taragni.in/?drishti=GENERATE-A-CODE-HERE
```

That's the whole loop. No dashboard, no admin panel — deliberately, per the Zero-Cost Extraction Protocol. If volume grows past a few customers a week and manual SQL inserts become the bottleneck, that's the trigger to build a proper intake form — not before.

## Optional: revoke or expire access
```sql
update public.drishti_access set active = false where access_code = 'CODE';
-- or set a hard expiry at insert time:
-- expires_at = now() + interval '12 months'
```

## Deploy
Same as before — this `index.html` drops into the same zip structure from the last deploy package. Push it to whichever host you're already using (Vercel/Netlify/Cloudflare Pages); no other files changed.
