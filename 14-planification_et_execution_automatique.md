🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Planification et exécution automatique

## Introduction

L'automatisation est l'une des forces principales de Bash. Cette section vous apprendra à planifier l'exécution de vos scripts, à les faire démarrer automatiquement avec le système, et à gérer les processus en arrière-plan. Ces techniques vous permettront de créer des systèmes entièrement automatisés.

## 1. Tâches planifiées avec cron

### Qu'est-ce que cron ?

Cron est un démon système qui exécute des tâches à des moments prédéfinis. Il lit les fichiers de configuration appelés "crontabs" qui définissent quand et comment exécuter les commandes.

### Format de la crontab

Chaque ligne de crontab suit ce format :
```
* * * * * commande_à_exécuter
│ │ │ │ │
│ │ │ │ └── Jour de la semaine (0-7, dimanche = 0 ou 7)
│ │ │ └──── Mois (1-12)
│ │ └────── Jour du mois (1-31)
│ └──────── Heure (0-23)
└────────── Minute (0-59)
```

### Gestion de la crontab

#### Commandes de base
```bash
# Afficher la crontab actuelle
crontab -l

# Éditer la crontab
crontab -e

# Supprimer toute la crontab
crontab -r

# Voir la crontab d'un autre utilisateur (root uniquement)
crontab -l -u username
```

### Exemples de planification

#### Exemples simples
```bash
# Exécuter tous les jours à 2h30 du matin
30 2 * * * /home/user/scripts/backup.sh

# Exécuter toutes les heures
0 * * * * /home/user/scripts/check_status.sh

# Exécuter toutes les 15 minutes
*/15 * * * * /home/user/scripts/monitor.sh

# Exécuter tous les lundis à 9h00
0 9 * * 1 /home/user/scripts/weekly_report.sh

# Exécuter le 1er de chaque mois à minuit
0 0 1 * * /home/user/scripts/monthly_cleanup.sh
```

#### Exemples plus complexes
```bash
# Exécuter du lundi au vendredi à 8h30 et 17h30
30 8,17 * * 1-5 /home/user/scripts/workday_task.sh

# Exécuter tous les 2 heures pendant les heures de bureau
0 8-18/2 * * 1-5 /home/user/scripts/business_hours.sh

# Exécuter seulement en janvier et juillet
0 0 1 1,7 * /home/user/scripts/biannual_task.sh

# Exécuter au démarrage du système
@reboot /home/user/scripts/startup.sh
```

### Script de sauvegarde automatique

```bash
#!/bin/bash

# Script de sauvegarde pour cron
# Fichier : /home/user/scripts/auto_backup.sh

# Configuration
SOURCE_DIR="/home/user/documents"
BACKUP_ROOT="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="$BACKUP_ROOT/backup_$DATE"
LOG_FILE="/var/log/auto_backup.log"
MAX_BACKUPS=7  # Garder 7 sauvegardes

# Fonction de log avec timestamp
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG_FILE"
}

# Fonction de nettoyage des anciennes sauvegardes
cleanup_old_backups() {
    log "Nettoyage des anciennes sauvegardes..."

    # Compter les sauvegardes existantes
    local backup_count=$(find "$BACKUP_ROOT" -maxdepth 1 -type d -name "backup_*" | wc -l)

    if [ "$backup_count" -gt "$MAX_BACKUPS" ]; then
        # Supprimer les plus anciennes
        local to_delete=$((backup_count - MAX_BACKUPS))
        find "$BACKUP_ROOT" -maxdepth 1 -type d -name "backup_*" | sort | head -n "$to_delete" | while read -r old_backup; do
            rm -rf "$old_backup"
            log "Suppression de l'ancienne sauvegarde : $(basename "$old_backup")"
        done
    fi

    log "Nettoyage terminé. $MAX_BACKUPS sauvegarde(s) conservée(s)"
}

# Fonction principale de sauvegarde
perform_backup() {
    log "Début de la sauvegarde automatique"

    # Vérifier que le répertoire source existe
    if [ ! -d "$SOURCE_DIR" ]; then
        log "ERREUR: Le répertoire source $SOURCE_DIR n'existe pas"
        exit 1
    fi

    # Créer le répertoire de sauvegarde
    if ! mkdir -p "$BACKUP_DIR"; then
        log "ERREUR: Impossible de créer le répertoire $BACKUP_DIR"
        exit 1
    fi

    # Effectuer la sauvegarde avec rsync
    if rsync -av --delete "$SOURCE_DIR/" "$BACKUP_DIR/" >> "$LOG_FILE" 2>&1; then
        log "Sauvegarde terminée avec succès dans $BACKUP_DIR"

        # Créer une archive compressée
        if tar -czf "$BACKUP_DIR.tar.gz" -C "$BACKUP_ROOT" "$(basename "$BACKUP_DIR")" 2>> "$LOG_FILE"; then
            log "Archive créée : $BACKUP_DIR.tar.gz"
            rm -rf "$BACKUP_DIR"  # Supprimer le répertoire non compressé
        else
            log "ERREUR: Échec de la création de l'archive"
        fi

        # Nettoyage des anciennes sauvegardes
        cleanup_old_backups

    else
        log "ERREUR: Échec de la sauvegarde"
        exit 1
    fi
}

# Rediriger toutes les sorties vers le log (pour cron)
exec >> "$LOG_FILE" 2>&1

# Exécution principale
perform_backup

log "Script de sauvegarde terminé"
```

#### Configuration cron pour la sauvegarde
```bash
# Éditer la crontab
crontab -e

# Ajouter cette ligne pour sauvegarder tous les jours à 2h00
0 2 * * * /home/user/scripts/auto_backup.sh

# Ou pour une sauvegarde plus fréquente (toutes les 6 heures)
0 */6 * * * /home/user/scripts/auto_backup.sh
```

### Script de monitoring système

