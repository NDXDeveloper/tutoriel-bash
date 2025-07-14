🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Tutoriel Bash - Chapitre 10 : Expressions régulières approfondies

## Syntaxe des regex (POSIX ERE/BRE, PCRE)

### Qu'est-ce qu'une expression régulière ?

Une expression régulière (regex) est un **langage de patterns** pour décrire du texte. C'est comme avoir un **code secret** pour dire "je cherche tous les mots qui commencent par une majuscule et finissent par 'ing'".

**Analogie :** Si chercher du texte avec des mots-clés simples c'est comme chercher "toutes les voitures rouges", les regex c'est comme dire "toutes les voitures de 4 lettres, commençant par 'F' et avec des chiffres à la fin".

### Les différents types de regex

```bash
#!/bin/bash

echo "=== Types d'expressions régulières ==="

# Créer un fichier de test
cat > test_regex.txt << 'EOF'
Bonjour123
hello world
email@domain.com
MAJUSCULE
123-456-789
test_file.txt
(555) 123-4567
http://example.com
EOF

echo "Fichier de test :"
cat test_regex.txt
echo ""

echo "=== 1. BRE (Basic Regular Expressions) ==="
echo "Utilisées par défaut dans grep et sed"

echo "Motif simple :"
grep "hello" test_regex.txt

echo ""
echo "Début de ligne (^) :"
grep "^[A-Z]" test_regex.txt

echo ""
echo "Fin de ligne ($) :"
grep "com$" test_regex.txt

echo ""
echo "=== 2. ERE (Extended Regular Expressions) ==="
echo "Utilisées avec grep -E, egrep"

echo "Alternative (|) :"
grep -E "hello|MAJUSCULE" test_regex.txt

echo ""
echo "Une ou plusieurs (+) :"
grep -E "[0-9]+" test_regex.txt

echo ""
echo "Zéro ou une (?) :"
grep -E "https?" test_regex.txt

echo ""
echo "Groupement () :"
grep -E "(test|hello)_?" test_regex.txt
```

### Métacaractères fondamentaux

```bash
#!/bin/bash

echo "=== Métacaractères fondamentaux ==="

# Créer des données variées pour les tests
cat > donnees_test.txt << 'EOF'
abc123
ABC789
test@example.com
user.name@domain.org
192.168.1.1
255.255.255.0
2025-07-13
13/07/2025
+33 6 12 34 56 78
06.12.34.56.78
file.txt
document.pdf
image.jpg
script.sh
EOF

echo "Données de test :"
cat donnees_test.txt
echo ""

echo "=== Métacaractères de base ==="

echo "1. Point (.) - n'importe quel caractère :"
echo "Pattern 'te.t' :"
grep "te.t" donnees_test.txt

echo ""
echo "2. Astérisque (*) - zéro ou plusieurs :"
echo "Pattern 'AB.*' :"
grep "AB.*" donnees_test.txt

echo ""
echo "3. Classes de caractères [...]:"
echo "Lignes avec des chiffres [0-9] :"
grep "[0-9]" donnees_test.txt

echo ""
echo "4. Classes négatives [^...] :"
echo "Lignes sans chiffres :"
grep "^[^0-9]*$" donnees_test.txt

echo ""
echo "5. Anchors ^ et $ :"
echo "Commence par un chiffre :"
grep "^[0-9]" donnees_test.txt

echo ""
echo "Se termine par .txt :"
grep "\\.txt$" donnees_test.txt

echo ""
echo "=== Quantificateurs ERE ==="

echo "6. Plus (+) - un ou plusieurs :"
echo "Un ou plusieurs chiffres :"
grep -E "[0-9]+" donnees_test.txt

echo ""
echo "7. Question (?) - zéro ou un :"
echo "http avec s optionnel :"
echo "https://example.com
http://test.org" | grep -E "https?"

echo ""
echo "8. Accolades {n,m} - nombre spécifique :"
echo "Exactement 3 chiffres consécutifs :"
grep -E "[0-9]{3}" donnees_test.txt

echo ""
echo "Entre 2 et 4 lettres :"
grep -E "^[a-z]{2,4}$" donnees_test.txt
```

### Classes de caractères prédéfinies

```bash
#!/bin/bash

echo "=== Classes de caractères prédéfinies ==="

# Créer un fichier avec différents types de caractères
cat > caracteres_mixtes.txt << 'EOF'
Mot123
MAJUSCULE
minuscule
Espace	Tabulation
Ligne avec des espaces
ÉàÇôñ
user@domain.com
$pecial!Ch@rs
EOF

echo "Fichier avec caractères mixtes :"
cat caracteres_mixtes.txt
echo ""

echo "=== Classes POSIX ==="

echo "1. [[:alpha:]] - lettres uniquement :"
grep -E "^[[:alpha:]]+$" caracteres_mixtes.txt

echo ""
echo "2. [[:digit:]] - chiffres uniquement :"
grep -E "[[:digit:]]" caracteres_mixtes.txt

echo ""
echo "3. [[:alnum:]] - lettres et chiffres :"
grep -E "^[[:alnum:]]+$" caracteres_mixtes.txt

echo ""
echo "4. [[:space:]] - espaces blancs :"
echo "Lignes avec espaces multiples :"
grep -E "[[:space:]]{2,}" caracteres_mixtes.txt

echo ""
echo "5. [[:upper:]] - majuscules :"
grep -E "^[[:upper:]]+$" caracteres_mixtes.txt

echo ""
echo "6. [[:lower:]] - minuscules :"
grep -E "^[[:lower:]]+$" caracteres_mixtes.txt

echo ""
echo "7. [[:punct:]] - ponctuation :"
grep -E "[[:punct:]]" caracteres_mixtes.txt

echo ""
echo "=== Équivalents courts (style Perl) ==="
echo "Note: Disponibles avec grep -P sur certains systèmes"

echo "\\d = [[:digit:]] :"
echo "\\w = [[:alnum:]_] :"
echo "\\s = [[:space:]] :"

# Test si grep -P est disponible
if grep -P "test" <<< "test" >/dev/null 2>&1; then
    echo "Support PCRE détecté"
    echo "Test avec \\d :"
    echo "test123" | grep -P "\\d+"
else
    echo "Support PCRE non disponible sur ce système"
    echo "Utilisation des classes POSIX recommandée"
fi
```

### Groupement et capture

```bash
#!/bin/bash

echo "=== Groupement et capture ==="

# Créer des données structurées
cat > donnees_structurees.txt << 'EOF'
Jean Dupont (25 ans)
Marie Martin (30 ans)
Pierre Bernard (28 ans)
Sophie Rousseau (35 ans)
2025-01-15 10:30:25
2025-02-20 14:45:12
contact@example.com
support@company.org
EOF

echo "Données structurées :"
cat donnees_structurees.txt
echo ""

echo "=== Groupement simple ==="

echo "1. Groupes avec parenthèses :"
echo "Noms avec âge :"
grep -E "([A-Z][a-z]+) ([A-Z][a-z]+) \\(([0-9]+) ans\\)" donnees_structurees.txt

echo ""
echo "2. Alternative dans un groupe :"
echo "Emails .com ou .org :"
grep -E "[a-z]+@[a-z]+\\.(com|org)" donnees_structurees.txt

echo ""
echo "3. Quantificateurs sur des groupes :"
echo "Mots répétés :"
echo "test test test" | grep -E "(test ){2,}"

echo ""
echo "=== Capture avec sed ==="

echo "4. Extraction de parties avec sed :"
echo "Extraire seulement les prénoms :"
sed -E 's/([A-Z][a-z]+) [A-Z][a-z]+ \\([0-9]+ ans\\)/\\1/' donnees_structurees.txt | grep -E "^[A-Z]"

echo ""
echo "5. Reformatage de dates :"
echo "DD-MM-YYYY vers YYYY/MM/DD :"
echo "15-07-2025" | sed -E 's/([0-9]{2})-([0-9]{2})-([0-9]{4})/\\3\\/\\2\\/\\1/'

echo ""
echo "6. Extraction d'informations d'URL :"
echo "https://www.example.com:8080/path/to/page" | \
sed -E 's|https?://([^:]+):([0-9]+)(.+)|Serveur: \\1, Port: \\2, Chemin: \\3|'
```

