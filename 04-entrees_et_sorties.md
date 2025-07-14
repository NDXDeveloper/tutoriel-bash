🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Tutoriel Bash - Chapitre 4 : Entrées et sorties

## Lecture des arguments de ligne de commande

### Qu'est-ce qu'un argument de ligne de commande ?

Les arguments de ligne de commande sont comme des **ingrédients que vous donnez à votre script** pour qu'il sache quoi cuisiner. Quand vous tapez une commande, tout ce qui suit le nom du script sont des arguments.

**Analogie :** Si votre script est un chef cuisinier, les arguments sont les ingrédients qu'il reçoit :
```bash
./mon_script.sh pomme banane orange
#              ↑     ↑      ↑
#              $1    $2     $3
```

### Exemple de base

Créons un script `salutations.sh` :

```bash
#!/bin/bash

# Vérifier si des arguments ont été fournis
if [ $# -eq 0 ]; then
    echo "Usage : $0 prénom [nom] [âge]"
    echo "Exemple : $0 Marie Dupont 25"
    exit 1
fi

# Utiliser les arguments
prenom=$1
nom=${2:-""}  # Valeur par défaut vide si pas fourni
age=${3:-"inconnu"}  # Valeur par défaut "inconnu"

echo "=== Salutations ==="
echo "Bonjour $prenom $nom !"
echo "Âge : $age ans"
echo "Script appelé : $0"
echo "Nombre d'arguments : $#"
```

**Test du script :**
```bash
chmod +x salutations.sh

# Test avec tous les arguments
./salutations.sh Marie Dupont 25

# Test avec arguments partiels
./salutations.sh Jean

# Test sans arguments
./salutations.sh
```

### Gestion avancée des arguments

```bash
#!/bin/bash

# Script de copie de fichiers amélioré
script_name=$(basename "$0")

# Fonction d'aide
afficher_aide() {
    echo "Usage : $script_name SOURCE DESTINATION"
    echo ""
    echo "Copie un fichier de SOURCE vers DESTINATION"
    echo ""
    echo "Arguments :"
    echo "  SOURCE      Fichier à copier"
    echo "  DESTINATION Où copier le fichier"
    echo ""
    echo "Options :"
    echo "  -h, --help  Afficher cette aide"
    echo ""
    echo "Exemples :"
    echo "  $script_name document.txt backup/document.txt"
    echo "  $script_name photo.jpg /tmp/photo_copie.jpg"
}

# Vérification des arguments
case "$1" in
    -h|--help)
        afficher_aide
        exit 0
        ;;
    "")
        echo "Erreur : Arguments manquants"
        echo "Utilisez '$script_name --help' pour voir l'aide"
        exit 1
        ;;
esac

if [ $# -lt 2 ]; then
    echo "Erreur : Il faut au moins 2 arguments"
    echo "Usage : $script_name SOURCE DESTINATION"
    exit 1
fi

# Variables pour les arguments
fichier_source="$1"
fichier_destination="$2"

# Vérifications
if [ ! -f "$fichier_source" ]; then
    echo "Erreur : Le fichier source '$fichier_source' n'existe pas"
    exit 1
fi

# Copie du fichier
echo "Copie de '$fichier_source' vers '$fichier_destination'..."
if cp "$fichier_source" "$fichier_destination"; then
    echo "Copie réussie !"
else
    echo "Erreur lors de la copie"
    exit 1
fi
```

### Boucle sur tous les arguments

```bash
#!/bin/bash

# Script qui traite tous les arguments fournis
echo "=== Traitement de tous les arguments ==="
echo "Script : $0"
echo "Nombre d'arguments : $#"
echo ""

# Méthode 1 : Parcours avec $@
echo "--- Méthode 1 : avec \$@ ---"
for argument in "$@"; do
    echo "Argument : '$argument'"
done

echo ""

# Méthode 2 : Parcours avec indices
echo "--- Méthode 2 : avec indices ---"
for i in $(seq 1 $#); do
    echo "Argument $i : '${!i}'"
done

echo ""

# Méthode 3 : Avec décalage (shift)
echo "--- Méthode 3 : avec shift ---"
compteur=1
while [ $# -gt 0 ]; do
    echo "Argument $compteur : '$1'"
    shift  # Décale les arguments vers la gauche
    ((compteur++))
done
```

