# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A 100% client-side (no backend/build step) dashboard for public-security data across municipalities of Pará, Brazil. It loads a municipal GeoJSON, aggregates crime data in-browser, and renders it as an interactive Leaflet choropleth map with Chart.js charts, rankings, monthly history, regional comparisons, and a neighborhood-pressure analysis. There is also a Gemini-powered natural-language query assistant ("IA" tab).

The entire application logic (HTML + CSS + JS) lives in a single file: `index.html` (~4700 lines). There is no bundler, package.json, or build process — it's static files served as-is.

## Running locally

The app **must** be served over HTTP — opening `index.html` via `file://` will fail because the GeoJSON is loaded with `fetch()`.

```bash
python -m http.server 8080
# or
npx serve .
```

Then visit `http://localhost:8080`. There is no test suite, linter, or build command in this repo.

## Architecture

### Files
- `index.html` — the entire app: styles, markup, and all JS logic (map, charts, filters, AI assistant).
- `login.html` — standalone Firebase email/password login page. Redirects to `index.html` on success.
- `firebase-config.js` — Firebase project config, loaded by both `index.html` and `login.html`.
- `dados_seguranca_puplica_pa.geojson` — the data source. Each `Feature` represents one municipality **in one month** (not one row per municipality). See `README.md` for the expected properties shape.
- `stats_municipios_seguranca_pa.csv` — supplementary municipal stats.

### Auth flow
Both pages load the Firebase Auth compat SDK. `index.html` registers `firebase.auth().onAuthStateChanged`: if there's no user it redirects to `login.html`; otherwise it calls `loadAll()` to fetch and render data. `login.html` does the reverse — if already authenticated it redirects straight to `index.html`. There is no server-side session/auth check; this is client-side gating only, consistent with the "no backend" design (the exposed Firebase and Gemini API keys are intentional for a static site of this kind, not accidental leaks).

### Data pipeline (all in `index.html`)
1. `loadAll()` fetches `GEOJSON_URL` and builds in-memory structures: `allFeatures`, `municipalData`/`municipalById`, `regionalData`, `stateSummary`, `stateMonthly`.
2. `FIELD_MAP` maps logical field names (`cidade`, `regional`, `cvli`, `populacao`, etc.) to the actual GeoJSON property keys — change this if the source data schema changes.
3. `METRICS` defines the six selectable indicators (CVLI/Furto/Roubo, each as raw count and as density per 100k) with color, formatter, and label — used to drive metric-selector buttons across all tabs.
4. `activeFilters` (regional/cidade/faixa/ano/mes) drives cascading filter dropdowns; `rebuildFilteredData()` recomputes `filteredMunicipalData`/`filteredMonthly`/`filteredRegionalData`/`filteredSummary` whenever filters change, and every tab re-renders from those filtered globals.

### UI structure
The right-hand panel is tab-based (`switchTab()`), with tabs: VISÃO (map + KPIs + narrative), RANKING, HISTÓRICO, REGIONAIS, VIZINHANÇA (neighborhood pressure: `município − média dos vizinhos`, epicentro vs. ilha de segurança), COMPARATIVO, and IA (AI chat). Charts are built with module-level `Chart.js` instances (`cCompare`, `cRank`, `cSerieOc`, `cSerieDen`, `cRegionais`, `cPizzaTipos`, `cCmpMensal`, `cCmpTrim`) that get destroyed/recreated on theme change (`rebuildChartsForTheme()`) and data change.

Theme (light/dark) is stored in `localStorage` and applied via `data-theme` attribute on `<html>`; CSS custom properties in `:root` / `:root[data-theme="light"]` drive all colors, including chart colors via `chartColors()`.

### AI assistant ("IA" tab)
Natural-language questions are turned into a structured query plan and executed against the in-memory data — no server round-trip beyond the Gemini call itself:
- `buildRagIndex()` builds normalized lookup lists (cidades, regionais, faixas, meses, anos) from the loaded data for fuzzy matching (`fuzzyMatch`, `matchCity`, `matchRegional`, `matchFaixa`).
- `parseQuestionToPlan()` calls Gemini (`callGemini`) to get a JSON plan, validated/cleaned by `sanitizePlan()`; if Gemini fails or returns something unusable, `fallbackPlan()` derives a plan heuristically from the question text (`detectMetric`, `detectAggregation`, `detectTopN`, `detectCityFromQuestion`, `detectRegionalFromQuestion`, `extractMonth`, `extractYear`).
- `executePlan()` filters features (`filterFeaturesByPlan`), aggregates (`aggregateByMunicipio`, `uniquePopulationForFeatures`), and returns a result object that's formatted (`formatMetricValue`) and rendered into the chat thread (`addTurn`/`renderThread`), optionally highlighting municipalities on the map via `aiHighlightIds`.

### Conventions to preserve when editing
- Keep new logic inside the existing single-file structure and section banners (`/* ═══ ... ═══ */` comments) rather than splitting into new files, unless asked to restructure.
- Reuse `FIELD_MAP` / `METRICS` when touching data fields or indicators instead of hardcoding GeoJSON property names.
- Any new UI text should be Portuguese (pt-BR), matching the rest of the app.
