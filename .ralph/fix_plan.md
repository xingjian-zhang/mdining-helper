# Ralph Fix Plan — MDining Chinese Menu Website

## Overview
Build a static website (GitHub Pages) that displays all Michigan Dining hall menus in Chinese, auto-updated daily via GitHub Actions.

**Architecture:**
- `generate_site.py` — scrapes all 5 halls, translates to Chinese via Claude API, outputs a single `index.html`
- `.github/workflows/daily-menu.yml` — cron job that runs the generator and deploys to `gh-pages` branch
- Static HTML with embedded CSS (no JS framework needed) — mobile-friendly, clean design

---

## Phase 1: Site Generator Script

### Task 1.1 — Create `generate_site.py` core structure ✅
- [x] New script that imports from `scraper.py`
- [x] Fetch menus for all 5 halls (reuse `compare.py`'s concurrent fetching pattern)
- [x] Collect all unique item names across all halls/meals
- [x] Output: structured data ready for HTML rendering — preserve ALL item fields (name, traits, allergens, nutrition)
- [x] Do NOT use compact mode — keep full nutrition facts, allergens, and traits for every item
- **Notes:** Implemented in `generate_site.py`. Tested: fetches 338 items across 5 halls.

### Task 1.2 — Add Chinese translation via Claude CLI ✅
- [x] Batch-translate all unique item names (reuse `menu.py`'s `translate_menu` pattern)
- [x] Build a name→translation mapping dict
- [x] Apply translations to all menu data before rendering
- [x] Handle translation failures gracefully (fall back to English)
- **Notes:** Uses `claude -p` with haiku model. Translation cache implemented (Task 1.4 done here too).

### Task 1.3 — HTML template and rendering ✅
- [x] Generate a self-contained `index.html` with embedded CSS
- [x] Page structure: bilingual header, hall tabs, meals → stations → items
- [x] Dietary trait badges (color-coded pills)
- [x] Allergen list (inline)
- [x] Nutrition facts (expandable `<details>` per item)
- [x] Mobile-responsive design
- [x] Chinese fonts: Noto Sans SC via Google Fonts
- [x] Dark/light theme: auto `prefers-color-scheme` + manual toggle
- [x] Language toggle: 3-state (both / 中文 only / EN only)
- **Notes:** Single self-contained HTML, 357 lines. All features working.

### Task 1.4 — Translation cache ✅
- [x] Save translations to `site/translations_cache.json`
- [x] On next run, load cache first, only translate new/unseen items
- [x] Reduces API calls and speeds up daily runs
- **Notes:** Implemented in `translate_with_cache()`. Cache auto-created on first run.

---

## Phase 2: GitHub Actions & Deployment

### Task 2.1 — GitHub Actions workflow ✅
- [x] Create `.github/workflows/daily-menu.yml`
- [x] Schedule: `cron: '0 10 * * *'` (10:00 UTC = 5/6 AM ET, before breakfast)
- [x] Also trigger on `workflow_dispatch` for manual runs
- [x] Steps: checkout → Python 3.12 → pip install → Claude CLI → generate → cache commit → deploy
- **Notes:** Uses `actions/deploy-pages@v4` with `upload-pages-artifact@v3`. Needs `ANTHROPIC_API_KEY` secret in repo settings. Also needs Pages enabled (Settings → Pages → Source: GitHub Actions).

### Task 2.2 — GitHub Pages configuration ✅
- [x] Add deployment action that pushes `site/` to GitHub Pages
- [x] Manual trigger via `workflow_dispatch`
- **Notes:** Uses the newer GitHub Pages deployment via Actions (not gh-pages branch). Translation cache is auto-committed back to main.

---

## Phase 3: Polish & Enhancements

### Task 3.1 — Error handling & resilience ✅
- [x] Handle individual hall scrape failures (show "暂无菜单" instead of crashing) — already in fetch_all_halls
- [x] Handle Claude CLI not available (skip translation, show English only) — already in translate_names
- [x] Add retry logic for flaky network requests — 3 retries with exponential backoff in scraper.py
- [x] Batch translations in groups of 50 to avoid CLI timeouts on large menus
- **Notes:** scraper.py retries ConnectionError/Timeout/HTTPError. generate_site.py batches translations.

### Task 3.2 — Visual polish
- [ ] Add dietary filter toggles (JavaScript, minimal)
- [ ] Smooth transitions between hall tabs
- [ ] Print-friendly CSS

### Task 3.3 — SEO & metadata
- [ ] Proper `<meta>` tags (charset, viewport, description)
- [ ] Open Graph tags for social sharing
- [ ] Favicon

---

## Completed
- [x] Project enabled for Ralph
- [x] CLI scraper, menu viewer, and comparison tool built
- [x] Chinese translation via Claude CLI working in menu.py
- [x] Phase 1 complete: `generate_site.py` with concurrent fetching, translation (with cache), and full HTML rendering
- [x] Phase 2 complete: GitHub Actions workflow with daily cron + manual dispatch, deploys to GitHub Pages

## Notes
- The existing `translate_menu()` in `menu.py` is the reference for how to call Claude CLI for translation
- All 5 halls are defined in `scraper.py:DINING_HALLS`
- Concurrent fetching pattern exists in `compare.py:fetch_all()`
- GitHub Actions needs `ANTHROPIC_API_KEY` as a repository secret
- Start with Task 1.1 → 1.3 (get a working local generator), then 2.1 → 2.2 (deploy it)
