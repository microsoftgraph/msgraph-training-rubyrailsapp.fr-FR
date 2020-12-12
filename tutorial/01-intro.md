<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="76528-101">Ce didacticiel vous apprend à créer une application Web Ruby on rails qui utilise l’API Microsoft Graph pour récupérer des informations de calendrier pour un utilisateur.</span><span class="sxs-lookup"><span data-stu-id="76528-101">This tutorial teaches you how to build a Ruby on Rails web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="76528-102">Si vous préférez télécharger simplement le didacticiel terminé, vous pouvez le télécharger de deux manières.</span><span class="sxs-lookup"><span data-stu-id="76528-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="76528-103">Téléchargez le [démarrage rapide de Ruby](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) pour obtenir du code de travail en quelques minutes.</span><span class="sxs-lookup"><span data-stu-id="76528-103">Download the [Ruby quick start](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) to get working code in minutes.</span></span>
> - <span data-ttu-id="76528-104">Téléchargez ou clonez le [référentiel GitHub](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span><span class="sxs-lookup"><span data-stu-id="76528-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="76528-105">Conditions requises</span><span class="sxs-lookup"><span data-stu-id="76528-105">Prerequisites</span></span>

<span data-ttu-id="76528-106">Avant de commencer ce didacticiel, les outils suivants doivent être installés sur votre ordinateur de développement.</span><span class="sxs-lookup"><span data-stu-id="76528-106">Before you start this tutorial, you should have the following tools installed on your development machine.</span></span>

- [<span data-ttu-id="76528-107">Ruby</span><span class="sxs-lookup"><span data-stu-id="76528-107">Ruby</span></span>](https://www.ruby-lang.org/en/downloads/)
- [<span data-ttu-id="76528-108">SQLite3</span><span class="sxs-lookup"><span data-stu-id="76528-108">SQLite3</span></span>](https://sqlite.org/index.html)
- [<span data-ttu-id="76528-109">Node.js</span><span class="sxs-lookup"><span data-stu-id="76528-109">Node.js</span></span>](https://nodejs.org/en/)
- [<span data-ttu-id="76528-110">Ténacité</span><span class="sxs-lookup"><span data-stu-id="76528-110">Yarn</span></span>](https://yarnpkg.com/)

<span data-ttu-id="76528-111">Vous devez également disposer d’un compte Microsoft personnel disposant d’une boîte aux lettres sur Outlook.com ou d’un compte professionnel ou scolaire Microsoft.</span><span class="sxs-lookup"><span data-stu-id="76528-111">You should also have either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span> <span data-ttu-id="76528-112">Si vous n’avez pas de compte Microsoft, vous disposez de deux options pour obtenir un compte gratuit :</span><span class="sxs-lookup"><span data-stu-id="76528-112">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="76528-113">Vous pouvez vous [inscrire pour obtenir un nouveau compte Microsoft personnel](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span><span class="sxs-lookup"><span data-stu-id="76528-113">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="76528-114">Vous pouvez vous [inscrire au programme pour les développeurs office 365](https://developer.microsoft.com/office/dev-program) pour obtenir un abonnement gratuit à Office 365.</span><span class="sxs-lookup"><span data-stu-id="76528-114">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="76528-115">Ce didacticiel a été écrit avec les versions suivantes des outils requis.</span><span class="sxs-lookup"><span data-stu-id="76528-115">This tutorial was written with the following versions of the required tools.</span></span> <span data-ttu-id="76528-116">Les étapes de ce guide peuvent fonctionner avec d’autres versions, mais cela n’a pas été testé.</span><span class="sxs-lookup"><span data-stu-id="76528-116">The steps in this guide may work with other versions, but that has not been tested.</span></span>
>
> - <span data-ttu-id="76528-117">Ruby version 2.6.6.</span><span class="sxs-lookup"><span data-stu-id="76528-117">Ruby version 2.6.6.</span></span>
> - <span data-ttu-id="76528-118">Version SQLite3 3.31.1</span><span class="sxs-lookup"><span data-stu-id="76528-118">SQLite3 version 3.31.1</span></span>
> - <span data-ttu-id="76528-119">Node.js version 14.15.0</span><span class="sxs-lookup"><span data-stu-id="76528-119">Node.js version 14.15.0</span></span>
> - <span data-ttu-id="76528-120">1.22.0 de la version du fil</span><span class="sxs-lookup"><span data-stu-id="76528-120">Yarn version 1.22.0</span></span>

## <a name="feedback"></a><span data-ttu-id="76528-121">Commentaires</span><span class="sxs-lookup"><span data-stu-id="76528-121">Feedback</span></span>

<span data-ttu-id="76528-122">Veuillez fournir des commentaires sur ce didacticiel dans le [référentiel GitHub](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span><span class="sxs-lookup"><span data-stu-id="76528-122">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>
