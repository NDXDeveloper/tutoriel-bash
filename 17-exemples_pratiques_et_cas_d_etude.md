🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Exemples pratiques et cas d'étude

## Introduction

Cette section présente des exemples complets de scripts Bash utilisés dans des environnements de production. Chaque cas d'étude intègre les bonnes pratiques vues précédemment : sécurité, gestion d'erreurs, logging, et optimisation. Ces scripts peuvent servir de base pour vos propres projets.

## 1. Scripts de sauvegarde et restauration

### Système de sauvegarde incrémentale

```bash
#!/bin/bash

################################################################################
# SYSTÈME DE SAUVEGARDE INCRÉMENTALE
################################################################################
# Description: Système complet de sauvegarde avec support incrémental,
#              compression, chiffrement et synchronisation distante
# Version: 3.1.0
# Auteur: Admin System <admin@example.com>
################################################################################

set -euo pipefail

# === CONFIGURATION ===
readonly SCRIPT_NAME="$(basename "$0")"
readonly SCRIPT_VERSION="3.1.0"
readonly CONFIG_FILE="/etc/backup/backup.conf"
readonly LOG_DIR="/var/log/backup"
readonly BACKUP_ROOT="/backup"
readonly STATE_DIR="/var/lib/backup"

# Valeurs par défaut
readonly DEFAULT_RETENTION_DAYS=30
readonly DEFAULT_COMPRESSION=true
readonly DEFAULT_ENCRYPTION=false
readonly DEFAULT_REMOTE_SYNC=false

# === INITIALISATION ===

# Créer les répertoires nécessaires
init_directories() {
    local dirs=("$LOG_DIR" "$BACKUP_ROOT" "$STATE_DIR")

    for dir in "${dirs[@]}"; do
        if [ ! -d "$dir" ]; then
            sudo mkdir -p "$dir"
            sudo chmod 750 "$dir"
        fi
    done
}

# Charger la configuration
load_config() {
    # Configuration par défaut
    SOURCES=()
    RETENTION_DAYS=$DEFAULT_RETENTION_DAYS
    COMPRESSION=$DEFAULT_COMPRESSION
    ENCRYPTION=$DEFAULT_ENCRYPTION
    REMOTE_SYNC=$DEFAULT_REMOTE_SYNC
    REMOTE_HOST=""
    ENCRYPTION_KEY=""
    EXCLUDE_PATTERNS=()

    # Charger le fichier de configuration s'il existe
    if [ -f "$CONFIG_FILE" ]; then
        # Vérifier les permissions du fichier de configuration
        local perms
        perms=$(stat -c "%a" "$CONFIG_FILE")
        if [ "$perms" != "600" ]; then
            log "WARN" "Configuration file has insecure permissions: $perms"
        fi

        source "$CONFIG_FILE"
    else
        log "WARN" "Configuration file not found: $CONFIG_FILE"
        create_default_config
    fi

    # Validation de la configuration
    validate_config
}

# Créer une configuration par défaut
create_default_config() {
    cat > "$CONFIG_FILE" << 'EOF'
# Configuration du système de sauvegarde

# Répertoires à sauvegarder
SOURCES=(
    "/home"
    "/etc"
    "/var/www"
    "/opt"
)

# Motifs à exclure
EXCLUDE_PATTERNS=(
    "*.tmp"
    "*.log"
    "*.cache"
    "*/temp/*"
    "*/cache/*"
    "*/.git/*"
)

# Paramètres de rétention
RETENTION_DAYS=30

# Compression (true/false)
COMPRESSION=true

# Chiffrement (true/false)
ENCRYPTION=false
ENCRYPTION_KEY="/etc/backup/backup.key"

# Synchronisation distante
REMOTE_SYNC=false
REMOTE_HOST="backup@remote-server:/backups/"

# Notifications
EMAIL_NOTIFICATIONS=true
ADMIN_EMAIL="admin@example.com"

# Webhook pour notifications (Slack, Discord, etc.)
WEBHOOK_URL=""
EOF

    chmod 600 "$CONFIG_FILE"
    log "INFO" "Default configuration created: $CONFIG_FILE"
}

# Validation de la configuration
validate_config() {
    # Vérifier que les sources existent
    for source in "${SOURCES[@]}"; do
        if [ ! -d "$source" ]; then
            log "WARN" "Source directory does not exist: $source"
        fi
    done

    # Vérifier la clé de chiffrement si activé
    if [ "$ENCRYPTION" = true ] && [ ! -f "$ENCRYPTION_KEY" ]; then
        log "ERROR" "Encryption enabled but key file not found: $ENCRYPTION_KEY"
        return 1
    fi

    # Valider les paramètres numériques
    if ! [[ "$RETENTION_DAYS" =~ ^[0-9]+$ ]] || [ "$RETENTION_DAYS" -lt 1 ]; then
        log "ERROR" "Invalid retention days: $RETENTION_DAYS"
        return 1
    fi
}

# === FONCTIONS DE LOGGING ===

log() {
    local level="$1"
    shift
    local message="$*"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    local log_file="$LOG_DIR/backup_$(date +%Y%m%d).log"

    echo "[$timestamp] [$level] $message" | tee -a "$log_file"

    # Rotation des logs (garder 30 jours)
    find "$LOG_DIR" -name "backup_*.log" -mtime +30 -delete 2>/dev/null || true
}

# === FONCTIONS DE NOTIFICATION ===

send_email_notification() {
    if [ "$EMAIL_NOTIFICATIONS" != true ] || [ -z "${ADMIN_EMAIL:-}" ]; then
        return 0
    fi

    local subject="$1"
    local message="$2"

    {
        echo "Serveur: $(hostname)"
        echo "Date: $(date)"
        echo "Script: $SCRIPT_NAME v$SCRIPT_VERSION"
        echo ""
        echo "$message"
    } | mail -s "$subject" "$ADMIN_EMAIL" 2>/dev/null || true
}

send_webhook_notification() {
    if [ -z "${WEBHOOK_URL:-}" ]; then
        return 0
    fi

    local status="$1"
    local message="$2"

    local payload=$(cat << EOF
{
    "text": "Sauvegarde système",
    "attachments": [
        {
            "color": "$status",
            "fields": [
                {
                    "title": "Serveur",
                    "value": "$(hostname)",
                    "short": true
                },
                {
                    "title": "Status",
                    "value": "$message",
                    "short": true
                },
                {
                    "title": "Date",
                    "value": "$(date)",
                    "short": false
                }
            ]
        }
    ]
}
EOF
)

    curl -s -X POST \
        -H "Content-Type: application/json" \
        -d "$payload" \
        "$WEBHOOK_URL" >/dev/null 2>&1 || true
}

# === FONCTIONS DE SAUVEGARDE ===

# Créer un nom de sauvegarde avec timestamp
generate_backup_name() {
    local backup_type="$1"  # full, incremental
    local timestamp=$(date +%Y%m%d_%H%M%S)

    echo "${backup_type}_${timestamp}"
}

# Déterminer le type de sauvegarde nécessaire
determine_backup_type() {
    local last_full_file="$STATE_DIR/last_full_backup"

    # Si pas de sauvegarde complète ou si elle date de plus de 7 jours
    if [ ! -f "$last_full_file" ]; then
        echo "full"
        return
    fi

    local last_full_date
    last_full_date=$(cat "$last_full_file")
    local current_date=$(date +%s)
    local age_days=$(( (current_date - last_full_date) / 86400 ))

    if [ $age_days -ge 7 ]; then
        echo "full"
    else
        echo "incremental"
    fi
}

# Construire les options d'exclusion pour tar
build_exclude_options() {
    local exclude_opts=()

    for pattern in "${EXCLUDE_PATTERNS[@]}"; do
        exclude_opts+=("--exclude=$pattern")
    done

    # Ajouter les exclusions système courantes
    exclude_opts+=(
        "--exclude=/proc/*"
        "--exclude=/sys/*"
        "--exclude=/dev/*"
        "--exclude=/tmp/*"
        "--exclude=/run/*"
        "--exclude=/mnt/*"
        "--exclude=/media/*"
        "--exclude=lost+found"
    )

    printf '%s\n' "${exclude_opts[@]}"
}

# Effectuer la sauvegarde d'un répertoire
backup_directory() {
    local source_dir="$1"
    local backup_name="$2"
    local backup_type="$3"

    log "INFO" "Starting $backup_type backup of: $source_dir"

    # Nom du fichier de sauvegarde
    local backup_file="$BACKUP_ROOT/${backup_name}_$(basename "$source_dir").tar"

    # Ajouter l'extension de compression si activée
    if [ "$COMPRESSION" = true ]; then
        backup_file="${backup_file}.gz"
    fi

    # Options tar de base
    local tar_opts=("tar")

    # Compression
    if [ "$COMPRESSION" = true ]; then
        tar_opts+=("-z")
    fi

    # Type de sauvegarde
    case "$backup_type" in
        "full")
            tar_opts+=("-c")
            ;;
        "incremental")
            local snapshot_file="$STATE_DIR/snapshot_$(basename "$source_dir")"
            tar_opts+=("-c" "--listed-incremental=$snapshot_file")
            ;;
    esac

    # Fichier de sortie
    tar_opts+=("-f" "$backup_file")

    # Options d'exclusion
    readarray -t exclude_opts < <(build_exclude_options)
    tar_opts+=("${exclude_opts[@]}")

    # Répertoire source
    tar_opts+=("-C" "$(dirname "$source_dir")" "$(basename "$source_dir")")

    # Exécuter la sauvegarde
    log "DEBUG" "Executing: ${tar_opts[*]}"

    if "${tar_opts[@]}" 2>>"$LOG_DIR/backup_$(date +%Y%m%d).log"; then
        log "INFO" "Backup completed: $backup_file"

        # Afficher la taille
        local size
        size=$(du -h "$backup_file" | cut -f1)
        log "INFO" "Backup size: $size"

        # Chiffrement si activé
        if [ "$ENCRYPTION" = true ]; then
            encrypt_backup_file "$backup_file"
        fi

        return 0
    else
        log "ERROR" "Backup failed for: $source_dir"
        return 1
    fi
}

# Chiffrer un fichier de sauvegarde
encrypt_backup_file() {
    local backup_file="$1"
    local encrypted_file="${backup_file}.enc"

    log "INFO" "Encrypting backup: $backup_file"

    if openssl enc -aes-256-cbc -salt -in "$backup_file" -out "$encrypted_file" -pass file:"$ENCRYPTION_KEY"; then
        rm -f "$backup_file"  # Supprimer le fichier non chiffré
        log "INFO" "Backup encrypted: $encrypted_file"
    else
        log "ERROR" "Encryption failed for: $backup_file"
        return 1
    fi
}

# Effectuer toutes les sauvegardes
perform_backup() {
    local backup_type
    backup_type=$(determine_backup_type)

    local backup_name
    backup_name=$(generate_backup_name "$backup_type")

    log "INFO" "Starting $backup_type backup session: $backup_name"

    local success_count=0
    local total_count=${#SOURCES[@]}
    local failed_sources=()

    # Sauvegarder chaque source
    for source in "${SOURCES[@]}"; do
        if [ -d "$source" ]; then
            if backup_directory "$source" "$backup_name" "$backup_type"; then
                ((success_count++))
            else
                failed_sources+=("$source")
            fi
        else
            log "WARN" "Source directory not found: $source"
            failed_sources+=("$source")
        fi
    done

    # Mettre à jour l'état pour les sauvegardes complètes
    if [ "$backup_type" = "full" ] && [ $success_count -gt 0 ]; then
        date +%s > "$STATE_DIR/last_full_backup"
    fi

    # Rapport final
    log "INFO" "Backup session completed: $success_count/$total_count successful"

    if [ ${#failed_sources[@]} -gt 0 ]; then
        log "ERROR" "Failed sources: ${failed_sources[*]}"
        return 1
    fi

    return 0
}

# === SYNCHRONISATION DISTANTE ===

sync_to_remote() {
    if [ "$REMOTE_SYNC" != true ] || [ -z "$REMOTE_HOST" ]; then
        log "INFO" "Remote sync disabled"
        return 0
    fi

    log "INFO" "Starting remote synchronization to: $REMOTE_HOST"

    # Options rsync
    local rsync_opts=(
        "rsync"
        "-av"
        "--delete"
        "--progress"
        "--timeout=3600"
        "--exclude=*.tmp"
    )

    # Synchroniser
    if "${rsync_opts[@]}" "$BACKUP_ROOT/" "$REMOTE_HOST"; then
        log "INFO" "Remote sync completed successfully"
        return 0
    else
        log "ERROR" "Remote sync failed"
        return 1
    fi
}

# === NETTOYAGE ET MAINTENANCE ===

cleanup_old_backups() {
    log "INFO" "Cleaning up backups older than $RETENTION_DAYS days"

    local deleted_count=0

    # Nettoyer les fichiers de sauvegarde
    while IFS= read -r -d '' backup_file; do
        rm -f "$backup_file"
        log "INFO" "Deleted old backup: $(basename "$backup_file")"
        ((deleted_count++))
    done < <(find "$BACKUP_ROOT" -name "*.tar*" -mtime +$RETENTION_DAYS -print0 2>/dev/null)

    log "INFO" "Cleanup completed: $deleted_count files deleted"
}

# Vérifier l'intégrité des sauvegardes
verify_backups() {
    log "INFO" "Verifying backup integrity"

    local corrupt_files=()

    # Vérifier les archives tar
    while IFS= read -r -d '' backup_file; do
        if [[ "$backup_file" == *.tar.gz ]]; then
            if ! tar -tzf "$backup_file" >/dev/null 2>&1; then
                corrupt_files+=("$backup_file")
            fi
        elif [[ "$backup_file" == *.tar ]]; then
            if ! tar -tf "$backup_file" >/dev/null 2>&1; then
                corrupt_files+=("$backup_file")
            fi
        fi
    done < <(find "$BACKUP_ROOT" -name "*.tar*" -mtime -7 -print0 2>/dev/null)

    if [ ${#corrupt_files[@]} -gt 0 ]; then
        log "ERROR" "Corrupt backup files detected: ${corrupt_files[*]}"
        return 1
    else
        log "INFO" "All recent backups verified successfully"
        return 0
    fi
}

# === RESTAURATION ===

list_available_backups() {
    echo "Sauvegardes disponibles:"
    echo "========================"

    find "$BACKUP_ROOT" -name "*.tar*" -type f -printf "%T@ %p\n" | \
    sort -nr | \
    while read -r timestamp filepath; do
        local date_str
        date_str=$(date -d "@$timestamp" '+%Y-%m-%d %H:%M:%S')
        local size
        size=$(du -h "$filepath" | cut -f1)
        printf "%-20s %-10s %s\n" "$date_str" "$size" "$(basename "$filepath")"
    done
}

restore_backup() {
    local backup_file="$1"
    local restore_path="${2:-/tmp/restore}"

    if [ ! -f "$backup_file" ]; then
        log "ERROR" "Backup file not found: $backup_file"
        return 1
    fi

    log "INFO" "Restoring backup: $backup_file to $restore_path"

    # Créer le répertoire de restauration
    mkdir -p "$restore_path"

    # Déchiffrer si nécessaire
    local file_to_extract="$backup_file"
    if [[ "$backup_file" == *.enc ]]; then
        if [ "$ENCRYPTION" != true ] || [ ! -f "$ENCRYPTION_KEY" ]; then
            log "ERROR" "Cannot decrypt: encryption not configured"
            return 1
        fi

        local decrypted_file="${backup_file%.enc}"
        if openssl enc -aes-256-cbc -d -in "$backup_file" -out "$decrypted_file" -pass file:"$ENCRYPTION_KEY"; then
            file_to_extract="$decrypted_file"
        else
            log "ERROR" "Decryption failed"
            return 1
        fi
    fi

    # Extraire l'archive
    if tar -xf "$file_to_extract" -C "$restore_path"; then
        log "INFO" "Restore completed successfully"

        # Nettoyer le fichier déchiffré temporaire si nécessaire
        if [ "$file_to_extract" != "$backup_file" ]; then
            rm -f "$file_to_extract"
        fi

        return 0
    else
        log "ERROR" "Restore failed"
        return 1
    fi
}

# === FONCTION PRINCIPALE ===

show_help() {
    cat << EOF
Usage: $SCRIPT_NAME [COMMAND] [OPTIONS]

Système de sauvegarde incrémentale avec chiffrement et synchronisation

COMMANDES:
    backup              Effectuer une sauvegarde (défaut)
    list                Lister les sauvegardes disponibles
    restore FILE [PATH] Restaurer une sauvegarde
    verify              Vérifier l'intégrité des sauvegardes
    cleanup             Nettoyer les anciennes sauvegardes
    sync                Synchroniser vers le serveur distant
    config              Afficher la configuration actuelle

OPTIONS:
    -c, --config FILE   Fichier de configuration
    -v, --verbose       Mode verbeux
    -h, --help          Afficher cette aide

EXEMPLES:
    $SCRIPT_NAME backup
    $SCRIPT_NAME list
    $SCRIPT_NAME restore /backup/full_20240115_120000_home.tar.gz /tmp/restore
    $SCRIPT_NAME verify

FICHIERS:
    Configuration: $CONFIG_FILE
    Logs: $LOG_DIR/
    État: $STATE_DIR/

EOF
}

main() {
    local command="${1:-backup}"

    # Traitement des options
    while [[ $# -gt 0 ]]; do
        case $1 in
            -c|--config)
                CONFIG_FILE="$2"
                shift 2
                ;;
            -v|--verbose)
                set -x
                shift
                ;;
            -h|--help)
                show_help
                exit 0
                ;;
            *)
                command="$1"
                shift
                ;;
        esac
    done

    # Initialisation
    init_directories
    load_config

    # Exécution de la commande
    case "$command" in
        backup)
            log "INFO" "Starting backup system v$SCRIPT_VERSION"

            if perform_backup; then
                cleanup_old_backups
                sync_to_remote
                verify_backups

                send_email_notification "Sauvegarde réussie" "Sauvegarde terminée avec succès"
                send_webhook_notification "good" "Sauvegarde réussie"

                log "INFO" "Backup system completed successfully"
            else
                send_email_notification "Échec de sauvegarde" "Erreurs détectées lors de la sauvegarde"
                send_webhook_notification "danger" "Échec de sauvegarde"

                log "ERROR" "Backup system completed with errors"
                exit 1
            fi
            ;;
        list)
            list_available_backups
            ;;
        restore)
            if [ $# -lt 2 ]; then
                echo "Usage: $SCRIPT_NAME restore BACKUP_FILE [RESTORE_PATH]"
                exit 1
            fi
            restore_backup "$2" "${3:-}"
            ;;
        verify)
            verify_backups
            ;;
        cleanup)
            cleanup_old_backups
            ;;
        sync)
            sync_to_remote
            ;;
        config)
            echo "Configuration actuelle:"
            cat "$CONFIG_FILE"
            ;;
        *)
            echo "Commande inconnue: $command"
            show_help
            exit 1
            ;;
    esac
}

# Point d'entrée
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
fi
```

