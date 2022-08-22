# Utilisation des outils de SIG et data analyse pour la planification du district

## Introduction
Selon l'UNICEF la microplanification est l’un des outils utilisés par les agents de santé pour s’assurer que les services de vaccination touchent chaque communauté. La microplanification est utilisée pour identifier les communautés prioritaires, pour surmonter les obstacles existants et pour dresser des plans de travail donnant lieu à des solutions.
Elle est souvent utilisée lors de la vaccination de routine ou des campagnes, lors de la distribution des MILDA, lors des campagnes de traitement de masse (onchocercose, filariose lymphatique , trachome...).
Avant d’élaborer les microplans, les responsables doivent disposer d’autant d’informations que possible sur les populations cibles. Les données démographiques, les rapports des programmes de santé et les cartes actualisées des aires de santé constituent des points de départ dans le processus de microplanification.

Une carte opérationnelle doit comprendre:

Une zone de desserte connue
- Les villages et les communautés couverts par les sites fixes, des sites pour services de proximité (stratégies avancées) ou lors de journées de santé de la mère et de l'enfant.
- Les points (POI) d'intêret importants (points d'eau, écoles, lieux de culte, domiciles de CHW, points de transits de nomades...)
- Les obstacles aux prestations de service (inondation, routes impraticables)
- Les distances et temps de trajet entre sites importants
- Les zones urbaines peuvent utiliser les plans des rues, les cartes satelittaires...)

L'utilisation de ces outils permet de répondre à des questions telles que:

- Quel est le lieu de culte le plus éloigné d'une formation sanitaire spécifique ?
- Quels sont les domiciles de relais communautaires distants de moins de 10 km du centre de santé?
- Quels sont les quartiers à plus de 5 km d'un site de vaccination en fixe ou en stratégie avancé?

Le présent travaille vise à montrer l'importance des systèmes d'information géographique, du mobile health (kobotoolbox) ainsi que les outils de data analyse (python et ses librairies) dans la microplanification.

## Datasets
Cinq (5) fichiers ont été utilisés pour ce travail de microplanification des activités de santé.

Le shapefile des aires du district de santé de YOKO qui comporte les données des 8 aires de santé.
Le fichier shapefile de l'aire de santé de NDJOLE
Le fichier shapefile de toutes les formations sanitaires du district de santé de YOKO
Le fichier excel des points d'intêrets d'une partie de l'aire de NDJOLE. Ces points ont été collectés à partir de l'outil kobocollect.
Le fichier geojson avec les limites administratives des arrondissements du Cameroun.
La projection pour la zone étudiée est : WGS 84 / UTM zone 32N EPSG:32632

## Acquisition, nettoyage et exploration de données.
- Acquerir les données
- Verifier leurs types
- Récupérer leur crs.
- Nettoyer les données
- Visualiser ces données

## Visualisation
## Visualiser plusieurs couches sur le même graphique
## Visualiser avec Folium

## Analyse spatiale
### Jointure spatiale: récupérer les points appartenant exclusivement à une surface
### Zone tampon
### distance
### Superficie
