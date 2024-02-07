---
lab:
  title: Gestion des packages avec Azure Artifacts
  module: 'Module 08: Design and implement a dependency management strategy'
---

# Gestion des packages avec Azure Artifacts

# Manuel de labo pour l’étudiant

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans l’article [Prérequis du labo AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- **Configurez l’exemple de projet EShopOnWeb :** si vous ne disposez pas encore de l’exemple de projet EShopOnWeb que vous pouvez utiliser pour ce labo, créez-en un en conformément aux instructions disponibles dans l’article [Prérequis du labo AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- Visual Studio 2022 Community Edition, disponible sur la [page des téléchargements de Visual Studio](https://visualstudio.microsoft.com/downloads/). L’installation de Visual Studio 2022 doit inclure les charges de travail **Développement ASP<nolink>.NET et web**, **Développement Azure** et **Développement multiplateforme .NET Core**.

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

Dans cet exercice, nous voulons faire un rappel sur la validation des prérequis du labo, notamment disposer d’une organisation Azure DevOps et créer le projet EShopOnWeb. Pour plus d’informations, consultez les instructions ci-dessus.

#### Tâche 1 : configurer la solution EShopOnWeb dans Visual Studio

Dans cette tâche, vous allez configurer Visual Studio pour préparer le labo.

1. Vérifiez que vous consultez le projet d’équipe **EShopOnWeb** sur le portail Azure DevOps.

    > **Remarque** : vous pouvez accéder directement à la page du projet à l’aide de l’URL [`<your-Azure-DevOps-account-name>`https://dev.azure.com//EShopOnWeb](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/EShopOnWeb), où l’espace réservé `<your-Azure-DevOps-account-name>` représente le nom de votre organisation Azure DevOps.

1. Dans le menu vertical situé à gauche du volet **EShopOnWeb**, cliquez sur **Dépôts**.
1. Dans le volet **Fichiers**, cliquez sur **Cloner**, sélectionnez la flèche déroulante en regard de **Cloner dans VS Code**, puis, dans le menu déroulant, sélectionnez **Visual Studio**.
1. Si vous êtes invité à continuer, cliquez sur **Ouvrir**.
1. Si vous y êtes invité, connectez-vous avec le compte d’utilisateur que vous avez utilisé pour configurer votre organisation Azure DevOps.
1. Dans l’interface Visual Studio, dans la fenêtre contextuelle **Azure DevOps**, acceptez le chemin local par défaut (C:\EShopOnWeb), puis cliquez sur **Cloner**. Cette opération importe automatiquement le projet dans Visual Studio.
1. Laissez la fenêtre Visual Studio ouverte, car vous allez l’utiliser dans ce labo.

### Exercice 1 : utiliser Azure Artifacts

Dans cet exercice, vous allez apprendre à utiliser Azure Artifacts en effectuant les étapes suivantes :

- Créer un flux et s’y connecter.
- Créer et publier un package NuGet.
- Importer un package NuGet.
- Mettre à jour un package NuGet.

#### Tâche 1 : créer un flux et s’y connecter

Dans cette tâche, vous allez créer un flux et vous y connecter.

1. Dans la fenêtre du navigateur web affichant les paramètres de votre projet dans le portail Azure DevOps, dans le volet de navigation vertical, sélectionnez **Artefacts**.
1. Dans le hub **Artifacts** affiché, cliquez sur **+ Créer un flux** en haut du volet.

    > **Remarque** : ce flux est une collection de packages NuGet disponibles pour les utilisateurs de l’organisation et s’ajoutera au flux NuGet public en tant qu’homologue. Le scénario de ce labo se concentre sur le flux de travail pour l’utilisation d’Azure Artifacts, de sorte que les décisions réelles prises en matière d’architecture et de développement sont purement utilisées à titre d’illustrations.  Ce flux inclut des fonctionnalités courantes qui peuvent être partagées entre les projets de cette organisation.

1. Dans le volet **Créer un flux**, dans la zone de texte **Nom**, tapez **EShopOnWebShared**, dans la section **Étendue**, sélectionnez l’option **Organisation**, conservez les valeurs par défaut des autres paramètres, puis cliquez sur **Créer**.

    > **Remarque** : les utilisateurs qui souhaitent se connecter à ce flux NuGet doivent configurer leur environnement.

1. Dans le hub **Artifacts**, cliquez sur **Connectez-vous au flux**.
1. Dans le volet **Connectez-vous au flux**, dans la section **NuGet**, sélectionnez **Visual Studio**, puis, dans le volet **Visual Studio**, copiez l’URL **source**. (https://pkgs.dev.azure.com/<Azure-DevOps-Org-Name>_packaging/EShopOnWebShared/nuget/v3/index.json)
1. Revenez à la fenêtre **Visual Studio**.
1. Dans la fenêtre Visual Studio, cliquez sur l’en-tête de menu **Outils**. Dans le menu déroulant, sélectionnez **Gestionnaire de package NuGet**, puis, dans le menu en cascade, sélectionnez **Paramètres du Gestionnaire de package**.
1. Dans la boîte de dialogue **Options**, cliquez sur **Sources de package**, puis cliquez sur le signe plus pour ajouter une nouvelle source de package.
1. En bas de la boîte de dialogue, dans la zone de texte **Nom**, remplacez **Source de package** par **EShopOnWebShared**, puis, dans la zone de texte **Source**, collez l’URL que vous avez copiée dans le portail Azure DevOps.
1. Cliquez sur **Mettre à jour**, puis sur **OK** pour finaliser l’ajout.

    > **Remarque** : Visual Studio est à présent connecté au nouveau flux.

#### Tâche 2 : créer et publier un package NuGet développé en interne

Dans cette tâche, vous allez créer et publier un package NuGet personnalisé développé en interne.

1. Dans la fenêtre Visual Studio que vous avez utilisée pour configurer la nouvelle source de package, dans le menu principal, cliquez sur **Fichier**. Dans le menu déroulant, cliquez sur **Nouveau**, puis, dans le menu en cascade, cliquez sur **Projet**.

    > **Remarque** : nous allons à présent créer un assembly partagé qui sera publié en tant que package NuGet afin que d’autres équipes puissent l’intégrer et rester à jour sans avoir à travailler directement avec la source du projet.

1. Sur la page **Modèles de projet récents** du volet **Créer un projet**, utilisez la zone de texte de recherche pour localiser le modèle **Bibliothèque de classes**, sélectionnez le modèle pour C#, puis cliquez sur **Suivant**.
1. Sur la page **Bibliothèque de classes** du volet **Créer un projet**, spécifiez les paramètres suivants, puis cliquez sur **Créer** :

    | Paramètre | Value |
    | --- | --- |
    | Nom du projet | **EShopOnWeb.Shared** |
    | Emplacement | accepter la valeur par défaut |
    | Solution | **Créer une solution** |
    | Nom de la solution | **EShopOnWeb.Shared** |
    
    Laissez le paramètre **Placer la solution et le projet dans le même répertoire** activé. 

1. Cliquez sur Suivant. Acceptez **.NET 6.0 (support à long terme)** comme option Framework.
1. Confirmez la création du projet en appuyant sur le bouton **Créer**.
    
1. Dans l’interface Visual Studio, dans le volet **Explorateur de solutions**, cliquez avec le bouton droit sur **Class1.cs**. Dans le menu contextuel, sélectionnez **Supprimer**, puis, lorsque vous êtes invité à confirmer la suppression, cliquez sur **OK**.
1. Appuyez sur **Ctrl + Maj + B** ou **cliquez avec le bouton droit sur le projet EShopOnWeb.Shared**, puis sélectionnez **Générer** pour générer le projet.

    > **Remarque** : dans la tâche suivante, nous allons utiliser **NuGet.exe** pour générer un package NuGet directement à partir du projet généré. Le projet doit toutefois être généré au préalable.

1. Basculez vers le navigateur web affichant le portail Azure DevOps.
1. Accédez au volet **Connectez-vous au flux**, puis, dans la section **NuGet**, sélectionnez **NuGet.exe**. Le volet **NuGet.exe** s’affiche alors.
1. Dans le volet **NuGet.exe**, cliquez sur **Obtenir les outils**.
1. Dans le volet **Obtenir les outils**, cliquez sur le lien **Télécharger la dernière version de NuGet**. Cette opération ouvre automatiquement un autre onglet de navigateur affichant la page des **versions de distribution NuGet disponibles**.
1. Sur la page des **versions de distribution NuGet disponibles**, sélectionnez **nuget.exe - recommended latest v6.x**, puis téléchargez l’exécutable dans le dossier local du **projet EShopOnWeb.Shared** (si vous avez conservé les emplacements de dossier par défaut, il doit s’agir de C:\EShopOnWeb\EShopOnWeb.Shared).
1. Sélectionnez le fichier **nuget.exe**, puis ouvrez ses propriétés en cliquant avec le bouton droit sur le fichier et en sélectionnant **Propriétés** dans le menu contextuel.
1. Dans la fenêtre contextuelle Propriétés, sous l’onglet **Général**,  sous la section Sécurité, sélectionnez **Débloquer**. Confirmez l’opération en appuyant sur **Appliquer**, puis sur **OK**.
1. Sur votre station de travail de labo, ouvrez le menu Démarrer et recherchez **Windows PowerShell**. Ensuite, dans le menu en cascade, cliquez sur **Ouvrir Windows PowerShell en tant qu’administrateur**.
1. Dans la fenêtre **Administrateur : Windows PowerShell**, accédez au dossier EShopOnWeb.Shared en exécutant la commande suivante : 

```
cd c:\EShopOnWeb\EShopOnWeb.Shared
```
Exécutez la commande suivante pour créer un fichier **.nupkg** à partir du projet.

    > **Note**: This is a shortcut to package the NuGet bits for deployment. NuGet is highly customizable. To learn more, refer to the [NuGet package creation page](https://docs.microsoft.com/en-us/nuget/create-packages/overview-and-workflow).

    ```
    .\nuget.exe pack ./EShopOnWeb.Shared.csproj
    ```

    > **Note**: Disregard any warnings displayed in the **Administrator: Windows PowerShell** window.

    > **Note**: NuGet builds a minimal package based on the information it is able to identify from the project. For example, note that the name is **EShopOnWeb.Shared.1.0.0.nupkg**. That version number was retrieved from the assembly.

1. Une fois le package créé, exécutez la commande suivante pour publier le package dans le flux **EShopOnWebShared** :

    > **Remarque** : vous devez fournir une **clé API**, qui peut être n’importe quelle chaîne non vide. Nous utilisons ici **AzDO**. Lorsque vous y êtes invité, connectez-vous à votre organisation Azure DevOps.

    ```
    .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO EShopOnWeb.Shared.1.0.0.nupkg
    ```
1. Attendez la confirmation de la réussite de l’opération de tirage (push) de package.
1. Basculez vers la fenêtre du navigateur web qui affiche le portail Azure DevOps, puis, dans le volet de navigation vertical, sélectionnez **Artefacts**.
1. Dans le volet du hub **Artefacts**, cliquez sur la liste déroulante dans le coin supérieur gauche, puis, dans la liste des flux, sélectionnez l’entrée **EShopOnWebShared**.

    > **Remarque** : le flux **EShopOnWebShared** doit inclure le nouveau package NuGet publié.

1. Cliquez sur le package NuGet pour afficher les détails correspondants.

#### Tâche 3 : importer un package NuGet open source dans le flux de packages Azure DevOps

Outre le développement de vos propres packages, vous pouvez envisager d’utiliser la bibliothèque de packages DotNet open source (https://www.nuget.org) de NuGet. Parmi les millions de packages disponibles, vous trouverez toujours quelque chose d’utile pour votre application.

Dans cette tâche, nous allons utiliser un exemple de package générique « Hello World », mais vous pouvez utiliser la même approche pour les autres packages de la bibliothèque.

1. Dans la même fenêtre PowerShell, exécutez la commande **nuget** suivante pour installer l’exemple de package :

```
.\nuget install HelloWorld -ExcludeVersion
```

1. Vérifiez la sortie du processus d’installation. La première ligne affiche les différents flux utilisés pour télécharger le package :
```
Feeds used:
  https://api.nuget.org/v3/index.json
  https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/eShopOnWeb/_packaging/EShopOnWebPFeed/nuget/v3/index.json
```
1. Ensuite, vous trouverez une sortie supplémentaire concernant le processus d’installation proprement dit.

```
Installing package 'Helloworld' to 'C:\eShopOnWeb\EShopOnWeb.Shared'.
  GET https://api.nuget.org/v3/registration5-gz-semver2/helloworld/index.json
  OK https://api.nuget.org/v3/registration5-gz-semver2/helloworld/index.json 114ms
MSBuild auto-detection: using msbuild version '17.5.0.10706' from 'C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\bin'.
  GET https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v3/registrations2-semver2/helloworld/index.json
  OK https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v3/registrations2-semver2/helloworld/index.json 698ms

Attempting to gather dependency information for package 'Helloworld.1.3.0.17' with respect to project 'C:\eShopOnWeb\EShopOnWeb.Shared', targeting 'Any,Version=v0.0'
Gathering dependency information took 21 ms
Attempting to resolve dependencies for package 'Helloworld.1.3.0.17' with DependencyBehavior 'Lowest'
Resolving dependency information took 0 ms
Resolving actions to install package 'Helloworld.1.3.0.17'
Resolved actions to install package 'Helloworld.1.3.0.17'
Retrieving package 'HelloWorld 1.3.0.17' from 'nuget.org'.
  GET https://api.nuget.org/v3-flatcontainer/helloworld/1.3.0.17/helloworld.1.3.0.17.nupkg
  OK https://api.nuget.org/v3-flatcontainer/helloworld/1.3.0.17/helloworld.1.3.0.17.nupkg 133ms
Installed HelloWorld 1.3.0.17 from https://api.nuget.org/v3/index.json with content hash 1Pbk5sGihV5JCE5hPLC0DirUypeW8hwSzfhD0x0InqpLRSvTEas7sPCVSylJ/KBzoxbGt2Iapg72WPbEYxLX9g==.
Adding package 'HelloWorld.1.3.0.17' to folder 'C:\eShopOnWeb\EShopOnWeb.Shared'
Added package 'HelloWorld.1.3.0.17' to folder 'C:\eShopOnWeb\EShopOnWeb.Shared'
Successfully installed 'HelloWorld 1.3.0.17' to C:\eShopOnWeb\EShopOnWeb.Shared
Executing nuget actions took 686 ms
```
1. Le package HelloWorld a été installé dans le sous-dossier **HelloWorld**, sous le dossier EShopOnWeb.Shared. Dans l’**Explorateur de solutions** de Visual Studio, accédez au projet **EShopOnWeb.Shared**, puis notez le sous-dossier **HelloWorld**. Cliquez sur la petite flèche à gauche du sous-dossier pour ouvrir le dossier et la liste de fichiers. 
1. Notez la présence du sous-dossier **lib**, qui contient un fichier de signature **signature.p7s** prouvant l’origine du package. Notez ensuite le fichier de package **HelloWorld.nupkg** lui-même. 

#### Tâche 4 : charger le package NuGet open source dans Azure Artifacts
Considérons que ce package est un package « approuvé » que notre équipe DevOps peut réutiliser. Nous allons le charger dans le flux de packages Azure Artifacts créé précédemment.

1. Dans la fenêtre PowerShell, exécutez la commande suivante :

 ```
.\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO c:\EShopOnWeb\EShopOnWeb.Shared\HelloWorld\HelloWorld.nupkg
```
    > **Remarque** : cette opération déclenche l’affichage d’un message d’erreur :

```
Response status code does not indicate success: 409 (Conflict - 'HelloWorld 1.3.0.17' cannot be published to the feed because it exists in at least one of the feed's upstream sources. Publishing this copy would prevent you from using 'HelloWorld 1.3.0.17' from 'NuGet Gallery'. For more information, see https://go.microsoft.com/fwlink/?linkid=864880 (DevOps Activity ID: AE08BE89-C2FA-4FF7-89B7-90805C88972C)).
```

Lorsque vous avez créé le flux de packages Azure DevOps Artifacts, par conception, il autoriser les **sources en amont**, telles que nuget.org dans l’exemple dotnet. Toutefois, rien n’empêche votre équipe DevOps de créer un flux de packages **« interne uniquement »**.

1. Accédez au portail Azure DevOps, accédez à **Artefacts**, puis sélectionnez le flux **EShopOnWebShared**. 
1. Cliquez sur **Rechercher des sources en amont**.
1. Dans la fenêtre **Accéder à un package en amont**, sélectionnez le type de package **Nuget**, puis entrez **HelloWorld** dans le champ de recherche.
1. Confirmez l’opération en appuyant sur le bouton **Rechercher**.
1. Cette opération affiche la liste de tous les packages HelloWorld avec les différentes versions disponibles.
1. Cliquez sur la **flèche gauche** pour revenir au flux **EShopOnWebShared**.   
1. Cliquez sur l’engrenage pour ouvrir **Paramètres de flux**. Sur la page Paramètres de flux, sélectionnez **Sources en amont**. 
1. Notez les différentes gestionnaires de packages en amont pour les différents langages de développement. Dans la liste, sélectionnez **Nuget.org**. Appuyez sur le bouton **Supprimer**, puis sur le bouton **Enregistrer**.

1. Une fois ces modifications enregistrées, il est possible de charger le package **HelloWorld** à l’aide de Nuget.exe à partir de la fenêtre PowerShell en exécutant à nouveau la commande suivante :

```
 .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO c:\EShopOnWeb\EShopOnWeb.Shared\HelloWorld\HelloWorld.nupkg
```

> **Remarque** : le chargement devrait à présent réussir. 

```
Pushing HelloWorld.nupkg to 'https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/'...
  PUT https://pkgs.dev.azure.com/<AZUREDEVOPSORGANIZATION>/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/
MSBuild auto-detection: using msbuild version '17.5.0.10706' from 'C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\bin'.
  Accepted https://pkgs.dev.azure.com/pdtdemoworld<AZUREDEVOPSORGANIZATION>/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/ 1645ms
Your package was pushed.
PS C:\eShopOnWeb\EShopOnWeb.Shared>
```

1. Dans le portail Azure DevOps, **actualisez** la page du flux de packages Artefacts. La liste des packages affiche le package personnalisé **EShopOnWeb.Shared** ainsi que le package public **HelloWorld**.
1. Dans la solution Visual Studio **EShopOnWeb.Shared**, cliquez avec le bouton droit sur le projet **EShopOnWeb.Shared**, puis, dans le menu contextuel, sélectionnez **Gérer les packages NuGet**. 
1. Dans la fenêtre du gestionnaire de packages NuGet, vérifiez que la **source du package** est définie sur **EShopOnWebShared**.
1. Cliquez sur **Parcourir**, puis attendez que la liste des packages NuGet se charge.
1. Cette liste affiche également le package personnalisé **EShopOnWeb.Shared** ainsi que le package public **HelloWorld**.

## Révision

Dans ce labo, vous avez appris à utiliser Azure Artifacts en effectuant les tâches suivantes :

- Création d’un flux et connexion à ce flux.
- Création et publication d’un package NuGet.
- Importation d’un package NuGet personnalisé.
- Importation d’un package NuGet public.
