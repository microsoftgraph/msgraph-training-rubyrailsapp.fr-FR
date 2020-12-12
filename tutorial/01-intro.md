<!-- markdownlint-disable MD002 MD041 -->

Ce didacticiel vous apprend à créer une application Web Ruby on rails qui utilise l’API Microsoft Graph pour récupérer des informations de calendrier pour un utilisateur.

> [!TIP]
> Si vous préférez télécharger simplement le didacticiel terminé, vous pouvez le télécharger de deux manières.
>
> - Téléchargez le [démarrage rapide de Ruby](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) pour obtenir du code de travail en quelques minutes.
> - Téléchargez ou clonez le [référentiel GitHub](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).

## <a name="prerequisites"></a>Conditions requises

Avant de commencer ce didacticiel, les outils suivants doivent être installés sur votre ordinateur de développement.

- [Ruby](https://www.ruby-lang.org/en/downloads/)
- [SQLite3](https://sqlite.org/index.html)
- [Node.js](https://nodejs.org/en/)
- [Ténacité](https://yarnpkg.com/)

Vous devez également disposer d’un compte Microsoft personnel disposant d’une boîte aux lettres sur Outlook.com ou d’un compte professionnel ou scolaire Microsoft. Si vous n’avez pas de compte Microsoft, vous disposez de deux options pour obtenir un compte gratuit :

- Vous pouvez vous [inscrire pour obtenir un nouveau compte Microsoft personnel](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).
- Vous pouvez vous [inscrire au programme pour les développeurs office 365](https://developer.microsoft.com/office/dev-program) pour obtenir un abonnement gratuit à Office 365.

> [!NOTE]
> Ce didacticiel a été écrit avec les versions suivantes des outils requis. Les étapes de ce guide peuvent fonctionner avec d’autres versions, mais cela n’a pas été testé.
>
> - Ruby version 2.6.6.
> - Version SQLite3 3.31.1
> - Node.js version 14.15.0
> - 1.22.0 de la version du fil

## <a name="feedback"></a>Commentaires

Veuillez fournir des commentaires sur ce didacticiel dans le [référentiel GitHub](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).
