# Créer un Serveur ARK sur Debian 12

## Prérequis
- Un système Debian 12 avec au minimum 4 Go de RAM et 4 cœurs de processeur.

## Étapes à Suivre

### 1. Augmenter la Possibilité de Création de Fichiers
Pour augmenter le nombre de fichiers pouvant être créés, modifiez certaines configurations système.

Ouvrez le fichier `/etc/sysctl.conf` :
```bash
sudo nano /etc/sysctl.conf

Ajoutez les lignes suivantes à la fin du fichier :
