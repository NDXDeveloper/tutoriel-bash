🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Outils de ligne de commande essentiels pour les scripts

## Introduction

Cette section présente les outils de ligne de commande indispensables pour créer des scripts Bash puissants et efficaces. Ces outils vous permettront de manipuler des fichiers, traiter des données, interagir avec des services réseau et gérer des formats de données modernes comme JSON.

## 1. find et xargs pour la gestion avancée des fichiers

### La commande find : recherche de fichiers

`find` est l'outil le plus puissant pour rechercher des fichiers et répertoires selon différents critères.

#### Syntaxe de base
```bash
find [répertoire] [critères] [actions]
```

#### Recherche par nom
```bash
#!/bin/bash

# Rechercher tous les fichiers .txt
find /home/user -name "*.txt"

# Recherche insensible à la casse
find /home/user -iname "*.TXT"

# Rechercher par nom exact
find /var/log -name "syslog"

# Rechercher dans le répertoire actuel
find . -name "*.sh"
```

#### Recherche par type
```bash
#!/bin/bash

# Rechercher seulement les fichiers
find /home/user -type f -name "*.log"

# Rechercher seulement les répertoires
find /home/user -type d -name "backup*"

# Rechercher les liens symboliques
find /usr/bin -type l
```

#### Recherche par taille
```bash
#!/bin/bash

# Fichiers plus grands que 100MB
find /var -type f -size +100M

# Fichiers plus petits que 1KB
find /tmp -type f -size -1k

# Fichiers entre 10MB et 50MB
find /home -type f -size +10M -size -50M
```

#### Recherche par date
```bash
#!/bin/bash

# Fichiers modifiés dans les 7 derniers jours
find /home/user -type f -mtime -7

# Fichiers plus anciens que 30 jours
find /var/log -type f -mtime +30

# Fichiers modifiés aujourd'hui
find /tmp -type f -mtime 0

# Fichiers accédés dans la dernière heure
find /var/log -type f -amin -60
```

#### Actions avec find
```bash
#!/bin/bash

# Afficher les détails des fichiers trouvés
find /home/user -name "*.txt" -ls

# Exécuter une commande sur chaque fichier trouvé
find /var/log -name "*.log" -exec ls -lh {} \;

# Demander confirmation avant chaque action
find /tmp -name "*.tmp" -ok rm {} \;

# Supprimer tous les fichiers .tmp (attention !)
find /tmp -name "*.tmp" -delete
```

### La commande xargs : traitement en lot

`xargs` permet d'exécuter des commandes sur une liste d'arguments, particulièrement utile avec `find`.

#### Utilisation basique de xargs
```bash
#!/bin/bash

# Compter les lignes dans tous les fichiers .txt
find . -name "*.txt" | xargs wc -l

# Rechercher un motif dans plusieurs fichiers
find /var/log -name "*.log" | xargs grep "ERROR"

# Copier tous les fichiers .conf vers un répertoire
find /etc -name "*.conf" | xargs -I {} cp {} /backup/configs/
```

#### Gestion des espaces dans les noms de fichiers
```bash
#!/bin/bash

# Méthode sécurisée avec -print0 et -0
find . -name "*.txt" -print0 | xargs -0 ls -l

# Alternative avec -exec (plus sûre)
find . -name "*.txt" -exec ls -l {} +
```

### Exemples pratiques de find et xargs

#### Script de nettoyage automatique
```bash
#!/bin/bash

echo "=== NETTOYAGE AUTOMATIQUE ==="

# Supprimer les fichiers temporaires anciens
echo "Suppression des fichiers temporaires..."
find /tmp -type f -name "*.tmp" -mtime +7 -delete
echo "✓ Fichiers temporaires supprimés"

# Nettoyer les logs anciens
echo "Suppression des logs anciens..."
find /var/log -type f -name "*.log" -mtime +30 -delete
echo "✓ Logs anciens supprimés"

# Compresser les gros fichiers
echo "Compression des gros fichiers..."
find /home/user -type f -size +100M -name "*.txt" -exec gzip {} \;
echo "✓ Gros fichiers compressés"

# Rapport des fichiers par extension
echo "=== RAPPORT PAR EXTENSION ==="
find /home/user -type f | sed 's/.*\.//' | sort | uniq -c | sort -nr
```

#### Script de sauvegarde sélective
```bash
#!/bin/bash

BACKUP_DIR="/backup/$(date +%Y%m%d)"
SOURCE_DIR="/home/user"

echo "Création du répertoire de sauvegarde..."
mkdir -p "$BACKUP_DIR"

# Sauvegarder tous les documents modifiés cette semaine
echo "Sauvegarde des documents récents..."
find "$SOURCE_DIR" -type f \( -name "*.doc" -o -name "*.pdf" -o -name "*.txt" \) \
    -mtime -7 -print0 | xargs -0 -I {} cp {} "$BACKUP_DIR"

# Créer un inventaire
echo "Création de l'inventaire..."
find "$BACKUP_DIR" -type f -ls > "$BACKUP_DIR/inventaire.txt"

echo "Sauvegarde terminée dans $BACKUP_DIR"
echo "Fichiers sauvegardés : $(find "$BACKUP_DIR" -type f | wc -l)"
```

## 2. cut, paste, join pour la manipulation de données

### La commande cut : extraction de colonnes

`cut` permet d'extraire des portions spécifiques de chaque ligne d'un fichier.

#### Extraction par position de caractères
```bash
#!/bin/bash

# Extraire les caractères 1 à 10
echo "Hello World" | cut -c1-10

# Extraire le 5ème caractère
echo "Hello World" | cut -c5

# Extraire du 3ème caractère à la fin
echo "Hello World" | cut -c3-
```

#### Extraction par délimiteur
```bash
#!/bin/bash

# Fichier CSV exemple
cat > employes.csv << EOF
nom,prenom,age,salaire,departement
Dupont,Jean,30,3000,IT
Martin,Marie,25,2800,RH
Durand,Pierre,35,3500,IT
EOF

# Extraire la colonne des noms (1ère colonne)
cut -d',' -f1 employes.csv

# Extraire nom et salaire (colonnes 1 et 4)
cut -d',' -f1,4 employes.csv

# Extraire de la 2ème à la 4ème colonne
cut -d',' -f2-4 employes.csv
```

