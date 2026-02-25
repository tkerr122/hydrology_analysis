# Dolly Sods Hydrology Analysis

A Jupyter notebook for conducting a hydrological assessment of the Dolly Sods Wilderness area in West Virginia using ArcPy. The analysis delineates stream networks at multiple flow accumulation thresholds and calculates a Topographic Wetness Index (TWI) raster to identify areas of high soil moisture.

---

## Notebook

| File | Purpose |
|---|---|
| `Dolly_Sods_Hydrology.ipynb` | Full hydrology analysis — stream delineation, TWI calculation, and model evaluation |

---

## Analysis Overview

The notebook is structured into four main sections:

### 1. Data Preparation
The countywide 1m LiDAR DEM is masked to the Dolly Sods Wilderness boundary and resampled to 3m resolution using bilinear interpolation. The resampling follows Riihimäki et al. (2021), who found that the D-Infinity (Dinf) flow routing algorithm performs best at 3m resolution for TWI calculation.

### 2. Stream Delineation
Flow direction is calculated using the Dinf algorithm, followed by flow accumulation. Stream networks are then thresholded at 11 values ranging from 5,000 to 505,000 (step of 50,000) and saved as both raster and vector polyline outputs. The threshold range is grounded in Maidment (2002), who recommends a drainage area of ~4.5 km² to define a stream at 30m resolution — equivalent to a threshold of ~500,000 at 3m.

### 3. TWI Calculation
TWI is calculated using the standard equation:

```
TWI = ln(flow_accumulation / tan(slope))
```

Flow accumulation is rescaled by adding 1 and multiplying by pixel size (3m). Slope is converted from degrees to radians, with a minimum floor of 0.001 to avoid division by zero. The resulting TWI raster is reclassified into 5 categories based on percentile breaks (Meles et al., 2020):

| Class | Percentile Range | Interpretation |
|---|---|---|
| 1 | < 10th | Very dry |
| 2 | 10th–25th | Dry |
| 3 | 25th–75th | Normal |
| 4 | 75th–90th | Wet |
| 5 | > 90th | Very wet |

### 4. Model Evaluation
Predicted stream networks are validated against observed hydrography data from the USGS National Map. The observed streams are reprojected, clipped, buffered to 6m (to account for minor pixel offsets), and converted to a binary raster. Accuracy is measured as the percentage of observed stream pixels that overlap with each predicted stream network, and results are plotted as a bar chart.

---

## Outputs

- `streams_<threshold>` — binary raster stream networks for each threshold value
- `streams_poly_<threshold>` — vector polyline stream networks for each threshold value
- `overlap_<threshold>` — raster showing pixel overlap between observed and predicted streams
- `TWI_raster` — continuous TWI raster
- `TWI_reclass` — TWI raster reclassified into 5 wetness categories

All outputs are saved to the ArcGIS geodatabase defined in the environment settings.

---

## Requirements

- **Software**: ArcGIS Pro with the Spatial Analyst extension
- **Environment**: ArcPy Python environment (`arcpro` kernel)
- **Input data**:
  - Dolly Sods Wilderness boundary shapefile (`ForestWilderness.shp`)
  - 1m LiDAR DEM mosaic (`DEM_Mosaic_FEMA_2018_Tucker_Randolph_WV_1m_UTM17.tif`)
  - Hydrography shapefile for model validation (`Dolly_Sods_hydrography.shp`)

Input paths are currently hardcoded to a local directory structure and will need to be updated before running.

---

## References

- Beven, K., & Kirkby, M. J. (1979). A physically based, variable contributing area model of basin hydrology. *Hydrological Sciences Journal, 24*(1), 43–69.
- Chowdhury, S. (2023). Modelling hydrological factors from DEM using GIS. *MethodsX, 10*, 102062.
- Kopecký, M., Macek, M., & Wild, J. (2020). Topographic Wetness Index calculation guidelines based on measured soil moisture and plant species composition. *Science of the Total Environment, 757*, 143785.
- Maidment, D. R. (Ed.). (2002). *Arc Hydro: GIS for Water Resources* (p. 73). ESRI Press.
- Meles, M. B., et al. (2020). Wetness index based on landscape position and topography (WILT). *Journal of Environmental Management, 255*, 109863.
- Riihimäki, H., et al. (2021). Topographic Wetness Index as a proxy for soil moisture. *Water Resources Research, 57*(10).
