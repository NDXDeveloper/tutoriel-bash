🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Tutoriel Bash - Chapitre 8 : Gestion des fichiers et répertoires

## Tests sur les fichiers

### Qu'est-ce que les tests de fichiers ?

Les tests de fichiers permettent de **vérifier l'état** d'un fichier ou répertoire avant de l'utiliser. C'est comme **regarder avant de traverser** : on vérifie si le fichier existe, s'il est lisible, s'il est vide, etc.

**Analogie :** Avant d'ouvrir une porte, vous vérifiez :
- Est-ce que la porte existe ?
- Est-elle verrouillée ?
- Y a-t-il quelqu'un derrière ?

De même, avant de manipuler un fichier, on vérifie ses propriétés.

### Tests de base d'existence

```bash
#!/bin/bash

echo "=== Tests d'existence de fichiers ==="

# Créer des éléments de test
touch fichier_test.txt
mkdir dossier_test
ln -s fichier_test.txt lien_symbolique

echo "Éléments créés pour les tests :"
ls -la

echo ""

# Test d'existence générale (-e)
fichiers_a_tester=("fichier_test.txt" "dossier_test" "fichier_inexistant.txt" "lien_symbolique")

for element in "${fichiers_a_tester[@]}"; do
    if [ -e "$element" ]; then
        echo "✅ '$element' existe"
    else
        echo "❌ '$element' n'existe pas"
    fi
done

echo ""

# Tests spécifiques de type
echo "=== Tests de type ==="

for element in "${fichiers_a_tester[@]}"; do
    if [ -f "$element" ]; then
        echo "📄 '$element' est un fichier régulier"
    elif [ -d "$element" ]; then
        echo "📁 '$element' est un répertoire"
    elif [ -L "$element" ]; then
        echo "🔗 '$element' est un lien symbolique"
    elif [ -e "$element" ]; then
        echo "❓ '$element' existe mais type inconnu"
    else
        echo "❌ '$element' n'existe pas"
    fi
done

# Nettoyer pour les tests suivants
rm -f lien_symbolique
```

### Tests de permissions

```bash
#!/bin/bash

echo "=== Tests de permissions ==="

# Créer des fichiers avec différentes permissions
touch fichier_lecture_seule.txt
touch fichier_executable.sh
touch fichier_normal.txt

chmod 444 fichier_lecture_seule.txt    # Lecture seule
chmod 755 fichier_executable.sh        # Exécutable
chmod 644 fichier_normal.txt           # Normal

echo "Fichiers créés avec permissions :"
ls -l fichier_*.txt fichier_*.sh

echo ""

# Fonction de test de permissions
tester_permissions() {
    local fichier="$1"

    echo "Tests pour '$fichier' :"

    if [ -r "$fichier" ]; then
        echo "  ✅ Lisible"
    else
        echo "  ❌ Non lisible"
    fi

    if [ -w "$fichier" ]; then
        echo "  ✅ Modifiable"
    else
        echo "  ❌ Non modifiable"
    fi

    if [ -x "$fichier" ]; then
        echo "  ✅ Exécutable"
    else
        echo "  ❌ Non exécutable"
    fi

    echo ""
}

# Tester les permissions
tester_permissions "fichier_lecture_seule.txt"
tester_permissions "fichier_executable.sh"
tester_permissions "fichier_normal.txt"
```

### Tests de contenu et de taille

```bash
#!/bin/bash

echo "=== Tests de contenu et taille ==="

# Créer des fichiers avec différents contenus
echo "Ce fichier contient du texte" > fichier_avec_contenu.txt
touch fichier_vide.txt
echo "Ligne 1
Ligne 2
Ligne 3" > fichier_plusieurs_lignes.txt

echo "Fichiers créés :"
ls -la fichier_*.txt

echo ""

# Fonction de test de contenu
analyser_fichier() {
    local fichier="$1"

    echo "Analyse de '$fichier' :"

    # Test d'existence
    if [ ! -f "$fichier" ]; then
        echo "  ❌ Fichier n'existe pas"
        return 1
    fi

    # Test de taille
    if [ -s "$fichier" ]; then
        local taille=$(stat -c%s "$fichier" 2>/dev/null || echo "0")
        echo "  📏 Fichier non vide ($taille octets)"
    else
        echo "  📄 Fichier vide"
    fi

    # Test de lisibilité
    if [ -r "$fichier" ]; then
        local nb_lignes=$(wc -l < "$fichier")
        local nb_mots=$(wc -w < "$fichier")
        echo "  📊 $nb_lignes ligne(s), $nb_mots mot(s)"

        # Afficher un aperçu si le fichier n'est pas trop gros
        if [ "$taille" -lt 1000 ]; then
            echo "  👁️  Aperçu :"
            head -3 "$fichier" | sed 's/^/    /'
            if [ $(wc -l < "$fichier") -gt 3 ]; then
                echo "    ..."
            fi
        fi
    else
        echo "  ❌ Fichier non lisible"
    fi

    echo ""
}

# Analyser tous les fichiers
for fichier in fichier_*.txt; do
    analyser_fichier "$fichier"
done
```

### Tests avancés et comparaisons

```bash
#!/bin/bash

echo "=== Tests avancés et comparaisons ==="

# Créer des fichiers pour les comparaisons
echo "Version 1" > version1.txt
sleep 2  # Attendre pour avoir des timestamps différents
echo "Version 2" > version2.txt
cp version1.txt copie_version1.txt

echo "Fichiers pour comparaison :"
ls -la version*.txt copie_*.txt

echo ""

# Fonction de comparaison de fichiers
comparer_fichiers() {
    local fichier1="$1"
    local fichier2="$2"

    echo "Comparaison de '$fichier1' et '$fichier2' :"

    # Vérifier l'existence
    if [ ! -f "$fichier1" ] || [ ! -f "$fichier2" ]; then
        echo "  ❌ Un des fichiers n'existe pas"
        return 1
    fi

    # Comparaison par le contenu
    if cmp -s "$fichier1" "$fichier2"; then
        echo "  ✅ Contenus identiques"
    else
        echo "  ❌ Contenus différents"
    fi

    # Comparaison par la taille
    local taille1=$(stat -c%s "$fichier1")
    local taille2=$(stat -c%s "$fichier2")

    if [ "$taille1" -eq "$taille2" ]; then
        echo "  ✅ Tailles identiques ($taille1 octets)"
    else
        echo "  ❌ Tailles différentes ($taille1 vs $taille2 octets)"
    fi

    # Comparaison par la date de modification
    if [ "$fichier1" -nt "$fichier2" ]; then
        echo "  📅 '$fichier1' est plus récent"
    elif [ "$fichier1" -ot "$fichier2" ]; then
        echo "  📅 '$fichier2' est plus récent"
    else
        echo "  📅 Même date de modification"
    fi

    echo ""
}

# Effectuer les comparaisons
comparer_fichiers "version1.txt" "version2.txt"
comparer_fichiers "version1.txt" "copie_version1.txt"
```

