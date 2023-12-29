---
lab:
  title: Contrôler les déploiements à l’aide de portes de mise en production classiques
  module: 'Module 04: Design and implement a release strategy'
---

# Contrôler les déploiements à l’aide de portes de mise en production classiques

## Manuel de labo de l’étudiant

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Créer une organisation ou une collection de projets](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifier un abonnement Azure existant ou en créer un.

- Vérifiez que vous disposez d’un compte Microsoft ou d’un compte Microsoft Entra avec le rôle Propriétaire dans l’abonnement Azure et le rôle Administrateur général dans le locataire Microsoft Entra associé à l’abonnement Azure. Pour plus d’informations, consultez [Répertorier les attributions de rôle Azure à l’aide du portail Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) et [Afficher et attribuer des rôles d’administrateur dans Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Présentation du labo

Ce laboratoire couvre la configuration des portes de déploiement et explique comment les utiliser pour contrôler l’exécution d’Azure Pipelines. Pour illustrer leur implémentation, vous allez configurer une définition de mise en production avec deux environnements pour une application web Azure. Vous allez déployer sur l’environnement DevTest uniquement lorsqu’il n’y a pas de bogues bloquants pour l’application et marquer l’environnement DevTest comme étant complet uniquement lorsqu’il n’y a pas d’alertes actives dans l’instance Application Insights d’Azure Monitor.

Un pipeline de mise en production spécifie le processus de mise en production de bout en bout pour qu’une application soit déployée dans différents environnements. Les déploiements sur chaque environnement sont entièrement automatisés à l’aide de travaux et de tâches. Dans l’idéal, vous ne souhaitez pas que les nouvelles mises à jour des applications soient exposées simultanément à tous les utilisateurs. Il est recommandé d’exposer les mises à jour de manière progressive, c’est-à-dire de les exposer à un sous-ensemble d’utilisateurs, de surveiller leur utilisation et de les exposer à d’autres utilisateurs en fonction de l’expérience de l’ensemble initial d’utilisateurs.

Les approbations et les portes vous permettent de prendre le contrôle sur le démarrage et l’achèvement des déploiements dans une mise en production. Vous pouvez attendre que les utilisateurs approuvent ou rejettent les déploiements avec des approbations manuellement. À l’aide de portes de mise en production, vous pouvez spécifier les critères d’intégrité de l’application à respecter avant que la version soit promue dans l’environnement suivant. Avant ou après tout déploiement d’environnement, toutes les portes spécifiées sont automatiquement évaluées jusqu’à ce qu’elles passent ou atteignent votre période d’expiration définie et échouent.

Les portes peuvent être ajoutées à un environnement dans la définition de mise en production à partir des conditions de pré-déploiement ou du panneau de conditions post-déploiement. Plusieurs portes peuvent être ajoutées aux conditions d’environnement pour garantir que toutes les entrées réussissent pour la mise en production.

Par exemple :

- Les portes de pré-déploiement garantissent l’absence de problème actif dans l’élément de travail ou le système de gestion des problèmes avant de déployer une build dans un environnement.
- Les portes de post-déploiement garantissent qu’aucun incident n’est survenu dans le système de surveillance ou de gestion des incidents de l’application après le déploiement avant de promouvoir la mise en production dans l’environnement suivant.

Il existe 4 types de portes incluses par défaut dans chaque compte.

- Appeler la fonction Azure : déclenche l’exécution d’une fonction Azure et garantir la réussite de l’opération.
- Interroger les alertes Azure Monitor : observe les règles d’alerte Azure Monitor configurées pour les alertes actives.
- Appeler l’API REST : appeler une API REST et continuer si elle retourne une réponse correcte.
- Interroger les éléments de travail : s’assurer que le nombre d’éléments de travail correspondants renvoyés par une requête est compris dans un seuil.

## Objectifs

À la fin de ce labo, vous serez en mesure d’accomplir les tâches suivantes :

- Configurez les pipelines de mise en production.
- Configurez les portes de mise en production.
- Testez les portes de mise en production.

## Durée estimée : 75 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

> **Remarque** : vous pouvez ignorer cet exercice si vous avez déjà créé ce projet dans le cadre des labos précédents.

Dans cet exercice, vous configurerez les prérequis du labo, qui se composent d’un nouveau projet Azure DevOps comprenant un référentiel basé sur [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tâche 1 : (à ignorer si vous l’avez déjà effectuée) créer et configurer le projet d’équipe

Dans le cadre de cette tâche, vous créerez un projet Azure DevOps **eShopOnWeb** qui sera utilisé dans plusieurs labos.

1. Sur votre ordinateur de labo, ouvrez votre organisation Azure DevOps dans votre navigateur. Cliquez sur **Nouveau projet**. Nommez votre projet **eShopOnWeb** et ne modifiez pas les autres champs par défaut. Cliquez sur **Créer**.

    ![Création d’un projet](images/create-project.png)

#### Tâche 2 : (à ignorer si vous l’avez déjà effectuée) importer un référentiel Git eShopOnWeb

Dans le cadre de cette tâche, vous importerez le référentiel Git eShopOnWeb qui sera utilisé dans plusieurs labos.

1. Sur votre ordinateur de labo, ouvrez votre organisation Azure DevOps dans votre navigateur ainsi que le projet **eShopOnWeb** que vous venez de créer. Cliquez sur **Repos > Fichiers**, puis sur **Importer un référentiel**. Cliquez sur **Importer**. Dans la fenêtre **Importer un référentiel Git**, collez l’URL https://github.com/MicrosoftLearning/eShopOnWeb.git, puis cliquez sur **Importer** :

    ![Importer un référentiel](images/import-repo.png)

2. Le référentiel est organisé de la manière suivante :
    - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
    - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
    - Le dossier **.azure** contient l’infrastructure Bicep&ARM en tant que modèles de code utilisés dans certains scénarios de labos.
    - **Définitions de workflow GitHub YAML du conteneur de dossiers .github**.
    - Le dossier **src** contient le site web .NET 7 utilisé dans les scénarios de labo.

#### Tâche 3 : (à ignorer si vous l’avez déjà effectuée) configurer un pipeline CI en tant que code avec YAML dans Azure DevOps

Dans le cadre de cette tâche, vous ajouterez une définition de build YAML au projet existant.

1. Revenez au volet **Pipelines** du hub **Pipelines**.
2. Dans la fenêtre **Créer votre premier pipeline**, cliquez sur **Créer un pipeline**.

    > **Remarque** : vous utiliserez l’assistant pour créer une définition de pipeline YAML basée sur votre projet.

3. Dans le volet **Où se trouve votre code ?**, sélectionnez l’option **Azure Repos Git (YAML)**.
4. Dans le volet **Sélectionner un référentiel**, cliquez sur **eShopOnWeb**.
5. Faites défiler le volet **Configurer votre pipeline** et sélectionnez **Fichier YAML Azure Pipelines existant**.
6. Dans le volet **Sélectionner un fichier YAML existant**, spécifiez les paramètres suivants :
   - Branche : **main**
   - Chemin d’accès : **.ado/eshoponweb-ci.yml**
7. Cliquez sur **Continuer** pour enregistrer ces paramètres.
8. Sur l’écran **Vérifier le YAML de votre pipeline**, cliquez sur **Exécuter** pour démarrer le processus de génération du pipeline.
9. Attendez la fin du processus de génération de pipeline. Ignorez les avertissements concernant le code source lui-même : ils n’ont aucune importance dans cet exercice de labo.

    > **Remarque** : chaque tâche issue du fichier YAML peut être révisée, y compris les avertissements et les erreurs.

10. Votre pipeline choisira un nom en fonction du nom du projet. **Renommons**-le pour mieux identifier le pipeline. Accédez à **Pipelines > Pipelines** et sélectionnez le pipeline que vous venez de créer. Cliquez sur les points de suspension, puis sélectionnez l’option **Renommer/Déplacer**. Nommez-le **eshoponweb-ci**, puis cliquez sur **Enregistrer**.

### Exercice 2 : créer les ressources Azure nécessaires pour le pipeline de mise en production

#### Tâche 1 : créer deux applications web Azure

Dans le cadre de cette tâche, vous créerez deux applications web Azure qui représentent les environnement **DevTest** et de **Production** dans lesquels vous déploierez l’application par l’intermédiaire d’Azure Pipelines.

1. Sur l’ordinateur de labo, démarrez un navigateur web, accédez au [**Portail Azure**](https://portal.azure.com) et connectez-vous avec le compte d’utilisateur qui a le rôle de Propriétaire dans l’abonnement Azure que vous utiliserez dans ce labo et qui a le rôle d’Administrateur général dans le locataire Microsoft Entra associé à cet abonnement.
2. Dans le Portail Azure, cliquez sur l’icône **Cloud Shell** située directement à droite de la zone de texte de recherche en haut de la page.
3. Si vous êtes invité à sélectionner **Bash** ou **PowerShell**, sélectionnez **Bash**.

   >**Remarque** : si c’est la première fois que vous démarrez **Cloud Shell** et que vous voyez le message **Vous n’avez aucun stockage monté**, sélectionnez l’abonnement que vous utilisez dans ce labo, puis sélectionnez **Créer un stockage**.

4. Dans l’invite **Bash** du volet **Cloud Shell**, exécutez la commande suivante pour créer un groupe de ressources (remplacez l’espace réservé de variable `<region>` par le nom de la région Azure qui hébergera les deux applications web Azure, par exemple « centralus », « westeurope » ou une autre région disponible de votre choix) :

    > **Remarque** : vous pouvez trouver les emplacements possibles en exécutant la commande suivante et en utilisant le **nom** sur `<region>` : `az account list-locations -o table`

    ```bash
    REGION='centralus'
    RESOURCEGROUPNAME='az400m04l09-RG'
    az group create -n $RESOURCEGROUPNAME -l $REGION
    ```

5. Comment créer un plan App Service

    ```bash
    SERVICEPLANNAME='az400m04l09-sp1'
    az appservice plan create -g $RESOURCEGROUPNAME -n $SERVICEPLANNAME --sku S1
    ```

6. Créez deux applications web avec des noms d’application uniques.

     ```bash
     SUFFIX=$RANDOM$RANDOM
     az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n RGATES$SUFFIX-DevTest
     az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n RGATES$SUFFIX-Prod
     ```

    > **Remarque** : enregistrez le nom de l’application web DevTest. Vous en aurez besoin plus tard dans ce labo.

7. Attendez la fin du processus d’approvisionnement des ressources App Services web, puis fermez le volet **Cloud Shell**.

#### Tâche 2 : configurer une ressource Application Insights

1. Sur le Portail Azure, utilisez la zone de texte **Rechercher des ressources, des services et des documents** en haut de la page pour rechercher **Application Insights**. Dans la liste des résultats qui apparaît, sélectionnez **Application Insights**.
2. Dans le volet **Application Insights**, sélectionnez **+ Créer**.
3. Dans l’onglet **Informations de base** du volet **Application Insights**, spécifiez les paramètres suivants (conservez les valeurs par défaut pour les autres) :

    | Paramètre | Valeur |
    | --- | --- |
    | Groupe de ressources | **az400m04l09-RG** |
    | Nom | Le nom de l’application web DevTest que vous avez enregistrée lors de la tâche précédente. |
    | Région | La région Azure où vous avez déployé les applications web de la tâche précédente. |
    | Mode ressource | **Classique** |

    > **Remarque** : ignorez le message vous déconseillant de poursuivre l’opération. Celle-ci est nécessaire pour éviter les défaillances de la tâche Activer l’intégration continue DevOps que vous utiliserez plus tard dans ce labo.

4. Cliquez sur **Vérifier + créer**, puis sur **Créer**.
5. Attendez la fin du processus de provisionnement.
6. Dans le Portail Azure, accédez au groupe de ressources **az400m04l09-RG** que vous venez de créer lors de la tâche précédente.
7. Dans la liste des ressources, cliquez sur l’application web **DevTest**.
8. Dans le menu vertical à gauche de la page de l’application web **DevTest**, cliquez sur **Application Insights** dans la section **Paramètres**.
9. Dans le volet **Application Insights**, cliquez sur **Activer Application Insights**.
10. Dans la section **Modifier votre ressource**, cliquez sur l’option **Sélectionner une ressource existante**. Dans la liste des ressources existantes, sélectionnez la ressource Application Insight que vous venez de créer. Sélectionnez ensuite **Appliquer** et, lorsque vous y êtes invité, cliquez sur **Oui**.
11. Attendez que la modification prenne effet.

    > **Remarque** : ici, vous créerez des alertes de surveillance que vous utiliserez plus tard dans ce labo.
12. Dans la même option de menu **Paramètres ** / **d’Application Insights** de l’application web, sélectionnez **Afficher les données d’Application Insights**. Cette action vous renvoie au volet Application Insights du Portail Azure.
13. Dans la section **Surveillance** du volet Ressources d’Application Insights, cliquez sur **Alertes**, puis sur **Créer > Règle d’alerte**.
14. Dans la zone de texte **Rechercher un signal** du volet **Sélectionner un signal**, saisissez **Requêtes**. Dans la liste des résultats qui apparaît, sélectionnez **Requêtes en échec**.
15. Dans la section **Condition** du volet **Créer une règle d’alerte**, laissez le paramètre **Seuil** défini sur **Statique** et validez les autres paramètres par défaut de la manière suivante :
    - Type d’agrégation : Nombre
    - Opérateur : Supérieur à
    - Unité: Nombre
16. Dans la zone de texte **Valeur du seuil**, saisissez **0**, puis cliquez sur **Next:Actions**. N’apportez aucune modification dans le volet des paramètres d’**actions** et réglez les paramètres suivants sous la section **Détails** :

    | Paramètre | Valeur |
    | --- | --- |
    | Niveau de gravité | **2- Avertissement** |
    | Nom de la règle d’alerte | **RGATESDevTest_FailedRequests** |
    | Options avancées : résoudre automatiquement les alertes | **désactivé** |

    > **Remarque** : les règles d’alerte liées aux métriques peuvent prendre jusqu’à 10 minutes pour s’activer.

    > **Remarque** : vous pouvez créer plusieurs règles d’alerte sur différentes métriques, telles que la disponibilité inférieure à 99 %, le temps de réponse du serveur inférieur à 5 secondes ou les exceptions du serveur supérieures à 0.

17. Confirmez la création de la règle d’alerte en cliquant sur **Vérifier + Créer**, puis confirmez votre choix en cliquant sur **Créer**. Attendez la fin de la création de la règle d’alerte.

### Exercice 3 : configurer le pipeline de mise en production

Dans le cadre de cet exercice, vous configurerez un pipeline de mise en production.

#### Tâche 1 : configurer des tâches de mise en production

Dans le cadre de cette tâche, vous configurerez les tâches de mise en production dans le cadre du pipeline de mise en production.

1. Dans le volet de navigation vertical du Portail Azure DevOps où est ouvert le projet **eShopOnWeb**, sélectionnez **Pipelines**. Dans la section **Pipelines**, cliquez ensuite sur **Mises en production**.
2. Cliquez sur **Nouveau pipeline**.
3. Dans la fenêtre **Sélectionner un modèle**, **cliquez sur** **Déploiement Azure App Service** (Déployer votre application sur Azure App Service. Faites votre choix entre application web sur Windows, Linux, conteneurs, Applications de fonction ou WebJobs) dans la liste des **propositions** de modèles.
4. Cliquez sur **Appliquer**.
5. Dans la fenêtre **Étape** qui s’affiche, modifiez le nom par défaut de l’étape « Étape 1 » et nommez-la **DevTest**. Fermez la fenêtre contextuelle en cliquant sur le bouton **X**. Vous êtes maintenant dans l’éditeur graphique du pipeline de mise en production qui affiche l’étape DevTest.
6. En haut de la page, changez le nom du pipeline actuel **Nouveau pipeline de mise en production** et nommez-le **eshoponweb-cd**.
7. Survolez l’étape DevTest avec votre curseur, puis cliquez sur le bouton **Cloner** pour copier l’étape DevTest vers une étape supplémentaire. Nommez cette étape **Production**.

    > **Remarque** : le pipeline contient désormais deux étapes nommées **DevTest** et **Production**.

8. Dans l’onglet **Pipeline**, sélectionnez le rectangle **Ajouter un artefact**, puis cliquez sur **eshoponweb-ci**dans le champ **Source (pipeline de build)**. Cliquez sur **Ajouter** pour confirmer la sélection de l’artefact.
9. Le **déclencheur de déploiement continu** (symbole d’éclair) apparaît dans le rectangle **Artefacts**. Cliquez dessus pour ouvrir les paramètres du **Déclencheur de déploiement continu**. Cliquez sur le bouton à bascule **Désactivé** pour activer le déclencheur. Ne modifiez aucun autre paramètre par défaut, et fermez le volet **Déclencheur de déploiement continu** en cliquant sur le symbole **x** dans le coin supérieur droit.
10. Dans l’étape **Environnements DevTest**, cliquez sur l’étiquette **1 travail, 1 tâche** et examinez les tâches de cette étape.

    > **Remarque** : l’environnement DevTest comprend une tâche qui publie le package d’artefacts sur l’application web Azure.

11. Dans le volet **Tous les pipelines > eshoponweb-cd**, assurez-vous que l’étape **DevTest** est sélectionnée. Dans la liste déroulante Abonnement **Abonnement Azure**, sélectionnez votre abonnement Azure et cliquez sur **Autoriser**. Si vous y êtes invité, authentifiez-vous à l’aide du compte d’utilisateur ayant le rôle de Propriétaire dans l’abonnement Azure.
12. Vérifiez que le type d’application est défini sur « Application web sur Windows ». Ensuite, dans la liste déroulante **Nom App Service**, sélectionnez le nom de l’application web **DevTest**.
13. Sélectionnez la tâche **Déployer Azure App Service**. Dans le champ **Package ou dossier**, modifiez la valeur par défaut de « $(System.DefaultWorkingDirectory)/\*\*/\*.zip » en « $(System.DefaultWorkingDirectory)/\*\*/Web.zip ».

    > Un point d’exclamation se trouve en regard de l’onglet Tâches : cela est tout à fait normal, puisqu’il faut configurer les paramètres de l’étape de production.

14. Dans le volet **Tous les pipelines > eshoponweb-cd**, accédez à l’onglet **Pipeline**. Dans l’étape de **production**, cliquez à présent sur l’étiquette **1 travail, 1 tâche**. À l’instar de l’étape précédente DevTest, renseignez les paramètres du pipeline. Dans l’onglet Tâches/le processus de déploiement de production de la liste déroulante **Abonnement Azure**, sélectionnez l’abonnement Azure que vous avez utilisé pour l’étape d’**environnement de DevTest**, qui se trouve sous les **connexions aux services Azure disponibles**, puisque vous avez créé la connexion aux services avant d’autoriser l’utilisation de l’abonnement.
15. Dans la liste déroulante **Nom App Service**, sélectionnez le nom de l’application web **Prod**.
16. Sélectionnez la tâche **Déployer Azure App Service**. Dans le champ **Package ou dossier**, modifiez la valeur par défaut de « $(System.DefaultWorkingDirectory)/\*\*/\*.zip » en « $(System.DefaultWorkingDirectory)/\*\*/Web.zip ».
17. Dans le volet **Tous les pipelines > eshoponweb-cd**, cliquez sur **Enregistrer**. Dans la boîte de dialogue **Enregistrer**, cliquez sur **OK**.

    Vous venez de configurer le pipeline de mise en production.

18. Dans le volet de navigation vertical de la fenêtre du navigateur qui affiche le projet **eShopOnWeb**, cliquez sur **Pipelines** dans la section **Pipelines**.
19. Dans le volet **Pipelines**, cliquez sur l’entrée représentant le pipeline de build **eshoponweb-ci**, puis sélectionnez **Exécuter le pipeline**.
20. Dans le volet **Exécuter le pipeline**, acceptez les paramètres par défaut, puis cliquez sur **Exécuter** pour déclencher le pipeline. **Attendez la fin de l’exécution du pipeline de build**.

    > **Remarque** : une fois le build terminé, la mise en production sera automatiquement déclenchée et l’application sera déployée dans les deux environnements. Validez les actions de mise en production une fois l’exécution du pipeline de build terminée.

21. Dans la **section Pipelines** du volet de navigation vertical, cliquez sur **Mises en production**. Dans le volet **eshoponweb-cd**, cliquez sur l’entrée représentant la version la plus récente.
22. Dans le volet **eshoponweb-cd > Release-1**, suivez la progression de la version et vérifiez que le déploiement sur les deux applications web a réussi.
23. Revenez au Portail Azure et accédez au groupe de ressources **az400m04l09-RG**. Dans la liste des ressources, cliquez sur l’application web **DevTest**. Dans le panneau de l’application web, cliquez sur **Parcourir** et assurez-vous que la page web (le site web d’e-commerce) se charge correctement dans un nouvel onglet du navigateur web.
24. Revenez au Portail Azure et accédez cette fois au groupe de ressources **az400m04l09-RG**. Dans la liste des ressources, cliquez sur l’application web **Production**. Dans le panneau de l’application web, cliquez sur **Parcourir** et assurez-vous que la page web se charge correctement dans un nouvel onglet du navigateur web.
25. Fermez l’onglet du navigateur web où est ouvert le site web **EShopOnWeb**.

    > **Remarque** : vous venez de configurer le CI/CD de l’application. Dans l’exercice suivant, vous configurerez des portes de qualité dans le cadre d’un pipeline de mise en production plus avancé.

### Exercice 4 : configurer les portes de mise en production

Dans le cadre de cet exercice, vous configurerez des portes de qualité dans le pipeline de mise en production.

#### Tâche 1 : configurer des portes de pré-déploiement pour les approbations

Dans le cadre de cette tâche, vous configurerez des portes de pré-déploiement.

1. Revenez à la fenêtre du navigateur web qui affiche le Portail Azure DevOps et ouvrez le projet **eShopOnWeb**. Dans la section **Pipelines** du volet de navigation vertical, cliquez sur **Mises en production**. Dans le volet **eshoponweb-cd**, cliquez sur **Modifier**.
2. Dans le volet **Tous les pipelines > eshoponweb-cd**, cliquez sur l’ovale représentant les **Conditions de pré-déploiement** dans le coin supérieur gauche du rectangle représentant l’étape d’**environnement DevTest**.
3. Dans le volet **Conditions de pré-déploiement**, placez le curseur des **approbations de pré-déploiement** sur **Activé**, puis, saisissez et sélectionnez le nom de votre compte Azure DevOps dans la zone de texte **Approbateurs**.

    > **Remarque** : dans un scénario réel, il doit s’agir d’un alias de nom d’équipe DevOps et non pas de votre propre nom.

4. **Enregistrez** les paramètres de pré-approbation et fermez la fenêtre contextuelle.
5. Cliquez sur **Créer une mise en production** et confirmez votre choix en cliquant sur le bouton **Créer** dans la fenêtre contextuelle.
6. Un message de confirmation vert indiquant que « Release-2 » a bien été créée apparaît. Cliquez sur le lien « Release-2 » pour en consulter les détails.
7. L’état de l’étape **DevTest** indique **En attente d’approbation**. Cliquez sur le bouton **Approuver**. Cela relance l’étape DevTest.

#### Tâche 2 : configurer des portes de post-déploiement pour Azure Monitor

Dans le cadre de cette tâche, vous activerez la porte de post-déploiement pour l’environnement DevTest.

1. Revenez au volet **Tous les pipelines > eshoponweb-cd**, puis cliquez sur l’ovale représentant les **Conditions de post-déploiement** dans le coin supérieur droit du rectangle représentant l’étape d’**environnement DevTest**.
2. Dans le volet **Conditions de post-déploiement**, placez le curseur **Portes** sur **Activé** et cliquez sur **+ Ajouter**. Dans le menu contextuel qui apparaît, cliquez sur **Interroger les alertes d’Azure Monitor**.
3. Dans la section **Interroger les alertes d’Azure Monitor** du volet **Conditions de post-déploiement**, ouvrez la liste déroulante **Abonnement Azure** et sélectionnez-y la **connexion de services** qui représente la connexion à votre abonnement Azure. Dans la liste déroulante **Groupe de ressources**, sélectionnez **az400m04l09-RG**.
4. Dans le volet **Conditions de post-déploiement**, développez la section **Avancé** et configurez les options suivantes :

   - Type de filtre : **Aucun**
   - Gravité : **Sev0, Sev1, Sev2, Sev3, Sev4**
   - Intervalle de temps : **Dernière heure**
   - État d’alerte : **Reconnue, Nouvelle**
   - Condition de la surveillance : **Déclenchée**

5. Dans le volet **Conditions de post-déploiement**, développez la section **Options d’évaluation** et configurez les options suivantes :

   - Définissez la valeur **Délai entre la réévaluation des portes** sur **5 minutes**.
   - Définissez la valeur **Délai d’expiration des portes** sur **8 minutes**.
   - Sélectionnez l’option **Demander les approbations lorsque les portes réussissent**.

    > **Remarque** : l’intervalle d’échantillonnage et le délai d’expiration fonctionnent ensemble de manière à ce que les portes appellent leurs fonctions à des intervalles adéquats et rejettent le déploiement si elles ne réussissent pas pendant le même intervalle d’échantillonnage compris dans la période d’expiration.

6. Fermez le volet **Conditions de post-déploiement** en cliquant sur le symbole **x** dans son coin supérieur droit.
7. Revenez au volet **eshoponweb-cd** et cliquez sur **Enregistrer**. Dans la boîte de dialogue **Enregistrer**, cliquez sur **OK**.

### Exercice 5 : tester les portes de mise en production

Dans le cadre de cet exercice, vous testerez les portes de mise en production en mettant à jour l’application, ce qui déclenchera un déploiement.

#### Tâche 1 : mettre à jour et déployer l’application après l’ajout des portes de mise en production

Dans le cadre de cette tâche, vous commencerez par générer des alertes pour l’application web DevTest, puis vous suivrez le processus de mise en production avec les portes de mise en production activées.

1. Dans le Portail Azure, accédez à la ressource **Application web DevTest** que vous venez de déployer.
2. Dans le volet Vue d’ensemble, le champ **URL** reprend le lien hypertexte de l’application web. Cliquez dessus pour ouvrir l’application web EShopOnWeb dans le navigateur.
3. Pour simuler un **échec de la demande**, ajoutez **/discount** à l’URL. Un message d’erreur apparaît alors, car cette page n’existe pas. Actualisez cette page plusieurs fois pour générer plusieurs événements.
4. Dans le champ « Rechercher des ressources, des services et des documents » du Portail Azure, saisissez **Application Insights** et sélectionnez la ressource **DevTest-AppInsights** créée lors de l’exercice précédent. Accédez ensuite aux **alertes**.
5. Au moins **1** nouvelle alerte doit se trouver dans la liste des résultats, d’un niveau de **sévérité 2**. Ouvrez les **alertes** pour accéder au service Alertes d’Azure Monitor.
6. Au moins **1** Failed_Alert de niveau de **gravité 2 : avertissement** doit se trouver dans la liste. Cette alerte a été déclenchée lorsque vous avez validé l’adresse URL du site web inexistant lors de l’exercice précédent.

    > **Remarque :** si vous ne voyez aucune alerte, patientez quelques minutes.

7. Revenez au Portail Azure DevOps et ouvrez le projet **EShopOnWeb**. Accédez aux **pipelines**, sélectionnez **Mises en production** et cliquez sur **eshoponweb-cd**.
8. Cliquez sur le bouton **Créer une mise en production**.
9. Attendez que le pipeline de mise en production démarre et **approuvez** l’action de mise en production de l’étape DevTest.
10. Attendez la fin de l’étape de mise en production de DevTest. Vous remarquerez que les **portes de post-déploiement** passent à l’état **Portes d’évaluation**.  Cliquez sur l’icône **Portes d’évaluation**.
11. **Interroger les alertes Azure Monitor** indique un état d’échec initial.
12. Laissez le pipeline de mise en production dans l’état « En attente » pendant 5 minutes. Une fois les 5 minutes écoulées, vous verrez que la deuxième évaluation a de nouveau échoué.
13. Ce comportement est normal, puisqu’une alerte Application Insights a été déclenchée pour l’application web DevTest.

    > **Remarque** : puisqu’une alerte est déclenchée par l’exception, la porte **Interroger Azure Monitor** échouera. Cela empêchera le déploiement dans l’environnement de **production**.

14. Patientez quelques minutes et validez à nouveau l’état des portes de mise en production. Quelques minutes après la vérification initiale des portes de mise en production et après le déclenchement de la première alerte Application Insights avec l’action « Déclenchée », une porte de mise en production doit réussir et permettre le déploiement de l’étape de mise en production.

    > **Remarque :** si votre porte échoue, fermez l’alerte.

### Exercice 6 : Supprimer les ressources du labo Azure.

Dans le cadre de cet exercice, vous supprimerez les ressources Azure provisionnées dans ce labo pour éliminer les frais inattendus.

>**Remarque** : N’oubliez pas de supprimer toutes les nouvelles ressources Azure que vous n’utilisez plus. La suppression des ressources inutilisées vous évitera d’encourir des frais inattendus.

#### Tâche 1 : supprimer les ressources du labo Azure

Dans le cadre de cette tâche, vous utiliserez Azure Cloud Shell pour supprimer les ressources Azure provisionnées dans ce labo afin d’éliminer les frais superflus.

1. Dans le Portail Azure, ouvrez la session shell **Bash** dans le volet **Cloud Shell**.
1. Listez tous les groupes de ressources créés dans les labos de ce module en exécutant la commande suivante :

    ```sh
    az group list --query "[?starts_with(name,'az400m04l09-RG')].name" --output tsv
    ```

1. Supprimez tous les groupes de ressources que vous avez créés dans les labos de ce module en exécutant la commande suivante :

    ```sh
    az group list --query "[?starts_with(name,'az400m04l09-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Remarque** : La commande s’exécute de façon asynchrone (comme déterminé par le paramètre --no-wait). Par conséquent, vous serez en mesure d’exécuter une autre commande Azure CLI immédiatement après au cours de la même session Bash, mais la suppression réelle du groupe de ressources prendra quelques minutes.

## Révision

Dans le cadre de ce labo, vous avez configuré des pipelines de mise en production, puis configuré et testé les portes de mise en production.
