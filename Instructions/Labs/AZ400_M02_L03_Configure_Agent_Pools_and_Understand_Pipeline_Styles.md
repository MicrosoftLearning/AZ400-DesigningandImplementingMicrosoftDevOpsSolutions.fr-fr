---
lab:
  title: Configurer des pools d’agents et comprendre les styles de pipeline
  module: 'Module 02: Implement CI with Azure Pipelines and GitHub Actions'
---

# Configurer des pools d’agents et comprendre les styles de pipeline

## Manuel de labo de l’étudiant

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Créer une organisation ou une collection de projets](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Page de téléchargement de [Git pour Windows](https://gitforwindows.org/). L’application sera installée dans le cadre des prérequis de ce labo.

- [Visual Studio Code](https://code.visualstudio.com/). L’application sera installée dans le cadre des prérequis de ce labo.

## Présentation du labo

Les pipelines basés sur YAML vous permettent d’implémenter entièrement CI/CD en tant que code, où les définitions de pipeline résident dans le même dépôt que le code qui fait partie de votre projet Azure DevOps. Les pipelines YAML prennent en charge un large éventail de fonctionnalités qui font partie des pipelines classiques, par exemple, les demandes de tirage, les révisions de code, l’historique, les branches et les modèles.

Quel que soit le style de pipeline choisi, pour générer votre code ou déployer votre solution avec Azure Pipelines, vous avez besoin d’un agent. Un agent héberge des ressources de calcul qui exécutent un travail à la fois. Les travaux peuvent être exécutés directement sur la machine hôte de l’agent ou dans un conteneur. Vous avez la possibilité d’exécuter vos travaux avec des agents hébergés par Microsoft, qui sont gérés pour vous, ou vous pouvez implémenter un agent autohébergé que vous configurez et gérez vous-même.

Dans ce labo, vous allez découvrir comment implémenter et utiliser des agents auto-hébergés avec des pipelines YAML.

## Objectifs

À la fin de ce labo, vous serez en mesure d’effectuer les tâches suivantes :

- Implémentez des pipelines YAML.
- Implémentez des agents autohébergés.

## Durée estimée : 45 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

Dans cet exercice, vous allez configurer les prérequis pour le labo, qui se composent d’un nouveau projet Azure DevOps avec un référentiel basé sur [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tâche 1 : (passer si terminée) créer et configurer le projet d’équipe

Dans cette tâche, vous allez créer un projet Azure DevOps **eShopOnWeb** à utiliser par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps. Cliquez sur **Nouveau projet**. Attribuez au projet le nom **eShopOnWeb** et conservez les valeurs par défaut des autres champs. Cliquez sur **Créer**.

#### Tâche 2 : (passer si terminée) importer le référentiel Git eShopOnWeb

Dans cette tâche, vous allez importer le référentiel Git eShopOnWeb qui sera utilisé par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps et le projet **eShopOnWeb** créé précédemment. Cliquez sur **Dépôts > Fichiers**, **Importer un dépôt**. Cliquez sur **Importer**. Dans la fenêtre **Importer un référentiel Git**, collez l’URL <https://github.com/MicrosoftLearning/eShopOnWeb.git>, puis cliquez sur **Importer** :

1. Le référentiel est organisé de la manière suivante :
    - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
    - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
    - Le dossier **infra** contient l’infrastructure Bicep & ARM en tant que modèles de code utilisés dans certains scénarios de labo.
    - Le dossier **.github** contient les définitions de workflow GitHub YAML.
    - Le dossier **src** contient le site web .NET 8 utilisé dans les scénarios de labo.

### Exercice 1 : créer un Azure Pipelines basé sur YAML

Dans cet exercice, vous allez créer un pipeline de build de cycle de vie d’application à l’aide d’un modèle basé sur YAML.

#### Tâche 1 : créer un pipeline YAML Azure DevOps

Dans cette tâche, vous allez créer un pipeline YAML Azure DevOps basé sur un modèle.

1. Dans le navigateur web qui affiche le portail Azure DevOps avec le projet **eShopOnWeb** ouvert, dans le volet de navigation vertical situé à gauche, cliquez sur **Pipelines**.
1. Cliquez sur le bouton **Créer un pipeline** si aucun autre pipeline n’a encore été créé ou sur **Nouveau pipeline** pour créer un pipeline supplémentaire.

1. Dans le volet **Où se trouve votre code ?**, cliquez sur **Azure Repos Git**.
1. Dans le volet **Sélectionner un référentiel**, cliquez sur **eShopOnWeb**.
1. Dans le volet **Configurer votre pipeline**, cliquez sur **Fichier YAML Azure Pipelines existant**.
1. Dans le volet **Sélectionner un fichier YAML existant**, sélectionnez **principal** pour la branche et **/.ado/eshoponweb-ci-pr.yml** pour le chemin d’accès.
1. Cliquez sur **Continuer**.
1. Dans le volet **Passer en revue le fichier YAML de votre pipeline**, passez en revue l’exemple de pipeline. Il s’agit d’un pipeline de build d’application .NET plutôt direct, qui effectue les opérations suivantes :

   - Une étape unique : Build
   - Un seul travail : Build
   - 3 tâches dans le travail de build :
   - Dotnet Restore
   - Dotnet Build
   - Dotnet Publish

1. Dans le volet **Passer en revue le fichier YAML de votre pipeline**, cliquez sur le symbole caret pointant vers le bas en regard du bouton **Exécuter**, puis sur **Enregistrer**.

    > Remarque : nous créons seulement la définition de pipeline pour l’instant, sans l’exécuter. Vous allez d’abord configurer un pool d’agents Azure DevOps et exécuter le pipeline dans un exercice ultérieur. 

### Exercice 2 : gérer les pools d’agents Azure DevOps

Dans cet exercice, vous allez implémenter un agent Azure DevOps auto-hébergé.

#### Tâche 1 : configurer un agent Azure DevOps auto-hébergé

Dans cette tâche, vous allez configurer votre machine virtuelle de labo en tant qu’agent d’auto-hébergement Azure DevOps et l’utiliser pour exécuter un pipeline de build.

1. Dans la machine virtuelle Lab (machine virtuelle lab) ou votre propre ordinateur, lancez un navigateur web, accédez au [portail Azure DevOps](https://dev.azure.com) et connectez-vous à l’aide du compte Microsoft associé à votre organisation Azure DevOps.

  > **Remarque** : tous les logiciels requis nécessaires doivent être installés sur l’ordinateur virtuel de labo. Si vous effectuez l’installation sur votre propre ordinateur, vous devez installer les Kits de développement logiciel (SDK) .NET version 8 ou ultérieure nécessaires pour générer le projet de démonstration. Reportez-vous à la section [Télécharger .NET](https://dotnet.microsoft.com/download/dotnet) pour en savoir plus.

1. Dans le portail Azure DevOps, dans le coin supérieur droit de la page Azure DevOps, cliquez sur l’icône **Paramètres utilisateur**. Selon que vous avez activé ou non les fonctionnalités en préversion, vous devriez voir un élément **Sécurité** ou **Jetons d’accès personnels** dans le menu. Si vous voyez **Sécurité**, cliquez dessus, puis sélectionnez **Jetons d’accès personnels**. Dans le volet **Jetons d’accès personnels**, cliquez sur **+ Nouveau jeton**.
1. Dans le volet **Créer un jeton d’accès personnel**, cliquez sur le lien **Afficher toutes les étendues**, spécifiez les paramètres suivants, puis cliquez sur **Créer** (conservez la valeur par défaut de tous les autres paramètres) :

    | Paramètre | Valeur |
    | --- | --- |
    | Nom | **eShopOnWeb** |
    | Étendue (définie personnalisée) | **Pools d’agents** (afficher plus d’options d’étendue ci-dessous si nécessaire)|
    | Autorisations | **Lire et gérer** |

1. Dans le volet **Réussite**, copiez la valeur du jeton d’accès personnel dans le Presse-papiers.

    > **Remarque** : veillez à copier le jeton. Vous ne pourrez plus la récupérer après avoir fermé ce volet.

1. Dans le volet **Réussite**, cliquez sur **Fermer**.
1. Dans le volet **Jeton d’accès personnels** du portail Azure DevOps, cliquez sur le symbole **Azure DevOps** dans le coin supérieur gauche, puis sur l’étiquette **Paramètres de l’organisation** dans le coin inférieur gauche.
1. Sur le côté gauche du volet **Vue d’ensemble**, dans la section **Pipelines** du menu vertical, cliquez sur **Pools d’agents**.
1. Dans le volet **Pools d’agents**, dans le coin supérieur droit, cliquez sur **Ajouter un pool**.
1. Dans le volet **Ajouter un pool d’agents**, dans la liste déroulante **Type de pool**, sélectionnez **Auto-hébergé**. Dans la zone de texte **Nom**, tapez **az400m03l03a-pool**, puis cliquez sur **Créer**.
1. Dans le volet **Pools d’agents**, cliquez sur l’entrée représentant le pool **az400m03l03a-pool** nouvellement créé.
1. Dans l’onglet **Travaux** du volet **az400m03l03a-pool**, cliquez sur le bouton **Nouvel agent**.
1. Dans le volet **Obtenir l’agent**, vérifiez que les onglets **Windows** et **x64** sont sélectionnés, puis cliquez sur **Télécharger** pour télécharger l’archive ZIP contenant les fichiers binaires de l’agent afin de le télécharger dans le dossier **Téléchargements** local dans votre profil utilisateur.

    > **Remarque** : si un message d’erreur indiquant que les paramètres système actuels vous empêchent de télécharger le fichier s’affiche à ce stade, dans la fenêtre Navigateur, dans le coin supérieur droit, cliquez sur le symbole d’engrenage désignant l’en-tête du menu **Paramètres**. Dans le menu déroulant, sélectionnez **Options Internet**. Dans la boîte de dialogue **Options Internet**, cliquez sur **Avancé**. Dans l’onglet **Avancé**, cliquez sur **Réinitialiser**. Dans la boîte de dialogue **Réinitialiser les paramètres du navigateur**, cliquez à nouveau sur **Réinitialiser**, cliquez sur **Fermer**, puis réessayez de télécharger.

1. Démarrez Windows PowerShell en tant qu’administrateur et, dans la console **Administrateur : Windows PowerShell**, exécutez les lignes suivantes pour créer le répertoire **C:\\agent** et extraire le contenu de l’archive téléchargée.

    ```powershell
    cd \
    mkdir agent ; cd agent
    $TARGET = Get-ChildItem "$Home\Downloads\vsts-agent-win-x64-*.zip"
    Add-Type -AssemblyName System.IO.Compression.FileSystem
    [System.IO.Compression.ZipFile]::ExtractToDirectory($TARGET, "$PWD")
    ```

1. Dans la même console **Administrateur : Windows PowerShell**, exécutez ce qui suit pour configurer l’agent :

    ```powershell
    .\config.cmd
    ```

1. Lorsque vous y êtes invité, spécifiez les valeurs des paramètres suivants :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Saisir l’URL du serveur | URL de votre organisation Azure DevOps, au format ****<https://dev.azure.com/>`<organization_name>`, où `<organization_name>` représente le nom de votre organisation Azure DevOps. |
    | Saisir le type d’authentification (appuyez sur Entrée pour PAT) | **Saisissez** |
    | Saisir votre jeton d’accès personnel | Jeton d’accès que vous avez enregistré précédemment dans cette tâche. |
    | Saisir le pool d’agents (appuyez sur Entrée par défaut) | **az400m03l03a-pool** |
    | Saisir le nom de l’agent | **az104-06-vm0** |
    | Saisir le dossier de travail (appuyez sur Entrée pour _work) | **Saisissez** |
    | **(Uniquement si l’option apparaît)** Saisir Décompresser pour les tâches de chaque étape. (appuyez sur Entrée pour N) | **AVERTISSEMENT** : appuyez uniquement sur **Entrée** si le message s’affiche.|
    | Souhaitez-vous saisir l’agent d’exécution comme service ? (Y/N) (appuyez sur Entrée pour N) | **Y** |
    | entrée pour activer SERVICE_SID_TYPE_UNRESTRICTED (Y/N) (appuyez sur Entrée pour N) | **Y** |
    | Entrez le compte d’utilisateur à utiliser pour le service (appuyez sur Entrée pour NT AUTHORITY\NETWORK SERVICE). | **Saisissez** |
    | Indiquez si vous voulez empêcher le démarrage du service immédiatement après la fin de la configuration. (Y/N) (appuyez sur Entrée pour N) | **Saisissez** |

    > **Remarque** : vous pouvez exécuter votre agent auto-hébergé en tant que service ou processus interactif. Vous pouvez commencer par le mode interactif, qui simplifie la vérification des fonctionnalités de l’agent. Pour une utilisation en production, vous devez envisager d’exécuter l’agent en tant que service ou en tant que processus interactif avec ouverture de session automatique activée, car les deux conservent leur état d’exécution et vérifient que l’agent démarre automatiquement si le système d’exploitation est redémarré.

1. Basculez vers la fenêtre du navigateur affichant le portail Azure DevOps et fermez le volet **Obtenir l’agent**.
1. De retour sous l’onglet **Agents** du volet **az400m03l03a-pool**, notez que l’agent nouvellement configuré est répertorié avec l’état **En ligne**.
1. Dans la fenêtre du navigateur web affichant le portail Azure DevOps, dans le coin supérieur gauche, cliquez sur l’étiquette **Azure DevOps**.
1. Dans la liste des projets, cliquez sur la vignette représentant votre projet **eShopOnWeb**.
1. Dans le volet **eShopOnWeb**, dans le volet de navigation vertical situé à gauche, dans la section **Pipelines**, cliquez sur **Pipelines**.
1. Dans l’onglet **Récent** du volet **Pipelines**, sélectionnez **eShopOnWeb**, puis, dans le volet **eShopOnWeb**, sélectionnez **Modifier**.
1. Dans le volet de modification **eShopOnWeb**, dans le pipeline YAML existant, remplacez la ligne 13, qui indique `vmImage: ubuntu-latest` désignant le pool d’agents cible, par le contenu suivant, désignant le nouveau pool d’agents autohébergés :

    ```yaml
    name: az400m03l03a-pool
    demands:
    - Agent.Name -equals az400m03-vm0
    ```

    > **AVERTISSEMENT** : Soyez prudent avec le copier-coller, vérifiez que vous avez le même retrait que celui illustré ci-dessus.

    ![Syntaxe du pool YAML](images/m3/eshoponweb-ci-pr-pool_v1.png)

1. Dans le volet de modification **eShopOnWeb**, dans le coin supérieur droit du volet, cliquez sur **Enregistrer et exécuter**. Le build basé sur le pipeline sera automatiquement déclenché.
1. Dans le portail Azure DevOps, dans le volet de navigation vertical situé à gauche, cliquez sur **Pipelines** dans la section **Pipelines**. Selon la configuration de votre laboratoire, le pipeline peut vous demander des autorisations. Cliquez sur **Autoriser** pour autoriser l’exécution du pipeline. 
1. Dans l’onglet **Récent** du volet **Pipelines**, cliquez sur l’entrée **eShopOnWeb**. Dans l’onglet **Exécutions** du volet **eShopOnWeb**, sélectionnez l’exécution la plus récente. Dans le volet **Résumé** de l’exécution, faites défiler la page vers le bas, puis, dans la section **Travaux**, cliquez sur **Phase 1** et surveillez le travail jusqu’à sa fin.

### Exercice 3 : supprimer les ressources du labo

1. Arrêtez et supprimez le service d’agent en exécutant `.\config.cmd remove` dans l’invite de commandes.
1. Supprimez le pool d’agents.
1. Révoquez le jeton PAT.
1. Annulez les modifications apportées au fichier **eshoponweb-ci-pr.yml**. Pour cela, accédez au fichier à partir de Repos/.ado/eshoponweb-ci-pr.yml, sélectionnez **Modifier**, supprimez les lignes 13-15 (extrait du pool d’agents), puis rétablissez le paramètre d’origine `vmImage: ubuntu-latest`. (En effet, vous utiliserez le même exemple de fichier de pipeline dans un prochain exercice de labo.)

![Rétablir les paramètres vmImage du pool de pipelines](images/m3/eshoponweb-ci-pr-vmimage_v1.png)

## Révision

Dans ce labo, vous allez découvrir comment implémenter et utiliser des agents auto-hébergés avec des pipelines YAML.
