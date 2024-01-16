---
lab:
  title: Configuration et exécution de tests fonctionnels
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# Configuration et exécution de tests fonctionnels

## Manuel de labo pour l’étudiant

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility?view=azure-devops).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans [Créer une organisation ou une collection de projets](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifier un abonnement Azure existant ou en créer un.

- Vérifiez que vous avez un compte Microsoft ou un compte Azure AD avec le rôle Propriétaire dans l’abonnement Azure, et le rôle Administrateur général dans le locataire Azure AD associé à l’abonnement Azure. Pour plus d’informations, consultez [Répertorier les attributions de rôle Azure à l’aide du portail Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) et [Afficher et attribuer des rôles d’administrateur dans Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Présentation du labo

[Selenium](http://www.seleniumhq.org/) est un framework de tests de logiciels portable et open source pour les applications web. Il peut fonctionner sur presque tous les systèmes d’exploitation. Il prend en charge tous les navigateurs modernes et plusieurs langages, notamment .NET (C#) et Java.

Dans ce labo, vous allez découvrir comment exécuter des cas de test Selenium sur une application web C# dans le cadre du pipeline de mise en production Azure DevOps.

## Objectifs

À la fin de ce labo, vous serez en mesure d’effectuer les tâches suivantes :

- Configurer un agent Azure DevOps auto-hébergé.
- Configurer le pipeline de mise en production.
- Déclencher la génération et la mise en production.
- Exécuter des tests dans Chrome et Firefox.

## Durée estimée : 60 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

Dans cet exercice, vous allez configurer les prérequis  pour le labo, qui incluent le projet d’équipe Parts Unlimited préconfiguré en fonction d’un modèle Générateur de démos Azure DevOps et de ressources Azure.

#### Tâche 1 : configurer le projet d’équipe

Dans cette tâche, vous allez utiliser le Générateur de démos Azure DevOps pour générer un nouveau projet basé sur le modèle **Selenium**.

1. Sur votre ordinateur de labo, démarrez un navigateur web, puis accédez à [Générateur de démos Azure DevOps](https://azuredevopsdemogenerator.azurewebsites.net). Ce site utilitaire automatise le processus de création d’un projet Azure DevOps au sein de votre compte prérempli de contenu (éléments de travail, référentiels, etc.) requis pour le labo.

    > **Remarque** : pour plus d’informations sur le site, consultez l’article [Qu’est-ce que le Générateur de démos Azure DevOps Services ?](https://docs.microsoft.com/azure/devops/demo-gen).

2. Cliquez sur **Connexion** et connectez-vous avec le compte Microsoft associé à votre abonnement Azure DevOps.
3. Si nécessaire, sur la page **Générateur de démos Azure DevOps**, cliquez sur **Accepter** pour accepter les demandes d’autorisation d’accès à votre abonnement Azure DevOps.
4. Sur la page **Créer un projet**, dans la zone de texte **Nouveau nom du projet**, tapez **Configuration et exécution de tests fonctionnels**. Dans la liste déroulante **Sélectionner une organisation**, sélectionnez votre organisation Azure DevOps, puis cliquez sur **Choisir un modèle**.
5. Dans la liste des modèles, dans la barre d’outils, cliquez sur **DevOps Labs**, sélectionnez le modèle **Selenium**, puis cliquez sur **Sélectionner un modèle**.
6. Sur la page **Créer un projet**, cliquez sur **Créer un projet**.

    > **Remarque** : attendez la fin du processus. Ce processus prend environ 2 minutes. Si le processus échoue, accédez à votre organisation DevOps, supprimez le projet, puis réessayez.

7. Sur la page **Créer un projet**, cliquez sur **Accéder au projet**.

#### Tâche 2 : Créez des ressources Azure

Dans cette tâche, vous allez approvisionner une machine virtuelle Azure exécutant Windows Server 2016 avec SQL Express 2017, Chrome et Firefox.

1. Cliquez ici dans le lien **[Déployer sur Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2Falmvm%2Fmaster%2Flabs%2Fvstsextend%2Fselenium%2Farmtemplate%2Fazuredeploy.json)**. Vous êtes alors redirigé automatiquement vers le panneau **Déploiement personnalisé** dans le portail Azure.
2. Si vous y êtes invité, connectez-vous avec le compte d’utilisateur doté du rôle Propriétaire dans l’abonnement Azure que vous allez utiliser dans ce labo et du rôle Administrateur général dans le locataire Azure AD associé à cet abonnement.
3. Dans le panneau **Déploiement personnalisé**, sélectionnez **Modifier le modèle**.
4. Dans le panneau **Modifier le modèle**, recherchez la ligne `"https://raw.githubusercontent.com/microsoft/azuredevopslabs/master/labs/vstsextend/selenium/armtemplate/chrome_firefox_VSTSagent_IIS.ps1"`, remplacez-la par `"https://raw.githubusercontent.com/MicrosoftLearning/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/master/Allfiles/Labs/11b/chrome_firefox_VSTSagent_IIS.ps1"`, puis cliquez sur **Enregistrer**.
5. Dans le volet **Déploiement personnalisé**, spécifiez les paramètres suivants :

    | Paramètre | Valeur |
    | --- | --- |
    | Abonnement | le nom de l’abonnement Azure que vous utilisez dans ce labo |
    | Resource group | Nom d’un nouveau groupe de ressources **az400m11l02-RG** |
    | Région | Nom de la région Azure dans laquelle vous souhaitez déployer les ressources Azure dans ce labo |
    | Nom de l'ordinateur virtuel | **az40011bvm** |

6. Cliquez sur **Vérifier + créer**, puis sur **Créer**.

    > **Remarque** : attendez la fin du processus. Cette opération prend environ 15 minutes.

### Exercice 1 : implémenter des tests Selenium à l’aide d’un agent Azure DevOps autohébergé

Dans cet exercice, vous allez implémenter des tests Selenium à l’aide d’un agent Azure DevOps autohébergé.

#### Tâche 1 : configurer un agent Azure DevOps autohébergé

Dans cette tâche, vous allez configurer un agent autohébergé à l’aide de la machine virtuelle que vous avez déployée dans l’exercice précédent. Selenium exige que l’agent soit exécuté en mode interactif pour exécuter les tests de l’interface utilisateur.

1. Dans la fenêtre du navigateur web qui affiche le portail Azure, recherchez et sélectionnez **Machines virtuelles**, puis, dans le panneau **Machines virtuelles**, sélectionnez **az40011bvm**.
2. Dans le panneau **az40011bvm**, sélectionnez **Connecter**. Dans le menu déroulant, sélectionnez **RDP**. Sous l’onglet **RDP** du panneau **az40011bvm \| Connecter**, sélectionnez **Télécharger le fichier RDP**, puis ouvrez le fichier téléchargé.
3. À l’invite, connectez-vous avec les informations d’identification suivantes :

    | Paramètre | Valeur |
    | --- | --- |
    | Nom d’utilisateur | **vmadmin** |
    | Mot de passe | **P2ssw0rd@123** |

4. Dans la session Bureau à distance sur **az40011bvm**, ouvrez une fenêtre de navigateur web Chrome, accédez à **<https://dev.azure.com>**, puis connectez-vous à votre organisation Azure DevOps.
5. Dans le coin inférieur gauche du portail **Azure DevOps**, cliquez sur **Paramètres de l’organisation**.
6. Dans le menu vertical situé à gauche de la page, dans la section **Pipelines**, cliquez sur **Pools d’agents**.
7. Dans le volet **Pools d’agents**, cliquez sur **Par défaut**.
8. Dans le volet **Par défaut**, cliquez sur **Nouvel agent**.
9. Dans le panneau **Obtenir l’agent**, vérifiez que l’onglet **Windows** et la section **x64** sont sélectionnés, puis cliquez sur **Télécharger**.
10. Démarrez l’Explorateur de fichiers, créez un répertoire **C:\\AzAgent**, puis extrayez-y le contenu du fichier zip de l’agent téléchargé résidant dans le dossier **Downloads**.
11. Dans la session Bureau à distance sur **az40011bvm**, cliquez avec le bouton droit sur le menu **Démarrer**, puis cliquez sur **Invite de commandes (admin)**.
12. Dans la fenêtre **Administrateur : invite de commandes**, exécutez la commande suivante pour démarrer l’installation des fichiers binaires de l’agent :

    ```cmd
    cd C:\AzAgent
    Config.cmd
    ```

13. Dans la fenêtre **Administrateur : invite de commandes**, lorsque vous êtes invité à **entrer l’URL du serveur**, tapez **https://dev.azure.com/\<your-DevOps-organization-name\>**, où **\<your-DevOps-organization-name\>** correspond au nom de votre organisation Azure DevOps, puis appuyez sur la touche **Entrée**.
14. Dans la fenêtre **Administrateur : invite de commandes**, lorsque vous êtes invité à **entrer le type d’authentification (appuyez sur Entrée pour PAT)**, appuyez sur la touche **Entrée**.
15. Dans la fenêtre **Administrateur : invite de commandes**, lorsque vous êtes invité à **entrer un jeton d’accès personnel**, basculez vers le portail Azure DevOps, puis fermez le panneau **Obtenir l’agent**. Dans le coin supérieur droit de la page Azure DevOps, cliquez sur l’icône **Paramètres utilisateur**. Dans le menu déroulant, cliquez sur **Jetons d’accès personnels**, puis, dans le volet **Jetons d’accès personnels**, cliquez sur **+ Nouveau jeton**.
16. Dans le volet **Créer un jeton d’accès personnel**, spécifiez les paramètres suivants, puis cliquez sur **Créer** (en conservant les valeurs par défaut de tous les autres paramètres) :

    | Paramètre | Valeur |
    | --- | --- |
    | Nom | **Labo Configuration et exécution de tests fonctionnels** |
    | Étendues | **Définition personnalisée** |
    | Étendues | Cliquez sur **Afficher toutes les étendues** (dans le bas de la fenêtre). |
    | Étendues | **Pools d’agents** - **Lire et gérer** |

17. Dans le volet **Réussite**, copiez la valeur du jeton d’accès personnel dans le Presse-papiers.

    > **Remarque** : veillez à copier le jeton. Vous ne pourrez plus la récupérer après avoir fermé ce volet.

18. Dans le volet **Réussite**, cliquez sur **Fermer**.
19. Revenez à la fenêtre **Administrateur : invite de commandes**, collez le contenu du Presse-papiers, puis appuyez sur la touche **Entrée**.
20. Dans la fenêtre **Administrateur : invite de commandes**, lorsque vous êtes invité à **entrer le pool d’agents (appuyez sur Entrée par défaut)**, appuyez sur la touche **Entrée**.
21. Dans la fenêtre **Administrateur : invite de commandes**, lorsque vous êtes invité à **entrer le nom de l’agent (appuyez sur Entrée pour az40011bvm)**, appuyez sur la touche **Entrée**.
22. Dans la fenêtre **Administrateur : invite de commandes**, lorsque vous êtes invité à **entrer le dossier de travail (appuyez sur Entrée pour _work)**, appuyez sur la touche **Entrée**.
23. Dans la fenêtre **Administrateur : invite de commandes**, lorsque vous êtes invité à **entrer l’agent d’exécution en tant que service (O/N) (appuyez sur Entrée pour N)**, appuyez sur la **touche Entrée**.
24. Dans la fenêtre **Administrateur : invite de commandes**, lorsque vous êtes invité à **configurer l’ouverture de session automatique et exécuter l’agent au démarrage (O/N) (appuyez sur Entrée pour N)**, appuyez sur la **touche Entrée**.
25. Une fois l’agent inscrit, dans la fenêtre **Administrateur : invite de commandes**, tapez **run.cmd**, puis appuyez sur **Entrée** pour démarrer l’agent.

    > **Remarque** : vous devez également installer l’infrastructure Dac Framework qui est utilisée par l’application que vous allez déployer plus tard dans le labo.

26. Dans la session Bureau à distance vers **az40011bvm**, démarrez une autre instance du navigateur web, accédez à la [page de téléchargement de Microsoft SQL Server Data-Tier Application Framework (18.2)](https://www.microsoft.com/download/details.aspx?id=58207&WT.mc_id=rss_alldownloads_extensions), puis cliquez sur **Télécharger**.
27. Dans la zone **Choisissez le téléchargement voulu**, cochez la case **EN\x64\DacFramework.msi**, puis cliquez sur **Suivant**. Vous déclenchez ainsi le téléchargement automatique du fichier **DacFramework.msi**.
28. Une fois le téléchargement du fichier **DacFramework.msi** terminé, utilisez-le pour exécuter l’installation de Microsoft SQL Server Data-Tier Application Framework avec les paramètres par défaut.

#### Tâche 2 : configurer un pipeline de mise en production

Dans cette tâche, vous allez configurer le pipeline de mise en production.

> **Remarque** : la machine virtuelle Azure a configuré l’agent de sorte à déployer les applications et exécuter des cas de test Selenium. La définition de mise en production utilise des **[Phases](https://docs.microsoft.com/vsts/build-release/concepts/process/phases)** pour effectuer le déploiement sur des serveurs cibles.

1. Dans la session Bureau à distance vers **az40011bvm**, dans la fenêtre du navigateur affichant le portail **Azure DevOps**, cliquez sur le symbole **Azure DevOps** dans le coin supérieur gauche.
2. Dans le volet affichant vos projets d’organisation, cliquez sur la vignette représentant le projet **Configuration et exécution de tests fonctionnels**.
3. Dans le volet **Configuration et exécution de tests fonctionnels**, dans le volet de navigation vertical, sélectionnez **Pipelines**. Dans la section **Pipelines**, cliquez sur **Mises en production**, puis, dans le volet **Selenium**, cliquez sur **Modifier**.
4. Dans le volet **Tous les pipelines > Selenium**, cliquez sur l’en-tête de l’onglet **Tâches** et, dans le menu déroulant, cliquez sur **Développement**.
5. Dans la liste des tâches de la phase de **développement**, passez en revue les phases de **déploiement IIS**, de **déploiement SQL** et de **déploiement d’exécution de test Selenium**.

   - **Phase de déploiement IIS** : dans cette phase, nous déployons l’application sur la machine virtuelle à l’aide des tâches suivantes :

     - **Gestion de l’application web IIS** : cette tâche s’exécute sur l’ordinateur cible sur lequel nous avons inscrit l’agent. Cela crée un *site web* et un *pool d’applications* localement avec le nom **PartsUnlimited** s’exécutant sur le port **82**, [**http://localhost:82**](http://localhost:82).
     - **Déploiement de l’application web IIS** : cette tâche déploie l’application sur le serveur IIS à l’aide de **Web Deploy**.

   - **Phase de déploiement de base de données** : dans cette phase, nous utilisons la tâche [**SQL Server Database Deploy**](https://github.com/Microsoft/vsts-tasks/blob/master/Tasks/SqlDacpacDeploymentOnMachineGroup/README.md) pour déployer le fichier [**dacpac**](https://docs.microsoft.com/sql/relational-databases/data-tier-applications/data-tier-applications) sur le serveur de base de données.

   - **Exécution des tests Selenium** : l’exécution de **tests de l’interface utilisateur** dans le cadre du processus de mise en production nous permet de détecter des modifications inattendues. La configuration de tests automatisés basés sur un navigateur permet d’améliorer la qualité de votre application, sans avoir à procéder manuellement. Dans cette phase, nous allons exécuter des tests Selenium sur l’application web déployée. Les tâches suivantes décrivent l’utilisation de Selenium pour tester les sites web dans le pipeline de mise en production.

     - **Programme d’installation de visual Studio Test Platform** : [la tâche Programme d’installation de visual Studio Test Platform](https://docs.microsoft.com/azure/devops/pipelines/tasks/tool/vstest-platform-tool-installer?view=vsts) acquiert la plateforme de test Microsoft à partir de nuget.org ou d’un flux spécifié, puis l’ajoute au cache des outils. Elle répond aux exigences de **vstest**, de sorte que toute tâche de test Visual Studio qui suit dans une build ou un pipeline de mise en production peut s’exécuter sans avoir besoin d’une installation complète de Visual Studio sur la machine de l’agent.
     - **Exécuter des tests de l’interface utilisateur avec Selenium** : cette [tâche](https://github.com/Microsoft/azure-pipelines-tasks/blob/master/Tasks/VsTestV2/README.md) utilise **vstest.console.exe** pour exécuter les cas de test Selenium sur les machines de l’agent.

6. Dans le volet **Tous les pipelines > Selenium**, cliquez sur la phase de **déploiement IIS** et, dans le volet **Tâche de l’agent**, vérifiez que le pool d’agents par **défaut** est sélectionné.
7. Répétez l’étape précédente pour le **déploiement SQL** et les phases d’**exécution des tests Selenium**. Cliquez sur **Enregistrer** pour enregistrer les modifications.

#### Tâche 3 : déclencher la build et la mise en production

Dans cette tâche, nous allons déclencher la **build** pour compiler des scripts Selenium C# avec l’application web. Les fichiers binaires résultants sont copiés vers un agent auto-hébergé et les scripts Selenium sont exécutés dans le cadre de la **version** automatisée.

1. Dans la session Bureau à distance vers **az40011bvm**, dans la fenêtre du navigateur affichant le portail **Azure DevOps**, dans le volet de navigation vertical, section **Pipelines**, cliquez sur **Pipelines**, puis, dans le volet **Pipelines**, cliquez sur **Selenium**.
2. Dans le volet **Selenium**, cliquez sur **Exécuter le pipeline**. Dans le **pipeline d’exécution**, cliquez sur **Exécuter**.

    > **Remarque** : cette build publiera les artefacts de test dans Azure DevOps, qui seront utilisés dans la version.

    > **Remarque** : une fois la génération réussie, une nouvelle mise en production est déclenchée.

3. Dans le volet d’exécutions du pipeline, section **Tâches**, cliquez sur **Phase 1** et surveillez la progression de la build jusqu’à son achèvement.
4. Dans la fenêtre du navigateur affichant le portail **Azure DevOps**, dans le volet de navigation vertical, section **Pipelines**, cliquez sur **Mises en production**, puis sur l’entrée représentant la mise en production. Dans le volet **Selenium > Mise en production-1**, cliquez sur **Développement**.
5. Dans le volet **Selenium > Mise en production-1 > Développement**, surveillez le déploiement correspondant.
6. Une fois la phase d’**exécution du test Selenium** démarrée, surveillez les tests du navigateur web.
7. Une fois la mise en production terminée, dans le volet **Selenium > Mise en production-1 > Développement**, cliquez sur l’onglet **Tests** pour analyser les résultats des tests. Sélectionnez les filtres requis dans le menu déroulant **Résultats** pour afficher les tests et leur état.

### Exercice 2 : supprimer les ressources du labo Azure

Dans cet exercice, vous allez supprimer les ressources Azure approvisionnées dans ce labo pour éviter des frais inattendus.

>**Remarque** : N’oubliez pas de supprimer toutes les nouvelles ressources Azure que vous n’utilisez plus. La suppression des ressources inutilisées vous évitera d’encourir des frais inattendus.

#### Tâche 1 : supprimer les ressources du labo Azure

Dans cette tâche, vous allez utiliser Azure Cloud Shell pour supprimer les ressources Azure approvisionnées dans ce labo pour éviter des frais inutiles.

1. Dans le Portail Azure, ouvrez la session shell **Bash** dans le volet **Cloud Shell**.
2. Listez tous les groupes de ressources créés dans les labos de ce module en exécutant la commande suivante :

    ```sh
    az group list --query "[?starts_with(name,'az400m11l02-RG')].name" --output tsv
    ```

3. Supprimez tous les groupes de ressources que vous avez créés dans les labos de ce module en exécutant la commande suivante :

    ```sh
    az group list --query "[?starts_with(name,'az400m11l02-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Remarque** : La commande s’exécute de façon asynchrone (comme déterminé par le paramètre --no-wait). Par conséquent, vous serez en mesure d’exécuter une autre commande Azure CLI immédiatement après au cours de la même session Bash, mais la suppression réelle du groupe de ressources prendra quelques minutes.

## Révision

Dans ce labo, vous allez découvrir comment exécuter des cas de test Sélénium sur une application web C# dans le cadre du pipeline de mise en production Azure DevOps.
