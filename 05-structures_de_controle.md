🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Tutoriel Bash - Chapitre 5 : Structures de contrôle

## Conditions (if, elif, else)

### Qu'est-ce qu'une condition ?

Les conditions permettent à votre script de **prendre des décisions**. C'est comme les feux de circulation : si le feu est vert, on avance ; si il est rouge, on s'arrête.

**Analogie :** Une condition est comme un garde qui pose une question :
- **SI** vous avez plus de 18 ans → vous pouvez entrer
- **SINON** → vous ne pouvez pas entrer

### Syntaxe de base du if

```bash
#!/bin/bash

# Structure if simple
age=20

if [ $age -ge 18 ]; then
    echo "Vous êtes majeur"
fi
```

**Points importants :**
- Les **espaces** autour des crochets `[ ]` sont **obligatoires**
- `then` sur une nouvelle ligne ou après `;`
- Terminer par `fi` (if à l'envers)

### if-else complet

```bash
#!/bin/bash

# Demander l'âge à l'utilisateur
read -p "Quel est votre âge ? " age

if [ $age -ge 18 ]; then
    echo "✓ Vous êtes majeur, vous pouvez voter !"
else
    echo "✗ Vous êtes mineur, patience !"
fi
```

### if-elif-else (choix multiples)

```bash
#!/bin/bash

# Système de notes
read -p "Entrez votre note sur 20 : " note

if [ $note -ge 16 ]; then
    echo "🏆 Excellent ! Félicitations !"
    mention="Très bien"
elif [ $note -ge 14 ]; then
    echo "😊 Très bien ! Bon travail !"
    mention="Bien"
elif [ $note -ge 12 ]; then
    echo "👍 Assez bien, continuez !"
    mention="Assez bien"
elif [ $note -ge 10 ]; then
    echo "✓ Passable, il faut travailler"
    mention="Passable"
else
    echo "😞 Insuffisant, courage !"
    mention="Insuffisant"
fi

echo "Mention : $mention"
```

### Conditions avec chaînes de caractères

```bash
#!/bin/bash

echo "=== Test de mot de passe ==="
read -s -p "Mot de passe : " mot_de_passe
echo ""

if [ "$mot_de_passe" = "secret123" ]; then
    echo "✓ Accès autorisé"
elif [ "$mot_de_passe" = "" ]; then
    echo "✗ Mot de passe vide"
else
    echo "✗ Mot de passe incorrect"
fi

echo ""
echo "=== Test de nom d'utilisateur ==="
read -p "Votre nom : " nom

if [ -z "$nom" ]; then
    echo "Nom vide !"
elif [ ${#nom} -lt 2 ]; then
    echo "Nom trop court"
else
    echo "Bonjour $nom !"
fi
```

### Conditions avec fichiers

```bash
#!/bin/bash

# Vérification de fichiers
read -p "Nom du fichier à vérifier : " fichier

echo "=== Vérification du fichier '$fichier' ==="

if [ -f "$fichier" ]; then
    echo "✓ Le fichier existe"

    if [ -r "$fichier" ]; then
        echo "✓ Le fichier est lisible"
    else
        echo "✗ Le fichier n'est pas lisible"
    fi

    if [ -w "$fichier" ]; then
        echo "✓ Le fichier est modifiable"
    else
        echo "✗ Le fichier n'est pas modifiable"
    fi

    if [ -x "$fichier" ]; then
        echo "✓ Le fichier est exécutable"
    else
        echo "✗ Le fichier n'est pas exécutable"
    fi

elif [ -d "$fichier" ]; then
    echo "ℹ️  C'est un répertoire, pas un fichier"
else
    echo "✗ Le fichier n'existe pas"
fi
```

## Opérateurs de comparaison

### Opérateurs pour les nombres

```bash
#!/bin/bash

# Démonstration des opérateurs numériques
a=10
b=20

echo "=== Comparaisons numériques ==="
echo "a = $a, b = $b"

# Égalité et inégalité
if [ $a -eq $b ]; then
    echo "a égal à b"
else
    echo "a différent de b"
fi

# Comparaisons
if [ $a -lt $b ]; then
    echo "a est plus petit que b"
fi

if [ $a -le $b ]; then
    echo "a est plus petit ou égal à b"
fi

if [ $a -gt 5 ]; then
    echo "a est plus grand que 5"
fi

if [ $a -ge 10 ]; then
    echo "a est plus grand ou égal à 10"
fi

if [ $a -ne $b ]; then
    echo "a est différent de b"
fi
```

### Tableau des opérateurs numériques

| Opérateur | Signification | Exemple |
|-----------|---------------|---------|
| `-eq` | Égal à | `[ $a -eq $b ]` |
| `-ne` | Différent de | `[ $a -ne $b ]` |
| `-lt` | Plus petit que | `[ $a -lt $b ]` |
| `-le` | Plus petit ou égal | `[ $a -le $b ]` |
| `-gt` | Plus grand que | `[ $a -gt $b ]` |
| `-ge` | Plus grand ou égal | `[ $a -ge $b ]` |

### Opérateurs pour les chaînes

```bash
#!/bin/bash

# Démonstration des opérateurs de chaînes
nom1="Marie"
nom2="Pierre"
nom3=""

echo "=== Comparaisons de chaînes ==="

# Égalité de chaînes
if [ "$nom1" = "Marie" ]; then
    echo "Le nom1 est Marie"
fi

# Inégalité de chaînes
if [ "$nom1" != "$nom2" ]; then
    echo "nom1 et nom2 sont différents"
fi

# Chaîne vide
if [ -z "$nom3" ]; then
    echo "nom3 est vide"
fi

# Chaîne non vide
if [ -n "$nom1" ]; then
    echo "nom1 n'est pas vide"
fi

# Comparaison alphabétique (avancé)
if [[ "$nom1" < "$nom2" ]]; then
    echo "$nom1 vient avant $nom2 dans l'alphabet"
fi
```

### Tableau des opérateurs de chaînes

| Opérateur | Signification | Exemple |
|-----------|---------------|---------|
| `=` | Égal à | `[ "$a" = "$b" ]` |
| `!=` | Différent de | `[ "$a" != "$b" ]` |
| `-z` | Chaîne vide | `[ -z "$a" ]` |
| `-n` | Chaîne non vide | `[ -n "$a" ]` |
| `<` | Plus petit (alphabétique) | `[[ "$a" < "$b" ]]` |
| `>` | Plus grand (alphabétique) | `[[ "$a" > "$b" ]]` |

### Opérateurs logiques

```bash
#!/bin/bash

# Opérateurs ET (&&) et OU (||)
age=25
nom="Marie"

echo "=== Opérateurs logiques ==="

# ET logique (&&)
if [ $age -ge 18 ] && [ -n "$nom" ]; then
    echo "Utilisateur majeur avec un nom valide"
fi

# OU logique (||)
if [ $age -lt 18 ] || [ -z "$nom" ]; then
    echo "Utilisateur mineur OU nom vide"
else
    echo "Utilisateur majeur avec un nom"
fi

# Négation avec !
if [ ! -f "fichier_inexistant.txt" ]; then
    echo "Le fichier n'existe pas (c'est normal)"
fi

# Combinaisons complexes
if [ $age -ge 18 ] && [ $age -le 65 ] && [ "$nom" != "" ]; then
    echo "Utilisateur en âge de travailler avec un nom"
fi
```

### Opérateurs de fichiers

```bash
#!/bin/bash

# Test complet sur les fichiers
fichier="test.txt"

# Créer un fichier de test
echo "Contenu de test" > "$fichier"

echo "=== Tests sur les fichiers ==="

if [ -e "$fichier" ]; then
    echo "✓ Le fichier existe"
fi

if [ -f "$fichier" ]; then
    echo "✓ C'est un fichier régulier"
fi

if [ -d "$fichier" ]; then
    echo "C'est un répertoire"
else
    echo "✓ Ce n'est pas un répertoire"
fi

if [ -s "$fichier" ]; then
    echo "✓ Le fichier n'est pas vide"
fi

if [ -r "$fichier" ]; then
    echo "✓ Le fichier est lisible"
fi

# Nettoyer
rm -f "$fichier"
```

### Tableau des opérateurs de fichiers

| Opérateur | Test | Exemple |
|-----------|------|---------|
| `-e` | Existe | `[ -e "$fichier" ]` |
| `-f` | Fichier régulier | `[ -f "$fichier" ]` |
| `-d` | Répertoire | `[ -d "$dossier" ]` |
| `-s` | Fichier non vide | `[ -s "$fichier" ]` |
| `-r` | Lisible | `[ -r "$fichier" ]` |
| `-w` | Modifiable | `[ -w "$fichier" ]` |
| `-x` | Exécutable | `[ -x "$fichier" ]` |

## Boucles (for, while, until)

### Boucle for - Parcours de listes

La boucle `for` répète des actions pour chaque élément d'une liste. C'est comme **distribuer des cartes** une par une.

```bash
#!/bin/bash

# Boucle for simple
echo "=== Fruits préférés ==="
for fruit in pomme banane orange kiwi; do
    echo "J'aime les ${fruit}s"
done

echo ""

# Boucle for avec tableau
couleurs=("rouge" "bleu" "vert" "jaune")
echo "=== Couleurs ==="
for couleur in "${couleurs[@]}"; do
    echo "Couleur : $couleur"
done
```

### Boucle for avec séquences

```bash
#!/bin/bash

# Compter de 1 à 5
echo "=== Compte de 1 à 5 ==="
for i in {1..5}; do
    echo "Numéro : $i"
done

echo ""

# Compter de 0 à 10 par pas de 2
echo "=== Nombres pairs ==="
for i in {0..10..2}; do
    echo "Nombre pair : $i"
done

echo ""

# Style C (plus technique)
echo "=== Style C ==="
for (( i=1; i<=3; i++ )); do
    echo "Itération $i"
done
```

### Boucle for avec fichiers

```bash
#!/bin/bash

# Créer quelques fichiers de test
touch fichier1.txt fichier2.txt fichier3.log

echo "=== Traitement des fichiers .txt ==="
for fichier in *.txt; do
    if [ -f "$fichier" ]; then
        echo "Traitement de : $fichier"
        echo "Taille : $(stat -c%s "$fichier") octets"
    fi
done

echo ""
echo "=== Tous les fichiers du répertoire ==="
for fichier in *; do
    if [ -f "$fichier" ]; then
        echo "📄 Fichier : $fichier"
    elif [ -d "$fichier" ]; then
        echo "📁 Dossier : $fichier"
    fi
done

# Nettoyer
rm -f fichier*.txt fichier*.log
```

### Boucle while - Tant que

La boucle `while` continue **tant qu'une condition est vraie**. C'est comme dire "continue à courir tant que tu n'es pas fatigué".

```bash
#!/bin/bash

# Compteur simple
compteur=1
echo "=== Compteur avec while ==="
while [ $compteur -le 5 ]; do
    echo "Compteur : $compteur"
    ((compteur++))  # Incrémenter
done

echo ""

# Demander jusqu'à obtenir la bonne réponse
echo "=== Jeu de devinette ==="
secret=7
tentative=0

while true; do
    read -p "Devinez le nombre (entre 1 et 10) : " nombre
    ((tentative++))

    if [ "$nombre" = "$secret" ]; then
        echo "🎉 Bravo ! Trouvé en $tentative tentative(s)"
        break
    elif [ "$nombre" -lt "$secret" ]; then
        echo "📈 Plus grand !"
    else
        echo "📉 Plus petit !"
    fi

    if [ $tentative -ge 3 ]; then
        echo "💡 Indice : c'est un nombre premier"
    fi
done
```

### Lecture de fichier avec while

```bash
#!/bin/bash

# Créer un fichier de test
cat > liste_courses.txt << EOF
Pommes
Lait
Pain
Fromage
Oeufs
EOF

echo "=== Liste de courses ==="
numero=1
while read -r article; do
    echo "$numero. $article"
    ((numero++))
done < liste_courses.txt

echo ""
echo "=== Traitement ligne par ligne ==="
while IFS= read -r ligne; do
    if [ ${#ligne} -gt 4 ]; then
        echo "Article long : $ligne (${#ligne} caractères)"
    else
        echo "Article court : $ligne"
    fi
done < liste_courses.txt

# Nettoyer
rm -f liste_courses.txt
```

### Boucle until - Jusqu'à ce que

La boucle `until` continue **jusqu'à ce qu'une condition devienne vraie**. C'est l'opposé de `while`.

```bash
#!/bin/bash

# Attendre qu'un fichier soit créé
fichier_attente="signal.txt"

echo "=== Attente de création du fichier '$fichier_attente' ==="
echo "Créez le fichier avec : touch $fichier_attente"

compteur=0
until [ -f "$fichier_attente" ]; do
    echo "Attente... ($compteur secondes)"
    sleep 1
    ((compteur++))

    # Éviter l'attente infinie
    if [ $compteur -ge 10 ]; then
        echo "Timeout ! Création automatique du fichier"
        touch "$fichier_attente"
    fi
done

echo "✓ Fichier détecté après $compteur secondes"

# Nettoyer
rm -f "$fichier_attente"
```

### Comparaison des boucles

```bash
#!/bin/bash

echo "=== Comparaison des trois types de boucles ==="

# Même résultat avec for
echo "--- Avec for ---"
for i in {1..3}; do
    echo "for: $i"
done

# Même résultat avec while
echo "--- Avec while ---"
i=1
while [ $i -le 3 ]; do
    echo "while: $i"
    ((i++))
done

# Même résultat avec until
echo "--- Avec until ---"
i=1
until [ $i -gt 3 ]; do
    echo "until: $i"
    ((i++))
done
```

## Instruction case

### Qu'est-ce que case ?

L'instruction `case` est comme un **aiguillage de train** : selon la valeur d'une variable, le script prend différentes directions. C'est plus élégant que de multiples `if-elif`.

### Syntaxe de base

```bash
#!/bin/bash

# Menu simple avec case
echo "=== Menu principal ==="
echo "1. Voir les fichiers"
echo "2. Voir la date"
echo "3. Voir l'utilisateur"
echo "4. Quitter"

read -p "Votre choix : " choix

case $choix in
    1)
        echo "📁 Fichiers du répertoire :"
        ls -la
        ;;
    2)
        echo "📅 Date actuelle :"
        date
        ;;
    3)
        echo "👤 Utilisateur actuel :"
        whoami
        ;;
    4)
        echo "👋 Au revoir !"
        exit 0
        ;;
    *)
        echo "❌ Choix invalide"
        ;;
esac
```

**Points importants :**
- Chaque cas se termine par `;;`
- `*)` capture tous les autres cas
- `esac` termine l'instruction (case à l'envers)

