🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Sécurité des scripts Bash

## Introduction

La sécurité est un aspect crucial du développement de scripts Bash, surtout lorsqu'ils sont utilisés en production ou avec des privilèges élevés. Cette section vous apprendra à identifier et prévenir les vulnérabilités courantes, à valider les entrées utilisateur, et à implémenter des pratiques de sécurité robustes.

## 1. Validation et assainissement des entrées

### Pourquoi valider les entrées ?

Les entrées non validées sont la source principale des vulnérabilités de sécurité. Un attaquant peut exploiter ces failles pour :
- Exécuter des commandes arbitraires
- Accéder à des fichiers sensibles
- Modifier des données critiques
- Compromettre le système

### Validation basique des entrées

```bash
#!/bin/bash

# === FONCTIONS DE VALIDATION ===

# Validation d'un nom d'utilisateur
validate_username() {
    local username="$1"

    # Vérifications de base
    if [ -z "$username" ]; then
        echo "Erreur: Nom d'utilisateur vide"
        return 1
    fi

    # Longueur appropriée
    if [ ${#username} -lt 3 ] || [ ${#username} -gt 32 ]; then
        echo "Erreur: Nom d'utilisateur doit faire entre 3 et 32 caractères"
        return 1
    fi

    # Caractères autorisés uniquement (alphanumériques, tiret, underscore)
    if ! [[ "$username" =~ ^[a-zA-Z0-9_-]+$ ]]; then
        echo "Erreur: Nom d'utilisateur contient des caractères invalides"
        return 1
    fi

    # Ne doit pas commencer par un chiffre ou un tiret
    if [[ "$username" =~ ^[0-9-] ]]; then
        echo "Erreur: Nom d'utilisateur ne peut pas commencer par un chiffre ou un tiret"
        return 1
    fi

    echo "Nom d'utilisateur valide: $username"
    return 0
}

# Validation d'une adresse email
validate_email() {
    local email="$1"

    # Vérification de base
    if [ -z "$email" ]; then
        echo "Erreur: Email vide"
        return 1
    fi

    # Longueur maximale raisonnable
    if [ ${#email} -gt 254 ]; then
        echo "Erreur: Email trop long"
        return 1
    fi

    # Format de base avec regex
    local email_regex='^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    if ! [[ "$email" =~ $email_regex ]]; then
        echo "Erreur: Format d'email invalide"
        return 1
    fi

    # Vérifications supplémentaires
    if [[ "$email" == *".."* ]] || [[ "$email" == *"@."* ]] || [[ "$email" == *".@"* ]]; then
        echo "Erreur: Email contient des points consécutifs invalides"
        return 1
    fi

    echo "Email valide: $email"
    return 0
}

# Validation d'un chemin de fichier
validate_file_path() {
    local file_path="$1"
    local base_dir="${2:-/home}"  # Répertoire de base autorisé

    # Vérification de base
    if [ -z "$file_path" ]; then
        echo "Erreur: Chemin de fichier vide"
        return 1
    fi

    # Longueur raisonnable
    if [ ${#file_path} -gt 4096 ]; then
        echo "Erreur: Chemin de fichier trop long"
        return 1
    fi

    # Interdire les caractères dangereux
    if [[ "$file_path" =~ [[:cntrl:]] ]]; then
        echo "Erreur: Chemin contient des caractères de contrôle"
        return 1
    fi

    # Résoudre le chemin absolu pour éviter les attaques de traversée
    local real_path
    if ! real_path=$(realpath -m "$file_path" 2>/dev/null); then
        echo "Erreur: Impossible de résoudre le chemin"
        return 1
    fi

    # Vérifier que le chemin reste dans le répertoire autorisé
    local real_base
    if ! real_base=$(realpath -m "$base_dir" 2>/dev/null); then
        echo "Erreur: Répertoire de base invalide"
        return 1
    fi

    # Vérification de traversée de répertoire
    if [[ "$real_path" != "$real_base"* ]]; then
        echo "Erreur: Tentative de sortie du répertoire autorisé"
        echo "Chemin demandé: $real_path"
        echo "Répertoire autorisé: $real_base"
        return 1
    fi

    echo "Chemin valide: $real_path"
    return 0
}

# Validation d'un nombre entier
validate_integer() {
    local value="$1"
    local min="${2:--2147483648}"  # int32 min par défaut
    local max="${3:-2147483647}"   # int32 max par défaut

    # Vérification de base
    if [ -z "$value" ]; then
        echo "Erreur: Valeur vide"
        return 1
    fi

    # Vérifier que c'est un nombre entier
    if ! [[ "$value" =~ ^-?[0-9]+$ ]]; then
        echo "Erreur: '$value' n'est pas un nombre entier"
        return 1
    fi

    # Vérifier les limites
    if [ "$value" -lt "$min" ] || [ "$value" -gt "$max" ]; then
        echo "Erreur: Valeur $value hors limites [$min, $max]"
        return 1
    fi

    echo "Nombre valide: $value"
    return 0
}

# Exemple d'utilisation
demo_validation() {
    echo "=== DÉMONSTRATION DE VALIDATION ==="

    # Test de validation de nom d'utilisateur
    echo "Test nom d'utilisateur:"
    validate_username "alice123" && echo "✓ alice123 accepté"
    validate_username "123invalid" && echo "✗ Erreur non détectée" || echo "✓ 123invalid rejeté"
    validate_username "user@invalid" && echo "✗ Erreur non détectée" || echo "✓ user@invalid rejeté"

    echo ""

    # Test de validation de chemin
    echo "Test chemin de fichier:"
    validate_file_path "/home/user/document.txt" "/home" && echo "✓ Chemin valide accepté"
    validate_file_path "/home/user/../../../etc/passwd" "/home" && echo "✗ Erreur non détectée" || echo "✓ Tentative de traversée rejetée"

    echo ""

    # Test de validation de nombre
    echo "Test nombre entier:"
    validate_integer "42" 1 100 && echo "✓ 42 accepté"
    validate_integer "150" 1 100 && echo "✗ Erreur non détectée" || echo "✓ 150 rejeté (hors limites)"
}
```

