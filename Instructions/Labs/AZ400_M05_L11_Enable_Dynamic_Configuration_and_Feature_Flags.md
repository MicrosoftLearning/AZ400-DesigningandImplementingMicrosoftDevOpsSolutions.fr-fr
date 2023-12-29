---
lab:
  title: Activer les indicateurs de fonctionnalité de configuration dynamiques
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
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

À la fin de ce labo, vous serez en mesure d’accomplir les tâches suivantes :

- Activer la configuration dynamique.
- Gérer les indicateurs de fonctionnalité.

## Durée estimée : 60 minutes

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

### Exercice 1 : (à ignorer si vous l’avez déjà effectué) importer et exécuter des pipelines CI/CD

Dans cet exercice, vous allez importer et exécuter le pipeline CI, configurer la connexion de service avec votre abonnement Azure, puis importer et exécuter le pipeline CD.

#### Tâche 1 : importer et exécuter le pipeline CI

Commençons par importer le pipeline CI nommé [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Accédez à **Pipelines > Pipelines**.
2. Cliquez sur le bouton **Créer un pipeline** (s’il n’existe pas de pipelines) ou sur le bouton **Nouveau pipeline** (si des pipelines ont déjà été créés).
3. Sélectionnez **Azure Repos Git (Yaml)**.
4. Sélectionnez le référentiel **eShopOnWeb**.
5. Sélectionnez **Fichier YAML Azure Pipelines existant**.
6. Sélectionnez le fichier **/.ado/eshoponweb-ci.yml**, puis cliquez sur **Continuer**.
7. Cliquez sur le bouton **Run** (Exécuter) pour exécuter le pipeline.
8. Votre pipeline choisira un nom en fonction du nom du projet. **Renommons**-le pour mieux identifier le pipeline. Accédez à **Pipelines > Pipelines** et sélectionnez le pipeline que vous venez de créer. Cliquez sur les trois points, puis sélectionnez l’option **Renommer/Déplacer**. Nommez-le **eshoponweb-ci**, puis cliquez sur **Enregistrer**.

#### Tâche 2 : Gérer la connexion de service

Vous pouvez créer une connexion à partir d’Azure Pipelines vers des services externes et distants pour les tâches s’exécutant dans un travail.

Dans cette tâche, vous allez créer un principal de service à l’aide d’Azure CLI, ce qui permettra à Azure DevOps d’effectuer les actions suivantes :

- Déployer des ressources sur votre abonnement Azure
- Déployer l’application eShopOnWeb

> **Remarque** : vous pouvez passer directement à la tâche suivante si vous possédez déjà un principal de service.

Vous aurez besoin d’un principal de service pour déployer des ressources Azure à partir d’Azure Pipelines.

Un principal de service est créé automatiquement par Azure Pipelines lorsque vous vous connectez à un abonnement Azure à partir d’une définition de pipeline ou lorsque vous créez une connexion de service à partir de la page des paramètres du projet (option automatique). Vous pouvez également créer manuellement le principal de service à partir du portail ou à l’aide d’Azure CLI et le réutiliser dans les projets.

1. Sur l’ordinateur de labo, démarrez un navigateur web, accédez au [**Portail Azure**](https://portal.azure.com) et connectez-vous avec le compte d’utilisateur qui a le rôle de Propriétaire dans l’abonnement Azure que vous utiliserez dans ce labo et qui a le rôle d’Administrateur général dans le locataire Microsoft Entra associé à cet abonnement.
2. Dans le Portail Azure, cliquez sur l’icône **Cloud Shell** située directement à droite de la zone de texte de recherche en haut de la page.
3. Si vous êtes invité à sélectionner **Bash** ou **PowerShell**, sélectionnez **Bash**.

   >**Remarque** : si c’est la première fois que vous démarrez **Cloud Shell** et que vous voyez le message **Vous n’avez aucun stockage monté**, sélectionnez l’abonnement que vous utilisez dans ce labo, puis sélectionnez **Créer un stockage**.

4. À partir de l’invite **Bash**, dans le volet **Cloud Shell**, exécutez les commandes suivantes pour récupérer les valeurs de l’attribut d’ID d’abonnement Azure :

    ```sh
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **Remarque** : copiez les deux valeurs dans un fichier texte. Vous en aurez besoin plus tard dans ce labo.

5. À partir de l’invite **Bash** dans le volet **Cloud Shell**, exécutez la commande suivante pour créer un principal de service :

    ```sh
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **Remarque** : la commande générera une sortie JSON. Copiez la sortie dans un fichier texte. Vous en aurez besoin plus tard dans ce labo.

6. Sur l’ordinateur du labo, démarrez ensuite un navigateur web et accédez au projet Azure DevOps **eShopOnWeb**. Cliquez sur **Paramètres du projet > Connexions de services (sous Pipelines)**, puis sélectionnez **Nouvelle connexion de services**.

7. Dans le volet **Nouvelle connexion de service**, sélectionnez **Azure Resource Manager**, puis **Suivant** (il peut être nécessaire de faire défiler l’écran vers le bas).

8. Ensuite, sélectionnez **Principal de service (manuel)** puis sélectionnez **Suivant**.

9. Remplissez les champs vides à l’aide des informations collectées lors des étapes précédentes :
    - ID et nom de l’abonnement
    - ID du principal de service (ou clientId), clé (ou mot de passe) et TenantId.
    - Dans **Nom de la connexion de service** tapez **azure subs**. Ce nom est référencé dans les pipelines YAML lorsque vous avez besoin d’une connexion de service Azure DevOps pour communiquer avec votre abonnement Azure.

10. Cliquez sur **Vérifier et enregistrer**.

#### Tâche 3 : importer et exécuter le pipeline CD

Vous allez à présent importer le pipeline CD nommé [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. Accédez à **Pipelines > Pipelines**.
2. Cliquez sur le bouton **Nouveau pipeline**.
3. Sélectionnez **Azure Repos Git (Yaml)**.
4. Sélectionnez le référentiel **eShopOnWeb**.
5. Sélectionnez **Fichier YAML Azure Pipelines existant**.
6. Sélectionnez le fichier **/.ado/eshoponweb-cd-webapp-code.yml**, puis cliquez sur **Continuer**.
7. Dans la définition du pipeline YAML, personnalisez les éléments suivants :
   - **YOUR-SUBSCRIPTION-ID** par votre ID d’abonnement Azure ;
   - **az400eshop-NAME** et remplacer NAME pour le rendre globalement unique.
   - **AZ400-EWebShop-NAME** avec le nom du groupe de ressources défini plus tôt dans le labo.

8. Cliquez sur **Enregistrer et exécuter**, puis attendez que le pipeline s’exécute correctement.

    > **Remarque** : le déploiement peut prendre quelques minutes.

    La définition CD se compose des tâches suivantes :
    - **Ressources** : il est prêt à se déclencher automatiquement en fonction de l’achèvement du pipeline CI. Il télécharge également le référentiel pour le fichier bicep.
    - **AzureResourceManagerTemplateDeployment** : déploie l’application web Azure à l’aide du modèle bicep.

9. Votre pipeline choisira un nom en fonction du nom du projet. **Renommons**-le pour mieux identifier le pipeline. Accédez à **Pipelines > Pipelines** et sélectionnez le pipeline que vous venez de créer. Cliquez sur les trois points, puis sélectionnez l’option **Renommer/Déplacer**. Nommez-le **eshoponweb-cd-webapp-code**, puis cliquez sur **Enregistrer**.

### Exercice 2 : gérer Azure App Configuration

Dans le cadre de cet exercice, vous créerez la ressource App Configuration dans Azure, activerez l’identité managée, puis testerez la solution complète.

> **Remarque** : cet exercice ne nécessite aucune compétence en matière de codage. Le code du site web met en œuvre les fonctionnalités d’Azure App Configuration existantes.

Si vous souhaitez savoir comment les mettre en œuvre dans votre application, consultez ces tutoriels : [Utiliser la configuration dynamique dans une application ASP.NET Core](https://learn.microsoft.com/azure/azure-app-configuration/enable-dynamic-configuration-aspnet-core) et [Gérer les indicateurs de fonctionnalités dans Azure App Configuration](https://learn.microsoft.com/azure/azure-app-configuration/manage-feature-flags).

#### Tâche 1 : créer la ressource App Configuration

1. Dans le Portail Azure, recherchez le service **App Configuration**.
2. Cliquez sur **Créer une configuration d’application**, puis sélectionnez :
    - Votre abonnement Azure.
    - Le groupe de ressources que vous venez de créer (il est normalement intitulé **AZ400-EWebShop-NAME**).
    - Emplacement.
    - Un nom unique, tel que **appcs-NAME-REGION**.
    - Sélectionnez le niveau tarifaire **Gratuit**.
3. Cliquez sur **Vérifier + créer**, puis sur **Créer**.
4. Une fois le service App Configuration créé, accédez à la **vue d’ensemble** et copiez/enregistrez la valeur du **point de terminaison**.

#### Tâche 2 : activer l’identité managée

1. Accédez à l’application web déployée à l’aide du pipeline (elle doit porter le nom **az400-webapp-NAME**).
2. Dans la section **Paramètres**, cliquez sur **Identité**, puis passez l’état sur **Activé** dans la section **Attribuée par le système**. Cliquez ensuite sur **Enregistrer > Oui** et patientez quelques secondes jusqu’à la fin de l’opération.
3. Revenez au service App Configuration. Cliquez sur **Contrôle d’accès**, puis sur **Ajouter une attribution de rôle**.
4. Dans la section **Rôle**, sélectionnez **Lecteur de données d’App Configuration**.
5. Dans la section **Membres**, cochez la case **Gérer l’identité**, puis sélectionnez l’identité managée de votre application web (elles doivent porter le même nom).
6. Cliquez sur **Vérifier et attribuer**.

#### Tâche 3 : configurer l’application web

Pour vous assurer que votre site web a accès à App Configuration, vous devez mettre à jour sa configuration.

1. Revenez à votre application web.
2. Dans la section **Paramètres**, cliquez sur **Configuration**.
3. Ajoutez deux nouveaux paramètres d’application :
    - Premier paramètre d’application
        - **Nom :** UseAppConfig
        - **Valeur : true**
    - Deuxième paramètre d’application
        - **Nom :** AppConfigEndpoint
        - **Valeur :** *la valeur que vous venez d’enregistrer/de copier à partir du point de terminaison App Configuration, qui ressemble à https://appcs-NAME-REGION.azconfig.io*.

4. Cliquez sur **Ok**, puis sur **Enregistrer** et attendez que les paramètres soient mis à jour.
5. Accédez à la **Vue d’ensemble** et cliquez sur **Parcourir**.
6. À cette étape, vous ne verrez aucune modification sur le site web, car App Configuration ne contient aucune donnée. Vous renseignerez ces données au cours des tâches suivantes.

#### Tâche 4 : tester la gestion de la configuration

1. Sur votre site web, sélectionnez **Visual Studio** dans la liste déroulante **Marque**, puis cliquez sur le bouton en forme de flèche (**>**).
2. Un message indiquant *« AUCUN RÉSULTAT NE CORRESPOND À VOTRE RECHERCHE »* apparaît. L’objectif de ce labo est de pouvoir mettre à jour cette valeur sans mettre à jour le code du site web ou le redéployer.
3. Pour ce faire, revenez à App Configuration.
4. Dans la section **Opérations**, sélectionnez **Explorateur de configuration**.
5. Cliquez sur **Créer > Clé-valeur**, puis ajoutez les données suivantes :
    - **Clé :** eShopWeb:Settings:NoResultsMessage
    - **Valeur :** *saisissez un message personnalisé*.
6. Cliquez sur **Appliquer**, puis revenez à votre site web et actualisez la page.
7. Votre nouveau message remplace l’ancienne valeur par défaut.

Félicitations ! Dans le cadre de cette tâche, vous avez testé l’**explorateur de configuration** dans Azure App Configuration.

#### Tâche 5 : tester l’indicateur de fonctionnalité

Poursuivez le test du gestionnaire de fonctionnalités.

1. Pour ce faire, revenez à App Configuration.
2. Dans la section **Opérations**, sélectionnez **Gestionnaire de fonctionnalités**.
3. Cliquez sur **Créer**, puis ajoutez les données suivantes :
    - **Activer l’indicateur de fonctionnalité :** Vérifié
    - **Nom de l’indicateur de fonctionnalité :** SalesWeekend
4. Cliquez sur **Appliquer**, puis revenez à votre site web et actualisez la page.
5. Une image comportant le texte « PROMOTION SUR TOUS LES T-SHIRTS CE WEEK-END ».
6. Vous pouvez désactiver cette fonctionnalité dans App Configuration : l’image disparaîtra.

Félicitations ! Dans le cadre de cette tâche, vous avez testé le **gestionnaire de fonctionnalités** dans Azure App Configuration.

### Exercice 3 : Supprimer les ressources du labo Azure.

Dans le cadre de cet exercice, vous supprimerez les ressources Azure provisionnées dans ce labo pour éliminer les frais inattendus.

>**Remarque** : N’oubliez pas de supprimer toutes les nouvelles ressources Azure que vous n’utilisez plus. La suppression des ressources inutilisées vous évitera d’encourir des frais inattendus.

#### Tâche 1 : supprimer les ressources du labo Azure

Dans le cadre de cette tâche, vous utiliserez Azure Cloud Shell pour supprimer les ressources Azure provisionnées dans ce labo afin d’éliminer les frais superflus.

1. Dans le Portail Azure, ouvrez la session shell **Bash** dans le volet **Cloud Shell**.
2. Listez tous les groupes de ressources créés dans les labos de ce module en exécutant la commande suivante :

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].name" --output tsv
    ```

3. Supprimez tous les groupes de ressources que vous avez créés dans les labos de ce module en exécutant la commande suivante :

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Remarque** : La commande s’exécute de façon asynchrone (comme déterminé par le paramètre --no-wait). Par conséquent, vous serez en mesure d’exécuter une autre commande Azure CLI immédiatement après au cours de la même session Bash, mais la suppression réelle du groupe de ressources prendra quelques minutes.

## Révision

Dans ce labo, vous avez appris à activer la configuration de façon dynamique et à gérer les indicateurs de caractéristique.
