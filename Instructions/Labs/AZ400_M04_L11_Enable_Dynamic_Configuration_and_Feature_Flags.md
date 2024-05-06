---
lab:
  title: Activer les indicateurs de fonctionnalité de configuration dynamiques
  module: 'Module 04: Implement a secure continuous deployment using Azure Pipelines'
---

# Activer les indicateurs de fonctionnalité de configuration dynamiques

## Manuel de labo de l’étudiant

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://learn.microsoft.com/azure/devops/server/compatibility).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Créer une organisation ou une collection de projets](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization?view=azure-devops).

- Identifier un abonnement Azure existant ou en créer un.

- Vérifiez que vous disposez d'un compte Microsoft ou d'un compte Microsoft Entra avec le rôle Contributeur ou Propriétaire dans l'abonnement Azure. Pour plus d’informations, consultez [Répertorier les attributions de rôle Azure à l’aide du portail Azure](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) et [Afficher et attribuer des rôles d’administrateur dans Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Présentation du labo

[Azure App Configuration](https://learn.microsoft.com/azure/azure-app-configuration/overview) est un service permettant de gérer de manière centralisée les paramètres d’application et les indicateurs de fonctionnalités. Les programmes modernes, en particulier ceux qui s’exécutent dans un cloud, ont généralement de nombreux composants distribués. La répartition des paramètres de configuration sur tous ces composants peut rendre les erreurs difficiles à corriger pendant le déploiement d’une application. Utilisez App Configuration pour stocker tous les paramètres de votre application et sécuriser leur accès dans un même endroit.

## Objectifs

À la fin de ce labo, vous serez en mesure d’effectuer les tâches suivantes :

- Activer la configuration dynamique.
- Gérer les indicateurs de fonctionnalité.

## Durée estimée : 60 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

Dans cet exercice, vous allez configurer les prérequis pour le labo, qui se composent d’un nouveau projet Azure DevOps avec un référentiel basé sur [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tâche 1 : (passer si terminée) créer et configurer le projet d’équipe

Dans cette tâche, vous allez créer un projet Azure DevOps **eShopOnWeb** à utiliser par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps. Cliquez sur **Nouveau projet**. Donnez au projet le nom **eShopOnWeb**, puis choisissez **Scrum** dans la liste déroulante **Processus d’élément de travail**. Cliquez sur **Créer**.

#### Tâche 2 : (passer si terminée) importer le référentiel Git eShopOnWeb

Dans cette tâche, vous allez importer le référentiel Git eShopOnWeb qui sera utilisé par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps et le projet **eShopOnWeb** créé précédemment. Cliquez sur **Dépôts > Fichiers**, **Importer**. Dans la fenêtre **Importer un dépôt Git**, collez l’URL <https://github.com/MicrosoftLearning/eShopOnWeb.git>, puis cliquez sur **Importer** :

1. Le référentiel est organisé de la manière suivante :
    - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
    - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
    - Le dossier **infra** contient l’infrastructure Bicep&ARM sous forme de modèles de code utilisés dans certains scénarios de labo.
    - **Définitions de workflow GitHub YAML du conteneur de dossiers .github**.
    - Le dossier **src** contient le site web .NET 8 utilisé dans les scénarios de labo.

#### Tâche 3 : (à ignorer si vous l’avez déjà effectuée) définir la branche principale en tant que branche par défaut

1. Accédez à **Dépôts > Branches**.
1. Pointez sur la branche **principale**, puis cliquez sur les points de suspension à droite de la colonne.
1. Cliquez sur **Définir comme branche par défaut**.

### Exercice 1 : (passer si terminé) importer et exécuter des pipelines CI/CD

Dans cet exercice, vous allez importer et exécuter le pipeline CI, configurer la connexion de service avec votre abonnement Azure, puis importer et exécuter le pipeline CD.

#### Tâche 1 : importer et exécuter le pipeline CI

Commençons par importer le pipeline CI nommé [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Accédez à **Pipelines > Pipelines**.
1. Cliquez sur le bouton **Créer un pipeline** (s’il n’y a pas de pipelines) ou **Nouveau pipeline** (si des pipelines ont déjà été créés).
1. Sélectionnez **Azure Repos Git (YAML)**.
1. Sélectionnez le référentiel **eShopOnWeb**.
1. Sélectionnez **Fichier YAML Azure Pipelines existant**.
1. Sélectionnez la branche **principale** et le fichier **/.ado/eshoponweb-ci.yml**, puis cliquez sur **Continuer**.
1. Cliquez sur le bouton **Run** (Exécuter) pour exécuter le pipeline.
1. Votre pipeline est nommé en fonction du nom du projet. **Renommons**-le pour mieux l’identifier. Accédez à **Pipelines > Pipelines**, puis cliquez sur le pipeline qui vient d’être créé. Cliquez sur les points de suspension, puis sur l’option **Renommer/Supprimer**. Nommez-le **eshoponweb-ci**, puis cliquez sur **Enregistrer**.

#### Tâche 2 : gérer la connexion de service

Vous pouvez créer une connexion à partir d’Azure Pipelines vers des services externes et distants pour les tâches s’exécutant dans un travail.

Dans cette tâche, vous allez créer un principal de service à l’aide d’Azure CLI, ce qui permettra à Azure DevOps d’effectuer les tâches suivantes :

- Déployer des ressources sur votre abonnement Azure
- Déployer l’application eShopOnWeb

> **Remarque** : si vous avez déjà un principal de service, vous pouvez passer directement à la tâche suivante.

Vous aurez besoin d’un principal de service pour déployer des ressources Azure à partir d’Azure Pipelines.

Un principal de service est créé automatiquement par Azure Pipeline lorsque vous vous connectez à un abonnement Azure à partir d’une définition de pipeline ou lorsque vous créez une connexion de service à partir de la page des paramètres du projet (option automatique). Vous pouvez également créer manuellement le principal de service à partir du portail ou à l’aide d’Azure CLI et le réutiliser dans les projets.

1. Sur l’ordinateur de labo, démarrez un navigateur web, accédez au [**portail Azure**](https://portal.azure.com) et connectez-vous avec le compte d’utilisateur qui a le rôle Propriétaire dans l’abonnement Azure que vous utiliserez dans ce labo et le rôle Administrateur général dans le locataire Microsoft Entra associé à cet abonnement.
1. Dans le portail Azure, ouvrez le volet **Cloud Shell** situé directement à droite de la zone de texte de recherche en haut de la page.
1. Si vous êtes invité à sélectionner **Bash** ou **PowerShell**, sélectionnez **Bash**.

   >**Remarque** : si c’est la première fois que vous démarrez **Cloud Shell** et que vous voyez le message **Vous n’avez aucun stockage monté**, sélectionnez l’abonnement que vous utilisez dans ce labo, puis sélectionnez **Créer un stockage**.

1. À partir de l’invite **Bash**, dans le volet **Cloud Shell**, exécutez les commandes suivantes pour récupérer les valeurs de l’attribut d’ID d’abonnement Azure :

    ```sh
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **Remarque** : copiez les deux valeurs dans un fichier texte. Vous en aurez besoin plus tard dans ce labo.

1. À partir de l’invite **Bash**, dans le volet **Cloud Shell**, exécutez la commande suivante pour créer un principal de service :

    ```sh
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **Remarque** : la commande génère une sortie JSON. Copiez la sortie dans un fichier texte. Vous en aurez besoin plus tard dans ce labo.

1. Ensuite, sur l’ordinateur de labo, démarrez un navigateur web et accédez au projet Azure DevOps **eShopOnWeb**. Cliquez sur **Paramètres du projet > Connexions de service (sous Pipelines)**, puis sur **Nouvelle connexion de service**.

1. Dans le panneau **Nouvelle connexion de service**, sélectionnez **Azure Resource Manager**, puis **Suivant** (vous devrez peut-être faire défiler la page vers le bas).

1. Choisissez ensuite **Principal de service (manuel)**, puis cliquez sur **Suivant**.

1. Renseignez les champs vides à l’aide des informations collectées lors des étapes précédentes :
    - ID et nom de l’abonnement
    - ID du principal de service (ou clientId), clé (ou mot de passe) et TenantId.
    - Dans le champ **Nom de connexion de service**, tapez **azure subs**. Ce nom est référencé dans les pipelines YAML lorsque vous avez besoin d’une connexion de service Azure DevOps pour communiquer avec votre abonnement Azure.

1. Cliquez sur **Vérifier et enregistrer**.

#### Tâche 3 : importer et exécuter le pipeline CD

Importons le pipeline CD nommé [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. Accédez à **Pipelines > Pipelines**.
1. Cliquez sur le bouton **Nouveau pipeline**.
1. Sélectionnez **Azure Repos Git (YAML)**.
1. Sélectionnez le référentiel **eShopOnWeb**.
1. Sélectionnez **Fichier YAML Azure Pipelines existant**.
1. Sélectionnez la branche **principale** et le fichier **/.ado/eshoponweb-cd-webapp-code.yml**, puis cliquez sur **Continuer**.
1. Dans la définition de pipeline YAML, personnalisez :
   - **YOUR-SUBSCRIPTION-ID** en fonction de votre ID d’abonnement Azure.
   - **az400eshop-NAME** remplacez NAME pour le rendre globalement unique.
   - **AZ400-EWebShop-NAME** en fonction du nom du groupe de ressources défini auparavant dans le labo.

1. Cliquez sur **Enregistrer et exécuter**, puis attendez que l’exécution du pipeline se termine.

    > **Remarque** : le déploiement peut prendre quelques minutes.

    La définition CD se compose des tâches suivantes :
    - **Ressources** : cette tâche est préparée pour se déclencher automatiquement en fonction de l’achèvement du pipeline CI. Elle télécharge également le référentiel pour le fichier bicep.
    - **AzureResourceManagerTemplateDeployment** : déploie l’application web Azure à l’aide du modèle bicep.

1. Votre pipeline est nommé en fonction du nom du projet. **Renommons**-le pour mieux l’identifier. Accédez à **Pipelines > Pipelines**, puis cliquez sur le pipeline qui vient d’être créé. Cliquez sur les points de suspension, puis sur l’option **Renommer/Supprimer**. Nommez-le **eshoponweb-cd-webapp-code**, puis cliquez sur **Enregistrer**.

### Exercice 2 : gérer Azure App Configuration

Dans cet exercice, vous allez créer la ressource App Configuration dans Azure, activer l’identité managée, puis tester la solution complète.

> **Remarque** : cet exercice ne nécessite aucune compétence de codage. Le code du site web implémente déjà les fonctionnalités Azure App Configuration.

Si vous souhaitez savoir comment les implémenter dans votre application, consultez les tutoriels [Utiliser la configuration dynamique dans une application ASP.NET Core](https://learn.microsoft.com/azure/azure-app-configuration/enable-dynamic-configuration-aspnet-core) et [Gérer les indicateurs de fonctionnalités dans Azure App Configuration](https://learn.microsoft.com/azure/azure-app-configuration/manage-feature-flags).

#### Tâche 1 : créer la ressource App Configuration

1. Dans le portail Azure, recherchez le service **App Configuration**.
1. Cliquez sur **Créer une configuration d’application**, puis sélectionnez :
    - Votre abonnement Azure.
    - Le groupe de ressources créé précédemment (il doit être nommé **AZ400-EWebShop-NAME**).
    - Emplacement.
    - Un nom unique tel que **appcs-NAME-REGION**, par exemple.
    - Sélectionnez le niveau de tarification **Gratuit**.
1. Cliquez sur **Vérifier + créer**, puis sur **Créer**.
1. Après avoir créé le service App Configuration, accédez à **Vue d’ensemble** et copiez/enregistrez la valeur du **point de terminaison**.

#### Tâche 2 : activer l’identité managée

1. Accédez à l’application web déployée à l’aide du pipeline (elle doit être nommée **az400-webapp-NAME**).
1. Dans la section **Paramètres**, cliquez sur **Identité**. Dans la section **Affecté par le système**, basculez l’état sur **Activé**, cliquez sur **Enregistrer > Oui**, puis attendez quelques secondes pour que l’opération se termine.
1. Revenez au service App Configuration, cliquez sur **Contrôle d’accès**, puis sur **Ajout de l’attribution de rôle**.
1. Dans la section **Rôle**, sélectionnez **Lecteur des données App Configuration**.
1. Dans la section **Membres**, activez l’option **Gérer l’identité**, puis sélectionnez l’identité managée de votre application web (elles doivent avoir le même nom).
1. Cliquez sur **Vérifier + attribuer**.

#### Tâche 3 : configurer l’application web

Pour vous assurer que votre site web accède à App Configuration, vous devez mettre à jour sa configuration.

1. Revenez à votre application web.
1. Dans la section **Paramètres**, cliquez sur **Configuration**.
1. Ajoutez deux nouveaux paramètres d’application :
    - Premier paramètre d’application
        - **Nom :** UseAppConfig
        - **Valeur** : true
    - Deuxième paramètre d’application
        - **Nom :** AppConfigEndpoint
        - **Valeur :***valeur enregistrée/copiée précédemment à partir du point de terminaison App Configuration. Elle doit ressembler à <https://appcs-NAME-REGION.azconfig.io>*

1. Cliquez sur **Ok**, puis **enregistrez** et attendez que les paramètres soient mis à jour.
1. Accédez à **Vue d’ensemble**, puis cliquez sur **Parcourir**.
1. À ce stade, vous ne constaterez aucune modification dans le site web, car App Configuration ne contient pas de données. C’est ce que vous allez faire dans les tâches suivantes.

#### Tâche 4 : tester la gestion de la configuration

1. Dans votre site web, dans la liste déroulante **Marque**, sélectionnez **Visual Studio**, puis cliquez sur le bouton fléché (**>**).
1. Le message *« AUCUN RÉSULTAT NE CORRESPOND À VOTRE RECHERCHE »* s’affiche. L’objectif de ce laboratoire est de pouvoir mettre à jour cette valeur sans mettre à jour le code du site web ou le redéployer.
1. Pour essayer, revenez à App Configuration.
1. Dans la section **Opérations**, sélectionnez **Explorateur de configurations**.
1. Cliquez sur **Créer > Clé-valeur**, puis ajoutez :
    - **Clé :** eShopWeb:Settings:NoResultsMessage
    - **Valeur :** *tapez votre message personnalisé*
1. Cliquez sur **Appliquer**, revenez à votre site web, puis actualisez la page.
1. Votre nouveau message devrait s’afficher à la place de l’ancienne valeur par défaut.

Félicitations ! Dans cette tâche, vous avez testé l’**explorateur de configurations** dans Azure App Configuration.

#### Tâche 5 : tester l’indicateur de fonctionnalité

Continuons à tester le Gestionnaire de fonctionnalités.

1. Pour essayer, revenez à App Configuration.
1. Dans la section **Opérations**, sélectionnez **Gestionnaire de fonctionnalités**.
1. Cliquez sur **Créer**, puis ajoutez :
    - **Activer l’indicateur de fonctionnalité :** option activée
    - **Nom de l’indicateur de fonctionnalités :** SalesWeekend
1. Cliquez sur **Appliquer**, revenez à votre site web, puis actualisez la page.
1. Vous devriez voir une image avec le texte « ALL T-SHIRTS ON SALE THIS WEEKEND ».
1. Si vous désactivez cette fonctionnalité dans App Configuration, l’image disparaîtrait.

Félicitations ! Dans cette tâche, vous avez testé le **Gestionnaire de fonctionnalités** dans Azure App Configuration.

### Exercice 3 : supprimer les ressources du labo Azure

Dans cet exercice, vous allez supprimer les ressources Azure approvisionnées dans ce labo pour éviter des frais inattendus.

>**Remarque** : N’oubliez pas de supprimer toutes les nouvelles ressources Azure que vous n’utilisez plus. La suppression des ressources inutilisées vous évitera d’encourir des frais inattendus.

#### Tâche 1 : supprimer les ressources du labo Azure

Dans cette tâche, vous allez utiliser Azure Cloud Shell pour supprimer les ressources Azure approvisionnées dans ce labo pour éviter des frais inutiles.

1. Dans le Portail Azure, ouvrez la session shell **Bash** dans le volet **Cloud Shell**.
1. Listez tous les groupes de ressources créés dans les labos de ce module en exécutant la commande suivante :

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].name" --output tsv
    ```

1. Supprimez tous les groupes de ressources que vous avez créés dans les labos de ce module en exécutant la commande suivante :

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Remarque** : La commande s’exécute de façon asynchrone (comme déterminé par le paramètre --no-wait). Par conséquent, vous serez en mesure d’exécuter une autre commande Azure CLI immédiatement après au cours de la même session Bash, mais la suppression réelle du groupe de ressources prendra quelques minutes.

## Révision

Dans ce labo, vous avez appris à activer dynamiquement la configuration et à gérer les indicateurs de fonctionnalité.
