# AB 617 Community Flag

This note summarizes the addition of an AB 617 community flag to the tract-level screening dataset and map.

The AB 617 flag is intended to identify whether a tract overlaps a selected AB 617 Community Air Protection Program boundary. It is not part of the emissions hazard calculation by default. Instead, it provides policy and program context that can help interpret screening results, especially where high hazard or relative screening scores coincide with communities already identified through AB 617.

## Source

AB 617 community boundaries were derived from CARB's statewide boundary layer:

- Source layer: Final AB 617 Community and Emission Study Area Boundaries
- Feature service: https://services6.arcgis.com/x7ftScCDR8g2kVFB/arcgis/rest/services/Final_AB_617_Community_and_Emission_Study_Area_Boundaries/FeatureServer/0
- Geography: AB 617 community and emission study area boundaries

For the Bay Area tract score file, the matching AB 617 communities are:

- Bayview Hunters Point, Southeast San Francisco
- East Oakland
- Richmond, North Richmond, San Pablo
- West Oakland

## Derived Fields

The AB 617 fields were added at the tract level:

```text
ab617_flag
ab617_any_overlap_flag
ab617_overlap_pct
ab617_community
ab617_boundary_type
ab617_match_method
```

The current flag uses tract-area overlap:

```text
ab617_flag = 1 if at least 5 percent of the tract area overlaps an AB 617 boundary
```

The overlap percentage was calculated using polygon overlay in California Albers. The `ab617_any_overlap_flag` field is also retained so edge tracts with any overlap can be reviewed separately.

## Bay Area Result

In the Bay Area scoring dataset, 217 of 1,463 tracts have `ab617_flag = 1` using the 5 percent overlap threshold. A total of 228 tracts have some overlap with an AB 617 boundary.

| AB 617 community | Flagged tracts |
|---|---:|
| East Oakland | 72 |
| Bayview Hunters Point, Southeast San Francisco | 66 |
| Richmond, North Richmond, San Pablo | 48 |
| West Oakland | 25 |
| East Oakland; West Oakland | 6 |

## Interpretation

The AB 617 flag should be used as contextual information rather than as an automatic score multiplier. A flagged tract does not necessarily have a higher emissions score, and an unflagged tract is not necessarily lower priority. The flag is most useful for identifying where the emissions-based screening results overlap with existing community air protection work.

A practical use is to summarize high-scoring tracts by AB 617 status, or to review whether tracts that move substantially under different score settings are also inside AB 617 communities. This can help distinguish technical score sensitivity from places where additional program context may be important.
