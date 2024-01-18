---
lab:
  title: Déploiements à l’aide de modèles Azure Bicep
  module: 'Module 06: Manage infrastructure as code using Azure and DSC'
---

# Déploiements à l’aide de modèles Azure Bicep

# Manuel de labo pour l’étudiant

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Créer une organisation ou une collection de projets](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops).

- Identifier un abonnement Azure existant ou en créer un.

- Vérifiez que vous avez un compte Microsoft ou un compte Azure AD avec le rôle Propriétaire dans l’abonnement Azure, et le rôle Administrateur général dans le locataire Azure AD associé à l’abonnement Azure.

- [Visual Studio Code](https://code.visualstudio.com/). L’application sera installée dans le cadre des prérequis de ce labo.

## Présentation du labo

Dans ce labo, vous allez créer un modèle Azure Bicep et le modulariser en utilisant le concept de modules d’Azure Bicep. Vous allez ensuite modifier le modèle de déploiement principal pour utiliser le module, puis déployer toutes les ressources sur Azure.

## Objectifs

À la fin de ce labo, vous serez en mesure d’effectuer les tâches suivantes :

- Comprendre et créer des modèles Azure Bicep.
- Créer un module Bicep réutilisable pour les ressources de stockage.
- Charger le modèle lié à dans le Stockage Blob Azure et générer un jeton SAS
- Modifier le modèle principal pour utiliser le module.
- Modifier le modèle principal pour mettre à jour les dépendances
- Déployer toutes les ressources sur Azure en utilisant des modèles Azure Bicep.

## Durée estimée : 60 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

Dans cet exercice, vous allez configurer les prérequis du labo, y compris Visual Studio Code.

#### Tâche 1 : installer et configurer Git et Visual Studio Code

Dans cette tâche, vous allez installer Visual Studio Code. Si vous avez déjà implémenté ce prérequis, passez directement à la tâche suivante.

1. Dans le cas contraire, sur l’ordinateur de labo, accédez à la [page de téléchargement de Visual Studio Code](https://code.visualstudio.com/) dans la fenêtre du navigateur web, téléchargez l’application et installez-la.

### Exercice 1 : créer et déployer des modèles Azure Bicep

Dans ce labo, vous allez créer un modèle Azure Bicep et un exemple de module. Vous modifierez ensuite le modèle de déploiement principal pour utiliser l’exemple de module et mettre à jour les dépendances, puis vous déploierez les modèles sur Azure.

#### Tâche 1 : créer un modèle Azure Bicep

Dans cette tâche, vous allez utiliser Visual Studio Code pour créer un modèle Azure Bicep.

1. Sur l’ordinateur de labo, démarrez Visual Studio Code et cliquez sur le menu de niveau supérieur **Fichier**. Dans le menu déroulant, sélectionnez **Préférences**. Dans le menu en cascade, sélectionnez **Extensions**. Dans la zone de texte **Rechercher des extensions**, tapez **Bicep** et sélectionnez celle publiée par Microsoft, puis cliquez sur **Installer** pour installer la prise en charge linguistique Azure Bicep.
1. Dans un navigateur web, connectez-vous à **<https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-simple-windows/main.bicep>**. Cliquez sur l’option **Raw** du fichier. Copiez le contenu de la fenêtre de code et collez-le dans l’Éditeur Visual Studio Code.

   > **Remarque** : au lieu de créer un modèle à partir de zéro, nous allons utiliser l’un des [modèles de démarrage rapide Azure](https://azure.microsoft.com/en-us/resources/templates/) nommé **Déployer une machine virtuelle de modèle Windows simple**. Les modèles sont téléchargeables à partir de GitHub - [vm-simple-windows](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-simple-windows).

1. Sur votre ordinateur de labo, ouvrez l'Explorateur de fichiers et créez le dossier local suivant dédié au stockage des modèles :

   - **C:\\templates**

1. Revenez à la fenêtre Visual Studio Code qui contient notre modèle main.bicep et cliquez sur le menu de niveau supérieur **Fichier**. Dans le menu déroulant, cliquez sur **Enregistrer sous**, puis enregistrez le modèle en tant que **main.bicep** dans le dossier local **C:\\templates** nouvellement créé.
1. Passez en revue le modèle pour mieux comprendre sa structure. Il existe cinq types de ressources inclues dans le modèle :

   - Microsoft.Storage/storageAccounts
   - Microsoft.Network/publicIPAddresses
   - Microsoft.Network/virtualNetworks
   - Microsoft.Network/networkInterfaces
   - Microsoft.Compute/virtualMachines

1. Dans Visual Studio Code, réenregistrez le fichier, mais cette fois, choisissez **C:\\templates** comme destination et **storage.bicep** comme nom de fichier.

   > **Remarque** : nous avons maintenant deux fichiers JSON identiques : **C:\\templates\\main.bicep** et **C:\\templates\\storage.bicep**.

#### Tâche 2 : créer un module de modèle pour les ressources de stockage

Dans cette tâche, vous allez modifier les modèles que vous avez enregistrés dans la tâche précédente afin que le module de modèle de stockage **storage.bicep ** crée un compte de stockage uniquement. Il sera importé par le premier modèle. Le module de modèle de stockage doit transmettre une valeur au modèle principal, **main.bicep**. Cette valeur sera définie dans l’élément de sortie du module de modèle de stockage.

1. Dans le fichier **storage.bicep** affiché dans la fenêtre Visual Studio Code, sous la **section des ressources**, supprimez tous les éléments de ressource à l’exception de la ressource **storageAccounts**. Vous devez obtenir une section de ressource semblable à celle-ci :

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

1. Supprimez ensuite toutes les définitions de variables :

   ```bicep
   var storageAccountName = 'bootdiags${uniqueString(resourceGroup().id)}'
   var nicName = 'myVMNic'
   var addressPrefix = '10.0.0.0/16'
   var subnetName = 'Subnet'
   var subnetPrefix = '10.0.0.0/24'
   var virtualNetworkName = 'MyVNET'
   var networkSecurityGroupName = 'default-NSG'
   var securityProfileJson = {
     uefiSettings: {
       secureBootEnabled: true
       vTpmEnabled: true
     }
     securityType: securityType
   }
   var extensionName = 'GuestAttestation'
   var extensionPublisher = 'Microsoft.Azure.Security.WindowsAttestation'
   var extensionVersion = '1.0'
   var maaTenantName = 'GuestAttestation'
   var maaEndpoint = substring('emptyString', 0, 0)
   ```

1. Supprimez ensuite toutes les valeurs de paramètre à l’exception de l’emplacement et ajoutez le code de paramètre suivant, ce qui donne ce résultat :

   ```bicep
   @description('Location for all resources.')
   param location string = resourceGroup().location

   @description('Name for the storage account.')
   param storageAccountName string
   ```

1. Ensuite, à la fin du fichier, supprimez la sortie actuelle et ajoutez-en une nouvelle appelée valeur de sortie storageURI. Modifiez la sortie pour qu’elle ressemble à celle ci-dessous.

   ```bicep
   output storageURI string = storageAccount.properties.primaryEndpoints.blob
   ```

1. Enregistrez le module de modèle storage.bicep. Le modèle de stockage doit maintenant ressembler à celui-ci :

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

1. Dans Visual Studio Code, cliquez sur le menu de niveau supérieur **Fichier**. Dans le menu déroulant, sélectionnez **Ouvrir un fichier**. Dans la boîte de dialogue Ouvrir un fichier, accédez à **C:\\templates\\main.bicep**, sélectionnez-le, puis cliquez sur **Ouvrir**.
1. Dans le fichier **main.bicep**, dans la section de ressource, supprimez l’élément de ressource de stockage.

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

1. Ajoutez ensuite le code suivant directement au même emplacement où se trouvait l’élément de ressource de stockage qui vient d’être supprimé :

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
   - Le module porte un nom symbolique : storageModule. Ce nom est utilisé pour configurer les dépendances.
   - Vous pouvez uniquement utiliser le mode de déploiement Incrémentiel lors de l’emploi de modules de modèles.
   - Un chemin relatif est utilisé pour votre module de modèle.
   - Utilisez des paramètres pour transmettre des valeurs du modèle principal aux modules de modèles.

> **Remarque** : avec les modèles Azure ARM, vous avez utilisé un compte de stockage pour charger le modèle lié afin de faciliter son utilisation par d’autres personnes. Avec les modules Azure Bicep, vous avez la possibilité de les charger dans le Registre de modules Azure Bicep qui dispose à la fois d’options de Registre public et privé. Vous trouverez plus d’informations dans la [documentation Azure Bicep](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/modules#file-in-registry).

1. Enregistrez le modèle.

#### Tâche 4 : déployer des ressources sur Azure à l’aide de modules de modèles

> **Remarque** : vous pouvez déployer des modèles de plusieurs façons, comme l’utilisation d’Azure CLI installée localement ou à partir d’Azure Cloud Shell ou d’un pipeline CI/CD. Dans ce labo, vous allez utiliser Azure CLI à partir d’Azure Cloud Shell.

> **Remarque** : contrairement aux modèles ARM, vous ne pouvez pas utiliser le portail Azure pour déployer directement des modèles Bicep.

> **Remarque** : pour utiliser Azure Cloud Shell, vous allez charger les fichiers main.bicep et storage.bicep dans le répertoire de base de Cloud Shell.

> **Remarque** : actuellement, Azure CLI ne prend pas en charge le déploiement de fichiers bicep distants. Vous pouvez générer les fichiers bicep pour obtenir le JSON du modèle ARM, les charger dans un compte de stockage, puis les déployer à distance.

1. Sur l’ordinateur de labo, dans le navigateur web affichant le portail Azure, cliquez sur l’icône **Cloud Shell** pour ouvrir Cloud Shell.
   > **Remarque** : si vous disposez dans cet exercice d’une session PowerShell antérieure toujours active, basculez vers Bash (étape suivante).
1. Dans le menu déroulant du volet Cloud Shell, sélectionnez **PowerShell**, cliquez sur **Bash** puis, lorsque vous y êtes invité, cliquez sur **Confirmer**.
1. Dans le volet Cloud Shell, cliquez sur l'icône **Charger/télécharger des fichiers** puis, dans le menu déroulant, cliquez sur **Charger**.
1. Dans la boîte de dialogue **Ouvrir**, accédez à C:\\templates\\**main.bicep**, sélectionnez-le, puis cliquez sur **Ouvrir**.
1. Suivez les mêmes étapes pour charger également le fichier **C:\\templates\\storage.bicep**.
1. À partir d’une session **Bash** dans le volet Cloud Shell, exécutez ce qui suit pour effectuer un déploiement à l’aide d’un modèle nouvellement chargé :

   ```bash
   az deployment group what-if --name az400m06l15deployment --resource-group az400m06l15-RG --template-file main.bicep
   ```

1. Lorsque vous êtes invité à fournir la valeur de « adminUsername », tapez **Student** et appuyez sur la touche **Entrée**.
1. Lorsque vous êtes invité à fournir la valeur de « adminPassword », tapez **Pa55w.rd1234** et appuyez sur la touche **Entrée**. (La saisie du mot de passe ne s’affiche pas.)
1. Passez en revue le résultat de cette commande qui valide votre déploiement et vous indique s’il existe des erreurs dans vos modèles. Cela est très utile, plus particulièrement lors du déploiement de modèles avec de nombreuses ressources et dans des environnements cloud critiques pour l’entreprise.

1. À partir d’une session **Bash** dans le volet Cloud Shell, exécutez ce qui suit pour effectuer un déploiement à l’aide d’un modèle nouvellement chargé :

   ```bash
   LOCATION='<region>'
   ```
   > **Remarque** : remplacez le nom de la région par une région proche de votre emplacement. Si vous ne savez pas quels emplacements sont disponibles, exécutez la commande `az account list-locations -o table`.
  
   ```bash
   az group create --name az400m06l15-RG --location $LOCATION
   ```

   ```bash   
   az deployment group create --name az400m06l15deployment --resource-group az400m06l15-RG --template-file main.bicep
   ```

1. Lorsque vous êtes invité à fournir la valeur de « adminUsername », tapez **Student** et appuyez sur la touche **Entrée**.
1. Lorsque vous êtes invité à fournir la valeur de « adminPassword », tapez **Pa55w.rd1234** et appuyez sur la touche **Entrée**. (La saisie du mot de passe ne s’affiche pas.)

1. Si vous recevez des erreurs lors de l’exécution de la commande ci-dessus pour déployer le modèle, essayez ce qui suit :

   - Si vous avez plusieurs abonnements Azure, vérifiez que vous avez défini le contexte d’abonnement sur celui qui convient, dans lequel le groupe de ressources est déployé.
   - Vérifiez que le modèle lié est accessible via l’URI que vous avez spécifié.

> **Remarque** : à l’étape suivante, vous pouvez désormais modulariser les définitions de ressources restantes dans le modèle de déploiement principal, telles que les définitions de ressources réseau et de machine virtuelle.

> **Remarque** : si vous n’envisagez pas d’utiliser les ressources déployées, vous devez les supprimer pour éviter les frais associés. Pour ce faire, vous pouvez simplement supprimer le groupe de ressources **az400m06l15-RG**.

### Exercice 2 : supprimer les ressources du labo Azure

Dans cet exercice, vous allez supprimer les ressources Azure approvisionnées dans ce labo pour éviter des frais inattendus.

> **Remarque** : N’oubliez pas de supprimer toutes les nouvelles ressources Azure que vous n’utilisez plus. La suppression des ressources inutilisées vous évitera d’encourir des frais inattendus.

#### Tâche 1 : supprimer les ressources du labo Azure

Dans cette tâche, vous allez utiliser Azure Cloud Shell pour supprimer les ressources Azure approvisionnées dans ce labo pour éviter des frais inutiles.

1. Dans le Portail Azure, ouvrez la session shell **Bash** dans le volet **Cloud Shell**.
1. Listez tous les groupes de ressources créés dans les labos de ce module en exécutant la commande suivante :

   ```bash
   az group list --query "[?starts_with(name,'az400m06l15-RG')].name" --output tsv
   ```

1. Supprimez tous les groupes de ressources que vous avez créés dans les labos de ce module en exécutant la commande suivante :

   ```bash
   az group list --query "[?starts_with(name,'az400m06l15-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

   > **Remarque** : La commande s’exécute de façon asynchrone (comme déterminé par le paramètre --no-wait). Par conséquent, vous serez en mesure d’exécuter une autre commande Azure CLI immédiatement après au cours de la même session Bash, mais la suppression réelle du groupe de ressources prendra quelques minutes.

## Révision

Dans ce labo, vous avez appris à créer un modèle Azure Resource Manager, à le modulariser à l’aide d’un modèle lié, à modifier le modèle de déploiement principal pour appeler le modèle lié et à mettre à jour les dépendances, puis à déployer les modèles sur Azure.
