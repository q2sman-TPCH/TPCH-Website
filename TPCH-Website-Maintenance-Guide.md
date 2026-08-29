# TPCH Website — Maintenance & Handover Guide

**Last updated:** August 2026
**Maintained by:** Kajal Mukherjee (President, TPCH)
**Purpose of this document:** So that anyone — including a future you, or someone else stepping in — can pick up website maintenance without having to reconstruct the whole picture from scratch.

---

## 1. What this site is

The TPCH website (**taurangaprobasi.nz**) is the online home of Tauranga Probasi Cultural Heritage Incorporated, a Bengali cultural association in Tauranga, New Zealand. It grew out of a need to publish the annual Durga Puja magazine ("Parampara") as a native web eMagazine, and has since expanded into a full site: festival info, gallery, celebrations calendar, membership, and more.

The site was rebuilt from scratch after a server breach at a previous hosting provider caused irrecoverable data loss. Everything now lives on infrastructure the organisation directly controls (see below).

---

## 2. The three accounts that matter

Website work touches **three separate logins**. Mixing these up is the most common source of confusion, so it's worth being deliberate about which one you're in.

| Account | Email | What it's for |
|---|---|---|
| **GitHub** | `q2sman-TPCH` org account | Hosts the website source code (HTML files) |
| **Cloudflare** | `q2sman@gmail.com` (personal) | Domain DNS, Pages hosting/deployment, security settings |
| **Google Search Console** | `tpchconnect@gmail.com` (org) | SEO monitoring, sitemap submission, indexing status |
| **1st Domains NZ** | Two separate accounts exist | Domain registration — TPCH org account **and** a separate personal account (used for Kajal's wife's domains) |

Also relevant:
- **TPCH contact email:** `tpchconnect@gmail.com`
- **Contact form backend:** Formspree, endpoint `https://formspree.io/f/mrewzkjj`, delivers to `tpchconnect@gmail.com`

---

## 3. Infrastructure map

```
Domain (taurangaprobasi.nz)
  registered at → 1st Domains NZ (TPCH org account)
  DNS managed at → Cloudflare (q2sman@gmail.com account)
  deployed via → Cloudflare Pages
  source code in → GitHub repo: q2sman-TPCH/TPCH-Website
```

**How a site update actually goes live:**
1. A file is added or changed in the GitHub repo (`q2sman-TPCH/TPCH-Website`)
2. Cloudflare Pages watches that repo and **auto-deploys** on every commit — no separate "deploy" step needed
3. Deployment usually takes 1–2 minutes after a commit lands

There is **no command-line git workflow** in use — all repo changes are made through the **GitHub web UI** (upload files directly in the browser). This is a deliberate choice, not a limitation — keep doing it this way unless someone comfortable with git wants to switch.

---

## 4. How to make a routine update (the common case)

Most updates are: *fix some text, swap a photo, tweak a page.*

1. Go to the GitHub repo: `github.com/q2sman-TPCH/TPCH-Website`
2. Navigate into the correct folder first (if uploading into a subfolder like `images/`) — uploading from the repo root will land files in the wrong place
3. Click **Add file → Upload files**
4. Upload the corrected file(s) — only upload what actually changed, not the whole site
5. Commit
6. Wait ~1–2 minutes, then check the live site to confirm

**Good habits already in place, worth keeping:**
- Collect all known issues first, then fix them in one batch rather than one-off patches
- Upload only the changed files, not a full resync
- Test on a real phone/device after visual changes, not just desktop browser
- Take a screenshot before committing anything structurally significant, for a sanity check

---

## 5. Known technical gotchas (learned the hard way)

These aren't obvious from looking at the code — they were discovered through actual incidents. Worth reading before making changes.

### 5.1 Clean URLs (no `.html` in links)
Cloudflare Pages **automatically strips `.html`** from URLs. `about.html` is served at `/about`, not `/about.html`. Both technically work (one redirects to the other), but:
- Always link internally using the clean form (`/about`, not `/about.html`)
- Sitemap entries must use the clean form too (already fixed as of Aug 2026)

