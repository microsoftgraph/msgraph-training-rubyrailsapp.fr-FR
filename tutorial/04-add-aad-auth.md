<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez étendre l’application de l’exercice précédent pour prendre en charge l’authentification avec Azure AD. Cela est nécessaire pour obtenir le jeton d’accès OAuth nécessaire pour appeler Microsoft Graph. Dans cette étape, vous allez intégrer la marque [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) dans l’application et créer une stratégie omniauth personnalisée.

Tout d’abord, créez un fichier distinct pour conserver votre ID d’application et votre clé secrète. Créez un fichier appelé `oauth_environment_variables.rb` dans le `./config` dossier et ajoutez le code suivant.

```ruby
ENV['AZURE_APP_ID'] = 'YOUR_APP_ID_HERE'
ENV['AZURE_APP_SECRET'] = 'YOUR_APP_SECRET_HERE'
ENV['AZURE_SCOPES'] = 'openid profile email offline_access user.read calendars.read'
```

Remplacez `YOUR_APP_ID_HERE` par l’ID de l’application dans le portail d’inscription de `YOUR_APP_SECRET_HERE` l’application et remplacez par le mot de passe que vous avez généré.

> [!IMPORTANT]
> Si vous utilisez le contrôle de code source tel que git, il est maintenant recommandé d’exclure le `oauth_environment_variables.rb` fichier du contrôle de code source afin d’éviter une fuite accidentelle de votre ID d’application et de votre mot de passe.

À présent, ajoutez du code pour charger ce fichier s’il est présent. Ouvrez le `./config/environment.rb` fichier et ajoutez le code suivant avant la `Rails.application.initialize!` ligne.

```ruby
# Load OAuth settings
oauth_environment_variables = File.join(Rails.root, 'config', 'oauth_environment_variables.rb')
load(oauth_environment_variables) if File.exist?(oauth_environment_variables)
```

## <a name="setup-omniauth"></a>Programme d’installation OmniAuth

Vous avez déjà installé le `omniauth-oauth2` GEM, mais pour le faire fonctionner avec les points de terminaison OAuth Azure, vous devez [créer une stratégie OAuth2](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy). Il s’agit d’une classe Ruby qui définit les paramètres permettant de passer des demandes OAuth au fournisseur Azure.

Créez un fichier appelé `microsoft_graph_auth.rb` dans le `./lib` dossier et ajoutez le code suivant.

```ruby
require 'omniauth-oauth2'

module OmniAuth
  module Strategies
    # Implements an OmniAuth strategy to get a Microsoft Graph
    # compatible token from Azure AD
    class MicrosoftGraphAuth < OmniAuth::Strategies::OAuth2
      option :name, :microsoft_graph_auth

      DEFAULT_SCOPE = 'openid email profile User.Read'.freeze

      # Configure the Microsoft identity platform endpoints
      option  :client_options,
              site:          'https://login.microsoftonline.com',
              authorize_url: '/common/oauth2/v2.0/authorize',
              token_url:     '/common/oauth2/v2.0/token'

      # Send the scope parameter during authorize
      option :authorize_options, [:scope]

      # Unique ID for the user is the id field
      uid { raw_info['id'] }

      # Get additional information after token is retrieved
      extra do
        {
          'raw_info' => raw_info
        }
      end

      def raw_info
        # Get user profile information from the /me endpoint
        @raw_info ||= access_token.get('https://graph.microsoft.com/v1.0/me').parsed
      end

      def authorize_params
        super.tap do |params|
          params['scope'.to_sym] = request.params['scope'] if request.params['scope']
          params[:scope] ||= DEFAULT_SCOPE
        end
      end

      # Override callback URL
      # OmniAuth by default passes the entire URL of the callback, including
      # query parameters. Azure fails validation because that doesn't match the
      # registered callback.
      def callback_url
        options[:redirect_uri] || (full_host + script_name + callback_path)
      end
    end
  end
end
```

Prenez un moment pour examiner ce que fait ce code.

- Il définit le `client_options` pour spécifier les points de terminaison de la plateforme d’identité Microsoft.
- Il spécifie que `scope` le paramètre doit être envoyé pendant la phase autoriser.
- Il mappe la `id` propriété de l’utilisateur en tant qu’ID unique pour l’utilisateur.
- Il utilise le jeton d’accès pour récupérer le profil de l’utilisateur à partir de Microsoft Graph `raw_info` pour remplir le hachage.
- Il remplace l’URL de rappel pour s’assurer qu’elle correspond au rappel enregistré dans le portail d’inscription des applications.

À présent que nous avons défini la stratégie, nous devons configurer OmniAuth pour l’utiliser. Créez un fichier appelé `omniauth_graph.rb` dans le `./config/initializers` dossier et ajoutez le code suivant.

