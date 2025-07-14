🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Tutoriel Bash - Chapitre 6 : Fonctions

## Définition et appel de fonctions

### Qu'est-ce qu'une fonction ?

Une fonction est comme une **recette de cuisine** que vous pouvez utiliser plusieurs fois. Au lieu de réécrire toutes les étapes à chaque fois, vous donnez un nom à votre recette et vous l'appelez quand vous en avez besoin.

**Analogie :** Si vous devez souvent faire du café :
- **Sans fonction :** Répéter les étapes "moudre, chauffer l'eau, verser..." à chaque fois
- **Avec fonction :** Créer une recette "faire_cafe" et l'appeler quand nécessaire

### Pourquoi utiliser des fonctions ?

1. **Éviter la répétition** : Écrivez une fois, utilisez partout
2. **Organisation** : Code plus lisible et structuré
3. **Maintenance** : Modifier une fois, effet partout
4. **Réutilisabilité** : Partager des morceaux de code
5. **Tests** : Plus facile de tester des petites parties

### Syntaxe de base

```bash
#!/bin/bash

# Méthode 1 : Définition simple
dire_bonjour() {
    echo "Bonjour tout le monde !"
}

# Méthode 2 : Avec le mot-clé function
function dire_bonsoir() {
    echo "Bonsoir et bonne nuit !"
}

# Méthode 3 : Style compact
saluer() { echo "Salut !"; }

# Appel des fonctions
echo "=== Test des fonctions ==="
dire_bonjour
dire_bonsoir
saluer
```

### Exemple concret : Fonctions utilitaires

```bash
#!/bin/bash

# Fonction pour afficher un titre encadré
afficher_titre() {
    echo "================================"
    echo "  $1"
    echo "================================"
}

# Fonction pour afficher un message d'erreur
afficher_erreur() {
    echo "❌ ERREUR : $1" >&2
}

# Fonction pour afficher un message de succès
afficher_succes() {
    echo "✅ SUCCÈS : $1"
}

# Fonction pour demander confirmation
demander_confirmation() {
    local question="$1"
    local reponse

    while true; do
        read -p "$question (o/n) : " reponse
        case "$reponse" in
            [oO]|[oO][uU][iI])
                return 0  # Vrai
                ;;
            [nN]|[nN][oO][nN])
                return 1  # Faux
                ;;
            *)
                echo "Répondez par 'oui' ou 'non'"
                ;;
        esac
    done
}

# Test des fonctions
afficher_titre "Système de gestion"

if demander_confirmation "Voulez-vous continuer ?"; then
    afficher_succes "Démarrage du système"
else
    afficher_erreur "Opération annulée"
    exit 1
fi
```

### Fonctions avec logique complexe

```bash
#!/bin/bash

# Fonction pour vérifier si un fichier est sûr
fichier_est_sur() {
    local fichier="$1"

    # Vérifications multiples
    if [ ! -f "$fichier" ]; then
        echo "Le fichier n'existe pas"
        return 1
    fi

    if [ ! -r "$fichier" ]; then
        echo "Le fichier n'est pas lisible"
        return 1
    fi

    # Vérifier la taille (pas trop gros)
    local taille=$(stat -c%s "$fichier" 2>/dev/null || echo "0")
    if [ "$taille" -gt 1048576 ]; then  # 1MB
        echo "Le fichier est trop gros (> 1MB)"
        return 1
    fi

    echo "Le fichier est sûr"
    return 0
}

# Fonction pour créer une sauvegarde
creer_sauvegarde() {
    local fichier_source="$1"
    local repertoire_backup="$HOME/backup"

    # Créer le répertoire de sauvegarde si nécessaire
    mkdir -p "$repertoire_backup"

    # Vérifier le fichier source
    if ! fichier_est_sur "$fichier_source"; then
        return 1
    fi

    # Créer le nom de sauvegarde avec timestamp
    local nom_base=$(basename "$fichier_source")
    local timestamp=$(date +"%Y%m%d_%H%M%S")
    local fichier_backup="$repertoire_backup/${nom_base}.backup_$timestamp"

    # Copier le fichier
    if cp "$fichier_source" "$fichier_backup"; then
        echo "✅ Sauvegarde créée : $fichier_backup"
        return 0
    else
        echo "❌ Erreur lors de la sauvegarde"
        return 1
    fi
}

# Test
echo "=== Test de sauvegarde ==="
echo "Contenu de test" > fichier_test.txt
creer_sauvegarde "fichier_test.txt"
```

## Paramètres et valeurs de retour

### Passage de paramètres

Les fonctions reçoivent des paramètres exactement comme les scripts : `$1`, `$2`, `$3`, etc.

```bash
#!/bin/bash

# Fonction avec paramètres
presenter_personne() {
    local nom="$1"
    local age="$2"
    local ville="$3"

    echo "=== Présentation ==="
    echo "Nom : $nom"
    echo "Âge : $age ans"
    echo "Ville : $ville"

    # Vérification des paramètres
    if [ $# -lt 3 ]; then
        echo "⚠️  Informations incomplètes (reçu $# paramètre(s))"
    fi
}

# Tests avec différents nombres de paramètres
echo "Test 1 : Tous les paramètres"
presenter_personne "Marie" "25" "Paris"

echo ""
echo "Test 2 : Paramètres manquants"
presenter_personne "Jean" "30"

echo ""
echo "Test 3 : Paramètres supplémentaires"
presenter_personne "Pierre" "35" "Lyon" "France" "Europe"
```

### Fonction calculatrice

