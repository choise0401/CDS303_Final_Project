# Crime-Type Profiles of Washington, D.C. Census Tracts in 2025

**Interpreting crime-based clusters with property assessment data**

**CDS 303 · Scientific Data Mining · Final Project**  
**Team members:** Seungeun Choi and Junhwa Hong

## Project Overview

Areas within the same city can have different patterns of reported crime. Our project examines Washington, D.C. census tracts and groups them according to the composition of their reported crimes in 2025. We will describe these groups using their original crime characteristics and examine whether their residential property characteristics differ.

Each row in our analysis table will represent one census tract. We will calculate the percentage of included crime records belonging to each offense category, standardize these percentages, and apply principal component analysis (PCA). We will then use K-means to group tracts with similar crime compositions. A Random Forest classifier and SHAP will help explain which original crime variables distinguish the resulting groups.

Property assessment records will provide additional context after properties have been assigned to the same census tracts. We will compare residential property types and assessed values across the crime-based groups. Property variables will not define the clusters or enter the main classifier.

## Research Question

> How do Washington, D.C. census tracts cluster based on the composition of their reported crimes in 2025, and which crime characteristics distinguish the resulting clusters?

Our additional exploratory question is:

> After linking property records to the same census tracts, how do the crime-based clusters differ in residential property-type composition and assessed residential property values?

The property comparison depends on the quality of address geocoding and tract assignment. Census tracts are statistical areas and do not necessarily match commonly understood neighborhood boundaries.

## Data

| Dataset | Provider and source | Unit in the source | Role in our project |
|---|---|---|---|
| Crime Incidents in 2025 | Metropolitan Police Department, through [Open Data DC](https://opendata.dc.gov/datasets/DCGIS::crime-incidents-in-2025) | Individual public crime record | Main dataset; offense percentages aggregated by census tract |
| Integrated Tax System Public Extract Facts | D.C. Office of Tax and Revenue / Office of the Chief Financial Officer, through [Open Data DC](https://opendata.dc.gov/datasets/DCGIS::integrated-tax-system-public-extract-facts-1/explore) | Property assessment record identified by SSL | Additional context; residential property types and assessed values |

The crime dataset's main fields are `OFFENSE`, `CENSUS_TRACT`, and `REPORT_DAT`. The selected property fields are `SSL`, `PROPERTY_ADDRESS`, `LAND_USE_DESCRIPTION`, and `APPRAISED_VALUE_CURRENT_TOTAL`.

We will aggregate each dataset separately and join the resulting tables by tract identifier. An individual crime record will not be matched to an individual property as though the property were the crime location.

## Analysis Plan

| Stage | Planned work |
|---|---|
| Preprocessing | Check the reporting period, identifiers, offense categories, missing values, and duplicate records; calculate tract-level offense percentages |
| Dimensionality reduction | Standardize crime percentages; use PCA to summarize variation and inspect component loadings |
| Clustering | Apply K-means; compare SSE, silhouette scores, cluster sizes, and stability across initializations |
| Classification and explanation | Predict cluster labels from the original crime percentages using Random Forest; interpret predictions with SHAP and cluster summaries |
| Property context | Geocode property addresses, assign compatible tract codes, and compare property summaries across crime clusters |

## Scope and Interpretation

This is a cross-sectional study of crime composition. We will not analyze changes over time or calculate population-adjusted crime rates. Offense percentages describe the mix of included records, so a high percentage of one offense does not by itself indicate a high crime rate or a less safe area.

The classifier will use the same original crime variables from which the clusters are derived. Its performance will therefore measure how well it reproduces the cluster assignments, rather than independently validating the clusters. SHAP will explain the classifier's predictions. Property assessments are not sale prices, and observed differences between clusters will not establish that crime causes changes in property values.

## Project Sessions

| Session | Content |
|---|---|
| [Session 1 — Paper Summary and Research Design](session1.md) | Four-step framework, research questions, variable selection, preprocessing plan, and additional property dataset |

Session 1 presents the research design and preliminary feasibility checks. Final cluster selection, Random Forest and SHAP results, and the property linkage will be reported in later sessions.