### Lookahead et lookbehind (avancé)

```bash
#!/bin/bash

echo "=== Lookahead et Lookbehind (concepts avancés) ==="

echo "Note: Ces fonctionnalités avancées ne sont pas disponibles"
echo "dans tous les outils POSIX, mais le concept est important."
echo ""

# Simulation avec des techniques alternatives
cat > texte_complexe.txt << 'EOF'
password123
admin_password
secure_pass
testpassword
password_old
my_password_new
EOF

echo "Texte pour démonstration :"
cat texte_complexe.txt
echo ""

echo "=== Simulation de lookahead positif ==="
echo "Chercher 'password' suivi de chiffres (simulation) :"

# Au lieu de password(?=\\d+), on utilise :
grep -E "password[0-9]" texte_complexe.txt

echo ""
echo "=== Simulation de lookbehind positif ==="
echo "Chercher 'password' précédé d'un underscore (simulation) :"

# Au lieu de (?<=_)password, on utilise :
grep -E "_password" texte_complexe.txt

echo ""
echo "=== Alternative POSIX pour des besoins complexes ==="
echo "Utiliser plusieurs passes avec grep :"

# Trouver les lignes avec 'password' ET des chiffres
grep "password" texte_complexe.txt | grep "[0-9]"

echo ""
echo "Ou utiliser awk pour des conditions complexes :"
awk '/password/ && /[0-9]/ {print "Trouvé:", $0}' texte_complexe.txt
```

### Comparaison des syntaxes

```bash
#!/bin/bash

echo "=== Comparaison des syntaxes regex ==="

# Créer un fichier de test unifié
cat > test_syntaxes.txt << 'EOF'
test123
TEST456
Test_789
test-abc
test.def
test@example
EOF

echo "Données de test :"
cat test_syntaxes.txt
echo ""

echo "=== BRE vs ERE vs PCRE ==="

echo "1. Groupement :"
echo "BRE (grep basique) - échapper les parenthèses :"
grep "\\(test\\)\\|\\(TEST\\)" test_syntaxes.txt

echo ""
echo "ERE (grep -E) - parenthèses directes :"
grep -E "(test)|(TEST)" test_syntaxes.txt

echo ""
echo "2. Quantificateurs :"
echo "BRE - un ou plusieurs (simulé avec .*) :"
grep "test.*[0-9]" test_syntaxes.txt

echo ""
echo "ERE - un ou plusieurs (natif) :"
grep -E "test.+[0-9]" test_syntaxes.txt

echo ""
echo "3. Alternative :"
echo "BRE - pas d'alternative native, utiliser plusieurs grep :"
grep "test" test_syntaxes.txt | grep -E "(123|456)"

echo ""
echo "ERE - alternative avec | :"
grep -E "test(123|456)" test_syntaxes.txt

echo ""
echo "=== Recommandations ==="
echo "• Utilisez ERE (grep -E) pour la plupart des cas"
echo "• BRE pour la compatibilité maximale"
echo "• PCRE (grep -P) pour des besoins très avancés"
echo "• awk pour des traitements complexes"

# Nettoyer les fichiers de test
rm -f test_regex.txt donnees_test.txt caracteres_mixtes.txt
rm -f donnees_structurees.txt texte_complexe.txt test_syntaxes.txt
```

## Utilisation avancée avec grep, sed, awk

### Patterns complexes avec grep

```bash
#!/bin/bash

echo "=== Patterns complexes avec grep ==="

# Créer un fichier de logs complexe
cat > logs_complexes.txt << 'EOF'
2025-07-13 08:15:23 [INFO] User login: alice@company.com from 192.168.1.100
2025-07-13 08:15:24 [ERROR] Database connection failed: timeout after 30s
2025-07-13 08:15:25 [WARNING] High memory usage: 85% (threshold: 80%)
2025-07-13 08:15:26 [INFO] File uploaded: document.pdf (2.5MB) by bob@company.com
2025-07-13 08:15:27 [DEBUG] Cache hit ratio: 95.2%
2025-07-13 08:15:28 [ERROR] API call failed: HTTP 500 for /api/users/123
2025-07-13 08:15:29 [INFO] User logout: alice@company.com
2025-07-13 08:15:30 [WARNING] Disk space low: 15% remaining on /var
EOF

echo "Logs complexes :"
cat logs_complexes.txt
echo ""

echo "=== Patterns avancés ==="

echo "1. Lignes avec email ET adresse IP :"
grep -E ".*@.*\.com.*[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}" logs_complexes.txt

echo ""
echo "2. Erreurs avec codes HTTP :"
grep -E "\\[ERROR\\].*HTTP [0-9]+" logs_complexes.txt

echo ""
echo "3. Pourcentages supérieurs à 80% :"
grep -E "([89][0-9]|9[0-9]|100)%" logs_complexes.txt

echo ""
echo "4. Timestamps entre 08:15:25 et 08:15:28 :"
grep -E "08:15:2[5-8]" logs_complexes.txt

echo ""
echo "5. Fichiers uploadés avec taille :"
grep -E "File uploaded:.*\\([0-9.]+[MK]B\\)" logs_complexes.txt

echo ""
echo "6. Combinaison complexe - Erreurs OU warnings avec données numériques :"
grep -E "\\[(ERROR|WARNING)\\].*[0-9]+" logs_complexes.txt

echo ""
echo "=== Techniques d'optimisation grep ==="

echo "7. Utiliser -F pour des chaînes fixes (plus rapide) :"
echo "Recherche de chaîne fixe :"
grep -F "alice@company.com" logs_complexes.txt

echo ""
echo "8. Utiliser -w pour des mots entiers :"
echo "Le mot 'User' en tant que mot entier :"
grep -w "User" logs_complexes.txt

echo ""
echo "9. Combiner plusieurs patterns avec -e :"
grep -e "ERROR" -e "WARNING" logs_complexes.txt
```

### Transformations avancées avec sed