```bash
#!/bin/bash

# Calculatrice avec paramètres
calculer() {
    local operation="$1"
    local nombre1="$2"
    local nombre2="$3"

    # Vérification du nombre de paramètres
    if [ $# -ne 3 ]; then
        echo "Usage: calculer OPERATION NOMBRE1 NOMBRE2"
        echo "Opérations: add, sub, mul, div"
        return 1
    fi

    # Vérification que ce sont des nombres
    if ! [[ "$nombre1" =~ ^[0-9]+$ ]] || ! [[ "$nombre2" =~ ^[0-9]+$ ]]; then
        echo "❌ Les paramètres doivent être des nombres entiers"
        return 1
    fi

    local resultat
    case "$operation" in
        add|+)
            resultat=$((nombre1 + nombre2))
            echo "$nombre1 + $nombre2 = $resultat"
            ;;
        sub|-)
            resultat=$((nombre1 - nombre2))
            echo "$nombre1 - $nombre2 = $resultat"
            ;;
        mul|x|\*)
            resultat=$((nombre1 * nombre2))
            echo "$nombre1 × $nombre2 = $resultat"
            ;;
        div|/)
            if [ "$nombre2" -eq 0 ]; then
                echo "❌ Division par zéro impossible"
                return 1
            fi
            resultat=$((nombre1 / nombre2))
            reste=$((nombre1 % nombre2))
            echo "$nombre1 ÷ $nombre2 = $resultat (reste: $reste)"
            ;;
        *)
            echo "❌ Opération inconnue: $operation"
            return 1
            ;;
    esac

    return 0
}

# Tests de la calculatrice
echo "=== Tests de la calculatrice ==="
calculer add 15 25
calculer sub 50 20
calculer mul 6 7
calculer div 20 3
calculer div 10 0  # Test d'erreur
calculer xyz 5 5   # Test d'erreur
```

### Valeurs de retour

En Bash, les fonctions peuvent retourner :
1. **Un code de statut** (0-255) avec `return`
2. **Du texte** via `echo` ou `printf`
3. **Modifier des variables globales**

```bash
#!/bin/bash

# Méthode 1 : Retour de code de statut
est_nombre_pair() {
    local nombre="$1"

    if [ $((nombre % 2)) -eq 0 ]; then
        return 0  # Vrai (pair)
    else
        return 1  # Faux (impair)
    fi
}

# Méthode 2 : Retour de texte
obtenir_jour_semaine() {
    local numero="$1"

    case "$numero" in
        1) echo "Lundi" ;;
        2) echo "Mardi" ;;
        3) echo "Mercredi" ;;
        4) echo "Jeudi" ;;
        5) echo "Vendredi" ;;
        6) echo "Samedi" ;;
        7) echo "Dimanche" ;;
        *) echo "Invalide" ;;
    esac
}

# Méthode 3 : Modifier une variable globale
resultat_global=""
calculer_carre() {
    local nombre="$1"
    resultat_global=$((nombre * nombre))
}

# Tests des différentes méthodes
echo "=== Test des valeurs de retour ==="

# Test 1 : Code de statut
if est_nombre_pair 8; then
    echo "8 est pair"
else
    echo "8 est impair"
fi

# Test 2 : Récupération de texte
jour=$(obtenir_jour_semaine 3)
echo "Le jour 3 est : $jour"

# Test 3 : Variable globale
calculer_carre 5
echo "Le carré de 5 est : $resultat_global"
```

### Fonction de validation avancée

```bash
#!/bin/bash

# Fonction de validation d'email (simple)
valider_email() {
    local email="$1"

    # Vérifications basiques
    if [ -z "$email" ]; then
        echo "Email vide"
        return 1
    fi

    if [[ ! "$email" =~ @ ]]; then
        echo "Email sans @"
        return 1
    fi

    if [[ ! "$email" =~ \. ]]; then
        echo "Email sans point"
        return 1
    fi

    # Plus de vérifications...
    local partie_avant="${email%@*}"
    local partie_apres="${email#*@}"

    if [ ${#partie_avant} -lt 1 ]; then
        echo "Partie avant @ trop courte"
        return 1
    fi

    if [ ${#partie_apres} -lt 3 ]; then
        echo "Partie après @ trop courte"
        return 1
    fi

    echo "Email valide"
    return 0
}

# Fonction pour obtenir des informations utilisateur
saisir_utilisateur() {
    local nom=""
    local email=""
    local age=""

    # Saisie du nom
    while [ -z "$nom" ]; do
        read -p "Nom complet : " nom
        if [ -z "$nom" ]; then
            echo "❌ Le nom ne peut pas être vide"
        fi
    done

    # Saisie de l'email
    while true; do
        read -p "Email : " email
        if valider_email "$email" >/dev/null; then
            break
        else
            echo "❌ $(valider_email "$email")"
        fi
    done

    # Saisie de l'âge
    while true; do
        read -p "Âge : " age
        if [[ "$age" =~ ^[0-9]+$ ]] && [ "$age" -ge 1 ] && [ "$age" -le 120 ]; then
            break
        else
            echo "❌ Âge invalide (1-120)"
        fi
    done

    # Retourner les informations (via echo)
    echo "$nom|$email|$age"
}

# Test de saisie
echo "=== Saisie d'utilisateur ==="
# infos=$(saisir_utilisateur)
# echo "Informations saisies : $infos"
```

## Variables locales dans les fonctions

### Pourquoi utiliser des variables locales ?

**Problème sans variables locales :**

```bash
#!/bin/bash

# Variables globales (problématique)
nom="Alice"
age=30

modifier_utilisateur() {
    nom="Bob"      # Modifie la variable globale !
    age=25         # Modifie la variable globale !
    echo "Dans la fonction : $nom, $age ans"
}

echo "Avant : $nom, $age ans"
modifier_utilisateur
echo "Après : $nom, $age ans"  # Oups ! Modifié par erreur
```

**Solution avec variables locales :**

```bash
#!/bin/bash

# Variables globales protégées
nom="Alice"
age=30

modifier_utilisateur_local() {
    local nom="Bob"      # Variable locale
    local age=25         # Variable locale
    echo "Dans la fonction : $nom, $age ans"
}

echo "Avant : $nom, $age ans"
modifier_utilisateur_local
echo "Après : $nom, $age ans"  # Inchangé !
```

### Portée des variables - Exemple détaillé

