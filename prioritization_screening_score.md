# Screening scoring options

This page summarizes the scoring options we have been discussing for the tract-level emissions.

At a high level, I think the key screening question is not just “where are emissions high?” and not just “where are toxicity-weighted values high?”, but rather “where do high emissions and high toxicity coincide?” The interaction-style approach seems to align most directly with that objective because it emphasizes tracts where both dimensions are elevated simultaneously.

I’ve shared the [resulting scores for comparison](https://carb.sharepoint.com/:x:/r/sites/AQPSD/CPIS/Data%20Visualization/Emission%20Mapping%20Tool/Working_Data/Bay%20Area%20Prototype%20Data/tract_eicsumn_pollutant_emissions_twe.csv?d=w0b7dd168e7504727b0433763e4d9f7e4&csf=1&web=1&e=T6IPCH), which may help illustrate how the different methods behave across tracts. The file currently includes raw toxics mass, toxicity-weighted emissions (TWE) endpoints, and several derived percentile scores.

You can also view an interactive map for comparing tract-level scores below:

[Open tract score comparison map](https://taufiqhassan.github.io/prioritization_screening_score/tract_score_maps_comparison.html)

## Updated weighted screening framework

A recent direction expands the toxics-only scoring discussion into a broader relative community emissions screening score. Under this framing, the main screening question is where emissions magnitude, toxic potential, and potential population exposure coincide. The score is still intended to be emissions-focused: it is not a health risk assessment, and it is not intended to replace environmental justice tools such as CalEnviroScreen. Instead, it provides an inventory-based perspective on which communities may warrant additional evaluation, emissions-reduction efforts, or air monitoring.

The proposed overall structure is:

```text
relative_screening_score = hazard_score × potential_exposure_score
```

The framework has two main parts. The hazard score represents what is being emitted and how toxic those emissions may be. The potential exposure score represents where larger populations, especially children and seniors, coincide with those hazard scores.

One weighted hazard version discussed was:

```text
hazard_score =
  0.20 × ME
+ 0.35 × CE
+ 0.25 × NCC
+ 0.20 × NCA
```

where `ME` is the merged emissions magnitude percentile, including criteria emissions and toxic emissions without available health values; `CE` is the cancer TWE percentile; `NCC` is the non-cancer chronic TWE percentile; and `NCA` is the non-cancer acute TWE percentile.

One concern with a simple equal-weight baseline is that the three TWE endpoints collectively receive most of the influence:

```text
hazard_score = 0.25 × E + 0.25 × CE + 0.25 × NCC + 0.25 × NCA
```

In that case, toxicity-related indicators collectively account for 75 percent of the hazard score, while emissions magnitude accounts for 25 percent. A cleaner interpretation may be to treat hazard as two major dimensions: emissions magnitude and toxicity potential.

```text
hazard_score = 0.50 × E + 0.50 × T

T = (CE + NCC + NCA) / 3
```

This is equivalent to:

```text
hazard_score =
  0.50 × E
+ 0.167 × CE
+ 0.167 × NCC
+ 0.167 × NCA
```

This gives emissions magnitude and toxicity potential equal importance, while allowing the three toxicity endpoints to share the toxicity portion of the score. Alternative weighting scenarios, such as 40/60 or 30/70 splits between emissions magnitude and toxicity potential, can be used to test whether community rankings are robust to reasonable weighting assumptions.

For potential exposure, the discussed structure uses population groups as percentile-based indicators:

```text
potential_exposure_score =
  0.40 × children
+ 0.40 × seniors
+ 0.20 × remaining_population
```

Companion notes document these additions in more detail. The [weighted hazard sensitivity note](weighted_hazard_sensitivity.md) compares the 50/50 base hazard setting with 40/60 and 30/70 toxicity-emphasis alternatives. The [population exposure and relative screening score note](population_exposure_relative_screening_score.md) documents the ACS population source, the Bay Area percentile basis, and how the potential exposure score is combined with the hazard score. The [AB 617 community flag note](ab617_community_flag.md) documents how AB 617 boundaries were joined to tracts and how the flag should be interpreted as program context.

Because the weights are policy and screening choices rather than statistical estimates, the proposed data assessment should test whether the highest-priority communities are stable under plausible alternatives. Spearman correlation can be used to compare the hazard indicators with each other and to compare baseline and alternative weighted scores. This is particularly useful for checking whether cancer, chronic, and acute TWE percentiles are highly correlated, since highly correlated endpoint indicators may collectively exert more influence than intended.

## 1. Current toxics percentile score in the dashboard

```text
dash_toxics_score =
  0.25 × toxics_mass_percentile
+ 0.25 × cancer_twe_percentile
+ 0.25 × chronic_twe_percentile
+ 0.25 × acute_twe_percentile
```

This is a transparent weighted-average score. It treats mass and the three toxicity endpoints equally after converting each to percentile rank. This is conceptually similar to indicator aggregation methods such as CalEnviroScreen, where different pollution indicators are normalized before being combined.

**Consideration:**

- High mass can partly compensate for low toxicity, and high toxicity can partly compensate for low mass.

## 2. Raw product percentile score

```text
raw_toxics_score = percent_rank(toxics_mass × cancer_twe × chronic_twe × acute_twe) * 100
```

This approach strongly emphasizes tracts that are simultaneously high across all dimensions.

**Consideration:**

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

**Consideration:**

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

1. How much toxics mass is emitted?

2. How toxic are those emissions?

It then combines them as an interaction term. In practice, this means high mass matters most when toxicity is also high. A tract with high mass but relatively low toxicity is moderated, and likewise a tract with high toxicity but relatively low mass is also moderated.

To me, this approach seems to align most closely with the screening objective of identifying locations where both emissions magnitude and toxicity are elevated together.

I would be very interested in hearing everyone’s thoughts on these approaches, particularly whether there are considerations, tradeoffs, or unintended effects that I may be overlooking. I’d also appreciate any opinions on which method seems most appropriate for the type of screening objective we are trying to support.

---

I also looked at a few existing screening frameworks for comparison:

+ **EPA RSEI** combines emissions with toxicity weighting, and in more advanced forms also incorporates exposure/dose and potentially exposed population. Conceptually, this supports the idea of interaction-style approaches.

+ **CalEnviroScreen** converts heterogeneous indicators into normalized percentile-based scores before combining them into broader screening metrics. This is broadly consistent with the percentile normalization approach used in several of the options above.

+ **ToxPi** aggregates normalized indicators into weighted components to create an overall prioritization score while maintaining interpretability across domains. This is conceptually similar to combining multiple normalized toxicity dimensions into a composite screening metric.

References:

- EPA RSEI: https://www.epa.gov/rsei/learn-about-rsei

- EPA RSEI toxicity weights: https://www.epa.gov/rsei/rsei-toxicity-weights

- CalEnviroScreen 4.0: https://oehha.ca.gov/calenviroscreen/report/calenviroscreen-40

- ToxPi software paper: https://bmcbioinformatics.biomedcentral.com/articles/10.1186/s12859-018-2089-2

- ToxPi R package introduction: https://toxpi.github.io/toxpiR/articles/introduction.html
