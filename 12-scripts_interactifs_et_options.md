🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Scripts interactifs et options

## Introduction

Les scripts interactifs permettent de créer des programmes qui communiquent avec l'utilisateur en temps réel. Cette section vous apprendra à créer des interfaces utilisateur simples, à valider les entrées, à gérer les signaux et à utiliser des options de ligne de commande professionnelles.

## 1. Menus et interfaces utilisateur simples

### Menu basique avec select

La commande `select` crée automatiquement un menu numéroté :

```bash
#!/bin/bash

echo "Que voulez-vous faire ?"
select action in "Lister les fichiers" "Afficher la date" "Quitter"; do
    case $action in
        "Lister les fichiers")
            ls -la
            ;;
        "Afficher la date")
            date
            ;;
        "Quitter")
            echo "Au revoir !"
            break
            ;;
        *)
            echo "Option invalide. Choisissez 1, 2 ou 3."
            ;;
    esac
done
```

### Menu personnalisé avec read

```bash
#!/bin/bash

show_menu() {
    echo "================================"
    echo "    MENU PRINCIPAL"
    echo "================================"
    echo "1. Gérer les fichiers"
    echo "2. Informations système"
    echo "3. Outils réseau"
    echo "4. Quitter"
    echo "================================"
    echo -n "Votre choix (1-4) : "
}

while true; do
    show_menu
    read choice

    case $choice in
        1)
            echo "Gestion des fichiers sélectionnée"
            # Appeler une sous-fonction
            ;;
        2)
            echo "Informations système :"
            uname -a
            uptime
            ;;
        3)
            echo "Outils réseau :"
            echo "IP : $(hostname -I)"
            ;;
        4)
            echo "Au revoir !"
            exit 0
            ;;
        *)
            echo "Choix invalide. Veuillez choisir entre 1 et 4."
            sleep 2
            ;;
    esac

    echo ""
    echo "Appuyez sur Entrée pour continuer..."
    read
    clear  # Efface l'écran
done
```

### Menu avec sous-menus

```bash
#!/bin/bash

# Fonction pour le menu principal
main_menu() {
    while true; do
        clear
        echo "=== MENU PRINCIPAL ==="
        echo "1. Gestion des fichiers"
        echo "2. Système"
        echo "3. Quitter"
        echo -n "Choix : "
        read choice

        case $choice in
            1) file_menu ;;
            2) system_menu ;;
            3) exit 0 ;;
            *) echo "Choix invalide" ; sleep 2 ;;
        esac
    done
}

# Sous-menu fichiers
file_menu() {
    while true; do
        clear
        echo "=== GESTION FICHIERS ==="
        echo "1. Lister les fichiers"
        echo "2. Créer un fichier"
        echo "3. Supprimer un fichier"
        echo "4. Retour au menu principal"
        echo -n "Choix : "
        read choice

        case $choice in
            1)
                echo "Fichiers dans le répertoire actuel :"
                ls -la
                ;;
            2)
                echo -n "Nom du fichier à créer : "
                read filename
                touch "$filename"
                echo "Fichier '$filename' créé avec succès"
                ;;
            3)
                echo -n "Nom du fichier à supprimer : "
                read filename
                if [ -f "$filename" ]; then
                    rm "$filename"
                    echo "Fichier '$filename' supprimé"
                else
                    echo "Fichier '$filename' non trouvé"
                fi
                ;;
            4)
                return  # Retourne au menu principal
                ;;
            *)
                echo "Choix invalide"
                ;;
        esac
        echo ""
        echo "Appuyez sur Entrée pour continuer..."
        read
    done
}

# Sous-menu système
system_menu() {
    while true; do
        clear
        echo "=== INFORMATIONS SYSTÈME ==="
        echo "1. Utilisation du disque"
        echo "2. Mémoire"
        echo "3. Processus"
        echo "4. Retour au menu principal"
        echo -n "Choix : "
        read choice

        case $choice in
            1) df -h ;;
            2) free -h ;;
            3) ps aux | head -20 ;;
            4) return ;;
            *) echo "Choix invalide" ;;
        esac
        echo ""
        echo "Appuyez sur Entrée pour continuer..."
        read
    done
}

# Lancement du programme
main_menu
```

