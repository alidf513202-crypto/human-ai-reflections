---
name: uae-car-finder
description: Search the UAE used/new car market for a specific car (make, model, year, specs) across Dubizzle, OpenSooq, Haraj and social media, and return a ranked list of real listings with price, mileage, spec and link. Use when the user gives a car name/model/specs and asks to find it for sale in the UAE (e.g. "ابحث لي عن لاندكروزر 2022 VXR", "find a 2021 Nissan Patrol in Dubai").
---

# UAE Car Finder / باحث السيارات في الإمارات

## 1. Parse the request
Extract and confirm (ask only if make or model is missing):
- **Make / الماركة**, **Model / الموديل**, **Year or range / السنة**
- **Trim / الفئة** (e.g. VXR, GXR, Platinum), **Spec**: GCC / American / Japanese / European
- **Budget (AED)**, **Max mileage (km)**, **Emirate / الإمارة**, colour, other must-haves
Normalise Arabic names to English (لاندكروزر → Land Cruiser, باترول → Patrol, كامري → Camry).

## 2. Search each source
Run searches in parallel. Prefer `WebSearch` with `allowed_domains`; then open promising
result pages (WebFetch or the `web-fetch` agent) to read real listing details.

| Source | Domains | Query pattern |
|---|---|---|
| Dubizzle | `uae.dubizzle.com`, `dubizzle.com` | `{make} {model} {year} {trim} for sale UAE` |
| OpenSooq | `ae.opensooq.com`, `opensooq.com` | `{make} {model} {year} للبيع الإمارات` |
| Haraj | `haraj.com.sa` (+ any UAE Haraj site the user names) | `{model} {year} حراج الإمارات` |
| Other marketplaces | `dubicars.com`, `yallamotor.com`, `carswitch.com`, `drivearabia.com` | `{make} {model} {year} UAE price` |
| Social media | `instagram.com`, `facebook.com`, `tiktok.com`, `x.com` | `{model} {year} للبيع دبي` / `"{model}" for sale Dubai` |

Dubizzle listing URLs follow `https://uae.dubizzle.com/motors/used-cars/{make}/{model}/` —
fetch that path (with filters if supported) when direct access works.

## 3. Filter and verify
- Drop listings that do not match year, trim, spec or budget; flag "export only" cars.
- Remove duplicates (same car posted on several sites: match price + mileage + colour).
- Never invent a listing, price or link. If a field is unknown, write "غير مذكور / not stated".
- Note the date seen; listings go stale quickly.

## 4. Output (reply in the user's language)
1. **Summary**: number of matches per source, price range (min / median / max AED).
2. **Table** sorted by best match then price:
   `# | Source | Title | Year | Trim/Spec | Mileage | Price AED | Location | Link`
3. **Market notes**: typical price for this spec, red flags (export only, import, salvage, missing service history).
4. **Coverage gaps**: list every source that could not be reached or returned nothing, and why.

## Limits to state honestly
- Search engines index listings with a delay; the newest ads may be missing.
- Social media posts (Instagram, Facebook, TikTok) are mostly behind login and poorly indexed — coverage there is partial.
- Some sites block automated access; when that happens, say so instead of guessing.
