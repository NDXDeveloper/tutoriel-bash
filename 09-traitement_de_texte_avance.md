🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Tutoriel Bash - Chapitre 9 : Traitement de texte avancé avec grep, sed, awk

## Introduction et utilisation de grep

### Qu'est-ce que grep ?

`grep` est un outil de **recherche de motifs** dans du texte. C'est comme une **fonction "Rechercher"** très puissante qui peut chercher des patterns complexes dans les fichiers.

**Analogie :** Imaginez que vous avez une pile de documents et que vous cherchez tous ceux qui contiennent le mot "contrat". `grep` fait exactement cela, mais de façon automatisée et avec des critères très sophistiqués.

**Signification :** grep = **G**lobal **R**egular **E**xpression **P**rint

### Utilisation de base de grep

```bash
#!/bin/bash

echo "=== Introduction à grep ==="

# Créer un fichier de test
cat > employes.txt << 'EOF'
Marie Dupont,Développeuse,IT,35000
Pierre Martin,Manager,Finance,45000
Sophie Bernard,Analyste,IT,38000
Jean Rousseau,Comptable,Finance,32000
Claire Moreau,Développeuse,IT,36000
Paul Durand,Directeur,Direction,60000
Alice Lambert,Analyste,Marketing,34000
Lucas Petit,Développeur,IT,37000
EOF

echo "Fichier de test créé :"
cat employes.txt
echo ""

# Recherches simples
echo "=== Recherches simples avec grep ==="

echo "1. Chercher tous les employés du département IT :"
grep "IT" employes.txt

echo ""
echo "2. Chercher tous les développeurs :"
grep "Développeu" employes.txt

echo ""
echo "3. Recherche insensible à la casse :"
grep -i "marie" employes.txt

echo ""
echo "4. Compter le nombre d'occurrences :"
echo "Nombre d'employés IT : $(grep -c "IT" employes.txt)"

echo ""
echo "5. Afficher les numéros de ligne :"
grep -n "Finance" employes.txt

echo ""
echo "6. Recherche inversée (qui ne contient PAS le motif) :"
echo "Employés qui ne sont PAS en IT :"
grep -v "IT" employes.txt
```

### Options utiles de grep

```bash
#!/bin/bash

echo "=== Options avancées de grep ==="

# Créer un fichier de logs pour les exemples
cat > app.log << 'EOF'
2025-07-13 10:15:23 [INFO] Application started
2025-07-13 10:15:24 [DEBUG] Loading configuration file
2025-07-13 10:15:25 [INFO] Database connection established
2025-07-13 10:15:26 [WARNING] High memory usage detected: 85%
2025-07-13 10:15:27 [ERROR] Failed to connect to external API
2025-07-13 10:15:28 [INFO] Retrying API connection
2025-07-13 10:15:29 [ERROR] API connection failed again
2025-07-13 10:15:30 [INFO] Switching to offline mode
EOF

echo "Fichier de logs :"
cat app.log
echo ""

# Options importantes
echo "=== Options importantes de grep ==="

echo "1. -i : Ignorer la casse"
grep -i "error" app.log

echo ""
echo "2. -n : Afficher les numéros de ligne"
grep -n "ERROR" app.log

echo ""
echo "3. -c : Compter les occurrences"
echo "Nombre d'erreurs : $(grep -c "ERROR" app.log)"

echo ""
echo "4. -l : Afficher seulement les noms de fichiers"
grep -l "ERROR" *.log 2>/dev/null

echo ""
echo "5. -A : Afficher N lignes après"
echo "Erreurs avec 2 lignes de contexte après :"
grep -A 2 "ERROR" app.log

echo ""
echo "6. -B : Afficher N lignes avant"
echo "Erreurs avec 1 ligne de contexte avant :"
grep -B 1 "ERROR" app.log

echo ""
echo "7. -C : Afficher N lignes de contexte (avant et après)"
echo "Erreurs avec 1 ligne de contexte :"
grep -C 1 "ERROR" app.log

echo ""
echo "8. --color : Coloration (automatique dans la plupart des terminaux)"
grep --color=always "ERROR" app.log

echo ""
echo "9. -o : Afficher seulement la partie qui correspond"
echo "Extraction des heures :"
grep -o "[0-9][0-9]:[0-9][0-9]:[0-9][0-9]" app.log
```

### Expressions régulières avec grep

```bash
#!/bin/bash

echo "=== Expressions régulières avec grep ==="

# Créer un fichier avec différents formats
cat > donnees_mixtes.txt << 'EOF'
jean.dupont@example.com
marie@test.org
contact@company.co.uk
invalid.email
user123@domain.com
admin@site.net
06.12.34.56.78
01-23-45-67-89
192.168.1.1
10.0.0.1
256.300.1.1
2025-07-13
13/07/2025
2025/07/13
13-07-2025
EOF

echo "Données de test :"
cat donnees_mixtes.txt
echo ""

echo "=== Patterns avec grep ==="

echo "1. Emails valides (pattern simple) :"
grep -E "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$" donnees_mixtes.txt

echo ""
echo "2. Numéros de téléphone français :"
grep -E "^0[1-9]([.\\-]?[0-9]{2}){4}$" donnees_mixtes.txt

echo ""
echo "3. Adresses IP (pattern basique) :"
grep -E "^[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}$" donnees_mixtes.txt

echo ""
echo "4. Dates au format YYYY-MM-DD :"
grep -E "^[0-9]{4}-[0-9]{2}-[0-9]{2}$" donnees_mixtes.txt

echo ""
echo "5. Lignes commençant par un chiffre :"
grep "^[0-9]" donnees_mixtes.txt

echo ""
echo "6. Lignes se terminant par .com ou .org :"
grep -E "\\.(com|org)$" donnees_mixtes.txt

echo ""
echo "7. Motifs avec caractères optionnels (?) :"
echo "Numéros avec séparateurs optionnels :"
grep -E "^0[1-9][.\\-]?[0-9]{2}[.\\-]?[0-9]{2}" donnees_mixtes.txt
```

### Fonction de recherche avancée

```bash
#!/bin/bash

# Fonction de recherche intelligente
rechercher_intelligent() {
    local motif="$1"
    local fichier="$2"
    local options="${3:-normal}"

    echo "=== Recherche intelligente ==="
    echo "Motif : '$motif'"
    echo "Fichier : '$fichier'"
    echo "Options : $options"
    echo ""

    if [ ! -f "$fichier" ]; then
        echo "❌ Fichier '$fichier' introuvable"
        return 1
    fi

    # Compter les occurrences
    local nb_occurrences=$(grep -c "$motif" "$fichier" 2>/dev/null || echo "0")
    echo "📊 Nombre d'occurrences : $nb_occurrences"

    if [ "$nb_occurrences" -eq 0 ]; then
        echo "❌ Aucune occurrence trouvée"

        # Suggestions
        echo "💡 Suggestions :"
        echo "  - Vérifiez l'orthographe"
        echo "  - Essayez la recherche insensible à la casse (-i)"
        echo "  - Utilisez des patterns plus généraux"
        return 1
    fi

    echo ""
    echo "🔍 Résultats :"

    case "$options" in
        "normal")
            grep -n --color=always "$motif" "$fichier"
            ;;
        "contexte")
            grep -n -C 2 --color=always "$motif" "$fichier"
            ;;
        "stats")
            echo "Lignes correspondantes :"
            grep -n "$motif" "$fichier" | nl
            echo ""
            echo "Statistiques :"
            echo "  - Première occurrence : ligne $(grep -n "$motif" "$fichier" | head -1 | cut -d: -f1)"
            echo "  - Dernière occurrence : ligne $(grep -n "$motif" "$fichier" | tail -1 | cut -d: -f1)"
            ;;
        "resume")
            local lignes_uniques=$(grep "$motif" "$fichier" | sort | uniq | wc -l)
            echo "  - Occurrences totales : $nb_occurrences"
            echo "  - Lignes uniques : $lignes_uniques"
            echo ""
            echo "Aperçu (5 premières occurrences) :"
            grep -n "$motif" "$fichier" | head -5
            ;;
    esac
}

# Tests de la fonction
echo "=== Tests de recherche intelligente ==="

rechercher_intelligent "ERROR" "app.log" "normal"
echo ""
rechercher_intelligent "INFO" "app.log" "stats"
echo ""
rechercher_intelligent "xyz123" "app.log" "normal"

# Nettoyer les fichiers de test
rm -f employes.txt app.log donnees_mixtes.txt
```

