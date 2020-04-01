---
title: Facebook Messenger にボットを接続する - Bot Service
description: Facebook Messenger へのボットの接続を構成する方法について説明します。
keywords: Facebook Messenger, ボット チャンネル, Facebook アプリ, アプリ ID, アプリ シークレット, Facebook ボット, 資格情報
manager: kamrani
ms.topic: article
ms.author: kamrani
ms.service: bot-service
ms.date: 01/16/2020
ms.openlocfilehash: 84544f5c4082f88991718c5d84dc2aead05f5eb2
ms.sourcegitcommit: 64b25f796f89e8bb6fa53d3c824b73b8ce4d6ed8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/25/2020
ms.locfileid: "80250141"
---
# <a name="connect-a-bot-to-facebook"></a>Facebook にボットを接続する

ボットは Facebook Messenger と Facebook Workplace の両方に接続できるため、両方のプラットフォームのユーザーと通信できます。 次のチュートリアルでは、これらの 2 つのチャネルにボットを接続する方法を説明します。

> [!NOTE]
> Facebook の UI は、使用しているバージョンによって多少異なる場合があります。

## <a name="connect-a-bot-to-facebook-messenger"></a>Facebook Messenger にボットを接続する

Facebook Messenger 向けの開発の詳細については、[Messenger プラットフォームのドキュメント](https://developers.facebook.com/docs/messenger-platform)を参照してください。 Facebook の[プレリリースのガイドライン](https://developers.facebook.com/docs/messenger-platform/product-overview/launch#app_public)、[クイック スタート](https://developers.facebook.com/docs/messenger-platform/guides/quick-start)、および[設定ガイド](https://developers.facebook.com/docs/messenger-platform/guides/setup)を参照してください。

Facebook Messenger を使用して通信するようにボットを構成するには、Facebook ページで Facebook Messenger を有効にしてから、ボットを接続します。

### <a name="copy-the-page-id"></a>ページ ID をコピーする

ボットは Facebook ページからアクセスします。

1. [新しい Facebook ページを作成する](https://www.facebook.com/bookmarks/pages)か、既存のページに移動します。

1. Facebook ページの **[About]\(概要\)** ページを開き、 **[Page ID]\(ページ ID\)** をコピーして保存します。

### <a name="create-a-facebook-app"></a>Facebook アプリを作成する

1. ブラウザーで、[[Create a new Facebook App]\(新しい Facebook アプリの作成\)](https://developers.facebook.com/quickstarts/?platform=web) に移動します。
1. アプリの名前を入力し、 **[Create New Facebook App ID]\(新しい Facebook アプリ ID の作成\)** ボタンをクリックします。

    ![アプリを作成する](media/channels/fb-create-messenger-bot-app.png)

1. 表示されたダイアログ ボックスで、電子メール アドレスを入力し、 **[Create App ID]\(アプリ ID の作成\)** ボタンをクリックします。

    ![アプリ ID を作成する](media/channels/fb-create-messenger-bot-app-id.png)

1. ウィザードの手順を実行します。

1. 必要なチェック情報を入力し、右上にある **[Skip Quick Start]\(クイック スタートをスキップ\)** ボタンをクリックします。

1. 次に表示されるウィンドウの左側のウィンドウで、 *[設定]* を展開し、 **[基本]** をクリックします。

1. 右側のウィンドウで、 **[アプリ ID]** および **[アプリ シークレット]** をコピーして保存します。

    ![アプリ ID とアプリ シークレットをコピー](media/channels/fb-messenger-bot-get-appid-secret.png)

1. 左側のウィンドウの *[設定]* で、 **[詳細]** をクリックします。

1. 右側のウィンドウで、 **[Allow API Access to App Settings]\(アプリ設定への API のアクセスを許可する\)** スライダーを **[はい]** に設定します。

    ![アプリ ID とアプリ シークレットをコピー](media/channels/fb-messenger-bot-api-settings.png)

1. 右下のページで、 **[変更の保存]** ボタンをクリックします。

### <a name="enable-messenger"></a>Messenger を有効にする

1. 左側のウィンドウで、 **[ダッシュボード]** をクリックします。
1. 右側のウィンドウで下にスクロールし、 **[Messenger]** ボックスの **[Set Up]\(設定\)** ボタンをクリックします。 Messenger のエントリは、左側のウィンドウの *[PRODUCTS]\(製品\)* セクションに表示されます。  

    ![Messenger を有効にする](media/channels/fb-messenger-bot-enable-messenger.png)

### <a name="generate-a-page-access-token"></a>ページ アクセス トークンを生成する

1. 左側のウィンドウで、Messenger のエントリの下にある **[設定]** をクリックします。
1. 右側のウィンドウで下にスクロールし、 **[Token Generation]\(トークンの生成\)** セクションでターゲット ページを選択します。

    ![Messenger を有効にする](media/channels/fb-messenger-bot-select-messenger-page.png)

1. アクセス トークンを生成するために、 **[アクセス許可の編集]** ボタンをクリックして、pages_messaging をアプリに許可します。
1. ウィザードの手順に従います。 最後の手順では、既定の設定をそのまま使用し、 **[完了]** ボタンをクリックします。 最後に、**ページ アクセス トークン**が生成されます。

    ![Messenger のアクセス許可](media/channels/fb-messenger-bot-permissions.png)

1. **[Page Access Token]\(ページ アクセス トークン\)** をコピーして保存します。

### <a name="enable-webhooks"></a>Webhook を有効にする

ボットから Facebook Messenger にメッセージやその他のイベントを送信するには、Webhook 統合を有効にする必要があります。 この時点では、Facebook 設定の手順は保留のままにしておき、後で戻ってきます。

1. ブラウザーで新しいウィンドウを開き、[Azure portal](https://portal.azure.com/) に移動します。

1. リソースの一覧でボット リソースの登録をクリックし、関連するブレードで **[チャネル]** をクリックします。

1. 右側のウィンドウで、 **[Facebook]** アイコンをクリックします。

1. ウィザードで、前の手順で保存した Facebook の情報を入力します。 情報が正しい場合、ウィザードの下部に**コールバック URL** と**トークンの確認**が表示されます。 それらをコピーして保存します。  

    ![FB Messenger チャネル構成](media/channels/fb-messenger-bot-config-channel.PNG)

1. **[保存]** ボタンをクリックします。

1. Facebook の設定に戻りましょう。 右側のウィンドウで下にスクロールし、 **[Webhook]** セクションで **[Subscribe To Events]\(イベントのサブスクライブ\)** ボタンをクリックします。 これは、メッセージング イベントを Facebook Messenger からボットに転送するためです。

    ![Webhook を有効にする](media/channels/fb-messenger-bot-webhooks.PNG)

1. 表示されたダイアログ ボックスで、前に保存した **[コールバック URL]** および **[トークンの確認]** の値を入力します。 **[サブスクリプション フィールド]** で、 *[message\_deliveries]* 、 *[messages]* 、 *[messaging\_optins]* 、 *[messaging\_postbacks]* を選択します。

    ![Webhook を構成する](media/channels/fb-messenger-bot-config-webhooks.png)

1. **[確認して保存]** ボタンをクリックします。

1. Webhook をサブスクライブする Facebook ページを選択します。 **[サブスクライブ]** ボタンをクリックします。

    ![Webhook ページを構成する](media/channels/fb-messenger-bot-config-webhooks-page.PNG)

### <a name="submit-for-review"></a>確認用に送信する

Facebook には、基本アプリ設定ページの [Privacy Policy URL]\(プライバシー ポリシーの URL\) と [ Terms of Service URL]\(利用規約の URL\) が必要です。 [[Code of Conduct]\(行動規範\)](https://investor.fb.com/corporate-governance/code-of-conduct/default.aspx) ページには、プライバシー ポリシーを作成するためのサード パーティのリソース リンクがあります。 [[Terms of Use]\(利用規約\)](https://www.facebook.com/terms.php) ページには、適切な利用規約を作成するために役立つサンプルの規約が掲載されています。

ボットが完了した後、Facebook には、Messenger に発行されたアプリの独自の[レビュー処理](https://developers.facebook.com/docs/messenger-platform/app-review)があります。 ボットは Facebook の[プラットフォーム ポリシー](https://developers.facebook.com/docs/messenger-platform/policy-overview)に準拠していることを確認するためにテストされます。

### <a name="make-the-app-public-and-publish-the-page"></a>アプリを発行してページを発行する

> [!NOTE]
> アプリが発行されるまでは、[開発モード](https://developers.facebook.com/docs/apps/managing-development-cycle)段階です。 プラグインと API の機能は、管理者、開発者、およびテスターのみが使用できます。

レビューが正常に完了したら、[App Dashboard]\(アプリ ダッシュボード\) の [App Review]\(アプリ レビュー\) でアプリを [Public]\(パブリック\) に設定します。
このボットに関連付けられた Facebook ページが発行されていることを確認します。 [Pages settings]\(ページ設定\) に状態が表示されます。

## <a name="connect-a-bot-to-facebook-workplace"></a>Facebook Workplace にボットを接続する

> [!NOTE]
> 2019 年 12 月 16 日に、Workplace by Facebook ではカスタム統合のセキュリティ モデルが変更されました。 Microsoft Bot Framework v4 を使用して構築された従来の統合は、2020 年 2 月 28 日より前に、以下の手順に従って、Bot Framework Facebook アダプターを使用するよう更新する必要があります。
>
> Workplace データへの制限付きアクセス (低秘密度のアクセス許可) を使用した統合については、Facebook が認める 2020 年 12 月 31 日までの継続使用の対象と見なされます。ただしその統合が、セキュリティ RFI を実施して合格し、なおかつ、2020 年 1 月 15 日より前に [Direct Support](https://my.workplace.com/work/admin/direct_support) 経由で開発者からアプリの継続使用を要請する連絡があった場合に限られます。
>
> Bot Framework アダプターは、[JavaScript (Node.js)](https://aka.ms/npm-botbuilder-adapter-facebook) および [C# (.NET)](https://aka.ms/botbuilder-dotnet-facebook-adapter) のボット向けに提供されています。

Facebook Workplace は、従業員が簡単に接続して共同作業できる Facebook のビジネス指向バージョンです。 これには、ライブ動画、ニュース フィード、グループ、Messenger、リアクション、検索、およびトレンドの投稿が含まれています。 また、以下もサポートします。

- 分析と統合。 企業が Workplace を既存の IT システムと統合するために使用する分析、統合、シングルサインオン、および ID プロバイダーを備えたダッシュボード。
- 複数の企業のグループ。 さまざまな組織の従業員が連携して共同作業を行うことができる共有スペース。

Facebook Workplace については、[Workplace ヘルプ センター](https://workplace.facebook.com/help/work/)を参照してください。Facebook Workplace の開発に関するガイドラインについては、[Workplace の開発者向けドキュメント](https://developers.facebook.com/docs/workplace)を参照してください。

ボットと共に Facebook Workplace を使用するには、ボットを接続するために、Workplace アカウントとカスタム統合を作成する必要があります。

### <a name="create-a-workplace-premium-account"></a>Workplace プレミアム アカウントを作成する

1. 会社を代表して [Workplace](https://www.facebook.com/workplace) に申請を送信します。
1. 申請が承認されると、参加するよう招待するメールが届きます。 応答にはしばらく時間がかかる場合があります。
1. 招待メールの **[Get Started]\(作業の開始\)** をクリックします。
1. プロファイル情報を入力します。
    > [!TIP]
    > 自分自身をシステム管理者として設定します。 カスタム統合を作成できるのはシステム管理者のみであることに注意してください。
1. **[Preview Profile]\(プロファイルのプレビュー\)** をクリックし、情報が正しいことを確認します。
1. *[Free Trial]\(無料試用版\)* にアクセスします。
1. **パスワード**を作成します。
1. **[Invite Coworkers]\(同僚を招待する\)** をクリックして、サインインするように従業員を招待します。 招待した従業員は、サインインするとすぐにメンバーになります。 彼らは、これらの手順で説明した内容と同様のサインイン プロセスを行います。

### <a name="create-a-custom-integration"></a>カスタム統合を作成する

以下で説明する手順に従って、Workplace の[カスタム統合](https://developers.facebook.com/docs/workplace/custom-integrations-new)を作成します。 カスタム統合を作成すると、アクセス許可が定義されたアプリと、Workplace コミュニティ内でのみ表示される "ボット" 型のページが作成されます。

1. **[Admin Panel]\(管理パネル\)** で、 **[Integrations]\(統合\)** タブを開きます。
1. **[Create your own custom App]\(カスタム アプリの作成\)** ボタンをクリックします。

    ![Workplace の統合](media/channels/fb-integration.png)

1. アプリの表示名とプロファイルの写真を選択します。 これらの情報は、"ボット" 型のページと共有されます。
1. **[Allow API Access to App Settings]\(アプリ設定への API のアクセスを許可する\)** を [Yes]\(はい\) に設定します。
1. 表示されたアプリ ID、アプリ シークレット、およびアプリ トークンをコピーし、安全に保管します。

    ![Workplace のキー](media/channels/fb-keys.png)

1. これで、カスタム統合の作成が完了しました。 次に示すように、Workplace コミュニティ内で "ボット" 型のページを見つけることができます。

    ![Workplace のページ](media/channels/fb-page.png)

### <a name="update-your-bot-code-with-facebook-adapter"></a>Facebook アダプターでボットのコードを更新する

自分のボットのソース コードを更新して、Workplace by Facebook と通信するためのアダプターを追加する必要があります。 アダプターは、[JavaScript (Node.js)](https://aka.ms/npm-botbuilder-adapter-facebook) および [C# (.NET)](https://aka.ms/botbuilder-dotnet-facebook-adapter) のボット向けに提供されています。

### <a name="provide-facebook-credentials"></a>Facebook の資格情報を入力する

前に Facebook Workplace からコピーした **Facebook アプリ ID**、**Facebook アプリ シークレット**、**ページ アクセス トークン**の各値を使用して、自分のボットの appsettings.json を更新する必要があります。 従来のページ ID ではなく、その **[About]\(詳細\)** ページの統合名の後に続く番号を使用します。 [JavaScript (Node.js)](https://aka.ms/npm-botbuilder-adapter-facebook) または [C# (.NET)](https://aka.ms/botbuilder-dotnet-facebook-adapter) で、これらの手順に従ってボットのソース コードを更新してください。

### <a name="submit-for-review"></a>確認用に送信する

詳細については、「**Facebook Messenger にボットを接続する**」セクションと [Workplace の開発者向けドキュメント](https://developers.facebook.com/docs/workplace)を参照してください。

### <a name="make-the-app-public-and-publish-the-page"></a>アプリを発行してページを発行する

詳細については、「**Facebook Messenger にボットを接続する**」セクションを参照してください。

### <a name="setting-the-api-version"></a>API バージョンを設定する

特定のバージョンの Graph API の廃止に関する Facebook からの通知を受け取った場合、[Facebook 開発者向けページ](https://developers.facebook.com)に移動します。 お使いのボットの **[アプリ設定]** に移動し、 **[設定] > [詳細] > [Upgrade API version]\(API バージョンのアップグレード\)** に移動し、 **[Upgrade All Calls]\(すべての呼び出しをアップグレード\)** を 3.0 に切り替えます。

![API バージョンのアップグレード](media/channels/fb-version-upgrade.png)

## <a name="connect-a-bot-to-facebook-using-the-facebook-adapter"></a>Facebook アダプターを使用してボットを Facebook に接続する

ボットを Facebook Workplace に接続するには、Bot Framework Facebook アダプターを使用します。 Facebook Messenger に接続するには、Facebook チャネルまたは Facebook アダプターを使用します。
Facebook アダプターは、[JavaScript (Node.js)](https://aka.ms/npm-botbuilder-adapter-facebook) および [C# (.NET)](https://aka.ms/botbuilder-dotnet-facebook-adapter) のボット向けに提供されています。

この記事では、アダプターを使用してボットを Facebook に接続する方法について説明します。  この記事では、Facebook に接続するよう EchoBot サンプルに変更を加える手順を解説します。

以下の手順では、Facebook アダプターの C# 実装について説明しています。 JavaScript アダプター (BotKit ライブラリに含まれます) の使用手順については、[BotKit Facebook のドキュメント](https://botkit.ai/docs/v4/platforms/facebook.html)を参照してください。

### <a name="prerequisites"></a>前提条件

- [EchoBot サンプル コード](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/02.echo-bot)
- Facebook for Developers アカウント。 アカウントをお持ちでない場合は、[こちらで作成](https://developers.facebook.com)できます。

### <a name="create-a-facebook-app-page-and-gather-credentials"></a>Facebook アプリとページを作成し、資格情報を収集する

1. [https://developers.facebook.com](https://developers.facebook.com) にログインします。 メイン メニューの **[My Apps]\(マイ アプリ\)** をクリックし、ドロップ ダウン メニューの **[Create App]\(アプリの作成\)** をクリックします。

![アプリを作成する](media/bot-service-channel-connect-facebook/create-app-button.png)

1. 表示されたポップアップ ウィンドウで新しいアプリの名前を入力し、 **[Create App ID]\(アプリ ID の作成\)** をクリックします。

![アプリの名前を定義する](media/bot-service-channel-connect-facebook/app-name.png)

#### <a name="set-up-messenger-and-associate-a-facebook-page"></a>Messenger を設定して Facebook ページを関連付ける

1. アプリの作成後、設定可能な製品が一覧表示されます。 **Messenger** 製品の横にある **[Set Up]\(設定\)** ボタンをクリックします。

1. 次に、新しいアプリを Facebook ページに関連付ける必要があります (使用したい既存のページがない場合は、 **[Access Tokens]\(アクセス トークン\)** セクションの **[Create New Page]\(ページの新規作成\)** をクリックして作成できます)。 **[Add or Remove Pages]\(ページの追加または削除\)** をクリックして、アプリの関連付け先のページを選択し、 **[Next]\(次へ\)** をクリックします。 **[Manage and access Page conversations on Messenger]\(Messenger におけるページの会話に対する管理とアクセス\)** 設定を有効のままにして、 **[Done]\(完了\)** をクリックします。

![Messenger を設定する](media/bot-service-channel-connect-facebook/app-page-permissions.png)

1. 自分のページを関連付けたら、 **[Generate Token]\(トークンの生成\)** ボタンをクリックして、ページ アクセス トークンを生成します。  このトークンを書き留めてください。後続の手順でボット アプリケーションを構成する際に必要になります。

#### <a name="obtain-your-app-secret"></a>アプリ シークレットを取得する

1. 左側のメニューの **[Settings]\(設定\)** をクリックし、 **[Basic]\(基本\)** をクリックして、自分のアプリの基本設定ページに移動します。

1. 基本設定ページで、 **[App Secret]\(アプリ シークレット\)** の横にある **[Show]\(表示\)** ボタンをクリックします。  このシークレットを書き留めてください。後続の手順で自分のボット アプリケーションを構成する際に必要になります。

### <a name="wiring-up-the-facebook-adapter-in-your-bot"></a>ボットに Facebook アダプターを接続する

自分の Facebook アプリ、ページ、資格情報が揃ったら、自分のボット アプリケーションを構成する必要があります。

#### <a name="install-the-facebook-adapter-nuget-package"></a>Facebook アダプターの NuGet パッケージをインストールする

[Microsoft.Bot.Builder.Adapters.Facebook](https://www.nuget.org/packages/Microsoft.Bot.Builder.Adapters.Facebook/) NuGet パッケージを追加します。 NuGet の使用方法について詳しくは、[Visual Studio でパッケージをインストールして管理する方法](https://aka.ms/install-manage-packages-vs)に関するページを参照してください。

#### <a name="create-a-facebook-adapter-class"></a>Facebook アダプター クラスを作成する

***FacebookAdapter*** クラスを継承する新しいクラスを作成します。 Facebook チャネルのアダプターとして機能するこのクラスには、エラー処理機能が含まれています (これは Azure Bot Service からの他の要求を処理するために使用される、サンプル内の ***BotFrameworkAdapterWithErrorHandler*** クラスに類似しています)。

```csharp
public class FacebookAdapterWithErrorHandler : FacebookAdapter
{
    public FacebookAdapterWithErrorHandler(IConfiguration configuration, ILogger<BotFrameworkHttpAdapter> logger)
            : base(configuration, logger)
        {
            OnTurnError = async (turnContext, exception) =>
            {
                // Log any leaked exception from the application.
                logger.LogError(exception, $"[OnTurnError] unhandled error : {exception.Message}");

                // Send a message to the user
                await turnContext.SendActivityAsync("The bot encountered an error or bug.");
                await turnContext.SendActivityAsync("To continue to run this bot, please fix the bot source code.");

                // Send a trace activity, which will be displayed in the Bot Framework Emulator
                await turnContext.TraceActivityAsync("OnTurnError Trace", exception.Message, "https://www.botframework.com/schemas/error", "TurnError");
            };
        }
}
```

#### <a name="create-a-new-controller-for-handling-facebook-requests"></a>Facebook の要求を処理するための新しいコントローラーを作成する

Azure Bot Service チャネルからの要求に使用される既定の "api/messages" ではなく、新しいエンドポイント "api/facebook" に、Facebook からの要求を処理する新しいコントローラーを作成します。  自分のボットに新しくエンドポイントを追加することによって、Bot Service チャネルからの要求だけでなく、Facebook からの要求も、同じボットを使用して受け入れることができます。

```csharp
[Route("api/facebook")]
[ApiController]
public class FacebookController : ControllerBase
{
    private readonly FacebookAdapter _adapter;
    private readonly IBot _bot;

    public FacebookController(FacebookAdapter adapter, IBot bot)
    {
        _adapter = adapter;
        _bot = bot;
    }

    [HttpPost]
    [HttpGet]
    public async Task PostAsync()
    {
        // Delegate the processing of the HTTP POST to the adapter.
        // The adapter will invoke the bot.
        await _adapter.ProcessAsync(Request, Response, _bot);
    }
}
```

#### <a name="inject-the-facebook-adapter-in-your-bot-startupcs"></a>ボットの startup.cs に Facebook アダプターを挿入する

自分の startup.cs ファイル内の ***ConfigureServices*** メソッドに次の行を追加します。 これによって Facebook アダプターが登録され、新しいコントローラー クラスで利用できるようになります。  前の手順で追加した構成設定が、アダプターによって自動的に使用されます。

```csharp
services.AddSingleton<FacebookAdapter, FacebookAdapterWithErrorHandler>();
```

追加後の ***ConfigureServices*** メソッドは次のようになります。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    // Create the default Bot Framework Adapter (used for Azure Bot Service channels and emulator).
    services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkAdapterWithErrorHandler>();

    // Create the Facebook Adapter
    services.AddSingleton<FacebookAdapter, FacebookAdapterWithErrorHandler>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```

#### <a name="obtain-a-url-for-your-bot"></a>ボットの URL を取得する

ボット プロジェクトにアダプターを接続したら、Facebook に対し、自分のアプリケーションの適切なエンドポイントを指定して、自分のボットがメッセージを受信できるようにする必要があります。 また、自分のボット アプリケーションの構成を完成させるためにも、この URL が必要となります。

この手順を実行するために、[ボットを Azure にデプロイ](https://aka.ms/bot-builder-deploy-az-cli)し、そのデプロイしたボットの URL を書き留めてください。

> [!NOTE]
> ボットを Azure にデプロイする準備が整っていない場合や、Facebook アダプターを使用している状態でボットをデバッグしたい場合は、[ngrok](https://www.ngrok.com) (Bot Framework エミュレーターを過去に使用したことがある場合は、既にインストールされている可能性があります) などのツールを使用して、ローカルで実行されているボットにトンネル接続し、そのパブリック アクセス用の URL を取得してください。
>
> ngrok トンネルを作成して自分のボットへの URL を取得したい場合は、ターミナル ウィンドウで次のコマンドを使用します (これは、ローカル ボットがポート 3978 で実行されていることを想定しています。それとは異なるポートでボットが実行されている場合は、コマンド内のポート番号を変更してください)。
>
> ```cmd
> ngrok.exe http 3978 -host-header="localhost:3978"
> ```

#### <a name="add-facebook-app-settings-to-your-bots-configuration-file"></a>ボットの構成ファイルに Facebook アプリの設定を追加する

次に示す設定を自分のボット プロジェクトの appSettings.json ファイルに追加します。 **FacebookAppSecret** と **FacebookAccessToken** は、Facebook アプリを作成、構成するときに収集した値を使用して設定してください。 **FacebookVerifyToken** は自分で生成したランダム文字列である必要があります。これは、自分のボットのエンドポイントが Facebook によって呼び出されたときに、その正当性を保証する目的で使用されます。

```json
  "FacebookVerifyToken": "",
  "FacebookAppSecret": "",
  "FacebookAccessToken": ""
```

上記の設定が済んだら、自分のボットを再デプロイ (ngrok を使用してローカルで実行している場合は再起動) する必要があります。

### <a name="complete-configuration-of-your-facebook-app"></a>Facebook アプリの構成を完成させる

最後の手順では、自分のボットがメッセージを確実に受信できるように、新しい Facebook アプリの Messenger エンドポイントを構成します。

1. 自分のアプリのダッシュボード内で、左側のメニューの **[Messenger]** をクリックし、 **[Settings]\(設定\)** をクリックします。

1. **[Webhooks]\(Webhook\)** セクションで、 **[Add Callback URL]\(コールバック URL の追加\)** をクリックします。

1. **[Callback URL]\(コールバック URL\)** ボックスに、自分のボットの URL と、新しく作成したコントローラーで指定した `api/facebook` エンドポイントを入力します。 たとえば、「 `https://yourboturl.com/api/facebook` 」のように入力します。 先ほど作成して自分のボット アプリケーションの appSettings.json ファイルで使用した確認トークンを、 **[Verify Token]\(トークンの確認\)** ボックスに入力します。

    ![コールバック URL を編集する](media/bot-service-channel-connect-facebook/edit-callback-url.png)

1. **[Verify and Save]\(確認して保存\)** をクリックします。 自分のボットが実行中であることを確認します。Facebook がお客様のアプリケーションのエンドポイントに対して要求を行い、**確認トークン**を使用してそれを検証します。

1. コールバック URL が検証されると **[Add Subscriptions]\(サブスクリプションの追加\)** ボタンが表示されるので、それをクリックします。  ポップアップ ウィンドウで次のサブスクリプションを選択して、 **[Save]\(保存\)** をクリックします。

    - **messages**
    - **messaging_postbacks**
    - **messaging_optins**
    - **messaging_deliveries**

    ![Webhook のサブスクリプション](media/bot-service-channel-connect-facebook/webhook-subscriptions.png)

### <a name="test-your-bot-with-adapter-in-facebook"></a>Facebook のアダプターを使用してボットをテストする

新しい Facebook アプリに関連付けた Facebook ページ経由でメッセージを送信して、自分のボットが Facebook に正しく接続されているかどうかをテストしてみましょう。  

1. 自分の Facebook ページに移動します。

1. **[Add a Button]\(ボタンの追加\)** をクリックします。

    ![ボタンの追加](media/bot-service-channel-connect-facebook/add-button.png)

1. **[Contact You]\(連絡する\)** と **[Send Message]\(メッセージを送信する\)** を選択して、 **[Next]\(次へ\)** をクリックします。

    ![ボタンの追加](media/bot-service-channel-connect-facebook/button-settings.png)

1. **"Where would you like this button to send people to? (このボタンをクリックした人をどこに誘導しますか)"** と聞かれたら、 **[Messenger]** を選択し、 **[Finish]\(完了\)** をクリックします。

    ![ボタンの追加](media/bot-service-channel-connect-facebook/button-settings-2.png)

1. 新しい **[Send Message]\(メッセージの送信\)** ボタンが Facebook ページに表示されるので、そこにマウス ポインターを合わせ、ポップアップ メニューから **[Test Button]\(ボタンのテスト\)** をクリックします。  これにより、自分のアプリとの間で Facebook Messenger を介した新しい会話が開始され、自分のボットへのメッセージ送信をテストすることができます。 ボットはメッセージを受信すると、そのメッセージのテキストをエコーする形でメッセージを返します。

この機能のテストには、[Facebook アダプターのサンプル ボット](https://aka.ms/csharp-61-facebook-adapter-sample)を使用することもできます。上記の手順で説明したのと同じ値を appSettings.json ファイルに設定してください。

## <a name="see-also"></a>関連項目

- **サンプル コード**。 [Facebook-events](https://aka.ms/facebook-events) のサンプル ボットを使用して、Facebook Messenger とのボット通信を探索します。
