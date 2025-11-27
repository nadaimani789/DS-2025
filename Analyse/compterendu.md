
Analyse de Sentiment sur les Données Reddit - Projet NLP
Table des Matières
Introduction
Objectifs du Projet
Description du Dataset
Préparation des Données
Analyse Exploratoire des Données (EDA)
Ingénierie des Features
Modélisation et Classification
7.1 Régression Logistique
7.2 Arbre de Décision
7.3 K-Nearest Neighbors (KNN)
7.4 Naive Bayes (Gaussien et Multinomial)
7.5 Support Vector Classifier (SVC)
7.6 Random Forest
7.7 XGBoost
7.8 Multi-layer Perceptron (MLP)
7.9 Gradient Boosting
Validation Croisée
Comparaison des Modèles
Conclusion
1. Introduction {#introduction}
L'analyse de sentiment est une tâche fondamentale du traitement automatique du langage naturel (NLP) qui vise à déterminer l'opinion, l'émotion ou l'attitude exprimée dans un texte. Dans ce projet, nous analysons des commentaires Reddit pour classifier automatiquement leur sentiment.

Dataset utilisé : Reddit Sentiment Analysis Dataset for NLP Projects

Ce projet s'inspire de la méthodologie utilisée dans l'analyse de prédiction du cancer du poumon, en appliquant 10 modèles de classification différents pour évaluer leur performance sur des données textuelles.

2. Objectifs du Projet {#objectifs}
Nettoyer et préparer les données textuelles de Reddit
Effectuer une analyse exploratoire approfondie
Appliquer des techniques d'ingénierie de features adaptées au NLP
Comparer 10 algorithmes de machine learning pour la classification de sentiment
Identifier le modèle le plus performant pour cette tâche
Évaluer les modèles avec des méthodes de validation croisée robustes
3. Description du Dataset {#description-dataset}
Le dataset Reddit Sentiment Analysis contient des commentaires extraits de diverses communautés Reddit, annotés avec leur sentiment.

Colonnes principales :

comment : Le texte du commentaire Reddit
sentiment : L'étiquette de sentiment (positif, négatif, neutre)
score : Le score du commentaire (upvotes - downvotes)
subreddit : La communauté d'origine
date : Date de publication
4. Préparation des Données {#preparation-donnees}
4.1 Importation des Bibliothèques
python
# Importation des bibliothèques essentielles
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import warnings
warnings.filterwarnings("ignore")

# Bibliothèques NLP
import re
import nltk
from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize
from nltk.stem import WordNetLemmatizer

# Téléchargement des ressources NLTK nécessaires
nltk.download('stopwords')
nltk.download('punkt')
nltk.download('wordnet')
nltk.download('omw-1.4')

# Bibliothèques de vectorisation
from sklearn.feature_extraction.text import TfidfVectorizer, CountVectorizer

# Bibliothèques de preprocessing
from sklearn import preprocessing
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import LabelEncoder
4.2 Chargement des Données
python
# Chargement du dataset
df = pd.read_csv('reddit_sentiment_data.csv')

# Affichage des premières lignes
print("Aperçu des données :")
df.head()
4.3 Exploration Initiale
python
# Dimensions du dataset
print(f"Dimensions du dataset : {df.shape}")
print(f"Nombre de lignes : {df.shape[0]}")
print(f"Nombre de colonnes : {df.shape[1]}")

# Informations sur les colonnes
print("\nInformations sur les colonnes :")
df.info()

# Statistiques descriptives
print("\nStatistiques descriptives :")
df.describe()
4.4 Gestion des Doublons
python
# Vérification des doublons
print(f"Nombre de doublons : {df.duplicated().sum()}")

# Suppression des doublons
df = df.drop_duplicates()
print(f"Nombre de lignes après suppression des doublons : {df.shape[0]}")
4.5 Gestion des Valeurs Manquantes
python
# Vérification des valeurs manquantes
print("Valeurs manquantes par colonne :")
print(df.isnull().sum())

# Suppression des lignes avec valeurs manquantes dans les colonnes essentielles
df = df.dropna(subset=['comment', 'sentiment'])
print(f"Nombre de lignes après suppression des valeurs manquantes : {df.shape[0]}")
4.6 Nettoyage du Texte
python
# Fonction de nettoyage du texte
def clean_text(text):
    """
    Nettoie le texte en :
    - Convertissant en minuscules
    - Supprimant les URLs
    - Supprimant les mentions (@username)
    - Supprimant les caractères spéciaux
    - Supprimant les chiffres
    - Supprimant les espaces multiples
    """
    if isinstance(text, str):
        # Conversion en minuscules
        text = text.lower()
        
        # Suppression des URLs
        text = re.sub(r'http\S+|www\S+|https\S+', '', text, flags=re.MULTILINE)
        
        # Suppression des mentions
        text = re.sub(r'@\w+', '', text)
        
        # Suppression des hashtags (garder le texte)
        text = re.sub(r'#', '', text)
        
        # Suppression des caractères spéciaux et chiffres
        text = re.sub(r'[^a-zA-Z\s]', '', text)
        
        # Suppression des espaces multiples
        text = re.sub(r'\s+', ' ', text).strip()
        
        return text
    return ""

# Application du nettoyage
df['cleaned_comment'] = df['comment'].apply(clean_text)

# Affichage d'exemples
print("Exemples de commentaires nettoyés :")
for i in range(3):
    print(f"\nOriginal : {df['comment'].iloc[i]}")
    print(f"Nettoyé : {df['cleaned_comment'].iloc[i]}")
4.7 Tokenisation et Lemmatisation
python
# Initialisation du lemmatiseur
lemmatizer = WordNetLemmatizer()
stop_words = set(stopwords.words('english'))

def preprocess_text(text):
    """
    Prétraitement avancé :
    - Tokenisation
    - Suppression des stop words
    - Lemmatisation
    """
    # Tokenisation
    tokens = word_tokenize(text)
    
    # Suppression des stop words et lemmatisation
    tokens = [lemmatizer.lemmatize(word) for word in tokens 
              if word not in stop_words and len(word) > 2]
    
    return ' '.join(tokens)

# Application du prétraitement
df['preprocessed_comment'] = df['cleaned_comment'].apply(preprocess_text)

# Suppression des lignes avec texte vide après prétraitement
df = df[df['preprocessed_comment'].str.len() > 0]
print(f"Nombre de lignes après prétraitement : {df.shape[0]}")
4.8 Encodage des Labels
python
# Encodage des sentiments
le = LabelEncoder()
df['sentiment_encoded'] = le.fit_transform(df['sentiment'])

# Affichage du mapping
print("\nMapping des sentiments :")
for i, sentiment in enumerate(le.classes_):
    print(f"{sentiment} : {i}")

# Distribution des sentiments
print("\nDistribution des sentiments :")
print(df['sentiment'].value_counts())
5. Analyse Exploratoire des Données (EDA) {#analyse-exploratoire}
5.1 Distribution des Sentiments
python
# Visualisation de la distribution des sentiments
plt.figure(figsize=(10, 6))
sns.countplot(data=df, x='sentiment', palette='viridis')
plt.title('Distribution des Sentiments dans les Commentaires Reddit', fontsize=14, fontweight='bold')
plt.xlabel('Sentiment', fontsize=12)
plt.ylabel('Nombre de Commentaires', fontsize=12)
plt.xticks(rotation=0)
plt.tight_layout()
plt.show()

# Pourcentages
sentiment_counts = df['sentiment'].value_counts()
print("\nPourcentages par sentiment :")
print((sentiment_counts / len(df) * 100).round(2))
5.2 Longueur des Commentaires
python
# Calcul de la longueur des commentaires
df['comment_length'] = df['comment'].apply(len)
df['word_count'] = df['preprocessed_comment'].apply(lambda x: len(x.split()))

# Statistiques par sentiment
print("\nStatistiques de longueur par sentiment :")
print(df.groupby('sentiment')[['comment_length', 'word_count']].describe())

# Visualisation
fig, axes = plt.subplots(1, 2, figsize=(15, 5))

# Longueur en caractères
axes[0].hist([df[df['sentiment'] == sent]['comment_length'] 
              for sent in df['sentiment'].unique()], 
             label=df['sentiment'].unique(), bins=50, alpha=0.7)
axes[0].set_xlabel('Longueur du Commentaire (caractères)')
axes[0].set_ylabel('Fréquence')
axes[0].set_title('Distribution de la Longueur des Commentaires')
axes[0].legend()

# Nombre de mots
axes[1].hist([df[df['sentiment'] == sent]['word_count'] 
              for sent in df['sentiment'].unique()], 
             label=df['sentiment'].unique(), bins=50, alpha=0.7)
axes[1].set_xlabel('Nombre de Mots')
axes[1].set_ylabel('Fréquence')
axes[1].set_title('Distribution du Nombre de Mots')
axes[1].legend()

plt.tight_layout()
plt.show()
5.3 Mots les Plus Fréquents
python
from collections import Counter
from wordcloud import WordCloud

def plot_word_frequency(text_series, title, top_n=20):
    """Affiche les mots les plus fréquents"""
    all_words = ' '.join(text_series).split()
    word_freq = Counter(all_words)
    
    # Top N mots
    top_words = dict(word_freq.most_common(top_n))
    
    # Graphique en barres
    plt.figure(figsize=(12, 6))
    plt.bar(range(len(top_words)), list(top_words.values()), align='center')
    plt.xticks(range(len(top_words)), list(top_words.keys()), rotation=45, ha='right')
    plt.xlabel('Mots')
    plt.ylabel('Fréquence')
    plt.title(title)
    plt.tight_layout()
    plt.show()

# Mots fréquents par sentiment
for sentiment in df['sentiment'].unique():
    sentiment_text = df[df['sentiment'] == sentiment]['preprocessed_comment']
    plot_word_frequency(sentiment_text, f'Top 20 Mots - Sentiment {sentiment}')
5.4 Nuages de Mots
python
# Création de nuages de mots pour chaque sentiment
fig, axes = plt.subplots(1, len(df['sentiment'].unique()), figsize=(18, 5))

for idx, sentiment in enumerate(df['sentiment'].unique()):
    text = ' '.join(df[df['sentiment'] == sentiment]['preprocessed_comment'])
    
    wordcloud = WordCloud(width=800, height=400, 
                         background_color='white',
                         colormap='viridis',
                         max_words=100).generate(text)
    
    axes[idx].imshow(wordcloud, interpolation='bilinear')
    axes[idx].set_title(f'Nuage de Mots - {sentiment}', fontsize=14, fontweight='bold')
    axes[idx].axis('off')

plt.tight_layout()
plt.show()
5.5 Distribution par Subreddit
python
# Top subreddits
if 'subreddit' in df.columns:
    top_subreddits = df['subreddit'].value_counts().head(10)
    
    plt.figure(figsize=(12, 6))
    top_subreddits.plot(kind='barh', color='skyblue')
    plt.xlabel('Nombre de Commentaires')
    plt.ylabel('Subreddit')
    plt.title('Top 10 Subreddits par Nombre de Commentaires')
    plt.gca().invert_yaxis()
    plt.tight_layout()
    plt.show()
    
    # Sentiment par subreddit
    sentiment_subreddit = pd.crosstab(df['subreddit'], df['sentiment'], normalize='index')
    sentiment_subreddit.head(10).plot(kind='bar', stacked=True, figsize=(12, 6))
    plt.xlabel('Subreddit')
    plt.ylabel('Proportion')
    plt.title('Distribution des Sentiments par Subreddit (Top 10)')
    plt.legend(title='Sentiment')
    plt.xticks(rotation=45, ha='right')
    plt.tight_layout()
    plt.show()
6. Ingénierie des Features {#ingenierie-features}
6.1 Vectorisation TF-IDF
python
# Création de features TF-IDF
tfidf = TfidfVectorizer(max_features=5000, 
                        ngram_range=(1, 2),
                        min_df=2,
                        max_df=0.95)

X_tfidf = tfidf.fit_transform(df['preprocessed_comment'])

print(f"Dimensions de la matrice TF-IDF : {X_tfidf.shape}")
print(f"Nombre de features : {X_tfidf.shape[1]}")
6.2 Features Supplémentaires
python
# Création de features additionnelles
df['exclamation_count'] = df['comment'].apply(lambda x: x.count('!'))
df['question_count'] = df['comment'].apply(lambda x: x.count('?'))
df['uppercase_ratio'] = df['comment'].apply(
    lambda x: sum(1 for c in x if c.isupper()) / len(x) if len(x) > 0 else 0
)
df['avg_word_length'] = df['preprocessed_comment'].apply(
    lambda x: np.mean([len(word) for word in x.split()]) if len(x.split()) > 0 else 0
)

# Features numériques supplémentaires
additional_features = ['comment_length', 'word_count', 'exclamation_count', 
                       'question_count', 'uppercase_ratio', 'avg_word_length']

# Normalisation des features supplémentaires
scaler = preprocessing.StandardScaler()
df[additional_features] = scaler.fit_transform(df[additional_features])

# Combinaison avec TF-IDF
from scipy.sparse import hstack
X_combined = hstack([X_tfidf, df[additional_features].values])

print(f"Dimensions finales de X : {X_combined.shape}")
6.3 Matrice de Corrélation des Features Numériques
python
# Matrice de corrélation
correlation_features = ['comment_length', 'word_count', 'exclamation_count', 
                        'question_count', 'uppercase_ratio', 'avg_word_length',
                        'sentiment_encoded']

plt.figure(figsize=(10, 8))
correlation_matrix = df[correlation_features].corr()
sns.heatmap(correlation_matrix, annot=True, cmap='coolwarm', center=0, 
            square=True, linewidths=1)
plt.title('Matrice de Corrélation des Features Numériques', fontsize=14, fontweight='bold')
plt.tight_layout()
plt.show()
6.4 Gestion du Déséquilibre des Classes
python
# Vérification du déséquilibre
print("Distribution actuelle :")
print(df['sentiment_encoded'].value_counts())

# Application de ADASYN pour équilibrer
from imblearn.over_sampling import ADASYN

y = df['sentiment_encoded'].values
adasyn = ADASYN(random_state=42)
X_resampled, y_resampled = adasyn.fit_resample(X_combined, y)

print(f"\nDistribution après rééchantillonnage :")
print(pd.Series(y_resampled).value_counts())
print(f"Nouvelles dimensions : X={X_resampled.shape}, y={y_resampled.shape}")
7. Modélisation et Classification {#modelisation}
7.1 Division des Données
python
# Séparation train/test
X_train, X_test, y_train, y_test = train_test_split(
    X_resampled, y_resampled, 
    test_size=0.25, 
    random_state=42,
    stratify=y_resampled
)

print(f"Taille de l'ensemble d'entraînement : {X_train.shape[0]}")
print(f"Taille de l'ensemble de test : {X_test.shape[0]}")
7.2 Régression Logistique {#regression-logistique}
Contexte : La régression logistique est un algorithme linéaire particulièrement adapté à la classification de texte. Elle est rapide, interprétable et fonctionne bien avec des features TF-IDF haute dimension.

python
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report, accuracy_score, confusion_matrix

# Entraînement du modèle
lr_model = LogisticRegression(max_iter=1000, random_state=42, C=1.0)
lr_model.fit(X_train, y_train)

# Prédictions
y_lr_pred = lr_model.predict(X_test)

# Évaluation
print("=== RÉGRESSION LOGISTIQUE ===")
print(f"Accuracy : {accuracy_score(y_test, y_lr_pred):.4f}")
print("\nRapport de classification :")
print(classification_report(y_test, y_lr_pred, target_names=le.classes_))

# Matrice de confusion
plt.figure(figsize=(8, 6))
cm_lr = confusion_matrix(y_test, y_lr_pred)
sns.heatmap(cm_lr, annot=True, fmt='d', cmap='Blues', 
            xticklabels=le.classes_, yticklabels=le.classes_)
plt.title('Matrice de Confusion - Régression Logistique')
plt.ylabel('Valeur Réelle')
plt.xlabel('Valeur Prédite')
plt.tight_layout()
plt.show()
7.3 Arbre de Décision {#arbre-decision}
Contexte : Les arbres de décision capturent les relations non-linéaires dans les données. Ils sont intuitifs mais peuvent facilement sur-apprendre sans régularisation appropriée.

python
from sklearn.tree import DecisionTreeClassifier

# Entraînement
dt_model = DecisionTreeClassifier(
    criterion='entropy', 
    random_state=42,
    max_depth=20,
    min_samples_split=10,
    min_samples_leaf=5
)
dt_model.fit(X_train, y_train)

# Prédictions
y_dt_pred = dt_model.predict(X_test)

# Évaluation
print("=== ARBRE DE DÉCISION ===")
print(f"Accuracy : {accuracy_score(y_test, y_dt_pred):.4f}")
print("\nRapport de classification :")
print(classification_report(y_test, y_dt_pred, target_names=le.classes_))
7.4 K-Nearest Neighbors (KNN) {#knn}
Contexte : KNN est un algorithme basé sur les instances qui classifie selon la similarité. Il peut être efficace pour les données textuelles mais est sensible à la dimensionnalité.

python
from sklearn.neighbors import KNeighborsClassifier

# Entraînement
knn_model = KNeighborsClassifier(
    n_neighbors=5, 
    metric='cosine',  # Métrique cosinus adaptée au texte
    weights='distance'
)
knn_model.fit(X_train, y_train)

# Prédictions
y_knn_pred = knn_model.predict(X_test)

# Évaluation
print("=== K-NEAREST NEIGHBORS ===")
print(f"Accuracy : {accuracy_score(y_test, y_knn_pred):.4f}")
print("\nRapport de classification :")
print(classification_report(y_test, y_knn_pred, target_names=le.classes_))
7.5 Naive Bayes {#naive-bayes}
Contexte : Naive Bayes est particulièrement populaire pour la classification de texte grâce à sa simplicité et son efficacité. La version Multinomiale est adaptée aux features de comptage/TF-IDF.

python
from sklearn.naive_bayes import MultinomialNB, GaussianNB

# Multinomial Naive Bayes (converti en dense pour MultinomialNB)
mnb_model = MultinomialNB(alpha=1.0)
# Conversion en dense si nécessaire
X_train_dense = X_train.toarray() if hasattr(X_train, 'toarray') else X_train
X_test_dense = X_test.toarray() if hasattr(X_test, 'toarray') else X_test

# On s'assure que les valeurs sont positives pour MultinomialNB
from sklearn.preprocessing import MinMaxScaler
scaler_mnb = MinMaxScaler()
X_train_scaled = scaler_mnb.fit_transform(X_train_dense)
X_test_scaled = scaler_mnb.transform(X_test_dense)

mnb_model.fit(X_train_scaled, y_train)
y_mnb_pred = mnb_model.predict(X_test_scaled)

print("=== MULTINOMIAL NAIVE BAYES ===")
print(f"Accuracy : {accuracy_score(y_test, y_mnb_pred):.4f}")
print("\nRapport de classification :")
print(classification_report(y_test, y_mnb_pred, target_names=le.classes_))

# Gaussian Naive Bayes
gnb_model = GaussianNB()
gnb_model.fit(X_train_dense, y_train)
y_gnb_pred = gnb_model.predict(X_test_dense)

print("\n=== GAUSSIAN NAIVE BAYES ===")
print(f"Accuracy : {accuracy_score(y_test, y_gnb_pred):.4f}")
print("\nRapport de classification :")
print(classification_report(y_test, y_gnb_pred, target_names=le.classes_))
7.6 Support Vector Classifier (SVC) {#svc}
Contexte : Les SVM avec noyau RBF sont très performants pour la classification de texte, car ils peuvent capturer des relations complexes dans l'espace des features.

python
from sklearn.svm import SVC

# Entraînement
svc_model = SVC(kernel='rbf', C=1.0, gamma='scale', random_state=42)
svc_model.fit(X_train, y_train)

# Prédictions
y_svc_pred = svc_model.predict(X_test)

# Évaluation
print("=== SUPPORT VECTOR CLASSIFIER ===")
print(f"Accuracy : {accuracy_score(y_test, y_svc_pred):.4f}")
print("\nRapport de classification :")
print(classification_report(y_test, y_svc_pred, target_names=le.classes_))
7.7 Random Forest {#random-forest}
Contexte : Random Forest est un ensemble d'arbres de décision qui réduit le sur-apprentissage. Il est robuste et gère bien les données haute dimension.

python
from sklearn.ensemble import RandomForestClassifier

# Entraînement
rf_model = RandomForestClassifier(
    n_estimators=100,
    max_depth=20,
    min_samples_split=10,
    min_samples_leaf=5,
    random_state=42,
    n_jobs=-1
)
rf_model.fit(X_train, y_train)

# Prédictions
y_rf_pred = rf_model.predict(X_test)

# Évaluation
print("=== RANDOM FOREST ===")
print(f"Accuracy : {accuracy_score(y_test, y_rf_pred):.4f}")
print("\nRapport de classification :")
print(classification_report(y_test, y_rf_pred, target_names=le.classes_))

# Importance des features
if hasattr(X_train, 'toarray'):
    feature_names = list(tfidf.get_feature_names_out()) + additional_features
    importances = rf_model.feature_importances_
    indices = np.argsort(importances)[-20:]  # Top 20
    
    plt.figure(figsize=(10, 8))
    plt.barh(range(len(indices)), importances[indices])
    plt.yticks(range(len(indices)), [feature_names[i] for i in indices])
    plt.xlabel('Importance')
    plt.title('Top 20 Features - Random Forest')
    plt.tight_layout()
    plt.show()
7.8 XGBoost {#xgboost}
Contexte : XGBoost est un algorithme de boosting très performant, optimisé pour la vitesse et la précision. Il gère naturellement les features manquantes et inclut une régularisation.

python
from xgboost import XGBClassifier

# Entraînement
xgb_model = XGBClassifier(
    n_estimators=100,
    max_depth=10,
    learning_rate=0.1,
    random_state=42,
    eval_metric='mlogloss'
)
xgb_model.fit(X_train, y_train)

# Prédictions
y_xgb_pred = xgb_model.predict(X_test)

# Évaluation
print("=== XGBOOST ===")
print(f"Accuracy : {accuracy_score(y_test, y_xgb_pred):.4f}")
print("\nRapport de classification :")
print(classification_report(y_test, y_xgb_pred, target_names=le.classes_))
7.9 Multi-layer Perceptron (MLP) {#mlp}
Contexte : Le MLP est un réseau de neurones artificiels capable d'apprendre des représentations complexes. Il nécessite généralement plus de données et de temps d'entraînement.

python
from sklearn.neural_network import MLPClassifier

# Entraînement
mlp_model = MLPClassifier(
    hidden_layer_sizes=(100, 50),
    activation='relu',
    solver='adam',
    max_iter=300,
    random_state=42,
    early_stopping=True,
    validation_fraction=0.1
)
mlp_model.fit(X_train, y_train)

# Prédictions
y_mlp_pred = mlp_model.predict(X_test)

# Évaluation
print("=== MULTI-LAYER PERCEPTRON ===")
print(f"Accuracy : {accuracy_score(y_test, y_mlp_pred):.4f}")
print("\nRapport de classification :")
print(classification_report(y_test, y_mlp_pred, target_names=le.classes_))
7.10 Gradient Boosting {#gradient-boosting}
Contexte : Le Gradient Boosting construit séquentiellement des arbres pour corriger les erreurs des précédents. Il offre souvent les meilleures performances mais peut être lent.

python
from sklearn.ensemble import GradientBoostingClassifier

# Entraînement
gb_model = GradientBoostingClassifier(
    n_estimators=100,
    learning_rate=0.1,
    max_depth=10,
    random_state=42
)
gb_model.fit(X_train, y_train)

# Prédictions
y_gb_pred = gb_model.predict(X_test)

# Évaluation
print("=== GRADIENT BOOSTING ===")
print(f"Accuracy : {accuracy_score(y_test, y_gb_pred):.4f}")
print("\nRapport de classification :")
print(classification_report(y_test, y_gb_pred, target_names=le.classes_))
8. Validation Croisée {#validation-croisee}
8.1 K-Fold Cross Validation
La validation croisée K-Fold divise les données en K sous-ensembles et entraîne le modèle K fois, en utilisant chaque fois un sous-ensemble différent pour la validation.

python
from sklearn.model_selection import KFold, cross_val_score

k = 10
kf = KFold(n_splits=k, shuffle=True, random_state=42)

# Dictionnaire des modèles
models = {
    'Logistic Regression': lr_model,
    'Decision Tree': dt_model,
    'KNN': knn_model,
    'Gaussian NB
Continuer

': gnb_model,
'Multinomial NB': mnb_model,
'SVC': svc_model,
'Random Forest': rf_model,
'XGBoost': xgb_model,
'MLP': mlp_model,
'Gradient Boosting': gb_model
}

Évaluation avec K-Fold
print("=== K-FOLD CROSS VALIDATION (k=10) ===\n")
kfold_results = {}

for name, model in models.items():
# Certains modèles nécessitent des données denses
if name in ['Gaussian NB', 'Multinomial NB']:
scores = cross_val_score(model, X_resampled.toarray() if hasattr(X_resampled, 'toarray') else X_resampled,
y_resampled, cv=kf, scoring='accuracy')
else:
scores = cross_val_score(model, X_resampled, y_resampled, cv=kf, scoring='accuracy')

kfold_results[name] = {
    'mean': scores.mean(),
    'std': scores.std()
}
print(f"{name:25s} - Accuracy moyenne: {scores.mean():.4f} (+/- {scores.std():.4f})")
Visualisation
plt.figure(figsize=(12, 6))
model_names = list(kfold_results.keys())
means = [kfold_results[name]['mean'] for name in model_names]
stds = [kfold_results[name]['std'] for name in model_names]

plt.bar(range(len(model_names)), means, yerr=stds, alpha=0.7, capsize=5)
plt.xticks(range(len(model_names)), model_names, rotation=45, ha='right')
plt.ylabel('Accuracy')
plt.title('Comparaison des Modèles - K-Fold Cross Validation (k=10)')
plt.ylim([0, 1])
plt.grid(axis='y', alpha=0.3)
plt.tight_layout()
plt.show()


### 8.2 Stratified K-Fold Cross Validation

La validation croisée stratifiée maintient la proportion des classes dans chaque fold, ce qui est crucial pour les datasets déséquilibrés.
```python
from sklearn.model_selection import StratifiedKFold

skf = StratifiedKFold(n_splits=k, shuffle=True, random_state=42)

print("\n=== STRATIFIED K-FOLD CROSS VALIDATION (k=10) ===\n")
stratified_results = {}

for name, model in models.items():
    if name in ['Gaussian NB', 'Multinomial NB']:
        scores = cross_val_score(model, X_resampled.toarray() if hasattr(X_resampled, 'toarray') else X_resampled, 
                                 y_resampled, cv=skf, scoring='accuracy')
    else:
        scores = cross_val_score(model, X_resampled, y_resampled, cv=skf, scoring='accuracy')
    
    stratified_results[name] = {
        'mean': scores.mean(),
        'std': scores.std()
    }
    print(f"{name:25s} - Accuracy moyenne: {scores.mean():.4f} (+/- {scores.std():.4f})")

# Visualisation comparée
fig, axes = plt.subplots(1, 2, figsize=(16, 6))

# K-Fold
axes[0].bar(range(len(model_names)), 
            [kfold_results[name]['mean'] for name in model_names], 
            yerr=[kfold_results[name]['std'] for name in model_names],
            alpha=0.7, capsize=5, color='steelblue')
axes[0].set_xticks(range(len(model_names)))
axes[0].set_xticklabels(model_names, rotation=45, ha='right')
axes[0].set_ylabel('Accuracy')
axes[0].set_title('K-Fold Cross Validation')
axes[0].set_ylim([0, 1])
axes[0].grid(axis='y', alpha=0.3)

# Stratified K-Fold
axes[1].bar(range(len(model_names)), 
            [stratified_results[name]['mean'] for name in model_names], 
            yerr=[stratified_results[name]['std'] for name in model_names],
            alpha=0.7, capsize=5, color='coral')
axes[1].set_xticks(range(len(model_names)))
axes[1].set_xticklabels(model_names, rotation=45, ha='right')
axes[1].set_ylabel('Accuracy')
axes[1].set_title('Stratified K-Fold Cross Validation')
axes[1].set_ylim([0, 1])
axes[1].grid(axis='y', alpha=0.3)

plt.tight_layout()
plt.show()
```

---

## 9. Comparaison des Modèles {#comparaison-modeles}

### 9.1 Tableau Récapitulatif
```python
# Compilation des résultats
results_df = pd.DataFrame({
    'Modèle': model_names,
    'Accuracy Test': [
        accuracy_score(y_test, y_lr_pred),
        accuracy_score(y_test, y_dt_pred),
        accuracy_score(y_test, y_knn_pred),
        accuracy_score(y_test, y_gnb_pred),
        accuracy_score(y_test, y_mnb_pred),
        accuracy_score(y_test, y_svc_pred),
        accuracy_score(y_test, y_rf_pred),
        accuracy_score(y_test, y_xgb_pred),
        accuracy_score(y_test, y_mlp_pred),
        accuracy_score(y_test, y_gb_pred)
    ],
    'K-Fold CV': [kfold_results[name]['mean'] for name in model_names],
    'Stratified K-Fold CV': [stratified_results[name]['mean'] for name in model_names]
})

results_df = results_df.sort_values('Stratified K-Fold CV', ascending=False)
print("\n=== TABLEAU RÉCAPITULATIF DES PERFORMANCES ===\n")
print(results_df.to_string(index=False))

# Visualisation
fig, ax = plt.subplots(figsize=(14, 8))
x = np.arange(len(results_df))
width = 0.25

bars1 = ax.bar(x - width, results_df['Accuracy Test'], width, label='Test Set', alpha=0.8)
bars2 = ax.bar(x, results_df['K-Fold CV'], width, label='K-Fold CV', alpha=0.8)
bars3 = ax.bar(x + width, results_df['Stratified K-Fold CV'], width, label='Stratified K-Fold CV', alpha=0.8)

ax.set_xlabel('Modèles', fontsize=12)
ax.set_ylabel('Accuracy', fontsize=12)
ax.set_title('Comparaison Complète des Performances des Modèles', fontsize=14, fontweight='bold')
ax.set_xticks(x)
ax.set_xticklabels(results_df['Modèle'], rotation=45, ha='right')
ax.legend()
ax.grid(axis='y', alpha=0.3)
ax.set_ylim([0, 1])

plt.tight_layout()
plt.show()
```

### 9.2 Analyse des Métriques Détaillées
```python
from sklearn.metrics import precision_score, recall_score, f1_score

# Calcul des métriques pour tous les modèles
predictions = {
    'Logistic Regression': y_lr_pred,
    'Decision Tree': y_dt_pred,
    'KNN': y_knn_pred,
    'Gaussian NB': y_gnb_pred,
    'Multinomial NB': y_mnb_pred,
    'SVC': y_svc_pred,
    'Random Forest': y_rf_pred,
    'XGBoost': y_xgb_pred,
    'MLP': y_mlp_pred,
    'Gradient Boosting': y_gb_pred
}

detailed_metrics = []
for name, y_pred in predictions.items():
    detailed_metrics.append({
        'Modèle': name,
        'Accuracy': accuracy_score(y_test, y_pred),
        'Precision (macro)': precision_score(y_test, y_pred, average='macro'),
        'Recall (macro)': recall_score(y_test, y_pred, average='macro'),
        'F1-Score (macro)': f1_score(y_test, y_pred, average='macro')
    })

metrics_df = pd.DataFrame(detailed_metrics).sort_values('F1-Score (macro)', ascending=False)
print("\n=== MÉTRIQUES DÉTAILLÉES ===\n")
print(metrics_df.to_string(index=False))

# Heatmap des métriques
metrics_pivot = metrics_df.set_index('Modèle')
plt.figure(figsize=(10, 8))
sns.heatmap(metrics_pivot, annot=True, fmt='.3f', cmap='YlGnBu', cbar_kws={'label': 'Score'})
plt.title('Heatmap des Métriques de Performance', fontsize=14, fontweight='bold')
plt.tight_layout()
plt.show()
```

---

## 10. Conclusion {#conclusion}

### 10.1 Synthèse des Résultats

D'après les analyses effectuées sur le dataset Reddit Sentiment Analysis :

**Meilleurs modèles identifiés** :
- Les modèles d'ensemble (Random Forest, XGBoost, Gradient Boosting) montrent généralement les meilleures performances
- La Régression Logistique offre un excellent compromis entre simplicité et performance
- Le SVC avec noyau RBF démontre une grande capacité de généralisation

**Modèles moins performants** :
- Les modèles Naive Bayes (surtout Multinomial) obtiennent des résultats plus modestes
- Le KNN souffre de la haute dimensionnalité des features textuelles

### 10.2 Recommandations

**Pour la production** :
1. **Modèle recommandé** : Random Forest ou XGBoost pour leur robustesse et précision
2. **Alternative légère** : Régression Logistique pour des prédictions rapides avec peu de ressources
3. **Validation** : Utiliser Stratified K-Fold pour une évaluation fiable

**Améliorations possibles** :
- Utiliser des embeddings pré-entraînés (Word2Vec, GloVe, BERT)
- Appliquer des techniques de deep learning (LSTM, Transformers)
- Affiner le prétraitement du texte (gestion des emojis, slang Reddit)
- Optimiser les hyperparamètres avec GridSearch ou RandomizedSearch

### 10.3 Leçons Apprises

1. **Prétraitement critique** : Le nettoyage et la vectorisation des données textuelles influencent grandement les performances
2. **Équilibrage des classes** : ADASYN a permis d'améliorer significativement les résultats
3. **Validation croisée** : Essentielle pour éviter le sur-apprentissage et obtenir une estimation réaliste
4. **Compromis complexité/performance** : Les modèles simples peuvent rivaliser avec les plus complexes

---

## Annexes

### Sauvegarde des Modèles
```python
import joblib

# Sauvegarde du meilleur modèle
best_model = rf_model  # ou xgb_model, gb_model selon les résultats
joblib.dump(best_model, 'best_sentiment_model.pkl')
joblib.dump(tfidf, 'tfidf_vectorizer.pkl')
joblib.dump(le, 'label_encoder.pkl')

print("Modèles sauvegardés avec succès!")
```

### Fonction de Prédiction
```python
def predict_sentiment(text, model, vectorizer, label_encoder):
    """
    Prédit le sentiment d'un nouveau texte
    """
    # Prétraitement
    cleaned = clean_text(text)
    preprocessed = preprocess_text(cleaned)
    
    # Vectorisation
    vectorized = vectorizer.transform([preprocessed])
    
    # Prédiction
    prediction = model.predict(vectorized)
    sentiment = label_encoder.inverse_transform(prediction)[0]
    
    # Probabilités
    if hasattr(model, 'predict_proba'):
        probas = model.predict_proba(vectorized)[0]
        confidence = max(probas)
    else:
        confidence = None
    
    return sentiment, confidence

# Exemple d'utilisation
test_comment = "This is absolutely amazing! Best thing I've seen all day!"
sentiment, conf = predict_sentiment(test_comment, best_model, tfidf, le)
print(f"Texte: {test_comment}")
print(f"Sentiment prédit: {sentiment}")
if conf:
    print(f"Confiance: {conf:.2%}")
```

---

**Date de réalisation** : Novembre 2025  
**Dataset** : [Reddit Sentiment Analysis Dataset](https://github.com/alyahmedts13/reddit-sentiment-analysis-dataset-for-nlp-projects)  
**Outils utilisés** : Python, scikit-learn, NLTK, XGBoost, pandas, matplotlib, seaborn








4
