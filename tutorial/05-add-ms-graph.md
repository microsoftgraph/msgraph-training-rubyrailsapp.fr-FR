<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="063c7-101">Dans cet exercice, vous allez incorporer Microsoft Graph dans l’application.</span><span class="sxs-lookup"><span data-stu-id="063c7-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="063c7-102">Pour cette application, vous allez utiliser la gemme [HTTParty](https://github.com/jnunemaker/httparty) pour passer des appels à Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="063c7-102">For this application, you will use the [httparty](https://github.com/jnunemaker/httparty) gem to make calls to Microsoft Graph.</span></span>

## <a name="create-a-graph-helper"></a><span data-ttu-id="063c7-103">Créer une application d’assistance Graph</span><span class="sxs-lookup"><span data-stu-id="063c7-103">Create a Graph helper</span></span>

1. <span data-ttu-id="063c7-104">Créez une assistance pour gérer tous vos appels d’API.</span><span class="sxs-lookup"><span data-stu-id="063c7-104">Create a helper to manage all of your API calls.</span></span> <span data-ttu-id="063c7-105">Exécutez la commande suivante dans votre interface CLI pour générer l’assistance.</span><span class="sxs-lookup"><span data-stu-id="063c7-105">Run the following command in your CLI to generate the helper.</span></span>

    ```Shell
    rails generate helper Graph
    ```

1. <span data-ttu-id="063c7-106">Ouvrez **./app/helpers/graph_helper. RB** et remplacez le contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="063c7-106">Open **./app/helpers/graph_helper.rb** and replace the contents with the following.</span></span>

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

<span data-ttu-id="063c7-107">Prenez un moment pour examiner ce que fait ce code.</span><span class="sxs-lookup"><span data-stu-id="063c7-107">Take a moment to review what this code does.</span></span> <span data-ttu-id="063c7-108">Il effectue une requête GET simple via la `httparty` marque pour le point de terminaison demandé.</span><span class="sxs-lookup"><span data-stu-id="063c7-108">It makes a simple GET request via the `httparty` gem to the requested endpoint.</span></span> <span data-ttu-id="063c7-109">Il envoie le jeton d’accès dans `Authorization` l’en-tête et inclut tous les paramètres de requête qui sont passés.</span><span class="sxs-lookup"><span data-stu-id="063c7-109">It sends the access token in the `Authorization` header, and it includes any query parameters that are passed.</span></span>

<span data-ttu-id="063c7-110">Par exemple, pour utiliser la `make_api_call` méthode permettant d’effectuer une opération `https://graph.microsoft.com/v1.0/me?$select=displayName`atteindre, vous pouvez l’appeler comme suit :</span><span class="sxs-lookup"><span data-stu-id="063c7-110">For example, to use the `make_api_call` method to do a GET to `https://graph.microsoft.com/v1.0/me?$select=displayName`, you could call it like so:</span></span>

```ruby
make_api_call `/v1.0/me`, access_token, { '$select': 'displayName' }
```

<span data-ttu-id="063c7-111">Vous la créerez ultérieurement lorsque vous implémenterez davantage de fonctionnalités Microsoft Graph dans l’application.</span><span class="sxs-lookup"><span data-stu-id="063c7-111">You'll build on this later as you implement more Microsoft Graph features into the app.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="063c7-112">Récupérer les événements de calendrier à partir d’Outlook</span><span class="sxs-lookup"><span data-stu-id="063c7-112">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="063c7-113">Dans votre interface CLI, exécutez la commande suivante pour ajouter un nouveau contrôleur.</span><span class="sxs-lookup"><span data-stu-id="063c7-113">In your CLI, run the following command to add a new controller.</span></span>

    ```Shell
    rails generate controller Calendar index
    ```

1. <span data-ttu-id="063c7-114">Ajoutez le nouvel itinéraire à **./config/routes.RB**.</span><span class="sxs-lookup"><span data-stu-id="063c7-114">Add the new route to **./config/routes.rb**.</span></span>

    ```ruby
    get 'calendar', to: 'calendar#index'
    ```

