🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Tutoriel Bash - Chapitre 3 : Variables et types de données

## Déclaration et utilisation des variables

### Qu'est-ce qu'une variable ?

Une variable est comme une **boîte avec une étiquette** dans laquelle vous pouvez stocker des informations. L'étiquette est le nom de la variable, et le contenu de la boîte est sa valeur.

**Analogie :** Imaginez des boîtes de rangement :
- Boîte étiquetée "NOM" → contient "Marie"
- Boîte étiquetée "AGE" → contient "25"
- Boîte étiquetée "VILLE" → contient "Paris"

### Syntaxe de base

```bash
# Déclaration d'une variable (PAS D'ESPACES autour du =)
nom_variable="valeur"

# Utilisation d'une variable (avec le symbole $)
echo $nom_variable
```

⚠️ **Erreur fréquente :** Ne jamais mettre d'espaces autour du `=` !

```bash
# ✅ Correct
nom="Marie"

# ❌ Incorrect
nom = "Marie"    # Erreur !
```

### Exemple pratique

Créons un script `variables_demo.sh` :

```bash
#!/bin/bash

# Déclaration de variables
nom="Marie"
age=25
ville="Paris"
profession="Développeuse"

# Utilisation des variables
echo "=== Informations personnelles ==="
echo "Nom : $nom"
echo "Âge : $age ans"
echo "Ville : $ville"
echo "Profession : $profession"

# Calcul avec des variables
prix_unitaire=15
quantite=3
total=$((prix_unitaire * quantite))

echo ""
echo "=== Calcul ==="
echo "Prix unitaire : $prix_unitaire €"
echo "Quantité : $quantite"
echo "Total : $total €"
```

### Différentes façons d'utiliser les variables

```bash
#!/bin/bash

nom="Jean"

# Méthode 1 : Basique
echo $nom

# Méthode 2 : Avec accolades (recommandée)
echo ${nom}

# Méthode 3 : Concaténation
echo "Bonjour ${nom}!"

# Méthode 4 : Dans une phrase
echo "L'utilisateur ${nom} est connecté."

# Méthode 5 : Avec suffixe
fichier="document"
echo "${fichier}.txt"    # Affiche : document.txt
echo "$fichier.txt"      # Affiche : document.txt
```

### Types de données en Bash

En Bash, tout est techniquement du **texte**, mais on peut distinguer :

**1. Chaînes de caractères (texte) :**
```bash
nom="Marie Dupont"
message="Bonjour tout le monde"
chemin="/home/utilisateur/documents"
```

**2. Nombres entiers :**
```bash
age=25
score=1500
annee=2025
```

**3. Nombres avec calculs :**
```bash
# Opérations arithmétiques
resultat=$((10 + 5))        # Addition
produit=$((7 * 3))          # Multiplication
division=$((20 / 4))        # Division
reste=$((17 % 5))           # Modulo (reste)

echo "Résultat : $resultat"
echo "Produit : $produit"
echo "Division : $division"
echo "Reste : $reste"
```

**4. Booléens (vrai/faux) :**
```bash
# En Bash, on utilise souvent des mots-clés
est_connecte="true"
fichier_existe="false"
debug_mode="on"
```

### Bonnes pratiques pour nommer les variables

```bash
# ✅ Noms descriptifs et clairs
nom_utilisateur="marie"
fichier_configuration="/etc/config.conf"
nombre_tentatives=3

# ✅ Style snake_case (recommandé)
prix_total=100
date_creation="2025-07-13"

# ✅ Constantes en majuscules
VERSION="1.0"
REPERTOIRE_BACKUP="/backup"

# ❌ Noms peu clairs
x="marie"
f="/etc/config.conf"
n=3
```

## Variables d'environnement

### Qu'est-ce qu'une variable d'environnement ?

Les variables d'environnement sont des variables **globales** disponibles pour tous les programmes de votre système. C'est comme des informations affichées sur un tableau d'affichage que tout le monde peut consulter.

### Variables d'environnement courantes

```bash
#!/bin/bash

echo "=== Variables d'environnement importantes ==="
echo "Nom d'utilisateur : $USER"
echo "Répertoire personnel : $HOME"
echo "Répertoire actuel : $PWD"
echo "Shell utilisé : $SHELL"
echo "Chemin de recherche : $PATH"
echo "Système d'exploitation : $OSTYPE"
```

