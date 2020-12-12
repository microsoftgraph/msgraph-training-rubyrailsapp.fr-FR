<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="b23a7-101">Dans cet exercice, vous allez étendre l’application de l’exercice précédent pour prendre en charge l’authentification avec Azure AD.</span><span class="sxs-lookup"><span data-stu-id="b23a7-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="b23a7-102">Cela est nécessaire pour obtenir le jeton d’accès OAuth nécessaire pour appeler Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="b23a7-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="b23a7-103">Dans cette étape, vous allez intégrer la marque [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) dans l’application et créer une stratégie omniauth personnalisée.</span><span class="sxs-lookup"><span data-stu-id="b23a7-103">In this step you will integrate the [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) gem into the application, and create a custom OmniAuth strategy.</span></span>

1. <span data-ttu-id="b23a7-104">Créez un fichier distinct pour conserver votre ID d’application et votre clé secrète.</span><span class="sxs-lookup"><span data-stu-id="b23a7-104">Create a separate file to hold your app ID and secret.</span></span> <span data-ttu-id="b23a7-105">Créez un fichier appelé `oauth_environment_variables.rb` dans le dossier **./config** et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="b23a7-105">Create a new file called `oauth_environment_variables.rb` in the **./config** folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/oauth_environment_variables.rb.example":::

1. <span data-ttu-id="b23a7-106">Remplacez `YOUR_APP_ID_HERE` par l’ID de l’application dans le portail d’inscription de l’application et remplacez `YOUR_APP_SECRET_HERE` par le mot de passe que vous avez généré.</span><span class="sxs-lookup"><span data-stu-id="b23a7-106">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="b23a7-107">Si vous utilisez le contrôle de code source tel que git, il est maintenant recommandé d’exclure le `oauth_environment_variables.rb` fichier du contrôle de code source afin d’éviter une fuite accidentelle de votre ID d’application et de votre mot de passe.</span><span class="sxs-lookup"><span data-stu-id="b23a7-107">If you're using source control such as git, now would be a good time to exclude the `oauth_environment_variables.rb` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

1. <span data-ttu-id="b23a7-108">Ouvrez **./config/Environment.RB** et ajoutez le code suivant avant la `Rails.application.initialize!` ligne.</span><span class="sxs-lookup"><span data-stu-id="b23a7-108">Open **./config/environment.rb** and add the following code before the `Rails.application.initialize!` line.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/environment.rb" id="LoadOAuthSettingsSnippet" highlight="4-6":::

## <a name="setup-omniauth"></a><span data-ttu-id="b23a7-109">Programme d’installation OmniAuth</span><span class="sxs-lookup"><span data-stu-id="b23a7-109">Setup OmniAuth</span></span>

<span data-ttu-id="b23a7-110">Vous avez déjà installé le `omniauth-oauth2` GEM, mais pour le faire fonctionner avec les points de terminaison OAuth Azure, vous devez [créer une stratégie OAuth2](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy).</span><span class="sxs-lookup"><span data-stu-id="b23a7-110">You've already installed the `omniauth-oauth2` gem, but in order to make it work with the Azure OAuth endpoints, you need to [create an OAuth2 strategy](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy).</span></span> <span data-ttu-id="b23a7-111">Il s’agit d’une classe Ruby qui définit les paramètres permettant de passer des demandes OAuth au fournisseur Azure.</span><span class="sxs-lookup"><span data-stu-id="b23a7-111">This is a Ruby class that defines the parameters for making OAuth requests to the Azure provider.</span></span>

1. <span data-ttu-id="b23a7-112">Créez un fichier appelé `microsoft_graph_auth.rb` dans le dossier **./lib**' \* \*, et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="b23a7-112">Create a new file called `microsoft_graph_auth.rb` in the **./lib**\`\*\* folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/lib/microsoft_graph_auth.rb" id="AuthStrategySnippet":::

    <span data-ttu-id="b23a7-113">Prenez un moment pour examiner ce que fait ce code.</span><span class="sxs-lookup"><span data-stu-id="b23a7-113">Take a moment to review what this code does.</span></span>

    - <span data-ttu-id="b23a7-114">Il définit le `client_options` pour spécifier les points de terminaison de la plateforme d’identité Microsoft.</span><span class="sxs-lookup"><span data-stu-id="b23a7-114">It sets the `client_options` to specify the Microsoft identity platform endpoints.</span></span>
    - <span data-ttu-id="b23a7-115">Il spécifie que le `scope` paramètre doit être envoyé pendant la phase autoriser.</span><span class="sxs-lookup"><span data-stu-id="b23a7-115">It specifies that the `scope` parameter should be sent during the authorize phase.</span></span>
    - <span data-ttu-id="b23a7-116">Il mappe la `id` propriété de l’utilisateur en tant qu’ID unique pour l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="b23a7-116">It maps the `id` property of the user as the unique ID for the user.</span></span>
    - <span data-ttu-id="b23a7-117">Il utilise le jeton d’accès pour récupérer le profil de l’utilisateur à partir de Microsoft Graph pour remplir le `raw_info` hachage.</span><span class="sxs-lookup"><span data-stu-id="b23a7-117">It uses the access token to retrieve the user's profile from Microsoft Graph to fill in the `raw_info` hash.</span></span>
    - <span data-ttu-id="b23a7-118">Il remplace l’URL de rappel pour s’assurer qu’elle correspond au rappel enregistré dans le portail d’inscription des applications.</span><span class="sxs-lookup"><span data-stu-id="b23a7-118">It overrides the callback URL to ensure that it matches the registered callback in the app registration portal.</span></span>