### 5.2 Images are embedded as base64 — this is a known problem
Several pages (notably `celebrations.html` and `gallery.html`) have photos embedded directly in the HTML as base64 text rather than as separate image files. This causes real problems:
- `celebrations.html` is **~24MB** because of this
- Browsers can't cache these images, so every page load re-downloads everything
- Standard text tools (grep, view) become unreliable on these files because of the huge embedded data blobs

**Working pattern for large files:** filter out the base64 noise before inspecting structure:
```
grep -v "data:image\|base64" file.html > /tmp/clean.html
```
Then use `grep -n` on the *original* file to find exact line numbers, and `sed -n 'START,ENDp'` to extract sections (including nav/footer, which contain the embedded logo) without corrupting the base64 data.

**The fix (not yet done):** migrate to a structured `images/` folder with real image files, referenced normally (`<img src="images/...">`). This needs to happen **before** the backlog of historical event photos gets added — adding more base64 photos on top of the current mess will make it worse.

### 5.3 GitHub case-sensitive folder renames
If you ever need to rename a folder to fix its casing (e.g. `Images` → `images`), GitHub's web UI can throw a case-collision error if you try to do it in one step. The fix: **delete the old one first, commit, then create the new one, commit separately.** Don't do it in the same commit.

### 5.4 Don't hand-code obfuscated emails
Cloudflare automatically injects email-obfuscation markup (`cfemail` spans) at the edge for any plain-text email address on the page. **Never try to replicate this in the source HTML** — just write the email address in plain text (e.g. `tpchconnect@gmail.com`) and let Cloudflare handle obfuscation automatically. Hand-coding it causes broken/garbled output.

### 5.5 Cloudflare's "Managed robots.txt" adds its own content
Cloudflare has a feature (AI Crawl Control → "Managed robots.txt") that auto-injects rules blocking AI-training crawlers (GPTBot, CCBot, etc.) while still allowing normal search engines. This is **on** and this is **fine** — it doesn't block Google Search indexing. If `robots.txt` ever looks different than expected, check here before assuming something's broken (Cloudflare dashboard → domain → AI Crawl Control → Overview).

---

## 6. Site structure (as of Aug 2026)

**9 core pages** + eMagazine section + 1 dedicated production page:

| Page | File | Notes |
|---|---|---|
| Home | `index.html` | |
| About Us | `about.html` | |
| Festival | `festival.html` | Has a featured-production card linking to Shyama |
| Durga Puja | `durgapuja.html` | |
| Shyama (production) | `shyama.html` | Dedicated page, follows the durgapuja.html hybrid pattern |
| Gallery | `gallery.html` | Base64 images — see §5.2 |
| Celebrations | `celebrations.html` | Base64 images, ~24MB — see §5.2 |
| eMagazine | `emagazine.html` | "Parampara | পরম্পরা", 3rd Edition (2025) live; 2024/2023 editions pending — checking if original files survived |
| Membership | `membership.html` | |
| Support Us | `support.html` | |
| Contact | `contact.html` | Form → Formspree → tpchconnect@gmail.com |

**Pattern for adding a new "production" page** (like Shyama): a featured-production card goes on the *originating event page* (e.g. festival.html), positioned above the gallery as the headline event, linking through to a standalone dedicated page. This mirrors how durgapuja.html works.

**Gallery pattern for productions:** dynamic photo galleries use a `manifest.json` file listing image filenames, fetched by JavaScript to build the gallery — falls back gracefully to a placeholder if empty. Example: `images/productions/shyama/manifest.json` (currently empty, pending real performance photos).

---

## 7. SEO setup (completed August 2026)

This was a full pass to fix Google not indexing the site at all. Everything below is live and confirmed working.

