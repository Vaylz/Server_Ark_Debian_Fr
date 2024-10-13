
# Documentation pour l'utilisation d'une sauvegarde sur le serveur ARK

Ce document décrit comment configurer votre serveur ARK pour qu'il lance toujours la même sauvegarde.

## Étape 1 : Configurer le serveur pour utiliser la sauvegarde

### Étape 1.1 : Localiser les fichiers de configuration

1. Accédez au dossier de votre serveur ARK. Si vous utilisez Docker, cela se trouve généralement dans le répertoire où vous avez installé votre serveur ARK.
2. Cherchez les fichiers de configuration importants : `GameUserSettings.ini` et `Game.ini`.

### Étape 1.2 : Modifier le fichier `GameUserSettings.ini`

1. Ouvrez le fichier `GameUserSettings.ini` dans un éditeur de texte.
2. Recherchez la section qui définit la carte et les paramètres de sauvegarde. Par exemple :
   ```ini
   [ServerSettings]
   MapName=TheIsland
   ```
3. Assurez-vous que le nom de la carte correspond à la sauvegarde. Par exemple, pour `TheIsland.ark`, cela devrait être `MapName=TheIsland`.

### Étape 1.3 : Modifier le fichier `Game.ini`

1. Ouvrez le fichier `Game.ini` de la même manière.
2. Vérifiez s'il y a des paramètres spécifiques pour la sauvegarde ou la carte, mais dans la plupart des cas, vous n'aurez pas besoin d'apporter des modifications ici.

### Étape 1.4 : Sauvegarder les modifications

1. Après avoir effectué les modifications, enregistrez le fichier.
2. Fermez l’éditeur de texte.

## Étape 2 : Redémarrer le serveur

1. Redémarrez votre serveur ARK pour appliquer les modifications.
2. Vérifiez que la sauvegarde a été chargée correctement en vous connectant à votre serveur.

## Remarques supplémentaires

- Pensez à effectuer des sauvegardes régulières de votre serveur pour éviter toute perte de données.
- Pour automatiser le processus à chaque redémarrage, vous pouvez créer un script qui copie la sauvegarde souhaitée dans le répertoire de votre serveur à chaque démarrage.

---
