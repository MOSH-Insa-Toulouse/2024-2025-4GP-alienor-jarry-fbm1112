# 2024-2025-4GP - Projet Capteur low-tech Graphite - Aliénor Jarry et Bastien Maïs

## Sommaire :
- [Contexte](#contexte)
- [Livrables](#livrables)
- [Matériel nécessaire](#matériel-nécessaire)
- [1. Simulation électronique du capteur sous LTSpice](#1-simulation-électronique-du-capteur-sous-ltspice)
- [2. Design du PCB sous KiCad](#2-design-du-pcb-sous-kicad)
- [3. Code Arduino](#3-code-arduino)
- [4. Application Android APK sous MIT App Inventor](#4-application-android-apk-sous-mit-app-inventor)
- [5. Réalisation du Shield](#5-réalisation-du-shield)
- [6. Banc de test](#6-banc-de-test)
- [7. Résultats](#7-résultats)
- [8. Datasheet du capteur](#8-datasheet-du-capteur)
- [Conclusion](#conclusion)
- [Contacts](#contacts)

## Contexte
Dans le cadre de l'UF "Du capteur au banc de test en open source hardware" se déroulant lors de notre 4ème année Génie Physique à l'INSA de Toulouse, nous avons développé un capteur à jauge de contrainte low-tech à base de crayon graphite. Ce projet repose sur le rapport scientfique "Pencil Drawn Strain Gauges and Chemiresistors on Paper", publié en 2014 par Cheng-Wei Lin*, Zhibo Zhao*, Jaemyung Kim & Jiaxing Huang.

Le capteur se compose uniquement d'un morceau de papier sur laquelle une trace de crayon à papier dépose une fine couche de graphite. Un dépôt de graphite sur du papier constitue un système granulaire, où des grains de tailles variables sont séparés par une distance appelée distance inter-grain. La conductance du système varie de manière exponentielle en fonction de cette ditance, laquelle peut être ajustée en déformant le papier. Ainsi, la contrainte exercée sur le matériau peut être déterminée en mesurant simplement la résistance aux bornes du capteur.

L'objectif est de concevoir un dispositif permettant la mesure de déformation à partir d'un capteur low-tech, puis d'évaluer ses performances en le comparant à un capteur commercial.

## Livrables
- Un Shield PCB branchée sur une carte Arduino UNO, intégrant un capteur graphite, un amplificateur de transimpédance et un module Bluetooth. Idéalement, il incluerait également un écran OLED, un encodeur rotatif, un potentiomètre numérique (remplaçant la résistance R2 dans le circuit amplificateur), ainsi qu'un connecteur pour un servomoteur et un capteur de flexion (afin de comparer ses mesures avec celles du capteur en graphite).
- Un code Arduino garantissant la gestion de tous les composants et l'acquisition des mesures (mesures de contrainte, échanges via Bluetooth, affichage OLED, encodeur rotatif, potentiomètre numérique et servomoteur).
- Une application APK Android qui gère l'interface entre le PCB et le code Arduino. L'application doit au minimum afficher la valeur de la résistance de la jauge de contrainte ainsi que sa variation relative sur le smartphone.
- Un code Arduino dédié au banc de test pour la mesure de la jauge de contrainte.
- Une datasheet du capteur en graphite (incluant le circuit de transimpédance).

## Matériel nécessaire
Pour concevoir notre dispositif électronique, voici la liste des composants nécessaires :
- Des résistances : 1 de 1 kΩ, 1 de 10 kΩ, 2 de 100 kΩ pour l'amplificateur transimpédance. 1 de 47 kΩ pour le flex sensor (capteur commercial)
- Des condensateurs : 2 de 100 nF et 1 de 1 μF pour l'amplificateur transimpédance. 1 de 100 nF pour l'encodeur rotatif
- Un amplificateur opérationnel LTC1050
- Un potentiomère numérique MCP41050
- Une carte Arduino UNO
- Un module Bluetooth HC05
- Un écran OLED 128×64
- Un encodeur rotatif
- Un servomoteur
- Un flex sensor
- Un morceau de papier et crayon pour la fabrication du capteur graphite

## 1. Simulation électronique du capteur sous LTSpice

Notre capteur graphite présente une résistance variable de l’ordre du gigaohm (GΩ). Lorsque une tension de 5V est appliquée à ses bornes, un courant extrêmement faible est généré, de l’ordre de 100 nA en moyenne.

Un tel signal est difficilement exploitable sans amplification. Pour y remédier, nous avons utilisé un montage transimpédance basé sur un amplificateur opérationnel (AOP). Ce montage permet de convertir ce courant en une tension suffisamment élevée pour être lue par le convertisseur analogique-numérique (ADC) d’une carte Arduino UNO. 

**Nous avons simulé ce montage à l’aide du logiciel LTspice, voici notre schéma du circuit analogique** : 
![Schéma LTSpice](./Images/schema_lt_spice.png)

Nous avons choisi l'AOP LTC1050 car il est adapté pour traiter de très faibles courants d'entrée. Son faible offset de tension assure une conversion précise du courant en tension.

**Deux éléments de simulation sont intégrés au circuit** :

- 🟥 Simulation du capteur  
- 🟪 Simulation du bruit

De plus, des filtres ont été ajoutés au montage afin d'atténuer les perturbations indésirables (par ex : bruits d'alimentation à 50 Hz) 

**Trois filtres assurant le traitement du signal** :

- 🟩 Filtre en entrée de l'AOP(R2, C1) :  
 C'est un filtre passe-bas passif de fréquence de coupure fc = 16 Hz. Il filtre les bruits en courant sur le signal d'entrée.

- 🟦Filtre couplé à l'AOP (R4, C2) :  
  C'est un filtre passe-bas actif avec fc = 1,6 Hz. Il filtre la composante du bruit à 50Hz du réseau électrique.

- 🟨 Filtre en sortie de l'AOP (R5, C4) :  
  C'est un filtre passe-bas passif avec fc = 1,6 kHz. Il élimine les parasites générés lors du traitement du signal.

De plus, la résistance R1 en entrée protège contre les décharges électrostatiques. Cette résistance en combinaison avec la capacité C5, forme un filtre pour atténuer les bruits de tension. La résistance R3 sera remplacé plus tard par un potentiomètre digital. Cela nous permettra d'ajuster le gain de notre AOP en fonctions de nos besoins.

**Voici la réponse de notre circuit afin de vérifier que le capteur est bien amplifié** :
![Réponse amplification LTSpice](./Images/amplification_lt_spice.png)
Nous voyons que le signal est amplifié à 1V. Ainsi, l'Arduino UNO pourra le mesurer. 

**Réponse avec un courant alternatif pour vérifier que le bruit est bien filtré** :
![Réponse filtrage LTSpice](./Images/filtrage_lt_spice.png)
Nous remarquons que le bruit est bien atténué à 50 Hz, d'environ 72 dB.

## 2. Design du PCB sous KiCad
paul et niels : 
      Afin de réaliser notre PCB, nous avons reproduit le circuit précédent sur Kicad 7.0. Nous avons remplacé la résistance R2 par un potentiomètre numérique afin de pouvoir faire varier le gain de notre AOP. Egalement, nous avons rajouté divers composants afin de pouvoir mesurer efficacement notre capteur graphite et comparé les résultats obtenus :

un flexsensor servant de témoin, afin de pouvoir comparer nos mesures avec celle du capteur en graphite
un module bluetooth HC-05 afin de pouvoir communiquer avec notre circuit depuis notre téléphone depuis une application mobile que nous coderons nous-même.
un écran OLED ainsi que trois boutons poussoirs afin de pouvoir visualiser le résultats de nos mesures et pouvoir naviguer simplement dans les différents menus permettant diverses mesures
      Tous nos composants seront installés sur un shield d'Arduino UNO.

      Nous avons commencé par réaliser les symboles des différents composants et reproduire le schéma électrique complet sur Kicad. Voici le schéma électrique de l'ensemble de notre montage :
Nous avons par la suite réalisé les empreintes de nos composants afin de les placer sur notre PCB. Notre difficulté principale a été de placer les composants de sorte qu'il n'y ait pas de via, notamment pour le GND. Voici le résultat final :
Et voici le rendu 3D que nous obtenu avec ces routages :
Toutes les ressources utilisées pour notre Kicad (empreintes, schéma etc...) sont disponibles dans notre dossier Kicad.


maelys : 
Cette étape du projet avait pour objectif de concevoir le PCB du circuit transimpédance à l’aide du logiciel KiCad, en s’appuyant sur un template de carte Arduino Uno. Plusieurs étapes ont été nécessaires :

Création de la schématique du circuit transimpédance, incluant la définition de symboles personnalisés pour les composants absents de la bibliothèque KiCad.
Conception des empreintes physiques de ces composants, en prenant en compte leurs caractéristiques techniques : nombre de pins, espacement, dimensions, géométrie, etc.
Routage du circuit généré via la vue schématique.
Mise en place d’un plan de masse pour relier efficacement les pistes au GND.
Voici le schéma électrique de l'ensemble de notre montage :
Nous avons conçu les empreintes physiques de nos composants afin de pouvoir les positionner correctement sur le PCB et nous avons fait le routage. La principale difficulté rencontrée a été d’optimiser le placement des composants afin d’éviter l’utilisation de vias, notamment pour les connexions au plan de masse (GND). Objectif réussi nous avons utilisé 0 via !

Voici le résultat final obtenu de notre routage :
Voici la version 3D :

erreurs réalises 
      

## 3. Réalisation du Shield 

## 4. Code Arduino 

## 5. Application Android APK sous MIT App Inventor

## 6. Banc de test

## 7. Résultats

## 8. Datasheet du capteur

## Conclusion

## Contacts
Aliénor Jarry : [ajarry@insa-toulouse.fr](mailto:ajarry@insa-toulouse.fr)  
Bastien Maïs : [mais@insa-toulouse.fr](mailto:mais@insa-toulouse.fr)  
