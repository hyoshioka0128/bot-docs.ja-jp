---
title: メッセージのインターセプト | Microsoft Docs
description: Bot Builder SDK for .NET を使用して、ユーザーとボット間のメッセージをインターセプトする方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 21607dae5c2a8d08ed4b7bf1b6e6983cd9bf1196
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999729"
---
# <a name="intercept-messages"></a>メッセージをインターセプトする

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-middleware.md)
> - [Node.js](../nodejs/bot-builder-nodejs-intercept-messages.md)

[!INCLUDE [Introducton to message logging](../includes/snippet-message-logging-intro.md)]

## <a name="intercept-and-log-messages"></a>メッセージのインターセプトとログ

次のサンプル コードは、Bot Builder SDK for .NET の**ミドルウェア**の概念を使用して、ユーザーとボット間で交換されるメッセージをインターセプトする方法を示します。 

最初に `DebugActivityLogger` クラスを作成し、`LogAsync` メソッドを定義して、インターセプトされたそれぞれのメッセージに対して実行する操作を指定します。 この例では、各メッセージに関するいくらかの情報を出力します。

```cs
public class DebugActivityLogger : IActivityLogger
{
    public async Task LogAsync(IActivity activity)
    {
        Debug.WriteLine($"From:{activity.From.Id} - To:{activity.Recipient.Id} - Message:{activity.AsMessageActivity()?.Text}");
    }
}
```

`Global.asax.cs` に次のコードを追加します。  ユーザーとボット間で交換 (双方向) されるすべてのメッセージによって、`DebugActivityLogger` クラスの `LogAsync` メソッドがトリガーされるようになりました。 

```cs
    public class WebApiApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
            var builder = new ContainerBuilder();
            builder.RegisterType<DebugActivityLogger>().AsImplementedInterfaces().InstancePerDependency();
            builder.Update(Conversation.Container);

            GlobalConfiguration.Configure(WebApiConfig.Register);
        }
    }
```

この例では、単純に各メッセージに関する情報を出力しますが、`LogAsync` メソッドを更新して、各メッセージに対して実行する操作を指定することができます。 

## <a name="sample-code"></a>サンプル コード 

Bot Builder SDK for .NET を使用してメッセージをインターセプトしてログに記録する方法を示す完全なサンプルについては、GitHub で<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-Middleware" target="_blank">ミドルウェアのサンプル</a>を参照してください。 

## <a name="additional-resources"></a>その他のリソース

- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Builder SDK for .NET リファレンス</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-Middleware" target="_blank">ミドルウェアのサンプル (GitHub)</a>