1. <span data-ttu-id="b23a7-119">Créez un fichier appelé `omniauth_graph.rb` dans le dossier **./config/Initializers** et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="b23a7-119">Create a new file called `omniauth_graph.rb` in the **./config/initializers** folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/omniauth_graph.rb" id="ConfigureOmniAuthSnippet":::

    <span data-ttu-id="b23a7-120">Ce code s’exécute au démarrage de l’application.</span><span class="sxs-lookup"><span data-stu-id="b23a7-120">This code will execute when the app starts.</span></span> <span data-ttu-id="b23a7-121">Il charge le middleware OmniAuth avec le `microsoft_graph_auth` fournisseur, configuré avec les variables d’environnement définies dans **oauth_environment_variables. RB**.</span><span class="sxs-lookup"><span data-stu-id="b23a7-121">It loads up the OmniAuth middleware with the `microsoft_graph_auth` provider, configured with the environment variables set in **oauth_environment_variables.rb**.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="b23a7-122">Implémentation de la connexion</span><span class="sxs-lookup"><span data-stu-id="b23a7-122">Implement sign-in</span></span>

<span data-ttu-id="b23a7-123">Maintenant que l’intergiciel OmniAuth est configuré, vous pouvez passer à l’ajout d’une connexion à l’application.</span><span class="sxs-lookup"><span data-stu-id="b23a7-123">Now that the OmniAuth middleware is configured, you can move on to adding sign-in to the app.</span></span>

1. <span data-ttu-id="b23a7-124">Exécutez la commande suivante dans votre interface CLI pour générer un contrôleur de connexion et de déconnexion.</span><span class="sxs-lookup"><span data-stu-id="b23a7-124">Run the following command in your CLI to generate a controller for sign-in and sign-out.</span></span>

    ```Shell
    rails generate controller Auth
    ```

1. <span data-ttu-id="b23a7-125">Open **./app/controllers/auth_controller. RB**.</span><span class="sxs-lookup"><span data-stu-id="b23a7-125">Open **./app/controllers/auth_controller.rb**.</span></span> <span data-ttu-id="b23a7-126">Ajoutez une méthode de rappel à la `AuthController` classe.</span><span class="sxs-lookup"><span data-stu-id="b23a7-126">Add a callback method to the `AuthController` class.</span></span> <span data-ttu-id="b23a7-127">Cette méthode est appelée par le middleware OmniAuth une fois que le flux OAuth est terminé.</span><span class="sxs-lookup"><span data-stu-id="b23a7-127">This method will be called by the OmniAuth middleware once the OAuth flow is complete.</span></span>

    ```ruby
    def callback
      # Access the authentication hash for omniauth
      data = request.env['omniauth.auth']

      # Temporary for testing!
      render json: data.to_json
    end
    ```

    <span data-ttu-id="b23a7-128">Pour l’instant, tout ceci est rendu le hachage fourni par OmniAuth.</span><span class="sxs-lookup"><span data-stu-id="b23a7-128">For now all this does is render the hash provided by OmniAuth.</span></span> <span data-ttu-id="b23a7-129">Vous l’utiliserez pour vérifier que la connexion fonctionne avant de poursuivre.</span><span class="sxs-lookup"><span data-stu-id="b23a7-129">You'll use this to verify that sign-in is working before moving on.</span></span>

1. <span data-ttu-id="b23a7-130">Ajoutez les itinéraires à **./config/routes.RB**.</span><span class="sxs-lookup"><span data-stu-id="b23a7-130">Add the routes to **./config/routes.rb**.</span></span>

    ```ruby
    # Add route for OmniAuth callback
    match '/auth/:provider/callback', :to => 'auth#callback', :via => [:get, :post]
    ```