### Assainissement des entrées

```bash
#!/bin/bash

# === FONCTIONS D'ASSAINISSEMENT ===

# Nettoyer une chaîne pour l'affichage
sanitize_for_display() {
    local input="$1"

    # Supprimer les caractères de contrôle
    local sanitized="${input//[[:cntrl:]]/ }"

    # Limiter la longueur pour éviter l'overflow d'affichage
    if [ ${#sanitized} -gt 100 ]; then
        sanitized="${sanitized:0:97}..."
    fi

    echo "$sanitized"
}

# Nettoyer un nom de fichier
sanitize_filename() {
    local filename="$1"

    # Supprimer les caractères dangereux et les espaces
    local sanitized="${filename//[^a-zA-Z0-9._-]/}"

    # Éviter les noms de fichiers qui commencent par un point (fichiers cachés)
    if [[ "$sanitized" == .* ]]; then
        sanitized="file_$sanitized"
    fi

    # Éviter les noms réservés
    case "$sanitized" in
        ""|"."|".."|"con"|"prn"|"aux"|"nul"|"com"[1-9]|"lpt"[1-9])
            sanitized="file_$sanitized"
            ;;
    esac

    # Limiter la longueur
    if [ ${#sanitized} -gt 255 ]; then
        sanitized="${sanitized:0:255}"
    fi

    # Assurer qu'il y a au moins un caractère
    if [ -z "$sanitized" ]; then
        sanitized="unnamed_file"
    fi

    echo "$sanitized"
}

# Échapper une chaîne pour l'utilisation dans une commande shell
escape_for_shell() {
    local input="$1"

    # Utiliser printf %q pour un échappement sûr
    printf '%q' "$input"
}

# Nettoyer une entrée pour l'utilisation dans une requête SQL (basique)
sanitize_for_sql() {
    local input="$1"

    # Échapper les apostrophes (simple protection, utiliser des requêtes préparées en production)
    local sanitized="${input//\'/\'\'}"

    # Supprimer les commentaires SQL potentiels
    sanitized="${sanitized//--/}"
    sanitized="${sanitized//\/\*/}"
    sanitized="${sanitized//\*\//}"

    echo "$sanitized"
}

# Exemple d'utilisation sécurisée
secure_file_operations() {
    local user_input="$1"

    echo "=== OPÉRATIONS SÉCURISÉES SUR FICHIERS ==="

    # Valider le chemin
    if ! validate_file_path "$user_input" "/tmp/safe_area"; then
        echo "Chemin rejeté pour des raisons de sécurité"
        return 1
    fi

    # Assainir le nom de fichier
    local safe_filename
    safe_filename=$(sanitize_filename "$(basename "$user_input")")
    local safe_path="/tmp/safe_area/$safe_filename"

    echo "Chemin original: $user_input"
    echo "Chemin sécurisé: $safe_path"

    # Créer le fichier de manière sécurisée
    if touch "$safe_path"; then
        echo "Fichier créé en sécurité: $safe_path"
    else
        echo "Erreur lors de la création du fichier"
        return 1
    fi
}
```

## 2. Gestion des permissions des fichiers et scripts

### Principes de sécurité des permissions

```bash
#!/bin/bash

# === GESTION SÉCURISÉE DES PERMISSIONS ===

# Configuration sécurisée des permissions de script
secure_script_setup() {
    local script_path="$1"

    echo "Configuration sécurisée pour: $script_path"

    # Vérifier que le script nous appartient
    if [ ! -O "$script_path" ]; then
        echo "ATTENTION: Le script n'appartient pas à l'utilisateur actuel"
        return 1
    fi

    # Définir des permissions sécurisées (propriétaire: lecture/écriture/exécution, autres: rien)
    chmod 700 "$script_path"
    echo "Permissions définies: 700 (rwx------)"

    # Vérifier que le répertoire parent est sécurisé
    local script_dir
    script_dir=$(dirname "$script_path")

    if [ -w "$script_dir" ] && [ ! -O "$script_dir" ]; then
        echo "ATTENTION: Le répertoire parent est accessible en écriture par d'autres"
        echo "Répertoire: $script_dir"
    fi

    # Afficher les permissions actuelles
    ls -la "$script_path"
}

# Création sécurisée de fichiers temporaires
create_secure_temp_file() {
    local prefix="${1:-secure_tmp}"

    # Utiliser mktemp pour créer un fichier temporaire sécurisé
    local temp_file
    if ! temp_file=$(mktemp "/tmp/${prefix}.XXXXXX"); then
        echo "Erreur: Impossible de créer un fichier temporaire"
        return 1
    fi

    # Définir des permissions restrictives immédiatement
    chmod 600 "$temp_file"

    echo "Fichier temporaire sécurisé créé: $temp_file"
    echo "$temp_file"
}

# Création sécurisée de répertoires
create_secure_directory() {
    local dir_path="$1"
    local permissions="${2:-700}"

    # Créer le répertoire avec des permissions sécurisées
    if mkdir -p "$dir_path"; then
        chmod "$permissions" "$dir_path"
        echo "Répertoire créé avec permissions $permissions: $dir_path"

        # Vérifier les permissions finales
        ls -ld "$dir_path"
        return 0
    else
        echo "Erreur: Impossible de créer le répertoire $dir_path"
        return 1
    fi
}

# Vérification de sécurité des fichiers
check_file_security() {
    local file_path="$1"

    echo "=== AUDIT DE SÉCURITÉ: $file_path ==="

    if [ ! -e "$file_path" ]; then
        echo "ERREUR: Fichier inexistant"
        return 1
    fi

    # Vérifier les permissions
    local perms
    perms=$(stat -c "%a" "$file_path")
    echo "Permissions: $perms"

    # Alertes de sécurité
    if [ "$((perms % 10))" -ne 0 ]; then
        echo "⚠️  ALERTE: Le fichier est accessible par 'others'"
    fi

    if [ "$(((perms / 10) % 10))" -gt 5 ]; then
        echo "⚠️  ALERTE: Le fichier est accessible en écriture par le groupe"
    fi

    if [ "$(((perms / 100) % 10))" -eq 7 ] && [ "$((perms % 100))" -ne 0 ]; then
        echo "⚠️  ALERTE: Fichier exécutable avec permissions larges"
    fi

    # Vérifier le propriétaire
    local owner
    owner=$(stat -c "%U" "$file_path")
    echo "Propriétaire: $owner"

    if [ "$owner" != "$(whoami)" ] && [ "$(id -u)" -ne 0 ]; then
        echo "⚠️  ALERTE: Vous n'êtes pas le propriétaire de ce fichier"
    fi

    # Vérifier les liens symboliques
    if [ -L "$file_path" ]; then
        local target
        target=$(readlink "$file_path")
        echo "Lien symbolique vers: $target"

        if [ ! -e "$target" ]; then
            echo "⚠️  ALERTE: Lien symbolique brisé"
        fi
    fi
}

# Configuration umask sécurisée
set_secure_umask() {
    # umask 022 : fichiers créés avec 644, répertoires avec 755
    # umask 077 : fichiers créés avec 600, répertoires avec 700 (plus sécurisé)

    local old_umask
    old_umask=$(umask)

    echo "Umask actuel: $old_umask"

    # Définir un umask sécurisé
    umask 077

    echo "Nouveau umask: $(umask)"
    echo "Nouveaux fichiers seront créés avec permissions 600"
    echo "Nouveaux répertoires seront créés avec permissions 700"
}
```