### Arguments avec options (introduction simple)

```bash
#!/bin/bash

# Script de sauvegarde simple avec options
repertoire_sauvegarde="$HOME/backup"
verbose=false

# Parcourir les arguments
while [ $# -gt 0 ]; do
    case "$1" in
        -v|--verbose)
            verbose=true
            echo "Mode verbose activé"
            ;;
        -d|--directory)
            shift  # Passer à l'argument suivant
            repertoire_sauvegarde="$1"
            echo "Répertoire de sauvegarde : $repertoire_sauvegarde"
            ;;
        -h|--help)
            echo "Usage : $0 [-v] [-d répertoire] fichier1 fichier2 ..."
            exit 0
            ;;
        -*)
            echo "Option inconnue : $1"
            exit 1
            ;;
        *)
            # Fichier à sauvegarder
            if [ "$verbose" = true ]; then
                echo "Sauvegarde de : $1"
            fi
            # Ici on ferait la sauvegarde...
            ;;
    esac
    shift
done
```

## Commande read pour l'interaction utilisateur

### Introduction à read

La commande `read` permet de **demander des informations à l'utilisateur** pendant l'exécution du script. C'est comme poser une question et attendre la réponse.

### Syntaxe de base

```bash
#!/bin/bash

# Demander le nom
echo "Comment vous appelez-vous ?"
read nom
echo "Bonjour $nom !"

# Demander l'âge
echo "Quel âge avez-vous ?"
read age
echo "Vous avez $age ans."
```

### read avec prompt (-p)

```bash
#!/bin/bash

# Plus élégant avec -p (prompt)
read -p "Votre nom : " nom
read -p "Votre âge : " age
read -p "Votre ville : " ville

echo ""
echo "=== Résumé ==="
echo "Nom : $nom"
echo "Âge : $age"
echo "Ville : $ville"
```

### Options utiles de read

```bash
#!/bin/bash

echo "=== Démonstration des options de read ==="

# 1. Lecture silencieuse (pour mots de passe)
read -s -p "Entrez votre mot de passe : " mot_de_passe
echo ""  # Nouvelle ligne après la saisie silencieuse
echo "Mot de passe saisi (ne l'affichons pas !)"

echo ""

# 2. Lecture avec timeout
echo "Vous avez 5 secondes pour répondre..."
if read -t 5 -p "Votre couleur préférée : " couleur; then
    echo "Votre couleur : $couleur"
else
    echo ""
    echo "Temps écoulé ! Couleur par défaut : bleu"
    couleur="bleu"
fi

echo ""

# 3. Lecture d'un seul caractère
echo "Appuyez sur une touche pour continuer..."
read -n 1 -s touche
echo "Touche pressée : '$touche'"

echo ""

# 4. Lecture avec valeur par défaut
read -p "Votre pays [France] : " pays
pays=${pays:-France}  # Valeur par défaut
echo "Pays : $pays"
```

### Validation des entrées utilisateur

```bash
#!/bin/bash

# Fonction pour demander un nombre
demander_nombre() {
    local prompt="$1"
    local variable_name="$2"
    local nombre

    while true; do
        read -p "$prompt" nombre

        # Vérifier si c'est un nombre
        if [[ "$nombre" =~ ^[0-9]+$ ]]; then
            eval "$variable_name=$nombre"
            break
        else
            echo "Erreur : Veuillez entrer un nombre valide."
        fi
    done
}

# Fonction pour demander oui/non
demander_oui_non() {
    local prompt="$1"
    local reponse

    while true; do
        read -p "$prompt (o/n) : " reponse
        case "$reponse" in
            [oO]|[oO][uU][iI])
                return 0  # Vrai
                ;;
            [nN]|[nN][oO][nN])
                return 1  # Faux
                ;;
            *)
                echo "Répondez par 'oui' ou 'non' (o/n)"
                ;;
        esac
    done
}

# Exemple d'utilisation
echo "=== Calculateur d'âge ==="

demander_nombre "Votre année de naissance : " annee_naissance
age_actuel=$((2025 - annee_naissance))

echo "Vous avez $age_actuel ans."

if demander_oui_non "Voulez-vous connaître votre âge en 2030 ?"; then
    age_2030=$((2030 - annee_naissance))
    echo "En 2030, vous aurez $age_2030 ans."
else
    echo "D'accord, bonne journée !"
fi
```

