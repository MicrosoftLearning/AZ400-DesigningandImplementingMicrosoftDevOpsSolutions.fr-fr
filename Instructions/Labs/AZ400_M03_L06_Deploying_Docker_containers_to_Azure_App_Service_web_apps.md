---
lab:
  title: Déploiement de conteneurs Docker sur des applications web Azure App Service
  module: 'Module 03: Implement CI with Azure Pipelines and GitHub Actions'
---

# Déploiement de conteneurs Docker sur des applications web Azure App Service

## Manuel de labo de l’étudiant

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://learn.microsoft.com/azure/devops/server/compatibility).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Créer une organisation ou une collection de projets](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifier un abonnement Azure existant ou en créer un.

- Vérifiez que vous disposez d'un compte Microsoft ou d'un compte Microsoft Entra avec le rôle Contributeur ou Propriétaire dans l'abonnement Azure. Pour plus d’informations, consultez [Répertorier les attributions de rôle Azure à l’aide du portail Azure](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) et [Afficher et attribuer des rôles d’administrateur dans Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Présentation du labo

Dans ce labo, vous allez découvrir comment utiliser un pipeline CI/CD Azure DevOps pour générer une image Docker personnalisée, l’envoyer sur Azure Container Registry et la déployer en tant que conteneur sur Azure App Service.

## Objectifs

À la fin de ce labo, vous serez en mesure d’accomplir les tâches suivantes :

- Générez une image Docker personnalisée à l’aide d’un agent Linux hébergé par Microsoft.
- Poussez (push) une image vers Azure Container Registry.
- Déployez une image Docker en tant que conteneur dans Azure App Service à l’aide d’Azure DevOps.

## Durée estimée : 30 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

