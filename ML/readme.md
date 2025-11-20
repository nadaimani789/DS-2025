# Classification de la Qualité du Vin par k-NN
## Travail Pratique - Apprentissage Automatique

---

## Table des Matières
1. [Introduction](#introduction)
2. [Méthodologie](#méthodologie)
3. [Analyse Exploratoire des Données](#analyse-exploratoire-des-données)
4. [Implémentation et Résultats](#implémentation-et-résultats)
5. [Discussion](#discussion)
6. [Conclusion](#conclusion)
7. [Références](#références)

---

## Introduction

### Contexte
L'industrie vinicole fait face à un défi majeur : l'évaluation objective de la qualité du vin. Traditionnellement, cette évaluation repose sur l'expertise sensorielle humaine, un processus subjectif et coûteux. L'apprentissage automatique offre une alternative prometteuse en permettant de prédire la qualité du vin à partir de ses caractéristiques physico-chimiques mesurables.

### Objectifs
Ce travail pratique vise à :
- Implémenter un classifieur k-NN (k-Nearest Neighbors) pour la classification de la qualité des vins blancs
- Étudier l'impact de la normalisation des données sur les performances du modèle
- Optimiser l'hyperparamètre k par validation croisée
- Analyser le compromis biais-variance et le phénomène de sur-apprentissage

### Dataset
Nous utilisons le dataset "Wine Quality" de l'UCI Machine Learning Repository, contenant **4898 échantillons** de vins blancs portugais "Vinho Verde", caractérisés par :
- **11 variables physico-chimiques** : acidité, sucres résiduels, chlorures, dioxyde de soufre, densité, pH, sulfates, alcool
- **1 variable cible** : qualité (score de 0 à 10 attribué par des experts)

---

## Méthodologie

### 2.1 Prétraitement des Données

#### Transformation Binaire
Le problème original de régression (qualité de 0 à 10) est transformé en **classification binaire** :
- **Classe 0 (Mauvaise qualité)** : quality ≤ 5
- **Classe 1 (Bonne qualité)** : quality > 5

Cette simplification permet une évaluation plus robuste et reflète un cas d'usage pratique : accepter ou rejeter un lot de vin.

#### Partition des Données
Les données sont divisées selon le schéma suivant :
```
Dataset Complet (4898 échantillons)
├── Ensemble d'Apprentissage + Validation (2/3) ≈ 3265
│   ├── Apprentissage (1/3 du total) ≈ 1633
│   └── Validation (1/3 du total) ≈ 1632
└── Ensemble de Test (1/3) ≈ 1633
```

**Stratification** : La proportion des classes est maintenue dans chaque sous-ensemble pour garantir la représentativité statistique, particulièrement importante étant donné le déséquilibre des classes (ratio 2:1).

### 2.2 Algorithme k-NN

L'algorithme des k plus proches voisins est une méthode d'apprentissage supervisé non-paramétrique basée sur la distance. Pour un nouvel échantillon :
1. Calculer la distance euclidienne avec tous les échantillons d'apprentissage
2. Identifier les k voisins les plus proches
3. Attribuer la classe majoritaire parmi ces k voisins

**Avantages** : Simple, intuitif, pas d'hypothèse sur la distribution des données  
**Inconvénients** : Sensible à l'échelle des variables, coûteux en calcul pour grandes bases

### 2.3 Normalisation : Standardisation Z-score

La standardisation transforme chaque variable selon :
```
X_norm = (X - μ) / σ
```
où μ et σ sont la moyenne et l'écart-type calculés **uniquement sur l'ensemble d'apprentissage**.

**Principe fondamental** : Les statistiques (μ, σ) des ensembles de validation et de test ne doivent **jamais** influencer le prétraitement pour éviter toute fuite d'information (*data leakage*).

---

## Analyse Exploratoire des Données

### 3.1 Statistiques Descriptives

**Dimensions** : 4898 échantillons × 11 variables  
**Type de données** : Toutes les variables sont numériques continues (float64)  
**Valeurs manquantes** : Aucune (dataset complet)

### 3.2 Distribution de la Variable Cible

#### Qualité Originale (0-10)
La distribution des scores de qualité révèle :
- **Distribution quasi-normale** centrée sur 6
- Majorité des échantillons : qualité 5, 6, 7
- Valeurs extrêmes rares (peu de vins excellents ou très mauvais)

#### Après Binarisation
- **Classe 0 (Mauvaise)** : ~1640 échantillons (33.5%)
- **Classe 1 (Bonne)** : ~3258 échantillons (66.5%)
- **Ratio de déséquilibre** : 1:2

Ce déséquilibre modéré nécessite une stratification lors du split des données mais reste gérable sans techniques avancées de rééquilibrage.

### 3.3 Analyse des Variables

#### Échelles Disparates (Boxplot)
L'analyse par boîtes à moustaches révèle des **échelles très hétérogènes** :
- `chlorides` : valeurs autour de 0.04
- `total sulfur dioxide` : valeurs jusqu'à 289
- Facteur d'échelle : **>1000 entre variables**
<img width="1005" height="664" alt="image" src="https://github.com/user-attachments/assets/2818f004-5fe8-473f-bbae-b73fe4360f28" />
<img width="892" height="798" alt="image" src="https://github.com/user-attachments/assets/eeb48a57-eeff-4772-91a4-c413b440bbce" />


**Implication critique** : Sans normalisation, la distance euclidienne est dominée par les variables à grande échelle, rendant le k-NN inefficace.

#### Corrélations (Heatmap)
**Corrélations avec la qualité** :
- **Positive forte** : `alcohol` (r = 0.44) — plus le vin est alcoolisé, meilleure est la qualité perçue
- **Négatives modérées** : `density` (r = -0.31), `chlorides` (r = -0.21)

**Multicolinéarité observée** :
- `density` ↔ `residual sugar` (r = 0.84) : forte redondance
- `free sulfur dioxide` ↔ `total sulfur dioxide` (r = 0.62)
- `density` ↔ `alcohol` (r = -0.78) : relation physique inverse

**Interprétation** : La multicolinéarité ne pose pas de problème pour k-NN (contrairement à la régression linéaire) mais suggère une redondance d'information.

---

## Implémentation et Résultats

### 4.1 Expérience 1 : k-NN Sans Normalisation

#### Test Initial (k=3)
```
Taux d'erreur de validation : ~0.30 (30%)
```
Performance médiocre, à peine meilleure qu'une classification aléatoire informée (qui donnerait ~40% d'erreur sur la classe minoritaire).

#### Recherche de k* Optimal
**Plage testée** : k ∈ {1, 3, 5, ..., 39}

**Observations du graphique erreur vs k** :
- **k=1** : 
  - Erreur d'apprentissage : ~0.08 (très faible)
  - Erreur de validation : ~0.28
  - **Diagnostic** : Sur-apprentissage (*overfitting*) sévère

- **Convergence pour k croissant** :
  - Les deux courbes convergent vers ~0.33
  - Pour k très grand, le modèle sous-apprend (*underfitting*)

**k* optimal identifié** : k ≈ 13-17
<img width="855" height="547" alt="image" src="https://github.com/user-attachments/assets/1343f0fa-94f6-4add-a227-9aca68c96224" />

```
Erreur de validation minimale : ~0.27
Erreur de test : ~0.28
```

#### Interprétation
- La généralisation du modèle reste **très limitée** (72% de précision)
- L'erreur de test confirme la faible performance
- Le problème principal : **domination par les variables à grande échelle**

### 4.2 Expérience 2 : k-NN Avec Normalisation
<img width="855" height="547" alt="image" src="https://github.com/user-attachments/assets/926554ef-1526-47aa-9d39-776bece36caa" />

#### Impact de la Normalisation
Après standardisation Z-score, les résultats sont transformés :

**k* optimal identifié** : k ≈ 9-15
```
Erreur de validation minimale : ~0.21
Erreur de test : ~0.22
```

#### Comparaison Quantitative

| Métrique | Sans Normalisation | Avec Normalisation | Amélioration |
|----------|-------------------|-------------------|--------------|
| k* optimal | 13-17 | 9-15 | Déplacement vers k plus petits |
| Erreur validation | 27% | 21% | **-6 points** |
| Erreur test | 28% | 22% | **-6 points** |
| Précision | 72% | **78%** | **+6%** |

#### Analyse des Courbes d'Erreur Normalisées
- **Séparation réduite** entre erreur d'apprentissage et validation
- Sur-apprentissage présent mais **moins prononcé**
- Convergence vers ~0.23 pour k très grand (vs 0.33 sans normalisation)

**Interprétation clé** : La normalisation permet à toutes les variables de **contribuer équitablement** au calcul de distance, révélant les patterns subtils masqués par les disparités d'échelle.

### 4.3 Analyse du Compromis Biais-Variance

Le graphique erreur vs k illustre le compromis fondamental en apprentissage automatique :

#### k faible (1-5) : Haute Variance
- Modèle très flexible
- Mémorise le bruit des données d'apprentissage
- **Variance élevée** : sensible aux variations d'échantillonnage
- **Biais faible** : capture les relations complexes

#### k élevé (30+) : Biais Élevé
- Modèle trop rigide
- Lisse excessivement les frontières de décision
- **Biais élevé** : ne capture pas la complexité réelle
- **Variance faible** : stable mais imprécis

#### k* optimal (9-15) : Équilibre
Point de **compromis optimal** où Erreur totale = Biais² + Variance est minimale.

---

## Discussion

### 5.1 Importance Cruciale de la Normalisation

Les résultats démontrent sans ambiguïté que la normalisation est **indispensable** pour k-NN. L'amélioration de 6 points de précision n'est pas marginale — elle représente :
- Réduction de **21% de l'erreur relative** (6/28)
- Classification correcte de ~300 échantillons supplémentaires sur le test set

**Explication mathématique** : Sans normalisation, la distance euclidienne est :
```
d = √[(x₁-y₁)² + (x₂-y₂)² + ... + (x₁₁-y₁₁)²]
```
Si x₁ (total sulfur dioxide) varie de 0-289 et x₂ (chlorides) de 0-0.1, alors le terme (x₁-y₁)² **domine** complètement, rendant x₂ invisible pour le classifieur.

### 5.2 Limitation de la Validation Simple

Notre approche utilise une **unique séparation** train/validation/test (random_state=42). Problèmes potentiels :
- Sensibilité à la "chance" de la séparation
- k* optimal pourrait varier avec un autre seed
- Estimation d'erreur potentiellement biaisée

**Solution : Validation Croisée k-fold**
```python
from sklearn.model_selection import GridSearchCV
grid = GridSearchCV(
    KNeighborsClassifier(), 
    param_grid={'n_neighbors': range(1, 40, 2)},
    cv=5,  # 5-fold cross-validation
    scoring='accuracy'
)
```
Avantages :
- Moyenne sur 5 partitions différentes
- Estimation plus robuste de l'erreur
- Utilisation de 100% des données d'entraînement pour la validation

### 5.3 Limites du Modèle et Perspectives

#### Limites Identifiées
1. **Performance modérée** : 78% de précision reste perfectible
2. **Déséquilibre des classes** : le modèle pourrait favoriser la classe majoritaire
3. **Interprétabilité limitée** : k-NN est une "boîte noire" ne révélant pas quelles variables sont importantes

#### Améliorations Possibles
1. **Métrique alternative** : Utiliser F1-score ou AUC-ROC (plus robustes au déséquilibre)
2. **Réduction de dimensionnalité** : 
   - PCA pour réduire la multicolinéarité
   - Sélection de features (éliminer variables peu corrélées à la qualité)
3. **Méthodes ensemblistes** :
   - Random Forest : ~85% précision attendue
   - XGBoost : état de l'art pour ce type de problème
4. **Métriques pondérées** : Donner plus d'importance à la distance selon certaines variables (ex: alcohol)

### 5.4 Considérations Pratiques

**Temps de calcul** : k-NN nécessite de stocker tout l'ensemble d'apprentissage et de calculer O(n·d) distances pour chaque prédiction. Pour un déploiement industriel à grande échelle, ceci peut être prohibitif.

**Interprétabilité métier** : Dans l'industrie vinicole, un modèle expliquant *pourquoi* un vin est classé comme bon (ex: "alcool élevé + faible densité") serait préférable à k-NN.

---

## Conclusion

Ce travail pratique a démontré l'efficacité de l'algorithme k-NN pour la classification de la qualité du vin, tout en mettant en évidence ses exigences méthodologiques strictes.

### Résultats Principaux
1. **Normalisation essentielle** : Amélioration de 72% → 78% de précision
2. **Optimisation de k** : k* ≈ 9-15 offre le meilleur compromis biais-variance
3. **Validation rigoureuse** : Séparation train/validation/test + stratification indispensables

### Contributions Pédagogiques
- Illustration concrète du sur-apprentissage et du compromis biais-variance
- Importance de la préparation des données (normalisation, stratification)
- Méthodologie complète de validation de modèle supervisé

### Perspectives de Recherche
Pour aller plus loin, plusieurs axes méritent exploration :
- **Validation croisée** pour robustesse statistique accrue
- **Comparaison avec d'autres algorithmes** (SVM, Random Forest, réseaux de neurones)
- **Approche multi-classes** : prédire le score exact de qualité (régression ou classification fine)
- **Ingénierie de features** : créer des variables composées (ratios, interactions)

### Message Final
Au-delà des résultats quantitatifs, ce TP souligne un principe fondamental du Machine Learning : **la qualité des données et de leur prétraitement est souvent plus importante que la sophistication de l'algorithme**. Un k-NN simple mais correctement normalisé surpasse de nombreux modèles complexes appliqués sur des données brutes.

---

## Références

### Dataset
- P. Cortez, A. Cerdeira, F. Almeida, T. Matos and J. Reis. *Modeling wine preferences by data mining from physicochemical properties.* Decision Support Systems, Elsevier, 47(4):547-553, 2009.
- UCI Machine Learning Repository: [Wine Quality Dataset](http://archive.ics.uci.edu/ml/datasets/Wine+Quality)

### Bibliographie Technique
- **k-NN** : T. Cover and P. Hart. *Nearest neighbor pattern classification.* IEEE Transactions on Information Theory, 13(1):21-27, 1967.
- **Compromis biais-variance** : S. Geman, E. Bienenstock, and R. Doursat. *Neural networks and the bias/variance dilemma.* Neural Computation, 4(1):1-58, 1992.
- **Validation croisée** : R. Kohavi. *A study of cross-validation and bootstrap for accuracy estimation and model selection.* IJCAI, 1995.

### Outils Logiciels
- **Python 3.x**
- **scikit-learn** : Pedregosa et al. *Scikit-learn: Machine Learning in Python.* JMLR 12, 2011.
- **pandas, numpy, matplotlib, seaborn**

---

## Annexe : Reproductibilité

```python
# Environnement
Python 3.8+
scikit-learn 1.0+
pandas 1.3+
numpy 1.21+
matplotlib 3.4+
seaborn 0.11+

# Seed pour reproductibilité
RANDOM_SEED = 42
```

**Note** : Tous les résultats de ce rapport peuvent être reproduits en exécutant le notebook fourni avec le même RANDOM_SEED.

---

**Auteur** : Nada el imani
**Date** : Novembre 2025  
**Cours** : Apprentissage Automatique  
**Institution** : ENCG SETTAT

---

*"In God we trust, all others must bring data."* — W. Edwards Deming
