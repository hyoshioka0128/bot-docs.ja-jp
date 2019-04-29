---
title: Azure Bot Service を介してボットに認証を追加する | Microsoft Docs
description: Azure Bot Service の認証機能を使用して SSO をボットに追加する方法について説明します。
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 04/09/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2f15817abe087650bc3f2bb998a32f177848cf50
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904535"
---
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

Direct Line の `/v3/directline/tokens/generate` REST エンドポイントを使用してメッセージ交換用のトークンを生成し、要求ペイロードでユーザー ID を指定します。 コード サンプルについては、ブログ記事「[Enhanced Direct Line Authentication Features (強化された Direct Line 認証機能)](https://blog.botframework.com/2018/09/25/enhanced-direct-line-authentication-features/)」を参照してください。

<!-- The eventual article about this should talk about the tokens/generate endpoint and its parameters: user, trustedOrigins, and [maybe] eTag.
Sample payload
{
  "user": {
    "id": "string",
    "name": "string",
    "aadObjectId": "string",
    "role": "string"
  },
  "trustedOrigins": [
    "string"
  ],
  "eTag": "string"
}
 -->

## <a name="prerequisites"></a>前提条件

- [ボットの基礎][concept-basics]と[状態の管理][concept-state]に関する知識。
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

## <a name="prepare-the-bot-sample-code"></a>ボットのサンプル コードの準備

お客様が選択したサンプルに応じて、C# と Node のいずれかを使用して作業することになります。

| サンプル | BotBuilder のバージョン | 対象 |
|:---|:---:|:---|
| [**CSharp**][cs-auth-sample] または [**JavaScript**][js-auth-sample] の**ボット認証** | v4 | OAuthCard サポート |
| [**CSharp**][cs-msgraph-sample] または [**JavaScript**][js-msgraph-sample] の**ボット認証 MSGraph** | v4 |  OAuth 2 を使用した Microsoft Graph API サポート |

1. 上にあるいずれかのサンプルのリンクをクリックして、GitHub リポジトリを複製します。
1. その特定のボット (C# または Node) を実行する方法について、GitHub の readme ページにある手順に従います。
1. C# ボット認証サンプルを使用している場合:

    1. `AuthenticationBot.cs` ファイルの `ConnectionName` の値は、ボットの OAuth 2.0 接続設定を構成したときに使用した値に設定します。
    1. `BotConfiguration.bot` ファイルの `appId` 値を、お客様のボットのアプリ ID に設定します。
    1. `BotConfiguration.bot` ファイルの `appPassword` 値を、お客様のボットのシークレットに設定します。

1. Node (JS) ボット認証サンプルを使用している場合:

    1. `bot.js` ファイルの `CONNECTION_NAME` の値は、ボットの OAuth 2.0 接続設定を構成したときに使用した値に設定します。
    1. `bot-authentication.bot` ファイルの `appId` 値を、お客様のボットのアプリ ID に設定します。
    1. `bot-authentication.bot` ファイルの `appPassword` 値を、お客様のボットのシークレットに設定します。

    > [!IMPORTANT]
    > シークレットに含まれる文字によっては、パスワードを XML でエスケープすることが必要な場合があります。 たとえば、アンパサンド (&) は `&amp;` のようにエンコードする必要があります。

    ```json
    {
        "name": "BotAuthentication",
        "secretKey": "",
        "services": [
            {
            "appId": "",
            "id": "http://localhost:3978/api/messages",
            "type": "endpoint",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages",
            "name": "BotAuthentication"
            }
        ]
    }
    ```

**Microsoft アプリ ID** と **Microsoft アプリ パスワード**の値を取得する方法がわからない場合は、次の記事の説明に従って新しいパスワードを作成できます。

[bot-channels-registration-password](../bot-service-quickstart-registration.md#bot-channels-registration-password)
  
または、[find-your-azure-bots-appid-and-appsecret](https://blog.botframework.com/2018/07/03/find-your-azure-bots-appid-and-appsecret) の説明に従って、デプロイの **Bot Channels Registration** でプロビジョニング済みの **Microsoft アプリ ID** と **Microsoft アプリ パスワード**を取得することもできます。

    > [!NOTE]
    > You could now publish this bot code to your Azure subscription (right-click on the project and choose **Publish**), but it is not necessary for this tutorial. You would need to set up a publishing configuration that uses the application and hosting plan that you used when configuration the bot in the Azure Portal.

## <a name="use-the-emulator-to-test-your-bot"></a>エミュレーターを使用してボットをテストする

ボットをローカルでテストするには、[Bot Emulator](https://github.com/Microsoft/BotFramework-Emulator) をインストールする必要があります。 v3 または v4 エミュレーターを使用できます。

1. ボットを開始します (デバッグは任意)。
1. ページの localhost ポート番号をメモします。 この情報はボットと対話するために必要になります。
1. エミュレーターを起動します。
1. ボットに接続します。 認証を使用するときは、ボットの構成で **Microsoft アプリ ID** と **Microsoft アプリ パスワード**が使用されていることを確認します
1. Azure Bot Service が使用できるようになったときにエミュレーターにトークンを返すことができるように、エミュレーターの設定で、**[Use a sign-in verification code for OAuthCards]\(OAuthCards に対してサインイン検証コードを使用する\)** がオンになっていて、**ngrok** が有効になっていることを確認します。

   まだ接続を構成していない場合、アドレスと、ボットの Microsoft アプリ ID およびパスワードを入力します。 ボットの URL に `/api/messages` を追加します。 URL は `http://localhost:portNumber/api/messages` のようになります。

1. `help` と入力すると、ボットで使用できるコマンドの一覧が表示され、認証機能をテストします。
1. サインインした後は、サインアウトするまで、資格情報を再度入力する必要はありません。
1. サインアウトして認証をキャンセルするには、`signout` と入力します。

<!--To restart completely from scratch you also need to:
1. Navigate to the **AppData** folder for your account.
1. Go to the **Roaming/botframework-emulator** subfolder.
1. Delete the **Cookies** and **Coolies-journal** files.
-->

> [!NOTE]
> ボット認証では Bot Connector Service を使用する必要があります。 このサービスはボットのボット チャネル登録情報にアクセスします。ボットのメッセージング エンドポイントをポータルで設定する必要があるのはそのためです。 認証では HTTPS も使用する必要があります。ローカルで実行するボットに対して HTTPS 転送先アドレスを作成する必要があったのはそのためです。

<!--The following is necessary for WebChat:
1. Use the **ngrok** command-line tool to get a forwarding HTTPS address for your bot.
   - For information on how to do this, see [Debug any Channel locally using ngrok](https://blog.botframework.com/2017/10/19/debug-channel-locally-using-ngrok/).
   - Any time you exit **ngrok**, you will need to redo this and the following step before starting the Emulator.
1. On the Azure Portal, go to the **Settings** blade for your bot.
   1. In the **Configuration** section, change the **Messaging endpoint** to the HTTPS forwarding address generated by **ngrok**.
   1. Click **Save** to save your change.
-->

## <a name="notes-on-the-token-retrieval-flow"></a>トークン取得フローに関する注意事項

ユーザーがボットに何らかの処理を要求し、それによってボットでユーザーのログインが必要になった場合、ボットによって `OAuthPrompt` が使用され、特定の接続に必要なトークンの取得が開始されます。 `OAuthPrompt` では、以下で構成されるトークン取得フローが作成されます。

1. 既に Azure Bot Service に現在のユーザーおよび接続用のトークンがあるかどうかを確認するためのチェック。 トークンがある場合は、そのトークンが返されます。
1. キャッシュされたトークンが Azure Bot Service にない場合は、`OAuthCard` が作成されます。これは、ユーザーがクリックできるサインイン ボタンです。
1. ユーザーが `OAuthCard` サインイン ボタンをクリックしたら、Azure Bot Service によって、ユーザーのトークンがボットに直接送信されます。または、チャット ウィンドウで 6 桁の認証コードがユーザーに表示されます。
1. ユーザーに認証コードが表示される場合は、ボットによって、この認証コードがユーザーのトークンと交換されます。

次のいくつかのコード スニペットは `OAuthPrompt` から取られたもので、プロンプト内でこれらの手順がどのように動作するかを示しています。

### <a name="check-for-a-cached-token"></a>キャッシュされたトークンの確認

このコードでは、ボットはまず、(現在の Activity 送信者によって識別される) ユーザーのトークンと、特定の ConnectionName (構成で使用される接続名) が Azure Bot Service に既に存在しているかどうかのクイック チェックを行います。 Azure Bot Service には、キャッシュされたトークンが既に存在しているか、存在していないかのどちらかです。 GetUserTokenAsync の呼び出しは、このクイック チェックを実行します。 Azure Bot Service にトークンが存在していてそれが返される場合、そのトークンはすぐに使用できます。 Azure Bot Service にトークンが存在しない場合、このメソッドは null を返します。 ボットはこの場合、ユーザーがログインするためのカスタマイズされた OAuthCard を送信できます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// First ask Bot Service if it already has a token for this user
var token = await adapter.GetUserTokenAsync(turnContext, connectionName, null, cancellationToken).ConfigureAwait(false);
if (token != null)
{
    // use the token to do exciting things!
}
else
{
    // If Bot Service does not have a token, send an OAuth card to sign in
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
public async getUserToken(context: TurnContext, code?: string): Promise<TokenResponse|undefined> {
    // Get the token and call validator
    const adapter: any = context.adapter as any; // cast to BotFrameworkAdapter
    return await adapter.getUserToken(context, this.settings.connectionName, code);
}
```

---

### <a name="send-an-oauthcard-to-the-user"></a>ユーザーへの OAuthCard の送信

任意のテキストやボタン テキストを使用して OAuthCard をカスタマイズできます。 重要な部分は次のとおりです。

- `ContentType` を `OAuthCard.ContentType` に設定します。
- `ConnectionName` プロパティを、使用する接続の名前に設定します。
- `CardAction` が `Type` `ActionTypes.Signin` である 1 つのボタンを含めます。サインイン リンクの値は何も指定する必要はありません。

この呼び出しの最後に、ボットは "トークンの戻りを待つ" 必要があります。 サインインのためにユーザー側で必要な処理が多い可能性があるため、この待機はメインの Activity ストリームで行われます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private async Task SendOAuthCardAsync(ITurnContext turnContext, IMessageActivity message, CancellationToken cancellationToken = default(CancellationToken))
{
    if (message.Attachments == null)
    {
        message.Attachments = new List<Attachment>();
    }

    message.Attachments.Add(new Attachment
    {
        ContentType = OAuthCard.ContentType,
        Content = new OAuthCard
        {
            Text = "Please sign in",
            ConnectionName = connectionName,
            Buttons = new[]
            {
                new CardAction
                {
                    Title = "Sign In",
                    Text = "Sign In",
                    Type = ActionTypes.Signin,
                },
            },
        },
    });

    await turnContext.SendActivityAsync(message, cancellationToken).ConfigureAwait(false);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
private async sendOAuthCardAsync(context: TurnContext, prompt?: string|Partial<Activity>): Promise<void> {
    // Initialize outgoing message
    const msg: Partial<Activity> =
        typeof prompt === 'object' ? {...prompt} : MessageFactory.text(prompt, undefined, InputHints.ExpectingInput);
    if (!Array.isArray(msg.attachments)) { msg.attachments = []; }

    const cards: Attachment[] = msg.attachments.filter((a: Attachment) => a.contentType === CardFactory.contentTypes.oauthCard);
    if (cards.length === 0) {
        // Append oauth card
        msg.attachments.push(CardFactory.oauthCard(
            this.settings.connectionName,
            this.settings.title,
            this.settings.text
        ));
    }

    // Send prompt
    await context.sendActivity(msg);
}
```

---

### <a name="wait-for-a-tokenresponseevent"></a>TokenResponseEvent の待機

このコードでは、ボットが `TokenResponseEvent` を待機しています (これがダイアログ スタックにルーティングされるしくみの詳細は後述)。 `WaitForToken` メソッドはまず、このイベントが送信されたかどうかを判断します。 送信された場合、そのイベントはボットによって使用できます。 送信されていない場合、`RecognizeTokenAsync` メソッドはボットに送信された任意のテキストを引数に取って `GetUserTokenAsync` に渡します。 この理由は、(WebChat のような) 一部のクライアントはマジック コード検証コードを必要とせず、`TokenResponseEvent` でトークンを直接送信できるからです。 それ以外のクライアントでは、引き続きマジック コードが必要です (Facebook や Slack など)。 Azure Bot Service はこれらのクライアントに 6 桁のマジック コードを提示し、ユーザーにはこのコードをチャット ウィンドウに入力するよう求めます。 理想的ではありませんが、これは "フォールバック" 動作であるため、`RecognizeTokenAsync` がコードを受け取った場合、ボットはこのコードを Azure Bot Service に送信してトークンを再取得できます。 この呼び出しも失敗した場合、エラーを報告するか、または何か別の処理を実行することができます。 ただし、ほとんどの場合、この時点でボットはユーザー トークンを取得しています。

各サンプルのボット コードを見ると、`Event` アクティビティと `Invoke` アクティビティもダイアログ スタックにルーティングされることがわかります。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// This can be called when the bot receives an Activity after sending an OAuthCard
private async Task<TokenResponse> RecognizeTokenAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (IsTokenResponseEvent(turnContext))
    {
        // The bot received the token directly
        var tokenResponseObject = turnContext.Activity.Value as JObject;
        var token = tokenResponseObject?.ToObject<TokenResponse>();
        return token;
    }
    else if (IsTeamsVerificationInvoke(turnContext))
    {
        var magicCodeObject = turnContext.Activity.Value as JObject;
        var magicCode = magicCodeObject.GetValue("state")?.ToString();

        var token = await adapter.GetUserTokenAsync(turnContext, _settings.ConnectionName, magicCode, cancellationToken).ConfigureAwait(false);
        return token;
    }
    else if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // make sure it's a 6-digit code
        var matched = _magicCodeRegex.Match(turnContext.Activity.Text);
        if (matched.Success)
        {
            var token = await adapter.GetUserTokenAsync(turnContext, _settings.ConnectionName, matched.Value, cancellationToken).ConfigureAwait(false);
            return token;
        }
    }

    return null;
}

private bool IsTokenResponseEvent(ITurnContext turnContext)
{
    var activity = turnContext.Activity;
    return activity.Type == ActivityTypes.Event && activity.Name == "tokens/response";
}

private bool IsTeamsVerificationInvoke(ITurnContext turnContext)
{
    var activity = turnContext.Activity;
    return activity.Type == ActivityTypes.Invoke && activity.Name == "signin/verifyState";
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
private async recognizeToken(context: TurnContext): Promise<PromptRecognizerResult<TokenResponse>> {
    let token: TokenResponse|undefined;
    if (this.isTokenResponseEvent(context)) {
        token = context.activity.value as TokenResponse;
    } else if (this.isTeamsVerificationInvoke(context)) {
        const code: any = context.activity.value.state;
        await context.sendActivity({ type: 'invokeResponse', value: { status: 200 }});
        token = await this.getUserToken(context, code);
    } else if (context.activity.type === ActivityTypes.Message) {
        const matched: RegExpExecArray = /(\d{6})/.exec(context.activity.text);
        if (matched && matched.length > 1) {
            token = await this.getUserToken(context, matched[1]);
        }
    }

    return token !== undefined ? { succeeded: true, value: token } : { succeeded: false };
}

private isTokenResponseEvent(context: TurnContext): boolean {
    const activity: Activity = context.activity;
    return activity.type === ActivityTypes.Event && activity.name === 'tokens/response';
}

private isTeamsVerificationInvoke(context: TurnContext): boolean {
    const activity: Activity = context.activity;
    return activity.type === ActivityTypes.Invoke && activity.name === 'signin/verifyState';
}
```

---

### <a name="message-controller"></a>メッセージ コントローラー

その後のボットの呼び出しでは、このサンプル ボットによってトークンが決してキャッシュされないことに注意してください。 これは、ボットがいつでも Azure Bot Service にトークンを要求できるからです。 トークンのライフサイクルの管理やトークンの更新などはすべて Azure Bot Service によって自動的に行われるため、ボットでこれらを行う必要はありません。

### <a name="further-reading"></a>参考資料

- 「[Bot Framework のその他のリソース](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help)」に追加サポートのリンクがあります。
- [Bot Framework SDK](https://github.com/microsoft/botbuilder) のリポジトリでは、Bot Builder SDK に関連付けられているリポジトリ、サンプル、ツール、および仕様の詳細を確認できます。

<!-- Footnote-style links -->

[Azure portal]: https://ms.portal.azure.com
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
