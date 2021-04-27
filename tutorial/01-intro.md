<!-- markdownlint-disable MD002 MD041 -->

Ce didacticiel vous apprend à créer une application web Ruby on Rails qui utilise l'API Microsoft Graph pour récupérer les informations de calendrier d'un utilisateur.

> [!TIP]
> Si vous préférez simplement télécharger le didacticiel terminé, vous pouvez le télécharger de deux manières.
>
> - Téléchargez [le démarrage rapide de Ruby](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) pour obtenir du code de travail en quelques minutes.
> - Téléchargez ou clonez [le GitHub de données.](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)

## <a name="prerequisites"></a>Configuration requise

Avant de commencer ce didacticiel, vous devez avoir installé les outils suivants sur votre ordinateur de développement.

- [Ruby](https://www.ruby-lang.org/en/downloads/)
- [SQLite3](https://sqlite.org/index.html)
- [Node.js](https://nodejs.org/en/)
- [Fils](https://yarnpkg.com/)

Vous devez également avoir un compte Microsoft personnel avec une boîte aux lettres sur Outlook.com, ou un compte scolaire ou scolaire Microsoft. Si vous n'avez pas de compte Microsoft, deux options s'offrent à vous pour obtenir un compte gratuit :

- Vous pouvez [vous inscrire à un nouveau compte Microsoft personnel.](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)
- Vous pouvez [vous inscrire au programme Office 365 développeur](https://developer.microsoft.com/office/dev-program) pour obtenir un abonnement Office 365 gratuit.

> [!NOTE]
> Ce didacticiel a été écrit avec les versions suivantes des outils requis. Les étapes de ce guide peuvent fonctionner avec d'autres versions, mais elles n'ont pas été testées.
>
> - Ruby version 3.0.1
> - SQLite3 version 3.35.5
> - Node.js version 14.15.0
> - La version 1.22.0

## <a name="feedback"></a>Commentaires

N'hésitez pas à nous faire part de vos commentaires sur ce didacticiel [dans GitHub référentiel.](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)
