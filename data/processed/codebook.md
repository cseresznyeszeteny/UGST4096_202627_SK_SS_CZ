# Codebook: processed data

This folder holds the two clean files of the project *Who Is Leaving the Periphery? Population change and age structure between the last two censuses in Hungary, Georgia and Nepal*. Both are produced by the notebook `prep/prep_data.ipynb` from the raw downloads in `data/raw/`. Do not edit them by hand; re-run the notebook instead.

| File | What it is |
|---|---|
| `HU-GE-NE_census.csv` | Census population by region, 5-year age group and sex, at the last two censuses of each country, with derived features |
| `HU-GE-NE_map.geojson` | Simplified region borders of the three countries, to draw the census data on a map |
| `codebook.md` | This file |

---

## 1. `HU-GE-NE_census.csv`

**Format:** comma-separated, UTF-8 (Hungarian accents are kept), header in the first row, empty cell = missing value (NA), decimals with a dot, numbers at full precision.

**Size:** 3 424 rows × 29 columns.

**One row = one country × region × age group × sex.** Every region has exactly 32 rows (16 age groups × 2 sexes).

| Country | Regions | Unit | Rows | Previous census | Latest census | Years between ($t$) |
|---|---|---|---|---|---|---|
| Hungary | 20 | 19 counties (*vármegye*) + Budapest | 640 | 2011 | 2022 | 11 |
| Georgia | 11 + Abkhazia | regions (*mkhare*), incl. Tbilisi and Adjara AR | 384 | 2014 | 2024 | 10 |
| Nepal | 75 | districts (*jilla*) | 2 400 | 2011 | 2021 | 10 |

### Sources

| Country | Census | Source |
|---|---|---|
| Hungary | 2011, 2022 | KSH (Hungarian Central Statistical Office), table WBS002 |
| Georgia | 2014, 2024 | Geostat (National Statistics Office of Georgia), population by region, age and sex |
| Nepal | 2011 | NSO (National Statistics Office, formerly CBS), district profiles, Table 2.2 |
| Nepal | 2021 | NSO, *National Population and Housing Census 2021, National Report*, Vol. 01, Table 18 |

Population = usually resident population, as published by each statistics office.

### Columns

#### Identifiers and labels

| Column | Type | Description | Values / example |
|---|---|---|---|
| `Country` | text | Country | `Hungary`, `Georgia`, `Nepal` |
| `Region` | text | Region name in the statistics office's spelling | `Győr-Moson-Sopron`, `Adjara AR`, `Kathmandu` |
| `Map_key` | text | The region's name exactly as in the map file; join key to `HU-GE-NE_map.geojson` together with `Country` | `Gyor Moson Sopron`, `Adjara`, `Kathmandu` |
| `Age_group` | text | 5-year age group; the oldest group is open | `0-4`, `5-9`, …, `70-74`, `75+` |
| `Sex` | text | Sex | `Male`, `Female` |
| `Age_order` | integer | Sort number of the age group (sort by this, not by `Age_group`) | `1` (0-4) … `16` (75+) |
| `Age_band` | text | Broad age band | `0-14`, `15-64`, `65+` |
| `Capital_city` | text | The region is the capital city | `Yes` for Budapest, Tbilisi, Kathmandu; else `No` |
| `Capital_area` | text | The region belongs to the wider capital area | `Yes` for Budapest, Pest, Tbilisi, Kathmandu, Lalitpur, Bhaktapur; else `No` |
| `Province` | text | Nepal only: province of the district; empty for Hungary and Georgia | `Koshi`, `Madhesh`, `Bagmati`, `Gandaki`, `Lumbini`, `Karnali`, `Sudur Paschim` |

#### Population (persons)

| Column | Type | Description |
|---|---|---|
| `2011` | integer | Population at the 2011 census (Hungary, Nepal) |
| `2014` | integer | Population at the 2014 census (Georgia) |
| `2021` | integer | Population at the 2021 census (Nepal) |
| `2022` | integer | Population at the 2022 census (Hungary) |
| `2024` | integer | Population at the 2024 census (Georgia) |
| `Year_prev`, `Year_latest` | integer | The country's previous and latest census year |
| `t_years` | integer | Years between the two censuses (`Year_latest` − `Year_prev`) |
| `Previous` | integer | The row's population at the previous census (the same as the matching year column) |
| `Latest` | integer | The row's population at the latest census (the same as the matching year column) |

Each country fills only its own two year columns; the other three are empty. `Previous` and `Latest` hold the same values in the same columns for all countries, so they are the easiest to use.

#### Features

Every feature describes **one row**. $P^{prev}$ and $P^{latest}$ = the row's `Previous` and `Latest`; $P_R$ = the total of the row's region; $P_C$ = the total of the row's country; $t$ = `t_years`.