### case avec motifs

```bash
#!/bin/bash

# Analyseur de fichiers par extension
read -p "Nom du fichier : " fichier

case "$fichier" in
    *.txt)
        echo "📝 Fichier texte"
        ;;
    *.jpg|*.jpeg|*.png|*.gif)
        echo "🖼️  Fichier image"
        ;;
    *.mp3|*.wav|*.flac)
        echo "🎵 Fichier audio"
        ;;
    *.mp4|*.avi|*.mkv)
        echo "🎬 Fichier vidéo"
        ;;
    *.sh)
        echo "⚙️  Script Bash"
        if [ -x "$fichier" ]; then
            echo "   ✓ Exécutable"
        else
            echo "   ✗ Non exécutable"
        fi
        ;;
    *.pdf)
        echo "📄 Document PDF"
        ;;
    *)
        echo "❓ Type de fichier inconnu"
        ;;
esac
```

### case avec plages et conditions

```bash
#!/bin/bash

# Système d'évaluation
read -p "Entrez votre note (0-20) : " note

case $note in
    1[6-9]|20)  # 16, 17, 18, 19, 20
        echo "🏆 Très bien ! Excellent travail !"
        ;;
    1[4-5])     # 14, 15
        echo "😊 Bien ! Bon travail !"
        ;;
    1[0-3])     # 10, 11, 12, 13
        echo "👍 Assez bien, continuez !"
        ;;
    [6-9])      # 6, 7, 8, 9
        echo "😐 Passable, il faut travailler"
        ;;
    [0-5])      # 0, 1, 2, 3, 4, 5
        echo "😞 Insuffisant, courage !"
        ;;
    *)
        echo "❌ Note invalide (doit être entre 0 et 20)"
        ;;
esac
```