```bash
#!/bin/bash

# Script de monitoring pour cron
# Fichier : /home/user/scripts/system_monitor.sh

# Configuration
LOG_FILE="/var/log/system_monitor.log"
ALERT_EMAIL="admin@example.com"
CPU_THRESHOLD=80
MEMORY_THRESHOLD=85
DISK_THRESHOLD=90

# Fonction de log
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Fonction d'envoi d'alerte
send_alert() {
    local subject="$1"
    local message="$2"

    {
        echo "Serveur : $(hostname)"
        echo "Date : $(date)"
        echo ""
        echo "$message"
    } | mail -s "$subject" "$ALERT_EMAIL"

    log "Alerte envoyée : $subject"
}

# Vérification CPU
check_cpu() {
    local cpu_usage=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}')

    log "CPU usage: ${cpu_usage}%"

    if (( $(echo "$cpu_usage > $CPU_THRESHOLD" | bc -l) )); then
        send_alert "Alerte CPU élevé" "Utilisation CPU: ${cpu_usage}% (seuil: ${CPU_THRESHOLD}%)"
        return 1
    fi
    return 0
}

# Vérification mémoire
check_memory() {
    local memory_info=$(free | grep Mem)
    local total=$(echo "$memory_info" | awk '{print $2}')
    local used=$(echo "$memory_info" | awk '{print $3}')
    local memory_percent=$(awk "BEGIN {printf \"%.1f\", ($used/$total)*100}")

    log "Memory usage: ${memory_percent}%"

    if (( $(echo "$memory_percent > $MEMORY_THRESHOLD" | bc -l) )); then
        send_alert "Alerte mémoire élevée" "Utilisation mémoire: ${memory_percent}% (seuil: ${MEMORY_THRESHOLD}%)"
        return 1
    fi
    return 0
}

# Vérification disque
check_disk() {
    local disk_usage=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')

    log "Disk usage: ${disk_usage}%"

    if [ "$disk_usage" -gt "$DISK_THRESHOLD" ]; then
        send_alert "Alerte espace disque" "Utilisation disque: ${disk_usage}% (seuil: ${DISK_THRESHOLD}%)"
        return 1
    fi
    return 0
}

# Vérification des services
check_services() {
    local services=("ssh" "nginx" "mysql")
    local failed_services=()

    for service in "${services[@]}"; do
        if ! systemctl is-active --quiet "$service" 2>/dev/null; then
            failed_services+=("$service")
            log "Service $service non actif"
        else
            log "Service $service OK"
        fi
    done

    if [ ${#failed_services[@]} -gt 0 ]; then
        send_alert "Services non disponibles" "Services en échec: ${failed_services[*]}"
        return 1
    fi
    return 0
}

# Fonction principale
main() {
    log "Début du monitoring système"

    local alerts=0

    # Effectuer toutes les vérifications
    check_cpu || ((alerts++))
    check_memory || ((alerts++))
    check_disk || ((alerts++))
    check_services || ((alerts++))

    if [ $alerts -eq 0 ]; then
        log "Tous les contrôles sont OK"
    else
        log "$alerts alerte(s) détectée(s)"
    fi

    log "Monitoring terminé"
}

# Exécution
main
```

#### Configuration cron pour le monitoring
```bash
# Vérifier le système toutes les 15 minutes
*/15 * * * * /home/user/scripts/system_monitor.sh

# Ou plus fréquemment pendant les heures de bureau
*/5 8-18 * * 1-5 /home/user/scripts/system_monitor.sh
```

### Variables d'environnement et cron

Cron a un environnement minimal. Il est important de définir les variables nécessaires :

```bash
# En haut de votre crontab
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
SHELL=/bin/bash
MAILTO=admin@example.com

# Puis vos tâches
0 2 * * * /home/user/scripts/backup.sh
```

### Gestion des logs avec cron

```bash
# Rediriger la sortie vers un fichier de log
0 2 * * * /home/user/scripts/backup.sh >> /var/log/backup.log 2>&1

# Envoyer les erreurs par email et la sortie normale vers un fichier
0 2 * * * /home/user/scripts/backup.sh >> /var/log/backup.log

# Supprimer complètement la sortie
0 2 * * * /home/user/scripts/backup.sh >/dev/null 2>&1
```

## 2. Exécution au démarrage du système

### Méthodes de démarrage automatique

#### 1. Crontab avec @reboot

```bash
# Éditer la crontab
crontab -e

# Ajouter une tâche de démarrage
@reboot /home/user/scripts/startup.sh
@reboot sleep 60 && /home/user/scripts/delayed_startup.sh
```

#### 2. Script d'initialisation systemd (méthode moderne)

Créer un service systemd :

```bash
# Créer le fichier de service
sudo nano /etc/systemd/system/mon-script.service
```

Contenu du fichier service :
```ini
[Unit]
Description=Mon Script de Démarrage
After=network.target

[Service]
Type=oneshot
ExecStart=/home/user/scripts/startup.sh
User=user
Group=user
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

Activation du service :
```bash
# Recharger systemd
sudo systemctl daemon-reload

# Activer le service au démarrage
sudo systemctl enable mon-script.service

# Démarrer le service maintenant
sudo systemctl start mon-script.service

# Vérifier le statut
sudo systemctl status mon-script.service
```

#### 3. Script de démarrage dans /etc/init.d (méthode traditionnelle)

```bash
#!/bin/bash

# Script de démarrage traditionnel
# Fichier : /etc/init.d/mon-service

### BEGIN INIT INFO
# Provides:          mon-service
# Required-Start:    $local_fs $network
# Required-Stop:     $local_fs $network
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Description:       Mon service personnalisé
### END INIT INFO

SERVICE_NAME="mon-service"
SERVICE_USER="user"
SERVICE_SCRIPT="/home/user/scripts/daemon.sh"
PID_FILE="/var/run/${SERVICE_NAME}.pid"

