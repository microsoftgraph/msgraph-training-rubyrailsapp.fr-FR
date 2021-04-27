<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="6cea6-101">Dans cet exercice, vous allez incorporer le Graph Microsoft dans l'application.</span><span class="sxs-lookup"><span data-stu-id="6cea6-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="6cea6-102">Pour cette application, vous allez utiliser le gem [httparty](https://github.com/jnunemaker/httparty) pour appeler Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="6cea6-102">For this application, you will use the [httparty](https://github.com/jnunemaker/httparty) gem to make calls to Microsoft Graph.</span></span>

## <a name="create-a-graph-helper"></a><span data-ttu-id="6cea6-103">Créer un Graph d'aide</span><span class="sxs-lookup"><span data-stu-id="6cea6-103">Create a Graph helper</span></span>

1. <span data-ttu-id="6cea6-104">Créez un système d'aide pour gérer tous vos appels d'API.</span><span class="sxs-lookup"><span data-stu-id="6cea6-104">Create a helper to manage all of your API calls.</span></span> <span data-ttu-id="6cea6-105">Exécutez la commande suivante dans votre CLI pour générer l'aide.</span><span class="sxs-lookup"><span data-stu-id="6cea6-105">Run the following command in your CLI to generate the helper.</span></span>

    ```Shell
    rails generate helper Graph
    ```

1. <span data-ttu-id="6cea6-106">Ouvrez **./app/helpers/graph_helper.rb** et remplacez le contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="6cea6-106">Open **./app/helpers/graph_helper.rb** and replace the contents with the following.</span></span>

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

<span data-ttu-id="6cea6-107">Prenez le temps de passer en revue ce que fait ce code.</span><span class="sxs-lookup"><span data-stu-id="6cea6-107">Take a moment to review what this code does.</span></span> <span data-ttu-id="6cea6-108">Il effectue une demande GET ou POST simple via le `httparty` gem au point de terminaison demandé.</span><span class="sxs-lookup"><span data-stu-id="6cea6-108">It makes a simple GET or POST request via the `httparty` gem to the requested endpoint.</span></span> <span data-ttu-id="6cea6-109">Il envoie le jeton d'accès dans l'en-tête et inclut tous les `Authorization` paramètres de requête transmis.</span><span class="sxs-lookup"><span data-stu-id="6cea6-109">It sends the access token in the `Authorization` header, and it includes any query parameters that are passed.</span></span>

<span data-ttu-id="6cea6-110">Par exemple, pour utiliser la `make_api_call` méthode pour faire une get `https://graph.microsoft.com/v1.0/me?$select=displayName` to, vous pouvez l'appeler comme ci-après :</span><span class="sxs-lookup"><span data-stu-id="6cea6-110">For example, to use the `make_api_call` method to do a GET to `https://graph.microsoft.com/v1.0/me?$select=displayName`, you could call it like so:</span></span>

```ruby
make_api_call 'GET', '/v1.0/me', access_token, {}, { '$select': 'displayName' }
```

<span data-ttu-id="6cea6-111">Vous pourrez vous appuyer sur cette fonctionnalité plus tard à mesure que vous implémenterez d'autres fonctionnalités Graph Microsoft dans l'application.</span><span class="sxs-lookup"><span data-stu-id="6cea6-111">You'll build on this later as you implement more Microsoft Graph features into the app.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="6cea6-112">Récupérer les événements de calendrier à partir d’Outlook</span><span class="sxs-lookup"><span data-stu-id="6cea6-112">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="6cea6-113">Dans votre CLI, exécutez la commande suivante pour ajouter un nouveau contrôleur.</span><span class="sxs-lookup"><span data-stu-id="6cea6-113">In your CLI, run the following command to add a new controller.</span></span>

    ```Shell
    rails generate controller Calendar index new
    ```

1. <span data-ttu-id="6cea6-114">Ajoutez le nouvel itinéraire **à ./config/routes.rb**.</span><span class="sxs-lookup"><span data-stu-id="6cea6-114">Add the new route to **./config/routes.rb**.</span></span>

    ```ruby
    get 'calendar', :to => 'calendar#index'
    ```

