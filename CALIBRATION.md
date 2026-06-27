# Calibration

## Calibrer la caméra

Cette étape est à faire une seule fois (sauf si on change de caméra ou d'objectif). Elle nécessite un damier imprimé de 8x11 cases (7x10 coins intérieurs), carreaux de 20 mm.

Ce document vous permetra d'avoir la procédure à suivre afin de réaliser une nouvelle calibration.

### 1. Création d'un damier de calibration

Pour réaliser une calibration, il est important d'avoir un damier de calibration, en temps normale nous utilisions un feuille coller sur une planche de bois.
Cependant, nous avons décider de réalisé une modélisation 3D de notre damier de calibration avant de l'imprimer en 3D. Afin d'avoir un outillage dédier à la calibration, l'avantage est de pouvoir avoir une grande certitude quant à la tailles de chaque careaux.

<img width="320" height="240" alt="Damier" src="https://github.com/user-attachments/assets/4420814c-de11-4c93-8367-a072344f585c"  />

Vous retroueverez au besoin le damier dans le bureau de M. Blazevic.

### 2. Acquisition des images de calibration

Une fois le que vous avez le damier, la prochaine étape est de réaliser l'acquisition d'une trentaine d'image de notre damier afin de redrésser l'image qui est courbé aux extrémitées.

Pour cela je vous conseil de détacher la caméra du plafond afin de facilité les déplacement du damier, celui-ci devant être proche de la caméra pour remplir au maximum le champ de vision de la caméra.

Puis vous devrais depuis la VM ROS2 lancé le programme capture.py (/home/ros2/aruco_ws/src/aruco_detection/scripts).
Une fois le programme lancé celui-ci vous prosera de réaliser les différentes capture d'images. Afin de réaliser une bonne calibration, il faut que l'ensemble du damier soit visible par la caméra, le damier doit recouvrir une grande parti du champ de vision (environ 70%). Veillez à ne pas capturer plusieurs fois 2 images, trop similaire, faites varier l'orientatin du damier sur les trois axes, déplacer le damier au diffrents coin de l'image (les plus grandes déformation ayant lieux aux extrémitées).

<img width="75%" height="auto" alt="image" src="https://github.com/user-attachments/assets/57d6b571-7eca-4261-88a8-98bc0b208dba" />




### 3. Génération du fichier YAML

Une fois la capture des images réalisée, vous devrez générer le fichier YAML contenant les coefficients de corrections de la caméra, pour cela éxecuter le programme correction.py (/home/ros2/aruco_ws/src/aruco_detection/scripts).

Une fois le fichier correctement générer, je vous invite à vérifier que la caméra est correctement redresser (vous pourrez utiliser le programme image_droite.py /home/ros2/aruco_ws/src/aruco_detection/scripts).

<img width="45%" height="auto" alt="Courbe" src="https://github.com/user-attachments/assets/5b76cf54-491e-470d-bb9f-12b3a4f6b357" />
<img width="45%" height="auto" alt="Droit" src="https://github.com/user-attachments/assets/d4728d1c-9d60-45d3-891f-fd749803c96a" />


Vous verrez que chaque bord de l'image redresser affichera des zones mortes, c'est parfaitement normale, c'est pour cela que nous avons dans notre programme de reconnaissance de formes les bords de l'images sont dans un 1er temps coupé afin de ne pas prendre en compte ces zones mortes.

Malgré cela si l'image n'est pas correctement redresser (on observe que les lignes droites comme les bords d'une table apparaissent courbés), vous deverez refaire les étapes 2 et 3.

Si vous avez le moindre doute concernant la calibration, je vous invite à lire la procédure rédiger par M. Blazevic, vous la trouverez dans la VM ROS2 (./home/ros2/Documents/Calibration_camera.rtf).

Le but est de générer un fichier YAML contenant la matrice intrinsèque de la caméra (focale, centre optique) et ses coefficients de distorsion. Ce fichier est ensuite chargé par l'application de vision.

Le format attendu du YAML :

```yaml
camera_matrix:
  - [fx, 0, cx]
  - [0, fy, cy]
  - [0, 0, 1]
dist_coeff:
  - [k1, k2, p1, p2, k3]
image_width: 1280
image_height: 720
```

Le fichier doit être placé à l'emplacement configuré dans `src/config.py` (par défaut : `../scripts/calibration_data_save/calibration.yaml`). Si le chemin change, modifier la constante `CALIB_YAML` dans `config.py`.

---

## Calibrer la zone de travail (homographie)

Si la caméra n'est pas parfaitement perpendiculaire à la table, les coordonnées pixel ne correspondent pas à une vue de dessus. La calibration table corrige ça.

1. Cliquer sur **CALIBRER TABLE**
2. La preview passe en mode calibration
3. Cliquer les 4 coins de la zone de travail dans l'ordre : haut-gauche, haut-droite, bas-droite, bas-gauche
4. Au 4e clic, l'homographie est calculée automatiquement et sauvegardée dans `table_corners.json`
5. Le statut affiche "Table calibrée : L x H px"

Une fois calibrée, tous les centroïdes sont transformés dans le repère de la table via `PerspectiveManager.transform_point()`.