## 3. Éviter les injections de commandes

### Vulnérabilités courantes

```bash
#!/bin/bash

# === EXEMPLES DE VULNÉRABILITÉS (À ÉVITER) ===

# ❌ DANGEREUX - Injection de commande possible
dangerous_file_operation() {
    local filename="$1"

    # Un attaquant pourrait passer: "file.txt; rm -rf /"
    eval "ls -la $filename"  # TRÈS DANGEREUX !
}

# ❌ DANGEREUX - Utilisation non sécurisée de l'entrée utilisateur
dangerous_search() {
    local search_term="$1"

    # Un attaquant pourrait passer: "term; cat /etc/passwd"
    local command="grep '$search_term' /var/log/app.log"
    bash -c "$command"  # DANGEREUX !
}

# ❌ DANGEREUX - Construction de commande avec concaténation
dangerous_backup() {
    local source_dir="$1"
    local backup_name="$2"

    # Un attaquant pourrait passer des caractères spéciaux
    tar -czf "/backup/$backup_name.tar.gz" $source_dir  # DANGEREUX !
}
```

### Techniques sécurisées

```bash
#!/bin/bash

# === TECHNIQUES SÉCURISÉES ===

# ✅ SÉCURISÉ - Utilisation de tableaux pour les arguments
secure_file_operation() {
    local filename="$1"

    # Valider l'entrée
    if ! validate_file_path "$filename"; then
        echo "Nom de fichier invalide"
        return 1
    fi

    # Utiliser des tableaux pour éviter l'injection
    local cmd=(ls -la "$filename")
    "${cmd[@]}"
}

# ✅ SÉCURISÉ - Éviter eval et bash -c
secure_search() {
    local search_term="$1"
    local log_file="/var/log/app.log"

    # Valider l'entrée
    if [ -z "$search_term" ] || [[ "$search_term" =~ [;&|`$] ]]; then
        echo "Terme de recherche invalide"
        return 1
    fi

    # Utiliser grep directement avec des arguments sécurisés
    grep -F "$search_term" "$log_file"
}

# ✅ SÉCURISÉ - Validation et échappement appropriés
secure_backup() {
    local source_dir="$1"
    local backup_name="$2"

    # Valider les entrées
    if ! validate_file_path "$source_dir"; then
        echo "Répertoire source invalide"
        return 1
    fi

    # Assainir le nom de sauvegarde
    backup_name=$(sanitize_filename "$backup_name")
    if [ -z "$backup_name" ]; then
        echo "Nom de sauvegarde invalide"
        return 1
    fi

    # Utiliser des variables correctement échappées
    local backup_path="/backup/${backup_name}.tar.gz"

    # Commande avec arguments sécurisés
    tar -czf "$backup_path" -C "$(dirname "$source_dir")" "$(basename "$source_dir")"
}

# ✅ SÉCURISÉ - Exécution de commandes avec validation stricte
secure_command_execution() {
    local user_command="$1"
    shift
    local args=("$@")

    # Liste blanche de commandes autorisées
    local allowed_commands=(
        "ls"
        "cat"
        "grep"
        "wc"
        "sort"
        "uniq"
    )

    # Vérifier que la commande est autorisée
    local command_allowed=false
    for allowed in "${allowed_commands[@]}"; do
        if [ "$user_command" = "$allowed" ]; then
            command_allowed=true
            break
        fi
    done

    if ! $command_allowed; then
        echo "Commande non autorisée: $user_command"
        return 1
    fi

    # Valider tous les arguments
    for arg in "${args[@]}"; do
        if [[ "$arg" =~ [;&|`$] ]]; then
            echo "Argument dangereux détecté: $arg"
            return 1
        fi
    done

    # Exécuter la commande de manière sécurisée
    "$user_command" "${args[@]}"
}

# ✅ SÉCURISÉ - Traitement de fichiers avec find
secure_file_processing() {
    local base_dir="$1"
    local pattern="$2"

    # Valider le répertoire de base
    if ! validate_file_path "$base_dir"; then
        echo "Répertoire de base invalide"
        return 1
    fi

    # Valider le motif (éviter les caractères d'injection)
    if [[ "$pattern" =~ [;&|`$] ]]; then
        echo "Motif de recherche dangereux"
        return 1
    fi

    # Utiliser find avec -print0 et read pour gérer les noms de fichiers avec espaces
    while IFS= read -r -d '' file; do
        echo "Traitement du fichier: $file"

        # Traitement sécurisé de chaque fichier
        wc -l "$file"

    done < <(find "$base_dir" -name "$pattern" -type f -print0)
}
```

### Protection contre l'injection dans les scripts web

```bash
#!/bin/bash