### Menu interactif complet avec case

```bash
#!/bin/bash

# Gestionnaire de fichiers simple
afficher_menu() {
    echo ""
    echo "╔════════════════════════════════════╗"
    echo "║        GESTIONNAIRE DE FICHIERS    ║"
    echo "╠════════════════════════════════════╣"
    echo "║ 1. Lister les fichiers             ║"
    echo "║ 2. Créer un fichier                ║"
    echo "║ 3. Supprimer un fichier            ║"
    echo "║ 4. Copier un fichier               ║"
    echo "║ 5. Voir le contenu d'un fichier    ║"
    echo "║ 6. Informations système            ║"
    echo "║ q. Quitter                         ║"
    echo "╚════════════════════════════════════╝"
}

while true; do
    afficher_menu
    read -p "Votre choix : " choix

    case "$choix" in
        1|l|list)
            echo "📁 Contenu du répertoire :"
            ls -lah
            ;;
        2|c|create)
            read -p "Nom du fichier à créer : " nom_fichier
            touch "$nom_fichier"
            echo "✓ Fichier '$nom_fichier' créé"
            ;;
        3|d|delete)
            read -p "Nom du fichier à supprimer : " nom_fichier
            if [ -f "$nom_fichier" ]; then
                rm "$nom_fichier"
                echo "✓ Fichier '$nom_fichier' supprimé"
            else
                echo "❌ Fichier introuvable"
            fi
            ;;
        4|cp|copy)
            read -p "Fichier source : " source
            read -p "Fichier destination : " destination
            if cp "$source" "$destination" 2>/dev/null; then
                echo "✓ Copie réussie"
            else
                echo "❌ Erreur lors de la copie"
            fi
            ;;
        5|v|view)
            read -p "Fichier à afficher : " nom_fichier
            if [ -f "$nom_fichier" ]; then
                echo "--- Contenu de '$nom_fichier' ---"
                cat "$nom_fichier"
                echo "--- Fin du fichier ---"
            else
                echo "❌ Fichier introuvable"
            fi
            ;;
        6|i|info)
            echo "💻 Informations système :"
            echo "   Utilisateur : $(whoami)"
            echo "   Date : $(date)"
            echo "   Répertoire : $(pwd)"
            echo "   Espace disque : $(df -h . | tail -1 | awk '{print $4}') disponible"
            ;;
        q|quit|exit)
            echo "👋 Au revoir !"
            exit 0
            ;;
        *)
            echo "❌ Choix invalide. Utilisez 1-6 ou q pour quitter"
            ;;
    esac

    echo ""
    read -p "Appuyez sur Entrée pour continuer..." -s
done
```

