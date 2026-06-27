# Prague Urban Cafe Oasis Analysis cz
An empirical data science project modeling the balance between urban commercial density and public green infrastructure. This project evaluates urban walkability and spots "concrete deserts" across Prague by isolating **"Cafe Oases"**—coffee shops located outside a standard 10-minute urban walking radius (500 meters) from any public park—and generating a neighborhood-level statistical leaderboard.

## Project Overview
Urban green spaces are vital to human mental well-being and mitigating urban heat islands. Conversely, coffee shops serve as premier indoor "Third Places" that anchor community vitality and walkability.

This project maps the structural intersection of these two spatial assets in Prague, Czechia. By evaluating the geographic proximity of every registered cafe to public parks, the project highlights which administrative districts display optimal green-commercial integration, and which act as localized commercial hotspots cut off from nature.

## Data Pipeline & Architecture
The workflow transitions through an end-to-end vector pipeline optimized for precise spatial geometry operations:

<img width="1672" height="1049" alt="Prague Cafe Oases Analysis - Workflow" src="https://github.com/user-attachments/assets/7b8daa67-7d2d-4734-b368-e3a64d89ece3" />


### 1. Layer Separation & Homogeneity
Raw vector spatial features are ingested from OpenStreetMap via `OSMnx`. The data is cleaned and separated into geometrically homogeneous layers to ensure topological validity:
- **Points:** `amenity == 'cafe'`
- **Polygons:** `leisure == 'park'`
### 2. Coordinate Reference System (CRS) Transformation
Geographic coordinates are initially evaluated using angular degrees (**WGS 84 / `EPSG:4326`**). Because degree lengths vary depending on latitude, flat metric calculations are impossible. The pipeline explicitly reprojects all vectors to **Web Mercator (`EPSG:3857`)**, translating spatial arrays into flat planar meters.
### 3. Proximity Buffering & Dissolving
A 500-meter expansion vector boundary (`.buffer(500)`) is cast around all park geometries to simulate a standard 10-minute pedestrian walking radius. Overlapping buffer boundaries are melted into a unified, continuous multi-polygon vector mask (`.dissolve()`), establishing a seamless binary proximity canvas that eliminates duplicate point-counting inside overlapping zones.
### 4. Spatial Inversion Filtering
Using the dissolved park mask, an optimized Point-in-Polygon spatial join (`gpd.sjoin`) flags all cafes located _inside_ park zones. A pandas bitwise NOT operator (`~`) inverts this index, isolating the true **Cafe Oases** stranded outside the 500-meter threshold.
### 5. Neighborhood Aggregation & Density Normalization
Prague's municipal administrative boundaries (`admin_level='9'`) are loaded and joined with the isolated points via `predicate='contains'`. Grouped counts are stitched back to the full district grid using a **Left Join** (`how='left'`) and padded with `.fillna(0)` to preserve zero-count districts. To prevent geographical size bias, raw counts are normalized against calculated land area:

$$\text{Oasis Density} = \frac{\text{Cafe Oases Count}}{\text{District Area in Sq KM}}$$

## Core Findings & Statistical Insights
Running descriptive summary statistics (`.describe()`) over the final dataset yields a definitive empirical verdict regarding Prague's urban blueprint:

| Metric                           | Computed Value              |
| -------------------------------- | --------------------------- |
| Total Cafes Analysed             | 1108                        |
| Isolated Cafe Oases (>500m away) | 131                         |
| Percentage of Isolated Cafes     | 11.82%                      |
| District Mean Density            | 0.114 cafes / $\text{km}^2$ |
| District Median (50% Percentile) | 0.000 cafes / $\text{km}^2$ |
| District Maximum Density         | 3.348 cafes / $\text{km}^2$ |

## Analytical Conclusions
- **Pervasive Green Integration:** The Median Density is a hard 0.00. This mathematically proves that in over 50% of Prague's administrative districts, 100% of coffee shops are located within a 10-minute walk of a park.
- **Extreme Right Skew:** The massive gap between the Median (0.00) and the Mean (0.114), combined with the heavily spiked maximum value (3.35), confirms that coffee-to-nature isolation is not a systemic city-wide crisis.
- **Hyper-Localized Commercialization:** Prague features exceptional green accessibility overall. The isolated "Oases" are highly concentrated anomalies confined almost exclusively to historic, ultra-dense commercial centers (e.g., _Praha 1 / Old Town_) where historic preservation and intensive commercial development have locked out green infrastructure.
