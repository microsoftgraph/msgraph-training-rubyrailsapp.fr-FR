<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="1295f-101">Dans cet exercice, vous allez utiliser [Ruby on Rails pour](https://rubyonrails.org/) créer une application web.</span><span class="sxs-lookup"><span data-stu-id="1295f-101">In this exercise you will use [Ruby on Rails](https://rubyonrails.org/) to build a web app.</span></span>

1. <span data-ttu-id="1295f-102">Si rails n'est pas déjà installé, vous pouvez l'installer à partir de votre interface de ligne de commande (CLI) à l'aide de la commande suivante.</span><span class="sxs-lookup"><span data-stu-id="1295f-102">If you don't already have Rails installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

    ```Shell
    gem install rails -v 6.1.3.1
    ```

1. <span data-ttu-id="1295f-103">Ouvrez votre CLI, accédez à un répertoire dans lequel vous avez le droit de créer des fichiers et exécutez la commande suivante pour créer une application Rails.</span><span class="sxs-lookup"><span data-stu-id="1295f-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Rails app.</span></span>

    ```Shell
    rails new graph-tutorial
    ```

1. <span data-ttu-id="1295f-104">Accédez à ce nouveau répertoire et entrez la commande suivante pour démarrer un serveur web local.</span><span class="sxs-lookup"><span data-stu-id="1295f-104">Navigate to this new directory and enter the following command to start a local web server.</span></span>

    ```Shell
    rails server
    ```

1. <span data-ttu-id="1295f-105">Ouvrez votre navigateur et accédez à `http://localhost:3000`.</span><span class="sxs-lookup"><span data-stu-id="1295f-105">Open your browser and navigate to `http://localhost:3000`.</span></span> <span data-ttu-id="1295f-106">Si tout fonctionne, vous verrez un « Yay!</span><span class="sxs-lookup"><span data-stu-id="1295f-106">If everything is working, you will see a "Yay!</span></span> <span data-ttu-id="1295f-107">Vous êtes sur Rails ! »</span><span class="sxs-lookup"><span data-stu-id="1295f-107">You're on Rails!"</span></span> <span data-ttu-id="1295f-108">Message.</span><span class="sxs-lookup"><span data-stu-id="1295f-108">message.</span></span> <span data-ttu-id="1295f-109">Si vous ne voyez pas ce message, consultez le guide de mise en [route de Rails.](http://guides.rubyonrails.org/)</span><span class="sxs-lookup"><span data-stu-id="1295f-109">If you don't see that message, check the [Rails getting started guide](http://guides.rubyonrails.org/).</span></span>

## <a name="install-gems"></a><span data-ttu-id="1295f-110">Installer des gems</span><span class="sxs-lookup"><span data-stu-id="1295f-110">Install gems</span></span>

<span data-ttu-id="1295f-111">Avant de passer à autre chose, installez d'autres gems que vous utiliserez ultérieurement :</span><span class="sxs-lookup"><span data-stu-id="1295f-111">Before moving on, install some additional gems that you will use later:</span></span>

- <span data-ttu-id="1295f-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) pour la gestion des flux de jeton OAuth et de la signature.</span><span class="sxs-lookup"><span data-stu-id="1295f-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="1295f-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) pour ajouter la protection CSRF à OmniAuth.</span><span class="sxs-lookup"><span data-stu-id="1295f-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) for adding CSRF protection to OmniAuth.</span></span>
- <span data-ttu-id="1295f-114">[httparty pour](https://github.com/jnunemaker/httparty) effectuer des appels à Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="1295f-114">[httparty](https://github.com/jnunemaker/httparty) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="1295f-115">[activerecord-session_store](https://github.com/rails/activerecord-session_store) pour le stockage des sessions dans la base de données.</span><span class="sxs-lookup"><span data-stu-id="1295f-115">[activerecord-session_store](https://github.com/rails/activerecord-session_store) for storing sessions in the database.</span></span>

1. <span data-ttu-id="1295f-116">Ouvrez **./Gemfile** et ajoutez les lignes suivantes.</span><span class="sxs-lookup"><span data-stu-id="1295f-116">Open **./Gemfile** and add the following lines.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. <span data-ttu-id="1295f-117">Dans votre CLI, exécutez la commande suivante.</span><span class="sxs-lookup"><span data-stu-id="1295f-117">In your CLI, run the following command.</span></span>

    ```Shell
    bundle install
    ```

1. <span data-ttu-id="1295f-118">Dans votre CLI, exécutez les commandes suivantes pour configurer la base de données pour le stockage des sessions.</span><span class="sxs-lookup"><span data-stu-id="1295f-118">In your CLI, run the following commands to configure the database for storing sessions.</span></span>

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. <span data-ttu-id="1295f-119">Créez un fichier appelé dans le répertoire `session_store.rb` **./config/initializers** et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="1295f-119">Create a new file called `session_store.rb` in the **./config/initializers** directory, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a><span data-ttu-id="1295f-120">Concevoir l’application</span><span class="sxs-lookup"><span data-stu-id="1295f-120">Design the app</span></span>

<span data-ttu-id="1295f-121">Dans cette section, vous allez créer l'interface utilisateur de base de l'application.</span><span class="sxs-lookup"><span data-stu-id="1295f-121">In this section you'll create the basic UI for the app.</span></span>

1. <span data-ttu-id="1295f-122">Ouvrez **./app/views/layouts/application.html.erb** et remplacez son contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="1295f-122">Open **./app/views/layouts/application.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    <span data-ttu-id="1295f-123">Ce code ajoute [Bootstrap pour](http://getbootstrap.com/) un style simple et [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) pour certaines icônes simples.</span><span class="sxs-lookup"><span data-stu-id="1295f-123">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) for some simple icons.</span></span> <span data-ttu-id="1295f-124">Il définit également une disposition globale avec une barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="1295f-124">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="1295f-125">Ouvrez **./app/assets/stylesheets/application.css** et ajoutez ce qui suit à la fin du fichier.</span><span class="sxs-lookup"><span data-stu-id="1295f-125">Open **./app/assets/stylesheets/application.css** and add the following to the end of the file.</span></span>

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. <span data-ttu-id="1295f-126">Génére un contrôleur de page d'accueil avec la commande suivante.</span><span class="sxs-lookup"><span data-stu-id="1295f-126">Generate a home page controller with the following command.</span></span>

    ```Shell
    rails generate controller Home index
    ```

1. <span data-ttu-id="1295f-127">Configurez `index` l'action sur le `Home` contrôleur comme page par défaut de l'application.</span><span class="sxs-lookup"><span data-stu-id="1295f-127">Configure the `index` action on the `Home` controller as the default page for the app.</span></span> <span data-ttu-id="1295f-128">Ouvrez **./config/routes.rb** et remplacez son contenu par ce qui suit</span><span class="sxs-lookup"><span data-stu-id="1295f-128">Open **./config/routes.rb** and replace its contents with the following</span></span>

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. <span data-ttu-id="1295f-129">Ouvrez **./app/view/home/index.html.erb** et remplacez son contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="1295f-129">Open **./app/view/home/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. <span data-ttu-id="1295f-130">Ajoutez un fichier PNG nommé **no-profile-photo.png** dans le répertoire **./app/assets/images.**</span><span class="sxs-lookup"><span data-stu-id="1295f-130">Add a PNG file named **no-profile-photo.png** in the **./app/assets/images** directory.</span></span>

1. <span data-ttu-id="1295f-131">Enregistrez toutes vos modifications et redémarrez le serveur.</span><span class="sxs-lookup"><span data-stu-id="1295f-131">Save all of your changes and restart the server.</span></span> <span data-ttu-id="1295f-132">L'application doit maintenant avoir une apparence très différente.</span><span class="sxs-lookup"><span data-stu-id="1295f-132">Now, the app should look very different.</span></span>

    ![Capture d’écran de la page d’accueil repensée](./images/create-app-01.png)