```bash
#!/bin/bash

# Variables globales
compteur_global=0
message="Message global"

demo_portee() {
    local compteur_local=100
    local message="Message local"

    echo "=== Dans la fonction ==="
    echo "Compteur local : $compteur_local"
    echo "Message local : $message"
    echo "Compteur global : $compteur_global"

    # Modifier les variables
    compteur_local=200
    compteur_global=50

    echo "Après modification dans la fonction :"
    echo "Compteur local : $compteur_local"
    echo "Compteur global : $compteur_global"
}

echo "=== Avant appel de fonction ==="
echo "Compteur global : $compteur_global"
echo "Message global : $message"

demo_portee

echo ""
echo "=== Après appel de fonction ==="
echo "Compteur global : $compteur_global"  # Modifié
echo "Message global : $message"           # Inchangé
# echo "Compteur local : $compteur_local"  # Erreur ! Variable inexistante
```

### Bonnes pratiques avec les variables locales

```bash
#!/bin/bash

# Fonction bien structurée
traiter_fichier() {
    # 1. Déclarer TOUTES les variables locales au début
    local fichier="$1"
    local ligne_count=0
    local mot_count=0
    local taille_fichier=0
    local extension=""

    # 2. Validation des paramètres
    if [ $# -ne 1 ]; then
        echo "Usage: traiter_fichier FICHIER"
        return 1
    fi

    if [ ! -f "$fichier" ]; then
        echo "❌ Fichier '$fichier' introuvable"
        return 1
    fi

    # 3. Traitement principal
    ligne_count=$(wc -l < "$fichier")
    mot_count=$(wc -w < "$fichier")
    taille_fichier=$(stat -c%s "$fichier")
    extension="${fichier##*.}"

    # 4. Affichage des résultats
    echo "=== Analyse de '$fichier' ==="
    echo "Lignes : $ligne_count"
    echo "Mots : $mot_count"
    echo "Taille : $taille_fichier octets"
    echo "Extension : .$extension"

    return 0
}

# Fonction de configuration avec variables locales
configurer_application() {
    local nom_app="$1"
    local version="$2"
    local repertoire_config="$HOME/.config/$nom_app"
    local fichier_config="$repertoire_config/config.conf"
    local fichier_log="$repertoire_config/app.log"

    echo "=== Configuration de $nom_app v$version ==="

    # Créer les répertoires
    if mkdir -p "$repertoire_config"; then
        echo "✅ Répertoire créé : $repertoire_config"
    else
        echo "❌ Impossible de créer le répertoire"
        return 1
    fi

    # Créer le fichier de configuration
    cat > "$fichier_config" << EOF
# Configuration de $nom_app
version=$version
date_creation=$(date)
repertoire_donnees=$repertoire_config/data
niveau_log=INFO
EOF

    echo "✅ Configuration créée : $fichier_config"

    # Créer le fichier de log
    echo "$(date) - Application $nom_app initialisée" > "$fichier_log"
    echo "✅ Log initialisé : $fichier_log"

    return 0
}

# Tests
echo "Contenu de test" > test.txt
traiter_fichier "test.txt"

echo ""
configurer_application "MonApp" "1.0.0"
```

### Variables read-only et constantes

```bash
#!/bin/bash

# Constantes globales (read-only)
readonly VERSION="1.0.0"
readonly APP_NAME="MonScript"
readonly CONFIG_DIR="$HOME/.monscript"

# Fonction avec constantes locales
calculer_avec_constantes() {
    local readonly PI=3.14159
    local readonly GRAVITY=9.81
    local rayon="$1"

    if [ -z "$rayon" ]; then
        echo "Usage: calculer_avec_constantes RAYON"
        return 1
    fi

    # Calculs
    local circonference=$(echo "$rayon * 2 * $PI" | bc -l)
    local aire=$(echo "$rayon * $rayon * $PI" | bc -l)

    echo "=== Calculs pour un cercle de rayon $rayon ==="
    echo "Circonférence : $circonference"
    echo "Aire : $aire"

    # PI=3.14  # Erreur ! Variable read-only

    return 0
}

# Fonction d'information
afficher_info_app() {
    local readonly BUILD_DATE=$(date)

    echo "=== Informations de l'application ==="
    echo "Nom : $APP_NAME"
    echo "Version : $VERSION"
    echo "Date de compilation : $BUILD_DATE"
    echo "Répertoire de config : $CONFIG_DIR"
}

# Tests
calculer_avec_constantes 5
echo ""
afficher_info_app
```

## Fonctions récursives

### Qu'est-ce que la récursion ?

La récursion, c'est quand une fonction **s'appelle elle-même**. C'est comme des poupées russes : chaque poupée contient une poupée plus petite.

**Analogie :** Pour descendre un escalier :
1. Si je suis en bas → stop
2. Sinon → descendre une marche, puis refaire l'étape 1

### Exemple simple : Compte à rebours

```bash
#!/bin/bash

# Fonction récursive de compte à rebours
compte_rebours() {
    local nombre="$1"

    # Condition d'arrêt (cas de base)
    if [ "$nombre" -le 0 ]; then
        echo "🚀 Décollage !"
        return 0
    fi

    # Afficher le nombre actuel
    echo "$nombre..."
    sleep 1

    # Appel récursif avec nombre-1
    compte_rebours $((nombre - 1))
}

# Test
echo "=== Compte à rebours ==="
compte_rebours 5
```

### Calcul factorielle (exemple classique)

```bash
#!/bin/bash

# Factorielle récursive
# factorielle(n) = n × factorielle(n-1)
# factorielle(0) = 1 (cas de base)
factorielle() {
    local n="$1"

    # Validation
    if [ -z "$n" ] || ! [[ "$n" =~ ^[0-9]+$ ]]; then
        echo "Erreur: nombre entier requis"
        return 1
    fi

    # Cas de base
    if [ "$n" -le 1 ]; then
        echo 1
        return 0
    fi

    # Cas récursif
    local resultat_precedent=$(factorielle $((n - 1)))
    local resultat=$((n * resultat_precedent))

    echo "$resultat"
}

# Tests de factorielle
echo "=== Calcul de factorielles ==="
for i in {0..6}; do
    resultat=$(factorielle $i)
    echo "$i! = $resultat"
done
```