| Column | Unit | Formula | Meaning | Can be summed? |
|---|---|---|---|---|
| `Abs_change` | persons | $P^{latest} - P^{prev}$ | change in the number of people | Yes: sums to the region or country change |
| `Annual_change_pct` | % per year | $\left(\left(P^{latest} / P^{prev}\right)^{1/t} - 1\right) \times 100$ | average yearly growth of this age-sex group | No: **never sum or average** |
| `Share_of_region_prev_pct` | % | $P^{prev} / P_R^{prev} \times 100$ | share of the region's population in this age-sex group (age structure of the region) | Yes: a region's 32 rows sum to 100 |
| `Share_of_region_latest_pct` | % | $P^{latest} / P_R^{latest} \times 100$ | the same at the latest census | Yes: sums to 100 per region |
| `Share_of_region_change_pp` | percentage points | latest share − previous share | change of the region share | Yes: sums to 0 per region |
| `Share_of_country_prev_pct` | % | $P^{prev} / P_C^{prev} \times 100$ | share of the whole country's population living in this region and in this age-sex group | Yes: a country's rows sum to 100 |
| `Share_of_country_latest_pct` | % | $P^{latest} / P_C^{latest} \times 100$ | the same at the latest census | Yes: sums to 100 per country |
| `Share_of_country_change_pp` | percentage points | latest share − previous share | change of the country share | Yes: sums to 0 per country |
| `Contribution_pp` | percentage points | $(P^{latest} - P^{prev}) / P_R^{prev} \times 100$ | how much this age-sex group added to (or took from) its region's total % change | Yes: a region's 32 rows sum to the region's total % change |

Examples of summing correctly: a region's share of its country = the sum of its 32 `Share_of_country_…` rows; a region's share aged 65+ = the sum of `Share_of_region_…` over the rows with `Age_band` = `65+`.

### Missing values

| Where | Why |
|---|---|
| The three year columns that do not belong to the row's country | that country had no census in those years |
| All population and feature columns of **Abkhazia** (Georgia) | not covered by either Georgian census; the 32 rows exist so the region can be shown as "no data" |
| `Province` for Hungary and Georgia | provinces exist only for Nepal |

### Things to know

- **Nepal's 75 districts:** two districts were split after 2011 (Nawalparasi into East and West, Rukum into East and West). Their 2021 halves were **added back together** so both censuses use the same 75 districts. Province of the merged districts = that of the larger half in 2021: Nawalparasi → Lumbini, Rukum → Karnali.
- **Top age group 75+:** Nepal's 2011 tables stop at 75+, so all countries were collapsed to 75+ (the other sources go higher).
- **Different census years:** the countries' censuses are 10–11 years apart and not in the same years; `Annual_change_pct` makes the growth comparable per year.
- **Tskhinvali (South Ossetia)** was not covered by the Georgian censuses either; it has no region of its own on the map and no rows.
- **Totals check:** for each country and census, the sum of all rows equals the official national total: Hungary 9 937 628 (2011) and 9 603 634 (2022); Georgia 3 713 804 (2014) and 3 929 581 (2024); Nepal 26 494 504 (2011) and 29 164 578 (2021).

---

## 2. `HU-GE-NE_map.geojson`

**Format:** GeoJSON, coordinates in longitude/latitude (WGS 84, EPSG:4326) with 5 decimals.

**Size:** 107 shapes, about 450 KB.

| Column | Description |
|---|---|
| `Country` | `Hungary`, `Georgia`, `Nepal` |
| `Map_key` | Region name; matches `Map_key` in the census file |
| `geometry` | Region border (polygon) |

| Country | Shapes | Source |
|---|---|---|
| Hungary | 20 | HDX, Hungary subnational administrative boundaries (COD-AB), level 1 |
| Georgia | 12 (incl. Abkhazia) | geoBoundaries, Georgia ADM1 |
| Nepal | 75 | HDX, Nepal subnational administrative boundaries (COD-AB, Survey Department of Nepal), level 2; the split halves of Nawalparasi and Rukum merged into one shape each |

**Simplified:** the borders were simplified to about 500 m precision, so the file is small enough for a dashboard. Neighbouring regions still share their borders exactly (no gaps or overlaps). Do not use the shapes for area measurements.

---

## 3. Using the two files together (e.g. in Tableau)

Connect to the map file as a spatial file and to the census file as a text file, and link them on **`Country` = `Country`** and **`Map_key` = `Map_key`**. Every one of the 107 regions appears in both files exactly once (the census file has 32 rows per region).

Credit the sources in any visualisation, e.g. *"Source: KSH census 2011, 2022; Geostat census 2014, 2024; NSO Nepal census 2011, 2021. Boundaries: HDX COD-AB, geoBoundaries."*