```bash
#!/bin/bash

echo "=== Transformations avancées avec sed ==="

# Créer un fichier de configuration
cat > config_original.txt << 'EOF'
# Configuration de production
server.host=localhost
server.port=8080
database.url=jdbc:mysql://localhost:3306/mydb
database.username=user
database.password=secret123
logging.level=DEBUG
cache.enabled=true
session.timeout=3600
max.connections=100
EOF

echo "Configuration originale :"
cat config_original.txt
echo ""

echo "=== Transformations complexes ==="

echo "1. Transformation environnement dev -> prod :"
sed -E \
    -e 's/localhost/prod-server.company.com/g' \
    -e 's/DEBUG/ERROR/g' \
    -e 's/timeout=3600/timeout=7200/g' \
    -e 's/max.connections=100/max.connections=500/g' \
    config_original.txt

echo ""
echo "2. Anonymisation des données sensibles :"
sed -E \
    -e 's/(password=)[^[:space:]]*/\\1***HIDDEN***/g' \
    -e 's/(username=)[^[:space:]]*/\\1***USER***/g' \
    config_original.txt

echo ""
echo "3. Conversion en variables d'environnement :"
sed -E \
    -e '/^#/d' \
    -e '/^$/d' \
    -e 's/\\./_/g' \
    -e 's/^([^=]+)=(.*)$/export \\U\\1\\E="\\2"/' \
    config_original.txt

echo ""
echo "4. Extraction et transformation de valeurs :"
echo "URLs de base de données extraites et reformatées :"
sed -nE 's/.*jdbc:([^:]+):\\/\\/([^:]+):([0-9]+)\\/(.+)/Type: \\1\\nServeur: \\2\\nPort: \\3\\nBase: \\4/p' config_original.txt

echo ""
echo "5. Ajout conditionnel de configuration :"
sed -E '/^cache.enabled=true/a\
cache.size=1024\
cache.ttl=300' config_original.txt

echo ""
echo "6. Reformatage complexe avec plusieurs substitutions :"
echo "Conversion en format JSON :"
sed -E \
    -e '/^#/d' \
    -e '/^$/d' \
    -e '1i\{' \
    -e 's/^([^=]+)=(.*)$/  "\\1": "\\2",/' \
    -e '$s/,$//' \
    -e '$a\}' \
    config_original.txt
```

### Traitement sophistiqué avec awk

```bash
#!/bin/bash

echo "=== Traitement sophistiqué avec awk ==="

# Créer un fichier de données de ventes
cat > ventes_detaillees.txt << 'EOF'
Date,Vendeur,Produit,Quantite,Prix_unitaire,Region,Type_client,Commission
2025-01-15,Alice,Ordinateur,2,800,Nord,Entreprise,8.5
2025-01-15,Bob,Souris,50,25,Sud,Particulier,5.0
2025-01-16,Alice,Clavier,30,45,Nord,Entreprise,7.5
2025-01-16,Charlie,Ecran,5,300,Est,Entreprise,10.0
2025-01-17,Bob,Ordinateur,1,800,Sud,Particulier,8.5
2025-01-17,Alice,Imprimante,3,150,Nord,Entreprise,9.0
2025-01-18,Charlie,Souris,20,25,Est,Particulier,5.0
2025-01-18,Bob,Clavier,15,45,Sud,Entreprise,7.5
EOF

echo "Données de ventes détaillées :"
cat ventes_detaillees.txt
echo ""

echo "=== Analyses complexes avec awk ==="

echo "1. Rapport complet par vendeur :"
awk -F',' 'NR > 1 {
    # Calculs par vendeur
    vendeur = $2
    ca = $4 * $5
    commission = ca * ($8 / 100)

    ca_vendeur[vendeur] += ca
    commission_vendeur[vendeur] += commission
    ventes_vendeur[vendeur]++

    # Suivi des produits par vendeur
    produits_vendeur[vendeur][$3] += $4

    # Totaux généraux
    ca_total += ca
    commission_totale += commission
} END {
    print "=== RAPPORT PAR VENDEUR ==="
    for (v in ca_vendeur) {
        print "Vendeur:", v
        print "  CA:", ca_vendeur[v] "€"
        print "  Commission:", int(commission_vendeur[v]) "€"
        print "  Nb ventes:", ventes_vendeur[v]
        print "  CA moyen/vente:", int(ca_vendeur[v]/ventes_vendeur[v]) "€"

        print "  Produits vendus:"
        for (p in produits_vendeur[v]) {
            print "    " p ":", produits_vendeur[v][p] "unités"
        }
        print ""
    }

    print "=== TOTAUX ==="
    print "CA total:", ca_total "€"
    print "Commission totale:", int(commission_totale) "€"
}' ventes_detaillees.txt

echo ""
echo "2. Analyse par région et type de client :"
awk -F',' 'NR > 1 {
    region = $6
    type_client = $7
    ca = $4 * $5

    ca_region[region] += ca
    ca_type[type_client] += ca
    ca_region_type[region][type_client] += ca

    clients_region[region]++
    clients_type[type_client]++
} END {
    print "=== ANALYSE GÉOGRAPHIQUE ET CLIENTS ==="
    print ""
    print "Par région:"
    for (r in ca_region) {
        print "  " r ":", ca_region[r] "€ (" clients_region[r] " ventes)"
    }

    print ""
    print "Par type de client:"
    for (t in ca_type) {
        print "  " t ":", ca_type[t] "€ (" clients_type[t] " ventes)"
    }

    print ""
    print "Croisement région/type:"
    for (r in ca_region_type) {
        for (t in ca_region_type[r]) {
            print "  " r " - " t ":", ca_region_type[r][t] "€"
        }
    }
}' ventes_detaillees.txt

echo ""
echo "3. Tendances temporelles :"
awk -F',' 'NR > 1 {
    split($1, date_parts, "-")
    jour = date_parts[3]
    ca = $4 * $5

    ca_jour[jour] += ca
    ventes_jour[jour]++
} END {
    print "=== ÉVOLUTION QUOTIDIENNE ==="
    for (j in ca_jour) {
        print "Jour " j ": " ca_jour[j] "€ (" ventes_jour[j] " ventes)"
    }

    # Calcul de la tendance (simplifié)
    print ""
    print "Analyse de tendance:"
    if (ca_jour["18"] > ca_jour["15"]) {
        print "  Tendance: Croissante"
    } else {
        print "  Tendance: Décroissante"
    }
}' ventes_detaillees.txt | sort -k2

echo ""
echo "4. Détection d'anomalies :"
awk -F',' 'NR > 1 {
    vendeur = $2
    ca = $4 * $5

    # Calculer la moyenne
    ca_vendeur[vendeur] += ca
    count_vendeur[vendeur]++
    ventes[NR] = vendeur ":" ca
} END {
    # Calculer les moyennes
    for (v in ca_vendeur) {
        moyenne[v] = ca_vendeur[v] / count_vendeur[v]
    }

    print "=== DÉTECTION D'\''ANOMALIES ==="
    print "Seuil: ±50% de la moyenne du vendeur"
    print ""

    for (i in ventes) {
        split(ventes[i], parts, ":")
        vendeur = parts[1]
        ca = parts[2]

        seuil_bas = moyenne[vendeur] * 0.5
        seuil_haut = moyenne[vendeur] * 1.5

        if (ca < seuil_bas) {
            print "🔻 Vente faible: " vendeur " - " ca "€ (moy: " int(moyenne[vendeur]) "€)"
        } else if (ca > seuil_haut) {
            print "🔺 Vente élevée: " vendeur " - " ca "€ (moy: " int(moyenne[vendeur]) "€)"
        }
    }
}' ventes_detaillees.txt
```

### Combinaison grep, sed, awk

