<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="35447-101">Ce didacticiel vous apprend à créer une application Web Ruby on rails qui utilise l’API Microsoft Graph pour récupérer des informations de calendrier pour un utilisateur.</span><span class="sxs-lookup"><span data-stu-id="35447-101">This tutorial teaches you how to build a Ruby on Rails web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="35447-102">Si vous préférez télécharger simplement le didacticiel terminé, vous pouvez le télécharger de deux manières.</span><span class="sxs-lookup"><span data-stu-id="35447-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="35447-103">Téléchargez le [démarrage rapide de Ruby](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) pour obtenir du code de travail en quelques minutes.</span><span class="sxs-lookup"><span data-stu-id="35447-103">Download the [Ruby quick start](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) to get working code in minutes.</span></span>
> - <span data-ttu-id="35447-104">Téléchargez ou clonez le [référentiel GitHub](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span><span class="sxs-lookup"><span data-stu-id="35447-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="35447-105">Conditions préalables</span><span class="sxs-lookup"><span data-stu-id="35447-105">Prerequisites</span></span>

<span data-ttu-id="35447-106">Avant de commencer ce didacticiel, [Ruby](https://www.ruby-lang.org/en/downloads/) doit être installé sur votre ordinateur de développement.</span><span class="sxs-lookup"><span data-stu-id="35447-106">Before you start this tutorial, you should have [Ruby](https://www.ruby-lang.org/en/downloads/) installed on your development machine.</span></span> <span data-ttu-id="35447-107">Si vous n’avez pas Ruby, visitez le lien précédent pour obtenir les options de téléchargement.</span><span class="sxs-lookup"><span data-stu-id="35447-107">If you do not have Ruby, visit the previous link for download options.</span></span>

> [!NOTE]
> <span data-ttu-id="35447-108">Ce didacticiel a été écrit avec Ruby version 2.4.4.</span><span class="sxs-lookup"><span data-stu-id="35447-108">This tutorial was written with Ruby version 2.4.4.</span></span> <span data-ttu-id="35447-109">Les étapes de ce guide peuvent fonctionner avec d’autres versions, mais cela n’a pas été testé.</span><span class="sxs-lookup"><span data-stu-id="35447-109">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="35447-110">Commentaires</span><span class="sxs-lookup"><span data-stu-id="35447-110">Feedback</span></span>

<span data-ttu-id="35447-111">Veuillez fournir des commentaires sur ce didacticiel dans le [référentiel GitHub](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span><span class="sxs-lookup"><span data-stu-id="35447-111">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>