## Contrôle de flux (break, continue, exit)

### break - Sortir d'une boucle

`break` permet de **sortir immédiatement** d'une boucle, comme appuyer sur le bouton d'arrêt d'urgence.

```bash
#!/bin/bash

# Recherche dans une liste
echo "=== Recherche avec break ==="
fruits=("pomme" "banane" "orange" "kiwi" "mangue")
recherche="orange"

for fruit in "${fruits[@]}"; do
    echo "Examen de : $fruit"

    if [ "$fruit" = "$recherche" ]; then
        echo "✓ Trouvé : $fruit !"
        break  # Sortir de la boucle
    fi
done

echo "Recherche terminée"

echo ""

# Saisie sécurisée avec break
echo "=== Saisie de mot de passe ==="
tentatives=0
max_tentatives=3

while true; do
    read -s -p "Mot de passe : " mot_de_passe
    echo ""
    ((tentatives++))

    if [ "$mot_de_passe" = "secret" ]; then
        echo "✓ Accès autorisé"
        break
    else
        echo "❌ Mot de passe incorrect"

        if [ $tentatives -ge $max_tentatives ]; then
            echo "🚫 Trop de tentatives, accès bloqué"
            break
        fi

        echo "Tentatives restantes : $((max_tentatives - tentatives))"
    fi
done
```

