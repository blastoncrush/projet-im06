# Rapport - semaine 1

## 1. Prétraitement des Images
L'objectif est d'homogénéiser le dataset (1645 images) pour l'extraction de caractéristiques tout en préservant l'intégrité visuelle.
* **Redimensionnement** : Ajustement de la dimension maximale à 1800 pixels avec une interpolation adaptée selon le facteur d'échelle (`INTER_AREA` pour la réduction, `INTER_CUBIC` pour l'agrandissement).
* **Padding Symétrique** : Extension de l'image en un carré parfait de 1800x1800 via une réflexion miroir, comme discuté lors de notre réunion.

## 2. Extraction Multi-modale de Caractéristiques (1856 Features)
Le pipeline extrait un vecteur exhaustif couvrant trois piliers de l'analyse d'image :

### A. Caractéristiques de Couleur
* **Histogrammes HSV** : Analyse de la distribution de la teinte, de la saturation et de la valeur sur 32 bins par canal.
* **Statistiques Lab** : Calcul de la moyenne, de l'écart-type, de l'asymétrie (*skewness*) et de l'aplatissement (*kurtosis*) dans l'espace perceptuel L*a*b*.
* **Palette Dominante** : Identification des 5 couleurs principales et de leurs proportions via l'algorithme K-Means.
* **Luminosité Spatiale** : Moyenne de luminance sur une grille 4x4 pour capturer la structure de composition globale.

### B. Caractéristiques de Texture
* **GLCM (Matrices de Co-occurrence)** : Calcul des propriétés d'Haralick (contraste, corrélation, énergie, etc.) à plusieurs distances et angles.
* **LBP (Local Binary Patterns)** : Analyse de la rugosité de surface à trois échelles (rayons 1, 2, 4).
* **Filtres de Gabor** : Banque de filtres (4 orientations x 3 fréquences) simulant la réponse du cortex visuel humain.

### C. Géométrie et Structure
* **Analyse FFT** : Calcul de la pente du spectre de puissance pour quantifier la complexité visuelle et le ratio hautes/basses fréquences.
* **Entropie de Shannon** : Mesure du désordre visuel au niveau global et sur des patches locaux de 64x64.
* **Gradients & HOG** : Magnitude de Sobel et Histogramme de Gradients Orientés pour capturer les formes et les contours.
* **Tenseur de Structure** : Mesure de la cohérence directionnelle locale.
* **Dimension Fractale** : Estimation par *box-counting* de la complexité de l'autosimilarité des contours.

## 3. Pipeline de Données et Nettoyage
* **Agrégation** : Compilation des vecteurs en une matrice de features $X$ de dimension $(1645, 1856)$.
* **Nettoyage** : 
    * Remplacement des valeurs infinies par des `NaN`.
    * Suppression des features trop peu fiables (plus de 20% de valeurs manquantes).
    * Imputation des `NaN` restants par la médiane de chaque colonne.
* **Normalisation** : Utilisation du `StandardScaler` pour normaliser les données.

## 4. Réduction de Dimension (PCA)
* 394 composantes ont été retenues pour expliquer 95% de la variance totale du dataset.
* **Visualisation** : Projection 2D (PC1 vs PC2) permettant d'observer les regroupements d'œuvres (ce qui n'a pas été pertinent, voir visualisation.png).

## 5. Analyse de Similarité et Validation
* **Recherche de Voisins** : Implementation d'une fonction de recherche utilisant la distance Cosinus dans l'espace PCA (plus pertinente que la distance Euclidienne).
* **Analyse de Cohérence des Peintres** : Étude comparant les distances "intra-peintre" et "inter-peintres" :
    * **Métriques de Rang** : Calcul du rang absolu et du percentile de chaque œuvre d'un même artiste par rapport au dataset global.
    * **Validation Statistique** : Utilisation du test de **Mann-Whitney U** pour vérifier si les œuvres d'un peintre sont significativement plus proches entre elles que de la moyenne des autres œuvres.

## 6. Résultats
Des formes générales comme des lignes verticales (voir fontana.jpg, damoah.jpg, riley.jpg) ou même des formes plus complexes (voir accardi.jpg) ou des textures (voir kossof.jpg) ou des palettes de couleurs (voir mitchell.jpg) trouvent de fortes similarités inter-peintre.

Certains peintres (voir domela.jpg, bergman.jpg) ont un style changeant entre les oeuvres, ce qui se traduit par des rangs très élevés (moins de similarité intra-peintre).