case "$1" in
    start)
        echo "Démarrage de $SERVICE_NAME..."
        if [ -f "$PID_FILE" ]; then
            echo "$SERVICE_NAME est déjà en cours d'exécution"
            exit 1
        fi

        su - "$SERVICE_USER" -c "$SERVICE_SCRIPT" &
        echo $! > "$PID_FILE"
        echo "$SERVICE_NAME démarré"
        ;;

    stop)
        echo "Arrêt de $SERVICE_NAME..."
        if [ -f "$PID_FILE" ]; then
            PID=$(cat "$PID_FILE")
            kill "$PID"
            rm -f "$PID_FILE"
            echo "$SERVICE_NAME arrêté"
        else
            echo "$SERVICE_NAME n'est pas en cours d'exécution"
        fi
        ;;

    restart)
        $0 stop
        sleep 2
        $0 start
        ;;

    status)
        if [ -f "$PID_FILE" ]; then
            PID=$(cat "$PID_FILE")
            if ps -p "$PID" > /dev/null; then
                echo "$SERVICE_NAME est en cours d'exécution (PID: $PID)"
            else
                echo "$SERVICE_NAME n'est pas en cours d'exécution"
                rm -f "$PID_FILE"
            fi
        else
            echo "$SERVICE_NAME n'est pas en cours d'exécution"
        fi
        ;;

    *)
        echo "Usage: $0 {start|stop|restart|status}"
        exit 1
        ;;
esac

exit 0
```

Activation du script :
```bash
# Rendre le script exécutable
sudo chmod +x /etc/init.d/mon-service

# Activer le service au démarrage
sudo update-rc.d mon-service defaults

# Contrôler le service
sudo service mon-service start
sudo service mon-service stop
sudo service mon-service status
```

### Script de démarrage complet

```bash
#!/bin/bash

# Script de démarrage système
# Fichier : /home/user/scripts/startup.sh

LOG_FILE="/var/log/startup_script.log"

# Fonction de log
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Attendre que le réseau soit disponible
wait_for_network() {
    log "Attente de la connectivité réseau..."

    local max_attempts=30
    local attempt=1

    while [ $attempt -le $max_attempts ]; do
        if ping -c 1 8.8.8.8 >/dev/null 2>&1; then
            log "Réseau disponible"
            return 0
        fi

        log "Tentative $attempt/$max_attempts - Réseau non disponible"
        sleep 10
        ((attempt++))
    done

    log "ERREUR: Réseau non disponible après $max_attempts tentatives"
    return 1
}

# Monter les systèmes de fichiers
mount_filesystems() {
    log "Montage des systèmes de fichiers..."

    # Exemple : monter un disque externe
    if [ -b /dev/sdb1 ]; then
        if ! mount /dev/sdb1 /mnt/external; then
            log "ERREUR: Impossible de monter /dev/sdb1"
        else
            log "Disque externe monté avec succès"
        fi
    fi
}

# Démarrer les services personnalisés
start_custom_services() {
    log "Démarrage des services personnalisés..."

    # Démarrer un serveur web personnalisé
    if [ -x "/home/user/bin/web_server" ]; then
        nohup /home/user/bin/web_server > /var/log/web_server.log 2>&1 &
        echo $! > /var/run/web_server.pid
        log "Serveur web démarré (PID: $!)"
    fi

    # Démarrer un script de monitoring
    if [ -x "/home/user/scripts/monitor_daemon.sh" ]; then
        nohup /home/user/scripts/monitor_daemon.sh > /var/log/monitor.log 2>&1 &
        echo $! > /var/run/monitor.pid
        log "Daemon de monitoring démarré (PID: $!)"
    fi
}

# Configuration du système
configure_system() {
    log "Configuration du système..."

    # Ajuster les paramètres réseau
    echo 'net.core.rmem_max = 16777216' >> /etc/sysctl.conf
    sysctl -p >/dev/null 2>&1

    # Créer des répertoires nécessaires
    mkdir -p /var/log/myapp
    mkdir -p /tmp/processing

    # Définir les permissions
    chmod 755 /var/log/myapp
    chmod 1777 /tmp/processing

    log "Configuration système terminée"
}

# Synchronisation des données
sync_data() {
    log "Synchronisation des données..."

    # Synchroniser avec un serveur distant
    if rsync -av --timeout=60 user@server:/data/ /local/data/; then
        log "Synchronisation réussie"
    else
        log "ERREUR: Échec de la synchronisation"
    fi
}

# Fonction principale
main() {
    log "=== DÉMARRAGE DU SCRIPT D'INITIALISATION ==="
    log "Hostname: $(hostname)"
    log "Utilisateur: $(whoami)"

    # Attendre un délai pour que le système se stabilise
    log "Attente de 30 secondes pour la stabilisation du système..."
    sleep 30

    # Exécuter les tâches de démarrage
    wait_for_network
    mount_filesystems
    configure_system
    start_custom_services
    sync_data

    log "=== SCRIPT D'INITIALISATION TERMINÉ ==="
}

# Exécution
main "$@"
```

## 3. Gestion des jobs en arrière-plan (nohup, &)

### Exécution en arrière-plan avec &

```bash
# Lancer un script en arrière-plan
./long_script.sh &

# Récupérer le PID du processus
SCRIPT_PID=$!
echo "Script lancé avec PID: $SCRIPT_PID"

# Attendre que le processus se termine
wait $SCRIPT_PID
echo "Script terminé"
```

### Protection contre la déconnexion avec nohup

```bash
# Lancer un script qui continue après déconnexion
nohup ./long_script.sh &

# Rediriger la sortie vers un fichier spécifique
nohup ./long_script.sh > output.log 2>&1 &

# Récupérer le PID et le sauvegarder
nohup ./long_script.sh &
echo $! > script.pid
```

### Script de gestion de daemon

```bash
#!/bin/bash

# Gestionnaire de daemon
# Fichier : /home/user/scripts/daemon_manager.sh

DAEMON_SCRIPT="/home/user/scripts/my_daemon.sh"
PID_FILE="/var/run/my_daemon.pid"
LOG_FILE="/var/log/my_daemon.log"
DAEMON_NAME="mon-daemon"

