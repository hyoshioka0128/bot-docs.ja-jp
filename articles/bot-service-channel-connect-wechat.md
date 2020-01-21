---
title: ボットを WeChat に接続する - Bot Service
description: ボットから WeChat への接続を構成する方法について説明します。
keywords: WeChat, Tencent, ボット チャネル, WeChat アプリ, WeChat ボット, アプリ ID, アプリ シークレット, 資格情報
author: seaen
manager: kamrani
ms.topic: article
ms.author: egorn
ms.service: bot-service
ms.date: 11/01/2019
ms.openlocfilehash: 9abba3093ce819f7ebc07bb03e342da797971f25
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791793"
---
# <a name="connect-a-bot-to-wechat"></a>ボットを WeChat に接続する

WeChat Official Accounts Platform を使用してユーザーと通信するようにボットを構成できます。

## <a name="download-wechat-adapter-for-bot-framework"></a>Bot Framework 用 WeChat アダプターのダウンロード

Microsoft Bot Framework 用の WeChat アダプターは、GitHub のオープンソース アダプターです。 [Bot Framework 用 WeChat アダプターをダウンロードしてください](https://github.com/microsoft/BotFramework-WeChat/)。

## <a name="create-a-wechat-account"></a>WeChat アカウントを作成する

WeChat を使用して会話するようにボットを構成するには、[WeChat Official Account Platform](https://mp.weixin.qq.com/?lang=en_US) に正式なアカウントを作成し、ボットをアプリに接続する必要があります。 現在は、サービス アカウントのみがサポートされています。

### <a name="change-your-prefer-language"></a>希望の言語を変更する

ログインする前に希望する表示言語を変更することができます。

 ![change_language](./media/channels/wechat-change-language.png)

### <a name="register-a-service-account"></a>サービス アカウントを登録する

実際のサービス アカウントは、WeChat による確認が必要であり、アカウントを確認する前に Webhook を有効にすることはできません。 独自のサービス アカウントを作成するには、[こちら](https://kf.qq.com/product/weixinmp.html#hid=87)の指示に従ってください。
簡単に説明すると、上部にある [Register Now] をクリックし、[Service Account] を選択して指示に従います。

 ![register_account](./media/channels/wechat-register-account.png)

### <a name="sandbox-account"></a>サンドボックス アカウント

WeChat とボットの統合をテストするだけの場合は、サービス アカウントを作成する代わりに、サンドボックス アカウントを使用できます。 [サンドボックス アカウント](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login)の作成の詳細をご確認ください。

## <a name="enable-wechat-adapter-to-bot"></a>ボットに対して WeChat アダプターを有効にする

ボット プロジェクトは、通常の Bot Framework SDK V4 プロジェクトです。 起動する前に、ボットを実行できることを確認する必要があります。 [Bot Framework 用 WeChat アダプター](https://github.com/microsoft/BotFramework-WeChat/)をダウンロードしてください。

### <a name="prerequisites"></a>前提条件

    .NET Core SDK (version 2.2.x)

### <a name="add-reference-to-wechat-adapter-source"></a>WeChat アダプター ソースへの参照の追加

WeChat アダプター プロジェクトを直接参照するか、~/BotFramework-WeChat/libraries/csharp_dotnetcore/outputpackages をローカルの NuGet ソースとして追加してください。

### <a name="inject-wechat-adapter-in-your-bot-startupcs"></a>Bot Startup.cs に WeChat アダプターを挿入する

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);

    // Create the storage we'll be using for User and Conversation state. (Memory is great for testing purposes.)
    services.AddSingleton<IStorage, MemoryStorage>();

    // Create the User state. (Used in this bot's Dialog implementation.)
    services.AddSingleton<UserState>();

    // Create the Conversation state. (Used by the Dialog system itself.)
    services.AddSingleton<ConversationState>();

    // Load WeChat settings.
    var wechatSettings = new WeChatSettings();
    Configuration.Bind("WeChatSettings", wechatSettings);
    services.AddSingleton<WeChatSettings>(wechatSettings);

    // Configure hosted serivce.
    services.AddSingleton<IBackgroundTaskQueue, BackgroundTaskQueue>();
    services.AddHostedService<QueuedHostedService>();
    services.AddSingleton<WeChatHttpAdapter>();

    // The Dialog that will be run by the bot.
    services.AddSingleton<MainDialog>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```

### <a name="update-your-bot-controller"></a>ボット コントローラーの更新

```csharp
[Route("api/messages")]
[ApiController]
public class BotController : ControllerBase
{  
    private readonly IBot _bot;
    private readonly WeChatHttpAdapter _weChatHttpAdapter;
    private readonly string Token;
    public BotController(IBot bot, WeChatHttpAdapter weChatAdapter)
    {
        _bot = bot;
        _weChatHttpAdapter = weChatAdapter;
    }

    [HttpPost("/WeChat")]
    [HttpGet("/WeChat")]
    public async Task PostWeChatAsync([FromQuery] SecretInfo secretInfo)
    {
        // Delegate the processing of the HTTP POST to the adapter.
        // The adapter will invoke the bot.
        await _weChatHttpAdapter.ProcessAsync(Request, Response, _bot, secretInfo);
    }
}
```

### <a name="setup-appsettingsjson"></a>appsettings.json の設定

ボットを起動する前に、appsettings.json を設定する必要があります。必要なものは以下で確認できます。

```json
"WeChatSettings": {
    "UploadTemporaryMedia": true,
    "PassiveResponseMode": false,
    "Token": "",
    "EncodingAESKey": "",
    "AppId": "",
    "AppSecret": ""
}
```

#### <a name="service-account"></a>サービス アカウント

既にサービス アカウントがあり、デプロイの準備ができている場合は、次のように、左側のナビゲーション バーの基本構成で、**AppID**、**AppSecret**、**EncodingAESKey**、および **Token** を見つけることができます。

IP ホワイトリストを設定する必要があることを忘れないでください。そうしなければ、WeChat は要求を受け入れません。

 ![serviceaccount_console](./media/channels/wechat-serviceaccount-console.png)

#### <a name="sandbox-account"></a>サンドボックス アカウント

サンドボックス アカウントに **EncodingAESKey** がなく、WeChat からのメッセージが暗号化されていなかった場合、EncodingAESKey を空白のままにします。 ここでは 3 つのパラメーター、**appID**、**appsecret**、**Token** だけがあります。

 ![sandbox_account](./media/channels/wechat-sandbox-account.png)

### <a name="start-bot-and-set-endpoint-url"></a>ボットを開始してエンドポイント URL を設定する

ボット バックエンドを設定できるようになりました。 この操作を実行するには、設定を保存する前にボットを開始する必要があります。これにより、WeChat は URL を確認する要求を送信します。
次のようなパターンでエンドポイントを設定してください: **https://your_end_point/WeChat** 。または BotController.cs で行ったものと同じように個人用設定を設定してください

 ![sandbox_account2](./media/channels/wechat-sandbox-account-2.png)

### <a name="subscribe-your-official-account"></a>オフィシャル アカウントをサブスクライブする

WeChat にテスト アカウントをサブスクライブする QR コードがあります。

 ![subscribe](./media/channels/wechat-subscribe.png)

## <a name="test-through-wechat"></a>WeChat によるテスト

すべてが完了したら、WeChat クライアントで試してみることができます。 テスト フォルダーでサンプル ボットを試すことができます。 このサンプル ボットには、WeChat アダプターがあり、エコー ボットやカード ボットと統合されています。

 ![チャット](./media/channels/wechat-chat.png)
