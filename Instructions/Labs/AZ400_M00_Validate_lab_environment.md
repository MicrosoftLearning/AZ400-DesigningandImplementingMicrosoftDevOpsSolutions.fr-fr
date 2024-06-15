---
lab:
  title: Valider l’environnement de labo
  module: 'Module 0: Welcome'
---

# Valider l’environnement de labo

## Manuel de labo de l’étudiant

## Instructions pour créer une organisation Azure DevOps (vous ne devez effectuer cette opération qu’une seule fois)

### Commencez ici si vous n’avez pas d’abonnement Azure

1. Obtenez un nouveau **code promotionnel de pass Azure** auprès de l’instructeur ou d’une autre source.
1. Utilisez une session de navigateur privé pour obtenir un nouveau **compte Microsoft (MSA) personnel** à l’adresse [https://account.microsoft.com](https://account.microsoft.com).
1. Dans la même session de navigateur, accédez à [https://www.microsoftazurepass.com](https://www.microsoftazurepass.com) pour échanger votre pass Azure via votre compte Microsoft (MSA). Pour plus d’informations, reportez-vous à [Échanger un pass Microsoft Azure](https://www.microsoftazurepass.com/Home/HowTo?Length=5). Suivez les instructions concernant l’échange.

### Commencez ici si vous avez un abonnement Azure

1. Ouvrez un navigateur, accédez à [https://portal.azure.com](https://portal.azure.com), puis recherchez **Azure DevOps** en haut de l'écran Portail Azure. Sur la page résultante, cliquez sur **Organisations Azure DevOps**.
1. Ensuite, cliquez sur le lien intitulé **Mes organisations Azure DevOps** ou accédez directement à [https://aex.dev.azure.com](https://aex.dev.azure.com).
1. Sur la page **Nous avons besoin de quelques détails supplémentaires**, sélectionnez **Continuer**.
1. Dans la zone de liste déroulante située à gauche, choisissez **Répertoire par défaut**, au lieu de « Compte Microsoft ».
1. Si vous y êtes invité (*« Nous avons besoin de quelques détails supplémentaires »*), indiquez vos nom, adresse personnelle et adresse e-mail, puis cliquez sur **Continuer**.
1. Une fois de retour dans [https://aex.dev.azure.com](https://aex.dev.azure.com), si vous avez sélectionné **Répertoire par défaut**, cliquez sur le bouton bleu **Créer une organisation**.
1. Acceptez les *conditions d’utilisation* en cliquant sur **Continuer**.
1. Si vous y êtes invité (*« Presque terminé  »*), conservez le nom par défaut de l‘organisation Azure DevOps (il doit s‘agir d‘un nom global unique) et choisissez un emplacement d‘hébergement proche de vous dans la liste.
1. Une fois que la nouvelle organisation créée s‘ouvre dans **Azure DevOps**, cliquez sur **Paramètres de l’organisation** dans l’angle inférieur gauche.
1. Dans l’écran **Paramètres de l’organisation**, cliquez sur **Facturation** (l’ouverture de cet écran prend quelques secondes).
1. Cliquez sur **Configurer la facturation**. À droite sur l’écran, sélectionnez l’abonnement **Azure Pass - Parrainage**, puis cliquez sur **Enregistrer** pour lier l’abonnement à l’organisation.
1. Une fois que l’écran affiche l’ID d’abonnement Azure lié en haut, modifiez le nombre de **travaux parallèles payants** pour **MS Hosted CI/CD** de 0 à **1**. Cliquez ensuite sur le bouton **ENREGISTRER** en bas.
1. Dans **Paramètres d’organisation**, accédez à la section **Pipelines** et cliquez sur **Paramètres**.
1. Basculez le commutateur sur **Désactivé** pour **Désactiver la création de pipelines de build classiques** et **Désactiver la création de pipelines de mise en production classiques**.

    > Remarque : la définition de **Désactiver la création de pipelines de mise en production classiques** sur **Activé** masque les options de création de pipeline de mise en production classique comme le menu **Mise en production** dans la section **Pipeline** de projets DevOps.

1. Dans **Paramètres d’organisation**, accédez à la section **Sécurité** et cliquez sur **Stratégies**.
1. Basculez le commutateur sur **Activé** pour **Autoriser les projets publics**.

    > Remarque : les extensions utilisées dans certains labos peuvent nécessiter un projet public pour autoriser l’utilisation de la version gratuite.

1. **Attendez au moins 3 heures avant d’utiliser les fonctionnalités CI/CD** afin que les nouveaux paramètres soient reflétés dans le back-end. Sinon, vous verrez toujours le message *« Aucun parallélisme hébergé n’a été acheté ou accordé »*.

## Instructions pour créer l’exemple de projet Azure DevOps (vous ne devez effectuer cette opération qu’une seule fois)

### Exercice 0 : configurer les prérequis du labo

> **Remarque** : assurez-vous d’avoir effectué les étapes pour créer votre organisation Azure DevOps avant de poursuivre ces étapes.

Dans cet exercice, vous allez configurer les prérequis pour le labo, qui se composent d’un nouveau projet Azure DevOps avec un référentiel basé sur [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tâche 1 : créer et configurer le projet d’équipe

Dans cette tâche, vous allez créer un projet Azure DevOps **eShopOnWeb** à utiliser par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps. Cliquez sur **Nouveau projet**. Attribuez à votre projet les paramètres suivants :
    - Nom : **eShopOnWeb**
    - Visibilité : **Privé**
    - Avancé : Contrôle de version : **Git**
    - Avancé : Processus d’élément de travail : **Scrum**

1. Cliquez sur **Créer**.

    ![Création d’un projet](images/create-project.png)

#### Tâche 2 : importer un référentiel Git eShopOnWeb

Dans cette tâche, vous allez importer le référentiel Git eShopOnWeb qui sera utilisé par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps et le projet **eShopOnWeb** créé précédemment. Cliquez sur **Dépôts > Fichiers**, **Importer un dépôt**. Cliquez sur **Importer**. Dans la fenêtre **Importer un dépôt Git**, collez l’URL <https://github.com/MicrosoftLearning/eShopOnWeb.git>, puis cliquez sur **Importer** :

    ![Importer un référentiel](images/import-repo.png)

1. Le référentiel est organisé de la manière suivante :
    - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
    - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
    - Le dossier **infra** contient l’infrastructure Bicep&ARM sous forme de modèles de code utilisés dans certains scénarios de labo.
    - **Définitions de workflow GitHub YAML du conteneur de dossiers .github**.
    - Le dossier **src** contient le site web .NET 8 utilisé dans les scénarios de labo.

Vous avez terminé les prérequis pour continuer avec chacun des labos du cours AZ-400.
