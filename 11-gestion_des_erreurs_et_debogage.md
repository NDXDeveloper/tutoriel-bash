🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Gestion des erreurs et débogage

## Introduction

La gestion des erreurs est un aspect crucial du scripting Bash. Un script robuste doit pouvoir détecter les erreurs, les gérer proprement et fournir des informations utiles pour le débogage. Cette section vous apprendra à rendre vos scripts plus fiables et plus faciles à maintenir.

## 1. Codes de retour et la variable $?

### Qu'est-ce qu'un code de retour ?

Chaque commande en Bash retourne un code de sortie (exit code) :
- **0** : succès
- **1-255** : erreur (différents types d'erreurs)

### La variable $?

La variable spéciale `$?` contient le code de retour de la dernière commande exécutée.

```bash
#!/bin/bash

# Exemple simple
ls /home
echo "Code de retour de ls : $?"

# Commande qui va échouer
ls /repertoire_inexistant
echo "Code de retour de ls (erreur) : $?"

# Test du code de retour
if [ $? -eq 0 ]; then
    echo "Commande réussie"
else
    echo "Commande échouée"
fi
```

### Exemple pratique : vérification d'une sauvegarde

```bash
#!/bin/bash

# Script de sauvegarde avec vérification
SOURCE="/home/user/documents"
BACKUP="/backup/documents_$(date +%Y%m%d)"

# Création de la sauvegarde
cp -r "$SOURCE" "$BACKUP"

# Vérification du succès
if [ $? -eq 0 ]; then
    echo "✓ Sauvegarde créée avec succès dans $BACKUP"
else
    echo "✗ Erreur lors de la création de la sauvegarde"
    exit 1
fi
```

## 2. Modes de débogage

### set -x : Mode trace

Active l'affichage de chaque commande avant son exécution.

```bash
#!/bin/bash

set -x  # Active le mode trace

echo "Début du script"
nom="Alice"
echo "Bonjour $nom"

# Sortie affichée :
# + echo 'Début du script'
# Début du script
# + nom=Alice
# + echo 'Bonjour Alice'
# Bonjour Alice
```

### set -e : Arrêt sur erreur

Le script s'arrête immédiatement si une commande retourne un code d'erreur non nul.

```bash
#!/bin/bash

set -e  # Arrêt sur erreur

echo "Cette ligne sera exécutée"
ls /repertoire_inexistant  # Cette commande va échouer
echo "Cette ligne ne sera JAMAIS exécutée"
```

### set -u : Variables non définies

Traite les variables non définies comme des erreurs.

```bash
#!/bin/bash

set -u  # Erreur sur variable non définie

echo "Début du script"
echo "Valeur : $variable_non_definie"  # Erreur !
```

### set -o pipefail : Erreur dans les pipes

Par défaut, seul le code de retour de la dernière commande d'un pipe est vérifié.

```bash
#!/bin/bash

set -o pipefail  # Erreur si n'importe quelle commande du pipe échoue

# Sans pipefail : le script continue même si grep échoue
# Avec pipefail : le script s'arrête si grep échoue
cat fichier_inexistant | grep "pattern" | sort
```

### Combinaison des modes

```bash
#!/bin/bash

# Combinaison recommandée pour un script robuste
set -euo pipefail

# Maintenant le script :
# - s'arrête sur toute erreur (-e)
# - traite les variables non définies comme des erreurs (-u)
# - détecte les erreurs dans les pipes (-o pipefail)
```

## 3. Gestion des erreurs avec trap

### Qu'est-ce que trap ?

`trap` permet d'exécuter des commandes lorsque le script reçoit des signaux ou se termine.

### Nettoyage automatique

```bash
#!/bin/bash

set -e

# Fichier temporaire
TEMP_FILE="/tmp/monscript_$$"

# Fonction de nettoyage
cleanup() {
    echo "Nettoyage en cours..."
    rm -f "$TEMP_FILE"
    echo "Nettoyage terminé"
}

# Définir le trap pour le nettoyage
trap cleanup EXIT

# Création du fichier temporaire
echo "Création du fichier temporaire"
touch "$TEMP_FILE"

# Simulation d'un travail
echo "Traitement en cours..."
sleep 2

# Le nettoyage se fera automatiquement même en cas d'erreur
```

### Gestion des interruptions

```bash
#!/bin/bash

# Fonction appelée lors d'une interruption
handle_interrupt() {
    echo ""
    echo "Script interrompu par l'utilisateur"
    echo "Nettoyage et fermeture..."
    exit 1
}

# Piéger les signaux d'interruption
trap handle_interrupt INT TERM

echo "Script en cours d'exécution (Ctrl+C pour arrêter)"
while true; do
    echo "Travail en cours..."
    sleep 1
done
```

### Gestion avancée des erreurs

```bash
#!/bin/bash

set -e

# Fonction de gestion d'erreur
error_handler() {
    local line_number=$1
    local error_code=$2
    echo "Erreur ligne $line_number : code de sortie $error_code"
    echo "Commande qui a échoué : $BASH_COMMAND"

    # Nettoyage si nécessaire
    cleanup

    exit $error_code
}

# Piéger les erreurs
trap 'error_handler $LINENO $?' ERR

cleanup() {
    echo "Nettoyage des ressources..."
    # Ajouter ici les opérations de nettoyage
}

# Votre code ici
echo "Début du script"
ls /repertoire_inexistant  # Cette erreur sera capturée
```

## 4. Journalisation et messages d'erreur

### Niveaux de messages

```bash
#!/bin/bash

# Fonction pour les messages d'information
info() {
    echo "[INFO] $(date '+%Y-%m-%d %H:%M:%S') : $1" >&2
}

# Fonction pour les avertissements
warn() {
    echo "[WARN] $(date '+%Y-%m-%d %H:%M:%S') : $1" >&2
}

# Fonction pour les erreurs
error() {
    echo "[ERROR] $(date '+%Y-%m-%d %H:%M:%S') : $1" >&2
}

# Utilisation
info "Démarrage du script"
warn "Attention : fichier de configuration par défaut utilisé"
error "Impossible de se connecter à la base de données"
```

### Journalisation dans un fichier

```bash
#!/bin/bash

LOG_FILE="/var/log/monscript.log"

# Fonction de journalisation
log() {
    local level=$1
    shift
    local message="$@"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')

    echo "[$timestamp] [$level] $message" >> "$LOG_FILE"

    # Afficher aussi à l'écran pour les erreurs
    if [ "$level" = "ERROR" ]; then
        echo "[$timestamp] [$level] $message" >&2
    fi
}

# Utilisation
log "INFO" "Script démarré"
log "WARN" "Utilisation du fichier de configuration par défaut"
log "ERROR" "Connexion à la base de données échouée"
```

### Redirection des erreurs

```bash
#!/bin/bash

# Rediriger toutes les erreurs vers un fichier
exec 2>/var/log/script_errors.log

# Ou rediriger à la fois vers un fichier et l'écran
exec 2> >(tee -a /var/log/script_errors.log)

echo "Message normal"
echo "Message d'erreur" >&2
```

## 5. Techniques et outils de débogage

### Débogage interactif

```bash
#!/bin/bash

# Activer le débogage pour une section spécifique
debug_mode=true

if [ "$debug_mode" = true ]; then
    set -x
fi

# Code à déboguer
nom="Alice"
echo "Bonjour $nom"

if [ "$debug_mode" = true ]; then
    set +x
fi
```

### Points d'arrêt manuels

```bash
#!/bin/bash

# Fonction pour créer un point d'arrêt
debug_pause() {
    if [ "${DEBUG:-false}" = "true" ]; then
        echo "Point d'arrêt atteint. Variables actuelles :"
        echo "  PWD = $PWD"
        echo "  USER = $USER"
        echo "Appuyez sur Entrée pour continuer..."
        read
    fi
}

# Utilisation
echo "Avant traitement"
debug_pause

# Traitement
echo "Traitement en cours"
debug_pause

echo "Après traitement"
```

### Utilisation de ShellCheck

ShellCheck est un outil d'analyse statique pour les scripts Bash.

```bash
# Installation (Ubuntu/Debian)
sudo apt-get install shellcheck

# Utilisation
shellcheck monscript.sh
```

Exemple de sortie ShellCheck :
```
Line 5: echo $nom
         ^-- SC2086: Double quote to prevent globbing and word splitting.
```

### Techniques de débogage avancées

```bash
#!/bin/bash

# Variable globale pour le débogage
DEBUG=${DEBUG:-false}

# Fonction de débogage
debug_echo() {
    if [ "$DEBUG" = "true" ]; then
        echo "[DEBUG] $1" >&2
    fi
}

# Fonction pour afficher les variables
debug_vars() {
    if [ "$DEBUG" = "true" ]; then
        echo "[DEBUG] Variables actuelles :" >&2
        for var in "$@"; do
            echo "[DEBUG]   $var = ${!var}" >&2
        done
    fi
}

# Utilisation
nom="Alice"
age=30

debug_echo "Début du traitement"
debug_vars nom age

# Pour activer le débogage :
# DEBUG=true ./monscript.sh
```

## Exemple complet : Script robuste avec gestion d'erreurs

```bash
#!/bin/bash

# Configuration stricte
set -euo pipefail

# Variables globales
readonly SCRIPT_NAME=$(basename "$0")
readonly LOG_FILE="/var/log/${SCRIPT_NAME}.log"
readonly TEMP_DIR="/tmp/${SCRIPT_NAME}_$$"

# Fonction de journalisation
log() {
    local level=$1
    shift
    local message="$@"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')

    echo "[$timestamp] [$level] $message" | tee -a "$LOG_FILE"

    if [ "$level" = "ERROR" ]; then
        echo "[$timestamp] [$level] $message" >&2
    fi
}

# Fonction de nettoyage
cleanup() {
    log "INFO" "Nettoyage en cours..."
    rm -rf "$TEMP_DIR"
    log "INFO" "Nettoyage terminé"
}

# Fonction de gestion d'erreur
error_handler() {
    local line_number=$1
    local error_code=$2
    log "ERROR" "Erreur ligne $line_number : code $error_code"
    log "ERROR" "Commande : $BASH_COMMAND"
    cleanup
    exit $error_code
}

# Configuration des traps
trap cleanup EXIT
trap 'error_handler $LINENO $?' ERR

# Fonction principale
main() {
    log "INFO" "Démarrage du script"

    # Création du répertoire temporaire
    mkdir -p "$TEMP_DIR"
    log "INFO" "Répertoire temporaire créé : $TEMP_DIR"

    # Votre code ici
    echo "Traitement en cours..."

    log "INFO" "Script terminé avec succès"
}

# Vérification des prérequis
if ! command -v curl &> /dev/null; then
    log "ERROR" "curl n'est pas installé"
    exit 1
fi

# Exécution du script
main "$@"
```

## Bonnes pratiques

### 1. Toujours vérifier les codes de retour
```bash
# Bon
if cp source destination; then
    echo "Copie réussie"
else
    echo "Erreur lors de la copie"
    exit 1
fi

# Ou avec set -e
set -e
cp source destination
echo "Copie réussie"
```

### 2. Utiliser des messages d'erreur informatifs
```bash
# Mauvais
echo "Erreur"

# Bon
echo "Erreur : impossible de créer le répertoire $DIR (permission refusée)"
```

### 3. Nettoyer les ressources
```bash
# Toujours nettoyer les fichiers temporaires
trap 'rm -f "$TEMP_FILE"' EXIT
```

### 4. Valider les entrées
```bash
# Vérifier que les arguments nécessaires sont fournis
if [ $# -lt 2 ]; then
    echo "Usage: $0 <source> <destination>"
    exit 1
fi
```

## Résumé

La gestion des erreurs et le débogage sont essentiels pour créer des scripts Bash robustes :

- **Codes de retour** : Utilisez `$?` pour vérifier le succès des commandes
- **Modes de débogage** : `set -e`, `set -u`, `set -x`, `set -o pipefail`
- **Trap** : Gérez les signaux et nettoyez les ressources
- **Journalisation** : Gardez une trace des opérations et erreurs
- **Outils** : Utilisez ShellCheck pour détecter les problèmes potentiels

En appliquant ces techniques, vos scripts seront plus fiables, plus faciles à maintenir et à déboguer.

⏭️
