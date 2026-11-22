[← Project home](index.md)

# Session 1 — Paper Summary and Research Design

**CDS 303 · Scientific Data Mining · Final Project**  
**Team members:** Seungeun Choi and Junhwa Hong

## 1. Understanding the Explainable Cluster Analysis Framework

We read *A comprehensive framework for explainable cluster analysis* and *Uncovering student profiles. An explainable cluster analysis approach to PISA 2022*. The first paper introduces a four-step framework and its implementation in the Clust-learn Python package. The second applies the framework to student data and shows how numerical cluster labels can become interpretable profiles. Together, they explain both how to build clusters and how to describe what separates them. [R1, R2]

### 1.1 Purpose of Each Step

| Step | Purpose | Approach described in the papers | Our planned application |
|---|---|---|---|
| Data preprocessing | Reduce the influence of missing values, errors, and outliers on the groups found later | A staged approach to missing-data treatment, including deletion and imputation; Isolation Forest for outlier detection | Check crime records, reporting dates, tract codes, and offense categories before calculating tract-level offense percentages |
| Dimensionality reduction | Summarize related variables with fewer, interpretable dimensions | Sparse PCA (SPCA) for numerical variables and multiple correspondence analysis (MCA) for categorical variables | Standardize the crime percentages and apply PCA; inspect explained variance and component loadings |
| Clustering | Group observations with similar characteristics | K-means, with cluster validity measures and descriptions using internal and external variables | Apply K-means in the retained PCA space and compare SSE, silhouette scores, cluster sizes, and consistency across initializations |
| Classification for explainability | Identify which original characteristics help distinguish the generated clusters | A supervised classifier using cluster labels as the target, followed by SHAP explanations | Train Random Forest on the original crime percentages and interpret its predictions using SHAP and cluster summaries |

These steps address different problems. Good preprocessing cannot tell us how many clusters are meaningful, and a good clustering score cannot tell us what a cluster represents. We need the complete workflow to connect the statistical output to our research question. [R1]

### 1.2 Why Cluster Labels Need Interpretation

A cluster number such as 0 or 1 has no substantive meaning on its own. K-means can produce the requested number of groups even when separation is weak. We therefore need to examine whether the groups reflect useful patterns or are mainly affected by scaling choices, rare offenses, or unusual observations.

Validity measures are useful diagnostics. For example, silhouette scores summarize how close observations are to their own cluster relative to other clusters. However, they do not establish that a grouping has a meaningful interpretation. We will also examine each cluster's original offense percentages, distributions, and number of tracts before choosing descriptive names.

The PISA study illustrates this distinction. It analyzed 30,800 students in Spain, used 74 contextual variables, reduced them to 16 derived variables, and identified seven student profiles. The authors interpreted the profiles using their characteristics and SHAP explanations rather than reporting only cluster numbers. Their XGBoost model achieved a test accuracy of 0.8643. These results demonstrate an application of the framework; they do not determine the appropriate cluster count or expected classification performance for our crime dataset. [R2]

### 1.3 How Classification Helps Characterize Clusters

After clustering, the generated cluster labels become the target of a supervised model. In our design, the original tract-level crime percentages will be the predictors, and `cluster_label` will be the target. The classifier will learn which combinations of crime characteristics reproduce the group assignments.

Global SHAP summaries will help identify variables that influence the classifier across observations. Local SHAP values will help describe contributions to a particular tract's predicted cluster. We will interpret these alongside the actual offense distributions within each cluster. An influential predictor alone is not enough to name a group without checking the underlying values.

Because the same original crime variables are used to derive and explain the clusters, high classification accuracy will indicate successful reproduction of the assignments. It will not independently validate the clustering or measure the prediction of future crimes. SHAP will explain the Random Forest predictions, rather than establish a causal explanation of K-means assignments.

### 1.4 Adapting the Framework to Our Project

Our implementation will use PCA, K-means, Random Forest, and SHAP. We retain the four-step structure while choosing a simpler set of methods for the project. PCA and SPCA are different methods, as are Random Forest and XGBoost; we are adapting the workflow rather than reproducing the papers' exact implementation.

