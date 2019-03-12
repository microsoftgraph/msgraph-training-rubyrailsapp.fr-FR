<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="7dd11-101">Dans cet exercice, vous allez incorporer Microsoft Graph dans l'application.</span><span class="sxs-lookup"><span data-stu-id="7dd11-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="7dd11-102">Pour cette application, vous allez utiliser la gemme [HTTParty](https://github.com/jnunemaker/httparty) pour passer des appels à Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="7dd11-102">For this application, you will use the [httparty](https://github.com/jnunemaker/httparty) gem to make calls to Microsoft Graph.</span></span>

## <a name="create-a-graph-helper"></a><span data-ttu-id="7dd11-103">Créer une application d'assistance Graph</span><span class="sxs-lookup"><span data-stu-id="7dd11-103">Create a Graph helper</span></span>

<span data-ttu-id="7dd11-104">Créez une assistance pour gérer tous vos appels d'API.</span><span class="sxs-lookup"><span data-stu-id="7dd11-104">Create a helper to manage all of your API calls.</span></span> <span data-ttu-id="7dd11-105">Exécutez la commande suivante dans votre interface CLI pour générer l'assistance.</span><span class="sxs-lookup"><span data-stu-id="7dd11-105">Run the following command in your CLI to generate the helper.</span></span>

```Shell
rails generate helper Graph
```

<span data-ttu-id="7dd11-106">Ouvrez le fichier nouvellement `./app/helpers/graph_helper.rb` créé et remplacez le contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="7dd11-106">Open the newly created `./app/helpers/graph_helper.rb` file and replace the contents with the following.</span></span>

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

<span data-ttu-id="7dd11-107">Prenez un moment pour examiner ce que fait ce code.</span><span class="sxs-lookup"><span data-stu-id="7dd11-107">Take a moment to review what this code does.</span></span> <span data-ttu-id="7dd11-108">Il effectue une requête GET simple via la `httparty` marque pour le point de terminaison demandé.</span><span class="sxs-lookup"><span data-stu-id="7dd11-108">It makes a simple GET request via the `httparty` gem to the requested endpoint.</span></span> <span data-ttu-id="7dd11-109">Il envoie le jeton d'accès dans `Authorization` l'en-tête et inclut tous les paramètres de requête qui sont passés.</span><span class="sxs-lookup"><span data-stu-id="7dd11-109">It sends the access token in the `Authorization` header, and it includes any query parameters that are passed.</span></span>

<span data-ttu-id="7dd11-110">Par exemple, pour utiliser la `make_api_call` méthode permettant d'effectuer une opération `https://graph.microsoft.com/v1.0/me?$select=displayName`atteindre, vous pouvez l'appeler comme suit:</span><span class="sxs-lookup"><span data-stu-id="7dd11-110">For example, to use the `make_api_call` method to do a GET to `https://graph.microsoft.com/v1.0/me?$select=displayName`, you could call it like so:</span></span>

```ruby
make_api_call `/v1.0/me`, access_token, { '$select': 'displayName' }
```

<span data-ttu-id="7dd11-111">Vous la créerez ultérieurement lorsque vous implémenterez davantage de fonctionnalités Microsoft Graph dans l'application.</span><span class="sxs-lookup"><span data-stu-id="7dd11-111">You'll build on this later as you implement more Microsoft Graph features into the app.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="7dd11-112">Obtenir des événements de calendrier à partir d'Outlook</span><span class="sxs-lookup"><span data-stu-id="7dd11-112">Get calendar events from Outlook</span></span>

<span data-ttu-id="7dd11-113">Commençons par ajouter la possibilité d'afficher des événements dans le calendrier de l'utilisateur.</span><span class="sxs-lookup"><span data-stu-id="7dd11-113">Let's start by adding the ability to view events on the user's calendar.</span></span> <span data-ttu-id="7dd11-114">Dans votre interface CLI, exécutez la commande suivante pour ajouter un nouveau contrôleur.</span><span class="sxs-lookup"><span data-stu-id="7dd11-114">In your CLI, run the following command to add a new controller.</span></span>

```Shell
rails generate controller Calendar index
```

<span data-ttu-id="7dd11-115">Maintenant que nous disposons de l'itinéraire disponible, mettez à jour le lien **calendrier** de `./app/view/layouts/application.html.erb` la barre de navigation dans pour l'utiliser.</span><span class="sxs-lookup"><span data-stu-id="7dd11-115">Now that we have the route available, update the **Calendar** link in the navbar in `./app/view/layouts/application.html.erb` to use it.</span></span> <span data-ttu-id="7dd11-116">Remplacez la ligne `<a class="nav-link" href="#">Calendar</a>` par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="7dd11-116">Replace the line `<a class="nav-link" href="#">Calendar</a>` with the following.</span></span>