## Introduction et utilisation de sed

### Qu'est-ce que sed ?

`sed` (Stream EDitor) est un **éditeur de flux** qui permet de transformer du texte automatiquement. C'est comme avoir un assistant qui peut modifier automatiquement des milliers de documents selon vos instructions.

**Analogie :** Si grep est comme une loupe pour trouver du texte, sed est comme un correcteur automatique qui peut remplacer, supprimer, ou ajouter du texte selon des règles.

### Utilisation de base de sed

```bash
#!/bin/bash

echo "=== Introduction à sed ==="

# Créer un fichier de test
cat > texte_original.txt << 'EOF'
Bonjour le monde
Ceci est un test
Le test est important
Un autre test
Fin du fichier
EOF

echo "Fichier original :"
cat texte_original.txt
echo ""

echo "=== Opérations de base avec sed ==="

echo "1. Remplacer la première occurrence sur chaque ligne :"
sed 's/test/exemple/' texte_original.txt

echo ""
echo "2. Remplacer toutes les occurrences (flag g) :"
sed 's/test/exemple/g' texte_original.txt

echo ""
echo "3. Remplacer seulement sur la ligne 3 :"
sed '3s/test/exemple/' texte_original.txt

echo ""
echo "4. Supprimer des lignes :"
echo "Supprimer la ligne 2 :"
sed '2d' texte_original.txt

echo ""
echo "5. Supprimer les lignes contenant 'test' :"
sed '/test/d' texte_original.txt

echo ""
echo "6. Afficher seulement certaines lignes :"
echo "Lignes 2 à 4 :"
sed -n '2,4p' texte_original.txt

echo ""
echo "7. Ajouter du texte :"
echo "Ajouter une ligne au début :"
sed '1i\--- Début du document ---' texte_original.txt

echo ""
echo "8. Ajouter du texte à la fin :"
sed '$a\--- Fin du document ---' texte_original.txt
```

### Transformations avancées avec sed

```bash
#!/bin/bash

echo "=== Transformations avancées avec sed ==="

# Créer un fichier de configuration
cat > config.txt << 'EOF'
# Configuration de l'application
server_host=localhost
server_port=8080
database_host=localhost
database_port=3306
debug_mode=true
log_level=DEBUG
max_connections=100
timeout=30
# Fin de configuration
EOF

echo "Configuration originale :"
cat config.txt
echo ""

echo "=== Exemples de transformation ==="

echo "1. Changer l'environnement de dev vers production :"
sed -e 's/localhost/production.server.com/g' \
    -e 's/debug_mode=true/debug_mode=false/' \
    -e 's/log_level=DEBUG/log_level=ERROR/' \
    config.txt

echo ""
echo "2. Commenter certaines lignes :"
echo "Commenter les lignes contenant 'debug' ou 'log' :"
sed -E 's/^(.*debug.*|.*log.*)/#\1/' config.txt

echo ""
echo "3. Extraire seulement les valeurs :"
echo "Valeurs de configuration :"
sed -n 's/^[^#]*=\\(.*\\)/\\1/p' config.txt

echo ""
echo "4. Transformation de format :"
echo "Conversion en format JSON :"
sed -n '/^[^#]/s/^\\([^=]*\\)=\\(.*\\)/  "\\1": "\\2",/p' config.txt

echo ""
echo "5. Numéroter les lignes non vides :"
sed '/^$/!=' config.txt | sed 'N; s/\\n/: /'
```

### Sed pour le traitement de données

```bash
#!/bin/bash

echo "=== Sed pour le traitement de données ==="

# Créer un fichier CSV
cat > donnees.csv << 'EOF'
nom,prenom,age,salaire
Dupont,Marie,25,35000
Martin,Pierre,30,42000
Bernard,Sophie,28,38000
Rousseau,Jean,35,45000
EOF

echo "Données CSV originales :"
cat donnees.csv
echo ""

echo "=== Transformations de données ==="

echo "1. Changer le délimiteur (virgule vers pipe) :"
sed 's/,/|/g' donnees.csv

echo ""
echo "2. Ajouter un préfixe aux salaires :"
sed 's/\\([0-9]*\\)$/\\1€/' donnees.csv

echo ""
echo "3. Extraire seulement les noms et prénoms :"
sed 's/^\\([^,]*\\),\\([^,]*\\),.*/\\1 \\2/' donnees.csv

echo ""
echo "4. Mettre les noms en majuscules :"
sed 's/^\\([^,]*\\)/\\U\\1\\E/' donnees.csv

echo ""
echo "5. Filtrer et transformer (âge > 30) :"
sed -n '/^[^,]*,[^,]*,3[1-9]/s/^\\([^,]*\\),\\([^,]*\\),\\([^,]*\\),\\(.*\\)/\\1 \\2 (\\3 ans)/p' donnees.csv
```

### Fonction utilitaire avec sed

```bash
#!/bin/bash

# Fonction de nettoyage de fichier
nettoyer_fichier() {
    local fichier="$1"
    local mode="${2:-basique}"

    if [ ! -f "$fichier" ]; then
        echo "❌ Fichier '$fichier' introuvable"
        return 1
    fi

    echo "Nettoyage de '$fichier' (mode: $mode)"

    case "$mode" in
        "basique")
            # Supprimer les lignes vides et les espaces en fin de ligne
            sed -e '/^$/d' -e 's/[[:space:]]*$//' "$fichier"
            ;;
        "commentaires")
            # Supprimer les commentaires et lignes vides
            sed -e '/^[[:space:]]*#/d' -e '/^$/d' -e 's/[[:space:]]*$//' "$fichier"
            ;;
        "emails")
            # Extraire et valider les emails
            sed -n 's/.*\\([a-zA-Z0-9._%+-]*@[a-zA-Z0-9.-]*\\.[a-zA-Z]\\{2,\\}\\).*/\\1/p' "$fichier"
            ;;
        "anonymiser")
            # Anonymiser les données sensibles
            sed -e 's/[a-zA-Z0-9._%+-]*@[a-zA-Z0-9.-]*\\.[a-zA-Z]\\{2,\\}/***@***.com/g' \
                -e 's/0[1-9][0-9]\\{8\\}/0X.XX.XX.XX.XX/g' \
                "$fichier"
            ;;
    esac
}

# Créer un fichier de test pour le nettoyage
cat > fichier_sale.txt << 'EOF'
# Fichier de test
marie.dupont@example.com
pierre.martin@test.org

# Commentaire à supprimer
06.12.34.56.78
# Autre commentaire
Contact: jean@company.fr
Téléphone: 01.23.45.67.89

EOF

echo "=== Tests de nettoyage ==="

echo "Fichier original :"
cat fichier_sale.txt

echo ""
echo "1. Nettoyage basique :"
nettoyer_fichier "fichier_sale.txt" "basique"

echo ""
echo "2. Suppression des commentaires :"
nettoyer_fichier "fichier_sale.txt" "commentaires"

echo ""
echo "3. Extraction d'emails :"
nettoyer_fichier "fichier_sale.txt" "emails"

echo ""
echo "4. Anonymisation :"
nettoyer_fichier "fichier_sale.txt" "anonymiser"

# Nettoyer
rm -f texte_original.txt config.txt donnees.csv fichier_sale.txt
```

## Introduction et utilisation de awk

### Qu'est-ce que awk ?

`awk` est un langage de programmation spécialisé dans le **traitement de données structurées**. C'est comme avoir un tableur en ligne de commande qui peut calculer, trier, et analyser des données automatiquement.

**Analogie :** Si sed est un correcteur automatique, awk est comme un comptable qui peut lire des tableaux, faire des calculs, et produire des rapports.

### Structure de base d'awk