Property characteristics will serve as external context because they are not used to construct the crime clusters. They may help us describe differences between the resulting groups, but they will not enter the main classifier.

## 2. Research Questions and Study Design

### 2.1 Main Research Question

> How do Washington, D.C. census tracts cluster based on the composition of their reported crimes in 2025, and which crime characteristics distinguish the resulting clusters?

### 2.2 Additional Exploratory Question

> After linking property records to the same census tracts, how do the crime-based clusters differ in residential property-type composition and assessed residential property values?

| Sub-question | Evidence we will examine |
|---|---|
| How does crime composition differ across tracts? | Offense percentages and the total number of included records in each tract |
| Which tracts have similar crime compositions? | PCA-space clusters and their original offense distributions |
| Which crime characteristics distinguish the groups? | Cluster summaries, Random Forest predictions, and SHAP values |
| How do property characteristics vary across crime clusters? | Type-specific assessed-value summaries and residential record composition after checking linkage quality |

### 2.3 Unit of Analysis and Scope

Each row of the modeling table will represent one census tract with valid included crime records. Census tracts are statistical geographic units and may differ from everyday neighborhood boundaries. We will aggregate the crime records and property records separately before joining them by compatible tract identifiers.

This is a cross-sectional analysis of the 2025 crime composition, not a time-series analysis. `REPORT_DAT` will define the reporting period rather than generate monthly trends, weekend measures, or nighttime indicators. We will not use population data, so the offense percentages will not represent crimes per resident. They will describe the composition of included records rather than a neighborhood safety ranking.

The crime data will define the clusters. The property data will provide additional context, subject to successful geocoding and compatible geographic assignment. The design does not estimate the causal effect of crime on property values.

## 3. Main Dataset: Crime Incidents in 2025