## 2. Validation des entrées utilisateur

### Validation basique

```bash
#!/bin/bash

# Fonction pour valider un nombre
validate_number() {
    local input=$1

    # Vérifier si c'est un nombre
    if [[ $input =~ ^[0-9]+$ ]]; then
        return 0  # Valide
    else
        return 1  # Invalide
    fi
}

# Demander un nombre avec validation
while true; do
    echo -n "Entrez un nombre positif : "
    read number

    if validate_number "$number"; then
        echo "Nombre valide : $number"
        break
    else
        echo "Erreur : Veuillez entrer un nombre valide"
    fi
done
```

### Validation d'email

```bash
#!/bin/bash

validate_email() {
    local email=$1
    local pattern="^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"

    if [[ $email =~ $pattern ]]; then
        return 0
    else
        return 1
    fi
}

while true; do
    echo -n "Entrez votre email : "
    read email

    if validate_email "$email"; then
        echo "Email valide : $email"
        break
    else
        echo "Email invalide. Format attendu : utilisateur@domaine.com"
    fi
done
```

### Validation avec choix limités

```bash
#!/bin/bash

# Fonction pour valider une réponse oui/non
validate_yes_no() {
    local input=$1

    case ${input,,} in  # ${input,,} convertit en minuscules
        oui|o|yes|y)
            return 0
            ;;
        non|n|no)
            return 1
            ;;
        *)
            return 2  # Entrée invalide
            ;;
    esac
}

# Demander confirmation
while true; do
    echo -n "Voulez-vous continuer ? (oui/non) : "
    read response

    if validate_yes_no "$response"; then
        echo "Vous avez choisi : OUI"
        break
    elif [ $? -eq 1 ]; then
        echo "Vous avez choisi : NON"
        break
    else
        echo "Réponse invalide. Veuillez répondre par 'oui' ou 'non'"
    fi
done
```

### Validation d'une plage de valeurs

```bash
#!/bin/bash

validate_range() {
    local number=$1
    local min=$2
    local max=$3

    if [[ $number =~ ^[0-9]+$ ]] && [ $number -ge $min ] && [ $number -le $max ]; then
        return 0
    else
        return 1
    fi
}

echo "Choisissez votre niveau de difficulté :"
echo "1. Facile"
echo "2. Moyen"
echo "3. Difficile"

while true; do
    echo -n "Votre choix (1-3) : "
    read level

    if validate_range "$level" 1 3; then
        case $level in
            1) echo "Niveau Facile sélectionné" ;;
            2) echo "Niveau Moyen sélectionné" ;;
            3) echo "Niveau Difficile sélectionné" ;;
        esac
        break
    else
        echo "Veuillez choisir entre 1 et 3"
    fi
done
```

### Validation avec timeout

```bash
#!/bin/bash

# Fonction avec timeout
read_with_timeout() {
    local prompt=$1
    local timeout=$2
    local default=$3

    echo -n "$prompt"
    if read -t $timeout response; then
        echo "$response"
    else
        echo ""
        echo "Timeout ! Utilisation de la valeur par défaut : $default"
        echo "$default"
    fi
}

# Utilisation
echo "Configuration du système..."
name=$(read_with_timeout "Nom d'utilisateur (10s) : " 10 "admin")
echo "Nom configuré : $name"
```

## 3. Gestion des signaux

### Signaux courants

Les signaux les plus utilisés :
- **SIGINT** (2) : Ctrl+C
- **SIGTERM** (15) : Demande d'arrêt propre
- **SIGKILL** (9) : Arrêt forcé (ne peut pas être piégé)
- **SIGHUP** (1) : Déconnexion du terminal

### Piégeage des signaux avec trap