### Script de restauration avancé

```bash
#!/bin/bash

################################################################################
# SCRIPT DE RESTAURATION AVANCÉ
################################################################################
# Description: Restauration sélective avec interface interactive
################################################################################

set -euo pipefail

readonly BACKUP_DIR="/backup"
readonly RESTORE_STAGING="/tmp/restore_staging"

# Interface de sélection de sauvegarde
select_backup_interactive() {
    echo "=== SÉLECTION DE SAUVEGARDE ==="

    # Lister les sauvegardes disponibles avec numérotation
    local backups=()
    local counter=1

    while IFS= read -r -d '' backup_file; do
        backups+=("$backup_file")
        local date_str
        date_str=$(stat -c %y "$backup_file" | cut -d' ' -f1)
        local size
        size=$(du -h "$backup_file" | cut -f1)
        printf "%2d. %-15s %-8s %s\n" "$counter" "$date_str" "$size" "$(basename "$backup_file")"
        ((counter++))
    done < <(find "$BACKUP_DIR" -name "*.tar*" -type f -print0 | sort -z)

    if [ ${#backups[@]} -eq 0 ]; then
        echo "Aucune sauvegarde trouvée"
        return 1
    fi

    # Sélection par l'utilisateur
    while true; do
        echo ""
        read -p "Sélectionnez une sauvegarde (1-${#backups[@]}): " selection

        if [[ "$selection" =~ ^[0-9]+$ ]] && [ "$selection" -ge 1 ] && [ "$selection" -le ${#backups[@]} ]; then
            SELECTED_BACKUP="${backups[$((selection-1))]}"
            echo "Sauvegarde sélectionnée: $(basename "$SELECTED_BACKUP")"
            return 0
        else
            echo "Sélection invalide. Veuillez choisir entre 1 et ${#backups[@]}"
        fi
    done
}

# Prévisualisation du contenu d'une sauvegarde
preview_backup_content() {
    local backup_file="$1"

    echo "=== CONTENU DE LA SAUVEGARDE ==="
    echo "Fichier: $(basename "$backup_file")"
    echo ""

    # Afficher les répertoires principaux
    if [[ "$backup_file" == *.tar.gz ]]; then
        tar -tzf "$backup_file" | head -20
    elif [[ "$backup_file" == *.tar ]]; then
        tar -tf "$backup_file" | head -20
    fi

    echo ""
    echo "(Affichage des 20 premiers éléments)"
}

# Restauration sélective avec interface
interactive_restore() {
    if ! select_backup_interactive; then
        return 1
    fi

    preview_backup_content "$SELECTED_BACKUP"

    echo ""
    read -p "Répertoire de destination [/tmp/restore]: " restore_path
    restore_path="${restore_path:-/tmp/restore}"

    # Confirmation
    echo ""
    echo "Résumé de la restauration:"
    echo "  Source: $(basename "$SELECTED_BACKUP")"
    echo "  Destination: $restore_path"
    echo ""

    read -p "Confirmer la restauration ? (oui/non): " confirm
    if [[ "${confirm,,}" != "oui" && "${confirm,,}" != "o" ]]; then
        echo "Restauration annulée"
        return 0
    fi

    # Effectuer la restauration
    restore_backup "$SELECTED_BACKUP" "$restore_path"
}
```

## 2. Surveillance système et rapports

### Système de monitoring complet

