# Gazebo Projet Programmation Robotique (Gazebo-ROS)
## Introduction

Dans le cadre du cours de programmation robotique, on a été amené à réaliser un projet sous Gazebo-ROS. L’objectif était de simuler le comportement d’un robot dans un environnement de notre choix, dans lequel le robot devrait interagir avec l’environnement (bouger, naviguer et éviter les obstacles). Ceci en 3D en utilisant le langage XML et python avec ROS melodic et GAZEBO & RVIZ. 
Dans ce Readme nous allons vous exposer la mise en œuvre du projet, puis ses fonctionnalités ainsi que les difficultés rencontrées lors de la réalisation de celui-ci.
## Mise en œuvre du Projet 
### **Création du Projet**
Dans ce projet nous avons respecté un arbre de création bien précis en fonction de chaque partie du projet.

Image de l'arbre de création 
![](https://i.imgur.com/orKsKug.png)

 
#### *Organisation du Projet*
Dans l'organisation de ce projet nous avons commencé par créer un dossier cours_ws dans lequel nous avons créé quatres sous dossier my_robot_description; my_robot_gazebo; my_robot_navigation et my_robot_control. Chacun d'eux contiendra des répertoires avec les différents code composant le projet. image de l'arborescence juste au-dessus.

Pour commencer nous avons créé un modéle de robot.

### **Création du Robot**
#### *Méthode URDF*

Pour créer un robot sur ROS-Gazebo, il est imperatif de faire d'abord sa description, et pour ce faire on a récours au fichier URDF (Universal Robot Description File).
Voici un schéma cinématique de notre robot en vue de dessus.
![](https://i.imgur.com/HZOCtJ9.png)

Pour créer le corps du robot, on a : 
    
-créer un package "my robot description" dansélequel on a mis notre fichier 01-                      description.urdf, qui défini le robot (son nom,        le corps du robot,l'afficheur du robot, etc.)

-ensuite on crée un fichier "display.launch" dans      lequel on precise le fichier URDF à charger pour      l'afficher, on y met aussi des sliders permettant de faire bouger les joints du robot. Puis on execute la ligne de commande;
```
roslaunch my_robot_description display.launch
```
ce qui affiche une fenetre vide; 
![](https://i.imgur.com/e5R3G1s.png)

Dans la fênetre on ajoute base_link et ça affiche la forme de notre robot. 

![](https://i.imgur.com/NQlE5HR.png)

#### *Méthode XACRO*

Une fois qu'on a un modèle URDF, le robot commence à avoir des éléments de plus en plus complexes, on ne peut donc plus coder manuellement, pour cela on a recours au macros XACRO. Un des avantages du XACRO est qu'il nous permet de déclarer des variables qui nous facilitera la tâche si l'on veut changer les dimensions d'un ou plusieurs composants du robot.
Le XACRO lis le URDF, et il est appellé ainsi: 
```
$ x ac r o −−i n o r d e r 02-description.xacro  > model . u r d f
```
la ligne de commande pour lancer le modèle robot.xacro est la même qu'au dessus juste il faut changer le nom du dossier que nous souhaitons ouvrir dans le display.launch
C'est grâce à cette ligne de commande que l'on choisi le fichier à ouvrir 
```
<arg name="model" default="$(find my_robot_description)/urdf/02-description.xacro"/>
```
Modèle avec xacro:
![](https://i.imgur.com/ZIcNQ4P.png)




### **Création du World**
Pour toute simulation de robot, il est préférable de faire les tests dans les conditions les plus proche de la réalite. C'est pour cela que nous avons besoin d'un world.
Pour commencer il y la méthode simple pour créer ce world 
#### *Méthode Simple*
Notre world peut se composer d'un soleil est d'un plan "ground_plane"
Code disponible juste dessous:
```
<?xml version="1.0"?>
<sdf version='1.6'>
    <world name='default'>
        <include>
            <uri>model://sun</uri>
        </include>
        <model name='ground_plane'>
            <static>1</static>
            <link name='link'>
                <collision name='collision'>
                    <geometry>
                        <plane>
                            <normal>0 0 1</normal>
                            <size>15 15</size>
                        </plane>
                    </geometry>
                    <surface>
                        <friction>
                            <ode>
                                <mu>100</mu>
                                <mu2>50</mu2>
                            </ode>
                        </friction>
                        <bounce/>
                        <contact>
                            <ode/>
                        </contact>
                    </surface>
                    <max_contacts>10</max_contacts>
                </collision>
                <visual name='visual'>
                    <cast_shadows>0</cast_shadows>
                    <geometry>
                        <plane>
                            <normal>0 0 1</normal>
                            <size>40 40</size>
                        </plane>
                    </geometry>
                    <material>
                        <script>
                            <uri>file://media/materials/scripts/gazebo.material</uri>
                            <name>Gazebo/WoodFloor</name>
                        </script>
                    </material>
                </visual>
                <velocity_decay>
                    <linear>0</linear>
                    <angular>0</angular>
                </velocity_decay>
                <self_collide>0</self_collide>
                <kinematic>0</kinematic>
                <gravity>1</gravity>
            </link>
        </model>
    </world>
</sdf>
```
On obtient le résultat sur l'image avec le code du dessus est aprés avoir tapé les commandes suivantes.

La première sert à ajouter le chemin du fichier world 
```
export GAZEBO_RESOURCE_PATH=~/cours_ws/src/my_robot/my_robot_gazebo/world/:$GAZEBO_RESOURCE_PATH
```
La seconde sert a ouvrir le World dans Gazebo
```
roslaunch my_robot_gazebo world.launch
```
Image du monde créé:

![](https://i.imgur.com/Dx4enaB.png)

Avec le world de base que nous venons de créer, je peux en faire ce que je veux comme rajouter des obstacles genérés aléatoirement. 
#### *Ajout D'Obstacles*
Pour ajouter des obstacles nous avons choisi la méthode ou nous avons créé un programme qui génére le nombre d'obstacle que nous voulons.
Code disponible en cliquant sur le lien ci-dessous.

[Génerateur d'Obstacle](https://colab.research.google.com/drive/1tZZCMmAjYU4CRFgQJMlaLibpEjnBEvin)

Une fois les obstacles génerés, on les ajoute à notre world.
Pour voir le résultat on utilise les commandes données juste au dessus, on obtient ceci; 

![](https://i.imgur.com/Xh9OHLd.png)

Nous avons pour finir ajouter des objets maillés à notre world

#### *Ajout D'Objets maillés*
Pour avoir un environement satisfaisant, on a ajouté aux géométries canoniques des maillages surfaciques texturés.Pour cela nous avons utilisé la même méthode que pour les objets canoniques vue au dessus 

[Générateur d'objets maillé](https://colab.research.google.com/drive/19Q-QYqiCwub351cEKuQuujDepv4k78kM?usp=sharing)

Résultat obtenu aprés avoir ajouter les objets maillés à notre monde. 

![](https://i.imgur.com/cPGNuuT.png)

![](https://i.imgur.com/AY2Ero7.png)

### *Création de l'environnement de simulation*

Pour avoir un environnement de simulation sous ROS dans GAZEBO, on a utilisé le format SDF (Standard Data Format). Elle encapsule une description complète du world comprenant le modèle, la scène, le physique et les plugins. 

### *Ajout des plugins* 
Les plugins nous permettent d'enrichir notre modèle URDF en simulant le fonctionnement de notre robot avec 
des capteurs et d'actionneurs (qui on les mêmes comportements visuels ou physiques que ceux
normalement renvoyés par de vrais capteurs) comme dans la réalité. 

Nous avons ajouté la caméra et un laserscan (une sorte de lidar) tout deux sont des plugins du type sensor 
La caméra offre de nombreux paramètres, portant sur la résolution de l'image, la fréquence
de rafraîchissement, profondeur de champ et le bruit du signal.
Quant au laserscan, il permet la simulation de capteurs de rayons dans Gazebo et faire la cartographie de l'environnement.
### *Lancement de la simulation*
Enfin une fois que nous avons fait toutes les étapes expliquées au-dessus. Nous avons lancé notre environnement de simulation avec les commandes suivantes.
Pour commencer si cela n'a pas été fait avant, il faudra sourcer avec la ligne de commande suivante 
```
source devel/setup.bash
```
Ensuite pour obtenir l'environnement avec notre monde il faut lui tranférer le dossier du world dans le registre où les mondes sont enregitrés. Cette tâche se fait avec la ligne de code suivante
```
export GAZEBO_RESOURCE_PATH=~/cours_ws/src/my_robot/my_robot_gazebo/world/:$GAZEBO_RESOURCE_PATH
```
Pour finir on lance notre environnement de simulation avec la commande suivante 
```
roslaunch my_robot_gazebo demo_gazebo.launch 
```
Une fois cette commande tapée, l'ouverture peut mettre un peu de temps une fois gazebo ouvert on obtient cela.
image ci dessous 

![](https://i.imgur.com/TU72Pi2.png)

## Les objectifs finals de ce projet
Les objectifs finals sont de créer une carte de notre monde afin de faire naviguer en autonomie notre robot dans le monde d'un point A à B.

### Difficultés rencontrées lors de la réalisasion
Pendant ce projet nous n'avons pas rencontré de problémes particuliers jusqu'à ce que nous essayons
de réaliser les objectifs du dessus. Le plus gros probléme que nous avons rencontré, en plus des erreurs, s'est la limite de nos machines car nous avons trés vite été confronté à une chute du real time factor qui est passé de 0,64 a 0,06 au moment du lancement de rviz en plus de l'environnement de simulation.Ce problème s'est produit sur nos deux machines ce qui nous a empêché d'aller plus loin.

# Conclusion sur le projet 
A travers ce projet nous avons pu voir comment nous pouvons simuler des robots sous gazebo à l'aide de ROS,avec un monde qui ce rapproche le plus de ce que notre robot rencontrera dans la réalité. cette expérience nous a permis d'obtenir des nouvelles compétences qui nous seront utiles dans le futur.  
Malgré les dificultés que nous avons rencontrées avec nos machines nous avons persité à réaliser la plupart des objectifs attendus. Nous pensons qu'avec des machines plus puissantes nous aurions pu réaliser la navigation et la cartographie car dans notre programme il y a tous pour le réaliser. 