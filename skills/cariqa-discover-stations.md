---
name: Discover charging stations near a location
description: Search for charging stations around a coordinate, render them on a map, filter by partner, and read live connector detail from the Cariqa Connect API.
api: openapi/cariqa-openapi-original.yml
operations:
  - stations_around_list
  - stations_details_retrieve
  - stations_partners_list
  - stations_tile_retrieve
---

# Discover charging stations near a location

Read-only discovery. Base URL `https://connect.cariqa.com/api/v1`, header
`Authorization: Bearer <token>`. Location data is available (real + fake) even in
the development environment.

## Steps

1. **List nearby stations** — `GET /api/v1/stations/around/`
   (`stations_around_list`) with `latitude`, `longitude`, and `distance`. Returns a
   paginated list (`count`, `next`, `previous`, `results`).
2. **Filter by partner (optional)** — `GET /api/v1/stations/partners/`
   (`stations_partners_list`) to retrieve valid partner/operator names to use as filters.
3. **Read station detail** — `GET /api/v1/stations/details/`
   (`stations_details_retrieve`) for full station info: EVSEs, live status, prices,
   opening times, amenities, and OCPI operator info.
4. **Render a map (optional)** — `GET /api/v1/stations/tile/`
   (`stations_tile_retrieve`) returns tile GeoJSON (FeatureCollection with price
   properties) for map visualization.

## Rules

- Live EVSE status can differ from real-time connector state; refresh detail before
  letting a user start charging.
- List responses are page-number paginated — follow `next` for more results.
- `423 Locked` on the partners endpoint means the resource is temporarily locked; retry later.
- `400` indicates invalid coordinates/parameters; `403` an invalid/expired token.
