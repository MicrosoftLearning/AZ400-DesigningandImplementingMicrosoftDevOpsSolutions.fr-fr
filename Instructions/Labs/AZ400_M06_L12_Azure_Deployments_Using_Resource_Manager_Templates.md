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

Dans ce labo, vous allez créer un modèle Azure Bicep et le modulariser en utilisant le concept de modules d’Azure Bicep. Vous allez ensuite modifier le modèle de déploiement principal pour utiliser le module, puis déployer toutes les ressources sur Azure.

## Objectifs

À la fin de ce labo, vous serez en mesure d’effectuer les tâches suivants :

- Comprendre la structure d’un modèle Azure Bicep.
- Créer un module Bicep réutilisable.
- Modifier le modèle principal pour utiliser le module.
- Déployer toutes les ressources sur Azure à l’aide de pipelines YAML Azure.

## Durée estimée : 45 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

Dans cet exercice, vous allez configurer les prérequis pour le labo, qui se composent d’un nouveau projet Azure DevOps avec un référentiel basé sur [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tâche 1 : (passer si terminée) créer et configurer le projet d’équipe

Dans cette tâche, vous allez créer un projet Azure DevOps **eShopOnWeb** à utiliser par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps. Cliquez sur **Nouveau projet**. Attribuez au projet le nom **eShopOnWeb** et conservez les valeurs par défaut des autres champs. Cliquez sur **Créer**.

    ![Création d’un projet](images/create-project.png)

#### Tâche 2 : (passer si terminée) importer le référentiel Git eShopOnWeb

Dans cette tâche, vous allez importer le référentiel Git eShopOnWeb qui sera utilisé par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps et le projet **eShopOnWeb** créé précédemment. Cliquez sur **Dépôts > Fichiers**, **Importer un dépôt**. Cliquez sur **Importer**. Dans la fenêtre **Importer un dépôt Git**, collez l’URL <https://github.com/MicrosoftLearning/eShopOnWeb.git>, puis cliquez sur **Importer** :

    ![Importer un référentiel](images/import-repo.png)

1. Le référentiel est organisé de la manière suivante :
    - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
    - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
    - Le dossier **infra** contient l’infrastructure Bicep&ARM en tant que modèles de code utilisés dans certains scénarios de labo.
    - **Définitions de workflow GitHub YAML du conteneur de dossiers .github**.
    - Le dossier **src** contient le site web .NET 8 utilisé dans les scénarios de labo.

### Exercice 1 : comprendre un modèle Azure Bicep et le simplifier à l’aide d’un module réutilisable

Dans ce labo, vous allez passer en revue un modèle Azure Bicep et le simplifier à l’aide d’un module réutilisable.

#### Tâche 1 : créer un modèle Azure Bicep

Dans cette tâche, vous allez utiliser Visual Studio Code pour créer un modèle Azure Bicep.

1. Dans l’onglet de navigateur dans lequel votre projet Azure DevOps est ouvert, accédez à **Référentiels** et à **Fichiers**. Ouvrez le dossier `.azure\bicep` et recherchez le fichier `simple-windows-vm.bicep`.

   ![Fichier simple-windows-vm.bicep](./images/m06/browsebicepfile.png)

1. Passez en revue le modèle pour mieux comprendre sa structure. Il existe certains paramètres avec des types, des valeurs par défaut et une validation, certaines variables, et quelques ressources avec ces types :

   - Microsoft.Storage/storageAccounts
   - Microsoft.Network/publicIPAddresses
   - Microsoft.Network/virtualNetworks
   - Microsoft.Network/networkInterfaces
   - Microsoft.Compute/virtualMachines

1. Faites attention à la simplicité des définitions de ressources et à la possibilité de référencer implicitement des noms symboliques au lieu d’être explicite `dependsOn` dans le modèle.

#### Tâche 2 : créer un module Bicep pour les ressources de stockage

Dans cette tâche, vous allez créer un module de modèle **storage.bicep** qui créera un compte de stockage uniquement et que le modèle principal importera. Le module de modèle de stockage doit transmettre une valeur au modèle principal, **main.bicep**. Cette valeur sera définie dans l’élément de sortie du module de modèle de stockage.

1. Tout d’abord, nous devons supprimer la ressource de stockage de notre modèle principal. Dans l’angle supérieur droit de la fenêtre de votre navigateur, cliquez sur le bouton **Modifier** :

   ![Bouton Modifier](./images/m06/edit.png)

1. Supprimez maintenant la ressource de stockage  :

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

1. Validez le fichier ; toutefois, nous n’en avons pas encore fini avec lui.

   ![Validation du fichier](./images/m06/commit.png)

1. Ensuite, placez votre souris sur le dossier bicep, cliquez sur l’icône de sélection, puis sélectionnez **Nouveau** et **Fichier**. Entrez le nom **storage.bicep** et cliquez sur **Créer**.

   ![Nouveau menu Fichier](./images/m06/newfile.png)

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

#### Tâche 3 : modifier le modèle principal pour utiliser le module de modèle

Dans cette tâche, vous allez modifier le modèle principal pour référencer le module de modèle que vous avez créé dans la tâche précédente.

1. Revenez au fichier `simple-windows-vm.bicep`, puis cliquez de nouveau sur le bouton **Modifier**.

1. Ensuite, ajoutez le code suivant après les variables :

   ```bicep
   module storageModule './storage.bicep' = {
     name: 'linkedTemplate'
     params: {
       location: location
       storageAccountName: storageAccountName
     }
   }
   ```

1. Nous devons également modifier la référence à l’URI d’objet blob du compte de stockage dans notre ressource de machine virtuelle pour utiliser la sortie du module à la place. Recherchez la ressource de machine virtuelle et remplacez le contenu de la section diagnosticsProfile par les éléments suivants :

   ```bicep
   diagnosticsProfile: {
     bootDiagnostics: {
       enabled: true
       storageUri: storageModule.outputs.storageURI
     }
   }
   ```

1. Les informations suivantes sont demandées dans le modèle principal :

   - Un module dans le modèle principal est utilisé pour créer le lien vers un autre modèle.
   - Le module porte un nom symbolique, `storageModule`. Ce nom est utilisé pour configurer les dépendances.
   - Vous pouvez uniquement utiliser le mode de déploiement **Incrémentiel** lors de l’emploi de modules de modèles.
   - Un chemin relatif est utilisé pour votre module de modèle.
   - Utilisez des paramètres pour transmettre des valeurs du modèle principal aux modules de modèles.

1. Validez le modèle.

### Exercice 2 : déployer les modèles sur Azure en utilisant des pipelines YAML

Dans ce labo, vous allez créer une connexion de service et l’utiliser dans un pipeline YAML Azure DevOps pour déployer votre modèle dans votre environnement Azure.

#### Tâche 1 : (à ignorer si elle est terminée) créer une connexion de service pour le déploiement

Dans cette tâche, vous allez créer un principal de service à l’aide d’Azure CLI, ce qui permettra à Azure DevOps de :

- Déployer des ressources sur un abonnement Azure
- Disposer d’un accès en lecture sur les secrets du coffre de clés créés ultérieurement.

> **Remarque** : si vous disposez déjà d’un principal de service, vous pouvez passer directement à la tâche suivante.

Vous aurez besoin d’un principal de service pour déployer des ressources Azure depuis Azure Pipelines. Étant donné que nous allons récupérer des secrets dans un pipeline, nous devons accorder l’autorisation au service lors de la création du coffre de clés Azure.

Un principal de service est créé automatiquement par Azure Pipelines, lorsque vous vous connectez à un abonnement Azure depuis une définition de pipeline ou lorsque vous créez une nouvelle connexion de service depuis la page des paramètres du projet (option automatique). Vous pouvez également créer manuellement le principal de service à partir du portail ou à l’aide d’Azure CLI, et le réutiliser dans d’autres projets.

1. Sur l’ordinateur de labo, démarrez un navigateur web, accédez au [**portail Azure**](https://portal.azure.com) et connectez-vous avec le compte d’utilisateur qui a le rôle Propriétaire dans l’abonnement Azure que vous utiliserez dans ce labo et le rôle Administrateur général dans le locataire Microsoft Entra associé à cet abonnement.
1. Dans le portail Azure, ouvrez le volet **Cloud Shell** situé directement à droite de la zone de texte de recherche en haut de la page.
1. Si vous êtes invité à sélectionner **Bash** ou **PowerShell**, sélectionnez **Bash**.

   >**Remarque** : si c’est la première fois que vous démarrez **Cloud Shell** et que vous voyez le message **Vous n’avez aucun stockage monté**, sélectionnez l’abonnement que vous utilisez dans ce labo, puis sélectionnez **Créer un stockage**.

1. À partir de l’invite **Bash**, dans le volet **Cloud Shell**, exécutez les commandes suivantes pour récupérer les valeurs de l’ID d’abonnement Azure et des attributs de nom d’abonnement :

    ```bash
    az account show --query id --output tsv
    az account show --query name --output tsv
    ```

    > **Remarque** : copiez les deux valeurs dans un fichier texte. Vous en aurez besoin plus tard dans ce labo.

1. À partir de l’invite **Bash**, dans le volet **Cloud Shell**, exécutez la commande suivante pour créer un principal de service (remplacez **myServicePrincipalName**par une chaîne unique de caractères composés de lettres et de chiffres) et **mySubscriptionID** par votre ID d’abonnement Azure :

    ```bash
    az ad sp create-for-rbac --name myServicePrincipalName \
                         --role contributor \
                         --scopes /subscriptions/mySubscriptionID
    ```

    > **Remarque** : la commande génère une sortie JSON. Copiez la sortie dans un fichier texte. Vous en aurez besoin plus tard dans ce labo.

1. Ensuite, sur l’ordinateur de labo, démarrez un navigateur web et accédez au projet Azure DevOps **eShopOnWeb**. Cliquez sur **Paramètres du projet > Connexions de service (sous Pipelines)**, puis sur **Nouvelle connexion de service**.

    ![Nouvelle connexion de service](images/new-service-connection.png)

1. Dans le panneau **Nouvelle connexion de service**, sélectionnez **Azure Resource Manager**, puis **Suivant** (vous devrez peut-être faire défiler la page vers le bas).

1. Choisissez ensuite **Principal de service (manuel)**, puis cliquez sur **Suivant**.

1. Renseignez les champs vides à l’aide des informations collectées lors des étapes précédentes :
    - ID et nom de l’abonnement.
    - ID du principal de service (appId), clé du principal de service (password) et ID du locataire (tenant).
    - Dans le champ **Nom de connexion de service**, tapez **azure subs**. Ce nom est référencé dans les pipelines YAML lorsque vous avez besoin d’une connexion de service Azure DevOps pour communiquer avec votre abonnement Azure.

    ![Connexion au service Azure](images/azure-service-connection.png)

1. Cliquez sur **Vérifier et enregistrer**.

#### Tâche 2 : déployer des ressources sur Azure à l’aide de pipelines YAML

1. Revenez au volet **Pipelines** du hub **Pipelines**.
1. Dans la fenêtre **Créer votre premier pipeline**, cliquez sur **Créer un pipeline**.

    > **Remarque** : nous allons utiliser l’Assistant pour créer une définition de pipeline YAML basée sur notre projet.

1. Dans le volet **Où se trouve votre code ?**, cliquez sur l’option **Azure Repos Git (YAML)**.
1. Dans le volet **Sélectionner un référentiel**, cliquez sur **eShopOnWeb**.
1. Dans le volet **Configurer votre pipeline**, faites défiler la page vers le bas, puis sélectionnez **Fichier YAML Azure Pipelines existant**.
1. Dans le panneau **Sélectionner un fichier YAML existant**, spécifiez les paramètres suivants :
   - Branche : **main**
   - Chemin : **.ado/eshoponweb-cd-windows-cm.yml**
1. Cliquez sur **Continuer** pour enregistrer les paramètres.
1. Dans la section des variables, choisissez un nom pour votre groupe de ressources, définissez l’emplacement souhaité et remplacez la valeur de la connexion de service par l’une de vos connexions de service existantes que vous avez créées précédemment.
1. Dans le coin supérieur droit, cliquez sur le bouton **Enregistrer et exécuter**. Lorsque la boîte de dialogue de validation s’affiche, cliquez sur **Enregistrer et réexécuter**.

   ![Enregistrer et exécuter le pipeline YAML après avoir apporté des modifications](./images/m06/saveandrun.png)

1. Attendez que le déploiement se termine, puis consultez les résultats.
   ![Déploiement de ressources réussi sur Azure à l’aide de pipelines YAML](./images/m06/deploy.png)

#### Exercice 3 : supprimer les ressources de labo Azure

Dans cette tâche, vous allez utiliser Azure Cloud Shell pour supprimer les ressources Azure approvisionnées dans ce labo pour éviter des frais inutiles.

1. Dans le Portail Azure, ouvrez la session shell **Bash** dans le volet **Cloud Shell**.
1. Supprimez tous les groupes de ressources que vous avez créés dans les labos de ce module en exécutant la commande suivante (remplacez le nom du groupe de ressources par le nom de votre choix) :

   ```bash
   az group list --query "[?starts_with(name,'AZ400-EWebShop-NAME')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

   > **Remarque** : La commande s’exécute de façon asynchrone (comme déterminé par le paramètre --no-wait). Par conséquent, vous serez en mesure d’exécuter une autre commande Azure CLI immédiatement après au cours de la même session Bash, mais la suppression réelle du groupe de ressources prendra quelques minutes.

## Révision

Dans ce labo, vous avez appris à créer un modèle Azure Bicep, à le modulariser à l’aide d’un module de modèle, à modifier le modèle de déploiement principal pour utiliser le module et les dépendances mises à jour, et enfin à déployer les modèles sur Azure via des pipelines YAML.
