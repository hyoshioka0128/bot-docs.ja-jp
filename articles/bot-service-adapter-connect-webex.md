---
title: ボットを Webex Teams に接続する | Microsoft Docs
description: Webex アダプターを介してボットから Webex への接続を構成する方法について説明します。
keywords: ボット アダプター, Webex, Webex ボット
author: garypretty
manager: kamrani
ms.topic: article
ms.author: gapretty
ms.service: bot-service
ms.date: 01/21/2020
ms.openlocfilehash: 2bf7d99d221a2e48c938c66fb7bef5c1e143f47f
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "76752804"
---
# <a name="connect-a-bot-to-webex-teams-using-the-webex-adapter"></a>Webex アダプターを使用して Webex Teams にボットを接続する

この記事では、SDK で提供されるアダプターを使用して、Webex にボットを接続する方法について説明します。  この記事では、Webex アプリに接続するよう EchoBot サンプルに変更を加える手順を解説します。

> [!NOTE]
> 以下の手順では、Webex アダプターの C# 実装について説明しています。 JavaScript 実装 (BotKit ライブラリに含まれます) の使用手順については、[BotKit Webex のドキュメント](https://botkit.ai/docs/v4/platforms/webex.html)を参照してください。

## <a name="prerequisites"></a>前提条件

* [EchoBot サンプル コード](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/02.echo-bot)

