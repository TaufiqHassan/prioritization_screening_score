# Weighted Hazard Sensitivity

This note summarizes how much the tract-level hazard ranking changes when the weighting is shifted away from the 50/50 base case.

The base hazard score treats emissions magnitude and toxicity potential as equal parts of the hazard concept:

```text
hazard_50_50 = 0.50 x E + 0.50 x T
```

where `E` is the merged emissions magnitude percentile and `T` is the average of the cancer, chronic, and acute TWE percentiles:

```text
T = (cancer_twe_percentile + chronic_twe_percentile + acute_twe_percentile) / 3
```

The toxicity-emphasis alternatives shift more weight to `T`:

```text
hazard_40_60 = 0.40 x E + 0.60 x T
hazard_30_70 = 0.30 x E + 0.70 x T
```

## Summary

The tract rankings are very stable under the 40/60 and 30/70 alternatives. Exact tract ranks often change by a few positions, but the high-priority groups remain largely the same.

| Scenario | Spearman vs 50/50 | Top 25 overlap | Top 50 overlap | Top 100 overlap |
|---|---:|---:|---:|---:|
| 40/60 | 0.9978 | 24 / 25 | 50 / 50 | 99 / 100 |
| 30/70 | 0.9912 | 23 / 25 | 48 / 50 | 95 / 100 |

Spearman correlation compares the ranked tract list under each alternative to the ranked tract list under 50/50. A value of 1.0 would mean the tract order is identical. Values above 0.99 indicate that the overall ordering is still very similar.


## Interpretation

The emissions magnitude component and toxicity potential component are strongly correlated across tracts:

```text
Spearman correlation between E and T = 0.819
Pearson correlation between E and T  = 0.820
```

This does not mean the same emissions are being counted twice. The two components use different pollutant subsets. `E` is based on criteria emissions plus toxic emissions for components without available TWE health values. `T` is based only on toxicity-weighted toxic emissions, using the cancer, chronic, and acute TWE endpoints. Criteria pollutants are not included in TWE.

The correlation is therefore better interpreted as a spatial/source co-location pattern. Tracts with larger or more active emitting sources often have both higher criteria or uncharacterized emissions and higher toxicity-weighted toxic emissions. Because those two hazard dimensions tend to identify similar places, shifting from 50/50 to 40/60 or 30/70 does not dramatically reorder the highest-priority tracts.

The tracts that move most are the places where `E` and `T` disagree. A tract with high emissions magnitude but low toxicity potential moves down when toxicity receives more weight. A tract with lower emissions magnitude but high toxicity potential moves up.

Only a small number of tracts have large disagreement between emissions magnitude and toxicity potential:

```text
Tracts where |E - T| > 50 percentile points: 27 of 1,463
Tracts where |E - T| > 70 percentile points: 2 of 1,463
```

## Suggested Use

Use 50/50 as the base hazard setting because it is easy to explain: emissions magnitude and toxicity potential receive equal importance. Use 40/60 and 30/70 as sensitivity scenarios. The sensitivity results support the conclusion that the priority pattern is robust under reasonable toxicity-emphasis alternatives.