# === SÉCURITÉ POUR LES SCRIPTS CGI/WEB ===

# Nettoyer les variables CGI
sanitize_cgi_input() {
    local input="$1"

    # Supprimer les caractères dangereux pour le web
    local sanitized="${input//[<>&\"\']/}"

    # Supprimer les séquences d'injection SQL basiques
    sanitized="${sanitized//union/}"
    sanitized="${sanitized//select/}"
    sanitized="${sanitized//insert/}"
    sanitized="${sanitized//delete/}"
    sanitized="${sanitized//drop/}"

    # Supprimer les tentatives d'injection de commandes
    sanitized="${sanitized//[;&|`$()]/}"

    # Décoder les entités URL basiques
    sanitized="${sanitized//%20/ }"
    sanitized="${sanitized//%3C/<}"
    sanitized="${sanitized//%3E/>}"

    echo "$sanitized"
}

# Traitement sécurisé des paramètres GET/POST
process_web_input() {
    local action="$1"
    local username="$2"
    local data="$3"

    echo "Content-Type: text/plain"
    echo ""

    # Valider l'action
    case "$action" in
        "view"|"edit"|"delete")
            ;;
        *)
            echo "Action non autorisée: $action"
            exit 1
            ;;
    esac

    # Valider et assainir le nom d'utilisateur
    username=$(sanitize_cgi_input "$username")
    if ! validate_username "$username"; then
        echo "Nom d'utilisateur invalide"
        exit 1
    fi

    # Valider et assainir les données
    data=$(sanitize_cgi_input "$data")

    # Traitement sécurisé selon l'action
    case "$action" in
        "view")
            # Afficher les informations utilisateur de manière sécurisée
            if [ -f "/var/data/users/$username.txt" ]; then
                cat "/var/data/users/$username.txt"
            else
                echo "Utilisateur non trouvé"
            fi
            ;;
        "edit")
            # Éditer les données de manière sécurisée
            echo "Modification des données pour $username"
            # Implémentation sécurisée ici
            ;;
        "delete")
            # Supprimer de manière sécurisée
            echo "Suppression demandée pour $username"
            # Implémentation sécurisée avec confirmation
            ;;
    esac
}
```

## 4. Gestion sécurisée des variables sensibles

### Stockage sécurisé des mots de passe et clés

```bash
#!/bin/bash

# === GESTION SÉCURISÉE DES SECRETS ===

# Configuration pour la gestion des secrets
readonly SECRETS_DIR="/etc/secrets"
readonly KEY_FILE="$SECRETS_DIR/master.key"

# Création d'un répertoire sécurisé pour les secrets
setup_secrets_directory() {
    # Créer le répertoire avec des permissions restrictives
    if [ ! -d "$SECRETS_DIR" ]; then
        sudo mkdir -p "$SECRETS_DIR"
        sudo chmod 700 "$SECRETS_DIR"
        sudo chown root:root "$SECRETS_DIR"
        echo "Répertoire des secrets créé: $SECRETS_DIR"
    fi
}

# ✅ SÉCURISÉ - Lecture de mot de passe sans écho
read_password_securely() {
    local prompt="${1:-Mot de passe: }"
    local password=""

    # Désactiver l'écho terminal
    echo -n "$prompt"
    read -s password
    echo  # Nouvelle ligne après la saisie

    # Valider que le mot de passe n'est pas vide
    if [ -z "$password" ]; then
        echo "Mot de passe vide non autorisé"
        return 1
    fi

    # Retourner le mot de passe via une variable globale (éviter echo)
    SECURE_PASSWORD="$password"
    return 0
}

# ✅ SÉCURISÉ - Stockage chiffré de secrets
store_secret() {
    local secret_name="$1"
    local secret_value="$2"

    # Valider le nom du secret
    if ! [[ "$secret_name" =~ ^[a-zA-Z0-9_-]+$ ]]; then
        echo "Nom de secret invalide"
        return 1
    fi

    setup_secrets_directory

    local secret_file="$SECRETS_DIR/$secret_name.enc"

    # Chiffrer le secret avec GPG (ou openssl)
    if command -v gpg >/dev/null 2>&1; then
        echo "$secret_value" | gpg --symmetric --cipher-algo AES256 --output "$secret_file"
    elif command -v openssl >/dev/null 2>&1; then
        echo "$secret_value" | openssl enc -aes-256-cbc -salt -out "$secret_file"
    else
        echo "Aucun outil de chiffrement disponible"
        return 1
    fi

    # Définir des permissions sécurisées
    chmod 600 "$secret_file"

    # Effacer la variable
    secret_value=""
    unset secret_value

    echo "Secret stocké de manière chiffrée: $secret_file"
}

# ✅ SÉCURISÉ - Lecture de secrets chiffrés
read_secret() {
    local secret_name="$1"
    local secret_file="$SECRETS_DIR/$secret_name.enc"

    if [ ! -f "$secret_file" ]; then
        echo "Secret non trouvé: $secret_name"
        return 1
    fi

    # Déchiffrer le secret
    local secret_value=""
    if command -v gpg >/dev/null 2>&1; then
        secret_value=$(gpg --quiet --decrypt "$secret_file" 2>/dev/null)
    elif command -v openssl >/dev/null 2>&1; then
        secret_value=$(openssl enc -aes-256-cbc -d -in "$secret_file" 2>/dev/null)
    else
        echo "Aucun outil de déchiffrement disponible"
        return 1
    fi

    if [ -z "$secret_value" ]; then
        echo "Erreur de déchiffrement"
        return 1
    fi

    # Retourner via variable globale
    SECURE_SECRET="$secret_value"
    return 0
}

# ✅ SÉCURISÉ - Nettoyage de mémoire pour les variables sensibles
clear_sensitive_variables() {
    # Effacer les variables sensibles
    SECURE_PASSWORD=""
    SECURE_SECRET=""
    unset SECURE_PASSWORD
    unset SECURE_SECRET

    # Effacer d'autres variables potentiellement sensibles
    local vars_to_clear=(
        "PASSWORD"
        "SECRET"
        "TOKEN"
        "API_KEY"
        "PRIVATE_KEY"
    )

    for var in "${vars_to_clear[@]}"; do
        if [ -n "${!var:-}" ]; then
            eval "$var="
            unset "$var"
        fi
    done

    echo "Variables sensibles nettoyées"
}

