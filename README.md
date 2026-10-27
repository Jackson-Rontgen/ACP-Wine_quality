# ACP sur le Wine Quality Dataset

## Objectif

Je voulais comprendre ce qui différencie les vins entre eux, et voir si ces différences sont liées à la qualité.
Pour ça, j'ai fait une ACP normée (analyse en composantes principales). Je l'ai d'abord programmée à la main avec NumPy, puis je l'ai vérifiée avec scikit-learn.

## Résultats en bref

- 4 axes résument 70,8 % de l'information des 11 mesures chimiques des vins rouges.
- L'axe 1 (28,2 %) est un axe d'acidité. L'axe 2 (17,5 %) est un axe de dioxyde de soufre.
- Ces deux axes sont indépendants : l'acidité d'un vin ne dit rien sur sa quantité de soufre.
- Sur les 6497 vins (rouges et blancs), l'axe 1 sépare nettement les deux types. Les blancs ont plus de dioxyde de soufre et de sucre, les rouges plus de chlorures et d'acidité volatile.
- Le lien avec la qualité existe mais il reste modéré.

![Vins rouges, plan 1-2](plan_1_2.png)

## Les données

Wine Quality Dataset de l'UCI Machine Learning Repository : des vins portugais *vinho verde*.

- `winequality-red.csv` : 1599 vins rouges
- `winequality-white.csv` : 4898 vins blancs

Chaque vin est décrit par 11 mesures chimiques : `fixed acidity`, `volatile acidity`, `citric acid`, `residual sugar`, `chlorides`, `free sulfur dioxide`, `total sulfur dioxide`, `density`, `pH`, `sulphates`, `alcohol`. Il y a aussi une note de qualité donnée par des dégustateurs.
La note n'entre pas dans l'ACP. Je m'en sers seulement pour colorer les graphiques.

Il n'y a aucune valeur manquante. Il y a des doublons (240 chez les rouges, 937 chez les blancs). Je les ai gardés, ce sont des mesures plausibles de vins différents.

Source à citer : P. Cortez, A. Cerdeira, F. Almeida, T. Matos et J. Reis, *Modeling wine preferences by data mining from physicochemical properties*, Decision Support Systems, Elsevier, 47(4):547-553, 2009.

## Ce que j'ai fait

**1. Explorer les données.** Statistiques descriptives, boxplots, matrice des corrélations.

**2. Centrer et réduire.** Sur les données brutes, `total sulfur dioxide` représente 90,3 % de la variance totale et `free sulfur dioxide` 9,1 %. Les neuf autres variables se partagent environ 0,5 %. Sans réduction, le premier axe ne serait que l'axe du dioxyde de soufre. C'est un effet des unités, pas une vraie information sur les vins.

**3. Faire l'ACP à la main.**
- matrice de corrélation V = (1/n) ZᵀZ, que j'ai vérifiée égale à `X.corr()`
- diagonalisation avec `np.linalg.eigh`, tri des valeurs propres, leur somme vaut 11 (la trace de V)
- composantes principales C = Z P, dont les variances sont égales aux valeurs propres

**4. Choisir le nombre d'axes.**

| Critère | Nombre d'axes |
|---|---|
| Kaiser (λ > 1) | 4 |
| Variance cumulée ≥ 70 % | 4 (70,8 %) |
| Variance cumulée ≥ 80 % | 6 (85,5 %), car 5 axes donnent seulement 79,5 % |
| Coude de l'éboulis | pas net : cassure après l'axe 2, palier après l'axe 6 |

Je garde 4 axes, parce que Kaiser et le seuil de 70 % sont d'accord.

**5. Interpréter.** Avec les cercles des corrélations et le cos². Je n'interprète une variable sur un plan que si son cos² dépasse 0,5.

**6. Vérifier avec scikit-learn.** `StandardScaler` et `PCA`.

**7. Comparer les vins rouges et blancs.** ACP sur les 6497 vins, sans donner le type à l'algorithme.

## Résultats

### Valeurs propres et inertie (vins rouges)

| Axe | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|
| Valeur propre | 3,10 | 1,93 | 1,55 | 1,21 | 0,96 | 0,66 |
| Inertie (%) | 28,2 | 17,5 | 14,1 | 11,0 | 8,7 | 6,0 |
| Cumul (%) | 28,2 | 45,7 | 59,8 | 70,8 | 79,5 | 85,5 |