| Item | Description |
|---|---|
| Dataset | Crime Incidents in 2025 |
| Provider | District of Columbia Metropolitan Police Department (MPD), through Open Data DC |
| Source | [Official dataset page](https://opendata.dc.gov/datasets/DCGIS::crime-incidents-in-2025) |
| Source observation | Individual public crime record |
| Analysis observation | Census tract with valid included records |
| Main fields | `OFFENSE`, `CENSUS_TRACT`, `REPORT_DAT` |
| Supporting fields | `CCN`, `OBJECTID`, `BLOCK`, `LATITUDE`, `LONGITUDE` |
| Study period | Records reported during 2025, checked against the source date definitions |

The public dataset covers selected DC Index Crime categories. It does not contain every offense or unreported crime. A report date can differ from the date an offense occurred. The published locations are generalized to the street-block level, so they cannot identify an exact affected property. We will follow the source's date definitions and check its time convention before filtering records. [R3]

We will retain the downloaded CSV as a fixed analysis snapshot and record its download date, row count, reporting period, and field names. Service counts can change as the source is updated.

`NEIGHBORHOOD_CLUSTER`, if present, is an existing geographic designation. It is different from the `cluster_label` generated by our model and will not be treated as the correct answer for our clustering.

## 4. Variable Selection and Preparation

### 4.1 Selected Source Fields

| Field | Description and unit | Type | Proposed role | Rationale and preparation |
|---|---|---|---|---|
| Crime: `OFFENSE` | Offense category; no numeric unit | Categorical | Source for clustering and classification inputs | Convert category counts into tract-level percentages; do not assign arbitrary ordered numeric codes |
| Crime: `CENSUS_TRACT` | Tract identifier; no numeric unit | Categorical identifier | Aggregation and linkage; excluded from inputs | Keep as a string; check missing values, whitespace, and identifier format |
| Crime: `REPORT_DAT` | Report date and time | Datetime | Contextual variable / period filter | Confirm the 2025 reporting window and time convention |
| Crime: `CCN` | Report identifier | Categorical identifier | Quality checks; excluded from inputs | Investigate repeated identifiers before deciding whether records are duplicates |
| Crime: `OBJECTID` | Source service row identifier | Identifier | Quality checks; excluded from inputs | Use for row inspection; do not assume permanent identity across extracts |
| Crime: `BLOCK` | Published street-block location | String | Location context; excluded from inputs | Do not treat it as an exact property-address match |
| Crime: `LATITUDE`, `LONGITUDE` | Published coordinates, degrees | Numerical | Location checks; excluded from inputs | Check suspicious locations; use valid source tract codes for aggregation |
| Tax: `SSL` | Property record identifier | Categorical identifier | Record tracking and linkage; excluded from inputs | Preserve as a string; reconnect geocoding results to property records |
| Tax: `PROPERTY_ADDRESS` | Address of the assessed property | String | Geographic assignment; excluded from inputs | Standardize and geocode the property address, not the owner's mailing address |
| Tax: `LAND_USE_DESCRIPTION` | Property-use category | Categorical | Contextual variable | Identify residential types and distinguish units from whole buildings |
| Tax: `APPRAISED_VALUE_CURRENT_TOTAL` | Current total assessed value, USD | Numerical | Contextual variable | Check zeros, assessment period, and record type; distinguish assessed value from sale price |
| Tax: `OTR_NEIGHBORHOOD_CODE`, `OTR_NEIGHBORHOOD_NAME` | Assessment-area identifiers and names | Categorical | Context and geographic checks | Do not assume these areas equal census tracts or crime neighborhood clusters |

We will exclude identifiers and location codes from the model inputs because their numeric appearance does not make them meaningful distance-based features. Crime fields such as `SHIFT`, `START_DATE`, and `END_DATE` will not enter the basic composition model. Other geographic identifiers, including ward, ANC, and police district, may support presentation but will not be mixed with tract-level predictors. `METHOD` will also be excluded from the basic inputs so that the initial design stays focused on offense composition.

Owner names, mailing addresses, tax amounts, sale prices, and prior or proposed assessments are not needed for the basic design. We will preserve the raw file and select the needed columns in a separate analysis table.

### 4.2 Derived Crime Variables

For each tract, we will divide the number of records in an offense category by the total number of included records in that tract and multiply by 100.

**Offense percentage = category record count ÷ total included tract record count × 100.**

| Derived variable | Meaning and unit | Type | Role | Rationale / preparation |
|---|---|---|---|---|
| `robbery_share` | Percentage classified as `ROBBERY` | Numerical | Clustering and classification input | Represent robbery within the tract's crime composition |
| `burglary_share` | Percentage classified as `BURGLARY` | Numerical | Clustering and classification input | Distinguish burglary from other theft categories |
| `motor_vehicle_theft_share` | Percentage classified as `MOTOR VEHICLE THEFT` | Numerical | Clustering and classification input | Represent theft of vehicles themselves |
| `theft_from_auto_share` | Percentage classified as `THEFT F/AUTO` | Numerical | Clustering and classification input | Represent theft from vehicles, separately from vehicle theft |
| `other_theft_share` | Percentage classified as `THEFT/OTHER` | Numerical | Clustering and classification input | Capture the remaining theft category in the source |
| `adw_share` | Percentage classified as `ASSAULT W/DANGEROUS WEAPON` | Numerical | Clustering and classification input | Represent this assault category; it is not limited to firearm incidents |
| `homicide_share` | Percentage classified as `HOMICIDE` | Numerical | Candidate input, subject to rarity review | Check sparse counts and their influence after standardization |
| `sex_abuse_share` | Percentage classified as `SEX ABUSE` | Numerical | Candidate input, subject to rarity review | Check the source definition, sparse counts, and influence |
| `arson_share` | Percentage classified as `ARSON` | Numerical | Candidate input, subject to rarity review | Check whether very few records create unstable, very small clusters |
| `crime_record_count` | Total included records in the tract, records | Numerical | Contextual variable | Show the sample size behind each percentage; not population-adjusted |
| `cluster_label` | Cluster assignment generated later | Categorical | Classification target only | Must never be included as a predictor |

The denominator will include all offense categories retained in the study population. Excluding a rare category from the model inputs will not silently change that denominator. Any category combination or exclusion will be documented with its justification. We will not combine substantively different offenses only because their counts are low.

### 4.3 Why These Variables Fit the Question

Percentages let us compare crime composition rather than only crime volume. Two tracts can have the same robbery percentage while having very different numbers of total records. We will therefore report `crime_record_count` alongside composition summaries.

The complete set of offense percentages sums to 100%, making the variables linearly dependent. PCA can summarize the resulting variation, but it does not remove every issue associated with compositional data. Standardization can also give rare categories substantial influence. We will examine these effects when deciding which variables and components to retain.

### 4.4 Preprocessing and Modeling Plan

1. **Fix the analysis snapshot.** Preserve the raw crime CSV and record its download date, columns, row count, and reporting range.
2. **Inspect record quality.** Check missing offense categories, invalid tract codes, and duplicate records. A repeated CCN alone will not justify deleting a row without inspecting the records.
3. **Aggregate by tract.** Count each offense category within each valid tract. A category absent within an observed tract receives a count of zero; a missing tract identifier does not become a zero-crime tract.
4. **Review small denominators.** Tracts with few records can have unstable percentages. A minimum of 20 records is a preliminary sensitivity-check threshold, not a confirmed final rule. We will compare the included tracts and results under alternative choices.
5. **Define coverage.** The crime file identifies tracts with records. It does not by itself establish a complete list of tracts with no crime records.
6. **Standardize and reduce dimensions.** Apply standardization and PCA, inspect explained variance and loadings, and retain enough components for the analysis. A two-dimensional visualization will not require fitting K-means on only two components.
7. **Select and assess clusters.** Compare candidate values of k using the SSE elbow, silhouette scores, cluster sizes, original-variable distributions, and repeated initializations. We will not choose k solely to obtain a preferred interpretation.
8. **Fit the explanatory classifier.** Use original crime percentages as predictors and generated cluster labels as the target. When cluster sizes permit, use a stratified train-test split. Compare accuracy, macro-F1, and the confusion matrix with a majority-class baseline. Perform any tuning using training data. Very small clusters will require reconsidering the design before forcing a split.
9. **Interpret the model.** Check the SHAP output dimensions and class order, then examine global and local explanations alongside observed cluster characteristics. Correlated percentages can affect how contributions are shared between predictors, so small importance-ranking differences will not be overinterpreted.

Our primary classification evaluation will describe reproduction of the fixed full-sample clustering. We will state that the labels were constructed using the full sample. If we also evaluate generalization to held-out observations, we will first split the data, fit standardization, PCA, and K-means on the training set, and assign test observations using those fixed transformations and centroids. These are distinct evaluation settings.

## 5. Additional Dataset: Property Assessment Records

### 5.1 Source and Purpose

| Item | Description |
|---|---|
| Dataset | Integrated Tax System Public Extract Facts |
| Provider | D.C. Office of Tax and Revenue / Office of the Chief Financial Officer |
| Source | [Official dataset page](https://opendata.dc.gov/datasets/DCGIS::integrated-tax-system-public-extract-facts-1/explore) |
| Available file | `Integrated_Tax_System_Public_Extract_Facts.csv` |
| Source observation | Property assessment record, identified by `SSL` |
| Selected fields | `SSL`, `PROPERTY_ADDRESS`, `LAND_USE_DESCRIPTION`, `APPRAISED_VALUE_CURRENT_TOTAL` |
| Purpose | Describe residential property types and assessed values across the crime-based clusters |

The file includes residential and nonresidential records. We will use the actual land-use descriptions to define residential categories before aggregation. A condominium unit, an individual house, and an entire apartment building are not equivalent units, so their total assessed values should not be pooled without regard to property type. [R4]

### 5.2 Assessment Period and Interpretation

An assessed value is not a sale price. A frequently updated extract also does not imply that every assessment represents the market value on the download date. We will confirm the applicable assessment period before interpreting the selected field.

OTR states that tax year 2026 assessments reflect property values as of January 1, 2025. This provides relevant timing context, but the CSV column names alone do not establish that every downloaded record belongs to that assessment cycle. We will not automatically interpret `PRIOR_TOTAL` as a particular calendar year's value. [R5]

Even where that assessment cycle applies, a January 1 assessment and an entire year of reported crime cover different reference periods. Property-use descriptions and addresses may reflect the extract's update date rather than historical conditions during all of 2025. Any comparison will acknowledge this timing limitation.

### 5.3 Linking Properties to Census Tracts

The available property CSV contains addresses but no tract or coordinate columns. Address geocoding is therefore a possible route to geographic assignment. The absence of coordinate fields does not make linkage impossible, but the two CSVs cannot be directly merged using a shared property-level key.

| Field or resource | Use in the linkage | Important distinction |
|---|---|---|
| Tax `PROPERTY_ADDRESS` | Geocode the property's location | Some addresses lack a street number or may produce ambiguous matches |
| Tax `SSL` | Attach location results to the original property record | Not a shared key with the crime dataset |
| Crime `CENSUS_TRACT` | Common unit for the final tract table | Must match the property tract definition and identifier format |
| Geocoding result and, if needed, official tract boundaries | Assign a compatible tract | Match quality and boundary vintage must be checked |
| Crime `BLOCK` | Understand the generalized crime location | Not an exact address key for matching a crime to a property |
| Tax OTR neighborhood fields | Geographic context and checks | Not interchangeable with census tracts |

We will prepare the linkage as follows:

1. Clean `PROPERTY_ADDRESS` while preserving street numbers, street names, and NW/NE/SW/SE designations. Owner mailing addresses will not substitute for the property location.
2. Create a unique-address list to avoid repeated geocoding requests, while retaining all valid SSL records. Multiple property records sharing an address will not be deleted solely for that reason.
3. Check an appropriate geocoding service's usage conditions and batch-processing options, then record its name and processing date.
4. Inspect match status, location precision, and unmatched addresses. A street-level or ambiguous result will not be treated as an exact building location.
5. Assign compatible census tracts using the service's tract information or coordinates and official boundaries. Record the boundary vintage and code format.
6. Aggregate valid residential records by tract and property type, then join those summaries to the crime-cluster table.
7. Report linkage success at both the unique-address and record levels, the number of linked records per tract, and whether missing matches are concentrated in particular property types or value ranges.

Geocoding and tract assignment remain planned work. If their accuracy or coverage is insufficient, we will narrow or defer the tract-level property comparison and explain the limitation. Geocoding services and geographic boundaries will support location assignment rather than add socioeconomic predictors to the model.

### 5.4 Planned Property Summaries

| Summary | Purpose | Preparation and interpretation |
|---|---|---|
| Median assessed value by tract and residential type | Compare assessed-value distributions across crime clusters | Separate individual units from whole buildings; check assessment period and zero values |
| Residential property-type record percentages | Compare the composition of linked residential records | Define categories from the observed land-use descriptions; record shares do not automatically equal shares of dwellings or households |
| Number of linked residential records per tract | Show the amount of information behind each summary | Do not label it the complete number of homes |
| Address and record linkage success rates | Assess coverage and potential bias | Report denominators and unresolved matches clearly |

These summaries will provide external context for the crime clusters. A difference in assessed values between clusters will be described as an observed association, not as evidence that crime changes property values.

## 6. Preliminary Feasibility Checks

The design-stage checks support the feasibility of creating tract-level crime inputs and inspecting the property data. They do not constitute final model results.

### 6.1 Property File Checks

| Check on the available CSV | Result |
|---|---:|
| Records | 215,770 |
| Columns | 37 |
| Blank SSL values | 0 |
| Duplicate SSL records beyond the first occurrence | 0 |
| Blank current total assessed values | 0 |
| Current total assessed values equal to zero | 40 |
| Negative current total assessed values | 0 |
| Blank property addresses | 26 |
| Blank land-use descriptions | 0 |
| Distinct untrimmed land-use descriptions | 86 |

The blank-value checks used empty or whitespace-only CSV values. They do not establish that every nonblank address is valid. Trimming category text may change the distinct-category count. The available CSV has no coordinate or census-tract fields.

### 6.2 Crime Service and Trial Aggregation

A preliminary official-service query on November 8, 2026 returned 24,156 records in nine offense categories. Six records lacked tract codes; the remaining 24,150 records were assigned to 206 distinct tract codes. These service counts will be reconciled with the fixed analysis CSV before final modeling. [R3]

| Offense category | Preliminary service record count |
|---|---:|
| THEFT/OTHER | 11,643 |
| THEFT F/AUTO | 5,363 |
| MOTOR VEHICLE THEFT | 3,935 |
| ROBBERY | 1,323 |
| ASSAULT W/DANGEROUS WEAPON | 933 |
| BURGLARY | 723 |
| HOMICIDE | 127 |
| SEX ABUSE | 105 |
| ARSON | 4 |
| **Total** | **24,156** |

A trial calculation produced nine tract-level offense percentages. Applying the preliminary 20-record threshold retained 199 tracts and excluded seven. Standardization and PCA ran successfully; retaining at least 90% cumulative explained variance produced seven components explaining approximately 95.97% in that trial.

| Trial k | Silhouette score | Smallest cluster, tracts |
|---:|---:|---:|
| 2 | 0.286 | 49 |
| 3 | 0.193 | 40 |
| 4 | 0.214 | 3 |
| 5 | 0.203 | 3 |
| 6 | 0.223 | 3 |

These trial K-means results used `random_state=42` and `n_init=10`. They demonstrate that the aggregation and algorithms can run. They do not establish the final value of k or strong substantive separation. The small clusters and very sparse arson category require further investigation, including repeated initializations and sensitivity to variable choices and the minimum-record threshold.

Final Random Forest and SHAP outputs have not yet been validated, and the property-to-tract linkage has not been completed. Later sessions will report those results after the model design and linkage quality have been assessed.

## References

- **[R1]** Alvarez-Garcia, M., Ibar-Alonso, R., & Arenas-Parra, M. (2024). A comprehensive framework for explainable cluster analysis. *Information Sciences, 663*, 120282. [DOI](https://doi.org/10.1016/j.ins.2024.120282) · [Open-access text](https://digibuo.uniovi.es/dspace/bitstream/handle/10651/73119/1-s2.0-S0020025524001956-main.pdf?isAllowed=y&sequence=1)
- **[R2]** Alvarez-Garcia, M., Arenas-Parra, M., & Ibar-Alonso, R. (2024). Uncovering student profiles. An explainable cluster analysis approach to PISA 2022. *Computers & Education, 223*, 105166. [DOI](https://doi.org/10.1016/j.compedu.2024.105166) · [Open-access text](https://digibuo.uniovi.es/dspace/bitstream/handle/10651/76089/1-s2.0-S0360131524001805-main.pdf?isAllowed=y&sequence=1)
- **[R3]** Open Data DC / Metropolitan Police Department. *Crime Incidents in 2025*. [Dataset](https://opendata.dc.gov/datasets/DCGIS::crime-incidents-in-2025) · [Service fields and query endpoint](https://maps2.dcgis.dc.gov/dcgis/rest/services/FEEDS/MPD/MapServer/7) · [DC Crime Cards](https://octo.dc.gov/service/dc-crime-cards) · [How MPD Reports Crime to the Public](https://mpdc.dc.gov/node/1823646)
- **[R4]** Open Data DC / Office of Tax and Revenue. *Integrated Tax System Public Extract Facts*. [Dataset](https://opendata.dc.gov/datasets/DCGIS::integrated-tax-system-public-extract-facts-1/explore) · [Service field definitions](https://services.arcgis.com/neT9SoYxizqTHZPH/arcgis/rest/services/ITSPE_FACTS_05022025/FeatureServer/0)
- **[R5]** Office of Tax and Revenue. *Office of Tax and Revenue Reports Significant Decline in District of Columbia Commercial Real Estate Values*. [Tax year 2026 assessment announcement](https://otr.cfo.dc.gov/release/office-tax-and-revenue-reports-significant-decline-district-columbia-commercial-real-estate)

Source-check date recorded during research design: November 8, 2026. The property file checks and the live crime-service query refer to different data snapshots; the fixed analysis files will be documented separately.

---

[← Project home](index.md)