```bash
#!/bin/bash

echo "=== Introduction à awk ==="

# Créer un fichier de données
cat > ventes.txt << 'EOF'
Produit Quantite Prix_unitaire
Ordinateur 5 800
Souris 20 25
Clavier 15 45
Ecran 8 300
Imprimante 3 150
EOF

echo "Données de ventes :"
cat ventes.txt
echo ""

echo "=== Structure de base d'awk ==="

echo "1. Afficher tout le fichier :"
awk '{print}' ventes.txt

echo ""
echo "2. Afficher seulement certaines colonnes :"
echo "Colonnes 1 et 3 (Produit et Prix) :"
awk '{print $1, $3}' ventes.txt

echo ""
echo "3. Afficher avec en-tête personnalisé :"
awk 'BEGIN {print "=== RAPPORT DE VENTES ==="} {print $1 ": " $3 "€"}' ventes.txt

echo ""
echo "4. Ignorer la première ligne (en-tête) :"
awk 'NR > 1 {print $1 ": " $2 " unités"}' ventes.txt

echo ""
echo "5. Calculs simples :"
echo "Total par produit (Quantité × Prix) :"
awk 'NR > 1 {total = $2 * $3; print $1 ": " total "€"}' ventes.txt

echo ""
echo "6. Condition avec if :"
echo "Produits chers (> 100€) :"
awk 'NR > 1 && $3 > 100 {print $1 " est cher: " $3 "€"}' ventes.txt
```

### Variables et calculs avec awk

```bash
#!/bin/bash

echo "=== Variables et calculs avec awk ==="

# Créer des données d'employés
cat > employes.txt << 'EOF'
Nom Prenom Age Salaire Departement
Marie Dupont 25 35000 IT
Pierre Martin 30 42000 Finance
Sophie Bernard 28 38000 IT
Jean Rousseau 35 45000 Finance
Claire Moreau 26 36000 IT
Paul Durand 40 60000 Direction
EOF

echo "Données des employés :"
cat employes.txt
echo ""

echo "=== Calculs et statistiques ==="

echo "1. Somme des salaires :"
awk 'NR > 1 {total += $4} END {print "Total salaires: " total "€"}' employes.txt

echo ""
echo "2. Salaire moyen :"
awk 'NR > 1 {total += $4; count++} END {print "Salaire moyen: " total/count "€"}' employes.txt

echo ""
echo "3. Statistiques par département :"
awk 'NR > 1 {
    dept[$5] += $4
    count[$5]++
}
END {
    for (d in dept) {
        print d ": " dept[d] "€ (moyenne: " dept[d]/count[d] "€)"
    }
}' employes.txt

echo ""
echo "4. Employés par tranche d'âge :"
awk 'NR > 1 {
    if ($3 < 30) young++
    else if ($3 < 40) middle++
    else senior++
}
END {
    print "< 30 ans: " young+0
    print "30-39 ans: " middle+0
    print ">= 40 ans: " senior+0
}' employes.txt

echo ""
echo "5. Rapport formaté :"
awk 'BEGIN {
    print "╔════════════════════════════════════════════════════════════╗"
    print "║                    RAPPORT EMPLOYÉS                       ║"
    print "╠════════════════════════════════════════════════════════════╣"
    printf "║ %-15s %-10s %-5s %-8s %-12s ║\n", "NOM", "PRENOM", "AGE", "SALAIRE", "DEPT"
    print "╠════════════════════════════════════════════════════════════╣"
}
NR > 1 {
    printf "║ %-15s %-10s %-5d %-8d %-12s ║\n", $1, $2, $3, $4, $5
}
END {
    print "╚════════════════════════════════════════════════════════════╝"
}' employes.txt
```

### Traitement de fichiers CSV avec awk

```bash
#!/bin/bash

echo "=== Traitement de fichiers CSV avec awk ==="

# Créer un fichier CSV complexe
cat > commandes.csv << 'EOF'
id_commande,date,client,produit,quantite,prix_unitaire,status
1001,2025-01-15,Entreprise A,Ordinateur,5,800,Livree
1002,2025-01-16,Particulier B,Souris,10,25,En_cours
1003,2025-01-17,Entreprise C,Clavier,20,45,Livree
1004,2025-01-18,Entreprise A,Ecran,3,300,Annulee
1005,2025-01-19,Particulier D,Imprimante,1,150,Livree
1006,2025-01-20,Entreprise C,Ordinateur,2,800,En_cours
EOF

echo "Fichier CSV des commandes :"
cat commandes.csv
echo ""

echo "=== Analyse du fichier CSV ==="

echo "1. Chiffre d'affaires total des commandes livrées :"
awk -F',' 'NR > 1 && $7 == "Livree" {
    total += $5 * $6
    count++
}
END {
    print "CA: " total "€ (" count " commandes)"
}' commandes.csv

echo ""
echo "2. Commandes par client :"
awk -F',' 'NR > 1 {
    client[$3]++
    ca_client[$3] += $5 * $6
}
END {
    print "Analyse par client:"
    for (c in client) {
        print "  " c ": " client[c] " commande(s), " ca_client[c] "€"
    }
}' commandes.csv

echo ""
echo "3. Produits les plus vendus :"
awk -F',' 'NR > 1 {
    produit[$4] += $5
}
END {
    print "Quantités vendues par produit:"
    for (p in produit) {
        print "  " p ": " produit[p] " unités"
    }
}' commandes.csv

echo ""
echo "4. Rapport mensuel (simulation) :"
awk -F',' 'BEGIN {
    print "=== RAPPORT MENSUEL ==="
    print ""
}
NR > 1 {
    if ($7 == "Livree") {
        livrees++
        ca_livre += $5 * $6
    } else if ($7 == "En_cours") {
        en_cours++
        ca_attente += $5 * $6
    } else {
        annulees++
    }
    total_commandes++
}
END {
    print "Commandes livrées: " livrees+0 " (" ca_livre+0 "€)"
    print "Commandes en cours: " en_cours+0 " (" ca_attente+0 "€)"
    print "Commandes annulées: " annulees+0
    print "Total commandes: " total_commandes
    print "Taux de livraison: " int((livrees/total_commandes)*100) "%"
}' commandes.csv
```

### Fonction d'analyse avancée avec awk

```bash
#!/bin/bash

# Fonction d'analyse de données avec awk
analyser_donnees() {
    local fichier="$1"
    local delimiteur="${2:-,}"
    local type_analyse="${3:-stats}"

    if [ ! -f "$fichier" ]; then
        echo "❌ Fichier '$fichier' introuvable"
        return 1
    fi

    echo "Analyse de '$fichier' (délimiteur: '$delimiteur', type: $type_analyse)"
    echo ""

    case "$type_analyse" in
        "structure")
            awk -F"$delimiteur" '
            NR == 1 {
                print "=== STRUCTURE DU FICHIER ==="
                print "Nombre de colonnes: " NF
                print "En-têtes:"
                for(i=1; i<=NF; i++) {
                    print "  Colonne " i ": " $i
                }
                print ""
                next
            }
            {
                lignes++
                if (NF != nb_colonnes) {
                    if (nb_colonnes == 0) nb_colonnes = NF
                    else erreurs++
                }
            }
            END {
                print "Nombre de lignes de données: " lignes
                if (erreurs > 0) {
                    print "⚠️  " erreurs " ligne(s) avec un nombre incorrect de colonnes"
                }
            }' "$fichier"
            ;;
        "stats")
            awk -F"$delimiteur" '
            NR == 1 {next}
            {
                lignes++
                for(i=1; i<=NF; i++) {
                    if ($i ~ /^[0-9]+$/) {
                        # Colonne numérique
                        sum[i] += $i
                        if (min[i] == "" || $i < min[i]) min[i] = $i
                        if (max[i] == "" || $i > max[i]) max[i] = $i
                        count[i]++
                    }
                }
            }
            END {
                print "=== STATISTIQUES ==="
                print "Total lignes: " lignes
                print ""
                for(i in sum) {
                    print "Colonne " i " (numérique):"
                    print "  Somme: " sum[i]
                    print "  Moyenne: " sum[i]/count[i]
                    print "  Min: " min[i]
                    print "  Max: " max[i]
                    print ""
                }
            }' "$fichier"
            ;;
        "resume")
            awk -F"$delimiteur" '
            NR == 1 {
                for(i=1; i<=NF; i++) headers[i] = $i
                next
            }
            {
                lignes++
                for(i=1; i<=NF; i++) {
                    valeurs[i][$i]++
                    if ($i != "") non_vides[i]++
                }
            }
            END {
                print "=== RÉSUMÉ PAR COLONNE ==="
                for(i=1; i<=length(headers); i++) {
                    print "Colonne " i " (" headers[i] "):"
                    print "  Valeurs non vides: " non_vides[i]+0 "/" lignes
                    print "  Valeurs uniques: " length(valeurs[i])

                    # Top 3 des valeurs les plus fréquentes
                    print "  Top 3 valeurs:"
                    count = 0
                    for(val in valeurs[i]) {
                        if (++count <= 3) {
                            print "    " val ": " valeurs[i][val] " fois"
                        }
                    }
                    print ""
                }
            }' "$fichier"
            ;;
    esac
}

# Tests de la fonction d'analyse
echo "=== Tests d'analyse avancée ==="

echo "1. Analyse de structure :"
analyser_donnees "commandes.csv" "," "structure"

echo ""
echo "2. Statistiques :"
analyser_donnees "commandes.csv" "," "stats"

echo ""
echo "3. Résumé par colonne :"
analyser_donnees "commandes.csv" "," "resume"

# Nettoyer
rm -f ventes.txt employes.txt commandes.csv
```

