# Monitoring Lake Urmia Surface Water Changes (2025–2026)

This project utilizes Google Earth Engine (GEE) to analyze and visualize the dramatic hydrological shifts in Lake Urmia, specifically focusing on the transition from the critical "Terminal Nadir" observed in late 2025 to the "Pulse Recovery" throughout the first half of 2026.

## Project Overview

Lake Urmia, located in northwestern Iran, has transitioned from one of the world's largest hypersaline lakes to a highly volatile basin. By integrating multi-sensor Landsat 8/9 data, this repository captures the contrast between the severe desiccation of 2025 and the sudden, spring-fed recovery in 2026. This research provides a data-driven look at the lake’s resilience and the impacts of regional water management and seasonal inflows. The analysis used a baseline "Nadir" composite from August 1 to September 30, 2025, compared against a "Pulse" composite from April 1 to May 31, 2026, and a final "Summer Equilibrium" observation from June 1 to June 25, 2026.

## Key Features

* **Multi-Phase Temporal Analysis:** Quantifies surface water extent across three distinct hydrological states: Terminal Nadir (2025), Spring Pulse (2026), and Summer Equilibrium (2026).
* **Change Detection Mapping:** A specialized visual layer highlighting "Persistent Water" (blue), "New Pulse Water" (cyan), and "Lost Water" (red).
* **Geopolitical Contextualization:** By mapping the spatial distribution of inflows, this research serves as a foundation for understanding the "Lagged Inflow Effect" influenced by dam management and climate variability.
* **Automated Data & Visual Export:** Scripts provided to export statistical trends and high-resolution maps directly to Google Drive.

## Visualizations

### 1. Surface Water Area Trend

This analysis captures recent volatility, documenting the shift from the lake’s low of ~2,702 km² in late 2025 to a recovery of ~3,363 km² by mid-2026.
Water Area 2025 (sq meters): 2702393424.7341213
Water Area 2026 (sq meters): 3307014090.704633
Water Area June 2026 (sq meters): 3363486774.940714

### 2. The "Pulse" Change Map

This map visualizes the spatial footprint of the recovery:

* **Blue:** Permanent/Persistent water.
* **Cyan:** New water coverage (Pulse).
* **Red:** Areas of continued desiccation.

### 3. Temporal Comparison (2025 vs. 2026)

A side-by-side comparison illustrating the rapid inundation of the flattened playa bed during the 2026 spring season.
### 1. Terminal Nadir (Aug-Sep 2025)
![Nadir 2025](visuals/LakeUrmia_Nadir_2025.png)

### 2. Pulse Recovery (Apr-May 2026)
![Pulse 2026](visuals/LakeUrmia_Pulse_2026.png)

### 3. Current Status (June 2026)
![Current 2026](visuals/LakeUrmia_Current_2026.png)

### 4. Recovery Change Map
![Change Map](visuals/Recovery_Change_Map.png)

## Google Earth Engine Script

The following script performs the multispectral classification using Landsat 8/9, computes the MNDWI (Modified Normalized Difference Water Index), and generates the area statistics.

```javascript
// 1. Define ROI and Merge Landsat 8/9 Collections
var roi = geometry; // Ensure your geometry polygon is imported
var l8 = ee.ImageCollection("LANDSAT/LC08/C02/T1_L2");
var l9 = ee.ImageCollection("LANDSAT/LC09/C02/T1_L2");
var mergedCol = l8.merge(l9).map(function(image) {
  var opticalBands = image.select('SR_B.').multiply(0.0000275).add(-0.2);
  return image.addBands(opticalBands, null, true);
});

// 2. Filter and Create Composites
var filtered = mergedCol.filterBounds(roi).filter(ee.Filter.lt('CLOUD_COVER', 10));

var nadir2025 = filtered.filterDate('2025-08-01', '2025-09-30').median().clip(roi);
var pulse2026 = filtered.filterDate('2026-04-01', '2026-05-31').median().clip(roi);
var summer2026 = filtered.filterDate('2026-06-01', '2026-06-25').median().clip(roi);

// 3. MNDWI Classification Function
function addMNDWI(image) {
  return image.addBands(image.normalizedDifference(['SR_B3', 'SR_B6']).rename('MNDWI'));
}

// 4. Area Calculation Logic
var waterPulse = addMNDWI(pulse2026).select('MNDWI').gt(0);
var areaPulse = waterPulse.multiply(ee.Image.pixelArea()).reduceRegion({
  reducer: ee.Reducer.sum(),
  geometry: roi,
  scale: 30,
  maxPixels: 1e13
});
print('Pulse Recovery Area (m²):', areaPulse.get('MNDWI'));

// 5. Change Detection Visualization
var changeMap = waterPulse.add(addMNDWI(nadir2025).select('MNDWI').gt(0).multiply(2));
Map.addLayer(changeMap.selfMask(), {min: 1, max: 3, palette: ['cyan', 'red', 'blue']}, 'Recovery Change Map');

```

---

