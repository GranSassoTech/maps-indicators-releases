# Format of an indicator release

Schema version: `1`.

## Indicator CSV — `<set_id>.csv`

UTF-8, comma-separated, header on the first line, `\n` line endings. Long format: one
row per territory, indicator and year. Rows are sorted by `indicator`,
`territory_code`, `year`.

| Column | Type | Content |
|---|---|---|
| `territory_code` | text | ISTAT code of the territory, zero-padded (comune 6 digits, provincia 3, regione 2, SLL 5) |
| `indicator` | text | indicator identifier, declared in the metadata file |
| `year` | integer | reference year |
| `value` | number | computed value; integers are written without decimals |

Territories with no value for a year have no row for that year (no empty values).
`(territory_code, indicator, year)` is unique.

## Metadata file — `<set_id>.metadata.json`

```jsonc
{
  "schema_version": "1",
  "set_id": "abitazioni_occupate_residenti",
  "version": "1.0.0",                       // version of the rule
  "generated_at": "2026-11-09T10:00:00+00:00",
  "format": { "columns": ["territory_code", "indicator", "year", "value"], "layout": "..." },

  "reference_geography": {
    "territory_type": "comune",
    "code_column": "territory_code",
    "code_scheme": "ISTAT",
    "as_of": "2025-12-31",                  // null when the data is not reconciled
    "description": "..."
  },

  "indicators": [{
    "id": "abitazioni_occupate_residenti",
    "title": "Dwellings occupied by residents",
    "definition": "...",
    "unit": "dwellings",
    "version": "1.0.0",
    "source_variables": [                   // Datastore variables, every dimension pinned
      { "attribute": "it1_df_dcss_hcue_com_8_com_1_0",
        "column_name": "occ_2021",
        "dimension_filters": { "indicator": "DW_RESPERS", "freq": "A" } }
    ],
    "rule": "indicators/abitazioni_occupate_residenti/indicator.py",
    "rationale_file": "indicators/abitazioni_occupate_residenti/rationale.md",
    "rationale": "..."                      // full text of the rationale
  }],

  "gold_read": {                            // what was asked to the gold layer, and when
    "mode": "gold request",
    "base_url": "https://datastore.maps.gransassotech.org",
    "requested_at": "...",
    "read_at": "...",
    "request_id": 1234,
    "table_name": "ind_abitazioni_occupate_residenti_20261109100000",
    "territory_type": "comune",
    "year_from": 2011,
    "year_to": 2021,
    "reconcile_to_current": true,
    "variables": [ /* same objects as source_variables */ ]
  },

  "code": {                                 // the code that produced the file
    "tool_version": "0.1.0",
    "module": "indicators/.../indicator.py",
    "commit": "<git sha of the indicators repository>",
    "dirty": false
  },

  "files": {
    "output":           { "name": "<set_id>.csv", "sha256": "...", "rows": 15808 },
    "extract":          { "name": "<set_id>__gold_extract.csv", "sha256": "...",
                          "canonical_sha256": "...", "rows": 15808 },
    "extract_metadata": { "name": "<set_id>__gold_extract_metadata.txt", "sha256": "..." }
  },

  "warnings": [ "known gaps found by the rule's checks" ],

  "release": {
    "tag": "abitazioni_occupate_residenti-v1.0.0-20261109",
    "repository": "https://github.com/GranSassoTech/maps-indicators-releases",
    "assets": { "output": "https://...", "metadata": "https://...",
                "extract": "https://...", "extract_metadata": "https://..." }
  }
}
```

### Checksums

- `sha256`: SHA-256 of the file bytes. Verifies a downloaded asset.
- `extract.canonical_sha256`: SHA-256 of the extract with rows and columns sorted. Used
  to tell whether the gold layer has changed since the release, independently of row
  order.

### Reference geography

With `reconcile_to_current: true` the gold layer brings every year to the territories in
force on 31/12/2025: values of municipalities merged since are added to the
municipality they merged into. Municipalities created by detaching territory are not
reconciled; the rationale of each indicator says how they are treated.

## Tags

`<set_id>-v<rule version>-<gold read date YYYYMMDD>`. A tag is never moved or reused.

## Later: entering the data catalogue

If the indicators have to enter the MAPS data catalogue, a release is ingested like any
other source: the raw source is the release asset URL, the metadata file feeds
OpenMetadata and the lineage to the source variables. This format is fixed now so that
step stays mechanical.