# Fonction pour démarrer le daemon
start_daemon() {
    if [ -f "$PID_FILE" ]; then
        local pid=$(cat "$PID_FILE")
        if ps -p "$pid" > /dev/null 2>&1; then
            echo "$DAEMON_NAME est déjà en cours d'exécution (PID: $pid)"
            return 1
        else
            echo "Fichier PID obsolète détecté, suppression..."
            rm -f "$PID_FILE"
        fi
    fi

    echo "Démarrage de $DAEMON_NAME..."
    nohup "$DAEMON_SCRIPT" > "$LOG_FILE" 2>&1 &
    local pid=$!
    echo $pid > "$PID_FILE"

    # Vérifier que le processus a bien démarré
    sleep 2
    if ps -p "$pid" > /dev/null 2>&1; then
        echo "$DAEMON_NAME démarré avec succès (PID: $pid)"
        return 0
    else
        echo "Erreur: $DAEMON_NAME n'a pas pu démarrer"
        rm -f "$PID_FILE"
        return 1
    fi
}

# Fonction pour arrêter le daemon
stop_daemon() {
    if [ ! -f "$PID_FILE" ]; then
        echo "$DAEMON_NAME n'est pas en cours d'exécution"
        return 1
    fi

    local pid=$(cat "$PID_FILE")

    if ! ps -p "$pid" > /dev/null 2>&1; then
        echo "$DAEMON_NAME n'est pas en cours d'exécution"
        rm -f "$PID_FILE"
        return 1
    fi

    echo "Arrêt de $DAEMON_NAME (PID: $pid)..."

    # Envoi du signal TERM
    kill -TERM "$pid"

    # Attendre l'arrêt gracieux
    local timeout=10
    while [ $timeout -gt 0 ] && ps -p "$pid" > /dev/null 2>&1; do
        sleep 1
        ((timeout--))
    done

    # Si le processus ne s'est pas arrêté, forcer l'arrêt
    if ps -p "$pid" > /dev/null 2>&1; then
        echo "Arrêt forcé de $DAEMON_NAME..."
        kill -KILL "$pid"
        sleep 2
    fi

    rm -f "$PID_FILE"
    echo "$DAEMON_NAME arrêté"
    return 0
}

# Fonction pour redémarrer le daemon
restart_daemon() {
    stop_daemon
    sleep 3
    start_daemon
}

# Fonction pour vérifier le statut
status_daemon() {
    if [ ! -f "$PID_FILE" ]; then
        echo "$DAEMON_NAME n'est pas en cours d'exécution"
        return 1
    fi

    local pid=$(cat "$PID_FILE")

    if ps -p "$pid" > /dev/null 2>&1; then
        echo "$DAEMON_NAME est en cours d'exécution (PID: $pid)"

        # Afficher des informations supplémentaires
        local start_time=$(ps -o lstart= -p "$pid")
        local cpu_usage=$(ps -o %cpu= -p "$pid")
        local mem_usage=$(ps -o %mem= -p "$pid")

        echo "  Démarré le: $start_time"
        echo "  CPU: ${cpu_usage}%"
        echo "  Mémoire: ${mem_usage}%"

        return 0
    else
        echo "$DAEMON_NAME n'est pas en cours d'exécution (PID file obsolète)"
        rm -f "$PID_FILE"
        return 1
    fi
}

# Fonction pour surveiller le daemon
monitor_daemon() {
    echo "Surveillance de $DAEMON_NAME (Ctrl+C pour arrêter)..."

    while true; do
        if status_daemon > /dev/null; then
            echo "[$(date '+%H:%M:%S')] $DAEMON_NAME OK"
        else
            echo "[$(date '+%H:%M:%S')] $DAEMON_NAME NOK - Tentative de redémarrage..."
            start_daemon
        fi
        sleep 30
    done
}

# Fonction d'aide
show_help() {
    cat << EOF
Usage: $0 {start|stop|restart|status|monitor}

Commandes:
  start     Démarrer le daemon
  stop      Arrêter le daemon
  restart   Redémarrer le daemon
  status    Afficher le statut du daemon
  monitor   Surveiller le daemon en continu

Fichiers:
  PID:      $PID_FILE
  Log:      $LOG_FILE
  Script:   $DAEMON_SCRIPT

EOF
}

# Traitement des arguments
case "${1:-}" in
    start)
        start_daemon
        ;;
    stop)
        stop_daemon
        ;;
    restart)
        restart_daemon
        ;;
    status)
        status_daemon
        ;;
    monitor)
        monitor_daemon
        ;;
    help|--help|-h)
        show_help
        ;;
    *)
        echo "Erreur: Commande invalide"
        show_help
        exit 1
        ;;
esac
```

### Exemple de daemon simple

```bash
#!/bin/bash

# Daemon simple
# Fichier : /home/user/scripts/my_daemon.sh

DAEMON_NAME="mon-daemon"
LOG_FILE="/var/log/${DAEMON_NAME}.log"
WORK_DIR="/tmp/${DAEMON_NAME}"

# Fonction de log
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$$] $1" >> "$LOG_FILE"
}

# Fonction de nettoyage
cleanup() {
    log "Signal de terminaison reçu, nettoyage en cours..."

    # Nettoyer les fichiers temporaires
    rm -rf "$WORK_DIR"

    log "Daemon arrêté proprement"
    exit 0
}

# Configurer les signaux
trap cleanup TERM INT

# Initialisation
log "Démarrage du daemon $DAEMON_NAME"
mkdir -p "$WORK_DIR"

# Boucle principale du daemon
counter=0
while true; do
    # Travail du daemon
    log "Itération $counter - Traitement en cours..."

    # Exemple de traitement
    {
        echo "Timestamp: $(date)"
        echo "PID: $$"
        echo "Counter: $counter"
        echo "Memory: $(free -h | grep Mem | awk '{print $3}')"
    } > "$WORK_DIR/status_$counter.txt"

    # Nettoyer les anciens fichiers de statut
    find "$WORK_DIR" -name "status_*.txt" -type f -mmin +60 -delete

    # Attendre avant la prochaine itération
    sleep 60

    ((counter++))
done
```

### Gestion avancée des processus

#### Script de surveillance de processus

```bash
#!/bin/bash

# Surveillance de processus critiques
# Redémarre automatiquement les processus qui s'arrêtent

PROCESSES=(
    "nginx:/usr/sbin/nginx"
    "mysql:/usr/sbin/mysqld"
    "ssh:/usr/sbin/sshd"
)