1. <span data-ttu-id="063c7-115">Ajoutez une nouvelle méthode à l’aide Graph pour [répertorier les événements de l’utilisateur](/graph/api/user-list-events?view=graph-rest-1.0).</span><span class="sxs-lookup"><span data-stu-id="063c7-115">Add a new method to the Graph helper to [list the user's events](/graph/api/user-list-events?view=graph-rest-1.0).</span></span> <span data-ttu-id="063c7-116">Ouvrez **./app/helpers/graph_helper. RB** et ajoutez la méthode suivante au `GraphHelper` module.</span><span class="sxs-lookup"><span data-stu-id="063c7-116">Open **./app/helpers/graph_helper.rb** and add the following method to the `GraphHelper` module.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    <span data-ttu-id="063c7-117">Que fait ce code ?</span><span class="sxs-lookup"><span data-stu-id="063c7-117">Consider what this code is doing.</span></span>

    - <span data-ttu-id="063c7-118">L’URL qui sera appelée est `/v1.0/me/events`.</span><span class="sxs-lookup"><span data-stu-id="063c7-118">The URL that will be called is `/v1.0/me/events`.</span></span>
    - <span data-ttu-id="063c7-119">Le `$select` paramètre limite les champs renvoyés pour chaque événement à ceux que notre vue utilisera réellement.</span><span class="sxs-lookup"><span data-stu-id="063c7-119">The `$select` parameter limits the fields returned for each events to just those our view will actually use.</span></span>
    - <span data-ttu-id="063c7-120">Le `$orderby` paramètre trie les résultats en fonction de la date et de l’heure de leur création, avec l’élément le plus récent en premier.</span><span class="sxs-lookup"><span data-stu-id="063c7-120">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>
    - <span data-ttu-id="063c7-121">Pour une réponse réussie, elle renvoie le tableau d’éléments contenus dans la `value` clé.</span><span class="sxs-lookup"><span data-stu-id="063c7-121">For a successful response, it returns the array of items contained in the `value` key.</span></span>

1. <span data-ttu-id="063c7-122">Ouvrez **./app/controllers/calendar_controller. RB** et remplacez l’intégralité de son contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="063c7-122">Open **./app/controllers/calendar_controller.rb** and replace its entire contents with the following.</span></span>

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

1. <span data-ttu-id="063c7-123">Redémarrez le serveur.</span><span class="sxs-lookup"><span data-stu-id="063c7-123">Restart the server.</span></span> <span data-ttu-id="063c7-124">Connectez-vous, puis cliquez sur le lien **calendrier** dans la barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="063c7-124">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="063c7-125">Si tout fonctionne, vous devriez voir une image mémoire JSON des événements dans le calendrier de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="063c7-125">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="063c7-126">Afficher les résultats</span><span class="sxs-lookup"><span data-stu-id="063c7-126">Display the results</span></span>

<span data-ttu-id="063c7-127">À présent, vous pouvez ajouter du code HTML pour afficher les résultats de manière plus conviviale.</span><span class="sxs-lookup"><span data-stu-id="063c7-127">Now you can add HTML to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="063c7-128">Ouvrez **./app/views/Calendar/index.html.Erb** et remplacez son contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="063c7-128">Open **./app/views/calendar/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    <span data-ttu-id="063c7-129">Cela permet de parcourir une collection d’événements et d’ajouter une ligne de tableau pour chacun d’eux.</span><span class="sxs-lookup"><span data-stu-id="063c7-129">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="063c7-130">Supprimez `render json: @events` la ligne de `index` l’action dans **./app/Controllers/calendar_controller. RB**.</span><span class="sxs-lookup"><span data-stu-id="063c7-130">Remove the `render json: @events` line from the `index` action in **./app/controllers/calendar_controller.rb**.</span></span>

1. <span data-ttu-id="063c7-131">Actualisez la page et l’application doit maintenant afficher un tableau d’événements.</span><span class="sxs-lookup"><span data-stu-id="063c7-131">Refresh the page and the app should now render a table of events.</span></span>

    ![Capture d’écran du tableau des événements](./images/add-msgraph-01.png)