```bash
#!/bin/bash

# Fonction appelée lors d'une interruption
handle_interrupt() {
    echo ""
    echo "Interruption détectée (Ctrl+C)"
    echo "Voulez-vous vraiment quitter ? (oui/non)"
    read response

    case ${response,,} in
        oui|o|yes|y)
            echo "Arrêt du programme..."
            exit 0
            ;;
        *)
            echo "Reprise du programme..."
            return
            ;;
    esac
}

# Piéger le signal SIGINT (Ctrl+C)
trap handle_interrupt INT

echo "Programme en cours d'exécution..."
echo "Appuyez sur Ctrl+C pour tester la gestion des signaux"

# Boucle infinie pour tester
counter=1
while true; do
    echo "Itération $counter ($(date))"
    sleep 2
    ((counter++))
done
```

### Gestion avancée des signaux

```bash
#!/bin/bash

# Variables globales
TEMP_FILES=()
RUNNING=true

# Fonction de nettoyage
cleanup() {
    echo "Nettoyage en cours..."

    # Supprimer les fichiers temporaires
    for file in "${TEMP_FILES[@]}"; do
        if [ -f "$file" ]; then
            rm "$file"
            echo "Fichier supprimé : $file"
        fi
    done

    echo "Nettoyage terminé"
}

# Fonction pour arrêt propre
graceful_exit() {
    echo ""
    echo "Arrêt demandé..."
    RUNNING=false
    cleanup
    exit 0
}

# Fonction pour rechargement de configuration
reload_config() {
    echo "Rechargement de la configuration..."
    # Recharger ici votre configuration
    echo "Configuration rechargée"
}

# Configuration des signaux
trap graceful_exit INT TERM  # Ctrl+C et arrêt système
trap reload_config HUP       # Rechargement config
trap cleanup EXIT            # Nettoyage automatique

# Simulation d'un travail avec fichiers temporaires
echo "Démarrage du programme..."
echo "Signaux gérés : INT, TERM, HUP"
echo "PID du processus : $$"

# Créer des fichiers temporaires
for i in {1..3}; do
    temp_file="/tmp/script_temp_$i_$$"
    touch "$temp_file"
    TEMP_FILES+=("$temp_file")
    echo "Fichier temporaire créé : $temp_file"
done

# Boucle principale
counter=1
while $RUNNING; do
    echo "Travail en cours... ($counter)"
    sleep 3
    ((counter++))
done
```

### Test des signaux

```bash
# Dans un autre terminal, vous pouvez tester :
# kill -HUP <PID>     # Recharger la configuration
# kill -TERM <PID>    # Arrêt propre
# kill -INT <PID>     # Équivalent à Ctrl+C
```

## 4. Scripts avec options (getopts)

### Introduction à getopts

`getopts` permet de gérer les options de ligne de commande de manière professionnelle.

### Exemple basique

```bash
#!/bin/bash

# Fonction d'aide
show_help() {
    echo "Usage: $0 [-h] [-v] [-f fichier] [-n nombre]"
    echo "Options:"
    echo "  -h          Afficher cette aide"
    echo "  -v          Mode verbeux"
    echo "  -f fichier  Spécifier un fichier"
    echo "  -n nombre   Spécifier un nombre"
    exit 0
}

# Variables par défaut
VERBOSE=false
FILE=""
NUMBER=0

# Traitement des options
while getopts "hvf:n:" option; do
    case $option in
        h)
            show_help
            ;;
        v)
            VERBOSE=true
            ;;
        f)
            FILE="$OPTARG"
            ;;
        n)
            NUMBER="$OPTARG"
            ;;
        \?)
            echo "Option invalide: -$OPTARG"
            echo "Utilisez -h pour l'aide"
            exit 1
            ;;
        :)
            echo "L'option -$OPTARG nécessite un argument"
            exit 1
            ;;
    esac
done

# Décaler les arguments traités
shift $((OPTIND-1))

# Utilisation des options
if $VERBOSE; then
    echo "Mode verbeux activé"
    echo "Fichier : ${FILE:-"non spécifié"}"
    echo "Nombre : $NUMBER"
    echo "Arguments restants : $@"
fi

# Validation des options
if [ -n "$FILE" ] && [ ! -f "$FILE" ]; then
    echo "Erreur : Le fichier '$FILE' n'existe pas"
    exit 1
fi

if [ "$NUMBER" -lt 0 ]; then
    echo "Erreur : Le nombre doit être positif"
    exit 1
fi

echo "Traitement avec les paramètres configurés..."
```