#### Exemples pratiques avec cut
```bash
#!/bin/bash

# Extraire les utilisateurs du système
cut -d':' -f1 /etc/passwd

# Extraire les adresses IP d'un log
grep "Failed login" /var/log/auth.log | cut -d' ' -f10

# Traiter un fichier de log Apache
cut -d' ' -f1,7,9 /var/log/apache2/access.log | head -10
```

### La commande paste : fusion de colonnes

`paste` permet de fusionner des lignes de plusieurs fichiers côte à côte.

#### Utilisation basique
```bash
#!/bin/bash

# Créer des fichiers d'exemple
echo -e "Jean\nMarie\nPierre" > prenoms.txt
echo -e "Dupont\nMartin\nDurand" > noms.txt
echo -e "30\n25\n35" > ages.txt

# Fusionner les fichiers avec une tabulation
paste prenoms.txt noms.txt ages.txt

# Fusionner avec un délimiteur personnalisé
paste -d',' prenoms.txt noms.txt ages.txt

# Résultat :
# Jean,Dupont,30
# Marie,Martin,25
# Pierre,Durand,35
```

#### Fusion avec des options avancées
```bash
#!/bin/bash

# Fusionner en série (toutes les lignes du premier fichier, puis du second)
paste -s prenoms.txt noms.txt

# Utiliser plusieurs délimiteurs
paste -d',;' file1.txt file2.txt file3.txt
```

### La commande join : jointure de fichiers

`join` effectue des jointures sur des fichiers triés selon une clé commune.

#### Préparation des données
```bash
#!/bin/bash

# Fichier des employés
cat > employes.txt << EOF
1 Jean Dupont
2 Marie Martin
3 Pierre Durand
EOF

# Fichier des salaires
cat > salaires.txt << EOF
1 3000
2 2800
3 3500
EOF

# Jointure simple
join employes.txt salaires.txt

# Résultat :
# 1 Jean Dupont 3000
# 2 Marie Martin 2800
# 3 Pierre Durand 3500
```

#### Options avancées de join
```bash
#!/bin/bash

# Jointure avec délimiteur personnalisé
join -t',' employes.csv salaires.csv

# Jointure sur une colonne différente
join -1 2 -2 1 fichier1.txt fichier2.txt

# Jointure externe (garder toutes les lignes)
join -a1 -a2 employes.txt salaires.txt
```

### Exemple pratique : Analyse de logs

```bash
#!/bin/bash

# Script d'analyse de logs web
LOG_FILE="/var/log/apache2/access.log"
REPORT_DIR="/tmp/web_report"

mkdir -p "$REPORT_DIR"

echo "=== ANALYSE DES LOGS WEB ==="

# Extraire les IPs et compter les accès
echo "Top 10 des adresses IP :"
cut -d' ' -f1 "$LOG_FILE" | sort | uniq -c | sort -nr | head -10 > "$REPORT_DIR/top_ips.txt"
cat "$REPORT_DIR/top_ips.txt"

# Extraire les codes de statut
echo -e "\nRépartition des codes de statut :"
cut -d' ' -f9 "$LOG_FILE" | grep -E '^[0-9]+$' | sort | uniq -c | sort -nr > "$REPORT_DIR/status_codes.txt"
cat "$REPORT_DIR/status_codes.txt"

# Extraire les pages les plus visitées
echo -e "\nPages les plus visitées :"
cut -d' ' -f7 "$LOG_FILE" | sort | uniq -c | sort -nr | head -10 > "$REPORT_DIR/top_pages.txt"
cat "$REPORT_DIR/top_pages.txt"

# Créer un rapport combiné
echo -e "\n=== RAPPORT COMPLET ===" > "$REPORT_DIR/rapport_complet.txt"
echo "Généré le $(date)" >> "$REPORT_DIR/rapport_complet.txt"
echo -e "\nTop IPs:" >> "$REPORT_DIR/rapport_complet.txt"
cat "$REPORT_DIR/top_ips.txt" >> "$REPORT_DIR/rapport_complet.txt"

echo "Rapport sauvegardé dans $REPORT_DIR"
```

## 3. Outils réseau (curl, wget, netcat)

### curl : client HTTP polyvalent

`curl` est l'outil de référence pour interagir avec des services web et des APIs.

#### Requêtes HTTP basiques
```bash
#!/bin/bash

# Télécharger une page web
curl https://httpbin.org/get

# Sauvegarder dans un fichier
curl -o page.html https://example.com

# Suivre les redirections
curl -L https://bit.ly/example

# Afficher les en-têtes HTTP
curl -I https://httpbin.org/get

# Mode verbeux pour le débogage
curl -v https://httpbin.org/get
```

#### Envoi de données
```bash
#!/bin/bash

# Requête POST avec des données
curl -X POST -d "name=Jean&age=30" https://httpbin.org/post

# Envoyer du JSON
curl -X POST \
    -H "Content-Type: application/json" \
    -d '{"name":"Jean","age":30}' \
    https://httpbin.org/post

# Uploader un fichier
curl -X POST -F "file=@document.pdf" https://httpbin.org/post

# Authentification basique
curl -u username:password https://httpbin.org/basic-auth/username/password
```

#### Script de monitoring d'API
```bash
#!/bin/bash

# Configuration
API_URL="https://api.example.com/health"
LOG_FILE="/var/log/api_monitor.log"
ALERT_EMAIL="admin@example.com"

# Fonction de log
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Fonction de test de l'API
check_api() {
    local response
    local http_code

    # Effectuer la requête avec timeout
    response=$(curl -s -w "%{http_code}" --max-time 10 "$API_URL")
    http_code="${response: -3}"

    if [ "$http_code" = "200" ]; then
        log_message "API OK - Code: $http_code"
        return 0
    else
        log_message "API ERROR - Code: $http_code"
        return 1
    fi
}

# Fonction d'alerte
send_alert() {
    local message="$1"
    echo "$message" | mail -s "Alerte API" "$ALERT_EMAIL"
    log_message "Alerte envoyée: $message"
}

# Test principal
log_message "Démarrage du monitoring API"

if check_api; then
    log_message "Service opérationnel"
else
    send_alert "L'API $API_URL ne répond pas correctement"
fi
```

