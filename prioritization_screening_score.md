# Screening scoring options

This page summarizes the scoring options we have been discussing for the tract-level emissions.

At a high level, I think the key screening question is not just “where are emissions high?” and not just “where are toxicity-weighted values high?”, but rather “where do high emissions and high toxicity coincide?” The interaction-style approach seems to align most directly with that objective because it emphasizes tracts where both dimensions are elevated simultaneously.

I’ve shared the [resulting scores for comparison](https://carb.sharepoint.com/:x:/r/sites/AQPSD/CPIS/Data%20Visualization/Emission%20Mapping%20Tool/Working_Data/Bay%20Area%20Prototype%20Data/tract_eicsumn_pollutant_emissions_twe.csv?d=w0b7dd168e7504727b0433763e4d9f7e4&csf=1&web=1&e=T6IPCH), which may help illustrate how the different methods behave across tracts. The file currently includes raw toxics mass, toxicity-weighted emissions (TWE) endpoints, and several derived percentile scores.

You can also view an interactive map for comparing tract-level scores below:

[Open tract score comparison map](https://taufiqhassan.github.io/prioritization_screening_score/tract_score_maps_comparison.html)


## 1. Current toxics percentile score in the dashboard

```text
dash_toxics_score =
  0.25 × toxics_mass_percentile
+ 0.25 × cancer_twe_percentile
+ 0.25 × chronic_twe_percentile
+ 0.25 × acute_twe_percentile
```

This is a transparent weighted-average score. It treats mass and the three toxicity endpoints equally after converting each to percentile rank. This is conceptually similar to indicator aggregation methods such as CalEnviroScreen, where different pollution indicators are normalized before being combined.

Consideration:

- High mass can partly compensate for low toxicity, and high toxicity can partly compensate for low mass.

## 2. Raw product percentile score

```text
raw_toxics_score = percent_rank(toxics_mass × cancer_twe × chronic_twe × acute_twe) * 100
```

This approach strongly emphasizes tracts that are simultaneously high across all dimensions.

Consideration:

- Raw mass and TWE endpoints are on different scales and do not share a common physical unit

- TWE already includes emissions in its calculation, so this can effectively double-count emissions.

- It also collapses to zero if any one endpoint is zero.

## 3. Percentile geometric mean score

```text
geo_toxics_score =
( toxics_mass_percentile
× cancer_twe_percentile
× chronic_twe_percentile
× acute_twe_percentile
)^(1/4)
```

This improves interpretability relative to the raw product approach by combining normalized percentile scores instead of raw values. It still emphasizes tracts that are consistently elevated across all dimensions.

Consideration:

- Still collapses to zero if any component percentile is zero.

---

NOTE: The zero scores found in Options 2 and 3 are primarily driven by tracts with zero acute TWE values. One possible approach is to apply a small floor value to those cases to avoid the score collapsing to zero, and I included an additional option demonstrating that approach. However, since the zero values may legitimately reflect the absence of acute toxic emissions rather than missing data, I think it is important to be cautious about artificially inflating those tracts without a clear rationale.

---

## 4. Mass-toxicity interaction score

```text
interaction_toxics_score =
(toxics_mass_percentile / 100)
× (
(cancer_twe_percentile
+ chronic_twe_percentile
+ acute_twe_percentile) / 3 / 100
)
× 100
```

This score separates the problem into two concepts:

How much toxics mass is emitted?

How toxic are those emissions?

It then combines them as an interaction term. In practice, this means high mass matters most when toxicity is also high. A tract with high mass but relatively low toxicity is moderated, and likewise a tract with high toxicity but relatively low mass is also moderated.

To me, this approach seems to align most closely with the screening objective of identifying locations where both emissions magnitude and toxicity are elevated together.

I would be very interested in hearing everyone’s thoughts on these approaches, particularly whether there are considerations, tradeoffs, or unintended effects that I may be overlooking. I’d also appreciate any opinions on which method seems most appropriate for the type of screening objective we are trying to support.

---

I also looked at a few existing screening frameworks for comparison:

- EPA RSEI combines emissions with toxicity weighting, and in more advanced forms also incorporates exposure/dose and potentially exposed population. Conceptually, this supports the idea of interaction-style approaches.

- CalEnviroScreen converts heterogeneous indicators into normalized percentile-based scores before combining them into broader screening metrics. This is broadly consistent with the percentile normalization approach used in several of the options above.

- ToxPi aggregates normalized indicators into weighted components to create an overall prioritization score while maintaining interpretability across domains. This is conceptually similar to combining multiple normalized toxicity dimensions into a composite screening metric.

References:

- EPA RSEI: https://www.epa.gov/rsei/learn-about-rsei

- EPA RSEI toxicity weights: https://www.epa.gov/rsei/rsei-toxicity-weights

- CalEnviroScreen 4.0: https://oehha.ca.gov/calenviroscreen/report/calenviroscreen-40

- ToxPi software paper: https://bmcbioinformatics.biomedcentral.com/articles/10.1186/s12859-018-2089-2

- ToxPi R package introduction: https://toxpi.github.io/toxpiR/articles/introduction.html
