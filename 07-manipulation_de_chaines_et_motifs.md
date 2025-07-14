🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Tutoriel Bash - Chapitre 7 : Manipulation de chaînes et motifs

## Opérations sur les chaînes

### Qu'est-ce que la manipulation de chaînes ?

La manipulation de chaînes, c'est comme **travailler avec du texte** : découper, coller, chercher, remplacer. C'est comme être un **éditeur de texte** avec des outils précis.

**Analogie :** Imaginez que vous avez une phrase écrite sur une bande de papier. Vous pouvez :
- Mesurer sa longueur
- Découper des morceaux
- Coller des bouts ensemble
- Chercher des mots dedans
- Remplacer des parties

### Longueur d'une chaîne

```bash
#!/bin/bash

# Différentes façons de mesurer la longueur
chaine="Bonjour le monde"

echo "=== Longueur d'une chaîne ==="
echo "Chaîne : '$chaine'"

# Méthode 1 : ${#variable}
longueur=${#chaine}
echo "Longueur avec \${#} : $longueur"

# Méthode 2 : wc (word count)
longueur_wc=$(echo -n "$chaine" | wc -c)
echo "Longueur avec wc : $longueur_wc"

# Méthode 3 : expr
longueur_expr=$(expr length "$chaine")
echo "Longueur avec expr : $longueur_expr"

echo ""

# Test avec différents types de chaînes
chaines=("" "a" "Hello" "Café français" "émojis 🚀🎉")
for test_chaine in "${chaines[@]}"; do
    echo "Longueur de '$test_chaine' : ${#test_chaine}"
done
```

### Concaténation (assembler des chaînes)

```bash
#!/bin/bash

echo "=== Concaténation de chaînes ==="

# Variables de base
prenom="Marie"
nom="Dupont"
age=25

# Méthode 1 : Juxtaposition simple
nom_complet="$prenom $nom"
echo "Nom complet : $nom_complet"

# Méthode 2 : Avec accolades pour plus de clarté
message="Bonjour ${prenom} ${nom}, vous avez ${age} ans."
echo "$message"

# Méthode 3 : Accumulation progressive
phrase="Aujourd'hui"
phrase+=" il fait"
phrase+=" beau"
phrase+=" dehors."
echo "Phrase construite : $phrase"

# Méthode 4 : Avec printf
message_formate=$(printf "Utilisateur: %s %s (âge: %d)" "$prenom" "$nom" "$age")
echo "Message formaté : $message_formate"

echo ""

# Exemple pratique : Construire un chemin de fichier
repertoire="/home/utilisateur"
sous_dossier="documents"
nom_fichier="rapport"
extension="txt"

chemin_complet="${repertoire}/${sous_dossier}/${nom_fichier}.${extension}"
echo "Chemin complet : $chemin_complet"
```

### Comparaison de chaînes

```bash
#!/bin/bash

echo "=== Comparaison de chaînes ==="

chaine1="Hello"
chaine2="hello"
chaine3="Hello"

# Comparaison exacte
if [ "$chaine1" = "$chaine3" ]; then
    echo "'$chaine1' est égal à '$chaine3'"
else
    echo "'$chaine1' n'est pas égal à '$chaine3'"
fi

# Comparaison insensible à la casse
if [ "${chaine1,,}" = "${chaine2,,}" ]; then
    echo "'$chaine1' et '$chaine2' sont égaux (insensible à la casse)"
fi

# Comparaison alphabétique
if [[ "$chaine1" < "$chaine2" ]]; then
    echo "'$chaine1' vient avant '$chaine2' alphabétiquement"
else
    echo "'$chaine1' vient après '$chaine2' alphabétiquement"
fi

echo ""

# Tests de chaînes vides et nulles
chaine_vide=""
chaine_nulle=

echo "=== Tests de chaînes vides ==="
if [ -z "$chaine_vide" ]; then
    echo "La chaîne vide est détectée comme vide"
fi

if [ -n "$chaine1" ]; then
    echo "'$chaine1' n'est pas vide"
fi

# Comparaison de longueurs
echo ""
echo "=== Comparaison de longueurs ==="
mots=("chat" "chien" "oiseau" "éléphant")
for mot in "${mots[@]}"; do
    if [ ${#mot} -gt 5 ]; then
        echo "'$mot' est un mot long (${#mot} caractères)"
    else
        echo "'$mot' est un mot court (${#mot} caractères)"
    fi
done
```

### Transformation de casse

```bash
#!/bin/bash

echo "=== Transformation de casse ==="

texte="Bonjour Le Monde"
echo "Texte original : '$texte'"

# Bash 4+ : Transformations intégrées
echo "Tout en minuscules : '${texte,,}'"
echo "Tout en majuscules : '${texte^^}'"
echo "Première lettre en majuscule : '${texte^}'"
echo "Première lettre de chaque mot : '${texte^^?( )}'"

echo ""

# Méthodes alternatives avec tr
echo "=== Avec la commande tr ==="
echo "Minuscules avec tr : '$(echo "$texte" | tr '[:upper:]' '[:lower:]')'"
echo "Majuscules avec tr : '$(echo "$texte" | tr '[:lower:]' '[:upper:]')'"

# Fonction pratique pour la casse
to_lower() {
    echo "${1,,}"
}

to_upper() {
    echo "${1^^}"
}

capitalize() {
    echo "${1^}"
}

echo ""
echo "=== Tests avec fonctions ==="
phrase="hello world from bash"
echo "Original : $phrase"
echo "Minuscules : $(to_lower "$phrase")"
echo "Majuscules : $(to_upper "$phrase")"
echo "Capitalisé : $(capitalize "$phrase")"
```

### Suppression d'espaces (trim)

```bash
#!/bin/bash

echo "=== Suppression d'espaces ==="

# Chaîne avec espaces en début et fin
chaine_avec_espaces="   Bonjour le monde   "
echo "Original : '$chaine_avec_espaces'"

# Suppression des espaces avec sed
chaine_trimmed=$(echo "$chaine_avec_espaces" | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')
echo "Trimmed avec sed : '$chaine_trimmed'"

# Suppression avec les expansions de paramètres
trim_spaces() {
    local var="$1"
    # Supprimer les espaces de début
    var="${var#"${var%%[![:space:]]*}"}"
    # Supprimer les espaces de fin
    var="${var%"${var##*[![:space:]]}"}"
    echo "$var"
}

echo "Trimmed avec fonction : '$(trim_spaces "$chaine_avec_espaces")'"

# Fonction complète de nettoyage
clean_string() {
    local chaine="$1"

    # Supprimer espaces début/fin
    chaine=$(trim_spaces "$chaine")

    # Remplacer espaces multiples par un seul
    chaine=$(echo "$chaine" | tr -s ' ')

    echo "$chaine"
}

chaine_sale="   Bonjour    le     monde   "
echo ""
echo "Chaîne sale : '$chaine_sale'"
echo "Chaîne nettoyée : '$(clean_string "$chaine_sale")'"
```

## Extraction de sous-chaînes

### Syntaxe de base d'extraction

```bash
#!/bin/bash

echo "=== Extraction de sous-chaînes ==="

phrase="Bonjour le monde merveilleux"
echo "Phrase originale : '$phrase'"
echo "Longueur : ${#phrase}"

# ${variable:position:longueur}
# Position commence à 0

echo ""
echo "=== Extractions par position ==="
echo "Premiers 7 caractères : '${phrase:0:7}'"
echo "À partir du caractère 8 : '${phrase:8}'"
echo "5 caractères à partir de la position 11 : '${phrase:11:5}'"
echo "Les 10 derniers caractères : '${phrase: -10}'"  # Espace important avant -
echo "Tout sauf les 5 derniers : '${phrase:0: -5}'"

echo ""
echo "=== Extractions relatives ==="
# Extraction depuis la fin
echo "Dernier caractère : '${phrase: -1}'"
echo "3 derniers caractères : '${phrase: -3}'"

# Extraction du milieu
longueur=${#phrase}
milieu=$((longueur / 2))
echo "Caractère du milieu : '${phrase:$milieu:1}'"
echo "5 caractères autour du milieu : '${phrase:$((milieu-2)):5}'"
```

### Fonctions d'extraction pratiques