```bash
#!/bin/bash

################################################################################
# SYSTÈME DE MONITORING SYSTÈME
################################################################################
# Description: Surveillance complète avec métriques, alertes et rapports
################################################################################

set -euo pipefail

readonly SCRIPT_NAME="$(basename "$0")"
readonly CONFIG_FILE="/etc/monitoring/monitor.conf"
readonly LOG_DIR="/var/log/monitoring"
readonly METRICS_DIR="/var/lib/monitoring"
readonly REPORT_DIR="/var/reports"

# Seuils par défaut
readonly DEFAULT_CPU_WARNING=80
readonly DEFAULT_CPU_CRITICAL=95
readonly DEFAULT_MEMORY_WARNING=85
readonly DEFAULT_MEMORY_CRITICAL=95
readonly DEFAULT_DISK_WARNING=85
readonly DEFAULT_DISK_CRITICAL=95
readonly DEFAULT_LOAD_WARNING=5.0
readonly DEFAULT_LOAD_CRITICAL=10.0

# Configuration par défaut
CPU_WARNING=$DEFAULT_CPU_WARNING
CPU_CRITICAL=$DEFAULT_CPU_CRITICAL
MEMORY_WARNING=$DEFAULT_MEMORY_WARNING
MEMORY_CRITICAL=$DEFAULT_MEMORY_CRITICAL
DISK_WARNING=$DEFAULT_DISK_WARNING
DISK_CRITICAL=$DEFAULT_DISK_CRITICAL
LOAD_WARNING=$DEFAULT_LOAD_WARNING
LOAD_CRITICAL=$DEFAULT_LOAD_CRITICAL

# Services à surveiller
MONITORED_SERVICES=("ssh" "nginx" "mysql" "postgresql")
MONITORED_PORTS=("22" "80" "443" "3306" "5432")
MONITORED_PROCESSES=()

# Notification
EMAIL_ALERTS=true
ADMIN_EMAIL="admin@example.com"
WEBHOOK_URL=""

# === FONCTIONS UTILITAIRES ===

init_monitoring() {
    # Créer les répertoires
    for dir in "$LOG_DIR" "$METRICS_DIR" "$REPORT_DIR"; do
        [ ! -d "$dir" ] && mkdir -p "$dir"
    done

    # Charger la configuration
    [ -f "$CONFIG_FILE" ] && source "$CONFIG_FILE"
}

log_metric() {
    local metric_name="$1"
    local value="$2"
    local unit="${3:-}"
    local timestamp=$(date +%s)

    echo "$timestamp $value $unit" >> "$METRICS_DIR/${metric_name}.dat"

    # Garder seulement les 24 dernières heures
    local cutoff=$((timestamp - 86400))
    awk -v cutoff="$cutoff" '$1 >= cutoff' "$METRICS_DIR/${metric_name}.dat" > "$METRICS_DIR/${metric_name}.tmp"
    mv "$METRICS_DIR/${metric_name}.tmp" "$METRICS_DIR/${metric_name}.dat"
}

# === COLLECTE DE MÉTRIQUES ===

collect_cpu_metrics() {
    # Utilisation CPU moyenne
    local cpu_usage
    cpu_usage=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}')

    log_metric "cpu_usage" "$cpu_usage" "%"

    # Charge système
    local load_1m load_5m load_15m
    read -r load_1m load_5m load_15m < <(uptime | awk -F'load average:' '{print $2}' | awk '{gsub(/,/, ""); print $1, $2, $3}')

    log_metric "load_1m" "$load_1m"
    log_metric "load_5m" "$load_5m"
    log_metric "load_15m" "$load_15m"

    # Vérification des seuils
    check_threshold "CPU" "$cpu_usage" "$CPU_WARNING" "$CPU_CRITICAL"
    check_threshold "Load 1m" "$load_1m" "$LOAD_WARNING" "$LOAD_CRITICAL"
}

collect_memory_metrics() {
    # Informations mémoire
    local mem_info
    mem_info=$(free -b | grep "^Mem:")

    local total used available
    read -r _ total used _ _ _ available <<< "$mem_info"

    local usage_percent
    usage_percent=$(awk "BEGIN {printf \"%.1f\", ($used/$total)*100}")

    log_metric "memory_total" "$total" "bytes"
    log_metric "memory_used" "$used" "bytes"
    log_metric "memory_usage_percent" "$usage_percent" "%"

    # Swap
    local swap_info
    swap_info=$(free -b | grep "^Swap:")
    if [ -n "$swap_info" ]; then
        read -r _ total used _ <<< "$swap_info"
        if [ "$total" -gt 0 ]; then
            local swap_percent
            swap_percent=$(awk "BEGIN {printf \"%.1f\", ($used/$total)*100}")
            log_metric "swap_usage_percent" "$swap_percent" "%"
        fi
    fi

    check_threshold "Memory" "$usage_percent" "$MEMORY_WARNING" "$MEMORY_CRITICAL"
}

collect_disk_metrics() {
    # Utilisation des disques
    df -h | grep -E '^/dev/' | while read -r filesystem size used available usage_percent mountpoint; do
        # Nettoyer le pourcentage
        usage_percent=${usage_percent%?}

        # Créer un nom de métrique sûr
        local metric_name="disk_usage_$(echo "$mountpoint" | tr '/' '_' | sed 's/^_/root/')"

        log_metric "$metric_name" "$usage_percent" "%"

        # Vérifier les seuils pour chaque partition
        check_threshold "Disk $mountpoint" "$usage_percent" "$DISK_WARNING" "$DISK_CRITICAL"
    done

    # I/O disque
    if command -v iostat >/dev/null 2>&1; then
        local io_data
        io_data=$(iostat -d 1 2 | tail -n +4 | awk 'NF>0 {print $1, $4, $5}' | tail -1)
        if [ -n "$io_data" ]; then
            local device read_rate write_rate
            read -r device read_rate write_rate <<< "$io_data"
            log_metric "disk_read_rate" "$read_rate" "KB/s"
            log_metric "disk_write_rate" "$write_rate" "KB/s"
        fi
    fi
}

collect_network_metrics() {
    # Statistiques réseau
    local rx_bytes tx_bytes

    # Interface principale (première interface non-loopback)
    local main_interface
    main_interface=$(ip route | grep default | awk '{print $5}' | head -1)

    if [ -n "$main_interface" ]; then
        local net_stats
        net_stats=$(cat "/sys/class/net/$main_interface/statistics/rx_bytes" "/sys/class/net/$main_interface/statistics/tx_bytes" 2>/dev/null)

        if [ -n "$net_stats" ]; then
            read -r rx_bytes tx_bytes <<< "$net_stats"
            log_metric "network_rx_bytes" "$rx_bytes" "bytes"
            log_metric "network_tx_bytes" "$tx_bytes" "bytes"
        fi
    fi

    # Connexions réseau actives
    local connections
    connections=$(ss -tuln | wc -l)
    log_metric "network_connections" "$connections"
}

collect_service_metrics() {
    local services_up=0
    local services_down=0
    local down_services=()

    for service in "${MONITORED_SERVICES[@]}"; do
        if systemctl is-active --quiet "$service" 2>/dev/null; then
            ((services_up++))
        else
            ((services_down++))
            down_services+=("$service")
        fi
    done

    log_metric "services_up" "$services_up"
    log_metric "services_down" "$services_down"

    # Alertes pour les services arrêtés
    if [ "$services_down" -gt 0 ]; then
        send_alert "CRITICAL" "Services arrêtés détectés: ${down_services[*]}"
    fi
}

collect_port_metrics() {
    local ports_open=0
    local ports_closed=0
    local closed_ports=()

    for port in "${MONITORED_PORTS[@]}"; do
        if ss -tuln | grep -q ":$port "; then
            ((ports_open++))
        else
            ((ports_closed++))
            closed_ports+=("$port")
        fi
    done

    log_metric "ports_open" "$ports_open"
    log_metric "ports_closed" "$ports_closed"

    # Alertes pour les ports fermés
    if [ "$ports_closed" -gt 0 ]; then
        send_alert "WARNING" "Ports fermés détectés: ${closed_ports[*]}"
    fi
}

# === VÉRIFICATION DES SEUILS ===

check_threshold() {
    local metric_name="$1"
    local current_value="$2"
    local warning_threshold="$3"
    local critical_threshold="$4"

    # Comparaison avec bc pour les nombres flottants
    if (( $(echo "$current_value >= $critical_threshold" | bc -l) )); then
        send_alert "CRITICAL" "$metric_name: $current_value (seuil critique: $critical_threshold)"
    elif (( $(echo "$current_value >= $warning_threshold" | bc -l) )); then
        send_alert "WARNING" "$metric_name: $current_value (seuil d'alerte: $warning_threshold)"
    fi
}

# === SYSTÈME D'ALERTES ===

send_alert() {
    local level="$1"
    local message="$2"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')

    # Logger l'alerte
    echo "[$timestamp] [$level] $message" >> "$LOG_DIR/alerts.log"

    # Éviter le spam d'alertes (maximum une alerte du même type par heure)
    local alert_key=$(echo "$message" | md5sum | cut -d' ' -f1)
    local last_alert_file="/tmp/last_alert_$alert_key"

    if [ -f "$last_alert_file" ]; then
        local last_alert_time
        last_alert_time=$(cat "$last_alert_file")
        local current_time=$(date +%s)
        local time_diff=$((current_time - last_alert_time))

        # Moins d'une heure depuis la dernière alerte similaire
        if [ $time_diff -lt 3600 ]; then
            return 0
        fi
    fi

    # Enregistrer l'heure de cette alerte
    date +%s > "$last_alert_file"

    # Envoyer les notifications
    if [ "$EMAIL_ALERTS" = true ] && [ -n "$ADMIN_EMAIL" ]; then
        {
            echo "Alerte système - $level"
            echo "========================"
            echo "Serveur: $(hostname)"
            echo "Date: $timestamp"
            echo ""
            echo "Message: $message"
            echo ""
            echo "-- Système de monitoring automatique"
        } | mail -s "[$level] Alerte système $(hostname)" "$ADMIN_EMAIL" 2>/dev/null || true
    fi

    # Webhook
    if [ -n "$WEBHOOK_URL" ]; then
        local color="warning"
        [ "$level" = "CRITICAL" ] && color="danger"

        local payload=$(cat << EOF
{
    "text": "Alerte système",
    "attachments": [
        {
            "color": "$color",
            "title": "$level - $(hostname)",
            "text": "$message",
            "ts": $(date +%s)
        }
    ]
}
EOF
)

        curl -s -X POST \
            -H "Content-Type: application/json" \
            -d "$payload" \
            "$WEBHOOK_URL" >/dev/null 2>&1 || true
    fi
}

# === GÉNÉRATION DE RAPPORTS ===

generate_daily_report() {
    local report_date="${1:-$(date +%Y-%m-%d)}"
    local report_file="$REPORT_DIR/daily_report_$report_date.html"

    cat > "$report_file" << EOF
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Rapport système - $report_date</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .header { background: #f4f4f4; padding: 20px; border-radius: 5px; margin-bottom: 20px; }
        .section { margin: 20px 0; padding: 15px; border: 1px solid #ddd; border-radius: 5px; }
        .metric { display: flex; justify-content: space-between; padding: 5px 0; }
        .warning { background-color: #fff3cd; border-color: #ffeaa7; }
        .critical { background-color: #f8d7da; border-color: #f5c6cb; }
        .good { background-color: #d4edda; border-color: #c3e6cb; }
        table { width: 100%; border-collapse: collapse; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #f2f2f2; }
    </style>
</head>
<body>
    <div class="header">
        <h1>Rapport système quotidien</h1>
        <p><strong>Serveur:</strong> $(hostname)</p>
        <p><strong>Date:</strong> $report_date</p>
        <p><strong>Généré le:</strong> $(date)</p>
    </div>
EOF

    # Résumé exécutif
    cat >> "$report_file" << EOF
    <div class="section">
        <h2>Résumé exécutif</h2>
        <div class="metric">
            <span>Statut global:</span>
            <span><strong>$(get_system_status)</strong></span>
        </div>
        <div class="metric">
            <span>Uptime:</span>
            <span>$(uptime -p)</span>
        </div>
        <div class="metric">
            <span>Charge moyenne (1m):</span>
            <span>$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | tr -d ',')</span>
        </div>
    </div>
EOF

    # Métriques système actuelles
    generate_current_metrics_section >> "$report_file"

    # Graphiques des tendances
    generate_trends_section "$report_date" >> "$report_file"

    # Alertes du jour
    generate_alerts_section "$report_date" >> "$report_file"

    # Services et ports
    generate_services_section >> "$report_file"

    # Fermeture HTML
    echo "</body></html>" >> "$report_file"

    echo "Rapport généré: $report_file"
}

get_system_status() {
    # Déterminer le statut global du système
    local status="OK"

    # Vérifier les alertes récentes
    if grep -q "CRITICAL" "$LOG_DIR/alerts.log" 2>/dev/null; then
        if tail -100 "$LOG_DIR/alerts.log" | grep "$(date +%Y-%m-%d)" | grep -q "CRITICAL"; then
            status="CRITIQUE"
        fi
    fi

    if [ "$status" = "OK" ] && grep -q "WARNING" "$LOG_DIR/alerts.log" 2>/dev/null; then
        if tail -100 "$LOG_DIR/alerts.log" | grep "$(date +%Y-%m-%d)" | grep -q "WARNING"; then
            status="ATTENTION"
        fi
    fi

    echo "$status"
}

generate_current_metrics_section() {
    cat << EOF
    <div class="section">
        <h2>Métriques actuelles</h2>
        <table>
            <tr><th>Métrique</th><th>Valeur</th><th>Statut</th></tr>
EOF

    # CPU
    local cpu_usage
    cpu_usage=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}')
    local cpu_status="good"
    (( $(echo "$cpu_usage >= $CPU_CRITICAL" | bc -l) )) && cpu_status="critical"
    (( $(echo "$cpu_usage >= $CPU_WARNING" | bc -l) )) && cpu_status="warning"

    echo "<tr class=\"$cpu_status\"><td>Utilisation CPU</td><td>${cpu_usage}%</td><td>$(get_status_text "$cpu_status")</td></tr>"

    # Mémoire
    local mem_usage
    mem_usage=$(free | grep Mem | awk '{printf "%.1f", ($3/$2)*100}')
    local mem_status="good"
    (( $(echo "$mem_usage >= $MEMORY_CRITICAL" | bc -l) )) && mem_status="critical"
    (( $(echo "$mem_usage >= $MEMORY_WARNING" | bc -l) )) && mem_status="warning"

    echo "<tr class=\"$mem_status\"><td>Utilisation mémoire</td><td>${mem_usage}%</td><td>$(get_status_text "$mem_status")</td></tr>"

    # Disque racine
    local disk_usage
    disk_usage=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')
    local disk_status="good"
    [ "$disk_usage" -ge "$DISK_CRITICAL" ] && disk_status="critical"
    [ "$disk_usage" -ge "$DISK_WARNING" ] && disk_status="warning"

    echo "<tr class=\"$disk_status\"><td>Utilisation disque /</td><td>${disk_usage}%</td><td>$(get_status_text "$disk_status")</td></tr>"

    echo "</table></div>"
}

get_status_text() {
    case "$1" in
        "good") echo "✅ OK" ;;
        "warning") echo "⚠️ Attention" ;;
        "critical") echo "🚨 Critique" ;;
        *) echo "❓ Inconnu" ;;
    esac
}

generate_trends_section() {
    local date="$1"

    cat << EOF
    <div class="section">
        <h2>Tendances (24h)</h2>
        <p>Analyse des métriques sur les dernières 24 heures:</p>
EOF

    # Analyser les tendances pour chaque métrique
    for metric_file in "$METRICS_DIR"/*.dat; do
        [ ! -f "$metric_file" ] && continue

        local metric_name
        metric_name=$(basename "$metric_file" .dat)

        # Calculer min, max, moyenne
        if [ -s "$metric_file" ]; then
            local stats
            stats=$(awk '{sum+=$2; if(min==""){min=max=$2}; if($2>max){max=$2}; if($2<min){min=$2}} END {print min, max, sum/NR}' "$metric_file")

            local min max avg
            read -r min max avg <<< "$stats"

            cat << EOF
        <div class="metric">
            <span><strong>$metric_name:</strong></span>
            <span>Min: ${min:-N/A} | Max: ${max:-N/A} | Moy: $(printf "%.1f" "${avg:-0}")</span>
        </div>
EOF
        fi
    done

    echo "</div>"
}

generate_alerts_section() {
    local date="$1"

    cat << EOF
    <div class="section">
        <h2>Alertes du jour</h2>
EOF

    if [ -f "$LOG_DIR/alerts.log" ]; then
        local alerts_today
        alerts_today=$(grep "$date" "$LOG_DIR/alerts.log" 2>/dev/null || true)

        if [ -n "$alerts_today" ]; then
            echo "<table><tr><th>Heure</th><th>Niveau</th><th>Message</th></tr>"

            echo "$alerts_today" | while IFS= read -r alert; do
                local time level message
                time=$(echo "$alert" | cut -d' ' -f2)
                level=$(echo "$alert" | sed 's/.*\[\([^]]*\)\].*/\1/')
                message=$(echo "$alert" | sed 's/.*] //')

                local row_class="good"
                [ "$level" = "WARNING" ] && row_class="warning"
                [ "$level" = "CRITICAL" ] && row_class="critical"

                echo "<tr class=\"$row_class\"><td>$time</td><td>$level</td><td>$message</td></tr>"
            done

            echo "</table>"
        else
            echo "<p>✅ Aucune alerte aujourd'hui</p>"
        fi
    else
        echo "<p>📝 Fichier d'alertes non trouvé</p>"
    fi

    echo "</div>"
}

generate_services_section() {
    cat << EOF
    <div class="section">
        <h2>Statut des services</h2>
        <table>
            <tr><th>Service</th><th>Statut</th><th>Depuis</th></tr>
EOF

    for service in "${MONITORED_SERVICES[@]}"; do
        if systemctl is-active --quiet "$service" 2>/dev/null; then
            local since
            since=$(systemctl show "$service" --property=ActiveEnterTimestamp --value 2>/dev/null | cut -d' ' -f1-2)
            echo "<tr class=\"good\"><td>$service</td><td>✅ Actif</td><td>$since</td></tr>"
        else
            echo "<tr class=\"critical\"><td>$service</td><td>🚨 Arrêté</td><td>-</td></tr>"
        fi
    done

    echo "</table></div>"
}

# === FONCTIONS PRINCIPALES ===

run_monitoring_cycle() {
    echo "Début du cycle de monitoring - $(date)"

    # Collecter toutes les métriques
    collect_cpu_metrics
    collect_memory_metrics
    collect_disk_metrics
    collect_network_metrics
    collect_service_metrics
    collect_port_metrics

    echo "Cycle de monitoring terminé - $(date)"
}

show_status() {
    echo "=== STATUT SYSTÈME ACTUEL ==="
    echo "Serveur: $(hostname)"
    echo "Date: $(date)"
    echo "Uptime: $(uptime -p)"
    echo ""

    echo "=== MÉTRIQUES ACTUELLES ==="

    # CPU
    local cpu_usage
    cpu_usage=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}')
    printf "%-20s: %6.1f%% " "CPU" "$cpu_usage"
    check_status_display "$cpu_usage" "$CPU_WARNING" "$CPU_CRITICAL"

    # Mémoire
    local mem_usage
    mem_usage=$(free | grep Mem | awk '{printf "%.1f", ($3/$2)*100}')
    printf "%-20s: %6.1f%% " "Mémoire" "$mem_usage"
    check_status_display "$mem_usage" "$MEMORY_WARNING" "$MEMORY_CRITICAL"

    # Disque
    local disk_usage
    disk_usage=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')
    printf "%-20s: %6s%% " "Disque /" "$disk_usage"
    check_status_display "$disk_usage" "$DISK_WARNING" "$DISK_CRITICAL"

    # Charge
    local load_1m
    load_1m=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | tr -d ',')
    printf "%-20s: %6s " "Charge (1m)" "$load_1m"
    check_status_display "$load_1m" "$LOAD_WARNING" "$LOAD_CRITICAL"

    echo ""
    echo "=== SERVICES ==="
    for service in "${MONITORED_SERVICES[@]}"; do
        printf "%-20s: " "$service"
        if systemctl is-active --quiet "$service" 2>/dev/null; then
            echo "✅ Actif"
        else
            echo "🚨 Arrêté"
        fi
    done
}

check_status_display() {
    local value="$1"
    local warning="$2"
    local critical="$3"

    if (( $(echo "$value >= $critical" | bc -l) )); then
        echo "🚨 CRITIQUE"
    elif (( $(echo "$value >= $warning" | bc -l) )); then
        echo "⚠️ ATTENTION"
    else
        echo "✅ OK"
    fi
}

show_help() {
    cat << EOF
Usage: $SCRIPT_NAME [COMMAND] [OPTIONS]

Système de monitoring et surveillance

COMMANDES:
    monitor             Effectuer un cycle de monitoring complet
    status              Afficher le statut actuel du système
    report [DATE]       Générer un rapport (défaut: aujourd'hui)
    daemon              Lancer en mode daemon (monitoring continu)
    alerts              Afficher les alertes récentes
    metrics METRIC      Afficher l'historique d'une métrique

OPTIONS:
    -c, --config FILE   Fichier de configuration
    -v, --verbose       Mode verbeux
    -h, --help          Afficher cette aide

EXEMPLES:
    $SCRIPT_NAME monitor
    $SCRIPT_NAME status
    $SCRIPT_NAME report 2024-01-15
    $SCRIPT_NAME metrics cpu_usage

FICHIERS:
    Configuration: $CONFIG_FILE
    Logs: $LOG_DIR/
    Métriques: $METRICS_DIR/
    Rapports: $REPORT_DIR/

EOF
}

main() {
    local command="${1:-monitor}"

    # Traitement des options
    while [[ $# -gt 0 ]]; do
        case $1 in
            -c|--config)
                CONFIG_FILE="$2"
                shift 2
                ;;
            -v|--verbose)
                set -x
                shift
                ;;
            -h|--help)
                show_help
                exit 0
                ;;
            *)
                command="$1"
                shift
                ;;
        esac
    done

    # Initialisation
    init_monitoring

    # Exécution de la commande
    case "$command" in
        monitor)
            run_monitoring_cycle
            ;;
        status)
            show_status
            ;;
        report)
            local date="${1:-$(date +%Y-%m-%d)}"
            generate_daily_report "$date"
            ;;
        daemon)
            echo "Démarrage du daemon de monitoring..."
            while true; do
                run_monitoring_cycle
                sleep 300  # 5 minutes
            done
            ;;
        alerts)
            if [ -f "$LOG_DIR/alerts.log" ]; then
                tail -50 "$LOG_DIR/alerts.log"
            else
                echo "Aucun fichier d'alertes trouvé"
            fi
            ;;
        metrics)
            local metric="${1:-cpu_usage}"
            if [ -f "$METRICS_DIR/$metric.dat" ]; then
                echo "Historique de $metric (dernières 24h):"
                tail -100 "$METRICS_DIR/$metric.dat" | while read -r timestamp value unit; do
                    local date_str
                    date_str=$(date -d "@$timestamp" '+%H:%M:%S')
                    printf "%s: %s %s\n" "$date_str" "$value" "$unit"
                done
            else
                echo "Métrique non trouvée: $metric"
                echo "Métriques disponibles:"
                ls -1 "$METRICS_DIR"/*.dat 2>/dev/null | sed 's|.*/||; s|\.dat$||' || echo "Aucune métrique disponible"
            fi
            ;;
        *)
            echo "Commande inconnue: $command"
            show_help
            exit 1
            ;;
    esac
}

# Point d'entrée
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
fi
```