### continue - Passer à l'itération suivante

`continue` permet de **sauter le reste** de l'itération actuelle et passer à la suivante.

```bash
#!/bin/bash

# Traitement sélectif avec continue
echo "=== Traitement des nombres pairs seulement ==="
for i in {1..10}; do
    # Ignorer les nombres impairs
    if [ $((i % 2)) -ne 0 ]; then
        continue  # Passer au nombre suivant
    fi

    echo "Traitement du nombre pair : $i"
    echo "  → Carré : $((i * i))"
done

echo ""

# Validation de fichiers avec continue
echo "=== Validation de fichiers ==="
fichiers=("script.sh" "document.txt" "" "image.jpg" "data")

for fichier in "${fichiers[@]}"; do
    # Ignorer les noms vides
    if [ -z "$fichier" ]; then
        echo "⚠️  Nom de fichier vide, ignoré"
        continue
    fi

    # Ignorer les fichiers sans extension
    if [[ "$fichier" != *.* ]]; then
        echo "⚠️  '$fichier' sans extension, ignoré"
        continue
    fi

    echo "✓ Fichier valide : $fichier"
done
```

### exit - Quitter le script

`exit` permet de **quitter complètement** le script avec un code de retour.

```bash
#!/bin/bash

# Vérification des prérequis avec exit
echo "=== Vérification des prérequis ==="

# Vérifier si l'utilisateur est root
if [ "$EUID" -eq 0 ]; then
    echo "❌ Ce script ne doit pas être exécuté en tant que root"
    exit 1  # Code d'erreur
fi

# Vérifier la présence d'un programme
if ! command -v git &> /dev/null; then
    echo "❌ Git n'est pas installé"
    echo "Installez Git avec : sudo apt install git"
    exit 2  # Code d'erreur différent
fi

# Vérifier l'existence d'un répertoire
if [ ! -d "$HOME/projets" ]; then
    echo "❌ Le répertoire ~/projets n'existe pas"
    read -p "Voulez-vous le créer ? (o/n) : " reponse
    if [ "$reponse" = "o" ]; then
        mkdir -p "$HOME/projets"
        echo "✓ Répertoire créé"
    else
        echo "Script annulé"
        exit 3
    fi
fi

echo "✓ Tous les prérequis sont satisfaits"
echo "Continuons..."

# Le reste du script...
exit 0  # Succès
```

### Codes de retour standards

```bash
#!/bin/bash

# Script démontrant les codes de retour
operation="$1"

case "$operation" in
    "success")
        echo "✓ Opération réussie"
        exit 0    # Succès
        ;;
    "warning")
        echo "⚠️  Avertissement détecté"
        exit 1    # Erreur générale
        ;;
    "notfound")
        echo "❌ Fichier non trouvé"
        exit 2    # Fichier non trouvé
        ;;
    "permission")
        echo "🚫 Permission refusée"
        exit 126  # Permission refusée
        ;;
    "command")
        echo "❓ Commande non trouvée"
        exit 127  # Commande non trouvée
        ;;
    *)
        echo "Usage : $0 {success|warning|notfound|permission|command}"
        exit 64   # Erreur d'usage
        ;;
esac
```

### Exemple complet avec tous les contrôles

```bash
#!/bin/bash

# Analyseur de logs avec contrôles de flux
fichier_log="$1"

# Vérification du paramètre
if [ $# -eq 0 ]; then
    echo "Usage : $0 fichier_log"
    exit 1
fi

# Vérification de l'existence du fichier
if [ ! -f "$fichier_log" ]; then
    echo "❌ Fichier '$fichier_log' introuvable"
    exit 2
fi

echo "=== Analyse du fichier de log : $fichier_log ==="

ligne_num=0
erreurs=0
avertissements=0

# Lecture ligne par ligne
while IFS= read -r ligne; do
    ((ligne_num++))

    # Ignorer les lignes vides
    if [ -z "$ligne" ]; then
        continue
    fi

    # Ignorer les commentaires
    if [[ "$ligne" =~ ^# ]]; then
        continue
    fi

    # Chercher les erreurs
    if [[ "$ligne" =~ ERROR|ERREUR ]]; then
        ((erreurs++))
        echo "🔴 Ligne $ligne_num - ERREUR : $ligne"

        # Arrêter si trop d'erreurs
        if [ $erreurs -ge 5 ]; then
            echo "⚠️  Trop d'erreurs détectées, arrêt de l'analyse"
            break
        fi
        continue
    fi

    # Chercher les avertissements
    if [[ "$ligne" =~ WARNING|ATTENTION ]]; then
        ((avertissements++))
        echo "🟡 Ligne $ligne_num - ATTENTION : $ligne"
        continue
    fi

    # Ligne normale (on peut la traiter)
    echo "✓ Ligne $ligne_num : OK"

done < "$fichier_log"

# Résumé
echo ""
echo "=== Résumé de l'analyse ==="
echo "Lignes analysées : $ligne_num"
echo "Erreurs trouvées : $erreurs"
echo "Avertissements : $avertissements"

# Déterminer le code de sortie selon les résultats
if [ $erreurs -gt 0 ]; then
    echo "🔴 État : ERREUR - Des problèmes critiques ont été détectés"
    exit 1
elif [ $avertissements -gt 0 ]; then
    echo "🟡 État : ATTENTION - Des avertissements ont été détectés"
    exit 1
else
    echo "✅ État : OK - Aucun problème détecté"
    exit 0
fi
```