### Menu interactif simple

```bash
#!/bin/bash

# Menu de gestion de fichiers simple
afficher_menu() {
    echo ""
    echo "=== Gestionnaire de fichiers ==="
    echo "1. Lister les fichiers"
    echo "2. Créer un fichier"
    echo "3. Supprimer un fichier"
    echo "4. Quitter"
    echo ""
}

while true; do
    afficher_menu
    read -p "Votre choix (1-4) : " choix

    case "$choix" in
        1)
            echo "=== Fichiers du répertoire actuel ==="
            ls -la
            ;;
        2)
            read -p "Nom du fichier à créer : " nom_fichier
            touch "$nom_fichier"
            echo "Fichier '$nom_fichier' créé."
            ;;
        3)
            read -p "Nom du fichier à supprimer : " nom_fichier
            if [ -f "$nom_fichier" ]; then
                rm "$nom_fichier"
                echo "Fichier '$nom_fichier' supprimé."
            else
                echo "Fichier '$nom_fichier' introuvable."
            fi
            ;;
        4)
            echo "Au revoir !"
            exit 0
            ;;
        *)
            echo "Choix invalide. Veuillez choisir entre 1 et 4."
            ;;
    esac

    read -p "Appuyez sur Entrée pour continuer..." -s
done
```

## Redirection des flux (stdin, stdout, stderr)

### Comprendre les flux

En informatique, un programme a **trois "tuyaux"** pour communiquer :

