---
lab:
  title: Gestion des packages avec Azure Artifacts
  module: 'Module 08: Design and implement a dependency management strategy'
---

# Gestion des packages avec Azure Artifacts

## Manuel de labo de l’étudiant

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Prérequis des labos du cours AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- **Configurez l’exemple de projet EShopOnWeb :** si vous ne disposez pas encore d’un exemple de projet EShopOnWeb que vous pouvez utiliser pour ce labo, créez-en un conformément aux instructions disponibles dans [Prérequis des labos du cours AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- Visual Studio 2022 Community Edition est disponible sur la [page des téléchargements de Visual Studio](https://visualstudio.microsoft.com/downloads/). L’installation de Visual Studio 2022 doit inclure les charges de travail **Développement ASP.NET et web<nolink>, **Développement Azure** et ****Développement multiplateforme .NET Core**.

## Présentation du labo

Azure Artifacts facilite la découverte, l’installation et la publication des packages NuGet, npm et Maven dans Azure DevOps. Il est étroitement intégré à d’autres fonctionnalités Azure DevOps telles que Build, ce qui permet à la gestion des packages d’être une partie transparente de vos workflows existants.

## Objectifs

À la fin de ce labo, vous serez en mesure d’accomplir les tâches suivantes :

- Créer et se connecter à un flux.
- Créer et publier un package NuGet.
- Importer un package NuGet.
- Mettre à jour un package NuGet.

## Durée estimée : 40 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

Dans le cadre de cet exercice, nous vous rappelons de valider les prérequis du laboratoire, de préparer votre organisation Azure DevOps et de créer le projet EShopOnWeb. Reportez-vous aux instructions ci-dessus pour en savoir plus.

#### Tâche 1 : configuration de la solution EShopOnWeb dans Visual Studio

Dans le cadre de cette tâche, vous configurerez Visual Studio pour préparer le labo.

1. Sur le Portail Azure DevOps, assurez-vous de vous trouver sur le projet d’équipe **EShopOnWeb**.

    > **Remarque** : vous pouvez accéder directement à la page du projet grâce à l’URL [`<your-Azure-DevOps-account-name>`https://dev.azure.com//EShopOnWeb](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/EShopOnWeb), dont l’espace réservé `<your-Azure-DevOps-account-name>` représente le nom de votre organisation Azure DevOps.

2. Dans le menu vertical situé à gauche du volet **EShopOnWeb**, cliquez sur **Repos**.
3. Dans le volet **Fichiers**, cliquez sur **Cloner**, sélectionnez la flèche déroulante en regard de **Cloner dans VS Code**, puis sélectionnez **Visual Studio** dans le menu déroulant qui apparaît.
4. Si un message vous demande de poursuivre, cliquez sur **Ouvrir**.
5. Si vous y êtes invité, connectez-vous avec le compte d’utilisateur que vous avez utilisé pour configurer votre organisation Azure DevOps.
6. Dans la fenêtre contextuelle **Azure DevOps** de l’interface de Visual Studio, acceptez le chemin local par défaut (C:\EShopOnWeb), puis cliquez sur **Cloner**. Le projet sera automatiquement importé dans Visual Studio.
7. Ne fermez pas la fenêtre Visual Studio, car vous l’utiliserez dans votre labo.

### Exercice 1 : utiliser Azure Artifacts

Dans le cadre de cet exercice, vous apprendrez à utiliser Azure Artifacts en procédant comme suit :

- Créer et se connecter à un flux.
- Créer et publier un package NuGet.
- Importer un package NuGet.
- Mettre à jour un package NuGet.

#### Tâche 1 : créer et se connecter à un flux

Dans le cadre de cette tâche, vous créerez et vous connecterez un flux.

1. Dans le volet de navigation vertical de la fenêtre du navigateur web affichant les paramètres de votre projet dans le Portail Azure DevOps, sélectionnez **Artefacts**.
2. Lorsque le hub des **artefacts** s’affiche, cliquez sur **+ Créer un flux** en haut du volet.

    > **Remarque** : ce flux rassemblera les packages NuGet disponibles pour les utilisateurs au sein de l’organisation et sera l’homologue du flux NuGet public. Le scénario de ce labo se concentre sur le workflow de l’utilisation d’Azure Artifacts : par conséquent, les choix architecturaux et de développement réels sont donnés à des fins d’exemple.  Ce flux inclura des fonctionnalités courantes pouvant être partagées entre les différents projets de cette organisation.

3. Ouvrez le volet **Créer un flux**. Dans la zone de texte **Nom**, saisissez **EShopOnWebShared** ; dans la section **Étendue**, sélectionnez l’option **Organisation** ; enfin, ne changez pas les valeurs par défaut des autres paramètres. Cliquez ensuite sur **Créer**.

    > **Remarque** : tout utilisateur souhaitant se connecter à ce flux NuGet doit configurer son environnement.

4. Revenez au hub des **artefacts**, puis cliquez sur **Connecter au flux**.
5. Dans la section **NuGet** du volet **Se connecter au flux**, sélectionnez **Visual Studio**. Dans le volet **Visual Studio**, copiez l’URL **source**. (https://pkgs.dev.azure.com/<Azure-DevOps-Org-Name>_packaging/EShopOnWebShared/nuget/v3/index.json)
6. Revenez à la fenêtre **Visual Studio **.
7. Dans la fenêtre Visual Studio, cliquez sur l’en-tête du menu **Outils**. Dans le menu déroulant qui s’affiche, sélectionnez **Gestionnaire de package NuGet**, puis cliquez sur **Paramètres du gestionnaire de package** dans le menu en cascade qui apparaît.
8. Dans la boîte de dialogue **Options**, sélectionnez **Sources des packages**, puis cliquez sur le symbole « + » pour ajouter une nouvelle source de package.
9. Dans la zone de texte **Nom** en bas de la boîte de dialogue, remplacez **Source du package** par **EShopOnWebShared**, puis collez l’URL que vous avez copiée dans le Portail Azure DevOps dans la zone de texte **Source**.
10. Cliquez sur **Mettre à jour**, puis sur **OK** pour finaliser l’ajout.

    > **Remarque** : Visual Studio est maintenant connecté au nouveau flux.

#### Tâche 2 : créer et publier un package NuGet développé en interne

Dans le cadre de cette tâche, vous créerez et publierez un package NuGet personnalisé développé en interne.

1. Dans le menu principal de la fenêtre Visual Studio que vous avez utilisée pour configurer la nouvelle source du package, cliquez sur **Fichier**. Cliquez ensuite sur **Nouveau** dans le menu déroulant qui s’affiche, puis sur **Projet** dans le menu en cascade qui apparaît.

    > **Remarque** : vous allez maintenant créer un assembly partagé qui sera publié en tant que package NuGet afin que d’autres équipes puissent l’intégrer et rester à jour sans devoir travailler directement avec la source du projet.

2. Sur la page **Modèles de projet récents** du volet **Créer un projet**, utilisez la zone de texte de recherche pour retrouver le modèle **Bibliothèque de classes**. Sélectionnez ensuite le modèle C#, puis cliquez sur **Suivant**.
3. Sur la page **Bibliothèque de classes** du volet **Créer un projet**, réglez les paramètres suivants, puis cliquez sur **Créer** :

    | Paramètre | Valeur |
    | --- | --- |
    | Nom du projet | **EShopOnWeb.Shared** |
    | Emplacement | accepter la valeur par défaut |
    | Solution | **Créer une solution** |
    | Nom de la solution | **EShopOnWeb.Shared** |

    Ne désactivez pas le paramètre **Placer la solution et le projet dans le même répertoire**.

4. Cliquez sur Suivant. Acceptez **.NET 7.0** en tant qu’option Framework.
5. Confirmez la création du projet en cliquant sur le bouton **Créer** .
6. Dans le volet **Explorateur de solutions**, effectuez un clic droit sur **Class1.cs**. Dans le menu qui apparaît, sélectionnez **Supprimer**, puis **OK** lorsque vous êtes invité à confirmer votre choix.
7. Appuyez sur **Ctrl+Maj+B** ou **effectuez un clic droit sur le projet EShopOnWeb.Shared**, puis sélectionnez **Générer** pour générer le projet.

    > **Remarque** : dans le cadre de la tâche suivante, vous utiliserez **NuGet.exe** pour générer un package NuGet directement à partir du projet généré. Le projet doit donc être généré en premier.

8. Revenez dans l’onglet du navigateur où vous avez ouvert le Portail Azure DevOps.
9. Accédez au volet **Connecter au flux**. Dans la section **NuGet**, cliquez sur **NuGet.exe**. Le volet **NuGet.exe** s’affiche.
10. Dans le volet **NuGet.exe**, cliquez sur **Obtenir les outils**.
11. Dans le volet **Obtenir les outils**, cliquez sur le lien **Télécharger le NuGet le plus récent** . Le navigateur ouvre automatiquement un autre onglet, qui affiche la page des **versions de distribution de NuGet disponibles**.
12. Sur la page des **versions de distribution de NuGet disponibles**, cliquez sur **nuget.exe : v6.x la plus récente recommandée** et téléchargez le fichier exécutable dans le dossier du **projet EShopOnWeb.Shared** (si vous avez conservé les emplacements par défaut des fichiers, il s’agit de C:\EShopOnWeb\EShopOnWeb.Shared).
13. Sélectionnez le fichier **nuget.exe**. Ouvrez ses propriétés en effectuant un clic droit sur le fichier, puis en sélectionnant **Propriétés** dans le menu contextuel.
14. Dans l’onglet **Général** de la fenêtre contextuelle des propriétés, sélectionnez **Débloquer** sous la section Sécurité. Confirmez votre choix en cliquant sur **Appliquer**, puis sur **OK**.
15. Depuis votre station de travail de labo, ouvrez le menu Démarrer et recherchez **Windows PowerShell**. Dans le menu en cascade qui apparaît, cliquez sur **Ouvrir Windows PowerShell en tant qu’administrateur**.
16. Dans la fenêtre **Administrateur : Windows PowerShell**, ouvrez le dossier EShopOnWeb.Shared en exécutant la commande ci-dessous :

    ```text
    cd c:\EShopOnWeb\EShopOnWeb.Shared
    ```

    Exécutez la commande ci-dessous pour créer un fichier **.nupkg** à partir du projet.

    > **Remarque** : il s’agit d’un raccourci pour générer un package reprenant les bits NuGet destinés au déploiement. NuGet offre des capacités de personnalisation avancées. Reportez-vous à la [page de création des packages NuGet](https://docs.microsoft.com/nuget/create-packages/overview-and-workflow) pour en savoir plus à ce sujet.

    ```text
    .\nuget.exe pack ./EShopOnWeb.Shared.csproj
    ```

    > **Remarque** : ignorez les avertissements de la fenêtre **Administrateur : Windows PowerShell**.

    > **Remarque** : NuGet génère un package minimal d’après les informations qu’il est capable d’identifier à partir du projet. Par exemple, le nom est **EShopOnWeb.Shared.1.0.0.nupkg** : ce numéro de version provient de l’assembly.

17. Une fois le package créé, exécutez la commande ci-dessous pour publier le package sur le flux **EShopOnWebShared** :

    > **Remarque** : vous devez indiquer une **clé API** qui peut être n’importe quelle chaîne non vide. Dans le cadre de cet exercice, vous utiliserez **AzDO**. Lorsque vous y êtes invité, connectez-vous à votre organisation Azure DevOps.

    ```text
    .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO EShopOnWeb.Shared.1.0.0.nupkg
    ```

18. Attendez de recevoir le message confirmant la réussite de la transmission de type push du package.
19. Dans le volet de navigation vertical de la fenêtre du navigateur web affichant le Portail Azure DevOps, sélectionnez **Artefacts**.
20. Dans le volet du hub des **artefacts**, cliquez sur la liste déroulante dans le coin supérieur gauche, puis sélectionnez l’entrée **EShopOnWebShared** dans la liste des flux.

    > **Remarque** : le flux **EShopOnWebShared** doit inclure le package NuGet que vous venez de publier.

21. Cliquez sur le package NuGet pour en afficher les détails.

#### Tâche 3 : importer un package NuGet open source dans le flux de packages Azure DevOps

Outre le développement de vos propres packages, vous pouvez utiliser la bibliothèque de packages DotNet NuGet open source (https://www.nuget.org)). L’un de ses millions de packages disponibles vous sera forcément utile pour votre application.

Dans le cadre de cette tâche, vous utiliserez un exemple générique de package « Hello World », mais l’approche que vous adopterez convient également à tous les autres packages de la bibliothèque.

1. Dans la même fenêtre PowerShell, exécutez la commande **nuget** ci-dessous pour installer l’exemple de package :

    ```text
    .\nuget install HelloWorld -ExcludeVersion
    ```

2. Vérifiez le résultat du processus d’installation. La première ligne indique les différents flux utilisés pour tenter de télécharger le package :

    ```text
    Feeds used:
      https://api.nuget.org/v3/index.json
      https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/eShopOnWeb/_packaging/EShopOnWebPFeed/nuget/v3/index.json
    ```

3. Le résultat complémentaire du processus d’installation proprement dit apparaît à la suite.

    ```text
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

4. Le package HelloWorld a été installé dans un sous-dossier **HelloWorld** du dossier EShopOnWeb.Shared. À partir de l’**explorateur de solutions** de Visual Studio, ouvrez le projet **EShopOnWeb.Shared** pour voir le sous-dossier **HelloWorld**. Cliquez sur la petite flèche à gauche du sous-dossier pour ouvrir le dossier et la liste des fichiers.
5. Le sous-dossier **lib** contient un fichier de signature **.p7s** qui vérifie l’origine du package. Le fichier du package **HelloWorld.nupkg** lui-même se trouve au même emplacement.

#### Tâche 4 : charger le package NuGet open source dans Azure Artifacts

Ce package est considéré comme « approuvé » et peut être réutilisé par l’équipe DevOps en le chargeant dans le flux de package Azure Artifacts créé à une étape précédente.

1. Dans la fenêtre PowerShell, exécutez les commandes ci-dessous :

    ```text
    .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO c:\EShopOnWeb\EShopOnWeb.Shared\HelloWorld\HelloWorld.nupkg
    ```

    > **Remarque** : cette action déclenche l’affichage d’un message d’erreur :

    ```text
    Response status code does not indicate success: 409 (Conflict - 'HelloWorld 1.3.0.17' cannot be published to the feed because it exists in at least one of the feed's upstream sources. Publishing this copy would prevent you from using 'HelloWorld 1.3.0.17' from 'NuGet Gallery'. For more information, see https://go.microsoft.com/fwlink/?linkid=864880 (DevOps Activity ID: AE08BE89-C2FA-4FF7-89B7-90805C88972C)).
    ```

Une fois le flux de packages Azure DevOps Artifacts créé, il autorise de base les **sources en amont**, telles que nuget.org dans l’exemple dotnet. Toutefois, rien n’empêche l’équipe DevOps de créer un flux de packages **réservé aux packages en interne**.

1. Ouvrez le Portail Azure DevOps, accédez à la section **Artefacts**, puis sélectionnez le flux **EShopOnWebShared**.
2. Cliquez sur **Rechercher des sources en amont**.
3. Dans la fenêtre **Accéder à un package en amont**, définissez le type de package sur **NuGet**, puis saisissez **HelloWorld** dans le champ de recherche.
4. Confirmez votre entrée en cliquant sur le bouton **Rechercher** .
5. Une liste de tous les packages HelloWorld avec leurs différentes versions disponibles s’affiche.
6. Appuyez sur la **flèche gauche du clavier** pour revenir au flux **EShopOnWebShared**.
7. Cliquez sur l’engrenage pour ouvrir les **paramètres de flux**. Sur la page des paramètres de flux, sélectionnez **Sources en amont**.
8. Plusieurs gestionnaires de packages en amont apparaissent pour différents langages de développement. Dans la liste, cliquez sur **Galerie NuGet**. Cliquez sur le bouton **Supprimer**, puis sur le bouton **Enregistrer**.

9. Une fois ces modifications enregistrées, il est possible de charger le package **HelloWorld** à l’aide de NuGet.exe depuis la fenêtre PowerShell en relançant la commande ci-dessous :

    ```text
     .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO c:\EShopOnWeb\EShopOnWeb.Shared\HelloWorld\HelloWorld.nupkg
    ```

    > **Remarque** : le chargement doit maintenant réussir. 

    ```text
    Pushing HelloWorld.nupkg to 'https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/'...
      PUT https://pkgs.dev.azure.com/<AZUREDEVOPSORGANIZATION>/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/
    MSBuild auto-detection: using msbuild version '17.5.0.10706' from 'C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\bin'.
      Accepted https://pkgs.dev.azure.com/pdtdemoworld<AZUREDEVOPSORGANIZATION>/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/ 1645ms
    Your package was pushed.
    PS C:\eShopOnWeb\EShopOnWeb.Shared>
    ```

10. Depuis le Portail Azure DevOps, **actualisez** la page des flux de packages d’artefacts. La liste des packages affiche à la fois le package au développement personnalisé **EShopOnWeb.Shared** ainsi que le package public **HelloWorld**.
11. Dans la solution **EShopOnWeb.Shared** de Visual Studio, effectuez un clic droit sur le projet **EShopOnWeb.Shared**, puis sélectionnez **Gérer les packages NuGet** dans le menu contextuel qui apparaît.
12. Dans la fenêtre Gestionnaire de packages NuGet, vérifiez que la **source du package** est définie sur **EShopOnWebShared**.
13. Cliquez sur **Parcourir**, puis attendez le chargement de la liste des packages NuGet.
14. Cette liste affiche à la fois le package au développement personnalisé **EShopOnWeb.Shared** ainsi que le package public **HelloWorld**.

## Révision

Dans le cadre de ce labo, vous avez appris à travailler avec Azure Artifacts en suivant les étapes ci-dessous :

- Créer et se connecter à un flux.
- Créer et publier un package NuGet.
- Importer un package NuGet personnalisé.
- Importer un package NuGet public.
