# Diagnostic de la sinistralité d'un portefeuille d'assurance automobile

Dashboard Power BI : diagnostic de la sinistralité auto et indice de prime pure par segment, validé statistiquement (écrêtement des sinistres graves, intervalles de confiance, GLM de Poisson).

> **En bref**
> - **Les jeunes conducteurs (18-25 ans) ont une fréquence de sinistres 2 fois plus élevée** que la moyenne. Leur indice de prime pure est de **225 %** une fois les sinistres graves écrêtés.
> - **132 sinistres graves (0,5 % des sinistres) représentent 33 % du coût total.** Un seul sinistre de 4,1 M€ suffisait à tripler l'indice de plusieurs segments : sans écrêtement, les conclusions auraient été fausses.
> - **Le Bonus-Malus fonctionne :** la fréquence monte régulièrement avec le coefficient, et il explique à lui seul l'essentiel du sur-risque des jeunes. L'enjeu est de tarifer les **nouveaux conducteurs**, qui démarrent tous à 100, dès la souscription.

![Tableau de bord](docs/dashboard.png)

[Voir le dashboard complet (PDF, 2 pages)](docs/dashboard.pdf) · [Notebook de validation statistique](notebooks/validation_statistique.ipynb)

Projet Power BI visant à identifier les segments de clientèle qui coûtent le plus cher dans un portefeuille d'assurance auto et à formuler des recommandations tarifaires fondées sur les données.

## Contexte

Un assureur constate une hausse de sa sinistralité sans parvenir à identifier les segments de clientèle à l'origine du phénomène.
L'objectif est de déterminer quels profils génèrent le plus de pertes techniques, de distinguer les signaux réellement significatifs des effets liés à de faibles volumes de données et de proposer des actions correctives.

## Objectif

- Mesurer le coût relatif des segments du portefeuille.
- Identifier les populations les plus risquées.
- Analyser les facteurs expliquant les écarts de sinistralité.
- Formuler des recommandations de tarification exploitables par les équipes métier.

## Données