# Configuration de trap pour nettoyer automatiquement
setup_security_traps() {
    trap 'clear_sensitive_variables' EXIT INT TERM
}
```

### Utilisation sécurisée des variables d'environnement

```bash
#!/bin/bash

# === VARIABLES D'ENVIRONNEMENT SÉCURISÉES ===

# ❌ DANGEREUX - Variables sensibles en clair
setup_database_connection_unsafe() {
    # Ces variables sont visibles dans 'ps aux' et l'environnement
    export DB_PASSWORD="motdepasse123"
    export API_KEY="secret-api-key"

    # Connexion à la base de données
    mysql -u user -p"$DB_PASSWORD" database
}

# ✅ SÉCURISÉ - Utilisation de fichiers de configuration
setup_database_connection_secure() {
    local config_file="/etc/app/database.conf"

    # Vérifier les permissions du fichier de configuration
    if [ ! -f "$config_file" ]; then
        echo "Fichier de configuration non trouvé: $config_file"
        return 1
    fi

    # Vérifier que le fichier n'est lisible que par le propriétaire
    local perms
    perms=$(stat -c "%a" "$config_file")
    if [ "$perms" != "600" ]; then
        echo "ATTENTION: Permissions du fichier de configuration trop larges ($perms)"
        echo "Utilisez: chmod 600 $config_file"
        return 1
    fi

    # Lire les paramètres de manière sécurisée
    local db_host db_user db_password

    # Éviter 'source' qui peut exécuter du code arbitraire
    while IFS='=' read -r key value; do
        # Ignorer les commentaires et lignes vides
        [[ "$key" =~ ^[[:space:]]*# ]] && continue
        [[ -z "$key" ]] && continue

        # Nettoyer les espaces
        key=$(echo "$key" | tr -d '[:space:]')
        value=$(echo "$value" | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')

        case "$key" in
            DB_HOST) db_host="$value" ;;
            DB_USER) db_user="$value" ;;
            DB_PASSWORD) db_password="$value" ;;
        esac
    done < "$config_file"

    # Valider les paramètres requis
    if [ -z "$db_host" ] || [ -z "$db_user" ] || [ -z "$db_password" ]; then
        echo "Paramètres de base de données incomplets"
        return 1
    fi

    # Utiliser les paramètres sans les exporter
    mysql -h "$db_host" -u "$db_user" -p"$db_password" database

    # Nettoyer immédiatement les variables
    db_password=""
    unset db_password
}

# ✅ SÉCURISÉ - Utilisation de variables temporaires
use_api_key_securely() {
    local api_endpoint="$1"
    local data="$2"

    # Lire la clé API de manière sécurisée
    if ! read_secret "api_key"; then
        echo "Impossible de lire la clé API"
        return 1
    fi

    # Utiliser la clé sans l'exposer dans l'environnement
    local response
    response=$(curl -s \
        -H "Authorization: Bearer $SECURE_SECRET" \
        -H "Content-Type: application/json" \
        -d "$data" \
        "$api_endpoint")

    # Nettoyer immédiatement
    clear_sensitive_variables

    echo "$response"
}

# ✅ SÉCURISÉ - Masquage des variables sensibles dans les logs
log_safely() {
    local message="$1"
    local log_file="${2:-/var/log/app.log}"

    # Masquer les motifs sensibles dans les logs
    local safe_message="$message"

    # Masquer les mots de passe
    safe_message=$(echo "$safe_message" | sed -E 's/(password|pwd|pass)[[:space:]]*[:=][[:space:]]*[^[:space:]]+/\1=***MASKED***/gi')

    # Masquer les clés API
    safe_message=$(echo "$safe_message" | sed -E 's/(api[_-]?key|token)[[:space:]]*[:=][[:space:]]*[^[:space:]]+/\1=***MASKED***/gi')

    # Masquer les URLs avec authentification
    safe_message=$(echo "$safe_message" | sed -E 's|://[^:]+:[^@]+@|://***:***@|g')

    # Écrire dans le log
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $safe_message" >> "$log_file"
}

# Vérification de la sécurité de l'environnement
check_environment_security() {
    echo "=== AUDIT DE SÉCURITÉ DE L'ENVIRONNEMENT ==="

    # Vérifier les variables d'environnement sensibles
    local sensitive_patterns=(
        "PASSWORD"
        "PASSWD"
        "SECRET"
        "KEY"
        "TOKEN"
        "CREDENTIAL"
    )

    echo "Variables d'environnement potentiellement sensibles:"
    for pattern in "${sensitive_patterns[@]}"; do
        env | grep -i "$pattern" | while read -r var; do
            local var_name="${var%%=*}"
            echo "⚠️  Variable sensible détectée: $var_name"
        done
    done

    # Vérifier les permissions des fichiers de configuration
    local config_files=(
        "/etc/passwd"
        "/etc/shadow"
        "/etc/ssh/ssh_host_*_key"
        "/root/.ssh/id_*"
        "/home/*/.ssh/id_*"
    )

    echo -e "\nVérification des fichiers sensibles:"
    for pattern in "${config_files[@]}"; do
        for file in $pattern; do
            if [ -f "$file" ]; then
                local perms
                perms=$(stat -c "%a" "$file")
                if [ "${perms: -1}" != "0" ]; then
                    echo "⚠️  Fichier sensible accessible par 'others': $file ($perms)"
                fi
            fi
        done
    done

    # Vérifier l'historique des commandes
    if [ -f "$HOME/.bash_history" ]; then
        echo -e "\nVérification de l'historique:"
        if grep -qi "password\|secret\|key" "$HOME/.bash_history"; then
            echo "⚠️  L'historique contient potentiellement des informations sensibles"
            echo "Considérez l'utilisation de HISTCONTROL=ignorespace"
        fi
    fi
}
```

## Exemples pratiques de scripts sécurisés

### Script d'authentification sécurisé

```bash
#!/bin/bash

