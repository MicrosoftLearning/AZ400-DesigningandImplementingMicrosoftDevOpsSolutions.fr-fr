---
lab:
  title: Gestion des packages avec Azure Artifacts
  module: 'Module 07: Design and implement a dependency management strategy'
---

# Gestion des packages avec Azure Artifacts

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans l’article [Prérequis du labo AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- **Configurez l’exemple de projet eShopOnWeb :** Si vous n’avez pas encore l’exemple de projet eShopOnWeb que vous pouvez utiliser pour ce labo, créez-en un en suivant les instructions disponibles dans [Prérequis pour le lab AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- Visual Studio 2022 Community Edition, disponible sur la [page des téléchargements de Visual Studio](https://visualstudio.microsoft.com/downloads/). L’installation de Visual Studio 2022 doit inclure les charges de travail **Développement ASP<nolink>.NET et web**, **Développement Azure** et **Développement multiplateforme .NET Core**.

- **SDK .NET Core :** [Télécharger et installer le SDK .NET Core (2.1.400+)](https://go.microsoft.com/fwlink/?linkid=2103972)

- **Fournisseur d’informations d’identification Azure Artifacts :** [Téléchargez et installez le fournisseur d’informations d’identification](https://go.microsoft.com/fwlink/?linkid=2099625).

## Présentation du labo

Azure Artifacts facilite la découverte, l’installation et la publication des packages NuGet, npm et Maven dans Azure DevOps. Il est étroitement intégré à d’autres fonctionnalités Azure DevOps telles que Build, ce qui permet à la gestion des packages d’être une partie transparente de vos workflows existants.

## Objectifs

À la fin de ce labo, vous serez en mesure d’effectuer les tâches suivantes :

- Créer et se connecter à un flux.
- Créer et publier un package NuGet.
- Importer un package NuGet.
- Mettre à jour un package NuGet.

## Durée estimée : 40 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

Dans cet exercice, vous allez configurer les prérequis du labo.

#### Tâche 1 : (passer si terminée) créer et configurer le projet d’équipe

Dans cette tâche, vous allez créer un projet Azure DevOps **eShopOnWeb** à utiliser par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps. Cliquez sur **Nouveau projet**. Attribuez au projet le nom **eShopOnWeb** et conservez les valeurs par défaut des autres champs. Cliquez sur **Créer**.

    ![Capture d’écran du volet de création d’un projet.](images/create-project.png)

#### Tâche 2 : (passer si terminée) importer le référentiel Git eShopOnWeb

Dans cette tâche, vous allez importer le référentiel Git eShopOnWeb qui sera utilisé par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps et le projet **eShopOnWeb** créé précédemment. Cliquez sur **Repos > Fichiers**, **Importer un référentiel**. Cliquez sur **Importer**. Dans la fenêtre **Importer un dépôt Git**, collez l’URL <https://github.com/MicrosoftLearning/eShopOnWeb.git>, puis cliquez sur **Importer** :

    ![Capture d’écran du volet Importer un référentiel.](images/import-repo.png)

1. Le référentiel est organisé de la manière suivante :
    - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
    - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
    - Le dossier **infra** contient l’infrastructure Bicep&ARM sous forme de modèles de code utilisés dans certains scénarios de labo.
    - **Définitions de workflow GitHub YAML du conteneur de dossiers .github**.
    - Le dossier **src** contient le site web .NET 8 utilisé dans les scénarios de labo.

#### Tâche 3 : (à ignorer si vous l’avez déjà effectuée) définir la branche principale en tant que branche par défaut

1. Accédez à **Repos > Branches**.
1. Pointez sur la branche **principale**, puis cliquez sur les points de suspension à droite de la colonne.
1. Cliquez sur **Définir comme branche par défaut**.

#### Tâche 4 : configurer la solution eShopOnWeb dans Visual Studio

Dans cette tâche, vous allez configurer Visual Studio pour préparer le labo.

1. Vérifiez que vous affichez le projet d’équipe **eShopOnWeb** sur le portail Azure DevOps.

    > **Remarque** : Vous pouvez accéder directement à la page du projet en accédant à l’URL [https://dev.azure.com/`<your-Azure-DevOps-account-name>`/eShopOnWeb](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/eShopOnWeb), où l’espace réservé `<your-Azure-DevOps-account-name>` représente le nom de votre organisation Azure DevOps.

1. Dans le menu vertical situé à gauche du volet **eShopOnWeb**, cliquez sur **Repos**.
1. Dans le volet **Fichiers**, cliquez sur **Cloner**, sélectionnez la flèche déroulante en regard de **Cloner dans VS Code**, puis, dans le menu déroulant, sélectionnez **Visual Studio**.
1. Si vous êtes invité à continuer, cliquez sur **Ouvrir**.
1. Si vous y êtes invité, connectez-vous avec le compte d’utilisateur que vous avez utilisé pour configurer votre organisation Azure DevOps.
1. Dans l’interface Visual Studio, dans la fenêtre contextuelle **Azure DevOps**, acceptez le chemin local par défaut (C:\eShopOnWeb), puis cliquez sur **Cloner**. Cette opération importe automatiquement le projet dans Visual Studio.
1. Laissez la fenêtre Visual Studio ouverte, car vous allez l’utiliser dans ce labo.

### Exercice 1 : utiliser Azure Artifacts

Dans cet exercice, vous allez apprendre à utiliser Azure Artifacts en effectuant les étapes suivantes :

- Créer et se connecter à un flux.
- Créer et publier un package NuGet.
- Importer un package NuGet.
- Mettre à jour un package NuGet.

#### Tâche 1 : créer un flux et s’y connecter

Dans cette tâche, vous allez créer un flux et vous y connecter.

1. Dans la fenêtre du navigateur web affichant les paramètres de votre projet dans le portail Azure DevOps, dans le volet de navigation vertical, sélectionnez **Artefacts**.
1. Dans le hub **Artifacts** affiché, cliquez sur **+ Créer un flux** en haut du volet.

    > **Remarque** : ce flux est une collection de packages NuGet disponibles pour les utilisateurs de l’organisation et s’ajoutera au flux NuGet public en tant qu’homologue. Le scénario de ce labo se concentre sur le flux de travail pour l’utilisation d’Azure Artifacts, de sorte que les décisions réelles prises en matière d’architecture et de développement sont purement utilisées à titre d’illustrations.  Ce flux inclut des fonctionnalités courantes qui peuvent être partagées entre les projets de cette organisation.

1. Dans le volet **Créer un flux**, dans la zone de texte **Nom**, tapez **`eShopOnWebShared`**, dans la section **Étendue**, sélectionnez l’option **Organisation**, laissez les autres paramètres avec leurs valeurs par défaut, puis cliquez sur **Créer**.

    > **Remarque** : les utilisateurs qui souhaitent se connecter à ce flux NuGet doivent configurer leur environnement.

1. Dans le hub **Artifacts**, cliquez sur **Connectez-vous au flux**.
1. Dans le volet **Connectez-vous au flux**, dans la section **NuGet**, sélectionnez **Visual Studio**, puis, dans le volet **Visual Studio**, copiez l’URL **source**. `https://pkgs.dev.azure.com/Azure-DevOps-Org-Name/_packaging/eShopOnWebShared/nuget/v3/index.json`
1. Revenez à la fenêtre **Visual Studio**.
1. Dans la fenêtre Visual Studio, cliquez sur l’en-tête de menu **Outils**. Dans le menu déroulant, sélectionnez **Gestionnaire de package NuGet**, puis, dans le menu en cascade, sélectionnez **Paramètres du Gestionnaire de package**.
1. Dans la boîte de dialogue **Options**, cliquez sur **Sources de package**, puis cliquez sur le signe plus pour ajouter une nouvelle source de package.
1. En bas de la boîte de dialogue, dans la zone de texte **Nom**, remplacez **Package source** par **eShopOnWebShared** et, dans la zone de texte **Source**, collez l’URL que vous avez copiée sur le portail Azure DevOps.
1. Cliquez sur **Mettre à jour**, puis sur **OK** pour finaliser l’ajout.

    > **Remarque** : Visual Studio est à présent connecté au nouveau flux.

#### Tâche 2 : créer et publier un package NuGet développé en interne

Dans cette tâche, vous allez créer et publier un package NuGet personnalisé développé en interne.

1. Dans la fenêtre Visual Studio que vous avez utilisée pour configurer la nouvelle source de package, dans le menu principal, cliquez sur **Fichier**. Dans le menu déroulant, cliquez sur **Nouveau**, puis, dans le menu en cascade, cliquez sur **Projet**.

    > **Remarque** : nous allons à présent créer un assembly partagé qui sera publié en tant que package NuGet afin que d’autres équipes puissent l’intégrer et rester à jour sans avoir à travailler directement avec la source du projet.

1. Sur la page **Modèles de projet récents** du volet **Créer un projet**, utilisez la zone de texte de recherche pour localiser le modèle **Bibliothèque de classes**, sélectionnez le modèle pour C#, puis cliquez sur **Suivant**.
1. Sur la page **Bibliothèque de classes** du volet **Créer un projet**, spécifiez les paramètres suivants, puis cliquez sur **Créer** :

    | Paramètre | Valeur |
    | --- | --- |
    | Nom du projet | **eShopOnWeb.Shared** |
    | Emplacement | accepter la valeur par défaut |
    | Solution | **Créer une solution** |
    | Nom de la solution | **eShopOnWeb.Shared** |

    Cochez la case pour **Placer la solution et le projet dans le même répertoire**.

1. Sélectionnez Suivant. Acceptez **.NET 8** en tant qu’option Framework.
1. Confirmez la création du projet en cliquant sur le bouton **Créer** .
1. Dans l’interface Visual Studio, dans le volet **Explorateur de solutions**, cliquez avec le bouton droit sur **Class1.cs**. Dans le menu contextuel, sélectionnez **Supprimer**, puis, lorsque vous êtes invité à confirmer la suppression, cliquez sur **OK**.
1. Appuyez sur **Ctrl + Maj + B** ou **cliquez avec le bouton droit sur le projet EShopOnWeb.Shared**, puis sélectionnez **Générer** pour générer le projet.
1. Sur votre station de travail de labo, ouvrez le menu Démarrer et recherchez **Windows PowerShell**. Ensuite, dans le menu en cascade, cliquez sur **Ouvrir Windows PowerShell en tant qu’administrateur**.
1. Dans la fenêtre **Administrateur : Windows PowerShell**, accédez au dossier eShopOnWeb.Shared en exécutant la commande suivante :

    ```text
    cd c:\eShopOnWeb\eShopOnWeb.Shared
    ```

    > **Remarque** : Le dossier **eShopOnWeb.Shared** est l’emplacement du fichier **eShopOnWeb.Shared.csproj**. Si vous avez choisi un autre emplacement, accédez à cet emplacement à la place.

1. Exécutez ce qui suit pour créer un fichier **.nupkg** à partir du projet.

    ```powershell
    dotnet pack .\eShopOnWeb.Shared.csproj
    ```

    > **Remarque** : La commande **dotnet pack** génère le projet et crée un package NuGet dans le dossier **bin\Release**. Si vous n’avez pas de dossier **Release**, vous pouvez utiliser le dossier **Debug** à la place.

    > **Remarque** : ignorez les avertissements affichés dans la fenêtre **Administrateur : Windows PowerShell**.

    > **Remarque** : dotnet pack génère un package minimal en fonction des informations qu’il est en mesure d’identifier à partir du projet. Par exemple, notez que le nom est **eShopOnWeb.Shared.1.0.0.nupkg**. Ce numéro de version a été récupéré de l’assembly.

1. Dans la fenêtre PowerShell, exécutez la commande suivante pour ouvrir le dossier **bin\Release** :

    ```powershell
    cd .\bin\Release
    ```

1. Exécutez ce qui suit pour publier le package sur le flux **eShopOnWebShared** :

    > **Important** : Vous devez installer le fournisseur d’informations d’identification pour que votre système d’exploitation puisse s’authentifier auprès d’Azure DevOps. Vous trouverez les instructions d’installation dans [Fournisseur d’informations d’identification Azure Artifacts](https://go.microsoft.com/fwlink/?linkid=2099625). Vous pouvez l’installer en exécutant la commande suivante dans la fenêtre PowerShell : `iex "& { $(irm https://aka.ms/install-artifacts-credprovider.ps1) } -AddNetfx"`

    > **Remarque** : vous devez fournir une **clé API**, qui peut être n’importe quelle chaîne non vide. On utilise **az** ici. Lorsque vous y êtes invité, connectez-vous à votre organisation Azure DevOps.

    ```powershell
    dotnet nuget push --source "eShopOnWebShared" --api-key az eShopOnWeb.Shared.1.0.0.nupkg
    ```

    > **Remarque** : Si l’invite n’apparaît pas, vous pouvez ajouter le paramètre **--interactive** à la commande.

1. Attendez la confirmation de la réussite de l’opération de tirage (push) de package.
1. Basculez vers la fenêtre du navigateur web qui affiche le portail Azure DevOps, puis, dans le volet de navigation vertical, sélectionnez **Artefacts**.
1. Dans le volet hub **Artifacts**, cliquez sur la liste déroulante dans le coin supérieur gauche et, dans la liste des flux, sélectionnez l’entrée **eShopOnWebShared**.

    > **Remarque** : Le flux **eShopOnWebShared** doit inclure le package NuGet récemment publié.

1. Cliquez sur le package NuGet pour afficher les détails correspondants.

#### Tâche 3 : importer un package NuGet open source dans le flux de packages Azure DevOps

Outre le développement de vos propres packages, pourquoi ne pas utiliser la bibliothèque Open Source NuGet (<https://www.nuget.org>) DotNet Package ? Parmi les millions de packages disponibles, vous trouverez toujours quelque chose d’utile pour votre application.

Dans cette tâche, nous allons utiliser un exemple de package générique nommé « Newtonsoft.Json », mais vous pouvez utiliser la même approche pour les autres packages de la bibliothèque.

1. Dans la même fenêtre PowerShell, accédez au dossier **eShopOnWeb.Shared**, exécutez la commande **dotnet** suivante pour installer l’exemple de package :

    ```powershell
    dotnet add package Newtonsoft.Json
    ```

1. Vérifiez la sortie du processus d’installation. Il affiche les différents flux qu’il essaiera pour télécharger le package :

    ```powershell
    Feeds used:
      https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json
      https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/eShopOnWeb/_packaging/eShopOnWebShared/nuget/v3/index.json
    ```

1. Ensuite, vous trouverez une sortie supplémentaire concernant le processus d’installation proprement dit.

    ```powershell
    Determining projects to restore...
    Writing C:\Users\AppData\Local\Temp\tmpxnq5ql.tmp
    info : X.509 certificate chain validation will use the default trust store selected by .NET for code signing.
    info : X.509 certificate chain validation will use the default trust store selected by .NET for timestamping.
    info : Adding PackageReference for package 'Newtonsoft.Json' into project 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
    info :   GET https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json
    info :   OK https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json 124ms
    info : Restoring packages for c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj...
    info :   GET https://api.nuget.org/v3/vulnerabilities/index.json
    info :   OK https://api.nuget.org/v3/vulnerabilities/index.json 84ms
    info :   GET https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/vulnerability.base.json
    info :   GET https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/2024.02.17.11.23.35/vulnerability.update.json
    info :   OK https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/vulnerability.base.json 14ms
    info :   OK https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/2024.02.17.11.23.35/vulnerability.update.json 30ms
    info : Package 'Newtonsoft.Json' is compatible with all the specified frameworks in project 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
    info : PackageReference for package 'Newtonsoft.Json' version '13.0.3' added to file 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
    info : Writing assets file to disk. Path: c:\eShopOnWeb\eShopOnWeb.Shared\obj\project.assets.json
    log  : Restored c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj (in 294 ms).
    ```

1. Le package Newtonsoft.Json a été installé dans les packages en tant que **Newtonsoft.Json**. Dans l’**Explorateur de solutions** Visual Studio, accédez au projet **eShopOnWeb.Shared**, développez Dépendances et notez **Newtonsoft.Json** sous les packages. Cliquez sur la petite flèche à gauche des packages pour ouvrir le dossier et la liste des fichiers.

#### Tâche 4 : charger le package NuGet open source dans Azure Artifacts

Considérons que ce package est un package « approuvé » que notre équipe DevOps peut réutiliser. Nous allons le charger dans le flux de packages Azure Artifacts créé précédemment.

1. Dans Visual Studio, cliquez avec le bouton droit sur le nouveau package **Newtonsoft.Json**, puis sélectionnez **Ouvrir le dossier dans l’Explorateur de fichiers** dans le menu contextuel. Vous verrez le nouveau package **Newtonsoft.Json** avec l’extension **.nupkg**.
2. Copiez le chemin complet à partir de la barre d’adresses de la fenêtre de l’Explorateur de fichiers.
3. Dans la fenêtre PowerShell, exécutez la commande suivante en remplaçant le chemin d’accès par celui que vous avez copié :

    ```powershell
    dotnet nuget push --source "eShopOnWebShared" --api-key az C:\eShopOnWeb\eShopOnWeb.Shared\Newtonsoft.Json\newtonsoft.json.13.0.3.nupkg
    ```

    > **Remarque** : cette opération déclenche l’affichage d’un message d’erreur :

    ```text
    Response status code does not indicate success: 409 (Conflict - 'Newtonsoft.Json 1.3.0.17' cannot be published to the feed because it exists in at least one of the feed's upstream sources. Publishing this copy would prevent you from using 'Newtonsoft.Json 1.3.0.17' from 'NuGet Gallery'. For more information, see https://go.microsoft.com/fwlink/?linkid=864880 (DevOps Activity ID: AE08BE89-C2FA-4FF7-89B7-90805C88972C)).
    ```

Lorsque vous avez créé le flux de packages Azure DevOps Artifacts, par conception, il autoriser les **sources en amont**, telles que nuget.org dans l’exemple dotnet. Toutefois, rien n’empêche votre équipe DevOps de créer un flux de packages **« interne uniquement »**.

1. Accédez au portail Azure DevOps, accédez à **Artifacts**, puis sélectionnez le flux **eShopOnWebShared**.
1. Cliquez sur **Rechercher des sources en amont**.
1. Dans la fenêtre **Accéder à un package en amont**, sélectionnez **NuGet** comme type de package, puis entrez **`Newtonsoft.Json`** dans le champ de recherche.
1. Confirmez l’opération en appuyant sur le bouton **Rechercher**.
1. Cela renvoie la liste de tous les packages Newtonsoft.Json avec les différentes versions disponibles.
1. Cliquez sur la **flèche gauche** pour revenir au flux **eShopOnWebShared**.
1. Cliquez sur l’engrenage pour ouvrir **Paramètres de flux**. Sur la page Paramètres de flux, sélectionnez **Sources en amont**.
1. Notez les différentes gestionnaires de packages en amont pour les différents langages de développement. Sélectionnez **Galerie NuGet** dans la liste. Appuyez sur le bouton **Supprimer**, puis sur le bouton **Enregistrer**.

1. Avec ces modifications enregistrées, il sera possible de charger le package **Newtonsoft.Json** à l’aide de NuGet.exe à partir de la fenêtre PowerShell, en relançant la commande suivante :

    ```text
     dotnet nuget push --source "eShopOnWebShared" --api-key az C:\eShopOnWeb\eShopOnWeb.Shared\Newtonsoft.Json\newtonsoft.json.13.0.3.nupkg
    ```

    > **Remarque** : Cela doit maintenant entraîner un chargement réussi.

    ```text
    Pushing newtonsoft.json.13.0.3.nupkg to 'https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/_packaging/5faffb6c-018b-4452-a4d6-72c6bffe79db/nuget/v2/'...
    PUT https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/_packaging/5faffb6c-018b-4452-a4d6-72c6bffe79db/nuget/v2/
    Accepted https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/_packaging/5faffb6c-018b-4452-a4d6-72c6bffe79db/nuget/v2/ 3160ms
    Your package was pushed.
    ```

1. Dans le portail Azure DevOps, **actualisez** la page du flux de packages Artefacts. La liste des packages affiche à la fois le package personnalisé **eShopOnWeb.Shared**, ainsi que le package public **Newtonsoft.Json**.
1. Dans la solution Visual Studio **eShopOnWeb.Shared**, cliquez avec le bouton droit sur le projet **eShopOnWeb.Shared**, puis sélectionnez **Gérer les packages NuGet** dans le menu contextuel.
1. Dans la fenêtre Gestionnaire de package NuGet, vérifiez que **Package source** est défini sur **eShopOnWebShared**.
1. Cliquez sur **Parcourir**, puis attendez que la liste des packages NuGet soit chargée.
1. Cette liste affiche également le package personnalisé **eShopOnWeb.Shared** ainsi que le package public **Newtonsoft.Json**.

## Révision

Dans ce labo, vous avez appris à utiliser Azure Artifacts en effectuant les tâches suivantes :

- Création d’un flux et connexion à ce flux.
- Création et publication d’un package NuGet.
- Importation d’un package NuGet personnalisé.
- Importation d’un package NuGet public.
