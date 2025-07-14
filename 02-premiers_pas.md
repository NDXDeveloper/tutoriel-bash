🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Tutoriel Bash - Chapitre 2 : Premiers pas

## Création d'un premier script

### Étape 1 : Choisir un emplacement
Commençons par créer un dossier dédié à nos scripts pour rester organisés :

```bash
# Créer un dossier pour nos scripts
mkdir ~/mes_scripts

# Se déplacer dans ce dossier
cd ~/mes_scripts

# Vérifier où nous sommes
pwd
```

💡 **Explication :** Le symbole `~` représente votre dossier personnel (comme `/home/votreusername` sur Linux).

### Étape 2 : Créer le fichier
Créons notre premier script appelé `premier_script.sh` :

```bash
# Créer un fichier vide
touch premier_script.sh

# Ou bien créer et ouvrir directement avec nano
nano premier_script.sh
```

### Étape 3 : Écrire le contenu
Dans votre éditeur, tapez exactement ce code :

```bash
#!/bin/bash

# Mon premier script Bash
echo "=== Mon Premier Script Bash ==="
echo "Bonjour ! Je suis un script automatisé."
echo "Aujourd'hui nous sommes le :"
date
echo "Voici quelques informations sur votre système :"
echo "Nom d'utilisateur : $(whoami)"
echo "Répertoire actuel : $(pwd)"
echo "=== Fin du script ==="
```

### Étape 4 : Sauvegarder
- **Avec nano :** Appuyez sur `Ctrl+X`, puis `Y` pour confirmer, puis `Entrée`
- **Avec vim :** Tapez `:wq` puis `Entrée`
- **Avec un éditeur graphique :** Utilisez `Ctrl+S` ou le menu Fichier → Sauvegarder

## Shebang et permissions d'exécution

### Qu'est-ce que le Shebang ?

La première ligne `#!/bin/bash` s'appelle le **shebang** (prononcé "chébang").

**Analogie :** C'est comme l'étiquette sur un pot de confiture qui indique "confiture de fraises". Le shebang dit à l'ordinateur : "Ce fichier contient du code Bash".

### Pourquoi c'est important ?

```bash
#!/bin/bash
```

**Décomposition :**
- `#!` : Les symboles magiques qui disent "attention, instruction spéciale !"
- `/bin/bash` : Le chemin vers le programme qui va interpréter votre script

### Variantes courantes du shebang

```bash
#!/bin/bash          # Version standard (recommandée pour débuter)
#!/usr/bin/env bash  # Version portable (fonctionne sur plus de systèmes)
#!/bin/sh           # Version basique (moins de fonctionnalités)
```

**Pour débuter :** Utilisez toujours `#!/bin/bash`.

### Comprendre les permissions

Actuellement, votre script ne peut pas être exécuté. Vérifiez :

```bash
ls -l premier_script.sh
```

Vous verrez quelque chose comme :
```
-rw-r--r-- 1 utilisateur groupe 245 date premier_script.sh
```

**Décryptage des permissions :**
- `rw-` : Le propriétaire peut lire (r) et écrire (w), mais pas exécuter
- `r--` : Le groupe peut seulement lire
- `r--` : Les autres peuvent seulement lire

### Rendre le script exécutable

```bash
# Donner les permissions d'exécution
chmod +x premier_script.sh

# Vérifier le changement
ls -l premier_script.sh
```

Maintenant vous devriez voir :
```
-rwxr-xr-x 1 utilisateur groupe 245 date premier_script.sh
```

Les `x` indiquent que le fichier est maintenant exécutable !

## Exécution d'un script

### Méthode 1 : Exécution directe (recommandée)

```bash
# Exécuter le script
./premier_script.sh
```

**Pourquoi le `./` ?**
- `.` représente le répertoire actuel
- `/` sépare le répertoire du nom du fichier
- Ensemble, `./` dit "cherche le fichier dans le répertoire actuel"

### Méthode 2 : Avec l'interpréteur Bash

```bash
# Exécuter en passant le fichier à bash
bash premier_script.sh
```

**Avantage :** Pas besoin de permissions d'exécution
**Inconvénient :** Plus long à taper

### Méthode 3 : Depuis n'importe où

Si vous voulez exécuter votre script depuis n'importe quel répertoire :

```bash
# Exécuter avec le chemin complet
~/mes_scripts/premier_script.sh

# Ou bien
bash ~/mes_scripts/premier_script.sh
```

### Résultat attendu

Votre script devrait afficher quelque chose comme :

```
=== Mon Premier Script Bash ===
Bonjour ! Je suis un script automatisé.
Aujourd'hui nous sommes le :
dim. 13 juil. 2025 14:30:25 CEST
Voici quelques informations sur votre système :
Nom d'utilisateur : votre_nom
Répertoire actuel : /home/votre_nom/mes_scripts
=== Fin du script ===
```

## Structure de base d'un script

### Anatomie d'un script Bash

Voici la structure recommandée pour tous vos scripts :

