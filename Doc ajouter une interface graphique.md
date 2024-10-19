# Documentation pour le Contrôle du Serveur ARK avec Apache

Cette documentation décrit comment configurer un serveur Apache pour contrôler un serveur ARK via une interface web.

## Étape 1 : Installation d'Apache

Mettez à jour votre liste de paquets et installez Apache :

```bash
sudo apt update
sudo apt upgrade
sudo apt install apache2
```

## Étape 2 : Création de l'interface de contrôle

Créez un répertoire pour l'interface de contrôle :

```bash
sudo mkdir /var/www/html/ark_server_control
sudo nano /var/www/html/ark_server_control/index.html
```

## Étape 3 : Ajout du code HTML

Ajoutez le code suivant dans `index.html` :

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bienvenue sur notre serveur ARK</title>
    <link rel="stylesheet" href="style.css">
    <script>
        // Fonction pour envoyer une requête AJAX sans recharger la page
        function executeServerAction(scriptPath) {
            var xhr = new XMLHttpRequest(); // Créer l'objet XMLHttpRequest

            // Vérification de l'état de la requête
            xhr.onreadystatechange = function() {
                if (xhr.readyState == 4 && xhr.status == 200) {
                    // Insérer la réponse dans la section messages
                    document.getElementById("messageContent").innerHTML = xhr.responseText;
                }
            };

            // Configurer et envoyer la requête GET vers le script CGI
            xhr.open("GET", scriptPath, true);
            xhr.send();
        }
    </script>
</head>
<body>
    <header>
        <h1>Serveur ARK: Survival Evolved</h1>
    </header>

    <main>
        <section id="status">
            <h2>État du serveur</h2>
            <!-- Bouton pour démarrer le serveur -->
            <button type="button" onclick="executeServerAction('/cgi-bin/startserver.sh')">Lancer le serveur</button>
            <br><br>
            <!-- Bouton pour arrêter le serveur -->
            <button type="button" onclick="executeServerAction('/cgi-bin/stopserver.sh')">Arrêter le serveur</button>
        </section>

        <!-- Section pour afficher les messages du serveur -->
        <section id="messages">
            <h2>Messages du serveur</h2>
            <div id="messageContent">
                Aucune action pour le moment.
            </div>
        </section>
    </main>
</body>
</html>
```
ouvrire le port 80 avec ufw :
```bash
sudo ufw allow 80/tcp
```
## Étape 4 : Création des scripts CGI

Créez les scripts `startserver.sh` et `stopserver.sh` dans `/usr/lib/cgi-bin/` :

```bash
sudo nano /usr/lib/cgi-bin/startserver.sh
```

Ajoutez le code suivant dans `startserver.sh` :

```bash
#!/bin/bash

# En-têtes HTTP
echo "Content-type: text/html"
echo ""

# Chemin vers le dossier ARK
ARK_DIR="/home/arkserver/server/ShooterGame/Binaries/Linux"

# Nom de la carte
MAP_NAME="TheIsland"
# Nombre de slots disponibles
PLAYER_SLOTS=5

# Vérification si le serveur est déjà en cours d'exécution
if pgrep -f "ShooterGameServer" > /dev/null; then
    echo "<html><body>"
    echo "<h1>Contrôle du Serveur</h1>"
    echo "<p>Le serveur ARK est déjà en cours d'exécution.</p>"
    echo "</body></html>"
    exit 1
fi

# Paramètres ulimit
#ulimit -n 100000

# Se déplacer dans le répertoire contenant ShooterGameServer
cd "$ARK_DIR" || {
    echo "<html><body>"
    echo "<h1>Erreur</h1>"
    echo "<p>Impossible de trouver le répertoire ARK à $ARK_DIR.</p>"
    echo "</body></html>"
    exit 1
}

# Vérification de l'existence du fichier ShooterGameServer
if [ ! -f "./ShooterGameServer" ]; then
    echo "<html><body>"
    echo "<h1>Erreur</h1>"
    echo "<p>Le fichier 'ShooterGameServer' n'a pas été trouvé dans le répertoire $ARK_DIR.</p>"
    echo "</body></html>"
    exit 1
