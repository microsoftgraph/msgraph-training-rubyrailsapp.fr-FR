<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez incorporer Microsoft Graph dans l’application. Pour cette application, vous allez utiliser la gemme [HTTParty](https://github.com/jnunemaker/httparty) pour passer des appels à Microsoft Graph.

## <a name="create-a-graph-helper"></a>Créer une application d’assistance Graph

1. Créez une assistance pour gérer tous vos appels d’API. Exécutez la commande suivante dans votre interface CLI pour générer l’assistance.

    ```Shell
    rails generate helper Graph
    ```

1. Ouvrez **./app/helpers/graph_helper. RB** et remplacez le contenu par ce qui suit.

    ```ruby
    require 'httparty'

    # Graph API helper methods
    module GraphHelper
      GRAPH_HOST = 'https://graph.microsoft.com'.freeze

      def make_api_call(endpoint, token, params = nil)
        headers = {
          Authorization: "Bearer #{token}"
        }

        query = params || {}

        HTTParty.get "#{GRAPH_HOST}#{endpoint}",
                     headers: headers,
                     query: query
      end
    end
    ```

Prenez un moment pour examiner ce que fait ce code. Il effectue une requête GET simple via la `httparty` marque pour le point de terminaison demandé. Il envoie le jeton d’accès dans `Authorization` l’en-tête et inclut tous les paramètres de requête qui sont passés.

Par exemple, pour utiliser la `make_api_call` méthode permettant d’effectuer une opération `https://graph.microsoft.com/v1.0/me?$select=displayName`atteindre, vous pouvez l’appeler comme suit :

```ruby
make_api_call `/v1.0/me`, access_token, { '$select': 'displayName' }
```

Vous la créerez ultérieurement lorsque vous implémenterez davantage de fonctionnalités Microsoft Graph dans l’application.

## <a name="get-calendar-events-from-outlook"></a>Récupérer les événements de calendrier à partir d’Outlook

1. Dans votre interface CLI, exécutez la commande suivante pour ajouter un nouveau contrôleur.

    ```Shell
    rails generate controller Calendar index
    ```

1. Ajoutez le nouvel itinéraire à **./config/routes.RB**.

    ```ruby
    get 'calendar', to: 'calendar#index'
    ```

1. Ajoutez une nouvelle méthode à l’aide Graph pour [répertorier les événements de l’utilisateur](/graph/api/user-list-events?view=graph-rest-1.0). Ouvrez **./app/helpers/graph_helper. RB** et ajoutez la méthode suivante au `GraphHelper` module.

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    Que fait ce code ?

    - L’URL qui sera appelée est `/v1.0/me/events`.
    - Le `$select` paramètre limite les champs renvoyés pour chaque événement à ceux que notre vue utilisera réellement.
    - Le `$orderby` paramètre trie les résultats en fonction de la date et de l’heure de leur création, avec l’élément le plus récent en premier.
    - Pour une réponse réussie, elle renvoie le tableau d’éléments contenus dans la `value` clé.

1. Ouvrez **./app/controllers/calendar_controller. RB** et remplacez l’intégralité de son contenu par ce qui suit.

    ```ruby
    # Calendar controller
    class CalendarController < ApplicationController
      include GraphHelper

      def index
        @events = get_calendar_events access_token || []
        render json: @events
      rescue RuntimeError => e
        @errors = [
          {
            message: 'Microsoft Graph returned an error getting events.',
            debug: e
          }
        ]
      end
    end
    ```

1. Redémarrez le serveur. Connectez-vous, puis cliquez sur le lien **calendrier** dans la barre de navigation. Si tout fonctionne, vous devriez voir une image mémoire JSON des événements dans le calendrier de l’utilisateur.

## <a name="display-the-results"></a>Afficher les résultats

À présent, vous pouvez ajouter du code HTML pour afficher les résultats de manière plus conviviale.

1. Ouvrez **./app/views/Calendar/index.html.Erb** et remplacez son contenu par ce qui suit.

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    Cela permet de parcourir une collection d’événements et d’ajouter une ligne de tableau pour chacun d’eux.

1. Supprimez `render json: @events` la ligne de `index` l’action dans **./app/Controllers/calendar_controller. RB**.

1. Actualisez la page et l’application doit maintenant afficher un tableau d’événements.

    ![Capture d’écran du tableau des événements](./images/add-msgraph-01.png)
