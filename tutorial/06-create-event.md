<!-- markdownlint-disable MD002 MD041 -->

Dans cette section, vous allez ajouter la possibilité de créer des événements dans le calendrier de l’utilisateur.

1. Ouvrez **./app/helpers/graph_helper. RB** et ajoutez la méthode suivante à la classe **Graph** .

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="CreateEventSnippet":::

1. Ouvrez **/app/controllers/calendar_controller** et ajoutez l’itinéraire suivant à la classe **CalendarController**

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/calendar_controller.rb" id="CreateEventRouteSnippet":::

1. Ouvrez **./config/routes.RB** et ajoutez le nouvel itinéraire.

    ```ruby
    post 'calendar/new', :to => 'calendar#create'
    ```

1. Ouvrez **/app/views/calendar/new.html. Erb** et remplacez son contenu par ce qui suit.

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/new.html.erb" id="NewEventFormSnippet":::

1. Enregistrez vos modifications et actualisez l’application. Sur la page **calendrier** , cliquez sur le bouton **nouvel événement** . Remplissez le formulaire et sélectionnez **créer** pour créer un événement.

    ![Capture d’écran du nouveau formulaire d’événement](images/create-event-01.png)
