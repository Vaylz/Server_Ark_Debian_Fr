# Créer un Serveur ARK sur Debian 12

## Prérequis
- Un système Debian 12 avec au minimum 4 Go de RAM et 4 cœurs de processeur.

Système recommandé : minimum 8 Go de RAM et 6 cœurs

## Étapes à Suivre

### 1. Augmenter la Possibilité de Création de Fichiers
Pour augmenter le nombre de fichiers pouvant être créés, modifiez certaines configurations système.

Ouvrez le fichier `/etc/sysctl.conf` :
```bash
sudo nano /etc/sysctl.conf
```

Ajoutez les lignes suivantes à la fin du fichier :
```bash
fs.file-max = 100000
```

Ensuite, ouvrez le fichier `/etc/security/limits.conf` :
```bash
* soft nofile 100000
* hard nofile 100000
```

Modifiez le fichier `/etc/pam.d/common-session` :
```bash
session required pam_limits.so
```
### 2. Installer des Paquets Nécessaires
Installez les paquets nécessaires pour faire fonctionner le serveur.

Installez les paquets nécessaires pour faire fonctionner le serveur.
```bash
#sudo add-apt-repository multiverse
sudo apt install software-properties-common
sudo dpkg --add-architecture i386
sudo apt update
sudo apt-get install lib32gcc-s1 lib32stdc++6
```

### 3. Installer SteamCMD
Téléchargez et installez SteamCMD.
```bash
wget http://media.steampowered.com/installer/steamcmd_linux.tar.gz
tar -xvzf steamcmd_linux.tar.gz
```
# Créer un Utilisateur pour le Serveur
Il est recommandé de créer un utilisateur dédié pour le serveur ARK.
```bash
sudo adduser arkserver
```
# Créer un Dossier pour le Serveur
Créez un dossier pour le serveur ARK
```bash
mkdir /home/arkserver/server
```

### 4. Installation du Serveur ARK
Lancez SteamCMD et installez le serveur ARK.
```bash
cd steamcmd
sudo ./steamcmd.sh
```
Dans SteamCMD, exécutez les commandes suivantes :
```bash
login anonymous
force_install_dir /home/arkserver/server
app_update 376030 validate
quit
```
### 5. Créer le Script de Démarrage et d'Arrêt pour le Serveur
Créez un script pour démarrer et arrêter le serveur. Créez un fichier nommé `startserver.sh` :
```bash
nano /home/arkserver/server/ShooterGame/Binaries/Linux/startserver.sh
```
Ajoutez le contenu suivant :
```bash
#!/bin/bash

# Nom de la carte 
MAP_NAME="TheIsland"
# Nombre de slots disponibles
PLAYER_SLOTS=5

# Vérification si le serveur est déjà en cours d'exécution
if pgrep -f "ShooterGameServer" > /dev/null; then
    echo "Le serveur ARK est déjà en cours d'exécution."
    exit 1
fi

# Paramètres ulimit
ulimit -n 100000

# Démarrage du serveur ARK
./ShooterGameServer "$MAP_NAME?listen?MaxPlayers=$PLAYER_SLOTS" -server -log &

# Récupérer le PID du serveur
SERVER_PID=$!

# Vérifier si le serveur a démarré avec succès
sleep 5  # Attendre 5 secondes pour voir si le serveur démarre correctement

if ps -p $SERVER_PID > /dev/null; then
    echo "Serveur ARK démarré avec succès (PID: $SERVER_PID)."
else
    echo "Échec du démarrage du serveur ARK."
    exit 1
fi
```
Cela permet de lancer le serveur avec la carte TheIsland

Rendez le script exécutable :
```bash
sudo chmod 722 /home/arkserver/start_server.sh
```
#Créer un Script pour Arrêter le Serveur
Créez un autre fichier nommé `stopserver.sh` :
```bash
nano /home/arkserver/server/ShooterGame/Binaries/Linux/stopserver.sh
```
Ajoutez le contenu suivant :
```bash
#!/bin/bash

# Forcer le serveur à sauvegarder avant de l'arrêter
echo "Forçage de la sauvegarde du serveur..."
rcon_command "SaveWorld"

# Attendre quelques secondes pour s'assurer que la sauvegarde est terminée
sleep 10

# Récupérer le PID du serveur
PID=$(pgrep -f "ShooterGameServer")

if [ -z "$PID" ]; then
    echo "Le serveur ARK n'est pas en cours d'exécution."
else
    # Envoie le signal pour arrêter le serveur
    kill -SIGTERM "$PID"
    echo "Serveur ARK arrêté."
fi
```
Rendez ce script exécutable aussi 

### 6. Démarrer et Arrêter le Serveur
Pour démarrer le serveur, exécutez :
```bash
sudo /home/arkserver/server/ShotGame/Binaire/Linux/./startserver.sh
```
```bash
sudo /home/arkserver/server/ShotGame/Binaire/Linux/./stopserver.sh
```