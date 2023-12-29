---
lab:
  title: Valider l’environnement de labo
  module: 'Module 0: Welcome'
---

# Valider l’environnement de labo

## Manuel de labo de l’étudiant

## Instructions pour créer une organisation Azure DevOps (vous ne devez effectuer cette opération qu’une seule fois)

### Commencez ici si vous n’avez pas d’abonnement Azure :
1. Obtenez un nouveau **code promotionnel Pass Azure** auprès de l’instructeur ou d’une autre source.
1. Ouvrez une session de navigateur privée pour obtenir un nouveau **compte Microsoft personnel (MSA)** à l’adresse https://account.microsoft.com[](https://account.microsoft.com) .
1. En conservant la même session de navigateur, accédez à l’adresse [https://www.microsoftazurepass.com](https://www.microsoftazurepass.com) pour récupérer votre Pass Azure à l’aide de votre compte Microsoft (MSA). Consultez [Récupérer un Pass Microsoft Azure](https://www.microsoftazurepass.com/Home/HowTo?Length=5) pour en savoir plus à ce sujet. Suivez les instructions de récupération.

### Commencez ici si vous avez un abonnement Azure :

1. Ouvrez votre navigateur et accédez à l’adresse [https://portal.azure.com](https://portal.azure.com). En haut de la page du Portail Azure, repérez l’intitulé**Azure DevOps**. Dans la page qui s’affiche, cliquez sur **Organisations Azure DevOps**.
1. Ensuite, cliquez sur le lien intitulé **Mes organisations Azure DevOps** ou accédez directement à l’adresse [https://aex.dev.azure.com](https://aex.dev.azure.com).
1. Sur la page **Nous avons besoin de quelques détails supplémentaires**, sélectionnez **Continuer**.
1. Dans la liste déroulante située à gauche de la page, sélectionnez **Répertoire par défaut** au lieu de « Compte Microsoft ».
1. Si vous recevez l’invite (*« Nous avons besoin de quelques détails supplémentaires »*), indiquez votre nom, votre adresse de messagerie et votre emplacement, puis cliquez sur **Continuer**.
1. Revenez à [https://aex.dev.azure.com](https://aex.dev.azure.com). Une fois le **Répertoire par défaut** sélectionné, cliquez sur le bouton de couleur bleue **Créer une organisation**.
1. Acceptez les *conditions d’utilisation du service* en cliquant sur **Continuer**.
1. Si vous recevez l’invite (*« Vous avez presque terminé »)*, laissez le nom de l’organisation Azure DevOps par défaut (il doit s’agir d’un nom global unique) et choisissez un emplacement d’hébergement proche de vous dans la liste.
1. Lorsque la nouvelle organisation s’ouvre dans **Azure DevOps**, cliquez sur **Paramètres de l’organisation** dans le coin inférieur gauche.
1. Sur l’écran des **Paramètres de l’organisation**, cliquez sur **Facturation** (l’ouverture de cet écran prend quelques secondes).
1. Cliquez sur **Configurer la facturation**. À droite de l’écran, sélectionnez l’abonnement **Pass Azure : Sponsorship**, puis cliquez sur **Enregistrer** pour associer l’abonnement à l’organisation.
1. Une fois que l’écran affiche l’ID d’abonnement Azure lié sur la partie supérieure, modifiez le nombre de **travaux parallèles payés** pour **CI/CD hébergé par MS** de 0 à **1**. Cliquez ensuite sur le bouton **ENREGISTRER** en bas de l’écran.
1. Dans les **paramètres de l’organisation**, accédez à la section **Pipelines**, puis cliquez sur **Paramètres**.
1. **Désélectionnez** le bouton bascule des options **Désactiver la création de pipelines de build classiques** et **Désactiver la création de pipelines de mise en production classiques**
    > Remarque : si le bouton bascule de l’option **Désactiver la création de pipelines de mise en production classiques** est **sélectionné**, les options de création de pipelines de mise en production classiques telles que le menu **Mise en production** n’apparaîtront pas dans la section **Pipeline** des projets DevOps.
1. Dans les **paramètres de l’organisation**, accédez à la section **Sécurité**, puis cliquez sur **Stratégies**.
1. **Activez** l’**accès aux applications tierces via Oauth** avec le bouton bascule correspondant.
    > Remarque : le paramètre OAuth permet d’activer les outils comme DemoDevOpsGenerator pour enregistrer des extensions. Sans ce paramètre et en l’absence des extensions requises, plusieurs labos peuvent échouer.
1. **Activez** les **projets publics** avec le bouton bascule correspondant.
    > Remarque : un projet public devra peut-être autoriser l’utilisation de la version gratuite pour les extensions utilisées dans certains labos.
1. **Attendez au moins 3 heures avant d’utiliser les fonctionnalités CI/CD** afin que les nouveaux paramètres soient reflétés dans le back-end. Sinon, vous verrez toujours le message *« Aucun parallélisme hébergé n’a été acheté ou accordé ».*

## Instructions pour créer l’exemple de projet Azure DevOps (vous ne devrez effectuer cette opération qu’une seule fois)

### Exercice 0 : configurer les prérequis du labo

> **Remarque** : vérifiez que vous avez suivi la procédure de création de votre organisation Azure DevOps avant de poursuivre.

Dans cet exercice, vous configurerez les prérequis du labo, qui se composent d’un nouveau projet Azure DevOps comprenant un référentiel basé sur [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tâche 1 : créer et configurer le projet d’équipe

Dans le cadre de cette tâche, vous créerez un projet Azure DevOps **eShopOnWeb** qui sera utilisé dans plusieurs labos.

1. Sur votre ordinateur de labo, ouvrez votre organisation Azure DevOps dans votre navigateur. Cliquez sur **Nouveau projet**. Paramétrez votre projet de la manière suivante :
    - nom : **eShopOnWeb**
    - visibilité : **Privé**
    - avancé : Contrôle de version : **Git**
    - Avancé : Processus d’élément de travail : **Scrum**

2. Cliquez sur **Créer**.

    ![Création d’un projet](images/create-project.png)

#### Tâche 2 : importer un référentiel Git eShopOnWeb

Dans le cadre de cette tâche, vous importerez le référentiel Git eShopOnWeb qui sera utilisé dans plusieurs labos.

1. Sur votre ordinateur de labo, ouvrez votre organisation Azure DevOps dans votre navigateur ainsi que le projet **eShopOnWeb** que vous venez de créer. Cliquez sur **Repos > Fichiers**, puis sur **Importer un référentiel**. Cliquez sur **Importer**. Dans la fenêtre **Importer un référentiel Git**, collez l’URL https://github.com/MicrosoftLearning/eShopOnWeb.git, puis cliquez sur **Importer** :

    ![Importer un référentiel](images/import-repo.png)

2. Le référentiel est organisé de la manière suivante :
    - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
    - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
    - Le dossier **.azure** contient l’infrastructure Bicep&ARM en tant que modèles de code utilisés dans certains scénarios de labos.
    - **Définitions de workflow GitHub YAML du conteneur de dossiers .github**.
    - Le dossier **src** contient le site web .NET 7 utilisé dans les scénarios de labo.

Vous avez terminé les prérequis pour continuer avec chacun des labos du cours AZ-400.