**Analogie :** Imaginez un robot avec trois tubes :
- **stdin (0)** : Tube d'entrée (ce qu'on lui donne)
- **stdout (1)** : Tube de sortie normale (ce qu'il dit)
- **stderr (2)** : Tube d'erreur (quand il se plaint)

```bash
#!/bin/bash

# Démonstration des trois flux
echo "Ceci va vers stdout (sortie normale)"
echo "Ceci aussi va vers stdout" >&1

echo "Ceci va vers stderr (erreur)" >&2

# Lecture depuis stdin
echo "Tapez quelque chose :"
read entree  # Lit depuis stdin
echo "Vous avez tapé : $entree"
```

### Redirection de base

```bash
#!/bin/bash

# Script de démonstration des redirections
echo "=== Démonstration des redirections ==="

# 1. Redirection de stdout vers un fichier
echo "Cette ligne va dans un fichier" > sortie.txt
echo "Cette ligne aussi" >> sortie.txt  # Ajouter (append)

# 2. Redirection de stderr
ls /repertoire_inexistant 2> erreurs.txt

# 3. Redirection de stdout ET stderr
ls /home /repertoire_inexistant > tout.txt 2>&1

# 4. Ignorer la sortie
ls /repertoire_inexistant 2> /dev/null

echo "Vérifiez les fichiers créés :"
echo "- sortie.txt : $(cat sortie.txt)"
echo "- erreurs.txt : $(cat erreurs.txt 2>/dev/null || echo 'vide')"
```

### Exemples pratiques de redirection

```bash
#!/bin/bash

# Script de sauvegarde avec logs
LOG_FILE="sauvegarde.log"
ERROR_FILE="erreurs.log"

# Fonction de log
log_message() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> "$LOG_FILE"
}

# Fonction de log d'erreur
log_error() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - ERREUR: $1" >> "$ERROR_FILE"
}

echo "=== Début de la sauvegarde ===" | tee "$LOG_FILE"

# Créer un fichier de test
echo "Contenu de test" > fichier_test.txt

# Sauvegarde avec redirection
if cp fichier_test.txt backup_fichier.txt > /dev/null 2>&1; then
    log_message "Fichier sauvegardé avec succès"
    echo "✓ Sauvegarde réussie"
else
    log_error "Échec de la sauvegarde"
    echo "✗ Échec de la sauvegarde"
fi

# Afficher les logs
echo ""
echo "=== Contenu du log ==="
cat "$LOG_FILE"

# Nettoyer si pas d'erreurs
if [ ! -s "$ERROR_FILE" ]; then
    echo "Aucune erreur détectée"
    rm -f "$ERROR_FILE"
fi
```

### Redirection avancée

```bash
#!/bin/bash

# Techniques avancées de redirection

echo "=== Redirections avancées ==="

# 1. Here document (heredoc)
cat << EOF > instructions.txt
Voici les instructions :
1. Lire ce fichier
2. Comprendre les redirections
3. Pratiquer avec des exemples
EOF

# 2. Here string
grep "Lire" <<< "$(cat instructions.txt)"

# 3. Redirection avec exec
exec 3> fichier_special.txt  # Ouvrir le descripteur 3
echo "Message spécial" >&3    # Écrire dans le descripteur 3
exec 3>&-                     # Fermer le descripteur 3

# 4. Tee pour dupliquer la sortie
echo "Ce message va à l'écran ET dans un fichier" | tee double_sortie.txt

# 5. Pipeline avec gestion d'erreurs
ls /home /repertoire_inexistant 2>&1 | grep -v "Permission denied" > resultats_filtres.txt

echo "Fichiers créés :"
ls -la *.txt
```

### Pipes (tuyaux) - Introduction

```bash
#!/bin/bash

# Les pipes permettent de connecter les programmes
echo "=== Démonstration des pipes ==="

# Exemple 1 : Compter les fichiers
echo "Nombre de fichiers dans /bin :"
ls /bin | wc -l

# Exemple 2 : Recherche et tri
echo ""
echo "Fichiers contenant 'test' triés :"
ls | grep test | sort

# Exemple 3 : Pipeline complexe
echo ""
echo "Top 5 des plus gros fichiers :"
ls -la | sort -k5 -nr | head -6 | tail -5

# Exemple 4 : Traitement de texte
echo ""
echo "Mots uniques dans ce script :"
cat "$0" | tr ' ' '\n' | sort | uniq | head -10
```

## Formatage de sortie avec printf

### Pourquoi printf ?

`printf` est plus puissant et plus précis que `echo` pour formater du texte. C'est comme avoir une **machine à écrire avec des règles de mise en page**.

**Différence avec echo :**
- `echo` : Simple et direct
- `printf` : Contrôle précis du formatage

### Syntaxe de base

```bash
#!/bin/bash

echo "=== Comparaison echo vs printf ==="

# Avec echo
echo "Nom: Marie, Age: 25"

# Avec printf
printf "Nom: %s, Age: %d\n" "Marie" 25

echo ""
echo "=== Spécificateurs de format ==="
printf "Chaîne: %s\n" "Bonjour"
printf "Nombre entier: %d\n" 42
printf "Nombre flottant: %.2f\n" 3.14159
printf "Caractère: %c\n" 65  # ASCII de 'A'
```

### Formatage de nombres

```bash
#!/bin/bash

echo "=== Formatage de nombres ==="

nombre=42
pi=3.14159265

# Nombres entiers
printf "Nombre simple: %d\n" $nombre
printf "Avec zéros: %05d\n" $nombre
printf "Aligné à droite: %10d\n" $nombre
printf "Aligné à gauche: %-10d|\n" $nombre

echo ""

# Nombres décimaux
printf "Pi simple: %f\n" $pi
printf "Pi 2 décimales: %.2f\n" $pi
printf "Pi largeur 10: %10.2f\n" $pi
printf "Pi format scientifique: %e\n" $pi
```

### Formatage de chaînes

```bash
#!/bin/bash

echo "=== Formatage de chaînes ==="

nom="Marie"
ville="Paris"

# Chaînes de caractères
printf "Nom: %s\n" "$nom"
printf "Nom largeur 10: '%10s'\n" "$nom"
printf "Nom aligné gauche: '%-10s'\n" "$nom"
printf "Nom tronqué: '%.3s'\n" "$nom"

echo ""

# Combinaisons
printf "%-15s | %10s\n" "Nom" "Ville"
printf "%-15s | %10s\n" "---------------" "----------"
printf "%-15s | %10s\n" "$nom" "$ville"
```

### Tableau formaté avec printf

```bash
#!/bin/bash

# Créer un tableau de données formaté
echo "=== Rapport de ventes ==="

# En-tête
printf "%-15s | %8s | %10s | %12s\n" "Produit" "Quantité" "Prix unit." "Total"
printf "%-15s-+-%8s-+-%10s-+-%12s\n" "---------------" "--------" "----------" "------------"

# Données
produits=("Ordinateur" "Souris" "Clavier" "Écran")
quantites=(5 20 15 8)
prix=(800.00 25.50 45.99 299.99)

for i in "${!produits[@]}"; do
    total=$(echo "${quantites[$i]} * ${prix[$i]}" | bc -l)
    printf "%-15s | %8d | %10.2f | %12.2f\n" \
           "${produits[$i]}" "${quantites[$i]}" "${prix[$i]}" "$total"
done

# Total général
echo ""
printf "%37s: %12.2f\n" "TOTAL GÉNÉRAL" "1234.56"
```

### printf avec couleurs

```bash
#!/bin/bash

# Codes couleur ANSI
ROUGE='\033[0;31m'
VERT='\033[0;32m'
JAUNE='\033[1;33m'
BLEU='\033[0;34m'
RESET='\033[0m'  # Réinitialiser

echo "=== Messages colorés ==="

printf "${ROUGE}Erreur: %s${RESET}\n" "Fichier non trouvé"
printf "${VERT}Succès: %s${RESET}\n" "Opération réussie"
printf "${JAUNE}Attention: %s${RESET}\n" "Vérifiez vos données"
printf "${BLEU}Info: %s${RESET}\n" "Traitement en cours"

echo ""

# Barre de progression simple
printf "Progression: ["
for i in {1..20}; do
    if [ $i -le 12 ]; then
        printf "${VERT}█${RESET}"
    else
        printf " "
    fi
done
printf "] 60%%\n"
```

### Exercice pratique : Générateur de factures

```bash
#!/bin/bash

# Générateur de factures simple
echo "=== Générateur de factures ==="

# Collecte des informations
read -p "Nom du client : " client
read -p "Date (JJ/MM/AAAA) : " date_facture
read -p "Numéro de facture : " numero

echo ""

# En-tête de facture
printf "╔════════════════════════════════════════════════╗\n"
printf "║                   FACTURE                      ║\n"
printf "╠════════════════════════════════════════════════╣\n"
printf "║ Client: %-38s ║\n" "$client"
printf "║ Date: %-40s ║\n" "$date_facture"
printf "║ N° Facture: %-32s ║\n" "$numero"
printf "╠════════════════════════════════════════════════╣\n"

# Tableau des articles
printf "║ %-20s | %6s | %8s | %8s ║\n" "Article" "Qté" "P.U." "Total"
printf "╠════════════════════════════════════════════════╣\n"

# Articles (simulation)
articles=("Service conseil" "Formation" "Support")
quantites=(10 2 5)
prix=(50.00 200.00 30.00)
total_facture=0

for i in "${!articles[@]}"; do
    total_ligne=$(echo "${quantites[$i]} * ${prix[$i]}" | bc -l)
    total_facture=$(echo "$total_facture + $total_ligne" | bc -l)

    printf "║ %-20s | %6d | %8.2f | %8.2f ║\n" \
           "${articles[$i]}" "${quantites[$i]}" "${prix[$i]}" "$total_ligne"
done

# Total
printf "╠════════════════════════════════════════════════╣\n"
printf "║ %39s %8.2f ║\n" "TOTAL:" "$total_facture"
printf "╚════════════════════════════════════════════════╝\n"
```

## Récapitulatif

✅ **Ce que vous avez appris :**

1. **Arguments de ligne de commande :** Utilisation de `$1`, `$2`, `$#`, validation
2. **Commande read :** Interaction utilisateur, options, validation
3. **Redirections :** stdin, stdout, stderr, pipes, fichiers
4. **printf :** Formatage précis, tableaux, couleurs

✅ **Points clés à retenir :**
- `$1`, `$2`... pour les arguments positionnels
- `read -p` pour les prompts élégants
- `>` pour rediriger, `>>` pour ajouter
- `printf` plus puissant qu'`echo` pour le formatage

✅ **Commandes importantes :**
```bash
read -p "Question : " variable
printf "Format: %s %d\n" "texte" 42
commande > fichier     # Redirection
commande | autre       # Pipe
```

✅ **Prochaines étapes :**
Dans le chapitre 5, nous découvrirons les structures de contrôle (if, for, while) pour rendre nos scripts intelligents !

---

💡 **Conseil :** Pratiquez en créant des scripts interactifs simples comme des calculatrices ou des menus avant de passer au chapitre suivant.

⏭️
