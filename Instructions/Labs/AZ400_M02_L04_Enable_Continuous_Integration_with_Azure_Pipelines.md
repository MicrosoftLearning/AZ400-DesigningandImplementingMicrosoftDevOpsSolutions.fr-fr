---
lab:
  title: Activer l’intégration continue avec Azure Pipelines
  module: 'Module 02: Implement CI with Azure Pipelines and GitHub Actions'
---

# Activer l’intégration continue avec Azure Pipelines

## Manuel de labo de l’étudiant

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Créer une organisation ou une collection de projets](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

## Présentation du labo

Dans ce labo, vous allez découvrir comment définir des pipelines de build dans Azure DevOps en utilisant YAML.
Les pipelines seront utilisés dans deux scénarios :

- Dans le cadre du processus de validation des demandes de tirage.
- Dans le cadre de l’implémentation de l’intégration continue.

## Objectifs

À la fin de ce labo, vous serez en mesure d’effectuer les tâches suivantes :

- Inclure la validation de la build dans le cadre d’une demande de tirage.
- Configurer le pipeline CI en tant que code avec YAML.

## Durée estimée : 45 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

Dans cet exercice, vous allez configurer les prérequis pour le labo, qui se composent d’un nouveau projet Azure DevOps avec un référentiel basé sur [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tâche 1 : (passer si terminée) créer et configurer le projet d’équipe

Dans cette tâche, vous allez créer un projet Azure DevOps **eShopOnWeb** à utiliser par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps. Cliquez sur **Nouveau projet**. Attribuez au projet le nom **eShopOnWeb** et conservez les valeurs par défaut des autres champs. Cliquez sur **Créer**.

#### Tâche 2 : (passer si terminée) importer le référentiel Git eShopOnWeb

Dans cette tâche, vous allez importer le référentiel Git eShopOnWeb qui sera utilisé par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps et le projet **eShopOnWeb** créé précédemment. Cliquez sur **Dépôts > Fichiers**, **Importer un dépôt**. Cliquez sur **Importer**. Dans la fenêtre **Importer un référentiel Git**, collez l’URL https://github.com/MicrosoftLearning/eShopOnWeb.git, puis cliquez sur **Importer** :

1. Le référentiel est organisé de la manière suivante :
    - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
    - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
    - Le dossier **infra** contient l’infrastructure Bicep & ARM en tant que modèles de code utilisés dans certains scénarios de labo.
    - Le dossier **.github** contient les définitions de workflow YAML GitHub.
    - Le dossier **src** contient le site web .NET utilisé dans les scénarios de labo.

#### Tâche 3 : (à ignorer si vous l’avez déjà effectuée) définir la branche principale en tant que branche par défaut

1. Accédez à **Dépôts > Branches**.
1. Pointez sur la branche **principale**, puis cliquez sur les points de suspension à droite de la colonne.
1. Cliquez sur **Définir comme branche par défaut**.

### Exercice 1 : inclure la validation du build dans le cadre d’une demande de tirage

Dans cet exercice, vous allez inclure la validation de build pour valider une demande de tirage.

#### Tâche 1 : importer la définition de build YAML

Dans cette tâche, vous allez importer la définition de build YAML qui sera utilisée comme stratégie de branche pour valider les demandes de tirage.

Commençons par importer le pipeline de build nommé [eshoponweb-ci-pr.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci-pr.yml).

1. Accédez à **Pipelines > Pipelines**.
1. Cliquez sur le bouton **Créer un pipeline** ou **Nouveau pipeline**.
1. Sélectionnez **Azure Repos Git (YAML)**.
1. Sélectionnez le référentiel **eShopOnWeb**.
1. Sélectionnez **Fichier YAML Azure Pipelines existant**.
1. Sélectionnez la branche **principale** et le fichier **/.ado/eshoponweb-ci-pr.yml**, puis cliquez sur **Continuer**

    La définition de build est composée des tâches suivantes :
    - **DotNet Restore** : avec la restauration des packages NuGet, vous pouvez installer toutes les dépendances de votre projet sans avoir à les stocker dans le contrôle de code source.
    - **DotNet Build** : permet de générer un projet et l’ensemble de ses dépendances.
    - **DotNet Test** : pilote de test .NET utilisé pour exécuter des tests unitaires.
    - **DotNet Publish** : publie l’application et ses dépendances dans un dossier en vue d’un déploiement sur un système d’hébergement. Dans ce cas, il s’agit de **Build.ArtifactStagingDirectory**.

1. Cliquez sur le bouton **Enregistrer** pour enregistrer la définition de pipeline.
1. Votre pipeline est nommé en fonction du nom du projet. **Renommons**-le pour mieux l’identifier. Accédez à **Pipelines > Pipelines**, puis cliquez sur le pipeline qui vient d’être créé. Cliquez sur les points de suspension et sur l’option **Renommer/Déplacer**. Nommez-le **eshoponweb-ci-pr**, puis cliquez sur **Enregistrer**.

#### Tâche 2 : ajouter des stratégies de branche

Dans cette tâche, vous allez ajouter des stratégies à la branche principale et autoriser uniquement les modifications à l’aide de demandes de tirage (pull) qui sont conformes aux stratégies définies. Vous souhaitez vous assurer que les modifications apportées à une branche sont vérifiées avant d’être fusionnées.

1. Accédez à la section **Repos > Branches**.
1. Sous l’onglet **Exploiter** du volet **Branches**, pointez sur l’entrée de branche **principale** pour afficher le symbole de points de suspension sur le côté droit.
1. Cliquez sur les points de suspension, puis, dans le menu contextuel, sélectionnez **Stratégies de branche**.
1. Sous l’onglet **principal** des paramètres du référentiel, activez l’option **Exiger un nombre minimal de réviseurs**. Ajoutez **1** réviseur, puis activez la case **Autoriser les demandeurs à approuver leurs propres changements** (car vous êtes le seul utilisateur de votre projet pour le labo).
1. Sous l’onglet **principal** des paramètres du référentiel, dans la section **Validation de build**, cliquez sur **+** (Ajouter une nouvelle stratégie de build), dans la liste **Pipeline de build**, sélectionnez **eshoponweb-ci-pr**, puis cliquez sur **Enregistrer**.

#### Tâche 3 : utilisation des demandes de tirage (pull)

Dans cette tâche, vous allez utiliser le portail Azure DevOps pour créer une demande de tirage (pull) en utilisant une nouvelle branche pour fusionner une modification dans la branche **principale** protégée.

1. Accédez à la section **Dépôts** dans le menu de navigation eShopOnWeb, puis cliquez sur **Branches**.
1. Créez une branche nommée **Feature01** basée sur la branche **principale**.
1. Cliquez sur **Feature01**, puis accédez au fichier **/eShopOnWeb/src/Web/Program.cs** dans le cadre de la branche **Feature01**.
1. Cliquez sur le bouton **Modifier** dans le coin supérieur droit.
1. Apportez la modification suivante sur la première ligne :

    ```csharp
    // Testing my PR
    ```

1. Cliquez sur **Valider > Valider** (laissez le message de validation par défaut).
1. Un message s’affiche pour vous proposer de créer une demande de tirage (car votre branche **Feature01** contient désormais des modifications supplémentaires par rapport à la branche **principale**). Cliquez sur **Créer une demande de tirage**.
1. Sous l’onglet **Nouvelle demande de tirage (pull request)**, conservez les valeurs par défaut, puis cliquez sur **Créer**.
1. La demande de tirage affiche certaines exigences en attente en fonction des stratégies appliquées à la branche **principale** cible.
    - Au moins 1 utilisateur doit examiner et approuver les modifications.
    - Validation de build : vous pouvez constater que le build **eshoponweb-ci-pr** a été déclenché automatiquement.

1. Une fois toutes les validations réussies, dans le coin supérieur droit, cliquez sur **Approuver**. À présent, dans la liste déroulante **Définir la saisie semi-automatique**, vous pouvez cliquer sur **Terminer**.
1. Sous l’onglet **Achever la demande de tirage**, cliquez sur **Terminer la fusion**.

### Exercice 2 : configurer le pipeline CI en tant que code avec YAML

Dans cet exercice, vous allez configurer le pipeline CI en tant que code avec YAML.

#### Tâche 1 : importer la définition de build YAML pour CI

Dans cette tâche, vous allez ajouter la définition de build YAML qui sera utilisée pour implémenter l’intégration continue.

Commençons par importer le pipeline CI nommé [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Accédez à **Pipelines > Pipelines**.
1. Cliquez sur le bouton **Nouveau pipeline**.
1. Sélectionnez **Azure Repos Git** (YAML).
1. Sélectionnez le référentiel **eShopOnWeb**.
1. Sélectionnez **Fichier YAML Azure Pipelines existant**.
1. Sélectionnez la branche **principale** et le fichier **/.ado/eshoponweb-ci.yml**, puis cliquez sur **Continuer**.

    La définition CI se compose des tâches suivantes :
    - **DotNet Restore** : avec la restauration des packages NuGet, vous pouvez installer toutes les dépendances de votre projet sans avoir à les stocker dans le contrôle de code source.
    - **DotNet Build** : permet de générer un projet et l’ensemble de ses dépendances.
    - **DotNet Test** : pilote de test .NET utilisé pour exécuter des tests unitaires.
    - **DotNet Publish** : publie l’application et ses dépendances dans un dossier en vue d’un déploiement sur un système d’hébergement. Dans ce cas, il s’agit de **Build.ArtifactStagingDirectory**.
    - **Publier l’artefact - Site web** : publiez l’artefact de l’application (créé à l’étape précédente) et rendez-le disponible en tant qu’artefact de pipeline.
    - **Publier l’artefact - Bicep** : publiez l’artefact d’infrastructure (fichier Bicep) et rendez-le disponible en tant qu’artefact de pipeline.

#### Tâche 2 : activer l’intégration continue

La définition de pipeline de build par défaut n’active pas l’intégration continue.

1. Cliquez sur le bouton **Modifier** dans le coin supérieur droit.
1. À présent, vous devez remplacer les lignes **# trigger:** et **# - main** par le code suivant :

    ```YAML
    trigger:
      branches:
        include:
        - main
      paths:
        include:
        - src/web/*
    ```

    Le pipeline de build est déclenché automatiquement si une modification est apportée à la branche principale et au code de l’application web (dossier src/web).

    Étant donné que vous avez activé les stratégies de branche, vous devez passer par une demande de tirage (pull request) pour mettre à jour votre code.

1. Cliquez sur le bouton  **Enregistrer** (et non **Enregistrer et exécuter**) pour enregistrer la définition du pipeline.
1. Sélectionnez **Créer une branche pour cette validation**.
1. Conservez le nom de branche par défaut et laissez l’option **Démarrer une demande de tirage** cochée.
1. Cliquez sur **Save**(Enregistrer).
1. Votre pipeline est nommé en fonction du nom du projet. **Renommons**-le pour mieux l’identifier. Accédez à **Pipelines > Pipelines**, puis cliquez sur le pipeline qui vient d’être créé. Cliquez sur les points de suspension et sur l’option **Renommer/Déplacer**. Nommez-le **eshoponweb-ci**, puis cliquez sur **Enregistrer**.
1. Accédez à **Référentiels > Demandes de tirage**.
1. Cliquez sur la demande de tirage **« Mettre à jour eshoponweb-ci.yml pour Azure Pipelines »**.
1. Une fois toutes les validations réussies, dans le coin supérieur droit, cliquez sur **Approuver**. Vous pouvez maintenant cliquer sur **Terminer**.
1. Dans l’onglet **Terminer la demande de tirage**, cliquez sur **Terminer la fusion**.

#### Tâche 3 : tester le pipeline CI

Dans cette tâche, vous allez créer une demande de tirage (pull request) en utilisant une nouvelle branche pour fusionner une modification dans la branche **principale** protégée et déclencher automatiquement le pipeline CI.

1. Accédez à la section **Référentiels**.
1. Créez une branche nommée **Feature02** basée sur la branche **principale** (main).
1. Cliquez sur la nouvelle branche **Feature02**.
1. Accédez au fichier **/eShopOnWeb/src/Web/Program.cs**, puis cliquez sur **Modifier** dans le coin supérieur droit.
1. Supprimez la première ligne :

    ```csharp
    // Testing my PR
    ```

1. Cliquez sur **Valider > Valider** (laissez le message de validation par défaut).
1. Un message proposant de créer une demande de tirage (pull request) s’affiche (car votre branche **Feature02** est désormais en avance dans les modifications par rapport à la branche **principale**).
1. Cliquez sur **Créer une demande de tirage**.
1. Sous l’onglet **Nouvelle demande de tirage (pull request)**, conservez les valeurs par défaut, puis cliquez sur **Créer**.
1. La demande de tirage affiche certaines exigences en attente en fonction des stratégies appliquées à la branche **principale** cible.
1. Une fois toutes les validations réussies, dans le coin supérieur droit, cliquez sur **Approuver**. À présent, dans la liste déroulante **Définir la saisie semi-automatique**, vous pouvez cliquer sur **Terminer**.
1. Dans l’onglet **Terminer la demande de tirage**, cliquez sur **Terminer la fusion**.
1. Revenez à **Pipelines > Pipelines**. Vous remarquerez que la build **eshoponweb-ci** a été déclenchée automatiquement après la fusion du code.
1. Cliquez sur la build **eshoponweb-ci**, puis sélectionnez la dernière exécution.
1. Une fois l’exécution réussie, cliquez sur **Associé >Publié** pour vérifier les artefacts publiés :
    - Bicep : artefact d’infrastructure.
    - Site web : artefact d’application.

## Révision

Dans ce labo, vous avez activé la validation des demandes de tirage à l’aide d’une définition de build et configuré un pipeline CI en tant que code avec YAML dans Azure DevOps.
