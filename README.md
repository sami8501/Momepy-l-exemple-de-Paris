# Momepy - Mesure des formes des bâtiments

Momepy est une bibliothèque Python puissante pour l'analyse quantitative de la forme urbaine. Ce README se concentre sur l'utilisation de Momepy pour mesurer et analyser les formes des bâtiments, en utilisant comme exemple une partie du centre de Paris.

## Installation

```bash
pip install momepy geopandas osmnx matplotlib libpysal
```

## Fonctionnalités principales

### 1. Importation des données et préparation

```python
import momepy
import geopandas as gpd
import osmnx as ox
import matplotlib.pyplot as plt

point = (48.856737, 2.341585)
dist = 1000
gdf = ox.geometries.geometries_from_point(point, dist=dist, tags={'building':True})
buildings = ox.projection.project_gdf(gdf)
```

### 2. Création de la tessellation

```python
buildings['uID'] = momepy.unique_id(buildings)
limit = momepy.buffered_limit(buildings)
tessellation = momepy.Tessellation(buildings, unique_id='uID', limit=limit).tessellation
```

### 3. Analyse des formes des bâtiments

#### Compacité circulaire

```python
buildings['circular_com'] = momepy.CircularCompactness(buildings).series
```

#### Aire

```python
buildings['area'] = momepy.Area(buildings).series
```

#### Élongation

```python
buildings['elongation'] = momepy.Elongation(buildings).series
```

#### Squareness

```python
buildings['squareness'] = momepy.Squareness(buildings).series
```

### 4. Analyse de la tessellation

```python
tessellation['cwa'] = momepy.CompactnessWeightedAxis(tessellation).series
```

### 5. Analyse du réseau routier

```python
streets = ox.graph_to_gdfs(ox.graph_from_place('Paris, France', network_type='walk'), nodes=False, edges=True)
streets = momepy.remove_false_nodes(streets)
streets["linearity"] = momepy.Linearity(streets).series
```

### 6. Analyse spatiale

```python
buildings["shared_walls"] = momepy.SharedWallsRatio(buildings).series
```

### 7. Analyse de voisinage

```python
import libpysal
queen_1 = libpysal.weights.contiguity.Queen.from_dataframe(tessellation, ids="uID")
buildings["neighbor_distance"] = momepy.NeighborDistance(buildings, queen_1, "uID").series
```

### 8. Analyse de connectivité

```python
graph = momepy.gdf_to_nx(streets)
graph = momepy.node_degree(graph)
graph = momepy.closeness_centrality(graph, radius=400, distance="mm_len")
graph = momepy.meshedness(graph, radius=400, distance="mm_len")
nodes, streets = momepy.nx_to_gdf(graph)
```

### 9. Clustering des types urbains

```python
from clustergram import Clustergram

# Préparation des données pour le clustering
merged = tessellation.merge(buildings.drop(columns=['geometry']), on='uID')
merged = merged.merge(streets.drop(columns='geometry'), on='nID', how='left')
merged = merged.merge(nodes.drop(columns='geometry'), on='nodeID', how='left')

# Clustering
cgram = Clustergram(range(1, 12), n_init=10, random_state=42)
cgram.fit(standardized_data)
merged["cluster"] = cgram.labels[8].values
```

## Visualisation

Le notebook fournit de nombreux exemples de visualisation des résultats, notamment :

```python
f, ax = plt.subplots(figsize=(30, 30))
buildings.plot(ax=ax, column='circular_com', legend=True, cmap='viridis')
ax.set_axis_off()
plt.show()
```

## Exemples d'utilisation

Le notebook illustre l'analyse détaillée des formes de bâtiments pour une partie du centre de Paris, démontrant :
- L'extraction des données de bâtiments d'OpenStreetMap
- Le calcul de diverses mesures de forme (compacité, élongation, etc.)
- L'analyse de la tessellation urbaine
- L'étude de la connectivité du réseau routier
- Le clustering des types urbains basé sur ces mesures

## Licence

Momepy est distribué sous licence BSD 3-Clause.

## Contact

Pour plus d'informations, visitez la [documentation officielle de Momepy](http://docs.momepy.org/).
