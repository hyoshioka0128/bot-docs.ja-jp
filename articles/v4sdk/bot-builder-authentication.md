---
title: Azure Bot Service を介してボットに認証を追加する | Microsoft Docs
description: Azure Bot Service の認証機能を使用して SSO をボットに追加する方法について説明します。
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 04/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4fd61d5d68b5b7b3a535afdd47d635eef3820622
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033630"
---
<!-- Related TODO:
- Check code in [Web Chat channel](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-channel-connect-webchat?view=azure-bot-service-4.0)
- Check guidance in [DirectLine authentication](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-authentication?view=azure-bot-service-4.0)
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

この記事の手順を応用して、このような機能を既存のボットに追加することができます。 以下のサンプル ボットは、新しい認証機能を示しています。

> [!NOTE]
> 認証機能は BotBuilder v3 でも使用できます。 ただし、この記事では v4 サンプル コードのみを扱います。

### <a name="about-this-sample"></a>このサンプルについて

Azure ボット リソースを作成する必要があります。また、新しい Azure AD (v1 または v2) アプリケーションを作成して、ボットが Office 365 にアクセスできるようにする必要があります。 そのボット リソースによって、ご自身のボットの資格情報が登録されます。ご自身のボット コードをローカルで実行している場合でも、これらの資格情報は認証機能のテストに必要です。

> [!IMPORTANT]
> Azure でボットを登録すると必ず、Azure AD アプリが割り当てられますが、 このアプリで保護されるのは、チャネルからボットへのアクセスです。
ユーザーに代わってボットが認証できるようにするアプリケーションごとに、追加の AAD アプリが必要です。

この記事では、Azure AD v1 または v2 トークンを使用して Microsoft Graph に接続するサンプル ボットを作成します。 また、関連付けられている Azure AD アプリを作成して登録する方法についても説明します。 このプロセスの一環として、[Microsoft/BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) GitHub リポジトリのコードを使用します。 この記事では、次のプロセスについて説明します。

- **ボット リソースの作成**
- **Azure AD アプリケーションの作成**
- **ボットへの Azure AD アプリケーションの登録**
- **ボットのサンプル コードの準備**

手順が終了すると、電子メールのチェックと送信、自分とその上司の情報の表示など、Azure AD アプリケーションに対するいくつかの単純なタスクに応答できるボットが完成します。このボットはローカルで実行されています。 これを行うために、ボットでは Azure AD アプリケーションからのトークンを Microsoft.Graph ライブラリに対して使用します。 OAuth サインイン機能をテストするためにご自身のボットを公開する必要はありませんが、ボットには有効な Azure アプリ ID とパスワードが必要になります。

これらの認証機能は、他の種類のボットとも連動します。 ただし、この記事では登録のみのボットを使用します。

### <a name="web-chat-and-direct-line-considerations"></a>Web チャットと Direct Line に関する考慮事項

<!-- Summarized from: https://blog.botframework.com/2018/09/25/enhanced-direct-line-authentication-features/ -->

Web チャットで Azure Bot Service 認証を使用する場合、考慮すべき重要なセキュリティの問題がいくつかあります。

1. 攻撃者が自身を他の誰かであるとボットに思わせる偽装を防ぎます。 Web チャットでは、攻撃者が自分の Web チャット インスタンスのユーザー ID を変えて、他の誰かになりすまします。

    これを防ぐために、ユーザー ID を推測できないようにします。 Direct Line チャネルで強化された認証オプションを有効にすると、Azure Bot Service が、ユーザー ID の変更を検出して拒否できます。 Direct Line からのご自身のボットへのメッセージのユーザー ID は、Web チャットを初期化したときに使ったものと必ず同じになります。 この機能では、ユーザー ID は必ず `dl_` で始まる必要があることに注意してください。

1. 適切なユーザーがサインインしていることを確認します。 ユーザーには、チャネルの ID と ID プロバイダーの ID の 2 つの ID があります。 Web チャットでは、Azure Bot Service によって、サインイン プロセスが必ず Web チャットと同じブラウザー セッションで完了することを保証できます。

    この保護を有効にするには、ボットの Web チャット クライアントをホストできる信頼されたドメインの一覧を含む Direct Line トークンを使用して、Web チャットを開始します。 次に、Direct Line 構成ページで信頼されているドメイン (origin) の一覧を静的に指定します。

## <a name="prerequisites"></a>前提条件