```bash
#!/bin/bash

# Fonction pour obtenir le premier mot
premier_mot() {
    local chaine="$1"
    echo "${chaine%% *}"  # Supprime tout après le premier espace
}

# Fonction pour obtenir le dernier mot
dernier_mot() {
    local chaine="$1"
    echo "${chaine##* }"  # Supprime tout avant le dernier espace
}

# Fonction pour obtenir les initiales
obtenir_initiales() {
    local nom_complet="$1"
    local initiales=""

    # Diviser en mots et prendre la première lettre
    for mot in $nom_complet; do
        initiales+="${mot:0:1}"
    done

    echo "${initiales^^}"  # En majuscules
}

# Fonction pour extraire l'extension d'un fichier
obtenir_extension() {
    local nom_fichier="$1"
    echo "${nom_fichier##*.}"  # Tout après le dernier point
}

# Fonction pour obtenir le nom sans extension
obtenir_nom_base() {
    local nom_fichier="$1"
    echo "${nom_fichier%.*}"  # Tout avant le dernier point
}

# Tests des fonctions
echo "=== Tests d'extraction ==="

nom_complet="Marie Claire Dupont"
echo "Nom complet : $nom_complet"
echo "Premier mot : $(premier_mot "$nom_complet")"
echo "Dernier mot : $(dernier_mot "$nom_complet")"
echo "Initiales : $(obtenir_initiales "$nom_complet")"

echo ""

fichiers=("document.txt" "image.jpg" "script.sh" "archive.tar.gz" "README")
for fichier in "${fichiers[@]}"; do
    echo "Fichier : $fichier"
    echo "  Nom de base : $(obtenir_nom_base "$fichier")"
    echo "  Extension : $(obtenir_extension "$fichier")"
done
```

### Extraction avec délimiteurs

```bash
#!/bin/bash

echo "=== Extraction avec délimiteurs ==="

# Données séparées par des virgules (CSV)
ligne_csv="Marie,Dupont,25,Paris,Développeuse"
echo "Ligne CSV : $ligne_csv"

# Méthode 1 : Avec IFS (Internal Field Separator)
IFS=',' read -ra champs <<< "$ligne_csv"
echo "Extraction avec IFS :"
for i in "${!champs[@]}"; do
    echo "  Champ $i : '${champs[$i]}'"
done

# Méthode 2 : Avec cut
echo ""
echo "Extraction avec cut :"
echo "  Nom (champ 1) : $(echo "$ligne_csv" | cut -d',' -f1)"
echo "  Prénom (champ 2) : $(echo "$ligne_csv" | cut -d',' -f2)"
echo "  Âge (champ 3) : $(echo "$ligne_csv" | cut -d',' -f3)"
echo "  Champs 1-3 : $(echo "$ligne_csv" | cut -d',' -f1-3)"

# Fonction d'extraction générique
extraire_champ() {
    local donnees="$1"
    local delimiteur="$2"
    local numero_champ="$3"

    echo "$donnees" | cut -d"$delimiteur" -f"$numero_champ"
}

echo ""
echo "=== Autres délimiteurs ==="

# Données avec différents délimiteurs
donnees_pipe="nom|prenom|age|ville"
donnees_tab="nom	prenom	age	ville"  # Tabulation
donnees_espace="nom prenom age ville"

echo "Avec pipe (|) : $(extraire_champ "$donnees_pipe" "|" "2")"
echo "Avec tab : $(extraire_champ "$donnees_tab" "	" "3")"
echo "Avec espace : $(extraire_champ "$donnees_espace" " " "4")"
```

### Extraction de motifs complexes

```bash
#!/bin/bash

echo "=== Extraction de motifs complexes ==="

# Extraction d'informations d'une URL
url="https://www.exemple.com:8080/chemin/vers/page.html?param=valeur"
echo "URL : $url"

# Extraction du protocole
protocole="${url%%://*}"
echo "Protocole : $protocole"

# Extraction du domaine
temp="${url#*://}"      # Supprimer protocole://
domaine="${temp%%/*}"   # Tout avant le premier /
domaine="${domaine%%:*}" # Supprimer le port s'il existe
echo "Domaine : $domaine"

# Extraction du port
temp="${url#*://}"
if [[ "$temp" =~ :[0-9]+/ ]]; then
    port="${temp#*:}"
    port="${port%%/*}"
    echo "Port : $port"
else
    echo "Port : 80 (par défaut)"
fi

# Extraction du chemin
chemin="${url#*://*/}"
chemin="/${chemin%%\?*}"  # Tout avant les paramètres
echo "Chemin : $chemin"

echo ""

# Extraction d'informations d'un email
email="marie.dupont@entreprise.com"
echo "Email : $email"

utilisateur="${email%@*}"
domaine_email="${email#*@}"

echo "Utilisateur : $utilisateur"
echo "Domaine : $domaine_email"

# Extraction du prénom et nom
prenom="${utilisateur%.*}"
nom="${utilisateur#*.}"
echo "Prénom probable : ${prenom^}"  # Première lettre en majuscule
echo "Nom probable : ${nom^}"
```

## Recherche et remplacement

### Remplacement de base

```bash
#!/bin/bash

echo "=== Remplacement de chaînes ==="

phrase="J'aime les pommes et les pommes sont délicieuses"
echo "Phrase originale : $phrase"

# ${variable/motif/remplacement} - Premier occurrence
phrase_modifiee="${phrase/pommes/oranges}"
echo "Premier remplacement : $phrase_modifiee"

# ${variable//motif/remplacement} - Toutes les occurrences
phrase_modifiee="${phrase//pommes/oranges}"
echo "Tous les remplacements : $phrase_modifiee"

# ${variable/#motif/remplacement} - Au début seulement
phrase_debut="${phrase/#J'aime/J'adore}"
echo "Remplacement au début : $phrase_debut"

# ${variable/%motif/remplacement} - À la fin seulement
phrase_fin="${phrase/% délicieuses/ excellentes}"
echo "Remplacement à la fin : $phrase_fin"

echo ""

# Remplacement avec des motifs
echo "=== Remplacement avec motifs ==="
fichier="document_ancien_2020.txt"
echo "Nom de fichier : $fichier"

# Remplacer l'année
nouveau_fichier="${fichier/2020/2025}"
echo "Nouvelle année : $nouveau_fichier"

# Remplacer les underscores par des tirets
fichier_tirets="${fichier//_/-}"
echo "Avec tirets : $fichier_tirets"
```

### Suppression de parties

```bash
#!/bin/bash

echo "=== Suppression de parties ==="

texte="   Bonjour le monde!!!   "
echo "Texte original : '$texte'"

# Suppression en remplaçant par rien
# Supprimer les espaces
sans_espaces="${texte// /}"
echo "Sans espaces : '$sans_espaces'"

# Supprimer les points d'exclamation
sans_exclamation="${texte//!/}"
echo "Sans exclamation : '$sans_exclamation'"

# Supprimer les espaces de début et fin (méthode alternative)
sans_espaces_bordure="${texte#"${texte%%[![:space:]]*}"}"
sans_espaces_bordure="${sans_espaces_bordure%"${sans_espaces_bordure##*[![:space:]]}"}"
echo "Sans espaces bordure : '$sans_espaces_bordure'"

echo ""

# Nettoyage d'un numéro de téléphone
telephone="01-23-45-67-89"
echo "Téléphone avec tirets : $telephone"
telephone_propre="${telephone//-/}"
echo "Téléphone sans tirets : $telephone_propre"

# Nettoyage avec espaces et autres caractères
telephone_sale="01 23.45-67 89"
echo "Téléphone sale : '$telephone_sale'"
telephone_nettoye="${telephone_sale//[^0-9]/}"  # Garder seulement les chiffres
echo "Téléphone nettoyé : '$telephone_nettoye'"
```

### Remplacement avec sed