### Interprétation des axes

- **Axe 1 : l'acidité.** À gauche on trouve `fixed acidity` (−0,86), `citric acid` (−0,82) et `density` (−0,70). À droite on trouve `pH` (+0,77). Un pH élevé veut dire un vin moins acide, donc c'est cohérent.
- **Axe 2 : le dioxyde de soufre.** Il est porté par `total sulfur dioxide` (+0,79) et `free sulfur dioxide` (+0,71).
- **Axe 3 :** aucune variable n'a un cos² supérieur à 0,5. Je ne l'interprète pas.
- **Axe 4 :** il est porté par `chlorides` (+0,73).

![Cercle des corrélations, plan 1-2](cercle_1_2.png)

### Lien avec la qualité

La corrélation entre la note et les axes vaut −0,11 (axe 1), −0,39 (axe 2), −0,40 (axe 3) et −0,04 (axe 4).
Les vins les mieux notés sont plutôt en bas du plan 1-2, donc parmi les moins sulfités. Mais le soufre seul explique peu la note (corrélation de −0,19 avec `total sulfur dioxide` et de −0,05 avec `free sulfur dioxide`).

### Vins rouges et blancs ensemble

| Axe | 1 | 2 | 3 | 4 |
|---|---|---|---|---|
| Valeur propre | 3,03 | 2,49 | 1,56 | 0,97 |
| Inertie (%) | 27,5 | 22,7 | 14,1 | 8,8 |

Les 4 axes résument 73,2 % de l'information. Ici Kaiser ne garde que 3 axes (la quatrième valeur propre vaut 0,97). Je garde quand même 4 axes pour rester cohérent avec l'ACP précédente.

L'axe 1 sépare les deux types : la moyenne vaut −2,51 pour les rouges et +0,82 pour les blancs.
- Côté blancs : `total sulfur dioxide` (+0,85), `free sulfur dioxide` (+0,75) et `residual sugar` (+0,60).
- Côté rouges : `chlorides` (−0,51, à la limite du seuil) et `volatile acidity` (−0,66, juste sous le seuil).

L'ACP retrouve donc toute seule la différence entre rouge et blanc.

![Vins rouges et blancs, plan 1-2](rouge_blanc_plan_1_2.png)

### Vérification avec scikit-learn

- `StandardScaler` donne la même matrice que mon centrage-réduction.
- Les taux d'inertie sont identiques (28,2 ; 17,5 ; 14,1 ; 11,0 %).
- Les valeurs propres sont très légèrement différentes (3,101 contre 3,099), parce que scikit-learn divise par n−1 et moi par n.
- Les composantes et les vecteurs propres sont égaux au signe près. Le signe d'un vecteur propre est défini à un facteur −1 près.

### Valeurs extrêmes

Les boxplots montrent des valeurs extrêmes, surtout pour `residual sugar` et `chlorides` : 405 vins sur 1599 sont signalés par la règle 1,5 × IQR sur au moins une variable.
Ce sont des mesures chimiques plausibles et pas des erreurs de saisie, donc je les ai gardées.

## Limites

- Une corrélation n'est pas une cause. L'ACP décrit des liens, pas des causes.
- Le plan 1-2 montre seulement 45,7 % de l'information, et 45 % des vins y sont bien représentés (cos² > 0,5). Le nuage donne une tendance, on ne lit pas vin par vin.
- La note de qualité est subjective, et plus de 80 % des vins rouges ont un 5 ou un 6.
- L'ACP est sensible aux valeurs extrêmes. Je les ai gardées, mais je n'ai pas testé leur influence sur les axes.
- Les axes 3 et 4 sont difficiles à interpréter, très peu de variables y sont bien représentées.
- Les résultats concernent seulement les *vinho verde* portugais.

## Pour aller plus loin

- Prédire le type de vin (rouge ou blanc) avec une régression logistique.
- Regrouper les vins avec un K-means sur les composantes principales.
- Passer les variables très asymétriques (`chlorides`, `residual sugar`, `sulphates`) au logarithme avant l'ACP.
