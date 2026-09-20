# CongestionView CA

**Spatial analysis of traffic congestion across California metropolitan regions using machine learning, spatial statistics, and geovisual analytics.**

`Python` · `GeoPandas` · `scikit-learn` · `PySAL` · `OSMnx` · `HERE Traffic API` · `BigQuery` · `CARTO`

> **Graduate Capstone Project — M.S. Spatial Data Science, Penn State University**

---

## Overview

CongestionView CA is a spatial data science project designed to identify and interpret patterns of roadway congestion across six California metropolitan regions:

**Fresno · Los Angeles · Sacramento · San Diego · San Francisco · San Jose**

The project integrates traffic-flow observations from the HERE Traffic API with OpenStreetMap roadway data and applies machine learning and spatial statistical methods to characterize congestion across roadway networks and peak travel periods.

The analysis combines **K-Means clustering**, **Global Moran's I**, and **Local Indicators of Spatial Association (LISA)** with interactive geovisual analytics dashboards to explore where congestion occurs, whether it exhibits statistically significant spatial structure, and how those patterns vary among metropolitan regions and roadway classes.

### Research Question

> To what extent does traffic congestion exhibit spatial clustering by road class and peak travel periods within California metropolitan regions, and can Geovisual Analytics and Machine Learning methods be used to help detect and interpret these patterns?

---

## Project Workflow

```text
HERE Traffic API                 OpenStreetMap
       │                              │
       │ Traffic Flow                 │ Road Network
       │                              │
       └──────────────┬───────────────┘
                      ▼
              Spatial Integration
               GeoPandas / OSMnx
                      │
                      ▼
              Feature Engineering
          ┌───────────┴───────────┐
          │                       │
     Jam Factor              Speed Ratio
     Road Class             Spatial Location
          │                       │
          └───────────┬───────────┘
                      ▼
               Spatial Analysis
          ┌───────────┴────────────┐
          │                        │
       K-Means                Moran's I
      Clustering                + LISA
          │                        │
          └───────────┬────────────┘
                      ▼
              Geospatial Outputs
                      │
                      ▼
              BigQuery / CARTO
                      │
                      ▼
          Interactive Dashboards
```

---

## Data

### HERE Traffic API

Traffic-flow observations were collected using the HERE Traffic API v7. Relevant attributes included:

* current traffic speed
* free-flow speed
* jam factor
* roadway geometry
* road name

A **speed ratio** was calculated as:

```text
Speed Ratio = Current Speed / Free-Flow Speed
```

Values were capped at `1.0`, with lower values indicating greater deviation from free-flow conditions.

A composite congestion score was also calculated:

```text
Congestion Score = (1 − Speed Ratio) + Jam Factor
```

### OpenStreetMap

OpenStreetMap road-network data were retrieved with OSMnx and spatially joined to HERE traffic segments.

OSM data supplied additional roadway characteristics including:

* highway / road classification
* roadway names
* network geometry

The analysis focused on drivable roadway networks within each study region.

> The complete traffic dataset is not distributed with this repository. See [`data/README.md`](data/README.md) for additional information.

---

## Machine Learning

### K-Means Clustering

K-Means clustering was used to group roadway segments according to similar traffic and spatial characteristics.

Features incorporated into the clustering workflow included:

* Jam Factor
* Speed Ratio
* OSM highway classification
* projected segment centroid coordinates

Features were standardized using `StandardScaler` prior to clustering.

The primary model used **k = 3**, with clusters subsequently interpreted according to their congestion characteristics:

| Cluster Interpretation | Description                                                    |
| ---------------------- | -------------------------------------------------------------- |
| **Free Flowing**       | Relatively low congestion and speeds near free-flow conditions |
| **Moderate Flow**      | Intermediate traffic conditions                                |
| **Congested**          | Higher congestion and greater deviation from free-flow speeds  |

Cluster labels were assigned dynamically by comparing average Jam Factor and Speed Ratio rather than assuming that the numeric K-Means cluster IDs represented a particular congestion level.

---

## Spatial Statistics

### Global Moran's I

Global Moran's I was used to determine whether roadway congestion exhibited spatial autocorrelation within each metropolitan region.

Traffic segments were projected to **EPSG:3857**, and segment centroids were used to construct spatial weights.

The analysis used:

* **K-nearest-neighbor weights:** `k = 8`
* **Row-standardized weights**
* **9,999 permutations**
* **Congestion score** as the analysis variable

Invalid geometries and exact duplicate centroids were removed prior to constructing the spatial weights.

Across the study regions and observation periods, the results generally indicated **positive but varying levels of spatial autocorrelation**, showing that congestion exhibited measurable spatial structure rather than being distributed independently across the roadway network.

### Local Indicators of Spatial Association

Local Moran's I was used to identify localized congestion relationships.

Segments were classified as:

| LISA Classification | Interpretation                                 |
| ------------------- | ---------------------------------------------- |
| **High–High**       | High values surrounded by high values          |
| **Low–Low**         | Low values surrounded by low values            |
| **High–Low**        | High value surrounded by lower values          |
| **Low–High**        | Low value surrounded by higher values          |
| **Not Significant** | Local relationship not significant at p ≤ 0.05 |

LISA results were also compared with OSM roadway classifications to investigate whether particular local spatial relationships were associated with specific types of roads.

Because local significance testing is performed across many roadway segments, the LISA results should be interpreted as exploratory spatial evidence; multiple local tests increase the possibility of Type I error.

---

## Key Findings

### Congestion conditions varied among metropolitan regions

