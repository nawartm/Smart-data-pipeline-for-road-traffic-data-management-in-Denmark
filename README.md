# 🚦 Smart Data Pipeline for Road Traffic Management in Denmark

> 🎯 *Prédire la durée des trajets routiers en temps réel et proposer des itinéraires alternatifs pour éviter les congestions — en utilisant Spark, Delta Lake, et les Forêts Aléatoires.*

Ce notebook Jupyter implémente un **pipeline de données intelligent** pour la gestion du trafic routier au Danemark, basé sur le jeu de données **CityPulse**. Il combine :
- 🔍 **Nettoyage et préparation des données** avec **PySpark**
- 🧠 **Modélisation prédictive** avec **Random Forest Regressor**
- 💾 **Stockage optimisé** avec **Delta Lake**
- 📊 **Visualisation et prise de décision** (alertes de congestion, suggestions d’itinéraires alternatifs)
- 🌍 **Calcul géospatial** avec `geopy` pour proposer des alternatives réalistes

---

## 🌍 Contexte & Objectif

Le trafic routier au Danemark est surveillé via des capteurs et rapports (CityPulse). L’objectif de ce projet est de :

1. **Prédire la durée des trajets** entre deux points en fonction de :
   - La vitesse moyenne (NDT_IN_KMH)
   - La distance (DISTANCE_IN_METERS)
   - Les coordonnées géographiques (latitude/longitude des points de départ et d’arrivée)

2. **Détecter les trajets à risque de congestion** (durée prédite > seuil)

3. **Proposer des itinéraires alternatifs** non congestionnés, en s’assurant qu’ils sont :
   - Suffisamment longs (> 2.5 km)
   - Différents des trajets congestionnés
   - Pas des boucles (point de départ ≠ point d’arrivée)

---

## 👥 Pour qui est ce projet ?

| Public | Ce qu’il y trouvera |
|--------|----------------------|
| 🚗 **Gestionnaires de trafic / Villes intelligentes** | Un prototype opérationnel pour anticiper les congestions et proposer des déviations. |
| 👩‍🎓 **Étudiants en Data Engineering / ML** | Un exemple complet de pipeline Spark + ML + Delta Lake + géospatial. |
| 👨‍💻 **Data Engineers / Data Scientists** | Une implémentation propre, modulaire, avec bonnes pratiques (cache, validation croisée, stockage Delta). |
| 👔 **Décideurs / Curieux** | Une démonstration concrète de comment les données peuvent améliorer la mobilité urbaine. |

---

## ⚙️ Étapes Techniques Réalisées

### 1. 📥 Chargement & Nettoyage des Données
- Lecture du CSV `CityPulseTrafic.csv` avec inférence de schéma
- Suppression des valeurs nulles (`na.drop()`)
- Conversion des coordonnées GPS en `double`

### 2. 🧱 Préparation des Features pour le ML
- Assemblage des features avec `VectorAssembler` :
  - `NDT_IN_KMH`, `DISTANCE_IN_METERS`, `POINT_1_LAT/LNG`, `POINT_2_LAT/LNG`
- Normalisation avec `MinMaxScaler` → mise à l’échelle [0,1]

### 3. 💾 Stockage dans Delta Lake
- Écriture des données nettoyées dans `/mnt/delta/BDTraffic_cleaned`
- Avantages : ACID, historique, performance, compatibilité Spark

### 4. 🧠 Modélisation avec Random Forest
- Division train/test (80/20)
- Validation croisée sur 2 folds avec grille d’hyperparamètres :
  - `numTrees`: [50, 100]
  - `maxDepth`: [10, 20]
  - `maxBins`: [64, 128]
- Métrique d’évaluation : **RMSE** (Root Mean Squared Error)

> ✅ **RMSE final : 17.73 secondes** → très bonne précision pour des prédictions de durée de trajet.

### 5. 📈 Visualisation & Analyse
- Graphique comparatif : durées réelles vs prédites
- Statistiques descriptives des prédictions
- Calcul du **95e percentile** → utilisé pour fixer le seuil d’alerte

### 6. 🚨 Détection de Congestion
- Seuil défini à **137 secondes** (~95e percentile)
- Ajout d’une colonne `alert` = `True` si prédiction > seuil
- Affichage des trajets à risque :
  - Ex: `Landevejen → Landevejen` (234.78 sec), `Randersvej → Vejlby Centervej` (200.42 sec)

### 7. 🌐 Suggestions d’Itinéraires Alternatifs
- Utilisation de `geopy.distance.geodesic` pour calculer les distances réelles entre points
- Filtrage des itinéraires :
  - Non congestionnés
  - Distance > 2.5 km
  - Pas de boucles (rue de départ ≠ rue d’arrivée)
  - Pas de doublons
- Exemples proposés :
  - `Hovedvejen → Møllebakken` (3.33 km)
  - `Århusvej → Oddervej` (12.84 km)
  - `Nordjyske Motorvej → 15` (2.65 km)

---

## 🧩 Technologies & Bibliothèques

```python
from pyspark.sql import SparkSession
from pyspark.ml.feature import VectorAssembler, MinMaxScaler
from pyspark.ml.regression import RandomForestRegressor
from pyspark.ml.evaluation import RegressionEvaluator
from pyspark.ml.tuning import ParamGridBuilder, CrossValidator
from delta import *
from geopy.distance import geodesic
import matplotlib.pyplot as plt
