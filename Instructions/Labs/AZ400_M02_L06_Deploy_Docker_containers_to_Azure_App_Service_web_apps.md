---
lab:
  title: Déployer des conteneurs Docker sur des applications web Azure App Service
  module: 'Module 02: Implement CI with Azure Pipelines and GitHub Actions'
---

# Déployer des conteneurs Docker sur des applications web Azure App Service

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://learn.microsoft.com/azure/devops/server/compatibility).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Créer une organisation ou une collection de projets](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifier un abonnement Azure existant ou en créer un.

- Vérifiez que vous disposez d'un compte Microsoft ou d'un compte Microsoft Entra avec le rôle Contributeur ou Propriétaire dans l'abonnement Azure. Pour plus d’informations, consultez [Répertorier les attributions de rôle Azure à l’aide du portail Azure](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) et [Afficher et attribuer des rôles d’administrateur dans Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Présentation du labo

Dans ce labo, vous allez découvrir comment utiliser un pipeline CI/CD Azure DevOps pour générer une image Docker personnalisée, l’envoyer sur Azure Container Registry et la déployer en tant que conteneur sur Azure App Service.

## Objectifs

À la fin de ce labo, vous serez en mesure d’effectuer les tâches suivants :

- Générez une image Docker personnalisée à l’aide d’un agent Linux hébergé par Microsoft.
- Poussez (push) une image vers Azure Container Registry.
- Déployez une image Docker en tant que conteneur dans Azure App Service à l’aide d’Azure DevOps.

## Durée estimée : 20 minutes

## Instructions

### Exercice 0 : (ignorer si terminé) configurer les prérequis du labo

