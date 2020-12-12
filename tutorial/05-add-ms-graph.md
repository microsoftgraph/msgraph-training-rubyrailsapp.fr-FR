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

      def make_api_call(method, endpoint, token, headers = nil, params = nil, payload = nil)
        headers ||= {}
        headers[:Authorization] = "Bearer #{token}"
        headers[:Accept] = 'application/json'

        params ||= {}

        case method.upcase
        when 'GET'
          HTTParty.get "#{GRAPH_HOST}#{endpoint}",
                       :headers => headers,
                       :query => params
        when 'POST'
          headers['Content-Type'] = 'application/json'
          HTTParty.post "#{GRAPH_HOST}#{endpoint}",
                        :headers => headers,
                        :query => params,
                        :body => payload ? payload.to_json : nil
        else
          raise "HTTP method #{method.upcase} not implemented"
        end
      end
    end
    ```

Prenez un moment pour examiner ce que fait ce code. Il effectue une requête GET ou POST simple via la marque `httparty` pour le point de terminaison demandé. Il envoie le jeton d’accès dans l' `Authorization` en-tête et inclut tous les paramètres de requête qui sont passés.

Par exemple, pour utiliser la `make_api_call` méthode permettant d’effectuer une opération atteindre `https://graph.microsoft.com/v1.0/me?$select=displayName` , vous pouvez l’appeler comme suit :

```ruby
make_api_call 'GET', '/v1.0/me', access_token, { '$select': 'displayName' }
```

Vous la créerez ultérieurement lorsque vous implémenterez davantage de fonctionnalités Microsoft Graph dans l’application.

## <a name="get-calendar-events-from-outlook"></a>Récupérer les événements de calendrier à partir d’Outlook

1. Dans votre interface CLI, exécutez la commande suivante pour ajouter un nouveau contrôleur.

    ```Shell
    rails generate controller Calendar index new
    ```

1. Ajoutez le nouvel itinéraire à **./config/routes.RB**.

    ```ruby
    get 'calendar', :to => 'calendar#index'
    ```

1. Ajoutez une nouvelle méthode à l’aide Graph pour [obtenir un affichage Calendrier](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0). Ouvrez **./app/helpers/graph_helper. RB** et ajoutez la méthode suivante au `GraphHelper` module.

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    Que fait ce code ?

    - L’URL qui sera appelée est `/v1.0/me/calendarview`.
        - L' `Prefer: outlook.timezone` en-tête entraîne l’ajustement des heures de début et de fin dans les résultats sur le fuseau horaire de l’utilisateur.
        - Les `startDateTime` `endDateTime` paramètres et définissent le début et la fin de l’affichage.
        - Le `$select` paramètre limite les champs renvoyés pour chaque événement à ceux que l’affichage utilise réellement.
        - Le `$orderby` paramètre trie les résultats par heure de début.
        - Le `$top` paramètre limite les résultats à 50 événements.
    - Pour une réponse réussie, elle renvoie le tableau d’éléments contenus dans la `value` clé.

1. Ajoutez une nouvelle méthode à l’aide Graph pour rechercher un [identificateur de fuseau horaire IANA](https://www.iana.org/time-zones) en fonction du nom du fuseau horaire Windows. Cette opération est nécessaire, car Microsoft Graph peut renvoyer des fuseaux horaires sous la forme de noms de fuseaux horaires Windows et la classe Ruby **DateTime** requiert des identificateurs de fuseau horaire IANA.

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="ZoneMappingSnippet":::

1. Ouvrez **./app/controllers/calendar_controller. RB** et remplacez l’intégralité de son contenu par ce qui suit.

    ```ruby
    # Calendar controller
    class CalendarController < ApplicationController
      include GraphHelper

      def index
        # Get the IANA identifier of the user's time zone
        time_zone = get_iana_from_windows(user_timezone)

        # Calculate the start and end of week in the user's time zone
        start_datetime = Date.today.beginning_of_week(:sunday).in_time_zone(time_zone).to_time
        end_datetime = start_datetime.advance(:days => 7)

        @events = get_calendar_view access_token, start_datetime, end_datetime, user_timezone || []
        render json: @events
      rescue RuntimeError => e
        @errors = [
          {
            :message => 'Microsoft Graph returned an error getting events.',
            :debug => e
          }
        ]
      end
    end
    ```

1. Redémarrez le serveur. Connectez-vous, puis cliquez sur le lien **calendrier** dans la barre de navigation. Si tout fonctionne, vous devriez voir une image mémoire JSON des événements dans le calendrier de l’utilisateur.

## <a name="display-the-results"></a>Afficher les résultats

À présent, vous pouvez ajouter du code HTML pour afficher les résultats de manière plus conviviale.

1. Ouvrez **/app/views/calendar/index.html. Erb** et remplacez son contenu par ce qui suit.

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    Cela permet de parcourir une collection d’événements et d’ajouter une ligne de tableau pour chacun d’eux.

1. Supprimez la `render json: @events` ligne de l' `index` action dans **./app/Controllers/calendar_controller. RB**.

1. Actualisez la page et l’application doit maintenant afficher un tableau d’événements.

    ![Capture d’écran du tableau des événements](./images/add-msgraph-01.png)