- **Source** : [freMTPL2freq / freMTPL2sev](https://www.openml.org/d/41214) - dataset public French Motor Third-Party Liability (Christophe Dutang, Computational Actuarial Science with R, CRC 2018)

**Volume analysé** :
- 678 007 contrats
- 26 383 sinistres
- 358 359 années d'exposition

**Variables clés** : exposition, âge conducteur, puissance véhicule, région, coefficient Bonus-Malus, montant des sinistres

## Méthodologie

### Préparation des données

- Nettoyage et contrôle qualité sous Power Query.
- Vérification des règles métier :
  - âge conducteur ≥ 18 ans ;
  - exposition comprise entre 0 et 1 an (aucune exposition supérieure à 1 dans le fichier utilisé) ;
  - Bonus-Malus compris entre 50 et 350.
- Jointure des données de fréquence et de sévérité.
- 256 sinistres du fichier des montants (920 000 €) n'ont pas de contrat correspondant : ils sont exclus des calculs.

### Modélisation

- Construction d'un modèle en étoile sous Power BI.
- Création de mesures DAX pour :
  - la fréquence des sinistres ;
  - le coût moyen par sinistre ;
  - la prime pure ;
  - l'indice de prime pure, brut et écrêté ;
  - les sinistres graves (seuil, nombre, part du coût) ;
  - la crédibilité des segments (intervalle de confiance de la fréquence, alerte sous 50 sinistres).

### Indicateur : l'indice de prime pure

En l'absence de primes réelles dans le jeu de données, chaque segment est comparé au portefeuille à partir de sa prime pure :

Prime pure = Fréquence des sinistres × Coût moyen par sinistre = Coût des sinistres / Exposition

**Indice de prime pure** = prime pure du segment / prime pure du portefeuille (100 % = moyenne).

Un indice de 225 % signifie qu'une année d'assurance de ce segment coûte 2,25 fois plus en sinistres que la moyenne. Ce n'est pas une rentabilité ni un loss ratio : il faudrait pour cela les primes réellement encaissées.

### Traitement des sinistres graves

Le coût des sinistres est très concentré : **0,5 % des sinistres représentent 33 % du coût total**, et le plus gros sinistre (4,1 M€) pèse à lui seul 6,8 % du coût.
Chaque sinistre est donc **plafonné au 99,5e percentile (34 495 €)** pour calculer l'indice écrêté. C'est la pratique actuarielle courante : la partie qui dépasse le seuil est mutualisée sur tout le portefeuille, au lieu d'être imputée au segment où le hasard l'a fait tomber.

### Contrôle statistique

- Vérification de la crédibilité des résultats selon les volumes observés.
- Mise en place d'un seuil de vigilance pour les segments comportant moins de 50 sinistres, et intervalle de confiance à 95 % sur la fréquence.
- Analyse croisée des variables afin d'identifier les principaux facteurs de risque et limiter les effets de confusion (exemple : âge du conducteur × puissance du véhicule).
- **Validation dans un notebook Python** ([validation_statistique.ipynb](notebooks/validation_statistique.ipynb)) :
  - reproduction des indices du dashboard ;
  - analyse de la concentration du coût et effet de l'écrêtement ;
  - intervalles de confiance exacts (loi de Poisson) ;
  - GLM de Poisson sur la fréquence (âge, puissance, Bonus-Malus, interaction), avec correction de la surdispersion.

## Résultats

### 1. Jeunes conducteurs : une fréquence deux fois plus élevée

Les conducteurs âgés de 18 à 25 ans représentent **5,7 % du portefeuille**, mais ont une fréquence de **14,8 %**, contre 7,4 % en moyenne. L'intervalle de confiance est serré (14,2 % à 15,4 %).
Leur indice de prime pure écrêté est de **225 %**, contre 85 % à 98 % pour les autres tranches d'âge.

L'indice brut (455 %) surestimait l'écart : un seul sinistre de 4,1 M€ (conducteur de 19 ans) en expliquait une grande partie.

![Indice brut et écrêté](docs/figures/indice_brut_vs_ecrete.png)

### 2. Effet combiné âge × puissance : un signal réel mais modeste

Les jeunes conducteurs de véhicules de puissance moyenne (7 à 9) ont une fréquence **2,3 fois** supérieure à la moyenne.
Le GLM de Poisson montre une **interaction âge × puissance d'environ +25 %** sur la fréquence : ces jeunes ont 25 % de sinistres en plus de ce que prédisent l'âge et la puissance séparément (p ≈ 0,01 après correction de la surdispersion).

Leur indice écrêté est de **295 %**. L'indice brut de 897,6 % et le « coût moyen 3,9 fois plus élevé » venaient presque entièrement du sinistre de 4,1 M€ : sans lui, le coût moyen n'est plus que 1,4 fois la moyenne.

### 3. Le Bonus-Malus fonctionne : l'enjeu est la souscription des nouveaux conducteurs

La fréquence augmente régulièrement avec le coefficient : **6,4 %** en bonus fort, 11,7 % en bonus léger, 21,2 % au niveau neutre (100), 34,6 % en malus et 56,8 % en malus fort.

Une fois le Bonus-Malus pris en compte dans le GLM, **l'effet propre de l'âge sur la fréquence disparaît** (relativité 1,05, non significative). Le Bonus-Malus capte donc déjà le sur-risque des jeunes conducteurs.

Le niveau neutre (100) a un indice écrêté de **359 %** (979 % en brut, à cause du même sinistre de 4,1 M€). Il regroupe surtout des nouveaux conducteurs : **42 % des contrats à 100 sont des jeunes**, contre 5,7 % dans le portefeuille. Un assuré sans historique démarre à 100, et le Bonus-Malus ne peut pas encore le distinguer : c'est à la souscription que le tarif doit faire la différence.

![Effet de l'âge avant et après prise en compte du Bonus-Malus](docs/figures/glm_age_bonus_malus.png)

## Recommandations métier

1. **Tarifer les nouveaux conducteurs dès la souscription.** Pour les contrats sans historique (Bonus-Malus à 100), appliquer une majoration selon l'âge, renforcée pour les jeunes au volant d'un véhicule de puissance moyenne ou élevée. Le Bonus-Malus prend ensuite le relais.
2. **Traiter les sinistres graves à part.** Écrêter les sinistres au-delà du seuil et financer leur surcoût par une charge mutualisée sur tout le portefeuille (ou par la réassurance), plutôt que de surtarifer un segment à cause d'un sinistre isolé.
3. **Ne pas réviser le tarif d'un segment sur un indice brut.** Toujours vérifier le nombre de sinistres, l'intervalle de confiance et l'indice écrêté. Par exemple, la région R21 (Champagne-Ardenne) affichait 240 % à cause d'un seul sinistre de 369 000 € : son indice écrêté est proche de la moyenne.

## Compétences mobilisées
- Power BI
- Power Query
- DAX
- Modélisation de données
- Data Visualisation
- Analyse statistique
- Modèles linéaires généralisés (GLM de Poisson)
- Python (pandas, statsmodels)
- Assurance IARD
- Data Storytelling
- Aide à la décision

## Stack technique
- **Power Query** : nettoyage et préparation des données
- **DAX** : mesures actuarielles (fréquence, sévérité, prime pure, indice de prime pure brut et écrêté, sinistres graves, crédibilité), mise en forme conditionnelle par règles
- **Power BI** : modélisation en étoile, arborescence de décomposition interactive, paramètre de champ, projet Power BI (.pbip) versionné sous Git
- **Python** : pandas, statsmodels (GLM de Poisson, quasi-Poisson), scipy, matplotlib

## Limites

- Les données ne contiennent pas les primes réellement facturées aux assurés. L'indice de prime pure compare donc des coûts entre segments ; il ne mesure ni une rentabilité ni un loss ratio.
- Certains segments présentent des volumes de sinistres limités. Par exemple, le Malus fort ne compte que 54 sinistres : sa fréquence est connue avec une marge large (environ 43 % à 74 %). Ces résultats doivent être interprétés avec prudence.
- Le seuil d'écrêtement (99,5e percentile) est un choix de méthode : un autre seuil changerait légèrement les indices, sans modifier les conclusions.
- Le GLM porte sur la fréquence. La sévérité est analysée de façon descriptive ; un GLM Gamma sur les coûts écrêtés compléterait la tarification.
- Le modèle de fréquence présente une surdispersion (≈ 3), due à des variables non incluses (région, âge du véhicule, densité). Elle est corrigée par une approche quasi-Poisson.
- Le jeu de données ne comporte pas d'informations relatives à la fraude, qui pourrait expliquer une partie de la sinistralité observée.
- L'analyse repose sur un historique figé et ne prend pas en compte l'évolution temporelle du portefeuille ou des comportements des assurés.

## Ouvrir le projet

**Dashboard Power BI**
1. Télécharger `freMTPL2freq.csv` et `freMTPL2sev.csv` (lien dans la section Données) et les placer dans le dossier `data/`.
2. Ouvrir `powerbi/Ratio Loss Auto.pbip` avec Power BI Desktop.
3. Dans **Transformer les données**, remplacer le chemin des deux sources par celui du dossier `data/`, puis **Actualiser**.

**Notebook**
```bash
pip install -r requirements.txt
jupyter notebook notebooks/validation_statistique.ipynb
```

## Structure du repo

```
├── README.md
├── requirements.txt
├── data/                          # fichiers CSV freMTPL2 (non versionnés)
├── powerbi/
│   ├── Ratio Loss Auto.pbip       # projet Power BI (à ouvrir avec Power BI Desktop)
│   ├── Ratio Loss Auto.Report/    # pages, visuels et thème
│   └── Ratio Loss Auto.SemanticModel/  # modèle de données et mesures DAX (model.bim)
├── notebooks/
│   └── validation_statistique.ipynb
└── docs/
    ├── dashboard.pdf              # export du dashboard (2 pages)
    ├── dashboard.png
    └── figures/                   # graphiques du notebook
```

## Contact

Essoréké KIHEOU - [LinkedIn](https://www.linkedin.com/in/essor%C3%A9k%C3%A9-kiheou-93996541b) - essorekekiheou@gmail.com
