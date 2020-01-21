---
title: Azure Bot Service を介してボットに認証を追加する - Bot Service
description: Azure Bot Service の認証機能を使用して SSO をボットに追加する方法について説明します。
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/04/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 75dc417f5ac3738b2b96a026860d8377f8ff9c25
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791321"
---
<!-- 

Related TODO:
- Check code in [Web Chat channel](https://docs.microsoft.com/azure/bot-service/bot-service-channel-connect-webchat?view=azure-bot-service-4.0)
- Check guidance in [DirectLine authentication](https://docs.microsoft.com/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-authentication?view=azure-bot-service-4.0)
-->

<!-- General TODO: (Feedback from CSE (Nafis))
- Add note that: token management is based on user ID
- Explain why/how to share existing website authentication with a bot.
- Risk: Even people who use a DirectLine token can be vulnerable to user ID impersonation.
    Docs/samples that show exchange of secret for token don't specify a user ID, so an attacker can impersonate a different user by modifying the ID client side. There's a [blog post](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fblog.botframework.com%2F2018%2F09%2F01%2Fusing-webchat-with-azure-bot-services-authentication%2F&data=02%7C01%7Cv-jofing%40microsoft.com%7Cee005e1c9d2c4f4e7ea508d6b231b422%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C636892323874079713&sdata=b0DWMxHzmwQvg5EJtlqKFDzR7fYKmg10fXma%2B8zGqEI%3D&reserved=0) that shows how to do this properly.
"Major issues":
- This doc is a sample walkthrough, but there's no deeper documentation explaining how the Azure Bot Service is handling tokens. How does the OAuth flow work? Where is it storing my users' access tokens? What's the security and best practices around using it?

"Minor issues":
- AAD v2 steps tell you to add delegated permission scopes during registration, but this shouldn't be necessary in AAD v2 due to dynamic scopes. (Ming, "This is currently necessary because scopes are not exposed through our runtime API. We don’t currently have a way for the developer to specify which scope he wants at runtime.")

- "The scope of the connection setting needs to have both openid and a resource in the Azure AD graph, such as Mail.Read." Unclear if I need to take some action at this point to make happen. Kind of out of context. I'm registering an AAD application in the portal, there's no connection setting
- Does the bot need all of these scopes for the samples? (e.g. "Read all users' basic profiles")
-->

# <a name="add-authentication-to-your-bot-via-azure-bot-service"></a>Azure Bot Service を介してボットに認証を追加する

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

Azure Bot Service および v4 SDK には新しいボット認証機能が追加されています。これにより、Azure AD (Azure Active Directory)、GitHub、Uber などの各種 ID プロバイダーにユーザーを認証するボットを容易に開発できます。 これらの機能は、一部のクライアント用の "_マジック コード検証_" を排除することで、ユーザー エクスペリエンスを向上させることができます。

これ以前は、ボットにおいて、OAuth コントローラーとログイン リンクを含め、ターゲット クライアントの ID とシークレットを保存し、ユーザー トークン管理を実行する必要がありました。 ボットはユーザーに対して Web サイトにサインインするように求め、これによりユーザーの本人確認に使用される "_マジック コード_" が生成されました。

OAuth コントローラーのホスティングやトークンのライフサイクルの管理は、すべて Azure Bot Service によって実行できるようになったため、ボット開発者がこれらを行う必要はなくなりました。

機能は、次のとおりです。

- チャネルの機能向上によって新しい WebChat や DirectLineJS ライブラリなどの新しい認証機能がサポートされ、6 桁のマジック コード検証の必要性を排除。
- Azure Portal で各種 OAuth アイデンティティ プロバイダーへの接続設定を追加、削除、および構成する機能の向上。
- Azure AD (v1 と v2 の両エンドポイント) や GitHub など、すぐに利用できる各種アイデンティティ プロバイダーのサポート。
- C# および Node.js の Bot Framework SDK の更新により、トークンの取得、OAuthCard の作成、TokenResponse イベントの処理が可能に。
- Azure AD に対して認証されるボットを作成する方法のサンプル。

Azure Bot Service で認証が処理されるしくみの詳細については、「[会話内のユーザー認証](bot-builder-concept-authentication.md)」を参照してください。

この記事の手順を応用して、このような機能を既存のボットに追加することができます。 以下のサンプル ボットは、新しい認証機能を示しています。

> [!NOTE]
> 認証機能は BotBuilder v3 でも使用できます。 ただし、この記事では v4 サンプル コードのみを扱います。

### <a name="about-this-sample"></a>このサンプルについて

Azure ボット リソースを作成する必要があります。また、以下のものが必要です。

1. 自分のボットを外部のリソース (Office 365 など) にアクセスできるようにするための Azure AD アプリ登録。
1. 別個のボット リソース。 そのボット リソースによって、ご自身のボットの資格情報が登録されます。ご自身のボット コードをローカルで実行している場合でも、これらの資格情報は認証機能のテストに必要です。

> [!IMPORTANT]
> Azure でボットを登録すると必ず、Azure AD アプリが割り当てられますが、 このアプリで保護されるのは、チャネルからボットへのアクセスです。 ユーザーに代わってボットが認証できるようにするアプリケーションごとに、追加の AAD アプリが必要です。

この記事では、Azure AD v1 または v2 トークンを使用して Microsoft Graph に接続するサンプル ボットを作成します。 また、関連付けられている Azure AD アプリを作成して登録する方法についても説明します。 このプロセスの一環として、[Microsoft/BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) GitHub リポジトリのコードを使用します。 この記事では、次のプロセスについて説明します。

- **ボット リソースの作成**
- **Azure AD アプリケーションの作成**
- **ボットへの Azure AD アプリケーションの登録**
- **ボットのサンプル コードの準備**

手順が終了すると、電子メールのチェックと送信、自分とその上司の情報の表示など、Azure AD アプリケーションに対するいくつかの単純なタスクに応答できるボットが完成します。このボットはローカルで実行されています。 これを行うために、ボットでは Azure AD アプリケーションからのトークンを Microsoft.Graph ライブラリに対して使用します。 OAuth サインイン機能をテストするためにご自身のボットを公開する必要はありませんが、ボットには有効な Azure アプリ ID とパスワードが必要になります。

### <a name="web-chat-and-direct-line-considerations"></a>Web チャットと Direct Line に関する考慮事項

<!-- Summarized from: https://blog.botframework.com/2018/09/25/enhanced-direct-line-authentication-features/ -->

> [!IMPORTANT]
> こちらの重要な[セキュリティに関する考慮事項](../rest-api/bot-framework-rest-direct-line-3-0-authentication.md#security-considerations)に留意してください。

## <a name="prerequisites"></a>前提条件

- [ボットの基本][concept-basics]、[状態の管理][concept-state]、[ダイアログ ライブラリ][concept-dialogs]、[連続して行われる会話フローを実装][simple-dialog]する方法、[ダイアログを再利用][component-dialogs]する方法に関する知識。
- Azure と OAuth 2.0 開発の知識。
- Visual Studio 2017 以降、Node.js、npm、git。
- 次のいずれかのサンプル。

| サンプル | BotBuilder のバージョン | 対象 |
|:---|:---:|:---|
| [**CSharp**][cs-auth-sample]、[**JavaScript**][js-auth-sample]、または [**Python**][python-auth-sample] の**ボット認証** | v4 | OAuthCard サポート |
| [**CSharp**][cs-msgraph-sample]、[**JavaScript**][js-msgraph-sample]、または [**Python**](https://aka.ms/bot-auth-msgraph-python-sample-code) の**ボット認証 MSGraph**| v4 |  OAuth 2 を使用した Microsoft Graph API サポート |

## <a name="create-your-bot-resource-on-azure"></a>Azure でご自身のボット リソースを作成する

[Azure portal](https://portal.azure.com/) を使用して、**ボット リソース**を作成します。

詳細については、「[Azure Bot Service を使用してボットを作成する](./abs-quickstart.md)」を参照してください。

## <a name="create-and-register-an-azure-ad-application"></a>Azure AD アプリケーションを作成して登録する

Microsoft Graph API に接続するためにご自身のボットが使用できる Azure AD アプリケーションが必要です。

このボットには Azure AD v1 または v2 エンドポイントを使用できます。
v1 と v2 の各エンドポイントの違いについては、[v1 と v2 の比較](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-compare)に関する記事と、[Azure AD v2.0 エンドポイントの概要](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview)に関する記事を参照してください。

### <a name="create-your-azure-ad-application"></a>Azure AD アプリケーションを作成する

次の手順を使用して、新しい Azure AD アプリケーションを作成します。 作成するアプリには v1 または v2 エンドポイントを使用できます。

> [!TIP]
> アプリケーションによって要求されたアクセス許可を委任することに同意できる、テナントで Azure AD プリケーションを作成し、登録する必要があります。

1. Azure portal で [[Azure Active Directory]][azure-aad-blade] パネルを開きます。
    適切なテナントにいない場合は、 **[ディレクトリの切り替え]** をクリックして適切なテナントに切り替えます (テナントを作成する方法については、[ポータルへのアクセスとテナントの作成](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant)に関するページをご覧ください)。
1. **[アプリの登録]** パネルを開きます。
1. **[アプリの登録]** パネルで、 **[新規登録]** をクリックします。
1. 必須のフィールドに入力してアプリ登録を作成します。

   1. アプリケーションに名前を付けます。
   1. ご自分のアプリケーションについて、 **[サポートされているアカウントの種類]** を選択します。 (このサンプルは、これらのオプションのどれを使用しても動作します。)
   1. **[リダイレクト URI]** で
       1. **[Web]** を選択します。
       1. URL を `https://token.botframework.com/.auth/web/redirect` に設定します。
   1. **[登録]** をクリックします。

      - アプリが作成された後、その **[概要]** ページが Azure によって表示されます。
      - **[アプリケーション (クライアント) ID]** の値を記録します。 この値は、後で Azure AD アプリケーションをご自身のボットに登録するときに、"_クライアント ID_" として使用します。
      - さらに、 **[ディレクトリ (テナント) ID]** の値を記録します。 これも、このアプリケーションを自分のボットに登録するために使用します。

1. ナビゲーション ウィンドウで **[証明書とシークレット]** をクリックして、自分のアプリケーションのシークレットを作成します。

   1. **[クライアント シークレット]** で、 **[新しいクライアント シークレット]** をクリックします。
   1. 説明を追加します (`bot login` など)。必要に応じてこのアプリのために作成する他のシークレットからこれを識別するためです。
   1. **[期限]** を **[無期限]** に設定します。
   1. **[追加]** をクリックします。
   1. このページを離れる前に、シークレットを記録してください。 この値は、後で Azure AD アプリケーションをご自身のボットに登録するときに、"_クライアント シークレット_" として使用します。

1. ナビゲーション ウィンドウで、 **[API のアクセス許可]** をクリックし、 **[API のアクセス許可]** パネルを開きます。 アプリの API アクセス許可を明示的に設定するのがベスト プラクティスです。

   1. **[アクセス許可の追加]** をクリックし、 **[API アクセス許可の要求]** ウィンドウを表示します。
   1. このサンプルでは、 **[Microsoft API]** と **[Microsoft Graph]** を選択します。
   1. **[委任されたアクセス許可]** を選択し、必要とするアクセス許可が選択されていることを確認します。 このサンプルでは、これらのアクセス許可が必要です。

      > [!NOTE]
      > **[管理者の同意が必要]** とマークされているアクセス許可は、ユーザーとテナント管理者の両方がログインすることを要求するため、ボットではこれらのアクセス許可を避けるのが一般的です。

      - **openid**
      - **profile**
      - **Mail.Read**
      - **Mail.Send**
      - **User.Read**
      - **User.ReadBasic.All**

   1. **[アクセス許可の追加]** をクリックします。 (ユーザーは、ボットを通じてこのアプリに初めてアクセスする際に、同意を付与する必要があります。)

これで、Azure AD アプリケーションが構成されました。

### <a name="register-your-azure-ad-application-with-your-bot"></a>Azure AD アプリケーションをボットに登録する

次に、作成した Azure AD アプリケーションをボットに登録します。

#### <a name="azure-ad-v1"></a>Azure AD v1

1. [Azure Portal](https://portal.azure.com/) で、ボットのリソース ページに移動します。
1. **[設定]** をクリックします。
1. ページ下部付近の **[OAuth Connection Settings]\(OAuth 接続設定\)** で、 **[設定の追加]** をクリックします。
1. 次のようにフォームに入力します。

    1. **[名前]** に、接続の名前を入力します。 この名前はご自身のボット コードで使用します。
    1. **[サービス プロバイダー]** で、 **[Azure Active Directory]** を選択します。 これを選択すると、Azure AD に固有のフィールドが表示されます。
    1. **[クライアント ID]** に、Azure AD v1 アプリケーションの設定時に記録したアプリケーション (クライアント) ID を入力します。
    1. **[クライアント シークレット]** に、作成したシークレットを入力して、Azure AD アプリへのアクセスをボットに許可します。
    1. **[付与タイプ]** に、`authorization_code` と入力します。
    1. **[ログイン URL]** に、`https://login.microsoftonline.com` と入力します。
    1. **[テナント ID]** に、AAD アプリを作成したときに選択したサポートされるアカウントの種類に基づいて、AAD アプリ用に以前記録した**ディレクトリ (テナント) ID** を入力するか、「**common**」と入力します。 割り当てる値を決定するには、次の条件に従います。

        - AAD アプリの作成時に、 *[Accounts in this organizational directory only (Microsoft only - Single tenant)]\(この組織のディレクトリ内のアカウントのみ (Microsoft のみ - シングル テナント)\)* または *[Accounts in any organizational directory(Microsoft AAD directory - Multi tenant)]\(任意の組織のディレクトリ内のアカウント (Microsoft AAD ディレクトリ - マルチテナント)\)* を選択した場合、AAD アプリ用に以前記録した**テナント ID** を入力します。

        - ただし、 *[Accounts in any organizational directory (Any AAD directory - Multi tenant and personal Microsoft accounts e.g. Skype, Xbox, Outlook.com)]\(任意の組織のディレクトリ内のアカウント (任意の AAD ディレクトリ - マルチテナント) と個人の Microsoft アカウント (Skype、Xbox、Outlook.com など)\)* を選択した場合、テナント ID の代わりに「**common**」という語を入力します。 それ以外の場合、AAD アプリは ID が選択されているテナントを通じて検証され、個人の MS アカウントは除外されます。

       これは、認証可能なユーザーに関連付けられるテナントになります。

    1. **[リソース URL]** に、「`https://graph.microsoft.com/`」と入力します。
    1. **[スコープ]** は空白のままにします。

1. **[保存]** をクリックします。

> [!NOTE]
> これらの値によって、アプリケーションは Microsoft Graph API 経由で Office 365 データにアクセスできます。

#### <a name="azure-ad-v2"></a>Azure AD v2

1. [Azure Portal](https://portal.azure.com/) で、ボットの [Bot Channels Registration]\(ボット チャネル登録\) ページに移動します。
1. **[設定]** をクリックします。
1. ページ下部付近の **[OAuth Connection Settings]\(OAuth 接続設定\)** で、 **[設定の追加]** をクリックします。
1. 次のようにフォームに入力します。

    1. **[名前]** に、接続の名前を入力します。 ボットのコードで使用します。
    1. **[サービス プロバイダー]** で、 **[Azure Active Directory v2]** を選択します。 これを選択すると、Azure AD に固有のフィールドが表示されます。
    1. **[クライアント ID]** に、Azure AD v1 アプリケーションの設定時に記録したアプリケーション (クライアント) ID を入力します。
    1. **[クライアント シークレット]** に、作成したシークレットを入力して、Azure AD アプリへのアクセスをボットに許可します。
    1. **[テナント ID]** に、AAD アプリを作成したときに選択したサポートされるアカウントの種類に基づいて、AAD アプリ用に以前記録した**ディレクトリ (テナント) ID** を入力するか、「**common**」と入力します。 割り当てる値を決定するには、次の条件に従います。

        - AAD アプリの作成時に、 *[Accounts in this organizational directory only (Microsoft only - Single tenant)]\(この組織のディレクトリ内のアカウントのみ (Microsoft のみ - シングル テナント)\)* または *[Accounts in any organizational directory(Microsoft AAD directory - Multi tenant)]\(任意の組織のディレクトリ内のアカウント (Microsoft AAD ディレクトリ - マルチテナント)\)* を選択した場合、AAD アプリ用に以前記録した**テナント ID** を入力します。

        - ただし、 *[Accounts in any organizational directory (Any AAD directory - Multi tenant and personal Microsoft accounts e.g. Skype, Xbox, Outlook.com)]\(任意の組織のディレクトリ内のアカウント (任意の AAD ディレクトリ - マルチテナント) と個人の Microsoft アカウント (Skype、Xbox、Outlook.com など)\)* を選択した場合、テナント ID の代わりに「**common**」という語を入力します。 それ以外の場合、AAD アプリは ID が選択されているテナントを通じて検証され、個人の MS アカウントは除外されます。

       これは、認証可能なユーザーに関連付けられるテナントになります。

    1. **[スコープ]** には、アプリケーション登録から選択したアクセス許可の名前を入力します。`Mail.Read Mail.Send openid profile User.Read User.ReadBasic.All`

        > [!NOTE]
        > Azure AD v2 の場合、 **[スコープ]** はスペースで区切った値のリストであり、大文字と小文字が区別されます。

1. **[保存]** をクリックします。

> [!NOTE]
> これらの値によって、アプリケーションは Microsoft Graph API 経由で Office 365 データにアクセスできます。

### <a name="test-your-connection"></a>接続をテストする

1. 接続エントリをクリックして、作成した接続を開きます。
1. **[Service Provider Connection Setting]\(サービス プロバイダー接続設定\)** ウィンドウの上部にある **[Test Connection]\(接続のテスト\)** をクリックします。
1. 初回は新しいブラウザー タブが開き、アプリが要求しているアクセス許可の一覧が表示され、承認を求められます。
1. **[Accept]\(受け入れる\)** をクリックします。
1. **[\<your-connection-name> への接続テストに成功しました]** ページにリダイレクトされます。

ボット コードでこの接続名を使用してユーザー トークンを取得できるようになりました。

## <a name="prepare-the-bot-code"></a>ボット コードを準備する

このプロセスを完了するには、ボットのアプリ ID とパスワードが必要になります。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

<!-- TODO: Add guidance (once we have it) on how not to hard-code IDs and ABS auth. -->

1. GitHub リポジトリから、使用したいサンプルを複製します: [**ボット認証**][cs-auth-sample]または[**ボット認証 MSGraph**][cs-msgraph-sample] を複製します。
1. **appsettings.json** を更新します。

    - `ConnectionName` を、お使いのボットに追加した OAuth 接続設定の名前に設定します。
    - `MicrosoftAppId` および `MicrosoftAppPassword` を、お使いのボットのアプリ ID とアプリ シークレットに設定します。

      お使いのボット シークレットに含まれる文字によっては、パスワードを XML でエスケープすることが必要な場合があります。 たとえば、アンパサンド (&) は `&amp;` のようにエンコードする必要があります。

    [!code-json[appsettings](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/appsettings.json)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. 使用する github リポジトリから [**ボット認証**][js-auth-sample]または[**ボット認証 MSGraph**][js-msgraph-sample] を複製します。
1. **.env** を更新します。

    - `connectionName` を、お使いのボットに追加した OAuth 接続設定の名前に設定します。
    - `MicrosoftAppId` および `MicrosoftAppPassword` の値を、お使いのボットのアプリ ID とアプリ シークレットに設定します。

      お使いのボット シークレットに含まれる文字によっては、パスワードを XML でエスケープすることが必要な場合があります。 たとえば、アンパサンド (&) は `&amp;` のようにエンコードする必要があります。

    [!code-txt[.env](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/.env)]

# <a name="pythontabpython"></a>[Python](#tab/python)

1. Github リポジトリから[**ボット認証**][python-auth-sample]サンプルを複製します。
1. **config.py** を更新します。

    - `ConnectionName` を、お使いのボットに追加した OAuth 接続設定の名前に設定します。
    - `MicrosoftAppId` および `MicrosoftAppPassword` を、お使いのボットのアプリ ID とアプリ シークレットに設定します。

      お使いのボット シークレットに含まれる文字によっては、パスワードを XML でエスケープすることが必要な場合があります。 たとえば、アンパサンド (&) は `&amp;` のようにエンコードする必要があります。

    [!code-python[config](~/../botbuilder-python/samples/python/18.bot-authentication/config.py)]

---

**Microsoft アプリ ID** と **Microsoft アプリ パスワード**の値を取得する方法がわからない場合は、[こちらの説明に従って](../bot-service-quickstart-registration.md#get-registration-password)新しいパスワードを作成できます。

> [!NOTE]
> ここで、このボット コードを Azure サブスクリプションに発行 (プロジェクトを右クリックして **[発行]** を選択) することもできますが、この記事では不要です。 Azure Portal でボットを構成するときに使用したアプリケーションとホスティング プランを使用する発行構成を設定する必要があります。

## <a name="test-the-bot-using-the-emulator"></a>エミュレーターを使ってボットをテストする

[Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) をインストールします (まだインストールしていない場合)。 「[エミュレーターを使用したデバッグ](../bot-service-debug-emulator.md)」もご覧ください。

<!-- auth config steps -->
ボット サンプル ログインを機能させるには、「[認証用にエミュレーターを構成する](../bot-service-debug-emulator.md#configure-the-emulator-for-authentication)」に示されているようにエミュレーターを構成する必要があります。

### <a name="testing"></a>テスト

認証メカニズムを構成したら、実際のボット サンプル テストを実行できます。  

1. お使いのマシン上でローカルでボット サンプルを実行します。
1. エミュレーターを起動します。
1. ボットに接続するときに、ボットのアプリ ID とパスワードを入力する必要があります。
    - Azure アプリの登録からアプリ ID とパスワードを取得します。 これらは、`appsettings.json` または `.env` ファイルでボット アプリに割り当てたものと同じ値です。 エミュレーターで、これらの値を構成ファイルで、または初めてボットに接続するときに割り当てます。
    - ボット コード内のパスワードを XML エスケープする必要がある場合は、ここでもそれを行う必要があります。
1. `help` と入力すると、ボットで使用できるコマンドの一覧が表示され、認証機能をテストします。
1. サインインした後は、サインアウトするまで、資格情報を再度入力する必要はありません。
1. サインアウトして認証をキャンセルするには、`logout` と入力します。

> [!NOTE]
> ボット認証では Bot Connector Service を使用する必要があります。 このサービスは、お使いのボットのボット チャネル登録情報にアクセスします。

## <a name="bot-authentication-example"></a>ボット認証の例

**ボット認証**サンプルでは、ダイアログは、ユーザーのログイン後、ユーザー トークンを取得するように設計されています。

![サンプル出力](media/how-to-auth/auth-bot-test.png)

## <a name="bot-authentication-msgraph-example"></a>ボット認証 MSGraph の例

**ボット認証 MSGraph** サンプルでは、ダイアログは、ユーザーのログイン後、制限された一部のコマンドを受け取るように設計されています。

![サンプル出力](media/how-to-auth/msgraph-bot-test.png)

---

## <a name="additional-information"></a>関連情報

ユーザーがボットに何らかの処理を要求し、それによってボットでユーザーのログインが必要になった場合、ボットによって `OAuthPrompt` が使用され、特定の接続に必要なトークンの取得が開始されます。 `OAuthPrompt` では、以下で構成されるトークン取得フローが作成されます。

1. 既に Azure Bot Service に現在のユーザーおよび接続用のトークンがあるかどうかを確認するためのチェック。 トークンがある場合は、そのトークンが返されます。
1. キャッシュされたトークンが Azure Bot Service にない場合は、`OAuthCard` が作成されます。これは、ユーザーがクリックできるサインイン ボタンです。
1. ユーザーが `OAuthCard` サインイン ボタンをクリックしたら、Azure Bot Service によって、ユーザーのトークンがボットに直接送信されます。または、チャット ウィンドウで 6 桁の認証コードがユーザーに表示されます。
1. ユーザーに認証コードが表示される場合は、ボットによって、この認証コードがユーザーのトークンと交換されます。

次のセクションでは、一般的な認証タスクが、サンプルによってどのように実装されているかを説明します。

### <a name="use-an-oauth-prompt-to-sign-the-user-in-and-get-a-token"></a>OAuth プロンプトを使用してユーザーをサインインさせて、トークンを取得する

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

![ボット アーキテクチャ](media/how-to-auth/architecture.png)

<!-- The two authentication samples have nearly identical architecture. Using 18.bot-authentication for the sample code. -->

**Dialogs\MainDialog.cs**

OAuth プロンプトを、コンストラクター内の **MainDialog** に追加します。 ここでは、接続名の値は **appsettings.json** ファイルから取得されました。

[!code-csharp[Add OAuthPrompt](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=23-31)]

ダイアログ ステップ内で、`BeginDialogAsync` を使用して OAuth プロンプトを起動します。これによりユーザーのサインインが求められます。

- ユーザーがサインイン済みの場合は、ユーザーに問い合わせることなく、トークン応答イベントが生成されます。
- それ以外の場合は、ユーザーのサインインが求められます。 ユーザーがサインインを試みた後、Azure Bot Service によってトークン応答イベントが送信されます。

[!code-csharp[Use the OAuthPrompt](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=49)]

次のダイアログ ステップ内で、前のステップの結果としてトークンが存在することを確認します。 null でない場合、ユーザーは正常にサインインしています。

[!code-csharp[Get the OAuthPrompt result](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=54-56)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

![ボット アーキテクチャ](media/how-to-auth/architecture-js.png)

**dialogs/mainDialog.js**

OAuth プロンプトを、コンストラクター内の **MainDialog** に追加します。 ここでは、接続名の値は **.env** ファイルから取得されました。

[!code-javascript[Add OAuthPrompt](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=24-29)]

ダイアログ ステップ内で、`beginDialog` を使用して OAuth プロンプトを起動します。これによりユーザーのサインインが求められます。

- ユーザーがサインイン済みの場合は、ユーザーに問い合わせることなく、トークン応答イベントが生成されます。
- それ以外の場合は、ユーザーのサインインが求められます。 ユーザーがサインインを試みた後、Azure Bot Service によってトークン応答イベントが送信されます。

[!code-javascript[Use OAuthPrompt](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=57)]

次のダイアログ ステップ内で、前のステップの結果としてトークンが存在することを確認します。 null でない場合、ユーザーは正常にサインインしています。

[!code-javascript[Get OAuthPrompt result](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=62-63)]


# <a name="pythontabpython"></a>[Python](#tab/python)

![ボット アーキテクチャ](media/how-to-auth/architecture-python.png)

**dialogs/main_dialog.py**

OAuth プロンプトを、コンストラクター内の **MainDialog** に追加します。 ここでは、接続名の値は **config.py** ファイルから取得されました。

[!code-python[Add OAuthPrompt](~/../botbuilder-python/samples/python/18.bot-authentication/dialogs/main_dialog.py?range=34-44)]

ダイアログ ステップ内で、`begin_dialog` を使用して OAuth プロンプトを起動します。これによりユーザーのサインインが求められます。

- ユーザーがサインイン済みの場合は、ユーザーに問い合わせることなく、トークン応答イベントが生成されます。
- それ以外の場合は、ユーザーのサインインが求められます。 ユーザーがサインインを試みた後、Azure Bot Service によってトークン応答イベントが送信されます。

[!code-python[Add OAuthPrompt](~/../botbuilder-python/samples/python/18.bot-authentication/dialogs/main_dialog.py?range=49)]

次のダイアログ ステップ内で、前のステップの結果としてトークンが存在することを確認します。 null でない場合、ユーザーは正常にサインインしています。

[!code-python[Add OAuthPrompt](~/../botbuilder-python/samples/python/18.bot-authentication/dialogs/main_dialog.py?range=54-65)]

---

### <a name="wait-for-a-tokenresponseevent"></a>TokenResponseEvent の待機

OAuth プロンプトを起動すると、そのプロンプトは、ユーザーのトークン取得元となるトークン応答イベントを待ちます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\AuthBot.cs**

**AuthBot** は `ActivityHandler` から派生し、トークン応答イベント アクティビティを明示的に処理します。 ここではアクティブなダイアログを続行します。これにより OAuth プロンプトでイベントを処理し、トークンを取得できます。

[!code-csharp[OnTokenResponseEventAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Bots/AuthBot.cs?range=32-38)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/authBot.js**

**AuthBot** は `ActivityHandler` から派生し、トークン応答イベント アクティビティを明示的に処理します。 ここではアクティブなダイアログを続行します。これにより OAuth プロンプトでイベントを処理し、トークンを取得できます。

[!code-javascript[onTokenResponseEvent](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/bots/authBot.js?range=29-31)]

# <a name="pythontabpython"></a>[Python](#tab/python)

**bots/auth_bot.py**

**AuthBot** は、トークン応答イベント アクティビティを明示的に処理します。 ここではアクティブなダイアログを続行します。これにより OAuth プロンプトでイベントを処理し、トークンを取得できます。

[!code-python[on_token_response_event](~/../botbuilder-python/samples/python/18.bot-authentication/bots/auth_bot.py?range=38-44)]

---

### <a name="log-the-user-out"></a>ユーザーをログアウトする

接続のタイムアウトに依存するのではなく、ユーザーが明示的にサインアウトまたはログアウトできるようにすることをお勧めします。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\LogoutDialog.cs**

[!code-csharp[Allow logout](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/LogoutDialog.cs?range=44-61&highlight=11)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/logoutDialog.js**

[!code-javascript[Allow logout](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/logoutDialog.js?range=31-42&highlight=7)]

# <a name="pythontabpython"></a>[Python](#tab/python)

**dialogs/logout_dialog.py**

[!code-python[allow logout](~/../botbuilder-python/samples/python/18.bot-authentication/dialogs/logout_dialog.py?range=27-34&highlight=6)]

---

### <a name="adding-teams-authentication"></a>Teams 認証の追加

Teams は、OAuth に関して他のチャネルとは多少異なる動作をするため、認証を適切に実装するにはいくつかの変更が必要です。 Teams Authentication Bot サンプルからコードを追加します ([C#][cs-teams-auth-sample]/[JavaScript][js-teams-auth-sample])。
 
他のチャネルと Teams の違いの 1 つは、Teams がボットに*イベント* アクティビティではなく*呼び出し*アクティビティを送信することです。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)  
**Bots/TeamsBot.cs**  
[!code-csharp[Invoke Activity](~/../botbuilder-samples/samples/csharp_dotnetcore/46.teams-auth/Bots/TeamsBot.cs?range=34-42&highlight=1)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)   

**bots/teamsBot.js**  
[!code-javascript[Invoke Activity](~/../botbuilder-samples/samples/javascript_nodejs/46.teams-auth/bots/teamsBot.js?range=16-25&highlight=1)]

# <a name="pythontabpython"></a>[Python](#tab/python)

現在、Microsoft Teams は、認証とボットとの統合方法が若干異なります。 認証については、[Teams のドキュメント](https://aka.ms/teams-docs)を参照してください。

---

*OAuth プロンプト*を使用する場合は、この呼び出しアクティビティをダイアログに転送する必要があります。 ここでは、`TeamsActivityHandler` でこれを行います。 次のコードをメイン ダイアログ ファイルに追加します。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)  
**Bots/DialogBot.cs**  
[!code-csharp[Dialogs Handler](~/../botbuilder-samples/samples/csharp_dotnetcore/46.teams-auth/Bots/DialogBot.cs?range=19)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)  
**Bots/dialogBot.js**  
[!code-javascript[Dialogs Handler](~/../botbuilder-samples/samples/javascript_nodejs/46.teams-auth/bots/dialogBot.js?range=6)]

# <a name="pythontabpython"></a>[Python](#tab/python)

現在、Microsoft Teams は、認証とボットとの統合方法が若干異なります。 認証については、[Teams のドキュメント](https://aka.ms/teams-docs)を参照してください。

---

最後に必ず、ボットのフォルダーの最上位レベルに適切な `TeamsActivityHandler` ファイル (C# ボットの場合は `TeamsActivityHandler.cs`、Javascript ボットの場合は `teamsActivityHandler.js`) を追加してください。

`TeamsActivityHandler` は*メッセージの反応*アクティビティも送信します。 メッセージの反応アクティビティは、*返信先 ID* フィールドを使用して元のアクティビティを参照します。 このアクティビティは、Microsoft Teams の[アクティビティ フィード][teams-activity-feed]でも表示される必要があります。

> [!NOTE]
> マニフェストを作成して、`validDomains` セクションに `token.botframework.com` を含める必要があります。そうしないと、 OAuthCard の **[サインイン]** ボタンをクリックしても、認証ウィンドウは開きません。 [App Studio](https://docs.microsoft.com/microsoftteams/platform/get-started/get-started-app-studio) を使用して、マニフェストを生成してください。

### <a name="further-reading"></a>関連項目

- 「[Bot Framework のその他のリソース](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help)」に追加サポートのリンクがあります。
- [Bot Framework SDK](https://github.com/microsoft/botbuilder) のリポジトリでは、Bot Builder SDK に関連付けられているリポジトリ、サンプル、ツール、および仕様の詳細を確認できます。

<!-- Footnote-style links -->

[Azure portal]: https://ms.portal.azure.com
[azure-aad-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview
[aad-registration-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredAppsPreview

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-dialog]: bot-builder-dialog-manage-conversation-flow.md
[dialog-prompts]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-auth-sample]: https://aka.ms/v4cs-bot-auth-sample
[js-auth-sample]: https://aka.ms/v4js-bot-auth-sample
[python-auth-sample]: https://aka.ms/bot-auth-python-sample-code

[cs-msgraph-sample]: https://aka.ms/v4cs-auth-msgraph-sample
[js-msgraph-sample]: https://aka.ms/v4js-auth-msgraph-sample
[cs-teams-auth-sample]:https://aka.ms/cs-teams-auth-sample
[js-teams-auth-sample]:https://aka.ms/js-teams-auth-sample
[teams-activity-feed]:[https://aka.ms/teams-activity-feed
