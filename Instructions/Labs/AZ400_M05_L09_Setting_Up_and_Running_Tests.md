---
lab:
  title: Configuration et exécution de tests
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# Configuration et exécution de tests

## Manuel de labo de l’étudiant

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Créer une organisation ou une collection de projets](https://learn.microsoft.com/dotnet/architecture/modern-web-apps-azure/test-asp-net-core-mvc-apps).

## Présentation du labo

Tous les logiciels, quel que soit leur niveau de complexité, peuvent un jour échouer de façon inattendue en réponse à des modifications. C’est pourquoi vous devez obligatoirement tester toutes vos applications, y compris les plus triviales ou les moins stratégiques, après y avoir apporté des modifications. Tester un logiciel manuellement est la méthode de test la plus lente, la moins fiable et la plus coûteuse.

Il existe de nombreuses sortes de tests automatisés pour les applications logicielles. Le test de base, le plus simple, est le test unitaire. À un niveau juste au-dessus, on trouve les tests d’intégration et les tests fonctionnels. Les autres types de tests, par exemple les tests d’IU, de charge, de contrainte et de détection de fumée, sortent du cadre de ce labo.

*Si vous souhaitez en savoir plus sur les différents types de tests, nous vous recommandons de lire cet article : [Tester des applications ASP.NET Core MVC](https://learn.microsoft.com/dotnet/architecture/modern-web-apps-azure/test-asp-net-core-mvc-apps).*

## Objectifs

Une fois ce labo terminé, vous pourrez configurer un pipeline CI pour une application .Net incluant :

- Tests unitaires
- Tests d’intégration
- Tests fonctionnels

## Durée estimée : 60 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

Dans cet exercice, vous allez configurer les prérequis pour le labo, qui se composent d’un nouveau projet Azure DevOps avec un référentiel basé sur [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tâche 1 : (passer si terminée) créer et configurer le projet d’équipe

Dans cette tâche, vous allez créer un projet Azure DevOps **eShopOnWeb** à utiliser par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps. Cliquez sur **Nouveau projet**. Attribuez au projet le nom **eShopOnWeb** et conservez les valeurs par défaut des autres champs. Cliquez sur **Créer**.

#### Tâche 2 : (ignorer si terminé) importer le référentiel Git eShopOnWeb

Dans cette tâche, vous allez importer le référentiel Git eShopOnWeb qui sera utilisé par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps et le projet **eShopOnWeb** créé précédemment. Cliquez sur **Dépôts > Fichiers**, **Importer un dépôt**. Cliquez sur **Importer**. Dans la fenêtre **Importer un référentiel Git**, collez l’URL https://github.com/MicrosoftLearning/eShopOnWeb.git, puis cliquez sur **Importer** :

1. Le référentiel est organisé de la manière suivante :
    - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
    - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
    - Le dossier **.azure** contient l’infrastructure Bicep & ARM en tant que modèles de code utilisés dans certains scénarios de labo.
    - Le dossier **.github** contient les définitions de workflow YAML GitHub.
    - Le dossier **src** contient le site web .NET utilisé dans les scénarios de labo.

### Exercice 1 : configurer des tests dans le pipeline CI

Dans cet exercice, vous allez configurer des tests dans le pipeline CI.

#### Tâche 1 : (ignorer si terminé) importer la définition de build YAML pour CI

Dans cette tâche, vous allez ajouter la définition de build YAML qui sera utilisée pour implémenter l’intégration continue.

Commençons par importer le pipeline CI nommé [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Accédez à **Pipelines > Pipelines**.
1. Cliquez sur le bouton **Nouveau pipeline**.
1. Sélectionnez **Azure Repos Git** (YAML).
1. Sélectionnez le référentiel **eShopOnWeb**.
1. Sélectionnez **Fichier YAML Azure Pipelines existant**.
1. Sélectionnez le fichier **/.ado/eshoponweb-ci.yml**, puis cliquez sur **Continuer**.

    La définition CI se compose des tâches suivantes :
    - **DotNet Restore** : avec la restauration des packages NuGet, vous pouvez installer toutes les dépendances de votre projet sans avoir à les stocker dans le contrôle de code source.
    - **DotNet Build** : permet de générer un projet et l’ensemble de ses dépendances.
    - **DotNet Test** : pilote de test .NET utilisé pour exécuter des tests unitaires.
    - **DotNet Publish** : publie l’application et ses dépendances dans un dossier en vue d’un déploiement sur un système d’hébergement. Dans ce cas, il s’agit de **Build.ArtifactStagingDirectory**.
    - **Publier l’artefact - Site web** : publiez l’artefact de l’application (créé à l’étape précédente) et rendez-le disponible en tant qu’artefact de pipeline.
    - **Publier l’artefact - Bicep** : publiez l’artefact d’infrastructure (fichier Bicep) et rendez-le disponible en tant qu’artefact de pipeline.
1. Cliquez sur le bouton  **Enregistrer** (et non **Enregistrer et exécuter**) pour enregistrer la définition du pipeline.

#### Tâche 2 : ajouter des tests au pipeline CI

Dans cette tâche, vous allez ajouter les tests d’intégration et fonctionnels au pipeline CI.

Vous pouvez remarquer que la tâche Tests unitaires fait déjà partie du pipeline.

- Les **tests unitaires** testent une seule partie de la logique de votre application. Pour mieux le décrire, on peut lister ce qu’il fait et ce qu’il ne fait pas. Un test unitaire ne teste pas la manière dont votre code fonctionne avec les dépendances ou l’infrastructure (cet aspect est vérifié par les tests d’intégration).

1. Vous devez maintenant ajouter la tâche Tests d’intégration après la tâche Tests unitaires :

    ```YAML
    - task: DotNetCoreCLI@2
      displayName: Integration Tests
      inputs:
        command: 'test'
        projects: 'tests/IntegrationTests/*.csproj'
    ```

    > Les **tests d’intégration** testent le fonctionnement de votre code avec les dépendances ou l’infrastructure. Même si vous avez la bonne idée d’encapsuler votre code qui interagit avec une infrastructure comme les bases de données et les systèmes de fichiers, il reste toujours une partie de ce code à tester. En outre, vous devez vérifier que les différentes couches de votre code interagissent comme prévu quand les dépendances de votre application sont entièrement résolues. Cette fonctionnalité relève des tests d’intégration.

1. Vous devez ensuite ajouter la tâche Tests fonctionnels après la tâche Tests d’intégration :

    ```YAML
    - task: DotNetCoreCLI@2
      displayName: Functional Tests
      inputs:
        command: 'test'
        projects: 'tests/FunctionalTests/*.csproj'
    ```

    > Les **tests fonctionnels** sont écrits du point de vue de l’utilisateur, et vérifient que le système répond aux exigences, contrairement aux tests d’intégration qui sont écrits du point de vue du développeur, pour vérifier que certains composants du système fonctionnent correctement ensemble.

16. Cliquez sur **Enregistrer**. Dans le volet **Enregistrer**, cliquez à nouveau sur **Enregistrer** pour valider les modifications directement dans la branche principale.

#### Tâche 4 : vérifier le résumé des tests

1. Cliquez sur **Exécuter** puis, dans l’onglet **Exécuter le pipeline**, cliquez à nouveau sur **Exécuter**.

1. Attendez que le pipeline démarre et jusqu’à ce qu’il termine correctement l’étape de build.

1. Une fois terminé, l’onglet **Test** s’affiche dans le cadre de l’exécution du pipeline. Cliquez dessus pour vérifier le résumé. Il se présente comme indiqué ci-dessous :

    ![Résumé des tests](images/AZ400_M05_L09_Tests_Summary.png)

1. Pour plus d’informations, au bas de la page, le tableau affiche une liste des différents tests d’exécution.

    >**Remarque** : si le tableau est vide, vous devez réinitialiser les filtres pour avoir tous les détails sur l’exécution des tests.

    ![Tableau des tests](images/AZ400_M05_L09_Tests_Table.png)

## Révision

Dans ce labo, vous avez appris à configurer et à exécuter différents types de tests à l’aide d’Azure Pipelines et de .Net.