### Voir toutes les variables d'environnement

```bash
# Afficher toutes les variables d'environnement
env

# Ou avec plus de détails
printenv

# Rechercher une variable spécifique
env | grep HOME
```

### Créer ses propres variables d'environnement

```bash
#!/bin/bash

# Variable locale (uniquement dans ce script)
ma_variable="valeur_locale"

# Variable d'environnement (disponible pour les sous-programmes)
export MA_VARIABLE_GLOBALE="valeur_globale"

# Tester la différence
echo "Variable locale : $ma_variable"
echo "Variable globale : $MA_VARIABLE_GLOBALE"

# Lancer un sous-shell pour tester
bash -c 'echo "Dans le sous-shell : $MA_VARIABLE_GLOBALE"'
```

### Exemple pratique avec variables d'environnement

```bash
#!/bin/bash

# Script de configuration personnalisé
export MON_PROJET_DIR="$HOME/mes_projets"
export MON_PROJET_CONFIG="$MON_PROJET_DIR/config"
export MON_PROJET_LOG="$MON_PROJET_DIR/logs"

# Créer la structure de répertoires
mkdir -p "$MON_PROJET_DIR"
mkdir -p "$MON_PROJET_CONFIG"
mkdir -p "$MON_PROJET_LOG"

echo "Projet configuré dans : $MON_PROJET_DIR"
echo "Configuration : $MON_PROJET_CONFIG"
echo "Logs : $MON_PROJET_LOG"
```

## Variables spéciales ($0, $1, $#, $?, etc.)

### Les variables magiques de Bash

Bash fournit automatiquement des variables spéciales très utiles :

```bash
#!/bin/bash

echo "=== Variables spéciales ==="
echo "Nom du script : $0"
echo "Premier argument : $1"
echo "Deuxième argument : $2"
echo "Troisième argument : $3"
echo "Nombre d'arguments : $#"
echo "Tous les arguments : $@"
echo "Tous les arguments (chaîne) : $*"
echo "PID du script : $$"
```

### Test pratique

Créez un script `test_arguments.sh` :

```bash
#!/bin/bash

echo "=== Analyse des arguments ==="
echo "Script exécuté : $0"
echo "Nombre d'arguments reçus : $#"

if [ $# -eq 0 ]; then
    echo "Aucun argument fourni."
    echo "Usage : $0 nom age ville"
    exit 1
fi

echo "Premier argument (nom) : $1"
echo "Deuxième argument (âge) : $2"
echo "Troisième argument (ville) : $3"

echo ""
echo "Tous les arguments : $@"
```

Testez-le :
```bash
chmod +x test_arguments.sh
./test_arguments.sh Marie 25 Paris
```

### Variable de code de retour ($?)

```bash
#!/bin/bash

# Commande qui réussit
ls /home
echo "Code de retour de 'ls /home' : $?"

# Commande qui échoue
ls /repertoire_inexistant 2>/dev/null
echo "Code de retour de 'ls inexistant' : $?"

# Utilisation pratique
if [ $? -eq 0 ]; then
    echo "La commande précédente a réussi"
else
    echo "La commande précédente a échoué"
fi
```

### Tableau récapitulatif des variables spéciales

| Variable | Description | Exemple |
|----------|-------------|---------|
| `$0` | Nom du script | `./mon_script.sh` |
| `$1`, `$2`, `$3`... | Arguments positionnels | `Marie`, `25`, `Paris` |
| `$#` | Nombre d'arguments | `3` |
| `$@` | Tous les arguments (liste) | `Marie 25 Paris` |
| `$*` | Tous les arguments (chaîne) | `"Marie 25 Paris"` |
| `$?` | Code de retour dernière commande | `0` (succès) ou `1` (erreur) |
| `$$` | PID du processus actuel | `12345` |
| `$!` | PID du dernier processus en arrière-plan | `12346` |

## Tableaux (arrays)

### Qu'est-ce qu'un tableau ?

Un tableau est comme un **casier à plusieurs compartiments numérotés**. Au lieu d'avoir une seule valeur, vous pouvez stocker plusieurs valeurs dans la même variable.

