<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="d7a7c-101">Dans cet exercice, vous allez utiliser [Ruby on rails](https://rubyonrails.org/) pour créer une application Web.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-101">In this exercise you will use [Ruby on Rails](https://rubyonrails.org/) to build a web app.</span></span> <span data-ttu-id="d7a7c-102">Si vous n’avez pas encore installé de rails, vous pouvez l’installer à partir de votre interface de ligne de commande (CLI) à l’aide de la commande suivante.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-102">If you don't already have Rails installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

```Shell
gem install rails
```

<span data-ttu-id="d7a7c-103">Ouvrez votre interface CLI, accédez à un répertoire où vous disposez des droits nécessaires pour créer des fichiers, puis exécutez la commande suivante pour créer une application de rails.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Rails app.</span></span>

```Shell
rails new graph-tutorial
```

<span data-ttu-id="d7a7c-104">Rails crée un nouveau répertoire appelé `graph-tutorial` et génère une application de rails.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-104">Rails creates a new directory called `graph-tutorial` and scaffolds a Rails app.</span></span> <span data-ttu-id="d7a7c-105">Accédez à ce nouveau répertoire et entrez la commande suivante pour démarrer un serveur Web local.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-105">Navigate to this new directory and enter the following command to start a local web server.</span></span>

```Shell
rails server
```

<span data-ttu-id="d7a7c-106">Ouvrez votre navigateur et accédez à `http://localhost:3000`.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-106">Open your browser and navigate to `http://localhost:3000`.</span></span> <span data-ttu-id="d7a7c-107">Si tout fonctionne, vous verrez un «yay !</span><span class="sxs-lookup"><span data-stu-id="d7a7c-107">If everything is working, you will see a "Yay!</span></span> <span data-ttu-id="d7a7c-108">Vous êtes sur rails.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-108">You're on Rails!"</span></span> <span data-ttu-id="d7a7c-109">Message.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-109">message.</span></span> <span data-ttu-id="d7a7c-110">Si vous ne voyez pas ce message, consultez le [Guide de prise](http://guides.rubyonrails.org/)en main des rails.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-110">If you don't see that message, check the [Rails getting started guide](http://guides.rubyonrails.org/).</span></span>

<span data-ttu-id="d7a7c-111">Avant de poursuivre, installez des gemmes supplémentaires que vous utiliserez plus tard :</span><span class="sxs-lookup"><span data-stu-id="d7a7c-111">Before moving on, install some additional gems that you will use later:</span></span>

- <span data-ttu-id="d7a7c-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) pour la gestion des flux de connexion et de jetons OAuth.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="d7a7c-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) pour l’ajout de la protection CSRF à omniauth.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) for adding CSRF protection to OmniAuth.</span></span>
- <span data-ttu-id="d7a7c-114">[HTTParty](https://github.com/jnunemaker/httparty) pour passer des appels à Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-114">[httparty](https://github.com/jnunemaker/httparty) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="d7a7c-115">[nokogiri](https://github.com/sparklemotion/nokogiri) pour traiter les corps de message html.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-115">[nokogiri](https://github.com/sparklemotion/nokogiri) to process HTML bodies of email.</span></span>
- <span data-ttu-id="d7a7c-116">[ActiveRecord-session_store](https://github.com/rails/activerecord-session_store) pour le stockage de sessions dans la base de données.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-116">[activerecord-session_store](https://github.com/rails/activerecord-session_store) for storing sessions in the database.</span></span>

<span data-ttu-id="d7a7c-117">Exécutez les commandes suivantes dans votre interface CLI.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-117">Run the following commands in your CLI.</span></span>

```Shell
bundle add omniauth-oauth2
bundle add omniauth-rails_csrf_protection
bundle add httparty
bundle add nokogiri
bundle add activerecord-session_store
rails generate active_record:session_migration
```

<span data-ttu-id="d7a7c-118">La dernière commande génère une sortie semblable à la suivante :</span><span class="sxs-lookup"><span data-stu-id="d7a7c-118">The last command generates output like the following:</span></span>

```Shell
create  db/migrate/20180618172216_add_sessions_table.rb
```

<span data-ttu-id="d7a7c-119">Ouvrez le fichier qui a été créé et recherchez la ligne suivante.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-119">Open the file that was created and locate the following line.</span></span>

```ruby
class AddSessionsTable < ActiveRecord::Migration
```

<span data-ttu-id="d7a7c-120">Modifiez cette ligne comme suit.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-120">Change that line to the following.</span></span>

```ruby
class AddSessionsTable < ActiveRecord::Migration[5.2]
```

> [!NOTE]
> <span data-ttu-id="d7a7c-121">Cela suppose que vous utilisez les rails 5.2. x.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-121">This assumes that you are using Rails 5.2.x.</span></span> <span data-ttu-id="d7a7c-122">Si vous utilisez une autre version, remplacez `5.2` par votre version.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-122">If you are using a different version, replace `5.2` with your version.</span></span>

<span data-ttu-id="d7a7c-123">Enregistrez le fichier et exécutez la commande suivante.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-123">Save the file and run the following command.</span></span>

```Shell
rake db:migrate
```

<span data-ttu-id="d7a7c-124">Enfin, configurez les rails pour utiliser le nouveau magasin de session.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-124">Finally, configure Rails to use the new session store.</span></span> <span data-ttu-id="d7a7c-125">Créez un fichier appelé `session_store.rb` dans le `./config/initializers` répertoire et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-125">Create a new file called `session_store.rb` in the `./config/initializers` directory, and add the following code.</span></span>

```ruby
Rails.application.config.session_store :active_record_store, key: '_graph_app_session'
```

## <a name="design-the-app"></a><span data-ttu-id="d7a7c-126">Concevoir l’application</span><span class="sxs-lookup"><span data-stu-id="d7a7c-126">Design the app</span></span>

<span data-ttu-id="d7a7c-127">Commencez par mettre à jour la disposition globale de l’application.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-127">Start by updating the global layout for the app.</span></span> <span data-ttu-id="d7a7c-128">Ouvrez `./app/views/layouts/application.html.erb` et remplacez son contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-128">Open `./app/views/layouts/application.html.erb` and replace its contents with the following.</span></span>

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

<span data-ttu-id="d7a7c-129">Ce code ajoute [bootstrap](http://getbootstrap.com/) pour les styles simples et [font Isard](https://fontawesome.com/) pour certaines icônes simples.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-129">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="d7a7c-130">Il définit également une disposition globale avec une barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-130">It also defines a global layout with a nav bar.</span></span>

<span data-ttu-id="d7a7c-131">Maintenant, `./app/assets/stylesheets/application.css` ouvrez et ajoutez le code suivant à la fin du fichier.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-131">Now open `./app/assets/stylesheets/application.css` and add the following to the end of the file.</span></span>

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

<span data-ttu-id="d7a7c-132">À présent, remplacez la page par défaut par une nouvelle.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-132">Now replace the default page with a new one.</span></span> <span data-ttu-id="d7a7c-133">Générez un contrôleur de page d’accueil à l’aide de la commande suivante.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-133">Generate a home page controller with the following command.</span></span>

```Shell
rails generate controller Home index
```

<span data-ttu-id="d7a7c-134">Configurez `index` ensuite l’action `Home` sur le contrôleur comme page par défaut de l’application.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-134">Then configure the `index` action on the `Home` controller as the default page for the app.</span></span> <span data-ttu-id="d7a7c-135">Ouvrez `./config/routes.rb` et remplacez le contenu par ce qui suit :</span><span class="sxs-lookup"><span data-stu-id="d7a7c-135">Open `./config/routes.rb` and replace the contents with the following</span></span>

```ruby
Rails.application.routes.draw do
  get 'home/index'
  root 'home#index'

  # Add future routes here

end
```

<span data-ttu-id="d7a7c-136">Ouvrez à présent `./app/view/home/index.html.erb` le fichier et remplacez son contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-136">Now open the `./app/view/home/index.html.erb` file and replace its contents with the following.</span></span>

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

<span data-ttu-id="d7a7c-137">Enregistrez toutes vos modifications et redémarrez le serveur.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-137">Save all of your changes and restart the server.</span></span> <span data-ttu-id="d7a7c-138">À présent, l’application doit être très différente.</span><span class="sxs-lookup"><span data-stu-id="d7a7c-138">Now, the app should look very different.</span></span>

![Capture d’écran de la page d’accueil repensée](./images/create-app-01.png)