# === SCRIPT D'AUTHENTIFICATION SÉCURISÉ ===

set -euo pipefail

readonly SCRIPT_NAME="$(basename "$0")"
readonly USERS_DB="/etc/secure_app/users.db"
readonly LOG_FILE="/var/log/auth_secure.log"
readonly MAX_LOGIN_ATTEMPTS=3
readonly LOCKOUT_DURATION=300  # 5 minutes

# Configuration de sécurité
setup_security_traps
set_secure_umask

# Fonction de log sécurisé
secure_log() {
    local level="$1"
    local message="$2"
    local user="${3:-unknown}"
    local ip="${4:-unknown}"

    # Assainir les entrées pour le log
    user=$(sanitize_for_display "$user")
    ip=$(sanitize_for_display "$ip")
    message=$(sanitize_for_display "$message")

    local log_entry="[$(date '+%Y-%m-%d %H:%M:%S')] [$level] User:$user IP:$ip - $message"
    echo "$log_entry" >> "$LOG_FILE"

    # Aussi afficher les erreurs
    if [ "$level" = "ERROR" ] || [ "$level" = "SECURITY" ]; then
        echo "$log_entry" >&2
    fi
}

# Validation d'adresse IP
validate_ip_address() {
    local ip="$1"

    # Format IPv4 basique
    if [[ $ip =~ ^([0-9]{1,3}\.){3}[0-9]{1,3}$ ]]; then
        # Vérifier que chaque octet est <= 255
        IFS='.' read -ra octets <<< "$ip"
        for octet in "${octets[@]}"; do
            if [ "$octet" -gt 255 ] || [ "$octet" -lt 0 ]; then
                return 1
            fi
        done
        return 0
    fi

    return 1
}

# Vérification des tentatives de connexion
check_login_attempts() {
    local username="$1"
    local client_ip="$2"
    local attempts_file="/tmp/login_attempts_${username}_${client_ip//\./_}"

    # Nettoyer les anciens fichiers de tentatives
    find /tmp -name "login_attempts_*" -mmin +10 -delete 2>/dev/null || true

    if [ -f "$attempts_file" ]; then
        local attempts
        attempts=$(cat "$attempts_file" 2>/dev/null || echo "0")

        if [ "$attempts" -ge "$MAX_LOGIN_ATTEMPTS" ]; then
            secure_log "SECURITY" "Account locked due to too many failed attempts" "$username" "$client_ip"
            echo "Compte temporairement verrouillé. Réessayez plus tard."
            return 1
        fi
    fi

    return 0
}

# Enregistrer une tentative de connexion échouée
record_failed_attempt() {
    local username="$1"
    local client_ip="$2"
    local attempts_file="/tmp/login_attempts_${username}_${client_ip//\./_}"

    local current_attempts=0
    if [ -f "$attempts_file" ]; then
        current_attempts=$(cat "$attempts_file" 2>/dev/null || echo "0")
    fi

    echo $((current_attempts + 1)) > "$attempts_file"
    chmod 600 "$attempts_file"
}

# Réinitialiser les tentatives après un succès
reset_failed_attempts() {
    local username="$1"
    local client_ip="$2"
    local attempts_file="/tmp/login_attempts_${username}_${client_ip//\./_}"

    rm -f "$attempts_file"
}

# Vérification de l'utilisateur et du mot de passe
verify_user_credentials() {
    local username="$1"
    local password="$2"

    # Valider le nom d'utilisateur
    if ! validate_username "$username"; then
        return 1
    fi

    # Vérifier que la base utilisateurs existe et est sécurisée
    if [ ! -f "$USERS_DB" ]; then
        secure_log "ERROR" "Users database not found" "$username"
        return 1
    fi

    # Vérifier les permissions de la base utilisateurs
    local db_perms
    db_perms=$(stat -c "%a" "$USERS_DB")
    if [ "$db_perms" != "600" ]; then
        secure_log "SECURITY" "Users database has insecure permissions" "$username"
        return 1
    fi

    # Rechercher l'utilisateur dans la base (format: username:hash_password:salt)
    local user_line
    user_line=$(grep "^$username:" "$USERS_DB" 2>/dev/null || true)

    if [ -z "$user_line" ]; then
        # Utilisateur non trouvé - ne pas révéler cette information
        secure_log "SECURITY" "Authentication failed - user not found" "$username"
        return 1
    fi

    # Extraire le hash du mot de passe et le salt
    local stored_hash salt
    IFS=':' read -r _ stored_hash salt <<< "$user_line"

    # Calculer le hash du mot de passe fourni
    local provided_hash
    provided_hash=$(echo -n "$password$salt" | sha256sum | cut -d' ' -f1)

    # Comparaison sécurisée (éviter les attaques de timing)
    if [ "$provided_hash" = "$stored_hash" ]; then
        return 0
    else
        return 1
    fi
}

# Fonction principale d'authentification
authenticate_user() {
    local username="$1"
    local client_ip="${2:-127.0.0.1}"

    # Valider l'adresse IP
    if ! validate_ip_address "$client_ip"; then
        secure_log "SECURITY" "Invalid IP address format" "$username" "$client_ip"
        return 1
    fi

    # Vérifier les tentatives de connexion précédentes
    if ! check_login_attempts "$username" "$client_ip"; then
        return 1
    fi

    # Demander le mot de passe de manière sécurisée
    if ! read_password_securely "Mot de passe pour $username: "; then
        secure_log "ERROR" "Failed to read password" "$username" "$client_ip"
        return 1
    fi

    # Vérifier les informations d'identification
    if verify_user_credentials "$username" "$SECURE_PASSWORD"; then
        # Succès de l'authentification
        reset_failed_attempts "$username" "$client_ip"
        secure_log "INFO" "Successful authentication" "$username" "$client_ip"

        # Nettoyer le mot de passe
        clear_sensitive_variables

        return 0
    else
        # Échec de l'authentification
        record_failed_attempt "$username" "$client_ip"
        secure_log "SECURITY" "Authentication failed - invalid credentials" "$username" "$client_ip"

        # Nettoyer le mot de passe
        clear_sensitive_variables

        return 1
    fi
}