### Déclaration de tableaux

```bash
#!/bin/bash

# Méthode 1 : Déclaration directe
fruits=("pomme" "banane" "orange" "kiwi")

# Méthode 2 : Déclaration élément par élément
couleurs[0]="rouge"
couleurs[1]="bleu"
couleurs[2]="vert"

# Méthode 3 : Déclaration avec declare
declare -a nombres=(10 20 30 40 50)
```

### Utilisation des tableaux

```bash
#!/bin/bash

# Déclaration
animaux=("chat" "chien" "oiseau" "poisson")

echo "=== Accès aux éléments ==="
echo "Premier animal : ${animaux[0]}"
echo "Deuxième animal : ${animaux[1]}"
echo "Dernier animal : ${animaux[-1]}"

echo ""
echo "=== Informations sur le tableau ==="
echo "Tous les animaux : ${animaux[@]}"
echo "Nombre d'animaux : ${#animaux[@]}"

echo ""
echo "=== Indices du tableau ==="
echo "Indices : ${!animaux[@]}"
```

### Parcourir un tableau

```bash
#!/bin/bash

# Tableau de villes
villes=("Paris" "Lyon" "Marseille" "Toulouse" "Nice")

echo "=== Méthode 1 : Parcours avec for...in ==="
for ville in "${villes[@]}"; do
    echo "Ville : $ville"
done

echo ""
echo "=== Méthode 2 : Parcours avec indices ==="
for i in "${!villes[@]}"; do
    echo "Index $i : ${villes[$i]}"
done

echo ""
echo "=== Méthode 3 : Parcours classique ==="
for (( i=0; i<${#villes[@]}; i++ )); do
    echo "Position $i : ${villes[$i]}"
done
```

### Opérations sur les tableaux

```bash
#!/bin/bash

# Tableau initial
langages=("Python" "JavaScript" "Java")

echo "Tableau initial : ${langages[@]}"

# Ajouter un élément
langages+=("C++")
echo "Après ajout : ${langages[@]}"

# Modifier un élément
langages[1]="TypeScript"
echo "Après modification : ${langages[@]}"

# Supprimer un élément
unset langages[2]
echo "Après suppression : ${langages[@]}"

# Taille du tableau
echo "Nombre d'éléments : ${#langages[@]}"
```

### Exemple pratique : Gestionnaire de tâches

```bash
#!/bin/bash

# Gestionnaire de tâches simple
declare -a taches=()

# Fonction pour ajouter une tâche
ajouter_tache() {
    echo "Entrez une nouvelle tâche :"
    read nouvelle_tache
    taches+=("$nouvelle_tache")
    echo "Tâche ajoutée : $nouvelle_tache"
}

# Fonction pour afficher les tâches
afficher_taches() {
    echo "=== Liste des tâches ==="
    if [ ${#taches[@]} -eq 0 ]; then
        echo "Aucune tâche."
    else
        for i in "${!taches[@]}"; do
            echo "$((i+1)). ${taches[$i]}"
        done
    fi
}

# Exemple d'utilisation
taches=("Faire les courses" "Appeler le médecin" "Finir le projet")
afficher_taches
```

## Portée des variables (local vs global)

### Variables globales vs locales

**Analogie :** Les variables globales sont comme des annonces au micro dans toute l'école, tandis que les variables locales sont comme des conversations privées dans une classe.

### Exemple de base

```bash
#!/bin/bash

# Variable globale
nom_global="Marie"

# Fonction avec variable locale
ma_fonction() {
    local nom_local="Jean"
    echo "Dans la fonction - Global : $nom_global"
    echo "Dans la fonction - Local : $nom_local"
}

# Test
echo "Avant la fonction - Global : $nom_global"
ma_fonction
echo "Après la fonction - Global : $nom_global"
echo "Après la fonction - Local : $nom_local"  # Sera vide !
```

### Problème sans 'local'

```bash
#!/bin/bash

compteur=0

# Fonction qui modifie accidentellement une variable globale
mauvaise_fonction() {
    compteur=100  # Modifie la variable globale !
    echo "Dans la fonction : $compteur"
}

echo "Avant : $compteur"
mauvaise_fonction
echo "Après : $compteur"  # Oups ! Modifié par erreur
```