```ruby
require 'microsoft_graph_auth'

Rails.application.config.middleware.use OmniAuth::Builder do
  provider :microsoft_graph_auth,
           ENV['AZURE_APP_ID'],
           ENV['AZURE_APP_SECRET'],
           scope: ENV['AZURE_SCOPES']
end
```

Ce code s’exécute au démarrage de l’application. Il charge le middleware OmniAuth avec le `microsoft_graph_auth` fournisseur, configuré avec les variables d’environnement définies dans. `oauth_environment_variables.rb`

## <a name="implement-sign-in"></a>Mettre en œuvre la connexion

Maintenant que l’intergiciel OmniAuth est configuré, vous pouvez passer à l’ajout d’une connexion à l’application. Exécutez la commande suivante dans votre interface CLI pour générer un contrôleur de connexion et de déconnexion.

```Shell
rails generate controller Auth
```

Ouvrez le fichier `./app/controllers/auth_controller.rb`. Ajoutez la méthode suivante à la classe `AuthController`.

```ruby
def signin
  redirect_to '/auth/microsoft_graph_auth'
end
```

Toutes les méthodes de cette méthode sont redirigées vers l’itinéraire que OmniAuth attend d’appeler notre stratégie personnalisée.

Ensuite, ajoutez une méthode de rappel à `AuthController` la classe. Cette méthode est appelée par le middleware OmniAuth une fois que le flux OAuth est terminé.

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Temporary for testing!
  render json: data.to_json
end
```

Pour l’instant, tout ceci est rendu le hachage fourni par OmniAuth. Nous allons l’utiliser pour vérifier que notre connexion fonctionne avant de poursuivre. Avant de tester, nous devons ajouter les itinéraires à `./config/routes.rb`.

```ruby
get 'auth/signin'

# Add route for OmniAuth callback
match '/auth/:provider/callback', to: 'auth#callback', via: [:get, :post]
```

Maintenant, mettez à jour les vues `signin` pour utiliser l’action. Ouvrez `./app/views/layouts/application.html.erb`. Remplacez la ligne `<a href="#" class="nav-link">Sign In</a>` par ce qui suit.

```html
<%= link_to "Sign In", {:controller => :auth, :action => :signin}, :class => "nav-link" %>
```

Ouvrez le `./app/views/home/index.html.erb` fichier et remplacez la `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` ligne par ce qui suit.

```html
<%= link_to "Click here to sign in", {:controller => :auth, :action => :signin}, :class => "btn btn-primary btn-large" %>
```

Démarrez le serveur et accédez à `https://localhost:3000`. Cliquez sur le bouton de connexion et vous devez être redirigé vers `https://login.microsoftonline.com`. Connectez-vous avec votre compte Microsoft et acceptez les autorisations demandées. Le navigateur redirige vers l’application en affichant le hachage généré par OmniAuth.

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

Maintenant que vous pouvez obtenir des jetons, il est temps de mettre en œuvre un moyen de les stocker dans l’application. Étant donné qu’il s’agit d’un exemple d’application, pour des raisons de simplicité, vous les stockerez dans la session. Une application réelle utiliserait une solution de stockage sécurisé plus fiable, comme une base de données.

Ouvrez le fichier `./app/controllers/application_controller.rb`. Vous allez ajouter toutes nos méthodes de gestion des jetons ici. Étant donné que tous les autres contrôleurs `ApplicationController` héritent de la classe, ils seront en mesure d’utiliser ces méthodes pour accéder aux jetons.

Ajoutez la méthode suivante à la classe `ApplicationController`. La méthode prend le hachage OmniAuth en tant que paramètre et extrait les bits pertinents des informations, puis il les stocke dans la session.

```ruby
def save_in_session(auth_hash)
  # Save the token info
  session[:graph_token_hash] = auth_hash.dig(:credentials)
  # Save the user's display name
  session[:user_name] = auth_hash.dig(:extra, :raw_info, :displayName)
  # Save the user's email address
  # Use the mail field first. If that's empty, fall back on
  # userPrincipalName
  session[:user_email] = auth_hash.dig(:extra, :raw_info, :mail) ||
                         auth_hash.dig(:extra, :raw_info, :userPrincipalName)
end
```

À présent, ajoutez des fonctions d’accès pour récupérer le nom d’utilisateur, l’adresse de messagerie et le jeton d’accès en dehors de la session.

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

Enfin, ajoutez du code qui s’exécutera avant le traitement d’une action.

```ruby
before_action :set_user

def set_user
  @user_name = user_name
  @user_email = user_email
end
```