LOG_FILE="/var/log/process_watchdog.log"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

check_and_restart() {
    local name="$1"
    local command="$2"

    if ! pgrep -f "$command" > /dev/null; then
        log "ALERTE: $name n'est pas en cours d'exécution"
        log "Tentative de redémarrage de $name..."

        case "$name" in
            nginx)
                systemctl start nginx
                ;;
            mysql)
                systemctl start mysql
                ;;
            ssh)
                systemctl start ssh
                ;;
        esac

        sleep 5

        if pgrep -f "$command" > /dev/null; then
            log "SUCCESS: $name redémarré avec succès"
        else
            log "ERREUR: Impossible de redémarrer $name"
        fi
    else
        log "OK: $name fonctionne normalement"
    fi
}

# Surveillance en boucle
while true; do
    for process_info in "${PROCESSES[@]}"; do
        IFS=':' read -r name command <<< "$process_info"
        check_and_restart "$name" "$command"
    done

    sleep 300  # Vérifier toutes les 5 minutes
done
```

## Exemple complet : Système de sauvegarde automatisé

```bash
#!/bin/bash

# Système de sauvegarde automatisé complet
# Combine cron, gestion de processus et monitoring
# Fichier : /home/user/scripts/backup_system.sh

set -euo pipefail

# === CONFIGURATION ===
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly CONFIG_FILE="$SCRIPT_DIR/backup_config.conf"
readonly LOG_DIR="/var/log/backup_system"
readonly PID_DIR="/var/run/backup_system"
readonly LOCK_FILE="/var/lock/backup_system.lock"

# Valeurs par défaut
DEFAULT_SOURCE_DIRS=("/home" "/etc" "/var/www")
DEFAULT_BACKUP_ROOT="/backup"
DEFAULT_RETENTION_DAYS=30
DEFAULT_MAX_CONCURRENT=2
DEFAULT_COMPRESSION=true
DEFAULT_REMOTE_HOST=""
DEFAULT_NOTIFICATION_EMAIL=""

# === FONCTIONS UTILITAIRES ===

# Chargement de la configuration
load_config() {
    # Créer un fichier de configuration par défaut s'il n'existe pas
    if [ ! -f "$CONFIG_FILE" ]; then
        cat > "$CONFIG_FILE" << EOF
# Configuration du système de sauvegarde
SOURCE_DIRS=("/home" "/etc" "/var/www")
BACKUP_ROOT="/backup"
RETENTION_DAYS=30
MAX_CONCURRENT=2
COMPRESSION=true
REMOTE_HOST=""
NOTIFICATION_EMAIL="admin@example.com"
WEBHOOK_URL=""
EOF
        echo "Fichier de configuration créé : $CONFIG_FILE"
    fi

    source "$CONFIG_FILE"

    # Utiliser les valeurs par défaut si non définies
    SOURCE_DIRS=${SOURCE_DIRS:-"${DEFAULT_SOURCE_DIRS[@]}"}
    BACKUP_ROOT=${BACKUP_ROOT:-$DEFAULT_BACKUP_ROOT}
    RETENTION_DAYS=${RETENTION_DAYS:-$DEFAULT_RETENTION_DAYS}
    MAX_CONCURRENT=${MAX_CONCURRENT:-$DEFAULT_MAX_CONCURRENT}
    COMPRESSION=${COMPRESSION:-$DEFAULT_COMPRESSION}
    REMOTE_HOST=${REMOTE_HOST:-$DEFAULT_REMOTE_HOST}
    NOTIFICATION_EMAIL=${NOTIFICATION_EMAIL:-$DEFAULT_NOTIFICATION_EMAIL}
}

# Initialisation des répertoires
init_directories() {
    for dir in "$LOG_DIR" "$PID_DIR" "$BACKUP_ROOT"; do
        if [ ! -d "$dir" ]; then
            mkdir -p "$dir"
            echo "Répertoire créé : $dir"
        fi
    done
}

# Fonction de log avec rotation
log() {
    local level="$1"
    shift
    local message="$*"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    local log_file="$LOG_DIR/backup_$(date +%Y%m%d).log"

    echo "[$timestamp] [$level] [$$] $message" | tee -a "$log_file"

    # Rotation des logs (garder 7 jours)
    find "$LOG_DIR" -name "backup_*.log" -type f -mtime +7 -delete 2>/dev/null || true
}

# Gestion des verrous pour éviter les exécutions simultanées
acquire_lock() {
    if [ -f "$LOCK_FILE" ]; then
        local lock_pid=$(cat "$LOCK_FILE")
        if ps -p "$lock_pid" > /dev/null 2>&1; then
            log "ERROR" "Une autre instance est déjà en cours d'exécution (PID: $lock_pid)"
            exit 1
        else
            log "WARN" "Verrou obsolète détecté, suppression"
            rm -f "$LOCK_FILE"
        fi
    fi

    echo $$ > "$LOCK_FILE"
    log "INFO" "Verrou acquis"
}

# Libération du verrou
release_lock() {
    if [ -f "$LOCK_FILE" ]; then
        rm -f "$LOCK_FILE"
        log "INFO" "Verrou libéré"
    fi
}

# Nettoyage à la sortie
cleanup() {
    log "INFO" "Nettoyage en cours..."

    # Tuer tous les processus enfants
    local children=$(jobs -p)
    if [ -n "$children" ]; then
        log "INFO" "Arrêt des processus de sauvegarde en cours..."
        kill $children 2>/dev/null || true
        wait
    fi

    release_lock
    log "INFO" "Nettoyage terminé"
}

# Configuration des signaux
trap cleanup EXIT INT TERM

# === FONCTIONS DE NOTIFICATION ===

send_email_notification() {
    local subject="$1"
    local message="$2"

    if [ -n "$NOTIFICATION_EMAIL" ] && command -v mail >/dev/null 2>&1; then
        {
            echo "Serveur: $(hostname)"
            echo "Date: $(date)"
            echo "Script: $0"
            echo ""
            echo "$message"
        } | mail -s "$subject" "$NOTIFICATION_EMAIL"
        log "INFO" "Notification envoyée par email"
    fi
}