### Suite de Fibonacci

```bash
#!/bin/bash

# Suite de Fibonacci récursive
# fibonacci(n) = fibonacci(n-1) + fibonacci(n-2)
# fibonacci(0) = 0, fibonacci(1) = 1
fibonacci() {
    local n="$1"

    # Validation
    if [ -z "$n" ] || ! [[ "$n" =~ ^[0-9]+$ ]]; then
        echo "Erreur: nombre entier requis"
        return 1
    fi

    # Cas de base
    if [ "$n" -eq 0 ]; then
        echo 0
        return 0
    elif [ "$n" -eq 1 ]; then
        echo 1
        return 0
    fi

    # Cas récursif
    local fib_n1=$(fibonacci $((n - 1)))
    local fib_n2=$(fibonacci $((n - 2)))
    local resultat=$((fib_n1 + fib_n2))

    echo "$resultat"
}

# Affichage de la suite de Fibonacci
echo "=== Suite de Fibonacci ==="
echo -n "Fibonacci: "
for i in {0..10}; do
    if [ $i -eq 0 ]; then
        echo -n "$(fibonacci $i)"
    else
        echo -n ", $(fibonacci $i)"
    fi
done
echo ""
```

### Fibonacci optimisé avec mémoïsation

```bash
#!/bin/bash

# Version optimisée avec cache (mémoïsation)
declare -A cache_fibonacci

fibonacci_memo() {
    local n="$1"

    # Vérifier le cache
    if [ -n "${cache_fibonacci[$n]}" ]; then
        echo "${cache_fibonacci[$n]}"
        return 0
    fi

    local resultat

    # Cas de base
    if [ "$n" -eq 0 ]; then
        resultat=0
    elif [ "$n" -eq 1 ]; then
        resultat=1
    else
        # Cas récursif
        local fib_n1=$(fibonacci_memo $((n - 1)))
        local fib_n2=$(fibonacci_memo $((n - 2)))
        resultat=$((fib_n1 + fib_n2))
    fi

    # Sauvegarder dans le cache
    cache_fibonacci[$n]=$resultat
    echo "$resultat"
}

# Comparaison de performance
echo "=== Comparaison de performance ==="

echo "Fibonacci normal (n=30):"
time fibonacci 30 >/dev/null

echo "Fibonacci avec cache (n=30):"
time fibonacci_memo 30 >/dev/null
```

### Parcours récursif de répertoires

```bash
#!/bin/bash

# Fonction récursive pour parcourir des répertoires
parcourir_repertoire() {
    local repertoire="$1"
    local niveau="${2:-0}"  # Niveau par défaut 0
    local prefix=""

    # Créer l'indentation selon le niveau
    for ((i=0; i<niveau; i++)); do
        prefix+="  "
    done

    # Parcourir tous les éléments du répertoire
    for element in "$repertoire"/*; do
        # Ignorer si le motif n'a pas de correspondance
        [ ! -e "$element" ] && continue

        local nom=$(basename "$element")

        if [ -d "$element" ]; then
            echo "${prefix}📁 $nom/"

            # Appel récursif pour les sous-répertoires
            # (avec limite de profondeur pour éviter l'infini)
            if [ $niveau -lt 3 ]; then
                parcourir_repertoire "$element" $((niveau + 1))
            else
                echo "${prefix}  ... (profondeur limitée)"
            fi
        else
            echo "${prefix}📄 $nom"
        fi
    done
}

# Test avec le répertoire actuel
echo "=== Structure du répertoire actuel ==="
parcourir_repertoire "."
```

### Résolution de problème : Tours de Hanoï

```bash
#!/bin/bash

# Résolution récursive des Tours de Hanoï
tours_hanoi() {
    local n="$1"        # Nombre de disques
    local source="$2"   # Tour source
    local dest="$3"     # Tour destination
    local temp="$4"     # Tour temporaire

    # Cas de base : un seul disque
    if [ "$n" -eq 1 ]; then
        echo "Déplacer disque 1 de $source vers $dest"
        return 0
    fi

    # Étape 1 : Déplacer n-1 disques vers la tour temporaire
    tours_hanoi $((n-1)) "$source" "$temp" "$dest"

    # Étape 2 : Déplacer le plus gros disque vers la destination
    echo "Déplacer disque $n de $source vers $dest"

    # Étape 3 : Déplacer n-1 disques de la tour temporaire vers la destination
    tours_hanoi $((n-1)) "$temp" "$dest" "$source"
}

# Fonction pour calculer le nombre minimum de mouvements
calculer_mouvements() {
    local n="$1"
    echo $((2**n - 1))
}

# Test des Tours de Hanoï
echo "=== Tours de Hanoï ==="
n=3
echo "Résolution pour $n disques :"
echo "Nombre minimum de mouvements : $(calculer_mouvements $n)"
echo ""
tours_hanoi $n "A" "C" "B"
```

### Validation récursive de structure

