# Appendix: Data sources, unit lists and name mappings

*Companion to `README.md`. Last updated: 25 Oct 2026.*

This appendix collects every fact the data builder needs to build the master file (`data/processed/HU-GE-NE_census.csv`): which file holds what, how each unit is named in each source, how units are merged, and which checks have already passed. All population figures below were read directly from the raw files.

---

## A1. File inventory

### Census data (`data/raw/census/`)

| File | Country, census | Content | Status |
|---|---|---|---|
| `hungary_2001-2011-2022_pop_age_sex_varmegye.xlsx` | Hungary 2001, 2011, 2022 | KSH table WBS002 *A népesség neme, életkora vármegyénként*, Excel export (sheet *Adattábla*; sheet *Adatlap* holds the source link). County names; single years, 5-year groups, `85–`, by sex | **Source file for Hungary.** Downloaded 25 Oct 2026 |
| `georgia_2014_pop_age_sex_mkhare.csv` | Georgia 2014 | Geostat: population by region, 5-year age group (to 85+), sex, urban/rural | Source file |
| `georgia_2024_pop_age_sex_mkhare.csv` | Georgia 2024 | Geostat: population by region **and municipality**, 5-year age group (to 85+), sex, urban/rural (as of 14 Nov 2024) | Source file |
| `district-profile-census-2011/` | Nepal 2011 | NSO district profiles, one Excel file per district (75), unpacked from the downloaded zip. Table 2.2 on sheet `Ind02L`, district totals in rows 8–10 (both sexes, male, female), 5-year groups to **75+** | Source file (offline copy; NSO servers down) |
| `Population Census_National Report_English.pdf` | Nepal 2021 | NPHC 2021 National Report, Vol. 01 (reprint May 2023). Table 18, report pp. 84–135 = PDF pp. 99–150; the notebook extracts it directly (Steps 5–7) | Source file for 2021 |

Files removed because the notebook does not use them: the KSH CSV export (`…varmegye.csv`, same figures as the Excel file with NUTS codes), the Georgian national single-year table, the original zip of the Nepal 2011 profiles (its unpacked folder is kept) and a first extraction of Table 18 to CSV (the notebook now extracts the table itself).

### Boundary files (`data/raw/geo/`)

| File | Country | Layer to use | Units | Status |
|---|---|---|---|---|
| `hun_admin_boundaries.geojson.zip` | Hungary | `hun_admin1.geojson` (plain layer, A10) | 20 (19 counties + Budapest) | OK. HDX COD-AB |
| `geoBoundaries-GEO-ADM1-all.zip` | Georgia | `geoBoundaries-GEO-ADM1.geojson` (or `_simplified`) | 12 (11 + Abkhazia); no separate Tskhinvali unit | OK. geoBoundaries |
| `npl_admin_boundaries.geojson.zip` | Nepal | `npl_admin2.geojson` (plain layer, 16 MB, A10) | 77 districts, all `Polygon` | OK. HDX COD-AB, source: Survey Department of Nepal, `valid_on` 2024-03-14, version v02. https://data.humdata.org/dataset/cod-ab-npl |

