<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="9be5c-101">Dans cet exercice, vous allez incorporer Microsoft Graph dans l’application.</span><span class="sxs-lookup"><span data-stu-id="9be5c-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="9be5c-102">Pour cette application, vous allez utiliser la gemme [HTTParty](https://github.com/jnunemaker/httparty) pour passer des appels à Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="9be5c-102">For this application, you will use the [httparty](https://github.com/jnunemaker/httparty) gem to make calls to Microsoft Graph.</span></span>

## <a name="create-a-graph-helper"></a><span data-ttu-id="9be5c-103">Créer une application d’assistance Graph</span><span class="sxs-lookup"><span data-stu-id="9be5c-103">Create a Graph helper</span></span>

1. <span data-ttu-id="9be5c-104">Créez une assistance pour gérer tous vos appels d’API.</span><span class="sxs-lookup"><span data-stu-id="9be5c-104">Create a helper to manage all of your API calls.</span></span> <span data-ttu-id="9be5c-105">Exécutez la commande suivante dans votre interface CLI pour générer l’assistance.</span><span class="sxs-lookup"><span data-stu-id="9be5c-105">Run the following command in your CLI to generate the helper.</span></span>

    ```Shell
    rails generate helper Graph
    ```

1. <span data-ttu-id="9be5c-106">Ouvrez **./app/helpers/graph_helper. RB** et remplacez le contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="9be5c-106">Open **./app/helpers/graph_helper.rb** and replace the contents with the following.</span></span>

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

<span data-ttu-id="9be5c-107">Prenez un moment pour examiner ce que fait ce code.</span><span class="sxs-lookup"><span data-stu-id="9be5c-107">Take a moment to review what this code does.</span></span> <span data-ttu-id="9be5c-108">Il effectue une requête GET ou POST simple via la marque `httparty` pour le point de terminaison demandé.</span><span class="sxs-lookup"><span data-stu-id="9be5c-108">It makes a simple GET or POST request via the `httparty` gem to the requested endpoint.</span></span> <span data-ttu-id="9be5c-109">Il envoie le jeton d’accès dans l' `Authorization` en-tête et inclut tous les paramètres de requête qui sont passés.</span><span class="sxs-lookup"><span data-stu-id="9be5c-109">It sends the access token in the `Authorization` header, and it includes any query parameters that are passed.</span></span>

<span data-ttu-id="9be5c-110">Par exemple, pour utiliser la `make_api_call` méthode permettant d’effectuer une opération atteindre `https://graph.microsoft.com/v1.0/me?$select=displayName` , vous pouvez l’appeler comme suit :</span><span class="sxs-lookup"><span data-stu-id="9be5c-110">For example, to use the `make_api_call` method to do a GET to `https://graph.microsoft.com/v1.0/me?$select=displayName`, you could call it like so:</span></span>

```ruby
make_api_call 'GET', '/v1.0/me', access_token, { '$select': 'displayName' }
```

<span data-ttu-id="9be5c-111">Vous la créerez ultérieurement lorsque vous implémenterez davantage de fonctionnalités Microsoft Graph dans l’application.</span><span class="sxs-lookup"><span data-stu-id="9be5c-111">You'll build on this later as you implement more Microsoft Graph features into the app.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="9be5c-112">Récupérer les événements de calendrier à partir d’Outlook</span><span class="sxs-lookup"><span data-stu-id="9be5c-112">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="9be5c-113">Dans votre interface CLI, exécutez la commande suivante pour ajouter un nouveau contrôleur.</span><span class="sxs-lookup"><span data-stu-id="9be5c-113">In your CLI, run the following command to add a new controller.</span></span>

    ```Shell
    rails generate controller Calendar index new
    ```

1. <span data-ttu-id="9be5c-114">Ajoutez le nouvel itinéraire à **./config/routes.RB**.</span><span class="sxs-lookup"><span data-stu-id="9be5c-114">Add the new route to **./config/routes.rb**.</span></span>

    ```ruby
    get 'calendar', :to => 'calendar#index'
    ```