```bash
#!/bin/bash

# Fonction récursive pour valider une structure de fichiers
valider_structure() {
    local repertoire="$1"
    local niveau="${2:-0}"
    local erreurs=0

    # Vérifications selon le niveau
    case $niveau in
        0)  # Racine du projet
            if [ ! -f "$repertoire/README.md" ]; then
                echo "❌ README.md manquant à la racine"
                ((erreurs++))
            fi
            if [ ! -d "$repertoire/src" ]; then
                echo "❌ Répertoire src/ manquant"
                ((erreurs++))
            fi
            ;;
        1)  # Premier niveau
            local nom=$(basename "$repertoire")
            if [ "$nom" = "src" ]; then
                if [ ! -f "$repertoire/main.sh" ]; then
                    echo "❌ main.sh manquant dans src/"
                    ((erreurs++))
                fi
            fi
            ;;
    esac

    # Parcourir récursivement les sous-répertoires
    for element in "$repertoire"/*; do
        if [ -d "$element" ] && [ $niveau -lt 2 ]; then
            local sous_erreurs=$(valider_structure "$element" $((niveau + 1)))
            erreurs=$((erreurs + sous_erreurs))
        fi
    done

    echo $erreurs
}

# Créer une structure de test
mkdir -p test_projet/src
touch test_projet/README.md
touch test_projet/src/main.sh

# Test de validation
echo "=== Validation de structure de projet ==="
erreurs=$(valider_structure "test_projet")
if [ "$erreurs" -eq 0 ]; then
    echo "✅ Structure valide"
else
    echo "❌ $erreurs erreur(s) trouvée(s)"
fi

# Nettoyer
rm -rf test_projet
```

### Exercices pratiques

```bash
#!/bin/bash

# Exercice 1 : Calculateur de puissance récursif
# puissance(base, exposant) = base × puissance(base, exposant-1)
# puissance(base, 0) = 1
puissance() {
    local base="$1"
    local exposant="$2"

    # À compléter...
}

# Exercice 2 : Somme des chiffres d'un nombre
# somme_chiffres(123) = 1 + 2 + 3 = 6
somme_chiffres() {
    local nombre="$1"

    # À compléter...
}

# Exercice 3 : Inverser une chaîne
# inverser("hello") = "olleh"
inverser_chaine() {
    local chaine="$1"

    # À compléter...
}
```

## Solutions des exercices

### Solution Exercice 1 : Puissance

```bash
puissance() {
    local base="$1"
    local exposant="$2"

    # Validation
    if [ -z "$base" ] || [ -z "$exposant" ]; then
        echo "Usage: puissance BASE EXPOSANT"
        return 1
    fi

    if ! [[ "$base" =~ ^[0-9]+$ ]] || ! [[ "$exposant" =~ ^[0-9]+$ ]]; then
        echo "❌ Les paramètres doivent être des nombres entiers positifs"
        return 1
    fi

    # Cas de base
    if [ "$exposant" -eq 0 ]; then
        echo 1
        return 0
    fi

    # Cas récursif
    local puissance_precedente=$(puissance "$base" $((exposant - 1)))
    local resultat=$((base * puissance_precedente))

    echo "$resultat"
}

# Test de la fonction puissance
echo "=== Test de la fonction puissance ==="
for base in 2 3 5; do
    for exp in 0 1 2 3 4; do
        resultat=$(puissance $base $exp)
        echo "$base^$exp = $resultat"
    done
    echo ""
done
```

### Solution Exercice 2 : Somme des chiffres

```bash
somme_chiffres() {
    local nombre="$1"

    # Validation
    if [ -z "$nombre" ]; then
        echo "Usage: somme_chiffres NOMBRE"
        return 1
    fi

    if ! [[ "$nombre" =~ ^[0-9]+$ ]]; then
        echo "❌ Le paramètre doit être un nombre entier positif"
        return 1
    fi

    # Cas de base : nombre à un seul chiffre
    if [ "$nombre" -lt 10 ]; then
        echo "$nombre"
        return 0
    fi

    # Cas récursif
    local dernier_chiffre=$((nombre % 10))
    local nombre_restant=$((nombre / 10))
    local somme_restante=$(somme_chiffres "$nombre_restant")
    local resultat=$((dernier_chiffre + somme_restante))

    echo "$resultat"
}

# Test de la fonction somme des chiffres
echo "=== Test de la fonction somme des chiffres ==="
nombres=(123 456 789 1234 9876)
for nombre in "${nombres[@]}"; do
    resultat=$(somme_chiffres "$nombre")
    echo "Somme des chiffres de $nombre = $resultat"
done
```

### Solution Exercice 3 : Inverser une chaîne

```bash
inverser_chaine() {
    local chaine="$1"

    # Validation
    if [ -z "$chaine" ]; then
        echo ""  # Chaîne vide retourne chaîne vide
        return 0
    fi

    # Cas de base : chaîne d'un caractère
    if [ ${#chaine} -eq 1 ]; then
        echo "$chaine"
        return 0
    fi

    # Cas récursif
    local premier_char="${chaine:0:1}"
    local reste="${chaine:1}"
    local reste_inverse=$(inverser_chaine "$reste")

    echo "${reste_inverse}${premier_char}"
}

# Test de la fonction inverser chaîne
echo "=== Test de la fonction inverser chaîne ==="
chaines=("hello" "world" "bash" "recursion" "a" "")
for chaine in "${chaines[@]}"; do
    resultat=$(inverser_chaine "$chaine")
    echo "'$chaine' inversé = '$resultat'"
done
```

## Exemples avancés de fonctions

### Système de cache pour fonctions