## 3. Automatisation de tâches d'administration

### Script d'administration système automatisé

```bash
#!/bin/bash

################################################################################
# AUTOMATISATION DES TÂCHES D'ADMINISTRATION
################################################################################
# Description: Automatisation des tâches courantes d'administration système
################################################################################

set -euo pipefail

readonly SCRIPT_NAME="$(basename "$0")"
readonly LOG_FILE="/var/log/admin_automation.log"
readonly CONFIG_FILE="/etc/admin/automation.conf"

# Configuration par défaut
AUTO_UPDATE=false
AUTO_CLEANUP=true
AUTO_BACKUP_CONFIGS=true
AUTO_LOG_ROTATION=true
AUTO_SECURITY_SCAN=false

# === FONCTIONS DE MAINTENANCE ===

system_cleanup() {
    log "INFO" "Début du nettoyage système"

    local cleaned_space=0

    # Nettoyage du cache des paquets
    if command -v apt-get >/dev/null 2>&1; then
        log "INFO" "Nettoyage du cache APT"
        local cache_size_before
        cache_size_before=$(du -sb /var/cache/apt 2>/dev/null | cut -f1 || echo "0")

        apt-get clean >/dev/null 2>&1 || true
        apt-get autoclean >/dev/null 2>&1 || true
        apt-get autoremove -y >/dev/null 2>&1 || true

        local cache_size_after
        cache_size_after=$(du -sb /var/cache/apt 2>/dev/null | cut -f1 || echo "0")
        local freed=$((cache_size_before - cache_size_after))
        cleaned_space=$((cleaned_space + freed))

        log "INFO" "Cache APT nettoyé: $(format_bytes $freed) libérés"
    fi

    # Nettoyage des logs anciens
    log "INFO" "Nettoyage des logs anciens"
    local logs_cleaned=0

    # Logs système de plus de 30 jours
    find /var/log -name "*.log.*" -mtime +30 -type f -exec rm -f {} \; 2>/dev/null && ((logs_cleaned++)) || true
    find /var/log -name "*.gz" -mtime +30 -type f -exec rm -f {} \; 2>/dev/null && ((logs_cleaned++)) || true

    # Journald cleanup
    if command -v journalctl >/dev/null 2>&1; then
        journalctl --vacuum-time=30d >/dev/null 2>&1 || true
        journalctl --vacuum-size=100M >/dev/null 2>&1 || true
    fi

    log "INFO" "Logs nettoyés: $logs_cleaned fichiers supprimés"

    # Nettoyage des fichiers temporaires
    log "INFO" "Nettoyage des fichiers temporaires"
    local temp_cleaned=0

    # /tmp (fichiers de plus de 7 jours)
    find /tmp -type f -mtime +7 -exec rm -f {} \; 2>/dev/null && ((temp_cleaned++)) || true

    # /var/tmp (fichiers de plus de 30 jours)
    find /var/tmp -type f -mtime +30 -exec rm -f {} \; 2>/dev/null && ((temp_cleaned++)) || true

    log "INFO" "Fichiers temporaires nettoyés: $temp_cleaned fichiers"

    # Nettoyage des core dumps
    find /var/crash -name "*.crash" -mtime +7 -exec rm -f {} \; 2>/dev/null || true
    find / -name "core.*" -mtime +7 -exec rm -f {} \; 2>/dev/null || true

    log "INFO" "Nettoyage système terminé: $(format_bytes $cleaned_space) libérés au total"
}

format_bytes() {
    local bytes="$1"
    local units=('B' 'KB' 'MB' 'GB' 'TB')
    local unit=0

    while (( bytes >= 1024 && unit < ${#units[@]} - 1 )); do
        bytes=$((bytes / 1024))
        ((unit++))
    done

    echo "${bytes}${units[$unit]}"
}

security_hardening() {
    log "INFO" "Application du durcissement de sécurité"

    # Mise à jour des permissions critiques
    chmod 700 /root 2>/dev/null || true
    chmod 600 /etc/shadow 2>/dev/null || true
    chmod 600 /etc/gshadow 2>/dev/null || true
    chmod 644 /etc/passwd 2>/dev/null || true
    chmod 644 /etc/group 2>/dev/null || true

    # Configuration SSH sécurisée
    if [ -f /etc/ssh/sshd_config ]; then
        local ssh_config_updated=false

        # Désactiver l'authentification par mot de passe root si pas déjà fait
        if ! grep -q "^PermitRootLogin no" /etc/ssh/sshd_config; then
            if grep -q "^PermitRootLogin" /etc/ssh/sshd_config; then
                sed -i 's/^PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
            else
                echo "PermitRootLogin no" >> /etc/ssh/sshd_config
            fi
            ssh_config_updated=true
        fi

        # Désactiver l'authentification par mot de passe vide
        if ! grep -q "^PermitEmptyPasswords no" /etc/ssh/sshd_config; then
            echo "PermitEmptyPasswords no" >> /etc/ssh/sshd_config
            ssh_config_updated=true
        fi

        if [ "$ssh_config_updated" = true ]; then
            log "INFO" "Configuration SSH mise à jour"
            systemctl reload ssh 2>/dev/null || true
        fi
    fi

```bash
    # Vérification des comptes sans mot de passe
    local accounts_no_password
    accounts_no_password=$(awk -F: '($2 == "") {print $1}' /etc/shadow 2>/dev/null || true)
    if [ -n "$accounts_no_password" ]; then
        log "WARN" "Comptes sans mot de passe détectés: $accounts_no_password"
    fi

    # Vérification des fichiers avec permissions SUID/SGID
    log "INFO" "Audit des fichiers SUID/SGID"
    find / -type f \( -perm -4000 -o -perm -2000 \) -exec ls -la {} \; 2>/dev/null > /var/log/suid_sgid_files.log

    # Désactivation des services non nécessaires
    local unnecessary_services=("telnet" "rsh" "rlogin" "ftp")
    for service in "${unnecessary_services[@]}"; do
        if systemctl is-enabled "$service" >/dev/null 2>&1; then
            systemctl disable "$service" >/dev/null 2>&1 || true
            log "INFO" "Service non sécurisé désactivé: $service"
        fi
    done

    log "INFO" "Durcissement de sécurité terminé"
}