1. <span data-ttu-id="9be5c-115">Ajoutez une nouvelle méthode à l’aide Graph pour [obtenir un affichage Calendrier](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0).</span><span class="sxs-lookup"><span data-stu-id="9be5c-115">Add a new method to the Graph helper to [get a calendar view](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0).</span></span> <span data-ttu-id="9be5c-116">Ouvrez **./app/helpers/graph_helper. RB** et ajoutez la méthode suivante au `GraphHelper` module.</span><span class="sxs-lookup"><span data-stu-id="9be5c-116">Open **./app/helpers/graph_helper.rb** and add the following method to the `GraphHelper` module.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    <span data-ttu-id="9be5c-117">Que fait ce code ?</span><span class="sxs-lookup"><span data-stu-id="9be5c-117">Consider what this code is doing.</span></span>

    - <span data-ttu-id="9be5c-118">L’URL qui sera appelée est `/v1.0/me/calendarview`.</span><span class="sxs-lookup"><span data-stu-id="9be5c-118">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
        - <span data-ttu-id="9be5c-119">L' `Prefer: outlook.timezone` en-tête entraîne l’ajustement des heures de début et de fin dans les résultats sur le fuseau horaire de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="9be5c-119">The `Prefer: outlook.timezone` header causes the start and end times in the results to be adjusted to the user's time zone.</span></span>
        - <span data-ttu-id="9be5c-120">Les `startDateTime` `endDateTime` paramètres et définissent le début et la fin de l’affichage.</span><span class="sxs-lookup"><span data-stu-id="9be5c-120">The `startDateTime` and `endDateTime` parameters set the start and end of the view.</span></span>
        - <span data-ttu-id="9be5c-121">Le `$select` paramètre limite les champs renvoyés pour chaque événement à ceux que l’affichage utilise réellement.</span><span class="sxs-lookup"><span data-stu-id="9be5c-121">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
        - <span data-ttu-id="9be5c-122">Le `$orderby` paramètre trie les résultats par heure de début.</span><span class="sxs-lookup"><span data-stu-id="9be5c-122">The `$orderby` parameter sorts the results by start time.</span></span>
        - <span data-ttu-id="9be5c-123">Le `$top` paramètre limite les résultats à 50 événements.</span><span class="sxs-lookup"><span data-stu-id="9be5c-123">The `$top` parameter limits the results to 50 events.</span></span>
    - <span data-ttu-id="9be5c-124">Pour une réponse réussie, elle renvoie le tableau d’éléments contenus dans la `value` clé.</span><span class="sxs-lookup"><span data-stu-id="9be5c-124">For a successful response, it returns the array of items contained in the `value` key.</span></span>

1. <span data-ttu-id="9be5c-125">Ajoutez une nouvelle méthode à l’aide Graph pour rechercher un [identificateur de fuseau horaire IANA](https://www.iana.org/time-zones) en fonction du nom du fuseau horaire Windows.</span><span class="sxs-lookup"><span data-stu-id="9be5c-125">Add a new method to the Graph helper to lookup an [IANA time zone identifier](https://www.iana.org/time-zones) based on a Windows time zone name.</span></span> <span data-ttu-id="9be5c-126">Cette opération est nécessaire, car Microsoft Graph peut renvoyer des fuseaux horaires sous la forme de noms de fuseaux horaires Windows et la classe Ruby **DateTime** requiert des identificateurs de fuseau horaire IANA.</span><span class="sxs-lookup"><span data-stu-id="9be5c-126">This is necessary because Microsoft Graph can return time zones as Windows time zone names, and the Ruby **DateTime** class requires IANA time zone identifiers.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="ZoneMappingSnippet":::

1. <span data-ttu-id="9be5c-127">Ouvrez **./app/controllers/calendar_controller. RB** et remplacez l’intégralité de son contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="9be5c-127">Open **./app/controllers/calendar_controller.rb** and replace its entire contents with the following.</span></span>

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

1. <span data-ttu-id="9be5c-128">Redémarrez le serveur.</span><span class="sxs-lookup"><span data-stu-id="9be5c-128">Restart the server.</span></span> <span data-ttu-id="9be5c-129">Connectez-vous, puis cliquez sur le lien **calendrier** dans la barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="9be5c-129">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="9be5c-130">Si tout fonctionne, vous devriez voir une image mémoire JSON des événements dans le calendrier de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="9be5c-130">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="9be5c-131">Afficher les résultats</span><span class="sxs-lookup"><span data-stu-id="9be5c-131">Display the results</span></span>

<span data-ttu-id="9be5c-132">À présent, vous pouvez ajouter du code HTML pour afficher les résultats de manière plus conviviale.</span><span class="sxs-lookup"><span data-stu-id="9be5c-132">Now you can add HTML to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="9be5c-133">Ouvrez **/app/views/calendar/index.html. Erb** et remplacez son contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="9be5c-133">Open **./app/views/calendar/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    <span data-ttu-id="9be5c-134">Cela permet de parcourir une collection d’événements et d’ajouter une ligne de tableau pour chacun d’eux.</span><span class="sxs-lookup"><span data-stu-id="9be5c-134">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="9be5c-135">Supprimez la `render json: @events` ligne de l' `index` action dans **./app/Controllers/calendar_controller. RB**.</span><span class="sxs-lookup"><span data-stu-id="9be5c-135">Remove the `render json: @events` line from the `index` action in **./app/controllers/calendar_controller.rb**.</span></span>

1. <span data-ttu-id="9be5c-136">Actualisez la page et l’application doit maintenant afficher un tableau d’événements.</span><span class="sxs-lookup"><span data-stu-id="9be5c-136">Refresh the page and the app should now render a table of events.</span></span>

    ![Capture d’écran du tableau des événements](./images/add-msgraph-01.png)
