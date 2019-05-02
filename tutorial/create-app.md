<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez utiliser [Ruby on rails](https://rubyonrails.org/) pour créer une application Web. Si vous n’avez pas encore installé de rails, vous pouvez l’installer à partir de votre interface de ligne de commande (CLI) à l’aide de la commande suivante.

```Shell
gem install rails
```

Ouvrez votre interface CLI, accédez à un répertoire où vous disposez des droits nécessaires pour créer des fichiers, puis exécutez la commande suivante pour créer une application de rails.

```Shell
rails new graph-tutorial
```

Rails crée un nouveau répertoire appelé `graph-tutorial` et génère une application de rails. Accédez à ce nouveau répertoire et entrez la commande suivante pour démarrer un serveur Web local.

```Shell
rails server
```

Ouvrez votre navigateur et accédez à `http://localhost:3000`. Si tout fonctionne, vous verrez un «yay! Vous êtes sur rails. Message. Si vous ne voyez pas ce message, consultez le [Guide de prise](http://guides.rubyonrails.org/)en main des rails.

Avant de poursuivre, installez des gemmes supplémentaires que vous utiliserez plus tard:

- [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) pour la gestion des flux de connexion et de jetons OAuth.
- [HTTParty](https://github.com/jnunemaker/httparty) pour passer des appels à Microsoft Graph.
- [nokogiri](https://github.com/sparklemotion/nokogiri) pour traiter les corps de message html.
- [ActiveRecord-session_store](https://github.com/rails/activerecord-session_store) pour le stockage de sessions dans la base de données.

Exécutez les commandes suivantes dans votre interface CLI.

```Shell
bundle add omniauth-oauth2
bundle add httparty
bundle add nokogiri
bundle add activerecord-session_store
rails generate active_record:session_migration
```

La dernière commande génère une sortie semblable à la suivante:

```Shell
create  db/migrate/20180618172216_add_sessions_table.rb
```

Ouvrez le fichier qui a été créé et recherchez la ligne suivante.

```ruby
class AddSessionsTable < ActiveRecord::Migration
```

Modifiez cette ligne comme suit.

```ruby
class AddSessionsTable < ActiveRecord::Migration[5.2]
```

> [!NOTE]
> Cela suppose que vous utilisez les rails 5.2. x. Si vous utilisez une autre version, remplacez `5.2` par votre version.

Enregistrez le fichier et exécutez la commande suivante.

```Shell
rake db:migrate
```

Enfin, configurez les rails pour utiliser le nouveau magasin de session. Créez un fichier appelé `session_store.rb` dans le `./config/initializers` répertoire et ajoutez le code suivant.

```ruby
Rails.application.config.session_store :active_record_store, key: '_graph_app_session'
```

## <a name="design-the-app"></a>Concevoir l’application

Commencez par mettre à jour la disposition globale de l’application. Ouvrez `./app/views/layouts/application.html.erb` et remplacez son contenu par ce qui suit.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Ruby Graph Tutorial</title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css"
      integrity="sha384-WskhaSGFgHYWDcbwN70/dfYBj47jz9qbsMId/iRN3ewGhXQFZCSftd1LZCfmhktB" crossorigin="anonymous">
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.1.0/css/all.css"
      integrity="sha384-lKuwvrZot6UHsBSfcMvOkWwlCMgc0TaWr+30HWe3a4ltaBwTZhyTEggF5tJv8tbt" crossorigin="anonymous">
    <script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js"
      integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/js/bootstrap.min.js"
      integrity="sha384-smHYKdLADwkXOn1EmN1qk/HfnUcbVRZyYmZ4qpPea6sjB/pTJ0euyQp0Mk8ck+5T" crossorigin="anonymous"></script>
    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

  <body>
    <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
      <div class="container">
        <%= link_to "Ruby Graph Tutorial", root_path, class: "navbar-brand" %>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse"
          aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarCollapse">
          <ul class="navbar-nav mr-auto">
            <li class="nav-item">
              <%= link_to "Home", root_path, class: "nav-link#{' active' if controller.controller_name == 'home'}" %>
            </li>
            <% if @user_name %>
              <li class="nav-item" data-turbolinks="false">
                <a class="nav-link" href="#">Calendar</a>
              </li>
            <% end %>
          </ul>
          <ul class="navbar-nav justify-content-end">
            <li class="nav-item">
              <a class="nav-link" href="https://developer.microsoft.com/graph/docs/concepts/overview" target="_blank">
                <i class="fas fa-external-link-alt mr-1"></i>Docs
              </a>
            </li>
            <% if @user_name %>
              <li class="nav-item dropdown">
                <a class="nav-link dropdown-toggle" data-toggle="dropdown" href="#" role="button" aria-haspopup="true" aria-expanded="false">
                  <% if @user_avatar %>
                    <img src=<%= @user_avatar %> class="rounded-circle align-self-center mr-2" style="width: 32px;">
                  <% else %>
                    <i class="far fa-user-circle fa-lg rounded-circle align-self-center mr-2" style="width: 32px;"></i>
                  <% end %>
                </a>
                <div class="dropdown-menu dropdown-menu-right">
                  <h5 class="dropdown-item-text mb-0"><%= @user_name %></h5>
                  <p class="dropdown-item-text text-muted mb-0"><%= @user_email %></p>
                  <div class="dropdown-divider"></div>
                  <a href="#" class="dropdown-item">Sign Out</a>
                </div>
              </li>
            <% else %>
              <li class="nav-item">
                <a href="#" class="nav-link">Sign In</a>
              </li>
            <% end %>
          </ul>
        </div>
      </div>
    </nav>
    <main role="main" class="container">
      <% if @errors %>
        <% @errors.each do |error| %>
          <div class="alert alert-danger" role="alert">
            <p class="mb-3"><%= error[:message] %></p>
            <%if error[:debug] %>
              <pre class="alert-pre border bg-light p-2"><code><%= error[:debug] %></code></pre>
            <% end %>
          </div>
        <% end %>
      <% end %>
      <%= yield %>
    </main>
  </body>
</html>
```

Ce code ajoute [bootstrap](http://getbootstrap.com/) pour les styles simples et [font Isard](https://fontawesome.com/) pour certaines icônes simples. Il définit également une disposition globale avec une barre de navigation.

Maintenant, `./app/assets/stylesheets/application.css` ouvrez et ajoutez le code suivant à la fin du fichier.

```css
body {
  padding-top: 4.5rem;
}

.alert-pre {
  word-wrap: break-word;
  word-break: break-all;
  white-space: pre-wrap;
}
```

À présent, remplacez la page par défaut par une nouvelle. Générez un contrôleur de page d’accueil à l’aide de la commande suivante.

```Shell
rails generate controller Home index
```

Configurez `index` ensuite l’action `Home` sur le contrôleur comme page par défaut de l’application. Ouvrez `./config/routes.rb` et remplacez le contenu par ce qui suit:

```ruby
Rails.application.routes.draw do
  get 'home/index'
  root 'home#index'

  # Add future routes here

end
```

Ouvrez à présent `./app/view/home/index.html.erb` le fichier et remplacez son contenu par ce qui suit.

```html
<div class="jumbotron">
  <h1>Ruby Graph Tutorial</h1>
  <p class="lead">This sample app shows how to use the Microsoft Graph API to access Outlook and OneDrive data from Ruby</p>
  <% if @user_name %>
    <h4>Welcome <%= @user_name %>!</h4>
    <p>Use the navigation bar at the top of the page to get started.</p>
  <% else %>
    <a href="#" class="btn btn-primary btn-large">Click here to sign in</a>
  <% end %>
</div>
```

Enregistrez toutes vos modifications et redémarrez le serveur. À présent, l’application doit être très différente.

![Capture d’écran de la page d’accueil repensée](./images/create-app-01.png)