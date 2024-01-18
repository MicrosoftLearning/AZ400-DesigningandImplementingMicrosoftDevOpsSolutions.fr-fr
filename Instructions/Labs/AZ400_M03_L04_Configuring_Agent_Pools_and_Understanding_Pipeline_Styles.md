---
lab:
  title: Configuration de pools d’agents et présentation des différents styles de pipeline
  module: 'Module 03: Implement CI with Azure Pipelines and GitHub Actions'
---

# Configuration de pools d’agents et présentation des différents styles de pipeline

# Manuel de labo pour l’étudiant

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Créer une organisation ou une collection de projets](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops).

- Page de téléchargement [Git pour Windows](https://gitforwindows.org/). L’application sera installée dans le cadre des prérequis de ce labo.

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

Dans cet exercice, vous allez configurer les prérequis du labo, notamment le projet d’équipe préconfiguré Parts Unlimited basé sur un modèle Azure DevOps Demo Generator.

#### Tâche 1 : configurer le projet d’équipe

Dans cette tâche, vous allez utiliser Azure DevOps Demo Generator pour générer un nouveau projet basé sur le modèle **PartsUnlimited**.

1. Sur votre ordinateur de labo, démarrez un navigateur web, puis accédez à [Générateur de démos Azure DevOps](https://azuredevopsdemogenerator.azurewebsites.net). Ce site utilitaire automatise le processus de création d’un projet Azure DevOps au sein de votre compte qui est prérempli avec le contenu (éléments de travail, référentiels, etc.) requis pour le labo.

    > **Remarque** : pour plus d’informations sur le site, rendez-vous sur <https://docs.microsoft.com/en-us/azure/devops/demo-gen>.

1. Cliquez sur **Connexion** et connectez-vous avec le compte Microsoft associé à votre abonnement Azure DevOps.
1. Si nécessaire, sur la page **Générateur de démos Azure DevOps**, cliquez sur **Accepter** pour accepter les demandes d’autorisation d’accès à votre abonnement Azure DevOps.
1. Dans la page **Créer un projet**, dans la zone de texte **Nouveau nom du projet**, tapez **Configuration des pools d’agents et présentation des styles de pipeline**. Dans la liste déroulante **Sélectionner une organisation**, sélectionnez votre organisation Azure DevOps, puis cliquez sur **Choisir un modèle**.
1. Dans la page **Choisir un modèle**, cliquez sur le modèle **PartsUnlimited**, puis sur **Sélectionner un modèle**.
1. Cliquez sur **Créer le projet**.

    > **Remarque** : attendez la fin du processus. Ce processus prend environ 2 minutes. Si le processus échoue, accédez à votre organisation DevOps, supprimez le projet, puis réessayez.

1. Sur la page **Créer un projet**, cliquez sur **Accéder au projet**.

### Exercice 1 : créer un pipeline Azure Pipelines basé sur YAML

Dans cet exercice, vous allez convertir un pipeline Azure Pipeline classique en un pipeline YAML.

#### Tâche 1 : créer un pipeline YAML Azure DevOps

Dans cette tâche, vous allez créer un pipeline YAML Azure DevOps basé sur un modèle.

1. À partir du navigateur web affichant le portail Azure DevOps avec le projet **Configuration des pools d’agents et présentation des styles de pipeline** ouvert, dans le volet de navigation vertical situé à gauche, cliquez sur **Pipelines**.
1. Dans l’onglet **Récent** du volet **Pipelines**, cliquez sur **Nouveau pipeline**.
1. Dans le volet **Où se trouve votre code ?**, cliquez sur **Azure Repos Git**.
1. Dans le volet **Sélectionner un référentiel**, cliquez sur **PartsUnlimited**.
1. Dans le volet **Passer en revue le fichier YAML de pipeline**, passez en revue l’exemple de pipeline, cliquez sur le caret pointant vers le bas en regard du bouton **Exécuter** et cliquez sur **Enregistrer**.

### Exercice 2 : gérer les pools d’agents Azure DevOps

Dans cet exercice, vous allez implémenter l’agent Azure DevOps auto-hébergé.

#### Tâche 1 : configurer un agent Azure DevOps auto-hébergé

Dans cette tâche, vous allez configurer la machine virtuelle LOD en tant qu’agent Azure DevOps auto-hébergé et l’utiliser pour exécuter un pipeline de build.

1. Dans la machine virtuelle Lab (machine virtuelle lab) ou votre propre ordinateur, lancez un navigateur web, accédez au [portail Azure DevOps](https://dev.azure.com) et connectez-vous à l’aide du compte Microsoft associé à votre organisation Azure DevOps.
1. Dans le portail Azure DevOps, dans le coin supérieur droit de la page Azure DevOps, cliquez sur l’icône **Paramètres utilisateur**. Selon que vous avez activé ou non les fonctionnalités en préversion, vous devriez voir un élément **Sécurité** ou **Jetons d’accès personnels** dans le menu. Si vous voyez **Sécurité**, cliquez dessus, puis sélectionnez **Jetons d’accès personnels**. Dans le volet **Jetons d’accès personnels**, cliquez sur **+ Nouveau jeton**.
1. Dans le volet **Créer un jeton d’accès personnel**, cliquez sur le lien **Afficher toutes les étendues**, spécifiez les paramètres suivants, puis cliquez sur **Créer** (conservez la valeur par défaut de tous les autres paramètres) :

    | Paramètre | Valeur |
    | --- | --- |
    | Nom | **Labo Configuration des pools d’agents et compréhension des styles de pipeline** |
    | Étendue (définie personnalisée) | **Pools d’agents** (afficher plus d’options d’étendue ci-dessous si nécessaire)|
    | Autorisations | **Lire et gérer** |

1. Dans le volet **Réussite**, copiez la valeur du jeton d’accès personnel dans le Presse-papiers.

    > **Remarque** : veillez à copier le jeton. Vous ne pourrez plus la récupérer après avoir fermé ce volet.

1. Dans le volet **Réussite**, cliquez sur **Fermer**.
1. Dans le volet **Jeton d’accès personnels** du portail Azure DevOps, cliquez sur le symbole **Azure DevOps** dans le coin supérieur gauche, puis sur l’étiquette **Paramètres de l’organisation** dans le coin inférieur gauche.
1. Sur le côté gauche du volet **Vue d’ensemble**, dans la section **Pipelines** du menu vertical, cliquez sur **Pools d’agents**.
1. Dans le volet **Pools d’agents**, dans le coin supérieur droit, cliquez sur **Ajouter un pool**.
1. Dans le volet **Ajouter un pool d’agents**, dans la liste déroulante **Type de pool**, sélectionnez **Auto-hébergé**. Dans la zone de texte **Nom**, tapez **az400m05l05a-pool**, puis cliquez sur **Créer**.
1. Dans le volet **Pools d’agents**, cliquez sur l’entrée représentant le pool **az400m05l05a-pool** nouvellement créé.
1. Sous l’onglet **Travaux** du volet **az400m05l05a-pool**, cliquez sur le bouton **Nouvel agent**.
1. Dans le volet **Obtenir l’agent**, vérifiez que les onglets **Windows** et **x64** sont sélectionnés, puis cliquez sur **Télécharger** pour télécharger l’archive ZIP contenant les fichiers binaires de l’agent afin de le télécharger dans le dossier **Téléchargements** local dans votre profil utilisateur.

    > **Remarque** : si vous recevez un message d’erreur à ce stade indiquant que les paramètres système actuels vous empêchent de télécharger le fichier, dans la fenêtre Internet Explorer, dans le coin supérieur droit, cliquez sur le symbole d’engrenage désignant l’en-tête de menu **Paramètres**. Dans le menu déroulant, sélectionnez **Options Internet**. Dans la boîte de dialogue **Options Internet**, cliquez sur **Avancé**. Sous l’onglet **Avancé**, cliquez sur **Réinitialiser**. Dans la boîte de dialogue **Réinitialiser les paramètres Internet Explorer**, cliquez à nouveau sur **Réinitialiser**, puis sur **Fermer**, puis réessayez le téléchargement.

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
    | Saisir le type d’authentification (appuyez sur Entrée pour PAT) | **Entrée** |
    | Saisir votre jeton d’accès personnel | Jeton d’accès que vous avez enregistré précédemment dans cette tâche. |
    | Saisir le pool d’agents (appuyez sur Entrée par défaut) | **az400m05l05a-pool** |
    | Saisir le nom de l’agent | **az104-06-vm0** |
    | Saisir le dossier de travail (appuyez sur Entrée pour _work) | **Entrée** |
    | **(Uniquement si l’option apparaît)** Saisir Décompresser pour les tâches de chaque étape. (appuyez sur Entrée pour N) | **AVERTISSEMENT** : appuyez uniquement sur **Entrée** si le message s’affiche.|
    | Souhaitez-vous saisir l’agent d’exécution comme service ? (Y/N) (appuyez sur Entrée pour N) | **Y** |
    | entrée pour activer SERVICE_SID_TYPE_UNRESTRICTED (Y/N) (appuyez sur Entrée pour N) | **Y** |
    | Entrez le compte d’utilisateur à utiliser pour le service (appuyez sur Entrée pour NT AUTHORITY\NETWORK SERVICE). | **Entrée** |
    | Indiquez si vous voulez empêcher le démarrage du service immédiatement après la fin de la configuration. (Y/N) (appuyez sur Entrée pour N) | **Entrée** |

    > **Remarque** : vous pouvez exécuter votre agent auto-hébergé en tant que service ou processus interactif. Vous pouvez commencer par le mode interactif, qui simplifie la vérification des fonctionnalités de l’agent. Pour une utilisation en production, vous devez envisager d’exécuter l’agent en tant que service ou en tant que processus interactif avec ouverture de session automatique activée, car les deux conservent leur état d’exécution et vérifient que l’agent démarre automatiquement si le système d’exploitation est redémarré.

1. Basculez vers la fenêtre du navigateur affichant le portail Azure DevOps et fermez le volet **Obtenir l’agent**.
1. De retour dans l’onglet **Agents** du volet **az400m05l05a-pool**, notez que l’agent nouvellement configuré est répertorié avec l’état **En ligne**.
1. Dans la fenêtre du navigateur web affichant le portail Azure DevOps, dans le coin supérieur gauche, cliquez sur l’étiquette **Azure DevOps**.
1. Dans la fenêtre du navigateur affichant la liste des projets, cliquez sur la vignette représentant votre projet **Configuration des pools d’agents et présentation des styles de pipeline**.
1. Dans le volet **Configuration des pools d’agents et présentation des styles de pipeline**, dans le volet de navigation vertical situé à gauche, dans la section **Pipelines**, cliquez sur **Pipelines**.
1. Dans l’onglet **Récent** du volet **Pipelines**, sélectionnez **PartsUnlimited**. Dans le volet **PartsUnlimited**, sélectionnez **Modifier**.
1. Dans le volet d’édition **PartsUnlimited**, dans le pipeline YAML existant, remplacez la ligne `vmImage: windows-2019` désignant le pool d’agents cibles par le contenu suivant, en désignant le pool d’agents auto-hébergé nouvellement créé :

    ```yaml
    name: az400m05l05a-pool
    demands:
    - agent.name -equals az400m05-vm0
    ```
    > **AVERTISSEMENT** : prenez garde aux copier/coller. Vérifiez que vous disposez de la même mise en retrait que ci-dessus.


1. Pour `Task: NugetToolInstaller@0`, cliquez sur **Paramètres (lien qui s’affiche au-dessus de la tâche de couleur grise)**, modifiez **Version de NuGet.exe à installer** > **4.0.0**, puis cliquez sur **Ajouter**.
1.  Dans le volet d’édition **PartsUnlimited**, dans le coin supérieur droit du volet, cliquez sur **Enregistrer**. Dans le volet **Enregistrer**, cliquez à nouveau sur **Enregistrer**. Le build basé sur le pipeline sera automatiquement déclenché.
1.  Dans le portail Azure DevOps, dans le volet de navigation vertical situé à gauche, cliquez sur **Pipelines** dans la section **Pipelines**.
1.  Dans l’onglet **Récent** du volet **Pipelines**, cliquez sur l’entrée **PartsUnlimited**. Dans l’onglet **Exécutions** du volet **PartsUnlimited**, sélectionnez l’exécution la plus récente. Dans le volet **Résumé** de l’exécution, faites défiler vers le bas. Dans la section **Travaux**, cliquez sur **Phase 1** et surveillez le travail jusqu’à sa fin.



## Révision

Dans ce labo, vous allez découvrir comment implémenter et utiliser des agents auto-hébergés avec des pipelines YAML.