Dans cet exercice, vous configurerez les prérequis du labo, qui se composent d’un nouveau projet Azure DevOps comprenant un référentiel basé sur [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tâche 1 : (à ignorer si vous l’avez déjà effectuée) créer et configurer le projet d’équipe

Dans le cadre de cette tâche, vous créerez un projet Azure DevOps **eShopOnWeb** qui sera utilisé dans plusieurs labos.

1. Sur votre ordinateur de labo, ouvrez votre organisation Azure DevOps dans votre navigateur. Cliquez sur **Nouveau projet**. Nommez votre projet **eShopOnWeb** et sélectionnez **Scrum** dans la liste déroulante **Processus d’élément de travail**. Cliquez sur **Créer**.

#### Tâche 2 : (à ignorer si vous l’avez déjà effectuée) importer un référentiel Git eShopOnWeb

Dans le cadre de cette tâche, vous importerez le référentiel Git eShopOnWeb qui sera utilisé dans plusieurs labos.

1. Sur votre ordinateur de labo, ouvrez votre organisation Azure DevOps dans votre navigateur ainsi que le projet **eShopOnWeb** que vous venez de créer. Cliquez sur **Repos > Fichiers**, puis sur **Importer**. Dans la fenêtre **Importer un référentiel Git**, collez l’URL https://github.com/MicrosoftLearning/eShopOnWeb.git, puis cliquez sur **Importer** :

2. Le référentiel est organisé de la manière suivante :
    - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
    - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
    - Le dossier **.azure** contient l’infrastructure Bicep&ARM en tant que modèles de code utilisés dans certains scénarios de labos.
    - **Définitions de workflow GitHub YAML du conteneur de dossiers .github**.
    - Le dossier **src** contient le site web .NET 7 utilisé dans les scénarios de labo.

#### Tâche 3 : (à ignorer si vous l’avez déjà effectuée) définir la branche principale en tant que branche par défaut

1. Accédez à **Repos > Branches**.
2. Survolez la branche **principale** avec votre curseur, puis cliquez sur les points de suspension à droite de la colonne.
3. Cliquez sur **Définir en tant que branche par défaut**.

### Exercice 1 : gérer la connexion de services

Dans cet exercice, vous allez configurer la connexion de services avec votre abonnement Azure, puis importer et exécuter le pipeline CI.

#### Tâche 1 : (à ignorer si vous l’avez déjà effectuée) gérer la connexion de services

Vous pouvez créer une connexion à partir d’Azure Pipelines vers des services externes et distants pour les tâches s’exécutant dans un travail.

Dans cette tâche, vous allez créer un principal de service à l’aide d’Azure CLI, ce qui permettra à Azure DevOps d’effectuer les actions suivantes :

- Déployer des ressources sur votre abonnement Azure.
- Envoyer l’image Docker vers Azure Container Registry.
- Ajoutez une attribution de rôle pour permettre à Azure App Service d’extraire l’image Docker à partir d’Azure Container Registry.

> **Remarque** : vous pouvez passer directement à la tâche suivante si vous possédez déjà un principal de service.

Vous aurez besoin d’un principal de service pour déployer des ressources Azure à partir d’Azure Pipelines.

Un principal de service est créé automatiquement par Azure Pipelines lorsque vous vous connectez à un abonnement Azure à partir d’une définition de pipeline ou lorsque vous créez une connexion de service à partir de la page des paramètres du projet (option automatique). Vous pouvez également créer manuellement le principal de service à partir du portail ou à l’aide d’Azure CLI et le réutiliser dans les projets.

1. Sur l’ordinateur de labo, démarrez un navigateur web, accédez au [**Portail Azure**](https://portal.azure.com) et connectez-vous avec le compte d’utilisateur qui a le rôle de Propriétaire dans l’abonnement Azure que vous utiliserez dans ce labo et qui a le rôle d’Administrateur général dans le locataire Microsoft Entra associé à cet abonnement.
2. Dans le Portail Azure, cliquez sur l’icône **Cloud Shell** située directement à droite de la zone de texte de recherche en haut de la page.
3. Si vous êtes invité à sélectionner **Bash** ou **PowerShell**, sélectionnez **Bash**.

   >**Remarque** : si c’est la première fois que vous démarrez **Cloud Shell** et que vous voyez le message **Vous n’avez aucun stockage monté**, sélectionnez l’abonnement que vous utilisez dans ce labo, puis sélectionnez **Créer un stockage**.

4. À partir de l’invite **Bash**, dans le volet **Cloud Shell**, exécutez les commandes suivantes pour récupérer les valeurs de l’attribut d’ID d’abonnement Azure :

    ```bash
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **Remarque** : copiez les deux valeurs dans un fichier texte. Vous en aurez besoin plus tard dans ce labo.

5. À partir de l’invite **Bash** dans le volet **Cloud Shell**, exécutez la commande suivante pour créer un principal de service :

    ```bash
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **Remarque** : la commande générera une sortie JSON. Copiez la sortie dans un fichier texte. Vous en aurez besoin plus tard dans ce labo.

6. Sur l’ordinateur du labo, démarrez ensuite un navigateur web et accédez au projet Azure DevOps **eShopOnWeb**. Cliquez sur **Paramètres du projet > Connexions de services (sous Pipelines)**, puis sélectionnez **Nouvelle connexion de services**.
7. Dans le volet **Nouvelle connexion de service**, sélectionnez **Azure Resource Manager**, puis **Suivant** (il peut être nécessaire de faire défiler l’écran vers le bas).
8. Ensuite, sélectionnez **Principal de service (manuel)** puis sélectionnez **Suivant**.
9. Remplissez les champs vides à l’aide des informations collectées lors des étapes précédentes :
    - ID et nom de l’abonnement.
    - ID du principal de service (appId), clé du principal de service (mot de passe) et ID de locataire (locataire).
    - Saisissez **azure-** pour **Nom de la connexion de service**. Ce nom est référencé dans les pipelines YAML lorsque vous avez besoin d’une connexion de service Azure DevOps pour communiquer avec votre abonnement Azure.

10. Cliquez sur **Vérifier et enregistrer**.

### Exercice 2 : importer et exécuter le pipeline CI

Dans le cadre de cet exercice, vous importerez et exécuterez le pipeline CI.

#### Tâche 1 : importer et exécuter le pipeline CI

1. Accédez à **Pipelines > Pipelines**.
2. Cliquez sur le bouton **Nouveau pipeline**.
3. Sélectionnez **Azure Repos Git (YAML)**.
4. Sélectionnez le référentiel **eShopOnWeb**.
5. Sélectionnez **Fichier YAML Azure Pipelines existant**
6. Sélectionnez le fichier **/.ado/eshoponweb-ci-docker..yml**, puis cliquez sur **Continuer**.
7. Dans la définition du pipeline YAML, personnalisez les éléments suivants :
   - Remplacez **YOUR-SUBSCRIPTION-ID** par votre ID d’abonnement Azure.
   - Remplacez **rg-az400-container-NAME** par le nom du groupe de ressources qui sera créé par le pipeline (il peut également s’agir d’un groupe de ressources existant).

8. Cliquez sur **Enregistrer et exécuter**, puis attendez que le pipeline s’exécute correctement.

    > **Remarque** : le déploiement peut prendre quelques minutes.

    La définition CI se compose des tâches suivantes :
    - **Ressources** : téléchargement des fichiers de référentiel qui seront utilisés lors des tâches suivantes.
    - **AzureResourceManagerTemplateDeployment** : déploie l’Azure Container Registry à l’aide du modèle bicep.
    - **PowerShell** : récupère la valeur du **serveur d’ouverture de session ACR** à partir de la sortie de la tâche précédente et crée un paramètre **acrLoginServer**.
    - [**Docker**](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/docker-v0?view=azure-pipelines) **- Générer** : génère l’image Docker et crée deux étiquettes (BuildID le plus récent et BuildID actuel).
    - **Docker - Transmettre** : transmet les images à Azure Container Registry.

9. Votre pipeline choisira un nom en fonction du nom du projet. **Renommons**-le pour mieux identifier le pipeline. Accédez à **Pipelines > Pipelines** et sélectionnez le pipeline que vous venez de créer. Cliquez sur les points de suspension, puis sélectionnez l’option **Renommer/Déplacer**. Nommez-le **eshoponweb-ci-docker**, puis cliquez sur **Enregistrer**.

10. Dans le [**Portail Azure**](https://portal.azure.com), recherchez l’Azure Container Registry dans le groupe de ressources que vous venez de créer (son nom doit être **rg-az400-container-NAME**). À gauche de l’écran, cliquez sur **Référentiels** sous **Services**, et assurez-vous que le référentiel **eshoponweb/web** a bien été créé. Lorsque vous cliquez sur le lien du référentiel, deux étiquettes doivent apparaître (l’une d’elles est **Le plus récent**) : il s’agit des images transmises. Si elles n’apparaissent pas, vérifiez l’état de votre pipeline.

### Exercice 3 : importer et exécuter le pipeline CD

Dans cet exercice, vous allez configurer la connexion de services avec votre abonnement Azure, puis importer et exécuter le pipeline CD.

#### Tâche 1 : ajouter une attribution de rôle

Dans le cadre de cette tâche, vous ajouterez une attribution de rôle pour permettre à Azure App Service d’extraire l’image Docker à partir d’Azure Container Registry.

1. Accédez au [**portail Azure**](https://portal.azure.com).
2. Dans le Portail Azure, cliquez sur l’icône **Cloud Shell** située directement à droite de la zone de texte de recherche en haut de la page.
3. Si vous êtes invité à sélectionner **Bash** ou **PowerShell**, sélectionnez **Bash**.
4. À partir de l’invite **Bash**, dans le volet **Cloud Shell**, exécutez les commandes suivantes pour récupérer les valeurs de l’attribut d’ID d’abonnement Azure :

    ```sh
    spId=$(az ad sp list --display-name sp-az400-azdo --query "[].id" --output tsv)
    echo $spId
    roleName=$(az role definition list --name "User Access Administrator" --query "[0].name" --output tsv)
    echo $roleName
    ```

5. Après avoir obtenu l’ID du principal de service et le nom du rôle, créez l’attribution de rôle en exécutant cette commande (remplacez **rg-az400-container-NAME** par le nom de votre groupe de ressources) :

    ```sh
    az role assignment create --assignee $spId --role $roleName --resource-group "rg-az400-container-NAME"
    ```

La sortie JSON qui confirme la réussite de l’exécution de la commande doit apparaître à l’écran.

#### Tâche 2 : importer et exécuter le pipeline CD

Dans le cadre de cette tâche, vous importerez et exécuterez le pipeline CD.

1. Accédez à **Pipelines > Pipelines**.
2. Cliquez sur le bouton **Nouveau pipeline**.
3. Sélectionnez **Azure Repos Git (YAML)**.
4. Sélectionnez le référentiel **eShopOnWeb**.
5. Sélectionnez **Fichier YAML Azure Pipelines existant**.
6. Sélectionnez le fichier **/.ado/eshoponweb-cd-webapp-docker.yml**, puis cliquez sur **Continuer**.
7. Dans la définition du pipeline YAML, personnalisez les éléments suivants :
   - Remplacez **YOUR-SUBSCRIPTION-ID** par votre ID d’abonnement Azure.
   - Remplacez **rg-az400-container-NAME** par le nom du groupe de ressources que vous avez défini plus tôt dans le labo.

8. Cliquez sur **Enregistrer et exécuter**, puis attendez que le pipeline s’exécute correctement.

    > **Remarque** : le déploiement peut prendre quelques minutes.
    
    > **Important** : si vous recevez le message d’erreur « TF402455 : les transmissions de type push vers cette branche ne sont pas autorisées. Vous devez envoyer une demande de tirage (pull request) pour mettre à jour cette branche. », désactivez la règle « Exiger un nombre minimal de réviseurs » que vous avez activée dans les labos précédents.

    La définition CD se compose des tâches suivantes :
    - **Ressources** : téléchargement des fichiers de référentiel qui seront utilisés lors des tâches suivantes.
    - **AzureResourceManagerTemplateDeployment** : déploie l’Azure App Service à l’aide du modèle bicep.
    - **AzureResourceManagerTemplateDeployment** : ajoute une attribution de rôle à l’aide de Bicep.

9. Votre pipeline choisira un nom en fonction du nom du projet. **Renommons**-le pour mieux identifier le pipeline. Accédez à **Pipelines > Pipelines** et survolez le pipeline que vous venez de créer avec votre curseur. Cliquez sur les points de suspension, puis sélectionnez l’option **Renommer/Déplacer**. Nommez-le **eshoponweb-cd-webapp-docker**, puis cliquez sur **Enregistrer**.

    > **Remarque 1** : l’utilisateur du modèle **/.azure/bicep/webapp-docker.bicep** crée un plan App Service, une application web avec l’identité managée attribuée par le système activée, et référence l’image Docker que vous venez de transmettre : **${acr.properties.loginServer}/eshoponweb/web:latest**.

    > **Remarque 2** : utiliser le modèle **/.azure/bicep/webapp-to-acr-roleassignment.bicep** crée une nouvelle attribution de rôle pour l’application web, où le rôle AcrPull est capable de récupérer l’image Docker. Cette opération peut être effectuée dans le premier modèle, mais puisque l’attribution de rôle peut prendre un certain temps à se propager, il est judicieux d’effectuer les deux tâches séparément.

#### Tâche 3 : tester la solution

1. Dans le Portail Azure, accédez au groupe de ressources que vous venez de créer. Trois ressources doivent maintenant apparaître : App Service, plan App Service et Container Registry.

1. Accédez à App Service, puis cliquez sur **Parcourir** pour ouvrir le site web.

Félicitations ! Dans le cadre de cet exercice, vous avez déployé un site web à l’aide d’une image Docker personnalisée.

### Exercice 4 : Supprimer les ressources du labo Azure.

Dans le cadre de cet exercice, vous supprimerez les ressources Azure provisionnées dans ce labo pour éliminer les frais inattendus.

>**Remarque** : N’oubliez pas de supprimer toutes les nouvelles ressources Azure que vous n’utilisez plus. La suppression des ressources inutilisées vous évitera d’encourir des frais inattendus.

#### Tâche 1 : supprimer les ressources du labo Azure

Dans le cadre de cette tâche, vous utiliserez Azure Cloud Shell pour supprimer les ressources Azure provisionnées dans ce labo afin d’éliminer les frais superflus.

1. Dans le Portail Azure, ouvrez la session shell **Bash** dans le volet **Cloud Shell**.
2. Listez tous les groupes de ressources créés dans les labos de ce module en exécutant la commande suivante :

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-container-')].name" --output tsv
    ```

3. Supprimez tous les groupes de ressources que vous avez créés dans les labos de ce module en exécutant la commande suivante :

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-container-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Remarque** : La commande s’exécute de façon asynchrone (comme déterminé par le paramètre --no-wait). Par conséquent, vous serez en mesure d’exécuter une autre commande Azure CLI immédiatement après au cours de la même session Bash, mais la suppression réelle du groupe de ressources prendra quelques minutes.

## Révision

Dans ce labo, vous avez découvert comment utiliser un pipeline CI/CD Azure DevOps pour générer une image Docker personnalisée, l’envoyer sur Azure Container Registry et la déployer en tant que conteneur sur Azure App Service.
