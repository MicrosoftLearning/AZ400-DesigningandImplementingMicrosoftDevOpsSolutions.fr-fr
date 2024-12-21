---
lab:
  title: Valider l’environnement de labo
  module: 'Module 0: Welcome'
---

# Valider l’environnement de labo

En préparation des labos, il est essentiel que votre environnement soit correctement configuré. Cette page vous guide tout au long du processus de configuration, en vous assurant que toutes les conditions préalables sont remplies.

- Les labos nécessitent **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://learn.microsoft.com/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers).

- **Configurer un abonnement Azure :** si vous n’avez pas encore d’abonnement Azure, créez-en un en suivant les instructions de cette page ou consultez [https://azure.microsoft.com/free](https://azure.microsoft.com/free) pour vous abonner gratuitement.

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour les labos, créez-en une conformément aux instructions sur cette page ou sur [Créer une organisation ou une collection de projets](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization).
- Page de téléchargement de [Git pour Windows](https://gitforwindows.org/). L’application sera installée dans le cadre des prérequis de ce labo.

- [Visual Studio Code](https://code.visualstudio.com/). Cela sera installé dans le cadre des prérequis de ce labo.

- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli). Installez Azure CLI sur les machines d’agent auto-hébergés.

- [Kit de développement logiciel (SDK) .NET - Dernière version](https://dotnet.microsoft.com/download/visual-studio-sdks) Installez le Kit de développement logiciel (SDK) sur les machines d’agent autohébergés.

## Instructions pour créer une organisation Azure DevOps (vous ne devez effectuer cette opération qu’une seule fois)

> **Remarque** : commencez à l’étape 3, si vous avez déjà configuré un **compte Microsoft personnel** et un abonnement Azure actif lié à ce compte.

1. Utilisez une session de navigateur privée pour obtenir un nouveau **compte Microsoft personnel (MSA)** à l’adresse `https://account.microsoft.com`.

1. À l’aide de la même session de navigateur, abonnez-vous gratuitement à Azure à l’adresse `https://azure.microsoft.com/free`.

1. Ouvrez un navigateur et accédez au Portail Azure sur `https://portal.azure.com`, puis recherchez **Azure DevOps** en haut de l’écran du Portail Azure. Dans la page qui s’affiche, cliquez sur **Organisations Azure DevOps**.

1. Ensuite, cliquez sur le lien intitulé **Mes organisations Azure DevOps** ou accédez directement à `https://aex.dev.azure.com`.

1. Sur la page **Nous avons besoin de quelques détails supplémentaires**, sélectionnez **Continuer**.

1. Dans la zone de liste déroulante de gauche, choisissez **Répertoire par défaut**, au lieu de **Compte Microsoft**.

1. Si vous recevez l’invite (_« Nous avons besoin de quelques détails supplémentaires »_, indiquez votre nom, votre adresse de messagerie et votre emplacement, puis cliquez sur **Continuer**.

1. Une fois de retour dans `https://aex.dev.azure.com`, si vous avez sélectionné **Répertoire par défaut**, cliquez sur le bouton bleu **Créer une organisation**.

1. Acceptez les _conditions d’utilisation_ en cliquant sur **Continuer**.

1. Si vous recevez l’invite (_« Vous avez presque terminé »)_, laissez le nom de l’organisation Azure DevOps par défaut (il doit s’agir d’un nom global unique) et choisissez un emplacement d’hébergement proche de vous dans la liste.

1. Une fois que l’organisation nouvellement créée s’ouvre dans **Azure DevOps**, sélectionnez **Paramètres de l’organisation** dans le coin inférieur gauche.

1. Dans l’écran **Paramètres de l’organisation**, sélectionnez **Facturation** (l’ouverture de cet écran prend quelques secondes).

1. Sélectionnez **Configurer la facturation** et, sur le côté droit de l’écran, sélectionnez votre **abonnement Azure**, puis sélectionnez **Enregistrer** pour lier l’abonnement à l’organisation.

1. Une fois que l’écran affiche l’ID d’abonnement Azure lié sur la partie supérieure, modifiez le nombre de **travaux parallèles payés** pour **CI/CD hébergé par MS** de 0 à **1**. Sélectionnez ensuite le bouton **ENREGISTRER** au bas de l’écran.

   > **Note** : vous pouvez **attendre quelques minutes avant d’utiliser les capacités CI/CD** afin que les nouveaux paramètres soient reflétés dans le serveur principal. Sinon, vous verrez toujours le message _« Aucun parallélisme hébergé n’a été acheté ou accordé »._

1. Dans **Paramètres d’organisation**, accédez à la section **Pipelines** et cliquez sur **Paramètres**.

1. Basculez le commutateur sur **Désactivé** pour **Désactiver la création de pipelines de build classiques** et **Désactiver la création de pipelines de mise en production classiques**.

   > **Note** : la définition de **Désactiver la création de pipelines de mise en production classiques** sur **Activé** masque les options de création de pipeline de mise en production classique comme le menu **Mise en production** dans la section **Pipeline** de DevOps Projects.

1. Dans **Paramètres d’organisation**, accédez à la section **Sécurité** et cliquez sur **Stratégies**.

1. Basculez le commutateur sur **Activé** pour **Autoriser les projets publics**.

   > **Note** : les extensions utilisées dans certains labos peuvent nécessiter un projet public pour autoriser l’utilisation de la version gratuite.

## Instructions pour créer et configurer le projet Azure DevOps (à faire une seule fois)

> **Remarque** : assurez-vous d’avoir effectué les étapes pour créer votre organisation Azure DevOps avant de poursuivre ces étapes.

Pour suivre toutes les instructions du labo, vous devez configurer un nouveau projet Azure DevOps, créer un référentiel basé sur l’application [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb), puis créer une connexion de service à votre abonnement Azure.

### Créer le projet d’équipe

Tout d’abord, vous allez créer un projet Azure DevOps **eShopOnWeb** à utiliser par plusieurs labos.

1. Ouvrez votre navigateur et accédez à votre organisation Azure DevOps.

1. Sélectionnez l’option **Nouveau projet** et utilisez les paramètres suivants :

   - nom : **eShopOnWeb**
   - Visibilité : **Privé**
   - Avancé : Contrôle de version : **Git**
   - Avancé : Processus d’élément de travail : **Scrum**

1. Sélectionnez **Créer**.

   ![Capture d’écran du volet de création d’un projet.](images/create-project.png)

### Importer un référentiel Git eShopOnWeb

À présent, vous allez importer eShopOnWeb dans votre référentiel Git.

1. Ouvrez votre navigateur et accédez à votre organisation Azure DevOps.

1. Ouvrez le projet **eShopOnWeb** que nous avons créé précédemment.

1. Sélectionnez **Repos > Fichiers**, **Importer un référentiel**, puis sélectionnez **Importer**.

1. Dans la fenêtre **Importer un référentiel Git**, collez l’URL `https://github.com/MicrosoftLearning/eShopOnWeb` suivante, puis sélectionnez **Importer** :

   ![Capture d’écran du volet Importer un référentiel.](images/import-repo.png)

1. Le référentiel est organisé de la manière suivante :

   - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
   - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
   - Le dossier **.azure** contient l’infrastructure de modèle ARM et Bicep en tant que modèles de code.
   - **Définitions de workflow GitHub YAML du conteneur de dossiers .github**.
   - Le dossier **src** contient le site web .NET 8 utilisé dans les scénarios de labo.

1. Laissez ouverte la fenêtre du navigateur web.

1. Accédez à **Repos > Branches**.

1. Pointez sur la branche **principale**, puis cliquez sur les points de suspension à droite de la colonne.

1. Cliquez sur **Définir comme branche par défaut**.

### Créer une connexion de service pour accéder aux ressources Azure

Vous devez créer une connexion de service dans Azure DevOps, ce qui vous permettra de déployer des ressources dans votre abonnement Azure et d’y accéder.

1. Lancez un navigateur web et accédez au portail Azure DevOps `https://aex.dev.azure.com`.

1. Connectez-vous à l’organisation Azure DevOps.

   > **Note** : si c’est la première fois que vous vous connectez à l’organisation Azure DevOps, vous devrez créer votre profil et accepter les conditions d’utilisation du service, puis sélectionner **Continuer**.

1. Ouvrez le projet **eShopOnWeb**, puis sélectionnez **Paramètres du projet** dans le coin inférieur gauche du portail.

1. Sous Pipelines, sélectionnez **Connexions de service**, puis le bouton **Créer une connexion de service**.

   ![Capture d’écran du bouton de création d’une connexion de service.](images/new-service-connection.png)

1. Dans le panneau **Nouvelle connexion de service**, sélectionnez **Azure Resource Manager**, puis **Suivant** (vous devrez peut-être faire défiler la page vers le bas).

1. Sélectionnez **Inscription d’application (automatique)** dans la dropbox **Type d’identité**.

1. Sélectionnez **Fédération des identités de charge de travail** et **Abonnement** sous le **niveau Étendue**.

   > **Remarque** : vous pouvez également utiliser l’**inscription d’application ou l’identité managée (manuelle)** si vous préférez configurer manuellement la connexion de service. Suivez les étapes de la [documentation Azure DevOps](https://learn.microsoft.com/azure/devops/pipelines/library/connect-to-azure) pour créer manuellement la connexion de service.

1. Renseignez les champs vides à l’aide des informations suivantes :

   - **Abonnement**: Sélectionnez votre abonnement Azure.
   - **Groupe de ressources** : sélectionnez le groupe de ressources dans lequel vous souhaitez déployer les ressources des services. Si vous n’avez pas de groupe de ressources, vous pouvez en créer un dans le portail Azure en suivant les instructions dans [Gérer les groupes de ressources Azure à l’aide du portail Azure](https://learn.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal).
   - **Nom de la connexion de service** : entrez **`azure subs`**. Ce nom sera référencé dans les pipelines YAML lors de l’accès à votre abonnement Azure.

1. Vérifiez que l’option **Accorder une autorisation d’accès à tous les pipelines** est décochée et sélectionnez **Enregistrer**.

   > **Remarque :** l’option **Accorder une autorisation d’accès à tous les pipelines** n’est pas recommandée pour les environnements de production. Elle est utilisée uniquement dans ce labo pour simplifier la configuration du pipeline.

   > **Note** : si vous voyez un message d’erreur indiquant que vous n’avez pas les autorisations nécessaires pour créer une connexion de service, réessayez ou configurez la connexion de service manuellement.

Vous avez maintenant effectué les étapes préalables nécessaires pour poursuivre les labos.