1. <span data-ttu-id="6cea6-115">Ajoutez une nouvelle méthode à l'Graph'aide pour [obtenir un affichage Calendrier.](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0)</span><span class="sxs-lookup"><span data-stu-id="6cea6-115">Add a new method to the Graph helper to [get a calendar view](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0).</span></span> <span data-ttu-id="6cea6-116">Ouvrez **./app/helpers/graph_helper.rb** et ajoutez la méthode suivante au `GraphHelper` module.</span><span class="sxs-lookup"><span data-stu-id="6cea6-116">Open **./app/helpers/graph_helper.rb** and add the following method to the `GraphHelper` module.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    <span data-ttu-id="6cea6-117">Que fait ce code ?</span><span class="sxs-lookup"><span data-stu-id="6cea6-117">Consider what this code is doing.</span></span>

    - <span data-ttu-id="6cea6-118">L’URL qui sera appelée est `/v1.0/me/calendarview`.</span><span class="sxs-lookup"><span data-stu-id="6cea6-118">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
        - <span data-ttu-id="6cea6-119">L'en-tête entraîne l'ajustement des heures de début et de fin dans les résultats en fonction du `Prefer: outlook.timezone` fuseau horaire de l'utilisateur.</span><span class="sxs-lookup"><span data-stu-id="6cea6-119">The `Prefer: outlook.timezone` header causes the start and end times in the results to be adjusted to the user's time zone.</span></span>
        - <span data-ttu-id="6cea6-120">Les `startDateTime` `endDateTime` paramètres et les paramètres définissent le début et la fin de l'affichage.</span><span class="sxs-lookup"><span data-stu-id="6cea6-120">The `startDateTime` and `endDateTime` parameters set the start and end of the view.</span></span>
        - <span data-ttu-id="6cea6-121">Le paramètre limite les champs renvoyés pour chaque événement à ceux que `$select` l'affichage utilisera réellement.</span><span class="sxs-lookup"><span data-stu-id="6cea6-121">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
        - <span data-ttu-id="6cea6-122">Le `$orderby` paramètre trie les résultats par heure de début.</span><span class="sxs-lookup"><span data-stu-id="6cea6-122">The `$orderby` parameter sorts the results by start time.</span></span>
        - <span data-ttu-id="6cea6-123">Le `$top` paramètre limite les résultats à 50 événements.</span><span class="sxs-lookup"><span data-stu-id="6cea6-123">The `$top` parameter limits the results to 50 events.</span></span>
    - <span data-ttu-id="6cea6-124">Pour une réponse réussie, elle renvoie le tableau d'éléments contenus dans la `value` clé.</span><span class="sxs-lookup"><span data-stu-id="6cea6-124">For a successful response, it returns the array of items contained in the `value` key.</span></span>

1. <span data-ttu-id="6cea6-125">Ajoutez une nouvelle méthode à l'Graph pour rechercher un identificateur de fuseau horaire [IANA](https://www.iana.org/time-zones) basé sur un Windows de fuseau horaire.</span><span class="sxs-lookup"><span data-stu-id="6cea6-125">Add a new method to the Graph helper to lookup an [IANA time zone identifier](https://www.iana.org/time-zones) based on a Windows time zone name.</span></span> <span data-ttu-id="6cea6-126">Cette étape est nécessaire, car Microsoft Graph des fuseaux horaires sous la Windows des noms de fuseau horaire et la classe Ruby **DateTime** requiert des identificateurs de fuseau horaire IANA.</span><span class="sxs-lookup"><span data-stu-id="6cea6-126">This is necessary because Microsoft Graph can return time zones as Windows time zone names, and the Ruby **DateTime** class requires IANA time zone identifiers.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="ZoneMappingSnippet":::

1. <span data-ttu-id="6cea6-127">Ouvrez **./app/controllers/calendar_controller.rb** et remplacez tout son contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="6cea6-127">Open **./app/controllers/calendar_controller.rb** and replace its entire contents with the following.</span></span>

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

1. <span data-ttu-id="6cea6-128">Redémarrez le serveur.</span><span class="sxs-lookup"><span data-stu-id="6cea6-128">Restart the server.</span></span> <span data-ttu-id="6cea6-129">Connectez-vous et cliquez **sur le lien** Calendrier dans la barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="6cea6-129">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="6cea6-130">Si tout fonctionne, vous devriez voir une image mémoire JSON des événements dans le calendrier de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="6cea6-130">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="6cea6-131">Afficher les résultats</span><span class="sxs-lookup"><span data-stu-id="6cea6-131">Display the results</span></span>

<span data-ttu-id="6cea6-132">Vous pouvez désormais ajouter du code HTML pour afficher les résultats de manière plus conviviale.</span><span class="sxs-lookup"><span data-stu-id="6cea6-132">Now you can add HTML to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="6cea6-133">Ouvrez **./app/views/calendar/index.html.erb** et remplacez son contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="6cea6-133">Open **./app/views/calendar/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    <span data-ttu-id="6cea6-134">Cela permet de parcourir une collection d’événements et d’ajouter une ligne de tableau pour chacun d’eux.</span><span class="sxs-lookup"><span data-stu-id="6cea6-134">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="6cea6-135">Supprimez `render json: @events` la ligne de `index` l'action **dans ./app/controllers/calendar_controller.rb**.</span><span class="sxs-lookup"><span data-stu-id="6cea6-135">Remove the `render json: @events` line from the `index` action in **./app/controllers/calendar_controller.rb**.</span></span>

1. <span data-ttu-id="6cea6-136">Actualisez la page et l'application doit maintenant restituer une table des événements.</span><span class="sxs-lookup"><span data-stu-id="6cea6-136">Refresh the page and the app should now render a table of events.</span></span>

    ![Capture d’écran du tableau des événements](./images/add-msgraph-01.png)