1. <span data-ttu-id="b23a7-131">Démarrez le serveur et accédez à `https://localhost:3000` .</span><span class="sxs-lookup"><span data-stu-id="b23a7-131">Start the server and browse to `https://localhost:3000`.</span></span> <span data-ttu-id="b23a7-132">Cliquez sur le bouton de connexion. vous serez redirigé vers `https://login.microsoftonline.com`.</span><span class="sxs-lookup"><span data-stu-id="b23a7-132">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="b23a7-133">Connectez-vous avec votre compte Microsoft et acceptez les autorisations demandées.</span><span class="sxs-lookup"><span data-stu-id="b23a7-133">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="b23a7-134">Le navigateur redirige vers l’application en affichant le hachage généré par OmniAuth.</span><span class="sxs-lookup"><span data-stu-id="b23a7-134">The browser redirects to the app, showing the hash generated by OmniAuth.</span></span>

    ```json
    {
      "provider": "microsoft_graph_auth",
      "uid": "eb52b3b2-c4ac-4b4f-bacd-d5f7ece55df0",
      "info": {
        "name": null
      },
      "credentials": {
        "token": "eyJ0eXAi...",
        "refresh_token": "OAQABAAA...",
        "expires_at": 1529517383,
        "expires": true
      },
      "extra": {
        "raw_info": {
          "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users(displayName,mail,mailboxSettings,userPrincipalName)/$entity",
          "displayName": "Lynne Robbins",
          "mail": "LynneR@contoso.OnMicrosoft.com",
          "userPrincipalName": "LynneR@contoso.OnMicrosoft.com",
          "id": "d294e784-840e-4f9f-bb1e-95c0a75f2f18@2d18179c-4386-4cbd-8891-7fd867c4f62e",
          "mailboxSettings": {
            "archiveFolder": "AAMkAGI2...",
            "timeZone": "Pacific Standard Time",
            "delegateMeetingMessageDeliveryOptions": "sendToDelegateOnly",
            "dateFormat": "M/d/yyyy",
            "timeFormat": "h:mm tt",
            "automaticRepliesSetting": {
              "status": "disabled",
              "externalAudience": "all",
              "internalReplyMessage": "",
              "externalReplyMessage": "",
              "scheduledStartDateTime": {
                "dateTime": "2020-12-09T17:00:00.0000000",
                "timeZone": "UTC"
              },
              "scheduledEndDateTime": {
                "dateTime": "2020-12-10T17:00:00.0000000",
                "timeZone": "UTC"
              }
            },
            "language": {
              "locale": "en-US",
              "displayName": "English (United States)"
            },
            "workingHours": {
              "daysOfWeek": [
                "monday",
                "tuesday",
                "wednesday",
                "thursday",
                "friday"
              ],
              "startTime": "08:00:00.0000000",
              "endTime": "17:00:00.0000000",
              "timeZone": {
                "name": "Pacific Standard Time"
              }
            }
          }
        }
      }
    }
    ```

## <a name="storing-the-tokens"></a><span data-ttu-id="b23a7-135">Stockage des jetons</span><span class="sxs-lookup"><span data-stu-id="b23a7-135">Storing the tokens</span></span>

<span data-ttu-id="b23a7-136">Maintenant que vous pouvez obtenir des jetons, nous vous conseillons d’implémenter un moyen de les stocker dans l’application.</span><span class="sxs-lookup"><span data-stu-id="b23a7-136">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="b23a7-137">Étant donné qu’il s’agit d’un exemple d’application, pour des raisons de simplicité, vous les stockerez dans la session.</span><span class="sxs-lookup"><span data-stu-id="b23a7-137">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="b23a7-138">Une application réelle utilise une solution de stockage sécurisé plus fiable, comme une base de données.</span><span class="sxs-lookup"><span data-stu-id="b23a7-138">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