```bash
#!/bin/bash

echo "=== Combinaison grep, sed, awk ==="

# Pipeline complexe d'analyse de logs
echo "Pipeline d'analyse de logs Apache/Nginx :"

# Créer un log Apache simulé
cat > apache_access.log << 'EOF'
192.168.1.100 - - [13/Jul/2025:08:15:23 +0000] "GET /index.html HTTP/1.1" 200 1024 "http://google.com" "Mozilla/5.0"
192.168.1.101 - - [13/Jul/2025:08:15:24 +0000] "POST /api/login HTTP/1.1" 200 256 "-" "API Client 1.0"
192.168.1.102 - - [13/Jul/2025:08:15:25 +0000] "GET /products HTTP/1.1" 200 2048 "http://example.com" "Mozilla/5.0"
192.168.1.100 - - [13/Jul/2025:08:15:26 +0000] "GET /cart HTTP/1.1" 404 512 "http://example.com" "Mozilla/5.0"
192.168.1.103 - - [13/Jul/2025:08:15:27 +0000] "POST /api/order HTTP/1.1" 500 128 "-" "API Client 1.0"
EOF

echo "Log Apache original :"
cat apache_access.log
echo ""

echo "=== Pipeline complet d'analyse ==="

cat apache_access.log | \
    # 1. Nettoyer et normaliser avec sed
    sed -E 's/\\[([^]]+)\\]/\\1/' | \
    sed -E 's/"([^"]*)" "([^"]*)"$/\\1|\\2/' | \
    # 2. Filtrer les erreurs avec grep
    grep -E "(40[0-9]|50[0-9])" | \
    # 3. Analyser avec awk
    awk '{
        split($0, parts, " ")
        ip = parts[1]
        status = parts[6]
        size = parts[7]

        print "Erreur détectée:"
        print "  IP:", ip
        print "  Code:", status
        print "  Taille:", size
        print "  URL:", parts[5]
        print ""
    }'

echo "=== Autre exemple: analyse de performance ==="

cat apache_access.log | \
    # Extraire les informations clés
    sed -E 's/^([0-9.]+).*"([A-Z]+) ([^"]+)".*([0-9]+) ([0-9]+).*/\\1|\\2|\\3|\\4|\\5/' | \
    # Analyser les performances
    awk -F'|' '{
        ip = $1
        method = $2
        url = $3
        status = $4
        size = $5

        # Statistiques par IP
        requests_ip[ip]++
        size_ip[ip] += size

        # Statistiques par URL
        requests_url[url]++

        # Statistiques par méthode
        methods[method]++

        total_size += size
        total_requests++
    } END {
        print "=== RAPPORT DE PERFORMANCE ==="
        print "Total requêtes:", total_requests
        print "Taille totale:", total_size, "octets"
        print "Taille moyenne:", int(total_size/total_requests), "octets"
        print ""

        print "Top IPs:"
        for (ip in requests_ip) {
            print "  " ip ":", requests_ip[ip], "req,", size_ip[ip], "octets"
        }

        print ""
        print "Méthodes HTTP:"
        for (method in methods) {
            print "  " method ":", methods[method]
        }
    }'

# Nettoyer
rm -f logs_complexes.txt config_original.txt ventes_detaillees.txt apache_access.log
```

## Validation de formats complexes (emails, dates, IPs, etc.)

### Validation d'emails

```bash
#!/bin/bash

echo "=== Validation d'emails avancée ==="

# Créer un fichier avec différents formats d'emails
cat > emails_test.txt << 'EOF'
simple@example.com
user.name@domain.org
test+tag@company.co.uk
user123@subdomain.example.com
invalid.email
@invalid.com
user@
user..double@example.com
user@domain.c
very.long.email.address@very.long.domain.name.example.com
user@[192.168.1.1]
admin@localhost
test@domain-with-dashes.com
user@domain_with_underscores.com
EOF

echo "Emails à tester :"
cat emails_test.txt
echo ""

echo "=== Patterns de validation d'emails ==="

echo "1. Validation basique :"
grep -E "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$" emails_test.txt

echo ""
echo "2. Validation plus stricte (pas de points consécutifs) :"
grep -E "^[a-zA-Z0-9]([a-zA-Z0-9._%-]*[a-zA-Z0-9])?@[a-zA-Z0-9]([a-zA-Z0-9.-]*[a-zA-Z0-9])?\\.[a-zA-Z]{2,}$" emails_test.txt

echo ""
echo "3. Validation avec sous-domaines :"
grep -E "^[a-zA-Z0-9._%+-]+@([a-zA-Z0-9-]+\\.)+[a-zA-Z]{2,}$" emails_test.txt

echo ""
echo "4. Emails invalides détectés :"
echo "Lignes ne passant PAS la validation :"
grep -v -E "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$" emails_test.txt

echo ""
echo "=== Fonction de validation avancée ==="

# Fonction de validation complète d'email
valider_email_complet() {
    local email="$1"
    local erreurs=()

    echo "Validation de : '$email'"

    # Test 1 : Format général
    if [[ ! "$email" =~ ^[^@]+@[^@]+$ ]]; then
        erreurs+=("Format général invalide")
    fi

    # Séparer partie locale et domaine
    local partie_locale="${email%@*}"
    local domaine="${email#*@}"

    # Test 2 : Partie locale
    if [ ${#partie_locale} -eq 0 ]; then
        erreurs+=("Partie locale vide")
    elif [ ${#partie_locale} -gt 64 ]; then
        erreurs+=("Partie locale trop longue (>64 caractères)")
    elif [[ ! "$partie_locale" =~ ^[a-zA-Z0-9._%+-]+$ ]]; then
        erreurs+=("Caractères invalides dans partie locale")
    elif [[ "$partie_locale" =~ ^\\.|\\.$|\\.\\.  ]]; then
        erreurs+=("Points mal placés dans partie locale")
    fi

    # Test 3 : Domaine
    if [ ${#domaine} -eq 0 ]; then
        erreurs+=("Domaine vide")
    elif [ ${#domaine} -gt 253 ]; then
        erreurs+=("Domaine trop long (>253 caractères)")
    elif [[ ! "$domaine" =~ \\. ]]; then
        erreurs+=("Domaine sans point")
    elif [[ ! "$domaine" =~ ^[a-zA-Z0-9.-]+$ ]]; then
        erreurs+=("Caractères invalides dans domaine")
    fi

    # Test 4 : Extension
    local extension="${domaine##*.}"
    if [ ${#extension} -lt 2 ]; then
        erreurs+=("Extension trop courte")
    elif [ ${#extension} -gt 6 ]; then
        erreurs+=("Extension trop longue")
    fi

    # Résultat
    if [ ${#erreurs[@]} -eq 0 ]; then
        echo "  ✅ Email valide"
        return 0
    else
        echo "  ❌ Email invalide :"
        for erreur in "${erreurs[@]}"; do
            echo "    - $erreur"
        done
        return 1
    fi
}

# Test de la fonction
echo "Tests avec la fonction de validation :"
emails_sample=("simple@example.com" "invalid.email" "user..double@test.com" "test@domain.co.uk")
for email in "${emails_sample[@]}"; do
    valider_email_complet "$email"
    echo ""
done
```

### Validation de dates