- [ボットの基本][concept-basics]、[状態の管理][concept-state]、[ダイアログ ライブラリ][concept-dialogs]、[連続して行われる会話フローを実装][simple-dialog]する方法、[ダイアログ プロンプトを使用してユーザー入力を収集][dialog-prompts]する方法、および[ダイアログを再利用][component-dialogs]する方法に関する知識。
- Azure と OAuth 2.0 開発の知識。
- Visual Studio 2017 以降、Node.js、npm、git。
- 次のいずれかのサンプル。

| サンプル | BotBuilder のバージョン | 対象 |
|:---|:---:|:---|
| [**CSharp**][cs-auth-sample] または [**JavaScript**][js-auth-sample] の**ボット認証** | v4 | OAuthCard サポート |
| [**CSharp**][cs-msgraph-sample] または [**JavaScript**][js-msgraph-sample] の**ボット認証 MSGraph** | v4 |  OAuth 2 を使用した Microsoft Graph API サポート |

## <a name="create-your-bot-resource-on-azure"></a>Azure でご自身のボット リソースを作成する

[Azure Portal](https://portal.azure.com/) を使用して**ボット チャネル登録**を作成します。

## <a name="create-and-register-an-azure-ad-application"></a>Azure AD アプリケーションを作成して登録する

Microsoft Graph API に接続するためにご自身のボットが使用できる Azure AD アプリケーションが必要です。

このボットには Azure AD v1 または v2 エンドポイントを使用できます。
v1 と v2 の各エンドポイントの違いについては、[v1 と v2 の比較](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-compare)に関する記事と、[Azure AD v2.0 エンドポイントの概要](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview)に関する記事を参照してください。

### <a name="create-your-azure-ad-application"></a>Azure AD アプリケーションを作成する

次の手順を使用して、新しい Azure AD アプリケーションを作成します。 作成するアプリには v1 または v2 エンドポイントを使用できます。

> [!TIP]
> 管理者権限があるテナントで Azure AD アプリケーションを作成し、登録する必要があります。

# <a name="azure-ad-v1tabaadv1"></a>[Azure AD v1](#tab/aadv1)

1. Azure portal で [[Azure Active Directory]][azure-aad-blade] パネルを開きます。
    適切なテナントにいない場合は、**[ディレクトリの切り替え]** をクリックして適切なテナントに切り替えます  (テナントを作成する方法については、[ポータルへのアクセスとテナントの作成](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-access-create-new-tenant)に関するページをご覧ください)。
1. **[アプリの登録]** パネルを開きます。
1. **[アプリの登録]** パネルで、**[新しいアプリケーションの登録]** をクリックします。
1. 必須のフィールドに入力してアプリ登録を作成します。

   1. アプリケーションに名前を付けます。
   1. **[アプリケーションの種類]** を **[Web アプリ/API]** に設定します。
   1. **[サインオン URL]** を `https://token.botframework.com/.auth/web/redirect` に設定します。
   1. **Create** をクリックしてください。

      - 作成されると、**[登録済みのアプリ]** ウィンドウに表示されます。
      - **[アプリケーション ID]** の値をメモします。 この値は、後で Azure AD アプリケーションをご自身のボットに登録するときに、"_クライアント ID_" として使用します。

1. **[設定]** をクリックしてアプリケーションを構成します。
1. **[キー]** をクリックして **[キー]** パネルを開きます。

   1. **[パスワード]** で、`BotLogin` キーを作成します。
   1. **[期間]** を **[有効期限なし]** に設定します。
   1. **[保存]** をクリックし、キー値をメモします。 この値は、後で Azure AD アプリケーションをご自身のボットに登録するときに、"_クライアント シークレット_" として使用します。
   1. **[キー]** パネルを閉じます。

1. **[必要なアクセス許可]** をクリックして、**[必要なアクセス許可]** パネルを開きます。

   1. **[追加]** をクリックします。
   1. **[API を選択します]** をクリックし、**[Microsoft Graph]** を選択して **[選択]** をクリックします。
   1. **[アクセス許可の選択]** をクリックします。 ご自身のアプリケーションで使用する委任されたアクセス許可を選択します。

      > [!NOTE]
      > **[管理者権限が必要]** と表示されているアクセス許可は、ユーザーとテナント管理者の両方がログインすることを要求するため、ボットではこれらのアクセス許可を避けるのが一般的です。

      次の Microsoft Graph 委任アクセス許可を選択します。
      - すべてのユーザーの基本プロファイルの読み取り
      - ユーザーのメールの読み取り
      - ユーザーとしてのメールの送信
      - サインインとユーザー プロファイルの読み取り
      - ユーザーの基本プロファイルの表示
      - ユーザーの電子メール アドレスの表示

   1. **[選択]** をクリックし、**[完了]** をクリックします。
   1. **[必要なアクセス許可]** パネルを閉じます。

これで、Azure AD v1 アプリケーションが構成されました。

# <a name="azure-ad-v2tabaadv2"></a>[Azure AD v2](#tab/aadv2)

1. [Microsoft アプリケーション登録ポータル](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)に移動します。
1. **[アプリの追加]** をクリックします
1. Azure AD アプリに名前を付けて、**[作成]** をクリックします。

    **[アプリケーション ID]** の GUID をメモします。 後でこれを接続設定のクライアント ID として入力します。

1. **[アプリケーション シークレット]** で **[新しいパスワードを生成]** をクリックします。

    ポップアップに表示されているパスワードをメモします。 後でこれを接続設定のクライアント シークレットとして入力します。

1. **[プラットフォーム]** で、**[プラットフォームの追加]** をクリックします。
1. **[プラットフォームの追加]** ポップアップで、**[Web]** をクリックします。

    1. **[暗黙的フローを許可する]** はチェックされたままにします。
    1. **[リダイレクト URL]** に、`https://token.botframework.com/.auth/web/redirect` と入力します。
    1. **[ログアウト URL]** は空白のままにします。

1. **[Microsoft Graph のアクセス許可]** で、追加の委任されたアクセス許可を追加できます。

    - この記事では、**Mail.Read**、**Mail.Send**、**openid**、**profile**、**User.Read**、および **User.ReadBasic.All** の各アクセス許可を追加します。
      接続設定のスコープは、**openid** と、**Mail.Read** のような Azure AD グラフ内のリソースの両方を持っている必要があります。
    - 選択したアクセス許可をメモします。 後でこれを接続設定のスコープとして入力します。

1. ページの下部にある **[保存]** をクリックします。

これで、Azure AD v2 アプリケーションが構成されました。

---

### <a name="register-your-azure-ad-application-with-your-bot"></a>Azure AD アプリケーションをボットに登録する

次に、作成した Azure AD アプリケーションをご自身のボットに登録します。

# <a name="azure-ad-v1tabaadv1"></a>[Azure AD v1](#tab/aadv1)

1. [Azure Portal](http://portal.azure.com/) で、ボットのリソース ページに移動します。
1. **[設定]** をクリックします。
1. ページ下部付近の **[OAuth Connection Settings]\(OAuth 接続設定\)** で、**[設定の追加]** をクリックします。
1. 次のようにフォームに入力します。

    1. **[名前]** に、接続の名前を入力します。 この名前はご自身のボット コードで使用します。
    1. **[サービス プロバイダー]** で、**[Azure Active Directory]** を選択します。 これを選択すると、Azure AD に固有のフィールドが表示されます。
    1. **[クライアント ID]** に、Azure AD v1 アプリケーションの設定時にメモしたアプリケーション ID を入力します。
    1. **[クライアント シークレット]** に、アプリケーションの `BotLogin` キーの設定時にメモしたキーを入力します。
    1. **[付与タイプ]** に、`authorization_code` と入力します。
    1. **[ログイン URL]** に、`https://login.microsoftonline.com` と入力します。
    1. **[テナント ID]** に、Azure Active Directory のテナント ID を入力します (例: `microsoft.com` または `common`)。

       これは、認証可能なユーザーに関連付けられるテナントになります。 すべてのユーザーがボット経由で自分自身を認証できるようにするには、`common` テナントを使用します。

    1. **[リソース URL]** に、「`https://graph.microsoft.com/`」と入力します。
    1. **[スコープ]** は空白のままにします。

1. **[Save]** をクリックします。

> [!NOTE]
> これらの値によって、アプリケーションは Microsoft Graph API 経由で Office 365 データにアクセスできます。

# <a name="azure-ad-v2tabaadv2"></a>[Azure AD v2](#tab/aadv2)

1. [Azure Portal](http://portal.azure.com/) で、ボットの [Bot Channels Registration]\(ボット チャネル登録\) ページに移動します。
1. **[設定]** をクリックします。
1. ページ下部付近の **[OAuth Connection Settings]\(OAuth 接続設定\)** で、**[設定の追加]** をクリックします。
1. 次のようにフォームに入力します。

    1. **[名前]** に、接続の名前を入力します。 ボットのコードで使用します。
    1. **[サービス プロバイダー]** で、**[Azure Active Directory v2]** を選択します。 これを選択すると、Azure AD に固有のフィールドが表示されます。
    1. **[クライアント ID]** に、アプリケーション登録からの Azure AD v2 アプリケーション ID を入力します。
    1. **[クライアント シークレット]** に、アプリケーション登録からの Azure AD v2 アプリケーション パスワードを入力します。
    1. **[テナント ID]** に、Azure Active Directory のテナント ID を入力します (例: `microsoft.com` または `common`)。

        これは、認証可能なユーザーに関連付けられるテナントになります。 すべてのユーザーがボット経由で自分自身を認証できるようにするには、`common` テナントを使用します。

    1. **[スコープ]** には、アプリケーション登録から選択したアクセス許可の名前を入力します。`Mail.Read Mail.Send openid profile User.Read User.ReadBasic.All`

        > [!NOTE]
        > Azure AD v2 の場合、**[スコープ]** はスペースで区切った値のリストであり、大文字と小文字が区別されます。

1. **[Save]** をクリックします。

> [!NOTE]
> これらの値によって、アプリケーションは Microsoft Graph API 経由で Office 365 データにアクセスできます。

---

### <a name="test-your-connection"></a>接続をテストする

1. 接続エントリをクリックして、作成した接続を開きます。
1. **[Service Provider Connection Setting]\(サービス プロバイダー接続設定\)** ウィンドウの上部にある **[Test Connection]\(接続のテスト\)** をクリックします。
1. 初回は新しいブラウザー タブが開き、アプリが要求しているアクセス許可の一覧が表示され、承認を求められます。
1. **[Accept]\(受け入れる\)** をクリックします。
1. **[\<your-connection-name> への接続テストに成功しました]** ページにリダイレクトされます。

ボット コードでこの接続名を使用してユーザー トークンを取得できるようになりました。

## <a name="prepare-the-bot-code"></a>ボット コードを準備する

このプロセスを完了するには、ボットのアプリ ID とパスワードが必要になります。 ご自身のボットのアプリ ID とパスワードを取得するには、次の操作を行います。

1. [Azure portal][] で、ご自身のチャネル登録ボットを作成したリソース グループに移動します。
1. **[デプロイ]** ウィンドウを開き、ボット用のデプロイを開きます。
1. **[入力]** ウィンドウを開き、ご自身のボットの **appId** と **appSecret** の値をコピーします。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

<!-- TODO: Add guidance (once we have it) on how not to hard-code IDs and ABS auth. -->

1. 使用する github リポジトリから [**ボット認証**][cs-auth-sample]または[**ボット認証 MSGraph**][cs-msgraph-sample] を複製します。
1. その特定のボットを実行する方法について、GitHub の readme ページに記載されている手順に従います。 <!--TODO: Can we remove this step and still have the instructions make sense? What is the minimum we need to say in its place? -->
1. **appsettings.json** を更新します。

    - `ConnectionName` を、お使いのボットに追加した OAuth 接続設定の名前に設定します。
    - `MicrosoftAppId` および `MicrosoftAppPassword` を、お使いのボットのアプリ ID とアプリ シークレットに設定します。

      お使いのボット シークレットに含まれる文字によっては、パスワードを XML でエスケープすることが必要な場合があります。 たとえば、アンパサンド (&) は `&amp;` のようにエンコードする必要があります。

[!code-json[appsettings](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/appsettings.json)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. 使用する github リポジトリから [**ボット認証**][js-auth-sample]または[**ボット認証 MSGraph**][js-msgraph-sample] を複製します。
1. その特定のボットを実行する方法について、GitHub の readme ページに記載されている手順に従います。 <!--TODO: Can we remove this step and still have the instructions make sense? What is the minimum we need to say in its place? -->
1. **.env** を更新します。

    - `connectionName` を、お使いのボットに追加した OAuth 接続設定の名前に設定します。
    - `MicrosoftAppId` および `MicrosoftAppPassword` の値を、お使いのボットのアプリ ID とアプリ シークレットに設定します。

      お使いのボット シークレットに含まれる文字によっては、パスワードを XML でエスケープすることが必要な場合があります。 たとえば、アンパサンド (&) は `&amp;` のようにエンコードする必要があります。

    [!code-txt[.env](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/.env)]

---

**Microsoft アプリ ID** と **Microsoft アプリ パスワード**の値を取得する方法がわからない場合は、次のいずれかを行います。

- [こちらの説明に従って](../bot-service-quickstart-registration.md#bot-channels-registration-password)、新しいパスワードを作成します
- [こちらの説明に従って](https://blog.botframework.com/2018/07/03/find-your-azure-bots-appid-and-appsecret)、デプロイの **Bot Channels Registration** でプロビジョニング済みの **Microsoft アプリ ID** と **Microsoft アプリ パスワード**を取得します

> [!NOTE]
> ここで、このボット コードを Azure サブスクリプションに発行 (プロジェクトを右クリックして **[発行]** を選択) することもできますが、この記事では不要です。 Azure Portal でボットを構成するときに使用したアプリケーションとホスティング プランを使用する発行構成を設定する必要があります。

## <a name="test-the-bot"></a>ボットのテスト

1. [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) をインストールします (まだインストールしていない場合)。
1. ご自身のマシンを使ってローカルでサンプルを実行します。
1. エミュレーターを起動し、お使いのボットに接続して、メッセージを送信します。

    - お使いのボットに接続するときに、ボットのアプリ ID とパスワードを入力する必要があります。
    - `help` と入力すると、ボットで使用できるコマンドの一覧が表示され、認証機能をテストします。
    - サインインした後は、サインアウトするまで、資格情報を再度入力する必要はありません。
    - サインアウトして認証をキャンセルするには、`logout` と入力します。

> [!NOTE]
> ボット認証では Bot Connector Service を使用する必要があります。 このサービスは、お使いのボットのボット チャネル登録情報にアクセスします。

# <a name="bot-authenticationtabbot-oauth"></a>[ボット認証](#tab/bot-oauth)

**ボット認証**サンプルでは、ダイアログは、ユーザーのログイン後、ユーザー トークンを取得するように設計されています。

![サンプル出力](media/how-to-auth/auth-bot-test.png)

# <a name="bot-authentication-msgraphtabbot-msgraph-auth"></a>[ボット認証 MSGraph](#tab/bot-msgraph-auth)

**ボット認証 MSGraph** サンプルでは、ダイアログは、ユーザーのログイン後、制限された一部のコマンドを受け取るように設計されています。

![サンプル出力](media/how-to-auth/msgraph-bot-test.png)

---

## <a name="additional-information"></a>追加情報

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

[!code-csharp[Get the OAuthPrompt result](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=54-58)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

![ボット アーキテクチャ](media/how-to-auth/architecture-js.png)

**dialogs/mainDialog.js**

OAuth プロンプトを、コンストラクター内の **MainDialog** に追加します。 ここでは、接続名の値は **.env** ファイルから取得されました。

[!code-javascript[Add OAuthPrompt](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=23-28)]

ダイアログ ステップ内で、`beginDialog` を使用して OAuth プロンプトを起動します。これによりユーザーのサインインが求められます。

- ユーザーがサインイン済みの場合は、ユーザーに問い合わせることなく、トークン応答イベントが生成されます。
- それ以外の場合は、ユーザーのサインインが求められます。 ユーザーがサインインを試みた後、Azure Bot Service によってトークン応答イベントが送信されます。

[!code-javascript[Use OAuthPrompt](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=57)]

次のダイアログ ステップ内で、前のステップの結果としてトークンが存在することを確認します。 null でない場合、ユーザーは正常にサインインしています。

[!code-javascript[Get OAuthPrompt result](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=61-64)]

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

[!code-javascript[onTokenResponseEvent](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/bots/authBot.js?range=28-33)]

---

### <a name="log-the-user-out"></a>ユーザーをログアウトする

接続のタイムアウトを使用するのではなく、ユーザーが明示的にサインアウトまたはログアウトできるようにすることをお勧めします。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\LogoutDialog.cs**

[!code-csharp[Allow logout](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/LogoutDialog.cs?range=20-61&highlight=35)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/logoutDialog.js**

[!code-javascript[Allow logout](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/logoutDialog.js?range=13-42&highlight=25)]

---

### <a name="further-reading"></a>参考資料

- 「[Bot Framework のその他のリソース](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help)」に追加サポートのリンクがあります。
- [Bot Framework SDK](https://github.com/microsoft/botbuilder) のリポジトリでは、Bot Builder SDK に関連付けられているリポジトリ、サンプル、ツール、および仕様の詳細を確認できます。

<!-- Footnote-style links -->

[Azure Portal]: https://ms.portal.azure.com
[azure-aad-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview
[aad-registration-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-dialog]: bot-builder-dialog-manage-conversation-flow.md
[dialog-prompts]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-auth-sample]: https://aka.ms/v4cs-bot-auth-sample
[js-auth-sample]: https://aka.ms/v4js-bot-auth-sample
[cs-msgraph-sample]: https://aka.ms/v4cs-auth-msgraph-sample
[js-msgraph-sample]: https://aka.ms/v4js-auth-msgraph-sample