### Exemple avancé : Outil de sauvegarde

```bash
#!/bin/bash

# Valeurs par défaut
SOURCE_DIR=""
BACKUP_DIR="/backup"
COMPRESSION=false
VERBOSE=false
DRY_RUN=false
EXCLUDE_PATTERNS=()

# Fonction d'aide
show_help() {
    cat << EOF
Usage: $0 [OPTIONS] -s SOURCE_DIR

Outil de sauvegarde avec options avancées

OPTIONS:
    -s SOURCE_DIR     Répertoire source (obligatoire)
    -d BACKUP_DIR     Répertoire de destination (défaut: /backup)
    -c                Activer la compression
    -v                Mode verbeux
    -n                Mode test (dry-run)
    -e PATTERN        Exclure les fichiers correspondant au motif
    -h                Afficher cette aide

EXEMPLES:
    $0 -s /home/user -d /backup -c -v
    $0 -s /var/www -e "*.log" -e "*.tmp" -n
    $0 -s /home/user -cvn

EOF
    exit 0
}

# Fonction de log
log() {
    if $VERBOSE; then
        echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
    fi
}

# Traitement des options
while getopts "s:d:cvne:h" option; do
    case $option in
        s)
            SOURCE_DIR="$OPTARG"
            ;;
        d)
            BACKUP_DIR="$OPTARG"
            ;;
        c)
            COMPRESSION=true
            ;;
        v)
            VERBOSE=true
            ;;
        n)
            DRY_RUN=true
            ;;
        e)
            EXCLUDE_PATTERNS+=("$OPTARG")
            ;;
        h)
            show_help
            ;;
        \?)
            echo "Option invalide: -$OPTARG"
            echo "Utilisez -h pour l'aide"
            exit 1
            ;;
        :)
            echo "L'option -$OPTARG nécessite un argument"
            exit 1
            ;;
    esac
done

# Validation des paramètres obligatoires
if [ -z "$SOURCE_DIR" ]; then
    echo "Erreur : Le répertoire source est obligatoire (-s)"
    echo "Utilisez -h pour l'aide"
    exit 1
fi

if [ ! -d "$SOURCE_DIR" ]; then
    echo "Erreur : Le répertoire source '$SOURCE_DIR' n'existe pas"
    exit 1
fi

# Affichage de la configuration
log "Configuration de la sauvegarde :"
log "  Source : $SOURCE_DIR"
log "  Destination : $BACKUP_DIR"
log "  Compression : $COMPRESSION"
log "  Mode test : $DRY_RUN"
log "  Motifs exclus : ${EXCLUDE_PATTERNS[*]}"

# Création du nom de sauvegarde
BACKUP_NAME="backup_$(basename "$SOURCE_DIR")_$(date +%Y%m%d_%H%M%S)"
if $COMPRESSION; then
    BACKUP_NAME="${BACKUP_NAME}.tar.gz"
else
    BACKUP_NAME="${BACKUP_NAME}.tar"
fi

BACKUP_PATH="$BACKUP_DIR/$BACKUP_NAME"

# Création du répertoire de destination
if $DRY_RUN; then
    log "MODE TEST : Création du répertoire $BACKUP_DIR"
else
    mkdir -p "$BACKUP_DIR"
    log "Répertoire de destination créé : $BACKUP_DIR"
fi

# Construction de la commande tar
TAR_CMD="tar"
if $COMPRESSION; then
    TAR_CMD="$TAR_CMD -czf"
else
    TAR_CMD="$TAR_CMD -cf"
fi

# Ajout des exclusions
for pattern in "${EXCLUDE_PATTERNS[@]}"; do
    TAR_CMD="$TAR_CMD --exclude=$pattern"
done

TAR_CMD="$TAR_CMD $BACKUP_PATH -C $(dirname "$SOURCE_DIR") $(basename "$SOURCE_DIR")"

# Exécution
if $DRY_RUN; then
    log "MODE TEST : Commande qui serait exécutée :"
    log "$TAR_CMD"
else
    log "Démarrage de la sauvegarde..."
    log "Commande : $TAR_CMD"

    if eval "$TAR_CMD"; then
        log "Sauvegarde terminée avec succès : $BACKUP_PATH"
        log "Taille : $(du -h "$BACKUP_PATH" | cut -f1)"
    else
        log "Erreur lors de la sauvegarde"
        exit 1
    fi
fi
```

