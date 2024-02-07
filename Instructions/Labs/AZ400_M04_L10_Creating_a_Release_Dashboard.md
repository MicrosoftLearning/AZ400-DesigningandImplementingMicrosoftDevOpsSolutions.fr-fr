---
lab:
  title: Création d’un tableau de bord de mise en production
  module: 'Module 04: Design and implement a release strategy'
---

# Création d’un tableau de bord de mise en production

# Manuel de labo pour l’étudiant

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Créer une organisation ou une collection de projets](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops).

- Identifier un abonnement Azure existant ou en créer un.

- Vérifier que vous disposez d’un compte Microsoft ou d’un compte Azure AD avec le rôle Propriétaire dans l’abonnement Azure. Pour plus d’informations, consultez [Répertorier les attributions de rôles Azure à l’aide du portail Azure](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal).

## Présentation du labo

Dans ce laboratoire, vous allez étudier la création d’un tableau de bord de mise en production et l’utilisation de l’API REST pour récupérer les données de mise en production Azure DevOps, que vous pouvez rendre disponibles pour vos applications ou tableaux de bord personnalisés.

Le laboratoire utilise la ressource Azure DevOps Starter, qui crée automatiquement un projet Azure DevOps qui génère et déploie une application dans Azure.

## Objectifs

À la fin de ce labo, vous serez en mesure d’effectuer les tâches suivantes :

- Créez un tableau de bord de mise en production.
- Utilisez l’API REST pour interroger les informations de mise en production.

## Durée estimée : 45 minutes

## Instructions

### Exercice 1 : créer un tableau de bord de mise en production

Dans cet exercice, vous allez créer un tableau de bord de mise en production dans une organisation Azure DevOps.

#### Tâche 1 : créer une ressource Azure DevOps Starter

Dans cette tâche, vous allez créer une ressource Azure DevOps Starter dans votre abonnement Azure. Cette opération crée automatiquement un projet correspondant dans votre organisation Azure DevOps.

