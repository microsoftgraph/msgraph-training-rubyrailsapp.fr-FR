<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez étendre l’application de l’exercice précédent pour prendre en charge l’authentification avec Azure AD. Cela est nécessaire pour obtenir le jeton d’accès OAuth nécessaire pour appeler Microsoft Graph. Dans cette étape, vous allez intégrer la marque [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) dans l’application et créer une stratégie omniauth personnalisée.

1. Créez un fichier distinct pour conserver votre ID d’application et votre clé secrète. Créez un fichier appelé `oauth_environment_variables.rb` dans le dossier **./config** et ajoutez le code suivant.

    :::code language="ruby" source="../demo/graph-tutorial/config/oauth_environment_variables.rb.example":::

1. Remplacez `YOUR_APP_ID_HERE` par l’ID de l’application dans le portail d’inscription de `YOUR_APP_SECRET_HERE` l’application et remplacez par le mot de passe que vous avez généré.

    > [!IMPORTANT]
    > Si vous utilisez le contrôle de code source tel que git, il est maintenant recommandé d’exclure le `oauth_environment_variables.rb` fichier du contrôle de code source afin d’éviter une fuite accidentelle de votre ID d’application et de votre mot de passe.

1. Ouvrez **./config/Environment.RB** et ajoutez le code suivant avant la `Rails.application.initialize!` ligne.

    :::code language="ruby" source="../demo/graph-tutorial/config/environment.rb" id="LoadOAuthSettingsSnippet" highlight="4-6":::

## <a name="setup-omniauth"></a>Programme d’installation OmniAuth

Vous avez déjà installé le `omniauth-oauth2` GEM, mais pour le faire fonctionner avec les points de terminaison OAuth Azure, vous devez [créer une stratégie OAuth2](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy). Il s’agit d’une classe Ruby qui définit les paramètres permettant de passer des demandes OAuth au fournisseur Azure.

1. Créez un fichier appelé `microsoft_graph_auth.rb` dans le dossier **./lib**' * *, et ajoutez le code suivant.

    :::code language="ruby" source="../demo/graph-tutorial/lib/microsoft_graph_auth.rb" id="AuthStrategySnippet":::

    Prenez un moment pour examiner ce que fait ce code.

    - Il définit le `client_options` pour spécifier les points de terminaison de la plateforme d’identité Microsoft.
    - Il spécifie que `scope` le paramètre doit être envoyé pendant la phase autoriser.
    - Il mappe la `id` propriété de l’utilisateur en tant qu’ID unique pour l’utilisateur.
    - Il utilise le jeton d’accès pour récupérer le profil de l’utilisateur à partir de Microsoft Graph `raw_info` pour remplir le hachage.
    - Il remplace l’URL de rappel pour s’assurer qu’elle correspond au rappel enregistré dans le portail d’inscription des applications.

1. Créez un fichier appelé `omniauth_graph.rb` dans le dossier **./config/Initializers** et ajoutez le code suivant.

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/omniauth_graph.rb" id="ConfigureOmniAuthSnippet":::

    Ce code s’exécute au démarrage de l’application. Il charge le middleware OmniAuth avec le `microsoft_graph_auth` fournisseur, configuré avec les variables d’environnement définies dans **oauth_environment_variables. RB**.

## <a name="implement-sign-in"></a>Implémentation de la connexion

Maintenant que l’intergiciel OmniAuth est configuré, vous pouvez passer à l’ajout d’une connexion à l’application.

1. Exécutez la commande suivante dans votre interface CLI pour générer un contrôleur de connexion et de déconnexion.

    ```Shell
    rails generate controller Auth
    ```

1. Open **./app/controllers/auth_controller. RB**. Ajoutez une méthode de rappel à `AuthController` la classe. Cette méthode est appelée par le middleware OmniAuth une fois que le flux OAuth est terminé.

    ```ruby
    def callback
      # Access the authentication hash for omniauth
      data = request.env['omniauth.auth']

      # Temporary for testing!
      render json: data.to_json
    end
    ```

    Pour l’instant, tout ceci est rendu le hachage fourni par OmniAuth. Vous l’utiliserez pour vérifier que la connexion fonctionne avant de poursuivre.

1. Ajoutez les itinéraires à **./config/routes.RB**.

    ```ruby
    # Add route for OmniAuth callback
    match '/auth/:provider/callback', to: 'auth#callback', via: [:get, :post]
    ```