# Création sécurisée d'un utilisateur
create_secure_user() {
    local username="$1"

    # Valider le nom d'utilisateur
    if ! validate_username "$username"; then
        echo "Nom d'utilisateur invalide"
        return 1
    fi

    # Vérifier que l'utilisateur n'existe pas déjà
    if grep -q "^$username:" "$USERS_DB" 2>/dev/null; then
        echo "L'utilisateur existe déjà"
        return 1
    fi

    # Demander le mot de passe
    if ! read_password_securely "Nouveau mot de passe pour $username: "; then
        return 1
    fi

    local password="$SECURE_PASSWORD"

    # Demander confirmation
    if ! read_password_securely "Confirmer le mot de passe: "; then
        clear_sensitive_variables
        return 1
    fi

    if [ "$password" != "$SECURE_PASSWORD" ]; then
        echo "Les mots de passe ne correspondent pas"
        clear_sensitive_variables
        return 1
    fi

    # Générer un salt aléatoire
    local salt
    salt=$(openssl rand -hex 16)

    # Calculer le hash du mot de passe
    local password_hash
    password_hash=$(echo -n "$password$salt" | sha256sum | cut -d' ' -f1)

    # Nettoyer immédiatement les mots de passe
    clear_sensitive_variables

    # Créer le répertoire de la base utilisateurs si nécessaire
    local db_dir
    db_dir=$(dirname "$USERS_DB")
    if [ ! -d "$db_dir" ]; then
        create_secure_directory "$db_dir" 700
    fi

    # Ajouter l'utilisateur à la base
    echo "$username:$password_hash:$salt" >> "$USERS_DB"
    chmod 600 "$USERS_DB"

    secure_log "INFO" "New user created" "$username"
    echo "Utilisateur $username créé avec succès"
}
```

### Script de transfert de fichiers sécurisé

```bash
#!/bin/bash

# === TRANSFERT DE FICHIERS SÉCURISÉ ===

set -euo pipefail

readonly UPLOAD_DIR="/var/uploads"
readonly QUARANTINE_DIR="/var/quarantine"
readonly MAX_FILE_SIZE=10485760  # 10MB
readonly ALLOWED_EXTENSIONS=("txt" "pdf" "jpg" "png" "doc" "docx")

# Configuration de sécurité
setup_security_traps

# Validation d'extension de fichier
validate_file_extension() {
    local filename="$1"
    local extension="${filename##*.}"
    extension="${extension,,}"  # Convertir en minuscules

    for allowed in "${ALLOWED_EXTENSIONS[@]}"; do
        if [ "$extension" = "$allowed" ]; then
            return 0
        fi
    done

    echo "Extension de fichier non autorisée: $extension"
    return 1
}

# Analyse antivirus basique
basic_virus_scan() {
    local filepath="$1"

    # Vérifier les signatures de fichiers malveillants courantes
    local malicious_patterns=(
        "eval.*base64"
        "<?php.*system"
        "cmd.exe"
        "powershell.*-enc"
        "/bin/sh.*-c"
    )

    for pattern in "${malicious_patterns[@]}"; do
        if grep -qi "$pattern" "$filepath" 2>/dev/null; then
            echo "Contenu potentiellement malveillant détecté"
            return 1
        fi
    done

    # Vérifier les en-têtes de fichiers
    local file_type
    file_type=$(file -b "$filepath")

    case "$file_type" in
        *"executable"*|*"script"*)
            echo "Type de fichier exécutable non autorisé"
            return 1
            ;;
    esac

    return 0
}

# Transfert sécurisé de fichier
secure_file_upload() {
    local source_file="$1"
    local uploaded_by="${2:-anonymous}"

    # Valider le fichier source
    if [ ! -f "$source_file" ]; then
        echo "Fichier source non trouvé: $source_file"
        return 1
    fi

    # Vérifier la taille du fichier
    local file_size
    file_size=$(stat -c%s "$source_file")
    if [ "$file_size" -gt "$MAX_FILE_SIZE" ]; then
        echo "Fichier trop volumineux: $file_size bytes (max: $MAX_FILE_SIZE)"
        return 1
    fi

    # Obtenir et valider le nom de fichier
    local original_filename
    original_filename=$(basename "$source_file")
    local safe_filename
    safe_filename=$(sanitize_filename "$original_filename")

    # Valider l'extension
    if ! validate_file_extension "$safe_filename"; then
        return 1
    fi

    # Créer les répertoires nécessaires
    create_secure_directory "$UPLOAD_DIR" 755
    create_secure_directory "$QUARANTINE_DIR" 700

    # Chemin de quarantaine temporaire
    local quarantine_file="$QUARANTINE_DIR/$(date +%s)_$safe_filename"

    # Copier le fichier en quarantaine
    if ! cp "$source_file" "$quarantine_file"; then
        echo "Erreur lors de la copie en quarantaine"
        return 1
    fi

    # Définir des permissions sécurisées
    chmod 600 "$quarantine_file"

    echo "Fichier en quarantaine: $quarantine_file"

    # Analyse de sécurité
    echo "Analyse de sécurité en cours..."
    if ! basic_virus_scan "$quarantine_file"; then
        echo "Fichier rejeté pour des raisons de sécurité"
        rm -f "$quarantine_file"
        return 1
    fi

    # Si tout est OK, déplacer vers le répertoire d'upload
    local final_path="$UPLOAD_DIR/$safe_filename"

    # Éviter les écrasements
    local counter=1
    while [ -f "$final_path" ]; do
        local name_without_ext="${safe_filename%.*}"
        local ext="${safe_filename##*.}"
        final_path="$UPLOAD_DIR/${name_without_ext}_${counter}.${ext}"
        ((counter++))
    done

    if mv "$quarantine_file" "$final_path"; then
        chmod 644 "$final_path"
        echo "Fichier téléchargé avec succès: $final_path"

        # Log de l'activité
        secure_log "INFO" "File uploaded: $final_path (original: $original_filename)" "$uploaded_by"

        return 0
    else
        echo "Erreur lors du déplacement final"
        rm -f "$quarantine_file"
        return 1
    fi
}
```

## Liste de contrôle de sécurité pour scripts Bash

### Contrôles essentiels

```bash
#!/bin/bash

