---
title: Azure Bot Service を介してボットに認証を追加する | Microsoft Docs
description: Azure Bot Service の認証機能を使用して SSO をボットに追加する方法について説明します。
author: JonathanFingold
ms.author: JonathanFingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 7/2/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 7fb430857cf954d2b9126a400335f28b091fa11d
ms.sourcegitcommit: f95702d27abbd242c902eeb218d55a72df56ce56
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/19/2018
ms.locfileid: "39304649"
---
# <a name="add-authentication-to-your-bot-via-azure-bot-service"></a>Azure Bot Service を介してボットに認証を追加する

このチュートリアルでは、Azure Bot Service の新しいボット認証機能を使用します。提供される機能を利用することで、Azure AD (Azure Active Directory)、GitHub、Uber などの各種 ID プロバイダーにユーザーを認証するボットを容易に開発できます。 またこれらの更新は、一部クライアント用の_マジック コード検証_を排除することによって、ユーザー エクスペリエンスの向上に向けて前進します。

これ以前は、ボットにおいて、OAuth コントローラーとログイン リンクを含め、ターゲット クライアントの ID とシークレットを保存し、ユーザー トークン管理を実行する必要がありました。
<!--
These capabilities were bundled in the BotAuth and AuthBot samples that are on GitHub.
-->

OAuth コントローラーのホスティングやトークンのライフサイクルの管理は、すべて Azure Bot Service によって実行できるようになったため、ボット開発者がこれらを行う必要はなくなりました。

機能は、次のとおりです。

- チャネルの機能向上によって新しい WebChat や DirectLineJS ライブラリなどの新しい認証機能がサポートされ、6 桁のマジック コード検証の必要性を排除。
- Azure Portal で各種 OAuth アイデンティティ プロバイダーへの接続設定を追加、削除、および構成する機能の向上。
- Azure AD (v1 と v2 の両エンドポイント) や GitHub など、すぐに利用できる各種アイデンティティ プロバイダーのサポート。
- C# および Node.js の Bot Builder SDK の更新により、トークンの取得、OAuthCard の作成、TokenResponse イベントの処理が可能に。
- Azure AD (v1 と v2 のエンドポイント) と GitHub に認証するボットの作成方法のサンプル。

この記事の手順を応用して、このような機能を既存のボットに追加することができます。 以下は、新しい認証機能の例を示すサンプル ボットです

