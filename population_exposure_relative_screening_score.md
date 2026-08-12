# Population Exposure and Relative Screening Score

This note summarizes the population addition to the tract-level screening dataset and how it is used with the hazard score.

The population fields are intended to add a potential exposure dimension to the emissions-focused hazard score. They do not represent measured exposure, modeled concentration, health risk, or population vulnerability. They only indicate where larger numbers of people in selected age groups coincide with higher emissions hazard scores.

## Population Source

Population fields were derived from the Census ACS 2024 5-year tract table for California:

- Source: https://www2.census.gov/programs-surveys/acs/replicate_estimates/2024/data/5-year/140/B01001_06.csv.zip
- Table: B01001, Sex by Age
- Geography: California census tracts

## Potential Exposure Score

The potential exposure score gives more weight to children and seniors while still retaining the rest of the population:

```text
potential_exposure_score =
  0.40 x children_percentile
+ 0.40 x seniors_percentile
+ 0.20 x remaining_population_percentile
```

## Relative Screening Score

The relative screening score combines the base hazard score with potential exposure:

```text
relative_screening_score = hazard_score x potential_exposure_score / 100
```

The division by 100 keeps the combined score on a 0-100 scale for mapping and comparison. Without this scaling, the raw product would range from 0 to 10,000.

## Interpretation

The relative screening score is highest where emissions hazard and potential population exposure are both high. A tract with a high hazard score but low population can move down after exposure is included. A tract with moderate hazard but a large population, especially children or seniors, can move up.

