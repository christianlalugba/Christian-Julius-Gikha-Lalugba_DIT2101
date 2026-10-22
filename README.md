# Healthcare Facilities Distribution in Sierra Leone — Data Exploration & Cleaning

## Overview and SDG 3 Link
Access to functional healthcare facilities varies across Sierra Leone's 16 districts, with rural and newer districts having far fewer facilities than major urban centres. This project explores, cleans, and visualises the national healthcare facility registry in support of **United Nations Sustainable Development Goal 3 (SDG 3: Good Health and Well-being)**, specifically **Target 3.8 (Achieve Universal Health Coverage)**. By diagnosing and fixing data-quality issues in the raw facility records, the project produces an audited, analysis-ready dataset and district-level visualisations of Sierra Leone's healthcare infrastructure.

---

## Dataset Information
* **Dataset Name:** Sierra Leone National Health Facilities Dataset Version 01 (`data/sle_hltfac_grid3_v01.csv`)
* **Publisher / Source:** Sierra Leone Ministry of Health and Sanitation (MoHS) and Center for International Earth Science Information Network (CIESIN), Columbia University — Geo-Referenced Infrastructure and Demographic Data for Development (**GRID3**), July 2023
* **Dataset URL:** https://doi.org/10.7916/ag19-s334
* **Licence:** Creative Commons Attribution 4.0 International (`CC BY 4.0`)
* **Raw Dataset Size:** `1,616` rows and `15` columns
* **Cleaned Dataset Size:** `1,614` rows and `16` columns (`14` cleaned original columns after dropping `comments`, plus `2` imputation flag columns `geo_date_imputed` and `coords_imputed`), with `0` missing values

---

## Folder Structure
```text
.
├── data/
│   ├── sle_hltfac_grid3_v01.csv                     # Raw dataset (1,616 rows × 15 columns, unmodified)
│   └── cleaned_data.csv                             # Cleaned dataset (1,614 rows × 16 columns, 0 missing values)
├── outputs/
│   ├── cleaning_log.csv                             # Audit log of all cleaning steps, cells/rows affected, methods, and reasons
│   ├── chart1_facilities_by_district.png            # Raw chart: facilities per district (N = 1,616)
│   ├── chart2_facilities_by_type.png                # Raw chart: facilities by type (N = 1,616)
│   ├── chart3_facilities_by_ownership.png           # Raw chart: facilities by ownership (N = 1,616)
│   ├── chart1_facilities_by_district_cleaned.png    # Cleaned chart: facilities per district (N = 1,614)
│   ├── chart2_facilities_by_type_cleaned.png        # Cleaned chart: facilities by type (N = 1,614)
│   └── chart3_facilities_by_ownership_cleaned.png   # Cleaned chart: facilities by ownership (N = 1,614)
├── report/                                          # Folder for the assignment Word report
├── 01_exploration.ipynb                             # Notebook 1: Exploratory data analysis and raw charts
├── 02_cleaning.ipynb                                # Notebook 2: Step-by-step data cleaning and cleaning log
├── 03_cleaned_charts.ipynb                          # Notebook 3: Cleaned charts from data/cleaned_data.csv
├── README.md                                        # Project documentation
├── requirements.txt                                 # Python dependencies
└── .gitignore                                       # Git ignore rules for Python and Jupyter
```

---

## Required Libraries & Setup
This project uses Python with the following libraries (listed in `requirements.txt`):
* `pandas`
* `matplotlib`
* `jupyter`

Install them with:
```bash
pip install -r requirements.txt
```

---

## Order to Run the Notebooks
Run the three Jupyter notebooks in numerical order:

1. **`01_exploration.ipynb`**  
   Loads the raw dataset (`data/sle_hltfac_grid3_v01.csv`), inspects its shape (`1,616` rows × `15` columns), column names, data types, first and last 5 rows, summary statistics, missing values per column, duplicate records, and unique categorical values, and saves the 3 baseline charts in `outputs/`.

2. **`02_cleaning.ipynb`**  
   Loads `data/sle_hltfac_grid3_v01.csv` without modifying the raw file and performs step-by-step cleaning with before-and-after checks:
   * **Data types:** Converts `geo_date` (`1,561` non-null cells) from `float64` decimal values (e.g. `2016.0`) to whole-number integer years.
   * **Text standardisation:** Strips leading/trailing whitespace across all text columns (`15` cells affected in `community`), verifies the `16` official district names, and standardises `200` all-caps values in `type` (`'HOSPITAL'` $\rightarrow$ `'Hospital'` for `72` rows, `'CLINIC'` $\rightarrow$ `'Clinic'` for `128` rows) while keeping acronyms (`MCHP`, `CHP`, `CHC`) uppercase.
   * **Missing values:** Drops `comments` (`1,523` missing cells, `94.25%` empty); fills `source_geo` (`56` missing cells) with `'Unknown'`; flags and fills `geo_date` (`55` missing cells) with the median year (`2016`); flags and fills `lat` and `long` (`55` rows / `110` cells) with each facility's district-level median coordinates.
   * **Duplicate verification and removal:** Inspects all `6` matched `(district, community, facility)` pairs side by side. Removes **`2` confirmed duplicate rows** (`OID_ 1613` *Ninkikoro MCHP* in Tonkolili, which is identical in every column except `OID_` with the same `dhis2_id = kSlAzC2rmUu` at `0.00 km`; and `OID_ 1200` *Komoya CHP* in Karene, an incomplete copy with `dhis2_id = 'No Data'` and missing raw coordinates) while keeping **`4` same-name pairs** (`1,614` rows retained).
   * **Outlier check:** Verifies all `1,614` rows against Sierra Leone's geographic bounding box (`lat` `6.9°` to `10.0°`, `long` `-13.4°` to `-10.2°`) and the `1.5 × IQR` rule (`0` outliers found).
   * Saves the audit log to `outputs/cleaning_log.csv` and the cleaned dataset to `data/cleaned_data.csv`.

3. **`03_cleaned_charts.ipynb`**  
   Loads `data/cleaned_data.csv` (`1,614` rows) and saves the 3 cleaned charts (`*_cleaned.png`) in `outputs/`.

---

## Main Limitations
* **Estimated GPS Coordinates (`coords_imputed`):** In the raw dataset, `55` rows (`54` rows in the final `1,614` cleaned rows after removing incomplete duplicate `OID_ 1200`) were missing latitude and longitude (`lat`/`long`). Filling these with the **district median coordinate** preserves the facilities for district-level counts, but the coordinates are estimated district-centre locations rather than true street-level GPS points. All affected rows are flagged with `coords_imputed = True`.
* **Estimated Survey Years (`geo_date_imputed`):** In the raw dataset, `55` rows (`54` rows in the final cleaned dataset) were missing `geo_date` and were filled with the median year (`2016`), even though a 2016 survey did not actually take place for those facilities. All affected rows are flagged with `geo_date_imputed = True`.
* **Same-Name Facilities Kept Across Chiefdoms:** Because the CSV file does not contain a `chiefdom` column, `4` same-name `(district, community, facility)` pairs (*Kpandebu CHC* and *Sembehun MCHP* in Kenema, *Mabai MCHP* in Tonkolili, and *Seidu MCHP* in Kono) were kept because they have distinct `dhis2_id` codes and/or GPS coordinates located `18.70 km` to `52.81 km` apart.
* **Multiple Data Collection Sources:** Facility coordinates and attributes were compiled from `5` different sources (`DPPI`, `FMFL`, `SARA`, `WHO`, `NEMS`) across survey years ranging from `2015` to `2023`.