1. <span data-ttu-id="b23a7-139">Open **./app/controllers/application_controller. RB**.</span><span class="sxs-lookup"><span data-stu-id="b23a7-139">Open **./app/controllers/application_controller.rb**.</span></span> <span data-ttu-id="b23a7-140">Ajoutez la méthode suivante à la classe `ApplicationController`.</span><span class="sxs-lookup"><span data-stu-id="b23a7-140">Add the following method to the `ApplicationController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="SaveInSessionSnippet":::

    <span data-ttu-id="b23a7-141">La méthode prend le hachage OmniAuth en tant que paramètre et extrait les bits pertinents des informations, puis il les stocke dans la session.</span><span class="sxs-lookup"><span data-stu-id="b23a7-141">The method takes the OmniAuth hash as a parameter and extracts the relevant bits of information, then stores that in the session.</span></span>

1. <span data-ttu-id="b23a7-142">Ajoutez des fonctions d’accesseur à la `ApplicationController` classe pour récupérer le nom d’utilisateur, l’adresse de messagerie et le jeton d’accès en dehors de la session.</span><span class="sxs-lookup"><span data-stu-id="b23a7-142">Add accessor functions to the `ApplicationController` class to retrieve the user name, email address, and access token back out of the session.</span></span>

    ```ruby
    def user_name
      session[:user_name]
    end

    def user_email
      session[:user_email]
    end

    def user_timezone
      session[:user_timezone]
    end

    def access_token
      session[:graph_token_hash][:token]
    end
    ```

1. <span data-ttu-id="b23a7-143">Ajoutez le code suivant à la `ApplicationController` classe qui s’exécutera avant le traitement d’une action.</span><span class="sxs-lookup"><span data-stu-id="b23a7-143">Add the following code to the `ApplicationController` class that will run before any action is processed.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="BeforeActionSnippet":::

    <span data-ttu-id="b23a7-144">Cette méthode définit les variables que la mise en page (dans **application.html. Erb**) utilise pour afficher les informations de l’utilisateur dans la barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="b23a7-144">This method sets the variables that the layout (in **application.html.erb**) uses to show the user's information in the nav bar.</span></span> <span data-ttu-id="b23a7-145">En l’ajoutant ici, il n’est pas nécessaire d’ajouter ce code dans chaque action de contrôleur.</span><span class="sxs-lookup"><span data-stu-id="b23a7-145">By adding it here, you don't have to add this code in every single controller action.</span></span> <span data-ttu-id="b23a7-146">Toutefois, cette opération s’exécute également pour les actions dans le `AuthController` , ce qui n’est pas optimal.</span><span class="sxs-lookup"><span data-stu-id="b23a7-146">However, this will also run for actions in the `AuthController`, which isn't optimal.</span></span>

1. <span data-ttu-id="b23a7-147">Ajoutez le code suivant à la `AuthController` classe dans **./app/Controllers/auth_controller. RB** pour ignorer l’action avant.</span><span class="sxs-lookup"><span data-stu-id="b23a7-147">Add the following code to the `AuthController` class in **./app/controllers/auth_controller.rb** to skip the before action.</span></span>

    ```ruby
    skip_before_action :set_user
    ```

1. <span data-ttu-id="b23a7-148">Mettez à jour la `callback` fonction dans la `AuthController` classe pour stocker les jetons dans la session et redirigez la redirection vers la page principale.</span><span class="sxs-lookup"><span data-stu-id="b23a7-148">Update the `callback` function in the `AuthController` class to store the tokens in the session and redirect back to the main page.</span></span> <span data-ttu-id="b23a7-149">Remplacez la fonction `callback` existante par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="b23a7-149">Replace the existing `callback` function with the following.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="CallbackSnippet":::

## <a name="implement-sign-out"></a><span data-ttu-id="b23a7-150">Mettre en œuvre la déconnexion</span><span class="sxs-lookup"><span data-stu-id="b23a7-150">Implement sign-out</span></span>

<span data-ttu-id="b23a7-151">Avant de tester cette nouvelle fonctionnalité, ajoutez une méthode pour vous déconnecter.</span><span class="sxs-lookup"><span data-stu-id="b23a7-151">Before you test this new feature, add a way to sign out.</span></span>

1. <span data-ttu-id="b23a7-152">Ajoutez l’action suivante à la `AuthController` classe.</span><span class="sxs-lookup"><span data-stu-id="b23a7-152">Add the following action to the `AuthController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="SignOutSnippet":::

1. <span data-ttu-id="b23a7-153">Ajoutez cette action à **./config/routes.RB**.</span><span class="sxs-lookup"><span data-stu-id="b23a7-153">Add this action to **./config/routes.rb**.</span></span>

    ```ruby
    get 'auth/signout'
    ```

1. <span data-ttu-id="b23a7-154">Redémarrez le serveur et suivez le processus de connexion.</span><span class="sxs-lookup"><span data-stu-id="b23a7-154">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="b23a7-155">Vous devez revenir sur la page d’accueil, mais l’interface utilisateur doit changer pour indiquer que vous êtes connecté.</span><span class="sxs-lookup"><span data-stu-id="b23a7-155">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Capture d’écran de la page d’accueil après la connexion](./images/add-aad-auth-01.png)