### Fonction utilitaire de validation de fichier

```bash
#!/bin/bash

# Fonction complète de validation de fichier
valider_fichier() {
    local fichier="$1"
    local type_attendu="${2:-fichier}"  # fichier, dossier, executable
    local taille_max="${3:-1048576}"    # 1MB par défaut

    local erreurs=()
    local avertissements=()

    echo "=== Validation de '$fichier' ==="

    # Test d'existence
    if [ ! -e "$fichier" ]; then
        erreurs+=("Fichier inexistant")
    else
        # Tests selon le type attendu
        case "$type_attendu" in
            "fichier")
                if [ ! -f "$fichier" ]; then
                    erreurs+=("N'est pas un fichier régulier")
                fi
                ;;
            "dossier")
                if [ ! -d "$fichier" ]; then
                    erreurs+=("N'est pas un répertoire")
                fi
                ;;
            "executable")
                if [ ! -f "$fichier" ]; then
                    erreurs+=("N'est pas un fichier")
                elif [ ! -x "$fichier" ]; then
                    erreurs+=("N'est pas exécutable")
                fi
                ;;
        esac

        # Tests de permissions
        if [ ! -r "$fichier" ]; then
            erreurs+=("Non lisible")
        fi

        # Test de taille pour les fichiers
        if [ -f "$fichier" ]; then
            local taille=$(stat -c%s "$fichier" 2>/dev/null || echo "0")
            if [ "$taille" -gt "$taille_max" ]; then
                avertissements+=("Fichier volumineux ($taille > $taille_max octets)")
            fi

            if [ "$taille" -eq 0 ]; then
                avertissements+=("Fichier vide")
            fi
        fi

        # Tests de sécurité
        if [[ "$fichier" =~ \.\./|^/ ]]; then
            avertissements+=("Chemin potentiellement dangereux")
        fi
    fi

    # Affichage des résultats
    if [ ${#erreurs[@]} -eq 0 ] && [ ${#avertissements[@]} -eq 0 ]; then
        echo "✅ Fichier valide"
        return 0
    else
        if [ ${#erreurs[@]} -gt 0 ]; then
            echo "❌ Erreurs détectées :"
            for erreur in "${erreurs[@]}"; do
                echo "  - $erreur"
            done
        fi

        if [ ${#avertissements[@]} -gt 0 ]; then
            echo "⚠️  Avertissements :"
            for avertissement in "${avertissements[@]}"; do
                echo "  - $avertissement"
            done
        fi

        if [ ${#erreurs[@]} -gt 0 ]; then
            return 1
        else
            return 0
        fi
    fi
}

# Tests de la fonction de validation
echo "=== Tests de validation ==="

# Créer des fichiers de test
echo "Contenu normal" > fichier_normal.txt
touch fichier_vide.txt
echo "#!/bin/bash
echo 'Hello World'" > script.sh
chmod +x script.sh
mkdir dossier_test

# Tests
valider_fichier "fichier_normal.txt" "fichier"
echo ""
valider_fichier "fichier_vide.txt" "fichier"
echo ""
valider_fichier "script.sh" "executable"
echo ""
valider_fichier "dossier_test" "dossier"
echo ""
valider_fichier "fichier_inexistant.txt" "fichier"

# Nettoyer les fichiers de test
rm -f fichier_*.txt script.sh version*.txt copie_*.txt
rm -rf dossier_test
```

## Parcours de répertoires

### Parcours simple avec for

```bash
#!/bin/bash

echo "=== Parcours simple de répertoires ==="

# Créer une structure de test
mkdir -p test_structure/{docs,images,scripts}
touch test_structure/docs/{readme.txt,manual.pdf}
touch test_structure/images/{photo1.jpg,photo2.png}
touch test_structure/scripts/{backup.sh,deploy.sh}
touch test_structure/config.ini

echo "Structure créée :"
tree test_structure 2>/dev/null || find test_structure -type f | sort

echo ""

# Parcours simple du répertoire racine
echo "=== Contenu du répertoire test_structure ==="
for element in test_structure/*; do
    if [ -f "$element" ]; then
        echo "📄 $(basename "$element") (fichier)"
    elif [ -d "$element" ]; then
        echo "📁 $(basename "$element") (dossier)"
    fi
done

echo ""

# Parcours avec gestion des extensions
echo "=== Classification par extension ==="
declare -A extensions

for fichier in test_structure/**/*; do
    if [ -f "$fichier" ]; then
        extension="${fichier##*.}"
        extensions["$extension"]=$((${extensions["$extension"]} + 1))
        echo "  $fichier → .$extension"
    fi
done

echo ""
echo "Résumé par extension :"
for ext in "${!extensions[@]}"; do
    echo "  .$ext : ${extensions[$ext]} fichier(s)"
done
```

### Parcours récursif avec find

