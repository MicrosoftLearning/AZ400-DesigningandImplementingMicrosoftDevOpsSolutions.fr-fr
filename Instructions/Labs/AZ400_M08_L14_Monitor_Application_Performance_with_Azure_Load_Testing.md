---
lab:
  title: Monitorer les performances des applications avec Test de charge Azure
  module: 'Module 08: Implement continuous feedback'
---

# Monitorer les performances des applications avec Test de charge Azure

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Créer une organisation ou une collection de projets](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifier un abonnement Azure existant ou en créer un.

- Vérifiez que vous disposez d’un compte Microsoft ou d’un compte Microsoft Entra avec le rôle Propriétaire dans l’abonnement Azure et le rôle Administrateur général dans le locataire Microsoft Entra associé à l’abonnement Azure. Pour plus d’informations, consultez [Répertorier les attributions de rôle Azure à l’aide du portail Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) et [Afficher et attribuer des rôles d’administrateur dans Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal#view-my-roles).

## Présentation du labo

**Test de charge Azure** est un service complètement managé qui permet de générer une charge à grande échelle. Il simule un trafic pour vos applications, quel que soit l’endroit où elles sont hébergées. Les développeurs, testeurs et ingénieurs d’assurance qualité peuvent l’utiliser pour optimiser les performances, la scalabilité ou la capacité d’une application.
Créez rapidement un test de charge pour votre application web à l’aide d’une URL, sans connaissance préalable des outils de test. Test de charge Azure réduit la complexité et l’infrastructure pour exécuter votre test de charge à grande échelle.
Pour des scénarios de test de charge plus avancés, vous pouvez créer un test de charge en réutilisant un script de test Apache JMeter existant, un outil de charge et d’évaluation des performances open source populaire. Par exemple, votre plan de test peut se composer de plusieurs requêtes d’application, vous souhaitez appeler des points de terminaison non-HTTP, ou vous utilisez des données et paramètres d’entrée pour rendre le test plus dynamique.

Dans ce labo, vous allez découvrir comment utiliser Test de charge Azure pour simuler des tests de performances sur une application web en direct avec différents scénarios de charge. Enfin, vous allez apprendre à intégrer Test de charge Azure dans vos pipelines CI/CD.

## Objectifs

À la fin de ce labo, vous serez en mesure d’effectuer les tâches suivants :

- Déployer des applications web Azure App Service.
- Créer et exécuter un pipeline CI/CD basé sur YAML.
- Déployer Test de charge Azure.
- Examiner les performances des applications web Azure à l’aide de Test de charge Azure.
- Intégrer Test de charge Azure à vos pipelines CI/CD.

## Durée estimée : 60 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

Dans cet exercice, vous allez configurer les prérequis du labo.

#### Tâche 1 : (passer si terminée) créer et configurer le projet d’équipe

Dans cette tâche, vous allez créer un projet Azure DevOps **eShopOnWeb** à utiliser par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps. Cliquez sur **Nouveau projet**. Donnez au projet le nom **eShopOnWeb**, puis choisissez **Scrum** dans la liste déroulante **Processus d’élément de travail**. Cliquez sur **Créer**.

   ![Capture d’écran du volet de création d’un nouveau projet.](images/create-project.png)

#### Tâche 2 : (passer si terminée) importer le référentiel Git eShopOnWeb

Dans cette tâche, vous allez importer le référentiel Git eShopOnWeb qui sera utilisé par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps et le projet **eShopOnWeb** créé précédemment. Cliquez sur **Repos > Fichiers**, **Importer**. Dans la fenêtre **Importer un référentiel Git**, collez l’URL <https://github.com/MicrosoftLearning/eShopOnWeb.git>, puis cliquez sur **Importer** :

   ![Capture d’écran du panneau Importer un référentiel.](images/import-repo.png)

1. Le référentiel est organisé de la manière suivante :
   - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
   - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
   - Le dossier **infra** contient l’infrastructure Bicep&ARM en tant que modèles de code utilisés dans certains scénarios de labo.
   - **Définitions de workflow GitHub YAML du conteneur de dossiers .github**.
   - Le dossier **src** contient le site web .NET 8 utilisé dans les scénarios de labo.

#### Tâche 3 : (à ignorer si vous l’avez déjà effectuée) définir la branche principale en tant que branche par défaut

1. Accédez à **Repos > Branches**.
1. Pointez sur la branche **principale**, puis cliquez sur les points de suspension à droite de la colonne.
1. Cliquez sur **Définir comme branche par défaut**.

#### Tâche 4 : Créer des ressources Azure

Dans cette tâche, vous allez créer une application web Azure à l’aide de Cloud Shell dans le portail Azure.

1. À partir de l’ordinateur de labo, lancez un navigateur web et accédez au [**portail Azure**](https://portal.azure.com), puis connectez-vous.
1. Dans le portail Azure, dans la barre d’outils, cliquez sur l’icône **Cloud Shell** située juste à droite de la zone de texte de recherche.
1. Si vous êtes invité à sélectionner **Bash** ou **PowerShell**, sélectionnez **Bash**.

   > **Remarque** : si c’est la première fois que vous démarrez **Cloud Shell** et que vous voyez le message **Vous n’avez aucun stockage monté**, sélectionnez l’abonnement que vous utilisez dans ce labo, puis sélectionnez **Créer un stockage**.

1. À partir de l’invite **Bash**, dans le volet **Cloud Shell**, exécutez la commande suivante pour créer un groupe de ressources (remplacez l’espace réservé `<region>` par le nom de la région Azure la plus proche de vous, par exemple « eastus »).

   ```bash
   RESOURCEGROUPNAME='az400m08l14-RG'
   LOCATION='<region>'
   az group create --name $RESOURCEGROUPNAME --location $LOCATION
   ```

1. Créez un plan App Service Windows en exécutant la commande suivante :

   ```bash
   SERVICEPLANNAME='az400l14-sp'
   az appservice plan create --resource-group $RESOURCEGROUPNAME \
       --name $SERVICEPLANNAME --sku B3
   ```

1. Créez une application web avec un nom unique.

   ```bash
   WEBAPPNAME=az400eshoponweb$RANDOM$RANDOM
   az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME
   ```

   > **Remarque** : notez le nom de l’application web. Vous en aurez besoin plus tard dans ce labo.

### Exercice 1 : configurer des pipelines CI/CD en tant que code avec YAML dans Azure DevOps

Dans cet exercice, vous allez configurer des pipelines CI/CD en tant que code avec YAML dans Azure DevOps.

#### Tâche 1 : ajouter un build YAML et déployer une définition

Dans cette tâche, vous allez ajouter une définition de build YAML au projet existant.

1. Revenez au volet **Pipelines** du hub **Pipelines**.
1. Cliquez sur **Nouveau pipeline** (ou Créer un pipeline s’il s’agit du premier pipeline que vous créez).

   > **Remarque** : nous allons utiliser l’Assistant pour créer une définition de pipeline YAML basée sur notre projet.

1. Dans le volet **Où se trouve votre code ?**, cliquez sur l’option **Azure Repos Git (YAML)**.
1. Dans le volet **Sélectionner un référentiel**, cliquez sur **eShopOnWeb**.
1. Dans le volet **Configurer votre pipeline**, faites défiler vers le bas et sélectionnez **Pipeline de démarrage**.
1. **Sélectionnez** toutes les lignes du pipeline de démarrage, puis supprimez-les.
1. **Copiez** le pipeline de modèle complet ci-dessous, sachant que vous devez apporter des modifications aux paramètres **avant d’enregistrer** les modifications :

   ```yml
   #Template Pipeline for CI/CD
   # trigger:
   # - main

   resources:
     repositories:
       - repository: self
         trigger: none

   stages:
     - stage: Build
       displayName: Build .Net Core Solution
       jobs:
         - job: Build
           pool:
             vmImage: ubuntu-latest
           steps:
             - task: DotNetCoreCLI@2
               displayName: Restore
               inputs:
                 command: "restore"
                 projects: "**/*.sln"
                 feedsToUse: "select"

             - task: DotNetCoreCLI@2
               displayName: Build
               inputs:
                 command: "build"
                 projects: "**/*.sln"

             - task: DotNetCoreCLI@2
               displayName: Publish
               inputs:
                 command: "publish"
                 publishWebProjects: true
                 arguments: "-o $(Build.ArtifactStagingDirectory)"

             - task: PublishBuildArtifacts@1
               displayName: Publish Artifacts ADO - Website
               inputs:
                 pathToPublish: "$(Build.ArtifactStagingDirectory)"
                 artifactName: Website

     - stage: Deploy
       displayName: Deploy to an Azure Web App
       jobs:
         - job: Deploy
           pool:
             vmImage: "windows-2019"
           steps:
             - task: DownloadBuildArtifacts@1
               inputs:
                 buildType: "current"
                 downloadType: "single"
                 artifactName: "Website"
                 downloadPath: "$(Build.ArtifactStagingDirectory)"
   ```

1. Définissez le curseur sur une nouvelle ligne à la fin de la définition YAML. **Veillez à positionner le curseur au niveau de la tâche précédente**.

   > **Remarque** : il s’agit de l’emplacement où les nouvelles tâches sont ajoutées.

1. Cliquez sur **Afficher l’Assistant** sur le côté droit du portail. Dans la liste des tâches, recherchez et sélectionnez la tâche **Déploiement Azure App Service**.
1. Dans le volet **Déploiement Azure App Service**, spécifiez les paramètres suivants, puis cliquez sur **Ajouter** :

   - Dans la liste déroulante **Abonnement Azure**, sélectionnez la connexion de service que vous venez de créer.
   - Vérifiez que **Type d’App Service** pointe vers Application web sur Windows.
   - Dans la liste déroulante **Nom d’App Service**, sélectionnez le nom de l’application web que vous avez déployée précédemment dans le labo (\*\*az400eshoponweb...).
   - Dans la zone de texte **Package ou dossier**, **mettez à jour** la valeur par défaut sur `$(Build.ArtifactStagingDirectory)/**/Web.zip`.
   - Développez **Paramètres d’application et de configuration**, puis, dans la zone de texte Paramètres d’application, ajoutez les paires clé-valeur suivantes : `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development`.

1. Confirmez les paramètres du volet Assistant en cliquant sur le bouton **Ajouter**.

   > **Remarque** : cette opération ajoute automatiquement la tâche de déploiement à la définition du pipeline YAML.

1. L’extrait de code ajouté à l’éditeur doit ressembler à ce qui suit et refléter votre nom pour les paramètres azureSubscription et WebappName :

   ```yml
   - task: AzureRmWebAppDeployment@4
     inputs:
       ConnectionType: "AzureRM"
       azureSubscription: "SERVICE CONNECTION NAME"
       appType: "webApp"
       WebAppName: "az400eshoponWeb369825031"
       packageForLinux: "$(Build.ArtifactStagingDirectory)/**/Web.zip"
       AppSettings: "-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development"
   ```

   > **Remarque** : le paramètre **packageForLinux** est trompeur dans le contexte de ce labo, mais il est valide pour Windows ou Linux.

1. Avant d’enregistrer les mises à jour dans le fichier yml, donnez-lui un nom plus clair. En haut de la fenêtre yaml-editor figure le nom **EShopOnweb/azure-pipelines-#.yml**. (Où # est un nombre, généralement 1, mais il peut être différent dans votre configuration.) Sélectionnez **ce nom de fichier** et renommez-le en **m08l14-pipeline.yml**.

1. Cliquez sur **Enregistrer**, dans le volet **Enregistrer**, puis cliquez de nouveau sur **Enregistrer** pour valider la modification directement dans la branche principale.

   > **Remarque** : étant donné que notre fichier CI-YAML d’origine n’a pas été configuré pour déclencher automatiquement une nouvelle build, nous devons lancer cette dernière manuellement.

1. Dans le menu de gauche Azure DevOps, accédez à **Pipelines**, puis sélectionnez à nouveau **Pipelines**. Ensuite, sélectionnez **Tout** pour ouvrir toutes les définitions de pipeline, pas seulement les définitions récentes.

   > **Remarque** : si vous avez conservé tous les pipelines précédents des exercices de labo précédents, il est possible que ce nouveau pipeline ait réutilisé un nom de séquence **eShopOnWeb (#)** par défaut pour le pipeline, comme illustré dans la capture d’écran ci-dessous. Sélectionnez un pipeline (comme celui avec le numéro de séquence le plus élevé), sélectionnez Modifier et vérifiez qu’il pointe vers le fichier de code m08l14-pipeline.yml.

   ![Capture d’écran d’Azure Pipelines avec eShopOnWeb en cours d’exécution.](images/m3/eshoponweb-m9l16-pipeline.png)

1. Confirmez l’exécution de ce pipeline en cliquant sur **Exécuter** à partir du volet qui s’affiche et en cliquant une fois de plus sur **Exécuter**.
1. Notez les 2 phases différentes affichées, **Générer la solution .NET Core** et **Déployer sur Azure Web App**.
1. Attendez que le pipeline démarre.

1. **Ignorez** les avertissements affichés pendant la phase de génération. Patientez jusqu’à ce que la phase de génération se termine correctement. (Vous pouvez sélectionner la phase de génération pour afficher plus de détails dans les journaux.)

1. Une fois que l’étape de déploiement souhaite commencer, vous êtes invité à utiliser les **autorisations nécessaires**, ainsi qu’une barre orange indiquant :

   ```text
   This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
   ```

1. Cliquez sur **Affichage**.
1. Dans le volet **En attente d’examen**, cliquez sur **Autoriser**.
1. Validez le message dans la fenêtre contextuelle **Autoriser**, puis confirmez en cliquant sur **Autoriser**.
1. Cette opération lance la phase de déploiement. Attendez qu’elle se termine.

#### Tâche 2 : examiner le site déployé

1. Revenez à la fenêtre du navigateur web qui affiche le portail Azure et accédez au panneau affichant les propriétés de l’application web Azure.
1. Dans le panneau de l’application web Azure, cliquez sur **Vue d’ensemble**, puis, dans le panneau Vue d’ensemble, cliquez sur **Parcourir** pour ouvrir votre site dans un nouvel onglet de navigateur web.
1. Vérifiez que le site déployé se charge comme prévu dans le nouvel onglet du navigateur, affichant le site web d’e-commerce eShopOnWeb.

### Exercice 2 : déployer et configurer Test de charge Azure

Dans cet exercice, vous allez déployer une ressource de test de charge Azure dans Azure et configurer différents scénarios de test de charge pour votre Azure App Service en cours d’exécution.

> **Important** : le Test de charge Azure est un **service payant**. Vous entraînez des coûts pour l’exécution de tests de charge. Veillez à supprimer les ressources une fois que vous avez terminé le labo pour éviter d’entraîner des coûts supplémentaires. Pour chaque « Ressource de test de charge » active pendant une partie d’un mois, vous serez facturé les frais mensuels et avez accès à 50 VUH inclus. Pour plus d’informations, consultez la [page de la tarification du Test de charge Azure](https://azure.microsoft.com/pricing/details/load-testing).

#### Tâche 1 : déployer Test de charge Azure

Dans cette tâche, vous allez déployer une nouvelle ressource de Test de charge Azure dans votre abonnement Azure.

1. À partir du portail Azure (<https://portal.azure.com>), accédez à **Créer une ressource Azure**.
1. Dans le champ de recherche « Rechercher dans les services et la Place de marché », entrez **`Azure Load Testing`**.
1. Dans les résultats de la recherche, sélectionnez **Test de charge Azure** (publié par Microsoft).
1. Dans la page Test de charge Azure, cliquez sur **Créer** pour démarrer le processus de déploiement.
1. Dans la page « Créer une ressource de test de charge », fournissez les détails nécessaires au déploiement de la ressource :

   - **Abonnement** : sélectionnez votre abonnement Azure.
   - **Groupe de ressources** : sélectionnez le groupe de ressources que vous avez utilisé pour déployer Web App Service dans l’exercice précédent.
   - **Nom :** `eShopOnWebLoadTesting`
   - **Région** : sélectionnez une région proche de vous.

   > **Remarque** : le service Test de charge Azure n’est pas disponible dans toutes les régions Azure.

1. Cliquez sur **Examiner et créer** pour valider vos paramètres.
1. Cliquez sur **Créer** pour confirmer et procéder au déploiement de la ressource Test de charge Azure.
1. Vous êtes redirigé vers la page « Déploiement en cours ». Patientez quelques minutes jusqu’à la fin du déploiement.
1. Cliquez sur **Accéder à la ressource** à partir de la page de progression du déploiement pour accéder à la ressource Test de charge Azure **eShopOnWebLoadTesting**.

   > **Remarque** : si vous avez fermé le panneau ou le portail Azure pendant le déploiement de la ressource Test de charge Azure, vous pouvez accéder à la ressource à partir du champ de recherche du portail Azure, de la liste des ressources ou de la liste des ressources consultées récemment.

#### Tâche 2 : créer des tests de Test de charge Azure

Dans cette tâche, vous allez créer plusieurs tests de Test de charge Azure à l’aide de différents paramètres de configuration de charge.

1. Dans le panneau de la ressource Test de charge Azure **eShopOnWebLoadTesting**, accédez à **Tests** dans **Tests**. Cliquez sur l’option de menu **+Créer**, puis sélectionnez **Créer un test basé sur une URL**.
1. Décochez la case **Activer les paramètres avancés** pour afficher les paramètres avancés.
1. Configurez les paramètres suivants pour créer un test de charge :

   - **URL de test** : entrez l’URL de l’instance Azure App Service que vous avez déployée dans l’exercice précédent (az400eshoponweb... azurewebsites.net), **y compris https://**
   - **Spécifier la charge** : utilisateurs virtuels
   - **Nombre d’utilisateurs virtuels** : 50
   - **Durée du test (minutes)**  : 5
   - **Durée d’accélération (minutes)**  :  1

1. Confirmez la configuration du test en cliquant sur **Vérifier et créer** (n’apportez aucune modification sous les autres onglets). Cliquez une fois de plus sur **Créer**.
1. Cela lance les tests Test de charge, qui exécuteront le test pendant cinq minutes.
1. Une fois le test en cours d’exécution, revenez à la page de la ressource Test de charge Azure **eShopOnWebLoadTesting**, puis accédez à **Tests**, sélectionnez **Tests** et consultez un test **Get_eshoponweb...**
1. Dans le menu du haut, cliquez sur **Créer**, **Créer un test basé sur une URL** pour créer un deuxième test de charge.
1. Configurez les paramètres suivants pour créer un autre test de charge :

   - **URL de test** : Entrez l’URL de l’instance Azure App Service que vous avez déployée dans l’exercice précédent (eShopOnWeb...azurewebsites.net), **y compris https://**
   - **Spécifier la charge** : demandes par seconde (RPS)
   - **Demandes par seconde (RPS)**  : 100
   - **Temps de réponse (millisecondes)**  : 500
   - **Durée du test (minutes)**  : 5
   - **Durée d’accélération (minutes)**  :  1

1. Confirmez la configuration du test en cliquant sur **Vérifier + créer**, puis sur **Créer** une fois de plus.
1. Le test s’exécute pendant environ cinq minutes.

#### Tâche 3 : valider les résultats des tests de Test de charge Azure

Dans cette tâche, vous allez valider le résultat d’un TestRun de Test de charge Azure.

Les deux tests rapides ont été réalisés et nous allons à présent les peaufiner, puis valider les résultats.

1. À partir de **Test de charge Azure**, accédez à **Tests**. Sélectionnez l’une des définitions de test pour ouvrir une vue plus détaillée en **cliquant** sur l’un des tests. Vous êtes alors redirigé vers la page de test plus détaillée. À partir de là, vous pouvez valider les détails des exécutions réelles, en sélectionnant le **TestRun_mm/jj/aa-hh :hh** dans la liste résultante.
1. Dans la page **TestRun** détaillée, identifiez le résultat réel de la simulation Test de charge Azure. Voici quelques-unes des valeurs :

   - Charge/Total des demandes
   - Durée
   - Temps de réponse (indique le résultat en secondes, reflétant le temps de réponse aux 90e centiles, ce qui signifie que, pour 90 % des demandes, le temps de réponse se trouve dans les résultats donnés)
   - Débit en demandes par seconde

1. D’autres valeurs indiquées ci-dessous. Plusieurs d’entre elles sont représentées à l’aide des vues de graphiques et graphiques linéaires du tableau de bord.
1. Prenez quelques minutes pour **comparer les résultats** des deux tests simulés entre eux et **identifier l’impact** d’un plus grand nombre d’utilisateurs sur les performances d’App Service.

### Exercice 3 : automatiser un test de charge avec intégration continue et livraison continue (CI/CD) dans Azure Pipelines

Commencez par automatiser les tests de charge dans le Test de charge Azure en l’ajoutant à un pipeline CI/CD. Après avoir exécuté un test de charge dans le portail Azure, vous exportez les fichiers de configuration et configurez un pipeline CI/CD dans Azure Pipelines (des fonctionnalités similaires existent pour GitHub Actions).

Une fois cet exercice terminé, vous disposez d’un workflow CI/CD configuré pour exécuter un test de charge avec Test de charge Azure.

#### Tâche 1 : identifier les détails de la connexion de service Azure DevOps

Dans cette tâche, vous allez accorder les autorisations requises à la connexion de service Azure DevOps.

1. Dans le **portail Azure DevOps**(<https://aex.dev.azure.com>), accédez au projet **eShopOnWeb**.
1. Dans le coin inférieur gauche, sélectionnez **Paramètres du projet**.
1. Sous la section **Pipelines**, sélectionnez **Connexions de service**.
1. Notez la connexion de service, portant le nom de votre abonnement Azure que vous avez utilisé pour déployer des ressources Azure au début de l’exercice de labo.
1. **Sélectionnez la connexion de service**. Dans l’onglet **Vue d’ensemble**, accédez à **Détails** et sélectionnez **Gérer les rôles de connexion de service**.
1. Vous êtes alors redirigé vers le portail Azure, à partir duquel il ouvre les détails du groupe de ressources dans le panneau contrôle d’accès (IAM).

#### Tâche 2 : accorder des autorisations à la ressource Test de charge Azure

Le Test de charge Azure utilise le contrôle d’accès en fonction du rôle (RBAC) Azure pour accorder des autorisations afin d’effectuer des activités spécifiques sur votre ressource de test de charge. Pour exécuter un test de charge à partir de votre pipeline CI/CD, vous accordez le rôle de **Contributeur au test de charge** à la connexion de service Azure DevOps.

1. Sélectionnez **+ Ajouter**, puis **Ajouter une attribution de rôle**.
1. Sous l’onglet **Rôle**, sélectionnez **Contributeur au test de charge** dans la liste des rôles de fonction de travail.
1. Dans l’**onglet Membres**, cliquez sur **Sélectionner des membres**, puis trouvez et sélectionnez votre compte d’utilisateur, et cliquez sur **Sélectionner**.
1. Dans l’onglet **Passer en revue + attribuer**, sélectionnez **Passer en revue + attribuer** pour ajouter l’attribution de rôle.

Vous pouvez maintenant utiliser la connexion de service dans votre définition de workflow Azure Pipelines pour accéder à votre ressource de test de charge Azure.

#### Tâche 3 : Exporter des fichiers d’entrée de test de charge et importer dans Azure Repos

Pour exécuter un test de charge avec le Test de charge Azure dans un workflow CI/CD, vous devez ajouter les paramètres de configuration de test de charge et tous les fichiers d’entrée dans votre référentiel de contrôle de code source. Si vous disposez d’un test de charge existant, vous pouvez télécharger les paramètres de configuration et tous les fichiers d’entrée à partir du portail Azure.

Effectuez les étapes suivantes pour télécharger les fichiers d’entrée d’un test de charge existant dans le Portail Azure :

1. Dans le **portail Azure**, accédez à votre ressource **Test de charge Azure**.
1. Dans le volet de gauche, sélectionnez **Tests** pour afficher la liste des tests de charge, puis sélectionnez **votre test**.
1. Sélectionnez les points de suspension (**...**) en regard de la série de tests de charge sur laquelle vous travaillez, puis sélectionnez **Télécharger le fichier d’entrée**.
1. Le navigateur télécharge un dossier compressé qui contient les fichiers d’entrée de test de charge.
1. Utilisez n’importe quel outil de compression pour extraire les fichiers d’entrée. Ce dossier contient les fichiers suivants :

   - _config.yaml_ : fichier de configuration YAML de test de charge. Vous référencez ce fichier dans la définition du workflow CI/CD.
   - _quick_test.jmx_ : script de test JMeter

1. Commitez tous les fichiers d’entrée extraits vers votre référentiel de contrôle de code source. Pour cela, accédez au **portail Azure DevOps**(<https://aex.dev.azure.com/>), puis au projet DevOps **eShopOnWeb**.
1. Sélectionnez **Repos**. Dans la structure du dossier du code source, remarquez le sous-dossier **tests**. Notez les points de suspension (…), puis sélectionnez **Nouveau > Dossier**.
1. Spécifiez **jmeter** comme nom de dossier et **placeholder.txt** pour le nom de fichier (un dossier ne peut pas être créé comme vide).
1. Cliquez sur **Valider** pour confirmer la création du fichier d’espace réservé et du dossier jmeter.
1. Dans la **Structure de dossiers**, accédez au sous-dossier **jmeter** nouvellement créé. Cliquez sur les **points de suspension (…)** et sélectionnez **Charger des fichiers**.
1. À l’aide de l’option **Parcourir**, accédez à l’emplacement du fichier ZIP extrait, puis sélectionnez **config.yaml** et **quick_test.jmx**.
1. Cliquez sur **Valider** pour confirmer le chargement du fichier dans le contrôle de code source.
1. Dans les référentiels, accédez au sous-dossier **/tests/jmeter** qui vient d’être créé.
1. Ouvrez le fichier de test de charge **config.yaml**. Cliquez sur **Modifier** pour autoriser la modification du fichier.
1. Remplacez les attributs **displayName** et **testId** par la valeur **ado_load_test**.

  ![Capture d’écran du fichier de configuration modifié.](images/config_edit.png)

#### Tâche 4 : mettre à jour le fichier de définition YAML du flux de travail CI/CD

1. Pour créer et exécuter un test de charge, la définition du workflow Azure Pipelines utilise l’extension de **tâche Test de charge Azure** disponible dans la Place de marché Azure DevOps. Ouvrez l’[extension de tâche Test de charge Azure](https://marketplace.visualstudio.com/items?itemName=AzloadTest.AzloadTesting) dans la Place de marché Azure DevOps, puis sélectionnez **Obtenir gratuitement**.
1. Sélectionnez votre organisation Azure DevOps, puis sélectionnez **Installer** pour installer l’extension.
1. À partir du portail Azure DevOps et du projet, accédez à **Pipelines** et sélectionnez le pipeline créé au début de cet exercice. Cliquez sur **Modifier**.
1. Dans le script YAML, accédez à la **ligne 56** et appuyez sur ENTRÉE/RETOUR pour ajouter une nouvelle ligne vide. (Juste avant la phase de déploiement du fichier YAML).
1. À la ligne 57, sélectionnez l’Assistant des tâches sur le côté droit et recherchez **Test de charge Azure**.
1. Complétez le volet graphique avec les paramètres corrects de votre scénario :

   - Abonnement Azure : sélectionnez l’abonnement où sont exécutées vos ressources Azure.
   - Fichier du test de charge : '$(Build.SourcesDirectory)/tests/jmeter/config.yaml'
   - Groupe de ressources du test de charge : le groupe de ressources qui contient vos ressources de test de charge Azure.
   - Nom de ressource de test de charge : `eShopOnWebLoadTesting`
   - Nom d’exécution du test de charge : ado_run
   - Description de l’exécution du test de charge : test de charge à partir d’ADO

1. Confirmez l’injection des paramètres en tant qu’extrait de code YAML en cliquant sur **Ajouter**.
1. Si la mise en retrait de l’extrait de code YAML donne des erreurs (lignes rouges ondulées), corrigez-les en ajoutant 2 espaces ou une tabulation pour positionner correctement l’extrait de code.
1. Votre code YAML doit ressembler à l’exemple d’extrait de code suivant :

   ```yml
        - task: AzureLoadTest@1
         inputs:
           azureSubscription: 'AZURE DEMO SUBSCRIPTION'
           loadTestConfigFile: '$(Build.SourcesDirectory)/tests/jmeter/config.yaml'
           resourceGroup: 'az400m08l14-RG'
           loadTestResource: 'eShopOnWebLoadTesting'
           loadTestRunName: 'ado_run'
           loadTestRunDescription: 'load testing from ADO'
   ```

1. Sous l’extrait de code YAML inséré, ajoutez une nouvelle ligne vide en appuyant sur les touches ENTRÉE/RETOUR.
1. Sous cette ligne vide, ajoutez un extrait de code pour la tâche Publier qui indique les résultats de la tâche de test de charge Azure pendant l’exécution du pipeline :

   ```yml
   - publish: $(System.DefaultWorkingDirectory)/loadTest
     artifact: loadTestResults
   ```

1. Si la mise en retrait de l’extrait de code YAML donne des erreurs (lignes rouges ondulées), corrigez-les en ajoutant 2 espaces ou une tabulation pour positionner correctement l’extrait de code.
1. Une fois les deux extraits de code ajoutés au pipeline CI/CD, **enregistrez** les modifications.
1. Une fois enregistrées, cliquez sur **Exécuter** pour déclencher le pipeline.
1. Confirmez la branche (principale) et cliquez sur le bouton **Exécuter** pour démarrer l’exécution du pipeline.
1. Sur la page d’état du pipeline, cliquez sur l’étape **Build** pour ouvrir les détails complets de journalisation des différentes tâches du pipeline.
1. Attendez que le pipeline démarre l’étape de build et arrive à la tâche **AzureLoadTest** dans son flux.
1. Pendant l’exécution de la tâche, accédez au **test de charge Azure** dans le portail Azure et découvrez comment le pipeline crée un RunTest nommé **adoloadtest1**. Vous pouvez le sélectionner pour afficher les valeurs de résultat du travail TestRun.
1. Revenez à la vue Exécution du pipeline CI/CD Azure DevOps, où la **tâche AzureLoadTest** a réussi. Les valeurs résultant du test de charge sont également visibles dans la sortie de journalisation détaillée :

   ```text
   Task         : Azure Load Testing
   Description  : Automate performance regression testing with Azure Load Testing
   Version      : 1.2.30
   Author       : Microsoft Corporation
   Help         : https://docs.microsoft.com/azure/load-testing/tutorial-cicd-azure-pipelines#azure-load-testing-task
   ==============================================================================
   Test '0d295119-12d0-482d-94be-a7b84787c004' already exists
   Uploaded test plan for the test
   Creating and running a testRun for the test
   View the load test run in progress at: https://portal.azure.com/#blade/Microsoft_Azure_CloudNativeTesting/NewReport//resourceId/%2fsubscriptions%4b75-a1e0-27fb2ea7f9f4%2fresourcegroups%2faz400m08l14-RG%2fproviders%2fmicrosoft.loadtestservice%2floadtests%2feshoponwebloadtesting/testId/0d295119-12d0-787c004/testRunId/161046f1-d2d3-46f7-9d2b-c8a09478ce4c
   TestRun completed

   -------------------Summary ---------------
   TestRun start time: Mon Jul 24 2023 21:46:26 GMT+0000 (Coordinated Universal Time)
   TestRun end time: Mon Jul 24 2023 21:51:50 GMT+0000 (Coordinated Universal Time)
   Virtual Users: 50
   TestStatus: DONE

   ------------------Client-side metrics------------

   Homepage
   response time         : avg=1359ms min=59ms med=539ms max=16629ms p(90)=3127ms p(95)=5478ms p(99)=13878ms
   requests per sec      : avg=37
   total requests        : 4500
   total errors          : 0
   total error rate      : 0
   Finishing: AzureLoadTest

   ```

1. Vous venez d’effectuer un test de charge automatisé dans le cadre d’une exécution de pipeline. Dans dernière tâche, vous allez spécifier des conditions d’échec. Cela signifie que l’étape de déploiement ne sera pas autorisée à démarrer si les performances de l’application web n’atteignent pas un certain seuil.

#### Tâche 5 : ajouter des critères d’échec/de réussite au pipeline de test de charge

Dans cette tâche, vous allez utiliser des critères d’échec de test de charge pour obtenir des alertes (résultat d’échec de l’exécution d’un pipeline) lorsque l’application ne répond pas à vos exigences de qualité.

1. Dans Azure DevOps, accédez au projet eShopOnWeb et ouvrez **Repos**.
1. Dans Repos, accédez au sous-dossier **/tests/jmeter** créé et utilisé précédemment.
1. Ouvrez le fichier de test de charge \*config.yaml*. Cliquez sur **Modifier\*\* pour autoriser la modification du fichier.
1. Remplacez `failureCriteria: []` si la valeur est présente, sinon ajoutez l’extrait de code suivant :

   ```text
   failureCriteria:
     - avg(response_time_ms) > 300
     - percentage(error) > 50
   ```

1. Enregistrez les modifications apportées à config.yaml en cliquant sur **Valider** à deux reprises.
1. Revenez à **Pipelines** et réexécutez le pipeline **eShopOnWeb**. Après quelques minutes, l’exécution se termine avec un état d’**échec** pour la tâche **AzureLoadTest**.
1. Ouvrez la vue de journalisation détaillée du pipeline et validez les détails d’**AzureLoadtest**. Un exemple de sortie similaire est présenté ci-dessous :

   ```text
   Creating and running a testRun for the test
   View the load test run in progress at: https://portal.azure.com/#blade/Microsoft_Azure_CloudNativeTesting/NewReport//resourceId/%2fsubscriptions%2fb86d9ae1-7552-47fb2ea7f9f4%2fresourcegroups%2faz400m08l14-RG%2fproviders%2fmicrosoft.loadtestservice%2floadtests%2feshoponwebloadtesting/testId/0d295119-12d0-a7b84787c004/testRunId/f4bec76a-8b49-44ee-a388-12af34f0d4ec
   TestRun completed

   -------------------Summary ---------------
   TestRun start time: Mon Jul 24 2023 23:00:31 GMT+0000 (Coordinated Universal Time)
   TestRun end time: Mon Jul 24 2023 23:06:02 GMT+0000 (Coordinated Universal Time)
   Virtual Users: 50
   TestStatus: DONE

   -------------------Test Criteria ---------------
   Results           :1 Pass 1 Fail

   Criteria                  :Actual Value        Result
   avg(response_time_ms) > 300                       1355.29               FAILED
   percentage(error) > 50                                                  PASSED


   ------------------Client-side metrics------------

   Homepage
   response time         : avg=1355ms min=58ms med=666ms max=16524ms p(90)=2472ms p(95)=5819ms p(99)=13657ms
   requests per sec      : avg=37
   total requests        : 4531
   total errors          : 0
   total error rate      : 0
   ##[error]TestResult: FAILED
   Finishing: AzureLoadTest

   ```

1. Notez que la dernière ligne de la sortie de test de charge indique **##[error]TestResult: FAILED** puisque nous avons défini un **FailCriteria** avec un temps de réponse moyen de > 300 ou avec un pourcentage d’erreur de > 20. Étant donné que nous voyons maintenant un temps de réponse moyen supérieur à 300, la tâche est marquée comme ayant échoué.

   > **Note** : imaginez-vous dans un scénario réel, vous allez valider les performances de votre App Service et, si les performances sont inférieures à un certain seuil, ce qui signifie généralement qu’il y a plus de charge sur l’application web, vous pouvez déclencher un nouveau déploiement vers un service Azure App Service supplémentaire. Comme nous ne pouvons pas contrôler le temps de réponse pour les environnements de labo Azure, nous avons décidé de rétablir la logique pour assurer l’échec.

1. L’état FAILED de la tâche de pipeline reflète en fait une validation réussie (SUCCESS) des critères d’exigences de Test de charge Azure.

   > [!IMPORTANT]
   > N’oubliez pas de supprimer les ressources créées dans le portail Azure pour éviter les frais inutiles.

## Révision

Dans cet exercice, vous avez déployé une application web sur Azure App Service à l’aide d’Azure Pipelines, ainsi qu’une ressource Test de charge Azure avec des TestRuns. Ensuite, vous avez intégré le fichier config.yaml de test de charge JMeter au contrôle de code source Azure Repos et vous avez étendu votre pipeline CI/CD avec Test de charge Azure. Dans le dernier exercice, vous avez appris à définir les critères de réussite de LoadTest.
