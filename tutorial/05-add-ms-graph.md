<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez incorporer le Graph Microsoft dans l'application. Pour cette application, vous allez utiliser le gem [httparty](https://github.com/jnunemaker/httparty) pour appeler Microsoft Graph.

## <a name="create-a-graph-helper"></a>Créer un Graph d'aide

1. Créez un système d'aide pour gérer tous vos appels d'API. Exécutez la commande suivante dans votre CLI pour générer l'aide.

    ```Shell
    rails generate helper Graph
    ```

1. Ouvrez **./app/helpers/graph_helper.rb** et remplacez le contenu par ce qui suit.

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

Prenez le temps de passer en revue ce que fait ce code. Il effectue une demande GET ou POST simple via le `httparty` gem au point de terminaison demandé. Il envoie le jeton d'accès dans l'en-tête et inclut tous les `Authorization` paramètres de requête transmis.

Par exemple, pour utiliser la `make_api_call` méthode pour faire une get `https://graph.microsoft.com/v1.0/me?$select=displayName` to, vous pouvez l'appeler comme ci-après :

```ruby
make_api_call 'GET', '/v1.0/me', access_token, {}, { '$select': 'displayName' }
```

Vous pourrez vous appuyer sur cette fonctionnalité plus tard à mesure que vous implémenterez d'autres fonctionnalités Graph Microsoft dans l'application.

## <a name="get-calendar-events-from-outlook"></a>Récupérer les événements de calendrier à partir d’Outlook

1. Dans votre CLI, exécutez la commande suivante pour ajouter un nouveau contrôleur.

    ```Shell
    rails generate controller Calendar index new
    ```

1. Ajoutez le nouvel itinéraire **à ./config/routes.rb**.

    ```ruby
    get 'calendar', :to => 'calendar#index'
    ```

1. Ajoutez une nouvelle méthode à l'Graph'aide pour [obtenir un affichage Calendrier.](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0) Ouvrez **./app/helpers/graph_helper.rb** et ajoutez la méthode suivante au `GraphHelper` module.

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    Que fait ce code ?

    - L’URL qui sera appelée est `/v1.0/me/calendarview`.
        - L'en-tête entraîne l'ajustement des heures de début et de fin dans les résultats en fonction du `Prefer: outlook.timezone` fuseau horaire de l'utilisateur.
        - Les `startDateTime` `endDateTime` paramètres et les paramètres définissent le début et la fin de l'affichage.
        - Le paramètre limite les champs renvoyés pour chaque événement à ceux que `$select` l'affichage utilisera réellement.
        - Le `$orderby` paramètre trie les résultats par heure de début.
        - Le `$top` paramètre limite les résultats à 50 événements.
    - Pour une réponse réussie, elle renvoie le tableau d'éléments contenus dans la `value` clé.

1. Ajoutez une nouvelle méthode à l'Graph pour rechercher un identificateur de fuseau horaire [IANA](https://www.iana.org/time-zones) basé sur un Windows de fuseau horaire. Cette étape est nécessaire, car Microsoft Graph des fuseaux horaires sous la Windows des noms de fuseau horaire et la classe Ruby **DateTime** requiert des identificateurs de fuseau horaire IANA.

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="ZoneMappingSnippet":::

1. Ouvrez **./app/controllers/calendar_controller.rb** et remplacez tout son contenu par ce qui suit.

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

1. Redémarrez le serveur. Connectez-vous et cliquez **sur le lien** Calendrier dans la barre de navigation. Si tout fonctionne, vous devriez voir une image mémoire JSON des événements dans le calendrier de l’utilisateur.

## <a name="display-the-results"></a>Afficher les résultats

Vous pouvez désormais ajouter du code HTML pour afficher les résultats de manière plus conviviale.

1. Ouvrez **./app/views/calendar/index.html.erb** et remplacez son contenu par ce qui suit.

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    Cela permet de parcourir une collection d’événements et d’ajouter une ligne de tableau pour chacun d’eux.

1. Supprimez `render json: @events` la ligne de `index` l'action **dans ./app/controllers/calendar_controller.rb**.

1. Actualisez la page et l'application doit maintenant restituer une table des événements.

    ![Capture d’écran du tableau des événements](./images/add-msgraph-01.png)