```bash
#!/bin/bash

echo "=== Parcours récursif avec find ==="

echo "Tous les fichiers (récursif) :"
find test_structure -type f | while read -r fichier; do
    taille=$(stat -c%s "$fichier")
    echo "  📄 $fichier ($taille octets)"
done

echo ""

echo "Tous les répertoires :"
find test_structure -type d | while read -r dossier; do
    nb_elements=$(find "$dossier" -maxdepth 1 | wc -l)
    echo "  📁 $dossier ($((nb_elements - 1)) élément(s))"
done

echo ""

echo "Fichiers par type :"
echo "  Scripts (.sh) :"
find test_structure -name "*.sh" -type f | sed 's/^/    /'

echo "  Images :"
find test_structure \( -name "*.jpg" -o -name "*.png" -o -name "*.gif" \) -type f | sed 's/^/    /'

echo "  Documents :"
find test_structure \( -name "*.txt" -o -name "*.pdf" -o -name "*.doc" \) -type f | sed 's/^/    /'
```

### Parcours avec actions conditionnelles

```bash
#!/bin/bash

echo "=== Parcours avec actions conditionnelles ==="

# Fonction de traitement par type de fichier
traiter_fichier() {
    local fichier="$1"
    local nom_fichier=$(basename "$fichier")
    local extension="${fichier##*.}"
    local taille=$(stat -c%s "$fichier" 2>/dev/null || echo "0")

    case "$extension" in
        sh)
            echo "🔧 Script: $nom_fichier"
            if [ -x "$fichier" ]; then
                echo "   ✅ Exécutable"
            else
                echo "   ⚠️  Non exécutable - Correction..."
                chmod +x "$fichier"
            fi
            ;;
        txt)
            echo "📝 Document texte: $nom_fichier"
            local nb_lignes=$(wc -l < "$fichier" 2>/dev/null || echo "0")
            echo "   📊 $nb_lignes ligne(s)"
            ;;
        jpg|png|gif)
            echo "🖼️  Image: $nom_fichier"
            echo "   📏 $taille octets"
            ;;
        pdf)
            echo "📄 Document PDF: $nom_fichier"
            echo "   📏 $taille octets"
            ;;
        *)
            echo "❓ Fichier: $nom_fichier (type: .$extension)"
            ;;
    esac
}

# Parcourir et traiter tous les fichiers
echo "Traitement de tous les fichiers :"
find test_structure -type f | while read -r fichier; do
    traiter_fichier "$fichier"
done
```

### Fonction de parcours personnalisée

```bash
#!/bin/bash

# Fonction de parcours récursif personnalisée
parcourir_recursif() {
    local repertoire="$1"
    local niveau="${2:-0}"
    local action="${3:-afficher}"

    # Créer l'indentation
    local indent=""
    for ((i=0; i<niveau; i++)); do
        indent+="  "
    done

    # Traiter le répertoire actuel
    if [ "$niveau" -eq 0 ]; then
        echo "📁 $(basename "$repertoire")/"
    fi

    # Parcourir le contenu
    for element in "$repertoire"/*; do
        [ ! -e "$element" ] && continue  # Ignorer si pas de correspondance

        local nom=$(basename "$element")

        if [ -d "$element" ]; then
            echo "${indent}📁 $nom/"

            # Appel récursif pour les sous-répertoires
            if [ "$niveau" -lt 5 ]; then  # Limiter la profondeur
                parcourir_recursif "$element" $((niveau + 1)) "$action"
            else
                echo "${indent}  ... (profondeur limitée)"
            fi
        else
            # Traiter selon l'action demandée
            case "$action" in
                "afficher")
                    echo "${indent}📄 $nom"
                    ;;
                "taille")
                    local taille=$(stat -c%s "$element")
                    echo "${indent}📄 $nom ($taille octets)"
                    ;;
                "permissions")
                    local perms=$(stat -c%A "$element")
                    echo "${indent}📄 $nom ($perms)"
                    ;;
                "complet")
                    local taille=$(stat -c%s "$element")
                    local date=$(stat -c%y "$element" | cut -d' ' -f1)
                    echo "${indent}📄 $nom ($taille octets, $date)"
                    ;;
            esac
        fi
    done
}

echo ""
echo "=== Parcours personnalisé ==="

echo "Affichage simple :"
parcourir_recursif "test_structure" 0 "afficher"

echo ""
echo "Avec tailles :"
parcourir_recursif "test_structure" 0 "taille"

echo ""
echo "Avec informations complètes :"
parcourir_recursif "test_structure" 0 "complet"
```

### Recherche et filtrage avancés

```bash
#!/bin/bash

echo "=== Recherche et filtrage avancés ==="

# Fonction de recherche multicritères
rechercher_fichiers() {
    local repertoire="$1"
    local -A criteres

    # Parser les critères (format: cle=valeur)
    shift
    for critere in "$@"; do
        if [[ "$critere" =~ ^([^=]+)=(.+)$ ]]; then
            criteres["${BASH_REMATCH[1]}"]="${BASH_REMATCH[2]}"
        fi
    done

    echo "Recherche dans '$repertoire' avec critères :"
    for cle in "${!criteres[@]}"; do
        echo "  $cle = ${criteres[$cle]}"
    done
    echo ""

    # Construire la commande find
    local cmd_find="find '$repertoire' -type f"

    # Ajouter les critères
    if [ -n "${criteres[nom]}" ]; then
        cmd_find+=" -name '${criteres[nom]}'"
    fi

    if [ -n "${criteres[extension]}" ]; then
        cmd_find+=" -name '*.${criteres[extension]}'"
    fi

    if [ -n "${criteres[taille_min]}" ]; then
        cmd_find+=" -size +${criteres[taille_min]}c"
    fi

    if [ -n "${criteres[taille_max]}" ]; then
        cmd_find+=" -size -${criteres[taille_max]}c"
    fi

    if [ -n "${criteres[jours]}" ]; then
        cmd_find+=" -mtime -${criteres[jours]}"
    fi

    # Exécuter la recherche
    local resultats=$(eval "$cmd_find" 2>/dev/null)

    if [ -z "$resultats" ]; then
        echo "Aucun fichier trouvé."
        return 1
    fi

    echo "Fichiers trouvés :"
    while IFS= read -r fichier; do
        local taille=$(stat -c%s "$fichier")
        local date=$(stat -c%y "$fichier" | cut -d' ' -f1)
        echo "  📄 $fichier ($taille octets, $date)"
    done <<< "$resultats"

    local nb_resultats=$(echo "$resultats" | wc -l)
    echo ""
    echo "Total : $nb_resultats fichier(s) trouvé(s)"
}

# Tests de recherche
echo "Recherche de tous les scripts :"
rechercher_fichiers "test_structure" "extension=sh"

echo ""
echo "Recherche de fichiers contenant 'photo' :"
rechercher_fichiers "test_structure" "nom=*photo*"

echo ""
echo "Recherche de petits fichiers (< 100 octets) :"
rechercher_fichiers "test_structure" "taille_max=100"
```

