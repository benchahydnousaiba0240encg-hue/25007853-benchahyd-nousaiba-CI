# CI02 – Prévision des importations de blé en provenance de France

Contrôle continu – Partie B, Science des données et commerce international
ENCG Settat · S7 Commerce international · Module BD & DS · 2026–2027
Enseignant : A. Larhlimi

| Information | Valeur |
|---|---|
| Nom et prénom | Nousaiba Benchahyd |
| Code Apogée | 2500 7853 |
| Groupe | CI |
| Sujet attribué | CI02 – Prévision des importations agricoles (cas du blé, partenaire France) |
| Date de l'épreuve | 19/11/2026 |
| Dépôt GitHub | https://github.com/benchahydnousaiba0240encg-hue/25007853-benchahyd-nousaiba-CI |
| Notebook | `notebooks/projet_CI02.ipynb` |

## 1. Problématique

Est-il possible d'anticiper la valeur financière annuelle des importations de blé en provenance de France à partir de l'historique des flux commerciaux et des volumes physiques transportés ?

- **Tâche** : régression supervisée sur séries temporelles.
- **Objectif fixé avant les résultats** : R² ≥ 0,85 sur le jeu de test et MAE réduite d'au moins 50 % par rapport à la référence naïve (moyenne).

## 2. Données

- **Source** : UN Comtrade (Nations unies), licence UN Open Data.
- **Extraction** : novembre 2026, importations (`flowCode = M`), partenaire France (`partnerISO = FRA`), produit SH 1001 (blé), fréquence annuelle, période 2009–2025.
- **URL de l'extraction** : https://comtradeplus.un.org/en/trade-statistics?type=C&freq=A&cl=HS&cmd=1001&period=2025,2024,2023,2022,2021,2020,2019,2018,2017,2016,2015&reporter=all&partner=251&flow=M&partner2=0&mot=0&customs=C00&breakdown=plus&aggregate=NONE
- **Dimensions** : 856 lignes × 47 colonnes, 128 pays déclarants.
- **Unité d'observation** : flux annuel d'importation de blé d'un pays déclarant en provenance de France.
- **Unités** : valeur en USD (`primaryValue`, variable cible), poids net en kg (`netWgt`).
- **Valeurs manquantes principales** : `fobvalue` 65,9 %, `altQtyUnitAbbr` 24,2 %, `grossWgt` 18,2 %, `altQty` 10,4 %, `netWgt` 1,9 %.

## 3. Méthode

- **Préparation** : `netWgt` imputé par `qty` si manquant ; tri par pays et par année ; variables retardées d'un an (`primaryValue_lag1`, `netWgt_lag1`, `price_per_kg_lag1`).
- **Variables explicatives** : `refYear`, `primaryValue_lag1`, `netWgt_lag1`, `price_per_kg_lag1`, `netWgt`.
- **Découpage chronologique sans mélange** : entraînement 2009–2022 (523 lignes), test 2023–2025 (205 lignes).
- **Standardisation** : `StandardScaler` ajusté sur l'entraînement uniquement.
- **Algorithme principal** : Random Forest Regressor (algorithme n° 5 des 29 du cours), `n_estimators = 100`, `max_depth = 10`, `random_state = 42`.
- **Alternative écartée** : régression linéaire multiple / Ridge (relation supposée linéaire, colinéarité entre valeur courante et retards).
- **Référence naïve** : `DummyRegressor(strategy="mean")`.
- **Métrique principale** : R². **Métriques secondaires** : MAE et RMSE (en millions USD).

## 4. Résultats

| Méthode | R² test | MAE test |
|---|---|---|
| Référence naïve (moyenne) | −0,0014 | 82,77 M$ |
| Random Forest | 0,9515 | 12,16 M$ |

- La MAE est réduite d'environ 85 % par rapport à la moyenne : l'objectif (R² ≥ 0,85, MAE réduite d'au moins 50 %) est atteint.
- Sur l'entraînement : R² = 0,9826 et MAE = 4,47 M$, ce qui indique un léger surapprentissage.
- Exemple hors entraînement : importations du Maroc en 2023, 379,8 M$ observés contre 367,5 M$ prédits (écart d'environ 3,2 %).

## 5. Limites et améliorations

- Seulement trois années de test (205 lignes), avec un choc de prix en 2022.
- Le poids net de l'année courante (`netWgt`) n'est pas connu à l'avance dans une vraie prévision.
- Hyperparamètres fixés, sans optimisation approfondie.
- Améliorations : validation glissante (`TimeSeriesSplit`) avec grille sur `max_depth` ; ajout de variables exogènes (prix mondial du blé, taux de change) sans le poids courant.

## 6. Reproductibilité

1. Cloner le dépôt puis installer les dépendances : `pip install -r requirements.txt`.
2. Placer l'extraction UN Comtrade dans `data/raw/` (fichier `PREVISION-IMPORTATION-FRANCE.csv`) ou la retélécharger avec l'URL ci-dessus.
3. Ouvrir `notebooks/projet_CI02.ipynb` (Google Colab ou Jupyter), adapter le chemin du fichier CSV si besoin, puis exécuter toutes les cellules.

Environnement utilisé : Python 3.13, pandas 2.2.3, scikit-learn 1.6.1. Graine aléatoire fixée à 42.

## 7. Contenu du dépôt

```
├── README.md
├── requirements.txt
├── .gitignore
├── DECLARATION_IA.md
├── data/raw/            # extraction UN Comtrade ou notice de téléchargement
├── docs/                # fiche du sujet CI02 et références
├── notebooks/projet_CI02.ipynb
├── figures/             # graphiques en couleurs
├── rapport/rapport.pdf
├── presentation/        # presentation.pdf et presentation.pptx (6 diapositives)
└── video/               # demonstration.mp4 ou lien dans ce README
```

## 8. Références

- UN Comtrade : https://comtradeplus.un.org/
- Documentation scikit-learn, `RandomForestRegressor` : https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestRegressor.html
- Énoncé du contrôle continu, Partie B (A. Larhlimi, ENCG Settat).

## 9. Déclaration d'usage des outils d'IA

Voir `DECLARATION_IA.md`.
