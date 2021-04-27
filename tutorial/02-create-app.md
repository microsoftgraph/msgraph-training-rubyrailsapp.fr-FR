<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez utiliser [Ruby on Rails pour](https://rubyonrails.org/) créer une application web.

1. Si rails n'est pas déjà installé, vous pouvez l'installer à partir de votre interface de ligne de commande (CLI) à l'aide de la commande suivante.

    ```Shell
    gem install rails -v 6.1.3.1
    ```

1. Ouvrez votre CLI, accédez à un répertoire dans lequel vous avez le droit de créer des fichiers et exécutez la commande suivante pour créer une application Rails.

    ```Shell
    rails new graph-tutorial
    ```

1. Accédez à ce nouveau répertoire et entrez la commande suivante pour démarrer un serveur web local.

    ```Shell
    rails server
    ```

1. Ouvrez votre navigateur et accédez à `http://localhost:3000`. Si tout fonctionne, vous verrez un « Yay! Vous êtes sur Rails ! » Message. Si vous ne voyez pas ce message, consultez le guide de mise en [route de Rails.](http://guides.rubyonrails.org/)

## <a name="install-gems"></a>Installer des gems

Avant de passer à autre chose, installez d'autres gems que vous utiliserez ultérieurement :

- [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) pour la gestion des flux de jeton OAuth et de la signature.
- [omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) pour ajouter la protection CSRF à OmniAuth.
- [httparty pour](https://github.com/jnunemaker/httparty) effectuer des appels à Microsoft Graph.
- [activerecord-session_store](https://github.com/rails/activerecord-session_store) pour le stockage des sessions dans la base de données.

1. Ouvrez **./Gemfile** et ajoutez les lignes suivantes.

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. Dans votre CLI, exécutez la commande suivante.

    ```Shell
    bundle install
    ```

1. Dans votre CLI, exécutez les commandes suivantes pour configurer la base de données pour le stockage des sessions.

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. Créez un fichier appelé dans le répertoire `session_store.rb` **./config/initializers** et ajoutez le code suivant.

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a>Concevoir l’application

Dans cette section, vous allez créer l'interface utilisateur de base de l'application.

1. Ouvrez **./app/views/layouts/application.html.erb** et remplacez son contenu par ce qui suit.

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    Ce code ajoute [Bootstrap pour](http://getbootstrap.com/) un style simple et [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) pour certaines icônes simples. Il définit également une disposition globale avec une barre de navigation.

1. Ouvrez **./app/assets/stylesheets/application.css** et ajoutez ce qui suit à la fin du fichier.

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. Génére un contrôleur de page d'accueil avec la commande suivante.

    ```Shell
    rails generate controller Home index
    ```

1. Configurez `index` l'action sur le `Home` contrôleur comme page par défaut de l'application. Ouvrez **./config/routes.rb** et remplacez son contenu par ce qui suit

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. Ouvrez **./app/view/home/index.html.erb** et remplacez son contenu par ce qui suit.

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. Ajoutez un fichier PNG nommé **no-profile-photo.png** dans le répertoire **./app/assets/images.**

1. Enregistrez toutes vos modifications et redémarrez le serveur. L'application doit maintenant avoir une apparence très différente.

    ![Capture d’écran de la page d’accueil repensée](./images/create-app-01.png)
