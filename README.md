# Powerbi-tarification-assurance
Dashboard Power BI - Diagnostic de la sinistralité auto et modélisation d'une prime actuarielle.

# Diagnostic de rentabilité d'un portefeuille d'assurance automobile

Projet Power BI visant à identifier les segments de clientèle responsables de la dégradation de la rentabilité d'un portefeuille d'assurance auto et à formuler des recommandations tarifaires fondées sur les données.

## Contexte 

Un assureur constate une hausse de sa sinistralité sans parvenir à identifier les segments de clientèle à l'origine du phénomène.
L'objectif est de déterminer quels profils génèrent le plus de pertes techniques, de distinguer les signaux réellement significatifs des effets liés à de faibles volumes de données et de proposer des actions correctives.

## Objectif

- Mesurer la rentabilité des segments du portefeuille.
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
  - exposition comprise entre 0 et 1 ;
  - Bonus-Malus compris entre 50 et 350.
- Jointure des données de fréquence et de sévérité.

### Modélisation

- Construction d'un modèle en étoile sous Power BI.
- Création de mesures DAX pour :
  - la fréquence des sinistres ;
  - le coût moyen par sinistre ;
  - la prime pure simulée ;
  - les indicateurs de rentabilité.

### Simulation actuarielle

En l'absence de primes réelles dans le jeu de données, simulation d'une prime pure actuarielle :

Prime pure = Fréquence des sinistres × Coût moyen par sinistre

### Contrôle statistique

- Vérification de la crédibilité des résultats selon les volumes observés.
- Mise en place d'un seuil de vigilance pour les segments comportant moins de 50 sinistres.
- Analyse croisée des variables afin d'identifier les principaux facteurs de risque et limiter les effets de confusion (exemple : âge du conducteur × puissance du véhicule).

## Résultats  

### 1. Jeunes conducteurs : principal facteur de risque
Les conducteurs âgés de 18 à 25 ans représentent seulement **5,7 %** **du portefeuille**, mais affichent **un indice de rentabilité de** **454,7 %**, contre **77 %** à **92 %** pour les autres tranches d'âge.

### 2. Effet combiné âge × puissance
Les jeunes conducteurs possédant des véhicules de puissance moyenne affichent un **indice de rentabilité** de **897,6 %**.
Cette sur-sinistralité s'explique par :
- une fréquence de sinistres **2,3** fois supérieure à la moyenne ;
- un coût moyen par sinistre **3,9** fois plus élevé.

### 3. Limite du système Bonus-Malus
Les contrats présentant un Bonus-Malus neutre (100) affichent l'indice de rentabilité le plus élevé du portefeuille (**978,6 %**).

Ce résultat suggère que le système basé uniquement sur l'historique de conduite évalue difficilement le risque des nouveaux assurés.

## Recommandation métier

Mettre en place une surprime à la souscription pour les jeunes conducteurs, particulièrement sur les véhicules de puissance moyenne à élevée, afin d'améliorer la rentabilité du portefeuille dès l'entrée du contrat.

## Compétences mobilisées
- Power BI
- Power Query
- DAX
- Modélisation de données
- Data Visualisation
- Analyse statistique
- Assurance IARD
- Data Storytelling
- Aide à la décision

## Stack technique
- **Power Query** : nettoyage et préparation des données
- **DAX** : mesures actuarielles (fréquence, sévérité, prime pure, Loss Ratio), mise en forme conditionnelle par règles
- **Power BI** : modélisation en étoile, arborescence de décomposition interactive, paramètre de champ

## Limites

- Les données ne contiennent pas les primes réellement facturées aux assurés. Les indicateurs de rentabilité reposent donc sur une prime actuarielle simulée calculée à partir de la fréquence et du coût moyen des sinistres.
- Certains segments présentent des volumes de sinistres limités. Par exemple, une région affiche un indice de rentabilité élevé (239,6 %), mais repose sur seulement 77 sinistres observés. Ces résultats doivent être interprétés avec prudence.
- Le jeu de données ne comporte pas d'informations relatives à la fraude, qui pourrait expliquer une partie de la sinistralité observée.
- L'analyse repose sur un historique figé et ne prend pas en compte l'évolution temporelle du portefeuille ou des comportements des assurés.

## Aperçu du dashboard
![Vue d'ensemble du dashboard](dashboard-overview_.png)
[Voir le dashboard complet (PDF)](dashboard-overview.pdf)


## Structure du repo

```
├── README.md
├── dashboard.pbix
├── screenshots/
└── docs/
    ├── note-cadrage.md
    └── methodologie.md
```

## Contact

Essoréké KIHEOU - [LinkedIn](https://www.linkedin.com/in/essor%C3%A9k%C3%A9-kiheou-93996541b) - essorekekiheou@gmail.com