```bash
#!/bin/bash

echo "=== Remplacement avec sed ==="

texte="Les chats sont des animaux. Les chats aiment dormir."
echo "Texte original : $texte"

# Remplacement simple avec sed
texte_modifie=$(echo "$texte" | sed 's/chats/chiens/')
echo "Premier remplacement (sed) : $texte_modifie"

# Remplacement global avec sed
texte_modifie=$(echo "$texte" | sed 's/chats/chiens/g')
echo "Remplacement global (sed) : $texte_modifie"

# Remplacement insensible à la casse
texte_majuscule="Les CHATS sont des animaux"
texte_modifie=$(echo "$texte_majuscule" | sed 's/chats/chiens/gi')
echo "Insensible à la casse : $texte_modifie"

echo ""

# Remplacements multiples
echo "=== Remplacements multiples ==="
texte_complexe="Jean a 25 ans et Marie a 30 ans"
echo "Texte complexe : $texte_complexe"

# Plusieurs remplacements en une fois
texte_transforme=$(echo "$texte_complexe" | sed -e 's/Jean/Pierre/g' -e 's/Marie/Sophie/g' -e 's/ans/années/g')
echo "Transformé : $texte_transforme"

# Avec un fichier de configuration
cat > remplacements.sed << 'EOF'
s/Jean/Pierre/g
s/Marie/Sophie/g
s/ans/années/g
s/a /possède /g
EOF

texte_avec_fichier=$(echo "$texte_complexe" | sed -f remplacements.sed)
echo "Avec fichier sed : $texte_avec_fichier"

# Nettoyer
rm -f remplacements.sed
```

### Fonctions de remplacement avancées

```bash
#!/bin/bash

# Fonction de nettoyage de nom de fichier
nettoyer_nom_fichier() {
    local nom="$1"

    # Remplacer les caractères interdits par des underscores
    nom="${nom//[<>:\"|?*]/_}"

    # Remplacer les espaces par des underscores
    nom="${nom// /_}"

    # Supprimer les underscores multiples
    nom="${nom//__/_}"

    # Supprimer les underscores en début et fin
    nom="${nom#_}"
    nom="${nom%_}"

    echo "$nom"
}

# Fonction de formatage de numéro de téléphone
formater_telephone() {
    local numero="$1"
    local format="${2:-français}"

    # Nettoyer le numéro (garder seulement les chiffres)
    numero="${numero//[^0-9]/}"

    case "$format" in
        français)
            if [ ${#numero} -eq 10 ]; then
                echo "${numero:0:2}.${numero:2:2}.${numero:4:2}.${numero:6:2}.${numero:8:2}"
            else
                echo "Numéro invalide"
            fi
            ;;
        international)
            if [ ${#numero} -eq 10 ]; then
                echo "+33 ${numero:1:1} ${numero:2:2} ${numero:4:2} ${numero:6:2} ${numero:8:2}"
            else
                echo "Numéro invalide"
            fi
            ;;
        *)
            echo "$numero"
            ;;
    esac
}

# Fonction de masquage de données sensibles
masquer_donnees() {
    local donnee="$1"
    local type="${2:-general}"

    case "$type" in
        email)
            local utilisateur="${donnee%@*}"
            local domaine="${donnee#*@}"
            local masque="${utilisateur:0:2}***@$domaine"
            echo "$masque"
            ;;
        carte)
            # Masquer les chiffres du milieu d'une carte bancaire
            if [ ${#donnee} -eq 16 ]; then
                echo "${donnee:0:4} **** **** ${donnee:12:4}"
            else
                echo "****"
            fi
            ;;
        general)
            local longueur=${#donnee}
            if [ $longueur -le 3 ]; then
                echo "***"
            else
                echo "${donnee:0:2}${'*' * (longueur-4)}${donnee: -2}"
            fi
            ;;
    esac
}

# Tests des fonctions avancées
echo "=== Tests des fonctions de remplacement ==="

noms_fichiers=("Mon Document.txt" "Photo<famille>.jpg" "Script|important.sh")
for nom in "${noms_fichiers[@]}"; do
    echo "Original : '$nom'"
    echo "Nettoyé : '$(nettoyer_nom_fichier "$nom")'"
    echo ""
done

telephones=("0123456789" "01 23 45 67 89" "01.23.45.67.89")
for tel in "${telephones[@]}"; do
    echo "Téléphone : '$tel'"
    echo "Format français : '$(formater_telephone "$tel" "français")'"
    echo "Format international : '$(formater_telephone "$tel" "international")'"
    echo ""
done

echo "=== Masquage de données ==="
echo "Email masqué : $(masquer_donnees 'marie.dupont@example.com' 'email')"
echo "Carte masquée : $(masquer_donnees '1234567890123456' 'carte')"
```

## Expressions régulières de base et globbing

### Introduction aux expressions régulières

Les expressions régulières (regex) sont comme des **motifs de recherche super-puissants**. C'est comme avoir une loupe magique qui peut trouver des patterns complexes dans le texte.

**Analogie :** Si chercher un mot dans un texte c'est comme chercher un objet précis, les regex c'est comme chercher "tous les objets rouges et ronds" ou "tous les numéros qui commencent par 06".

```bash
#!/bin/bash

echo "=== Introduction aux expressions régulières ==="

# Test de base avec grep
texte="Jean a 25 ans
Marie a 30 ans
Pierre a 35 ans
Sophie a 28 ans"

echo "Texte de test :"
echo "$texte"
echo ""

# Recherche simple
echo "Recherche de 'Marie' :"
echo "$texte" | grep "Marie"

echo ""

# Recherche avec pattern
echo "Recherche des nombres :"
echo "$texte" | grep "[0-9]"

echo ""

# Recherche au début de ligne
echo "Noms commençant par 'M' :"
echo "$texte" | grep "^M"
```

### Métacaractères de base

```bash
#!/bin/bash

echo "=== Métacaractères de base ==="

# Créer un fichier de test
cat > test_regex.txt << 'EOF'
chat
chien
oiseau
123
abc123
test@example.com
marie.dupont@gmail.com
06.12.34.56.78
01-23-45-67-89
Bonjour le monde
BONJOUR LE MONDE
EOF

echo "Contenu du fichier de test :"
cat test_regex.txt
echo ""

# . (point) : n'importe quel caractère
echo "Lignes contenant 'ch.' (ch + un caractère) :"
grep "ch." test_regex.txt

echo ""

# ^ : début de ligne
echo "Lignes commençant par un chiffre :"
grep "^[0-9]" test_regex.txt

echo ""

# $ : fin de ligne
echo "Lignes se terminant par 'nde' :"
grep "nde$" test_regex.txt

echo ""

# * : zéro ou plusieurs occurrences
echo "Lignes contenant 'bo' suivi de n'importe quoi :"
grep -i "bo.*" test_regex.txt

echo ""

# [] : classe de caractères
echo "Lignes contenant des chiffres :"
grep "[0-9]" test_regex.txt

echo ""

# [^] : négation
echo "Lignes ne contenant QUE des lettres (pas de chiffres) :"
grep "^[^0-9]*$" test_regex.txt

# Nettoyer
rm -f test_regex.txt
```

### Classes de caractères prédéfinies

```bash
#!/bin/bash

echo "=== Classes de caractères prédéfinies ==="

# Créer des données de test
donnees="abc123 XYZ789 test@domain.com 06.12.34.56.78"

echo "Données de test : $donnees"
echo ""

# [[:alpha:]] : lettres
echo "Lettres seulement :"
echo "$donnees" | grep -o "[[:alpha:]]*"

echo ""

# [[:digit:]] : chiffres
echo "Chiffres seulement :"
echo "$donnees" | grep -o "[[:digit:]]*"

echo ""

# [[:alnum:]] : lettres et chiffres
echo "Alphanumériques :"
echo "$donnees" | grep -o "[[:alnum:]]*"

echo ""

# [[:space:]] : espaces
echo "Remplacer les espaces par des underscores :"
echo "$donnees" | sed 's/[[:space:]]/_/g'

echo ""

# Validation avec classes de caractères
valider_email() {
    local email="$1"

    # Pattern simple pour email
    if [[ "$email" =~ ^[[:alnum:]._+-]+@[[:alnum:].-]+\.[[:alpha:]]{2,}$ ]]; then
        echo "✅ Email valide : $email"
    else
        echo "❌ Email invalide : $email"
    fi
}

echo "=== Validation d'emails ==="
emails=("test@example.com" "marie.dupont@gmail.com" "invalid.email" "user@domain")
for email in "${emails[@]}"; do
    valider_email "$email"
done
```

### Expressions régulières étendues

