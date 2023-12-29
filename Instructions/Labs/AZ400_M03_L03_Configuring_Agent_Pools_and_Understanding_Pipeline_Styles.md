---
lab:
  title: Configurer les pools d’agents et comprendre les styles de pipelines
  module: 'Module 03: Implement CI with Azure Pipelines and GitHub Actions'
---

# Configurer les pools d’agents et comprendre les styles de pipelines

## Manuel de labo de l’étudiant

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Créer une organisation ou une collection de projets](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Page de téléchargement de [Git pour Windows](https://gitforwindows.org/). Cela sera installé dans le cadre des prérequis de ce labo.

- [Visual Studio Code](https://code.visualstudio.com/). Cela sera installé dans le cadre des prérequis de ce labo.

## Présentation du labo

Les pipelines basés sur YAML vous permettent d’implémenter entièrement CI/CD en tant que code, où les définitions de pipeline résident dans le même dépôt que le code qui fait partie de votre projet Azure DevOps. Les pipelines YAML prennent en charge un large éventail de fonctionnalités qui font partie des pipelines classiques, par exemple, les demandes de tirage, les révisions de code, l’historique, les branches et les modèles.

Quel que soit le style de pipeline choisi, pour générer votre code ou déployer votre solution avec Azure Pipelines, vous avez besoin d’un agent. Un agent héberge des ressources de calcul qui exécutent un travail à la fois. Les travaux peuvent être exécutés directement sur la machine hôte de l’agent ou dans un conteneur. Vous avez la possibilité d’exécuter vos travaux avec des agents hébergés par Microsoft, qui sont gérés pour vous, ou vous pouvez implémenter un agent autohébergé que vous configurez et gérez vous-même.

Dans ce labo, vous allez découvrir comment implémenter et utiliser des agents auto-hébergés avec des pipelines YAML.

## Objectifs

À la fin de ce labo, vous serez en mesure d’accomplir les tâches suivantes :

- Implémentez des pipelines YAML.
- Implémentez des agents autohébergés.

## Durée estimée : 45 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

Dans cet exercice, vous configurerez les prérequis du labo, qui se composent d’un nouveau projet Azure DevOps comprenant un référentiel basé sur [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tâche 1 : (à ignorer si vous l’avez déjà effectuée) créer et configurer le projet d’équipe

Dans le cadre de cette tâche, vous créerez un projet Azure DevOps **eShopOnWeb** qui sera utilisé dans plusieurs labos.

1. Sur votre ordinateur de labo, ouvrez votre organisation Azure DevOps dans votre navigateur. Cliquez sur **Nouveau projet**. Nommez votre projet **eShopOnWeb** et ne modifiez pas les autres champs par défaut. Cliquez sur **Créer**.

#### Tâche 2 : (à ignorer si vous l’avez déjà effectuée) importer un référentiel Git eShopOnWeb

Dans le cadre de cette tâche, vous importerez le référentiel Git eShopOnWeb qui sera utilisé dans plusieurs labos.

1. Sur votre ordinateur de labo, ouvrez votre organisation Azure DevOps dans votre navigateur ainsi que le projet **eShopOnWeb** que vous venez de créer. Cliquez sur **Repos > Fichiers**, puis sur **Importer un référentiel**. Cliquez sur **Importer**. Dans la fenêtre **Importer un référentiel Git**, collez l’URL https://github.com/MicrosoftLearning/eShopOnWeb.git, puis cliquez sur **Importer** :

2. Le référentiel est organisé de la manière suivante :
    - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
    - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
    - Le dossier **.azure** contient l’infrastructure Bicep et ARM en tant que modèles de code utilisés dans certains scénarios de labos.
    - Le dossier **.github** contient les définitions de workflow GitHub YAML.
    - Le dossier **src** contient le site web .NET 7 utilisé dans les scénarios de labos.

### Exercice 1 : créer un Azure Pipelines basé sur YAML

Dans le cadre de cet exercice, vous créerez un pipeline de build de cycle de vie de l’application à partir d’un modèle YAML.

#### Tâche 1 : créer un pipeline Azure DevOps YAML

Dans le cadre de cette tâche, vous créerez un pipeline Azure DevOps YAML à partir d’un modèle.

1. Dans le navigateur web qui affiche le Portail Azure DevOps où le projet **EShopOnWeb** est ouvert, cliquez sur **Pipelines** dans le volet de navigation vertical situé à gauche de l’écran.
2. Dans l’onglet **Récent** du volet **Pipelines**, cliquez sur **Nouveau pipeline**.
3. Dans le volet **Où se trouve votre code ?**, sélectionnez **Azure Repos Git**.
4. Dans le volet **Sélectionner un référentiel**, cliquez sur **EShopOnWeb**.
5. Dans le volet **Configurer votre pipeline**, sélectionnez **Fichier YAML Azure Pipelines existant**.
6. Dans **Sélectionner un fichier YAML existant**, sélectionnez **principale** pour la branche et **/.ado/eshoponweb-ci-pr.yml** pour le chemin d’accès.
7. Cliquez sur **Continuer**.
8. Passez en revue l’exemple de pipeline dans le volet **Examiner le YAML de votre pipeline**. Il s’agit d’un pipeline de build d’application .NET plutôt simple, qui effectue les opérations suivantes :
- Une étape unique : générer
- Un travail unique : générer
- 3 tâches dans le travail de génération :
- dotnet restore
- dotnet build
- dotnet publish

9. Dans le volet **Examiner le YAML de votre pipeline**, cliquez sur le caret pointant vers le bas en regard du bouton **Exécuter**, puis sur **Enregistrer**.

    > Remarque : vous créerez seulement la définition de pipeline, sans l’exécuter pour le moment. Vous commencerez par configurer un pool d’agents Azure DevOps, et vous exécuterez le pipeline dans un autre exercice. 

### Exercice 2 : gérer les pools d’agents Azure DevOps

Dans le cadre de cet exercice, vous mettrez en œuvre un agent Azure DevOps auto-hébergé.

#### Tâche 1 : configurer un agent Azure DevOps auto-hébergé

Dans le cadre de cette tâche, vous configurerez votre ordinateur virtuel de labo en tant qu’agent Azure DevOps auto-hébergé et l’utiliserez pour exécuter un pipeline de build.

1. Sur l’ordinateur virtuel de labo ou sur votre propre ordinateur, démarrez un navigateur web, accédez au [Portail Azure DevOps](https://dev.azure.com) et connectez-vous à l’aide du compte Microsoft associé à votre organisation Azure DevOps.

  > **Remarque** : tous les logiciels requis nécessaires doivent être installés sur l’ordinateur virtuel de labo. Si vous effectuez l’installation sur votre propre ordinateur, vous devez installer les kits SDK .NET 7.0.x ou ultérieurs nécessaires pour générer le projet de démo. Reportez-vous à la section [Télécharger .NET](https://dotnet.microsoft.com/download/dotnet) pour en savoir plus.

1. Dans le coin supérieur droit de la page Azure DevOps du Portail Azure DevOps, cliquez sur l’icône **Paramètres utilisateur**. Selon que vous avez activé ou non les fonctionnalités d’évaluation, l’élément **Sécurité** ou **Jetons d’accès personnel** apparaît dans le menu : si vous voyez l’option **Sécurité**, cliquez dessus, puis sélectionnez **Jetons d’accès personnel**. Dans le volet **Jetons d’accès personnel**, cliquez sur **+ Nouveau jeton**.
2. Dans le volet **Créer un jeton d’accès personnel**, cliquez sur le lien **Afficher toutes les étendues**. Spécifiez les paramètres suivants, puis cliquez sur **Créer** (ne modifiez pas les autres valeurs par défaut) :

    | Paramètre | Valeur |
    | --- | --- |
    | Nom | **EShopOnWeb** |
    | Étendue (définition personnalisée) | **Pools d’agents** (affichez davantage d’étendues ci-dessous si nécessaire)|
    | Autorisations | **Lire et gérer** |

3. Dans le volet **Opération réussie**, copiez la valeur du jeton d’accès personnel dans le presse-papiers.

    > **Remarque** : assurez-vous de copier le jeton. Vous ne pourrez plus le récupérer une fois ce volet fermé.

4. Dans le volet **Opération réussie**, cliquez sur **Fermer**.
5. Dans le volet **Jeton d’accès personnel** du Portail Azure DevOps, cliquez sur le symbole **Azure DevOps** dans le coin supérieur gauche, puis sélectionnez l’étiquette des **paramètres de l’organisation** dans le coin inférieur gauche.
6. Cliquez sur **Pools d’agents** dans la section **Pipelines** dans la partie gauche du volet **Vue d’ensemble**.
7. Cliquez sur **Ajouter un pool** dans le coin supérieur droit du volet **Pools d’agents**.
8. Dans la liste déroulante **Type de pool** du volet **Ajouter un pool d’agents**, sélectionnez **Auto-hébergé**. Saisissez **az400m03l03a-pool** dans la zone de texte **Nom**, puis cliquez sur **Créer**.
9.  Dans le volet **Pools d’agents**, cliquez sur l’entrée représentant le pool **az400m03l03a-pool** que vous venez de créer.
10. Dans l’onglet **Travaux** du volet **az400m03l03a-pool**, cliquez sur le bouton **Nouvel agent**.
11. Dans le volet **Obtenir l’agent**, vérifiez que les onglets **Windows** et **x64** sont sélectionnés, puis cliquez sur **Télécharger** pour télécharger l’archive zip contenant les fichiers binaires de l’agent afin de les télécharger dans le dossier local **Téléchargements** de votre profil utilisateur.

    > **Remarque** : si un message d’erreur apparaît à ce stade pour vous indiquer que les paramètres système actuels vous empêchent de télécharger le fichier, cliquez sur le symbole d’engrenage dans le coin supérieur droit de la fenêtre Parcourir qui désigne l’en-tête du menu **Paramètres**. Dans le menu déroulant qui s’affiche, sélectionnez **Options Internet**, puis, dans la boîte de dialogue **Options Internet**, cliquez sur **Avancé**. Dans l’onglet **Avancé**, cliquez sur **Réinitialiser**, puis, dans la boîte de dialogue **Réinitialiser les paramètres du navigateur**, sélectionnez une nouvelle fois l’option **Réinitialiser**. Cliquez enfin sur **Fermer**, puis relancez le téléchargement.

12. Démarrez Windows PowerShell en tant qu’administrateur. Dans la console **Administrateur : Windows PowerShell**, exécutez les lignes suivantes pour créer le répertoire **C:\\agent** et y extraire le contenu de l’archive téléchargée :

    ```powershell
    cd \
    mkdir agent ; cd agent
    $TARGET = Get-ChildItem "$Home\Downloads\vsts-agent-win-x64-*.zip"
    Add-Type -AssemblyName System.IO.Compression.FileSystem
    [System.IO.Compression.ZipFile]::ExtractToDirectory($TARGET, "$PWD")
    ```

14. Toujours dans la console **Administrateur : Windows PowerShell**, exécutez les commandes suivantes pour configurer l’agent :

    ```powershell
    .\config.cmd
    ```

15. Lorsque vous y êtes invité, spécifiez les valeurs des paramètres suivants :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Saisir l’URL du serveur | L’URL de votre organisation Azure DevOps au format ****<https://dev.azure.com/>`<organization_name>`, où `<organization_name>` représente le nom de votre organisation Azure DevOps. |
    | Saisir le type d’authentification (appuyer sur la touche Entrée pour PAT) | **Entrée** |
    | Saisir le jeton d’accès personnel | Le jeton d’accès que vous venez d’enregistrer dans le cadre de cette tâche. |
    | Saisir le pool d’agents (appuyer sur la touche Entrée pour renseigner les valeurs par défaut) | **az400m03l03a-pool** |
    | Saisir le nom de l’agent | **az400m03-vm0** |
    | Saisir le dossier de travail (appuyer sur la touche Entrée pour _work) | **Entrée** |
    | **(Uniquement si l’option apparaît)** Saisir Décompresser les fichiers pour les tâches de chaque étape. (appuyer sur la touche Entrée pour N) | **AVERTISSEMENT** : appuyez sur la touche **Entrée** seulement si le message s’affiche.|
    | Souhaitez-vous entrer l'agent d'exécution comme service ? (Y/N) (appuyer sur la touche Entrée pour N) | **Y** |
    | Saisir Activer SERVICE_SID_TYPE_UNRESTRICTED (Y/N) (appuyer sur la touche Entrée pour N) | **Y** |
    | Saisir Compte d’utilisateur à utiliser pour le service (appuyez sur la touche Entrée pour NT AUTHORITY\NETWORK SERVICE) | **Entrée** |
    | Souhaitez-vous préciser s'il faut empêcher le démarrage du service immédiatement après la fin de la configuration ? (Y/N) (appuyer sur la touche Entrée pour N) | **Entrée** |

    > **Remarque** : vous pouvez exécuter l’agent auto-hébergé en tant que service ou processus interactif. Il est recommandé de commencer par le mode interactif qui simplifie la vérification des fonctionnalités de l’agent. Pour une utilisation en production, il est conseillé d’exécuter l’agent en tant que service ou en tant que processus interactif avec l’ouverture de session automatique, puisque les deux options conservent leur état d’exécution et garantissent que l’agent démarre automatiquement si le système d’exploitation est redémarré.

16. Revenez à la fenêtre du navigateur qui affiche le Portail Azure DevOps et fermez le volet **Obtenir l’agent**.
17. Dans l’onglet **Agents** du volet **az400m03l03a-pool**, l’agent que vous venez de configurer a désormais l’état **En ligne**.
18. Dans le coin supérieur gauche de la fenêtre du navigateur web affichant le Portail Azure DevOps, cliquez sur l’étiquette **Azure DevOps**.
19. Dans la fenêtre du navigateur affichant la liste des projets, cliquez sur la vignette représentant votre projet **Configurer les pools d’agents et comprendre les styles de pipelines**.
20. Dans la section **Pipelines** du volet de navigation vertical à gauche du volet **EShopOnWeb**, cliquez sur **Pipelines**.
21. Dans l’onglet **Récent** du volet **Pipelines**, sélectionnez **EShopOnWeb**. Dans le volet **EShopOnWeb**, cliquez sur **Modifier**.
22. Dans le pipeline basé surYAML du volet de modification d’**EShopOnWeb**, remplacez la ligne 13 qui indique `vmImage: ubuntu-latest` pour désigner le pool d’agents cible par le contenu suivant afin de désigner le pool d’agents auto-hébergé que vous venez de créer :

    ```yaml
    name: az400m03l03a-pool
    demands:
    - Agent.Name -equals az400m03-vm0
    ```

    > **AVERTISSEMENT** : avant de copier/coller le texte, assurez-vous que vous disposez du même retrait qu’illustré ci-dessus.

23. Dans le coin supérieur droit du volet de modification d’**EShopOnWeb**, cliquez sur **Enregistrer**. Dans le volet **Enregistrer**, cliquez une nouvelle fois sur **Enregistrer**. Le build basé sur ce pipeline sera automatiquement déclenché.
24. Dans la section **Pipelines** du volet de navigation vertical à gauche du Portail Azure DevOps, cliquez sur **Pipelines**.
25. Dans l’onglet **Récent** du volet **Pipelines**, cliquez sur l’entrée **EShopOnWeb**. Dans l’onglet **Exécutions** du volet **EShopOnWeb**, sélectionnez la dernière exécution. Atteignez la section **Travaux** en bas de l’onglet **Résumé** de l’exécution, puis cliquez sur **Phase 1** et surveillez le travail jusqu’à son achèvement.

### Exercice 3 : supprimer les ressources utilisées dans le labo

1. Arrêtez et supprimez le service d’agent en exécutant `.\config.cmd remove` depuis l’invite de commandes.
2. Supprimez le pool d’agents.
3. Révoquez le jeton PAT.

## Révision

Dans ce labo, vous avez découvert comment mettre en œuvre et utiliser des agents auto-hébergés avec des pipelines YAML.
