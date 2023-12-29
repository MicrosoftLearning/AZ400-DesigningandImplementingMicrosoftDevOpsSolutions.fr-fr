---
lab:
  title: Déploiements à l’aide de modèles Azure Bicep
  module: 'Module 06: Manage infrastructure as code using Azure and DSC'
---

# Déploiements à l’aide de modèles Azure Bicep

## Manuel de labo de l’étudiant

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Créer une organisation ou une collection de projets](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifier un abonnement Azure existant ou en créer un.

- Vérifiez que vous disposez d’un compte Microsoft ou d’un compte Microsoft Entra avec le rôle Propriétaire dans l’abonnement Azure et le rôle Administrateur général dans le locataire Microsoft Entra associé à l’abonnement Azure. Pour plus d’informations, consultez [Répertorier les attributions de rôle Azure à l’aide du portail Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) et [Afficher et attribuer des rôles d’administrateur dans Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Présentation du labo

Dans ce labo, vous allez créer un modèle Azure Bicep et le modulariser en utilisant le concept de modules d’Azure Bicep. Vous allez ensuite modifier le modèle de déploiement principal pour utiliser le module, puis déployer toutes les ressources sur Azure.

## Objectifs

À la fin de ce labo, vous serez en mesure d’accomplir les tâches suivantes :

- Comprendre la structure d’un modèle Azure Bicep.
- Créer un module Bicep réutilisable.
- Modifier le modèle principal pour utiliser le module.
- Déployer toutes les ressources sur Azure à l’aide de pipelines YAML Azure.

## Durée estimée : 45 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

Dans cet exercice, vous configurerez les prérequis du labo, qui se composent d’un nouveau projet Azure DevOps comprenant un référentiel basé sur [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tâche 1 : (à ignorer si vous l’avez déjà effectuée) créer et configurer le projet d’équipe

Dans le cadre de cette tâche, vous créerez un projet Azure DevOps **eShopOnWeb** qui sera utilisé dans plusieurs labos.

1. Sur votre ordinateur de labo, ouvrez votre organisation Azure DevOps dans votre navigateur. Cliquez sur **Nouveau projet**. Nommez votre projet **eShopOnWeb** et ne modifiez pas les autres champs par défaut. Cliquez sur **Créer**.

    ![Création d’un projet](images/create-project.png)

#### Tâche 2 : (à ignorer si vous l’avez déjà effectuée) importer un référentiel Git eShopOnWeb

Dans le cadre de cette tâche, vous importerez le référentiel Git eShopOnWeb qui sera utilisé dans plusieurs labos.

1. Sur votre ordinateur de labo, ouvrez votre organisation Azure DevOps dans votre navigateur ainsi que le projet **eShopOnWeb** que vous venez de créer. Cliquez sur **Repos > Fichiers**, puis sur **Importer un référentiel**. Cliquez sur **Importer**. Dans la fenêtre **Importer un référentiel Git**, collez l’URL https://github.com/MicrosoftLearning/eShopOnWeb.git, puis cliquez sur **Importer** :

    ![Importer un référentiel](images/import-repo.png)

1. Le référentiel est organisé de la manière suivante :
    - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
    - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
    - Le dossier **.azure** contient l’infrastructure Bicep&ARM en tant que modèles de code utilisés dans certains scénarios de labos.
    - **Définitions de workflow GitHub YAML du conteneur de dossiers .github**.
    - Le dossier **src** contient le site web .NET 7 utilisé dans les scénarios de labo.

### Exercice 1 : comprendre un modèle Azure Bicep et le simplifier à l’aide d’un module réutilisable

Dans le cadre de ce labo, vous examinerez un modèle Azure Bicep et le simplifierez à l’aide d’un module réutilisable.

#### Tâche 1 : créer un modèle Azure Bicep

Dans le cadre de cette tâche, vous utiliserez Visual Studio Code pour créer un modèle Azure Bicep.

1. Dans l’onglet du navigateur où votre projet Azure DevOps est ouvert, accédez à **Repos**, puis aux **fichiers**. Recherchez le dossier `.azure\bicep` et ouvrez le fichier `simple-windows-vm.bicep`.

   ![Fichier Simple-windows-vm.bicep](./images/m06/browsebicepfile.png)

1. Passez en revue le modèle pour mieux comprendre sa structure. Il se compose de plusieurs paramètres avec des types, des valeurs par défaut et des validations, plusieurs variables et plusieurs ressources des types suivants :

   - Microsoft.Storage/storageAccounts
   - Microsoft.Network/publicIPAddresses
   - Microsoft.Network/virtualNetworks
   - Microsoft.Network/networkInterfaces
   - Microsoft.Compute/virtualMachines

1. Portez une attention particulière à la simplicité des définitions de ressources et à la possibilité de référencer implicitement des noms symboliques au lieu de noms explicites `dependsOn` dans le modèle.

#### Tâche 2 : créer un module Bicep réutilisable pour les ressources de stockage

Dans le cadre de cette tâche, vous créerez un module de modèle de stockage **storage.bicep** qui créera un compte de stockage uniquement et sera importé par le modèle principal. Le module de modèle de stockage doit transmettre une valeur au modèle principal **main.bicep**. Cette valeur sera définie dans l’élément de sortie du module de modèle de stockage.

1. Tout d’abord, il faut supprimer la ressource de stockage du modèle principal. Dans le coin supérieur droit de la fenêtre du navigateur, cliquez sur le bouton **Modifier** :

   ![Bouton Modifier](./images/m06/edit.png)

1. Supprimez maintenant la ressource de stockage :

   ```bicep
   resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'Storage'
   }
   ```

1. Validez le fichier. Vous allez toutefois continuer à l’utiliser.

   ![Valider le fichier](./images/m06/commit.png)

1. Survolez ensuite le dossier bicep avec votre curseur, puis cliquez sur l’icône des points de suspension. Sélectionnez ensuite **Nouveau**, puis **Fichier**. Indiquez le nom **storage.bicep** et cliquez sur **Créer**.

   ![Menu Nouveau fichier](./images/m06/newfile.png)

1. Copiez maintenant l’extrait de code suivant dans le fichier et validez vos modifications :

   ```bicep
   @description('Location for all resources.')
   param location string = resourceGroup().location

   @description('Name for the storage account.')
   param storageAccountName string

   resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'Storage'
   }

   output storageURI string = storageAccount.properties.primaryEndpoints.blob
   ```

#### Tâche 3 : modifier le modèle principal de manière à utiliser l’exemple de module

Dans le cadre de cette tâche, vous modifierez le modèle principal de manière à référencer le module de modèle que vous avez créé lors de la tâche précédente.

1. Revenez au fichier `simple-windows-vm.bicep` et cliquez une nouvelle fois sur le bouton **Modifier**.

1. Ajoutez ensuite le code suivant après les variables :

   ```bicep
   module storageModule './storage.bicep' = {
     name: 'linkedTemplate'
     params: {
       location: location
       storageAccountName: storageAccountName
     }
   }
   ```

1. Vous devez également modifier la référence à l’URI de l’objet blob du compte de stockage dans votre ressource d’ordinateur virtuel pour utiliser la sortie du module à la place. Recherchez la ressource d’ordinateur virtuel et remplacez la section diagnosticsProfile par les éléments suivants :

   ```bicep
   diagnosticsProfile: {
     bootDiagnostics: {
       enabled: true
       storageUri: storageModule.outputs.storageURI
     }
   }
   ```

1. Passez les informations suivantes en revue dans le modèle principal :

   - Un module du modèle principal est utilisé pour créer le lien avec un autre modèle.
   - Le nom symbolique du module est `storageModule`. Ce nom est utilisé pour configurer toutes les dépendances.
   - Seul le mode de déploiement **Incrémentiel** peut être utilisé avec les modules de modèle.
   - Un chemin d’accès relatif est utilisé dans votre module de modèle.
   - Utilisez les paramètres pour transmettre les valeurs du modèle principal aux modules de modèle.

1. Validez le modèle.

### Exercice 2 : déployer les modèles sur Azure à l’aide de pipelines YAML

Dans le cadre de ce labo, vous créerez une connexion de services et l’utiliserez dans un pipeline YAML Azure DevOps pour déployer votre modèle dans votre environnement Azure.

#### Tâche 1 : (à ignorer si vous l’avez déjà effectuée) créer une connexion de services pour le déploiement

Dans cette tâche, vous allez créer un principal de service à l’aide d’Azure CLI, ce qui permettra à Azure DevOps d’effectuer les actions suivantes :

- Déployer des ressources sur votre abonnement Azure.
- Disposez d’un accès en lecture sur les secrets Key Vault créés ultérieurement.

> **Remarque** : vous pouvez passer directement à la tâche suivante si vous possédez déjà un principal de service.

Vous aurez besoin d’un principal de service pour déployer des ressources Azure à partir d’Azure Pipelines. Étant donné que vous récupérerez des secrets dans un pipeline, vous devez octroyer l’autorisation au service lorsque vous créerez l’Azure Key Vault.

Un principal de service est créé automatiquement par Azure Pipelines lorsque vous vous connectez à un abonnement Azure à partir d’une définition de pipeline ou lorsque vous créez une connexion de service à partir de la page des paramètres du projet (option automatique). Vous pouvez également créer manuellement le principal de service à partir du portail ou à l’aide d’Azure CLI et le réutiliser dans les projets.

1. Sur l’ordinateur de labo, démarrez un navigateur web, accédez au [**Portail Azure**](https://portal.azure.com) et connectez-vous avec le compte d’utilisateur qui a le rôle de Propriétaire dans l’abonnement Azure que vous utiliserez dans ce labo et qui a le rôle d’Administrateur général dans le locataire Microsoft Entra associé à cet abonnement.
1. Dans le Portail Azure, cliquez sur l’icône **Cloud Shell** située directement à droite de la zone de texte de recherche en haut de la page.
1. Si vous êtes invité à sélectionner **Bash** ou **PowerShell**, sélectionnez **Bash**.

   >**Remarque** : si c’est la première fois que vous démarrez **Cloud Shell** et que vous voyez le message **Vous n’avez aucun stockage monté**, sélectionnez l’abonnement que vous utilisez dans ce labo, puis sélectionnez **Créer un stockage**.

1. À partir de l’invite **Bash**, dans le volet **Cloud Shell**, exécutez les commandes suivantes pour récupérer les valeurs de l’ID d’abonnement Azure et des attributs de nom d’abonnement :

    ```bash
    az account show --query id --output tsv
    az account show --query name --output tsv
    ```

    > **Remarque** : copiez les deux valeurs dans un fichier texte. Vous en aurez besoin plus tard dans ce labo.

1. À partir de l’invite **Bash**, dans le volet **Cloud Shell**, exécutez la commande suivante pour créer un principal de service (remplacez **myServicePrincipalName** par n’importe quelle chaîne unique de caractères composés de lettres et de chiffres) et **mySubscriptionID** par votre Azure subscriptionId :

    ```bash
    az ad sp create-for-rbac --name myServicePrincipalName \
                         --role contributor \
                         --scopes /subscriptions/mySubscriptionID
    ```

    > **Remarque** : la commande générera une sortie JSON. Copiez la sortie dans un fichier texte. Vous en aurez besoin plus tard dans ce labo.

1. Sur l’ordinateur du labo, démarrez ensuite un navigateur web et accédez au projet Azure DevOps **eShopOnWeb**. Cliquez sur **Paramètres du projet > Connexions de services (sous Pipelines)**, puis sélectionnez **Nouvelle connexion de services**.

    ![Nouvelle connexion de services](images/new-service-connection.png)

1. Dans le volet **Nouvelle connexion de service**, sélectionnez **Azure Resource Manager**, puis **Suivant** (il peut être nécessaire de faire défiler l’écran vers le bas).

1. Ensuite, sélectionnez **Principal de service (manuel)** puis sélectionnez **Suivant**.

1. Remplissez les champs vides à l’aide des informations collectées lors des étapes précédentes :
    - ID et nom de l’abonnement.
    - ID du principal de service (appId), clé du principal de service (mot de passe) et ID de locataire (locataire).
    - Dans **Nom de la connexion de service** tapez **azure subs**. Ce nom est référencé dans les pipelines YAML lorsque vous avez besoin d’une connexion de service Azure DevOps pour communiquer avec votre abonnement Azure.

    ![Connexion au service Azure](images/azure-service-connection.png)

1. Cliquez sur **Vérifier et enregistrer**.

#### Tâche 2 : déployer des ressources sur Azure avec des pipelines YAML
1. Revenez au volet **Pipelines** du hub **Pipelines**.
1. Dans la fenêtre **Créer votre premier pipeline**, cliquez sur **Créer un pipeline**.

    > **Remarque** : vous utiliserez l’assistant pour créer une définition de pipeline YAML basée sur votre projet.

1. Dans le volet **Où se trouve votre code ?**, sélectionnez l’option **Azure Repos Git (YAML)**.
1. Dans le volet **Sélectionner un référentiel**, cliquez sur **EShopOnWeb**.
1. Faites défiler le volet **Configurer votre pipeline** et sélectionnez **Fichier YAML Azure Pipelines existant**.
1. Dans le volet **Sélectionner un fichier YAML existant**, spécifiez les paramètres suivants :
   - Branche : **main**
   - Chemin d’accès : **.ado/eshoponweb-cd-windows-cm.yml**
1. Cliquez sur **Continuer** pour enregistrer ces paramètres.
1. Dans la section des variables, nommez votre groupe de ressources, définissez son emplacement souhaité et remplacez la valeur de la connexion de services par l’une de vos connexions de services existantes que vous venez de créer.
1. Cliquez sur le bouton **Enregistrer et exécuter** dans le coin supérieur droit. Lorsque la boîte de dialogue de validation s’affiche, cliquez une nouvelle fois sur **Enregistrer et exécuter**.

   ![Enregistrer et exécuter le pipeline YAML après les modifications](./images/m06/saveandrun.png)

1. Attendez la fin du déploiement, puis passez en revue les résultats.
   ![Déploiement de ressources réussi sur Azure à l’aide de pipelines YAML](./images/m06/deploy.png)

#### Tâche 3 : supprimer les ressources du labo Azure

Dans le cadre de cette tâche, vous utiliserez Azure Cloud Shell pour supprimer les ressources Azure provisionnées dans ce labo afin d’éliminer les frais superflus.

1. Dans le Portail Azure, ouvrez la session shell **Bash** dans le volet **Cloud Shell**.
1. Supprimez tous les groupes de ressources que vous avez créés dans les labos de ce module en exécutant la commande suivante (remplacez le nom du groupe de ressources par le nom de votre choix) :

   ```bash
   az group list --query "[?starts_with(name,'AZ400-EWebShop-NAME')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

   > **Remarque** : La commande s’exécute de façon asynchrone (comme déterminé par le paramètre --no-wait). Par conséquent, vous serez en mesure d’exécuter une autre commande Azure CLI immédiatement après au cours de la même session Bash, mais la suppression réelle du groupe de ressources prendra quelques minutes.

## Révision

Dans le cadre de ce labo, vous avez appris à créer un modèle Azure Bicep, à le modulariser à l’aide d’un module de modèle, à modifier le modèle de déploiement principal pour utiliser le module et à mettre à jour les dépendances, puis à déployer les modèles sur Azure à l’aide de pipelines YAML.