fi

# Démarrage du serveur ARK
./ShooterGameServer "$MAP_NAME?listen?MaxPlayers=$PLAYER_SLOTS" -server -log &

# Récupérer le PID du serveur
SERVER_PID=$!

# Vérifier si le serveur a démarré avec succès
sleep 5

echo "<html><body>"

if ps -p $SERVER_PID > /dev/null; then
    echo "<h1>Contrôle du Serveur</h1>"
    echo "<p>Serveur ARK démarré avec succès (PID: $SERVER_PID).</p>"
else
    echo "<h1>Contrôle du Serveur</h1>"
    echo "<p>Échec du démarrage du serveur ARK.</p>"
fi

echo "</body></html>"
exit 0


```


Puis, créez `stopserver.sh` :

```bash
sudo nano /usr/lib/cgi-bin/stopserver.sh
```

Ajoutez le code suivant dans `stopserver.sh` :

```bash
#!/bin/bash

# En-têtes HTTP
echo "Content-type: text/html"
echo ""

# Forcer le serveur à sauvegarder avant de l'arrêter
echo "<html><body>"
echo "<h1>Contrôle du Serveur</h1>"
echo "<p>Forçage de la sauvegarde du serveur...</p>"

# Fichier de sauvegarde (vous pouvez spécifier un chemin complet si nécessaire)
backup_file="/path/to/your/directory/backup.log"

# Simule la commande RCON pour sauvegarder le monde
echo "Exécution de la commande de sauvegarde..."
# Remplacez 'rcon_command' par votre commande RCON pour sauvegarder le monde
rcon_command "SaveWorld" >> "$backup_file" 2>&1

# Attendre que la sauvegarde soit terminée
sleep 10

# Récupérer le PID du serveur
PID=$(pgrep -f "ShooterGameServer")

if [ -z "$PID" ]; then
    echo "<p>Le serveur ARK n'est pas en cours d'exécution.</p>"
else
    kill -SIGTERM "$PID"
    echo "<p>Serveur ARK arrêté.</p>"
fi

echo "</body></html>"
exit 0

```
rendre les script  executible

```bash
sudo chmod +x /usr/lib/cgi-bin/startserver.sh /usr/lib/cgi-bin/stopserver.sh
```

## Étape 5 : Configuration d'Apache pour les scripts CGI

Activez le module CGI d'Apache :

```bash
sudo a2enmod cgi
```

Ouvrez le fichier de configuration d'Apache :

```bash
cd /etc/apache2/sites-available/000-default.conf
```
Faite une copie du fichier de 000-default.conf

```bash
sudo cp 000-default.conf ark.conf
```
Ajoutez la configuration suivante dans le fichier ark.conf :

```apache
<Directory "/var/www/cgi-bin">
    AllowOverride None
    Options ExecCGI
    Order allow,deny
    Allow from all
</Directory>
```
et modifier l'emplacement ou ce trouve votre fichier html

Puis redémarrez Apache :

```bash
sudo systemctl restart apache2
```
## Étape 6 : Creation d'un group ARKgroup

On va créer un groupe appelé `arkgroup`

```bash
sudo groupadd arkgroup
```
Ajouter les utilisateurs arkserver et www-data au groupe

```bash
sudo usermod -aG arkgroup arkserver
sudo usermod -aG arkgroup www-data
```
Ensuite, il faut changer le groupe propriétaire du répertoire ARK pour qu'il soit sous la gestion de arkgroup. Cela va permettre à tous les utilisateurs dans ce groupe d'accéder au dossier.

```bash
sudo chown -R arkserver:arkgroup /home/arkserver
sudo chmod -R 770 /home/arkserver
```
Attention de bien vérifier si le group arkgroup a les droits sur l'utilisateur arkserver

## Étape 7 : Accédez à votre interface

Vous pouvez maintenant accéder à votre site via l'URL suivante :

```
http://votre-serveur/ark_server_control/index.html
```