Notes:
- The earlier geoBoundaries Nepal file (`geoBoundaries-NPL-ADM2-all.zip`) was **removed**. It was traced from Wikimedia Commons (boundaries "representative of 2006"), had 75 polygons with two "Bara" and two "Saptari", and was missing Parsa, Siraha and Rupandehi.
- The HDX `_em` layers (edge-matched to neighbouring countries' borders) are not used.
- **Pest is invalid in the HDX Hungary file:** its outer border touches itself in one point (18.9353 E, 47.5226 N). The notebook repairs it with `make_valid` (area unchanged) before simplifying.

### Outputs (`data/processed/`), written by `prep/prep_data.ipynb`

| File | Content |
|---|---|
| `HU-GE-NE_census.csv` | 3 424 rows (Hungary 640, Georgia 384 incl. Abkhazia's 32 empty rows, Nepal 2 400) × 29 columns: identifiers, labels and population columns, then the features (see `data/processed/codebook.md`) |
| `HU-GE-NE_map.geojson` | 107 shapes (Hungary 20, Georgia 12, Nepal 75 after merging), columns `Country`, `Map_key`, `geometry`; borders simplified with `simplify_coverage` at 0.005° (about 500 m), keeping shared borders gap-free; coordinates at 5 decimals; 453 KB |

---

## A2. Census years and $t$

| Country | Previous census | Latest census | $t$ (years) | Master-file columns |
|---|---|---|---|---|
| Hungary | 2011 | 2022 | 11 | `2011`, `2022` |
| Georgia | 2014 | 2024 | 10 | `2014`, `2024` |
| Nepal | 2011 | 2021 | 10 | `2011`, `2021` |

$t$ = latest census year − previous census year.

---

## A3. Validation already done

| Source | Checks | Result |
|---|---|---|
| Hungary (KSH WBS002) | The 20 counties add up to the national total | Passed: 9 937 628 (2011), 9 603 634 (2022) |
| | Missing cells | Two blank cells in 2022: males aged 99 in Vas and Tolna (suppressed for privacy). Not needed, because `85–` is published as a whole |
| Georgia 2014 | The 11 regions add up to the national total | Passed: 3 713 804 (1 772 864 male, 1 940 940 female) |
| Georgia 2024 | The 11 regions add up to the national total | Passed: 3 929 581 (1 881 004 male, 2 048 577 female) |
| Nepal 2011 (75 workbooks) | Male + female = total, and the age groups add up to the total, in every district | Passed: all 75 |
| | The 75 districts add up to the official national total | Passed: 26 494 504 (12 849 041 male, 13 645 463 female) |
| Nepal 2021 (Table 18) | Male + female = total in every row; the age groups add up to "All Ages" in all 90 areas | Passed (repeated in the notebook, Step 7) |
| | First extraction (xpdf) and a second, independent one compared row by row | Passed: all 1 890 rows identical; the notebook's pdfplumber extraction gives the same totals |
| | All printed percentages recalculated from the counts | Passed: all match to the PDF's 2 decimals (5 670 percentages in the notebook, Step 7) |
| | Visual spot checks against page images (Kathmandu, Bhaktapur, Humla, Jumla, Baglung, Rolpa, Syangja, Kanchanpur) | Passed |
| | The 77 districts add up to the official national total | Passed: 29 164 578 (14 253 551 male, 14 911 027 female) |
| Nepal map (HDX) | All 77 map names match the 2021 data names (after "(East)" → "East") | Passed |

---

## A4. Age groups: from each source to the 16 master-file groups

Master-file groups (the same for all countries, with 75+ as the top group because Nepal 2011 stops there): `0-4`, `5-9`, `10-14`, `15-19`, `20-24`, `25-29`, `30-34`, `35-39`, `40-44`, `45-49`, `50-54`, `55-59`, `60-64`, `65-69`, `70-74`, `75+` (with `Age_order` 1–16).

| Source | Labels in the source | To build `75+` | Rows to drop |
|---|---|---|---|
| Hungary (Excel) | `0 éves`…, `5–9 éves`, …, `70–74 éves`, `75–79 éves`, `80–84 éves`, `85– éves`; the first group is `5 évesnél fiatalabb` (= 0–4) | `75–79` + `80–84` + `85–` | `Összesen` (total), single years, wider bands (`15–29`, `65–84`, …), `15 évesnél fiatalabb` |
| Georgia 2014, 2024 | `0-4` … `80-84`, `85+` | `75-79` + `80-84` + `85+` | Region total rows |
| Nepal 2011 (`Ind02L`) | `00 - 04` … `70 - 74`, `75 +` | already `75 +` | Column D (total) |
| Nepal 2021 (PDF Table 18) | `00-04 Yrs.` … `90-94 Yrs.`, `95+ Yrs.`, `All Ages` | `75-79` + `80-84` + `85-89` + `90-94` + `95+` | `All Ages` |

Sex labels: Hungary `Férfi` / `Nő` (`Összesen` = both, drop); Georgia "Males"/"Females" (2014 file: "Male"/"Female"); Nepal `Male` / `Female`. All become `Male` / `Female`.

---

## A5. Nepal: merged districts

Two 2011 districts were split before 2021. So that both censuses use the same 75 districts, the 2021 halves are added back together, both in the data and on the map.

| Master-file unit (`Region`) | 2011 | 2021 parts | 2021 total | Map polygons to dissolve | Province of the parts | `Province` in the master file |
|---|---|---|---|---|---|---|
| **Nawalparasi** | 643 508 (one district) | Nawalparasi (East) 378 079 + Nawalparasi (West) 386 868 | 764 947 | `Nawalparasi East` (NP0447) + `Nawalparasi West` (NP0547) | East: Gandaki, West: Lumbini | **Lumbini** (the larger half in 2021; confirmed) |
| **Rukum** | 208 567 (one district) | Rukum (East) 56 786 + Rukum (West) 166 740 | 223 526 | `Rukum East` (NP0552) + `Rukum West` (NP0652) | East: Lumbini, West: Karnali | **Karnali** (the larger half in 2021; confirmed) |

- Add the halves **row by row** (each age group × sex), not only the totals.
- The merged 2021 unit should cover the same area as the 2011 district. Check this on the map after dissolving: the outline should show no gaps or extra pieces against the neighbouring districts.
- The P-codes of each pair share their last two digits (…47, …52), the old district number.
- Mention the province assignment in the method notes.

---

## A6. Region names: Nepal

**`Region` uses the 2021 NSO spelling (decided).** The district names *inside* the 2011 workbooks (cell `T1` of sheet `Ind02L`) are **already identical** to the 2021 names for all 75 districts. Only the 2011 *file names* are misspelled, so never take names from the file names:

| 2011 file name says | Correct name (inside the file, 2021, map) |
|---|---|
| `Illam` | Ilam |
| `Tehrathum` | Terhathum |
| `Dhanusha` | Dhanusa |
| `Sindhupalchowk` | Sindhupalchok |
| `Kavre` | Kavrepalanchok |
| `Makawanpur` | Makwanpur |
| `Chitwan` | Chitawan |
| `Tahanun` (and "Distirct") | Tanahu |

The HDX map uses the same names, except that the split halves have no brackets ("Nawalparasi East" instead of "Nawalparasi (East)").

### Full list (75 master-file units)

Populations are totals (both sexes). Provinces are taken from the HDX map (`adm1_name`); NSO and the map both spell the last one **"Sudur Paschim"**.

| `Region` | 2011 file no. | 2011 population | 2021 name(s) | 2021 population | HDX map name(s) | HDX P-code(s) | `Province` |
|---|---|---|---|---|---|---|---|
| Taplejung | 01 | 127461 | Taplejung | 120590 | Taplejung | NP0101 | Koshi |
| Panchthar | 02 | 191817 | Panchthar | 172400 | Panchthar | NP0102 | Koshi |
| Ilam | 03 | 290254 | Ilam | 279534 | Ilam | NP0103 | Koshi |
| Jhapa | 04 | 812650 | Jhapa | 998054 | Jhapa | NP0104 | Koshi |
| Morang | 05 | 965370 | Morang | 1148156 | Morang | NP0105 | Koshi |
| Sunsari | 06 | 763487 | Sunsari | 926962 | Sunsari | NP0106 | Koshi |
| Dhankuta | 07 | 163412 | Dhankuta | 150599 | Dhankuta | NP0107 | Koshi |
| Terhathum | 08 | 101577 | Terhathum | 88731 | Terhathum | NP0108 | Koshi |
| Sankhuwasabha | 09 | 158742 | Sankhuwasabha | 158041 | Sankhuwasabha | NP0109 | Koshi |
| Bhojpur | 10 | 182459 | Bhojpur | 157923 | Bhojpur | NP0110 | Koshi |
| Solukhumbu | 11 | 105886 | Solukhumbu | 104851 | Solukhumbu | NP0111 | Koshi |
| Okhaldhunga | 12 | 147984 | Okhaldhunga | 139552 | Okhaldhunga | NP0113 | Koshi |
| Khotang | 13 | 206312 | Khotang | 175298 | Khotang | NP0112 | Koshi |
| Udayapur | 14 | 317532 | Udayapur | 340721 | Udayapur | NP0114 | Koshi |
| Saptari | 15 | 639284 | Saptari | 706255 | Saptari | NP0215 | Madhesh |
| Siraha | 16 | 637328 | Siraha | 739953 | Siraha | NP0216 | Madhesh |
| Dhanusa | 17 | 754777 | Dhanusa | 867747 | Dhanusa | NP0217 | Madhesh |
| Mahottari | 18 | 627580 | Mahottari | 706994 | Mahottari | NP0218 | Madhesh |
| Sarlahi | 19 | 769729 | Sarlahi | 862470 | Sarlahi | NP0219 | Madhesh |
| Sindhuli | 20 | 296192 | Sindhuli | 300026 | Sindhuli | NP0320 | Bagmati |
| Ramechhap | 21 | 202646 | Ramechhap | 170302 | Ramechhap | NP0321 | Bagmati |
| Dolakha | 22 | 186557 | Dolakha | 172767 | Dolakha | NP0322 | Bagmati |
| Sindhupalchok | 23 | 287798 | Sindhupalchok | 262624 | Sindhupalchok | NP0323 | Bagmati |
| Kavrepalanchok | 24 | 381937 | Kavrepalanchok | 364039 | Kavrepalanchok | NP0324 | Bagmati |
| Lalitpur | 25 | 468132 | Lalitpur | 551667 | Lalitpur | NP0325 | Bagmati |
| Bhaktapur | 26 | 304651 | Bhaktapur | 432132 | Bhaktapur | NP0326 | Bagmati |
| Kathmandu | 27 | 1744240 | Kathmandu | 2041587 | Kathmandu | NP0327 | Bagmati |
| Nuwakot | 28 | 277471 | Nuwakot | 263391 | Nuwakot | NP0328 | Bagmati |
| Rasuwa | 29 | 43300 | Rasuwa | 46689 | Rasuwa | NP0329 | Bagmati |
| Dhading | 30 | 336067 | Dhading | 325710 | Dhading | NP0330 | Bagmati |
| Makwanpur | 31 | 420477 | Makwanpur | 466073 | Makwanpur | NP0331 | Bagmati |
| Rautahat | 32 | 686722 | Rautahat | 813573 | Rautahat | NP0232 | Madhesh |
| Bara | 33 | 687708 | Bara | 763137 | Bara | NP0233 | Madhesh |
| Parsa | 34 | 601017 | Parsa | 654471 | Parsa | NP0234 | Madhesh |
| Chitawan | 35 | 579984 | Chitawan | 719859 | Chitawan | NP0335 | Bagmati |
| Gorkha | 36 | 271061 | Gorkha | 251027 | Gorkha | NP0436 | Gandaki |
| Lamjung | 37 | 167724 | Lamjung | 155852 | Lamjung | NP0437 | Gandaki |
| Tanahu | 38 | 323288 | Tanahu | 321153 | Tanahu | NP0440 | Gandaki |
| Syangja | 39 | 289148 | Syangja | 253024 | Syangja | NP0441 | Gandaki |
| Kaski | 40 | 492098 | Kaski | 600051 | Kaski | NP0439 | Gandaki |
| Manang | 41 | 6538 | Manang | 5658 | Manang | NP0438 | Gandaki |
| Mustang | 42 | 13452 | Mustang | 14452 | Mustang | NP0445 | Gandaki |
| Myagdi | 43 | 113641 | Myagdi | 107033 | Myagdi | NP0444 | Gandaki |
| Parbat | 44 | 146590 | Parbat | 130887 | Parbat | NP0442 | Gandaki |
| Baglung | 45 | 268613 | Baglung | 249211 | Baglung | NP0443 | Gandaki |
| Gulmi | 46 | 280160 | Gulmi | 246494 | Gulmi | NP0551 | Lumbini |
| Palpa | 47 | 261180 | Palpa | 245027 | Palpa | NP0546 | Lumbini |
| Nawalparasi | 48 | 643508 | Nawalparasi (East) + Nawalparasi (West) | 764947 (378079 + 386868) | Nawalparasi East + Nawalparasi West | NP0447 + NP0547 | **Lumbini** (East: Gandaki, West: Lumbini) |
| Rupandehi | 49 | 880196 | Rupandehi | 1121957 | Rupandehi | NP0548 | Lumbini |
| Kapilbastu | 50 | 571936 | Kapilbastu | 682961 | Kapilbastu | NP0549 | Lumbini |
| Arghakhanchi | 51 | 197632 | Arghakhanchi | 177086 | Arghakhanchi | NP0550 | Lumbini |
| Pyuthan | 52 | 228102 | Pyuthan | 232019 | Pyuthan | NP0555 | Lumbini |
| Rolpa | 53 | 224506 | Rolpa | 234793 | Rolpa | NP0554 | Lumbini |
| Rukum | 54 | 208567 | Rukum (East) + Rukum (West) | 223526 (56786 + 166740) | Rukum East + Rukum West | NP0552 + NP0652 | **Karnali** (East: Lumbini, West: Karnali) |
| Salyan | 55 | 242444 | Salyan | 238515 | Salyan | NP0653 | Karnali |
| Dang | 56 | 552583 | Dang | 674993 | Dang | NP0556 | Lumbini |
| Banke | 57 | 491313 | Banke | 603194 | Banke | NP0557 | Lumbini |
| Bardiya | 58 | 426576 | Bardiya | 459900 | Bardiya | NP0558 | Lumbini |
| Surkhet | 59 | 350804 | Surkhet | 415126 | Surkhet | NP0659 | Karnali |
| Dailekh | 60 | 261770 | Dailekh | 252313 | Dailekh | NP0661 | Karnali |
| Jajarkot | 61 | 171304 | Jajarkot | 189360 | Jajarkot | NP0660 | Karnali |
| Dolpa | 62 | 36700 | Dolpa | 42774 | Dolpa | NP0662 | Karnali |
| Jumla | 63 | 108921 | Jumla | 118349 | Jumla | NP0663 | Karnali |
| Kalikot | 64 | 136948 | Kalikot | 145292 | Kalikot | NP0664 | Karnali |
| Mugu | 65 | 55286 | Mugu | 64549 | Mugu | NP0665 | Karnali |
| Humla | 66 | 50858 | Humla | 55394 | Humla | NP0666 | Karnali |
| Bajura | 67 | 134912 | Bajura | 138523 | Bajura | NP0768 | Sudur Paschim |
| Bajhang | 68 | 195159 | Bajhang | 189085 | Bajhang | NP0767 | Sudur Paschim |
| Achham | 69 | 257477 | Achham | 228852 | Achham | NP0769 | Sudur Paschim |
| Doti | 70 | 211746 | Doti | 204831 | Doti | NP0770 | Sudur Paschim |
| Kailali | 71 | 775709 | Kailali | 904666 | Kailali | NP0771 | Sudur Paschim |
| Kanchanpur | 72 | 451248 | Kanchanpur | 513757 | Kanchanpur | NP0772 | Sudur Paschim |
| Dadeldhura | 73 | 142094 | Dadeldhura | 139602 | Dadeldhura | NP0773 | Sudur Paschim |
| Baitadi | 74 | 250898 | Baitadi | 242157 | Baitadi | NP0774 | Sudur Paschim |
| Darchula | 75 | 133274 | Darchula | 133310 | Darchula | NP0775 | Sudur Paschim |
| **Total** | | **26494504** | | **29164578** | | | |

---

## A7. Region names: Georgia

The two Geostat files label two regions differently, and the map uses its own short names. Master-file names (**confirmed**):

| `Region` | 2014 file label | 2024 file label | Map name (geoBoundaries) | ISO 3166-2 | 2014 population | 2024 population |
|---|---|---|---|---|---|---|
| Tbilisi | `C. Tbilisi` | `C. Tbilisi` | Tbilisi | GE-TB | 1 108 717 | 1 331 485 |
| Adjara AR | `Autonomous Republic of Adjara` | `Adjara of Autonomous Republic` | Adjara | GE-AJ | 333 953 | 402 929 |
| Guria | Guria | Guria | Guria | GE-GU | 113 350 | 102 408 |
| Imereti | Imereti | Imereti | Imereti | GE-IM | 533 906 | 510 741 |
| Kakheti | Kakheti | Kakheti | Kakheti | GE-KA | 318 583 | 303 833 |
| Mtskheta-Mtianeti | Mtskheta-Mtianeti | Mtskheta-Mtianeti | Mtskheta-Mtianeti | GE-MM | 94 573 | 94 039 |
| Racha-Lechkhumi and Kvemo Svaneti | same | same | Racha-Lechkhumi and Kvemo Svaneti | GE-RL | 32 089 | 29 901 |
| Samegrelo-Zemo Svaneti | same | same | Samegrelo-Zemo Svaneti | GE-SZ | 330 761 | 305 597 |
| Samtskhe-Javakheti | `Samtskhe-Javakheti` (hyphen) | same | `Samtskhe–Javakheti` (**en dash**) | GE-SJ | 160 504 | 155 282 |
| Kvemo Kartli | Kvemo Kartli | Kvemo Kartli | Kvemo Kartli | GE-KK | 423 986 | 441 630 |
| Shida Kartli | Shida Kartli | Shida Kartli | Shida Kartli | GE-SK | 263 382 | 251 736 |
| *(no data)* | — | — | Abkhazia | GE-AB | — | — |
| **Total** | | | | | **3 713 804** | **3 929 581** |

- **Adjara's label flips** between the files ("Autonomous Republic of Adjara" in 2014, "Adjara of Autonomous Republic" in 2024), so matching on the raw label fails.
- **"C." means city.** In the 2024 file, other city rows (C. Batumi, C. Kutaisi, C. Rustavi …) and all "… Municipality" rows are units **below** a region and must be dropped; only the 11 region rows are kept.
- **The map spells Samtskhe–Javakheti with an en dash (–)**, while Geostat uses a hyphen (-). A name join fails on this one unless `Map_key` copies the en dash.
- Abkhazia and Tskhinvali (South Ossetia) are not covered by either census. Abkhazia is a map polygon with no data rows; Tskhinvali has no separate polygon, so on the map it is part of the neighbouring regions (mostly Shida Kartli), whose census figures do not include it. Say so in the map note.
- Foreign citizens (6.6% of the 2024 population) are included in the published 2024 figures.

---

## A8. Region names: Hungary

`Region` = KSH name without the word *vármegye*. The map (HDX) writes the names **without accents**, and calls Pest "Pest megye".

| `Region` | KSH Excel label | KSH CSV code (NUTS 3) | Map name (HDX `adm1_name`) | HDX P-code | 2011 population | 2022 population |
|---|---|---|---|---|---|---|
| Budapest | Budapest | HU110 | Budapest | HU005 | 1 729 040 | 1 685 342 |
| Pest | Pest vármegye | HU120 | Pest megye | HU014 | 1 217 476 | 1 333 533 |
| Fejér | Fejér vármegye | HU211 | Fejer | HU007 | 425 847 | 419 656 |
| Komárom-Esztergom | Komárom-Esztergom vármegye | HU212 | Komarom Esztergom | HU012 | 304 568 | 300 631 |
| Veszprém | Veszprém vármegye | HU213 | Veszprem | HU019 | 351 898 | 335 361 |
| Győr-Moson-Sopron | Győr-Moson-Sopron vármegye | HU221 | Gyor Moson Sopron | HU008 | 447 985 | 465 945 |
| Vas | Vas vármegye | HU222 | Vas | HU018 | 256 629 | 249 513 |
| Zala | Zala vármegye | HU223 | Zala | HU020 | 282 179 | 260 800 |
| Baranya | Baranya vármegye | HU231 | Baranya | HU002 | 386 441 | 354 022 |
| Somogy | Somogy vármegye | HU232 | Somogy | HU015 | 317 307 | 293 470 |
| Tolna | Tolna vármegye | HU233 | Tolna | HU017 | 230 361 | 207 931 |
| Borsod-Abaúj-Zemplén | Borsod-Abaúj-Zemplén vármegye | HU311 | Borsod Abauj Zemplen | HU004 | 686 266 | 623 024 |
| Heves | Heves vármegye | HU312 | Heves | HU010 | 308 882 | 285 892 |
| Nógrád | Nógrád vármegye | HU313 | Nograd | HU013 | 202 427 | 182 459 |
| Hajdú-Bihar | Hajdú-Bihar vármegye | HU321 | Hajdu Bihar | HU009 | 546 721 | 519 141 |
| Jász-Nagykun-Szolnok | Jász-Nagykun-Szolnok vármegye | HU322 | Jasz Nagykun Szolnok | HU011 | 386 594 | 355 809 |
| Szabolcs-Szatmár-Bereg | Szabolcs-Szatmár-Bereg vármegye | HU323 | Szabolcs Szatmar Bereg | HU016 | 559 272 | 529 381 |
| Bács-Kiskun | Bács-Kiskun vármegye | HU331 | Bacs Kiskun | HU001 | 520 331 | 495 318 |
| Békés | Békés vármegye | HU332 | Bekes | HU003 | 359 948 | 315 222 |
| Csongrád-Csanád | Csongrád-Csanád vármegye | HU333 | Csongrad | HU006 | 417 456 | 391 184 |
| **Total** | | | | | **9 937 628** | **9 603 634** |

- In the Excel file, **"Budapest" and "Pest" appear twice**: once as a region (NUTS 2) and once as a county. The region rows ("Budapest" region, "Pest" without *vármegye*) have the same figures and must be dropped. So must "Magyarország" and the seven "… régió" rows.
- The map's name for Csongrád-Csanád is the old name "Csongrad".
- The HDX P-codes (HU001 …) are alphabetical and differ from the KSH NUTS codes.

---

## A9. Capital flags (both the city alone and the wider capital area)

| Country | `Capital_city` = Yes | `Capital_area` = Yes |
|---|---|---|
| Hungary | Budapest | Budapest, Pest |
| Georgia | Tbilisi | Tbilisi (no wider unit) |
| Nepal | Kathmandu | Kathmandu, Lalitpur, Bhaktapur |

All other units: `No`.

---

## A10. Decisions for the master file

| Point | Decision |
|---|---|
| **`Map_key`** | **Map-file names**, copied exactly: the "Map name" columns of A6 (Nepal), A7 (Georgia) and A8 (Hungary). The two dissolved Nepali polygons are named **`Nawalparasi`** and **`Rukum`** (set when dissolving), the same as their `Region` |
| Georgian `Region` names | As in A7 |
| Province of merged Nepali districts | Larger half in 2021: Nawalparasi → Lumbini, Rukum → Karnali (A5) |
| HDX layer for Hungary and Nepal | Plain layers (`hun_admin1.geojson`, `npl_admin2.geojson`). Neighbouring countries are not drawn, so edge-matching doesn't matter |

### `Map_key` traps to watch

| Country | Trap | Example |
|---|---|---|
| Hungary | No accents, spaces instead of hyphens, "Pest megye", old name "Csongrad" | `Gyor Moson Sopron`, `Pest megye`, `Csongrad` |
| Georgia | En dash in one name; short names | `Samtskhe–Javakheti`, `Tbilisi`, `Adjara` |
| Nepal | Merged polygons renamed when dissolving | `Nawalparasi`, `Rukum` |
