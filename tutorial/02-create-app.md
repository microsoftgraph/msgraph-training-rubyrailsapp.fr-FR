<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez utiliser [Ruby on rails](https://rubyonrails.org/) pour créer une application Web.

1. Si vous n’avez pas encore installé de rails, vous pouvez l’installer à partir de votre interface de ligne de commande (CLI) à l’aide de la commande suivante.

    ```Shell
    gem install rails -v 6.0.2.2
    ```

1. Ouvrez votre interface CLI, accédez à un répertoire où vous disposez des droits nécessaires pour créer des fichiers, puis exécutez la commande suivante pour créer une application de rails.

    ```Shell
    rails new graph-tutorial
    ```

1. Accédez à ce nouveau répertoire et entrez la commande suivante pour démarrer un serveur Web local.

    ```Shell
    rails server
    ```

1. Ouvrez votre navigateur et accédez à `http://localhost:3000`. Si tout fonctionne, vous verrez un «yay ! Vous êtes sur rails. Message. Si vous ne voyez pas ce message, consultez le [Guide de prise](http://guides.rubyonrails.org/)en main des rails.

## <a name="install-gems"></a>Installer gemme

Avant de poursuivre, installez des gemmes supplémentaires que vous utiliserez plus tard :

- [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) pour la gestion des flux de connexion et de jetons OAuth.
- [omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) pour l’ajout de la protection CSRF à omniauth.
- [HTTParty](https://github.com/jnunemaker/httparty) pour passer des appels à Microsoft Graph.
- [ActiveRecord-session_store](https://github.com/rails/activerecord-session_store) pour le stockage de sessions dans la base de données.

1. Ouvrez **./Gemfile** et ajoutez les lignes suivantes.

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. Dans votre interface CLI, exécutez la commande suivante.

    ```Shell
    bundle install
    ```

1. Dans votre interface CLI, exécutez les commandes suivantes pour configurer la base de données pour le stockage des sessions.

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. Créez un fichier appelé `session_store.rb` dans le répertoire **./config/Initializers** et ajoutez le code suivant.

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a>Concevoir l’application

Dans cette section, vous allez créer l’interface utilisateur de base pour l’application.

1. Ouvrez **./app/views/layouts/application.html.Erb** et remplacez son contenu par ce qui suit.

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    Ce code ajoute [Bootstrap](http://getbootstrap.com/) pour la mise en forme simple et [Font Awesome](https://fontawesome.com/) pour certaines icônes simples. Il définit également une disposition globale avec une barre de navigation.

1. Ouvrez **./app/Assets/StyleSheets/application.CSS** et ajoutez le code suivant à la fin du fichier.

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. Générez un contrôleur de page d’accueil à l’aide de la commande suivante.

    ```Shell
    rails generate controller Home index
    ```

1. Configurez l' `index` action `Home` sur le contrôleur comme page par défaut de l’application. Ouvrez **./config/routes.RB** et remplacez son contenu par ce qui suit :

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. Ouvrez **./app/View/Home/index.html.Erb** et remplacez son contenu par ce qui suit.

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. Enregistrez toutes vos modifications et redémarrez le serveur. À présent, l’application doit être très différente.

    ![Capture d’écran de la page d’accueil repensée](./images/create-app-01.png)