### wget : téléchargeur de fichiers

`wget` est spécialisé dans le téléchargement de fichiers et la navigation récursive.

#### Téléchargements basiques
```bash
#!/bin/bash

# Télécharger un fichier
wget https://example.com/file.zip

# Télécharger avec un nom personnalisé
wget -O mon_fichier.zip https://example.com/file.zip

# Télécharger en arrière-plan
wget -b https://example.com/gros_fichier.iso

# Reprendre un téléchargement interrompu
wget -c https://example.com/gros_fichier.iso
```

#### Téléchargements avancés
```bash
#!/bin/bash

# Télécharger récursivement un site
wget -r -np -k https://example.com/docs/

# Limiter la profondeur de récursion
wget -r -l 2 https://example.com/

# Télécharger seulement certains types de fichiers
wget -r -A "*.pdf,*.doc" https://example.com/documents/

# Limiter la vitesse de téléchargement
wget --limit-rate=200k https://example.com/file.zip
```

#### Script de sauvegarde de site web
```bash
#!/bin/bash

# Configuration
SITE_URL="https://example.com"
BACKUP_DIR="/backup/websites/$(date +%Y%m%d)"
LOG_FILE="/var/log/website_backup.log"

# Fonction de log
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Créer le répertoire de sauvegarde
mkdir -p "$BACKUP_DIR"
cd "$BACKUP_DIR"

log "Début de la sauvegarde de $SITE_URL"

# Télécharger le site
wget -r -np -k -p -E -nc \
    --user-agent="Mozilla/5.0 (compatible; backup-bot)" \
    --wait=1 \
    --limit-rate=500k \
    "$SITE_URL" 2>&1 | tee -a "$LOG_FILE"

if [ $? -eq 0 ]; then
    log "Sauvegarde terminée avec succès"

    # Créer une archive
    tar -czf "backup_$(date +%Y%m%d_%H%M%S).tar.gz" *
    log "Archive créée"
else
    log "Erreur lors de la sauvegarde"
fi
```

### netcat : couteau suisse réseau

`netcat` (ou `nc`) est un outil polyvalent pour les communications réseau.

#### Tests de connectivité
```bash
#!/bin/bash

# Tester si un port est ouvert
nc -zv google.com 80

# Scanner une plage de ports
nc -zv example.com 20-25

# Tester avec timeout
timeout 5 nc -zv example.com 443
```

#### Transfert de fichiers
```bash
#!/bin/bash

# Sur le serveur (récepteur)
nc -l 8080 > fichier_recu.txt

# Sur le client (expéditeur)
nc server_ip 8080 < fichier_a_envoyer.txt
```

#### Script de surveillance réseau
```bash
#!/bin/bash

# Liste des services à surveiller
declare -A SERVICES=(
    ["Web Server"]="example.com 80"
    ["SSH Server"]="example.com 22"
    ["Database"]="db.example.com 3306"
    ["Mail Server"]="mail.example.com 25"
)

LOG_FILE="/var/log/network_monitor.log"

# Fonction de test
test_service() {
    local name="$1"
    local host="$2"
    local port="$3"

    if timeout 5 nc -z "$host" "$port" 2>/dev/null; then
        echo "[$(date '+%H:%M:%S')] ✓ $name ($host:$port) - OK"
        return 0
    else
        echo "[$(date '+%H:%M:%S')] ✗ $name ($host:$port) - FAILED"
        return 1
    fi
}

# Test de tous les services
echo "=== SURVEILLANCE RÉSEAU - $(date) ===" | tee -a "$LOG_FILE"

for service_name in "${!SERVICES[@]}"; do
    IFS=' ' read -r host port <<< "${SERVICES[$service_name]}"
    test_service "$service_name" "$host" "$port" | tee -a "$LOG_FILE"
done
```

## 4. Traitement de JSON avec jq

### Installation et concepts de base

```bash
# Installation (Ubuntu/Debian)
sudo apt-get install jq

# Installation (CentOS/RHEL)
sudo yum install jq
```

### Utilisation basique de jq

#### Formatage et navigation
```bash
#!/bin/bash

# JSON d'exemple
JSON='{"name":"Jean","age":30,"city":"Paris","skills":["bash","python","javascript"]}'

# Formatter le JSON
echo "$JSON" | jq '.'

# Extraire une valeur
echo "$JSON" | jq '.name'

# Extraire plusieurs valeurs
echo "$JSON" | jq '.name, .age'

# Naviguer dans les tableaux
echo "$JSON" | jq '.skills[0]'

# Obtenir la longueur d'un tableau
echo "$JSON" | jq '.skills | length'
```

#### Filtrage et transformation
```bash
#!/bin/bash

# JSON plus complexe
USERS='[
  {"name":"Jean","age":30,"city":"Paris","active":true},
  {"name":"Marie","age":25,"city":"Lyon","active":false},
  {"name":"Pierre","age":35,"city":"Paris","active":true}
]'

# Filtrer les utilisateurs actifs
echo "$USERS" | jq '.[] | select(.active == true)'

# Extraire seulement les noms
echo "$USERS" | jq '.[].name'

# Filtrer par ville
echo "$USERS" | jq '.[] | select(.city == "Paris")'

# Créer un nouvel objet
echo "$USERS" | jq '.[] | {nom: .name, ville: .city}'
```

### Exemples pratiques avec APIs

