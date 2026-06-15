# Jasper 2024 Wildfire Burn Severity Mapping
## Remote Sensing Project — Sentinel-2 + ArcGIS Pro
**Method:** UN-SPIDER Recommended Practice (dNBR)  
**Study Area:** Jasper National Park, Alberta, Canada  
**Fire Event:** Jasper Wildfire Complex, July–August 2024

---

## Background

The 2024 Jasper wildfire (started ~July 22, 2024) was one of Canada's most destructive urban-interface fires in decades. It burned ~358 km² in Jasper National Park and destroyed approximately 30–40% of the Jasper townsite. This project maps burn severity using the Normalized Burn Ratio (NBR) differencing method.

---

## Project Folder Structure

```
jasper_burn_severity/
├── 01_raw_data/
│   ├── pre_fire/          ← Sentinel-2 imagery before fire (June 2024)
│   └── post_fire/         ← Sentinel-2 imagery after fire (Aug–Sep 2024)
├── 02_processed/
│   ├── pre_fire_corrected/    ← After atmospheric correction
│   └── post_fire_corrected/   ← After atmospheric correction
├── 03_analysis/
│   ├── nbr/               ← Pre_NBR.tif and Post_NBR.tif
│   ├── dnbr/              ← dNBR raster
│   └── classified/        ← Final classified map + reclass rules
├── 04_boundaries/         ← Jasper NP boundary shapefile
├── 05_outputs/
│   ├── maps/              ← Exported map images/PDFs
│   └── reports/           ← Area statistics report
└── PROJECT_GUIDE.md       ← This file
```

---

## Key Formulas

```
NBR  = (Band 8A − Band 12) / (Band 8A + Band 12)
dNBR = Pre-fire NBR − Post-fire NBR
```

**USGS Burn Severity Classification (dNBR values):**

| dNBR Range       | Class                    |
|-----------------|--------------------------|
| < −0.500        | Enhanced Regrowth, High  |
| −0.500 to −0.251| Enhanced Regrowth, Low   |
| −0.250 to −0.101| Unburned                 |
| −0.100 to 0.099 | Low Severity             |
| 0.100 to 0.269  | Moderate-Low Severity    |
| 0.270 to 0.439  | Moderate-High Severity   |
| > 0.440         | High Severity            |

---

## STEP 1 — Download Sentinel-2 Data

### Register & Login
1. Go to: **https://dataspace.copernicus.eu**
2. Create a free account (or login if you have one)
3. Click "Copernicus Browser" to access the search interface

### Search Parameters for Pre-fire Image

| Parameter         | Value                                      |
|------------------|---------------------------------------------|
| Area of Interest | Draw box around Jasper, AB (52.8°N, 118.1°W)|
| Date Range       | **2024-06-01 to 2024-07-20**               |
| Satellite        | Sentinel-2                                  |
| Product Type     | L1C or L2A                                  |
| Max Cloud Cover  | < 10%                                       |

**Target tile:** `11UNT` or `11UNS` (covers Jasper area)

**Recommended pre-fire image:**  
Look for an image from **June or early July 2024** with minimal cloud cover over Jasper townsite and the Athabasca River valley.

### Search Parameters for Post-fire Image

| Parameter         | Value                                      |
|------------------|---------------------------------------------|
| Date Range       | **2024-08-10 to 2024-09-30**               |
| Same tile        | Same as pre-fire (11UNT or 11UNS)           |
| Max Cloud Cover  | < 10%                                       |

**Recommended post-fire image:**  
Look for **August or September 2024** after the fire was controlled.

### What to Download
- For each image (pre and post), download the full `.zip` product
- After unzipping, you need these files from `GRANULE/.../IMG_DATA/`:
  - **Band 8A** (`*B8A*.jp2`) — Near-Infrared (NIR), 20m resolution
  - **Band 12** (`*B12*.jp2`) — Short-Wave Infrared (SWIR), 20m resolution
- Also save cloud mask from `QI_DATA/MSK_CLOUDS_B00.gml`

### Save Files To:
- Pre-fire bands → `01_raw_data/pre_fire/`
- Post-fire bands → `01_raw_data/post_fire/`

---

## STEP 2 — Download Jasper National Park Boundary

### Option A: Statistics Canada (Recommended)
1. Go to: **https://www12.statcan.gc.ca/census-recensement/2021/geo/sip-pis/boundary-limites/index2021-eng.cfm**
2. Download "National Parks" or search for protected areas shapefile

### Option B: Parks Canada Open Data
1. Go to: **https://open.canada.ca/data/en/dataset**
2. Search: "Jasper National Park boundary"
3. Download as shapefile (.shp)

### Option C: Direct from Parks Canada GIS
- URL: **https://atlas.gc.ca** (National Atlas of Canada)
- Search for Jasper National Park polygon

Save the boundary shapefile to: `04_boundaries/`

---

## STEP 3 — Open Images in QGIS

1. Open QGIS
2. In the **Browser Panel**, navigate to `01_raw_data/pre_fire/`
3. Find the `.jp2` files for B8A and B12
4. Drag both into the map canvas
5. Repeat for post-fire bands

---

## STEP 4 — Atmospheric Correction (TOA)

### Install SCP Plugin
- Menu: **Plugins → Manage and Install Plugins**
- Search: `Semi-Automatic Classification`
- Click **Install**

### Apply DOS1 Correction to Pre-fire Images
1. **SCP → Preprocessing → Sentinel-2**
2. Set input folder: `01_raw_data/pre_fire/`
3. Select the MTD_MSIL1C.xml metadata file in the same folder
4. Check: **Apply DOS1 atmospheric correction**
5. Check: **Use NoData value**
6. Remove all bands **except 8A and 12** (click the minus icon for others)
7. Click **Run** → set output to `02_processed/pre_fire_corrected/`