Dans cet exercice, vous allez configurer les prérequis pour le labo, qui se composent d’un nouveau projet Azure DevOps avec un référentiel basé sur [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tâche 1 : (passer si terminée) créer et configurer le projet d’équipe

Dans cette tâche, vous allez créer un projet Azure DevOps **eShopOnWeb** à utiliser par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps. Cliquez sur **Nouveau projet**. Donnez au projet le nom **eShopOnWeb**, puis choisissez **Scrum** dans la liste déroulante **Processus d’élément de travail**. Cliquez sur **Créer**.

#### Tâche 2 : (passer si terminée) importer le référentiel Git eShopOnWeb

Dans cette tâche, vous allez importer le référentiel Git eShopOnWeb qui sera utilisé par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps et le projet **eShopOnWeb** créé précédemment. Cliquez sur **Dépôts > Fichiers**, **Importer**. Dans la fenêtre **Importer un dépôt Git**, collez l’URL <https://github.com/MicrosoftLearning/eShopOnWeb.git>, puis cliquez sur **Importer** :

1. Le référentiel est organisé de la manière suivante :
    - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
    - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
    - Le dossier **infra** contient l’infrastructure Bicep&ARM sous forme de modèles de code utilisés dans certains scénarios de labo.
    - **Définitions de workflow GitHub YAML du conteneur de dossiers .github**.
    - Le dossier **src** contient le site web .NET 8 utilisé dans les scénarios de labo.

#### Tâche 3 : (à ignorer si vous l’avez déjà effectuée) définir la branche principale en tant que branche par défaut

1. Accédez à **Dépôts > Branches**.
1. Pointez sur la branche **principale**, puis cliquez sur les points de suspension à droite de la colonne.
1. Cliquez sur **Définir comme branche par défaut**.

### Exercice 1 : importer et exécuter le pipeline CI

Dans cet exercice, vous allez configurer la connexion de service avec votre abonnement Azure, puis importer et exécuter le pipeline CI.

#### Tâche 1 : importer et exécuter le pipeline CI

1. Accédez à **Pipelines > Pipelines**.
1. Cliquez sur le bouton **Nouveau pipeline** (ou **Créer un pipeline** si vous n'avez pas d'autres pipelines précédemment créés)
1. Sélectionnez **Azure Repos Git (YAML)**.
1. Sélectionnez le référentiel **eShopOnWeb**.
1. Sélectionnez **Fichier YAML Azure Pipelines existant**
1. Sélectionnez la branche **principale** et le fichier **/.ado/eshoponweb-ci-docker.yml**, puis cliquez sur **Continuer**
1. Dans la définition de pipeline YAML, personnalisez :
   - **YOUR-SUBSCRIPTION-ID** avec votre ID d’abonnement Azure.
   - Remplacez le **resourceGroup** par le nom du groupe de ressources utilisé pendant la création de la connexion de service, par exemple **AZ400-RG1**.

1. Examinez la définition du pipeline. La définition CI se compose des tâches suivantes :
    - **Ressources** : télécharge les fichiers du référentiel qui seront utilisés dans les tâches suivantes.
    - **AzureResourceManagerTemplateDeployment** : déploie Azure Container Registry à l’aide du modèle bicep.
    - **PowerShell** : récupérer la valeur du **Serveur de connexion ACR** de la sortie de la tâche précédente et créer un paramètre **acrLoginServer**.
    - [**Docker**](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/docker-v0?view=azure-pipelines)**- Build** : générer l’image Docker et créer deux balises (Dernier et BuildID actuel).
    - **Docker - Envoyer** : envoyer l’image Docker vers Azure Container Registry.

1. Cliquez sur **Enregistrer et exécuter**.

1. Ouvrez l’exécution du pipeline. Si vous voyez le message d’avertissement « Ce pipeline a besoin d’une autorisation pour accéder à une ressource avant que cette exécution puisse continuer à build », sélectionnez **Afficher**, puis **Autoriser** et à nouveau **Autoriser**. Cela permet au pipeline d’accéder à l’abonnement Azure.

    > **Remarque** : le déploiement peut prendre quelques minutes.

1. Votre pipeline est nommé en fonction du nom du projet. **Renommons**-le pour mieux identifier le pipeline. Accédez à **Pipelines > Pipelines**, puis cliquez sur le pipeline qui vient d’être créé. Cliquez sur les points de suspension et sur l’option **Renommer/déplacer**. Nommez-le **eshoponweb-ci-docker**, puis cliquez sur **Enregistrer**.

1. Accédez au [**portail Azure**](https://portal.azure.com), recherchez Azure Container Registry dans le groupe de ressources récemment créé (il doit être nommé **AZ400-RG1**). Sur le côté gauche, cliquez sur **Dépôts** sous **Services** et vérifiez que le dépôt **eshoponweb/web** a été créé. Lorsque vous cliquez sur le lien du référentiel, vous devez voir deux balises (l’une d’elles est **la plus récente**), il s’agit des images envoyées. Si ce n’est pas le cas, vérifiez l’état de votre pipeline.

### Exercice 2 : importer et exécuter le pipeline CD

Dans cet exercice, vous allez configurer la connexion de service avec votre abonnement Azure, puis importer et exécuter le pipeline CD.

#### Tâche 1 : importer et exécuter le pipeline CD

Dans cette tâche, vous allez importer et exécuter le pipeline CD.

1. Accédez à **Pipelines > Pipelines**.
1. Cliquez sur le bouton **Nouveau pipeline**.
1. Sélectionnez **Azure Repos Git (YAML)**.
1. Sélectionnez le référentiel **eShopOnWeb**.
1. Sélectionnez **Fichier YAML Azure Pipelines existant**.
1. Sélectionnez la branche **principale** et le fichier **/.ado/eshoponweb-cd-webapp-docker.ym**, puis cliquez sur **Continuer**
1. Dans la définition de pipeline YAML, personnalisez :
   - **YOUR-SUBSCRIPTION-ID** avec votre ID d’abonnement Azure.
   - Remplacez le **resourceGroup** par le nom du groupe de ressources utilisé pendant la création de la connexion de service, par exemple **AZ400-RG1**.
   - Remplacez **emplacement** par la région Azure dans laquelle les ressources sont déployées.

1. Examinez la définition du pipeline. La définition CD se compose des tâches suivantes :
    - **Ressources** : télécharge les fichiers du référentiel qui seront utilisés dans les tâches suivantes.
    - **AzureResourceManagerTemplateDeployment** : déploie Azure App Service à l’aide du modèle bicep.
    - **AzureResourceManagerTemplateDeployment** : ajouter des attributions de rôle à l’aide de Bicep.

1. Cliquez sur **Enregistrer et exécuter**.

1. Ouvrez l’exécution du pipeline. Si vous voyez le message d’avertissement « Ce pipeline a besoin d’une autorisation pour accéder à une ressource avant que cette exécution puisse continuer à déployer », sélectionnez **Afficher**, puis **Autoriser** et à nouveau **Autoriser**. Cela permet au pipeline d’accéder à l’abonnement Azure.

    > **Remarque** : le déploiement peut prendre quelques minutes.

    > [!IMPORTANT]
    > Si vous recevez le message d’erreur « TF402455 : les envois push vers cette branche ne sont pas autorisés, vous devez utiliser une demande de tirage (pull request) pour mettre à jour cette branche », vous devez décocher la règle de protection de branche « Exiger un nombre minimal de réviseurs » activée dans les labos précédents.

1. Votre pipeline est nommé en fonction du nom du projet. **Renommons**-le pour mieux identifier le pipeline. Accédez à **Pipelines > Pipelines** et pointez sur le pipeline récemment créé. Cliquez sur les points de suspension et sur l’option **Renommer/déplacer**. Nommez-le **eshoponweb-cd-webapp-docker**, puis cliquez sur **Enregistrer**.

    > **Remarque 1** : L’utilisation du modèle **/infra/webapp-docker.bicep** crée un plan App Service, une application web avec l’identité managée affectée par le système activée, et fait référence à l’image Docker envoyée précédemment : **${acr.properties.loginServer}/eshoponweb/web:latest**.

    > **Remarque 2** : L’utilisation du modèle **/infra/webapp-to-acr-roleassignment.bicep** crée une attribution de rôle pour l’application web avec le rôle AcrPull afin de pouvoir récupérer l’image Docker. Cela peut être effectué dans le premier modèle, mais étant donné que la propagation de l’attribution de rôle peut prendre un certain temps, il est judicieux d’effectuer les deux tâches séparément.

#### Tâche 2 : tester la solution

1. Dans le portail Azure, accédez au groupe de ressources récemment créé. Vous devez maintenant voir trois ressources (App Service, Plan App Service et Container Registry).

1. Accédez à App Service, puis cliquez sur **Parcourir**. Vous accédez au site web.

> [!IMPORTANT]
> N’oubliez pas de supprimer les ressources créées dans le portail Azure pour éviter les modifications inutiles.

## Révision

Dans ce labo, vous avez découvert comment utiliser un pipeline CI/CD d’Azure DevOps pour générer une image Docker personnalisée, l’envoyer vers Azure Container Registry et la déployer en tant que conteneur sur Azure App Service.
