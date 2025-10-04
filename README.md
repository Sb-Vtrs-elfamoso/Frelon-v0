# Frelon-v0
## 1. Méthodologie
Dans le cadre du projet de protection des attaques de frelons sur les ruches d'abeilles, la partie de détection est un enjeu majeur :
* Le **temps de détection** est composé du  temps d'inférence du modèle IA mais aussi de tout le code permettant la détection. Compte-tenu de la complexité des modèles, il peut être approximé que temps de détection = temps d'inférence. Ce temps de détection doit être optimisé : si le temps de détection est trop long (>50 ms), en ajoutant le temps de ciblage, le système ne pourra jamais atteindre sa cible à cause d'une latence trop élevée.
* La **précision de détection** correspond à la capacité du modèle à trouver les frelons de manière exhaustive et à la bonne possition. Il faut la maximiser, car sans cette précision, le système aura une visée approximative qui ne menacera pas les frelons, voire pire, qui menacera les abeilles.
* Le **taux de faux postifs** (quand il y a une détection mais qui en réalité n'en est pas une (comme le fait de détecter un frelon alors que c'est une abeille!)) est aussi à controler et à minimiser, notamment pour ne pas menacer les abeilles. En effet, si le système devient lui même une menace pour les abeilles, il devient totalement inutile.

Pour cela, l'étude suivante à été mise en place : évaluer la performance (temps d'inférence / précision / taux de faux positifs) de différents modèles en simulant leur exécution sur **STM32N6** pour le temps d'exécution et en analysant la précision sur un dataset créé "à la mano". Cela permet ainsi d'avoir une base solide pour la suite du projet.

Par ailleurs, afin de pouvoir de faciliter la suite, le dataset a été créé en externe sur Roboflow (cf. section Dataset), permettant de mettre à jour à l'avenir les données. Les modèles aussi peuvent être changés (avec quelques modifications mineures de code). Et le benchmark a été réalisé sur la plateforme online de ST Edge AI (une implémentation en locale du framework a été essayée sans succès).

Ce repository est donc la brique "Training" dans la réalisation du projet de protection contre les frelons.

---

A Noter : L'analyse du temps d'exécution et de précision/taux de faux positifs sont pour le moment totalement décorélées car de nombreux points sont encore à travailler : quelle est la robustesse avec une caméra réelle, selon la météo, sans le fond blanc ? Qui nécessiteront à l'avenir du travail plus précisemment sur le dataset.

---

### 1. 1. Dataset
#### 1. 1. 1. Caractéristiques
Le Dataset sert à l'entrainement, le test et la validation du modèle. Provenant d'images filmées par vision en plongée sur le devant de la ruche. Afin d'améliorer les preformances dans un premier temps, un fond blanc a été posé sous la zone de détection. Il y a ainsi moins de difficultés de détection liées au background.

Par ailleurs, le dataset a été développé dans un but de détection de frelons, mais aussi d'abeilles. Cepedendant, la labellisation d'abeilles n'est pas systématique et rigoureuse. Il n'est donc pas fiable d'entrainer un modèle sur la catégorie 'abeille'.

Dans un premier temps, il a été choisi de ne pas toucher à ce dataset afin de pouvoir mettre en place une première approche fiable qui permetra par la suite d'améliorer la qualité et la diversité des données et par conséquent, la performance et la robustesse des modèles.

Dataset disponible et modifiable sur [Roboflow](https://app.roboflow.com/frelonproject/hornet-bees-6bl4k/3)

#### 1. 1. 2. Pre-Processing
Avant d'être traité dans les notebooks présents dans ce repo, le dataset a été prétraité : 
* Resize to 480x480 (Stretch) qui sera la définition maximale traitée par les algorithmes.

---

A Noter : La réduction de la taille des images en input des modèles permet de réduire les temps d'exécution mais cela réduit aussi la précision des modèles.

---

#### 1. 1. 3. Augmentations
Ensuite, afin d'améliorer la robustesse et la qualité de la détection dans un environnement réel. Des augmentations ont été apportées (Ce sont des modifications des images qui permettent d'aggrandir artificiellement le dataset et de lui donner plus de diversité) :
* Changements d'orientations
  * 50% probability of horizontal flip
  * 50% probability of vertical flip
  * Equal probability of one of the following 90-degree rotations: none, clockwise, counter-clockwise, upside-down
  * Random rotation of between -15 and +15 degrees
* Déformations
  * Random shear of between -14° to +14° horizontally and -14° to +14° vertically
* Changements de rendus visuels (comme pourrait le faire la météo)
  * Random brigthness adjustment of between -23 and +23 percent
  * Random exposure adjustment of between -15 and +15 percent
* Application d'effets (comme pourraient le faire la vitesse de déplacement ou des parasites dans les images de la caméra)
  * Random Gaussian blur of between 0 and 2 pixels
  * Salt and pepper noise was applied to 1.76 percent of pixels