### Solution avec 'local'

```bash
#!/bin/bash

compteur=0

# Fonction qui utilise une variable locale
bonne_fonction() {
    local compteur=100  # Variable locale
    echo "Dans la fonction : $compteur"
}

echo "Avant : $compteur"
bonne_fonction
echo "Après : $compteur"  # Reste inchangé
```

### Exemple pratique complet

```bash
#!/bin/bash

# Variables globales
SCRIPT_NAME="Gestionnaire de comptes"
VERSION="1.0"

# Fonction de calcul avec variables locales
calculer_interets() {
    local capital=$1
    local taux=$2
    local duree=$3

    local interets=$((capital * taux * duree / 100))
    local total=$((capital + interets))

    echo "=== Calcul d'intérêts ==="
    echo "Capital initial : $capital €"
    echo "Taux : $taux %"
    echo "Durée : $duree ans"
    echo "Intérêts : $interets €"
    echo "Total : $total €"

    # Retourner le résultat
    return $total
}

# Fonction d'affichage avec variables locales
afficher_bienvenue() {
    local utilisateur=$1
    local date_actuelle=$(date +"%d/%m/%Y")

    echo "==============================="
    echo "  $SCRIPT_NAME v$VERSION"
    echo "==============================="
    echo "Bonjour $utilisateur !"
    echo "Date : $date_actuelle"
    echo ""
}

# Programme principal
afficher_bienvenue "Marie"
calculer_interets 1000 5 3
```

### Bonnes pratiques pour la portée

```bash
#!/bin/bash

# ✅ Bonnes pratiques

# 1. Toujours déclarer les variables locales
ma_fonction() {
    local nom="Jean"
    local age=25
    # ... reste du code
}

# 2. Constantes globales en majuscules
readonly VERSION="1.0"
readonly CONFIG_FILE="/etc/config.conf"

# 3. Variables de travail locales
traiter_fichier() {
    local fichier=$1
    local ligne_count=0

    while read -r ligne; do
        ((ligne_count++))
        # traitement...
    done < "$fichier"

    echo "Traité $ligne_count lignes"
}

# 4. Éviter les variables globales modifiables
# ❌ Éviter
status="en_cours"

# ✅ Préférer
get_status() {
    echo "en_cours"
}
```

### Exemple d'exercice pratique

Créez un script `calculatrice.sh` qui utilise variables locales et globales :

```bash
#!/bin/bash

# Variables globales
TITRE="Calculatrice Simple"
HISTORIQUE=()

# Fonction addition
addition() {
    local a=$1
    local b=$2
    local resultat=$((a + b))

    HISTORIQUE+=("$a + $b = $resultat")
    echo "$resultat"
}

# Fonction soustraction
soustraction() {
    local a=$1
    local b=$2
    local resultat=$((a - b))

    HISTORIQUE+=("$a - $b = $resultat")
    echo "$resultat"
}

# Fonction affichage historique
afficher_historique() {
    echo "=== Historique ==="
    for calcul in "${HISTORIQUE[@]}"; do
        echo "$calcul"
    done
}

# Test
echo "=== $TITRE ==="
addition 10 5
soustraction 20 8
afficher_historique
```

## Récapitulatif

✅ **Ce que vous avez appris :**

1. **Variables de base :** Déclaration, utilisation, nommage
2. **Variables d'environnement :** Accès aux informations système
3. **Variables spéciales :** Arguments, codes de retour, PID
4. **Tableaux :** Stockage de plusieurs valeurs, parcours
5. **Portée :** Différence entre variables locales et globales

✅ **Points clés à retenir :**
- Pas d'espaces autour du `=` lors de la déclaration
- Utiliser `${variable}` pour la clarté
- Variables locales avec `local` dans les fonctions
- Variables d'environnement avec `export`
- Tableaux avec `("élément1" "élément2")`

✅ **Prochaines étapes :**
Dans le chapitre 4, nous apprendrons à interagir avec l'utilisateur et à gérer les entrées/sorties !

---

💡 **Astuce :** Pratiquez avec de petits scripts pour bien maîtriser ces concepts avant de passer au chapitre suivant.

⏭️