## Manipulation de chemins

### Extraction d'informations de chemins

```bash
#!/bin/bash

echo "=== Manipulation de chemins ==="

# Exemples de chemins
chemins=(
    "/home/utilisateur/documents/rapport.pdf"
    "./scripts/backup.sh"
    "../config/app.conf"
    "images/photo.jpg"
    "/var/log/system.log"
    "fichier_sans_extension"
    "/chemin/vers/fichier.tar.gz"
)

echo "Analyse des chemins :"
for chemin in "${chemins[@]}"; do
    echo ""
    echo "Chemin : '$chemin'"

    # Nom de fichier complet
    nom_fichier=$(basename "$chemin")
    echo "  Nom de fichier : '$nom_fichier'"

    # Répertoire parent
    repertoire=$(dirname "$chemin")
    echo "  Répertoire : '$repertoire'"

    # Extension
    if [[ "$nom_fichier" == *.* ]]; then
        extension="${nom_fichier##*.}"
        nom_base="${nom_fichier%.*}"
        echo "  Nom de base : '$nom_base'"
        echo "  Extension : '$extension'"
    else
        echo "  Pas d'extension"
    fi

    # Type de chemin
    if [[ "$chemin" == /* ]]; then
        echo "  Type : Chemin absolu"
    elif [[ "$chemin" == ./* ]]; then
        echo "  Type : Chemin relatif (répertoire actuel)"
    elif [[ "$chemin" == ../* ]]; then
        echo "  Type : Chemin relatif (répertoire parent)"
    else
        echo "  Type : Chemin relatif"
    fi
done
```

### Construction de chemins

```bash
#!/bin/bash

echo "=== Construction de chemins ==="

# Fonction pour joindre des chemins proprement
joindre_chemin() {
    local chemin_base="$1"
    shift
    local resultat="$chemin_base"

    for partie in "$@"; do
        # Supprimer les slashes en fin de chemin_base
        resultat="${resultat%/}"

        # Supprimer les slashes en début de partie
        partie="${partie#/}"

        # Joindre avec un slash
        if [ -n "$partie" ]; then
            resultat="$resultat/$partie"
        fi
    done

    echo "$resultat"
}

# Tests de construction
echo "Construction de chemins :"
echo "Base : '/home/user', parties : 'documents', 'projets', 'mon_script.sh'"
chemin_construit=$(joindre_chemin "/home/user" "documents" "projets" "mon_script.sh")
echo "Résultat : '$chemin_construit'"

echo ""
echo "Base : './scripts/', parties : '../config/', 'app.conf'"
chemin_construit=$(joindre_chemin "./scripts/" "../config/" "app.conf")
echo "Résultat : '$chemin_construit'"

# Fonction de normalisation de chemin
normaliser_chemin() {
    local chemin="$1"

    # Remplacer les doubles slashes
    chemin="${chemin//\/\//\/}"

    # Supprimer le slash final (sauf pour la racine)
    if [ "$chemin" != "/" ]; then
        chemin="${chemin%/}"
    fi

    # Traiter les . et .. (simple)
    # Note : Pour une normalisation complète, utiliser readlink -f
    echo "$chemin"
}

echo ""
echo "=== Normalisation de chemins ==="
chemins_a_normaliser=(
    "/home//user/documents/"
    "./scripts/../config"
    "///var///log///"
    "."
)

for chemin in "${chemins_a_normaliser[@]}"; do
    normalise=$(normaliser_chemin "$chemin")
    echo "'$chemin' → '$normalise'"
done
```

### Résolution et validation de chemins

```bash
#!/bin/bash

echo "=== Résolution et validation de chemins ==="

# Fonction de validation de chemin
valider_chemin() {
    local chemin="$1"
    local type_attendu="${2:-any}"  # file, dir, any

    echo "Validation de '$chemin' :"

    # Expansion du chemin
    local chemin_absolu
    if [[ "$chemin" == /* ]]; then
        chemin_absolu="$chemin"
    else
        chemin_absolu="$(pwd)/$chemin"
    fi

    echo "  Chemin absolu : '$chemin_absolu'"

    # Normalisation avec readlink si disponible
    if command -v readlink >/dev/null 2>&1; then
        local chemin_normalise=$(readlink -m "$chemin" 2>/dev/null || echo "$chemin")
        echo "  Chemin normalisé : '$chemin_normalise'"
    fi

    # Tests d'existence et de type
    if [ ! -e "$chemin" ]; then
        echo "  ❌ N'existe pas"
        return 1
    fi

    case "$type_attendu" in
        "file")
            if [ -f "$chemin" ]; then
                echo "  ✅ Fichier valide"
            else
                echo "  ❌ N'est pas un fichier"
                return 1
            fi
            ;;
        "dir")
            if [ -d "$chemin" ]; then
                echo "  ✅ Répertoire valide"
            else
                echo "  ❌ N'est pas un répertoire"
                return 1
            fi
            ;;
        "any")
            if [ -f "$chemin" ]; then
                echo "  ✅ Fichier"
            elif [ -d "$chemin" ]; then
                echo "  ✅ Répertoire"
            else
                echo "  ✅ Existe (type spécial)"
            fi
            ;;
    esac

    # Tests de permissions
    local permissions=""
    [ -r "$chemin" ] && permissions+="r" || permissions+="-"
    [ -w "$chemin" ] && permissions+="w" || permissions+="-"
    [ -x "$chemin" ] && permissions+="x" || permissions+="-"

    echo "  Permissions : $permissions"

    return 0
}

# Tests de validation
echo "Tests de validation :"
chemins_test=(
    "test_structure"
    "test_structure/docs/readme.txt"
    "test_structure/scripts/backup.sh"
    "chemin_inexistant"
    "."
    ".."
)

for chemin in "${chemins_test[@]}"; do
    valider_chemin "$chemin"
    echo ""
done
```

