# Repository Analysis

## Snapshot
- **Project type:** Home Assistant custom integration (`custom_components/leneda`) plus a standalone Node server and a separate Vite frontend source tree.
- **Current integration version:** `2.0.3`.
- **Primary domain:** Lux energy metering data (electricity + gas), including community-sharing metrics and dashboard visualization.

## Architecture Overview
1. **Home Assistant integration backend (Python):**
   - Entry setup/unload in `__init__.py`.
   - API client in `api.py`.
   - Data orchestration and caching in `coordinator.py`.
   - Entity exposure in `sensor.py`.
   - Config UI flow in `config_flow.py`.
   - HTTP endpoints and panel/static serving in `http_api.py` + `panel.py`.
   - Persistent billing/config state in `storage.py`.

2. **Frontend delivery model:**
   - Prebuilt static bundle shipped in `custom_components/leneda/frontend`.
   - Source frontend in `frontend-src` (TypeScript + Vite).
   - Home Assistant panel serves `index.html` and static assets from integration package.

3. **Standalone mode:**
   - `standalone/server.js` serves the same built frontend outside Home Assistant and proxies Leneda API requests.

## Functional Coverage
- Multi-meter support with meter-type routing (consumption/production/gas).
- OBIS-code-driven sensor model for electricity and gas.
- Aggregated period metrics (daily/weekly/monthly; some yearly live fetch handling).
- Reference power exceedance calculations for billing context.
- Dedicated dashboard panel and associated JSON API endpoints.

## Strengths
- **Clear separation of concerns:** API client, coordinator, sensors, config flow, HTTP panel/API are split by responsibility.
- **Dual deployment model:** Works as HA panel and standalone web app.
- **Good user-facing feature breadth:** Gas support, sharing metrics, billing/exceedance context, rich dashboard UX.
- **Security checks present in static serving:** Directory traversal/path escape checks in panel static view.

## Risks / Technical Debt
- **Large coordinator complexity:** `coordinator.py` centralizes many calculations and fetch paths, increasing maintenance burden.
- **Potential duplicate source of truth:** Frontend build artifacts committed in integration package while source lives in `frontend-src`; release process discipline is required to avoid drift.
- **Sparse visible automated tests:** No explicit test suite directory detected in repo layout.
- **Runtime coupling to HA internals:** expected for integration, but reduces local/offline testability unless abstractions are added.

## Suggested Next Steps (Prioritized)
1. Add focused automated tests for:
   - OBIS-to-meter routing,
   - key coordinator calculations (self-consumption/export/exceedance),
   - HTTP API range responses.
2. Split coordinator logic into smaller modules (fetch, transforms, billing math) to reduce cognitive load.
3. Add CI checks for frontend-build freshness (e.g., verify committed `custom_components/leneda/frontend/assets` matches `frontend-src` build output).
4. Add developer docs for release workflow (build frontend, copy artifacts, bump manifest/changelog).

## Operational Notes
- Repository includes screenshot assets and branding files used in docs/UI.
- `hacs.json` and `manifest.json` indicate HACS-distributed custom integration packaging.