send_webhook_notification() {
    local status="$1"
    local message="$2"

    if [ -n "$WEBHOOK_URL" ] && command -v curl >/dev/null 2>&1; then
        local payload=$(cat << EOF
{
    "text": "Sauvegarde système",
    "status": "$status",
    "message": "$message",
    "hostname": "$(hostname)",
    "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
)

        curl -s -X POST \
            -H "Content-Type: application/json" \
            -d "$payload" \
            "$WEBHOOK_URL" >/dev/null || true

        log "INFO" "Notification webhook envoyée"
    fi
}

# === FONCTIONS DE SAUVEGARDE ===

# Sauvegarde d'un répertoire
backup_directory() {
    local source_dir="$1"
    local backup_name="$2"
    local timestamp="$3"

    log "INFO" "Début sauvegarde: $source_dir"

    if [ ! -d "$source_dir" ]; then
        log "ERROR" "Répertoire source inexistant: $source_dir"
        return 1
    fi

    local backup_file="$BACKUP_ROOT/${backup_name}_${timestamp}"

    # Ajouter l'extension selon la compression
    if [ "$COMPRESSION" = true ]; then
        backup_file="${backup_file}.tar.gz"
        local tar_options="-czf"
    else
        backup_file="${backup_file}.tar"
        local tar_options="-cf"
    fi

    # Fichier temporaire pour éviter les sauvegardes incomplètes
    local temp_file="${backup_file}.tmp"

    # Effectuer la sauvegarde
    if tar $tar_options "$temp_file" -C "$(dirname "$source_dir")" "$(basename "$source_dir")" 2>/dev/null; then
        mv "$temp_file" "$backup_file"
        log "INFO" "Sauvegarde réussie: $backup_file"

        # Calculer et afficher la taille
        local size=$(du -h "$backup_file" | cut -f1)
        log "INFO" "Taille de $backup_name: $size"

        return 0
    else
        log "ERROR" "Échec sauvegarde: $source_dir"
        rm -f "$temp_file"
        return 1
    fi
}

# Nettoyage des anciennes sauvegardes
cleanup_old_backups() {
    log "INFO" "Nettoyage des sauvegardes anciennes (>$RETENTION_DAYS jours)"

    local deleted_count=0

    find "$BACKUP_ROOT" -type f \( -name "*.tar.gz" -o -name "*.tar" \) -mtime +$RETENTION_DAYS | while read -r old_backup; do
        if rm "$old_backup"; then
            log "INFO" "Suppression: $(basename "$old_backup")"
            ((deleted_count++))
        else
            log "ERROR" "Échec suppression: $old_backup"
        fi
    done

    log "INFO" "Nettoyage terminé"
}

# Synchronisation vers serveur distant
sync_to_remote() {
    if [ -z "$REMOTE_HOST" ]; then
        log "INFO" "Pas de serveur distant configuré"
        return 0
    fi

    log "INFO" "Synchronisation vers $REMOTE_HOST"

    # Utiliser rsync pour la synchronisation
    if rsync -av --delete --timeout=3600 "$BACKUP_ROOT/" "$REMOTE_HOST"; then
        log "INFO" "Synchronisation réussie"
        return 0
    else
        log "ERROR" "Échec de la synchronisation"
        return 1
    fi
}

# === FONCTIONS DE MONITORING ===

check_disk_space() {
    local backup_partition=$(df "$BACKUP_ROOT" | tail -1 | awk '{print $5}' | sed 's/%//')

    if [ "$backup_partition" -gt 90 ]; then
        log "ERROR" "Espace disque critique: ${backup_partition}%"
        send_email_notification "ALERTE: Espace disque critique" \
            "L'espace disque sur la partition de sauvegarde est à ${backup_partition}%"
        return 1
    elif [ "$backup_partition" -gt 80 ]; then
        log "WARN" "Espace disque élevé: ${backup_partition}%"
    fi

    return 0
}

generate_backup_report() {
    local timestamp="$1"
    local report_file="$LOG_DIR/rapport_${timestamp}.txt"

    {
        echo "=== RAPPORT DE SAUVEGARDE ==="
        echo "Date: $(date)"
        echo "Serveur: $(hostname)"
        echo ""

        echo "Répertoires sauvegardés:"
        for dir in "${SOURCE_DIRS[@]}"; do
            if [ -d "$dir" ]; then
                echo "  ✓ $dir"
            else
                echo "  ✗ $dir (inexistant)"
            fi
        done

        echo ""
        echo "Sauvegardes créées:"
        find "$BACKUP_ROOT" -name "*_${timestamp}*" -type f -ls

        echo ""
        echo "Utilisation de l'espace:"
        df -h "$BACKUP_ROOT"

        echo ""
        echo "Sauvegardes totales:"
        ls -la "$BACKUP_ROOT" | grep -E '\.(tar|tar\.gz)$' | wc -l

    } > "$report_file"

    log "INFO" "Rapport généré: $report_file"
}

# === FONCTION PRINCIPALE ===

perform_full_backup() {
    local timestamp=$(date +%Y%m%d_%H%M%S)
    local success_count=0
    local total_count=0
    local failed_dirs=()

    log "INFO" "=== DÉBUT SAUVEGARDE COMPLÈTE ==="
    log "INFO" "Timestamp: $timestamp"

    # Vérifier l'espace disque
    if ! check_disk_space; then
        log "ERROR" "Sauvegarde annulée due à l'espace disque insuffisant"
        return 1
    fi

    # Créer un sémaphore pour limiter les sauvegardes concurrentes
    local semaphore_dir="$PID_DIR/semaphore"
    mkdir -p "$semaphore_dir"

    # Lancer les sauvegardes en parallèle
    for source_dir in "${SOURCE_DIRS[@]}"; do
        if [ -d "$source_dir" ]; then
            ((total_count++))

            # Attendre qu'un slot se libère
            while [ $(ls "$semaphore_dir" | wc -l) -ge $MAX_CONCURRENT ]; do
                sleep 1
            done

            # Lancer la sauvegarde en arrière-plan
            (
                local backup_name=$(basename "$source_dir" | tr '/' '_')
                local pid_file="$semaphore_dir/backup_${backup_name}_$$"
                touch "$pid_file"

                if backup_directory "$source_dir" "$backup_name" "$timestamp"; then
                    echo "SUCCESS:$source_dir" > "${pid_file}.result"
                else
                    echo "FAILED:$source_dir" > "${pid_file}.result"
                fi

                rm -f "$pid_file"
            ) &

        else
            log "WARN" "Répertoire ignoré (inexistant): $source_dir"
        fi
    done

    # Attendre que toutes les sauvegardes se terminent
    wait

    # Compter les résultats
    for result_file in "$semaphore_dir"/*.result; do
        if [ -f "$result_file" ]; then
            local result=$(cat "$result_file")
            if [[ $result == SUCCESS:* ]]; then
                ((success_count++))
            else
                local failed_dir=${result#FAILED:}
                failed_dirs+=("$failed_dir")
            fi
            rm -f "$result_file"
        fi
    done

    # Nettoyer le répertoire sémaphore
    rmdir "$semaphore_dir" 2>/dev/null || true

    log "INFO" "Sauvegardes terminées: $success_count/$total_count réussies"

    # Nettoyage des anciennes sauvegardes
    cleanup_old_backups

    # Synchronisation distante
    sync_to_remote

    # Générer le rapport
    generate_backup_report "$timestamp"

    # Notifications
    if [ $success_count -eq $total_count ]; then
        local message="Sauvegarde complète réussie: $success_count/$total_count répertoires"
        log "INFO" "$message"
        send_email_notification "Sauvegarde réussie" "$message"
        send_webhook_notification "success" "$message"
    else
        local message="Sauvegarde partielle: $success_count/$total_count répertoires. Échecs: ${failed_dirs[*]}"
        log "ERROR" "$message"
        send_email_notification "Sauvegarde partielle" "$message"
        send_webhook_notification "warning" "$message"
    fi

    log "INFO" "=== FIN SAUVEGARDE COMPLÈTE ==="

    return $(( total_count - success_count ))
}

# === FONCTIONS DE GESTION ===

# Daemon de surveillance
run_daemon() {
    log "INFO" "Démarrage du daemon de surveillance"

    while true; do
        # Vérifications périodiques
        check_disk_space

        # Statistiques
        local backup_count=$(find "$BACKUP_ROOT" -name "*.tar*" -type f | wc -l)
        local oldest_backup=$(find "$BACKUP_ROOT" -name "*.tar*" -type f -printf '%T@ %p\n' | sort -n | head -1 | cut -d' ' -f2-)

        log "INFO" "Surveillance: $backup_count sauvegardes, plus ancienne: $(basename "$oldest_backup" 2>/dev/null || echo 'aucune')"

        # Attendre 1 heure
        sleep 3600
    done
}

# Installation des tâches cron
install_cron() {
    local script_path=$(realpath "$0")

    echo "Installation des tâches cron..."

    # Sauvegarder la crontab actuelle
    crontab -l > /tmp/current_cron 2>/dev/null || touch /tmp/current_cron

    # Supprimer les anciennes entrées du script
    grep -v "$script_path" /tmp/current_cron > /tmp/new_cron

    # Ajouter les nouvelles tâches
    cat >> /tmp/new_cron << EOF

# === SYSTÈME DE SAUVEGARDE AUTOMATISÉ ===
# Sauvegarde complète tous les jours à 2h00
0 2 * * * $script_path backup >/dev/null 2>&1

# Nettoyage hebdomadaire le dimanche à 3h00
0 3 * * 0 $script_path cleanup >/dev/null 2>&1

# Vérification de l'espace disque toutes les 6 heures
0 */6 * * * $script_path check >/dev/null 2>&1
EOF

    # Installer la nouvelle crontab
    crontab /tmp/new_cron
    rm -f /tmp/current_cron /tmp/new_cron

    echo "Tâches cron installées avec succès"
    echo "Voir avec: crontab -l"
}

# Désinstallation des tâches cron
uninstall_cron() {
    local script_path=$(realpath "$0")

    echo "Désinstallation des tâches cron..."

    crontab -l 2>/dev/null | grep -v "$script_path" | crontab -

    echo "Tâches cron supprimées"
}

# Affichage du statut
show_status() {
    echo "=== STATUT DU SYSTÈME DE SAUVEGARDE ==="
    echo ""

    echo "Configuration:"
    echo "  Répertoires source: ${SOURCE_DIRS[*]}"
    echo "  Répertoire de sauvegarde: $BACKUP_ROOT"
    echo "  Rétention: $RETENTION_DAYS jours"
    echo "  Compression: $COMPRESSION"
    echo "  Serveur distant: ${REMOTE_HOST:-'aucun'}"
    echo ""

    echo "Statistiques:"
    if [ -d "$BACKUP_ROOT" ]; then
        local backup_count=$(find "$BACKUP_ROOT" -name "*.tar*" -type f 2>/dev/null | wc -l)
        local total_size=$(du -sh "$BACKUP_ROOT" 2>/dev/null | cut -f1)
        echo "  Nombre de sauvegardes: $backup_count"
        echo "  Taille totale: $total_size"

        if [ $backup_count -gt 0 ]; then
            local newest=$(find "$BACKUP_ROOT" -name "*.tar*" -type f -printf '%T@ %p\n' 2>/dev/null | sort -n | tail -1 | cut -d' ' -f2-)
            local oldest=$(find "$BACKUP_ROOT" -name "*.tar*" -type f -printf '%T@ %p\n' 2>/dev/null | sort -n | head -1 | cut -d' ' -f2-)
            echo "  Plus récente: $(basename "$newest") ($(date -r "$newest" '+%Y-%m-%d %H:%M'))"
            echo "  Plus ancienne: $(basename "$oldest") ($(date -r "$oldest" '+%Y-%m-%d %H:%M'))"
        fi
    else
        echo "  Répertoire de sauvegarde non trouvé"
    fi
    echo ""

    echo "Espace disque:"
    df -h "$BACKUP_ROOT" 2>/dev/null || echo "  Information non disponible"
    echo ""

    echo "Processus actifs:"
    if [ -f "$LOCK_FILE" ]; then
        local lock_pid=$(cat "$LOCK_FILE")
        if ps -p "$lock_pid" > /dev/null 2>&1; then
            echo "  Sauvegarde en cours (PID: $lock_pid)"
        else
            echo "  Verrou obsolète détecté"
        fi
    else
        echo "  Aucune sauvegarde en cours"
    fi
    echo ""

    echo "Logs récents:"
    local today_log="$LOG_DIR/backup_$(date +%Y%m%d).log"
    if [ -f "$today_log" ]; then
        echo "  Fichier du jour: $today_log"
        echo "  Dernières entrées:"
        tail -5 "$today_log" | sed 's/^/    /'
    else
        echo "  Aucun log pour aujourd'hui"
    fi
}

# === FONCTION PRINCIPALE ET GESTION DES ARGUMENTS ===

show_help() {
    cat << EOF
Usage: $0 [COMMAND] [OPTIONS]

Système de sauvegarde automatisé avec planification

COMMANDES:
    backup              Effectuer une sauvegarde complète
    daemon              Lancer le daemon de surveillance
    cleanup             Nettoyer les anciennes sauvegardes
    check               Vérifier l'espace disque et le système
    status              Afficher le statut du système
    install-cron        Installer les tâches cron
    uninstall-cron      Désinstaller les tâches cron
    config              Afficher la configuration
    help                Afficher cette aide

OPTIONS:
    -c, --config FILE   Utiliser un fichier de configuration spécifique
    -v, --verbose       Mode verbeux
    -n, --dry-run       Mode test (ne pas effectuer les sauvegardes)

EXEMPLES:
    $0 backup                    # Sauvegarde manuelle
    $0 daemon                    # Lancer le daemon
    $0 install-cron              # Installer la planification
    $0 status                    # Voir le statut
    $0 -c /etc/backup.conf backup # Utiliser une config spécifique

FICHIERS:
    Configuration:   $CONFIG_FILE
    Logs:           $LOG_DIR/
    PID files:      $PID_DIR/
    Lock file:      $LOCK_FILE

EOF
}

# Fonction principale
main() {
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
            -n|--dry-run)
                DRY_RUN=true
                shift
                ;;
            -h|--help|help)
                show_help
                exit 0
                ;;
            *)
                break
                ;;
        esac
    done

    # Initialisation
    load_config
    init_directories

    # Commande à exécuter
    local command="${1:-backup}"

    case "$command" in
        backup)
            acquire_lock
            perform_full_backup
            ;;
        daemon)
            acquire_lock
            run_daemon
            ;;
        cleanup)
            cleanup_old_backups
            ;;
        check)
            check_disk_space
            ;;
        status)
            show_status
            ;;
        install-cron)
            install_cron
            ;;
        uninstall-cron)
            uninstall_cron
            ;;
        config)
            echo "Configuration actuelle:"
            cat "$CONFIG_FILE"
            ;;
        *)
            echo "Erreur: Commande inconnue '$command'"
            echo "Utilisez '$0 help' pour voir l'aide"
            exit 1
            ;;
    esac
}