The composition of the K-Means congestion classes differed across the six study areas. Fresno, Sacramento, San Francisco, and San Jose contained larger shares of roadway observations classified as **Moderate Flow**, while Los Angeles and San Diego contained larger shares classified as **Free Flowing** within the collected observations.

### Congestion exhibited measurable spatial structure

Global Moran's I was positive across the analyzed snapshots, although the magnitude varied by region and observation period. This indicates that roadway congestion exhibited spatial organization that could be quantified using spatial statistical methods.

Some region-period combinations showed relatively weak spatial structure, demonstrating that the strength of congestion clustering was not uniform across metropolitan areas or observation periods.

### Local congestion relationships were more complex than simple hotspot patterns

LISA analysis showed that statistically significant local relationships included not only High–High and Low–Low clusters but also High–Low and Low–High spatial outliers.

When significant LISA observations were examined by roadway class, **High–Low relationships were frequently prominent**, while High–High clusters were more selective across roadway classes and regions.

These results suggest that local congestion patterns cannot be explained solely as concentrated congestion along a single category of major roadway.

### Interactive visualization supported interpretation of the statistical results

The final geovisual analytics environment connected statistical classifications with their geographic context, allowing congestion patterns to be examined by location, roadway type, congestion class, and observation period.

---

## Interactive Geovisual Analytics

Two CARTO dashboards were developed to support interactive exploration of the results.

### Congestion Clusters Dashboard

The congestion dashboard visualizes K-Means classifications and traffic conditions across the six metropolitan regions.

Interactive components include:

* congestion cluster segment summaries
* historical time-series visualization
* congestion quartile distribution
* Speed Ratio histogram
* roadway-level popups
* roadway-type filtering
* viewport-responsive statistics

https://clausa.app.carto.com/map/8169f426-4359-416f-9d48-64b65d0fae04

### Spatial Statistics Dashboard

The spatial statistics dashboard focuses on Local Moran's I results and their relationship with congestion.

Components include:

* LISA cluster map
* LISA classification distribution
* LISA time series
* congestion cluster summaries
* congestion-score distribution
* roadway-level LISA statistics
* viewport-responsive visualizations

https://clausa.app.carto.com/map/cbb4ac2e-fabe-4ae7-ba9a-87a270b06862

---

## Repository Structure

```text
congestionview-ca/
│
├── notebooks/
│   ├── 01_congestion_pipeline.ipynb
│   ├── 02_clustering_statistics.ipynb
│   ├── 03_spatial_statistics.ipynb
│   └── 04_lisa_road_class_analysis.ipynb
│
├── src/
│   ├── congestion_pipeline.py
│   ├── clustering_statistics.py
│   └── lisa_road_class_statistics.py
│
├── data/
│   └── README.md
│
├── outputs/
│   └── README.md
│
├── figures/
│
├── docs/
│   └── congestionview-ca-final-report.pdf
│
├── .env.example
├── .gitignore
├── requirements.txt
├── LICENSE
└── README.md
```

### Notebooks

**`01_congestion_pipeline.ipynb`**
Primary analytical workflow, including HERE traffic collection, OSM enrichment, feature engineering, K-Means clustering, congestion scoring, Moran's I, and LISA.

**`02_clustering_statistics.ipynb`**
Summarizes congestion-cluster composition across metropolitan regions and AM/PM travel periods.

**`03_spatial_statistics.ipynb`**
Summarizes and compares Global Moran's I results across regions and observation periods.

**`04_lisa_road_class_analysis.ipynb`**
Examines the relationship between statistically significant LISA classifications and OSM roadway classes.

---

## Running the Project

### 1. Clone the repository

```bash
git clone https://github.com/YOUR-USERNAME/congestionview-ca.git
cd congestionview-ca
```

### 2. Create a Python environment

```bash
python -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure HERE API access

Create a `.env` file or configure the `HERE_API_KEY` environment variable using your own HERE API credentials.

```bash
export HERE_API_KEY="your_api_key"
```

API credentials are intentionally excluded from this repository.

### 5. Run the analysis

The notebooks in [`notebooks/`](notebooks/) document the research workflow and analysis.

Reusable Python implementations are available under [`src/`](src/).

Some analyses depend on research datasets that are not distributed with the repository and therefore cannot be reproduced directly from the repository alone.

---

## Technologies

**Data Analysis & Machine Learning**

Python · pandas · NumPy · scikit-learn

**Geospatial Analysis**

GeoPandas · OSMnx · Shapely · PySAL · ESDA

**Visualization**

CARTO · Folium · Matplotlib

**Data & Infrastructure**

HERE Traffic API · OpenStreetMap · BigQuery · GeoJSON

---

## Research Report

The complete capstone report provides additional information about the research design, literature, methodology, statistical results, limitations, and interpretation.

**[Read the full research report](docs/congestionview-ca-final-report.pdf)**

---

## Limitations & Future Work

The project represents traffic conditions observed during selected collection periods rather than a continuous long-term traffic monitoring dataset. Expanding data collection across longer temporal periods would support stronger comparisons among weekdays, weekends, seasons, and changing travel conditions.

Future development could include:

* automated cloud-based traffic-data collection
* longer-term temporal analysis
* predictive congestion modeling
* additional spatial-weight specifications
* evaluation of the interactive dashboards with transportation professionals and other intended users
* deployment of a public-facing web application for exploring the analysis

---

## Author

**Patrick Palmer**

M.S. Spatial Data Science — Penn State University

Spatial data analysis · transportation analytics · geospatial machine learning · geovisual analytics
