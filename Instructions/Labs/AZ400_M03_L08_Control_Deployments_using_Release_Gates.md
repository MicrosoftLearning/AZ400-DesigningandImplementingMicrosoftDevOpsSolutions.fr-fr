---
lab:
  title: Contrôler les déploiements en utilisant des jalons de version
  module: 'Module 03: Design and implement a release strategy'
---

# Contrôler les déploiements en utilisant des jalons de version

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Créer une organisation ou une collection de projets](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifier un abonnement Azure existant ou en créer un.

- Vérifiez que vous disposez d’un compte Microsoft ou d’un compte Microsoft Entra avec le rôle Propriétaire dans l’abonnement Azure et le rôle Administrateur général dans le locataire Microsoft Entra associé à l’abonnement Azure. Pour plus d’informations, consultez [Répertorier les attributions de rôle Azure à l’aide du portail Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) et [Afficher et attribuer des rôles d’administrateur dans Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Présentation du labo

Ce laboratoire couvre la configuration des portes de déploiement et explique comment les utiliser pour contrôler l’exécution d’Azure Pipelines. Pour illustrer leur implémentation, vous allez configurer une définition de mise en production avec deux environnements pour une application web Azure. Vous déployerez l’environnement DevTest uniquement lorsqu’il n’y a pas de blocages pour l’application et vous marquerez l’environnement DevTest comme étant complet uniquement lorsqu’il n’y a pas d’alertes actives dans l’Application Insights d’Azure Monitor.

Un pipeline de mise en production spécifie le processus de mise en production de bout en bout pour qu’une application soit déployée dans différents environnements. Les déploiements sur chaque environnement sont entièrement automatisés à l’aide de travaux et de tâches. Dans l’idéal, vous ne souhaitez pas que les nouvelles mises à jour des applications soient exposées simultanément à tous les utilisateurs. Il est recommandé d’exposer les mises à jour de manière progressive, c’est-à-dire de les exposer à un sous-ensemble d’utilisateurs, de surveiller leur utilisation et de les exposer à d’autres utilisateurs en fonction de l’expérience de l’ensemble initial d’utilisateurs.

Les approbations et les portes vous permettent de prendre le contrôle sur le démarrage et l’achèvement des déploiements dans une mise en production. Vous pouvez attendre que les utilisateurs approuvent ou rejettent les déploiements avec des approbations manuellement. À l’aide de portes de mise en production, vous pouvez spécifier les critères d’intégrité de l’application à respecter avant que la version soit promue dans l’environnement suivant. Avant ou après tout déploiement d’environnement, toutes les portes spécifiées sont automatiquement évaluées jusqu’à ce qu’elles passent ou atteignent votre période d’expiration définie et échouent.

Les portes peuvent être ajoutées à un environnement dans la définition de mise en production à partir des conditions de pré-déploiement ou du panneau de conditions post-déploiement. Plusieurs portes peuvent être ajoutées aux conditions d’environnement pour garantir que toutes les entrées réussissent pour la mise en production.

Par exemple :

- Les portes de pré-déploiement garantissent l’absence de problème actif dans l’élément de travail ou le système de gestion des problèmes avant de déployer une build dans un environnement.
- Les portes de post-déploiement garantissent qu’aucun incident n’est survenu dans le système de surveillance ou de gestion des incidents de l’application après le déploiement avant de promouvoir la mise en production dans l’environnement suivant.

Il existe 4 types de portails inclus par défaut dans chaque compte.

- Appeler la fonction Azure : déclenche l’exécution d’une fonction Azure et garantir la réussite de l’opération.
- Interroger les alertes Azure Monitor : observe les règles d’alerte Azure Monitor configurées pour les alertes actives.
- Appeler l’API REST : appeler une API REST et continuer si elle retourne une réponse correcte.
- Interroger les éléments de travail : s’assurer que le nombre d’éléments de travail correspondants renvoyés par une requête est compris dans un seuil.

## Objectifs

À la fin de ce labo, vous serez en mesure d’effectuer les tâches suivantes :

- Configurez les pipelines de mise en production.
- Configurez les portes de mise en production.
- Testez les portes de mise en production.

## Durée estimée : 75 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

Dans cet exercice, vous allez configurer les prérequis du labo.

#### Tâche 1 : (passer si terminée) créer et configurer le projet d’équipe

Dans cette tâche, vous allez créer un projet Azure DevOps **eShopOnWeb** à utiliser par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps. Cliquez sur **Nouveau projet**. Attribuez au projet le nom **eShopOnWeb** et conservez les valeurs par défaut des autres champs. Cliquez sur **Créer**.

   ![Capture d’écran du volet de création d’un nouveau projet.](images/create-project.png)

#### Tâche 2 : (passer si terminée) importer le référentiel Git eShopOnWeb

Dans cette tâche, vous allez importer le référentiel Git eShopOnWeb qui sera utilisé par plusieurs labos.

1. Sur votre ordinateur de labo, dans une fenêtre de navigateur, ouvrez votre organisation Azure DevOps et le projet **eShopOnWeb** créé précédemment. Cliquez sur **Repos > Fichiers**, **Importer un référentiel**. Cliquez sur **Importer**. Dans la fenêtre **Importer un référentiel Git**, collez l’URL <https://github.com/MicrosoftLearning/eShopOnWeb.git>, puis cliquez sur **Importer** :

   ![Capture d’écran du volet Importer un référentiel.](images/import-repo.png)

1. Le référentiel est organisé de la manière suivante :
   - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
   - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
   - Le dossier **infra** contient l’infrastructure Bicep&ARM sous forme de modèles de code utilisés dans certains scénarios de labo.
   - **Définitions de workflow GitHub YAML du conteneur de dossiers .github**.
   - Le dossier **src** contient le site web .NET 8 utilisé dans les scénarios de labo.

1. Accédez à **Dépôts > Branches**.
1. Pointez sur la branche **principale**, puis cliquez sur les points de suspension à droite de la colonne.
1. Cliquez sur **Définir comme branche par défaut**.

#### Tâche 3 : configurer les pipelines CI en tant que code avec YAML dans Azure DevOps

Dans cette tâche, vous allez ajouter une définition de build YAML au projet existant.

1. Revenez au volet **Pipelines** du hub **Pipelines**.
1. Dans la fenêtre **Créer votre premier pipeline**, cliquez sur **Créer un pipeline**.

   > **Remarque** : nous allons utiliser l’Assistant pour créer une définition de pipeline YAML basée sur notre projet.

1. Dans le volet **Où se trouve votre code ?**, cliquez sur l’option **Azure Repos Git (YAML)**.
1. Dans le volet **Sélectionner un référentiel**, cliquez sur **eShopOnWeb**.
1. Dans le volet **Configurer votre pipeline**, faites défiler la page vers le bas, puis sélectionnez **Fichier YAML Azure Pipelines existant**.
1. Dans le panneau **Sélectionner un fichier YAML existant**, spécifiez les paramètres suivants :
   - Branche : **main**
   - Chemin : **.ado/eshoponweb-ci.yml**
1. Cliquez sur **Continuer** pour enregistrer les paramètres.
1. Dans l’écran **Passer en revue le fichier YAML de pipeline**, cliquez sur **Exécuter** pour démarrer le processus de pipeline de build.
1. Attendez la fin du pipeline de build. Ignorez les avertissements concernant le code source lui-même, car ils ne sont pas pertinents pour cet exercice de labo.

   > **Remarque** : chaque tâche du fichier YAML est disponible pour révision, y compris les avertissements et les erreurs.

1. Votre pipeline est nommé en fonction du nom du projet. **Renommons**-le pour mieux identifier le pipeline. Accédez à **Pipelines > Pipelines**, puis cliquez sur le pipeline qui vient d’être créé. Cliquez sur les points de suspension et sur l’option **Renommer/déplacer**. Nommez-le **`eshoponweb-ci`** et cliquez sur **Enregistrer**.

### Exercice 1 : créer les ressources Azure nécessaires pour le pipeline de mise en production

#### Tâche 1 : créer deux applications web Azure

Dans cette tâche, vous allez créer deux applications web Azure représentant les environnements **DevTest** et **Production**, dans lesquels vous allez déployer l’application via Azure Pipelines.

1. Sur l’ordinateur de labo, démarrez un navigateur web, accédez au [**portail Azure**](https://portal.azure.com) et connectez-vous avec le compte d’utilisateur qui a le rôle Propriétaire dans l’abonnement Azure que vous utiliserez dans ce labo et le rôle Administrateur général dans le locataire Microsoft Entra associé à cet abonnement.
1. Dans le portail Azure, cliquez sur l’icône **Cloud Shell**, située directement à droite de la zone de texte de recherche en haut de la page.
1. Si vous êtes invité à sélectionner **Bash** ou **PowerShell**, sélectionnez **Bash**.

   > **Remarque** : si c’est la première fois que vous démarrez **Cloud Shell** et que vous voyez le message **Vous n’avez aucun stockage monté**, sélectionnez l’abonnement que vous utilisez dans ce labo, puis sélectionnez **Créer un stockage**.

1. À partir de l’invite **Bash**, dans le volet **Cloud Shell**, exécutez la commande suivante pour créer un groupe de ressources (remplacez l’`<region>`espace réservé de variable par le nom de la région Azure qui hébergera les deux applications web Azure, par exemple « westeurope » ou « centralus » ou toute autre région disponible de votre choix) :

   > **Remarque** : les emplacements possibles sont disponibles en exécutant la commande suivante. Utilisez le **Nom** sur `<region>` : `az account list-locations -o table`

   ```bash
   REGION='<region>'
   RESOURCEGROUPNAME='az400m03l08-RG'
   az group create -n $RESOURCEGROUPNAME -l $REGION
   ```

1. Pour créer un plan App Service

   ```bash
   SERVICEPLANNAME='az400m03l08-sp1'
   az appservice plan create -g $RESOURCEGROUPNAME -n $SERVICEPLANNAME --sku S1
   ```

1. Créez deux applications web avec des noms d’application uniques.

   ```bash
   SUFFIX=$RANDOM$RANDOM
   az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n RGATES$SUFFIX-DevTest
   az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n RGATES$SUFFIX-Prod
   ```

   > **Remarque** : enregistrez le nom de l’application web DevTest. Vous en aurez besoin plus tard dans ce labo.

1. Attendez la fin du processus d’approvisionnement des ressources des services d’application web, puis fermez le volet **Cloud Shell**.

#### Tâche 2 : configurer une ressource Application Insights

1. Dans le portail Azure, utilisez la zone de texte **Rechercher des ressources, services et documents** en haut de la page pour rechercher **Application Insights**, puis, dans la liste des résultats, sélectionnez **Application Insights**.
1. Dans le volet **Application Insights**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** du panneau **Application Insights**, spécifiez les paramètres suivants (laissez les valeurs par défaut des autres paramètres) :

   | Paramètre        | Valeur                                                                                 |
   | -------------- | ------------------------------------------------------------------------------------- |
   | Resource group | **az400m03l08-RG**                                                                    |
   | Nom           | Nom de l’application web DevTest que vous avez enregistrée dans la tâche précédente                     |
   | Région         | Même région Azure sur laquelle vous avez déployé les applications web plus tôt dans la tâche précédente |

1. Cliquez sur **Vérifier + créer**, puis sur **Créer**.
1. Attendez la fin du processus de provisionnement.
1. Dans le portail Azure, accédez au groupe de ressources **az400m03l08-RG** que vous avez créé dans la tâche précédente.
1. Dans la liste des ressources, cliquez sur l’application web **DevTest**.
1. Dans la page de l’application web **DevTest**, dans le menu vertical de gauche, dans la section **Surveillance**, cliquez sur **Application Insights**.
1. Dans le panneau **Application Insights**, sélectionnez **Activer Application Insights**.
1. Dans la section **Modifier votre ressource**, cliquez sur l’option **Sélectionner une ressource existante**. Dans la liste des ressources existantes, sélectionnez la ressource Application Insight nouvellement créée, cliquez sur **Appliquer** et, lorsque vous y êtes invité, cliquez sur **Oui**.
1. Attendez que la modification prenne effet.

   > **Remarque** : vous allez maintenant créer des alertes de surveillance, que vous utiliserez ultérieurement dans ce labo.

1. Dans la même option de menu **Paramètres** / **Application Insights** de l’application web, sélectionnez **Afficher les données Application Insights**. Le panneau Application Insights s’ouvre dans le portail Azure.
1. Dans le panneau des ressources Application Insights, dans la section **Surveillance**, cliquez sur **Alertes**, puis sur **Créer > règle d’alerte**.
1. Dans le panneau **Créer une règle d’alerte**, dans la section **Condition**, cliquez sur le lien **Afficher tous les signaux** et saisissez **Demandes**. Dans la liste des résultats, sélectionnez **Demandes ayant échoué**.
1. Dans le panneau **Créer une règle d’alerte**, dans la section **Condition**, laissez le **Seuil** défini sur **Statique**, validez les autres paramètres par défaut comme suit :

   - Type d’agrégation : Count
   - Opérateur : Supérieur à
   - Unité: Nombre

1. Dans la zone de texte **Valeur seuil**, saisissez **0** puis cliquez sur **Next:Actions**. N’apportez aucune modification dans le panneau de paramètres **Actions** et définissez les paramètres suivants sous la section **Détails** :

   | Paramètre                                        | Valeur                            |
   | ---------------------------------------------- | -------------------------------- |
   | Niveau de gravité                                       | **2 - Avertissement**                   |
   | Nom de la règle d’alerte                                | **RGATESDevTest_FailedRequests** |
   | Options avancées : résoudre automatiquement les alertes | **désactivé**                      |

   > **Remarque** : les règles d’alerte de métrique peuvent prendre jusqu’à 10 minutes pour s’activer.

   > **Remarque** : vous pouvez créer plusieurs règles d’alerte sur différentes métriques, telles que la disponibilité < 99 %, le temps de réponse du serveur > 5 secondes ou les exceptions de serveur > 0

1. Confirmez la création de la règle d’alerte en cliquant sur **Vérifier + créer**, puis confirmez une fois de plus en cliquant sur **Créer**. Attendez que la règle d’alerte soit créée.

### Exercice 2 : configurer le pipeline de mise en production

Dans cet exercice, vous allez configurer un pipeline de mise en production.

#### Tâche 1 : configurer les tâches de mise en production

Dans cette tâche, vous allez configurer les tâches de mise en production dans le cadre du pipeline de mise en production.

1. Dans le projet **eShopOnWeb** du portail Azure DevOps, dans le volet de navigation vertical, sélectionnez **Pipelines**, puis, dans la section **Pipelines**, cliquez sur **Mises en production**.
1. Cliquez sur **Nouveau pipeline**.
1. Dans la fenêtre **Sélectionner un modèle**, **choisissez****Déploiement d’Azure App Service** (Déployer votre application sur Azure App Service. Choisissez l’application web parmi Windows, Linux, des conteneurs, des applications de fonction ou WebJobs) sous la **liste proposée** des modèles.
1. Cliquez sur **Appliquer**.
1. Dans la fenêtre **Étape** qui s’affiche, modifiez le nom par défaut de l’étape « Étape 1 » en **DevTest**. Fermez la fenêtre contextuelle à l’aide du bouton **X**. Vous êtes maintenant dans l’éditeur graphique du pipeline de mise en production, montrant l’étape DevTest.
1. En haut de la page, renommez le pipeline actuel de **Nouveau pipeline de mise en production** en **eshoponweb-cd**.
1. Pointez la souris sur l’étape DevTest puis cliquez sur le bouton**Cloner** pour copier l’étape DevTest vers une étape supplémentaire. Nommez cette étape **Production**.

   > **Remarque** : le pipeline contient désormais deux étapes nommées **DevTest** et **Production**.

1. Sous l’onglet **Pipeline**, sélectionnez le rectangle **Ajouter un artefact**, puis sélectionnez **eshoponweb-ci** dans le champ **Source (pipeline de build).** Cliquez sur **Ajouter** pour confirmer la sélection de l’artefact.
1. Dans le rectangle **Artefacts**, notez le **Déclencheur de déploiement continu** (éclair). Cliquez dessus pour ouvrir les paramètres du **Déclencheur de déploiement continu**. Cliquez sur **Désactivé** pour basculer le commutateur et l'activer. Conservez tous les autres paramètres par défaut et fermez le volet **Déclencheur de déploiement continu** en cliquant sur la marque **x** dans son coin supérieur droit.
1. Dans l’étape **Environnements DevTest**, cliquez sur l'étiquette **1 travail, 1 tâche** et passez en revue les tâches de cette étape.

   > **Remarque** : l’environnement DevTest a 1 tâche qui, respectivement, publie le package d’artefacts sur Azure Web App.

1. Dans le volet **Tous les pipelines > eshoponweb-cd**, vérifiez que l’étape **DevTest** est sélectionnée. Dans la liste déroulante **Abonnement Azure**, sélectionnez votre abonnement Azure et cliquez sur **Autoriser**. Si vous y êtes invité, authentifiez-vous à l’aide du compte d’utilisateur avec le rôle Propriétaire dans l’abonnement Azure.
1. Vérifiez que le type d’application est défini sur Application web sur Windows. Ensuite, dans la liste déroulante **Nom d’App Service**, sélectionnez le nom de l’application web **DevTest**.
1. Sélectionnez la tâche **Déployer Azure App Service**. Dans le champ **Package ou dossier**, mettez à jour la valeur par défaut de « $(System.DefaultWorkingDirectory)//\*\*\*.zip » à « $(System.DefaultWorkingDirectory)/\*\*/Web.zip »

   > **Remarque** : notez le point d’exclamation en regard de l’onglet Tâches. Cela est attendu, car nous devons configurer les paramètres de l’étape de production.

1. Ouvrez le volet **Paramètres d’application et de configuration** et entrez `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development` dans la zone **Paramètres de l’application**.

1. Dans le volet **Tous les pipelines > eshoponweb-cd**, accédez à l’onglet **Pipeline** et, cette fois, dans l'étape **Production**, cliquez sur l’étiquette **1 travail, 1 tâche**. À l’instar de l’étape précédente DevTest, complétez les paramètres du pipeline. Sous l’onglet Tâches / processus Déploiement en production, dans la liste déroulante **Abonnement Azure**, sélectionnez l’abonnement Azure que vous avez utilisé pour l'étape **Environnement DevTest**, affiché sous **Connexions de service Azure disponibles**, puisque nous avons déjà créé la connexion de service lors de l’autorisation de l’abonnement.
1. Dans la liste déroulante **Nom d’App Service**, sélectionnez le nom de l’application web **Prod**.
1. Sélectionnez la tâche **Déployer Azure App Service**. Dans le champ **Package ou dossier**, mettez à jour la valeur par défaut de « $(System.DefaultWorkingDirectory)//\*\*\*.zip » à « $(System.DefaultWorkingDirectory)/\*\*/Web.zip »
1. Ouvrez le volet **Paramètres d’application et de configuration** et entrez `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development` dans la zone **Paramètres de l’application**.
1. Dans le volet **Tous les pipelines > eshoponweb-cd**, cliquez sur **Enregistrer** et, dans la boîte de dialogue **Enregistrer**, cliquez sur **OK**.

   Vous avez maintenant correctement configuré le pipeline de mise en production.

1. Dans la fenêtre du navigateur affichant le projet **eShopOnWeb**, dans le volet de navigation vertical, dans la section **Pipelines**, cliquez sur **Pipelines**.
1. Dans le volet **Pipelines**, cliquez sur l’entrée représentant le pipeline de build **eshoponweb-ci**, puis cliquez sur **Exécuter le pipeline**.
1. Dans le volet **Exécuter le pipeline**, acceptez les paramètres par défaut, puis cliquez sur **Exécuter** pour déclencher le pipeline. **Attendez la fin de l’exécution du pipeline**.

   > **Remarque** : une fois le build généré, la mise en production sera déclenchée automatiquement et l’application sera déployée dans les deux environnements. Validez les actions de mise en production, une fois le pipeline de build terminé.

1. Dans le volet de navigation vertical, dans la section **Pipelines**, cliquez sur **Mises en production** et, dans le volet **eshoponweb-cd**, cliquez sur l’entrée représentant la mise en production la plus récente.
1. Dans le panneau **eshoponweb-cd > Release-1**, suivez la progression de la mise en production et vérifiez que le déploiement sur les deux applications web s’est terminé correctement.
1. Basculez vers l’interface du portail Azure, puis accédez au groupe de ressources **az400m03l08-RG**. Dans la liste des ressources, cliquez sur l’application web **DevTest**. Dans le panneau de l’application web, cliquez sur **Parcourir** et vérifiez que la page web (site web e-commerce) se charge correctement dans un nouvel onglet de navigateur web.
1. Revenez à l’interface du portail Azure, cette fois en accédant au groupe de ressources **az400m03l08-RG**. Dans la liste des ressources, cliquez sur l’application web **Production**. Dans le panneau de l’application web, cliquez sur **Parcourir** et vérifiez que la page web se charge correctement dans un nouvel onglet de navigateur web.
1. Fermez l’onglet du navigateur web affichant le site web **EShopOnWeb**.

   > **Remarque** : vous disposez maintenant de l’application avec CI/CD configuré. Dans l’exercice suivant, nous allons configurer des murs qualité dans le cadre d’un pipeline de mise en production plus avancé.

### Exercice 3 : configurer les portes de mise en production

Dans cet exercice, vous allez configurer des murs qualité dans le pipeline de mise en production.

#### Tâche 1 : configurer des portes de prédéploiement pour les approbations

Dans cette tâche, vous allez configurer des portes de prédéploiement.

1. Basculez vers la fenêtre du navigateur web qui affiche le portail Azure DevOps, puis ouvrez le projet **eShopOnWeb**. Dans le volet de navigation vertical, dans la section **Pipelines**, cliquez sur **Mises en production** puis, dans le volet **eshoponweb-cd**, cliquez sur **Modifier**.
1. Dans le volet **Tous les pipelines > eshoponweb-cd**, sur le bord gauche du rectangle représentant l’étape **Environnement DevTest**, cliquez sur la forme ovale représentant les **Conditions de prédéploiement**.
1. Dans le volet **Conditions de prédéploiement**, définissez le curseur **Approbations de prédéploiement** sur **Activé** puis, dans la zone de texte **Approbateurs**, saisissez e nom de votre compte Azure DevOps et sélectionnez-le.

   > **Remarque** : dans un scénario réel, il doit s’agir d’un alias de nom d’équipe DevOps au lieu de votre propre nom.

1. **Enregistrez** les paramètres d’approbation préalable et fermez la fenêtre contextuelle.
1. Cliquez sur **Créer une mise en production** et confirmez en appuyant sur le bouton **Créer** dans la fenêtre contextuelle.
1. Notez le message de confirmation vert indiquant que « Release-2 » a été créé. Cliquez sur le lien « Release-2 » pour accéder à ses détails.
1. Notez que l’étape **DevTest** est dans un état **Approbation en attente**. Cliquez sur le bouton **Approuver**. Cela relance l’étape DevTest.

#### Tâche 2 : configurer des portes de post-déploiement pour Azure Monitor

Dans cette tâche, vous allez activer le mur de post-déploiement pour l’environnement DevTest.

1. De retour dans le volet **Tous les pipelines > eshoponweb-cd**, sur le bord droit du rectangle représentant l’étape **Environnement DevTest**, cliquez sur la forme ovale représentant les **Conditions post-déploiement**.
1. Dans le volet **Conditions post-déploiement**, définissez le curseur **Portes** sur **Activé**, cliquez sur **+ Ajouter** puis, dans le menu contextuel, cliquez sur **Interroger les alertes Azure Monitor**.
1. Dans le volet **Conditions post-déploiement**, dans la section **Interroger les alertes Azure Monitor**, dans la liste déroulante **Abonnement Azure**, sélectionnez l’entrée de **connexion de service** représentant la connexion à votre abonnement Azure puis, dans la liste déroulante **Groupe de ressources**, sélectionnez l’entrée **az400m03l08-RG**.
1. Dans le volet **Conditions post-déploiement**, développez la section **Avancé** et configurez les options suivantes :

   - Type de filtre : **Aucun**
   - Gravité : **Sev0, Sev1, Sev2, Sev3, Sev4**
   - Intervalle de temps : **Dernière heure**
   - État d’alerte : **Reconnu, Nouveau**
   - Condition d’analyse : **Déclenché**

1. Dans le volet **Conditions post-déploiement**, développez les **Options d’évaluation** et configurez les options suivantes :

   - Définissez la valeur de **Délai entre la réévaluation des portes** sur **5 minutes**.
   - Définissez la valeur de **Délai d’expiration avant l’échec des portes** sur **8 minutes**.
   - Sélectionnez l’option **En cas de réussite des portes, demander les approbations**.

   > **Remarque** : l’intervalle d’échantillonnage et le délai d’expiration fonctionnent ensemble afin que les portes appellent leurs fonctions à intervalles appropriés et rejettent le déploiement s’ils ne réussissent pas pendant le même intervalle d’échantillonnage au cours de la période d’expiration.

1. Fermez le volet **Conditions post-déploiement** en cliquant sur le **x** dans son coin supérieur droit.
1. De retour dans le volet **eshoponweb-cd**, cliquez sur **Enregistrer** puis, dans la boîte de dialogue **Enregistrer**, cliquez sur **OK**.

### Exercice 4 : tester les portes de mise en production

Dans cet exercice, vous allez tester les portes de mise en production en mettant à jour l’application, ce qui déclenchera un déploiement.

#### Tâche 1 : mettre à jour et déployer l’application après l’ajout de portes de mise en production

Dans cette tâche, vous allez d’abord générer des alertes pour l’application web DevTest, puis suivre le processus de mise en production avec les portes de mise en production activées.

1. À partir du portail Azure, accédez à la ressource **Application web DevTest** déployée précédemment.
1. Dans le volet Vue d’ensemble, notez le champ **URL** montrant le lien hypertexte de l’application web. Cliquez sur ce lien qui vous redirige vers l’application web eShopOnWeb dans le navigateur.
1. Pour simuler une **demande ayant échoué**, ajoutez **/discount** à l’URL, ce qui entraînera un message d’erreur, car cette page n’existe pas. Actualisez cette page plusieurs fois pour générer plusieurs événements.
1. Dans le portail Azure, dans le champ « Rechercher des ressources, des services et des documents », entrez **`Application Insights`** et sélectionnez la ressource **DevTest-AppInsights** créée dans l’exercice précédent. Accédez ensuite aux **Alertes**.
1. Il doit y avoir au moins **1** nouvelle alerte dans la liste des résultats, avec une **Gravité 2**. Entrez **`Alerts`** pour ouvrir le service Alertes d’Azure Monitor.
1. Notez qu’au moins **1** Failed_Alert avec **gravité 2 - Avertissement** doit être affichée dans la liste. Celle-ci a été déclenchée lorsque vous avez validé l’adresse URL du site web inexistante dans l’exercice précédent.

   > **Remarque :** si aucune alerte n’apparaît encore, attendez quelques minutes.

1. Revenez dans le portail Azure DevOps et ouvrez le projet **eShopOnWeb**. Accédez à **Pipelines**, sélectionnez **Mises en productions** et **eshoponweb-cd**.
1. Cliquez sur le bouton **Créer une mise en production**.
1. Attendez que le pipeline de mise en production démarre, puis **approuvez** l’action de mise en production de l’étape DevTest.
1. Attendez que l’étape de mise en production DevTest se termine correctement. Vous remarquerez que les **Portes de post-déploiement** passent à l’état **Portes d’évaluation**. Cliquez sur l’icône **Portes d’évaluation**.
1. Pour **Interroger les alertes Azure Monitor**, notez l’état d’échec initial.
1. Laissez le pipeline de mise en production en attente pendant les 5 minutes qui suivent. Une fois les 5 minutes écoulées, notez que la 2e évaluation échoue à nouveau.
1. Ce comportement est attendu, car une alerte Application Insights a été déclenchée pour l’application web DevTest.

   > **Remarque** : étant donné qu’une alerte est déclenchée par l’exception, la porte **Interroger Azure Monitor** échoue. Cela empêche à son tour le déploiement dans l’environnement de **production**.

1. Patientez quelques minutes et validez à nouveau l’état des portes de mise en production. Quelques minutes après la vérification initiale des portes de mise en production, et puisque l’alerte Application Insight initiale a été déclenchée avec l’action « Déclenché », une porte de mise en production doit aboutir, permettant ainsi de déployer l’étape de mise en production.

   > **Remarque :** si votre porte échoue, fermez l’alerte.

   > [!IMPORTANT]
   > N’oubliez pas de supprimer les ressources créées dans le portail Azure pour éviter les frais inutiles.

## Révision

Dans ce labo, vous avez configuré des pipelines de mise en production, puis configuré et testé des portes de mise en production.