### Utilitaires de manipulation de chemins

```bash
#!/bin/bash

echo "=== Utilitaires de manipulation de chemins ==="

# Fonction pour créer un chemin sécurisé
creer_chemin_securise() {
    local nom_fichier="$1"
    local repertoire_base="${2:-./}"

    # Nettoyer le nom de fichier
    # Supprimer les caractères dangereux
    nom_fichier="${nom_fichier//[<>:\"|?*]/}"
    nom_fichier="${nom_fichier//\//_}"  # Remplacer / par _
    nom_fichier="${nom_fichier//\\/_}"  # Remplacer \ par _

    # Supprimer les espaces en début et fin
    nom_fichier=$(echo "$nom_fichier" | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')

    # Remplacer les espaces par des underscores
    nom_fichier="${nom_fichier// /_}"

    # Limiter la longueur
    if [ ${#nom_fichier} -gt 100 ]; then
        nom_fichier="${nom_fichier:0:100}"
    fi

    # Construire le chemin final
    local chemin_final=$(joindre_chemin "$repertoire_base" "$nom_fichier")

    echo "$chemin_final"
}

# Fonction pour créer un nom de fichier unique
creer_nom_unique() {
    local chemin_base="$1"
    local compteur=1
    local chemin_original="$chemin_base"

    # Séparer nom de base et extension
    local repertoire=$(dirname "$chemin_base")
    local nom_fichier=$(basename "$chemin_base")
    local extension=""
    local nom_base="$nom_fichier"

    if [[ "$nom_fichier" == *.* ]]; then
        extension=".${nom_fichier##*.}"
        nom_base="${nom_fichier%.*}"
    fi

    # Chercher un nom disponible
    while [ -e "$chemin_base" ]; do
        if [ "$compteur" -eq 1 ]; then
            chemin_base="$repertoire/${nom_base}_$compteur$extension"
        else
            chemin_base="$repertoire/${nom_base}_$compteur$extension"
        fi
        ((compteur++))

        # Éviter les boucles infinies
        if [ "$compteur" -gt 1000 ]; then
            chemin_base="$repertoire/${nom_base}_$(date +%s)$extension"
            break
        fi
    done

    echo "$chemin_base"
}

# Fonction pour obtenir un chemin relatif
obtenir_chemin_relatif() {
    local chemin_source="$1"
    local chemin_cible="$2"

    # Utiliser realpath si disponible
    if command -v realpath >/dev/null 2>&1; then
        realpath --relative-to="$chemin_source" "$chemin_cible" 2>/dev/null
    else
        # Méthode alternative simple
        echo "$chemin_cible"
    fi
}

# Tests des utilitaires
echo "Tests des utilitaires de chemins :"

echo ""
echo "Création de chemins sécurisés :"
noms_dangereux=(
    "mon fichier<>important.txt"
    "script|dangereux.sh"
    "  fichier avec espaces  "
    "très_long_nom_de_fichier_qui_dépasse_largement_les_limites_raisonnables_et_qui_pourrait_poser_des_problèmes.txt"
)

for nom in "${noms_dangereux[@]}"; do
    securise=$(creer_chemin_securise "$nom" "test_structure/docs")
    echo "  '$nom' → '$securise'"
done

echo ""
echo "Création de noms uniques :"
# Créer quelques fichiers pour tester l'unicité
touch test_structure/nouveau_fichier.txt
touch test_structure/nouveau_fichier_1.txt

for i in {1..3}; do
    unique=$(creer_nom_unique "test_structure/nouveau_fichier.txt")
    echo "  Nom unique $i : '$unique'"
    touch "$unique"  # Créer le fichier pour le test suivant
done
```

## Opérations courantes (copie, déplacement, suppression)

### Copie de fichiers et répertoires

```bash
#!/bin/bash

echo "=== Opérations de copie ==="

# Fonction de copie sécurisée
copier_securise() {
    local source="$1"
    local destination="$2"
    local options="${3:-normal}"  # normal, force, backup

    echo "Copie de '$source' vers '$destination' (mode: $options)"

    # Vérifications préliminaires
    if [ ! -e "$source" ]; then
        echo "  ❌ Source inexistante"
        return 1
    fi

    # Créer le répertoire de destination si nécessaire
    local dir_dest=$(dirname "$destination")
    if [ ! -d "$dir_dest" ]; then
        echo "  📁 Création du répertoire de destination : $dir_dest"
        mkdir -p "$dir_dest" || {
            echo "  ❌ Impossible de créer le répertoire de destination"
            return 1
        }
    fi

    # Gestion des différents modes
    case "$options" in
        "normal")
            if [ -e "$destination" ]; then
                echo "  ⚠️  La destination existe déjà"
                read -p "  Écraser ? (o/n) : " reponse
                if [[ ! "$reponse" =~ ^[oO] ]]; then
                    echo "  ❌ Copie annulée"
                    return 1
                fi
            fi
            ;;
        "backup")
            if [ -e "$destination" ]; then
                local backup="${destination}.backup.$(date +%Y%m%d_%H%M%S)"
                echo "  💾 Sauvegarde existant vers : $backup"
                mv "$destination" "$backup"
            fi
            ;;
        "force")
            # Pas de vérification, écraser directement
            ;;
    esac

    # Effectuer la copie
    if [ -d "$source" ]; then
        echo "  📁 Copie récursive de répertoire..."
        if cp -r "$source" "$destination"; then
            echo "  ✅ Copie réussie"
        else
            echo "  ❌ Erreur lors de la copie"
            return 1
        fi
    else
        echo "  📄 Copie de fichier..."
        if cp "$source" "$destination"; then
            echo "  ✅ Copie réussie"

            # Vérification de l'intégrité
            if command -v sha256sum >/dev/null 2>&1; then
                local hash_source=$(sha256sum "$source" | cut -d' ' -f1)
                local hash_dest=$(sha256sum "$destination" | cut -d' ' -f1)

                if [ "$hash_source" = "$hash_dest" ]; then
                    echo "  ✅ Intégrité vérifiée"
                else
                    echo "  ❌ Erreur d'intégrité!"
                    return 1
                fi
            fi
        else
            echo "  ❌ Erreur lors de la copie"
            return 1
        fi
    fi

    # Afficher les informations sur le fichier copié
    if [ -e "$destination" ]; then
        local taille=$(stat -c%s "$destination" 2>/dev/null || echo "N/A")
        echo "  📊 Taille : $taille octets"
    fi

    return 0
}

# Tests de copie
echo "Tests de copie :"

# Créer des fichiers de test pour la copie
echo "Contenu original" > test_structure/fichier_a_copier.txt
mkdir -p test_structure/backup

echo ""
echo "1. Copie normale :"
copier_securise "test_structure/fichier_a_copier.txt" "test_structure/backup/copie1.txt" "normal"

echo ""
echo "2. Copie avec backup :"
copier_securise "test_structure/fichier_a_copier.txt" "test_structure/backup/copie1.txt" "backup"

echo ""
echo "3. Copie de répertoire :"
copier_securise "test_structure/docs" "test_structure/backup/docs_copy" "normal"
```