```bash
#!/bin/bash

echo "=== Validation de dates ==="

# Créer différents formats de dates
cat > dates_test.txt << 'EOF'
2025-07-13
13/07/2025
13-07-2025
07/13/2025
2025/07/13
13.07.2025
2025.07.13
13 juillet 2025
2025-13-07
2025-07-32
2025-02-29
2024-02-29
1999-12-31
2000-01-01
EOF

echo "Dates à tester :"
cat dates_test.txt
echo ""

echo "=== Patterns de validation de dates ==="

echo "1. Format ISO (YYYY-MM-DD) :"
grep -E "^[0-9]{4}-[0-9]{2}-[0-9]{2}$" dates_test.txt

echo ""
echo "2. Format français (DD/MM/YYYY) :"
grep -E "^[0-9]{2}/[0-9]{2}/[0-9]{4}$" dates_test.txt

echo ""
echo "3. Format américain (MM/DD/YYYY) :"
grep -E "^[0-9]{2}/[0-9]{2}/[0-9]{4}$" dates_test.txt

echo ""
echo "4. Tous les formats avec séparateurs :"
grep -E "^[0-9]{2,4}[./-][0-9]{2}[./-][0-9]{2,4}$" dates_test.txt

echo ""
echo "=== Validation avancée des dates ==="

# Fonction de validation de date
valider_date() {
    local date="$1"
    local format="${2:-auto}"

    echo "Validation de : '$date'"

    # Détection automatique du format
    if [ "$format" = "auto" ]; then
        if [[ "$date" =~ ^[0-9]{4}-[0-9]{2}-[0-9]{2}$ ]]; then
            format="iso"
        elif [[ "$date" =~ ^[0-9]{2}/[0-9]{2}/[0-9]{4}$ ]]; then
            format="fr"
        elif [[ "$date" =~ ^[0-9]{2}\\.[0-9]{2}\\.[0-9]{4}$ ]]; then
            format="de"
        else
            echo "  ❌ Format non reconnu"
            return 1
        fi
    fi

    # Extraction des composants selon le format
    local annee mois jour
    case "$format" in
        "iso")
            annee="${date%-*-*}"
            reste="${date#*-}"
            mois="${reste%-*}"
            jour="${reste#*-}"
            ;;
        "fr")
            jour="${date%/*/*/*}"
            reste="${date#*/}"
            mois="${reste%/*}"
            annee="${reste#*/}"
            ;;
        "de")
            jour="${date%.*.*}"
            reste="${date#*.}"
            mois="${reste%.*}"
            annee="${reste#*.}"
            ;;
    esac

    echo "  Format détecté : $format"
    echo "  Année: $annee, Mois: $mois, Jour: $jour"

    # Validations
    local erreurs=()

    # Année
    if [ "$annee" -lt 1900 ] || [ "$annee" -gt 2100 ]; then
        erreurs+=("Année hors limites (1900-2100)")
    fi

    # Mois
    if [ "$mois" -lt 1 ] || [ "$mois" -gt 12 ]; then
        erreurs+=("Mois invalide (1-12)")
    fi

    # Jour
    if [ "$jour" -lt 1 ] || [ "$jour" -gt 31 ]; then
        erreurs+=("Jour invalide (1-31)")
    fi

    # Validation spécifique par mois
    case "$mois" in
        02)  # Février
            local max_jour=28
            # Année bissextile
            if [ $((annee % 4)) -eq 0 ] && { [ $((annee % 100)) -ne 0 ] || [ $((annee % 400)) -eq 0 ]; }; then
                max_jour=29
            fi
            if [ "$jour" -gt "$max_jour" ]; then
                erreurs+=("Jour invalide pour février en $annee (max: $max_jour)")
            fi
            ;;
        04|06|09|11)  # Mois à 30 jours
            if [ "$jour" -gt 30 ]; then
                erreurs+=("Jour invalide pour le mois $mois (max: 30)")
            fi
            ;;
    esac

    # Résultat
    if [ ${#erreurs[@]} -eq 0 ]; then
        echo "  ✅ Date valide"
        return 0
    else
        echo "  ❌ Date invalide :"
        for erreur in "${erreurs[@]}"; do
            echo "    - $erreur"
        done
        return 1
    fi
}

# Tests de validation
echo "Tests de validation de dates :"
dates_sample=("2025-07-13" "2025-02-29" "2024-02-29" "13/07/2025" "2025-13-07")
for date in "${dates_sample[@]}"; do
    valider_date "$date"
    echo ""
done
```

### Validation d'adresses IP

```bash
#!/bin/bash

echo "=== Validation d'adresses IP ==="

# Créer différentes adresses IP
cat > ips_test.txt << 'EOF'
192.168.1.1
10.0.0.1
172.16.255.254
127.0.0.1
255.255.255.255
256.1.1.1
192.168.1
192.168.1.1.1
0.0.0.0
999.999.999.999
192.168.01.1
192.168.1.001
EOF

echo "Adresses IP à tester :"
cat ips_test.txt
echo ""

echo "=== Patterns de validation IP ==="

echo "1. Pattern simple (syntaxe seulement) :"
grep -E "^[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}$" ips_test.txt

echo ""
echo "2. Pattern plus strict (pas de zéros en tête) :"
grep -E "^([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\\.([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\\.([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\\.([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])$" ips_test.txt

echo ""
echo "=== Fonction de validation IP avancée ==="

# Fonction de validation IP complète
valider_ip() {
    local ip="$1"

    echo "Validation de : '$ip'"

    # Test du format de base
    if [[ ! "$ip" =~ ^[0-9.]+$ ]]; then
        echo "  ❌ Caractères invalides"
        return 1
    fi

    # Séparer en octets
    IFS='.' read -ra octets <<< "$ip"

    # Vérifier le nombre d'octets
    if [ ${#octets[@]} -ne 4 ]; then
        echo "  ❌ Doit contenir exactement 4 octets"
        return 1
    fi

    # Valider chaque octet
    local erreurs=()
    for i in "${!octets[@]}"; do
        local octet="${octets[$i]}"

        # Vérifier que c'est un nombre
        if [[ ! "$octet" =~ ^[0-9]+$ ]]; then
            erreurs+=("Octet $((i+1)) invalide : '$octet'")
            continue
        fi

        # Vérifier les zéros en tête
        if [ ${#octet} -gt 1 ] && [[ "$octet" =~ ^0 ]]; then
            erreurs+=("Octet $((i+1)) : zéros en tête non autorisés")
        fi

        # Vérifier la valeur
        if [ "$octet" -lt 0 ] || [ "$octet" -gt 255 ]; then
            erreurs+=("Octet $((i+1)) hors limites : $octet (0-255)")
        fi
    done

    # Classification de l'IP
    local classe=""
    local type=""
    local premier_octet="${octets[0]}"

    if [ "$premier_octet" -ge 1 ] && [ "$premier_octet" -le 126 ]; then
        classe="A"
    elif [ "$premier_octet" -ge 128 ] && [ "$premier_octet" -le 191 ]; then
        classe="B"
    elif [ "$premier_octet" -ge 192 ] && [ "$premier_octet" -le 223 ]; then
        classe="C"
    elif [ "$premier_octet" -ge 224 ] && [ "$premier_octet" -le 239 ]; then
        classe="D (Multicast)"
    elif [ "$premier_octet" -ge 240 ] && [ "$premier_octet" -le 255 ]; then
        classe="E (Expérimental)"
    fi

    # Déterminer le type (privée/publique)
    if [[ "$ip" =~ ^10\\. ]] ||
       [[ "$ip" =~ ^172\\.(1[6-9]|2[0-9]|3[01])\\. ]] ||
       [[ "$ip" =~ ^192\\.168\\. ]]; then
        type="Privée"
    elif [[ "$ip" =~ ^127\\. ]]; then
        type="Loopback"
    elif [[ "$ip" =~ ^169\\.254\\. ]]; then
        type="Link-local"
    else
        type="Publique"
    fi

    # Résultat
    if [ ${#erreurs[@]} -eq 0 ]; then
        echo "  ✅ IP valide"
        echo "  📊 Classe : $classe"
        echo "  🌐 Type : $type"
        return 0
    else
        echo "  ❌ IP invalide :"
        for erreur in "${erreurs[@]}"; do
            echo "    - $erreur"
        done
        return 1
    fi
}

# Tests de validation IP
echo "Tests de validation IP :"
ips_sample=("192.168.1.1" "256.1.1.1" "10.0.0.1" "192.168.01.1" "127.0.0.1")
for ip in "${ips_sample[@]}"; do
    valider_ip "$ip"
    echo ""
done
```