```bash
#!/bin/bash

echo "=== Expressions régulières étendues ==="

# + : une ou plusieurs occurrences
# ? : zéro ou une occurrence
# {n} : exactement n occurrences
# {n,m} : entre n et m occurrences
# () : groupement
# | : alternative

texte_test="contact@example.com
info@test.org
support@company.co.uk
invalid-email
123@456
user@domain.com"

echo "Emails de test :"
echo "$texte_test"
echo ""

# Validation d'email plus précise
echo "Emails valides (pattern étendu) :"
echo "$texte_test" | grep -E "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"

echo ""

# Validation de numéros de téléphone français
telephones="06.12.34.56.78
01-23-45-67-89
0123456789
06 12 34 56 78
+33 6 12 34 56 78
123456"

echo "Numéros de téléphone :"
echo "$telephones"
echo ""

echo "Numéros français valides :"
echo "$telephones" | grep -E "^(0[1-9]|\\+33[ ]?[1-9])([.\\- ]?[0-9]{2}){4}$"

echo ""

# Extraction de données avec groupes
echo "=== Extraction avec groupes ==="
log_entry="2025-07-13 14:30:25 [INFO] User login successful for user: marie@example.com"

# Extraire la date
date_pattern="([0-9]{4}-[0-9]{2}-[0-9]{2})"
if [[ "$log_entry" =~ $date_pattern ]]; then
    echo "Date extraite : ${BASH_REMATCH[1]}"
fi

# Extraire l'email
email_pattern="([a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,})"
if [[ "$log_entry" =~ $email_pattern ]]; then
    echo "Email extrait : ${BASH_REMATCH[1]}"
fi
```

### Globbing (expansion de motifs)

Le globbing est le système de **motifs simples** utilisé par le shell pour les noms de fichiers. C'est plus simple que les regex mais très utile.

**Analogie :** Si les regex sont comme un microscope précis, le globbing c'est comme une loupe simple mais efficace pour les fichiers.

```bash
#!/bin/bash

echo "=== Globbing - Motifs de fichiers ==="

# Créer des fichiers de test
touch fichier1.txt fichier2.txt document.doc image.jpg script.sh
touch test1.log test2.log backup.tar.gz
mkdir dossier1 dossier2

echo "Fichiers créés pour les tests :"
ls -la

echo ""

# * : n'importe quelle séquence de caractères
echo "Tous les fichiers .txt :"
ls *.txt

echo ""
echo "Tous les fichiers commençant par 'fichier' :"
ls fichier*

echo ""

# ? : un seul caractère
echo "Fichiers 'test' + un caractère + '.log' :"
ls test?.log

echo ""

# [] : classe de caractères
echo "Fichiers se terminant par 1 ou 2 :"
ls *[12]*

echo ""

# {} : alternatives (brace expansion)
echo "Fichiers .txt ou .log :"
ls *.{txt,log}

echo ""

# Patterns plus complexes
echo "Fichiers contenant des chiffres :"
ls *[0-9]*

echo ""
echo "Fichiers ne contenant PAS de chiffres :"
ls *[!0-9]*

# Nettoyer les fichiers de test
rm -f *.txt *.doc *.jpg *.sh *.log *.tar.gz
rmdir dossier1 dossier2
```

### Fonctions de validation avec regex

```bash
#!/bin/bash

echo "=== Fonctions de validation avec regex ==="

# Validation d'adresse IP
valider_ip() {
    local ip="$1"
    local pattern="^([0-9]{1,3}\.){3}[0-9]{1,3}$"

    if [[ "$ip" =~ $pattern ]]; then
        # Vérifier que chaque octet est <= 255
        IFS='.' read -ra octets <<< "$ip"
        for octet in "${octets[@]}"; do
            if [ "$octet" -gt 255 ] || [ "$octet" -lt 0 ]; then
                echo "❌ IP invalide : $ip (octet $octet hors limites)"
                return 1
            fi
        done
        echo "✅ IP valide : $ip"
        return 0
    else
        echo "❌ IP invalide : $ip (format incorrect)"
        return 1
    fi
}

# Validation de numéro de téléphone français
valider_telephone_fr() {
    local tel="$1"

    # Nettoyer le numéro
    local tel_propre="${tel//[^0-9]/}"

    # Vérifier le format
    if [[ "$tel_propre" =~ ^0[1-9][0-9]{8}$ ]]; then
        echo "✅ Téléphone valide : $tel ($tel_propre)"
        return 0
    else
        echo "❌ Téléphone invalide : $tel"
        return 1
    fi
}

# Validation de code postal français
valider_code_postal() {
    local code="$1"

    if [[ "$code" =~ ^[0-9]{5}$ ]]; then
        echo "✅ Code postal valide : $code"
        return 0
    else
        echo "❌ Code postal invalide : $code"
        return 1
    fi
}

# Validation de date (format YYYY-MM-DD)
valider_date() {
    local date="$1"
    local pattern="^([0-9]{4})-([0-9]{2})-([0-9]{2})$"

    if [[ "$date" =~ $pattern ]]; then
        local annee="${BASH_REMATCH[1]}"
        local mois="${BASH_REMATCH[2]}"
        local jour="${BASH_REMATCH[3]}"

        # Validations basiques
        if [ "$mois" -lt 1 ] || [ "$mois" -gt 12 ]; then
            echo "❌ Date invalide : $date (mois incorrect)"
            return 1
        fi

        if [ "$jour" -lt 1 ] || [ "$jour" -gt 31 ]; then
            echo "❌ Date invalide : $date (jour incorrect)"
            return 1
        fi

        echo "✅ Date valide : $date"
        return 0
    else
        echo "❌ Date invalide : $date (format attendu: YYYY-MM-DD)"
        return 1
    fi
}

# Tests des validations
echo "=== Tests de validation ==="

ips=("192.168.1.1" "10.0.0.1" "256.1.1.1" "192.168.1" "192.168.1.1.1")
for ip in "${ips[@]}"; do
    valider_ip "$ip"
done

echo ""

telephones=("0123456789" "06.12.34.56.78" "01 23 45 67 89" "1234567890" "0023456789")
for tel in "${telephones[@]}"; do
    valider_telephone_fr "$tel"
done

echo ""

codes_postaux=("75001" "13000" "1234" "123456" "abcde")
for code in "${codes_postaux[@]}"; do
    valider_code_postal "$code"
done

echo ""

dates=("2025-07-13" "2025-13-01" "2025-07-32" "25-07-13" "2025/07/13")
for date in "${dates[@]}"; do
    valider_date "$date"
done
```

### Parseur de logs avancé

```bash
#!/bin/bash

echo "=== Parseur de logs avancé ==="

# Créer un fichier de log de test
cat > test.log << 'EOF'
2025-07-13 10:15:23 [INFO] User marie@example.com logged in from IP 192.168.1.100
2025-07-13 10:16:45 [ERROR] Failed login attempt for user john@test.com from IP 10.0.0.5
2025-07-13 10:17:12 [WARN] High CPU usage detected: 85%
2025-07-13 10:18:33 [INFO] File uploaded: document.pdf (size: 2.5MB) by sophie@company.org
2025-07-13 10:19:01 [ERROR] Database connection failed - timeout after 30s
2025-07-13 10:20:15 [INFO] User pierre@domain.fr logged out
EOF

echo "Contenu du fichier de log :"
cat test.log
echo ""

# Fonction d'extraction d'informations de log
analyser_log() {
    local fichier_log="$1"

    # Patterns pour extraction
    local pattern_timestamp="([0-9]{4}-[0-9]{2}-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2})"
    local pattern_niveau="\\[([A-Z]+)\\]"
    local pattern_email="([a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,})"
    local pattern_ip="([0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3})"

    echo "=== Analyse du fichier de log ==="

    # Compter les entrées par niveau
    echo "Répartition par niveau :"
    grep -o "\\[[A-Z]*\\]" "$fichier_log" | sort | uniq -c

    echo ""

    # Extraire tous les emails
    echo "Emails trouvés :"
    grep -oE "$pattern_email" "$fichier_log" | sort | uniq

    echo ""

    # Extraire toutes les IPs
    echo "Adresses IP trouvées :"
    grep -oE "$pattern_ip" "$fichier_log" | sort | uniq

    echo ""

    # Analyser les erreurs
    echo "Détail des erreurs :"
    while IFS= read -r ligne; do
        if [[ "$ligne" =~ $pattern_timestamp.*\\[ERROR\\].*(.+) ]]; then
            local timestamp="${BASH_REMATCH[1]}"
            local message="${ligne#*] }"
            echo "  $timestamp: $message"
        fi
    done < "$fichier_log"

    echo ""

    # Analyser les connexions
    echo "Activité de connexion :"
    while IFS= read -r ligne; do
        if [[ "$ligne" =~ logged\ in.*($pattern_email).*($pattern_ip) ]]; then
            echo "  Connexion: ${BASH_REMATCH[1]} depuis ${BASH_REMATCH[2]}"
        elif [[ "$ligne" =~ logged\ out.*($pattern_email) ]]; then
            echo "  Déconnexion: ${BASH_REMATCH[1]}"
        fi
    done < "$fichier_log"
}

# Analyser le fichier de log
analyser_log "test.log"

# Nettoyer
rm -f test.log
```

