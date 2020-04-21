<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="41de9-101">Dans cet exercice, vous allez utiliser [Ruby on rails](https://rubyonrails.org/) pour créer une application Web.</span><span class="sxs-lookup"><span data-stu-id="41de9-101">In this exercise you will use [Ruby on Rails](https://rubyonrails.org/) to build a web app.</span></span>

1. <span data-ttu-id="41de9-102">Si vous n’avez pas encore installé de rails, vous pouvez l’installer à partir de votre interface de ligne de commande (CLI) à l’aide de la commande suivante.</span><span class="sxs-lookup"><span data-stu-id="41de9-102">If you don't already have Rails installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

    ```Shell
    gem install rails -v 6.0.2.2
    ```

1. <span data-ttu-id="41de9-103">Ouvrez votre interface CLI, accédez à un répertoire où vous disposez des droits nécessaires pour créer des fichiers, puis exécutez la commande suivante pour créer une application de rails.</span><span class="sxs-lookup"><span data-stu-id="41de9-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Rails app.</span></span>

    ```Shell
    rails new graph-tutorial
    ```

1. <span data-ttu-id="41de9-104">Accédez à ce nouveau répertoire et entrez la commande suivante pour démarrer un serveur Web local.</span><span class="sxs-lookup"><span data-stu-id="41de9-104">Navigate to this new directory and enter the following command to start a local web server.</span></span>

    ```Shell
    rails server
    ```

1. <span data-ttu-id="41de9-105">Ouvrez votre navigateur et accédez à `http://localhost:3000`.</span><span class="sxs-lookup"><span data-stu-id="41de9-105">Open your browser and navigate to `http://localhost:3000`.</span></span> <span data-ttu-id="41de9-106">Si tout fonctionne, vous verrez un «yay !</span><span class="sxs-lookup"><span data-stu-id="41de9-106">If everything is working, you will see a "Yay!</span></span> <span data-ttu-id="41de9-107">Vous êtes sur rails.</span><span class="sxs-lookup"><span data-stu-id="41de9-107">You're on Rails!"</span></span> <span data-ttu-id="41de9-108">Message.</span><span class="sxs-lookup"><span data-stu-id="41de9-108">message.</span></span> <span data-ttu-id="41de9-109">Si vous ne voyez pas ce message, consultez le [Guide de prise](http://guides.rubyonrails.org/)en main des rails.</span><span class="sxs-lookup"><span data-stu-id="41de9-109">If you don't see that message, check the [Rails getting started guide](http://guides.rubyonrails.org/).</span></span>

## <a name="install-gems"></a><span data-ttu-id="41de9-110">Installer gemme</span><span class="sxs-lookup"><span data-stu-id="41de9-110">Install gems</span></span>

<span data-ttu-id="41de9-111">Avant de poursuivre, installez des gemmes supplémentaires que vous utiliserez plus tard :</span><span class="sxs-lookup"><span data-stu-id="41de9-111">Before moving on, install some additional gems that you will use later:</span></span>

- <span data-ttu-id="41de9-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) pour la gestion des flux de connexion et de jetons OAuth.</span><span class="sxs-lookup"><span data-stu-id="41de9-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="41de9-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) pour l’ajout de la protection CSRF à omniauth.</span><span class="sxs-lookup"><span data-stu-id="41de9-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) for adding CSRF protection to OmniAuth.</span></span>
- <span data-ttu-id="41de9-114">[HTTParty](https://github.com/jnunemaker/httparty) pour passer des appels à Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="41de9-114">[httparty](https://github.com/jnunemaker/httparty) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="41de9-115">[ActiveRecord-session_store](https://github.com/rails/activerecord-session_store) pour le stockage de sessions dans la base de données.</span><span class="sxs-lookup"><span data-stu-id="41de9-115">[activerecord-session_store](https://github.com/rails/activerecord-session_store) for storing sessions in the database.</span></span>

1. <span data-ttu-id="41de9-116">Ouvrez **./Gemfile** et ajoutez les lignes suivantes.</span><span class="sxs-lookup"><span data-stu-id="41de9-116">Open **./Gemfile** and add the following lines.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. <span data-ttu-id="41de9-117">Dans votre interface CLI, exécutez la commande suivante.</span><span class="sxs-lookup"><span data-stu-id="41de9-117">In your CLI, run the following command.</span></span>

    ```Shell
    bundle install
    ```

1. <span data-ttu-id="41de9-118">Dans votre interface CLI, exécutez les commandes suivantes pour configurer la base de données pour le stockage des sessions.</span><span class="sxs-lookup"><span data-stu-id="41de9-118">In your CLI, run the following commands to configure the database for storing sessions.</span></span>

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. <span data-ttu-id="41de9-119">Créez un fichier appelé `session_store.rb` dans le répertoire **./config/Initializers** et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="41de9-119">Create a new file called `session_store.rb` in the **./config/initializers** directory, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a><span data-ttu-id="41de9-120">Concevoir l’application</span><span class="sxs-lookup"><span data-stu-id="41de9-120">Design the app</span></span>

<span data-ttu-id="41de9-121">Dans cette section, vous allez créer l’interface utilisateur de base pour l’application.</span><span class="sxs-lookup"><span data-stu-id="41de9-121">In this section you'll create the basic UI for the app.</span></span>

1. <span data-ttu-id="41de9-122">Ouvrez **./app/views/layouts/application.html.Erb** et remplacez son contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="41de9-122">Open **./app/views/layouts/application.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    <span data-ttu-id="41de9-123">Ce code ajoute [Bootstrap](http://getbootstrap.com/) pour la mise en forme simple et [Font Awesome](https://fontawesome.com/) pour certaines icônes simples.</span><span class="sxs-lookup"><span data-stu-id="41de9-123">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="41de9-124">Il définit également une disposition globale avec une barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="41de9-124">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="41de9-125">Ouvrez **./app/Assets/StyleSheets/application.CSS** et ajoutez le code suivant à la fin du fichier.</span><span class="sxs-lookup"><span data-stu-id="41de9-125">Open **./app/assets/stylesheets/application.css** and add the following to the end of the file.</span></span>

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. <span data-ttu-id="41de9-126">Générez un contrôleur de page d’accueil à l’aide de la commande suivante.</span><span class="sxs-lookup"><span data-stu-id="41de9-126">Generate a home page controller with the following command.</span></span>

    ```Shell
    rails generate controller Home index
    ```

1. <span data-ttu-id="41de9-127">Configurez l' `index` action `Home` sur le contrôleur comme page par défaut de l’application.</span><span class="sxs-lookup"><span data-stu-id="41de9-127">Configure the `index` action on the `Home` controller as the default page for the app.</span></span> <span data-ttu-id="41de9-128">Ouvrez **./config/routes.RB** et remplacez son contenu par ce qui suit :</span><span class="sxs-lookup"><span data-stu-id="41de9-128">Open **./config/routes.rb** and replace its contents with the following</span></span>

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. <span data-ttu-id="41de9-129">Ouvrez **./app/View/Home/index.html.Erb** et remplacez son contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="41de9-129">Open **./app/view/home/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. <span data-ttu-id="41de9-130">Enregistrez toutes vos modifications et redémarrez le serveur.</span><span class="sxs-lookup"><span data-stu-id="41de9-130">Save all of your changes and restart the server.</span></span> <span data-ttu-id="41de9-131">À présent, l’application doit être très différente.</span><span class="sxs-lookup"><span data-stu-id="41de9-131">Now, the app should look very different.</span></span>

    ![Capture d’écran de la page d’accueil repensée](./images/create-app-01.png)