backup_system_configs() {
    log "INFO" "Sauvegarde des configurations système"

    local backup_dir="/var/backups/configs/$(date +%Y%m%d_%H%M%S)"
    mkdir -p "$backup_dir"

    # Fichiers de configuration critiques à sauvegarder
    local config_files=(
        "/etc/passwd"
        "/etc/shadow"
        "/etc/group"
        "/etc/fstab"
        "/etc/hosts"
        "/etc/ssh/sshd_config"
        "/etc/sudoers"
        "/etc/crontab"
        "/etc/nginx"
        "/etc/apache2"
        "/etc/mysql"
        "/etc/postgresql"
    )

    local backed_up=0

    for config in "${config_files[@]}"; do
        if [ -e "$config" ]; then
            if cp -r "$config" "$backup_dir/" 2>/dev/null; then
                ((backed_up++))
            fi
        fi
    done

    # Créer une archive compressée
    if [ $backed_up -gt 0 ]; then
        tar -czf "${backup_dir}.tar.gz" -C "$(dirname "$backup_dir")" "$(basename "$backup_dir")"
        rm -rf "$backup_dir"

        # Nettoyer les anciennes sauvegardes (garder 30 jours)
        find /var/backups/configs -name "*.tar.gz" -mtime +30 -delete 2>/dev/null || true

        log "INFO" "Configurations sauvegardées: $backed_up fichiers dans ${backup_dir}.tar.gz"
    else
        log "WARN" "Aucune configuration n'a pu être sauvegardée"
        rmdir "$backup_dir" 2>/dev/null || true
    fi
}

system_updates() {
    if [ "$AUTO_UPDATE" != true ]; then
        log "INFO" "Mises à jour automatiques désactivées"
        return 0
    fi

    log "INFO" "Vérification des mises à jour système"

    if command -v apt-get >/dev/null 2>&1; then
        # Système basé sur Debian/Ubuntu
        log "INFO" "Mise à jour de la liste des paquets"
        apt-get update >/dev/null 2>&1 || true

        # Vérifier les mises à jour disponibles
        local updates_available
        updates_available=$(apt list --upgradable 2>/dev/null | wc -l)

        if [ "$updates_available" -gt 1 ]; then
            log "INFO" "$((updates_available - 1)) mises à jour disponibles"

            # Appliquer les mises à jour de sécurité uniquement
            apt-get upgrade -y -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold" >/dev/null 2>&1 || true

            log "INFO" "Mises à jour appliquées"
        else
            log "INFO" "Système à jour"
        fi

    elif command -v yum >/dev/null 2>&1; then
        # Système basé sur RHEL/CentOS
        log "INFO" "Vérification des mises à jour YUM"
        yum check-update >/dev/null 2>&1 || true
        yum update -y >/dev/null 2>&1 || true
        log "INFO" "Mises à jour YUM appliquées"

    elif command -v dnf >/dev/null 2>&1; then
        # Système basé sur Fedora
        log "INFO" "Vérification des mises à jour DNF"
        dnf check-update >/dev/null 2>&1 || true
        dnf upgrade -y >/dev/null 2>&1 || true
        log "INFO" "Mises à jour DNF appliquées"
    fi
}

log_rotation_management() {
    log "INFO" "Gestion de la rotation des logs"

    # Forcer la rotation des logs si nécessaire
    if command -v logrotate >/dev/null 2>&1; then
        logrotate -f /etc/logrotate.conf 2>/dev/null || true
        log "INFO" "Rotation des logs forcée"
    fi

    # Compression des logs volumineux
    local large_logs
    large_logs=$(find /var/log -name "*.log" -size +100M -type f 2>/dev/null)

    if [ -n "$large_logs" ]; then
        echo "$large_logs" | while read -r log_file; do
            if gzip "$log_file" 2>/dev/null; then
                log "INFO" "Log compressé: $log_file"
            fi
        done
    fi
}

# === MAINTENANCE AUTOMATISÉE ===

run_daily_maintenance() {
    log "INFO" "=== DÉBUT MAINTENANCE QUOTIDIENNE ==="

    local start_time=$(date +%s)

    # Sauvegarde des configurations
    if [ "$AUTO_BACKUP_CONFIGS" = true ]; then
        backup_system_configs
    fi

    # Nettoyage système
    if [ "$AUTO_CLEANUP" = true ]; then
        system_cleanup
    fi

    # Rotation des logs
    if [ "$AUTO_LOG_ROTATION" = true ]; then
        log_rotation_management
    fi

    # Durcissement de sécurité
    if [ "$AUTO_SECURITY_SCAN" = true ]; then
        security_hardening
    fi

    # Mises à jour
    system_updates

    local end_time=$(date +%s)
    local duration=$((end_time - start_time))

    log "INFO" "=== FIN MAINTENANCE QUOTIDIENNE === (Durée: ${duration}s)"
}

# === GESTION DES UTILISATEURS ===

audit_user_accounts() {
    log "INFO" "Audit des comptes utilisateurs"

    local audit_report="/var/log/user_audit_$(date +%Y%m%d).log"

    {
        echo "=== AUDIT DES COMPTES UTILISATEURS ==="
        echo "Date: $(date)"
        echo "Serveur: $(hostname)"
        echo ""

        echo "=== COMPTES AVEC SHELL DE CONNEXION ==="
        awk -F: '$7 ~ /bash|sh|zsh|fish/ {print $1 " (" $5 ")"}' /etc/passwd
        echo ""

        echo "=== COMPTES RÉCEMMENT CONNECTÉS ==="
        last -n 20 | head -15
        echo ""

        echo "=== COMPTES SUDO ==="
        getent group sudo | cut -d: -f4 | tr ',' '\n' 2>/dev/null || echo "Groupe sudo non trouvé"
        echo ""

        echo "=== COMPTES AVEC UID 0 ==="
        awk -F: '$3 == 0 {print $1}' /etc/passwd
        echo ""

        echo "=== COMPTES VERROUILLÉS ==="
        awk -F: '$2 ~ /^!/ {print $1}' /etc/shadow 2>/dev/null || echo "Accès shadow refusé"

    } > "$audit_report"

    log "INFO" "Audit utilisateurs sauvegardé: $audit_report"
}

