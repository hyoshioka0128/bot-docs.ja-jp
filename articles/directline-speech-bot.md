---
title: Direct Line Speech ボットを開発する | Microsoft Docs
description: Direct Line Speech ボットを開発します
keywords: Direct Line Speech ボットの開発, Speech ボット
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 07/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d4b44fdc2d063c2d91020435473346fdfbda589b
ms.sourcegitcommit: 6a83b2c8ab2902121e8ee9531a7aa2d85b827396
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/09/2019
ms.locfileid: "68970614"
---
# <a name="use-direct-line-speech-in-your-bot"></a>ボットで Direct Line Speech を使用する

[!INCLUDE [applies-to-v4](includes/applies-to.md)]

Direct Line Speech では、Direct Line Speech チャネルとボットの間のメッセージ交換に、Bot Framework の新しい WebSocket をベースにしたストリーミング機能が使用されます。 Azure portal で Direct Line Speech チャネルを有効にした後、これらの WebSocket 接続をリッスンし、受け入れるように、ご自身のボットを更新する必要があります。 これらの手順ではその方法を説明します。

## <a name="add-the-nuget-package"></a>NuGet パッケージを取得する
Direct Line Speech プレビューについては、ご自身のボットに追加する必要がある追加 NuGet パッケージが存在します。 これらのパッケージは、myget.org NuGet フィードでホストされています。
1.  Visual Studio でボットのプロジェクトを開きます。

2.  ご自身のボット プロジェクトのプロパティで、[NuGet パッケージの管理] に移動します。

3.  `Microsoft.Bot.Builder.StreamingExtensions` パッケージを追加します。 プレビュー パッケージを表示するには、[プレリリースを含める] をオンにする必要があります。

4.  プロジェクトへのパッケージ追加を完了するプロンプトを受け入れます。

## <a name="set-the-speak-field-on-activities-you-want-spoken-to-the-user"></a>ユーザーに対して読み上げるアクティビティの Speak フィールドを設定する
ボットから送信される、ユーザーに対して読み上げる任意のアクティビティの Speak フィールドを設定する必要があります。 

```cs
public IActivity Speak(string message)
{
    var activity = MessageFactory.Text(message);
    string body = @"<speak version='1.0' xmlns='https://www.w3.org/2001/10/synthesis' xml:lang='en-US'>
        <voice name='Microsoft Server Speech Text to Speech Voice (en-US, JessaNeural)'>" +
        $"{message}" + "</voice></speak>";
    activity.Speak = body;
    return activity;
}
```

## <a name="option-1-update-your-net-core-bot-code-_if-your-bot-has-a-botcontrollercs_"></a>オプション 1:"_ボットに BotController.cs が含まれる場合_" に、.NET Core ボット コードを更新する
EchoBot などのテンプレートのいずれかを使用して、Azure portal から新しいボットを作成する場合、単一の POST エンドポイントを公開する ASP.NET MVC コントローラーが含まれるボットが表示されます。 これらの手順では、エンドポイントも公開するように、これを展開する方法について説明します。これにより、GET エンドポイントである WebSocket ストリーミング エンドポイントが受け入れられます。
1.  ご自身のソリューションで、Controllers フォルダーの BotController.cs を開きます

2.  クラスで PostAsync メソッドを検索し、その装飾を [HttpPost] から [HttpPost, HttpGet] に更新します。
```cs
[HttpPost, HttpGet]
public async Task PostAsync()
{ 
    await _adapter.ProcessAsync(Request, Response, _bot);
}
```

3.  BotController.cs を保存して閉じます

4.  ご自身のソリューションのルートで Startup.cs を開きます。

5.  新しい名前空間を追加します。

```cs
using Microsoft.Bot.Builder.StreamingExtensions;
```

6.  ConfigureServices メソッドで、AdapterWithErrorHandler の使用を、適切なservices.AddSingleton 呼び出しの WebSocketEnabledHttpAdapter に置き換えます。

```cs
public void ConfigureServices(IServiceCollection services)
{
    ...    

    // Create the Bot Framework Adapter.
    services.AddSingleton<IBotFrameworkHttpAdapter, WebSocketEnabledHttpAdapter>();

    services.AddTransient<IBot, EchoBot>();

    ...
}
```

7. 引き続き Startup.cs で、Configure メソッドの下部に移動します。 app.UseMvc(); 呼び出しの前で、app.UseWebSockets(); を呼び出して (Use 呼び出しの順序は重要なので、これは重要です)、追加します。 メソッドが完了すると、以下のような内容になっているはずです。

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

8.  ご自身のボット コードの残りの部分は同じままです。

## <a name="option-2-update-your-net-core-bot-code-_if-your-bot-uses-addbot-and-usebotframework-instead-of-a-botcontroller_"></a>オプション 2:"_BotController ではなく AddBot と UseBotFramework がボットで使用されている場合_" に、.NET Core ボット コードを更新する

バージョン 4.3.2 よりも前の Bot Builder SDK v4 を使用してボットを作成した場合、そのボットに BotController が含まれず、代わりに Startup.cs ファイルの AddBot() メソッドと UseBotFramework() メソッドが使用され、ボットがメッセージを受信する POST エンドポイントが公開されている可能性があります。 新しいストリーミング エンドポイントを公開するには、BotController を追加し、AddBot() メソッドと UseBotFramework() メソッドを削除する必要があります。 これらの手順では、必要な変更について説明します。

1.  BotController.cs というファイルを追加して、新しい MVC コントローラーをご自身のボット プロジェクトに追加します。 コントローラー コードをこのファイルに追加します。

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
2.  Startup.cs ファイルで、Configure メソッド特定します。 UseBotFramework() 行を削除し、これらの行があるかどうかを確認します。

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

3.  引き続き Startup.cs ファイルで、ConfigureServices メソッド特定します。 AddBot() 行を削除し、これらの行があるかどうかを確認します。

```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    services.AddSingleton<ICredentialProvider, ConfigurationCredentialProvider>();

    services.AddSingleton<IChannelProvider, ConfigurationChannelProvider>();

    // Create the Bot Framework Adapter.
    services.AddSingleton<IBotFrameworkHttpAdapter, WebSocketEnabledHttpAdapter>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```
4.  ご自身のボット コードの残りの部分は同じままです。

## <a name="additional-information"></a>追加情報

アクティビティの操作の詳細については、「[ボットのしくみ](v4sdk/bot-builder-basics.md)」および[テキスト メッセージを送受信する](v4sdk/bot-builder-howto-send-messages.md)方法を参照してください。

## <a name="next-steps"></a>次の手順
> [!div class="nextstepaction"]
> [ボットを Direct Line Speech に接続する (プレビュー)](./bot-service-channel-connect-directlinespeech.md)