#### Script de météo
```bash
#!/bin/bash

# Configuration
API_KEY="votre_cle_api"
CITY="Paris"
API_URL="http://api.openweathermap.org/data/2.5/weather"

# Fonction pour obtenir la météo
get_weather() {
    local city="$1"
    local response

    response=$(curl -s "$API_URL?q=$city&appid=$API_KEY&units=metric")

    if [ $? -eq 0 ]; then
        # Extraire les informations avec jq
        local temp=$(echo "$response" | jq -r '.main.temp')
        local description=$(echo "$response" | jq -r '.weather[0].description')
        local humidity=$(echo "$response" | jq -r '.main.humidity')
        local city_name=$(echo "$response" | jq -r '.name')

        echo "=== MÉTÉO POUR $city_name ==="
        echo "Température: ${temp}°C"
        echo "Description: $description"
        echo "Humidité: ${humidity}%"
    else
        echo "Erreur lors de la récupération des données météo"
    fi
}

# Utilisation
get_weather "$CITY"
```

#### Script de monitoring GitHub
```bash
#!/bin/bash

# Configuration
GITHUB_USER="votre_utilisateur"
GITHUB_REPO="votre_depot"
API_URL="https://api.github.com/repos/$GITHUB_USER/$GITHUB_REPO"

# Fonction pour obtenir les informations du dépôt
get_repo_info() {
    local response

    response=$(curl -s "$API_URL")

    if [ $? -eq 0 ]; then
        echo "=== INFORMATIONS DU DÉPÔT ==="
        echo "Nom: $(echo "$response" | jq -r '.full_name')"
        echo "Description: $(echo "$response" | jq -r '.description')"
        echo "Langage principal: $(echo "$response" | jq -r '.language')"
        echo "Étoiles: $(echo "$response" | jq -r '.stargazers_count')"
        echo "Forks: $(echo "$response" | jq -r '.forks_count')"
        echo "Issues ouvertes: $(echo "$response" | jq -r '.open_issues_count')"
        echo "Dernière mise à jour: $(echo "$response" | jq -r '.updated_at')"
    else
        echo "Erreur lors de la récupération des informations"
    fi
}

# Fonction pour obtenir les derniers commits
get_recent_commits() {
    local response

    response=$(curl -s "$API_URL/commits?per_page=5")

    if [ $? -eq 0 ]; then
        echo -e "\n=== DERNIERS COMMITS ==="
        echo "$response" | jq -r '.[] | "\(.commit.author.date) - \(.commit.message) (\(.author.login // "Unknown"))"'
    fi
}

# Exécution
get_repo_info
get_recent_commits
```

### Traitement de fichiers JSON complexes

#### Analyse de logs JSON
```bash
#!/bin/bash

# Supposons un fichier de logs au format JSON
LOG_FILE="app_logs.json"

# Exemple de contenu :
cat > "$LOG_FILE" << 'EOF'
{"timestamp":"2024-01-15T10:30:00Z","level":"INFO","message":"User login","user_id":123}
{"timestamp":"2024-01-15T10:31:00Z","level":"ERROR","message":"Database connection failed","error_code":500}
{"timestamp":"2024-01-15T10:32:00Z","level":"INFO","message":"User logout","user_id":123}
{"timestamp":"2024-01-15T10:33:00Z","level":"WARN","message":"High memory usage","memory_percent":85}
EOF

echo "=== ANALYSE DES LOGS ==="

# Compter les logs par niveau
echo "Répartition par niveau :"
jq -r '.level' "$LOG_FILE" | sort | uniq -c

# Extraire seulement les erreurs
echo -e "\nMessages d'erreur :"
jq 'select(.level == "ERROR")' "$LOG_FILE"

# Analyse des utilisateurs
echo -e "\nActivité des utilisateurs :"
jq -r 'select(.user_id) | "\(.timestamp) - User \(.user_id): \(.message)"' "$LOG_FILE"

# Statistiques avancées
echo -e "\nStatistiques :"
echo "Total des logs: $(wc -l < "$LOG_FILE")"
echo "Nombre d'erreurs: $(jq 'select(.level == "ERROR")' "$LOG_FILE" | wc -l)"
echo "Utilisateurs uniques: $(jq -r '.user_id // empty' "$LOG_FILE" | sort -u | wc -l)"
```

## Exemple complet : Script de monitoring système avec APIs

```bash
#!/bin/bash

# Script de monitoring système complet avec APIs et JSON
# Collecte des métriques système et envoi vers une API de monitoring

# Configuration
MONITORING_API="https://api.monitoring.example.com/metrics"
API_KEY="votre_cle_api"
HOSTNAME=$(hostname)
LOG_FILE="/var/log/system_monitor.log"

# Fonction de log
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Fonction pour collecter les métriques système
collect_metrics() {
    local timestamp=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

    # Utilisation CPU
    local cpu_usage=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}')

    # Utilisation mémoire
    local memory_info=$(free | grep Mem)
    local total_mem=$(echo "$memory_info" | awk '{print $2}')
    local used_mem=$(echo "$memory_info" | awk '{print $3}')
    local memory_percent=$(awk "BEGIN {printf \"%.2f\", ($used_mem/$total_mem)*100}")

    # Utilisation disque
    local disk_usage=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')

    # Charge système
    local load_avg=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | sed 's/,//')

    # Créer l'objet JSON avec jq
    local metrics=$(jq -n \
        --arg timestamp "$timestamp" \
        --arg hostname "$HOSTNAME" \
        --argjson cpu "$cpu_usage" \
        --argjson memory "$memory_percent" \
        --argjson disk "$disk_usage" \
        --argjson load "$load_avg" \
        '{
            timestamp: $timestamp,
            hostname: $hostname,
            metrics: {
                cpu_usage: $cpu,
                memory_usage: $memory,
                disk_usage: $disk,
                load_average: $load
            }
        }')

    echo "$metrics"
}

# Fonction pour envoyer les métriques
send_metrics() {
    local metrics="$1"
    local response
    local http_code

    response=$(curl -s -w "%{http_code}" \
        -X POST \
        -H "Content-Type: application/json" \
        -H "Authorization: Bearer $API_KEY" \
        -d "$metrics" \
        "$MONITORING_API")

    http_code="${response: -3}"

    if [ "$http_code" = "200" ] || [ "$http_code" = "201" ]; then
        log "Métriques envoyées avec succès"
        return 0
    else
        log "Erreur lors de l'envoi des métriques (HTTP $http_code)"
        return 1
    fi
}

# Fonction pour analyser les métriques localement
analyze_metrics() {
    local metrics="$1"

    # Extraire les valeurs avec jq
    local cpu=$(echo "$metrics" | jq -r '.metrics.cpu_usage')
    local memory=$(echo "$metrics" | jq -r '.metrics.memory_usage')
    local disk=$(echo "$metrics" | jq -r '.metrics.disk_usage')

    log "Métriques actuelles - CPU: ${cpu}%, Mémoire: ${memory}%, Disque: ${disk}%"

    # Alertes
    if (( $(echo "$cpu > 90" | bc -l) )); then
        log "ALERTE: Utilisation CPU élevée ($cpu%)"
    fi

    if (( $(echo "$memory > 85" | bc -l) )); then
        log "ALERTE: Utilisation mémoire élevée ($memory%)"
    fi

    if [ "$disk" -gt 90 ]; then
        log "ALERTE: Espace disque faible ($disk%)"
    fi
}

# Fonction pour sauvegarder les métriques localement
save_metrics_locally() {
    local metrics="$1"
    local metrics_file="/var/log/metrics_$(date +%Y%m%d).json"

    echo "$metrics" >> "$metrics_file"
    log "Métriques sauvegardées dans $metrics_file"
}

# Fonction principale
main() {
    log "Démarrage de la collecte de métriques"

    # Collecter les métriques
    local metrics=$(collect_metrics)

    # Analyser localement
    analyze_metrics "$metrics"

    # Sauvegarder localement
    save_metrics_locally "$metrics"

    # Envoyer vers l'API
    if ! send_metrics "$metrics"; then
        log "Impossible d'envoyer vers l'API, métriques sauvegardées localement"
    fi

    log "Collecte terminée"
}

# Vérification des dépendances
for cmd in jq curl bc; do
    if ! command -v "$cmd" &> /dev/null; then
        echo "Erreur: $cmd n'est pas installé"
        echo "Installation requise pour continuer"
        exit 1
    fi
done

# Exécution du script principal
main "$@"
```

