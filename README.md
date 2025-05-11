# 2024-2025-4GP - Projet Capteur low-tech Graphite - Aliénor Jarry et Bastien Maïs

## Sommaire :
- [Contexte](#contexte)
- [Livrables](#livrables)
- [Matériel nécessaire](#matériel-nécessaire)
- [1. Simulation électronique du capteur sous LTSpice](#1-simulation-électronique-du-capteur-sous-ltspice)
- [2. Design du PCB sous KiCad](#2-design-du-pcb-sous-kicad)
- [3. Réalisation du Shield](#3-réalisation-du-shield)
- [4. Code Arduino](#4-code-arduino)
- [5. Application Android APK sous MIT App Inventor](#5-application-android-apk-sous-mit-app-inventor)
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

De plus, la résistance R1 en entrée protège contre les décharges électrostatiques. Cette résistance en combinaison avec la capacité C5, forme un filtre pour atténuer les bruits de tension. 

**Voici la réponse de notre circuit afin de vérifier que le capteur est bien amplifié** :
![Réponse amplification LTSpice](./Images/amplification_lt_spice.png)
Nous voyons que le signal est amplifié à 1V. Ainsi, l'Arduino UNO pourra le mesurer. 

**Réponse avec un courant alternatif pour vérifier que le bruit est bien filtré** :
![Réponse filtrage LTSpice](./Images/filtrage_lt_spice.png)
Nous remarquons que le bruit est bien atténué à 50 Hz, d'environ 72 dB.

## 2. Design du PCB sous KiCad

Notre PCB a été conçue sur le logigiel Kicad (version 9).

Nous avons tout d'abord réalisé la schématique en reproduisant le circuit transimpédance (en enlevant la partie simulant le bruit). Nous avons  remplacé la résistance R3 par un potentiomètre digital. Cela nous permet d'ajuster le gain de notre AOP en fonctions de nos besoins. Également, nous avons créé les symboles des différents composants/modules intégrés. Ces modules assureront une mesure précise de notre capteur en graphite et permettra de comparer les résultats obtenus.

**Voici le schéma électrique de l'ensemble de notre montage** :
![Schematique ](./Images/Schematique.png)

Ensuite, nous avons réalisé les empreintes de nos composants en prenant en compte leurs caractéristiques techniques : nombre de pins, espacement, dimensions, géométrie,...
afin de les placer sur notre PCB. 

Puis, nous sommes allés dans l'onglet "éditeur de PCB" sous Kicad pour designer notre circuit. Nous nous sommes appuyés sur un modèle de carte Arduino Uno. Nous avons ainsi placé nos différents composants, de manière à regouper les composants du circuit transmpédance. De plus les modules, et les composants du circuit transimpédance devaient être placés proches de leurs branchements Arduino respectifs. Nous sommes ensuite passés à la partie routage du circuit. Notre principale difficulté a été d’optimiser le placement des composants afin de limiter au maximum l’utilisation de vias, notamment pour les connexions au GND. Nous avons tout de même trois vias sur notre PCB. Pour garantir une bonne connexion entre toutes les broches GND des composants, nous avons mis en place un plan de masse.

**Voici le résultat final obtenu de notre routage** : 
![Routage_PCB ](./Images/Routage_PCB.png)

**Voici le rendu 3D de notre PCB, avec ses différents modules et composants intégrés**:
![PCB_3D ](./Images/PCB_3D.png)

Toutes les ressources utilisées pour notre Kicad (empreintes, schéma etc...) sont disponibles dans notre dossier Kicad.

### **Erreurs réalisées** : 

Nous avons fait quelques erreurs concernant les empreintes de nos modules.

**Servomoteur** :
Nous avons inversé les broches GND et 5V. 

**Bluetooth HC-05** : 
L'empreinte de notre Bluetooth n'est pas correcte. Nous avions mis dans l'ordre la broche VCC, GND, TX, RX, ENABLE et STATE alors que la broche ENABLE est la première. Voici les broches correctes du Bluetooth :

<img src="./Images/HC-05.png" alt="HC-05" width="230"/>

Un conseil serait de se baser sur la datasheet de chaques composant afin de réaliser les empreintes! De plus, il faut faire attention concerant les broches TX et RX du Bluetooth. La broche RX de la carte Arduino doit être raccordée à la broche TX du Bluetooth, et la broche TX du Arduino à la broche RX du Bluetooth.
Nous avons résolu le problème des pins Bluetooth en utilisant des connecteur mâles, femelles permettant de brancher correctement le Bluetooth.


## 3. Réalisation du Shield 

Une fois notre PCB finalisé sous KiCad, nous l’avons envoyé à Cathy afin qu’elle puisse vérifier que la PCB était conforme et prête pour une impression correcte. Puis, nous avons généré le masque de gravure de notre PCB que Cathy s’est chargée d’imprimer. Ensuite, nous sommes allées avec Cathy au GEI afin de tirer notre PCB.

Nous n’avons pas pu manipuler directement, mais nous avons tout de même pu observer le processus de fabrication de la carte. Cathy a procédé ainsi :

1. Insolation UV de la plaque en époxy recouverte d'une fine couche de cuivre (destinée à recevoir le circuit imprimé) et d'une résine photosensible, à travers le masque, pour durcir la résine dans les zones exposées.
2. Développement : la plaque est plongée dans un révélateur pour éliminer la résine non exposée à la lumière UV, laissant ainsi un motif de résine durcie correspondant au circuit imprimé.
3. Gravure : La plaque est ensuite immergée dans un bain de perchlorure de fer, qui dissout le cuivre non protégé par la résine durcie, formant ainsi les pistes du circuit imprimé.
4. Nettoyage à l’acétone pour éliminer les résidus de résine restants après la gravure.

Nous tenions à grandement remercier Cathy pour son aide tout au long du projet, et en particulier pour l'impression de notre PCB.

### Assemblage du circuit

Nous avons ensuite procédé au perçage de la plaquette pour permettre l'insertion des différents composants, en veillant à respecter la taille des trous. Un forêt plus petit était nécessaire pour les résistances et les condensateurs. Une fois le perçage effectué, nous avons soudé les composants sur la PCB. Il faut bien faire attention à souder uniquement sur les pastilles, sauf pour les pastilles GND où un débordement est moins problématique, car elles sont toutes reliées au plan de masse. Un excès de soudure sur ces pastilles pourrait en effet créer des courts-circuits. De plus, nous avons dû ajouter un fil reliant la broche 5V du flex sensor au 5V du module Bluetooth, car une portion de cuivre sur la piste les reliant a été retirée accidentellement en tentant d’enlever un surplus de soudure.

Voici une photo de l'assemblage de notre circuit (sans les modules) ainsi qu'une photo de nos soudures :
<p align="center">
  <img src="/Images/assemblage_circuit_front.png" alt="assemblage_PCB" width="150"/>
  <img src="/Images/assemblage_circuit_back.png" alt="soudure" width="150"/>
</p>

## 4. Code Arduino 

## 5. Application Android APK sous MIT App Inventor

## 6. Banc de test
dire que servomoteur n'a finalement pas été utilisé !!

## 7. Résultats

## 8. Datasheet du capteur

## Conclusion

## Contacts
Aliénor Jarry : [ajarry@insa-toulouse.fr](mailto:ajarry@insa-toulouse.fr)  
Bastien Maïs : [mais@insa-toulouse.fr](mailto:mais@insa-toulouse.fr)  