cleanup_old_users() {
    log "INFO" "Nettoyage des comptes utilisateurs inactifs"

    # Trouver les utilisateurs qui ne se sont pas connectés depuis 90 jours
    local inactive_users=()

    while IFS=: read -r username _ uid _ _ home shell; do
        # Ignorer les comptes système (UID < 1000)
        [ "$uid" -lt 1000 ] && continue

        # Ignorer les comptes sans shell de connexion
        [[ "$shell" =~ (nologin|false|sync|halt|shutdown) ]] && continue

        # Vérifier la dernière connexion
        local last_login
        last_login=$(last -n 1 "$username" 2>/dev/null | head -1 | awk '{print $4, $5, $6}')

        if [ -z "$last_login" ] || [ "$last_login" = "wtmp" ]; then
            # Jamais connecté ou pas dans wtmp
            if [ -d "$home" ]; then
                # Vérifier l'âge du répertoire home
                local home_age
                home_age=$(find "$home" -maxdepth 0 -mtime +90 2>/dev/null)
                if [ -n "$home_age" ]; then
                    inactive_users+=("$username")
                fi
            fi
        fi
    done < /etc/passwd

    if [ ${#inactive_users[@]} -gt 0 ]; then
        log "WARN" "Comptes inactifs détectés: ${inactive_users[*]}"
        log "INFO" "Exécutez manuellement la suppression si nécessaire"
    else
        log "INFO" "Aucun compte inactif détecté"
    fi
}

# === MONITORING SYSTÈME ===

check_system_health() {
    log "INFO" "Vérification de l'état du système"

    local issues=()

    # Vérifier l'espace disque
    while read -r filesystem size used avail percent mountpoint; do
        local usage=${percent%?}
        if [ "$usage" -gt 90 ]; then
            issues+=("Disque $mountpoint plein à ${usage}%")
        fi
    done < <(df -h | grep -v "^Filesystem")

    # Vérifier la mémoire
    local mem_usage
    mem_usage=$(free | grep Mem | awk '{printf "%.0f", ($3/$2)*100}')
    if [ "$mem_usage" -gt 90 ]; then
        issues+=("Mémoire utilisée à ${mem_usage}%")
    fi

    # Vérifier la charge système
    local load_avg
    load_avg=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | tr -d ',')
    local cpu_count
    cpu_count=$(nproc)

    if (( $(echo "$load_avg > $cpu_count * 2" | bc -l) )); then
        issues+=("Charge système élevée: $load_avg")
    fi

    # Vérifier les services critiques
    local critical_services=("ssh" "systemd-logind" "cron")
    for service in "${critical_services[@]}"; do
        if ! systemctl is-active --quiet "$service" 2>/dev/null; then
            issues+=("Service critique arrêté: $service")
        fi
    done

    # Rapport final
    if [ ${#issues[@]} -gt 0 ]; then
        log "WARN" "Problèmes détectés:"
        for issue in "${issues[@]}"; do
            log "WARN" "  - $issue"
        done
        return 1
    else
        log "INFO" "Système en bonne santé"
        return 0
    fi
}

# === UTILITAIRES ===

log() {
    local level="$1"
    shift
    local message="$*"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')

    echo "[$timestamp] [$level] $message" | tee -a "$LOG_FILE"
}

create_default_config() {
    cat > "$CONFIG_FILE" << 'EOF'
# Configuration de l'automatisation administrative

# Mises à jour automatiques (true/false)
AUTO_UPDATE=false

# Nettoyage automatique (true/false)
AUTO_CLEANUP=true

# Sauvegarde des configurations (true/false)
AUTO_BACKUP_CONFIGS=true

# Rotation des logs (true/false)
AUTO_LOG_ROTATION=true

# Scan de sécurité (true/false)
AUTO_SECURITY_SCAN=false

# Notifications email
EMAIL_NOTIFICATIONS=true
ADMIN_EMAIL="admin@example.com"

# Services critiques à surveiller
CRITICAL_SERVICES=("ssh" "nginx" "mysql" "postgresql")

# Seuils d'alerte
DISK_WARNING_THRESHOLD=85
MEMORY_WARNING_THRESHOLD=90
LOAD_WARNING_THRESHOLD=5.0
EOF

    chmod 600 "$CONFIG_FILE"
    log "INFO" "Configuration par défaut créée: $CONFIG_FILE"
}

show_help() {
    cat << EOF
Usage: $SCRIPT_NAME [COMMAND] [OPTIONS]

Automatisation des tâches d'administration système

COMMANDES:
    daily               Exécuter la maintenance quotidienne complète
    cleanup             Nettoyage système uniquement
    security            Durcissement de sécurité uniquement
    backup-configs      Sauvegarde des configurations uniquement
    updates             Mises à jour système uniquement
    user-audit          Audit des comptes utilisateurs
    health-check        Vérification de l'état du système
    install-cron        Installer les tâches automatiques

OPTIONS:
    -c, --config FILE   Fichier de configuration
    -v, --verbose       Mode verbeux
    -h, --help          Afficher cette aide

EXEMPLES:
    $SCRIPT_NAME daily
    $SCRIPT_NAME cleanup
    $SCRIPT_NAME security
    $SCRIPT_NAME health-check

FICHIERS:
    Configuration: $CONFIG_FILE
    Log: $LOG_FILE

EOF
}

install_cron_jobs() {
    log "INFO" "Installation des tâches cron d'automatisation"

    local script_path
    script_path=$(realpath "$0")

    # Sauvegarder la crontab actuelle
    crontab -l > /tmp/current_cron 2>/dev/null || touch /tmp/current_cron

    # Supprimer les anciennes entrées
    grep -v "$script_path" /tmp/current_cron > /tmp/new_cron

    # Ajouter les nouvelles tâches
    cat >> /tmp/new_cron << EOF

# === AUTOMATISATION ADMINISTRATIVE ===
# Maintenance quotidienne à 3h00
0 3 * * * $script_path daily >/dev/null 2>&1

# Vérification de santé toutes les heures
0 * * * * $script_path health-check >/dev/null 2>&1

# Audit utilisateurs hebdomadaire (dimanche à 4h00)
0 4 * * 0 $script_path user-audit >/dev/null 2>&1

# Nettoyage quotidien des logs (minuit)
0 0 * * * $script_path cleanup >/dev/null 2>&1
EOF

    # Installer la nouvelle crontab
    crontab /tmp/new_cron
    rm -f /tmp/current_cron /tmp/new_cron

    log "INFO" "Tâches cron installées avec succès"
}

main() {
    local command="${1:-daily}"

    # Créer les répertoires nécessaires
    mkdir -p "$(dirname "$LOG_FILE")" "$(dirname "$CONFIG_FILE")"

    # Charger la configuration
    if [ -f "$CONFIG_FILE" ]; then
        source "$CONFIG_FILE"
    else
        create_default_config
        source "$CONFIG_FILE"
    fi

    # Traitement des options
    while [[ $# -gt 0 ]]; do
        case $1 in
            -c|--config)
                CONFIG_FILE="$2"
                source "$CONFIG_FILE"
                shift 2
                ;;
            -v|--verbose)
                set -x
                shift
                ;;
            -h|--help)
                show_help
                exit 0
                ;;
            *)
                command="$1"
                shift
                ;;
        esac
    done

    # Exécution de la commande
    case "$command" in
        daily)
            run_daily_maintenance
            ;;
        cleanup)
            system_cleanup
            ;;
        security)
            security_hardening
            ;;
        backup-configs)
            backup_system_configs
            ;;
        updates)
            system_updates
            ;;
        user-audit)
            audit_user_accounts
            cleanup_old_users
            ;;
        health-check)
            check_system_health
            ;;
        install-cron)
            install_cron_jobs
            ;;
        *)
            echo "Commande inconnue: $command"
            show_help
            exit 1
            ;;
    esac
}

# Point d'entrée
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
fi
```

## 4. Scripts de déploiement et d'intégration continue

### Pipeline de déploiement automatisé

```bash
#!/bin/bash

################################################################################
# PIPELINE DE DÉPLOIEMENT AUTOMATISÉ
################################################################################
# Description: Système complet de déploiement avec tests et rollback
################################################################################

set -euo pipefail

readonly SCRIPT_NAME="$(basename "$0")"
readonly SCRIPT_VERSION="2.0.0"
readonly CONFIG_FILE="deploy.conf"
readonly LOG_DIR="logs"
readonly BACKUP_DIR="backups"

# Configuration par défaut
APPLICATION_NAME=""
SOURCE_REPO=""
BRANCH="main"
BUILD_COMMAND=""
TEST_COMMAND=""
DEPLOY_PATH=""
WEB_ROOT=""
BACKUP_ENABLED=true
HEALTH_CHECK_URL=""
SLACK_WEBHOOK=""
EMAIL_NOTIFICATIONS=true

# === INITIALISATION ===

init_deployment() {
    local dirs=("$LOG_DIR" "$BACKUP_DIR" "tmp")

    for dir in "${dirs[@]}"; do
        [ ! -d "$dir" ] && mkdir -p "$dir"
    done

    # Charger la configuration
    if [ -f "$CONFIG_FILE" ]; then
        source "$CONFIG_FILE"
    else
        create_default_deploy_config
        echo "Configuration créée: $CONFIG_FILE"
        echo "Veuillez la modifier avant de continuer"
        exit 1
    fi

    validate_config
}

create_default_deploy_config() {
    cat > "$CONFIG_FILE" << 'EOF'
# Configuration de déploiement

# Application
APPLICATION_NAME="mon-app"
SOURCE_REPO="https://github.com/user/repo.git"
BRANCH="main"

# Commandes de build et test
BUILD_COMMAND="npm install && npm run build"
TEST_COMMAND="npm test"

# Chemins de déploiement
DEPLOY_PATH="/var/www"
WEB_ROOT="/var/www/html"

# Options
BACKUP_ENABLED=true
HEALTH_CHECK_URL="https://example.com/health"

# Notifications
SLACK_WEBHOOK=""
EMAIL_NOTIFICATIONS=true
ADMIN_EMAIL="admin@example.com"

# Serveurs (pour déploiement multi-serveurs)
SERVERS=("server1.example.com" "server2.example.com")
DEPLOY_USER="deploy"

# Base de données
DB_MIGRATION_COMMAND=""
DB_BACKUP_COMMAND="mysqldump -u user -p password database > backup.sql"
EOF
}

validate_config() {
    local required_vars=("APPLICATION_NAME" "SOURCE_REPO" "DEPLOY_PATH")

    for var in "${required_vars[@]}"; do
        if [ -z "${!var:-}" ]; then
            log "ERROR" "Variable de configuration manquante: $var"
            exit 1
        fi
    done

    # Vérifier les répertoires
    if [ ! -w "$(dirname "$DEPLOY_PATH")" ]; then
        log "ERROR" "Répertoire de déploiement non accessible en écriture: $DEPLOY_PATH"
        exit 1
    fi
}

# === FONCTIONS DE LOGGING ===

log() {
    local level="$1"
    shift
    local message="$*"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    local log_file="$LOG_DIR/deploy_$(date +%Y%m%d).log"

    echo "[$timestamp] [$level] $message" | tee -a "$log_file"

    # Rotation des logs
    find "$LOG_DIR" -name "deploy_*.log" -mtime +30 -delete 2>/dev/null || true
}

# === NOTIFICATIONS ===

send_notification() {
    local status="$1"
    local message="$2"
    local deployment_id="${3:-unknown}"

    # Slack
    if [ -n "$SLACK_WEBHOOK" ]; then
        local color="good"
        [ "$status" = "failed" ] && color="danger"
        [ "$status" = "warning" ] && color="warning"

        local payload=$(cat << EOF
{
    "text": "Déploiement $APPLICATION_NAME",
    "attachments": [
        {
            "color": "$color",
            "fields": [
                {
                    "title": "Application",
                    "value": "$APPLICATION_NAME",
                    "short": true
                },
                {
                    "title": "Statut",
                    "value": "$status",
                    "short": true
                },
                {
                    "title": "Serveur",
                    "value": "$(hostname)",
                    "short": true
                },
                {
                    "title": "ID Déploiement",
                    "value": "$deployment_id",
                    "short": true
                },
                {
                    "title": "Message",
                    "value": "$message",
                    "short": false
                }
            ]
        }
    ]
}
EOF
)

        curl -s -X POST \
            -H "Content-Type: application/json" \
            -d "$payload" \
            "$SLACK_WEBHOOK" >/dev/null 2>&1 || true
    fi

    # Email
    if [ "$EMAIL_NOTIFICATIONS" = true ] && [ -n "${ADMIN_EMAIL:-}" ]; then
        {
            echo "Déploiement: $APPLICATION_NAME"
            echo "Statut: $status"
            echo "Serveur: $(hostname)"
            echo "Date: $(date)"
            echo "ID: $deployment_id"
            echo ""
            echo "Message: $message"
        } | mail -s "[$status] Déploiement $APPLICATION_NAME" "$ADMIN_EMAIL" 2>/dev/null || true
    fi
}

# === GESTION DES VERSIONS ===

