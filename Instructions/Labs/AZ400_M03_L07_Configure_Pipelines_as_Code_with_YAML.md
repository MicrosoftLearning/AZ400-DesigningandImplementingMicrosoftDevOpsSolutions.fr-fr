---
lab:
  title: Configuration de pipelines en tant que code avec YAML
  module: 'Module 03: Design and implement a release strategy'
---

# Configuration de pipelines en tant que code avec YAML

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Créer une organisation ou une collection de projets](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifier un abonnement Azure existant ou en créer un.

- Vérifiez que vous disposez d’un compte Microsoft ou d’un compte Microsoft Entra avec le rôle Propriétaire dans l’abonnement Azure et le rôle Administrateur général dans le locataire Microsoft Entra associé à l’abonnement Azure. Pour plus d’informations, consultez [Répertorier les attributions de rôle Azure à l’aide du portail Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) et [Afficher et attribuer des rôles d’administrateur dans Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Présentation du labo

De nombreuses équipes préfèrent définir leurs pipelines de build et de mise en production à l’aide de YAML. Cela leur permet d’accéder aux mêmes fonctionnalités de pipeline que celles qui utilisent le concepteur visuel, mais avec un fichier de balisage qui peut être géré comme n’importe quel autre fichier source. Pour ajouter les définitions de build YAML à un projet, il suffit d’insérer les fichiers correspondants à la racine du référentiel. Azure DevOps fournit également des modèles par défaut pour les types de projets populaires et un concepteur YAML pour simplifier le processus de définition des tâches de build et de mise en production.

## Objectifs

À la fin de ce labo, vous serez en mesure d’effectuer les tâches suivants :

- Configurer les pipelines CI/CD en tant que code avec YAML dans Azure DevOps

## Durée estimée : 45 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

Dans cet exercice, vous allez configurer les prérequis du labo.

#### Tâche 1 : (passer si terminée) créer et configurer le projet d’équipe

Dans cette tâche, vous allez créer un projet Azure DevOps **eShopOnWeb_MultiStageYAML** qui sera utilisé dans plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps. Cliquez sur **Nouveau projet**. Donnez au projet le nom **eShopOnWeb_MultiStageYAML** et conservez les valeurs par défaut des autres champs. Cliquez sur **Créer**.

   ![Capture d’écran du volet de création d’un nouveau projet.](images/create-project.png)

#### Tâche 2 : (passer si terminée) importer le référentiel Git eShopOnWeb

Dans cette tâche, vous allez importer le référentiel Git eShopOnWeb qui sera utilisé par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps et le projet **eShopOnWeb_MultiStageYAML** créé précédemment. Cliquez sur **Dépôts > Fichiers**, **Importer un référentiel**. Cliquez sur **Importer**. Dans la fenêtre **Importer un référentiel Git**, collez l’URL https://github.com/MicrosoftLearning/eShopOnWeb.git, puis cliquez sur **Importer** :

   ![Capture d’écran du volet Importer un référentiel.](images/import-repo.png)

1. Le référentiel est organisé de la manière suivante :
   - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
   - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
   - Le dossier **infra** contient l’infrastructure Bicep&ARM sous forme de modèles de code utilisés dans certains scénarios de labo.
   - **Définitions de workflow GitHub YAML du conteneur de dossiers .github**.
   - Le dossier **src** contient le site web .NET 8 utilisé dans les scénarios de labo.

1. Accédez à **Dépôts > Branches**.
1. Pointez sur la branche **principale**, puis cliquez sur les points de suspension à droite de la colonne.
1. Cliquez sur **Définir comme branche par défaut**.

    > **Remarque** : si la branche primaire est déjà sélectionnée comme la branche par défaut, l’option **Définir comme branche par défaut** est grisée. Dans ce cas, continuez en suivant les instructions

#### Tâche 3 : Créer des ressources Azure

Dans cette tâche, vous allez créer une application web Azure à l’aide du portail Azure.

1. Sur l’ordinateur de labo, démarrez un navigateur web, accédez au [**portail Azure**](https://portal.azure.com) et connectez-vous avec le compte d’utilisateur qui a le rôle Propriétaire dans l’abonnement Azure que vous utiliserez dans ce labo et le rôle Administrateur général dans le locataire Microsoft Entra associé à cet abonnement.
1. Dans le portail Azure, dans la barre d’outils, cliquez sur l’icône **Cloud Shell** située juste à droite de la zone de texte de recherche.
1. Si vous êtes invité à sélectionner **Bash** ou **PowerShell**, sélectionnez **Bash**.

   > **Remarque** : Si c’est la première fois que vous démarrez **Cloud Shell** et que la fenêtre contextuelle **Prise en main** s'affiche, sélectionnez **Aucun compte de stockage requis** et l’abonnement que vous utilisez dans ce labo, puis cliquez sur **Appliquer**.

1. À partir de l’invite **Bash**, dans le volet **Cloud Shell**, exécutez la commande suivante pour créer un groupe de ressources (remplacez l’espace réservé `<region>` par le nom de la région Azure la plus proche de vous, par exemple « centralus », « westeurope » ou une autre région de votre choix).

   ```bash
   LOCATION='<region>'
   ```

   ```bash
   RESOURCEGROUPNAME='az400m03l07-RG'
   az group create --name $RESOURCEGROUPNAME --location $LOCATION
   ```

1. Créez un plan Windows App Service en exécutant la commande suivante :

   ```bash
   SERVICEPLANNAME='az400m03l07-sp1'
   az appservice plan create --resource-group $RESOURCEGROUPNAME --name $SERVICEPLANNAME --sku B3
   ```

    > **Remarque** : Si vous recevez une erreur telle que « L’abonnement n’est pas enregistré pour utiliser l’espace de noms " Microsoft.Web " » lorsque vous exécutez la commande précédente, exécutez `az provider register --namespace Microsoft.Web` suivant, puis exécutez à nouveau la commande qui a généré l’erreur.

1. Créez une application web avec un nom unique.

   ```bash
   WEBAPPNAME=eshoponWebYAML$RANDOM$RANDOM
   az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME
   ```

   > **Remarque** : notez le nom de l’application web. Vous en aurez besoin plus tard dans ce labo.

1. Fermez Azure Cloud Shell, mais laissez le portail Azure ouvert dans le navigateur.

### Exercice 1 : configurer des pipelines CI/CD en tant que code avec YAML dans Azure DevOps

Dans cet exercice, vous allez configurer des pipelines CI/CD en tant que code avec YAML dans Azure DevOps.

#### Tâche 1 : ajouter une définition de build YAML

Dans cette tâche, vous allez ajouter une définition de build YAML au projet existant.

1. Revenez au volet **Pipelines** du hub **Pipelines**.
1. Dans la fenêtre **Créer votre premier pipeline**, cliquez sur **Créer un pipeline**.

   > **Remarque** : nous allons utiliser l’Assistant pour créer une définition de pipeline YAML basée sur notre projet.

1. Dans le volet **Où se trouve votre code ?**, cliquez sur l’option **Azure Repos Git (YAML)**.
1. Dans le volet **Sélectionner un dépôt**, cliquez sur **eShopOnWeb_MultiStageYAML**.
1. Dans le volet **Configurer votre pipeline**, faites défiler la page vers le bas, puis sélectionnez **Fichier YAML Azure Pipelines existant**.
1. Dans le panneau **Sélectionner un fichier YAML existant**, spécifiez les paramètres suivants :
   - Branche : **main**
   - Chemin : **.ado/eshoponweb-ci.yml**
1. Cliquez sur **Continuer** pour enregistrer les paramètres.
1. Dans l’écran **Passer en revue le fichier YAML de pipeline**, cliquez sur **Exécuter** pour démarrer le processus de pipeline de build.
1. Attendez la fin du pipeline de build. Ignorez les avertissements concernant le code source lui-même, car ils ne sont pas pertinents pour cet exercice de labo.

   > **Remarque** : chaque tâche du fichier YAML est disponible pour révision, y compris les avertissements et les erreurs.

#### Tâche 2 : ajouter la livraison continue à la définition YAML

Dans cette tâche, vous allez ajouter la livraison continue à la définition YAML du pipeline que vous avez créé dans la tâche précédente.

> **Remarque** : une fois les processus de build et de test réussis, nous pouvons à présent ajouter la livraison à la définition YAML.

1. Dans le volet d’exécution du pipeline, cliquez sur le symbole de points de suspension dans le coin supérieur droit, puis, dans le menu déroulant, cliquez sur **Modifier le pipeline**.
1. Dans le volet affichant le contenu du fichier **eShopOnWeb_MultiStageYAML/.ado/eshoponweb-ci.yml**, accédez à la fin du fichier (ligne 56), puis appuyez sur **Entrée/Retour** pour ajouter une nouvelle ligne vide.
1. À la ligne **57**, ajoutez le contenu suivant pour définir la phase de **mise en production** dans le pipeline YAML.

   > **Remarque** : vous pouvez définir les phases dont vous avez besoin pour mieux organiser et suivre la progression du pipeline.

   ```yaml
   - stage: Deploy
     displayName: Deploy to an Azure Web App
     jobs:
       - job: Deploy
         pool:
           vmImage: "windows-latest"
         steps:
   ```

1. Définissez le curseur sur une nouvelle ligne à la fin de la définition YAML.

   > **Remarque** : il s’agit de l’emplacement où les nouvelles tâches sont ajoutées.

1. Dans la liste des tâches à droite du volet de code, recherchez et sélectionnez la tâche **Déploiement Azure App Service**.
1. Dans le volet **Déploiement Azure App Service**, spécifiez les paramètres suivants, puis cliquez sur **Ajouter** :

   - Dans la liste déroulante **Abonnement Azure**, sélectionnez l’abonnement Azure dans lequel vous avez déployé les ressources Azure précédemment dans le labo, cliquez sur **Autoriser**, puis, lorsque vous y êtes invité, authentifiez-vous à l’aide du même compte d’utilisateur que celui que vous avez utilisé pendant le déploiement des ressources Azure.
   - Dans la liste déroulante **Nom de l’App Service**, sélectionnez le nom de l’application web que vous avez déployée précédemment dans le labo.
   - Dans la zone de texte **Package ou dossier**, **mettez à jour** la valeur par défaut sur `$(Build.ArtifactStagingDirectory)/**/Web.zip`.
   - Ouvrez la section **Paramètres d’application et de configuration** et, dans la zone de texte **Paramètres de l’application**, ajoutez `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development`

1. Confirmez les paramètres du volet Assistant en cliquant sur le bouton **Ajouter**.

   > **Remarque** : cette opération ajoute automatiquement la tâche de déploiement à la définition du pipeline YAML.

1. L’extrait de code ajouté à l’éditeur doit ressembler à ce qui suit et refléter votre nom pour les paramètres azureSubscription et WebappName :

   ```yaml
   - task: AzureRmWebAppDeployment@4
     inputs:
       ConnectionType: "AzureRM"
       azureSubscription: "AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)"
       appType: "webApp"
       WebAppName: "eshoponWebYAML369825031"
       packageForLinux: "$(Build.ArtifactStagingDirectory)/**/Web.zip"
       AppSettings: "-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development"
   ```

1. Vérifiez que la tâche est répertoriée en tant qu’enfant de la tâche **étapes**. Si ce n’est pas le cas, sélectionnez toutes les lignes de la tâche ajoutée, puis appuyez deux fois sur la touche de **tabulation** pour mettre en retrait la tâche de quatre espaces, afin qu’elle soit répertoriée en tant qu’enfant de la tâche **étapes**.

   > **Remarque** : le paramètre **packageForLinux** est trompeur dans le contexte de ce labo, mais il est valide pour Windows ou Linux.

   > **Remarque**  : par défaut, ces deux phases s’exécutent de manière indépendante. Par conséquent, la sortie de build de la première phase peut ne pas être disponible pour la deuxième phase sans modifications supplémentaires. Pour implémenter ces modifications, nous allons ajouter une nouvelle tâche pour télécharger l’artefact de déploiement au début de la phase de déploiement.

1. Placez le curseur sur la première ligne sous le nœud **étapes** de la phase de **déploiement**, puis appuyez sur Entrée/Retour pour ajouter une nouvelle ligne vide (ligne 64).
1. Dans le volet **Tâches**, recherchez et sélectionnez la tâche **Télécharger les artefacts de build**.
1. Indiquez les paramètres suivantes pour cette tâche :
   - Télécharger les artefacts produits par : **Build actuelle**
   - Type de téléchargement : **Artefact spécifique**
   - Nom de l’artefact : **sélectionnez « site web » dans la liste** (ou **saisissez directement « `Website` »** s’il n’apparaît pas automatiquement dans cette liste)
   - Répertoire de destination : **$(Build.ArtifactStagingDirectory)**
1. Cliquez sur **Ajouter**.
1. L’extrait de code ajouté doit ressembler à ce qui suit :

   ```yaml
   - task: DownloadBuildArtifacts@1
     inputs:
       buildType: "current"
       downloadType: "single"
       artifactName: "Website"
       downloadPath: "$(Build.ArtifactStagingDirectory)"
   ```

1. Si la mise en retrait YAML est désactivée et que la tâche ajoutée toujours sélectionnée dans l’éditeur, appuyez deux fois sur **Tabulation** pour mettre en retrait quatre espaces.

   > **Remarque** : ici aussi, vous pouvez ajouter une ligne vide avant et après pour faciliter la lecture.

1. Cliquez sur **Valider et enregistrer**, puis, dans le volet **Valider et enregistrer**, cliquez à nouveau sur **Enregistrer** pour valider la modification directement dans la branche primaire.

   > **Remarque** : étant donné que notre fichier CI-YAML d’origine n’a pas été configuré pour déclencher automatiquement une nouvelle build, nous devons lancer cette dernière manuellement.

1. Dans le menu de gauche Azure DevOps, accédez à **Pipelines**, puis sélectionnez à nouveau **Pipelines**.
1. Ouvrez le pipeline **eShopOnWeb_MultiStageYAML**, puis cliquez sur **Exécuter le pipeline**.
1. Confirmez l’**exécution** dans le volet affiché.
1. Notez les 2 phases différentes affichées, **Générer la solution .NET Core** et **Déployer sur Azure Web App**.
1. Attendez que le pipeline démarre et qu’il termine correctement la phase de génération.
1. Une fois que l’étape de déploiement souhaite commencer, vous êtes invité à utiliser les **autorisations nécessaires**, ainsi qu’une barre orange indiquant :

   ```text
   This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
   ```

1. Cliquez sur **Affichage**.
1. Dans le volet **En attente d’examen**, cliquez sur **Autoriser**.
1. Validez le message dans la fenêtre **Autoriser l’accès ?**, puis confirmez en cliquant sur **Autoriser**.
1. Cette opération lance la phase de déploiement. Attendez qu’elle se termine.

   > **Remarque** : en cas d’échec du déploiement en raison d’un problème lié à la syntaxe du pipeline YAML, utilisez le code suivant comme référence :

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
           PathtoPublish: '$(Build.SourcesDirectory)/infra/webapp.bicep'
           ArtifactName: 'Bicep'
           publishLocation: 'Container'

    - stage: Deploy
      displayName: Deploy to an Azure Web App
      jobs:
      - job: Deploy
        pool:
          vmImage: 'windows-latest'
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
            AppSettings: '-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development'
   ```

#### Tâche 3 : examiner le site déployé

1. Revenez à la fenêtre du navigateur web qui affiche le portail Azure et accédez au panneau affichant les propriétés de l’application web Azure.
1. Dans le panneau de l’application web Azure, cliquez sur **Vue d’ensemble**, puis, dans le panneau Vue d’ensemble, cliquez sur **Parcourir** pour ouvrir votre site dans un nouvel onglet de navigateur web.
1. Vérifiez que le site déployé se charge comme prévu dans le nouvel onglet du navigateur, affichant le site web d’e-commerce eShopOnWeb.

### Exercice 2 : configurer des paramètres d’environnement pour des pipelines CI/CD en tant que code avec YAML dans Azure DevOps

Dans cet exercice, vous allez ajouter des approbations à un pipeline YAML dans Azure DevOps.

#### Tâche 1 : configurer des environnements de pipeline

Les pipelines YAML en tant que code n’ont pas de portes de mise en production/qualité, comme c’est le cas pour les pipelines de mise en production classiques Azure DevOps. Toutefois, certaines similitudes peuvent être configurées pour les pipelines YAML en tant que code à l’aide d’**environnements**. Dans cette tâche, vous allez utiliser ce mécanisme pour configurer des approbations pour la phase de génération.

1. À partir du projet Azure DevOps **eShopOnWeb_MultiStageYAML**, accédez à **Pipelines**.
1. Sous le menu Pipelines à gauche, sélectionnez **Environnements**.
1. Cliquez sur **Créer un environnement**.
1. Dans le volet **Nouvel environnement**, ajoutez un nom pour l’environnement, appelé **`approvals`**.
1. Sous **Ressources**, sélectionnez **Aucune**.
1. Confirmez les paramètres en appuyant sur le bouton **Créer**.
1. Une fois l’environnement créé, sélectionnez l’onglet **Approbations et contrôles** dans le nouvel environnement **approbations**.
1. Dans **Ajouter votre premier contrôle**, sélectionnez **Approbations**.
1. Ajoutez le nom de votre compte d’utilisateur Azure DevOps dans le champ **Approbateurs**.

   > **Remarque :** dans un scénario réel, il s’agirait du nom de l’équipe DevOps travaillant sur ce projet.

1. Confirmez les paramètres d’approbation définis en appuyant sur le bouton **Créer**.
1. Enfin, nous devons ajouter les paramètres « environment: approvals » nécessaires au code du pipeline YAML pour la phase de déploiement. Pour ce faire, accédez à **Dépôts**, accédez au dossier **.ado**, puis sélectionnez le fichier de pipeline en tant que code **eshoponweb-ci.yml**.
1. Dans l’affichage Contenu, cliquez sur le bouton **Modifier** pour basculer en mode d’édition.
1. Accédez au début du **travail de déploiement** (-job: Deploy à la ligne 60).
1. Ajoutez une nouvelle ligne vide juste en dessous, puis ajoutez l’extrait de code suivant :

   ```yaml
   environment: approvals
   ```

   L’extrait de code doit ressembler à ceci :

   ```yaml
   jobs:
     - job: Deploy
       environment: approvals
       pool:
         vmImage: "windows-latest"
   ```

1. Comme l’environnement est un paramètre spécifique d’une phase de déploiement, il ne peut pas être utilisé par des « travaux ». Par conséquent, nous devons apporter des modifications supplémentaires à la définition de travail actuelle.
1. À la ligne **60**, renommez « - job: Deploy » en **- deployment: Deploy**.
1. Ensuite, sous la ligne **63** (vmImage: windows-latest), ajoutez une nouvelle ligne vide.
1. Collez l’extrait de code YAML suivant :

   ```yaml
   strategy:
     runOnce:
       deploy:
   ```

1. Sélectionnez l’extrait de code restant (ligne **67** jusqu’à la fin), puis utilisez la touche de **tabulation** pour corriger la mise en retrait YAML.

   L’extrait de code YAML résultant doit maintenant ressembler à ceci et tenir compte de l’**étape de déploiement** :

   ```yaml
   - stage: Deploy
     displayName: Deploy to an Azure Web App
     jobs:
       - deployment: Deploy
         environment: approvals
         pool:
           vmImage: "windows-latest"
         strategy:
           runOnce:
             deploy:
               steps:
                 - task: DownloadBuildArtifacts@1
                   inputs:
                     buildType: "current"
                     downloadType: "single"
                     artifactName: "Website"
                     downloadPath: "$(Build.ArtifactStagingDirectory)"
                 - task: AzureRmWebAppDeployment@4
                   inputs:
                     ConnectionType: "AzureRM"
                     azureSubscription: "AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)"
                     appType: "webApp"
                     WebAppName: "eshoponWebYAML369825031"
                     packageForLinux: "$(Build.ArtifactStagingDirectory)/**/Web.zip"
                     AppSettings: "-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development"
   ```

1. Confirmez les modifications apportées au fichier YAML du code en cliquant sur **Valider**, puis à nouveau sur **Valider** dans le volet de validation qui s’affiche.
1. Accédez au menu du projet Azure DevOps à gauche, sélectionnez **Pipelines**, **Pipelines** et remarquez la présence du pipeline **EshopOnWeb_MultiStageYAML** utilisé précédemment.
1. Ouvrez le pipeline.
1. Cliquez sur **Exécuter le pipeline** pour déclencher une nouvelle exécution de pipeline. Confirmez l’opération en cliquant sur **Exécuter**.
1. Comme auparavant, la phase de génération démarre comme prévu. Attendez la fin de cette phase.
1. Ensuite, étant donné que nous avons configuré _environment: approvals_ pour la phase de déploiement, l’opération demande une confirmation d’approbation avant de se lancer.
1. Cette opération est visible dans l’affichage Pipeline, qui indique **En attente (1 vérification en cours)**. Un message de notification s’affiche également en indiquant que **1 approbation doit être examinée avant que cette exécution puisse se poursuivre vers Déploiement sur une application web Azure**.
1. Cliquez sur le bouton **Vérifier** en regard de ce message.
1. Dans le volet d’affichage **En attente de révision**, cliquez sur le bouton **Approuver**.
1. Cette opération permet à la phase de déploiement de se lancer et de déployer le code source Azure Web App.

   > **Remarque :** bien que cet exemple utilise uniquement les approbations, sachez que les autres vérifications telles qu’Azure Monitor, l’API REST, etc., peuvent être utilisées de manière similaire.

   > [!IMPORTANT]
   > N’oubliez pas de supprimer les ressources créées dans le portail Azure pour éviter les modifications inutiles.

## Révision

Dans ce labo, vous avez configuré des pipelines CI/CD en tant que code avec YAML dans Azure DevOps.