```bash
#!/bin/bash

# Système de cache global pour optimiser les fonctions récursives
declare -A cache_general

# Fonction générique de cache
obtenir_du_cache() {
    local cle="$1"
    echo "${cache_general[$cle]}"
}

sauver_dans_cache() {
    local cle="$1"
    local valeur="$2"
    cache_general[$cle]="$valeur"
}

# Fibonacci avec cache optimisé
fibonacci_cache() {
    local n="$1"
    local cle="fib_$n"

    # Vérifier le cache
    local resultat_cache=$(obtenir_du_cache "$cle")
    if [ -n "$resultat_cache" ]; then
        echo "$resultat_cache"
        return 0
    fi

    local resultat
    if [ "$n" -eq 0 ]; then
        resultat=0
    elif [ "$n" -eq 1 ]; then
        resultat=1
    else
        local fib_n1=$(fibonacci_cache $((n - 1)))
        local fib_n2=$(fibonacci_cache $((n - 2)))
        resultat=$((fib_n1 + fib_n2))
    fi

    sauver_dans_cache "$cle" "$resultat"
    echo "$resultat"
}

# Fonction de combinaisons (n choose k)
combinaison() {
    local n="$1"
    local k="$2"
    local cle="comb_${n}_${k}"

    # Vérifier le cache
    local resultat_cache=$(obtenir_du_cache "$cle")
    if [ -n "$resultat_cache" ]; then
        echo "$resultat_cache"
        return 0
    fi

    local resultat

    # Cas de base
    if [ "$k" -eq 0 ] || [ "$k" -eq "$n" ]; then
        resultat=1
    else
        # C(n,k) = C(n-1,k-1) + C(n-1,k)
        local comb1=$(combinaison $((n-1)) $((k-1)))
        local comb2=$(combinaison $((n-1)) "$k")
        resultat=$((comb1 + comb2))
    fi

    sauver_dans_cache "$cle" "$resultat"
    echo "$resultat"
}

# Tests du système de cache
echo "=== Tests avec système de cache ==="

echo "Fibonacci(35) avec cache :"
time fibonacci_cache 35

echo ""
echo "Triangle de Pascal (combinaisons) :"
for n in {0..6}; do
    echo -n "Ligne $n: "
    for k in $(seq 0 $n); do
        echo -n "$(combinaison $n $k) "
    done
    echo ""
done
```

### Fonctions de transformation de données

```bash
#!/bin/bash

# Fonction récursive pour appliquer une transformation à tous les éléments
mapper() {
    local transformation="$1"
    shift  # Enlever le premier argument
    local elements=("$@")

    # Cas de base : liste vide
    if [ ${#elements[@]} -eq 0 ]; then
        return 0
    fi

    # Appliquer la transformation au premier élément
    local premier="${elements[0]}"
    local resultat_transformation=$(eval "$transformation '$premier'")
    echo "$resultat_transformation"

    # Appel récursif sur le reste de la liste
    if [ ${#elements[@]} -gt 1 ]; then
        mapper "$transformation" "${elements[@]:1}"
    fi
}

# Fonctions de transformation
doubler() {
    echo $(($1 * 2))
}

mettre_en_majuscules() {
    echo "$1" | tr '[:lower:]' '[:upper:]'
}

ajouter_prefixe() {
    echo "prefix_$1"
}

# Fonction récursive de filtrage
filtrer() {
    local condition="$1"
    shift
    local elements=("$@")

    # Cas de base : liste vide
    if [ ${#elements[@]} -eq 0 ]; then
        return 0
    fi

    local premier="${elements[0]}"

    # Tester la condition sur le premier élément
    if eval "$condition '$premier'"; then
        echo "$premier"
    fi

    # Appel récursif sur le reste
    if [ ${#elements[@]} -gt 1 ]; then
        filtrer "$condition" "${elements[@]:1}"
    fi
}

# Conditions de filtrage
est_pair() {
    [ $(($1 % 2)) -eq 0 ]
}

est_grand() {
    [ $1 -gt 50 ]
}

contient_a() {
    [[ "$1" =~ a ]]
}

# Tests des fonctions de transformation
echo "=== Tests de transformation de données ==="

echo "Doubler les nombres :"
nombres=(1 2 3 4 5)
mapper "doubler" "${nombres[@]}"

echo ""
echo "Mettre en majuscules :"
mots=("hello" "world" "bash" "script")
mapper "mettre_en_majuscules" "${mots[@]}"

echo ""
echo "Filtrer les nombres pairs :"
nombres=(1 2 3 4 5 6 7 8 9 10)
filtrer "est_pair" "${nombres[@]}"

echo ""
echo "Filtrer les mots contenant 'a' :"
mots=("chat" "chien" "oiseau" "poisson" "lapin")
filtrer "contient_a" "${mots[@]}"
```

### Analyseur syntaxique récursif simple

```bash
#!/bin/bash

# Analyseur récursif pour expressions arithmétiques simples
# Grammaire : expr = terme ('+' terme | '-' terme)*
#            terme = facteur ('*' facteur | '/' facteur)*
#            facteur = nombre | '(' expr ')'

# Variables globales pour l'analyseur
expression=""
position=0

# Fonction pour obtenir le caractère actuel
char_actuel() {
    if [ $position -lt ${#expression} ]; then
        echo "${expression:$position:1}"
    else
        echo ""
    fi
}

# Avancer d'une position
avancer() {
    ((position++))
}

# Ignorer les espaces
ignorer_espaces() {
    while [[ "$(char_actuel)" =~ [[:space:]] ]]; do
        avancer
    done
}

# Lire un nombre
lire_nombre() {
    local nombre=""
    while [[ "$(char_actuel)" =~ [0-9] ]]; do
        nombre+="$(char_actuel)"
        avancer
    done
    echo "$nombre"
}

# Analyser un facteur
analyser_facteur() {
    ignorer_espaces
    local char=$(char_actuel)

    if [[ "$char" =~ [0-9] ]]; then
        # C'est un nombre
        lire_nombre
    elif [ "$char" = "(" ]; then
        # Expression entre parenthèses
        avancer  # consommer '('
        local resultat=$(analyser_expression)
        ignorer_espaces
        if [ "$(char_actuel)" = ")" ]; then
            avancer  # consommer ')'
            echo "$resultat"
        else
            echo "Erreur: ')' attendu"
            return 1
        fi
    else
        echo "Erreur: nombre ou '(' attendu"
        return 1
    fi
}

# Analyser un terme
analyser_terme() {
    local gauche=$(analyser_facteur)

    while true; do
        ignorer_espaces
        local operateur=$(char_actuel)

        if [ "$operateur" = "*" ] || [ "$operateur" = "/" ]; then
            avancer
            local droite=$(analyser_facteur)

            if [ "$operateur" = "*" ]; then
                gauche=$((gauche * droite))
            else
                if [ "$droite" -eq 0 ]; then
                    echo "Erreur: division par zéro"
                    return 1
                fi
                gauche=$((gauche / droite))
            fi
        else
            break
        fi
    done

    echo "$gauche"
}

# Analyser une expression complète
analyser_expression() {
    local gauche=$(analyser_terme)

    while true; do
        ignorer_espaces
        local operateur=$(char_actuel)

        if [ "$operateur" = "+" ] || [ "$operateur" = "-" ]; then
            avancer
            local droite=$(analyser_terme)

            if [ "$operateur" = "+" ]; then
                gauche=$((gauche + droite))
            else
                gauche=$((gauche - droite))
            fi
        else
            break
        fi
    done

    echo "$gauche"
}

# Fonction principale d'évaluation
evaluer_expression() {
    expression="$1"
    position=0

    local resultat=$(analyser_expression)

    ignorer_espaces
    if [ $position -lt ${#expression} ]; then
        echo "Erreur: caractères inattendus à la fin"
        return 1
    fi

    echo "$resultat"
}

# Tests de l'analyseur
echo "=== Tests de l'analyseur d'expressions ==="
expressions=(
    "2 + 3"
    "10 - 4"
    "3 * 4"
    "15 / 3"
    "2 + 3 * 4"
    "(2 + 3) * 4"
    "10 - 2 * 3"
    "(10 - 2) * 3"
    "2 * (3 + 4)"
)

for expr in "${expressions[@]}"; do
    resultat=$(evaluer_expression "$expr")
    echo "$expr = $resultat"
done
```

