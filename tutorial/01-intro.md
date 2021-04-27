<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="452f8-101">Ce didacticiel vous apprend à créer une application web Ruby on Rails qui utilise l'API Microsoft Graph pour récupérer les informations de calendrier d'un utilisateur.</span><span class="sxs-lookup"><span data-stu-id="452f8-101">This tutorial teaches you how to build a Ruby on Rails web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="452f8-102">Si vous préférez simplement télécharger le didacticiel terminé, vous pouvez le télécharger de deux manières.</span><span class="sxs-lookup"><span data-stu-id="452f8-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="452f8-103">Téléchargez [le démarrage rapide de Ruby](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) pour obtenir du code de travail en quelques minutes.</span><span class="sxs-lookup"><span data-stu-id="452f8-103">Download the [Ruby quick start](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) to get working code in minutes.</span></span>
> - <span data-ttu-id="452f8-104">Téléchargez ou clonez [le GitHub de données.](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)</span><span class="sxs-lookup"><span data-stu-id="452f8-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="452f8-105">Configuration requise</span><span class="sxs-lookup"><span data-stu-id="452f8-105">Prerequisites</span></span>

<span data-ttu-id="452f8-106">Avant de commencer ce didacticiel, vous devez avoir installé les outils suivants sur votre ordinateur de développement.</span><span class="sxs-lookup"><span data-stu-id="452f8-106">Before you start this tutorial, you should have the following tools installed on your development machine.</span></span>

- [<span data-ttu-id="452f8-107">Ruby</span><span class="sxs-lookup"><span data-stu-id="452f8-107">Ruby</span></span>](https://www.ruby-lang.org/en/downloads/)
- [<span data-ttu-id="452f8-108">SQLite3</span><span class="sxs-lookup"><span data-stu-id="452f8-108">SQLite3</span></span>](https://sqlite.org/index.html)
- [<span data-ttu-id="452f8-109">Node.js</span><span class="sxs-lookup"><span data-stu-id="452f8-109">Node.js</span></span>](https://nodejs.org/en/)
- [<span data-ttu-id="452f8-110">Fils</span><span class="sxs-lookup"><span data-stu-id="452f8-110">Yarn</span></span>](https://yarnpkg.com/)

<span data-ttu-id="452f8-111">Vous devez également avoir un compte Microsoft personnel avec une boîte aux lettres sur Outlook.com, ou un compte scolaire ou scolaire Microsoft.</span><span class="sxs-lookup"><span data-stu-id="452f8-111">You should also have either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span> <span data-ttu-id="452f8-112">Si vous n'avez pas de compte Microsoft, deux options s'offrent à vous pour obtenir un compte gratuit :</span><span class="sxs-lookup"><span data-stu-id="452f8-112">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="452f8-113">Vous pouvez [vous inscrire à un nouveau compte Microsoft personnel.](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)</span><span class="sxs-lookup"><span data-stu-id="452f8-113">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="452f8-114">Vous pouvez [vous inscrire au programme Office 365 développeur](https://developer.microsoft.com/office/dev-program) pour obtenir un abonnement Office 365 gratuit.</span><span class="sxs-lookup"><span data-stu-id="452f8-114">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="452f8-115">Ce didacticiel a été écrit avec les versions suivantes des outils requis.</span><span class="sxs-lookup"><span data-stu-id="452f8-115">This tutorial was written with the following versions of the required tools.</span></span> <span data-ttu-id="452f8-116">Les étapes de ce guide peuvent fonctionner avec d'autres versions, mais elles n'ont pas été testées.</span><span class="sxs-lookup"><span data-stu-id="452f8-116">The steps in this guide may work with other versions, but that has not been tested.</span></span>
>
> - <span data-ttu-id="452f8-117">Ruby version 3.0.1</span><span class="sxs-lookup"><span data-stu-id="452f8-117">Ruby version 3.0.1</span></span>
> - <span data-ttu-id="452f8-118">SQLite3 version 3.35.5</span><span class="sxs-lookup"><span data-stu-id="452f8-118">SQLite3 version 3.35.5</span></span>
> - <span data-ttu-id="452f8-119">Node.js version 14.15.0</span><span class="sxs-lookup"><span data-stu-id="452f8-119">Node.js version 14.15.0</span></span>
> - <span data-ttu-id="452f8-120">La version 1.22.0</span><span class="sxs-lookup"><span data-stu-id="452f8-120">Yarn version 1.22.0</span></span>

## <a name="feedback"></a><span data-ttu-id="452f8-121">Commentaires</span><span class="sxs-lookup"><span data-stu-id="452f8-121">Feedback</span></span>

<span data-ttu-id="452f8-122">N'hésitez pas à nous faire part de vos commentaires sur ce didacticiel [dans GitHub référentiel.](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)</span><span class="sxs-lookup"><span data-stu-id="452f8-122">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>
