# SB 535 Community Flag

This note summarizes the addition of an SB 535 disadvantaged community flag to the tract-level screening dataset and map.

The SB 535 flag is intended to identify whether a tract is included in California's SB 535 disadvantaged communities census tract layer. It is not part of the emissions hazard calculation by default. Instead, it provides equity and program context that can help interpret screening results, especially where high hazard or relative screening scores coincide with communities already designated under SB 535.

## Source

The flag was derived from the official SB 535 disadvantaged communities census tract layer:

- Source: https://gis.data.ca.gov/api/download/v1/items/15b93bb7650943dab83038359b6240ec/csv?layers=1
- Catalog page: https://catalog.data.gov/dataset/sb-535-disadvantaged-communities-2022-census-tracts-bb685
- Publisher: California Office of Environmental Health Hazard Assessment / CalEPA
- Geography: California census tracts

## Derived Fields

The SB 535 fields were added at the tract level:

```text
sb535_flag
sb535_dac_category
```

The flag is a direct tract ID join:

```text
sb535_flag = 1 if the tract appears in the SB 535 disadvantaged communities census tract source
```

The `sb535_dac_category` field stores the source disadvantaged community category for flagged tracts.

## Bay Area Result

In the Bay Area scoring dataset, 116 of 1,463 tracts have `sb535_flag = 1`.

## Interpretation

The SB 535 flag should be used as contextual information rather than as an automatic score multiplier. A flagged tract does not necessarily have a higher emissions score, and an unflagged tract is not necessarily lower priority. The flag is most useful for identifying where the emissions-based screening results overlap with communities already recognized under California disadvantaged community screening policy.

A practical use is to summarize high-scoring tracts by SB 535 status, or to review whether tracts that move substantially under different score settings are also SB 535 disadvantaged communities. This can help connect technical score sensitivity with existing equity and policy context.