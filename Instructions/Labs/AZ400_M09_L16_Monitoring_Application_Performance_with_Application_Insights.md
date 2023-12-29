---
lab:
  title: Surveiller les performances des applications avec le test de charge Azure
  module: 'Module 09: Implement continuous feedback'
---

# Surveiller les performances des applications avec Application Insights et le test de charge Azure

## Manuel de labo de l’étudiant

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Créer une organisation ou une collection de projets](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifier un abonnement Azure existant ou en créer un.

- Vérifiez que vous disposez d’un compte Microsoft ou d’un compte Microsoft Entra avec le rôle Propriétaire dans l’abonnement Azure et le rôle Administrateur général dans le locataire Microsoft Entra associé à l’abonnement Azure. Pour plus d’informations, consultez [Répertorier les attributions de rôle Azure à l’aide du portail Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) et [Afficher et attribuer des rôles d’administrateur dans Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal#view-my-roles).

## Présentation du labo

**Test de charge Azure** est un service complètement managé qui permet de générer une charge à grande échelle. Il simule un trafic pour vos applications, quel que soit l’endroit où elles sont hébergées. Les développeurs, testeurs et ingénieurs d’assurance qualité peuvent l’utiliser pour optimiser les performances, la scalabilité ou la capacité d’une application.
Créez rapidement un test de charge pour votre application web à l’aide d’une URL, sans connaissance préalable des outils de test. Test de charge Azure réduit la complexité et l’infrastructure pour exécuter votre test de charge à grande échelle.
Pour des scénarios de test de charge plus avancés, vous pouvez créer un test de charge en réutilisant un script de test Apache JMeter existant, un outil de charge et d’évaluation des performances open source populaire. Par exemple, votre plan de test peut se composer de plusieurs requêtes d’application, vous souhaitez appeler des points de terminaison non-HTTP, ou vous utilisez des données et paramètres d’entrée pour rendre le test plus dynamique.

Dans le cadre de ce labo, vous découvrirez comment utiliser le test de charge Azure pour simuler des tests de performances sur une application web en direct avec différents scénarios de charge. Enfin, vous apprendrez à intégrer le test de charge Azure dans vos pipelines CI/CD. 

## Objectifs

À la fin de ce labo, vous serez en mesure d’accomplir les tâches suivantes :

- Déployer des applications web Azure App Service.
- Composer et exécuter un pipeline CI/CD basé sur YAML.
- Déployer le test de charge Azure.
- Examiner les performances des applications web Azure à l’aide du test de charge Azure.
- Intégrer le test de charge Azure dans vos pipelines CI/CD.

## Durée estimée : 60 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

Dans cet exercice, vous configurerez les prérequis du labo, qui se composent d’un nouveau projet Azure DevOps comprenant un référentiel basé sur [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tâche 1 : (à ignorer si vous l’avez déjà effectuée) créer et configurer le projet d’équipe

Dans le cadre de cette tâche, vous créerez un projet Azure DevOps **eShopOnWeb** qui sera utilisé dans plusieurs labos.

1. Sur votre ordinateur de labo, ouvrez votre organisation Azure DevOps dans votre navigateur. Cliquez sur **Nouveau projet**. Nommez votre projet **eShopOnWeb** et sélectionnez **Scrum** dans la liste déroulante **Processus d’élément de travail**. Cliquez sur **Créer**.

    ![Création d’un projet](images/create-project.png)

#### Tâche 2 : (à ignorer si vous l’avez déjà effectuée) importer un référentiel Git eShopOnWeb

Dans le cadre de cette tâche, vous importerez le référentiel Git eShopOnWeb qui sera utilisé dans plusieurs labos.

1. Sur votre ordinateur de labo, ouvrez votre organisation Azure DevOps dans votre navigateur ainsi que le projet **eShopOnWeb** que vous venez de créer. Cliquez sur **Repos > Fichiers**, puis sur **Importer**. Dans la fenêtre **Importer un référentiel Git**, collez l’URL https://github.com/MicrosoftLearning/eShopOnWeb.git, puis cliquez sur **Importer** :

    ![Importer un référentiel](images/import-repo.png)

2. Le référentiel est organisé de la manière suivante :
    - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
    - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
    - Le dossier **.azure** contient l’infrastructure Bicep&ARM en tant que modèles de code utilisés dans certains scénarios de labos.
    - **Définitions de workflow GitHub YAML du conteneur de dossiers .github**.
    - Le dossier **src** contient le site web .NET 7 utilisé dans les scénarios de labo.

#### Tâche 2 : Créez des ressources Azure

Dans cette tâche, vous créerez une application web Azure à l’aide du Portail Azure.

1. Sur l’ordinateur de labo, démarrez un navigateur web, accédez au [**Portail Azure**](https://portal.azure.com) et connectez-vous avec le compte d’utilisateur qui a le rôle de Propriétaire dans l’abonnement Azure que vous utiliserez dans ce labo et qui a le rôle d’Administrateur général dans le locataire Microsoft Entra associé à cet abonnement.
2. Dans la barre d’outils du Portail Azure, sélectionnez l’icône **Cloud Shell** située directement à droite de la zone de texte de recherche.
3. Si vous êtes invité à sélectionner **Bash** ou **PowerShell**, sélectionnez **Bash**.
    >**Remarque** : si c’est la première fois que vous démarrez **Cloud Shell** et que vous voyez le message **Vous n’avez aucun stockage monté**, sélectionnez l’abonnement que vous utilisez dans ce labo, puis sélectionnez **Créer un stockage**.

4. Dans l’invite **Bash** du volet **Cloud Shell**, exécutez la commande suivante pour créer un groupe de ressources (remplacez l’espace réservé `<region>` par le nom de la région Azure la plus proche de vous, par exemple « eastus »).

    ```bash
    RESOURCEGROUPNAME='az400m09l16-RG'
    LOCATION='<region>'
    az group create --name $RESOURCEGROUPNAME --location $LOCATION
    ```

5. Exécutez la commande suivante pour créer un plan Windows App Service :

    ```bash
    SERVICEPLANNAME='az400l16-sp'
    az appservice plan create --resource-group $RESOURCEGROUPNAME \
        --name $SERVICEPLANNAME --sku B3 
    ```

6. Créez une application web avec un nom unique.

    ```bash
    WEBAPPNAME=partsunlimited$RANDOM$RANDOM
    az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME 
    ```

    > **Remarque** : enregistrez le nom de l’application web. Vous en aurez besoin plus tard dans ce labo.

### Exercice 1 : configurer des pipelines CI/CD en tant que code avec YAML dans Azure DevOps

Dans le cadre de cet exercice, vous configurerez de pipelines CI/CD en tant que code avec YAML dans Azure DevOps.

#### Tâche 1 : ajouter un build YAML et déployer la définition

Dans le cadre de cette tâche, vous ajouterez une définition de build YAML au projet existant.

1. Revenez au volet **Pipelines** du hub **Pipelines**.
2. Dans la fenêtre **Créer votre premier pipeline**, cliquez sur **Créer un pipeline**.

    > **Remarque** : vous utiliserez l’assistant pour créer une définition de pipeline YAML basée sur votre projet.

3. Dans le volet **Où se trouve votre code ?**, sélectionnez l’option **Azure Repos Git (YAML)**.
4. Dans le volet **Sélectionner un référentiel**, cliquez sur **eShopOnWeb**.
5. Faites défiler le volet **Configurer votre pipeline** et sélectionnez **Pipeline de démarrage**.
6. **Sélectionnez** toutes les lignes du pipeline de démarrage, puis supprimez-les.
7. **Copiez** le pipeline de modèle complet ci-dessous. N’oubliez pas que vous devrez modifier les paramètres **avant d’enregistrer** les modifications :

```
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
        command: 'restore'
        projects: '**/*.sln'
        feedsToUse: 'select'

    - task: DotNetCoreCLI@2
      displayName: Build
      inputs:
        command: 'build'
        projects: '**/*.sln'
    
    - task: DotNetCoreCLI@2
      displayName: Publish
      inputs:
        command: 'publish'
        publishWebProjects: true
        arguments: '-o $(Build.ArtifactStagingDirectory)'
    
    - task: PublishBuildArtifacts@1
      displayName: Publish Artifacts ADO - Website
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: Website
    
- stage: Deploy
  displayName: Deploy to an Azure Web App
  jobs:
  - job: Deploy
    pool:
      vmImage: 'windows-2019'
    steps:
    - task: DownloadBuildArtifacts@1
      inputs:
        buildType: 'current'
        downloadType: 'single'
        artifactName: 'Website'
        downloadPath: '$(Build.ArtifactStagingDirectory)'

```
4. Placez le curseur sur une nouvelle ligne à la fin de la définition YAML(ligne 69).

    > **Remarque** : il s’agit de l’emplacement où les nouvelles tâches seront ajoutées.

5. Dans la liste des tâches situées à droite du volet de code, recherchez et sélectionnez la tâche **Déployer Azure App Service**.
6. Dans le volet **Déployer Azure App Service**, spécifiez les paramètres suivants, puis cliquez sur **Ajouter** :

    - Dans la liste déroulante **Abonnement Azure**, sélectionnez l’abonnement Azure où vous avez déployé les ressources Azure que vous avez créées dans le cadre de ce labo. Cliquez ensuite sur **Autoriser** et, lorsque vous y êtes invité, authentifiez-vous à l’aide du même compte d’utilisateur que celui que vous avez utilisé pendant le déploiement des ressources Azure.
    - Dans la liste déroulante **Nom App Service**, sélectionnez le nom de l’application web que vous avez déployée dans le cadre de ce labo.
    - Dans la zone de texte **Package ou dossier**, **passez** la valeur par défaut à `$(Build.ArtifactStagingDirectory)/**/Web.zip`.
7. Confirmez les paramètres en cliquant sur le bouton **Ajouter** du volet Assistant.

    > **Remarque** : cette opération ajoute automatiquement la tâche de déploiement à la définition du pipeline YAML.

8. L’extrait de code ajouté à l’éditeur doit ressembler à l’exemple ci-dessous et doit refléter votre nom pour les paramètres azureSubscription et WebappName :

> **Remarque** : le paramètre **packageForLinux** est trompeur dans le contexte de ce labo, mais il est correct pour Windows ou Linux.

    ```yaml
        - task: AzureRmWebAppDeployment@4
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: 'AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)'
            appType: 'webApp'
            WebAppName: 'eshoponWebYAML369825031'
            packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
    ```
9. Cliquez sur **Enregistrer**. Dans le volet **Enregistrer**, cliquez à nouveau sur **Enregistrer** pour valider la modification directement dans la branche principale.

    > **Remarque** : puisque le CI-YAML d’origine n’a pas été configuré de manière à déclencher automatiquement un nouveau build, celui-ci doit être lancé manuellement.

10. Dans le menu gauche d’Azure DevOps, accédez à **Pipelines** et sélectionnez une nouvelle fois **Pipelines**.
11. Ouvrez le pipeline **EShopOnWeb_MultiStageYAML** et cliquez sur **Exécuter le pipeline**.
12. Confirmez l’**exécution** dans le volet qui apparaît.
13. Deux étapes différentes, **Générer la solution .Net Core** et **Déployer sur l’application web Azure** s’affichent.
14. Attendez le lancement du pipeline, puis la fin de l’étape de génération.
15. Dès que l’étape de déploiement démarre, vous êtes invité à renseigner les **autorisations nécessaires**. Une barre orange apparaît à l’écran et indique :

    ```text
    This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
    ```

16. Cliquez sur **Vue**.
17. Dans le volet **En attente de révision**, cliquez sur **Autoriser**.
18. Validez le message dans la fenêtre contextuelle **Autoriser**, puis confirmez votre choix en cliquant sur **Autoriser**.
19. Cela lance l’étape de déploiement. Attendez la fin de ce processus.

#### Tâche 2 : passer en revue le site déployé

1. Revenez à la fenêtre du navigateur web où est ouvert le Portail Azure et accédez au volet affichant les propriétés de l’application web Azure.
2. Dans le volet Application web Azure, cliquez sur **Vue d’ensemble**. Dans le volet Vue d’ensemble, cliquez sur **Parcourir** pour ouvrir votre site dans un nouvel onglet du navigateur web.
3. Vérifiez que le site déployé se charge comme prévu dans le nouvel onglet du navigateur et que le site web d’e-commerce EShopOnWeb s’affiche.

### Exercice 2 : déployer et configurer le test de charge Azure

Dans le cadre de cet exercice, vous déploierez une ressource de test de charge Azure dans Azure et configurerez différents scénarios de test de charge pour votre Azure App Service en direct.

#### Tâche 1 : déployer le test de charge Azure

Dans le cadre de cette tâche, vous déploierez une nouvelle ressource de test de charge Azure dans votre abonnement Azure.

1. Depuis le Portail Azure (https://portal.azure.com)), accédez à **Créer une ressource Azure**.
2. Dans le champ de recherche « Service Search et Place de marché », saisissez **Test de charge Azure**.
3. Dans les résultats de la recherche, sélectionnez **Test de charge Azure** (publié par Microsoft).
4. Sur la page Test de charge Azure, cliquez sur **Créer** pour démarrer le processus de déploiement.
5. Sur la page « Créer une ressource de test de charge », indiquez les renseignements nécessaires au déploiement des ressources :
- **Abonnement** : sélectionnez votre abonnement Azure.
- **Groupe de ressources** : sélectionnez le groupe de ressources que vous avez utilisé pour déployer l’application de service web lors de l’exercice précédent.
- **Nom** : EShopOnWebLoadTesting
- **Région** : sélectionnez une région proche de la vôtre.

    > **Remarque** : le service de test de charge Azure n’est pas disponible dans toutes les régions Azure.

6. Cliquez sur **Vérifier et créer** pour valider vos paramètres.
7. Cliquez sur **Créer** pour confirmer et déployer la ressource de test de charge Azure.
8. Vous êtes redirigé vers la page « Déploiement en cours ». Patientez quelques minutes jusqu’à la fin di déploiement.
9. Sur la page de progression du déploiement, cliquez sur **Accéder à la ressource** pour accéder à la ressource de test de charge Azure **EShopOnWebLoadTesting**. 

    > **Remarque** : si vous avez fermé le volet ou fermé le Portail Azure pendant le déploiement de la ressource de test de charge Azure, vous pouvez la retrouver à partir du champ Recherche du Portail Azure ou dans la liste des Ressources/Récent. 

#### Tâche 2 : créer des tests de charge Azure

Dans le cadre de cette tâche, vous créerez différents tests de charge Azure à l’aide de plusieurs paramètres de configuration de charge. 

10. Dans le volet des ressource de test de charge Azure d’**EShopOnWebLoadTesting**, vous verrez l’option **Démarrer avec un test rapide**. Cliquez sur le bouton **Test rapide**.
11. Renseignez les paramètres suivants pour créer un test de charge :
- **URL de test** : saisissez l’URL de l’Azure App Service que vous avez déployé lors de l’exercice précédent (EShopOnWeb…azurewebsites.net), **en incluant https://**.
- **Spécifier la charge** : utilisateurs virtuels
- **Nombre d’utilisateurs virtuels** : 50
- **Durée du test (en secondes)**  : 120
- **Durée de l’accélération (en secondes)**  : 0
12. Confirmez la configuration du test en cliquant sur **Exécuter le test**.
13. Le test prendra environ 2 minutes. 
14. Une fois le test en cours d’exécution, revenez à la page des ressources de test de charge Azure **EShopOnWebLoadLoadTesting** et accédez à **Tests**. Sélectionnez **Tests** : vous verrez le test **Get_eshoponweb…**
15. Dans le menu supérieur, cliquez sur **Créer**, puis sur **Créer un test rapide** afin de créer un deuxième test de charge.
16. Renseignez les paramètres suivants pour créer un autre test de charge :
- **URL de test** : saisissez l’URL de l’Azure App Service que vous avez déployé lors de l’exercice précédent (EShopOnWeb…azurewebsites.net), **en incluant https://**.
- **Spécifier la charge** : demandes par seconde (RPS)
- **Demandes par seconde (RPS)**  : 100
- **Délai de réponse (en millisecondes)**  : 500
- **Durée du test (en secondes)**  : 120
- **Durée de l’accélération (en secondes)**  : 0
17. Confirmez la configuration du test en cliquant sur **Exécuter le test**.
18. Le test prendra environ 2 minutes.

#### Tâche 3 : valider les résultats des tests de charge Azure

Dans le cadre de cette tâche, vous validerez le résultat d’un test de charge Azure TestRun. 

Une fois les deux tests rapides terminés, vous y apporterez certaines modifications et en validerez les résultats.

19. Dans le volet des ressources d’**EShopOnWebLoadTesting**, accédez à la section **Tests**, puis sélectionnez le premier test Get_eshoponwebyaml… Cliquez sur **Modifier** dans le menu supérieur.
20. Ici, le portail vous permet de remplacer le **nom du test** généré par défaut par un nom plus descriptif. Il vous permet également d’apporter des modifications à l’un des paramètres définis précédemment.
21. Ouvrez l’onglet **Plan du test** du volet **Modifier le test**. 
22. C’est là que vous pouvez gérer le fichier de script du test de charge **Apache JMeter**, qui est le fichier utilisé par Azure Load Testing en tant que cadre. Vous verrez un fichier nommé **quick_test.jmx**. Sélectionnez-le pour l’**ouvrir** sur l’ordinateur virtuel du labo. Dans la fenêtre contextuelle, sélectionnez **Visual Studio Code** comme éditeur pour ouvrir le fichier.
23. Examinez la structure de langage XML du fichier.

    > Remarque : pour obtenir davantage d’informations et mieux comprendre la syntaxe plus avancée d’Apache JMeter, cliquez sur le lien [Test de charge Azure : Jmeter](https://learn.microsoft.com/en-us/azure/load-testing/how-to-create-and-run-load-test-with-jmeter-script) suivant.

24. Revenez à la vue **Tests** pour voir les deux tests. **Cliquez** sur l’un d’eux pour ouvrir une vue plus détaillée. Vous êtes alors redirigé vers la page de test comprenant davantage de détails. À partir de là, vous pouvez valider les détails des exécutions effectuées en sélectionnant **TestRun_mm/dd/yy-hh:hh** dans la liste qui apparaît.
25. Sur la page ** TestRun** détaillée, identifiez le résultat réel de la simulation de test de charge Azure. Les valeurs suivantes apparaissent :
- Charge/Total des demandes
- Durée
- Temps de réponse (indique un résultat en secondes qui représente le 90e temps de réponse du centile, ce qui signifie que pour 90 % des demandes, le temps de réponse est compris dans les résultats donnés)
- Débit en nombre de requêtes par seconde
26. D’autres valeurs indiquées ci-dessous. Plusieurs d’entre elles sont représentées à l’aide des vues de graphiques et graphiques linéaires du tableau de bord.
27. Prenez le temps de **comparer les résultats** des deux tests simulés et **identifier les répercussions** d’un plus grand nombre d’utilisateurs sur les performances d’App Service.

### Exercice 2 : automatiser un test de charge avec CI/CD dans les pipelines Azure DevOps

Commencez par automatiser les tests de charge dans le Test de charge Azure en l’ajoutant à un pipeline CI/CD. Après avoir exécuté un test de charge dans le Portail Azure, vous exportez les fichiers de configuration et configurez un pipeline CI/CD dans Azure Pipelines (des fonctionnalités similaires existent pour GitHub Actions).

À la fin de cet exercice, vous disposez d’un workflow CI/CD configuré de manière à exécuter un test de charge avec le test de charge Azure.

#### Tâche 1 : identifier les détails de la connexion de services ADO

Dans le cadre de cette tâche, vous accorderez les autorisations requises au principal de service de la connexion de services Azure DevOps.

1. Dans le **Portail Azure DevOps** (https://dev.azure.com)), accédez au projet **EShopOnWeb**.
2. Sélectionnez **Paramètres du projet** dans le coin inférieur gauche.
3. Dans la section **Pipelines**, sélectionnez **Connexions de services**.
4. La connexion de services porte le nom de votre abonnement Azure que vous avez utilisé pour déployer les ressources Azure au début de l’exercice de labo.
5. **Sélectionnez la connexion de services**. Dans l’onglet **Vue d’ensemble**, accédez aux **détails** et sélectionnez **Gérer le principal de service**.
6. Vous êtes alors redirigé vers le Portail Azure, qui ouvre les détails du **principal de service** pour l’objet d’identité.
7. Copiez la valeur **Nom d’affichage** (au format Name_of_ADO_Organization_EShopOnWeb_-b86d9ae1-7552-4b75-a1e0-27fb2ea7f9f4) et mettez-la de côté : vous en aurez besoin lors des étapes suivantes.

#### Tâche 2 : accorder des autorisations au principal de service

Le Test de charge Azure utilise le contrôle d’accès en fonction du rôle (RBAC) Azure pour accorder des autorisations afin d’effectuer des activités spécifiques sur votre ressource de test de charge. Pour exécuter un test de charge à partir de votre pipeline CI/CD, vous accordez le rôle de **Contributeur au test de charge** au principal de service.

1. Dans le **Portail Azure**, accédez à votre ressource de **test de charge Azure**.
2. Sélectionnez **Contrôle d’accès (IAM)**  > Ajouter > Ajouter une attribution de rôle.
3. Sous l’onglet **Rôle**, sélectionnez **Contributeur au test de charge** dans la liste des rôles de fonction de travail.
4. Dans l’**onglet Membres**, cliquez sur **Sélectionner des membres**, puis utilisez le **nom d’affichage** que vous venez de copier pour rechercher le principal de service.
5. Sélectionnez le **principal de service**, puis cliquez sur **Sélectionner**.
6. Dans l’onglet **Passer en revue + attribuer**, sélectionnez **Passer en revue + attribuer** pour ajouter l’attribution de rôle.

Vous pouvez maintenant utiliser la connexion de service dans votre définition de workflow Azure Pipelines pour accéder à votre ressource de test de charge Azure.

#### Tâche 3 : exporter des fichiers d’entrée de test de charge et les importer dans le contrôle de code source d’Azure DevOps

Pour exécuter un test de charge avec le Test de charge Azure dans un workflow CI/CD, vous devez ajouter les paramètres de configuration de test de charge et tous les fichiers d’entrée dans votre référentiel de contrôle de code source. Si vous disposez d’un test de charge existant, vous pouvez télécharger les paramètres de configuration et tous les fichiers d’entrée à partir du portail Azure.

Effectuez les étapes suivantes pour télécharger les fichiers d’entrée d’un test de charge existant dans le Portail Azure :

1. Dans le **Portail Azure**, accédez à votre ressource de **test de charge Azure**.
2. Dans le volet de gauche, sélectionnez **Tests** pour afficher la liste des tests de charge, puis sélectionnez **votre test**.
3. Sélectionnez les points de suspension (**...**) en regard de la série de tests de charge sur laquelle vous travaillez, puis sélectionnez **Télécharger le fichier d’entrée**.
4. Le navigateur télécharge un dossier compressé qui contient les fichiers d’entrée de test de charge.
5. Utilisez n’importe quel outil de compression pour extraire les fichiers d’entrée. Ce dossier contient les fichiers suivants :

- *config.yaml* : le fichier de configuration YAML du test de charge. Vous référencez ce fichier dans la définition du workflow CI/CD.
- *quick_test.jmx* : le script de test JMeter.

6. Commitez tous les fichiers d’entrée extraits vers votre référentiel de contrôle de code source. Pour ce faire, accédez au **Portail Azure DevOps** (https://dev.azure.com)), puis ouvrez le projet DevOps **EShopOnWeb**. 
7. Sélectionnez **Repos**. Dans la structure du dossier du code source, vous verrez le sous-dossier des **tests**. Vous verrez des points de suspension (…) qui apparaissent à l’écran. Sélectionnez ensuite **Nouveau > Dossier**.
8. Nommez le dossier **jmeter** et nommez le fichier **placeholder.txt** (remarque : le dossier ne peut pas être créé comme étant vide).
9. Cliquez sur **Valider** pour confirmer la création du fichier placeholder et du dossier jmeter.
10. Dans la **structure de dossiers**, accédez au sous-dossier **jmeter** que vous venez de créer. Cliquez sur les **points de suspension** (…) et sélectionnez **Charger des fichiers**.
11. Utilisez l’option **Parcourir** pour accéder à l’emplacement du fichier zip extrait, puis sélectionnez **config.yaml** et **quick_test.jmx**.
12. Cliquez sur **Valider** pour confirmer le chargement du fichier dans le contrôle de code source.

#### Tâche 4 : mettre à jour le fichier de définition YAML du workflow CI/CD

Dans le cadre de cette tâche, vous importerez l’extension Test de charge Azure : Place de marché Azure DevOps, et mettrez à jour le pipeline CI/CD existant avec la tâche AzureLoadTest.

1. Pour créer et exécuter un test de charge, la définition du workflow Azure Pipelines utilise l’extension de **tâche Test de charge Azure** disponible dans la Place de marché Azure DevOps. Ouvrez l’[extension de tâche Test de charge Azure](https://marketplace.visualstudio.com/items?itemName=AzloadTest.AzloadTesting) dans la Place de marché Azure DevOps, puis sélectionnez **Obtenir gratuitement**.
2. Sélectionnez votre organisation Azure DevOps, puis sélectionnez **Installer** pour installer l’extension.
3. Dans le projet ouvert sur le Portail Azure DevOps, accédez aux **pipelines** et sélectionnez le pipeline créé au début de cet exercice. Cliquez sur **Modifier**.
4. Dans le script YAML, accédez à la **ligne 56** et appuyez sur les touches ENTRÉE/RETOUR pour ajouter une nouvelle ligne vide. (Juste avant l’étape de déploiement du fichier YAML.)
5. À la ligne 57, sélectionnez l’Assistant des tâches sur le côté droit et recherchez **Test de charge Azure**.
6. Renseignez les paramètres adéquats de votre scénario dans le volet graphique :
- Abonnement Azure (1) : sélectionnez l’abonnement où sont exécutées vos ressources Azure.
- Fichier du test de charge : '$(Build.SourcesDirectory)/tests/jmeter/config.yaml' 
- Groupe de ressources du test de charge : le groupe de ressources qui contient vos ressources de test de charge Azure.
- Nom de la ressource de test de charge : ESHopOnWebLoadTesting
- Nom d’exécution du test de charge : ado_run
- Description de l’exécution du test de charge : test de charge à partir d’ADO
7. Confirmer l’injection des paramètres en tant qu’extrait de code YAML en cliquant sur **Ajouter**.
8. Si la mise en retrait de l’extrait de code YAML génère des erreurs (lignes rouges ondulées), corrigez-les en ajoutant 2 espaces ou une tabulation pour placer correctement l’extrait de code.  
9. Votre code YAML doit ressembler à l’exemple d’extrait de code ci-dessous :
```
     - task: AzureLoadTest@1
      inputs:
        azureSubscription: 'AZURE DEMO SUBSCRIPTION(b86d9ae1-1234-4b75-a8e7-27fb2ea7f9f4)'
        loadTestConfigFile: '$(Build.SourcesDirectory)/tests/jmeter/config.yaml'
        resourceGroup: 'az400m05l11-RG'
        loadTestResource: 'EShopOnWebLoadTesting'
        loadTestRunName: 'ado_run'
        loadTestRunDescription: 'load testing from ADO'
```
10. Sous l’extrait de code YAML inséré, ajoutez une nouvelle ligne vide en appuyant sur les touches ENTRÉE/RETOUR. 
11. Sous cette ligne vide, ajoutez un extrait de code pour la tâche Publier qui indique les résultats de la tâche de test de charge Azure pendant l’exécution du pipeline :

```
    - publish: $(System.DefaultWorkingDirectory)/loadTest
      artifact: loadTestResults
```
12.  Si la mise en retrait de l’extrait de code YAML génère des erreurs (lignes rouges ondulées), corrigez-les en ajoutant 2 espaces ou une tabulation pour placer correctement l’extrait de code.  
13. Une fois les deux extraits de code ajoutés au pipeline CI/CD, **enregistrez** les modifications. 
14. Une fois les modifications enregistrées, cliquez sur **Exécuter** pour déclencher le pipeline.
15. Confirmez la branche (principale) et cliquez sur le bouton **Exécuter** pour lancer l’exécution du pipeline.
16. Sur la page d’état du pipeline, cliquez sur l’étape **Génération** pour ouvrir les détails complets de journalisation des différentes tâches du pipeline.
17. Attendez que le pipeline démarre l’étape de génération et arrive à la tâche **AzureLoadTest** dans son flux. 
18. Pendant l’exécution de la tâche, accédez au **test de charge Azure** dans le Portail Azure pour observer comment le pipeline crée un RunTest nommé **adoloadtest1**. Vous pouvez le sélectionner pour afficher les valeurs de résultat du travail TestRun.
19. Revenez à la vue Exécution du pipeline CI/CD Azure DevOps : la **tâche AzureLoadTest** a réussi. Les valeurs résultant du test de charge sont également visibles dans la sortie de journalisation détaillée :

```
Task         : Azure Load Testing
Description  : Automate performance regression testing with Azure Load Testing
Version      : 1.2.30
Author       : Microsoft Corporation
Help         : https://docs.microsoft.com/azure/load-testing/tutorial-cicd-azure-pipelines#azure-load-testing-task
==============================================================================
Test '0d295119-12d0-482d-94be-a7b84787c004' already exists
Uploaded test plan for the test
Creating and running a testRun for the test
View the load test run in progress at: https://portal.azure.com/#blade/Microsoft_Azure_CloudNativeTesting/NewReport//resourceId/%2fsubscriptions%4b75-a1e0-27fb2ea7f9f4%2fresourcegroups%2faz400m05l11-rg%2fproviders%2fmicrosoft.loadtestservice%2floadtests%2feshoponwebloadtesting/testId/0d295119-12d0-787c004/testRunId/161046f1-d2d3-46f7-9d2b-c8a09478ce4c
TestRun completed

-------------------Summary ---------------
TestRun start time: Mon Jul 24 2023 21:46:26 GMT+0000 (Coordinated Universal Time)
TestRun end time: Mon Jul 24 2023 21:51:50 GMT+0000 (Coordinated Universal Time)
Virtual Users: 50
TestStatus: DONE

------------------Client-side metrics------------

Homepage
response time        : avg=1359ms min=59ms med=539ms max=16629ms p(90)=3127ms p(95)=5478ms p(99)=13878ms
requests per sec     : avg=37
total requests       : 4500
total errors         : 0
total error rate     : 0
Finishing: AzureLoadTest

```
20. Vous venez d’effectuer un test de charge automatisé dans le cadre d’une exécution de pipeline. Dans le cadre de la dernière tâche, vous spécifierez des conditions d’échec. Cela signifie que l’étape de déploiement ne sera pas autorisée à démarrer si les performances de l’application web n’atteignent pas un certain seuil. 

#### Tâche 5 : ajouter des critères d’échec/de réussite au pipeline de test de charge

Dans le cadre de cette tâche, vous utiliserez des critères d’échec de test de charge pour être prévenu (et obtenir un échec d’exécution de pipeline en résultat) si l’application ne répond pas à vos exigences de qualité.

1. Dans Azure DevOps, accédez au projet EShopOnWeb et ouvrez **Repos**.
2. Dans Repos, ouvrez le sous-dossier **/tests/jmeter** que vous venez de créer et d’utiliser.
3. Ouvrez le fichier de test de charge *config.yaml**. Cliquez sur **Modifier** pour autoriser la modification du fichier.
4. À la fin du fichier, ajoutez cet extrait de code :

```
failureCriteria:
  - avg(response_time_ms) > 300
  - percentage(error) > 50
```
5. Enregistrez les modifications apportées à config.yaml en cliquant deux fois sur **Valider**.
6. Revenez aux **pipelines** et exécutez une nouvelle fois le pipeline **EShopOnWeb**. Après quelques minutes, l’exécution prend fin et affiche l’état **Échec** pour la tâche **AzureLoadTest**. 
7. Ouvrez la vue de journalisation détaillée du pipeline et validez les détails d’**AzureLoadtest**. Votre sortie ressemble à l’exemple ci-dessous :

```
Creating and running a testRun for the test
View the load test run in progress at: https://portal.azure.com/#blade/Microsoft_Azure_CloudNativeTesting/NewReport//resourceId/%2fsubscriptions%2fb86d9ae1-7552-47fb2ea7f9f4%2fresourcegroups%2faz400m05l11-rg%2fproviders%2fmicrosoft.loadtestservice%2floadtests%2feshoponwebloadtesting/testId/0d295119-12d0-a7b84787c004/testRunId/f4bec76a-8b49-44ee-a388-12af34f0d4ec
TestRun completed

-------------------Summary ---------------
TestRun start time: Mon Jul 24 2023 23:00:31 GMT+0000 (Coordinated Universal Time)
TestRun end time: Mon Jul 24 2023 23:06:02 GMT+0000 (Coordinated Universal Time)
Virtual Users: 50
TestStatus: DONE

-------------------Test Criteria ---------------
Results          :1 Pass 1 Fail

Criteria                     :Actual Value        Result
avg(response_time_ms) > 300                       1355.29               FAILED
percentage(error) > 50                                                  PASSED


------------------Client-side metrics------------

Homepage
response time        : avg=1355ms min=58ms med=666ms max=16524ms p(90)=2472ms p(95)=5819ms p(99)=13657ms
requests per sec     : avg=37
total requests       : 4531
total errors         : 0
total error rate     : 0
##[error]TestResult: FAILED
Finishing: AzureLoadTest
```

8. La dernière ligne de la sortie du test de charge indique **##[error]TestResult: FAILED** : puisque vous avez défini un temps de réponse moyen supérieur à 300 ou un pourcentage d’erreur supérieur à 20 en tant que **FailCriteria**, si le temps de réponse moyen est supérieur à 300, la tâche est marquée comme ayant échoué. 

    > Remarque : dans un scénario réel, si vous validez les performances de votre App Service et si les performances sont inférieures à un certain seuil (ce qui signifie généralement que la charge est plus importante sur l’application web), vous pouvez déclencher un nouveau déploiement vers un autre Azure App Service. Il est impossible de contrôler le temps de réponse pour les environnements de labo Azure : la logique a donc été rétablie pour garantir l’échec.

9.  L’état FAILED de la tâche de pipeline reflète en réalité l’état SUCCESS de la validation des critères de test de charge Azure.

### Exercice 3 : Supprimer les ressources du labo Azure.

Dans le cadre de cet exercice, vous supprimerez les ressources Azure provisionnées dans ce labo pour éliminer les frais inattendus.

>**Remarque** : N’oubliez pas de supprimer toutes les nouvelles ressources Azure que vous n’utilisez plus. La suppression des ressources inutilisées vous évitera d’encourir des frais inattendus.

#### Tâche 1 : supprimer les ressources du labo Azure

Dans le cadre de cette tâche, vous utiliserez Azure Cloud Shell pour supprimer les ressources Azure provisionnées dans ce labo afin d’éliminer les frais superflus.

1. Dans le Portail Azure, ouvrez la session shell **Bash** dans le volet **Cloud Shell**.
2. Listez tous les groupes de ressources créés dans les labos de ce module en exécutant la commande suivante :

    ```sh
    az group list --query "[?starts_with(name,'az400m16l01')].name" --output tsv
    ```

3. Supprimez tous les groupes de ressources que vous avez créés dans les labos de ce module en exécutant la commande suivante :

    ```sh
    az group list --query "[?starts_with(name,'az400m16l01')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Remarque** : La commande s’exécute de façon asynchrone (comme déterminé par le paramètre --no-wait). Par conséquent, vous serez en mesure d’exécuter une autre commande Azure CLI immédiatement après au cours de la même session Bash, mais la suppression réelle du groupe de ressources prendra quelques minutes.

## Révision

Dans le cadre de cet exercice, vous avez déployé une application web sur Azure App Service à l’aide d’Azure Pipelines, et avez déployé une ressource de test de charge Azure avec TestRuns. Ensuite, vous avez intégré le fichier config.yaml de test de charge Jmeter au contrôle de code source Azure DevOps Repos et étendu votre pipeline CI/CD avec le test de charge Azure. Dans le dernier exercice, vous avez appris à définir les critères de réussite du LoadTest.