```bash
#!/bin/bash

# =====================================
# Nom du script : exemple_structure.sh
# Description : Exemple de structure de script
# Auteur : Votre nom
# Date : 13 juillet 2025
# Version : 1.0
# =====================================

# =============
# CONFIGURATION
# =============

# Variables globales
SCRIPT_NAME="Exemple de structure"
VERSION="1.0"

# =============
# FONCTIONS
# =============

# Fonction d'affichage d'aide
afficher_aide() {
    echo "Usage: $0 [options]"
    echo "Options:"
    echo "  -h, --help    Afficher cette aide"
    echo "  -v, --version Afficher la version"
}

# Fonction principale
fonction_principale() {
    echo "=== $SCRIPT_NAME v$VERSION ==="
    echo "Démarrage du script..."

    # Votre code principal ici
    echo "Traitement en cours..."

    echo "Script terminé avec succès !"
}

# =============
# PROGRAMME PRINCIPAL
# =============

# Appel de la fonction principale
fonction_principale
```

### Sections importantes expliquées

**1. En-tête informatif :**
```bash
# =====================================
# Nom du script : mon_script.sh
# Description : Ce que fait le script
# Auteur : Votre nom
# Date : Date de création
# Version : Numéro de version
# =====================================
```

**Pourquoi c'est important :**
- Vous vous souvenez de ce que fait le script 6 mois plus tard
- Autres utilisateurs comprennent rapidement le but
- Facilite la maintenance et les mises à jour

**2. Section Configuration :**
```bash
# Variables globales
REPERTOIRE_SAUVEGARDE="/home/backup"
FICHIER_LOG="mon_script.log"
DEBUG=false
```

**3. Section Fonctions :**
```bash
# Fonction pour afficher des messages d'erreur
afficher_erreur() {
    echo "ERREUR: $1" >&2
    exit 1
}
```

**4. Programme principal :**
```bash
# Point d'entrée du script
echo "Début du script"
# Votre logique principale ici
echo "Fin du script"
```

### Script d'exemple complet et commenté

Créons un deuxième script plus élaboré :

```bash
nano script_info_systeme.sh
```

Contenu :

```bash
#!/bin/bash

# =====================================
# Nom : script_info_systeme.sh
# Description : Affiche des informations sur le système
# Auteur : Votre nom
# Date : 13 juillet 2025
# Version : 1.0
# =====================================

# =============
# VARIABLES
# =============

TITRE="Informations Système"
DATE_ACTUELLE=$(date)
UTILISATEUR=$(whoami)

# =============
# FONCTIONS
# =============

# Fonction pour afficher un titre encadré
afficher_titre() {
    echo "================================"
    echo "  $1"
    echo "================================"
}

# Fonction pour afficher une section
afficher_section() {
    echo
    echo "--- $1 ---"
}

# =============
# PROGRAMME PRINCIPAL
# =============

# Affichage du titre principal
afficher_titre "$TITRE"

# Informations de base
afficher_section "Informations de base"
echo "Date et heure : $DATE_ACTUELLE"
echo "Utilisateur : $UTILISATEUR"
echo "Répertoire actuel : $(pwd)"

# Informations système
afficher_section "Informations système"
echo "Système d'exploitation : $(uname -s)"
echo "Version du noyau : $(uname -r)"
echo "Architecture : $(uname -m)"

# Informations sur l'espace disque
afficher_section "Espace disque"
echo "Espace disque disponible :"
df -h | head -5

# Message de fin
echo
echo "Script terminé avec succès !"
```

### Bonnes pratiques pour débuter

**1. Toujours commencer par le shebang :**
```bash
#!/bin/bash
```

**2. Utiliser des commentaires explicites :**
```bash
# Ceci fait quelque chose d'important
commande_importante

# TODO: Améliorer cette partie plus tard
```

**3. Utiliser des noms de variables clairs :**
```bash
# ✅ Bien
nom_utilisateur="jean"
fichier_configuration="/etc/config.conf"

# ❌ Pas bien
u="jean"
f="/etc/config.conf"
```

**4. Tester fréquemment :**
Après chaque modification, testez votre script :
```bash
./mon_script.sh
```

**5. Gérer les erreurs simplement :**
```bash
# Vérifier si une commande a fonctionné
if ! commande_qui_peut_echouer; then
    echo "Erreur : La commande a échoué"
    exit 1
fi
```

### Exercice pratique

Créez un script appelé `mon_journal.sh` qui :
1. Affiche la date du jour
2. Demande votre humeur
3. Sauvegarde votre réponse dans un fichier `journal.txt`

**Solution :**
```bash
#!/bin/bash

# Script de journal personnel
echo "=== Journal du $(date +%d/%m/%Y) ==="
echo "Comment vous sentez-vous aujourd'hui ?"
read -p "Votre humeur : " humeur

echo "$(date): $humeur" >> journal.txt
echo "Entrée sauvegardée dans journal.txt"
```

### Récapitulatif

✅ **Ce que vous avez appris :**
- Créer un fichier script Bash
- Comprendre et utiliser le shebang
- Rendre un script exécutable avec `chmod +x`
- Exécuter un script avec `./nom_script.sh`
- Structurer proprement un script avec commentaires et sections

✅ **Prochaines étapes :**
Dans le chapitre 3, nous apprendrons à utiliser des variables pour rendre nos scripts plus flexibles et puissants !

---

💡 **Astuce :** Gardez vos premiers scripts simples et ajoutez de la complexité progressivement. Chaque script qui fonctionne est une victoire !

⏭️
