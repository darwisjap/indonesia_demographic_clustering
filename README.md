# Capstone: Regional Clustering in Indonesia for Supply Chain Optimization
---
# Introduction
---
**Background**
* Indonesia is the 4th most populous country with over 270M people.
* It is 16th largest economy in the world based on nominal GDP.
* One of the emerging markets in Southeast Asia which offers ample opportunities for business.

**Problem Statement**

* Indonesia geographical complexity increases transportation costs and logistics challenges.
* Non-uniform demographic distribution need to be compensated with a strategic planning. 

**Our Goal**

* Based on geographic and demographic characteristics in Indonesia, to define the optimum clustering of regions.
* We will use 3 different clustering models: KMEANS, weighted KMEANS, and DBSCAN, which are measured using inertia (elbow method) and silhouette score.
* Insights will be useful for potential new businesses to strategically place their distribution centres (DC) to minimize transportation cost for supply chain.

# Data Source
--- 
* Indonesia Population: [Indonesia Ministry of Home Affairs (Kemendagri)](https://prodeskel.binapemdes.kemendagri.go.id/)
* Indonesia Gross Regional Domestic Product (GPDR): [Indonesia Statistics (BPS)](https://www.bps.go.id/indicator/171/2193/1/-seri-2010-pdrb-atas-dasar-harga-berlaku-menurut-pengeluaran-kabupaten-kota.html)
* Indonesia Minimum Wage: [Indonesia Statistics (BPS)](https://www.bps.go.id/indicator/19/220/1/upah-minimum-regional-propinsi.html)
* Indonesia City GeoJSON: [Github](https://github.com/okzapradhana/indonesia-city-geojson/tree/master)

# Data Preparation
---
- Officially, there are 520 cities and regencies in Indonesia. However, our datasets do not have information for all of them. Missing cities/regencies are mainly from eastern part of Indonesia.
- City/Regencies naming are inconsistent across different datasets (i.e. use of prefix, misspelling, using of spaces). To accommodate that, we use `FuzzyWuzzy` algorithm to ensure that most of cities are preserved. Our final merged dataset has the information of 480 out of 520 cities/regencies.
- We use Google Map API to extract geo coordinate information from cities/regencies in our dataset.

# Exploratory Data Analysis
--- 
We use `Folium` to visualize demographic distribution in Indonesia:
- **Population**: Indonesia population is concentrated in Java island. This population concentration will influence how our clusters are formed when weightage is implemented.
- **Regional GDP**: As expected, capital city Jakarta has the highest regional GDP due to economic significance in the area. Other big cities such as Medan, Surabaya, Makassar, Balikpapan also have relatively high regional GDP as they are the economic hub in the area.
- **Minimum Wage**: Minimum wage is well-balance across all the regions in the country. We do not expect this feature to be influencing the clusters.

# Data Preprocessing
--- 
We have 3 features that will be weighted in one of our models. However, this model can only accept 1D array.  
We use `MinMaxScaler` to scale the features and sum them together as a 'consolidated' weight for each city/regency.  
The rationale of using MinMaxScaler over other algorithms (e.g. StandardScaler) is for all cities to have overall positive influence towards the cluster. MinMaxScaler will return `0 to 1` result as oppose to `-1 to 1` in StandarScaler which may not be suitable in our application.

# Data Modeling and Evaluation
---
We use 3 different models in our evaluation:
- K-MEANS
    - Assign K initial cluster centroids
    - Repeatedly move centroids to minimize distance between each points to the cluster centroid (SSE)
    - Follow spherical clustering behaviour
- K-MEANS (Weighted)
    - Same concept as K-means, but with additional sample weightage
    - Points with larger weight attract centroid closer to its direction
- DBSCAN
    - No predefined number of clusters (unlike K-means)
    - Clusters are formed based on ε (min. distance with neighbour) and min. number of neighbours
    - Follow arbitrary shape clustering behaviour

Result:
- K-MEANS vs K-MEANS (weighted): Cluster shifts are seen across different islands in weighted version, especially where there are demographic significance in the region (Java). This supports that external factors will affect clusters formation which will be valuable for business consideration.
- DBSCAN: Regions with high population density are grouped together. Meanwhile, regions with sparse population distribution is considered as outliers. Due to its clustering characteristics, DBSCAN is not suitable in this application where minimum distance is our goal.

We decided that KMEANS weighted is the most suitable model, and then use the elbow method (inertia) and compliment it with Silhoutte score to obtain the optimum clustering. At K=15, inertia starts to stagnant and at the same time, silhoutte score is at the peak. So K=15 is defined the optimum number of clusters.

# Cost Benefit Analysis
---
We assume 2 scenarios in this analysis:
1. Scenario A (baseline): No data science clustering knowledge --> cluster the regions simply by province.
2. Scenario B: Understand about data science clustering --> cluster the regions by kmeans algorithm with weighted factors (population, GDP, minimum wage)

**Steps:**
1. Calculate Distance Metrics (using Haversine distance since latitude, longitude is based on spherical coordinates)
    - A: Sum all distances between cities and their province center (average coordinate of all cities within that province)
    - B: Sum all distances between cities and their centroids
2. Multiply those distances with cost per km to get transportation cost.
3. Calculate infrastructure cost by multiplying number of clusters with total cost of building.
4. Total Cost is the total of transportation and infrastructure cost.

**Calculation:**

Transportation cost per km: \$0.45/km (based on [fuel price of \$1.5/L](https://www.cnbcindonesia.com/news/20230909130251-4-470955/harga-bbm-naik-cek-tarif-di-spbu-pertamina-shell-vivo-bp), and [container truck fuel efficiency of 0.3L/km](https://www.smmt.co.uk/wp-content/uploads/sites/2/Heavy-CV-Fuel-Consumption-Fact-Sheet.pdf))  
Infrastructure cost of a distribution centre: \$450K/unit ([estimated](https://www.smmt.co.uk/wp-content/uploads/sites/2/Heavy-CV-Fuel-Consumption-Fact-Sheet.pdf))

Cost | 38 Province Clustering | K-means Clustering
--- | --- | ---
Total Distance Covered (km) | 56970 | 83281
Number of Distribution Centers (unit) | 38 | 15
Total Transportation Cost (per trip) | \$25,636.50 | \$37,476.45
Total Infrastructure Cost | \$17,100,000.00 | \$6,300,000.00 
Total Cost | \$17,125,636.50 | \$6,337,476.45 

K-means clustering are able to minimize cost by over \$10M.  
<i>Note: This is an overly simplified scenarios which is only part of a complex supply chain operation, i.e. transportation route, mode of transportation, coverage efficiency, etc.</i>

# Conclusion and Recommendation
---

* K-means clustering is more suitable compared to DBSCAN due to its spherical clustering behaviour
* Weighted model successfully considers external factors and affects clustering and the corresponding centroids
* Through clustering algorithm, estimated $10M cost saving (simplified calculation)

**Future Development**
* In this study, clustering is not accounting sea area. Area masking can help to further improve clustering.
* Cost Benefit Analysis is simplified version of the whole supply chain operation. Only cost efficiency is included, can consider to include coverage efficiency.

# Reference 
---
- Indonesia Statistics (BPS): [Indonesia 2021 Census](https://www.bps.go.id/website/materi_ind/materiBrsInd-20210121151046.pdf)
- Wikipedia: [Most densely populated island in the world](https://en.wikipedia.org/wiki/List_of_islands_by_population)
- CNCB (ID): [Fuel Price Sep 2023](https://www.cnbcindonesia.com/news/20230909130251-4-470955/harga-bbm-naik-cek-tarif-di-spbu-pertamina-shell-vivo-bp)
- Society of Motor Manufacturers and Traders (UK): [Heavy Commercial Vehicle Fuel Efficiency](https://www.smmt.co.uk/wp-content/uploads/sites/2/Heavy-CV-Fuel-Consumption-Fact-Sheet.pdf)
- Liputan 6 (ID): [Painting Company Avian Plan to Build 750B Rupiah worth of Distribution Centres](https://www.liputan6.com/saham/read/4906188/avia-avian-siapkan-belanja-modal-rp-750-miliar-untuk-bangun-pabrik?page=2)