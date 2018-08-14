---
title: Bot Builder SDK for .NET を使用したボットの作成 | Microsoft Docs
description: ボットの強力な構築フレームワークである Bot Builder SDK for .NET を使用してボットを作成します。
keywords: Bot Builder SDK, ボットの作成, クイック スタート, .NET, 使用の開始
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 04/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 98c6103f0cd38bf6d3f477735dafcf01952304d5
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304148"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-v4-for-net"></a>Bot Builder SDK v4 for .NET を使用したボットの作成
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

このクイック スタートでは、Bot Application テンプレートと Bot Builder SDK for .NET を使用してボットを作成し、Bot Framework Emulator でテストする方法を説明します。 これは、[Microsoft Bot Builder SDK v4](https://github.com/Microsoft/botbuilder-dotnet) に基づいています。

## <a name="prerequisites"></a>前提条件
- Visual Studio [2017](https://www.visualstudio.com/downloads)
- [C#](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4) 用 Bot Builder SDK v4 テンプレート
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)
- [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) および [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index) での非同期プログラミングに関する知識

## <a name="create-a-bot"></a>ボットの作成
前提条件セクションでダウンロードした BotBuilderVSIX.vsix テンプレートをインストールします。 

Visual Studio で、新しいボット プロジェクトを作成します。

![Visual Studio プロジェクト](../media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> 必要に応じて、[NuGet パッケージ](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio)を更新します。

## <a name="explore-code"></a>コードの確認
Startup.cs ファイルを開いて `ConfigureServices(IServiceCollection services)` メソッドのコードを確認します。 `CatchExceptionMiddleware` ミドルウェアがメッセージング パイプラインに追加されています。 これにより、他のミドルウェアまたは OnTurn メソッドによってスローされた例外が処理されます。 

```cs
options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
{
    await context.TraceActivity("EchoBot Exception", exception);
    await context.SendActivity("Sorry, it looks like something went wrong!");
}));
```

会話の状態ミドルウェアは、インメモリ ストレージを使用します。 このミドルウェアはストレージに対して EchoState の読み取りと書き込みを実行します。  EchoState クラスのターン カウントはボットに送信されるメッセージ数を記録します。 同様の手法を使用して、ターン間の状態を維持することができます。

```cs
 IStorage dataStore = new MemoryStorage();
 options.Middleware.Add(new ConversationState<EchoState>(dataStore));
```

`Configure(IApplicationBuilder app, IHostingEnvironment env)` メソッドは `.UseBotFramework` を呼び出して、ボット アダプターに受信アクティビティをルーティングします。 

EchoBot.cs ファイルの `OnTurn(ITurnContext context)` メソッドは、受信アクティビティの種類を確認し、ユーザーに返信を送信するために使用されます。 

```cs
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        var state = context.GetConversationState<EchoState>();

        // Bump the turn count. 
        state.TurnCount++;

        // Echo back to the user whatever they typed.
        await context.SendActivity($"Turn {state.TurnCount}: You sent '{context.Activity.Text}'");
    }
}
```
## <a name="start-your-bot"></a>ボットの起動

- `default.html` ページはブラウザーに表示されます。
- ページの localhost ポート番号をメモしてください。 この情報はボットと対話するために必要です。

### <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続

この時点では、ボットはローカルで実行されています。
次に、エミュレーターを起動し、エミュレーターのボットに接続します。

1. エミュレーターの [ようこそ] タブにある **[新しいボット構成を作成する]** リンクをクリックします。 

2. **ボット名**を入力してから、ボット コードへのディレクトリ パスを入力します。 ボットの構成ファイルはこのパスに保存されます。

3. **[エンドポイント URL]** フィールドに `http://localhost:port-number/api/messages` と入力します。*port-number* は、アプリケーションを実行しているブラウザーに示されているポート番号と同じにします。

4. **[接続]** をクリックしてボットに接続します。 **[Microsoft アプリ ID]** と **[Microsoft アプリ パスワード]** を指定する必要はありません。 現段階では、これらのフィールドは空白のままにしておくことができます。 この情報は、後ほど、ボットを登録するときに取得します。

## <a name="interact-with-your-bot"></a>ボットの操作

ボットに「こんにちは」を送信すると、ボットはメッセージに対して「ターン 1: こんにちはを送信しました」と応答します。

## <a name="next-steps"></a>次の手順

次に、[ボットを Azure に配置](../bot-builder-howto-deploy-azure.md)するか、またはボットとそのしくみを説明する概念をご覧ください。

> [!div class="nextstepaction"]
> [ボットの基本的な概念](../v4sdk/bot-builder-basics.md)
