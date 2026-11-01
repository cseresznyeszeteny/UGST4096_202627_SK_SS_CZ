# Who Is Leaving the Periphery?

**Population change and age structure between the last two censuses in Hungary, Georgia and Nepal**

This project compares three countries that are at different stages of the demographic transition. Hungary is shrinking and ageing, Georgia is shaped by emigration and the pull of Tbilisi, and Nepal is still young and growing while people move from the hills to the plains and the Kathmandu Valley. For each country it looks at where the population grew or shrank between the last two censuses, how much of the growth went to the capital region, and how the age structure differs.

The repository contains the raw census and boundary downloads, one Python notebook that turns them into two clean files, and the documentation of those files. The clean files are made for visualisation in Tableau.

## Countries and regions

The regional units were chosen so that they have roughly the same *population size*, not the same administrative level. This keeps the maps comparable across countries.

| Country | Censuses | Years between | Unit | Units |
|---|---|---|---|---|
| Hungary | 2011, 2022 | 11 | 19 counties (*vármegye*) + Budapest | 20 |
| Georgia | 2014, 2024 | 10 | regions (*mkhare*), incl. Tbilisi and Adjara AR | 11 (+ Abkhazia, no data) |
| Nepal | 2011, 2021 | 10 | districts (*jilla*) | 75 |

## Repository contents

```
├── README.md                this file
├── appendix.md              detailed reference: file inventory, checks, age-group and name mappings, unit lists
├── prep/
│   └── prep_data.ipynb      data preparation notebook (raw -> processed)
└── data/
    ├── raw/                 original downloads, unchanged
    │   ├── census/          KSH (Hungary), Geostat (Georgia), NSO (Nepal: 75 district workbooks for 2011, PDF report for 2021)
    │   └── geo/             boundary files: HDX COD-AB (Hungary, Nepal), geoBoundaries (Georgia)
    └── processed/           written by the notebook
        ├── HU-GE-NE_census.csv
        ├── HU-GE-NE_map.geojson
        └── codebook.md      column-by-column description of the two files
```

## Data sources

| Country | Census data | Boundaries |
|---|---|---|
| Hungary | KSH (Hungarian Central Statistical Office), table WBS002 *A népesség neme, életkora vármegyénként* | HDX, Hungary COD-AB, level 1 |
| Georgia | Geostat (National Statistics Office of Georgia), population by region, age and sex, 2014 and 2024 | geoBoundaries, Georgia ADM1 |
| Nepal | NSO: 2011 district profiles (Table 2.2); *National Population and Housing Census 2021, National Report*, Vol. 01, Table 18 | HDX, Nepal COD-AB (Survey Department of Nepal), level 2 |

Population is the usually resident population as published by each statistics office. Growth rates and shares are calculated from the census counts; no published rates are used. The Nepal files are offline copies, because the NSO servers were down at download time.

## What the notebook does

`prep/prep_data.ipynb` reads only `data/raw/` and works in 29 explained steps:

1. **Census data.** Nepal 2021 is extracted directly from the PDF report and checked against its printed percentages. Nepal 2011 is read from the 75 district workbooks. Georgia and Hungary are read from the statistics-office exports. All sources are brought to the same format: 16 five-year age groups (the oldest is 75+, because Nepal 2011 stops there) by sex, per region and census.
2. **Harmonising units.** Two Nepali districts (Nawalparasi and Rukum) were split in two after 2011; their 2021 halves are added back together so both censuses use the same 75 districts. Region names are cleaned, and each region gets a `Map_key` that matches the name in the boundary file.
3. **Labels and features.** Capital flags (the city alone and the wider capital area), Nepali province, broad age band, and per-row features: absolute change, annualised change, shares of region and country, and contribution to the region's change.
4. **Checks.** Every country and census total is checked against the official national total, and the notebook stops if any check fails.
5. **Maps.** The three boundary layers are read, one invalid shape (Pest) is repaired, the split Nepali districts are merged, and the borders are simplified to about 500 m without gaps between neighbours.
6. **Final check.** The census table and the map contain exactly the same 107 regions.

## Outputs

| File | Content |
|---|---|
| `data/processed/HU-GE-NE_census.csv` | 3 424 rows × 29 columns. One row = country × region × 5-year age group × sex, with population at both censuses, labels and row-level features |
| `data/processed/HU-GE-NE_map.geojson` | 107 simplified region shapes (Hungary 20, Georgia 12 incl. Abkhazia, Nepal 75) with `Country` and `Map_key` |

The two files are joined on `Country` + `Map_key`. Every column, formula and missing value is described in `data/processed/codebook.md`.

## Reproducing

1. Create a Python 3.12 environment with a Jupyter kernel, e.g.
   ```
   conda create -n UGST4096 python=3.12 ipykernel -y
   conda activate UGST4096
   python -m ipykernel install --user --name UGST4096 --display-name "Python (UGST4096)"
   ```
2. Open `prep/prep_data.ipynb` with that kernel and run all cells. The first cell installs the libraries it needs (`pandas`, `openpyxl`, `pdfplumber`, `geopandas`, `shapely`, `pyogrio`, `matplotlib`).

## Things to keep in mind

- **Different census years** (2011/2022, 2014/2024, 2011/2021): the annualised growth rate makes the countries comparable per year.
- **Georgia:** Abkhazia and Tskhinvali (South Ossetia) were not covered by either census. Abkhazia is in the data as an empty region so it can be shown as "no data"; Tskhinvali has no shape of its own. Foreign citizens (6.6% of the 2024 population) are included in the 2024 figures.
- **Nepal:** the 2021 census is estimated to have missed about 772 000 people, mostly in cities, and labour migrants living abroad are not part of the resident population.
- **Top age group 75+** for all countries, so detail above 75 is lost.
- **Maps** are simplified for display and should not be used to measure areas.