# Point d'entrée
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
fi
```

## Bonnes pratiques pour la planification et l'automatisation

### 1. Gestion des erreurs et logging

```bash
# Toujours rediriger les sorties dans cron
0 2 * * * /path/to/script.sh >> /var/log/script.log 2>&1

# Ou utiliser un wrapper de logging
0 2 * * * /usr/local/bin/log-wrapper /path/to/script.sh
```

### 2. Variables d'environnement

```bash
# Définir l'environnement dans la crontab
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
SHELL=/bin/bash
HOME=/root

# Ou dans le script
export PATH="/usr/local/bin:/usr/bin:/bin"
export LANG="en_US.UTF-8"
```

### 3. Vérifications avant exécution

```bash
#!/bin/bash

# Vérifier que le script ne tourne pas déjà
LOCK_FILE="/var/lock/$(basename "$0").lock"

if [ -f "$LOCK_FILE" ]; then
    if ps -p "$(cat "$LOCK_FILE")" > /dev/null; then
        echo "Script déjà en cours d'exécution"
        exit 1
    else
        rm -f "$LOCK_FILE"
    fi
fi

echo $$ > "$LOCK_FILE"
trap 'rm -f "$LOCK_FILE"' EXIT
```

### 4. Gestion des ressources

```bash
# Limiter l'utilisation CPU/mémoire
nice -n 19 ionice -c 3 /path/to/script.sh