Cette méthode définit les variables utilisées par la disposition ( `application.html.erb`en) pour afficher les informations de l’utilisateur dans la barre de navigation. En l’ajoutant ici, il n’est pas nécessaire d’ajouter ce code dans chaque action de contrôleur. Toutefois, cette opération s’exécute également pour les actions `AuthController`dans le, ce qui n’est pas optimal. Ajoutez le code suivant à la `AuthController` classe dans `./app/controllers/auth_controller.rb` pour ignorer l’action avant.

```ruby
skip_before_action :set_user
```

Ensuite, mettez à `callback` jour la fonction `AuthController` dans la classe pour stocker les jetons dans la session et rediriger vers la page principale. Remplacez la fonction `callback` existante par ce qui suit.

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Save the data in the session
  save_in_session data

  redirect_to root_url
end
```

## <a name="implement-sign-out"></a>Mettre en œuvre la déconnexion

Avant de tester cette nouvelle fonctionnalité, ajoutez une méthode pour vous déconnecter. Ajoutez l’action suivante à la `AuthController` classe.

```ruby
def signout
  reset_session
  redirect_to root_url
end
```

Ajoutez cette action à `./config/routes.rb`.

```ruby
get 'auth/signout'
```

Maintenant, mettez à jour l’affichage `signout` pour utiliser l’action. Ouvrez `./app/views/layouts/application.html.erb`. Remplacez la ligne `<a href="#" class="dropdown-item">Sign Out</a>` par :

```html
<%= link_to "Sign Out", {:controller => :auth, :action => :signout}, :class => "dropdown-item" %>
```

Redémarrez le serveur et suivez le processus de connexion. Vous devez revenir sur la page d’accueil, mais l’interface utilisateur doit changer pour indiquer que vous êtes connecté.

![Capture d’écran de la page d’accueil après la connexion](./images/add-aad-auth-01.png)

Cliquez sur Avatar de l’utilisateur dans le coin supérieur droit pour accéder au lien **déconnexion** . Cliquez sur **déconnexion** pour réinitialiser la session et revenir à la page d’accueil.

![Capture d’écran du menu déroulant avec le lien Déconnexion](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Actualisation des jetons

Si vous examinez attentivement le hachage généré par OmniAuth, vous remarquerez qu’il y a deux jetons dans le `token` hachage `refresh_token`: et. La valeur dans `token` est le jeton d’accès, qui est envoyé dans `Authorization` l’en-tête des appels d’API. Il s’agit du jeton qui permet à l’application d’accéder à Microsoft Graph pour le compte de l’utilisateur.

Toutefois, ce jeton est éphémère. Le jeton expire une heure après son émission. C’est ici que `refresh_token` la valeur devient utile. Le jeton d’actualisation permet à l’application de demander un nouveau jeton d’accès sans demander à l’utilisateur de se reconnecter. Mettez à jour le code de gestion des jetons pour implémenter l’actualisation des jetons.

Ouvrez `./app/controllers/application_controller.rb` et ajoutez les instructions `require` suivantes en haut :

```ruby
require 'microsoft_graph_auth'
require 'oauth2'
```

Ajoutez ensuite la méthode suivante à la `ApplicationController` classe.

```ruby
def refresh_tokens(token_hash)
  oauth_strategy = OmniAuth::Strategies::MicrosoftGraphAuth.new(
    nil, ENV['AZURE_APP_ID'], ENV['AZURE_APP_SECRET']
  )

  token = OAuth2::AccessToken.new(
    oauth_strategy.client, token_hash[:token],
    refresh_token: token_hash[:refresh_token]
  )

  # Refresh the tokens
  new_tokens = token.refresh!.to_hash.slice(:access_token, :refresh_token, :expires_at)

  # Rename token key
  new_tokens[:token] = new_tokens.delete :access_token

  # Store the new hash
  session[:graph_token_hash] = new_tokens
end
```

Cette méthode utilise la marque [oauth2](https://github.com/oauth-xx/oauth2) (une dépendance de la `omniauth-oauth2` gemme) pour actualiser les jetons et met à jour la session.

Maintenant, cette méthode doit être utilisée. Pour ce faire, rendez `access_token` l’accesseur `ApplicationController` de la classe un peu plus intelligent. Au lieu de simplement renvoyer le jeton de la session, il commence par vérifier s’il est proche de la date d’expiration. Si c’est le cas, il est actualisé avant de renvoyer le jeton. Remplacez la méthode `access_token` actuelle par ce qui suit.

```ruby
def access_token
  token_hash = session[:graph_token_hash]

  # Get the expiry time - 5 minutes
  expiry = Time.at(token_hash[:expires_at] - 300)

  if Time.now > expiry
    # Token expired, refresh
    new_hash = refresh_tokens token_hash
    new_hash[:token]
  else
    token_hash[:token]
  end
end
```