### Système de plugins avec fonctions

```bash
#!/bin/bash

# Système de plugins utilisant des fonctions récursives

# Liste des plugins chargés
declare -a plugins_charges=()

# Charger un plugin
charger_plugin() {
    local nom_plugin="$1"

    # Vérifier si le plugin existe
    if declare -f "plugin_${nom_plugin}_init" >/dev/null; then
        # Initialiser le plugin
        "plugin_${nom_plugin}_init"
        plugins_charges+=("$nom_plugin")
        echo "✅ Plugin '$nom_plugin' chargé"
    else
        echo "❌ Plugin '$nom_plugin' introuvable"
        return 1
    fi
}

# Exécuter une commande sur tous les plugins
executer_sur_plugins() {
    local commande="$1"
    shift
    local arguments=("$@")

    for plugin in "${plugins_charges[@]}"; do
        local fonction="plugin_${plugin}_${commande}"
        if declare -f "$fonction" >/dev/null; then
            echo "=== Plugin $plugin : $commande ==="
            "$fonction" "${arguments[@]}"
        fi
    done
}

# Plugin de calcul
plugin_calc_init() {
    echo "Plugin Calculatrice initialisé"
}

plugin_calc_executer() {
    local operation="$1"
    local a="$2"
    local b="$3"

    case "$operation" in
        add) echo "$a + $b = $((a + b))" ;;
        sub) echo "$a - $b = $((a - b))" ;;
        mul) echo "$a × $b = $((a * b))" ;;
        div)
            if [ "$b" -ne 0 ]; then
                echo "$a ÷ $b = $((a / b))"
            else
                echo "Division par zéro impossible"
            fi
            ;;
        *) echo "Opération inconnue : $operation" ;;
    esac
}

# Plugin de fichiers
plugin_files_init() {
    echo "Plugin Gestionnaire de fichiers initialisé"
}

plugin_files_executer() {
    local action="$1"
    local fichier="$2"

    case "$action" in
        info)
            if [ -f "$fichier" ]; then
                echo "Fichier : $fichier"
                echo "Taille : $(stat -c%s "$fichier") octets"
                echo "Modifié : $(stat -c%y "$fichier")"
            else
                echo "Fichier '$fichier' introuvable"
            fi
            ;;
        create)
            touch "$fichier"
            echo "Fichier '$fichier' créé"
            ;;
        delete)
            if [ -f "$fichier" ]; then
                rm "$fichier"
                echo "Fichier '$fichier' supprimé"
            else
                echo "Fichier '$fichier' introuvable"
            fi
            ;;
    esac
}

# Plugin de texte
plugin_text_init() {
    echo "Plugin Traitement de texte initialisé"
}

plugin_text_executer() {
    local action="$1"
    local texte="$2"

    case "$action" in
        upper) echo "$texte" | tr '[:lower:]' '[:upper:]' ;;
        lower) echo "$texte" | tr '[:upper:]' '[:lower:]' ;;
        reverse) echo "$texte" | rev ;;
        length) echo "Longueur : ${#texte}" ;;
    esac
}

# Tests du système de plugins
echo "=== Système de plugins ==="

# Charger les plugins
charger_plugin "calc"
charger_plugin "files"
charger_plugin "text"

echo ""
echo "=== Test des plugins ==="

# Test plugin calc
executer_sur_plugins "executer" "add" "15" "25"

echo ""

# Test plugin text
executer_sur_plugins "executer" "upper" "hello world"

echo ""

# Créer un fichier de test pour le plugin files
echo "Test content" > test_plugin.txt
executer_sur_plugins "executer" "info" "test_plugin.txt"

# Nettoyer
rm -f test_plugin.txt
```

## Bonnes pratiques pour les fonctions

### Structure et documentation

