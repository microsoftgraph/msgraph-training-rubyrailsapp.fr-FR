<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="da282-101">Dans cette section, vous allez ajouter la possibilité de créer des événements dans le calendrier de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="da282-101">In this section you will add the ability to create events on the user's calendar.</span></span>

1. <span data-ttu-id="da282-102">Ouvrez **./app/helpers/graph_helper. RB** et ajoutez la méthode suivante à la classe **Graph** .</span><span class="sxs-lookup"><span data-stu-id="da282-102">Open **./app/helpers/graph_helper.rb** and add the following method to the **Graph** class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="CreateEventSnippet":::

1. <span data-ttu-id="da282-103">Ouvrez **/app/controllers/calendar_controller** et ajoutez l’itinéraire suivant à la classe **CalendarController**</span><span class="sxs-lookup"><span data-stu-id="da282-103">Open **./app/controllers/calendar_controller** and add the following route to the **CalendarController** class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/calendar_controller.rb" id="CreateEventRouteSnippet":::

1. <span data-ttu-id="da282-104">Ouvrez **./config/routes.RB** et ajoutez le nouvel itinéraire.</span><span class="sxs-lookup"><span data-stu-id="da282-104">Open **./config/routes.rb** and add the new route.</span></span>

    ```ruby
    post 'calendar/new', :to => 'calendar#create'
    ```

1. <span data-ttu-id="da282-105">Ouvrez **/app/views/calendar/new.html. Erb** et remplacez son contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="da282-105">Open **./app/views/calendar/new.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/new.html.erb" id="NewEventFormSnippet":::

1. <span data-ttu-id="da282-106">Enregistrez vos modifications et actualisez l’application.</span><span class="sxs-lookup"><span data-stu-id="da282-106">Save your changes and refresh the app.</span></span> <span data-ttu-id="da282-107">Sur la page **calendrier** , cliquez sur le bouton **nouvel événement** .</span><span class="sxs-lookup"><span data-stu-id="da282-107">On the **Calendar** page, select the **New event** button.</span></span> <span data-ttu-id="da282-108">Remplissez le formulaire et sélectionnez **créer** pour créer un événement.</span><span class="sxs-lookup"><span data-stu-id="da282-108">Fill in the form and select **Create** to create a new event.</span></span>

    ![Capture d’écran du nouveau formulaire d’événement](images/create-event-01.png)
