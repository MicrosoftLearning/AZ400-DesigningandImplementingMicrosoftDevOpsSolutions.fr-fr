---
lab:
  title: Partage des connaissances de l’équipe à l’aide de wikis de projet Azure
  module: 'Module 09: Implement continuous feedback'
---

# Partage des connaissances de l’équipe à l’aide de wikis de projet Azure

## Manuel de labo de l’étudiant

## Configuration de laboratoire requise

- Ce labo nécessite **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour ce labo, créez-en une conformément aux instructions disponibles dans l’article [Prérequis du labo AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- **Configurer l’exemple de projet eShopOnWeb :** Si vous ne disposez pas encore de l’exemple de projet eShopOnWeb que vous pouvez utiliser pour ce labo, créez-le en suivant les instructions disponibles dans [Prérequis pour le labo AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

## Présentation du labo

Dans ce labo, vous allez créer et configurer un wiki dans Azure DevOps, ce qui comprend la gestion de contenu Markdown et la création d’un diagramme Mermaid.

## Objectifs

À la fin de ce labo, vous serez en mesure d’effectuer les tâches suivantes :

- Créer un wiki dans un projet Azure.
- Ajouter et modifier du contenu Markdown.
- Créer un diagramme Mermaid.

## Durée estimée : 45 minutes

## Instructions

### Exercice 0 : configurer les prérequis du labo

Dans cet exercice, nous souhaitons vous rappeler que vous devez valider les prérequis du labo, avoir préparé une organisation Azure DevOps et avoir créé le projet eShopOnWeb. Pour plus d’informations, consultez les instructions ci-dessus.

### Exercice 1 : publier du code en tant que wiki

Dans cet exercice, vous allez publier un référentiel Azure DevOps en tant que wiki et gérer le wiki publié.

> **Remarque** : le contenu que vous gérez dans un référentiel Git peut être publié sur un wiki Azure DevOps. Par exemple, le contenu écrit pour prendre en charge un kit de développement logiciel, une documentation produit ou des fichiers README peut être publié directement sur un wiki. Vous avez la possibilité de publier plusieurs wikis au sein d’un même projet d’équipe Azure DevOps.

#### Tâche 1 : publier une branche d’un référentiel Azure DevOps en tant que wiki

Dans cette tâche, vous allez publier une branche d’un référentiel Azure DevOps en tant que wiki.

> **Remarque** : si votre wiki publié correspond à une version de produit, vous pouvez publier de nouvelles branches lorsque vous publiez de nouvelles versions de votre produit.

1. Dans le menu vertical situé sur le côté gauche, cliquez sur **Référentiels**, dans la section supérieure du volet **Fichiers**, vérifiez que le référentiel **eShopOnWeb** est sélectionné (choisissez-le dans la liste déroulante située en haut avec l’icône Git). Dans la liste déroulante de branche (en haut de « Fichiers » avec l’icône de branche), sélectionnez **main** et passez en revue le contenu de la branche principale.
1. À gauche du volet **Fichiers**, dans la liste de la hiérarchie de dossiers et de fichiers du référentiel, développez le dossier **src** et accédez au sous-dossier **Web > wwwroot > images**. Dans le sous-dossier **Images**, recherchez l’entrée **brand.png**, pointez avec le pointeur de la souris sur son extrémité droite pour afficher le symbole de points de suspension verticaux (trois points) représentant le menu **Plus**, puis cliquez sur **Télécharger** pour télécharger le fichier **brand.png** dans le dossier **Downloads** local sur votre ordinateur de labo.

    >**Remarque** : vous utiliserez cette image dans l’exercice suivant de ce labo.

1. Nous allons stocker les fichiers sources Wiki dans un dossier distinct dans la structure de dossiers actuelle Référentiels. Dans **Dépôts**, sélectionnez **Fichiers**. Notez le titre du référentiel **eShopOnWeb** au-dessus de la structure de dossiers. **Sélectionnez les points de suspension (3 points)**, choisissez **Nouveau / Dossier**, puis indiquez **Documents** comme titre du nouveau dossier. Comme un référentiel ne vous permet pas de créer un dossier vide, indiquez **READ.ME** comme nouveau nom de fichier.
1. Confirmez la création du dossier et du fichier en **appuyant sur le bouton Créer**.
1. Le fichier READ.ME s’ouvre en mode d’affichage intégré. Étant donné qu’il est stocké « en tant que code », vous devez **valider** les modifications apportées en cliquant sur le bouton **Valider**. Dans la fenêtre de validation, confirmez à nouveau en appuyant sur **Valider**.
1. Dans le menu vertical Azure DevOps sur le côté gauche, cliquez sur **Vue d’ensemble**, dans la section **Vue d’ensemble**, sélectionnez **Wiki**, puis **Publier du code en tant que Wiki*.
1. Dans le volet **Publier du code en tant que Wiki**, spécifiez les paramètres suivants, puis cliquez sur **Publier**.

    | Paramètre | Valeur |
    | ------- | ----- |
    | Référentiel | **eShopOnWeb** |
    | Branche | **main** |
    | Dossier | **/Documents** |
    | Nom du wiki | **eShopOnWeb (Documents)** |

    >**Remarque** : cette opération ouvre automatiquement la section Wiki et publie **l’éditeur**, où vous pouvez fournir un titre de page wiki, ainsi qu’ajouter le contenu réel. Vous êtes invité à utiliser le format MarkDown, mais utilisez le ruban pour vous aider dans le cadre de la syntaxe MarkDown.

1. Dans le champ **Titre** de la page wiki, entrez « Welcome to our Online Retail Store! ».

1. Dans le corps de la page wiki, collez le texte suivant :

    ```text
    ##Welcome to Our Online Retail Store!
    At our online retail store, we offer a **wide range of products** to meet the **needs of our customers**. Our selection includes everything from *clothing and accessories to electronics, home decor, and more*.
    
    We pride ourselves on providing a seamless shopping experience for our customers. Our website offers the following benefits:
    1. user-friendly,
    1. and easy to navigate, 
    1. allowing you to find what you're looking for,
    1. quickly and easily. 
    
    We also offer a range of **_payment and shipping options_** to make your shopping experience as convenient as possible.
    
    ### about the team
    Our team is dedicated to providing exceptional customer service. If you have any questions or concerns, our knowledgeable and friendly support team is always available to assist you. We also offer a hassle-free return policy, so if you're not completely satisfied with your purchase, you can easily return it for a refund or exchange.
    
    ### Physical Stores
    |Location|Area|Hours|
    |--|--|--|
    | New Orleans | Home and DIY  |07.30am-09.30pm  |
    | Seattle | Gardening | 10.00am-08.30pm  |
    | New York | Furniture Specialists  | 10.00am-09.00pm |
    
    ## Our Store Qualities
    - We're committed to providing high-quality products
    - Our products are offered at affordable prices 
    - We work with reputable suppliers and manufacturers 
    - We ensure that our products meet our strict standards for quality and durability. 
    - Plus, we regularly offer sales and discounts to help you save even more.
    
    #Summary
    Thank you for choosing our online retail store for your shopping needs. We look forward to serving you!
    ```

1. Cet exemple de texte vous donne un aperçu de plusieurs fonctionnalités courantes de la syntaxe MarkDown, y compris le titre, les sous-titres (##), la mise en gras (**), la mise en italique (*), la création de tables, etc.

1. Une fois terminé, **appuyez** sur le bouton Enregistrer dans le coin supérieur droit.

1. **Actualisez** votre navigateur, ou sélectionnez une autre option du portail DevOps, puis revenez à la section Wiki. Le wiki **EshopOnWeb (Documents)** s’affiche à présent, avec la **page d’accueil** **Welcome to our Online Retail Store**.

#### Tâche 2 : gérer le contenu d’un wiki publié

Dans cette tâche, vous allez gérer le contenu du wiki que vous avez publié dans la tâche précédente.

1. Dans le menu vertical situé sur le côté gauche, cliquez sur **Référentiels**, vérifiez que le menu déroulant dans la section supérieure du volet **Fichiers** affiche le référentiel **eShopOnWeb** et la branche **main**, dans l’arborescence des dossiers du référentiel, sélectionnez le dossier **Documents**, puis sélectionnez le fichier **Welcome-to-our-Online-Retail-Store!.md**.
1. Le format MarkDown est visible ici en tant que format de texte brut, ce qui vous permet également de continuer à modifier le contenu du fichier à partir d’ici.

> **Remarque** : étant donné que les fichiers sources wiki sont gérés en tant que code source, n’oubliez pas que toutes les pratiques du contrôle de code source traditionnel (clonage, demandes de tirage, approbations, etc.) peuvent désormais être appliquées aux pages wiki.

### Exercice 2 : créer et gérer un wiki de projet

Dans cet exercice, vous allez créer et gérer un wiki de projet.

> **Remarque** : vous pouvez créer et gérer un wiki indépendamment des référentiels existants.

#### Tâche 1 : créer un wiki de projet incluant un diagramme Mermaid et une image

Dans cette tâche, vous allez créer un wiki de projet et y ajouter un diagramme Mermaid et une image.

1. Sur votre ordinateur de labo, dans le portail Azure DevOps affichant le **volet Wiki** du projet **EShopOnweb**, une fois le contenu du wiki **eShopOnWeb (Documents)** sélectionné, en haut du volet, cliquez sur l’en-tête de liste déroulante **eShopOnWeb (Documents)** (icône de flèche vers le bas), puis, dans la liste déroulante, sélectionnez **Créer un wiki de projet**.
1. Dans la zone de texte **Titre de la page**, tapez **Conception de projet**.
1. Placez le curseur dans le corps de la page, cliquez sur l’icône la plus à gauche dans la barre d’outils représentant le paramètre d’en-tête, puis, dans la liste déroulante, cliquez sur **En-tête 1**. Cette opération ajoute automatiquement le caractère de hachage (**#**) au début de la ligne.
1. Directement après le nouveau caractère **#** ajouté, tapez **Authentification et Autorisation**, puis appuyez sur la touche **Entrée**.
1. Cliquez sur l’icône la plus à gauche dans la barre d’outils représentant le paramètre d’en-tête, puis, dans la liste déroulante, cliquez sur **En-tête 2**. Cette opération ajoute automatiquement le caractère de hachage (**##**) au début de la ligne.
1. Directement après le nouveau caractère **##** ajouté, tapez **Flux d’autorisation Azure DevOps OAuth 2.0**, puis appuyez sur la touche **Entrée**.
1. **Copiez et collez** le code suivant pour insérer un diagramme Mermaid sur votre wiki.

    ```text
    ::: mermaid
    sequenceDiagram
     participant U as User
     participant A as Your app
     participant D as Azure DevOps
     U->>A: Use your app
     A->>D: Request authorization for user
     D-->>U: Request authorization
     U->>D: Grant authorization
     D-->>A: Send authorization code
     A->>D: Get access token
     D-->>A: Send access token
     A->>D: Call REST API with access token
     D-->>A: Respond to REST API
     A-->>U: Relay REST API response
    :::
    ```

    >**Remarque** : pour plus d’informations sur la syntaxe Mermaid, consultez la page de [présentation de Mermaid](https://mermaid-js.github.io/mermaid/#/).

1. À droite du volet de l’éditeur, dans le volet d’aperçu, cliquez sur **Charger le diagramme**, puis passez en revue les résultats.

    >**Remarque** : la sortie doit ressembler à l’organigramme qui montre comment [autoriser l’accès aux API REST avec OAuth 2.0](https://docs.microsoft.com/azure/devops/integrate/get-started/authentication/oauth).

1. Dans le coin supérieur droit du volet de l’éditeur, cliquez sur l’accent circonflexe vers le bas en regard du bouton **Enregistrer**, puis, dans le menu déroulant, cliquez sur **Enregistrer avec le message de révision**.
1. Dans la boîte de dialogue **Enregistrer la page**, tapez **Section Authentification et autorisation avec diagramme Mermaid OAuth 2.0**, puis cliquez sur **Enregistrer**.
1. Dans le volet de l’éditeur **Conception de projet**, placez le curseur à la fin de l’élément Mermaid que vous avez ajouté précédemment dans cette tâche, appuyez sur la touche **Entrée** pour ajouter une ligne supplémentaire, cliquez sur l’icône la plus à gauche dans la barre d’outils représentant le paramètre d’en-tête, puis, dans la liste déroulante, cliquez sur **En-tête 2**. Cette opération ajoute automatiquement le caractère de hachage double (**##**) au début de la ligne.
1. Directement après le nouveau caractère **##** ajouté, tapez **Interface utilisateur**, puis appuyez sur la touche **Entrée**.
1. Dans le volet de l’éditeur **Conception de projet**, dans la barre d’outils, cliquez sur l’icône de trombone représentant l’action **Insérer un fichier**, dans la boîte de dialogue **Ouvrir**, accédez au dossier **Downloads**, sélectionnez le fichier **Brand.png** que vous avez téléchargé dans l’exercice précédent, puis cliquez sur **Ouvrir**.
1. Dans le volet de l’éditeur **Conception de projet**, passez en revue le volet d’aperçu et vérifiez que l’image est correctement affichée.
1. Dans le coin supérieur droit du volet de l’éditeur, cliquez sur l’accent circonflexe vers le bas en regard du bouton **Enregistrer**, puis, dans le menu déroulant, cliquez sur **Enregistrer avec le message de révision**.
1. Dans la boîte de dialogue **Enregistrer la page**, tapez **Section Interface utilisateur avec image Tailwind Traders**, puis cliquez sur **Enregistrer**.
1. Dans le volet de l’éditeur, dans le coin supérieur droit, cliquez sur **Fermer**.

#### Tâche 2 : gérer un wiki de projet

Dans cette tâche, vous allez gérer le nouveau wiki de projet.

>**Remarque** : vous commencerez par rétablir la modification la plus récente de la page wiki.

1. Sur votre ordinateur de labo, dans le portail Azure DevOps affichant le **volet Wiki** du projet **eShopOnWeb**, une fois le contenu du wiki **Conception de projet** sélectionné, dans le coin supérieur droit, cliquez sur le symbole des points de suspension verticaux, puis, dans le menu déroulant, cliquez sur **Afficher les révisions**.
1. Dans le volet **Révisions**, cliquez sur l’entrée représentant la modification la plus récente.
1. Dans le volet résultant, passez en revue la comparaison entre la version précédente et la version actuelle du document, cliquez sur **Rétablir**, lorsque vous êtes invité à confirmer l’opération, cliquez à nouveau sur **Rétablir**, puis cliquez sur **Afficher la page**.
1. Dans le volet **Conception de projet**, vérifiez que la modification a été rétablie.

    >**Remarque** : vous allez à présent ajouter une autre page au wiki de projet et la définir comme page d’accueil du wiki.

1. Dans le volet **Conception de projet**, dans le coin inférieur gauche, cliquez sur **+ Nouvelle page**.
1. Dans le volet de l’éditeur de page, dans la zone de texte **Titre de la page**, tapez **Vue d’ensemble Conception de projet**, cliquez sur **Enregistrer**, puis sur **Fermer**.
1. Dans le volet répertoriant les pages du wiki de projet **Conception de projet**, recherchez l’entrée **Vue d’ensemble Conception de projet**, sélectionnez-la avec le pointeur de la souris, puis faites-la glisser sur l’entrée de page **Conception de projet**.
1. Confirmez les modifications apportées en appuyant sur le bouton **Déplacer** dans la fenêtre qui s’affiche.
1. Vérifiez que l’entrée **Vue d’ensemble Conception de projet** est affichée comme page de niveau supérieur avec l’icône d’accueil qui la désigne comme page d’accueil du wiki.

## Révision

Dans ce labo, vous avez créé et configuré un wiki dans Azure DevOps, ce qui comprend la gestion de contenu Markdown et la création d’un diagramme Mermaid.
