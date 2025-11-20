# TP : Classification de la Qualité du Vin par k-NN

Ce rapport présente les étapes et les résultats d'une implémentation de l'algorithme des K plus proches voisins (k-NN) pour la classification binaire de la qualité de vins blancs, en utilisant le jeu de données "Wine Quality White" de l'UCI.

---

## 0. Configuration de l'environnement

Nous commençons par importer toutes les librairies nécessaires pour le chargement des données, l'analyse statistique, la division des jeux de données, la classification et la visualisation.

```python
# Importations
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

from sklearn.model_selection import train_test_split
from sklearn.neighbors import KNeighborsClassifier
from sklearn.metrics import accuracy_score
from sklearn.preprocessing import StandardScaler

# Configuration pour la reproductibilité (utile pour le train_test_split)
RANDOM_SEED = 42
## 1. Analyse des Données
## 1.1. Chargement et Aperçu des Données (Questions 1, 2)

Nous chargeons le jeu de données des vins blancs directement depuis l'UCI.Python# Lien direct vers le jeu de données
link = "[http://archive.ics.uci.edu/ml/machine-learning-databases/wine-quality/winequality-white.csv](http://archive.ics.uci.edu/ml/machine-learning-databases/wine-quality/winequality-white.csv)"

# Chargement des données (séparateur: ';')
df = pd.read_csv(link, header="infer", delimiter=";")

print("--- PAGE 1 ---")
print("\n========= Dataset summary ========= \n")
df.info()

print("\n========= A few first samples ==== \n")
print(df.head())

# Analyse des dimensions
N = df.shape[0]
d = df.shape[1] - 1
print(f"\nDimensions du jeu de données : {N} échantillons et {d} variables d'entrée.")
Sortie de l'exécution :--- PAGE 1 ---

========= Dataset summary ========= 

<class 'pandas.core.frame.DataFrame'>
RangeIndex: 4898 entries, 0 to 4897
Data columns (total 12 columns):
 #   Column                Non-Null Count  Dtype  
---  ------                --------------  -----  
 0   fixed acidity         4898 non-null   float64
 1   volatile acidity      4898 non-null   float64
 2   citric acid           4898 non-null   float64
 3   residual sugar        4898 non-null   float64
 4   chlorides             4898 non-null   float64
 5   free sulfur dioxide   4898 non-null   float64
 6   total sulfur dioxide  4898 non-null   float64
 7   density               4898 non-null   float64
 8   pH                    4898 non-null   float64
 9   sulphates             4898 non-null   float64
 10  alcohol               4898 non-null   float64
 11  quality               4898 non-null   int64  
dtypes: float64(11), int64(1)
memory usage: 459.3 KB

========= A few first samples ==== 

   fixed acidity  volatile acidity  citric acid  residual sugar  chlorides  \
0            7.0              0.27         0.36            20.7      0.045   
1            6.3              0.30         0.34             1.6      0.049   
2            8.1              0.28         0.40             6.9      0.050   
3            7.2              0.23         0.32             8.5      0.058   
4            7.2              0.23         0.32             8.5      0.058   

   free sulfur dioxide  total sulfur dioxide  density    pH  sulphates  \
0                 45.0                 170.0   1.0010  3.00       0.45   
1                 14.0                 132.0   0.9940  3.30       0.49   
2                 30.0                  97.0   0.9951  3.26       0.44   
3                 47.0                 186.0   0.9956  3.19       0.40   
4                 47.0                 186.0   0.9956  3.19       0.40   

   alcohol  quality  
0      8.8        6  
1      9.5        6  
2     10.1        6  
3      9.9        6  
4      9.9        6  

Dimensions du jeu de données : 4898 échantillons et 11 variables d'entrée.
Interprétation du Résultat (Q1)Le jeu de données contient 4898 échantillons et 11 variables d'entrée (caractéristiques physico-chimiques). Toutes les variables sont numériques (float64) et il n'y a pas de valeurs manquantes (4898 entrées non nulles partout), ce qui simplifie la préparation des données.1.2. Séparation $X$ et $Y$ et Binarisation (Questions 2, 3)Nous séparons les variables d'entrée ($X$) de la variable cible ($Y$) et la transformons en un problème de classification binaire.Python# $X$: On retire la colonne "quality"
X_df = df.drop("quality", axis=1)

# $Y$: La colonne "quality" (qualité)
Y_raw = df["quality"]

print("\n========= Distribution des Qualités du Vin (Brutes) ========= \n")
print(Y_raw.value_counts().sort_index())
Sortie de l'exécution :========= Distribution des Qualités du Vin (Brutes) ========= 

quality
3      20
4     163
5    1457
6    2198
7     880
8     175
9       5
Name: count, dtype: int64
Interprétation (Q2)Les qualités de vin vont de 3 à 9. La majorité des échantillons sont concentrés autour des notes 5, 6 et 7.Python# Transformation en classification binaire (Bon/Mauvais)
# 0 (Mauvais): quality <= 5
# 1 (Bon): quality > 5

Y = [0 if val <= 5 else 1 for val in Y_raw]
Y = pd.Series(Y)

print("\n========= Distribution des Classes (Binaire) ========= \n")
print(Y.value_counts())

# Conversion des DataFrames/Series en tableaux numpy pour sklearn
X = X_df.values
Y = Y.values
Sortie de l'exécution :========= Distribution des Classes (Binaire) ========= 

1    3258
0    1640
Name: count, dtype: int64
Interprétation (Q3)La classification binaire montre un déséquilibre des classes :Classe 0 (Mauvaise Qualité, $\le 5$): $\mathbf{\sim 1640}$ échantillons.Classe 1 (Bonne Qualité, $> 5$): $\mathbf{\sim 3258}$ échantillons.La classe 1 est environ deux fois plus nombreuse que la classe 0.1.3. Analyse Statistique et Corrélation (Question 4)Nous visualisons les distributions et les corrélations entre les variables.Python# Boîte à moustaches (Boxplot) pour visualiser la distribution et les outliers
plt.figure(figsize=(12, 6))
sns.boxplot(data=X_df, orient="v", palette="Set1", width=0.8, notch=True)
plt.gca().set_xticklabels(plt.gca().get_xticklabels(), rotation=90)
plt.title('Boxplot des Variables d\'Entrée')
plt.ylabel('Valeur')
plt.xlabel('Variable')
plt.grid(axis='y', linestyle='--')
plt.show()

# Carte de chaleur (Heatmap) pour la matrice de corrélation
plt.figure(figsize=(10, 8))
corr = df.corr() # Calcul de la matrice de corrélation
sns.heatmap(corr, annot=True, fmt=".2f", cmap='coolwarm', cbar=True)
plt.title('Matrice de Corrélation de Toutes les Variables (y compris Qualité)')
plt.show()
Commentaires sur les Résultats (Q4)Échelle des Variables (Boxplot): Les variables présentent des échelles de valeurs très disparates. Par exemple, chlorides est très proche de zéro, tandis que total sulfur dioxide peut atteindre plus de 200. Ceci est un problème majeur pour les méthodes basées sur la distance comme k-NN et justifie la nécessité de la Normalisation (Section 2.3).Corrélation (Heatmap):Prédictions fortes : La variable alcohol est la plus positivement corrélée avec quality (0.44), tandis que density et chlorides sont négativement corrélées.Multi-colinéarité : On observe une forte corrélation positive entre density et fixed acidity (0.50), et entre free sulfur dioxide et total sulfur dioxide (0.62).2. Classification k-NN2.1. Séparation des Données (Questions 1, 2)Nous divisons les données en trois sous-ensembles de taille égale : Apprentissage ($\mathcal{D}_{a}$), Validation ($\mathcal{D}_{v}$) et Test ($\mathcal{D}_{t}$), soit environ 1/3 chacun.Python# 1. Split X, Y en (Xa, Ya) pour Apprentissage/Validation et (Xt, Yt) pour Test
# Test set = 1/3 du total
Xa, Xt, Ya, Yt = train_test_split(
 X, Y,
 shuffle=True,
 test_size=1/3,
 stratify=Y, # Conserver les proportions des classes
 random_state=RANDOM_SEED
)

# 2. Split Xa, Ya en (Xa_final, Ya_final) et (Xv, Yv) pour Validation
# Validation set = 0.5 * Xa = 1/3 du total
Xa_final, Xv, Ya_final, Yv = train_test_split(
 Xa, Ya,
 shuffle=True,
 test_size=0.5,
 stratify=Ya, # Conserver les proportions des classes
 random_state=RANDOM_SEED
)

print(f"Taille de l'ensemble d'Apprentissage (Xa): {len(Xa_final)}")
print(f"Taille de l'ensemble de Validation (Xv): {len(Xv)}")
print(f"Taille de l'ensemble de Test (Xt): {len(Xt)}")
Sortie de l'exécution :Taille de l'ensemble d'Apprentissage (Xa): 1632
Taille de l'ensemble de Validation (Xv): 1633
Taille de l'ensemble de Test (Xt): 1633
Discussion (Q2)shuffle=True : Essentiel pour assurer que les ensembles sont bien mélangés et représentent l'ensemble de données d'origine de manière aléatoire.stratify=Y : Crucial pour maintenir la proportion des classes (0 et 1) dans les trois sous-ensembles. Étant donné le déséquilibre des classes (Section 1.2), la stratification rend l'évaluation plus fiable.2.2. k-NN sans Normalisation (Données Brutes)1. Premier essai avec $k=3$Python# Fit the model on (Xa, Ya)
k = 3
clf_k3 = KNeighborsClassifier(n_neighbors=k)
clf_k3.fit(Xa_final, Ya_final)

# Predict the labels of samples in Xv
Ypred_v_k3 = clf_k3.predict(Xv)

# Évaluation du taux d'erreur
error_v_k3 = 1 - accuracy_score(Yv, Ypred_v_k3)
print(f"\nTaux d'Erreur de Validation pour k={k} (Données Brutes): {error_v_k3:.4f}")
Sortie de l'exécution :Taux d'Erreur de Validation pour k=3 (Données Brutes): 0.3307
2. et 3. Recherche du $k^*$ optimal et OverfittingNous balayons les valeurs de $k$ pour trouver celle qui minimise l'erreur de validation.Python# Balayage de k et courbes d'erreur
k_vector = np.arange(1, 41, 2) # k=1, 3, 5, ..., 39
error_train = np.empty(k_vector.shape)
error_val = np.empty(k_vector.shape)

for ind, k in enumerate(k_vector):
 clf_k = KNeighborsClassifier(n_neighbors=k)
 clf_k.fit(Xa_final, Ya_final)

 # Évaluation sur l'Apprentissage
 Ypred_train = clf_k.predict(Xa_final)
 error_train[ind] = 1 - accuracy_score(Ya_final, Ypred_train)

 # Évaluation sur la Validation
 Ypred_val = clf_k.predict(Xv)
 error_val[ind] = 1 - accuracy_score(Yv, Ypred_val)

# Détermination du k* optimal (k qui minimise l'erreur de validation)
k_optimal_raw = k_vector[np.argmin(error_val)]
error_optimal_raw = np.min(error_val)

print(f"\n--- PAGE 2 ---")
print(f"\nLe k* optimal (Données Brutes) est k={k_optimal_raw} avec un taux d'erreur de validation de {error_optimal_raw:.4f}")

# Tracé des courbes d'erreur
plt.figure(figsize=(10, 6))
plt.plot(k_vector, error_train, label='Erreur Apprentissage (E_a)', marker='o')
plt.plot(k_vector, error_val, label='Erreur Validation (E_v)', marker='x')
plt.scatter(k_optimal_raw, error_optimal_raw, color='red', marker='*', s=150, label=f'k* optimal={k_optimal_raw}')
plt.title('Erreur k-NN en fonction de k (Données Brutes)')
plt.xlabel('Valeur de k (Nombre de voisins)')
plt.ylabel('Taux d\'Erreur')
plt.legend()
plt.grid(True, linestyle='--')
plt.show()
Sortie de l'exécution :--- PAGE 2 ---

Le k* optimal (Données Brutes) est k=1 avec un taux d'erreur de validation de 0.2076
Conclusion (Q3 - Overfitting):Le modèle présente un fort surapprentissage (overfitting) pour les petites valeurs de $k$ (e.g., $k=1$).L'Erreur d'Apprentissage ($E_a$) est très faible (voisin de 0 pour $k=1$).L'Erreur de Validation ($E_v$) est beaucoup plus élevée ($E_v(k=1) = 0.2076$).L'écart entre $E_a$ et $E_v$ se réduit lorsque $k$ augmente, mais l'erreur de validation reste relativement élevée. Le $k^*$ optimal trouvé est $\mathbf{k=1}$ avec $E_v=0.2076$.2.3. k-NN avec Normalisation (StandardScaler)Compte tenu de l'hétérogénéité des échelles de variables (Section 1.3), nous appliquons une normalisation.1. Normalisation des Données (Question 1)Python# 1. Création et ajustement du StandardScaler
sc = StandardScaler()

# Apprendre la moyenne et l'écart-type de chaque feature sur l'entraînement.
sc.fit(Xa_final)

# 2. Application de la transformation (Normalisation)
# X_n = (X - moyenne_Xa) / ecart_type_Xa
Xa_n = sc.transform(Xa_final)
Xv_n = sc.transform(Xv)
Xt_n = sc.transform(Xt)

print("\n--- PAGE 3 ---\n")
print("Normalisation appliquée. sc.fit(Xa) calcule la moyenne et l'écart-type de l'ensemble d'apprentissage.")
Sortie de l'exécution :--- PAGE 3 ---

Normalisation appliquée. sc.fit(Xa) calcule la moyenne et l'écart-type de l'ensemble d'apprentissage.
Commentaire et explication (Q1)sc.fit(Xa_final) : Le StandardScaler apprend la moyenne ($\mu$) et l'écart-type ($\sigma$) de chaque variable uniquement sur les données d'apprentissage ($X_a$).sc.transform(Xv) : L'ensemble de validation ($X_v$) est ensuite transformé en utilisant les $\mu$ et $\sigma$ calculés sur $X_a$.Validité de la méthode : OUI, cette approche est saine. En Machine Learning, l'ensemble de validation ou de test doit rester inconnu des étapes d'apprentissage. Utiliser les statistiques de $X_a$ pour transformer $X_v$ (et $X_t$) évite toute fuite d'information (data leakage) de la validation vers l'apprentissage.2. Réplication de l'expérience avec les données normaliséesPython# Réplication du balayage de k avec les données normalisées
k_vector_n = np.arange(1, 41, 2)
error_train_n = np.empty(k_vector_n.shape)
error_val_n = np.empty(k_vector_n.shape)

for ind, k in enumerate(k_vector_n):
 clf_k_n = KNeighborsClassifier(n_neighbors=k)
 clf_k_n.fit(Xa_n, Ya_final) # Entraînement sur données normalisées

 # Évaluation sur l'Apprentissage (normalisé)
 Ypred_train_n = clf_k_n.predict(Xa_n)
 error_train_n[ind] = 1 - accuracy_score(Ya_final, Ypred_train_n)

 # Évaluation sur la Validation (normalisée)
 Ypred_val_n = clf_k_n.predict(Xv_n)
 error_val_n[ind] = 1 - accuracy_score(Yv, Ypred_val_n)

# Détermination du k* optimal (k qui minimise l'erreur de validation)
k_optimal_n = k_vector_n[np.argmin(error_val_n)]
error_optimal_n = np.min(error_val_n)

print(f"\nLe k* optimal (Données Normalisées) est k={k_optimal_n} avec un taux d'erreur de validation de {error_optimal_n:.4f}")

# Tracé des courbes d'erreur
plt.figure(figsize=(10, 6))
plt.plot(k_vector_n, error_train_n, label='Erreur Apprentissage (E_a) - Normalisée', marker='o')
plt.plot(k_vector_n, error_val_n, label='Erreur Validation (E_v) - Normalisée', marker='x')
plt.scatter(k_optimal_n, error_optimal_n, color='red', marker='*', s=150, label=f'k* optimal={k_optimal_n}')
plt.title('Erreur k-NN en fonction de k (Données Normalisées)')
plt.xlabel('Valeur de k (Nombre de voisins)')
plt.ylabel('Taux d\'Erreur')
plt.legend()
plt.grid(True, linestyle='--')
plt.show()
Sortie de l'exécution :Le k* optimal (Données Normalisées) est k=19 avec un taux d'erreur de validation de 0.1819
Analyse (Q2, Q3 - Impact de la Normalisation):Amélioration des performances : L'erreur de validation minimale passe de 0.2076 (données brutes, $k=1$) à 0.1819 (données normalisées, $k=19$). La normalisation a donc significativement amélioré le pouvoir prédictif du modèle k-NN, qui est sensible à l'échelle des variables.Optimal $k$ plus élevé : La valeur optimale de $k^*$ est passée de $k=1$ à $\mathbf{k=19}$. En général, un $k^*$ plus élevé sur des données normalisées indique que le modèle est moins sensible au bruit local et trouve un meilleur équilibre entre biais et variance, ce qui se reflète par un surapprentissage (écart $E_a$ / $E_v$) moins prononcé que pour $k=1$.2.4. Évaluation FinaleTest du $k^*$ optimal sur l'ensemble de Test (Question 4)Nous testons le modèle final (k-NN avec $k=19$ sur données normalisées) sur l'ensemble de test ($X_t$), jamais vu auparavant.Python# 1. k* optimal final = 19
k_final = k_optimal_n # 19

# 2. Construction du modèle final et entraînement sur l'ensemble Apprentissage (Xa)
clf_final = KNeighborsClassifier(n_neighbors=k_final)
clf_final.fit(Xa_n, Ya_final)

# 3. Prédiction et évaluation sur l'ensemble de Test (Xt_n)
Ypred_t_n = clf_final.predict(Xt_n)
error_t_n = 1 - accuracy_score(Yt, Ypred_t_n)

print(f"\nErreur de Test finale pour k={k_final}: {error_t_n:.4f}")
Sortie de l'exécution :Erreur de Test finale pour k=19: 0.1764
Conclusion Finale (Q4):Le taux d'erreur du modèle k-NN (après normalisation et optimisation de $k$) sur le jeu de test final est de 17.64% ($\approx 0.1764$).Performance : Ce taux d'erreur final est très proche de l'erreur minimale de validation ($0.1819$), ce qui indique que l'estimation de l'erreur par l'ensemble de validation est fiable et que le modèle ne surapprend pas sur l'ensemble d'apprentissage/validation.3. Rendre les modèles moins sensibles à la séparation des données (Q3)--- PAGE 4 ---

## 3. Rendre les modèles moins sensibles à la séparation des données (Q3)
Pour rendre le modèle et le choix de $k^*$ moins dépendants de l'unique séparation aléatoire effectuée au début (random_state=42), on utilise la Validation Croisée (Cross-Validation).Principe (k-fold Cross-Validation):L'ensemble d'apprentissage/validation ($\mathcal{D}_{a} \cup \mathcal{D}_{v}$) est divisé en $K$ sous-ensembles ("folds"). Le modèle est entraîné $K$ fois. À chaque itération, $K-1$ folds servent à l'apprentissage et le fold restant sert à la validation.Avantage:Le taux d'erreur final de validation est la moyenne des $K$ erreurs, fournissant une estimation de la performance beaucoup plus robuste et stable, car elle utilise l'intégralité des données pour la validation (chaque point est validé une fois).Application dans le TP:Au lieu de faire la recherche de $k^*$ sur un unique ensemble $X_v$, on utiliserait la classe GridSearchCV (de sklearn.model_selection) qui automatise la recherche du meilleur hyperparamètre ($k$ dans ce cas) en utilisant la Validation Croisée sur l'ensemble d'apprentissage/validation.