| サンプル | BotBuilder のバージョン | 説明 |
|:---|:---:|:---|
| [AadV1Bot](https://github.com/Microsoft/BotBuilder/tree/master/CSharp/Samples/AadV1Bot) | v3 | Azure AD v1 エンドポイントを使用して、v3 C# SDK での OAuthCard サポートの例を示します |
| [AadV2Bot](https://github.com/Microsoft/BotBuilder/tree/master/CSharp/Samples/AadV2Bot) | v3 |  Azure AD v2 エンドポイントを使用して、v3 C# SDK での OAuthCard サポートの例を示します |
| [GitHubBot](https://github.com/Microsoft/BotBuilder/tree/master/CSharp/Samples/GitHubBot) | v3 |  GitHub を使用して、v3 C# SDK での OAuthCard サポートの例を示します |
| [BasicOAuth](https://github.com/Microsoft/BotBuilder/tree/master/CSharp/Samples/Microsoft.Bot.Sample.BasicOAuth) | v3 |  v3 C# SDK での OAuth 2.0 サポートの例を示します |

> [!NOTE]
> 認証機能は Node.js と BotBuilder v3 の組み合わせでも動作します。 ただし、この記事ではサンプルの C#コードのみを扱います。

追加情報とサポートについては、「[Bot Framework のその他のリソース](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-resources-links-help)」を参照してください。

## <a name="overview"></a>概要

このチュートリアルでは、Azure AD v1 または v2 トークンを使用して Microsoft Graph に接続するサンプル ボットを作成します。 <!--verify this info and fix wording--> このプロセスの一環として、GitHub リポジトリのコードを使用します。このチュートリアルでは、ボット アプリケーションを含めたセットアップ方法について説明します。

- [ボットと認証アプリケーションの作成](#create-your-bot-and-an-authentication-application)
- [ボットのサンプル コードの準備](#prepare-the-bot-sample-code)
- [エミュレーターを使用してボットをテストする](#use-the-emulator-to-test-your-bot)

これらの手順を完了するには、Visual Studio 2017、npm、node、および git がインストールされている必要があります。 また、Azure、OAuth 2.0、およびボット開発についてある程度理解している必要があります。

手順が終了すると、電子メールのチェックと送信、自分とその上司の情報の表示など、Azure AD アプリケーションに対するいくつかの単純なタスクに応答できるボットが完成します。 これを行うために、ボットでは Azure AD アプリケーションからのトークンを Microsoft.Graph ライブラリに対して使用します。

最後のセクションでは、ボット コードの一部を解説します

- [トークン取得フローに関する注意事項](#notes-on-the-token-retrieval-flow)

## <a name="create-your-bot-and-an-authentication-application"></a>ボットと認証アプリケーションの作成

ボット コードの発行先になる登録ボットを作成する必要があります。また、Azure AD (v1 または v2) アプリケーションを作成し、ボットが Office 365 にアクセスできるようにする必要があります。

> [!NOTE]
> これらの認証機能は他の種類のボットと連動します。 ただし、このチュートリアルでは登録のみのボットを使用します。

### <a name="register-an-application-in-azure-ad"></a>アプリケーションを Azure AD に登録する

Microsoft Graph API や、Azure AD で保護された独自リソースなどに接続するためにボットが使用できる Azure AD アプリケーションが必要です。

このボットには Azure AD v1 または v2 エンドポイントを使用できます。
v1 と v2 の各エンドポイントの違いについては、[v1 と v2 の比較](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-compare)に関する記事と、[Azure AD v2.0 エンドポイントの概要](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview)に関する記事を参照してください。

#### <a name="to-create-an-azure-ad-v1-application"></a>Azure AD v1 アプリケーションを作成するには

1. [Azure portal で Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview) に移動します。
1. **[アプリの登録]** をクリックします。
1. **[アプリの登録]** パネルで、**[新しいアプリケーションの登録]** をクリックします。
1. 必須のフィールドに入力してアプリ登録を作成します。
   1. アプリケーションに名前を付けます。
   1. **[アプリケーションの種類]** を **[Web アプリ/API]** に設定します。
   1. **[サインオン URL]** を `https://token.botframework.com/.auth/web/redirect` に設定します。
   1. **Create** をクリックしてください。
      - 作成されると、**[Registered app]\(登録済みアプリ\)** ブレードに表示されます。
      - **[アプリケーション ID]** の値をメモします。 後でこれを _[クライアント ID]_ として入力します。
1. **[設定]** をクリックしてアプリケーションを構成します。
1. **[キー]** をクリックして **[キー]** パネルを開きます。
   1. **[パスワード]** で、`BotLogin` キーを作成します。
   1. **[期間]** を **[有効期限なし]** に設定します。
   1. **[保存]** をクリックし、キー値をメモします。 後で_アプリケーション シークレット_にこれを指定します。
   1. **[キー]** パネルを閉じます。
1. **[必要なアクセス許可]** をクリックして、**[必要なアクセス許可]** パネルを開きます。
   1. **[追加]** をクリックします。
   1. **[API を選択します]** をクリックし、**[Microsoft Graph]** を選択して **[選択]** をクリックします。
   1. **[アクセス許可の選択]** をクリックします。 アプリケーションで使用するアプリケーションのアクセス許可を選択します。

      > [!NOTE]
      > **[管理者権限が必要]** と表示されているアクセス許可は、ユーザーとテナント管理者の両方がログインすることを要求するため、ボットではこれらのアクセス許可を避けるのが一般的です。

      次の Microsoft Graph 委任アクセス許可を選択します。
      - すべてのユーザーの基本プロファイルの読み取り
      - ユーザーのメールの読み取り
      - サインインとユーザー プロファイルの読み取り
      - ユーザーとしてのメールの送信
      - ユーザーの基本プロファイルの表示
      - ユーザーの電子メール アドレスの表示

   1. **[選択]** をクリックし、**[完了]** をクリックします。
   1. **[必要なアクセス許可]** パネルを閉じます。

これで、Azure AD v1 アプリケーションが構成されました。

#### <a name="to-create-an-azure-ad-v2-application"></a>Azure AD v2 アプリケーションを作成するには

1. [Microsoft アプリケーション登録ポータル](https://apps.dev.microsoft.com)に移動します。
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
    - このチュートリアルでは、**Mail.Read**、**Mail.Send**、**openid**、**profile**、**User.Read**、および **User.ReadBasic.All** の各アクセス許可を追加します。
      接続設定のスコープは、**openid** と、**Mail.Read** のような Azure AD グラフ内のリソースの両方を持っている必要があります。
    - 選択したアクセス許可をメモします。 後でこれを接続設定のスコープとして入力します。

1. ページの下部にある **[保存]** をクリックします。

### <a name="create-your-bot-on-azure"></a>Azure でのボットの作成

[Azure Portal](https://portal.azure.com/) を使用して**ボット チャネル登録**を作成します。

### <a name="register-your-azure-ad-application-with-your-bot"></a>Azure AD アプリケーションをボットに登録する

次に、作成した Azure AD アプリケーションをボットに登録します。

#### <a name="to-register-an-azure-ad-v1-application"></a>Azure AD v1 アプリケーションを登録するには

1. [Azure Portal](http://portal.azure.com/) で、ボットのリソース ページに移動します。
1. **[設定]** をクリックします。
1. ページ下部付近の **[OAuth Connection Settings]\(OAuth 接続設定\)** で、**[設定の追加]** をクリックします。
1. 次のようにフォームに入力します。
    1. **[名前]** に、接続の名前を入力します。 ボットのコードで使用します。
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

ボット コードでこの接続名を使用してユーザー トークンを取得できるようになりました。

#### <a name="to-register-an-azure-ad-v2-application"></a>Azure AD v2 アプリケーションを登録するには

1. [Azure Portal](http://portal.azure.com/) で、ボットの [Bot Channels Registration]\(ボット チャネル登録\) ブレードに移動します。
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

ボット コードでこの接続名を使用してユーザー トークンを取得できるようになりました。

#### <a name="to-test-your-connection"></a>接続をテストするには

1. 作成した接続を開きます。
1. **[Service Provider Connection Setting]\(サービス プロバイダー接続設定\)** ブレードの上部にある **[Test Connection]\(接続のテスト\)** をクリックします。
1. 初回は新しいブラウザー タブが開き、アプリが要求しているアクセス許可の一覧が表示され、承認を求められます。
1. **[Accept]\(受け入れる\)** をクリックします。
1. **[Test Connection to `<your-connection-name>' Succeeded]\('<接続名>' へのテスト接続は成功しました\)** ページにリダイレクトされます。

## <a name="prepare-the-bot-sample-code"></a>ボットのサンプル コードの準備

1. https://github.com/Microsoft/BotBuilder にある github リポジトリを複製します。
1. ソリューション `BotBuilder\CSharp\Microsoft.Bot.Builder.sln`を開き、ビルドします。
1. そのソリューションを閉じて、`BotBuilder\CSharp\Samples\Microsoft.Bot.Builder.Samples.sln` を開きます。
1. スタートアップ プロジェクトを設定します。
    - v1 Azure AD アプリケーションを使用するボットには、`Microsoft.Bot.Sample.AadV1Bot` プロジェクトを使用します。
    - v2 Azure AD アプリケーションを使用するボットには、`Microsoft.Bot.Sample.AadV2Bot` プロジェクトを使用します。
1. `Web.config` ファイルを開き、次のようにアプリの設定を変更します。
    1. `ConnectionName` は、ボットの OAuth 2.0 接続設定を構成するときに使用した値に設定します。
    1. `MicrosoftAppId` 値をボットのアプリ ID に設定します。
    1. `MicosoftAppPassword` 値をボットのシークレットに設定します。

    > [!IMPORTANT]
    > シークレットに含まれる文字によっては、パスワードを XML でエスケープすることが必要な場合があります。 たとえば、アンパサンド (&) は `&amp;` のようにエンコードする必要があります。

    ```xml
    <appSettings>
        <add key="ConnectionName" value="<your-AAD-connection-name>"/>
        <add key="MicrosoftAppId" value="<your-bot-appId>" />
        <add key="MicrosoftAppPassword" value="<your-bot-password>" />
    </appSettings>
    ```

    **Microsoft アプリ ID** と **Microsoft アプリ パスワード**の値が分からない場合、Azure Portal でボットに対してプロビジョニングされた Azure アプリ サービスの **ApplicationSettings** を調べます。

    > [!NOTE]
    > ここで、このボット コードを Azure サブスクリプションに発行 (プロジェクトを右クリックして **[発行]** を選択) することもできますが、このチュートリアルでは不要です。 Azure Portal でボットを構成するときに使用したアプリケーションとホスティング プランを使用する発行構成を設定する必要があります。

## <a name="use-the-emulator-to-test-your-bot"></a>エミュレーターを使用してボットをテストする

ボットをローカルでテストするには、[Bot Emulator](https://github.com/Microsoft/BotFramework-Emulator) をインストールする必要があります。 v3 または v4 エミュレーターを使用できます。

1. ボットを開始します (デバッグは任意)。
1. ページの localhost ポート番号をメモします。 この情報はボットと対話するために必要になります。
1. エミュレーターを起動します。
1. ボットに接続します。

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

ユーザーがボットに何らかの処理を要求し、ユーザーがログインしていることをその処理がボットに要求する場合、ボットは `Microsoft.Bot.Builder.Dialogs.GetTokenDialog` を使用して、特定の接続用のトークンの取得を開始できます。 以下のいくつかのスニペットは `GetTokenDialog` クラスからの抜粋です。

### <a name="check-for-a-cached-token"></a>キャッシュされたトークンの確認

このコードでは、ボットはまず、(現在の Activity 送信者によって識別される) ユーザーのトークンと、特定の ConnectionName (構成で使用される接続名) が Azure Bot Service に既に存在しているかどうかのクイック チェックを行います。 Azure Bot Service には、キャッシュされたトークンが既に存在しているか、存在していないかのどちらかです。 GetUserTokenAsync の呼び出しは、この "クイック チェック" を実行します。 Azure Bot Service にトークンが存在していてそれが返される場合、そのトークンはすぐに使用できます。 Azure Bot Service にトークンが存在しない場合、このメソッドは null を返します。 ボットはこの場合、ユーザーがログインするためのカスタマイズされた OAuthCard を送信できます。

```csharp
// First ask Bot Service if it already has a token for this user
var token = await context.GetUserTokenAsync(ConnectionName).ConfigureAwait(false);
if (token != null)
{
    // use the token to do exciting things!
}
else
{
    // If Bot Service does not have a token, send an OAuth card to sign in
    await SendOAuthCardAsync(context, (Activity)context.Activity);
}
```

### <a name="send-an-oauthcard-to-the-user"></a>ユーザーへの OAuthCard の送信

任意のテキストやボタン テキストを使用して OAuthCard をカスタマイズできます。 重要な部分は次のとおりです。

- `ContentType` を `OAuthCard.ContentType` に設定します。
- `ConnectionName` プロパティを、使用する接続の名前に設定します。
- `CardAction` が `Type` `ActionTypes.Signin` である 1 つのボタンを含めます。サインイン リンクの値は何も指定する必要はありません。

この呼び出しの最後に、ボットは "トークンの戻りを待つ" 必要があります。 サインインのためにユーザー側で必要な処理が多い可能性があるため、この待機はメインの Activity ストリームで行われます。

```csharp
private async Task SendOAuthCardAsync(IDialogContext context, Activity activity)
{
    await context.PostAsync($"To do this, you'll first need to sign in.");

    var reply = await context.Activity.CreateOAuthReplyAsync(_connectionName, _signInMessage, _buttonLabel).ConfigureAwait(false);
    await context.PostAsync(reply);

    context.Wait(WaitForToken);
}
```

### <a name="wait-for-a-tokenresponseevent"></a>TokenResponseEvent の待機

このコードでは、ボットのダイアログ クラスが `TokenResponseEvent` を待機しています (これが Dialog スタックにルーティングされるしくみの詳細は後述)。 `WaitForToken` メソッドはまず、このイベントが送信されたかどうかを判断します。 送信された場合、そのイベントはボットによって使用できます。 送信されていない場合、`WaitForToken` メソッドはボットに送信された任意のテキストを引数に取って `GetUserTokenAsync` に渡します。 この理由は、(WebChat のような) 一部のクライアントはマジック コード検証コードを必要とせず、`TokenResponseEvent` でトークンを直接送信できるからです。 それ以外のクライアントでは、引き続きマジック コードが必要です (Facebook や Slack など)。 Azure Bot Service はこれらのクライアントに 6 桁のマジック コードを提示し、ユーザーにはこのコードをチャット ウィンドウに入力するよう求めます。 理想的ではありませんが、これは "フォールバック" 動作であるため、`WaitForToke`n がコードを受け取った場合、ボットはこのコードを Azure Bot Service に送信してトークンを再取得できます。 この呼び出しも失敗した場合、エラーを報告するか、または何か別の処理を実行することができます。 ただし、ほとんどの場合、この時点でボットはユーザー トークンを取得しています。

**MessageController.cs** ファイルを見ると、このタイプの `Event` アクティビティもダイアログ スタックにルーティングされることが分かります。

```csharp
private async Task WaitForToken(IDialogContext context, IAwaitable<object> result)
{
    var activity = await result as Activity;

    var tokenResponse = activity.ReadTokenResponseContent();
    if (tokenResponse != null)
    {
        // Use the token to do exciting things!
    }
    else
    {
        if (!string.IsNullOrEmpty(activity.Text))
        {
            tokenResponse = await context.GetUserTokenAsync(ConnectionName,
                                                               activity.Text);
            if (tokenResponse != null)
            {
                // Use the token to do exciting things!
                return;
            }
        }
        await context.PostAsync($"Hmm. Something went wrong. Let's try again.");
        await SendOAuthCardAsync(context, activity);
    }
}
```

### <a name="message-controller"></a>メッセージ コントローラー

その後のボットの呼び出しでは、このサンプル ボットによってトークンが決してキャッシュされないことに注意してください。 これは、ボットがいつでも Azure Bot Service にトークンを要求できるからです。 トークンのライフサイクルの管理やトークンの更新などはすべて Azure Bot Service によって自動的に行われるため、ボットでこれらを行う必要はありません。

```csharp
else if(message.Type == ActivityTypes.Event)
{
    if(message.IsTokenResponseEvent())
    {
        await Conversation.SendAsync(message, () => new Dialogs.RootDialog());
    }
}
```
