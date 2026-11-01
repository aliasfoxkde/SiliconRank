# Data provenance, freshness, and interpretation

## Current data architecture

The current product data is curated in
`src/lib/data/seed.ts`. The UI derives normalized metrics from these
records; `scripts/build-api-data.cjs` bundles the seed and normalization
code into `mcp-server/data.json`.

This guide describes the checked-in implementation and must be revalidated when the seed schema, source registry, or MCP bundle pipeline changes.

This is a static-data pipeline, not a live ingestion service. The
`SourceRegistry.refreshInterval` field describes an intended refresh
cadence; it does not itself schedule collection or prove that a source was
refreshed on that cadence. Treat displayed values as the latest curated
snapshot present in the repository, not as live market data.

## Provenance fields

The type definitions in `src/types/index.ts` distinguish several kinds of
evidence:

| Record | Provenance fields | Interpretation |
|---|---|---|
| Benchmark result | `sourceId`, `rawScore`, optional `sampleSize`, optional `testConditions`, `confidence`, `observedAt` | A sourced observation. Compare only like-for-like benchmark versions and test conditions. |
| Specification snapshot | `sourceId`, reported specification values | A source-attributed specification snapshot. The current type does not include a per-field observation timestamp, so freshness is not independently represented for every spec. |
| Price snapshot | `sourceId`, `priceUsd`, `condition`, `region`, `observedAt`, optional `notes` | A dated price observation. Region and condition are part of the value, not optional decoration. |
| Derived metric | `formulaVersion`, `inputsJson`, `confidence`, `computedAt` | A calculated result. It is not a raw benchmark or vendor-reported specification. |
| Source registry | `baseUrl`, `sourceType`, optional `refreshInterval`, optional `licenseNote`, `reliabilityWeight`, `active` | Source metadata and a weighting policy; it is not evidence that a license has been granted or a refresh has run. |

## Measured, reported, estimated, and derived values

- **Measured benchmark:** a raw score recorded from a named benchmark source.
  Keep benchmark version, sample size, and test conditions with it.
- **Vendor-reported specification:** a value copied from a manufacturer source.
  It is not an independently measured result.
- **Estimated price/specification:** explicitly mark estimates; do not present
  them as current retail quotes or measured device performance.
- **Derived metric:** computed from underlying fields and a versioned formula.
  Show its confidence and formula provenance where it affects ranking.
- **Normalized score:** a transformed score for comparison within the declared
  benchmark category. It is not interchangeable with the raw score.

Price conditions currently include `new`, `used`, `msrp`,
`cloud-only`, `estimate`, and `oem`. Never compare price values without
considering condition, region, and observation date.

## Quality and coverage

The Data Quality panel reports field completeness and the share of records in
configured completeness bands. Completeness is not the same as factual
accuracy, source freshness, or independent verification. A record can be
complete and still contain an outdated or incorrect value.

## Refresh and publication policy

Before describing the dataset as current or live, a refresh pipeline needs to:

1. Fetch source data using documented, permitted interfaces.
2. Preserve source URL, retrieval timestamp, source record identifier, and
   license/attribution information.
3. Validate units, model identity, benchmark version, sample size, region,
   and price condition.
4. Distinguish measured values from estimates and vendor-reported values.
5. Record changes and rejected records for auditability.
6. Rebuild the static data bundle and verify the UI and MCP data agree.
7. Publish the data snapshot date and a reproducible diff.

Until that pipeline exists and has run, do not advertise the refresh intervals
in the source registry as actual automated update guarantees.