### Utilisation du script de sauvegarde

```bash
# Exemples d'utilisation :

# Sauvegarde simple
./backup.sh -s /home/user

# Sauvegarde avec compression et mode verbeux
./backup.sh -s /home/user -c -v

# Sauvegarde avec exclusions
./backup.sh -s /var/www -e "*.log" -e "*.tmp" -e "cache/*"

# Mode test pour vérifier la commande
./backup.sh -s /home/user -cvn -d /tmp/backup
```

## Exemple complet : Gestionnaire de tâches interactif

```bash
#!/bin/bash

# Fichier de stockage des tâches
TASKS_FILE="$HOME/.tasks.txt"

# Couleurs pour l'affichage
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Fonction pour afficher avec couleur
color_echo() {
    local color=$1
    shift
    echo -e "${color}$@${NC}"
}

# Fonction de nettoyage
cleanup() {
    clear
    color_echo $GREEN "Merci d'avoir utilisé le gestionnaire de tâches !"
    exit 0
}

# Gestion des signaux
trap cleanup INT TERM

# Initialiser le fichier de tâches s'il n'existe pas
[ ! -f "$TASKS_FILE" ] && touch "$TASKS_FILE"

# Fonction pour afficher le menu
show_menu() {
    clear
    color_echo $BLUE "=================================="
    color_echo $BLUE "    GESTIONNAIRE DE TÂCHES"
    color_echo $BLUE "=================================="
    echo "1. Ajouter une tâche"
    echo "2. Lister les tâches"
    echo "3. Marquer une tâche comme terminée"
    echo "4. Supprimer une tâche"
    echo "5. Rechercher une tâche"
    echo "6. Quitter"
    color_echo $BLUE "=================================="
    echo -n "Votre choix (1-6) : "
}

# Fonction pour ajouter une tâche
add_task() {
    echo -n "Entrez la nouvelle tâche : "
    read task

    if [ -n "$task" ]; then
        echo "[$(date '+%Y-%m-%d %H:%M')] TODO: $task" >> "$TASKS_FILE"
        color_echo $GREEN "Tâche ajoutée avec succès !"
    else
        color_echo $RED "Tâche vide. Aucune tâche ajoutée."
    fi
}

# Fonction pour lister les tâches
list_tasks() {
    if [ ! -s "$TASKS_FILE" ]; then
        color_echo $YELLOW "Aucune tâche trouvée."
        return
    fi

    color_echo $BLUE "LISTE DES TÂCHES :"
    color_echo $BLUE "=================="

    local count=1
    while IFS= read -r line; do
        if [[ $line == *"TODO:"* ]]; then
            color_echo $YELLOW "$count. $line"
        else
            color_echo $GREEN "$count. $line"
        fi
        ((count++))
    done < "$TASKS_FILE"
}

# Fonction pour marquer une tâche comme terminée
complete_task() {
    list_tasks

    if [ ! -s "$TASKS_FILE" ]; then
        return
    fi

    echo -n "Numéro de la tâche à marquer comme terminée : "
    read task_num

    if ! [[ "$task_num" =~ ^[0-9]+$ ]]; then
        color_echo $RED "Numéro invalide."
        return
    fi

    local total_lines=$(wc -l < "$TASKS_FILE")
    if [ "$task_num" -gt "$total_lines" ] || [ "$task_num" -lt 1 ]; then
        color_echo $RED "Numéro de tâche invalide."
        return
    fi

    # Marquer la tâche comme terminée
    sed -i "${task_num}s/TODO:/DONE:/" "$TASKS_FILE"
    color_echo $GREEN "Tâche marquée comme terminée !"
}

# Fonction pour supprimer une tâche
delete_task() {
    list_tasks

    if [ ! -s "$TASKS_FILE" ]; then
        return
    fi

    echo -n "Numéro de la tâche à supprimer : "
    read task_num

    if ! [[ "$task_num" =~ ^[0-9]+$ ]]; then
        color_echo $RED "Numéro invalide."
        return
    fi

    local total_lines=$(wc -l < "$TASKS_FILE")
    if [ "$task_num" -gt "$total_lines" ] || [ "$task_num" -lt 1 ]; then
        color_echo $RED "Numéro de tâche invalide."
        return
    fi

    # Confirmation
    echo -n "Êtes-vous sûr de vouloir supprimer cette tâche ? (oui/non) : "
    read confirmation

    case ${confirmation,,} in
        oui|o|yes|y)
            sed -i "${task_num}d" "$TASKS_FILE"
            color_echo $GREEN "Tâche supprimée !"
            ;;
        *)
            color_echo $YELLOW "Suppression annulée."
            ;;
    esac
}

# Fonction pour rechercher une tâche
search_task() {
    echo -n "Terme de recherche : "
    read search_term

    if [ -z "$search_term" ]; then
        color_echo $RED "Terme de recherche vide."
        return
    fi

    color_echo $BLUE "RÉSULTATS DE LA RECHERCHE :"
    color_echo $BLUE "=========================="

    local count=0
    local line_num=1

    while IFS= read -r line; do
        if [[ $line == *"$search_term"* ]]; then
            if [[ $line == *"TODO:"* ]]; then
                color_echo $YELLOW "$line_num. $line"
            else
                color_echo $GREEN "$line_num. $line"
            fi
            ((count++))
        fi
        ((line_num++))
    done < "$TASKS_FILE"

    if [ $count -eq 0 ]; then
        color_echo $YELLOW "Aucune tâche trouvée contenant '$search_term'."
    else
        color_echo $BLUE "$count tâche(s) trouvée(s)."
    fi
}

# Fonction principale
main() {
    while true; do
        show_menu
        read choice

        case $choice in
            1)
                add_task
                ;;
            2)
                list_tasks
                ;;
            3)
                complete_task
                ;;
            4)
                delete_task
                ;;
            5)
                search_task
                ;;
            6)
                cleanup
                ;;
            *)
                color_echo $RED "Choix invalide. Veuillez choisir entre 1 et 6."
                ;;
        esac

        echo ""
        echo "Appuyez sur Entrée pour continuer..."
        read
    done
}

# Démarrage du programme
main
```

## Bonnes pratiques

### 1. Validation systématique des entrées
```bash
# Toujours valider les entrées utilisateur
validate_input() {
    local input=$1
    # Votre logique de validation ici
    return 0  # ou 1 selon la validation
}
```

### 2. Messages d'erreur clairs
```bash
# Bon
echo "Erreur : Le fichier '$filename' n'existe pas."

# Mauvais
echo "Erreur fichier"
```

### 3. Gestion propre des signaux
```bash
# Toujours nettoyer les ressources
trap cleanup EXIT INT TERM
```

### 4. Interface utilisateur intuitive
```bash
# Utiliser des menus clairs avec des options numérotées
# Fournir de l'aide contextuelle
# Confirmer les actions destructrices
```

## Résumé

Les scripts interactifs et les options permettent de créer des outils professionnels :

- **Menus** : Utilisez `select` ou `read` pour créer des interfaces utilisateur
- **Validation** : Vérifiez toujours les entrées utilisateur
- **Signaux** : Gérez les interruptions avec `trap`
- **Options** : Utilisez `getopts` pour des options de ligne de commande professionnelles

Ces techniques transforment vos scripts simples en outils interactifs robustes et professionnels.

⏭️