### Système de templates avec substitution

```bash
#!/bin/bash

echo "=== Système de templates ==="

# Créer un template d'email
cat > email_template.txt << 'EOF'
Bonjour {{NOM}},

Nous vous informons que votre commande #{{NUMERO_COMMANDE}}
d'un montant de {{MONTANT}} € a été {{STATUS}}.

{{#IF_EXPEDIE}}
Votre colis sera livré à l'adresse suivante :
{{ADRESSE}}

Numéro de suivi : {{TRACKING}}
{{/IF_EXPEDIE}}

{{#IF_ERREUR}}
Motif : {{MOTIF_ERREUR}}
{{/IF_ERREUR}}

Cordialement,
L'équipe {{ENTREPRISE}}

---
Date : {{DATE}}
Email : {{EMAIL_SUPPORT}}
EOF

echo "Template d'email :"
cat email_template.txt
echo ""

# Fonction de substitution de template
substituer_template() {
    local template="$1"
    local -n variables=$2  # Référence au tableau associatif

    local contenu
    contenu=$(cat "$template")

    # Substitution des variables simples
    for cle in "${!variables[@]}"; do
        local valeur="${variables[$cle]}"
        contenu="${contenu//\{\{$cle\}\}/$valeur}"
    done

    # Gestion des conditions IF (simple)
    # Supprimer les blocs IF_* si la variable n'existe pas ou est vide
    while [[ "$contenu" =~ \{\{#IF_([A-Z_]+)\}\}([^{]*)\{\{/IF_[A-Z_]+\}\} ]]; do
        local condition="${BASH_REMATCH[1]}"
        local bloc="${BASH_REMATCH[2]}"
        local bloc_complet="${BASH_REMATCH[0]}"

        if [ -n "${variables[$condition]}" ]; then
            # Garder le contenu du bloc
            contenu="${contenu/$bloc_complet/$bloc}"
        else
            # Supprimer tout le bloc
            contenu="${contenu/$bloc_complet/}"
        fi
    done

    echo "$contenu"
}

# Test du système de templates
declare -A donnees_commande=(
    ["NOM"]="Marie Dupont"
    ["NUMERO_COMMANDE"]="CMD-2025-001"
    ["MONTANT"]="149.99"
    ["STATUS"]="expédiée"
    ["ADRESSE"]="123 Rue de la Paix, 75001 Paris"
    ["TRACKING"]="FR123456789"
    ["ENTREPRISE"]="TechShop"
    ["DATE"]="$(date '+%d/%m/%Y')"
    ["EMAIL_SUPPORT"]="support@techshop.com"
    ["IF_EXPEDIE"]="true"
)

echo "=== Email généré (commande expédiée) ==="
substituer_template "email_template.txt" donnees_commande

echo ""
echo "=== Email généré (commande en erreur) ==="

# Modifier les données pour une erreur
donnees_commande["STATUS"]="annulée"
donnees_commande["MOTIF_ERREUR"]="Stock insuffisant"
unset donnees_commande["IF_EXPEDIE"]
donnees_commande["IF_ERREUR"]="true"

substituer_template "email_template.txt" donnees_commande

# Nettoyer
rm -f email_template.txt
```

### Générateur de mots de passe avec motifs

```bash
#!/bin/bash

echo "=== Générateur de mots de passe avec motifs ==="

# Fonction de génération de mot de passe selon un motif
generer_mot_de_passe() {
    local motif="$1"
    local mot_de_passe=""

    # Définition des ensembles de caractères
    local minuscules="abcdefghijklmnopqrstuvwxyz"
    local majuscules="ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    local chiffres="0123456789"
    local symboles="!@#$%^&*()_+-=[]{}|;:,.<>?"

    # Parcourir chaque caractère du motif
    for (( i=0; i<${#motif}; i++ )); do
        local char="${motif:$i:1}"
        local ensemble=""

        case "$char" in
            'l')  ensemble="$minuscules" ;;      # lettre minuscule
            'L')  ensemble="$majuscules" ;;      # lettre majuscule
            'd')  ensemble="$chiffres" ;;        # chiffre
            's')  ensemble="$symboles" ;;        # symbole
            'a')  ensemble="$minuscules$majuscules" ;;  # lettre (min ou maj)
            'c')  ensemble="$minuscules$majuscules$chiffres" ;;  # alphanumérique
            'x')  ensemble="$minuscules$majuscules$chiffres$symboles" ;;  # tout
            *)    mot_de_passe+="$char"; continue ;;  # caractère littéral
        esac

        # Choisir un caractère aléatoire de l'ensemble
        local index=$((RANDOM % ${#ensemble}))
        mot_de_passe+="${ensemble:$index:1}"
    done

    echo "$mot_de_passe"
}

# Fonction de validation de force de mot de passe
evaluer_force_mot_de_passe() {
    local mot_de_passe="$1"
    local score=0
    local commentaires=()

    # Longueur
    if [ ${#mot_de_passe} -ge 12 ]; then
        score=$((score + 25))
        commentaires+=("✅ Longueur suffisante")
    elif [ ${#mot_de_passe} -ge 8 ]; then
        score=$((score + 15))
        commentaires+=("⚠️  Longueur acceptable")
    else
        commentaires+=("❌ Trop court")
    fi

    # Présence de minuscules
    if [[ "$mot_de_passe" =~ [a-z] ]]; then
        score=$((score + 15))
        commentaires+=("✅ Contient des minuscules")
    fi

    # Présence de majuscules
    if [[ "$mot_de_passe" =~ [A-Z] ]]; then
        score=$((score + 15))
        commentaires+=("✅ Contient des majuscules")
    fi

    # Présence de chiffres
    if [[ "$mot_de_passe" =~ [0-9] ]]; then
        score=$((score + 15))
        commentaires+=("✅ Contient des chiffres")
    fi

    # Présence de symboles
    if [[ "$mot_de_passe" =~ [^a-zA-Z0-9] ]]; then
        score=$((score + 30))
        commentaires+=("✅ Contient des symboles")
    fi

    # Évaluation finale
    local niveau
    if [ $score -ge 80 ]; then
        niveau="🟢 FORT"
    elif [ $score -ge 60 ]; then
        niveau="🟡 MOYEN"
    else
        niveau="🔴 FAIBLE"
    fi

    echo "Mot de passe : $mot_de_passe"
    echo "Score : $score/100 - $niveau"
    for commentaire in "${commentaires[@]}"; do
        echo "  $commentaire"
    done
}

# Tests du générateur
echo "=== Tests avec différents motifs ==="

motifs=(
    "lllldddd"              # 4 lettres + 4 chiffres
    "LlllLlll-dddd"         # Maj + min + min + min + Maj + min + min + min + tiret + 4 chiffres
    "xxxxxxxxxxxs"          # 11 caractères aléatoires + 1 symbole
    "Llllddss"              # Maj + 3 min + 2 chiffres + 2 symboles
    "ccccccccsss"           # 8 alphanumériques + 3 symboles
)

for motif in "${motifs[@]}"; do
    echo ""
    echo "Motif : '$motif'"
    mot_de_passe=$(generer_mot_de_passe "$motif")
    evaluer_force_mot_de_passe "$mot_de_passe"
done
```

### Exercices pratiques