### Combinaison de tous les contrôles de flux

```bash
#!/bin/bash

# Système de menu avec gestion complète des flux
afficher_menu_principal() {
    clear
    echo "╔══════════════════════════════════════╗"
    echo "║           SYSTÈME DE FICHIERS        ║"
    echo "╠══════════════════════════════════════╣"
    echo "║ 1. Gestionnaire de fichiers          ║"
    echo "║ 2. Calculatrice                      ║"
    echo "║ 3. Jeux                              ║"
    echo "║ 4. Outils système                    ║"
    echo "║ q. Quitter                           ║"
    echo "╚══════════════════════════════════════╝"
}

# Sous-menu fichiers
menu_fichiers() {
    while true; do
        echo ""
        echo "=== GESTIONNAIRE DE FICHIERS ==="
        echo "1. Lister fichiers"
        echo "2. Rechercher fichier"
        echo "3. Créer fichier"
        echo "r. Retour au menu principal"

        read -p "Choix : " choix

        case "$choix" in
            1)
                ls -la
                ;;
            2)
                read -p "Nom à rechercher : " nom
                find . -name "*$nom*" 2>/dev/null
                ;;
            3)
                read -p "Nom du nouveau fichier : " fichier
                if [ -z "$fichier" ]; then
                    echo "❌ Nom vide, opération annulée"
                    continue
                fi
                touch "$fichier"
                echo "✓ Fichier '$fichier' créé"
                ;;
            r|R)
                break  # Retour au menu principal
                ;;
            *)
                echo "❌ Choix invalide"
                continue
                ;;
        esac

        read -p "Appuyez sur Entrée..." -s
    done
}

# Calculatrice simple
calculatrice() {
    while true; do
        echo ""
        echo "=== CALCULATRICE ==="

        read -p "Premier nombre (ou 'q' pour quitter) : " num1
        if [ "$num1" = "q" ]; then
            break
        fi

        # Vérifier si c'est un nombre
        if ! [[ "$num1" =~ ^[0-9]+$ ]]; then
            echo "❌ Veuillez entrer un nombre valide"
            continue
        fi

        read -p "Opération (+, -, *, /) : " op
        read -p "Deuxième nombre : " num2

        if ! [[ "$num2" =~ ^[0-9]+$ ]]; then
            echo "❌ Veuillez entrer un nombre valide"
            continue
        fi

        case "$op" in
            +)
                resultat=$((num1 + num2))
                ;;
            -)
                resultat=$((num1 - num2))
                ;;
            \*)
                resultat=$((num1 * num2))
                ;;
            /)
                if [ "$num2" -eq 0 ]; then
                    echo "❌ Division par zéro impossible"
                    continue
                fi
                resultat=$((num1 / num2))
                ;;
            *)
                echo "❌ Opération invalide"
                continue
                ;;
        esac

        echo "✓ Résultat : $num1 $op $num2 = $resultat"
    done
}

# Jeu de devinette
jeu_devinette() {
    echo ""
    echo "=== JEU DE DEVINETTE ==="
    echo "Je pense à un nombre entre 1 et 100..."

    secret=$((RANDOM % 100 + 1))
    tentatives=0
    max_tentatives=7

    while [ $tentatives -lt $max_tentatives ]; do
        ((tentatives++))
        read -p "Tentative $tentatives/$max_tentatives - Votre nombre : " guess

        # Vérification de l'entrée
        if ! [[ "$guess" =~ ^[0-9]+$ ]] || [ "$guess" -lt 1 ] || [ "$guess" -gt 100 ]; then
            echo "❌ Entrez un nombre entre 1 et 100"
            ((tentatives--))  # Ne pas compter cette tentative
            continue
        fi

        if [ "$guess" -eq "$secret" ]; then
            echo "🎉 Bravo ! Vous avez trouvé en $tentatives tentative(s) !"
            break
        elif [ "$guess" -lt "$secret" ]; then
            echo "📈 Plus grand !"
        else
            echo "📉 Plus petit !"
        fi

        # Indices selon le nombre de tentatives
        if [ $tentatives -eq 3 ]; then
            if [ $((secret % 2)) -eq 0 ]; then
                echo "💡 Indice : le nombre est pair"
            else
                echo "💡 Indice : le nombre est impair"
            fi
        elif [ $tentatives -eq 5 ]; then
            if [ "$secret" -le 50 ]; then
                echo "💡 Indice : le nombre est ≤ 50"
            else
                echo "💡 Indice : le nombre est > 50"
            fi
        fi
    done

    if [ $tentatives -eq $max_tentatives ] && [ "$guess" -ne "$secret" ]; then
        echo "😞 Perdu ! Le nombre était : $secret"
    fi

    read -p "Appuyez sur Entrée pour continuer..." -s
}

# Menu jeux
menu_jeux() {
    while true; do
        echo ""
        echo "=== JEUX ==="
        echo "1. Devinette"
        echo "2. Pierre-Papier-Ciseaux"
        echo "r. Retour"

        read -p "Choix : " choix

        case "$choix" in
            1)
                jeu_devinette
                ;;
            2)
                echo "🚧 Jeu en développement..."
                read -p "Appuyez sur Entrée..." -s
                ;;
            r|R)
                break
                ;;
            *)
                echo "❌ Choix invalide"
                ;;
        esac
    done
}

# Outils système
outils_systeme() {
    while true; do
        echo ""
        echo "=== OUTILS SYSTÈME ==="
        echo "1. Informations système"
        echo "2. Espace disque"
        echo "3. Processus en cours"
        echo "4. Test de connectivité"
        echo "r. Retour"

        read -p "Choix : " choix

        case "$choix" in
            1)
                echo "💻 Système : $(uname -s)"
                echo "🏠 Utilisateur : $(whoami)"
                echo "📅 Date : $(date)"
                echo "⏰ Uptime : $(uptime)"
                ;;
            2)
                echo "💾 Espace disque :"
                df -h
                ;;
            3)
                echo "⚙️  Top 10 des processus :"
                ps aux | head -11
                ;;
            4)
                read -p "Adresse à tester (ex: google.com) : " adresse
                if [ -n "$adresse" ]; then
                    if ping -c 1 "$adresse" &>/dev/null; then
                        echo "✅ $adresse est accessible"
                    else
                        echo "❌ $adresse n'est pas accessible"
                    fi
                fi
                ;;
            r|R)
                break
                ;;
            *)
                echo "❌ Choix invalide"
                ;;
        esac

        read -p "Appuyez sur Entrée..." -s
    done
}

# Programme principal
main() {
    # Vérification des prérequis (exemple)
    if [ ! -w "." ]; then
        echo "❌ Pas de permissions d'écriture dans ce répertoire"
        exit 1
    fi

    echo "🚀 Démarrage du système..."

    # Boucle principale
    while true; do
        afficher_menu_principal
        read -p "Votre choix : " choix_principal

        case "$choix_principal" in
            1)
                menu_fichiers
                ;;
            2)
                calculatrice
                ;;
            3)
                menu_jeux
                ;;
            4)
                outils_systeme
                ;;
            q|Q|quit|exit)
                echo ""
                echo "👋 Merci d'avoir utilisé le système !"
                echo "À bientôt !"
                exit 0
                ;;
            *)
                echo "❌ Choix invalide. Utilisez 1-4 ou 'q' pour quitter."
                read -p "Appuyez sur Entrée..." -s
                ;;
        esac
    done
}

# Gestion des signaux (Ctrl+C)
trap 'echo -e "\n\n🛑 Interruption détectée. Au revoir !"; exit 130' INT

# Lancement du programme
main
```