## Tri et filtrage avec sort, uniq

### Introduction à sort

`sort` permet de **trier les lignes** d'un fichier ou d'un flux de données. C'est comme avoir un assistant qui peut ranger vos documents dans l'ordre que vous voulez.

```bash
#!/bin/bash

echo "=== Introduction à sort ==="

# Créer des données à trier
cat > notes.txt << 'EOF'
Marie:18
Pierre:12
Sophie:15
Jean:20
Claire:16
Paul:14
Alice:19
Lucas:13
EOF

echo "Notes originales :"
cat notes.txt
echo ""

echo "=== Tri simple ==="

echo "1. Tri alphabétique (par défaut) :"
sort notes.txt

echo ""
echo "2. Tri numérique par note :"
sort -t':' -k2 -n notes.txt

echo ""
echo "3. Tri inverse (décroissant) :"
sort -t':' -k2 -nr notes.txt

echo ""
echo "4. Tri par nom seulement :"
sort -t':' -k1 notes.txt

echo ""

# Données plus complexes
cat > employes_complet.txt << 'EOF'
Dupont:Marie:25:35000:IT
Martin:Pierre:30:42000:Finance
Bernard:Sophie:28:38000:IT
Rousseau:Jean:35:45000:Finance
Moreau:Claire:26:36000:IT
Durand:Paul:40:60000:Direction
EOF

echo "Données employés :"
cat employes_complet.txt
echo ""

echo "=== Tri multi-critères ==="

echo "1. Tri par département puis par salaire :"
sort -t':' -k5,5 -k4,4nr employes_complet.txt

echo ""
echo "2. Tri par âge puis par nom :"
sort -t':' -k3,3n -k2,2 employes_complet.txt

echo ""
echo "3. Tri par salaire (ordre décroissant) :"
sort -t':' -k4,4nr employes_complet.txt
```

### Options avancées de sort

```bash
#!/bin/bash

echo "=== Options avancées de sort ==="

# Créer un fichier avec des données variées
cat > donnees_variees.txt << 'EOF'
Article A:10.50:2025-01-15
Article B:125.00:2025-01-10
Article C:25.75:2025-01-20
Article D:8.25:2025-01-12
Article E:200.00:2025-01-18
EOF

echo "Données variées :"
cat donnees_variees.txt
echo ""

echo "=== Tri par types ==="

echo "1. Tri numérique par prix :"
sort -t':' -k2 -n donnees_variees.txt

echo ""
echo "2. Tri par date :"
sort -t':' -k3 donnees_variees.txt

echo ""
echo "3. Tri par prix en version humaine :"
sort -t':' -k2 -h donnees_variees.txt 2>/dev/null || sort -t':' -k2 -n donnees_variees.txt

echo ""

# Fichier avec doublons
cat > avec_doublons.txt << 'EOF'
pomme
banane
orange
pomme
kiwi
banane
mangue
orange
pomme
EOF

echo "Fichier avec doublons :"
cat avec_doublons.txt
echo ""

echo "=== Gestion des doublons ==="

echo "1. Tri simple (garde les doublons) :"
sort avec_doublons.txt

echo ""
echo "2. Tri avec suppression des doublons (-u) :"
sort -u avec_doublons.txt

echo ""
echo "3. Compter les occurrences :"
sort avec_doublons.txt | uniq -c

echo ""
echo "4. Afficher seulement les doublons :"
sort avec_doublons.txt | uniq -d
```

### Introduction à uniq

```bash
#!/bin/bash

echo "=== Introduction à uniq ==="

# Créer un fichier de logs
cat > logs.txt << 'EOF'
2025-07-13 10:15:23 [INFO] User login
2025-07-13 10:15:24 [INFO] User login
2025-07-13 10:15:25 [ERROR] Database error
2025-07-13 10:15:26 [INFO] User login
2025-07-13 10:15:27 [ERROR] Database error
2025-07-13 10:15:28 [WARN] High memory
2025-07-13 10:15:29 [INFO] User logout
2025-07-13 10:15:30 [ERROR] Database error
EOF

echo "Fichier de logs :"
cat logs.txt
echo ""

echo "=== Utilisation de uniq ==="

echo "1. Supprimer les doublons consécutifs :"
uniq logs.txt

echo ""
echo "2. Compter les occurrences :"
uniq -c logs.txt

echo ""
echo "3. Analyser par type de message :"
echo "Types de messages et leurs fréquences :"
cut -d']' -f1 logs.txt | cut -d'[' -f2 | sort | uniq -c

echo ""
echo "4. Messages uniques seulement :"
uniq -u logs.txt

echo ""
echo "5. Messages répétés seulement :"
uniq -d logs.txt

echo ""

# Analyse avancée
echo "=== Analyse statistique des logs ==="

echo "Analyse complète :"
awk '{
    # Extraire le niveau
    match($0, /\[([A-Z]+)\]/, arr)
    niveau = arr[1]
    niveaux[niveau]++
    total++

    # Extraire l'heure
    heure = substr($2, 1, 2)
    heures[heure]++
}
END {
    print "=== STATISTIQUES DES LOGS ==="
    print "Total messages: " total
    print ""
    print "Par niveau:"
    for (n in niveaux) {
        print "  " n ": " niveaux[n] " (" int(niveaux[n]/total*100) "%)"
    }
    print ""
    print "Par heure:"
    for (h in heures) {
        print "  " h "h: " heures[h] " message(s)"
    }
}' logs.txt
```

### Fonctions de tri et filtrage

```bash
#!/bin/bash

# Fonction de tri intelligent
tri_intelligent() {
    local fichier="$1"
    local critere="${2:-alpha}"  # alpha, num, date, taille
    local ordre="${3:-asc}"      # asc, desc

    if [ ! -f "$fichier" ]; then
        echo "❌ Fichier '$fichier' introuvable"
        return 1
    fi

    echo "Tri de '$fichier' (critère: $critere, ordre: $ordre)"

    local options=""

    # Déterminer les options de tri
    case "$critere" in
        "alpha")
            options=""
            ;;
        "num")
            options="-n"
            ;;
        "date")
            options="-k1,1"  # Suppose que la date est en première colonne
            ;;
        "taille")
            options="-h"
            ;;
    esac

    # Ajouter l'ordre
    if [ "$ordre" = "desc" ]; then
        options="$options -r"
    fi

    # Effectuer le tri
    sort $options "$fichier"
}

# Fonction d'analyse de doublons
analyser_doublons() {
    local fichier="$1"

    if [ ! -f "$fichier" ]; then
        echo "❌ Fichier '$fichier' introuvable"
        return 1
    fi

    echo "=== Analyse des doublons dans '$fichier' ==="

    local total_lignes=$(wc -l < "$fichier")
    local lignes_uniques=$(sort "$fichier" | uniq | wc -l)
    local doublons=$((total_lignes - lignes_uniques))

    echo "Total lignes: $total_lignes"
    echo "Lignes uniques: $lignes_uniques"
    echo "Doublons: $doublons"

    if [ $doublons -gt 0 ]; then
        echo ""
        echo "Lignes dupliquées :"
        sort "$fichier" | uniq -d | nl

        echo ""
        echo "Fréquence des doublons :"
        sort "$fichier" | uniq -c | sort -nr | head -5
    else
        echo "✅ Aucun doublon détecté"
    fi
}

# Tests des fonctions
echo "=== Tests des fonctions de tri ==="

echo "1. Tri alphabétique croissant :"
tri_intelligent "avec_doublons.txt" "alpha" "asc"

echo ""
echo "2. Tri alphabétique décroissant :"
tri_intelligent "avec_doublons.txt" "alpha" "desc"

echo ""
echo "3. Analyse des doublons :"
analyser_doublons "avec_doublons.txt"

# Nettoyer
rm -f notes.txt employes_complet.txt donnees_variees.txt avec_doublons.txt logs.txt
```