```bash
#!/bin/bash

echo "=== Exercices pratiques ==="

# Exercice 1 : Analyseur de fichiers CSV
analyser_csv() {
    local fichier="$1"

    # À compléter :
    # 1. Compter le nombre de lignes
    # 2. Identifier le délimiteur (virgule, point-virgule, tabulation)
    # 3. Extraire les en-têtes
    # 4. Valider que toutes les lignes ont le même nombre de colonnes
    # 5. Détecter les types de données de chaque colonne

    echo "Fonction à implémenter"
}

# Exercice 2 : Validateur d'adresses email avancé
valider_email_avance() {
    local email="$1"

    # À compléter :
    # 1. Vérifier le format général
    # 2. Valider que le domaine a au moins un point
    # 3. Vérifier que les caractères sont autorisés
    # 4. Contrôler la longueur des parties
    # 5. Optionnel : vérifier que le domaine existe (avec dig)

    echo "Fonction à implémenter"
}

# Exercice 3 : Extracteur d'URLs d'un texte
extraire_urls() {
    local texte="$1"

    # À compléter :
    # 1. Détecter les URLs HTTP et HTTPS
    # 2. Extraire les domaines
    # 3. Classer par type (site web, image, document)
    # 4. Détecter les URLs cassées (format invalide)

    echo "Fonction à implémenter"
}

echo "Exercices à compléter dans ce script !"
```

## Solutions des exercices

### Solution Exercice 1 : Analyseur CSV

```bash
#!/bin/bash

analyser_csv() {
    local fichier="$1"

    if [ ! -f "$fichier" ]; then
        echo "❌ Fichier '$fichier' introuvable"
        return 1
    fi

    echo "=== Analyse du fichier CSV : $fichier ==="

    # 1. Compter les lignes
    local nb_lignes=$(wc -l < "$fichier")
    echo "Nombre de lignes : $nb_lignes"

    if [ $nb_lignes -eq 0 ]; then
        echo "❌ Fichier vide"
        return 1
    fi

    # 2. Détecter le délimiteur
    local premiere_ligne=$(head -n1 "$fichier")
    local delimiteur=""

    if [[ "$premiere_ligne" =~ , ]]; then
        local nb_virgules=$(echo "$premiere_ligne" | tr -cd ',' | wc -c)
        delimiteur=","
        echo "Délimiteur détecté : virgule ($nb_virgules colonnes)"
    elif [[ "$premiere_ligne" =~ \; ]]; then
        local nb_pointvirgules=$(echo "$premiere_ligne" | tr -cd ';' | wc -c)
        delimiteur=";"
        echo "Délimiteur détecté : point-virgule ($nb_pointvirgules colonnes)"
    elif [[ "$premiere_ligne" =~ $'\t' ]]; then
        delimiteur=$'\t'
        echo "Délimiteur détecté : tabulation"
    else
        echo "❌ Délimiteur non détecté"
        return 1
    fi

    # 3. Extraire les en-têtes
    echo "En-têtes :"
    IFS="$delimiteur" read -ra headers <<< "$premiere_ligne"
    for i in "${!headers[@]}"; do
        echo "  Colonne $((i+1)): ${headers[$i]}"
    done

    local nb_colonnes=${#headers[@]}

    # 4. Vérifier la cohérence des colonnes
    echo "Vérification de la cohérence :"
    local ligne_num=1
    local erreurs=0

    while IFS= read -r ligne; do
        ((ligne_num++))
        local nb_champs=$(echo "$ligne" | tr -cd "$delimiteur" | wc -c)
        nb_champs=$((nb_champs + 1))

        if [ $nb_champs -ne $nb_colonnes ]; then
            echo "  ❌ Ligne $ligne_num: $nb_champs champs au lieu de $nb_colonnes"
            ((erreurs++))
        fi
    done < <(tail -n +2 "$fichier")

    if [ $erreurs -eq 0 ]; then
        echo "  ✅ Toutes les lignes ont le bon nombre de colonnes"
    else
        echo "  ❌ $erreurs ligne(s) avec un nombre incorrect de colonnes"
    fi

    # 5. Détecter les types de données
    echo "Types de données détectés :"
    for i in $(seq 0 $((nb_colonnes-1))); do
        local colonne_num=$((i+1))
        local echantillon=$(tail -n +2 "$fichier" | cut -d"$delimiteur" -f$colonne_num | head -10)

        local type="texte"
        if echo "$echantillon" | grep -qE '^[0-9]+$'; then
            type="entier"
        elif echo "$echantillon" | grep -qE '^[0-9]*\.[0-9]+$'; then
            type="décimal"
        elif echo "$echantillon" | grep -qE '^[0-9]{4}-[0-9]{2}-[0-9]{2}$'; then
            type="date"
        elif echo "$echantillon" | grep -qE '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'; then
            type="email"
        fi

        echo "  ${headers[$i]}: $type"
    done
}

# Créer un fichier CSV de test
cat > test.csv << 'EOF'
nom,prenom,age,email,date_inscription
Dupont,Marie,25,marie.dupont@example.com,2025-01-15
Martin,Pierre,30,pierre.martin@test.org,2025-02-20
Bernard,Sophie,28,sophie.bernard@company.fr,2025-03-10
EOF

echo "=== Test de l'analyseur CSV ==="
analyser_csv "test.csv"

# Nettoyer
rm -f test.csv
```

### Solution Exercice 2 : Validateur email avancé

```bash
#!/bin/bash

valider_email_avance() {
    local email="$1"
    local erreurs=()

    echo "=== Validation avancée de : $email ==="

    # 1. Vérification du format général
    if [[ ! "$email" =~ ^[^@]+@[^@]+$ ]]; then
        erreurs+=("Format général invalide")
    fi

    # Séparer utilisateur et domaine
    local utilisateur="${email%@*}"
    local domaine="${email#*@}"

    # 2. Validation de la partie utilisateur
    if [ ${#utilisateur} -eq 0 ]; then
        erreurs+=("Partie utilisateur vide")
    elif [ ${#utilisateur} -gt 64 ]; then
        erreurs+=("Partie utilisateur trop longue (max 64 caractères)")
    elif [[ ! "$utilisateur" =~ ^[a-zA-Z0-9._%+-]+$ ]]; then
        erreurs+=("Caractères invalides dans la partie utilisateur")
    fi

    # 3. Validation du domaine
    if [ ${#domaine} -eq 0 ]; then
        erreurs+=("Domaine vide")
    elif [ ${#domaine} -gt 253 ]; then
        erreurs+=("Domaine trop long (max 253 caractères)")
    elif [[ ! "$domaine" =~ \. ]]; then
        erreurs+=("Domaine sans point")
    elif [[ ! "$domaine" =~ ^[a-zA-Z0-9.-]+$ ]]; then
        erreurs+=("Caractères invalides dans le domaine")
    fi

    # 4. Validation de l'extension
    local extension="${domaine##*.}"
    if [ ${#extension} -lt 2 ]; then
        erreurs+=("Extension trop courte")
    elif [ ${#extension} -gt 6 ]; then
        erreurs+=("Extension trop longue")
    fi

    # 5. Vérifications supplémentaires
    if [[ "$email" =~ \.\. ]]; then
        erreurs+=("Points consécutifs détectés")
    fi

    if [[ "$email" =~ ^\.|\.$|@\.|\.@ ]]; then
        erreurs+=("Point en position invalide")
    fi

    # Affichage des résultats
    if [ ${#erreurs[@]} -eq 0 ]; then
        echo "✅ Email valide"

        # Optionnel : vérification DNS (si dig est disponible)
        if command -v dig >/dev/null 2>&1; then
            if dig +short MX "$domaine" >/dev/null 2>&1; then
                echo "✅ Domaine avec serveur mail"
            else
                echo "⚠️  Domaine sans serveur mail détecté"
            fi
        fi

        return 0
    else
        echo "❌ Email invalide :"
        for erreur in "${erreurs[@]}"; do
            echo "  - $erreur"
        done
        return 1
    fi
}

# Tests du validateur avancé
emails_test=(
    "test@example.com"
    "marie.dupont@gmail.com"
    "user+tag@domain.co.uk"
    "@invalid.com"
    "user@"
    "user..double@example.com"
    "verylongusernamethatexceedssixtyfourcharacterslimitwhichisnotallowed@example.com"
    "user@domain"
    "user@domain.c"
    "valid@valid-domain.com"
)

for email in "${emails_test[@]}"; do
    valider_email_avance "$email"
    echo ""
done
```

### Solution Exercice 3 : Extracteur d'URLs