generate_deployment_id() {
    echo "$(date +%Y%m%d_%H%M%S)_$(git rev-parse --short HEAD 2>/dev/null || echo 'manual')"
}

get_current_version() {
    if [ -f "$DEPLOY_PATH/.version" ]; then
        cat "$DEPLOY_PATH/.version"
    else
        echo "unknown"
    fi
}

save_version() {
    local deployment_id="$1"
    echo "$deployment_id" > "$DEPLOY_PATH/.version"
}

# === SAUVEGARDE ET ROLLBACK ===

create_backup() {
    if [ "$BACKUP_ENABLED" != true ]; then
        log "INFO" "Sauvegarde désactivée"
        return 0
    fi

    local deployment_id="$1"
    local backup_path="$BACKUP_DIR/backup_$deployment_id.tar.gz"

    log "INFO" "Création de la sauvegarde: $backup_path"

    if [ -d "$DEPLOY_PATH" ]; then
        tar -czf "$backup_path" -C "$(dirname "$DEPLOY_PATH")" "$(basename "$DEPLOY_PATH")" 2>/dev/null
        log "INFO" "Sauvegarde créée: $backup_path"

        # Nettoyer les anciennes sauvegardes (garder 10)
        ls -1t "$BACKUP_DIR"/backup_*.tar.gz 2>/dev/null | tail -n +11 | xargs -r rm -f
    else
        log "WARN" "Répertoire de déploiement inexistant, pas de sauvegarde"
    fi
}

rollback_deployment() {
    local backup_file="$1"

    if [ ! -f "$backup_file" ]; then
        log "ERROR" "Fichier de sauvegarde non trouvé: $backup_file"
        return 1
    fi

    log "INFO" "Rollback vers: $backup_file"

    # Supprimer le déploiement actuel
    if [ -d "$DEPLOY_PATH" ]; then
        rm -rf "$DEPLOY_PATH"
    fi

    # Restaurer la sauvegarde
    if tar -xzf "$backup_file" -C "$(dirname "$DEPLOY_PATH")"; then
        log "INFO" "Rollback terminé avec succès"
        return 0
    else
        log "ERROR" "Échec du rollback"
        return 1
    fi
}

list_backups() {
    echo "Sauvegardes disponibles:"
    ls -1t "$BACKUP_DIR"/backup_*.tar.gz 2>/dev/null | while read -r backup; do
        local backup_name
        backup_name=$(basename "$backup" .tar.gz)
        local date_str
        date_str=$(echo "$backup_name" | cut -d'_' -f2-3 | tr '_' ' ')
        local size
        size=$(du -h "$backup" | cut -f1)
        printf "  %-25s %-10s %s\n" "$backup_name" "$size" "$date_str"
    done
}

# === DÉPLOIEMENT ===

checkout_code() {
    local deployment_id="$1"
    local checkout_dir="tmp/checkout_$deployment_id"

    log "INFO" "Récupération du code source: $SOURCE_REPO ($BRANCH)"

    # Nettoyer le répertoire de checkout
    rm -rf "$checkout_dir"

    # Cloner le repository
    if git clone --depth 1 --branch "$BRANCH" "$SOURCE_REPO" "$checkout_dir"; then
        log "INFO" "Code récupéré dans: $checkout_dir"
        echo "$checkout_dir"
        return 0
    else
        log "ERROR" "Échec de la récupération du code"
        return 1
    fi
}

build_application() {
    local source_dir="$1"

    if [ -z "$BUILD_COMMAND" ]; then
        log "INFO" "Aucune commande de build configurée"
        return 0
    fi

    log "INFO" "Construction de l'application: $BUILD_COMMAND"

    cd "$source_dir"

    if eval "$BUILD_COMMAND"; then
        log "INFO" "Build réussi"
        return 0
    else
        log "ERROR" "Échec du build"
        return 1
    fi
}

run_tests() {
    local source_dir="$1"

    if [ -z "$TEST_COMMAND" ]; then
        log "INFO" "Aucun test configuré"
        return 0
    fi

    log "INFO" "Exécution des tests: $TEST_COMMAND"

    cd "$source_dir"

    if eval "$TEST_COMMAND"; then
        log "INFO" "Tests réussis"
        return 0
    else
        log "ERROR" "Échec des tests"
        return 1
    fi
}

deploy_application() {
    local source_dir="$1"
    local deployment_id="$2"

    log "INFO" "Déploiement de l'application vers: $DEPLOY_PATH"

    # Déploiement avec maintenance page
    if [ -n "$WEB_ROOT" ] && [ -d "$WEB_ROOT" ]; then
        create_maintenance_page
    fi

    # Créer le répertoire de déploiement temporaire
    local temp_deploy="$DEPLOY_PATH.new"
    rm -rf "$temp_deploy"

    # Copier les fichiers
    if cp -r "$source_dir" "$temp_deploy"; then
        log "INFO" "Fichiers copiés vers le répertoire temporaire"
    else
        log "ERROR" "Échec de la copie des fichiers"
        return 1
    fi

    # Migration de base de données si configurée
    if [ -n "$DB_MIGRATION_COMMAND" ]; then
        log "INFO" "Exécution des migrations de base de données"
        cd "$temp_deploy"
        if ! eval "$DB_MIGRATION_COMMAND"; then
            log "ERROR" "Échec des migrations de base de données"
            rm -rf "$temp_deploy"
            return 1
        fi
    fi

    # Basculement atomique
    if [ -d "$DEPLOY_PATH" ]; then
        mv "$DEPLOY_PATH" "$DEPLOY_PATH.old"
    fi

    if mv "$temp_deploy" "$DEPLOY_PATH"; then
        log "INFO" "Déploiement atomique réussi"

        # Nettoyer l'ancien déploiement
        rm -rf "$DEPLOY_PATH.old"

        # Sauvegarder la version
        save_version "$deployment_id"

        # Supprimer la page de maintenance
        remove_maintenance_page

        return 0
    else
        log "ERROR" "Échec du basculement atomique"

        # Restaurer l'ancien déploiement si possible
        if [ -d "$DEPLOY_PATH.old" ]; then
            mv "$DEPLOY_PATH.old" "$DEPLOY_PATH"
            log "INFO" "Ancien déploiement restauré"
        fi

        rm -rf "$temp_deploy"
        remove_maintenance_page
        return 1
    fi
}

create_maintenance_page() {
    if [ -z "$WEB_ROOT" ]; then
        return 0
    fi

    log "INFO" "Activation de la page de maintenance"

    cat > "$WEB_ROOT/maintenance.html" << 'EOF'
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Maintenance en cours</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
        }
        .maintenance-box {
            text-align: center;
            background: rgba(255, 255, 255, 0.1);
            padding: 40px;
            border-radius: 10px;
            backdrop-filter: blur(10px);
        }
        .spinner {
            border: 4px solid rgba(255, 255, 255, 0.3);
            border-top: 4px solid white;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
            margin: 20px auto;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
    <meta http-equiv="refresh" content="30">
</head>
<body>
    <div class="maintenance-box">
        <h1>🔧 Maintenance en cours</h1>
        <div class="spinner"></div>
        <p>Nous mettons à jour notre application.</p>
        <p>Merci de patienter quelques instants...</p>
        <small>Cette page se actualise automatiquement.</small>
    </div>
</body>
</html>
EOF

    # Rediriger temporairement vers la page de maintenance
    if [ -f "$WEB_ROOT/.htaccess" ]; then
        cp "$WEB_ROOT/.htaccess" "$WEB_ROOT/.htaccess.backup"
    fi

    cat > "$WEB_ROOT/.htaccess" << 'EOF'
RewriteEngine On
RewriteCond %{REQUEST_URI} !^/maintenance\.html$
RewriteRule ^(.*)$ /maintenance.html [R=503,L]
ErrorDocument 503 /maintenance.html
Header always set Retry-After "60"
EOF
}

remove_maintenance_page() {
    if [ -z "$WEB_ROOT" ]; then
        return 0
    fi

    log "INFO" "Suppression de la page de maintenance"

    # Restaurer l'ancien .htaccess
    if [ -f "$WEB_ROOT/.htaccess.backup" ]; then
        mv "$WEB_ROOT/.htaccess.backup" "$WEB_ROOT/.htaccess"
    else
        rm -f "$WEB_ROOT/.htaccess"
    fi

    # Supprimer la page de maintenance
    rm -f "$WEB_ROOT/maintenance.html"
}

# === VÉRIFICATIONS DE SANTÉ ===

perform_health_check() {
    if [ -z "$HEALTH_CHECK_URL" ]; then
        log "INFO" "Aucune URL de health check configurée"
        return 0
    fi

    log "INFO" "Vérification de santé: $HEALTH_CHECK_URL"

    local max_attempts=10
    local attempt=1
    local wait_time=10

    while [ $attempt -le $max_attempts ]; do
        log "INFO" "Tentative $attempt/$max_attempts"

        local response
        response=$(curl -s -o /dev/null -w "%{http_code}" --max-time 30 "$HEALTH_CHECK_URL" 2>/dev/null || echo "000")

        if [ "$response" = "200" ]; then
            log "INFO" "Health check réussi (HTTP $response)"
            return 0
        else
            log "WARN" "Health check échoué (HTTP $response)"

            if [ $attempt -lt $max_attempts ]; then
                log "INFO" "Attente de ${wait_time}s avant nouvelle tentative"
                sleep $wait_time
            fi
        fi

        ((attempt++))
    done

    log "ERROR" "Health check échoué après $max_attempts tentatives"
    return 1
}

smoke_tests() {
    log "INFO" "Exécution des tests de smoke"

    local tests_passed=0
    local tests_total=0

    # Test 1: Vérifier que l'application répond
    ((tests_total++))
    if [ -n "$HEALTH_CHECK_URL" ]; then
        if perform_health_check; then
            ((tests_passed++))
        fi
    else
        # Fallback: vérifier que les fichiers existent
        if [ -f "$DEPLOY_PATH/index.html" ] || [ -f "$DEPLOY_PATH/index.php" ] || [ -f "$DEPLOY_PATH/app.js" ]; then
            ((tests_passed++))
        fi
    fi

    # Test 2: Vérifier les permissions
    ((tests_total++))
    if [ -r "$DEPLOY_PATH" ] && [ -x "$DEPLOY_PATH" ]; then
        ((tests_passed++))
    fi

    # Test 3: Vérifier la configuration
    ((tests_total++))
    if [ -f "$DEPLOY_PATH/.version" ]; then
        ((tests_passed++))
    fi

    log "INFO" "Tests de smoke: $tests_passed/$tests_total réussis"

    if [ $tests_passed -eq $tests_total ]; then
        return 0
    else
        return 1
    fi
}

# === DÉPLOIEMENT PRINCIPAL ===

full_deployment() {
    local deployment_id
    deployment_id=$(generate_deployment_id)

    log "INFO" "=== DÉBUT DÉPLOIEMENT $deployment_id ==="
    log "INFO" "Application: $APPLICATION_NAME"
    log "INFO" "Branche: $BRANCH"
    log "INFO" "Version actuelle: $(get_current_version)"

    # Notification de début
    send_notification "started" "Déploiement démarré" "$deployment_id"

    # Sauvegarde
    create_backup "$deployment_id"

    # Récupération du code
    local source_dir
    if ! source_dir=$(checkout_code "$deployment_id"); then
        send_notification "failed" "Échec de la récupération du code" "$deployment_id"
        return 1
    fi

    # Build
    if ! build_application "$source_dir"; then
        send_notification "failed" "Échec du build" "$deployment_id"
        return 1
    fi

    # Tests
    if ! run_tests "$source_dir"; then
        send_notification "failed" "Échec des tests" "$deployment_id"
        return 1
    fi

    # Déploiement
    if ! deploy_application "$source_dir" "$deployment_id"; then
        send_notification "failed" "Échec du déploiement" "$deployment_id"
        return 1
    fi

    # Tests de smoke
    if ! smoke_tests; then
        log "ERROR" "Tests de smoke échoués, rollback..."

        # Trouver la dernière sauvegarde
        local last_backup
        last_backup=$(ls -1t "$BACKUP_DIR"/backup_*.tar.gz 2>/dev/null | head -1)

        if [ -n "$last_backup" ] && rollback_deployment "$last_backup"; then
            send_notification "failed" "Déploiement échoué, rollback effectué" "$deployment_id"
        else
            send_notification "failed" "Déploiement ET rollback échoués" "$deployment_id"
        fi
        return 1
    fi

    # Nettoyage
    rm -rf "$source_dir"

    log "INFO" "=== DÉPLOIEMENT TERMINÉ AVEC SUCCÈS ==="
    send_notification "success" "Déploiement réussi vers la version $deployment_id" "$deployment_id"

    return 0
}

