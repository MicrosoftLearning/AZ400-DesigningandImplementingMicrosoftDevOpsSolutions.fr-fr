---
lab:
  title: "Déploiement de conteneurs Docker sur des applications web Azure\_App\_Service"
  module: 'Module 03: Implement CI with Azure Pipelines and GitHub Actions'
---

# Déploiement de conteneurs Docker sur des applications web Azure App Service

# Manuel de labo pour l’étudiant

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://learn.microsoft.com/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Créer une organisation ou une collection de projets](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization?view=azure-devops).

- Identifier un abonnement Azure existant ou en créer un.

- Vérifier que vous disposez d’un compte Microsoft ou d’un compte Azure AD avec le rôle Propriétaire ou Contributeur dans l’abonnement Azure. Pour plus d’informations, consultez [Répertorier les attributions de rôle Azure à l’aide du portail Azure](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) et [Afficher et attribuer des rôles d’administrateur dans Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Présentation du labo

Dans ce labo, vous allez découvrir comment utiliser un pipeline CI/CD Azure DevOps pour générer une image Docker personnalisée, l’envoyer sur Azure Container Registry et la déployer en tant que conteneur sur Azure App Service.

## Objectifs

À la fin de ce labo, vous serez en mesure d’effectuer les tâches suivants :

- Générez une image Docker personnalisée à l’aide d’un agent Linux hébergé par Microsoft.
- Poussez (push) une image vers Azure Container Registry.
- Déployez une image Docker en tant que conteneur dans Azure App Service à l’aide d’Azure DevOps.

## Durée estimée : 30 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

Dans cet exercice, vous allez configurer les prérequis pour le labo, qui se composent d’un nouveau projet Azure DevOps avec un référentiel basé sur [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tâche 1 : (passer si terminée) créer et configurer le projet d’équipe

Dans cette tâche, vous allez créer un projet Azure DevOps **eShopOnWeb** à utiliser par plusieurs labos.

1.  Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps. Cliquez sur **Nouveau projet**. Donnez au projet le nom **eShopOnWeb**, puis choisissez **Scrum** dans la liste déroulante **Processus d’élément de travail**. Sélectionnez **Créer**.

#### Tâche 2 : (passer si terminée) importer un référentiel Git eShopOnWeb

Dans cette tâche, vous allez importer le référentiel Git eShopOnWeb qui sera utilisé par plusieurs labos.

1.  Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps et le projet **eShopOnWeb** créé précédemment. Cliquez sur **Dépôts > Fichiers**, **Importer**. Dans la fenêtre **Importer un dépôt Git**, collez l’URL https://github.com/MicrosoftLearning/eShopOnWeb.git, puis cliquez sur **Importer** : 

1.  Le référentiel est organisé de la manière suivante :
    - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
    - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
    - Le dossier **.azure** contient l’infrastructure Bicep&ARM en tant que modèles de code utilisés dans certains scénarios de labo.
    - Conteneur de dossiers **.github** pour les définitions de flux de travail GitHub.
    - Le dossier **src** contient le site web .NET 6 utilisé dans les scénarios de labo.

#### Tâche 3 : (passer si terminée) définir la branche principale en tant que branche par défaut

1. Accédez à **Référentiels > Branches**.
1. Pointez sur la branche **principale**, puis cliquez sur les points de suspension à droite de la colonne.
1. Cliquez sur **Définir en tant que branche par défaut**.

### Exercice 1 : gérer la connexion de service

Dans cet exercice, vous allez configurer la connexion de service avec votre abonnement Azure, puis importer et exécuter le pipeline CI.

#### Tâche 1 : (ignorer si terminé) gérer la connexion de service

Vous pouvez créer une connexion à partir d’Azure Pipelines vers des services externes et distants pour les tâches s’exécutant dans un travail.

Dans cette tâche, vous allez créer un principal de service à l’aide d’Azure CLI, ce qui permettra à Azure DevOps d’effectuer les tâches suivantes :
- Déployer des ressources sur votre abonnement Azure
- Envoyer l’image Docker vers Azure Container Registry
- Ajouter une attribution de rôle pour autoriser Azure App Service à extraire l’image Docker d’Azure Container Registry

> **Remarque** : si vous avez déjà un principal de service, vous pouvez passer directement à la tâche suivante.

Vous aurez besoin d’un principal de service pour déployer des ressources Azure à partir d’Azure Pipelines.

Un principal de service est créé automatiquement par Azure Pipeline lorsque vous vous connectez à un abonnement Azure à partir d’une définition de pipeline ou lorsque vous créez une connexion de service à partir de la page des paramètres du projet (option automatique). Vous pouvez également créer manuellement le principal de service à partir du portail ou à l’aide d’Azure CLI et le réutiliser dans les projets. 

1.  Sur l’ordinateur de labo, démarrez un navigateur web, accédez au [**portail Azure**](https://portal.azure.com), puis connectez-vous avec le compte d’utilisateur qui a le rôle Propriétaire dans l’abonnement Azure utilisé dans ce labo et qui a le rôle Administrateur général dans le locataire Azure AD associé à cet abonnement.
1.  Dans le portail Azure, ouvrez le volet **Cloud Shell** situé directement à droite de la zone de texte de recherche en haut de la page. 
1.  Si vous êtes invité à sélectionner **Bash** ou **PowerShell**, sélectionnez **Bash**. 

   >**Remarque** : si c’est la première fois que vous démarrez **Cloud Shell** et que vous voyez le message **Vous n’avez aucun stockage monté**, sélectionnez l’abonnement que vous utilisez dans ce labo, puis sélectionnez **Créer un stockage**. 

1.  À partir de l’invite **Bash**, dans le volet **Cloud Shell**, exécutez les commandes suivantes pour récupérer les valeurs de l’attribut d’ID d’abonnement Azure : 

    ```
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **Remarque** : copiez les deux valeurs dans un fichier texte. Vous en aurez besoin plus tard dans ce labo.

1.  À partir de l’invite **Bash**, dans le volet **Cloud Shell**, exécutez la commande suivante pour créer un principal de service :

    ```
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **Remarque** : la commande génère une sortie JSON. Copiez la sortie dans un fichier texte. Vous en aurez besoin plus tard dans ce laboratoire.

1. Ensuite, sur l’ordinateur de labo, démarrez un navigateur web et accédez au projet Azure DevOps **eShopOnWeb**. Cliquez sur **Paramètres du projet > Connexions de service (sous Pipelines)**, puis sur **Nouvelle connexion de service**.

1. Dans le panneau **Nouvelle connexion de service**, sélectionnez **Azure Resource Manager**, puis **Suivant** (vous devrez peut-être faire défiler la page vers le bas).

1. Choisissez ensuite **Principal de service (manuel)**, puis cliquez sur **Suivant**.

1. Renseignez les champs vides à l’aide des informations collectées lors des étapes précédentes :
    - ID et nom de l’abonnement
    - ID du principal de service (ou clientId), clé (ou mot de passe) et TenantId.
    - Dans **Nom de la connexion de service**, saisissez **azure-connection**. Ce nom est référencé dans les pipelines YAML lorsque vous avez besoin d’une connexion de service Azure DevOps pour communiquer avec votre abonnement Azure.

1. Cliquez sur **Vérifier et enregistrer**.

### Exercice 2 : importer et exécuter le pipeline CI

Dans cet exercice, vous allez importer et exécuter le pipeline CI.

#### Tâche 1 : importer et exécuter le pipeline CI

1. Accédez à **Pipelines > Pipelines**.

1. Cliquez sur le bouton **Nouveau pipeline**.

1. Sélectionnez **Azure Repos Git (YAML)**.

1. Sélectionnez le référentiel **eShopOnWeb**.

1. Sélectionnez **Fichier YAML Azure Pipelines existant**.

1. Sélectionnez le fichier **/.ado/eshoponweb-ci-docker.yml**, puis cliquez sur **Continuer**.

1. Dans la définition de pipeline YAML, personnalisez :
- **YOUR-SUBSCRIPTION-ID** en fonction de votre ID d’abonnement Azure.
- **rg-az400-container-NAME** avec le nom du groupe de ressources qui sera créé par le pipeline (il peut également s’agir d’un groupe de ressources existant).

1. Cliquez sur **Enregistrer et exécuter**, puis attendez que le pipeline s’exécute correctement.

    > **Remarque** : le déploiement peut prendre quelques minutes.

    La définition CI se compose des tâches suivantes :
    - **Ressources** : télécharge les fichiers du référentiel qui seront utilisés dans les tâches suivantes.
    - **AzureResourceManagerTemplateDeployment** : déploie Azure Container Registry à l’aide du modèle bicep.
    - **PowerShell** : récupérer la valeur du **Serveur de connexion ACR** de la sortie de la tâche précédente et créer un paramètre **acrLoginServer**.
    - [**Docker**](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/docker-v0?view=azure-pipelines)**- Build** : générer l’image Docker et créer deux balises (dernier et BuildID actuel).
    - **Docker - Envoyer** : envoyer l’image Docker vers Azure Container Registry.

1. Votre pipeline est nommé en fonction du nom du projet. **Renommons**-le pour mieux l’identifier. Accédez à **Pipelines > Pipelines**, puis cliquez sur le pipeline qui vient d’être créé. Cliquez sur les points de suspension et sur l’option **Renommer/déplacer**. Nommez-le **eshoponweb-ci-docker**, puis cliquez sur **Enregistrer**.

1. Accédez au [**portail Azure**](https://portal.azure.com), recherchez Azure Container Registry dans le groupe de ressources récemment créé (il doit être nommé **rg-az400-container-NAME**). Assurez-vous que **eshoponweb/web** a été créé et contient deux balises (l’une d’elles est **Plus récent**).

### Exercice 3 : importer et exécuter le pipeline CD

Dans cet exercice, vous allez configurer la connexion de service avec votre abonnement Azure, puis importer et exécuter le pipeline CD.

#### Tâche 1 : ajouter une attribution de rôle

Dans cette tâche, vous allez ajouter une attribution de rôle pour permettre à Azure App Service d’extraire l’image Docker d’Azure Container Registry.

1. Accédez au [**portail Azure**](https://portal.azure.com).
1. Dans le portail Azure, ouvrez le volet **Cloud Shell** situé directement à droite de la zone de texte de recherche en haut de la page. 
1. Si vous êtes invité à sélectionner **Bash** ou **PowerShell**, sélectionnez **Bash**. 

1. À partir de l’invite **Bash**, dans le volet **Cloud Shell**, exécutez les commandes suivantes pour récupérer les valeurs de l’attribut d’ID d’abonnement Azure : 

    ```sh
    spId=$(az ad sp list --display-name sp-az400-azdo --query "[].id" --output tsv)
    echo $spId
    roleName=$(az role definition list --name "User Access Administrator" --query [0].name --output tsv)
    echo $roleName
    ```

1. Après avoir obtenu l’ID du principal de service et le nom du rôle, nous allons créer l’attribution de rôle en exécutant cette commande (remplacez **rg-az400-container-NAME** par le nom de votre groupe de ressources).

    ```sh
    az role assignment create --assignee $spId --role $roleName --resource-group "rg-az400-container-NAME"
    ```

Vous devez maintenant voir la sortie JSON qui confirme la réussite de l’exécution de la commande.

#### Tâche 2 : importer et exécuter le pipeline CD

Dans cette tâche, vous allez importer et exécuter le pipeline CI.

1. Accédez à **Pipelines > Pipelines**.

1. Cliquez sur le bouton **Nouveau pipeline**.

1. Sélectionnez **Azure Repos Git (YAML)**.

1. Sélectionnez le référentiel **eShopOnWeb**.

1. Sélectionnez **Fichier YAML Azure Pipelines existant**.

1. Sélectionnez le fichier **/.ado/eshoponweb-cd-webapp-docker.yml**, puis cliquez sur **Continuer**.

1. Dans la définition de pipeline YAML, personnalisez :
- **YOUR-SUBSCRIPTION-ID** en fonction de votre ID d’abonnement Azure.
- **rg-az400-container-NAME** avec le nom du groupe de ressources défini précédemment dans le labo.

1. Cliquez sur **Enregistrer et exécuter**, puis attendez que le pipeline s’exécute correctement.

    > **Remarque** : le déploiement peut prendre quelques minutes.

    La définition CI se compose des tâches suivantes :
    - **Ressources** : télécharge les fichiers du référentiel qui seront utilisés dans les tâches suivantes.
    - **AzureResourceManagerTemplateDeployment** : déploie Azure App Service à l’aide du modèle bicep.
    - **AzureResourceManagerTemplateDeployment** : ajouter des attributions de rôle à l’aide de Bicep.

1. Votre pipeline est nommé en fonction du nom du projet. **Renommons**-le pour mieux l’identifier. Accédez à **Pipelines > Pipelines** et pointez sur le pipeline récemment créé. Cliquez sur les points de suspension et sur l’option **Renommer/déplacer**. Nommez-le **eshoponweb-cd-webapp-docker**, puis cliquez sur **Enregistrer**.

    > **Remarque 1** : l’utilisation du modèle **/.azure/bicep/webapp-docker.bicep** crée un plan App Service, une application web avec l’identité managée affectée par le système activée et fait référence à l’image Docker envoyée précédemment : **${acr.properties.loginServer}/eshoponweb/web :latest**.

    > **Remarque 2** : l’utilisation du modèle **/.azure/bicep/webapp-to-acr-roleassignment.bicep** crée une attribution de rôle pour l’application web avec le rôle AcrPull pour pouvoir récupérer l’image Docker. Cela peut être effectué dans le premier modèle, mais étant donné que la propagation de l’attribution de rôle peut prendre un certain temps, il est judicieux d’effectuer les deux tâches séparément.

#### Tâche 3 : tester la solution

1. Dans le portail Azure, accédez au groupe de ressources récemment créé. Vous devez maintenant voir trois ressources (App Service, Plan App Service et Container Registry).

1. Accédez à App Service, puis cliquez sur **Parcourir**. Vous accédez au site web.

Félicitations ! Dans cet exercice, vous avez déployé un site web à l’aide d’une image Docker personnalisée.

### Exercice 4 : Supprimer les ressources du labo Azure.

Dans cet exercice, vous allez supprimer les ressources Azure approvisionnées dans ce labo pour éviter des frais inattendus.

>**Remarque** : N’oubliez pas de supprimer toutes les nouvelles ressources Azure que vous n’utilisez plus. La suppression des ressources inutilisées vous évitera d’encourir des frais inattendus.

#### Tâche 1 : supprimer les ressources du labo Azure

Dans cette tâche, vous allez utiliser Azure Cloud Shell pour supprimer les ressources Azure approvisionnées dans ce labo pour éviter des frais inutiles.

1. Dans le Portail Azure, ouvrez la session shell **Bash** dans le volet **Cloud Shell**.
1. Listez tous les groupes de ressources créés dans les labos de ce module en exécutant la commande suivante :

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-container-')].name" --output tsv
    ```

1. Supprimez tous les groupes de ressources que vous avez créés dans les labos de ce module en exécutant la commande suivante :

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-container-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Remarque** : La commande s’exécute de façon asynchrone (comme déterminé par le paramètre --no-wait). Par conséquent, vous serez en mesure d’exécuter une autre commande Azure CLI immédiatement après au cours de la même session Bash, mais la suppression réelle du groupe de ressources prendra quelques minutes.

## Révision

Dans ce labo, vous avez découvert comment utiliser un pipeline CI/CD Azure DevOps pour générer une image Docker personnalisée, l’envoyer vers Azure Container Registry et la déployer en tant que conteneur sur Azure App Service.