```bash
#!/bin/bash

extraire_urls() {
    local texte="$1"

    echo "=== Extraction d'URLs ==="

    # Pattern pour URLs
    local pattern_url="https?://[a-zA-Z0-9.-]+(/[a-zA-Z0-9./_?&=%+-]*)?"

    # Extraire toutes les URLs
    local urls=($(echo "$texte" | grep -oE "$pattern_url"))

    if [ ${#urls[@]} -eq 0 ]; then
        echo "Aucune URL trouvée"
        return 0
    fi

    echo "URLs trouvées : ${#urls[@]}"
    echo ""

    # Classer les URLs
    declare -A domaines
    declare -a sites_web
    declare -a images
    declare -a documents
    declare -a autres

    for url in "${urls[@]}"; do
        # Extraire le domaine
        local domaine=$(echo "$url" | sed -E 's|https?://([^/]+).*|\1|')
        domaines["$domaine"]=$((${domaines["$domaine"]} + 1))

        # Classer par type
        if [[ "$url" =~ \.(jpg|jpeg|png|gif|bmp|svg)(\?.*)?$ ]]; then
            images+=("$url")
        elif [[ "$url" =~ \.(pdf|doc|docx|xls|xlsx|ppt|pptx|zip|tar|gz)(\?.*)?$ ]]; then
            documents+=("$url")
        elif [[ "$url" =~ \.(html|htm|php|asp|jsp)(\?.*)?$ ]] || [[ ! "$url" =~ \.[a-zA-Z]{2,4}(\?.*)?$ ]]; then
            sites_web+=("$url")
        else
            autres+=("$url")
        fi

        # Valider l'URL
        local statut="✅"
        if [[ ! "$url" =~ ^https?://[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}(/.*)?$ ]]; then
            statut="❌ Format invalide"
        elif [[ "$url" =~ [<>\"'{}|\\^`\[\]] ]]; then
            statut="⚠️ Caractères suspects"
        fi

        echo "  $statut $url"
    done

    echo ""
    echo "=== Classification ==="
    echo "Sites web (${#sites_web[@]}) :"
    for site in "${sites_web[@]}"; do
        echo "  🌐 $site"
    done

    echo ""
    echo "Images (${#images[@]}) :"
    for image in "${images[@]}"; do
        echo "  🖼️  $image"
    done

    echo ""
    echo "Documents (${#documents[@]}) :"
    for doc in "${documents[@]}"; do
        echo "  📄 $doc"
    done

    if [ ${#autres[@]} -gt 0 ]; then
        echo ""
        echo "Autres (${#autres[@]}) :"
        for autre in "${autres[@]}"; do
            echo "  ❓ $autre"
        done
    fi

    echo ""
    echo "=== Domaines les plus fréquents ==="
    for domaine in "${!domaines[@]}"; do
        echo "  $domaine: ${domaines[$domaine]} URL(s)"
    done | sort -k2 -nr | head -5
}

# Test de l'extracteur d'URLs
texte_test="Visitez notre site https://www.example.com pour plus d'informations.
Téléchargez le PDF https://files.example.com/document.pdf et
consultez les images https://cdn.example.com/photo1.jpg et
https://cdn.example.com/photo2.png. Voir aussi https://blog.example.com/article
et http://old-site.com/page.html. URL cassée: https://invalid..domain/test"

echo "=== Test de l'extracteur d'URLs ==="
echo "Texte d'exemple :"
echo "$texte_test"
echo ""

extraire_urls "$texte_test"
```

## Outils avancés de manipulation de chaînes

### AWK pour le traitement de texte structuré

```bash
#!/bin/bash

echo "=== AWK pour le traitement de texte ==="

# Créer des données de test
cat > donnees.csv << 'EOF'
nom,prenom,age,salaire,department
Dupont,Marie,25,35000,IT
Martin,Pierre,30,42000,Finance
Bernard,Sophie,28,38000,IT
Rousseau,Jean,35,45000,Marketing
Moreau,Claire,26,33000,Finance
EOF

echo "Données de test :"
cat donnees.csv
echo ""

# Calculs avec AWK
echo "=== Calculs avec AWK ==="

# Salaire moyen
echo "Salaire moyen :"
awk -F',' 'NR>1 {sum+=$4; count++} END {print "Moyenne: " sum/count " €"}' donnees.csv

# Salaire par département
echo ""
echo "Salaire moyen par département :"
awk -F',' 'NR>1 {dept[$5]+=$4; count[$5]++} END {for(d in dept) print d ": " dept[d]/count[d] " €"}' donnees.csv

# Employés par tranche d'âge
echo ""
echo "Répartition par âge :"
awk -F',' 'NR>1 {
    if($3<30) young++;
    else if($3<40) middle++;
    else senior++
} END {
    print "< 30 ans: " young+0
    print "30-39 ans: " middle+0
    print ">= 40 ans: " senior+0
}' donnees.csv

# Formatage avancé
echo ""
echo "Rapport formaté :"
awk -F',' 'BEGIN {
    print "╔══════════════════════════════════════╗"
    print "║           RAPPORT EMPLOYÉS           ║"
    print "╠══════════════════════════════════════╣"
    printf "║ %-15s %-10s %-8s ║\n", "NOM", "DEPT", "SALAIRE"
    print "╠══════════════════════════════════════╣"
}
NR>1 {
    printf "║ %-15s %-10s %8s ║\n", $2" "$1, $5, $4"€"
}
END {
    print "╚══════════════════════════════════════╝"
}' donnees.csv

# Nettoyer
rm -f donnees.csv
```

### SED pour les transformations complexes

```bash
#!/bin/bash

echo "=== SED pour transformations complexes ==="

# Créer un fichier de configuration
cat > config.txt << 'EOF'
# Configuration de l'application
server_host=localhost
server_port=8080
database_url=mysql://localhost:3306/mydb
debug_mode=true
log_level=INFO
# Commentaire à garder
max_connections=100
timeout=30
EOF

echo "Configuration originale :"
cat config.txt
echo ""

# Transformation du fichier de configuration
echo "=== Transformations avec sed ==="

# Changer l'environnement de développement vers production
sed_script='
# Commenter le mode debug
s/^debug_mode=true/#debug_mode=false/

# Changer le niveau de log
s/log_level=INFO/log_level=WARN/

# Augmenter les connexions pour la production
s/max_connections=100/max_connections=500/

# Ajouter un timestamp
1i\
# Configuration générée le '$(date)'\

# Ajouter une section de monitoring
$a\
\
# Section monitoring\
monitoring_enabled=true\
metrics_port=9090
'

echo "Configuration transformée :"
sed "$sed_script" config.txt

echo ""

# Validation et nettoyage de logs
cat > app.log << 'EOF'
2025-07-13 10:15:23 [DEBUG] Starting application...
2025-07-13 10:15:24 [INFO] Database connected
2025-07-13 10:15:25 [DEBUG] Loading configuration...
2025-07-13 10:15:26 [WARN] Configuration file not found, using defaults
2025-07-13 10:15:27 [ERROR] Failed to load module: authentication
2025-07-13 10:15:28 [INFO] Server started on port 8080
2025-07-13 10:15:29 [DEBUG] Ready to accept connections
EOF

echo "=== Nettoyage de logs ==="
echo "Log original :"
cat app.log
echo ""

echo "Log nettoyé (sans DEBUG) :"
sed '/\[DEBUG\]/d' app.log

echo ""
echo "Log avec timestamps raccourcis :"
sed 's/2025-07-13 \([0-9:]*\)/[\1]/' app.log

echo ""
echo "Log avec niveaux colorés (simulé) :"
sed -e 's/\[ERROR\]/[🔴ERROR🔴]/g' \
    -e 's/\[WARN\]/[🟡WARN🟡]/g' \
    -e 's/\[INFO\]/[🟢INFO🟢]/g' \
    -e 's/\[DEBUG\]/[🔵DEBUG🔵]/g' app.log

# Nettoyer
rm -f config.txt app.log
```

### Processeur de markdown simple

```bash
#!/bin/bash

echo "=== Processeur Markdown simple ==="

# Créer un fichier markdown de test
cat > document.md << 'EOF'
# Mon Document

Ceci est un **texte en gras** et ceci est en *italique*.

## Section importante

Voici une liste :
- Premier élément
- Deuxième élément
- Troisième élément

### Code

Voici du `code inline` et un bloc de code :

```bash
echo "Hello World"
```

