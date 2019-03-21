---
title: メッセージをインターセプトする | Microsoft Docs
description: Bot Framework SDK for .NET を使用して、ユーザーとボット間のメッセージをインターセプトする方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/01/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0e8873d2914d42b928004c31c14c8d60cb35b2a3
ms.sourcegitcommit: 4139ef7ebd8bb0648b8af2406f348b147817d4c7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/19/2019
ms.locfileid: "58073768"
---
# <a name="intercept-messages"></a>メッセージをインターセプトする

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-middleware.md)
> - [Node.js](../nodejs/bot-builder-nodejs-intercept-messages.md)

[!INCLUDE [Introducton to message logging](../includes/snippet-message-logging-intro.md)]

## <a name="intercept-and-log-messages"></a>メッセージのインターセプトとログ

次のサンプル コードは、Bot Framework SDK for .NET の**ミドルウェア**の概念を使用して、ユーザーとボット間で交換されるメッセージをインターセプトする方法を示します。 

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

Bot Framework SDK for .NET を使用してメッセージをインターセプトしてログに記録する方法を示す完全なサンプルについては、GitHub で<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-Middleware" target="_blank">ミドルウェアのサンプル</a>を参照してください。 

## <a name="additional-resources"></a>その他のリソース

- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Framework SDK for .NET リファレンス</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-Middleware" target="_blank">ミドルウェアのサンプル (GitHub)</a>