### Déplacement et renommage

```bash
#!/bin/bash

echo "=== Opérations de déplacement ==="

# Fonction de déplacement sécurisé
deplacer_securise() {
    local source="$1"
    local destination="$2"
    local options="${3:-normal}"  # normal, force, backup

    echo "Déplacement de '$source' vers '$destination' (mode: $options)"

    # Vérifications
    if [ ! -e "$source" ]; then
        echo "  ❌ Source inexistante"
        return 1
    fi

    if [ "$source" -ef "$destination" ]; then
        echo "  ⚠️  Source et destination identiques"
        return 0
    fi

    # Créer le répertoire de destination si nécessaire
    local dir_dest=$(dirname "$destination")
    if [ ! -d "$dir_dest" ]; then
        echo "  📁 Création du répertoire de destination : $dir_dest"
        mkdir -p "$dir_dest"
    fi

    # Gestion de l'écrasement
    if [ -e "$destination" ]; then
        case "$options" in
            "normal")
                echo "  ⚠️  La destination existe déjà"
                read -p "  Écraser ? (o/n) : " reponse
                if [[ ! "$reponse" =~ ^[oO] ]]; then
                    echo "  ❌ Déplacement annulé"
                    return 1
                fi
                ;;
            "backup")
                local backup="${destination}.backup.$(date +%Y%m%d_%H%M%S)"
                echo "  💾 Sauvegarde existant vers : $backup"
                mv "$destination" "$backup"
                ;;
        esac
    fi

    # Effectuer le déplacement
    if mv "$source" "$destination"; then
        echo "  ✅ Déplacement réussi"

        # Vérifier que la source n'existe plus
        if [ ! -e "$source" ]; then
            echo "  ✅ Source supprimée"
        else
            echo "  ⚠️  La source existe encore (cas spécial)"
        fi

        return 0
    else
        echo "  ❌ Erreur lors du déplacement"
        return 1
    fi
}

# Fonction de renommage intelligent
renommer_intelligent() {
    local ancien_nom="$1"
    local nouveau_nom="$2"

    echo "Renommage de '$ancien_nom' vers '$nouveau_nom'"

    # Vérifications
    if [ ! -e "$ancien_nom" ]; then
        echo "  ❌ Fichier source inexistant"
        return 1
    fi

    # Nettoyer le nouveau nom
    nouveau_nom=$(creer_chemin_securise "$nouveau_nom" "$(dirname "$ancien_nom")")

    # Rendre unique si nécessaire
    if [ -e "$nouveau_nom" ] && [ "$ancien_nom" != "$nouveau_nom" ]; then
        nouveau_nom=$(creer_nom_unique "$nouveau_nom")
        echo "  📝 Nom ajusté pour éviter les conflits : $(basename "$nouveau_nom")"
    fi

    # Effectuer le renommage
    if [ "$ancien_nom" != "$nouveau_nom" ]; then
        if mv "$ancien_nom" "$nouveau_nom"; then
            echo "  ✅ Renommage réussi vers : $nouveau_nom"
        else
            echo "  ❌ Erreur lors du renommage"
            return 1
        fi
    else
        echo "  ℹ️  Aucun changement nécessaire"
    fi
}

# Tests de déplacement
echo "Tests de déplacement :"

# Créer des fichiers pour les tests
echo "Fichier à déplacer" > test_structure/a_deplacer.txt
echo "Fichier à renommer" > test_structure/a_renommer.txt

echo ""
echo "1. Déplacement simple :"
deplacer_securise "test_structure/a_deplacer.txt" "test_structure/backup/fichier_deplace.txt" "normal"

echo ""
echo "2. Renommage intelligent :"
renommer_intelligent "test_structure/a_renommer.txt" "fichier<>renommé avec espaces.txt"
```

### Suppression sécurisée