### 1. 2. Modèles
Suite à ces Augmentations les modèles peuvent être entrainés. Afin de choisir des candidats potentiels au niveau des modèles, des modèles déjà testés pour la détection d'objets par ST sont dispnibles sur [HuggingFace](https://huggingface.co/STMicroelectronics) et sur [Github](https://github.com/STMicroelectronics/stm32ai-modelzoo/tree/main/object_detection). Parmis ceux-ci on peut retrouver :
* ssd mobilenet v2
* st mobilenet v1
* st yolo lc v1
* st yolo x
* tiny yolo v2
* yolo v8n
* blazeface front
* yolo 11n


Parmis les benchmark, on peut retrouver les temps d'inférences pour chaques modèles sur la plateforme visée : STM32N6570-DK. En filtrant par le temps maximal visé de 50ms et en cherchant à conserver la définition maximale, on obtient les modèles suivants :
* [SSD Mobilenet v2 0.35 FPN-lite](https://github.com/STMicroelectronics/stm32ai-modelzoo/blob/main/object_detection/ssd_mobilenet_v2_fpnlite/README.md) Int 8 en résolution 256x256x3
* [ST SSD Mobilenet v1 0.25](https://github.com/STMicroelectronics/stm32ai-modelzoo/blob/main/object_detection/st_ssd_mobilenet_v1/README.md) Int 8 en résolution 256x256x3
* [st_yolo_lc_v1](https://github.com/STMicroelectronics/stm32ai-modelzoo/blob/main/object_detection/st_yolo_lc_v1/README.md) Int 8 en résolution 256x256x3
* [st_yolo_x_nano](https://github.com/STMicroelectronics/stm32ai-modelzoo/blob/main/object_detection/st_yolo_x/README.md) Int 8 en résolution 480x480x3
* [tiny_yolo_v2](https://github.com/STMicroelectronics/stm32ai-modelzoo/blob/main/object_detection/tiny_yolo_v2/README.md) Int 8 en résolution 224x224x3

Dans un premier temps, **2 modèles sont sélectionnés** :
* [ST SSD Mobilenet v1 0.25](https://github.com/STMicroelectronics/stm32ai-modelzoo/blob/main/object_detection/st_ssd_mobilenet_v1/README.md) Int 8 en résolution 256x256x3, pour sa rapidité d'exécution (inférence en 11 ms).
* [st_yolo_x_nano](https://github.com/STMicroelectronics/stm32ai-modelzoo/blob/main/object_detection/st_yolo_x/README.md) Int 8 en résolution 480x480x3, pour sa capacité à traiter des images avec une meilleure résolution donc avec une meilleure précision possiblement.

---

A Noter : Le temps d'inférence ne dépend pas de l'entraiment du modèles, mais de l'architecture de celui-ci. Certaines optimisations architecturales,qui influencent le temps d'inférence, peuvent dépendre des poids et donc de l'entrainement mais cela reste marginal et ne sera pas étudié ici. 

---

### 1. 3. Entrainement
#### Fine-Tuning

##### Base Person
###############################################

Pour cela, les modèles .h5 sont récupérés depuis le model zoo, ils sont fine tunes sur la base coco.

##############################################

##### Base Image Net

Ces modèles sont, dans un premier temps, fine-tunés à partir de leur entrainement sur [ImageNet](https://www.image-net.org/) (Dataset de détection d'objets sur images) qui contient de nombreuses classes et d'images d'insectes. (La qualité des données pour les insectes est discutable comme le montre cet [article montrant la mauvaise représentation de l'ensemble de la biodiversité](https://arxiv.org/abs/2208.11695))

###############################################

Pour cela, les modèles .h5 sont récupérés depuis le model zoo, ils sont entrainés sur imagenet (seulement classes liées aux insectes).
Ensuite, ils sont fine tunes sur la base coco de frelons.

##############################################

#### Entrainement from scratch
###############################################

Pour cela, les modèles .h5 sont récupérés depuis le model zoo, ils sont entrainés sur la base coco de frelons.

##############################################


### Benchmark sur ST Edge AI

### Pour continuer
#### Inférence sur STM32N6
#### Amélioration du Dataset

## Conda Environnement `ml`

Ce projet utilise un environnement Conda avec Python 3.10 et plusieurs bibliothèques pour
l'entraînement, l'évaluation et le benchmark de modèles de détection.

---

### Installation de l'environnement

#### 1. Créer l'environnement à partir du fichier `environment.yml`

```
conda env create -f environment.yml
```
#### 2. Activer l'environnement

```
conda activate ml
conda install -c conda-forge cmake ninja git
```
#### 3. Ajouter le noyau Jupyter

```
python -m ipykernel install --user --name=ml --display-name "Python (ml)"
```
#### 4. Choisir ce noyau pour executer les scripts des notebooks
