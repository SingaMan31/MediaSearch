# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Anime/media discovery SPA — vanilla HTML/CSS/JS, no build step, no dependencies. Open `anisearch.html` directly in a browser.

## Running the App

```
# Open directly in default browser (PowerShell)
Start-Process anisearch.html

# Or drag anisearch.html into any browser
```

No build, no install, no dev server needed.

## Architecture

**Everything lives in `anisearch.html`** (~1,600 lines). The file is one HTML document with embedded `<style>` and `<script>` blocks.

The script is divided into sections separated by `═══` comment banners:

| Section | Lines (approx) | Purpose |
|---|---|---|
| Constants & State | 408–455 | API keys, mode flags, result arrays, caches |
| Utilities | 460–515 | Debounce, HTML escape, date formatting |
| Search History | 517–555 | LocalStorage, mode-namespaced |
| Mode Switching | 580–650 | Anime ↔ Movies & Series toggle |
| Card Rendering | 677–765 | `renderCards()` builds the grid |
| Filter / Sort | 784–890 | Applied client-side on the fetched result array |
| Search | 894–945 | Calls Jikan (anime) or TMDB (media), then `renderCards()` |
| Surprise Me | 949–990 | Random entry via Jikan/TMDB random endpoints |
| Trailers | 992–1035 | YouTube Data API v3 search |
| Anime Modal | 1049–1295 | Detail panel: characters, stats, recommendations |
| TMDB Modal | 1297–1540 | Detail panel: cast, providers, recommendations |
| Event Listeners | 1545–1600 | Search, filters, keyboard (Esc), mode toggle |
| Init | 1605–1611 | `DOMContentLoaded` bootstrap |

## State Model

Global variables (no framework):

```js
appMode       // 'anime' | 'media' — current mode
all[]         // anime results from last Jikan search
tmdbAll[]     // media results from last TMDB search
workMode      // bool — hides mature genres (Mystery, Psychological, etc.)
scoreOnly     // bool — filter to scored entries only
sortBy        // 'score' | 'popularity' | 'title' | 'episodes'
```

Caches (Maps):
- `animeDetailCache` — Jikan anime details keyed by MAL ID
- `tmdbDetailCache` — TMDB item details keyed by TMDB ID
- `ytCache` — YouTube trailer URLs keyed by query string

## External APIs

| API | Base URL | Key location in code |
|---|---|---|
| Jikan (MyAnimeList) | `https://api.jikan.moe/v4` | No key needed |
| TMDB | `https://api.themoviedb.org/3` | `TMDB_KEY` constant |
| YouTube Data v3 | `https://www.googleapis.com/youtube/v3` | `YT_KEY` constant |

Images served from `https://image.tmdb.org/t/p/` (TMDB CDN) and MyAnimeList CDN.

## UI Language

All UI text is in **Spanish**. Keep labels, button text, and error messages in Spanish when editing.

## Key Patterns

- **Search debounce**: 720 ms — controlled by `debounce()` utility; don't tighten it to avoid hammering rate-limited APIs.
- **Filtering is client-side**: `renderCards()` receives the full result array and applies `workMode`, `scoreOnly`, genre chips, and `sortBy` each time — no extra API call.
- **Modal open/close**: `openAnimeModal(id)` / `openTMDBModal(id, type)` fetch details on demand; `closeModal()` removes the overlay. ESC key is wired in the event listeners section.
- **Skeleton screens**: inserted as HTML strings before the fetch completes, replaced by real cards on resolution.