## Traitement de fichiers CSV/logs

### Traitement de fichiers CSV

```bash
#!/bin/bash

echo "=== Traitement spécialisé de fichiers CSV ==="

# Créer un fichier CSV complexe
cat > ventes_2025.csv << 'EOF'
date,vendeur,produit,quantite,prix_unitaire,region,client_type
2025-01-15,Marie,Ordinateur,2,800,Nord,Entreprise
2025-01-16,Pierre,Souris,50,25,Sud,Particulier
2025-01-17,Sophie,Clavier,30,45,Est,Entreprise
2025-01-18,Marie,Ecran,5,300,Nord,Entreprise
2025-01-19,Lucas,Imprimante,8,150,Ouest,Particulier
2025-01-20,Pierre,Ordinateur,3,800,Sud,Entreprise
2025-01-21,Sophie,Souris,25,25,Est,Particulier
2025-01-22,Marie,Clavier,15,45,Nord,Particulier
2025-01-23,Lucas,Ecran,2,300,Ouest,Entreprise
2025-01-24,Pierre,Imprimante,4,150,Sud,Particulier
EOF

echo "Fichier CSV des ventes :"
cat ventes_2025.csv
echo ""

echo "=== Analyses CSV ==="

echo "1. Chiffre d'affaires total :"
awk -F',' 'NR > 1 {ca += $4 * $5} END {print "CA Total: " ca "€"}' ventes_2025.csv

echo ""
echo "2. Top vendeurs :"
awk -F',' 'NR > 1 {
    vendeur[$2] += $4 * $5
    ventes[$2] += $4
} END {
    print "Performance par vendeur:"
    for (v in vendeur) {
        print "  " v ": " vendeur[v] "€ (" ventes[v] " unités)"
    }
}' ventes_2025.csv | sort -k2 -nr

echo ""
echo "3. Analyse par région :"
awk -F',' 'NR > 1 {
    region[$6] += $4 * $5
    clients[$6][$7]++
} END {
    print "Chiffre d'affaires par région:"
    for (r in region) {
        print "  " r ": " region[r] "€"
        for (type in clients[r]) {
            print "    " type ": " clients[r][type] " vente(s)"
        }
    }
}' ventes_2025.csv

echo ""
echo "4. Produits les plus vendus :"
awk -F',' 'NR > 1 {produits[$3] += $4} END {
    print "Quantités vendues:"
    for (p in produits) print produits[p], p
}' ventes_2025.csv | sort -nr

echo ""
echo "5. Évolution temporelle (par semaine) :"
awk -F',' 'NR > 1 {
    # Extraire la semaine (simplifié)
    split($1, date, "-")
    jour = date[3]
    if (jour <= 7) semaine = "Semaine 1"
    else if (jour <= 14) semaine = "Semaine 2"
    else if (jour <= 21) semaine = "Semaine 3"
    else semaine = "Semaine 4"

    ca_semaine[semaine] += $4 * $5
} END {
    print "CA par semaine:"
    for (s in ca_semaine) {
        print "  " s ": " ca_semaine[s] "€"
    }
}' ventes_2025.csv
```

### Traitement de fichiers de logs

```bash
#!/bin/bash

echo "=== Traitement spécialisé de logs ==="

# Créer un fichier de log réaliste
cat > server.log << 'EOF'
2025-07-13 08:15:23 192.168.1.100 GET /index.html 200 1024 0.125
2025-07-13 08:15:24 192.168.1.101 POST /api/login 200 256 0.089
2025-07-13 08:15:25 192.168.1.102 GET /products 200 2048 0.234
2025-07-13 08:15:26 192.168.1.100 GET /cart 404 512 0.067
2025-07-13 08:15:27 192.168.1.103 POST /api/order 500 128 1.234
2025-07-13 08:15:28 192.168.1.104 GET /index.html 200 1024 0.098
2025-07-13 08:15:29 192.168.1.101 GET /profile 403 256 0.045
2025-07-13 08:15:30 192.168.1.105 POST /api/login 200 256 0.123
EOF

echo "Fichier de log du serveur :"
cat server.log
echo ""

echo "=== Analyses de logs ==="

echo "1. Statistiques par code de statut :"
awk '{codes[$5]++} END {
    print "Codes de statut:"
    for (code in codes) {
        status = (code == 200) ? "Succès" :
                 (code == 404) ? "Non trouvé" :
                 (code == 403) ? "Interdit" :
                 (code == 500) ? "Erreur serveur" : "Autre"
        print "  " code " (" status "): " codes[code]
    }
}' server.log

echo ""
echo "2. Top 5 des IPs les plus actives :"
awk '{ips[$3]++} END {
    for (ip in ips) print ips[ip], ip
}' server.log | sort -nr | head -5

echo ""
echo "3. Pages les plus demandées :"
awk '{pages[$4]++} END {
    for (page in pages) print pages[page], page
}' server.log | sort -nr

echo ""
echo "4. Analyse des performances :"
awk '{
    temps_total += $7
    taille_total += $6
    requetes++

    if ($7 > 1.0) lentes++
    if ($5 >= 400) erreurs++
} END {
    print "=== PERFORMANCES ==="
    print "Requêtes totales: " requetes
    print "Temps moyen: " temps_total/requetes " sec"
    print "Taille moyenne: " int(taille_total/requetes) " octets"
    print "Requêtes lentes (>1s): " lentes+0
    print "Taux d'erreur: " int((erreurs/requetes)*100) "%"
}' server.log

echo ""
echo "5. Analyse par tranche horaire :"
awk '{
    # Extraire l'heure et les minutes
    split($2, time, ":")
    heure = time[1]
    minute = time[2]

    # Grouper par tranches de 15 minutes
    tranche = int(minute/15) * 15
    periode = heure ":" sprintf("%02d", tranche)

    requetes_periode[periode]++
    if ($5 >= 400) erreurs_periode[periode]++
} END {
    print "Activité par période (15 min):"
    for (p in requetes_periode) {
        err = erreurs_periode[p] + 0
        print "  " p ": " requetes_periode[p] " req (" err " err)"
    }
}' server.log | sort
```

### Fonction universelle d'analyse de logs