## Exercices pratiques

### Exercice 1 : Gestionnaire de notes

```bash
#!/bin/bash

# Créez un script qui :
# 1. Demande le nom de l'étudiant
# 2. Demande les notes de 3 matières
# 3. Calcule la moyenne
# 4. Affiche la mention selon la moyenne
# 5. Sauvegarde le résultat dans un fichier

echo "=== GESTIONNAIRE DE NOTES ==="

# Votre code ici...
```

### Exercice 2 : Analyseur de fichiers

```bash
#!/bin/bash

# Créez un script qui :
# 1. Parcourt tous les fichiers du répertoire actuel
# 2. Classe les fichiers par type (texte, image, script, etc.)
# 3. Compte le nombre de fichiers de chaque type
# 4. Affiche un rapport final

echo "=== ANALYSEUR DE FICHIERS ==="

# Votre code ici...
```

### Exercice 3 : Menu de sauvegarde

```bash
#!/bin/bash

# Créez un système de menu qui propose :
# 1. Sauvegarde complète
# 2. Sauvegarde sélective
# 3. Restauration
# 4. Vérification des sauvegardes
# Avec gestion d'erreurs et confirmations

echo "=== SYSTÈME DE SAUVEGARDE ==="

# Votre code ici...
```

## Solutions des exercices

### Solution Exercice 1 : Gestionnaire de notes