### Validation de numéros de téléphone

```bash
#!/bin/bash

echo "=== Validation de numéros de téléphone ==="

# Créer différents formats de téléphones
cat > telephones_test.txt << 'EOF'
06.12.34.56.78
06 12 34 56 78
06-12-34-56-78
0612345678
+33 6 12 34 56 78
+33612345678
01.23.45.67.89
05 56 78 90 12
09 87 65 43 21
07123456789
0123456789
1234567890
+1 555 123 4567
+44 20 7946 0958
EOF

echo "Numéros de téléphone à tester :"
cat telephones_test.txt
echo ""

echo "=== Patterns de validation téléphone ==="

echo "1. Numéros français (format libre) :"
grep -E "^(\\+33|0)[1-9]([0-9. -]){8}[0-9]$" telephones_test.txt

echo ""
echo "2. Format français strict (10 chiffres) :"
grep -E "^0[1-9][0-9]{8}$" telephones_test.txt

echo ""
echo "3. Format international (+33) :"
grep -E "^\\+33 ?[1-9]([0-9 ]){8}[0-9]$" telephones_test.txt

echo ""
echo "=== Fonction de validation téléphone ==="

# Fonction de validation complète de téléphone
valider_telephone() {
    local numero="$1"
    local pays="${2:-FR}"

    echo "Validation de : '$numero' (Pays: $pays)"

    # Nettoyer le numéro (supprimer espaces, points, tirets)
    local numero_propre=$(echo "$numero" | tr -d ' .-')

    case "$pays" in
        "FR")
            # Validation française
            local erreurs=()

            # Format international
            if [[ "$numero_propre" =~ ^\\+33 ]]; then
                numero_propre="${numero_propre#+33}"
                # Ajouter le 0
                numero_propre="0$numero_propre"
            fi

            # Vérifications
            if [ ${#numero_propre} -ne 10 ]; then
                erreurs+=("Doit contenir exactement 10 chiffres")
            fi

            if [[ ! "$numero_propre" =~ ^0[1-9] ]]; then
                erreurs+=("Doit commencer par 0 suivi de 1-9")
            fi

            if [[ ! "$numero_propre" =~ ^[0-9]+$ ]]; then
                erreurs+=("Doit contenir seulement des chiffres")
            fi

            # Classification du numéro
            local type=""
            local prefix="${numero_propre:0:2}"
            case "$prefix" in
                01) type="Île-de-France (fixe)" ;;
                02) type="Nord-Ouest (fixe)" ;;
                03) type="Nord-Est (fixe)" ;;
                04) type="Sud-Est (fixe)" ;;
                05) type="Sud-Ouest (fixe)" ;;
                06|07) type="Mobile" ;;
                08) type="Numéro spécial" ;;
                09) type="Non géographique" ;;
                *) type="Inconnu" ;;
            esac

            # Résultat
            if [ ${#erreurs[@]} -eq 0 ]; then
                echo "  ✅ Numéro valide"
                echo "  📱 Type : $type"
                echo "  📞 Format normalisé : $numero_propre"
                return 0
            else
                echo "  ❌ Numéro invalide :"
                for erreur in "${erreurs[@]}"; do
                    echo "    - $erreur"
                done
                return 1
            fi
            ;;
        "US")
            # Validation américaine (format simplifié)
            if [[ "$numero_propre" =~ ^\\+1 ]]; then
                numero_propre="${numero_propre#+1}"
            fi

            if [ ${#numero_propre} -eq 10 ] && [[ "$numero_propre" =~ ^[2-9][0-9]{9}$ ]]; then
                echo "  ✅ Numéro US valide"
                echo "  📞 Format : ${numero_propre:0:3}-${numero_propre:3:3}-${numero_propre:6:4}"
                return 0
            else
                echo "  ❌ Format US invalide"
                return 1
            fi
            ;;
        *)
            echo "  ❌ Pays non supporté : $pays"
            return 1
            ;;
    esac
}

# Tests de validation téléphone
echo "Tests de validation téléphone :"
telephones_sample=("06.12.34.56.78" "+33 6 12 34 56 78" "01.23.45.67.89" "1234567890")
for tel in "${telephones_sample[@]}"; do
    valider_telephone "$tel" "FR"
    echo ""
done
```

### Validation d'URLs

```bash
#!/bin/bash

echo "=== Validation d'URLs ==="

# Créer différentes URLs
cat > urls_test.txt << 'EOF'
http://example.com
https://www.example.com
https://subdomain.example.com:8080/path?param=value
ftp://files.example.com/file.txt
https://example.com/path/to/file.html#section
http://192.168.1.1:3000
https://localhost:8080/api/v1/users
invalid-url
http://
https://example
https://example.com/path with spaces
https://example.com/path%20encoded
EOF

echo "URLs à tester :"
cat urls_test.txt
echo ""

echo "=== Patterns de validation URL ==="

echo "1. URLs HTTP/HTTPS basiques :"
grep -E "^https?://[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}" urls_test.txt

echo ""
echo "2. URLs complètes avec port et chemin :"
grep -E "^https?://[a-zA-Z0-9.-]+(:[0-9]+)?(/[^[:space:]]*)?$" urls_test.txt

echo ""
echo "3. Tous protocoles :"
grep -E "^[a-z]+://[^[:space:]]+$" urls_test.txt

echo ""
echo "=== Fonction de validation URL avancée ==="

# Fonction de validation URL complète
valider_url() {
    local url="$1"

    echo "Validation de : '$url'"

    # Pattern complet pour URL
    local pattern="^(https?|ftp)://([a-zA-Z0-9.-]+|\\[[0-9a-fA-F:]+\\])(:[0-9]+)?(/[^[:space:]#]*)?(\#[^[:space:]]*)?(\?[^[:space:]#]*)?$"

    if [[ ! "$url" =~ $pattern ]]; then
        echo "  ❌ Format URL invalide"
        return 1
    fi

    # Extraction des composants
    local protocole=$(echo "$url" | sed -E 's|^([^:]+)://.*|\\1|')
    local reste="${url#*://}"
    local serveur=$(echo "$reste" | sed -E 's|([^/:]+).*|\\1|')
    local port=""
    local chemin=""

    # Extraction du port
    if [[ "$serveur" =~ :[0-9]+$ ]]; then
        port="${serveur##*:}"
        serveur="${serveur%:*}"
    fi

    # Extraction du chemin
    if [[ "$reste" =~ / ]]; then
        chemin="/${reste#*/}"
        chemin="${chemin%\?*}"  # Supprimer les paramètres
        chemin="${chemin%#*}"   # Supprimer l'ancre
    fi

    echo "  📊 Analyse :"
    echo "    Protocole : $protocole"
    echo "    Serveur : $serveur"
    [ -n "$port" ] && echo "    Port : $port"
    [ -n "$chemin" ] && echo "    Chemin : $chemin"

    # Validations spécifiques
    local avertissements=()

    # Vérifier le port
    if [ -n "$port" ] && ([ "$port" -lt 1 ] || [ "$port" -gt 65535 ]); then
        avertissements+=("Port hors limites (1-65535)")
    fi

    # Vérifier le serveur
    if [[ ! "$serveur" =~ ^[a-zA-Z0-9.-]+$ ]] && [[ ! "$serveur" =~ ^[0-9.]+$ ]]; then
        avertissements+=("Nom de serveur suspect")
    fi

    # Vérifier les espaces dans l'URL
    if [[ "$url" =~ [[:space:]] ]]; then
        avertissements+=("Espaces non encodés détectés")
    fi

    # Résultat
    echo "  ✅ URL structurellement valide"

    if [ ${#avertissements[@]} -gt 0 ]; then
        echo "  ⚠️  Avertissements :"
        for avert in "${avertissements[@]}"; do
            echo "    - $avert"
        done
    fi

    return 0
}

# Tests de validation URL
echo "Tests de validation URL :"
urls_sample=("https://www.example.com:8080/path" "http://192.168.1.1" "invalid-url" "ftp://files.test.com")
for url in "${urls_sample[@]}"; do
    valider_url "$url"
    echo ""
done

# Nettoyer tous les fichiers de test
rm -f emails_test.txt dates_test.txt ips_test.txt telephones_test.txt urls_test.txt
```