# Utiliser des timeouts
timeout 3600 /path/to/script.sh
```

### 5. Monitoring et alertes

```bash
# Script de vérification de la dernière exécution
#!/bin/bash

LAST_RUN_FILE="/var/log/last_backup.timestamp"
MAX_AGE=86400  # 24 heures

if [ -f "$LAST_RUN_FILE" ]; then
    LAST_RUN=$(cat "$LAST_RUN_FILE")
    CURRENT_TIME=$(date +%s)
    AGE=$((CURRENT_TIME - LAST_RUN))

    if [ $AGE -gt $MAX_AGE ]; then
        echo "ALERTE: Dernière sauvegarde il y a $((AGE/3600)) heures"
        # Envoyer une alerte
    fi
else
    echo "ALERTE: Aucune trace de sauvegarde"
fi
```

## Résumé

La planification et l'exécution automatique permettent de créer des systèmes robustes et autonomes :

- **Cron** : Planification précise des tâches avec une syntaxe flexible
- **Démarrage système** : Scripts qui s'exécutent automatiquement au boot
- **Jobs en arrière-plan** : Gestion des processus avec `nohup`, `&`, et les daemons
- **Monitoring** : Surveillance continue et alertes automatiques

En combinant ces techniques, vous pouvez créer des systèmes d'automatisation complets qui fonctionnent de manière autonome et fiable, avec une surveillance et une récupération d'erreurs appropriées.

⏭️