# === DÉPLOIEMENT MULTI-SERVEURS ===

deploy_to_servers() {
    local deployment_id
    deployment_id=$(generate_deployment_id)

    log "INFO" "Déploiement multi-serveurs: ${SERVERS[*]}"

    local successful_servers=()
    local failed_servers=()

    # Déployer sur chaque serveur
    for server in "${SERVERS[@]}"; do
        log "INFO" "Déploiement sur: $server"

        if deploy_to_single_server "$server" "$deployment_id"; then
            successful_servers+=("$server")
            log "INFO" "Déploiement réussi sur: $server"
        else
            failed_servers+=("$server")
            log "ERROR" "Déploiement échoué sur: $server"
        fi
    done

    # Rapport final
    log "INFO" "Résumé du déploiement multi-serveurs:"
    log "INFO" "  Succès: ${successful_servers[*]}"
    log "INFO" "  Échecs: ${failed_servers[*]}"

    if [ ${#failed_servers[@]} -gt 0 ]; then
        send_notification "warning" "Déploiement partiel: ${#successful_servers[@]}/${#SERVERS[@]} serveurs" "$deployment_id"
        return 1
    else
        send_notification "success" "Déploiement réussi sur tous les serveurs" "$deployment_id"
        return 0
    fi
}

deploy_to_single_server() {
    local server="$1"
    local deployment_id="$2"

    # Synchroniser le script de déploiement
    if ! scp "$0" "$CONFIG_FILE" "${DEPLOY_USER}@${server}:/tmp/"; then
        log "ERROR" "Échec de la synchronisation vers $server"
        return 1
    fi

    # Exécuter le déploiement distant
    if ssh "${DEPLOY_USER}@${server}" "cd /tmp && chmod +x $(basename "$0") && ./$(basename "$0") deploy"; then
        return 0
    else
        return 1
    fi
}

# === INTERFACE DE LIGNE DE COMMANDE ===

show_status() {
    echo "=== STATUT DU DÉPLOIEMENT ==="
    echo "Application: $APPLICATION_NAME"
    echo "Version actuelle: $(get_current_version)"
    echo "Répertoire: $DEPLOY_PATH"
    echo "Dernière modification: $(stat -c %y "$DEPLOY_PATH" 2>/dev/null || echo 'N/A')"
    echo ""

    echo "=== SAUVEGARDES DISPONIBLES ==="
    list_backups
    echo ""

    echo "=== SANTÉ DE L'APPLICATION ==="
    if perform_health_check; then
        echo "✅ Application en bonne santé"
    else
        echo "❌ Problème détecté"
    fi
}

show_help() {
    cat << EOF
Usage: $SCRIPT_NAME [COMMAND] [OPTIONS]

Pipeline de déploiement automatisé

COMMANDES:
    deploy              Déploiement complet (défaut)
    multi-deploy        Déploiement sur plusieurs serveurs
    rollback [BACKUP]   Rollback vers une sauvegarde
    status              Afficher le statut actuel
    health-check        Vérifier la santé de l'application
    backup              Créer une sauvegarde manuelle
    list-backups        Lister les sauvegardes disponibles
    clean               Nettoyer les fichiers temporaires

OPTIONS:
    -c, --config FILE   Fichier de configuration (défaut: $CONFIG_FILE)
    -b, --branch NAME   Branche à déployer (défaut: $BRANCH)
    -v, --verbose       Mode verbeux
    -h, --help          Afficher cette aide

EXEMPLES:
    $SCRIPT_NAME deploy
    $SCRIPT_NAME rollback backup_20240115_120000.tar.gz
    $SCRIPT_NAME --branch develop deploy
    $SCRIPT_NAME multi-deploy

FICHIERS:
    Configuration: $CONFIG_FILE
    Logs: $LOG_DIR/
    Sauvegardes: $BACKUP_DIR/

EOF
}

main() {
    local command="${1:-deploy}"

    # Traitement des options
    while [[ $# -gt 0 ]]; do
        case $1 in
            -c|--config)
                CONFIG_FILE="$2"
                shift 2
                ;;
            -b|--branch)
                BRANCH="$2"
                shift 2
                ;;
            -v|--verbose)
                set -x
                shift
                ;;
            -h|--help)
                show_help
                exit 0
                ;;
            *)
                command="$1"
                shift
                ;;
        esac
    done

    # Initialisation
    init_deployment

    # Exécution de la commande
    case "$command" in
        deploy)
            full_deployment
            ;;
        multi-deploy)
            deploy_to_servers
            ;;
        rollback)
            local backup_file="${1:-}"
            if [ -z "$backup_file" ]; then
                echo "Sauvegardes disponibles:"
                list_backups
                echo ""
                read -p "Entrez le nom de la sauvegarde: " backup_file
            fi

            if [ -f "$BACKUP_DIR/$backup_file" ]; then
                rollback_deployment "$BACKUP_DIR/$backup_file"
            else
                log "ERROR" "Sauvegarde non trouvée: $backup_file"
                exit 1
            fi
            ;;
        status)
            show_status
            ;;
        health-check)
            perform_health_check
            ;;
        backup)
            local manual_id="manual_$(date +%Y%m%d_%H%M%S)"
            create_backup "$manual_id"
            ;;
        list-backups)
            list_backups
            ;;
        clean)
            log "INFO" "Nettoyage des fichiers temporaires"
            rm -rf tmp/checkout_*
            find "$LOG_DIR" -name "*.log" -mtime +30 -delete 2>/dev/null || true
            log "INFO" "Nettoyage terminé"
            ;;
        *)
            echo "Commande inconnue: $command"
            show_help
            exit 1
            ;;
    esac
}

# Point d'entrée
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
fi
```

## Intégration avec CI/CD (GitLab CI exemple)

### Configuration GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy-staging
  - deploy-production

variables:
  DEPLOY_SCRIPT: "scripts/deploy.sh"

# Tests automatiques
test:
  stage: test
  script:
    - echo "Exécution des tests..."
    - npm test
    - bash scripts/run-tests.sh
  only:
    - merge_requests
    - main
    - develop

# Build de l'application
build:
  stage: build
  script:
    - echo "Build de l'application..."
    - npm install
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour
  only:
    - main
    - develop

# Déploiement staging automatique
deploy-staging:
  stage: deploy-staging
  script:
    - chmod +x $DEPLOY_SCRIPT
    - $DEPLOY_SCRIPT --config staging.conf deploy
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

# Déploiement production manuel
deploy-production:
  stage: deploy-production
  script:
    - chmod +x $DEPLOY_SCRIPT
    - $DEPLOY_SCRIPT --config production.conf deploy
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - main
```

### Script d'intégration CI/CD

```bash
#!/bin/bash

################################################################################
# INTÉGRATION CI/CD
################################################################################
# Description: Script d'intégration pour pipelines CI/CD
################################################################################

set -euo pipefail

# Variables CI/CD
CI_COMMIT_SHA="${CI_COMMIT_SHA:-$(git rev-parse HEAD)}"
CI_COMMIT_REF_NAME="${CI_COMMIT_REF_NAME:-$(git branch --show-current)}"
CI_PIPELINE_ID="${CI_PIPELINE_ID:-manual}"
CI_PROJECT_NAME="${CI_PROJECT_NAME:-unknown}"

# === FONCTIONS CI/CD ===

setup_ci_environment() {
    log "INFO" "Configuration de l'environnement CI/CD"
    log "INFO" "Commit: $CI_COMMIT_SHA"
    log "INFO" "Branche: $CI_COMMIT_REF_NAME"
    log "INFO" "Pipeline: $CI_PIPELINE_ID"
    log "INFO" "Projet: $CI_PROJECT_NAME"

    # Créer les répertoires nécessaires
    mkdir -p artifacts reports

    # Configurer Git si nécessaire
    if [ -n "${GITLAB_USER_EMAIL:-}" ]; then
        git config --global user.email "$GITLAB_USER_EMAIL"
        git config --global user.name "$GITLAB_USER_NAME"
    fi
}

generate_ci_report() {
    local deployment_id="$1"
    local status="$2"
    local report_file="reports/deployment_report.json"

    cat > "$report_file" << EOF
{
  "deployment_id": "$deployment_id",
  "status": "$status",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "commit": "$CI_COMMIT_SHA",
  "branch": "$CI_COMMIT_REF_NAME",
  "pipeline": "$CI_PIPELINE_ID",
  "project": "$CI_PROJECT_NAME",
  "environment": "${CI_ENVIRONMENT_NAME:-unknown}",
  "url": "${CI_ENVIRONMENT_URL:-}",
  "artifacts": {
    "logs": "$LOG_DIR/deploy_$(date +%Y%m%d).log",
    "backup": "$BACKUP_DIR/backup_$deployment_id.tar.gz"
  }
}
EOF

    log "INFO" "Rapport CI/CD généré: $report_file"
}

notify_deployment_status() {
    local status="$1"
    local deployment_id="$2"

    # Notification spécifique CI/CD
    local message="Déploiement $status
Projet: $CI_PROJECT_NAME
Branche: $CI_COMMIT_REF_NAME
Commit: ${CI_COMMIT_SHA:0:8}
Pipeline: $CI_PIPELINE_ID"

    send_notification "$status" "$message" "$deployment_id"

    # Génération du rapport
    generate_ci_report "$deployment_id" "$status"
}

# Intégration avec les fonctions de déploiement existantes
ci_full_deployment() {
    setup_ci_environment

    # Utiliser la fonction de déploiement standard
    if full_deployment; then
        notify_deployment_status "success" "$(get_current_version)"
        return 0
    else
        notify_deployment_status "failed" "unknown"
        return 1
    fi
}
```

## Résumé des exemples pratiques

### Scripts développés

1. **Système de sauvegarde incrémentale**
   - Sauvegarde automatisée avec compression et chiffrement
   - Gestion de la rétention et synchronisation distante
   - Interface complète avec rapports

2. **Système de monitoring complet**
   - Collecte de métriques système
   - Génération de rapports HTML
   - Système d'alertes multi-canaux
   - Tableau de bord en temps réel

3. **Automatisation administrative**
   - Maintenance système automatisée
   - Audit de sécurité et durcissement
   - Gestion des utilisateurs
   - Nettoyage et optimisation

4. **Pipeline de déploiement**
   - Déploiement avec tests automatiques
   - Rollback automatique en cas d'échec
   - Déploiement multi-serveurs
   - Intégration CI/CD

### Caractéristiques communes

**🔒 Sécurité intégrée :**
- Validation des entrées
- Gestion des permissions
- Logging sécurisé
- Gestion des secrets

**📊 Monitoring et alertes :**
- Logging détaillé
- Notifications multi-canaux
- Métriques et rapports
- Health checks

**🛠️ Robustesse :**
- Gestion d'erreurs complète
- Rollback automatique
- Opérations atomiques
- Nettoyage automatique

**⚙️ Flexibilité :**
- Configuration externe
- Interface CLI complète
- Mode dry-run
- Extensibilité

Ces scripts représentent des cas d'usage réels en environnement de production et peuvent servir de base pour vos propres projets. Ils intègrent toutes les bonnes pratiques vues dans le tutoriel et démontrent comment créer des outils d'automatisation professionnels avec Bash.

⏭️
