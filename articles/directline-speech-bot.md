---
title: Direct Line Speech ボットを開発する - Bot Service
description: Direct Line Speech ボットを開発します
keywords: Direct Line Speech ボットの開発, Speech ボット
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/01/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2aaafb7c46097a178761b356299612d874b1a823
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75795965"
---
# <a name="use-direct-line-speech-in-your-bot"></a>ボットで Direct Line Speech を使用する

[!INCLUDE [applies-to-v4](includes/applies-to.md)]

Direct Line Speech では、Direct Line Speech チャネルとボットの間のメッセージ交換に、Bot Framework の新しい WebSocket をベースにしたストリーミング機能が使用されます。 Azure portal で Direct Line Speech チャネルを有効にした後、これらの WebSocket 接続をリッスンし、受け入れるように、ご自身のボットを更新する必要があります。 これらの手順ではその方法を説明します。  

## <a name="step-1-upgrade-to-the-46-sdk"></a>手順 1:4.6 SDK にアップグレードする 

Direct Line Speech の場合は、Bot Builder SDK のバージョン 4.6 以降を使用していることを確認します。 

## <a name="step-2-update-your-net-core-bot-codeif-your-bot-uses-addbot-and-usebotframework-instead-of-a-botcontroller"></a>手順 2:BotController ではなく AddBot と UseBotFramework がボットで使用されている場合に、.NET Core ボット コードを更新する 

バージョン 4.3.2 よりも前の Bot Builder SDK v4 を使用してボットを作成した場合、そのボットに BotController が含まれず、代わりに Startup.cs ファイルの AddBot() メソッドと UseBotFramework() メソッドが使用され、ボットがメッセージを受信する POST エンドポイントが公開されている可能性があります。 新しいストリーミング エンドポイントを公開するには、BotController を追加し、AddBot() メソッドと UseBotFramework() メソッドを削除する必要があります。 これらの手順では、必要な変更について説明します。 これらの変更を既に行っている場合は、次の手順に進みます。 

BotController.cs というファイルを追加して、新しい MVC コントローラーをご自身のボット プロジェクトに追加します。 コントローラー コードをこのファイルに追加します。 

```cs

[Route("api/messages")] 

[ApiController] 

public class BotController : ControllerBase 
{ 
    private readonly IBotFrameworkHttpAdapter _adapter; 
    private readonly IBot _bot; 
    public BotController(IBotFrameworkHttpAdapter adapter, IBot bot) 
    { 
        _adapter = adapter; 

        _bot = bot; 
    } 

    [HttpPost, HttpGet] 
    public async Task ProcessMessageAsync() 
    { 
        await _adapter.ProcessAsync(Request, Response, _bot); 
    } 
} 
```

**Startup.cs** ファイルで、Configure メソッド特定します。 `UseBotFramework()` 行を削除し、`UseWebSockets` にこれらの行があるかどうかを確認します。 

```cs

public void Configure(IApplicationBuilder app, IHostingEnvironment env) 
{ 
    ... 
    app.UseDefaultFiles(); 
    app.UseStaticFiles(); 
    app.UseWebSockets(); 
    app.UseMvc(); 
    ... 
} 
```

引き続き Startup.cs ファイルで、ConfigureServices メソッド特定します。 `AddBot()` 行を削除し、`IBot` と `BotFrameworkHttpAdapter` を追加するための行があることを確認します。 

```cs

public void ConfigureServices(IServiceCollection services) 
{ 
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1); 
    services.AddSingleton<ICredentialProvider, ConfigurationCredentialProvider>(); 
    services.AddSingleton<IChannelProvider, ConfigurationChannelProvider>(); 
    
    // Create the Bot Framework Adapter. 
    services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkHttpAdapter>(); 

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot. 
    services.AddTransient<IBot, EchoBot>(); 
} 
```

ご自身のボット コードの残りの部分は同じままです。 

## <a name="step3-ensure-websockets-are-enabled"></a>手順 3:WebSocket が有効になっていることを確認する 

EchoBot などのテンプレートのいずれかを使用して、Azure portal から新しいボットを作成する場合、GET と POST エンドポイントを公開し、WebSocket も使用する ASP.NET MVC コントローラーが含まれるボットが表示されます。 次の手順では、アップグレードする場合、またはテンプレートからボットを作成しなかった場合に、これらの要素をボットに追加する方法を説明します。 

ご自身のソリューションで、Controllers フォルダーの **BotController.cs** を開きます 

クラスで `PostAsync` メソッドを検索し、その装飾を [HttpPost] から [HttpPost, HttpGet] に更新します。 

```cs

[HttpPost, HttpGet] 
public async Task PostAsync() 
{ 
    await _adapter.ProcessAsync(Request, Response, _bot); 
} 
```

BotController.cs を保存して閉じます 

ご自身のソリューションのルートで **Startup.cs** を開きます。 

Startup.cs で、Configure メソッドの下部に移動します。  `app.UseMvc()` の呼び出しの前に、 `app.UseWebSockets()` の呼び出しを追加します。 これは重要です。これらの use 呼び出しの順序が重要であるためです。 メソッドが完了すると、このような内容になっているはずです。 

```cs
public void Configure(IApplicationBuilder app, IHostingEnvironment env) 
{ 
    ... 
    app.UseDefaultFiles(); 
    app.UseStaticFiles(); 
    app.UseWebSockets(); 
    app.UseMvc(); 
    ... 
} 

```
ご自身のボット コードの残りの部分は同じままです。 

 

手順 4:必要に応じて、アクティビティの [読み上げ] フィールドを設定して、ユーザーに読み上げられるものをカスタマイズする 

既定では、ユーザーに Direct Line Speech を介して送信されるすべてのメッセージが読み上げられます。  

必要に応じて、ボットから送信されたアクティビティの [読み上げ] フィールドを設定することによって、メッセージの読み上げ方法をカスタマイズできます。 

```cs 

public IActivity Speak(string message) 
{ 
    var activity = MessageFactory.Text(message); 
    string body = @"<speak version='1.0' xmlns='https://www.w3.org/2001/10/synthesis' xml:lang='en-US'> 

        <voice name='Microsoft Server Speech Text to Speech Voice (en-US, JessaRUS)'>" + 
        $"{message}" + "</voice></speak>"; 

    activity.Speak = body; 
    return activity; 
} 
```

次のスニペットは、前述の Speak 関数の使用法を示します。 

```cs

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken) 
{ 
    await turnContext.SendActivityAsync(Speak($"Echo: {turnContext.Activity.Text}"), cancellationToken); 
} 
``` 

## <a name="additional-information"></a>関連情報 

- ボイスが有効なボットを作成して使用する完全な例については、「[チュートリアル: Speech SDK を使用して音声でボットを有効にする](https://docs.microsoft.com/ja-jp/azure/cognitive-services/speech-service/tutorial-voice-enable-your-bot-speech-sdk)」を参照してください。 

- アクティビティの操作の詳細については、「[ボットのしくみ](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-basics)」および [テキスト メッセージを送受信する方法](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-howto-send-messages?view=azure-bot-service-4.0)に関する記事を参照してください。 

 
