🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Tutoriel Bash - Chapitre 1 : Introduction au scripting Bash

## Qu'est-ce qu'un script Bash ?

### Définition simple
Un script Bash est un fichier texte contenant une série de commandes que l'ordinateur peut exécuter automatiquement, une après l'autre. C'est comme créer une recette de cuisine pour votre ordinateur : vous écrivez les étapes à suivre, et l'ordinateur les exécute dans l'ordre.

### Analogie pratique
Imaginez que vous devez chaque matin :
1. Ouvrir votre ordinateur
2. Vérifier vos emails
3. Sauvegarder vos fichiers importants
4. Nettoyer les fichiers temporaires

Au lieu de faire ces tâches manuellement chaque jour, vous pouvez écrire un script Bash qui les fera automatiquement pour vous !

### Bash vs Terminal
- **Terminal** : C'est l'interface où vous tapez des commandes une par une
- **Bash** : C'est le "langage" que comprend le terminal
- **Script Bash** : C'est un fichier contenant plusieurs commandes Bash

### Exemple concret
Voici à quoi ressemble un script Bash très simple :

```bash
#!/bin/bash
echo "Bonjour ! Je suis votre premier script Bash"
date
echo "Voici les fichiers dans le répertoire actuel :"
ls -l
```

## Pourquoi utiliser Bash pour automatiser ?

### 1. **Gain de temps énorme**
- Vous écrivez une fois, vous utilisez mille fois
- Pas besoin de retaper les mêmes commandes
- Exécution instantanée de tâches complexes

**Exemple pratique :** Au lieu de taper 20 commandes pour sauvegarder vos photos, vous créez un script qui fait tout en un clic.

### 2. **Réduction des erreurs**
- Les humains font des fautes de frappe
- Les scripts exécutent toujours les mêmes commandes exactement
- Moins de risque d'oublier une étape importante

### 3. **Disponibilité universelle**
- Bash est installé sur pratiquement tous les systèmes Linux et macOS
- Gratuit et open source
- Pas besoin d'installer des logiciels supplémentaires

### 4. **Simplicité d'apprentissage**
- Si vous savez utiliser le terminal, vous connaissez déjà les bases
- Syntaxe relativement simple comparée à d'autres langages
- Beaucoup d'exemples et de documentation disponibles

### 5. **Puissance et flexibilité**
- Peut automatiser quasiment n'importe quelle tâche système
- Excellent pour la gestion de fichiers, la surveillance système, les sauvegardes
- Peut interagir avec d'autres programmes et services

### Cas d'usage courants pour les débutants

**Administration système :**
- Sauvegarder automatiquement des fichiers importants
- Nettoyer les fichiers temporaires
- Surveiller l'espace disque disponible

**Développement :**
- Automatiser la compilation de programmes
- Déployer des applications sur des serveurs
- Tester automatiquement du code

**Tâches quotidiennes :**
- Organiser automatiquement des photos par date
- Télécharger et traiter des fichiers depuis Internet
- Générer des rapports automatiques

## Environnement et prérequis

### Systèmes compatibles

**✅ Nativement supportés :**
- **Linux** (toutes les distributions : Ubuntu, CentOS, Debian, etc.)
- **macOS** (Terminal intégré)
- **Unix** et dérivés

**🔧 Avec installation supplémentaire :**
- **Windows** :
  - Windows Subsystem for Linux (WSL) - recommandé
  - Git Bash
  - Cygwin
  - PowerShell (syntaxe différente mais concepts similaires)

### Vérification de votre environnement

Ouvrez votre terminal et tapez ces commandes pour vérifier que tout fonctionne :

```bash
# Vérifier que Bash est installé
bash --version

# Vérifier le shell actuel
echo $SHELL

# Tester une commande simple
echo "Hello World"
```

**Résultat attendu :** Vous devriez voir la version de Bash s'afficher et le message "Hello World".

### Outils recommandés pour débuter

**Éditeur de texte :**
- **nano** : Simple et intégré au système (parfait pour débuter)
- **vim** ou **emacs** : Plus avancés
- **VS Code** : Interface graphique avec coloration syntaxique
- **gedit** (Linux) ou **TextEdit** (macOS) : Éditeurs graphiques simples

**Pour vérifier la syntaxe :**
- **ShellCheck** : Outil en ligne gratuit (shellcheck.net) pour vérifier vos scripts

### Connaissances préalables recommandées

**Indispensable :**
- Savoir ouvrir un terminal
- Comprendre la notion de fichier et de répertoire
- Connaître quelques commandes de base : `ls`, `cd`, `pwd`

**Utile mais pas obligatoire :**
- Notions de base sur les systèmes de fichiers
- Concepts de permissions de fichiers
- Utilisation basique d'un éditeur de texte

### Test de préparation

Avant de continuer, assurez-vous de pouvoir faire ces actions :

1. **Ouvrir un terminal**
2. **Naviguer dans les dossiers :**
   ```bash
   pwd          # Afficher le répertoire actuel
   ls           # Lister les fichiers
   cd Documents # Changer de répertoire
   ```

3. **Créer et modifier un fichier :**
   ```bash
   touch mon_test.txt    # Créer un fichier vide
   nano mon_test.txt     # L'ouvrir avec nano
   ```

4. **Exécuter une commande simple :**
   ```bash
   echo "Je suis prêt pour Bash !"
   ```

### Prochaines étapes

Dans le chapitre suivant, nous apprendrons à créer votre premier script Bash fonctionnel ! Nous verrons :
- Comment créer un fichier script
- La ligne "shebang" mystérieuse
- Comment rendre un script exécutable
- Comment lancer votre premier script

**Conseil :** Gardez un terminal ouvert pendant que vous lisez ce tutoriel pour tester les exemples au fur et à mesure !

---

💡 **Astuce pour débutants :** N'hésitez pas à expérimenter ! Bash est très tolérant, et la plupart des erreurs ne peuvent pas endommager votre système. Le meilleur moyen d'apprendre est de pratiquer.

⏭️