## Scripts d'automatisation avec les outils CLI

### Script de déploiement automatisé

```bash
#!/bin/bash

# Script de déploiement utilisant curl, jq et find
# Déploie une application web et notifie les services externes

set -euo pipefail

# Configuration
APP_NAME="mon-app"
VERSION="1.0.0"
DEPLOY_DIR="/var/www/$APP_NAME"
BACKUP_DIR="/backup/deployments"
WEBHOOK_URL="https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK"
HEALTH_CHECK_URL="https://app.example.com/health"

# Couleurs pour l'affichage
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

# Fonction d'affichage avec couleurs
print_status() {
    local color=$1
    shift
    echo -e "${color}[$(date '+%H:%M:%S')] $@${NC}"
}

# Fonction de notification Slack
notify_slack() {
    local message="$1"
    local color="$2"

    local payload=$(jq -n \
        --arg text "$message" \
        --arg color "$color" \
        '{
            text: $text,
            attachments: [
                {
                    color: $color,
                    fields: [
                        {
                            title: "Application",
                            value: "'$APP_NAME'",
                            short: true
                        },
                        {
                            title: "Version",
                            value: "'$VERSION'",
                            short: true
                        },
                        {
                            title: "Serveur",
                            value: "'$(hostname)'",
                            short: true
                        }
                    ]
                }
            ]
        }')

    curl -s -X POST \
        -H "Content-Type: application/json" \
        -d "$payload" \
        "$WEBHOOK_URL" > /dev/null
}

# Fonction de sauvegarde
backup_current_version() {
    print_status $BLUE "Création de la sauvegarde..."

    if [ -d "$DEPLOY_DIR" ]; then
        local backup_name="backup_${APP_NAME}_$(date +%Y%m%d_%H%M%S)"
        local backup_path="$BACKUP_DIR/$backup_name.tar.gz"

        mkdir -p "$BACKUP_DIR"
        tar -czf "$backup_path" -C "$(dirname "$DEPLOY_DIR")" "$(basename "$DEPLOY_DIR")"

        print_status $GREEN "Sauvegarde créée : $backup_path"
        echo "$backup_path"
    else
        print_status $YELLOW "Aucune version précédente à sauvegarder"
        echo ""
    fi
}

# Fonction de déploiement
deploy_application() {
    print_status $BLUE "Déploiement de l'application..."

    # Simuler le téléchargement et l'extraction
    local temp_dir="/tmp/deploy_${APP_NAME}_$$"
    mkdir -p "$temp_dir"

    # Ici vous téléchargeriez votre application
    print_status $BLUE "Téléchargement de la version $VERSION..."

    # Simulation avec des fichiers d'exemple
    cat > "$temp_dir/index.html" << EOF
<!DOCTYPE html>
<html>
<head><title>$APP_NAME v$VERSION</title></head>
<body><h1>$APP_NAME Version $VERSION</h1></body>
</html>
EOF

    cat > "$temp_dir/health.json" << EOF
{
    "status": "ok",
    "version": "$VERSION",
    "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF

    # Déployer les fichiers
    mkdir -p "$DEPLOY_DIR"
    cp -r "$temp_dir"/* "$DEPLOY_DIR"

    # Nettoyer le répertoire temporaire
    rm -rf "$temp_dir"

    # Définir les permissions appropriées
    find "$DEPLOY_DIR" -type f -exec chmod 644 {} \;
    find "$DEPLOY_DIR" -type d -exec chmod 755 {} \;

    print_status $GREEN "Application déployée avec succès"
}

# Fonction de vérification de santé
health_check() {
    print_status $BLUE "Vérification de l'état de l'application..."

    local max_attempts=5
    local attempt=1

    while [ $attempt -le $max_attempts ]; do
        print_status $YELLOW "Tentative $attempt/$max_attempts..."

        local response=$(curl -s -w "%{http_code}" --max-time 10 "$HEALTH_CHECK_URL" || echo "000")
        local http_code="${response: -3}"
        local body="${response%???}"

        if [ "$http_code" = "200" ]; then
            # Vérifier le contenu JSON si disponible
            if echo "$body" | jq . > /dev/null 2>&1; then
                local status=$(echo "$body" | jq -r '.status // "unknown"')
                local version=$(echo "$body" | jq -r '.version // "unknown"')

                if [ "$status" = "ok" ] && [ "$version" = "$VERSION" ]; then
                    print_status $GREEN "✓ Application en bonne santé (v$version)"
                    return 0
                else
                    print_status $YELLOW "✗ Application répond mais état incorrect"
                fi
            else
                print_status $GREEN "✓ Application répond (HTTP $http_code)"
                return 0
            fi
        else
            print_status $RED "✗ Application ne répond pas (HTTP $http_code)"
        fi

        sleep 10
        ((attempt++))
    done

    print_status $RED "✗ Échec de la vérification de santé"
    return 1
}

# Fonction de rollback
rollback() {
    local backup_file="$1"

    if [ -n "$backup_file" ] && [ -f "$backup_file" ]; then
        print_status $YELLOW "Rollback en cours..."

        # Supprimer la version défaillante
        rm -rf "$DEPLOY_DIR"

        # Restaurer la sauvegarde
        tar -xzf "$backup_file" -C "$(dirname "$DEPLOY_DIR")"

        print_status $GREEN "Rollback terminé"
        notify_slack "🔄 Rollback effectué pour $APP_NAME" "warning"
    else
        print_status $RED "Impossible de faire le rollback : sauvegarde introuvable"
    fi
}

# Fonction de nettoyage des anciennes sauvegardes
cleanup_old_backups() {
    print_status $BLUE "Nettoyage des anciennes sauvegardes..."

    # Garder seulement les 5 dernières sauvegardes
    find "$BACKUP_DIR" -name "backup_${APP_NAME}_*.tar.gz" -type f \
        | sort -r | tail -n +6 | xargs -r rm -f

    local remaining=$(find "$BACKUP_DIR" -name "backup_${APP_NAME}_*.tar.gz" | wc -l)
    print_status $GREEN "Nettoyage terminé. $remaining sauvegarde(s) conservée(s)"
}

# Fonction principale de déploiement
main() {
    print_status $BLUE "=== DÉBUT DU DÉPLOIEMENT ==="
    print_status $BLUE "Application: $APP_NAME"
    print_status $BLUE "Version: $VERSION"
    print_status $BLUE "Serveur: $(hostname)"

    # Notification de début
    notify_slack "🚀 Début du déploiement de $APP_NAME v$VERSION" "good"

    # Sauvegarde de la version actuelle
    local backup_file
    backup_file=$(backup_current_version)

    # Déploiement
    if deploy_application; then
        # Vérification de santé
        if health_check; then
            print_status $GREEN "=== DÉPLOIEMENT RÉUSSI ==="
            notify_slack "✅ Déploiement réussi de $APP_NAME v$VERSION" "good"

            # Nettoyage des anciennes sauvegardes
            cleanup_old_backups
        else
            print_status $RED "=== ÉCHEC DE LA VÉRIFICATION ==="
            rollback "$backup_file"
            notify_slack "❌ Échec du déploiement de $APP_NAME v$VERSION - Rollback effectué" "danger"
            exit 1
        fi
    else
        print_status $RED "=== ÉCHEC DU DÉPLOIEMENT ==="
        notify_slack "❌ Échec du déploiement de $APP_NAME v$VERSION" "danger"
        exit 1
    fi
}

# Vérification des prérequis
check_prerequisites() {
    local missing_tools=()

    for tool in curl jq tar; do
        if ! command -v "$tool" &> /dev/null; then
            missing_tools+=("$tool")
        fi
    done

    if [ ${#missing_tools[@]} -gt 0 ]; then
        print_status $RED "Outils manquants: ${missing_tools[*]}"
        print_status $RED "Veuillez installer ces outils avant de continuer"
        exit 1
    fi

    # Vérifier les répertoires
    if [ ! -w "$(dirname "$DEPLOY_DIR")" ]; then
        print_status $RED "Impossible d'écrire dans $(dirname "$DEPLOY_DIR")"
        exit 1
    fi
}

# Gestion des options de ligne de commande
show_help() {
    cat << EOF
Usage: $0 [OPTIONS]

Script de déploiement automatisé

OPTIONS:
    -v VERSION     Version à déployer (défaut: $VERSION)
    -a APP_NAME    Nom de l'application (défaut: $APP_NAME)
    -d DEPLOY_DIR  Répertoire de déploiement (défaut: $DEPLOY_DIR)
    -h             Afficher cette aide

EXEMPLES:
    $0                           # Déploiement avec les valeurs par défaut
    $0 -v 2.0.0 -a myapp         # Déploiement d'une version spécifique
    $0 -d /var/www/test          # Déploiement dans un répertoire spécifique

EOF
}

# Traitement des options
while getopts "v:a:d:h" option; do
    case $option in
        v) VERSION="$OPTARG" ;;
        a) APP_NAME="$OPTARG" ;;
        d) DEPLOY_DIR="$OPTARG" ;;
        h) show_help; exit 0 ;;
        *) echo "Option invalide. Utilisez -h pour l'aide."; exit 1 ;;
    esac
done

# Exécution
check_prerequisites
main
```