* [https://developer.webex.com/my-apps](https://developer.webex.com/my-apps) でログインしてアプリケーションを作成または管理するのに十分なアクセス許可で、Webex チームにアクセスします。 Webex チームへのアクセス権がない場合は、[www.webex.com](https://www.webex.com) で無料のアカウントを作成できます。

## <a name="create-a-webex-bot-app"></a>Webex ボット アプリを作成する

1. [Webex 開発者向けダッシュボード](https://developer.webex.com/my-apps)にログインし、[Create a new app]\(新しいアプリの作成\) ボタンをクリックします。

2. 次の画面で、[Create a bot]\(ボットの作成\) をクリックして Webex ボットの作成を選択します。

3. 次の画面で、ボットの適切な名前、ユーザー名、説明を入力し、アイコンを選ぶか、ご自身の画像をアップロードします。

    ![ボットを設定する](~/media/bot-service-adapter-connect-webex/create-bot.png)

    [Add bot]\(ボットの追加\) ボタンをクリックします。

4. 次のページで、新しい Webex アプリのアクセス トークンが提供されるので、そのトークンを書き留めてください。ボットを構成する際に必要になります。

    ![ボットを設定する](~/media/bot-service-adapter-connect-webex/create-bot-settings.png)

## <a name="wiring-up-the-webex-adapter-in-your-bot"></a>ボットに Webex アダプターを接続する

Webex アプリの構成を完了するには、Webex アダプターを自分のボットに接続しておく必要があります。

### <a name="install-the-webex-adapter-nuget-package"></a>Webex アダプターの NuGet パッケージをインストールする

[Microsoft.Bot.Builder.Adapters.Webex](https://www.nuget.org/packages/Microsoft.Bot.Builder.Adapters.Webex/) NuGet パッケージを追加します。 NuGet の使用方法について詳しくは、[Visual Studio でパッケージをインストールして管理する方法](https://aka.ms/install-manage-packages-vs)に関するページを参照してください

### <a name="create-a-webex-adapter-class"></a>Webex アダプター クラスを作成する

***WebexAdapter*** クラスを継承する新しいクラスを作成します。 このクラスは、Webex チャネルのアダプターとして機能します。 Azure Bot Service からの要求を処理するために使用される ***BotFrameworkAdapterWithErrorHandler*** クラスがサンプルに含まれていますが、それによく似たエラー処理機能を備えています。

```csharp
public class WebexAdapterWithErrorHandler : WebexAdapter
{
    public WebexAdapterWithErrorHandler(IConfiguration configuration, ILogger<BotFrameworkHttpAdapter> logger)
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

### <a name="create-a-new-controller-for-handling-webex-requests"></a>Webex の要求を処理するための新しいコントローラーを作成する

Azure Bot Service チャネルからの要求に使用される既定の "api/messages" ではなく、新しいエンドポイント "api/webex" に、自分の Webex アプリからの要求を処理する新しいコントローラーを作成します。  自分のボットに新しくエンドポイントを追加することによって、Bot Service チャネル (または追加のアダプター) からの要求だけでなく、Webex からの要求も、同じボットを使用して受け入れることができます。

```csharp
[Route("api/webex")]
[ApiController]
public class WebexController : ControllerBase
{
    private readonly WebexAdapter _adapter;
    private readonly IBot _bot;

    public WebexController(WebexAdapter adapter, IBot bot)
    {
        _adapter = adapter;
        _bot = bot;
    }

    [HttpPost]
    public async Task PostAsync()
    {
        // Delegate the processing of the HTTP POST to the adapter.
        // The adapter will invoke the bot.
        await _adapter.ProcessAsync(Request, Response, _bot);
    }
}
```

### <a name="inject-webex-adapter-in-your-bot-startupcs"></a>ボットの Startup.cs に Webex アダプターを挿入する

自分の Startup.cs ファイル内の ***ConfigureServices*** メソッドに次の行を追加します。これによって Webex アダプターが登録され、新しいコントローラー クラスで利用できるようになります。  次の手順で説明する構成設定が、アダプターによって自動的に使用されます。

```csharp
services.AddSingleton<WebexAdapter, WebexAdapterWithErrorHandler>();
```

追加後の ***ConfigureServices*** メソッドは次のようになります。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    // Create the default Bot Framework Adapter (used for Azure Bot Service channels and emulator).
    services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkAdapterWithErrorHandler>();

    // Create the Webex Adapter
    services.AddSingleton<WebexAdapter, WebexAdapterWithErrorHandler>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```

### <a name="add-webex-adapter-settings-to-your-bots-configuration-file"></a>ボットの構成ファイルに Webex アダプターの設定を追加する

1. 自分のボット プロジェクトの appSettings.json ファイルに次の 4 つの設定を追加します。

    ```json
      "WebexAccessToken": "",
      "WebexPublicAddress": "",
      "WebexSecret": "",
      "WebexWebhookName": ""
    ```

2. Webex ボット アクセス トークン内の **WebexAccessToken** 設定に、前の手順で自分の Webex ボット アプリを作成する際に生成したものを入力します。 現時点では、他の 3 つの設定は空のままにしておいてください。これらに必要な情報は後続の手順で収集します。

## <a name="complete-configuration-of-your-webex-app-and-bot"></a>Webex アプリとボットの構成を完成させる

### <a name="create-and-update-a-webex-webhook"></a>Webex Webhook を作成して更新する

自分のボット プロジェクトで Webex アプリを作成し、アダプターを接続したら、最後の手順となります。Webex Webhook を構成して、ボット上の正しいエンドポイントを指すよう設定し、自分のアプリをサブスクライブして、ボットがメッセージと添付ファイルを確実に受信できるようにしましょう。 そのためには、自分のボットが実行されていて、エンドポイントへの URL が有効であることを Webex が検証できるようになっている必要があります。

1. この手順を実行するために、[ボットを Azure にデプロイ](https://aka.ms/bot-builder-deploy-az-cli)し、そのデプロイしたボットへの URL を書き留めてください。 Webex メッセージングのエンドポイントは、自分のボットの URL です。これは、デプロイされているアプリケーション (または ngrok エンドポイント) の URL に "/api/webex" を追加したものになります (例: `https://yourbotapp.azurewebsites.net/api/webex`)。

    > [!NOTE]
    > 自分のボットを Azure にデプロイする準備が整っていない場合や、Webex アダプターを使用している状態でボットをデバッグしたい場合は、[ngrok](https://www.ngrok.com) (Bot Framework エミュレーターを過去に使用したことがある場合は、既にインストールされている可能性があります) などのツールを使用して、ローカルで実行されているボットにトンネル接続し、そのパブリック アクセス用の URL を取得してください。
    >
    > ngrok トンネルを作成して自分のボットへの URL を取得したい場合は、ターミナル ウィンドウで次のコマンドを使用します (これは、ローカル ボットがポート 3978 で実行されていることを想定しています。それとは異なるポートでボットが実行されている場合は、コマンド内のポート番号を変更してください)。
    >
    > ```cmd
    > ngrok.exe http 3978 -host-header="localhost:3978"
    > ```

2. [https://developer.webex.com/docs/api/v1/webhooks](https://developer.webex.com/docs/api/v1/webhooks) に移動します。

3. "Update a Webhook" という説明が記載されている PUT メソッドのリンク `https://api.ciscospark.com/v1/webhooks/{webhookId}` をクリックします。 エンドポイントに要求を送信するためのフォームが展開されます。

    ![ボットを設定する](~/media/bot-service-adapter-connect-webex/webex-webhook-put-endpoint.png)

4. フォームに次の詳細を入力します。

    * 名前 (Webhook の名前を入力します。例: "Messages Webhook")
    * ターゲット URL (自分のボットの Webex エンドポイントへの完全な URL を入力します。例: `https://yourbotapp.azurewebsites.net/api/webex`)
    * シークレット (ここには、Webhook をセキュリティで保護するための任意のシークレットを入力してください)
    * 状態 (既定の設定である "active" のままにしておきます)

    ![ボットを設定する](~/media/bot-service-adapter-connect-webex/webex-webhook-form.png)

5. **[Run]** をクリックすると、Webhook が作成され、成功のメッセージが表示されます。

### <a name="complete-the-remaining-settings-in-your-bot-application"></a>ボット アプリケーションの残りの設定を完了する

自分のボットの appsettings.json ファイルで未設定となっていた 3 つの設定を完了します (**WebexAccessToken** については、前の手順で設定済みです)。

* WebexPublicAddress (自分のボットの Webex エンドポイントへの完全な URL)
* WebexSecret (前の手順で Webhook を作成する際に指定したシークレット)
* WebexWebhookName (前の手順で指定した Webhook の名前)

## <a name="re-deploy-your-bot-in-your-webex-team"></a>Webex チームにボットを再デプロイする

ボットの appsettings.json の設定に対する構成が完了したら、自分のボットを再デプロイ (ngrok を使用してローカル エンドポイントにトンネル接続している場合はボットを再起動) する必要があります。  Webex アプリとボットの構成はこれで完了です。  
[https://www.webex.com](https://www.webex.com) で自分の Webex チームにログインし、他の人に連絡するのと同じ方法で、ボットにメッセージを送信してチャットできるようになりました。

![ボットを設定する](~/media/bot-service-adapter-connect-webex/webex-contact-person.png)