```bash
#!/bin/bash

echo "=== Opérations de suppression ==="

# Fonction de suppression sécurisée
supprimer_securise() {
    local cible="$1"
    local mode="${2:-normal}"  # normal, force, corbeille

    echo "Suppression de '$cible' (mode: $mode)"

    # Vérifications
    if [ ! -e "$cible" ]; then
        echo "  ❌ Élément inexistant"
        return 1
    fi

    # Afficher les informations sur l'élément à supprimer
    if [ -f "$cible" ]; then
        local taille=$(stat -c%s "$cible")
        echo "  📄 Fichier ($taille octets)"
    elif [ -d "$cible" ]; then
        local nb_elements=$(find "$cible" -type f | wc -l)
        echo "  📁 Répertoire ($nb_elements fichier(s))"
    fi

    # Gestion selon le mode
    case "$mode" in
        "normal")
            echo "  ⚠️  Suppression définitive"
            read -p "  Confirmer la suppression ? (oui/non) : " confirmation
            if [ "$confirmation" != "oui" ]; then
                echo "  ❌ Suppression annulée"
                return 1
            fi
            ;;
        "corbeille")
            # Simuler une corbeille
            local corbeille="$HOME/.local/share/Trash/files"
            if [ ! -d "$corbeille" ]; then
                mkdir -p "$corbeille"
            fi

            local nom_corbeille="$(basename "$cible").$(date +%Y%m%d_%H%M%S)"
            echo "  🗑️  Déplacement vers la corbeille : $nom_corbeille"

            if mv "$cible" "$corbeille/$nom_corbeille"; then
                echo "  ✅ Déplacé vers la corbeille"
                return 0
            else
                echo "  ❌ Erreur lors du déplacement vers la corbeille"
                return 1
            fi
            ;;
        "force")
            echo "  ⚡ Suppression forcée"
            ;;
    esac

    # Effectuer la suppression
    if [ -d "$cible" ]; then
        if rm -rf "$cible"; then
            echo "  ✅ Répertoire supprimé"
        else
            echo "  ❌ Erreur lors de la suppression du répertoire"
            return 1
        fi
    else
        if rm -f "$cible"; then
            echo "  ✅ Fichier supprimé"
        else
            echo "  ❌ Erreur lors de la suppression du fichier"
            return 1
        fi
    fi

    return 0
}

# Fonction de nettoyage par critères
nettoyer_repertoire() {
    local repertoire="$1"
    local criteres="$2"  # temp, vieux, gros, vide

    echo "Nettoyage de '$repertoire' (critères: $criteres)"

    if [ ! -d "$repertoire" ]; then
        echo "  ❌ Répertoire inexistant"
        return 1
    fi

    local fichiers_supprimes=0

    case "$criteres" in
        "temp")
            echo "  🧹 Suppression des fichiers temporaires..."
            find "$repertoire" -type f \( -name "*.tmp" -o -name "*.temp" -o -name "*~" -o -name "*.bak" \) | while read -r fichier; do
                echo "    Suppression : $(basename "$fichier")"
                rm -f "$fichier"
                ((fichiers_supprimes++))
            done
            ;;
        "vieux")
            echo "  📅 Suppression des fichiers de plus de 30 jours..."
            find "$repertoire" -type f -mtime +30 | while read -r fichier; do
                echo "    Suppression : $(basename "$fichier") ($(stat -c%y "$fichier" | cut -d' ' -f1))"
                rm -f "$fichier"
                ((fichiers_supprimes++))
            done
            ;;
        "gros")
            echo "  📏 Suppression des fichiers > 10MB..."
            find "$repertoire" -type f -size +10M | while read -r fichier; do
                local taille=$(stat -c%s "$fichier")
                echo "    Suppression : $(basename "$fichier") ($taille octets)"
                rm -f "$fichier"
                ((fichiers_supprimes++))
            done
            ;;
        "vide")
            echo "  📄 Suppression des fichiers vides..."
            find "$repertoire" -type f -empty | while read -r fichier; do
                echo "    Suppression : $(basename "$fichier")"
                rm -f "$fichier"
                ((fichiers_supprimes++))
            done

            echo "  📁 Suppression des répertoires vides..."
            find "$repertoire" -type d -empty | while read -r dossier; do
                if [ "$dossier" != "$repertoire" ]; then
                    echo "    Suppression : $(basename "$dossier")"
                    rmdir "$dossier"
                fi
            done
            ;;
    esac

    echo "  ✅ Nettoyage terminé"
}

# Tests de suppression
echo "Tests de suppression :"

# Créer des fichiers de test
echo "Fichier temporaire" > test_structure/fichier.tmp
touch test_structure/fichier_vide.txt
echo "Fichier normal" > test_structure/fichier_normal.txt

echo ""
echo "1. Suppression avec corbeille :"
# supprimer_securise "test_structure/fichier.tmp" "corbeille"

echo ""
echo "2. Nettoyage des fichiers temporaires :"
nettoyer_repertoire "test_structure" "temp"

echo ""
echo "3. Nettoyage des fichiers vides :"
nettoyer_repertoire "test_structure" "vide"
```

### Utilitaires de gestion de fichiers