## Script d'analyse de performance

```bash
#!/bin/bash

# Script d'analyse de performance système et application
# Utilise curl pour tester les performances web et jq pour analyser les résultats

set -euo pipefail

# Configuration
REPORT_DIR="/tmp/performance_report_$(date +%Y%m%d_%H%M%S)"
CONFIG_FILE="performance_config.json"

# Créer le répertoire de rapport
mkdir -p "$REPORT_DIR"

# Configuration par défaut
create_default_config() {
    cat > "$CONFIG_FILE" << 'EOF'
{
    "websites": [
        {
            "name": "Site Principal",
            "url": "https://example.com",
            "timeout": 30,
            "expected_status": 200
        },
        {
            "name": "API REST",
            "url": "https://api.example.com/health",
            "timeout": 10,
            "expected_status": 200,
            "headers": {
                "Authorization": "Bearer token123"
            }
        }
    ],
    "thresholds": {
        "response_time_warning": 2.0,
        "response_time_critical": 5.0,
        "cpu_warning": 80,
        "memory_warning": 85,
        "disk_warning": 90
    }
}
EOF
    echo "Configuration par défaut créée dans $CONFIG_FILE"
}

# Fonction de test de performance web
test_website_performance() {
    local site_config="$1"
    local name=$(echo "$site_config" | jq -r '.name')
    local url=$(echo "$site_config" | jq -r '.url')
    local timeout=$(echo "$site_config" | jq -r '.timeout')
    local expected_status=$(echo "$site_config" | jq -r '.expected_status')

    echo "Test de performance pour : $name"

    # Extraire les en-têtes s'ils existent
    local headers_args=""
    if echo "$site_config" | jq -e '.headers' > /dev/null; then
        while IFS="=" read -r key value; do
            headers_args="$headers_args -H \"$key: $value\""
        done < <(echo "$site_config" | jq -r '.headers | to_entries[] | "\(.key)=\(.value)"')
    fi

    # Effectuer le test avec curl et mesurer les temps
    local curl_format='{
        "time_namelookup": %{time_namelookup},
        "time_connect": %{time_connect},
        "time_appconnect": %{time_appconnect},
        "time_pretransfer": %{time_pretransfer},
        "time_redirect": %{time_redirect},
        "time_starttransfer": %{time_starttransfer},
        "time_total": %{time_total},
        "speed_download": %{speed_download},
        "speed_upload": %{speed_upload},
        "size_download": %{size_download},
        "size_upload": %{size_upload},
        "http_code": %{http_code}
    }'

    local result=$(eval "curl -s -w '$curl_format' --max-time $timeout $headers_args -o /dev/null '$url'")

    # Ajouter des métadonnées
    local full_result=$(echo "$result" | jq \
        --arg name "$name" \
        --arg url "$url" \
        --arg timestamp "$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
        --argjson expected_status "$expected_status" \
        '. + {
            name: $name,
            url: $url,
            timestamp: $timestamp,
            expected_status: $expected_status,
            status: (if .http_code == $expected_status then "OK" else "ERROR" end)
        }')

    echo "$full_result" > "$REPORT_DIR/web_${name// /_}.json"
    echo "$full_result"
}

# Fonction de collecte des métriques système
collect_system_metrics() {
    echo "Collecte des métriques système..."

    # CPU
    local cpu_usage=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}')

    # Mémoire
    local memory_info=$(free -b | grep Mem)
    local total_mem=$(echo "$memory_info" | awk '{print $2}')
    local used_mem=$(echo "$memory_info" | awk '{print $3}')
    local available_mem=$(echo "$memory_info" | awk '{print $7}')
    local memory_percent=$(awk "BEGIN {printf \"%.2f\", ($used_mem/$total_mem)*100}")

    # Disque
    local disk_info=$(df / | tail -1)
    local disk_total=$(echo "$disk_info" | awk '{print $2}')
    local disk_used=$(echo "$disk_info" | awk '{print $3}')
    local disk_percent=$(echo "$disk_info" | awk '{print $5}' | sed 's/%//')

    # Charge système
    local load_avg_1=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | sed 's/,//')
    local load_avg_5=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $2}' | sed 's/,//')
    local load_avg_15=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $3}' | sed 's/,//')

    # Processus
    local process_count=$(ps aux | wc -l)
    local zombie_count=$(ps aux | awk '$8 ~ /^Z/ { count++ } END { print count+0 }')

    # Créer l'objet JSON
    local system_metrics=$(jq -n \
        --arg timestamp "$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
        --arg hostname "$(hostname)" \
        --argjson cpu "$cpu_usage" \
        --argjson memory_percent "$memory_percent" \
        --argjson memory_total "$total_mem" \
        --argjson memory_used "$used_mem" \
        --argjson memory_available "$available_mem" \
        --argjson disk_percent "$disk_percent" \
        --argjson disk_total "$disk_total" \
        --argjson disk_used "$disk_used" \
        --argjson load_1m "$load_avg_1" \
        --argjson load_5m "$load_avg_5" \
        --argjson load_15m "$load_avg_15" \
        --argjson process_count "$process_count" \
        --argjson zombie_count "$zombie_count" \
        '{
            timestamp: $timestamp,
            hostname: $hostname,
            cpu: {
                usage_percent: $cpu
            },
            memory: {
                usage_percent: $memory_percent,
                total_bytes: $memory_total,
                used_bytes: $memory_used,
                available_bytes: $memory_available
            },
            disk: {
                usage_percent: $disk_percent,
                total_kb: $disk_total,
                used_kb: $disk_used
            },
            load_average: {
                "1m": $load_1m,
                "5m": $load_5m,
                "15m": $load_15m
            },
            processes: {
                total: $process_count,
                zombie: $zombie_count
            }
        }')

    echo "$system_metrics" > "$REPORT_DIR/system_metrics.json"
    echo "$system_metrics"
}

# Fonction d'analyse des résultats
analyze_results() {
    echo "=== ANALYSE DES RÉSULTATS ==="

    # Charger les seuils
    local thresholds=$(jq '.thresholds' "$CONFIG_FILE")
    local response_warning=$(echo "$thresholds" | jq -r '.response_time_warning')
    local response_critical=$(echo "$thresholds" | jq -r '.response_time_critical')
    local cpu_warning=$(echo "$thresholds" | jq -r '.cpu_warning')
    local memory_warning=$(echo "$thresholds" | jq -r '.memory_warning')
    local disk_warning=$(echo "$thresholds" | jq -r '.disk_warning')

    # Analyser les métriques système
    if [ -f "$REPORT_DIR/system_metrics.json" ]; then
        local system_data=$(cat "$REPORT_DIR/system_metrics.json")

        echo "Métriques système :"
        echo "  CPU: $(echo "$system_data" | jq -r '.cpu.usage_percent')%"
        echo "  Mémoire: $(echo "$system_data" | jq -r '.memory.usage_percent')%"
        echo "  Disque: $(echo "$system_data" | jq -r '.disk.usage_percent')%"
        echo "  Charge: $(echo "$system_data" | jq -r '.load_average."1m"')"

        # Alertes système
        local cpu_current=$(echo "$system_data" | jq -r '.cpu.usage_percent')
        local memory_current=$(echo "$system_data" | jq -r '.memory.usage_percent')
        local disk_current=$(echo "$system_data" | jq -r '.disk.usage_percent')

        if (( $(echo "$cpu_current > $cpu_warning" | bc -l) )); then
            echo "  ⚠️  ALERTE: CPU élevé ($cpu_current%)"
        fi

        if (( $(echo "$memory_current > $memory_warning" | bc -l) )); then
            echo "  ⚠️  ALERTE: Mémoire élevée ($memory_current%)"
        fi

        if (( $(echo "$disk_current > $disk_warning" | bc -l) )); then
            echo "  ⚠️  ALERTE: Disque plein ($disk_current%)"
        fi
    fi

    echo ""

    # Analyser les tests web
    for web_file in "$REPORT_DIR"/web_*.json; do
        if [ -f "$web_file" ]; then
            local web_data=$(cat "$web_file")
            local name=$(echo "$web_data" | jq -r '.name')
            local status=$(echo "$web_data" | jq -r '.status')
            local response_time=$(echo "$web_data" | jq -r '.time_total')
            local http_code=$(echo "$web_data" | jq -r '.http_code')

            echo "Site web: $name"
            echo "  Status: $status (HTTP $http_code)"
            echo "  Temps de réponse: ${response_time}s"

            # Analyse des temps de réponse
            if (( $(echo "$response_time > $response_critical" | bc -l) )); then
                echo "  🚨 CRITIQUE: Temps de réponse très élevé"
            elif (( $(echo "$response_time > $response_warning" | bc -l) )); then
                echo "  ⚠️  ATTENTION: Temps de réponse élevé"
            else
                echo "  ✅ OK: Temps de réponse acceptable"
            fi

            echo ""
        fi
    done
}

# Fonction de génération de rapport HTML
generate_html_report() {
    local html_file="$REPORT_DIR/rapport.html"

    cat > "$html_file" << 'EOF'
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Rapport de Performance</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .header { background: #f4f4f4; padding: 20px; border-radius: 5px; }
        .metric { margin: 10px 0; padding: 10px; border-left: 4px solid #007cba; }
        .warning { border-left-color: #ff9800; }
        .critical { border-left-color: #f44336; }
        .ok { border-left-color: #4caf50; }
        table { width: 100%; border-collapse: collapse; margin: 20px 0; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #f2f2f2; }
    </style>
</head>
<body>
    <div class="header">
        <h1>Rapport de Performance</h1>
        <p>Généré le: <span id="timestamp"></span></p>
        <p>Serveur: <span id="hostname"></span></p>
    </div>

    <h2>Métriques Système</h2>
    <div id="system-metrics"></div>

    <h2>Tests Web</h2>
    <div id="web-tests"></div>

    <script>
        // Le JavaScript sera injecté ici pour charger les données JSON
    </script>
</body>
</html>
EOF

    echo "Rapport HTML généré : $html_file"
}

# Fonction principale
main() {
    echo "=== ANALYSE DE PERFORMANCE ==="
    echo "Répertoire de rapport: $REPORT_DIR"

    # Vérifier le fichier de configuration
    if [ ! -f "$CONFIG_FILE" ]; then
        echo "Fichier de configuration non trouvé, création..."
        create_default_config
    fi

    # Collecter les métriques système
    collect_system_metrics > /dev/null

    # Tester les sites web
    echo ""
    echo "Tests de performance web :"
    jq -c '.websites[]' "$CONFIG_FILE" | while read -r site; do
        test_website_performance "$site" > /dev/null
    done

    echo ""

    # Analyser les résultats
    analyze_results

    # Générer le rapport HTML
    generate_html_report

    echo ""
    echo "=== ANALYSE TERMINÉE ==="
    echo "Fichiers de rapport disponibles dans: $REPORT_DIR"
}

# Vérification des dépendances
for cmd in jq curl bc; do
    if ! command -v "$cmd" &> /dev/null; then
        echo "Erreur: $cmd n'est pas installé"
        exit 1
    fi
done

# Exécution
main "$@"
```