### Apply DOS1 Correction to Post-fire Images
- Repeat the same steps using `01_raw_data/post_fire/` as input
- Output to `02_processed/post_fire_corrected/`

> **Note:** If you downloaded L2A products (already surface reflectance), you can skip the DOS1 correction step and use the bands directly.

---

## STEP 5 — Process Cloud Masks

For each image (pre and post):
1. Load `MSK_CLOUDS_B00.gml` from the `QI_DATA/` folder
2. Right-click layer → **Save As** → save as shapefile in `02_processed/`
3. Match the CRS to your other layers (EPSG:32611 for Jasper area)
4. Open the **Attribute Table** of the saved shapefile
5. Enable **Toggle Editing Mode**
6. Open **Field Calculator** → Create new field:
   - Name: `value`
   - Type: Integer
   - Expression: `1`
7. Save and disable editing mode

---

## STEP 6 — Calculate Pre-fire NBR

1. **SCP → Band Calc**
2. Click the **Refresh** button to list available rasters
3. Double-click pre-fire Band 8A → it appears as `raster1`
4. Double-click pre-fire Band 12 → it appears as `raster2`
5. Enter formula:
   ```
   (raster1 - raster2) / (raster1 + raster2)
   ```
6. Click **Run** → save output as `03_analysis/nbr/Pre_NBR.tif`

---

## STEP 7 — Calculate Post-fire NBR

- Repeat Step 6 using post-fire corrected bands
- Save output as `03_analysis/nbr/Post_NBR.tif`

---

## STEP 8 — Calculate dNBR

1. **SCP → Band Calc**
2. Click **Refresh**
3. Double-click `Pre_NBR` → `raster1`
4. Double-click `Post_NBR` → `raster2`
5. Formula:
   ```
   raster1 - raster2
   ```
6. Save as `03_analysis/dnbr/dNBR_jasper.tif`

---

## STEP 9 — Apply Cloud Mask

1. **Raster → Conversion → Rasterize**
2. Input layer: pre-fire cloud mask shapefile
3. Output: select the dNBR raster file
4. Click pencil icon → add flag: `-burn 500`
5. Click **Run**
6. Repeat for post-fire cloud shapefile

---

## STEP 10 — Clip to Study Area

1. Load Jasper NP boundary: **Layer → Add Layer → Add Vector Layer**
2. **SCP → Preprocessing → Clip Multiple Rasters**
3. Refresh → select `dNBR_jasper.tif`
4. Check: **Use shapefile for clipping**
5. Refresh to show the boundary shapefile → select it
6. Output prefix: `Jasper`
7. Output folder: `03_analysis/dnbr/`

---

## STEP 11 — Classify Burn Severity (ArcGIS or QGIS)

### In QGIS (Symbology method):
1. Right-click clipped dNBR layer → **Properties → Symbology**
2. Change Render type: **Singleband Pseudocolor**
3. Add 7 classes manually with these value ranges and colors:

| Value Range    | Label                  | Suggested Color |
|---------------|------------------------|----------------|
| < −0.500      | Enhanced Regrowth High | Dark Green     |
| −0.500–−0.251 | Enhanced Regrowth Low  | Light Green    |
| −0.250–−0.101 | Unburned               | Yellow-Green   |
| −0.100–0.099  | Low Severity           | Yellow         |
| 0.100–0.269   | Moderate-Low Severity  | Orange         |
| 0.270–0.439   | Moderate-High Severity | Dark Orange    |
| > 0.440       | High Severity          | Dark Red       |

### In ArcGIS (user to complete):
- Use **Reclassify** tool with the USGS ranges above
- Or use **Classify** in Symbology with Manual method

---

## STEP 12 — Calculate Area Statistics

### Convert dNBR to integers (×1000):
1. **Raster → Raster Calculator**
2. Expression: `"Jasper_dNBR" * 1000`
3. Save as `Jasper_dNBR_1000.tif`

### Reclassify:
1. **Processing → Toolbox** → search `r.reclass`
2. Input: `Jasper_dNBR_1000.tif`
3. Rules file: `03_analysis/classified/USGS_Fire_Reclass_Rules.txt`
4. Output: `03_analysis/classified/Jasper_classified.tif`

### Area report:
1. **Processing → Toolbox** → search `r.report`
2. Input: `Jasper_classified.tif`
3. Units: `h` (hectares) or `k` (km²)
4. Save report to `05_outputs/reports/burn_severity_report.txt`

---

## STEP 13 — Map Composition

1. **Project → New Print Layout**
2. Add Map frame
3. Add Legend (remove irrelevant layers)
4. Add Scale Bar (km units)
5. Add Title: "Jasper Wildfire 2024 — Burn Severity Map"
6. Add North Arrow
7. Add text box with data source, date, and your name
8. Export to `05_outputs/maps/` as PDF and PNG

---

## Data Sources & Credits

| Data | Source | URL |
|------|--------|-----|
| Sentinel-2 imagery | Copernicus/ESA | https://dataspace.copernicus.eu |
| Park boundary | Parks Canada / StatCan | https://open.canada.ca |
| Classification scheme | USGS | Key & Benson, 2006 |
| Method | UN-SPIDER | https://un-spider.org |

---

## Portfolio Notes

For your portfolio, highlight:
- The scale and significance of the Jasper 2024 fire (national news, > 358 km²)
- Your use of free, open-source satellite data (Copernicus)
- Application of internationally recognized USGS classification scheme
- Quantified area statistics by severity class
- Relevant to emergency management, Parks Canada, insurance, and restoration planning
