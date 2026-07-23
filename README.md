# 🏎️ F1TENTH Autonomous Racing Project

Bienvenue sur le dépôt du projet F1TENTH. Ce projet vise à concevoir, assembler et programmer une voiture de course autonome à l'échelle 1/10ème, capable de naviguer à haute vitesse en évitant les obstacles grâce à un LiDAR et des capteurs ultrasons.

## 🛠️ 1. Architecture Matérielle

* **Châssis :** Tamiya TT-02 (Upgradé alu & roulements)
* **Propulsion :** Moteur Brushless Sensored Xerun V10 G4R + Contrôleur **VESC 6 EDU**
* **Calculateur :** Raspberry Pi 5 (16GB) IP Ethernet : 192.168.50.1 IP WIFI: 10.42.0.52  , rapsicar@jerem sudo mdp jerem
* **Alimentation :** Batterie NiMh 7.2V + UBEC Hobbywing 5V/5A (pour alimenter la Pi)
* **Capteurs :**
  * LiDAR 2D Slamtec C1 (360°)
  * Caméra Intel RealSense D435i (Profondeur 3D)
  * 3x Capteurs Ultrasons HC-SR04 (Arrière, Gauche, Droit)

> ⚠️ **ATTENTION MATÉRIEL (CRITIQUE) - RASPBERRY PI 5**
> Les broches GPIO du Raspberry Pi 5 fonctionnent strictement en **3.3V**. Les capteurs HC-SR04 renvoient un signal ECHO en **5V**. 
> **Il est impératif d'utiliser un pont diviseur de tension (1kΩ / 2kΩ)** sur les broches ECHO avant de les relier à la Pi, sous peine de griller la carte mère.

## 🧠 2. Architecture Logicielle (ROS 2 Jazzy)

Le projet repose sur ROS 2 Jazzy et est divisé en plusieurs nœuds clés :

* **`autogap.py` (Navigation) :** Implémente l'algorithme *Follow the Gap*. Il analyse le scan LiDAR, ignore le propre châssis du robot (zone aveugle de 20cm), applique un "Biais Central" pour privilégier la ligne droite, et calcule la commande de direction et de vitesse. Il intègre une machine d'état (`AVANCE`, `FREINAGE`, `RECULE`) pour se dégager des impasses.
* **`e_stop_node.py` (Sécurité) :** Nœud d'arrêt d'urgence. Il agit comme un *Kill Switch* manuel via la manette (Bouton R1) avec la priorité maximale.
* **`ultrasonic_node.py` :** Gère les 3 capteurs HC-SR04 en mode "Round-Robin" (chacun son tour) via la librairie `lgpio` pour éviter les interférences acoustiques et les crashs de threads sur Pi 5.
* **`ackermann_mux` :** Multiplexeur qui gère les priorités (E-Stop > AutoGap > Téléopération).

## ⚙️ 3. Installation et Dépendances

Sur la Raspberry Pi 5 (Ubuntu 24.04 + ROS 2 Jazzy) :

```bash
# 1. Installer les dépendances système
sudo apt update
sudo apt install python3-lgpio ros-jazzy-serial-driver ros-jazzy-slam-toolbox ros-jazzy-nav2-map-server ros-jazzy-realsense2-camera

# 2. Cloner le dépôt
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
git clone [URL_DE_CE_DEPOT] .

# 3. Compiler le workspace (Utiliser symlink pour le développement Python)
cd ~/ros2_ws
colcon build --symlink-install
source install/setup.bash