# === LISTE DE CONTRÔLE DE SÉCURITÉ ===

security_checklist() {
    echo "=== LISTE DE CONTRÔLE DE SÉCURITÉ BASH ==="
    echo ""

    cat << 'EOF'
□ VALIDATION DES ENTRÉES
  □ Valider tous les paramètres d'entrée
  □ Utiliser des listes blanches plutôt que des listes noires
  □ Vérifier les longueurs maximales
  □ Échapper les caractères spéciaux
  □ Valider les formats (email, IP, etc.)

□ GESTION DES FICHIERS
  □ Valider les chemins de fichiers
  □ Prévenir la traversée de répertoires (../)
  □ Utiliser des permissions restrictives (600/700)
  □ Vérifier les propriétaires des fichiers
  □ Utiliser mktemp pour les fichiers temporaires

□ EXÉCUTION DE COMMANDES
  □ Éviter eval et bash -c avec des entrées utilisateur
  □ Utiliser des tableaux pour les arguments
  □ Implémenter des listes blanches de commandes
  □ Échapper correctement les variables
  □ Valider toutes les entrées avant exécution

□ VARIABLES SENSIBLES
  □ Ne jamais stocker de mots de passe en clair
  □ Utiliser read -s pour les saisies sensibles
  □ Nettoyer les variables après utilisation
  □ Éviter les variables d'environnement pour les secrets
  □ Utiliser des fichiers de configuration sécurisés

□ PERMISSIONS ET ACCÈS
  □ Principe du moindre privilège
  □ Vérifier les permissions avant l'exécution
  □ Utiliser umask approprié
  □ Contrôler l'accès aux ressources
  □ Auditer régulièrement les permissions

□ LOGGING ET MONITORING
  □ Logger toutes les actions importantes
  □ Masquer les informations sensibles dans les logs
  □ Surveiller les tentatives d'accès
  □ Implémenter des alertes de sécurité
  □ Rotation et protection des logs

□ GESTION D'ERREURS
  □ set -euo pipefail pour un comportement strict
  □ Gérer toutes les conditions d'erreur
  □ Ne pas révéler d'informations sensibles dans les erreurs
  □ Nettoyer en cas d'interruption (trap)
  □ Valider les codes de retour

□ CONFIGURATION
  □ Désactiver les fonctionnalités non nécessaires
  □ Configurer des timeouts appropriés
  □ Limiter les ressources (ulimit)
  □ Sécuriser l'environnement d'exécution
  □ Utiliser des configurations externes sécurisées

EOF
}

# Fonction d'audit automatique
perform_security_audit() {
    local script_file="$1"

    if [ ! -f "$script_file" ]; then
        echo "Fichier de script non trouvé: $script_file"
        return 1
    fi

    echo "=== AUDIT DE SÉCURITÉ: $script_file ==="
    echo ""

    local issues=0

    # Vérifier les pratiques dangereuses
    echo "Recherche de pratiques dangereuses:"

    # eval avec variables
    if grep -n "eval.*\$" "$script_file"; then
        echo "⚠️  Utilisation potentiellement dangereuse d'eval avec variables"
        ((issues++))
    fi

    # bash -c avec variables
    if grep -n "bash -c.*\$" "$script_file"; then
        echo "⚠️  Utilisation potentiellement dangereuse de bash -c avec variables"
        ((issues++))
    fi

    # Variables non quotées
    if grep -n '\$[A-Za-z_][A-Za-z0-9_]*[^"]' "$script_file" | grep -v '^[[:space:]]*#'; then
        echo "⚠️  Variables potentiellement non quotées détectées"
        ((issues++))
    fi

    # Mots de passe en dur
    if grep -ni "password\|passwd" "$script_file" | grep -v '^[[:space:]]*#'; then
        echo "⚠️  Références à des mots de passe détectées - vérifier s'ils sont en dur"
        ((issues++))
    fi

    # Commandes avec sudo sans validation
    if grep -n "sudo.*\$" "$script_file"; then
        echo "⚠️  Utilisation de sudo avec variables - vérifier la validation"
        ((issues++))
    fi

    echo ""

    # Vérifier les permissions du script
    echo "Vérification des permissions:"
    check_file_security "$script_file"

    echo ""
    echo "Audit terminé. Problèmes potentiels détectés: $issues"

    if [ $issues -eq 0 ]; then
        echo "✅ Aucun problème de sécurité évident détecté"
        return 0
    else
        echo "⚠️  Veuillez examiner les problèmes détectés"
        return 1
    fi
}
```

## Résumé des bonnes pratiques de sécurité

### Principes fondamentaux

1. **Validation systématique** : Valider toutes les entrées utilisateur
2. **Principe du moindre privilège** : Utiliser les permissions minimales nécessaires
3. **Défense en profondeur** : Implémenter plusieurs couches de sécurité
4. **Échec sécurisé** : En cas d'erreur, échouer de manière sécurisée
5. **Transparence** : Logger les actions importantes pour l'audit

### Actions immédiates à implémenter

```bash
# Dans tous vos scripts de production
set -euo pipefail                    # Comportement strict
setup_security_traps                 # Nettoyage automatique
validate_all_inputs                  # Validation des entrées
use_secure_file_operations          # Opérations fichiers sécurisées
implement_proper_logging            # Logging sécurisé
```

### Outils recommandés

- **ShellCheck** : Analyse statique de code Bash
- **lynis** : Audit de sécurité système
- **rkhunter** : Détection de rootkits
- **fail2ban** : Protection contre les attaques par force brute

La sécurité en Bash nécessite une vigilance constante et l'application systématique de ces principes. Un script sécurisé est un script qui assume que toutes les entrées sont potentiellement malveillantes et qui implémente des défenses appropriées à chaque niveau.

⏭️