```bash
#!/bin/bash

# Fonction d'analyse universelle de logs
analyser_logs() {
    local fichier="$1"
    local format="${2:-auto}"  # auto, apache, nginx, custom
    local periode="${3:-hour}" # hour, day, minute

    if [ ! -f "$fichier" ]; then
        echo "❌ Fichier de log '$fichier' introuvable"
        return 1
    fi

    echo "=== Analyse du fichier de log '$fichier' ==="
    echo "Format: $format, Période: $periode"
    echo ""

    # Détection automatique du format
    if [ "$format" = "auto" ]; then
        if grep -q "^\[" "$fichier"; then
            format="apache"
        elif grep -q " - - \[" "$fichier"; then
            format="nginx"
        else
            format="simple"
        fi
        echo "Format détecté: $format"
    fi

    # Analyse selon le format
    case "$format" in
        "apache"|"nginx")
            # Format Apache/Nginx standard
            awk '{
                # Extraction des éléments principaux
                ip = $1
                date = $4
                method = $6
                url = $7
                status = $9
                size = $10

                # Statistiques
                ips[ip]++
                status_codes[status]++
                total_size += size
                requests++

                if (status >= 400) errors++

                # Extraction de l heure
                gsub(/\[/, "", date)
                split(date, d, ":")
                hour = d[2]
                hours[hour]++
            }
            END {
                print "=== STATISTIQUES GLOBALES ==="
                print "Total requêtes: " requests
                print "Taille totale: " total_size " octets"
                print "Erreurs: " errors " (" int(errors/requests*100) "%)"
                print ""

                print "=== TOP 5 IPs ==="
                for (ip in ips) print ips[ip], ip
                print "" | "sort -nr | head -5"
                close("sort -nr | head -5")

                print "=== CODES DE STATUT ==="
                for (code in status_codes) {
                    print "  " code ": " status_codes[code]
                }
            }' "$fichier"
            ;;
        "simple")
            # Format simple (timestamp ip action status)
            awk '{
                # Variables selon la position
                timestamp = $1 " " $2
                ip = $3
                action = $4
                status = $5

                # Statistiques
                ips[ip]++
                actions[action]++
                if (status >= 400) errors++
                requests++

                # Groupement temporel
                split($2, time, ":")
                hour = time[1]
                hours[hour]++
            }
            END {
                print "=== ANALYSE SIMPLE ==="
                print "Total requêtes: " requests
                print "Erreurs: " errors+0
                print ""

                print "Actions les plus fréquentes:"
                for (action in actions) {
                    print "  " action ": " actions[action]
                }
                print ""

                print "Activité par heure:"
                for (hour in hours) {
                    print "  " hour "h: " hours[hour] " requêtes"
                }
            }' "$fichier" | sort
            ;;
    esac
}

# Fonction de nettoyage de logs
nettoyer_logs() {
    local fichier="$1"
    local age_jours="${2:-30}"
    local taille_max="${3:-100M}"

    echo "Nettoyage des logs : $fichier"
    echo "Critères : > $age_jours jours ou > $taille_max"

    if [ ! -f "$fichier" ]; then
        echo "❌ Fichier introuvable"
        return 1
    fi

    # Vérifier l'âge
    local age_fichier=$(find "$fichier" -mtime +$age_jours)
    local taille_fichier=$(stat -c%s "$fichier" 2>/dev/null || echo "0")

    # Convertir la taille max en octets (simplifié)
    local taille_max_bytes
    case "$taille_max" in
        *M) taille_max_bytes=$((${taille_max%M} * 1024 * 1024)) ;;
        *K) taille_max_bytes=$((${taille_max%K} * 1024)) ;;
        *) taille_max_bytes="$taille_max" ;;
    esac

    if [ -n "$age_fichier" ] || [ "$taille_fichier" -gt "$taille_max_bytes" ]; then
        echo "📦 Création d'une archive..."
        gzip -c "$fichier" > "${fichier}.$(date +%Y%m%d).gz"

        echo "🧹 Nettoyage du fichier original..."
        > "$fichier"  # Vider le fichier sans le supprimer

        echo "✅ Nettoyage terminé"
        echo "Archive créée : ${fichier}.$(date +%Y%m%d).gz"
    else
        echo "ℹ️  Aucun nettoyage nécessaire"
    fi
}

# Tests des fonctions
echo "=== Tests d'analyse de logs ==="

analyser_logs "server.log" "simple"

echo ""
echo "=== Test de nettoyage (simulation) ==="
# nettoyer_logs "server.log" "0" "1K"  # Décommentez pour tester

# Nettoyer
rm -f ventes_2025.csv server.log
```

## Pipes et chaînage de commandes

### Introduction aux pipes

Les **pipes** (tuyaux) permettent de **connecter des commandes** entre elles. C'est comme construire une chaîne de montage où chaque outil fait sa partie du travail.

**Analogie :** Imaginez une usine avec plusieurs machines :
1. Machine A produit des pièces
2. Machine B les nettoie
3. Machine C les peint
4. Machine D les emballe

Les pipes font la même chose avec les données texte.

```bash
#!/bin/bash

echo "=== Introduction aux pipes ==="

# Créer des données pour les exemples
cat > donnees_demo.txt << 'EOF'
Alice,25,Paris,Développeuse
Bob,30,Lyon,Designer
Charlie,28,Paris,Développeur
Diana,32,Marseille,Manager
Eve,26,Lyon,Analyste
Frank,35,Paris,Architecte
Grace,29,Toulouse,Développeuse
Henry,31,Bordeaux,Manager
EOF

echo "Données de démonstration :"
cat donnees_demo.txt
echo ""

echo "=== Exemples de pipes simples ==="

echo "1. Compter les lignes :"
cat donnees_demo.txt | wc -l

echo ""
echo "2. Trier et afficher :"
cat donnees_demo.txt | sort

echo ""
echo "3. Chercher et compter :"
echo "Nombre de développeurs :"
cat donnees_demo.txt | grep -i "développeu" | wc -l

echo ""
echo "4. Extraire et trier les villes :"
cat donnees_demo.txt | cut -d',' -f3 | sort | uniq

echo ""
echo "5. Pipeline complexe - Analyse des âges :"
echo "Âge moyen :"
cat donnees_demo.txt | cut -d',' -f2 | awk '{sum += $1; count++} END {print sum/count " ans"}'
```

### Chaînages complexes

```bash
#!/bin/bash

echo "=== Chaînages complexes ==="

# Créer un fichier de log plus détaillé
cat > access.log << 'EOF'
192.168.1.100 - - [13/Jul/2025:08:15:23 +0000] "GET /index.html HTTP/1.1" 200 1024
192.168.1.101 - - [13/Jul/2025:08:15:24 +0000] "POST /api/login HTTP/1.1" 200 256
192.168.1.102 - - [13/Jul/2025:08:15:25 +0000] "GET /products HTTP/1.1" 200 2048
192.168.1.100 - - [13/Jul/2025:08:15:26 +0000] "GET /cart HTTP/1.1" 404 512
192.168.1.103 - - [13/Jul/2025:08:15:27 +0000] "POST /api/order HTTP/1.1" 500 128
192.168.1.104 - - [13/Jul/2025:08:15:28 +0000] "GET /index.html HTTP/1.1" 200 1024
192.168.1.101 - - [13/Jul/2025:08:15:29 +0000] "GET /profile HTTP/1.1" 403 256
192.168.1.105 - - [13/Jul/2025:08:15:30 +0000] "POST /api/login HTTP/1.1" 200 256
EOF

echo "=== Analyses avec chaînages complexes ==="

echo "1. Top 3 des IPs les plus actives :"
cat access.log | \
    awk '{print $1}' | \
    sort | \
    uniq -c | \
    sort -nr | \
    head -3

echo ""
echo "2. Pages les plus demandées (GET seulement) :"
cat access.log | \
    grep '"GET ' | \
    awk '{print $7}' | \
    sort | \
    uniq -c | \
    sort -nr

echo ""
echo "3. Analyse des erreurs :"
cat access.log | \
    awk '$9 >= 400 {print $9, $7}' | \
    sort | \
    uniq -c | \
    sort -nr

echo ""
echo "4. Taille totale des réponses par IP :"
cat access.log | \
    awk '{ips[$1] += $10} END {for(ip in ips) print ip, ips[ip]}' | \
    sort -k2 -nr

echo ""
echo "5. Rapport complet en une ligne :"
cat access.log | \
    awk '{
        total++;
        ips[$1]++;
        sizes += $10;
        if($9 >= 400) errors++
    } END {
        print "Total:", total, "req |",
              "IPs uniques:", length(ips), "|",
              "Taille moy:", int(sizes/total), "octets |",
              "Erreurs:", errors+0
    }'
```

### Techniques avancées de chaînage