1. Démarrez le serveur et accédez à `https://localhost:3000`. Cliquez sur le bouton de connexion. vous serez redirigé vers `https://login.microsoftonline.com`. Connectez-vous avec votre compte Microsoft et acceptez les autorisations demandées. Le navigateur redirige vers l’application en affichant le hachage généré par OmniAuth.

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
          "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
          "id": "eb52b3b2-c4ac-4b4f-bacd-d5f7ece55df0",
          "businessPhones": [
            "+1 425 555 0109"
          ],
          "displayName": "Adele Vance",
          "givenName": "Adele",
          "jobTitle": "Retail Manager",
          "mail": "AdeleV@contoso.onmicrosoft.com",
          "mobilePhone": null,
          "officeLocation": "18/2111",
          "preferredLanguage": "en-US",
          "surname": "Vance",
          "userPrincipalName": "AdeleV@contoso.onmicrosoft.com"
        }
      }
    }
    ```

## <a name="storing-the-tokens"></a>Stockage des jetons

Maintenant que vous pouvez obtenir des jetons, nous vous conseillons d’implémenter un moyen de les stocker dans l’application. Étant donné qu’il s’agit d’un exemple d’application, pour des raisons de simplicité, vous les stockerez dans la session. Une application réelle utilise une solution de stockage sécurisé plus fiable, comme une base de données.

1. Open **./app/controllers/application_controller. RB**. Ajoutez la méthode suivante à la classe `ApplicationController`.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="SaveInSessionSnippet":::

    La méthode prend le hachage OmniAuth en tant que paramètre et extrait les bits pertinents des informations, puis il les stocke dans la session.

1. Ajoutez des fonctions d’accesseur à la `ApplicationController` classe pour récupérer le nom d’utilisateur, l’adresse de messagerie et le jeton d’accès en dehors de la session.

    ```ruby
    def user_name
      session[:user_name]
    end

    def user_email
      session[:user_email]
    end

    def access_token
      session[:graph_token_hash][:token]
    end
    ```

1. Ajoutez le code suivant à la `ApplicationController` classe qui s’exécutera avant le traitement d’une action.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="BeforeActionSnippet":::

    Cette méthode définit les variables que la mise en page (dans **application. html. Erb**) utilise pour afficher les informations de l’utilisateur dans la barre de navigation. En l’ajoutant ici, il n’est pas nécessaire d’ajouter ce code dans chaque action de contrôleur. Toutefois, cette opération s’exécute également pour les actions `AuthController`dans le, ce qui n’est pas optimal.

1. Ajoutez le code suivant à la `AuthController` classe dans **./app/Controllers/auth_controller. RB** pour ignorer l’action avant.

    ```ruby
    skip_before_action :set_user
    ```

1. Mettez à `callback` jour la fonction `AuthController` dans la classe pour stocker les jetons dans la session et redirigez la redirection vers la page principale. Remplacez la fonction `callback` existante par ce qui suit.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="CallbackSnippet":::

## <a name="implement-sign-out"></a>Mettre en œuvre la déconnexion

Avant de tester cette nouvelle fonctionnalité, ajoutez une méthode pour vous déconnecter.

1. Ajoutez l’action suivante à la `AuthController` classe.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="SignOutSnippet":::

1. Ajoutez cette action à **./config/routes.RB**.

    ```ruby
    get 'auth/signout'
    ```

1. Redémarrez le serveur et suivez le processus de connexion. Vous devez revenir sur la page d’accueil, mais l’interface utilisateur doit changer pour indiquer que vous êtes connecté.

    ![Capture d’écran de la page d’accueil après la connexion](./images/add-aad-auth-01.png)

1. Cliquez sur Avatar de l’utilisateur dans le coin supérieur droit pour accéder au lien **déconnexion** . Le fait de cliquer sur **Se déconnecter** réinitialise la session et vous ramène à la page d’accueil.

    ![Capture d’écran du menu déroulant avec le lien de déconnexion](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Actualisation des jetons

Si vous examinez attentivement le hachage généré par OmniAuth, vous remarquerez qu’il y a deux jetons dans le `token` hachage `refresh_token`: et. La valeur dans `token` est le jeton d’accès, qui est envoyé dans `Authorization` l’en-tête des appels d’API. Il s’agit du jeton qui permet à l’application d’accéder à Microsoft Graph pour le compte de l’utilisateur.

Cependant, ce jeton est de courte durée. Le jeton expire une heure après son émission. C’est ici que `refresh_token` la valeur devient utile. Le jeton d’actualisation permet à l’application de demander un nouveau jeton d’accès sans obliger l’utilisateur à se reconnecter. Mettez à jour le code de gestion des jetons pour implémenter l’actualisation des jetons.

1. Ouvrez **./app/controllers/application_controller. RB** et ajoutez les instructions `require` suivantes en haut :

    ```ruby
    require 'microsoft_graph_auth'
    require 'oauth2'
    ```

1. Ajoutez la méthode suivante à la classe `ApplicationController`.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="RefreshTokensSnippet":::

    Cette méthode utilise la marque [oauth2](https://github.com/oauth-xx/oauth2) (une dépendance de la `omniauth-oauth2` gemme) pour actualiser les jetons et met à jour la session.

1. Remplacez la méthode `access_token` actuelle par ce qui suit.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="AccessTokenSnippet":::

    Au lieu de simplement renvoyer le jeton de la session, il commence par vérifier s’il est proche de la date d’expiration. Si c’est le cas, il est actualisé avant de renvoyer le jeton.