```html
<%= link_to "Calendar", {:controller => :calendar, :action => :index}, class: "nav-link#{' active' if controller.controller_name == 'calendar'}" %>
```

<span data-ttu-id="7dd11-117">Ajoutez une nouvelle méthode à l'aide Graph pour [répertorier les événements de l'utilisateur](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_list_events).</span><span class="sxs-lookup"><span data-stu-id="7dd11-117">Add a new method to the Graph helper to [list the user's events](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_list_events).</span></span> <span data-ttu-id="7dd11-118">Ouvrez `./app/helpers/graph_helper.rb` et ajoutez la méthode suivante au `GraphHelper` module.</span><span class="sxs-lookup"><span data-stu-id="7dd11-118">Open `./app/helpers/graph_helper.rb` and add the following method to the `GraphHelper` module.</span></span>

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

<span data-ttu-id="7dd11-119">Examinez ce que fait ce code.</span><span class="sxs-lookup"><span data-stu-id="7dd11-119">Consider what this code is doing.</span></span>

- <span data-ttu-id="7dd11-120">L'URL qui sera appelée est `/v1.0/me/events`.</span><span class="sxs-lookup"><span data-stu-id="7dd11-120">The URL that will be called is `/v1.0/me/events`.</span></span>
- <span data-ttu-id="7dd11-121">Le `$select` paramètre limite les champs renvoyés pour chaque événement à ceux que notre vue utilisera réellement.</span><span class="sxs-lookup"><span data-stu-id="7dd11-121">The `$select` parameter limits the fields returned for each events to just those our view will actually use.</span></span>
- <span data-ttu-id="7dd11-122">Le `$orderby` paramètre trie les résultats en fonction de la date et de l'heure de leur création, avec l'élément le plus récent en premier.</span><span class="sxs-lookup"><span data-stu-id="7dd11-122">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>
- <span data-ttu-id="7dd11-123">Pour une réponse réussie, elle renvoie le tableau d'éléments contenus dans la `value` clé.</span><span class="sxs-lookup"><span data-stu-id="7dd11-123">For a successful response, it returns the array of items contained in the `value` key.</span></span>

<span data-ttu-id="7dd11-124">À présent, vous pouvez le tester.</span><span class="sxs-lookup"><span data-stu-id="7dd11-124">Now you can test this.</span></span> <span data-ttu-id="7dd11-125">Ouvrez `./app/controllers/calendar_controller.rb` et mettez à `index` jour l'action pour appeler cette méthode et afficher les résultats.</span><span class="sxs-lookup"><span data-stu-id="7dd11-125">Open `./app/controllers/calendar_controller.rb` and update the `index` action to call this method and render the results.</span></span>

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

<span data-ttu-id="7dd11-126">Redémarrez le serveur.</span><span class="sxs-lookup"><span data-stu-id="7dd11-126">Restart the server.</span></span> <span data-ttu-id="7dd11-127">Connectez-vous, puis cliquez sur le lien **calendrier** dans la barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="7dd11-127">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="7dd11-128">Si tout fonctionne, vous devez voir un vidage JSON des événements sur le calendrier de l'utilisateur.</span><span class="sxs-lookup"><span data-stu-id="7dd11-128">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="7dd11-129">Afficher les résultats</span><span class="sxs-lookup"><span data-stu-id="7dd11-129">Display the results</span></span>

<span data-ttu-id="7dd11-130">À présent, vous pouvez ajouter du code HTML et CSS pour afficher les résultats de manière plus conviviale.</span><span class="sxs-lookup"><span data-stu-id="7dd11-130">Now you can add HTML and CSS to display the results in a more user-friendly manner.</span></span>

<span data-ttu-id="7dd11-131">Ouvrez `./app/views/calendar/index.html.erb` et remplacez son contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="7dd11-131">Open `./app/views/calendar/index.html.erb` and replace its contents with the following.</span></span>

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

<span data-ttu-id="7dd11-132">Cela permet d'exécuter une boucle dans une collection d'événements et d'ajouter une ligne de tableau pour chacun d'eux.</span><span class="sxs-lookup"><span data-stu-id="7dd11-132">That will loop through a collection of events and add a table row for each one.</span></span> <span data-ttu-id="7dd11-133">Supprimez `render json: @events` la ligne de `index` l'action `./app/controllers/calendar_controller.rb` dans et l'application doit maintenant afficher un tableau d'événements.</span><span class="sxs-lookup"><span data-stu-id="7dd11-133">Remove the `render json: @events` line from the `index` action in `./app/controllers/calendar_controller.rb` and the app should now render a table of events.</span></span>

![Capture d'écran du tableau des événements](./images/add-msgraph-01.png)