---
lab:
  title: Configuration des pipelines en tant que code avec YAML
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# Configuration des pipelines en tant que code avec YAML

## Manuel de labo de l’étudiant

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Créer une organisation ou une collection de projets](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifier un abonnement Azure existant ou en créer un.

- Vérifiez que vous disposez d’un compte Microsoft ou d’un compte Microsoft Entra avec le rôle Propriétaire dans l’abonnement Azure et le rôle Administrateur général dans le locataire Microsoft Entra associé à l’abonnement Azure. Pour plus d’informations, consultez [Répertorier les attributions de rôle Azure à l’aide du portail Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) et [Afficher et attribuer des rôles d’administrateur dans Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Présentation du labo

De nombreuses équipes préfèrent définir leurs pipelines de build et de mise en production à l’aide de YAML. Cela leur permet d’accéder aux mêmes fonctionnalités de pipeline que celles qui utilisent le concepteur visuel, mais avec un fichier de balisage qui peut être géré comme n’importe quel autre fichier source. Pour ajouter les définitions de build YAML à un projet, il suffit d’insérer les fichiers correspondants à la racine du référentiel. Azure DevOps fournit également des modèles par défaut pour les types de projets populaires et un concepteur YAML pour simplifier le processus de définition des tâches de build et de mise en production.

## Objectifs

À la fin de ce labo, vous serez en mesure d’accomplir les tâches suivantes :

- Configurer les pipelines CI/CD en tant que code avec YAML dans Azure DevOps

## Durée estimée : 60 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

Dans cet exercice, vous configurerez les prérequis du labo, qui se composent d’un nouveau projet Azure DevOps comprenant un référentiel basé sur [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tâche 1 : (à ignorer si vous l’avez déjà effectuée) créer et configurer le projet d’équipe

Dans le cadre de cette tâche, vous créerez un projet Azure DevOps **eShopOnWeb_MultiStageYAML** qui sera utilisé dans plusieurs labos.

1. Sur votre ordinateur de labo, ouvrez votre organisation Azure DevOps dans votre navigateur. Cliquez sur **Nouveau projet**. Nommez votre projet **eShopOnWeb_MultiStageYAML** et ne modifiez pas les autres champs par défaut. Cliquez sur **Créer**.

    ![Création d’un projet](images/create-project.png)

#### Tâche 2 : (à ignorer si vous l’avez déjà effectuée) importer un référentiel Git eShopOnWeb

Dans le cadre de cette tâche, vous importerez le référentiel Git eShopOnWeb qui sera utilisé dans plusieurs labos.

1. Sur votre ordinateur de labo, ouvrez votre organisation Azure DevOps dans votre navigateur ainsi que le projet **eShopOnWeb_MultiStageYAML** que vous venez de créer. Cliquez sur **Repos > Fichiers**, puis sur **Importer un référentiel**. Cliquez sur **Importer**. Dans la fenêtre **Importer un référentiel Git**, collez l’URL https://github.com/MicrosoftLearning/eShopOnWeb.git, puis cliquez sur **Importer** :

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

    > **Remarque :** exécutez la commande suivante dans Azure Cloud Shell - Bash pour obtenir la liste des régions et leur alias :

    ```bash
    az account list-locations -o table
    ```

4. Dans l’invite **Bash** du volet **Cloud Shell**, exécutez la commande suivante pour créer un groupe de ressources (remplacez l’espace réservé `<region>` par le nom de la région Azure la plus proche de vous, par exemple « centralus », « westeurope » ou une autre région de votre choix).

    ```bash
    LOCATION='<region>'
    ```

    ```bash
    RESOURCEGROUPNAME='az400m05l11-RG'
    az group create --name $RESOURCEGROUPNAME --location $LOCATION
    ```

5. Exécutez la commande suivante pour créer un plan Windows App Service :

    ```bash
    SERVICEPLANNAME='az400m05l11-sp1'
    az appservice plan create --resource-group $RESOURCEGROUPNAME --name $SERVICEPLANNAME --sku B3
    ```

6. Créez une application web avec un nom unique.

    ```bash
    WEBAPPNAME=eshoponWebYAML$RANDOM$RANDOM
    az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME
    ```

    > **Remarque** : enregistrez le nom de l’application web. Vous en aurez besoin plus tard dans ce labo.

7. Fermez Azure Cloud Shell, mais laissez le Portail Azure ouvert dans le navigateur.

### Exercice 1 : configurer des pipelines CI/CD en tant que code avec YAML dans Azure DevOps

Dans le cadre de cet exercice, vous configurerez de pipelines CI/CD en tant que code avec YAML dans Azure DevOps.

#### Tâche 1 : ajouter une définition de build YAML

Dans le cadre de cette tâche, vous ajouterez une définition de build YAML au projet existant.

1. Revenez au volet **Pipelines** du hub **Pipelines**.
2. Dans la fenêtre **Créer votre premier pipeline**, cliquez sur **Créer un pipeline**.

    > **Remarque** : vous utiliserez l’assistant pour créer une définition de pipeline YAML basée sur votre projet.

3. Dans le volet **Où se trouve votre code ?**, sélectionnez l’option **Azure Repos Git (YAML)**.
4. Dans le volet **Sélectionner un référentiel**, cliquez sur **eShopOnWeb_MultiStageYAML**.
5. Faites défiler le volet **Configurer votre pipeline** et sélectionnez **Fichier YAML Azure Pipelines existant**.
6. Dans le volet **Sélectionner un fichier YAML existant**, spécifiez les paramètres suivants :
   - Branche : **main**
   - Chemin d’accès : **.ado/eshoponweb-ci.yml**
7. Cliquez sur **Continuer** pour enregistrer ces paramètres.
8. Sur l’écran **Vérifier le YAML de votre pipeline**, cliquez sur **Exécuter** pour démarrer le processus de génération du pipeline.
9. Attendez la fin du processus de génération de pipeline. Ignorez les avertissements concernant le code source lui-même : ils n’ont aucune importance dans cet exercice de labo.

    > **Remarque** : chaque tâche issue du fichier YAML peut être révisée, y compris les avertissements et les erreurs.

#### Tâche 2 : ajouter une livraison continue à la définition YAML

Dans le cadre de cette tâche, vous ajouterez une livraison continue à la définition YAML du pipeline que vous avez créé dans la tâche précédente.

> **Remarque** : puisque les processus de génération et de test sont réussis, il est désormais possible d’ajouter la livraison à la définition YAML.

1. Dans le volet d’exécution du pipeline, cliquez sur les points de suspension dans le coin supérieur droit. Dans le menu déroulant qui s’affiche, cliquez sur **Modifier le pipeline**.
2. Dans le volet qui affiche le contenu du fichier **eShopOnWeb_MultiStageYAML/.ado/eshoponweb-ci.yml**, accédez à la fin du fichier (ligne 56), puis appuyez sur la touche **Entrée/Retour** pour ajouter une nouvelle ligne vide.
3. À la ligne **57**, ajoutez le contenu suivant pour définir l’étape de **mise en production** dans le pipeline YAML.

    > **Remarque** : vous pouvez définir n’importe quelle étape dont vous avez besoin pour mieux organiser et suivre la progression du pipeline.

    ```yaml
    - stage: Deploy
      displayName: Deploy to an Azure Web App
      jobs:
      - job: Deploy
        pool:
          vmImage: 'windows-2019'
        steps:
    ```

4. Placez le curseur sur une nouvelle ligne à la fin de la définition YAML.

    > **Remarque** : il s’agit de l’emplacement où les nouvelles tâches seront ajoutées.

5. Dans la liste des tâches situées à droite du volet de code, recherchez et sélectionnez la tâche **Déployer Azure App Service**.
6. Dans le volet **Déployer Azure App Service**, spécifiez les paramètres suivants, puis cliquez sur **Ajouter** :

    - Dans la liste déroulante **Abonnement Azure**, sélectionnez l’abonnement Azure où vous avez déployé les ressources Azure que vous avez créées dans le cadre de ce labo. Cliquez ensuite sur **Autoriser** et, lorsque vous y êtes invité, authentifiez-vous à l’aide du même compte d’utilisateur que celui que vous avez utilisé pendant le déploiement des ressources Azure.
    - Dans la liste déroulante **Nom App Service**, sélectionnez le nom de l’application web que vous avez déployée dans le cadre de ce labo.
    - Dans la zone de texte **Package ou dossier**, **passez** la valeur par défaut à `$(Build.ArtifactStagingDirectory)/**/Web.zip`.
7. Confirmez les paramètres en cliquant sur le bouton **Ajouter** du volet Assistant.

    > **Remarque** : cette opération ajoute automatiquement la tâche de déploiement à la définition du pipeline YAML.

8. L’extrait de code ajouté à l’éditeur doit ressembler à l’exemple ci-dessous et doit refléter votre nom pour les paramètres azureSubscription et WebappName :

    ```yaml
        - task: AzureRmWebAppDeployment@4
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: 'AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)'
            appType: 'webApp'
            WebAppName: 'eshoponWebYAML369825031'
            packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
    ```

9. Vérifiez que la tâche est répertoriée en tant que tâche enfant de la tâche **étapes**. Si ce n’est pas le cas, sélectionnez toutes les lignes de la tâche que vous venez d’ajouter et appuyez deux fois sur la touche de **tabulation** pour ajouter un retrait de quatre espaces afin qu’elle soit répertoriée en tant que tâche enfant de la tâche **étapes**.

    > **Remarque** : le paramètre **packageForLinux** est trompeur dans le contexte de ce labo, mais il est correct pour Windows ou Linux.

    > **Remarque** : ces deux étapes s’exécutent de façon autonome par défaut. Par conséquent, la sortie de build de la première étape peut être indisponible pour la deuxième étape si aucune modification supplémentaire n’y est apportée. Pour mettre en œuvre ces modifications, vous ajouterez une nouvelle tâche afin de télécharger l’artefact de déploiement au début de l’étape de déploiement.

10. Survolez avec votre curseur la première ligne sous le nœud **étapes** de l’étape **déploiement**, puis appuyez sur les touches Entrée/Retour pour ajouter une nouvelle ligne vide (ligne 64).
11. Dans le volet **Tâches**, recherchez la tâche **Télécharger les artefacts du build** et sélectionnez-la.
12. Indiquez les paramètres suivants pour cette tâche :
    - Télécharger les artefacts produits par : **build actuel**
    - Type de téléchargement : **artefact spécifique**
    - Nom de l’artefact : **sélectionnez « Site web » dans la liste** (ou **saisissez directement « Site web »** si l’option n’apparaît pas automatiquement dans la liste)
    - Répertoire de destination : **$(Build.ArtifactStagingDirectory)**
13. Cliquez sur **Ajouter**.
14. L’extrait de code ajouté doit ressembler à ce qui suit :

    ```yaml
        - task: DownloadBuildArtifacts@0
          inputs:
            buildType: 'current'
            downloadType: 'single'
            artifactName: 'Website'
            downloadPath: '$(Build.ArtifactStagingDirectory)'
    ```

15. Si la mise en retrait YAML est désactivée, sélectionnez la tâche ajoutée dans l’éditeur, puis appuyez deux fois sur la touche **tabulation** pour ajouter un retrait de quatre espaces.

    > **Remarque** : vous pouvez également ajouter ici une ligne vide avant et après l’élément pour faciliter la lecture.

16. Cliquez sur **Enregistrer**. Dans le volet **Enregistrer**, cliquez à nouveau sur **Enregistrer** pour valider la modification directement dans la branche principale.

    > **Remarque** : puisque le CI-YAML d’origine n’a pas été configuré de manière à déclencher automatiquement un nouveau build, celui-ci doit être lancé manuellement.

17. Dans le menu gauche d’Azure DevOps, accédez à **Pipelines** et sélectionnez une nouvelle fois **Pipelines**.
18. Ouvrez le pipeline **EShopOnWeb_MultiStageYAML** et cliquez sur **Exécuter le pipeline**.
19. Confirmez l’**exécution** dans le volet qui apparaît.
20. Deux étapes différentes, **Générer la solution .Net Core** et **Déployer sur l’application web Azure** s’affichent.
21. Attendez le lancement du pipeline, puis la fin de l’étape de génération.
22. Dès que l’étape de déploiement démarre, vous êtes invité à renseigner les **autorisations nécessaires**. Une barre orange apparaît à l’écran et indique :

    ```text
    This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
    ```

23. Cliquez sur **Vue**.
24. Dans le volet **En attente de révision**, cliquez sur **Autoriser**.
25. Validez le message dans la fenêtre contextuelle **Autoriser**, puis confirmez votre choix en cliquant sur **Autoriser**.
26. Cela lance l’étape de déploiement. Attendez la fin de ce processus.

     > **Remarque** : si le déploiement échoue en raison d’un problème au niveau de la syntaxe du pipeline YAML, utilisez l’élément suivant comme référence :

     ```yaml
    #NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
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
          displayName: Test
          inputs:
            command: 'test'
            projects: 'tests/UnitTests/*.csproj'
        
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
        
        - task: PublishBuildArtifacts@1
          displayName: Publish Artifacts ADO - Bicep
          inputs:
            PathtoPublish: '$(Build.SourcesDirectory)/.azure/bicep/webapp.bicep'
            ArtifactName: 'Bicep'
            publishLocation: 'Container'
    
    - stage: Deploy
      displayName: Deploy to an Azure Web App
      jobs:
      - job: Deploy
        pool:
          vmImage: 'windows-2019'
        steps:
        - task: DownloadBuildArtifacts@0
          inputs:
            buildType: 'current'
            downloadType: 'single'
            artifactName: 'Website'
            downloadPath: '$(Build.ArtifactStagingDirectory)'
        - task: AzureRmWebAppDeployment@4
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: 'AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)'
            appType: 'webApp'
            WebAppName: 'eshoponWebYAML369825031'
            packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
    
    ```

#### Tâche 4 : passer en revue le site déployé

1. Revenez à la fenêtre du navigateur web où est ouvert le Portail Azure et accédez au volet affichant les propriétés de l’application web Azure.
2. Dans le volet Application web Azure, cliquez sur **Vue d’ensemble**. Dans le volet Vue d’ensemble, cliquez sur **Parcourir** pour ouvrir votre site dans un nouvel onglet du navigateur web.
3. Vérifiez que le site déployé se charge comme prévu dans le nouvel onglet du navigateur et que le site web d’e-commerce EShopOnWeb s’affiche.

### Exercice 2 : configurer des paramètres d’environnement pour des pipelines CI/CD en tant que code avec YAML dans Azure DevOps

Dans le cadre de cet exercice, vous ajouterez des approbations à un pipeline YAML dans Azure DevOps.

#### Tâche 1 : configurer des environnements de pipeline

Contrairement aux pipelines de mise en production Azure DevOps classiques, les pipelines YAML en tant que code n’ont pas de portes de mise en production/qualité. Toutefois, des réglages similaires peuvent être configurés pour les pipelines YAML en tant que code à l’aide des **environnements**. Dans le cadre de cette tâche, vous utiliserez ce mécanisme pour configurer les approbations en vue de l’étape de génération.

1. Depuis le projet Azure DevOps **EShopOnWeb_MultiStageYAML**, accédez aux **pipelines**.
2. Dans le menu Pipelines à gauche, sélectionnez **Environnements**.
3. Cliquez sur **Créer un environnement**.
4. Dans le volet **Nouvel environnement**, nommez l’environnement **Approbations**.
5. Sous **Ressources**, sélectionnez **Aucune**.
6. Confirmez les paramètres en cliquant sur le bouton **Créer**.
7. Une fois l’environnement créé, cliquez sur les points de suspension (…) en regard du bouton « Ajouter une ressource ».
8. Sélectionnez **Approbations et vérifications**.
9. Sous **Ajouter votre première vérification**, sélectionnez **Approbations**.
10. Ajoutez votre nom de compte d’utilisateur Azure DevOps au champ **Approbateurs**.

    > **Remarque :** dans un scénario réel, ce nom est celui de votre équipe DevOps qui travaille sur ce projet.

11. Confirmez les paramètres d’approbation que vous avez définis en cliquant sur le bouton **Créer**.
12. Il ne reste plus qu’à ajouter les paramètres « environmement: approvals » requis au code du pipeline YAML en prévision de l’étape de déploiement. Pour ce faire, ouvrez **Repos**, accédez au dossier **.ado** puis sélectionnez le fichier **eshoponweb-ci.yml** de pipeline en tant que code.
13. Dans la vue Contenu, cliquez sur le bouton **Modifier** pour passer en mode Modification.
14. Accédez au début du **travail de déploiement**(-job: Deploy à la ligne 60).
15. Ajoutez une nouvelle ligne vide juste en dessous, puis ajoutez l’extrait de code suivant :

    ```yaml
      environment: approvals
    ```

    Après cette opération, l’extrait de code doit ressembler à ceci :

    ```yaml
     jobs:
      - job: Deploy
        environment: approvals
        pool:
          vmImage: 'windows-2019'
    ```

16. Puisque l’environnement est un paramètre spécifique d’une étape de déploiement, il ne peut pas être utilisé par des « travaux ». Il faut donc apporter des modifications supplémentaires à la définition du travail actuel.
17. À la ligne **60**, renommez « -job: Deploy » en **- deployment: Deploy**.
18. Sous la ligne **63** (vmImage: Windows-2019), ajoutez ensuite une autre ligne vide.
19. Collez-y l’extrait YAML suivant :

    ```yaml
        strategy:
          runOnce:
            deploy:
    ```

20. Sélectionnez l’extrait de code restant (de la ligne **67** jusqu’à la fin) et appuyez sur la touche **tabulation** pour corriger la mise en retrait YAML.

    Suite à cette opération, l’extrait de code YAML doit ressembler à ceci et refléter l’**étape de déploiement** :

    ```yaml
    - stage: Deploy
      displayName: Deploy to an Azure Web App
      jobs:
      - deployment: Deploy
        environment: approvals
        pool:
          vmImage: 'windows-2019'
        strategy:
          runOnce:
            deploy:
              steps:
              - task: DownloadBuildArtifacts@0
                inputs:
                  buildType: 'current'
                  downloadType: 'single'
                  artifactName: 'Website'
                  downloadPath: '$(Build.ArtifactStagingDirectory)'
              - task: AzureRmWebAppDeployment@4
                inputs:
                  ConnectionType: 'AzureRM'
                  azureSubscription: 'AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)'
                  appType: 'webApp'
                  WebAppName: 'eshoponWebYAML369825031'
                  packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
    ```

21. Confirmez les modifications apportées au fichier YAML du code en cliquant sur **Valider**, puis une nouvelle fois sur **Valider** dans le volet de validation qui s’affiche.
22. À gauche du menu du projet Azure DevOps, sélectionnez **Pipelines**. Sélectionnez une nouvelle fois **Pipelines** : le pipeline **EshopOnWeb_MultiStageYAML** que vous avez déjà utilisé apparaît.
23. Ouvrez le pipeline.
24. Cliquez sur **Exécuter le pipeline** pour déclencher une nouvelle exécution de pipeline. Confirmez votre choix en cliquant sur **Exécuter**.
25. Comme pour la tâche précédente, l’étape de génération démarre comme prévu. Attendez la fin de ce processus.
26. Puisque *environment:approvals* est configuré pour l’étape de déploiement, une confirmation d’approbation vous est demandée avant son lancement.
27. Cette demande s’affiche dans la vue Pipeline et indique **En attente (0/1 vérification réussie)**. Un message de notification vous indique aussi que **l’approbation doit être examinée avant que cette exécution puisse continuer à être déployée sur une application web Azure**.
28. Cliquez sur le bouton **Afficher** près de ce message.
29. Dans le volet **Vérifications et validations manuellement pour le déploiement sur l’application web Azure** qui s’affiche, cliquez sur le message **En attente d’approbation**.
30. Cliquez sur **Approuver**.
31. Cela permet de lancer l’étape de déploiement et de correctement déployer le code source de l’application web Azure.

    > **Remarque :** bien que seules les approbations aient été utilisées dans le cadre de cet exemple, les autres vérifications telles qu’Azure Monitor, API REST, etc. peuvent être utilisées de manière similaire.

### Exercice 3 : Supprimer les ressources du labo Azure.

Dans le cadre de cet exercice, vous supprimerez les ressources Azure provisionnées dans ce labo pour éliminer les frais inattendus.

>**Remarque** : N’oubliez pas de supprimer toutes les nouvelles ressources Azure que vous n’utilisez plus. La suppression des ressources inutilisées vous évitera d’encourir des frais inattendus.

#### Tâche 1 : supprimer les ressources du labo Azure

Dans le cadre de cette tâche, vous utiliserez Azure Cloud Shell pour supprimer les ressources Azure provisionnées dans ce labo afin d’éliminer les frais superflus.

1. Dans le Portail Azure, ouvrez la session shell **Bash** dans le volet **Cloud Shell**.
2. Listez tous les groupes de ressources créés dans les labos de ce module en exécutant la commande suivante :

    ```sh
    az group list --query "[?starts_with(name,'az400m05l11-RG')].name" --output tsv
    ```

3. Supprimez tous les groupes de ressources que vous avez créés dans les labos de ce module en exécutant la commande suivante :

    ```sh
    az group list --query "[?starts_with(name,'az400m05l11-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Remarque** : La commande s’exécute de façon asynchrone (comme déterminé par le paramètre --no-wait). Par conséquent, vous serez en mesure d’exécuter une autre commande Azure CLI immédiatement après au cours de la même session Bash, mais la suppression réelle du groupe de ressources prendra quelques minutes.

## Révision

Dans le cadre de ce labo, vous avez configuré des pipelines CI/CD en tant que code avec YAML dans Azure DevOps.
