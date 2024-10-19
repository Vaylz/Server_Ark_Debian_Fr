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
    <title>Lancer le script Bash</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
            color: #333;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: flex-start;
            height: 100vh;
            margin: 0;
            padding: 20px;
        }

        h1 {
            color: #4CAF50;
            margin-bottom: 20px;
            text-align: center;
            width: 100%;
        }

        form {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 15px; /* Espacement entre les éléments du formulaire */
        }

        button {
            background-color: #4CAF50;
            border: none;
            color: white;
            padding: 15px 32px;
            text-align: center;
            text-decoration: none;
            display: inline-block;
            font-size: 16px;
            cursor: pointer;
            border-radius: 5px;
            transition: background-color 0.3s, transform 0.2s; /* Transition ajoutée */
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1); /* Ombre ajoutée */
        }

        button:hover {
            background-color: #45a049;
            transform: translateY(-2px); /* Légère élévation lors du survol */
        }

        #message {
            margin-top: 20px;
            font-size: 16px;
            color: #555;
            text-align: center;
            width: 100%;
        }
    </style>
</head>
<body>
    <h1>Exécuter un script Bash depuis une page Web</h1>
    <form action="/cgi-bin/startserver.sh" method="GET">
        <button type="submit">Lancer le script</button>
    </form>
    <br>
    <form action="/cgi-bin/stopserver.sh" method="GET">
        <button type="submit">Lancer le script1</button>
    </form>

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
ulimit -n 100000

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

Ajoutez le code suivant dans `stop.sh` :

```bash
#!/bin/bash

# En-têtes HTTP
echo "Content-type: text/html"
echo ""

# Forcer le serveur à sauvegarder avant de l'arrêter
echo "<html><body>"
echo "<h1>Contrôle du Serveur</h1>"
echo "<p>Forçage de la sauvegarde du serveur...</p>"

# Simule la commande RCON pour sauvegarder le monde
# rcon_command "SaveWorld"

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