```bash
#!/bin/bash

echo "=== GESTIONNAIRE DE NOTES ==="

# Saisie des informations
read -p "Nom de l'étudiant : " nom_etudiant

if [ -z "$nom_etudiant" ]; then
    echo "❌ Le nom ne peut pas être vide"
    exit 1
fi

# Saisie des notes avec validation
declare -a notes=()
matieres=("Mathématiques" "Français" "Histoire")

for matiere in "${matieres[@]}"; do
    while true; do
        read -p "Note en $matiere (0-20) : " note

        if [[ "$note" =~ ^[0-9]+$ ]] && [ "$note" -ge 0 ] && [ "$note" -le 20 ]; then
            notes+=("$note")
            break
        else
            echo "❌ Veuillez entrer une note entre 0 et 20"
        fi
    done
done

# Calcul de la moyenne
total=0
for note in "${notes[@]}"; do
    total=$((total + note))
done

moyenne=$((total * 100 / ${#notes[@]}))  # Moyenne * 100 pour garder 2 décimales
moyenne_affichage=$((moyenne / 100)).$((moyenne % 100))

# Détermination de la mention
if [ $moyenne -ge 1600 ]; then
    mention="Très bien"
elif [ $moyenne -ge 1400 ]; then
    mention="Bien"
elif [ $moyenne -ge 1200 ]; then
    mention="Assez bien"
elif [ $moyenne -ge 1000 ]; then
    mention="Passable"
else
    mention="Insuffisant"
fi

# Affichage des résultats
echo ""
echo "=== BULLETIN DE $nom_etudiant ==="
for i in "${!matieres[@]}"; do
    echo "${matieres[$i]} : ${notes[$i]}/20"
done
echo "Moyenne générale : $moyenne_affichage/20"
echo "Mention : $mention"

# Sauvegarde
fichier_bulletin="bulletin_${nom_etudiant// /_}.txt"
{
    echo "Bulletin de $nom_etudiant"
    echo "Date : $(date)"
    echo "========================"
    for i in "${!matieres[@]}"; do
        echo "${matieres[$i]} : ${notes[$i]}/20"
    done
    echo "Moyenne : $moyenne_affichage/20"
    echo "Mention : $mention"
} > "$fichier_bulletin"

echo ""
echo "✅ Bulletin sauvegardé dans : $fichier_bulletin"
```

### Solution Exercice 2 : Analyseur de fichiers

```bash
#!/bin/bash

echo "=== ANALYSEUR DE FICHIERS ==="

# Compteurs
declare -A compteurs
declare -A extensions

# Initialisation
compteurs[texte]=0
compteurs[image]=0
compteurs[script]=0
compteurs[document]=0
compteurs[audio]=0
compteurs[video]=0
compteurs[autre]=0

# Parcours des fichiers
for fichier in *; do
    # Ignorer les répertoires
    if [ -d "$fichier" ]; then
        continue
    fi

    # Obtenir l'extension
    extension="${fichier##*.}"
    extension=$(echo "$extension" | tr '[:upper:]' '[:lower:]')

    # Classification
    case "$extension" in
        txt|md|log|conf|cfg)
            type="texte"
            ;;
        jpg|jpeg|png|gif|bmp|svg)
            type="image"
            ;;
        sh|bash|py|pl|rb)
            type="script"
            ;;
        pdf|doc|docx|odt)
            type="document"
            ;;
        mp3|wav|flac|ogg)
            type="audio"
            ;;
        mp4|avi|mkv|mov)
            type="video"
            ;;
        *)
            type="autre"
            ;;
    esac

    # Compter
    ((compteurs[$type]++))

    # Garder trace des extensions
    if [ -n "${extensions[$extension]}" ]; then
        extensions[$extension]=$((extensions[$extension] + 1))
    else
        extensions[$extension]=1
    fi

    echo "📄 $fichier → $type (.$extension)"
done

# Rapport final
echo ""
echo "=== RAPPORT D'ANALYSE ==="
total=0
for type in texte image script document audio video autre; do
    count=${compteurs[$type]}
    if [ $count -gt 0 ]; then
        echo "$type : $count fichier(s)"
        total=$((total + count))
    fi
done

echo ""
echo "=== EXTENSIONS TROUVÉES ==="
for ext in "${!extensions[@]}"; do
    echo ".$ext : ${extensions[$ext]} fichier(s)"
done

echo ""
echo "TOTAL : $total fichier(s) analysé(s)"
```

## Récapitulatif

✅ **Ce que vous avez appris dans ce chapitre :**

1. **Conditions (if/elif/else) :** Prendre des décisions dans vos scripts
2. **Opérateurs de comparaison :** Comparer nombres, chaînes et fichiers
3. **Boucles (for/while/until) :** Répéter des actions automatiquement
4. **Instruction case :** Gérer plusieurs choix élégamment
5. **Contrôle de flux :** Utiliser break, continue, exit pour contrôler l'exécution

✅ **Points clés à retenir :**

**Conditions :**
```bash
if [ condition ]; then
    # actions
elif [ autre_condition ]; then
    # autres actions
else
    # actions par défaut
fi
```

**Boucles :**
```bash
# For
for item in liste; do
    # traitement
done

# While
while [ condition ]; do
    # actions
done

# Until
until [ condition ]; do
    # actions
done
```

**Case :**
```bash
case "$variable" in
    motif1)
        # actions
        ;;
    motif2|motif3)
        # actions
        ;;
    *)
        # défaut
        ;;
esac
```

**Contrôle de flux :**
- `break` : Sortir d'une boucle
- `continue` : Passer à l'itération suivante
- `exit` : Quitter le script (avec code de retour)

✅ **Prochaines étapes :**
Dans le chapitre 6, nous découvrirons les fonctions pour organiser et réutiliser notre code !

---

💡 **Conseil pratique :** Ces structures de contrôle sont le cœur de la programmation Bash. Pratiquez en créant des petits scripts qui utilisent ces concepts avant de passer au chapitre suivant. Essayez de refaire les exercices proposés !

⏭️
