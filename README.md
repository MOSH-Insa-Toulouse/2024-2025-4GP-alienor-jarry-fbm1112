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

Notre capteur graphite présente une résistance variable de l’ordre du gigaohm(GΩ). Lorsque une tension de 5 V est appliquée à ses bornes, un courant extrêmement faible est généré, de l’ordre de 100 nA en moyenne.

Un tel signal est difficilement exploitable sans amplification. Pour y remédier, nous avons utilisé un montage transimpédance basé sur un amplificateur opérationnel (AOP). Ce montage permet de convertir ce courant en une tension suffisamment élevée pour être lue par le convertisseur analogique-numérique (ADC) d’une carte Arduino UNO. 

**Nous avons simulé ce montage à l’aide du logiciel LTspice, voici notre schéma du circuit analogique** : 
![Schéma LTSpice](./Images/schema_lt_spice.png)

Nous avons choisi l'AOP LTC1050 car il est adapté pour traiter de très faibles courants d'entrée. Son faible offset de tension assure une conversion précise du courant en tension.

**Deux éléments de simulation sont intégrés au circuit** :

- 🟥 Rectangle rouge : Simulation du capteur  
- 🟪 Rectangle violet : Simulation du bruit

De plus, des filtres ont été ajoutés au montage afin d'atténuer les perturbations indésirables (par ex : bruits d'alimentation à 50 Hz) 

**Trois filtres assurant le traitement du signal** :

- 🟩 Filtre en entrée de l'AOP(C1,R2) :  
 C'est un filtre passe-bas passif de fréquence de coupure fc = 16 Hz. Il filtre les bruits en courant sur le signal d'entrée.

- 🟦Filtre couplé à l'AOP (C2, R4) :  
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