```bash
#!/bin/bash

echo "=== Utilitaires de gestion de fichiers ==="

# Fonction de synchronisation simple
synchroniser_repertoires() {
    local source="$1"
    local destination="$2"
    local mode="${3:-normal}"  # normal, miroir

    echo "Synchronisation de '$source' vers '$destination' (mode: $mode)"

    # Vérifications
    if [ ! -d "$source" ]; then
        echo "  ❌ Répertoire source inexistant"
        return 1
    fi

    if [ ! -d "$destination" ]; then
        echo "  📁 Création du répertoire de destination"
        mkdir -p "$destination"
    fi

    local fichiers_copies=0
    local fichiers_mis_a_jour=0
    local fichiers_supprimes=0

    # Copier/mettre à jour les fichiers
    find "$source" -type f | while read -r fichier_source; do
        local chemin_relatif="${fichier_source#$source/}"
        local fichier_dest="$destination/$chemin_relatif"
        local dir_dest=$(dirname "$fichier_dest")

        # Créer le répertoire de destination si nécessaire
        if [ ! -d "$dir_dest" ]; then
            mkdir -p "$dir_dest"
        fi

        # Vérifier si le fichier doit être copié/mis à jour
        local doit_copier=false

        if [ ! -f "$fichier_dest" ]; then
            echo "    📄 Nouveau : $chemin_relatif"
            doit_copier=true
            ((fichiers_copies++))
        elif [ "$fichier_source" -nt "$fichier_dest" ]; then
            echo "    🔄 Mise à jour : $chemin_relatif"
            doit_copier=true
            ((fichiers_mis_a_jour++))
        fi

        if [ "$doit_copier" = true ]; then
            cp "$fichier_source" "$fichier_dest"
        fi
    done

    # Mode miroir : supprimer les fichiers qui n'existent plus dans la source
    if [ "$mode" = "miroir" ]; then
        find "$destination" -type f | while read -r fichier_dest; do
            local chemin_relatif="${fichier_dest#$destination/}"
            local fichier_source="$source/$chemin_relatif"

            if [ ! -f "$fichier_source" ]; then
                echo "    🗑️  Suppression : $chemin_relatif"
                rm -f "$fichier_dest"
                ((fichiers_supprimes++))
            fi
        done
    fi

    echo "  ✅ Synchronisation terminée"
    echo "    Nouveaux fichiers : $fichiers_copies"
    echo "    Fichiers mis à jour : $fichiers_mis_a_jour"
    if [ "$mode" = "miroir" ]; then
        echo "    Fichiers supprimés : $fichiers_supprimes"
    fi
}

# Fonction de rapport sur l'espace disque
rapport_espace_disque() {
    local repertoire="${1:-.}"

    echo "Rapport d'espace disque pour '$repertoire' :"

    if [ ! -d "$repertoire" ]; then
        echo "  ❌ Répertoire inexistant"
        return 1
    fi

    # Taille totale
    local taille_totale=$(du -sb "$repertoire" 2>/dev/null | cut -f1)
    echo "  📊 Taille totale : $(numfmt --to=iec --suffix=B $taille_totale)"

    # Top 10 des plus gros fichiers
    echo ""
    echo "  📄 Top 10 des plus gros fichiers :"
    find "$repertoire" -type f -exec stat -c "%s %n" {} \; 2>/dev/null | \
        sort -nr | head -10 | while read taille fichier; do
        local taille_lisible=$(numfmt --to=iec --suffix=B $taille)
        local nom_relatif="${fichier#$repertoire/}"
        echo "    $taille_lisible - $nom_relatif"
    done

    # Répartition par extension
    echo ""
    echo "  📊 Répartition par extension :"
    find "$repertoire" -type f | sed 's/.*\.//' | sort | uniq -c | sort -nr | head -5 | \
        while read count ext; do
            echo "    .$ext : $count fichier(s)"
        done
}

# Fonction de création de structure de projet
creer_structure_projet() {
    local nom_projet="$1"
    local type_projet="${2:-standard}"  # standard, web, script

    echo "Création de la structure de projet '$nom_projet' (type: $type_projet)"

    if [ -e "$nom_projet" ]; then
        echo "  ❌ Le projet existe déjà"
        return 1
    fi

    # Structure de base
    mkdir -p "$nom_projet"

    case "$type_projet" in
        "standard")
            mkdir -p "$nom_projet"/{src,docs,tests,config}
            touch "$nom_projet/README.md"
            touch "$nom_projet/.gitignore"
            echo "# $nom_projet

## Description
Description du projet ici.

## Installation
Instructions d'installation.

## Usage
Instructions d'utilisation." > "$nom_projet/README.md"
            ;;
        "web")
            mkdir -p "$nom_projet"/{src/{css,js,images},docs,tests}
            touch "$nom_projet/src/index.html"
            touch "$nom_projet/src/css/style.css"
            touch "$nom_projet/src/js/script.js"
            echo "<!DOCTYPE html>
<html>
<head>
    <title>$nom_projet</title>
    <link rel=\"stylesheet\" href=\"css/style.css\">
</head>
<body>
    <h1>$nom_projet</h1>
    <script src=\"js/script.js\"></script>
</body>
</html>" > "$nom_projet/src/index.html"
            ;;
        "script")
            mkdir -p "$nom_projet"/{scripts,config,logs,docs}
            touch "$nom_projet/scripts/main.sh"
            chmod +x "$nom_projet/scripts/main.sh"
            echo "#!/bin/bash

# Script principal pour $nom_projet

echo \"Démarrage de $nom_projet\"" > "$nom_projet/scripts/main.sh"
            ;;
    esac

    echo "  ✅ Structure créée avec succès"
    echo "  📁 Contenu :"
    tree "$nom_projet" 2>/dev/null || find "$nom_projet" -type f | sort
}

# Tests des utilitaires
echo "Tests des utilitaires :"

# Créer un autre répertoire pour tester la synchronisation
mkdir -p test_sync_dest

echo ""
echo "1. Synchronisation :"
synchroniser_repertoires "test_structure/docs" "test_sync_dest" "normal"

echo ""
echo "2. Rapport d'espace disque :"
rapport_espace_disque "test_structure"

echo ""
echo "3. Création de structure de projet :"
creer_structure_projet "mon_nouveau_projet" "web"

# Nettoyer les structures de test
rm -rf test_structure test_sync_dest mon_nouveau_projet 2>/dev/null
```

## Récapitulatif

✅ **Ce que vous avez appris dans ce chapitre :**

1. **Tests sur les fichiers** : Vérification d'existence, permissions, contenu, comparaisons
2. **Parcours de répertoires** : Méthodes simples et récursives, avec actions conditionnelles
3. **Manipulation de chemins** : Construction, normalisation, validation de chemins
4. **Opérations courantes** : Copie, déplacement, suppression sécurisés avec vérifications

✅ **Points clés à retenir :**

**Tests de fichiers :**
```bash
[ -f fichier ]      # Est un fichier
[ -d repertoire ]   # Est un répertoire
[ -e element ]      # Existe
[ -r fichier ]      # Lisible
[ -w fichier ]      # Modifiable
[ -x fichier ]      # Exécutable
[ -s fichier ]      # Non vide
```

**Manipulation de chemins :**
```bash
dirname "/path/to/file"     # /path/to
basename "/path/to/file"    # file
realpath "path"             # Chemin absolu normalisé
```

**Opérations sécurisées :**
```bash
# Toujours vérifier avant d'agir
if [ -f "$source" ] && [ -d "$(dirname "$dest")" ]; then
    cp "$source" "$dest"
fi
```

**Parcours récursif :**
```bash
find repertoire -type f | while read fichier; do
    # traiter $fichier
done
```

✅ **Bonnes pratiques :**
- Toujours vérifier l'existence avant manipulation
- Créer des sauvegardes avant suppression
- Valider les chemins pour éviter les erreurs
- Utiliser des noms de fichiers sécurisés
- Gérer les erreurs et informer l'utilisateur
- Limiter les droits nécessaires

✅ **Prochaines étapes :**
Dans le chapitre 9, nous découvrirons le traitement de texte avancé avec grep, sed, et awk !

---

💡 **Conseil pratique :** La gestion de fichiers est cruciale en administration système. Pratiquez ces fonctions sur des données de test avant de les utiliser sur des fichiers importants. Toujours faire des sauvegardes !

⏭️