### 7.1 What's in place
- **`robots.txt`** — live at `/robots.txt` (partially Cloudflare-managed, see §5.5)
- **`sitemap.xml`** — live at `/sitemap.xml`, lists all 11 pages using clean URLs
- **www → non-www redirect** — a Cloudflare Redirect Rule (301) sends `www.taurangaprobasi.nz/*` to `taurangaprobasi.nz/*`, preventing duplicate-content issues
- **Meta tags** — every page has a tailored `<meta name="description">`, `<link rel="canonical">`, and Open Graph tags (`og:title`, `og:description`, `og:type`, `og:url`, `og:image`)
- **Social preview image** — `images/og-image-v2.jpg`, built from the actual TPCH logo (silver-grey circular badge with gradient koru), sized 1200×630 for Facebook/WhatsApp previews
- **Google Search Console** — domain property verified for `taurangaprobasi.nz` (covers www + non-www, http + https automatically), sitemap submitted and successfully read (11 pages discovered)

### 7.2 Why "Indian Bengali" language matters
There's another Bengali association in the area that is Bangladeshi in origin — TPCH is specifically an **Indian Bengali / West Bengal** heritage organisation, and the two are not merged. Because meta descriptions don't directly affect search ranking (they only affect click-through once you're already showing up), the actual fix was to make sure phrases like "Indian Bengali," "West Bengal," and "Kolkata tradition" appear in the **visible body text and meta descriptions** of key pages (About, Durga Puja, Support, Membership) — that's what Google actually matches against searches like "Indian Durga Puja Tauranga."

### 7.3 How to check indexing progress
Google Search Console (`search.google.com/search-console`, logged in as `tpchconnect@gmail.com`) → select the `taurangaprobasi.nz` property:
- **Pages** (left sidebar, under Indexing) — shows what Google has actually indexed
- **Performance** — shows real search impressions/clicks once data accumulates (takes days to weeks to populate)

Indexing is not instant — expect it to take anywhere from a few days to a couple of weeks after submission for pages to start appearing in search results.

### 7.4 Still pending
- The other pages beyond the initial homepage batch were done in one pass — but if new pages get added in future (e.g. a new production page, a new eMagazine edition), they need the same treatment: meta description, canonical URL, Open Graph tags, and an entry added to `sitemap.xml`.

---

## 8. Other known pending / parked items

- **Base64 image migration** (§5.2) — needs to happen before adding the historical photo backlog
- **eMagazine 2024 & 2023 editions** — checking whether original source files survived the earlier server breach
- **Pradipta's personal site** (`pradiptamukherjee.com` / `.co.nz`) — Rabindra Sangeet artist site, planned as a static site on Cloudflare Pages, not yet started
- **Social media strategy** — Facebook scheduling via Meta Business Suite, YouTube Brand Account setup for TPCH, short-form reels (CapCut for editing)
- **Populating Shyama's photo gallery** — once real performance photos are available, update `images/productions/shyama/manifest.json`

---

## 9. Quick troubleshooting

**"My change isn't showing up on the live site"**
Check the Cloudflare Pages deployment log (Cloudflare dashboard → Workers & Pages → the `tpch-website` project) to confirm the deploy actually ran and succeeded. Also try a hard refresh / different browser — could just be local caching.

**"A page is showing a 404"**
Check the exact filename and casing in the GitHub repo — GitHub is case-sensitive, and a mismatch between a link and the actual filename will 404 silently.

**"Images look broken / page is huge and slow"**
Likely a base64 embedding issue — see §5.2.

**"Emails on the page look like garbage text"**
Someone probably hand-coded Cloudflare's email obfuscation markup — see §5.4. Fix: replace with plain text email address.

**"www version and non-www version show different things"**
Shouldn't happen anymore — the redirect rule (§7.1) should catch this. If it does happen, check Cloudflare → Rules → Redirect Rules to confirm "Redirect www to non-www" is still active.

---

## 10. Who to ask / where things live

- **Working files during active sessions:** Claude typically works from files fetched live from GitHub, not a persistent local copy — there's no separate "master" copy to keep in sync
- **This document** should be updated whenever a significant structural or infrastructure change is made, so it doesn't go stale
