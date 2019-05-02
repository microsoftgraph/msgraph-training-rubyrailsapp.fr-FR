<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez incorporer Microsoft Graph dans l’application. Pour cette application, vous allez utiliser la gemme [HTTParty](https://github.com/jnunemaker/httparty) pour passer des appels à Microsoft Graph.

## <a name="create-a-graph-helper"></a>Créer une application d’assistance Graph

Créez une assistance pour gérer tous vos appels d’API. Exécutez la commande suivante dans votre interface CLI pour générer l’assistance.

```Shell
rails generate helper Graph
```

Ouvrez le fichier nouvellement `./app/helpers/graph_helper.rb` créé et remplacez le contenu par ce qui suit.

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

Par exemple, pour utiliser la `make_api_call` méthode permettant d’effectuer une opération `https://graph.microsoft.com/v1.0/me?$select=displayName`atteindre, vous pouvez l’appeler comme suit:

```ruby
make_api_call `/v1.0/me`, access_token, { '$select': 'displayName' }
```

Vous la créerez ultérieurement lorsque vous implémenterez davantage de fonctionnalités Microsoft Graph dans l’application.

## <a name="get-calendar-events-from-outlook"></a>Obtenir des événements de calendrier à partir d’Outlook

Commençons par ajouter la possibilité d’afficher des événements dans le calendrier de l’utilisateur. Dans votre interface CLI, exécutez la commande suivante pour ajouter un nouveau contrôleur.

```Shell
rails generate controller Calendar index
```

Maintenant que nous disposons de l’itinéraire disponible, mettez à jour le lien **calendrier** de `./app/view/layouts/application.html.erb` la barre de navigation dans pour l’utiliser. Remplacez la ligne `<a class="nav-link" href="#">Calendar</a>` par ce qui suit.

```html
<%= link_to "Calendar", {:controller => :calendar, :action => :index}, class: "nav-link#{' active' if controller.controller_name == 'calendar'}" %>
```

Ajoutez une nouvelle méthode à l’aide Graph pour [répertorier les événements de l’utilisateur](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_list_events). Ouvrez `./app/helpers/graph_helper.rb` et ajoutez la méthode suivante au `GraphHelper` module.

```ruby
def get_calendar_events(token)
  get_events_url = '/v1.0/me/events'

  query = {
    '$select': 'subject,organizer,start,end',
    '$orderby': 'createdDateTime DESC'
  }

  response = make_api_call get_events_url, token, query

  raise response.parsed_response.to_s || "Request returned #{response.code}" unless response.code == 200
  response.parsed_response['value']
end
```

Examinez ce que fait ce code.

- L’URL qui sera appelée est `/v1.0/me/events`.
- Le `$select` paramètre limite les champs renvoyés pour chaque événement à ceux que notre vue utilisera réellement.
- Le `$orderby` paramètre trie les résultats en fonction de la date et de l’heure de leur création, avec l’élément le plus récent en premier.
- Pour une réponse réussie, elle renvoie le tableau d’éléments contenus dans la `value` clé.

À présent, vous pouvez le tester. Ouvrez `./app/controllers/calendar_controller.rb` et mettez à `index` jour l’action pour appeler cette méthode et afficher les résultats.

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

Redémarrez le serveur. Connectez-vous, puis cliquez sur le lien **calendrier** dans la barre de navigation. Si tout fonctionne, vous devez voir un vidage JSON des événements sur le calendrier de l’utilisateur.

## <a name="display-the-results"></a>Afficher les résultats

À présent, vous pouvez ajouter du code HTML et CSS pour afficher les résultats de manière plus conviviale.

Ouvrez `./app/views/calendar/index.html.erb` et remplacez son contenu par ce qui suit.

```html
<h1>Calendar</h1>
<table class="table">
  <thead>
    <tr>
      <th scope="col">Organizer</th>
      <th scope="col">Subject</th>
      <th scope="col">Start</th>
      <th scope="col">End</th>
    </tr>
  </thead>
  <tbody>
    <% @events.each do |event| %>
      <tr>
        <td><%= event['organizer']['emailAddress']['name'] %></td>
        <td><%= event['subject'] %></td>
        <td><%= event['start']['dateTime'].to_time(:utc).localtime.strftime('%-m/%-d/%y %l:%M %p') %></td>
        <td><%= event['end']['dateTime'].to_time(:utc).localtime.strftime('%-m/%-d/%y %l:%M %p') %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

Cela permet d’exécuter une boucle dans une collection d’événements et d’ajouter une ligne de tableau pour chacun d’eux. Supprimez `render json: @events` la ligne de `index` l’action `./app/controllers/calendar_controller.rb` dans et l’application doit maintenant afficher un tableau d’événements.

![Capture d’écran du tableau des événements](./images/add-msgraph-01.png)