[Lien vers Google](https://www.google.com)

> Ceci est une citation
> sur plusieurs lignes
EOF

echo "Document Markdown original :"
cat document.md
echo ""

# Fonction de conversion Markdown vers HTML
convertir_markdown_html() {
    local fichier="$1"

    # Initialiser le HTML
    echo "<!DOCTYPE html>"
    echo "<html><head><meta charset='UTF-8'><title>Document</title></head><body>"

    local dans_bloc_code=false
    local dans_citation=false

    while IFS= read -r ligne; do
        # Bloc de code
        if [[ "$ligne" =~ ^```.*$ ]]; then
            if [ "$dans_bloc_code" = true ]; then
                echo "</pre></code>"
                dans_bloc_code=false
            else
                echo "<code><pre>"
                dans_bloc_code=true
            fi
            continue
        fi

        if [ "$dans_bloc_code" = true ]; then
            echo "$ligne"
            continue
        fi

        # Titres
        if [[ "$ligne" =~ ^###\ (.*)$ ]]; then
            echo "<h3>${BASH_REMATCH[1]}</h3>"
        elif [[ "$ligne" =~ ^##\ (.*)$ ]]; then
            echo "<h2>${BASH_REMATCH[1]}</h2>"
        elif [[ "$ligne" =~ ^#\ (.*)$ ]]; then
            echo "<h1>${BASH_REMATCH[1]}</h1>"

        # Citations
        elif [[ "$ligne" =~ ^>\ (.*)$ ]]; then
            if [ "$dans_citation" = false ]; then
                echo "<blockquote>"
                dans_citation=true
            fi
            echo "${BASH_REMATCH[1]}<br>"
        else
            if [ "$dans_citation" = true ]; then
                echo "</blockquote>"
                dans_citation=false
            fi

            # Listes
            if [[ "$ligne" =~ ^-\ (.*)$ ]]; then
                echo "<li>${BASH_REMATCH[1]}</li>"

            # Ligne vide
            elif [ -z "$ligne" ]; then
                echo "<br>"

            # Paragraphe normal
            else
                # Gras et italique
                ligne="${ligne//\*\*([^*]*)\*\*/<strong>\1<\/strong>}"
                ligne="${ligne//\*([^*]*)\*/<em>\1<\/em>}"

                # Code inline
                ligne="${ligne//\`([^\`]*)\`/<code>\1<\/code>}"

                # Liens
                ligne=$(echo "$ligne" | sed -E 's/\[([^\]]*)\]\(([^)]*)\)/<a href="\2">\1<\/a>/g')

                echo "<p>$ligne</p>"
            fi
        fi
    done < "$fichier"

    if [ "$dans_citation" = true ]; then
        echo "</blockquote>"
    fi

    echo "</body></html>"
}

echo "=== Conversion en HTML ==="
convertir_markdown_html document.md > document.html

echo "Document HTML généré :"
head -20 document.html

# Nettoyer
rm -f document.md document.html
```

### Système de templating avancé

```bash
#!/bin/bash

echo "=== Système de templating avancé ==="

# Créer un template complexe
cat > template_avance.tpl << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>{{TITRE}}</title>
    <style>
        {{#CSS}}
        body { font-family: Arial, sans-serif; }
        .header { background-color: {{COULEUR_HEADER|#333}}; color: white; padding: 10px; }
        {{/CSS}}
    </style>
</head>
<body>
    <div class="header">
        <h1>{{TITRE}}</h1>
        {{#SOUS_TITRE}}
        <h2>{{SOUS_TITRE}}</h2>
        {{/SOUS_TITRE}}
    </div>

    <div class="content">
        {{#UTILISATEURS}}
        <div class="user">
            <h3>{{nom}} {{prenom}}</h3>
            <p>Email: {{email}}</p>
            <p>Âge: {{age}} ans</p>
            {{#ADMIN}}
            <span class="badge">Administrateur</span>
            {{/ADMIN}}
        </div>
        {{/UTILISATEURS}}

        {{#AUCUN_UTILISATEUR}}
        <p>Aucun utilisateur trouvé.</p>
        {{/AUCUN_UTILISATEUR}}
    </div>

    <footer>
        <p>Généré le {{DATE}} à {{HEURE}}</p>
    </footer>
</body>
</html>
EOF

# Moteur de template avancé
traiter_template() {
    local template="$1"
    local -n data=$2

    local contenu=$(cat "$template")
    local resultat=""

    # Variables simples avec valeurs par défaut
    while [[ "$contenu" =~ \{\{([A-Z_]+)(\|([^}]+))?\}\} ]]; do
        local variable="${BASH_REMATCH[1]}"
        local defaut="${BASH_REMATCH[3]}"
        local valeur="${data[$variable]}"

        if [ -z "$valeur" ] && [ -n "$defaut" ]; then
            valeur="$defaut"
        fi

        contenu="${contenu//${BASH_REMATCH[0]}/$valeur}"
    done

    # Blocs conditionnels
    while [[ "$contenu" =~ \{\{#([A-Z_]+)\}\}(.*?)\{\{/\1\}\} ]]; do
        local condition="${BASH_REMATCH[1]}"
        local bloc="${BASH_REMATCH[2]}"
        local bloc_complet="${BASH_REMATCH[0]}"

        if [ -n "${data[$condition]}" ] && [ "${data[$condition]}" != "false" ]; then
            contenu="${contenu/$bloc_complet/$bloc}"
        else
            contenu="${contenu/$bloc_complet/}"
        fi
    done

    echo "$contenu"
}

# Données pour le template
declare -A donnees_template=(
    ["TITRE"]="Gestion des Utilisateurs"
    ["SOUS_TITRE"]="Tableau de bord administrateur"
    ["CSS"]="true"
    ["COULEUR_HEADER"]="#2c3e50"
    ["DATE"]="$(date +%d/%m/%Y)"
    ["HEURE"]="$(date +%H:%M)"
    ["UTILISATEURS"]="true"
)

echo "Template généré :"
traiter_template "template_avance.tpl" donnees_template | head -30

# Nettoyer
rm -f template_avance.tpl
```

## Récapitulatif

✅ **Ce que vous avez appris dans ce chapitre :**

1. **Opérations sur les chaînes** : Longueur, concaténation, comparaison, transformation de casse
2. **Extraction de sous-chaînes** : Position, motifs, délimiteurs
3. **Recherche et remplacement** : Motifs simples, sed, fonctions avancées
4. **Expressions régulières et globbing** : Patterns, validation, extraction de données

✅ **Points clés à retenir :**

**Opérations de base :**
```bash
longueur=${#chaine}                    # Longueur
concatenation="$chaine1$chaine2"       # Concaténation
minuscules="${chaine,,}"               # Minuscules
majuscules="${chaine^^}"               # Majuscules
```

**Extraction :**
```bash
sous_chaine="${chaine:position:longueur}"   # Par position
premier_mot="${chaine%% *}"                 # Premier mot
dernier_mot="${chaine##* }"                 # Dernier mot
extension="${fichier##*.}"                  # Extension
```

**Remplacement :**
```bash
nouveau="${chaine/ancien/nouveau}"          # Premier
nouveau="${chaine//ancien/nouveau}"         # Tous
nouveau="${chaine/#debut/nouveau}"          # Début
nouveau="${chaine/%fin/nouveau}"            # Fin
```

**Expressions régulières :**
```bash
if [[ "$chaine" =~ pattern ]]; then
    echo "Match trouvé : ${BASH_REMATCH[0]}"
fi
```

**Globbing :**
```bash
*.txt           # Tous les .txt
fichier?.log    # fichier + 1 caractère + .log
*[0-9]*         # Contenant des chiffres
*.{jpg,png}     # .jpg ou .png
```

✅ **Outils utiles :**
- **grep** : Recherche avec regex
- **sed** : Transformation de flux
- **awk** : Traitement de données structurées
- **cut** : Extraction de colonnes
- **tr** : Transformation de caractères

✅ **Cas d'usage pratiques :**
- Validation de données (emails, IPs, téléphones)
- Parsing de logs et fichiers de configuration
- Génération de documents (HTML, rapports)
- Nettoyage et formatage de données
- Extraction d'informations depuis du texte

✅ **Prochaines étapes :**
Dans le chapitre 8, nous découvrirons la gestion des fichiers et répertoires !

---

💡 **Conseil pratique :** La manipulation de chaînes est fondamentale en Bash. Pratiquez avec des données réelles (logs, CSV, etc.) pour bien maîtriser ces concepts avant de passer au chapitre suivant.

⏭️