1. <span data-ttu-id="b23a7-157">Cliquez sur Avatar de l’utilisateur dans le coin supérieur droit pour accéder au lien **déconnexion** .</span><span class="sxs-lookup"><span data-stu-id="b23a7-157">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="b23a7-158">Le fait de cliquer sur **Se déconnecter** réinitialise la session et vous ramène à la page d’accueil.</span><span class="sxs-lookup"><span data-stu-id="b23a7-158">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Capture d’écran du menu déroulant avec le lien de déconnexion](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="b23a7-160">Actualisation des jetons</span><span class="sxs-lookup"><span data-stu-id="b23a7-160">Refreshing tokens</span></span>

<span data-ttu-id="b23a7-161">Si vous examinez attentivement le hachage généré par OmniAuth, vous remarquerez qu’il y a deux jetons dans le hachage : `token` et `refresh_token` .</span><span class="sxs-lookup"><span data-stu-id="b23a7-161">If you look closely at the hash generated by OmniAuth, you'll notice there are two tokens in the hash: `token` and `refresh_token`.</span></span> <span data-ttu-id="b23a7-162">La valeur dans `token` est le jeton d’accès, qui est envoyé dans l' `Authorization` en-tête des appels d’API.</span><span class="sxs-lookup"><span data-stu-id="b23a7-162">The value in `token` is the access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="b23a7-163">Il s’agit du jeton qui permet à l’application d’accéder à Microsoft Graph pour le compte de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="b23a7-163">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="b23a7-164">Cependant, ce jeton est de courte durée.</span><span class="sxs-lookup"><span data-stu-id="b23a7-164">However, this token is short-lived.</span></span> <span data-ttu-id="b23a7-165">Le jeton expire une heure après son émission.</span><span class="sxs-lookup"><span data-stu-id="b23a7-165">The token expires an hour after it is issued.</span></span> <span data-ttu-id="b23a7-166">C’est ici que la `refresh_token` valeur devient utile.</span><span class="sxs-lookup"><span data-stu-id="b23a7-166">This is where the `refresh_token` value becomes useful.</span></span> <span data-ttu-id="b23a7-167">Le jeton d’actualisation permet à l’application de demander un nouveau jeton d’accès sans obliger l’utilisateur à se reconnecter.</span><span class="sxs-lookup"><span data-stu-id="b23a7-167">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="b23a7-168">Mettez à jour le code de gestion des jetons pour implémenter l’actualisation des jetons.</span><span class="sxs-lookup"><span data-stu-id="b23a7-168">Update the token management code to implement token refresh.</span></span>

1. <span data-ttu-id="b23a7-169">Ouvrez **./app/controllers/application_controller. RB** et ajoutez les `require` instructions suivantes en haut :</span><span class="sxs-lookup"><span data-stu-id="b23a7-169">Open **./app/controllers/application_controller.rb** and add the following `require` statements at the top:</span></span>

    ```ruby
    require 'microsoft_graph_auth'
    require 'oauth2'
    ```

1. <span data-ttu-id="b23a7-170">Ajoutez la méthode suivante à la classe `ApplicationController`.</span><span class="sxs-lookup"><span data-stu-id="b23a7-170">Add the following method to the `ApplicationController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="RefreshTokensSnippet":::

    <span data-ttu-id="b23a7-171">Cette méthode utilise la marque [oauth2](https://github.com/oauth-xx/oauth2) (une dépendance de la `omniauth-oauth2` gemme) pour actualiser les jetons et met à jour la session.</span><span class="sxs-lookup"><span data-stu-id="b23a7-171">This method uses the [oauth2](https://github.com/oauth-xx/oauth2) gem (a dependency of the `omniauth-oauth2` gem) to refresh the tokens, and updates the session.</span></span>

1. <span data-ttu-id="b23a7-172">Remplacez la `access_token` méthode actuelle par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="b23a7-172">Replace the current `access_token` method with the following.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="AccessTokenSnippet":::

    <span data-ttu-id="b23a7-173">Au lieu de simplement renvoyer le jeton de la session, il commence par vérifier s’il est proche de la date d’expiration.</span><span class="sxs-lookup"><span data-stu-id="b23a7-173">Instead of just returning the token from the session, it will first check if it is close to expiration.</span></span> <span data-ttu-id="b23a7-174">Si c’est le cas, il est actualisé avant de renvoyer le jeton.</span><span class="sxs-lookup"><span data-stu-id="b23a7-174">If it is, then it will refresh before returning the token.</span></span>