```bash
#!/bin/bash

echo "=== Techniques avancées de chaînage ==="

echo "=== 1. Utilisation de tee pour dupliquer le flux ==="
cat donnees_demo.txt | \
    tee >(echo "Sauvegarde dans fichier...") | \
    grep "Paris" | \
    wc -l

echo ""
echo "=== 2. Traitement en parallèle ==="
# Analyser les données sous plusieurs angles simultanément
(
    echo "=== Analyse par ville ==="
    cat donnees_demo.txt | cut -d',' -f3 | sort | uniq -c
) &

(
    echo "=== Analyse par profession ==="
    cat donnees_demo.txt | cut -d',' -f4 | sort | uniq -c
) &

(
    echo "=== Statistiques d'âge ==="
    cat donnees_demo.txt | cut -d',' -f2 | awk '{
        sum += $1;
        if(min == 0 || $1 < min) min = $1;
        if($1 > max) max = $1;
        count++
    } END {
        print "Âge min:", min, "- max:", max, "- moyen:", int(sum/count)
    }'
) &

wait  # Attendre que tous les processus se terminent

echo ""
echo "=== 3. Chaînage conditionnel ==="
# Exécuter des commandes selon le résultat
cat donnees_demo.txt | \
    grep "Manager" | \
    wc -l | \
    awk '{
        if ($1 > 1)
            print "Plusieurs managers trouvés (" $1 ")"
        else if ($1 == 1)
            print "Un seul manager trouvé"
        else
            print "Aucun manager trouvé"
    }'

echo ""
echo "=== 4. Pipeline avec validation ==="
# Vérifier les données à chaque étape
cat donnees_demo.txt | \
    awk -F',' 'NF == 4 {print}' | \
    tee >(wc -l | awk '{print "Lignes valides:", $1}' >&2) | \
    awk -F',' '$2 >= 25 && $2 <= 40 {print}' | \
    tee >(wc -l | awk '{print "Dans la tranche d'\''âge:", $1}' >&2) | \
    sort -t',' -k2,2n

echo ""
echo "=== 5. Agrégation complexe ==="
# Rapport détaillé avec plusieurs niveaux d'agrégation
cat donnees_demo.txt | \
    awk -F',' '{
        # Groupement par ville
        ville[$3]++
        age_ville[$3] += $2

        # Groupement par profession
        prof[$4]++
        age_prof[$4] += $2

        # Statistiques globales
        total++
        age_total += $2
    } END {
        print "=== RAPPORT COMPLET ==="
        print "Total personnes:", total
        print "Âge moyen global:", int(age_total/total), "ans"
        print ""

        print "Par ville:"
        for (v in ville) {
            print "  " v ":", ville[v], "pers., âge moy:", int(age_ville[v]/ville[v])
        }

        print ""
        print "Par profession:"
        for (p in prof) {
            print "  " p ":", prof[p], "pers., âge moy:", int(age_prof[p]/prof[p])
        }
    }'
```

### Optimisation des pipelines

```bash
#!/bin/bash

echo "=== Optimisation des pipelines ==="

# Créer un gros fichier de test pour les performances
echo "Création d'un fichier de test..."
seq 1 10000 | awk '{
    villes[1]="Paris"; villes[2]="Lyon"; villes[3]="Marseille"
    profs[1]="Dev"; profs[2]="Designer"; profs[3]="Manager"
    print "User" $1 "," (20 + $1 % 20) "," villes[($1 % 3) + 1] "," profs[($1 % 3) + 1]
}' > gros_fichier.txt

echo "Fichier créé : $(wc -l < gros_fichier.txt) lignes"

echo ""
echo "=== Comparaison de performances ==="

echo "1. Pipeline optimisé (une seule lecture) :"
time (
    awk -F',' '{
        total++
        villes[$3]++
        ages += $2
        if ($4 == "Dev") devs++
    } END {
        print "Total:", total
        print "Développeurs:", devs+0
        print "Âge moyen:", int(ages/total)
        print "Villes:", length(villes)
    }' gros_fichier.txt > /dev/null
)

echo ""
echo "2. Pipeline non optimisé (lectures multiples) :"
time (
    total=$(wc -l < gros_fichier.txt)
    devs=$(grep "Dev" gros_fichier.txt | wc -l)
    age_moy=$(cut -d',' -f2 gros_fichier.txt | awk '{s+=$1}END{print int(s/NR)}')
    villes=$(cut -d',' -f3 gros_fichier.txt | sort | uniq | wc -l)

    echo "Total: $total" > /dev/null
    echo "Développeurs: $devs" > /dev/null
    echo "Âge moyen: $age_moy" > /dev/null
    echo "Villes: $villes" > /dev/null
)

echo ""
echo "=== Techniques d'optimisation ==="

echo "1. Utiliser awk pour les calculs complexes (une seule passe)"
echo "2. Éviter les boucles sur de gros fichiers"
echo "3. Utiliser sort -u au lieu de sort | uniq"
echo "4. Filtrer tôt dans le pipeline"
echo "5. Utiliser des outils spécialisés quand disponibles"

# Exemple d'optimisation
echo ""
echo "Exemple - Compter les lignes uniques :"
echo "Lent : sort fichier | uniq | wc -l"
echo "Rapide : sort -u fichier | wc -l"

# Nettoyer
rm -f gros_fichier.txt
```

### Fonctions utilitaires de pipeline

```bash
#!/bin/bash

# Fonction de pipeline pour analyse de données
analyser_pipeline() {
    local fichier="$1"
    local delimiteur="${2:-,}"
    local colonnes="$3"  # ex: "1,3,5" pour colonnes 1, 3 et 5

    if [ ! -f "$fichier" ]; then
        echo "❌ Fichier '$fichier' introuvable"
        return 1
    fi

    echo "Analyse pipeline de '$fichier'"
    echo "Délimiteur: '$delimiteur'"
    echo "Colonnes: $colonnes"
    echo ""

    # Construire le pipeline dynamiquement
    if [ -n "$colonnes" ]; then
        # Extraire les colonnes spécifiées
        cut -d"$delimiteur" -f"$colonnes" "$fichier" | \
            awk -F"$delimiteur" '{
                for(i=1; i<=NF; i++) {
                    col[i][$i]++
                    if($i ~ /^[0-9]+$/) {
                        sum[i] += $i
                        count[i]++
                    }
                }
                total++
            } END {
                print "=== ANALYSE PAR COLONNE ==="
                for(i=1; i<=NF; i++) {
                    print "Colonne " i ":"
                    print "  Valeurs uniques:", length(col[i])
                    if(count[i] > 0) {
                        print "  Somme:", sum[i]
                        print "  Moyenne:", sum[i]/count[i]
                    }
                    print ""
                }
                print "Total lignes analysées:", total
            }'
    else
        # Analyse complète
        awk -F"$delimiteur" '{
            for(i=1; i<=NF; i++) {
                valeurs[i][$i]++
                if($i ~ /^[0-9]+(\.[0-9]+)?$/) {
                    numerique[i] = 1
                    sum[i] += $i
                    if(min[i] == "" || $i < min[i]) min[i] = $i
                    if(max[i] == "" || $i > max[i]) max[i] = $i
                    count_num[i]++
                }
            }
            lignes++
        } END {
            print "=== ANALYSE COMPLÈTE ==="
            print "Nombre de colonnes:", NF
            print "Nombre de lignes:", lignes
            print ""

            for(i=1; i<=NF; i++) {
                print "Colonne " i ":"
                print "  Valeurs uniques:", length(valeurs[i])
                if(numerique[i]) {
                    print "  Type: numérique"
                    print "  Min:", min[i]
                    print "  Max:", max[i]
                    print "  Moyenne:", sum[i]/count_num[i]
                } else {
                    print "  Type: texte"
                }
                print ""
            }
        }' "$fichier"
    fi
}

# Fonction de transformation de données
transformer_donnees() {
    local fichier="$1"
    local transformation="$2"  # uppercase, lowercase, normalize, anonymize

    case "$transformation" in
        "uppercase")
            awk '{print toupper($0)}' "$fichier"
            ;;
        "lowercase")
            awk '{print tolower($0)}' "$fichier"
            ;;
        "normalize")
            # Normaliser les espaces et la ponctuation
            sed 's/[[:space:]]\+/ /g' "$fichier" | \
            sed 's/^[[:space:]]*//;s/[[:space:]]*$//' | \
            tr '[:upper:]' '[:lower:]'
            ;;
        "anonymize")
            # Anonymiser les données sensibles
            sed -E 's/[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}/***@***.com/g' "$fichier" | \
            sed -E 's/0[1-9][0-9]{8}/0X.XX.XX.XX.XX/g' | \
            awk -F',' '{
                for(i=1; i<=NF; i++) {
                    if($i ~ /^[A-Za-z]+$/ && length($i) > 2) {
                        $i = substr($i,1,1) "***"
                    }
                }
                print
            }' OFS=','
            ;;
        "csv_to_json")
            # Conversion CSV vers JSON (simple)
            awk -F',' 'NR==1 {
                for(i=1; i<=NF; i++) headers[i] = $i
                print "["
                next
            } {
                printf "  {"
                for(i=1; i<=NF; i++) {
                    printf "\"%s\": \"%s\"", headers[i], $i
                    if(i < NF) printf ", "
                }
                printf "}"
                if(NR > 2) printf ","
                print ""
            } END {
                print "]"
            }' "$fichier"
            ;;
    esac
}

# Tests des fonctions utilitaires
echo "=== Tests des fonctions utilitaires ==="

echo "1. Analyse de colonnes spécifiques :"
analyser_pipeline "donnees_demo.txt" "," "1,2"

echo ""
echo "2. Transformation en majuscules :"
echo "Avant transformation :"
head -3 donnees_demo.txt
echo ""
echo "Après transformation :"
transformer_donnees "donnees_demo.txt" "uppercase" | head -3

echo ""
echo "3. Anonymisation :"
echo "Données anonymisées :"
transformer_donnees "donnees_demo.txt" "anonymize" | head -3

echo ""
echo "4. Conversion CSV vers JSON :"
echo "Format JSON :"
transformer_donnees "donnees_demo.txt" "csv_to_json"
```