### Outil de validation universel

```bash
#!/bin/bash

echo "=== Outil de validation universel ==="

# Fonction de validation automatique
valider_automatique() {
    local donnee="$1"

    echo "Détection automatique du type pour : '$donnee'"

    # Tests de détection
    if [[ "$donnee" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$ ]]; then
        echo "  🔍 Type détecté : Email"
        valider_email_complet "$donnee"

    elif [[ "$donnee" =~ ^[0-9]{4}[-/.][0-9]{2}[-/.][0-9]{2}$ ]] || [[ "$donnee" =~ ^[0-9]{2}[-/.][0-9]{2}[-/.][0-9]{4}$ ]]; then
        echo "  🔍 Type détecté : Date"
        valider_date "$donnee"

    elif [[ "$donnee" =~ ^[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}$ ]]; then
        echo "  🔍 Type détecté : Adresse IP"
        valider_ip "$donnee"

    elif [[ "$donnee" =~ ^(\\+|0)[0-9 .-]{8,}$ ]]; then
        echo "  🔍 Type détecté : Numéro de téléphone"
        valider_telephone "$donnee"

    elif [[ "$donnee" =~ ^https?:// ]] || [[ "$donnee" =~ ^ftp:// ]]; then
        echo "  🔍 Type détecté : URL"
        valider_url "$donnee"

    else
        echo "  ❓ Type non reconnu"
        echo "  ℹ️  Analyse générale :"
        echo "    Longueur : ${#donnee} caractères"

        if [[ "$donnee" =~ ^[0-9]+$ ]]; then
            echo "    Contenu : Numérique uniquement"
        elif [[ "$donnee" =~ ^[a-zA-Z]+$ ]]; then
            echo "    Contenu : Alphabétique uniquement"
        elif [[ "$donnee" =~ ^[a-zA-Z0-9]+$ ]]; then
            echo "    Contenu : Alphanumérique"
        else
            echo "    Contenu : Mixte avec caractères spéciaux"
        fi

        return 1
    fi
}

# Batch de validation
valider_batch() {
    local fichier="$1"

    if [ ! -f "$fichier" ]; then
        echo "❌ Fichier '$fichier' introuvable"
        return 1
    fi

    echo "=== Validation en lot de '$fichier' ==="

    local total=0
    local valides=0
    local invalides=0

    while IFS= read -r ligne; do
        [ -z "$ligne" ] && continue  # Ignorer les lignes vides

        ((total++))
        echo ""
        echo "--- Élément $total ---"

        if valider_automatique "$ligne"; then
            ((valides++))
        else
            ((invalides++))
        fi
    done < "$fichier"

    echo ""
    echo "=== RAPPORT FINAL ==="
    echo "Total éléments : $total"
    echo "Valides : $valides"
    echo "Invalides : $invalides"
    echo "Taux de validité : $((valides * 100 / total))%"
}

# Créer un fichier de test mixte
cat > donnees_mixtes.txt << 'EOF'
test@example.com
192.168.1.1
2025-07-13
06.12.34.56.78
https://www.example.com
invalid-data
admin@company.org
256.1.1.1
13/07/2025
+33 6 12 34 56 78
EOF

echo "Test de validation automatique :"
valider_batch "donnees_mixtes.txt"

# Nettoyer
rm -f donnees_mixtes.txt
```

## Récapitulatif

✅ **Ce que vous avez appris dans ce chapitre :**

1. **Syntaxe des regex** : Différences entre BRE, ERE et PCRE, métacaractères
2. **Utilisation avancée** : Techniques sophistiquées avec grep, sed, awk
3. **Validation de formats** : Emails, dates, IPs, téléphones, URLs avec fonctions complètes

✅ **Points clés à retenir :**

**Types de regex :**
```bash
grep "pattern"           # BRE (Basic Regular Expressions)
grep -E "pattern"        # ERE (Extended Regular Expressions)
grep -P "pattern"        # PCRE (Perl Compatible) - si disponible
```

**Métacaractères essentiels :**
```bash
.           # N'importe quel caractère
*           # Zéro ou plusieurs du caractère précédent
+           # Un ou plusieurs (ERE)
?           # Zéro ou un (ERE)
^           # Début de ligne
$           # Fin de ligne
[abc]       # Un caractère parmi a, b, c
[^abc]      # Tout sauf a, b, c
{n,m}       # Entre n et m occurrences (ERE)
()          # Groupement (ERE)
|           # Alternative (ERE)
```

**Classes de caractères POSIX :**
```bash
[[:alpha:]]    # Lettres
[[:digit:]]    # Chiffres
[[:alnum:]]    # Lettres et chiffres
[[:space:]]    # Espaces blancs
[[:upper:]]    # Majuscules
[[:lower:]]    # Minuscules
[[:punct:]]    # Ponctuation
```

**Patterns de validation courants :**
```bash
# Email
^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$

# IP
^([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\.([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\.([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\.([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])$

# Date ISO
^[0-9]{4}-[0-9]{2}-[0-9]{2}$

# Téléphone français
^0[1-9][0-9]{8}$

# URL
^https?://[a-zA-Z0-9.-]+(:[0-9]+)?(/.*)?$
```

**Techniques avancées :**
- **Capture avec sed** : `sed -E 's/(group1)(group2)/\1-\2/'`
- **Validation multicritères** : Combiner plusieurs tests
- **Détection automatique** : Identifier le type de données
- **Pipeline complexes** : grep | sed | awk avec regex

✅ **Bonnes pratiques :**

1. **Choisir le bon type de regex :**
   - BRE pour la compatibilité maximale
   - ERE pour la plupart des cas (plus lisible)
   - PCRE pour des besoins très spécifiques

2. **Optimisation :**
   - Utiliser des anchors (^ $) quand possible
   - Éviter les patterns trop gourmands (.*)
   - Préférer [0-9] à \d pour la portabilité

3. **Lisibilité :**
   - Commenter les regex complexes
   - Diviser en plusieurs étapes si nécessaire
   - Tester avec des données variées

4. **Validation robuste :**
   - Valider le format ET la logique
   - Gérer les cas limites
   - Fournir des messages d'erreur clairs

✅ **Cas d'usage pratiques :**

**Administration système :**
- Validation de fichiers de configuration
- Analyse de logs avec patterns complexes
- Nettoyage de données d'entrée