1. Sur votre ordinateur de labo, démarrez un navigateur web, accédez au [**portail Azure**](https://portal.azure.com), puis connectez-vous avec le compte d’utilisateur disposant du rôle Propriétaire ou Contributeur dans l’abonnement Azure que vous utiliserez dans ce labo.
1. Dans le portail Azure, recherchez et sélectionnez le type de ressource **DevOps Starter**, puis, dans le panneau **DevOps Starter**, cliquez sur **+ Créer**.
1. Dans le panneau **DevOps Starter**, dans le volet **Démarrer avec une nouvelle application**, sélectionnez la vignette **.NET**, puis, en haut, en regard de **Configurer DevOps Starter avec GitHub**, modifiez les paramètres, cliquez **ici**, puis sélectionnez **Azure DevOps**, **Terminé** et **Suivant : Cadre >**.
1. Dans le panneau **DevOps Starter**, dans le volet **Choisir un framework d’application**, sélectionnez la vignette **ASP<nolink>.NET Core**, déplacez le curseur **Ajouter une base de données** à la position **Activé**, puis cliquez sur **Suivant : Service >**.
1. Dans le panneau **DevOps Starter**, dans le volet **Sélectionner un service Azure pour déployer l’application**, vérifiez que la vignette **Application web Windows** est sélectionnée, puis cliquez sur **Suivant : Créer >**.
1. Dans le panneau **DevOps Starter**, dans le volet **C’est bientôt fini**, spécifiez les paramètres suivants :

    | Paramètre | Value |
    | ------- | ----- |
    | Nom du projet | **Création d’un tableau de bord de mise en production** |
    | Organisation Azure DevOps | Nom de l’organisation Azure DevOps que vous allez utiliser dans ce labo |
    | Abonnement | le nom de l’abonnement Azure que vous utilisez dans ce labo |
    | Nom de l’application web | Chaîne globalement unique comprenant entre 2 et 60 caractères, composée de lettres, de chiffres et de tirets, et commençant et se terminant par une lettre ou un chiffre. |
    | Emplacement | Nom de la région Azure dans laquelle vous allez déployer une application web Azure et une base de données Azure SQL |

1. Dans le panneau **DevOps Starter**, dans le volet **C’est bientôt fini**, cliquez sur **Paramètres supplémentaires**.
1. Dans le volet **Paramètres supplémentaires**, spécifiez les paramètres suivants, puis cliquez sur **OK**.

    | Paramètre | Valeur |
    | ------- | ----- |
    | Groupe de ressources | **az400m10l02-rg** |
    | Niveau tarifaire | **F1 Gratuit** |
    | Emplacement d’Application Insights | Nom de la même région Azure que celle choisie pour l’emplacement de l’application web Azure |
    | Nom du serveur | Entrez une chaîne globalement unique comprenant entre 3 et 63 caractères, composée de lettres, de chiffres et de tirets, et commençant et se terminant par une lettre ou un chiffre |
    | Entrer le nom d’utilisateur | **dbadmin** |
    | Emplacement | Nom de la même région Azure que celle choisie pour l’emplacement de l’application web Azure |
    | Nom de la base de données | **az400m10l02-db** |

1. Dans le panneau **DevOps Starter**, dans le volet **C’est bientôt fini**, cliquez sur **Terminé**, puis sur **Vérifier + créer**.

    > **Remarque** : Attendez la fin du déploiement. L’approvisionnement de la ressource **DevOps Starter** doit prendre environ 2 minutes.

1. Après avoir reçu la confirmation de l’approvisionnement de la ressource DevOps Starter, cliquez sur le bouton **Accéder à la ressource**. Le navigateur est alors redirigé vers le panneau DevOps Starter.
1. Dans le panneau DevOps Starter, suivez la progression du pipeline CI/CD jusqu’à ce qu’il se termine.

    > **Remarque** : la création de l’application web Azure correspondante et de la base de données Azure SQL peut prendre environ 5 minutes. Le processus crée automatiquement un projet Azure DevOps qui inclut un référentiel prêt à déployer, ainsi que les pipelines de build et de mise en production. Les ressources Azure sont créées dans le cadre du pipeline de déploiement déclenché automatiquement.

#### Tâche 2 : créer des mises en production Azure DevOps

Dans cette tâche, vous allez créer plusieurs mises en production Azure DevOps, y compris une qui entraînera un échec de déploiement.

1. Dans le navigateur web affichant le portail Azure, sur la page DevOps Starter, dans la barre d’outils, cliquez sur **Page d’accueil du projet**. Cette opération ouvre automatiquement un autre onglet de navigateur qui affiche le projet **Création d’un tableau de bord de mise en production** dans le portail Azure Devops. Si vous êtes invité à vous connecter, authentifiez-vous avec les informations d’identification de votre organisation Azure DevOps.

    > **Remarque** : vous allez commencer par créer une mise en production qui sera déployée avec succès.

1. Dans le portail Azure DevOps, dans le menu vertical situé à gauche, cliquez sur **Dépôts**, dans la liste des dossiers du référentiel, accédez au dossier **Application\\aspnet-core-dotnet-core\\Pages**, puis cliquez sur l’entrée **Index.cshtml**.
1. Dans le volet **Index.cshtml**, cliquez sur **Modifier**, à la ligne **20**, remplacez `<div class="description line-2"> Your ASP.NET Core app is up and running on Azure</div>` par `<div class="description line-2"> Your ASP.NET Core app v1.1 is up and running on Azure</div>`, cliquez sur **Valider**, puis, dans le volet **Validation**, cliquez à nouveau sur **Valider**. Le pipeline de build sera automatiquement déclenché.
1. Dans le portail Azure DevOps, dans le volet de navigation vertical situé à gauche, cliquez sur **Pipelines**.
1. Sous l’onglet **Récent** du volet **Pipelines**, cliquez sur l’entrée **az400m10l02-CI**, sous l’onglet **Exécutions** du volet **az400m10l02-CI**, sélectionnez l’exécution la plus récente, sous l’onglet **Résumé** de l’exécution, dans la section **Travaux**, cliquez sur **Générer**, puis surveillez le travail jusqu’à sa fin.
1. Une fois le travail terminé, dans le portail Azure DevOps, dans le volet de navigation vertical situé à gauche, dans la section **Pipelines**, cliquez sur **Mises en production**.
1. Dans le volet **az400m10l02 - CD**, sous l’onglet **Mises en production**, cliquez sur l’entrée **Release-2**, sous l’onglet **Pipeline** du volet **Release-2**, cliquez sur la phase **dev**, dans le volet **dev**, cliquez sur **Afficher les journaux**, puis surveillez la progression du déploiement jusqu’à sa fin.

    > **Remarque** : vous allez à présent créer une mise en production dont le déploiement va échouer. L’échec est dû au test d’assemblys intégrés, qui considère que la modification associée à la nouvelle mise en production n’est pas valide.

1. Dans le portail Azure DevOps, dans le menu vertical situé à gauche, cliquez sur **Dépôts**, dans la liste des dossiers du référentiel, accédez au dossier **Application\\aspnet-core-dotnet-core\\Pages**, puis cliquez sur l’entrée **Index.cshtml**.
1. Dans le volet **Index.cshtml**, cliquez sur **Modifier**. À la ligne **4**, remplacez `ViewData["Title"] = "Home Page - ASP.NET Core";` par `ViewData["Title"] = "Home Page v1.2 - ASP.NET Core";`. Cliquez sur **Valider**. Dans le volet **Validation**, cliquez à nouveau sur **Valider**. Le pipeline de build sera automatiquement déclenché.
1. Dans le portail Azure DevOps, dans le volet de navigation vertical situé à gauche, cliquez sur **Pipelines**.
1. Sous l’onglet **Récent** du volet **Pipelines**, cliquez sur l’entrée **az400m10l02-CI**, sous l’onglet **Exécutions** du volet **az400m10l02-CI**, sélectionnez l’exécution la plus récente, sous l’onglet **Résumé** de l’exécution, dans la section **Travaux**, cliquez sur **Générer**, puis surveillez le travail jusqu’à sa fin.
1. Une fois le travail terminé, dans le portail Azure DevOps, dans le volet de navigation vertical situé à gauche, dans la section **Pipelines**, cliquez sur **Mises en production**.
1. Dans le volet **az400m10l02 - CD**, sous l’onglet **Mises en production**, cliquez sur l’entrée **Release-3**, sous l’onglet **Pipeline** du volet **Release-3**, cliquez sur la phase **dev**, dans le volet **dev**, cliquez sur **Afficher les journaux**, puis surveillez la progression du déploiement jusqu’à son échec pendant la phase **Assemblys de tests**.

#### Tâche 3 : créer un tableau de bord de mise en production Azure DevOps

Dans cette tâche, vous allez créer un tableau de bord et y ajouter des widgets liés à la mise en production.

1. Dans le portail Azure DevOps, dans le menu vertical situé à gauche, cliquez sur **Vue d’ensemble**, dans la section **Vue d’ensemble**, cliquez sur **Tableaux de bord**, puis cliquez sur **Ajouter un widget**.
1. Dans le volet **Ajouter un widget**, faites défiler la liste des widgets vers le bas, sélectionnez l’entrée **État du déploiement**, puis cliquez sur **Ajouter**.
1. Utilisez la procédure décrite à l’étape précédente pour ajouter les widgets **Release Health Details**, **Release Health Overview** et **Release Pipeline Overview**.
    > **Remarque** : installez les widgets **Release Health Details** et **Release Health Overview** à partir de la Place de marché [Team Project Health](https://marketplace.visualstudio.com/items?itemName=ms-devlabs.TeamProjectHealth).
1. Utilisez la souris pour faire glisser le widget **Release Pipeline Overview** à droite du widget **État du déploiement** pour éviter la nécessité de faire défiler verticalement le tableau de bord, puis cliquez sur **Modification terminée**.
1. Dans le volet du tableau de bord, dans le rectangle représentant le widget **État du déploiement**, cliquez sur **Configurer le widget**.
1. Dans le volet **Configuration**, spécifiez les paramètres suivants (en laissant tous les autres paramètres avec leur valeur par défaut), puis cliquez sur **Enregistrer**.

    | Paramètre | Valeur |
    | ------- | ----- |
    | Pipeline de build | **az400m10l02 - CI** |
    | Pipelines de mise en production liés | **az400m10l02 - CD ; az400m10l02 - CD\dev** |

1. Dans le volet du tableau de bord, pointez sur le coin supérieur droit du rectangle représentant le widget **Release Health Overview** pour afficher les points de suspension représentant le menu **Autres actions**, cliquez dessus, puis, dans le menu déroulant, cliquez sur **Configurer**.  
1. Dans le volet **Configuration**, spécifiez les paramètres suivants (en laissant tous les autres paramètres avec leur valeur par défaut), puis cliquez sur **Enregistrer**.

    | Paramètre | Valeur |
    | ------- | ----- |
    | Sélectionnez une ou plusieurs définitions de mise en production | **az400m10l02 - CD** |

1. Dans le volet du tableau de bord, pointez sur le coin supérieur droit du rectangle représentant le widget **Release Health Details** pour afficher les points de suspension représentant le menu **Autres actions**, cliquez dessus, puis, dans le menu déroulant, cliquez sur **Configurer**.  
1. Dans le volet **Configuration**, spécifiez les paramètres suivants (en laissant tous les autres paramètres avec leur valeur par défaut), puis cliquez sur **Enregistrer**.

    | Paramètre | Valeur |
    | ------- | ----- |
    | Définition | **az400m10l02 - CD** |

1. Dans le volet du tableau de bord, pointez sur le coin supérieur droit du rectangle représentant le widget **Release Pipeline Overview** pour afficher les points de suspension représentant le menu **Autres actions**, cliquez dessus, puis, dans le menu déroulant, cliquez sur **Configurer**.  
1. Dans le volet **Configuration**, spécifiez les paramètres suivants (en laissant tous les autres paramètres avec leur valeur par défaut), puis cliquez sur **Enregistrer**.

    | Paramètre | Valeur |
    | ------- | ----- |
    | Pipeline de mise en production | **az400m10l02 - CD** |

1. Dans le volet du tableau de bord, cliquez sur **Actualiser** pour mettre à jour le contenu affiché par les widgets.

    > **Remarque** : les liens sur les widgets vous permettent d’accéder directement aux volets correspondants dans le portail Azure DevOps.

### Exercice 2 : interroger les informations de mise en production via l’API REST

Dans cet exercice, vous allez interroger les informations de mise en production via l’API REST à l’aide de Postman.

#### Tâche 1 : générer un jeton d’accès personnel Azure DevOps

Dans cette tâche, vous allez générer un jeton d’accès personnel Azure DevOps qui sera utilisé pour vous authentifier à partir de l’application Postman que vous allez installer dans la prochaine tâche de cet exercice.

1. Sur l’ordinateur de labo, dans la fenêtre du navigateur web affichant le portail Azure DevOps, dans le coin supérieur droit de la page Azure DevOps, cliquez sur l’icône **Paramètres utilisateur**. Dans le menu déroulant, cliquez sur **Jetons d’accès personnels**, puis, dans le volet **Jetons d’accès personnels**, cliquez sur **+ Nouveau jeton**.
1. Dans le volet **Créer un jeton d’accès personnel**, cliquez sur le lien **Afficher toutes les étendues**, spécifiez les paramètres suivants, puis cliquez sur **Créer** (conservez la valeur par défaut de tous les autres paramètres) :

    | Paramètre | Valeur |
    | --- | --- |
    | Nom | **Labo Création d’un tableau de bord de mise en production** |
    | Étendue | **Version release** |
    | Autorisations | **Lire** |
    | Étendue | **Créer** |
    | Autorisations | **Lire** |

1. Dans le volet **Réussite**, copiez la valeur du jeton d’accès personnel dans le Presse-papiers.

    > **Remarque** : assurez-vous d’enregistrer la valeur du jeton. Vous ne pourrez plus la récupérer après avoir fermé ce volet.

1. Dans le volet **Réussite**, cliquez sur **Fermer**.

#### Tâche 2 : interroger les informations de mise en production via l’API REST à l’aide de Postman

Dans cette tâche, vous allez interroger les informations de mise en production via l’API REST à l’aide de Postman.

1. Sur l’ordinateur de labo, démarrez un navigateur web et accédez à [la page de téléchargement de Postman](https://www.postman.com/downloads/), cliquez sur le bouton **Télécharger l’application**, dans le menu déroulant, cliquez sur **Windows 64 bits**, cliquez sur le fichier téléchargé, puis exécutez l’installation. Une fois l’installation terminée, l’application de bureau Postman démarre automatiquement.
1. Dans le volet **Créer un compte Postman**, indiquez votre adresse e-mail, un nom d’utilisateur et un mot de passe, puis cliquez sur **Créer un compte gratuit**.

    > **Remarque** : vous recevrez un message électronique de Postman pour activer votre compte Postman et terminer le processus d’approvisionnement de compte.

1. Une fois que vous êtes connecté, dans la fenêtre de l’application de bureau Postman, dans le coin supérieur gauche, cliquez sur **+ Nouveau**, dans le volet **Créer**, cliquez sur **Requête**, dans le volet **ENREGISTRER LA REQUÊTE**, dans la zone de texte **Nom de la requête**, tapez **Get-Releases**, cliquez sur **+ Créer une collection**, dans la zone de texte **Nommer votre collection**, tapez **Requêtes Azure DevOps az400m10l02**, cliquez sur la coche située à droite, puis cliquez sur le bouton **Enregistrer dans Requêtes Azure DevOps az400m10l02**.
1. Ouvrez une autre fenêtre de navigateur web, accédez à [la page Microsoft Docs **Mises en production - Liste**](https://docs.microsoft.com/en-us/rest/api/azure/devops/release/releases/list?view=azure-devops-rest-6.0), puis passez en revue son contenu.
1. Revenez à l’application de bureau Postman, dans le volet Launchpad dans la section supérieure droite de la fenêtre de l’application, cliquez sur l’en-tête de l’onglet **Autorisation**, dans la liste déroulante **TYPE**, sélectionnez l’entrée **Authentification de base**, puis, dans la zone de texte **Mot de passe**, collez la valeur du jeton que vous avez copiée dans la tâche précédente (ne définissez pas la valeur de la zone de texte **Nom d’utilisateur**).
1. Dans le volet Launchpad dans la section supérieure droite de la fenêtre de l’application, vérifiez que **GET** apparaît dans la liste déroulante, dans la zone de texte **Entrer l’URL de la demande**, tapez ce qui suit, puis cliquez sur **Envoyer** (remplacez la valeur de `<organization_name>` par le nom de votre organisation Azure DevOps) afin d’afficher la liste de toutes les mises en production :

    ```url
    https://vsrm.dev.azure.com/<organization_name>/Creating%20a%20Release%20Dashboard/_apis/release/releases?api-version=6.0
    ```

1. Passez en revue la sortie affichée sous l’onglet **Corps** dans la section inférieure droite de la fenêtre de l’application et vérifiez qu’elle contient la liste des mises en production que vous avez créées dans l’exercice précédent de ce labo.
1. Basculez vers la fenêtre du navigateur web affichant le contenu de Microsoft Docs, accédez à [la page Microsoft Docs **Déploiements - Liste**](https://docs.microsoft.com/en-us/rest/api/azure/devops/release/deployments/list?view=azure-devops-rest-6.0), puis passez en revue son contenu.
1. Dans le volet Launchpad dans la section supérieure droite de la fenêtre de l’application, vérifiez que **GET** apparaît dans la liste déroulante, dans la zone de texte **Entrer l’URL de la demande**, tapez ce qui suit, puis cliquez sur **Envoyer** (remplacez la valeur de `<organization_name>` par le nom de votre organisation Azure DevOps) afin d’afficher la liste de tous les déploiements :

    ```url
    https://vsrm.dev.azure.com/<organization_name>/Creating%20a%20Release%20Dashboard/_apis/release/deployments?api-version=6.0
    ```

1. Passez en revue la sortie affichée sous l’onglet **Corps** dans la section inférieure droite de la fenêtre de l’application et vérifiez qu’elle contient la liste des déploiements que vous avez lancés dans l’exercice précédent de ce labo.
1. Dans le volet Launchpad dans la section supérieure droite de la fenêtre de l’application, vérifiez que **GET** apparaît dans la liste déroulante, dans la zone de texte **Entrer l’URL de la demande**, tapez ce qui suit, puis cliquez sur **Envoyer** (remplacez la valeur de `<organization_name>` par le nom de votre organisation Azure DevOps) afin d’afficher la liste de tous les déploiements :

    ```url
    https://vsrm.dev.azure.com/<organization_name>/Creating%20a%20Release%20Dashboard/_apis/release/deployments?DeploymentStatus=failed&api-version=6.0
    ```

1. Passez en revue la sortie affichée sous l’onglet **Corps** dans la section inférieure droite de la fenêtre de l’application et vérifiez qu’elle contient uniquement le déploiement ayant échoué que vous avez lancé dans l’exercice précédent de ce labo.

### Exercice 3 : supprimer les ressources du labo Azure

Dans cet exercice, vous allez supprimer les ressources Azure approvisionnées dans ce labo pour éviter des frais inattendus.

>**Remarque** : N’oubliez pas de supprimer toutes les nouvelles ressources Azure que vous n’utilisez plus. La suppression des ressources inutilisées vous évitera d’encourir des frais inattendus.

#### Tâche 1 : supprimer les ressources du labo Azure

Dans cette tâche, vous allez utiliser Azure Cloud Shell pour supprimer les ressources Azure approvisionnées dans ce labo pour éviter des frais inutiles.

1. Dans le Portail Azure, ouvrez la session shell **Bash** dans le volet **Cloud Shell**.
1. Listez tous les groupes de ressources créés dans les labos de ce module en exécutant la commande suivante :

    ```sh
    az group list --query "[?starts_with(name,'az400m10l02-rg')].name" --output tsv
    ```

1. Supprimez tous les groupes de ressources que vous avez créés dans les labos de ce module en exécutant la commande suivante :

    ```sh
    az group list --query "[?starts_with(name,'az400m10l02-rg')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Remarque** : La commande s’exécute de façon asynchrone (comme déterminé par le paramètre --no-wait). Par conséquent, vous serez en mesure d’exécuter une autre commande Azure CLI immédiatement après au cours de la même session Bash, mais la suppression réelle du groupe de ressources prendra quelques minutes.

## Révision

Dans ce labo, vous avez appris à créer et à configurer un tableau de bord de mise en production et à utiliser l’API REST pour récupérer les données de mise en production Azure DevOps.