### Pipeline de traitement en temps réel

```bash
#!/bin/bash

echo "=== Pipeline de traitement en temps réel ==="

# Simulateur de logs en temps réel
simuler_logs() {
    local duree="$1"
    local fin=$(($(date +%s) + duree))

    echo "Simulation de logs pendant $duree secondes..."

    while [ $(date +%s) -lt $fin ]; do
        # Générer une ligne de log aléatoire
        ip="192.168.1.$((RANDOM % 255))"
        status=$((RANDOM % 4 == 0 ? 404 : 200))
        size=$((RANDOM % 5000 + 100))
        timestamp=$(date '+%Y-%m-%d %H:%M:%S')

        echo "$timestamp $ip GET /page$((RANDOM % 10)).html $status $size"
        sleep 0.5
    done
}

# Pipeline de monitoring en temps réel
monitor_temps_reel() {
    local fenetre="${1:-10}"  # Fenêtre d'analyse en secondes

    echo "Monitoring en temps réel (fenêtre: ${fenetre}s)"
    echo "Appuyez sur Ctrl+C pour arrêter"
    echo ""

    # Créer un fichier temporaire pour la fenêtre glissante
    local temp_file="/tmp/monitor_$$"

    while read -r ligne; do
        echo "$ligne" >> "$temp_file"

        # Nettoyer les entrées trop anciennes
        local cutoff=$(date -d "$fenetre seconds ago" '+%Y-%m-%d %H:%M:%S' 2>/dev/null || date -v-${fenetre}S '+%Y-%m-%d %H:%M:%S' 2>/dev/null)

        if [ -n "$cutoff" ]; then
            grep "^$cutoff\|$(date '+%Y-%m-%d %H:%M')" "$temp_file" > "${temp_file}.new" 2>/dev/null
            mv "${temp_file}.new" "$temp_file" 2>/dev/null
        fi

        # Analyse de la fenêtre actuelle
        if [ -s "$temp_file" ]; then
            clear
            echo "=== MONITORING TEMPS RÉEL (dernières ${fenetre}s) ==="
            echo "Timestamp: $(date)"
            echo ""

            awk '{
                total++
                ips[$3]++
                if($5 >= 400) errors++
                pages[$4]++
            } END {
                print "Requêtes:", total+0
                print "IPs uniques:", length(ips)
                print "Erreurs:", errors+0
                if(total > 0) print "Taux erreur:", int(errors/total*100) "%"
                print ""

                print "Top IPs:"
                for(ip in ips) print "  " ip ":", ips[ip] | "sort -k3 -nr | head -3"
                close("sort -k3 -nr | head -3")

                print ""
                print "Pages populaires:"
                for(page in pages) print "  " page ":", pages[page] | "sort -k3 -nr | head -3"
                close("sort -k3 -nr | head -3")
            }' "$temp_file"
        fi
    done

    # Nettoyer
    rm -f "$temp_file" "${temp_file}.new" 2>/dev/null
}

# Fonction de benchmark de pipeline
benchmark_pipeline() {
    local fichier="$1"
    local nb_tests="${2:-5}"

    echo "Benchmark de différents pipelines sur '$fichier'"
    echo "Nombre de tests: $nb_tests"
    echo ""

    # Test 1: grep + wc
    echo "Test 1: grep + wc"
    for i in $(seq 1 $nb_tests); do
        time (grep "Paris" "$fichier" | wc -l > /dev/null) 2>&1 | grep real
    done | awk '{sum += $2} END {print "Temps moyen:", sum/NR "s"}'

    echo ""

    # Test 2: awk seul
    echo "Test 2: awk seul"
    for i in $(seq 1 $nb_tests); do
        time (awk '/Paris/ {count++} END {print count+0}' "$fichier" > /dev/null) 2>&1 | grep real
    done | awk '{sum += $2} END {print "Temps moyen:", sum/NR "s"}'

    echo ""

    # Test 3: sed + wc
    echo "Test 3: sed + wc"
    for i in $(seq 1 $nb_tests); do
        time (sed -n '/Paris/p' "$fichier" | wc -l > /dev/null) 2>&1 | grep real
    done | awk '{sum += $2} END {print "Temps moyen:", sum/NR "s"}'
}

# Démonstration du monitoring (simulation)
echo "Démonstration du pipeline de monitoring :"
echo "(Simulation de 5 secondes de logs)"

# Lancer le simulateur et le moniteur
(simuler_logs 5) | monitor_temps_reel 3 &
monitor_pid=$!

# Attendre et nettoyer
sleep 6
kill $monitor_pid 2>/dev/null
wait $monitor_pid 2>/dev/null

echo ""
echo "Simulation terminée"

# Nettoyer tous les fichiers de test
rm -f donnees_demo.txt access.log
```

## Récapitulatif

✅ **Ce que vous avez appris dans ce chapitre :**

1. **grep** : Recherche de motifs avec expressions régulières
2. **sed** : Transformation et édition de flux de texte
3. **awk** : Traitement de données structurées et calculs
4. **sort/uniq** : Tri et élimination de doublons
5. **Traitement CSV/logs** : Analyse de données spécialisées
6. **Pipes** : Chaînage de commandes pour créer des pipelines puissants

✅ **Points clés à retenir :**

**grep :**
```bash
grep "motif" fichier              # Recherche simple
grep -i "motif" fichier           # Insensible à la casse
grep -E "regex" fichier           # Expressions régulières étendues
grep -v "motif" fichier           # Inverser (lignes sans motif)
grep -A 2 -B 2 "motif" fichier    # Contexte avant/après
```

**sed :**
```bash
sed 's/ancien/nouveau/' fichier        # Remplacer première occurrence
sed 's/ancien/nouveau/g' fichier       # Remplacer toutes les occurrences
sed '/motif/d' fichier                 # Supprimer lignes contenant motif
sed -n '2,5p' fichier                  # Afficher lignes 2 à 5
```

**awk :**
```bash
awk '{print $1, $3}' fichier          # Colonnes 1 et 3
awk -F',' '{print $2}' fichier        # Délimiteur virgule
awk '$2 > 100 {print}' fichier        # Condition sur colonne 2
awk '{sum += $1} END {print sum}' fichier  # Somme de la colonne 1
```

**Pipes :**
```bash
commande1 | commande2 | commande3     # Pipeline simple
cmd | tee fichier | autre_cmd          # Dupliquer le flux
cmd | sort | uniq -c | sort -nr        # Pattern fréquent
```

✅ **Cas d'usage pratiques :**
- **Analyse de logs** : Extraire erreurs, compter requêtes, identifier tendances
- **Traitement CSV** : Calculs statistiques, regroupements, rapports
- **Monitoring temps réel** : Surveillance de systèmes avec pipelines
- **Nettoyage de données** : Normalisation, validation, anonymisation
- **Rapports automatiques** : Génération de statistiques et tableaux de bord

✅ **Optimisations importantes :**
- Utiliser awk pour une seule passe sur de gros fichiers
- Filtrer tôt dans le pipeline pour réduire les données
- Préférer `sort -u` à `sort | uniq`
- Éviter les boucles multiples sur le même fichier

✅ **Prochaines étapes :**
Dans le chapitre 10, nous approfondirons les expressions régulières !

---

💡 **Conseil pratique :** Maîtriser ces outils (grep, sed, awk) et savoir les combiner avec des pipes vous rendra extrêmement efficace pour traiter des données en ligne de commande. Pratiquez avec vos propres fichiers de logs ou données CSV !

⏭️