**Développement :**
- Validation de formulaires
- Parsing de fichiers de données
- Transformation de formats

**Sécurité :**
- Détection d'intrusions dans les logs
- Validation d'entrées utilisateur
- Nettoyage de données sensibles

✅ **Outils et commandes clés :**
```bash
# Test de regex
echo "test" | grep -E "pattern"

# Validation avec retour de code
if [[ "$data" =~ ^pattern$ ]]; then
    echo "Valide"
fi

# Extraction avec sed
echo "$data" | sed -E 's/pattern/\1/'

# Analyse avec awk
awk '/pattern/ {action}' file
```

✅ **Ressources pour approfondir :**
- Documentation POSIX pour la portabilité
- Sites de test en ligne (regex101.com)
- Man pages : `man 7 regex`
- Exemples spécifiques à votre domaine

✅ **Prochaines étapes :**
Dans le chapitre 11, nous aborderons la gestion des erreurs et le débogage !

---

💡 **Conseil pratique :** Les expressions régulières sont puissantes mais peuvent devenir complexes. Commencez simple, testez souvent, et documentez vos patterns complexes. Une regex qui fonctionne est mieux qu'une regex parfaite mais illisible !

### Exercices pratiques

```bash
#!/bin/bash

echo "=== EXERCICES PRATIQUES ==="

# Exercice 1 : Créer un validateur de mots de passe
echo "Exercice 1 : Validateur de mots de passe"
echo "Critères : 8+ caractères, 1 majuscule, 1 minuscule, 1 chiffre, 1 caractère spécial"

valider_mot_de_passe() {
    local mdp="$1"

    # À compléter par l'utilisateur
    echo "Fonction à implémenter"
}

# Exercice 2 : Parser un fichier de log personnalisé
echo ""
echo "Exercice 2 : Parser des logs personnalisés"
echo "Format : [NIVEAU] timestamp utilisateur action détails"

extraire_info_log() {
    local ligne_log="$1"

    # À compléter par l'utilisateur
    echo "Fonction à implémenter"
}

# Exercice 3 : Validateur de numéro de carte bancaire (format simplifié)
echo ""
echo "Exercice 3 : Validateur de carte bancaire"
echo "Format : 4 groupes de 4 chiffres séparés par espaces ou tirets"

valider_carte_bancaire() {
    local numero="$1"

    # À compléter par l'utilisateur
    echo "Fonction à implémenter"
}

echo ""
echo "💡 Solutions disponibles en fin de chapitre ou sur demande !"
```

### Solutions des exercices

```bash
#!/bin/bash

echo "=== SOLUTIONS DES EXERCICES ==="

# Solution Exercice 1 : Validateur de mots de passe
valider_mot_de_passe_solution() {
    local mdp="$1"
    local erreurs=()

    echo "Validation du mot de passe : '***'"

    # Longueur minimum
    if [ ${#mdp} -lt 8 ]; then
        erreurs+=("Trop court (minimum 8 caractères)")
    fi

    # Au moins une majuscule
    if [[ ! "$mdp" =~ [A-Z] ]]; then
        erreurs+=("Doit contenir au moins une majuscule")
    fi

    # Au moins une minuscule
    if [[ ! "$mdp" =~ [a-z] ]]; then
        erreurs+=("Doit contenir au moins une minuscule")
    fi

    # Au moins un chiffre
    if [[ ! "$mdp" =~ [0-9] ]]; then
        erreurs+=("Doit contenir au moins un chiffre")
    fi

    # Au moins un caractère spécial
    if [[ ! "$mdp" =~ [^a-zA-Z0-9] ]]; then
        erreurs+=("Doit contenir au moins un caractère spécial")
    fi

    # Résultat
    if [ ${#erreurs[@]} -eq 0 ]; then
        echo "  ✅ Mot de passe fort"
        return 0
    else
        echo "  ❌ Mot de passe faible :"
        for erreur in "${erreurs[@]}"; do
            echo "    - $erreur"
        done
        return 1
    fi
}

# Solution Exercice 2 : Parser de logs
extraire_info_log_solution() {
    local ligne_log="$1"

    # Pattern pour logs : [NIVEAU] timestamp utilisateur action détails
    if [[ "$ligne_log" =~ ^\[([A-Z]+)\][[:space:]]+([0-9-]+[[:space:]]+[0-9:]+)[[:space:]]+([a-zA-Z0-9_]+)[[:space:]]+([a-zA-Z_]+)[[:space:]]+(.*)$ ]]; then
        local niveau="${BASH_REMATCH[1]}"
        local timestamp="${BASH_REMATCH[2]}"
        local utilisateur="${BASH_REMATCH[3]}"
        local action="${BASH_REMATCH[4]}"
        local details="${BASH_REMATCH[5]}"

        echo "Parsing réussi :"
        echo "  Niveau : $niveau"
        echo "  Timestamp : $timestamp"
        echo "  Utilisateur : $utilisateur"
        echo "  Action : $action"
        echo "  Détails : $details"
        return 0
    else
        echo "Format de log non reconnu"
        return 1
    fi
}

# Solution Exercice 3 : Validateur de carte bancaire
valider_carte_bancaire_solution() {
    local numero="$1"

    echo "Validation carte bancaire : '${numero:0:4} **** **** ****'"

    # Nettoyer le numéro
    local numero_propre=$(echo "$numero" | tr -d ' -')

    # Vérifier le format
    if [[ ! "$numero_propre" =~ ^[0-9]{16}$ ]]; then
        echo "  ❌ Doit contenir exactement 16 chiffres"
        return 1
    fi

    # Identifier le type de carte (simplifié)
    local type=""
    local premier_chiffre="${numero_propre:0:1}"
    local deux_premiers="${numero_propre:0:2}"

    case "$premier_chiffre" in
        4) type="Visa" ;;
        5) type="MasterCard" ;;
        3)
            if [[ "$deux_premiers" =~ ^3[47]$ ]]; then
                type="American Express"
            else
                type="Autres (Diners Club, etc.)"
            fi
            ;;
        *) type="Type inconnu" ;;
    esac

    echo "  ✅ Format valide"
    echo "  💳 Type détecté : $type"
    echo "  🔒 Numéro formaté : ${numero_propre:0:4} ${numero_propre:4:4} ${numero_propre:8:4} ${numero_propre:12:4}"

    return 0
}

# Tests des solutions
echo "Tests des solutions :"

echo ""
echo "1. Test mots de passe :"
mots_de_passe=("password" "Password1!" "Abc123@def" "short")
for mdp in "${mots_de_passe[@]}"; do
    valider_mot_de_passe_solution "$mdp"
    echo ""
done

echo "2. Test parsing de logs :"
logs=("[INFO] 2025-07-13 10:30:25 alice login Connexion réussie depuis 192.168.1.100"
      "[ERROR] 2025-07-13 10:31:00 bob failed_login Tentative échouée"
      "Format invalide")
for log in "${logs[@]}"; do
    echo "Log : $log"
    extraire_info_log_solution "$log"
    echo ""
done

echo "3. Test cartes bancaires :"
cartes=("4532 1234 5678 9012" "5555-4444-3333-2222" "378282246310005" "123456789")
for carte in "${cartes[@]}"; do
    valider_carte_bancaire_solution "$carte"
    echo ""
done
```

**Note finale :** Ce chapitre vous donne une base solide pour maîtriser les expressions régulières en Bash. Continuez à pratiquer avec vos propres données et n'hésitez pas à consulter la documentation pour des besoins spécifiques !

⏭️