```bash
#!/bin/bash

#############################################
# BONNES PRATIQUES POUR LES FONCTIONS
#############################################

# ✅ Fonction bien documentée
#############################################
# Calcule l'aire d'un rectangle
# Arguments:
#   $1 - Largeur (nombre entier positif)
#   $2 - Hauteur (nombre entier positif)
# Retourne:
#   0 - Succès (affiche l'aire)
#   1 - Erreur de paramètres
# Exemple:
#   aire_rectangle 10 20  # Affiche "200"
#############################################
aire_rectangle() {
    local largeur="$1"
    local hauteur="$2"
    local aire

    # Validation des paramètres
    if [ $# -ne 2 ]; then
        echo "Erreur: Exactement 2 paramètres requis" >&2
        echo "Usage: aire_rectangle LARGEUR HAUTEUR" >&2
        return 1
    fi

    if ! [[ "$largeur" =~ ^[0-9]+$ ]] || ! [[ "$hauteur" =~ ^[0-9]+$ ]]; then
        echo "Erreur: Les paramètres doivent être des nombres entiers positifs" >&2
        return 1
    fi

    # Calcul
    aire=$((largeur * hauteur))

    # Sortie
    echo "$aire"
    return 0
}

# ✅ Fonction avec gestion d'erreurs complète
#############################################
# Lit et valide un fichier de configuration
# Arguments:
#   $1 - Chemin du fichier de configuration
# Variables globales modifiées:
#   CONFIG_* - Variables de configuration
# Retourne:
#   0 - Configuration chargée avec succès
#   1 - Fichier introuvable
#   2 - Fichier invalide
#############################################
charger_configuration() {
    local fichier_config="$1"
    local ligne_num=0

    # Validation du fichier
    if [ ! -f "$fichier_config" ]; then
        echo "Erreur: Fichier de configuration '$fichier_config' introuvable" >&2
        return 1
    fi

    if [ ! -r "$fichier_config" ]; then
        echo "Erreur: Impossible de lire '$fichier_config'" >&2
        return 1
    fi

    # Lecture ligne par ligne
    while IFS='=' read -r cle valeur; do
        ((ligne_num++))

        # Ignorer les lignes vides et commentaires
        [[ "$cle" =~ ^[[:space:]]*$ ]] && continue
        [[ "$cle" =~ ^[[:space:]]*# ]] && continue

        # Valider le format
        if [[ ! "$cle" =~ ^[A-Z_][A-Z0-9_]*$ ]]; then
            echo "Erreur ligne $ligne_num: Clé invalide '$cle'" >&2
            return 2
        fi

        # Nettoyer et assigner
        cle=$(echo "$cle" | tr -d '[:space:]')
        valeur=$(echo "$valeur" | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')

        declare -g "CONFIG_$cle"="$valeur"
        echo "Configuration chargée: $cle = $valeur"

    done < "$fichier_config"

    echo "Configuration chargée avec succès ($ligne_num lignes)"
    return 0
}

# ✅ Fonction utilitaire réutilisable
#############################################
# Affiche un message avec horodatage et niveau
# Arguments:
#   $1 - Niveau (INFO, WARN, ERROR)
#   $2 - Message
# Variables d'environnement:
#   LOG_FILE - Fichier de log (optionnel)
#   LOG_LEVEL - Niveau minimum (optionnel)
#############################################
log_message() {
    local niveau="$1"
    local message="$2"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    local ligne_log="[$timestamp] [$niveau] $message"

    # Validation
    if [ $# -ne 2 ]; then
        echo "Usage: log_message NIVEAU MESSAGE" >&2
        return 1
    fi

    # Vérifier le niveau si défini
    case "${LOG_LEVEL:-INFO}" in
        ERROR)
            [[ "$niveau" != "ERROR" ]] && return 0
            ;;
        WARN)
            [[ "$niveau" != "ERROR" && "$niveau" != "WARN" ]] && return 0
            ;;
        INFO)
            # Afficher tous les niveaux
            ;;
    esac

    # Couleurs selon le niveau
    case "$niveau" in
        ERROR)
            echo -e "\033[31m$ligne_log\033[0m" >&2
            ;;
        WARN)
            echo -e "\033[33m$ligne_log\033[0m"
            ;;
        INFO)
            echo "$ligne_log"
            ;;
        *)
            echo "$ligne_log"
            ;;
    esac

    # Écrire dans le fichier de log si défini
    if [ -n "$LOG_FILE" ]; then
        echo "$ligne_log" >> "$LOG_FILE"
    fi
}

# Tests des bonnes pratiques
echo "=== Tests des bonnes pratiques ==="

# Test fonction aire
echo "Test aire rectangle :"
aire_rectangle 10 20
aire_rectangle 5    # Erreur volontaire
echo ""

# Test logging
LOG_LEVEL="INFO"
log_message "INFO" "Démarrage de l'application"
log_message "WARN" "Attention: fichier manquant"
log_message "ERROR" "Erreur critique détectée"
```

## Récapitulatif

✅ **Ce que vous avez appris dans ce chapitre :**

1. **Définition de fonctions** : Syntaxe, organisation, réutilisabilité
2. **Paramètres et retours** : Passage d'arguments, codes de retour, récupération de valeurs
3. **Variables locales** : Portée, protection des variables globales
4. **Récursion** : Fonctions qui s'appellent elles-mêmes, optimisation avec cache

✅ **Points clés à retenir :**

**Définition de fonction :**
```bash
nom_fonction() {
    local variable_locale="valeur"
    # traitement
    echo "résultat"
    return 0
}
```

**Appel de fonction :**
```bash
resultat=$(nom_fonction "param1" "param2")
if nom_fonction "test"; then
    echo "Succès"
fi
```

**Variables locales :**
```bash
ma_fonction() {
    local param="$1"        # Toujours local
    local resultat=""       # Déclarer au début
    # traitement...
}
```

**Récursion :**
```bash
fonction_recursive() {
    # Cas de base (condition d'arrêt)
    if [ condition_base ]; then
        echo "valeur_base"
        return 0
    fi

    # Cas récursif
    fonction_recursive parametre_modifie
}
```

✅ **Bonnes pratiques :**
- Toujours déclarer les variables en `local`
- Valider les paramètres en début de fonction
- Documenter les fonctions complexes
- Utiliser des noms de fonctions descriptifs
- Gérer les erreurs avec des codes de retour
- Limiter la profondeur de récursion

✅ **Prochaines étapes :**
Dans le chapitre 7, nous découvrirons la manipulation de chaînes de caractères et les expressions régulières !

---

💡 **Conseil pratique :** Les fonctions sont essentielles pour créer du code maintenable. Commencez par transformer vos scripts en petites fonctions réutilisables, puis progressez vers la récursion quand vous en avez besoin.

⏭️