## Bonnes pratiques pour l'utilisation des outils CLI

### 1. Gestion des erreurs
```bash
# Toujours vérifier les codes de retour
if ! curl -s "$URL" > /dev/null; then
    echo "Erreur: Impossible de contacter $URL"
    exit 1
fi

# Utiliser des timeouts
curl --max-time 30 "$URL"
```

### 2. Validation des données JSON
```bash
# Valider le JSON avant traitement
if echo "$json_data" | jq . > /dev/null 2>&1; then
    # Traitement du JSON valide
    echo "$json_data" | jq '.field'
else
    echo "Erreur: JSON invalide"
fi
```

### 3. Optimisation des performances
```bash
# Réutiliser les connexions curl
curl -s --keepalive-time 2 "$URL1" "$URL2" "$URL3"

# Paralléliser avec xargs
find . -name "*.txt" | xargs -P 4 -I {} process_file {}
```

### 4. Sécurité
```bash
# Éviter l'injection de commandes
safe_filename=$(printf '%q' "$filename")

# Utiliser des variables d'environnement pour les secrets
API_KEY="${API_KEY:-}"
if [ -z "$API_KEY" ]; then
    echo "Erreur: API_KEY non définie"
    exit 1
fi
```

## Résumé

Les outils de ligne de commande essentiels transforment vos scripts Bash en outils puissants :

- **find et xargs** : Recherche et traitement de fichiers en masse
- **cut, paste, join** : Manipulation de données tabulaires
- **curl, wget, netcat** : Interactions réseau et téléchargements
- **jq** : Traitement de données JSON modernes

Maîtriser ces outils vous permet de créer des scripts d'automatisation, de monitoring et d'analyse de données de niveau professionnel.

⏭️
