# Guide de démarrage rapide de Zephyr sur Aarch Linux

> La rédaction et la traduction de cette note a été assisté par IA (Mistral/ChatGPT).

## Introduction
Note : Ce tutoriel s’inspire du [Quick Start Guide officiel de Zéphyr](https://zephyrproject.org/zephyr-os-getting-started-on-manjaro-arch-linux/), mais tente d'aller au-delà.
Ici, vous trouverez :
1. Des explications détaillées sur les paquets (aarch linux) à installer et leur rôle, pour comprendre chaque étape de la configuration de Zéphyr.
2. Un accompagnement pas à pas pour créer un nouveau projet et avec une sensibilisation à l’usage du Device Tree.
3. Une illustration concrète de l’un des atouts majeurs de Zéphyr : sa portabilité exceptionnelle, qui permet de transférer un projet d’une carte à une autre en quelques ajustements seulement.

---
## Prérequis
Vous devez déjà savoir installer et configurer :
- Neovim ou LazyVim
- Kitty
- Zsh
- Git

Pour une utilisation optimal de ce tutoriel, vous devez avoir à disposition :
- [esp32-c3-devkitm-1](https://docs.zephyrproject.org/latest/boards/espressif/esp32c3_devkitm/doc/index.html)
- [stm32-nucleo-f446re](https://www.st.com/en/evaluation-tools/nucleo-f446re.html)

D'autres cartes d'évaluations peuvent être utilisées. Elles demanderont cependant d'éditer des surcouches de devices tree spécifiques sur le projet blink (ce qui est un bon exercise !).

---
## Sommaire
- [[#Dépendances]]
- [[#West]]
- [[#Création d’un espace de travail]]
- [[#Règles USB (udev)]]
- [[#Exemple : faire clignoter une LED ]]   

---

## Dépendances

Installer les paquets nécessaire à Zéphir :
```bash
sudo pacman -S python-pip python-setuptools python-wheel python-pyserial gperf wget curl xz ninja file cmake bison flex gcc dtc openocd arm-none-eabi-gcc arm-none-eabi-gdb patchelf dfu-util gcovr python-anytree python-breathe python-intelhex python-packaging python-ply python-pyaml python-pyelftools python-pkwalify python-tabulate ccache doxygen python-jsonschema
```
*Commande d'installations*

Veuillez trouver une description de chaque paquet installé ci-dessous :

|Paquet|Usage|
|---|---|
|git|Système de gestion de versions distribué.|
|python-pip|Installateur officiel de paquets Python.|
|python-setuptools|Bibliothèque Python facilitant la construction et la distribution de paquets.|
|python-wheel|Extension de setuptools pour exploiter le format binaire **wheel**.|
|python-pyserial|Bibliothèque Python pour la communication série (UART). Requise pour flasher le firmware et interagir avec le matériel.|
|gperf|Générateur de tables de hachage (GNU).|
|wget|Client HTTP/HTTPS/FTP en ligne de commande (GNU).|
|curl|Client de transfert de données (plus flexible que wget, permet également la modification/création de ressources).|
|xz|Outil de compression/décompression sans perte.|
|ninja|Système de build optimisé pour la vitesse (successeur de Make, utilisé souvent avec CMake, Meson ou Gyp).|
|file|Détermine le type d’un fichier.|
|cmake|Outil de génération et gestion de build multiplateforme.|
|bison|Générateur d’analyseurs syntaxiques (GNU).|
|make|Automatisation de la compilation (build system).|
|flex|Générateur d’analyseurs lexicaux (souvent utilisé avec Bison).|
|gcc|Compilateur C/C++ (GNU).|
|dtc|Compilateur de Device Tree.|
|openocd|Outil de débogage matériel (Open On-Chip Debugger).|
|arm-none-eabi-gcc|Compilateur croisé pour processeurs ARM EABI.|
|arm-none-eabi-binutils|Outils binaires pour ARM EABI (assemblage, édition de liens, etc.).|
|arm-none-eabi-gdb|Débogueur GNU pour ARM EABI.|
|patchelf|Permet de modifier les exécutables ELF (ex. RPATH, linker dynamique).|
|dfu-util|Outil de mise à jour de firmware via USB (DFU).|
|gcovr|Générateur de rapports de couverture de code (gcov).|
|python-pytest|Cadre de tests Python.|
|python-anytree|Manipulation d’arbres de données en Python.|
|python-breathe|Intégration de Doxygen dans Sphinx pour la documentation.|
|python-intelhex|Gestion du format Intel HEX (fichiers firmware).|
|python-packaging|Outils de packaging Python.|
|python-ply|Analyse lexicale et syntaxique en Python.|
|python-pyaml|Gestion du format YAML en Python.|
|python-pyelftools|Manipulation de fichiers ELF en Python.|
|python-pkwalify|Validation YAML/JSON en Python.|
|python-tabulate|Génération de tableaux formatés.|
|ccache|Cache de compilation (accélère les builds répétés).|
|doxygen|Générateur de documentation.|
|python-jsonschema|Validation de schémas JSON.|
*Description requis des paquets à installer pour l'usage de Zéphyr*

---
## West

> [!NOTE]  
> West est l’outil **métaprojet** de Zephyr. Il facilite :
> 
> - La gestion des répertoires et dépendances
>     
> - La compilation (native ou croisée)
>     
> - Le flashage du firmware sur la carte
>     
> - Le débogage
>     
> 
> West peut être étendu via des **plugins**.
> 
> - [Dépôt GitHub de West](https://github.com/zephyrproject-rtos/west)
>     
> - [Documentation officielle](https://docs.zephyrproject.org/latest/develop/west/index.html)
>     

### Installation de West
1. Cloner le dépôt AUR :
```bash
cd /tmp
git clone https://aur.archlinux.org/python-west.git
```
2. Construire le paquet :
```bash
cd python-west
makepkg -s
```
3. Installer le paquet :
```bash
sudo pacman -U python-west*.*
```
---

## Création d’un espace de travail

> [!NOTE]  
> L’espace de travail Zephyr centralise tous les fichiers sources, dépendances et révisions de Zephyr OS.

1. Créer un répertoire dans `${HOME}` :
```bash
mkdir ~/zephyr-workspace
```
2. Créer un environnement virtuel Python et l’activer :
```bash
python3 -m venv ~/zephyr-workspace/.venv
source ~/zephyr-workspace/.venv/bin/activate
```
3. Installer West et initialiser l’espace :
```bash
pip install west
west init ~/zephyr-workspace
cd ~/zephyr-workspace
west update
```
Arborescence obtenue :
```
zephyr-workspace
└── zephyr
```
4. Installer les dépendances Python spécifiques à Zephyr :
```bash
west packages pip --install
```
5. Installer le SDK Zephyr :
```bash
west sdk install
```

---
## Règles USB (udev)
Les cartes (MCU/FPGA) nécessitent l’accès à des périphériques USB pour le flash et le débogage.  
Zephyr fournit des règles `udev` prêtes à l’emploi.
1. Télécharger les règles udev :
```bash
wget https://raw.githubusercontent.com/zephyrproject-rtos/openocd/refs/heads/zephyr-20220611/contrib/60-openocd.rules -O 60-openocd.rules
```
2. Déplacer le fichier :
```bash
sudo mv 60-openocd.rules /etc/udev/rules.d/
```
3. Recharger les règles :
```bash
sudo udevadm control --reload
```
4. Si le périphérique est déjà branché :
```bash
sudo udevadm trigger
```
---
## Exemple : faire clignoter une LED
Nous allons compiler et flasher l’exemple `blink` sur une carte **ESP32-C3 DevKitM** ou **STM32 Nucleo-F446RE**.
### 1. Création du projet
1. Créer un dossier projet :
```bash
mkdir -p ~/zephyr-workspace/apps/blink
```
2. Récupérer le dépôt :
```bash
git clone <repo_blink> ~/zephyr-workspace/apps/blink
```
Le dépôt contient :
- `boards/` : fichiers **overlay** (Device Tree) pour chaque carte supportée
- `CMakeLists.txt` : configuration de build
- `prj.conf` : configuration du projet (modules Zephyr activés)
- `src/main.c` : code source de l’application
### 2. Compilation
1. Activer l’environnement virtuel :
```bash
source ~/zephyr-workspace/.venv/bin/activate
```
2. Exporter les variables nécessaires :
```bash
west zephyr-export
```
3. Lister les cartes disponibles :
```bash
west boards | grep esp32
```
4. Compiler pour la carte cible :
```bash
west build -p always -b esp32c3_devkitm blink
```
### 3. Préparation de la carte et initiation au device-tree
Pour rendre le tutoriel plus générique, nous n’utilisons pas la LED intégrée mais une LED externe branchée sur la **GPIO2**.

[Figure 1: Esp32 schéma du montage](./doc/ZephyrEsp32BlinkTutorial.drawio.png)

#### Device Tree utilisé
Le device tree a pour rôle de :
1. configurer la **pin 2** en mode GPIO
2. définir un **alias** qui d’identifie cette pin dans le code
```dtc
/ {
    aliases {
        my-led = &led0;
    };

    leds {
        compatible = "gpio-leds";
        led0: user_led {
            gpios = <&gpio0 2 GPIO_ACTIVE_HIGH>;
        };
    };
};
```
*Device tree utilisé*
#### Points clés
- `my-led` : alias accessible depuis le code
- `led0` : identifiant local dans le device tree
- `user_led` : label purement descriptif
- `gpio0` : contrôleur GPIO
- `2` : numéro de pin utilisé
- `GPIO_ACTIVE_HIGH` : LED active à l’état haut

Voici un extrait du code C qui comprend l'import de la LED à partir de son alias.
```C
#include <zephyr/drivers/gpio.h>
#include <zephyr/kernel.h>

// Settings
static const int32_t sleep_time_ms = 500;
static const struct gpio_dt_spec led =
    GPIO_DT_SPEC_GET(DT_ALIAS(my_led), gpios);
```
_Exemple d’import de `led0` via l’alias `my-led`_

La variable `led` est ensuite manipulée via les fonctions HAL (Hardware Abstraction Layer) de Zephyr :
```C
gpio_is_ready_dt(&led) // Check if GPIO is initialized
gpio_pin_configure_dt(&led, GPIO_OUTPUT); // Set GPIO as output
gpio_pin_set_dt(&led, state); // Set GPIO state
```
*Exemples de fonctions HAL pour le contrôle du GPIO*
### 4. Flash du firmware
1. Installer `esptool` pour ESP32 :
```bash
sudo pacman -S esptool
```
2. Flasher le code :
```bash
west flash
```
### 5. Flash du firmware sur une nucleo-f446re
L’une des grandes forces de Zéphyr réside dans sa **portabilité inégalée** pour vos projets bare-metal. Pour l’illustrer, nous allons porter le projet _blink_ sur une carte **Nucleo-F446RE**, en utilisant cette fois la **LED2 (LD2)** intégrée.

[Figure 2: schéma Nucleo-F446re](./doc/ZephyrNucleoF446re.png)

L’intégration du projet _blink_ sur STM32 avec Zéphyr se limite à la création d’un **device tree overlay** spécifique à la Nucleo-F446RE. Voici son contenu :
```dtc
/ {
    aliases {
        my-led = &green_led_2;
    };
};
```
_Device tree overlay pour la Nucleo-F446RE_

Ce fichier de surcouche (_overlay_) se contente de définir un alias (`my-led`) pointant vers le label `green_led_2`, déjà présent dans le _device tree_ de base de la Nucleo-F446RE.

**Étape supplémentaire** : avant de flasher la carte, assurez-vous d’installer les outils nécessaires pour programmer les microcontrôleurs STM32. Sous Arch Linux, utilisez la commande suivante :
```bash
sudo pacman -S stm32cubeprogrammer
```

Enfin, compilez et flashez le projet en une seule commande avec **West** :
```bash
west build -p always -b nucleo_f446re 
west flash
```
