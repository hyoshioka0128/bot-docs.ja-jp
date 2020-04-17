---
title: テキスト メッセージを送受信する - Bot Service
description: Bot Framework SDK 内でテキスト メッセージを送受信する方法について説明します。
keywords: メッセージの送信, メッセージ アクティビティ, 単純なテキスト メッセージ, メッセージ, テキスト メッセージ, メッセージの受信
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9dc5bfeab8bc56e81888be5e9463be167fcd2b18
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "77071810"
---
# <a name="send-and-receive-text-message"></a>テキスト メッセージを送受信する

[!INCLUDE[applies-to](../includes/applies-to.md)]

ボットでユーザーと通信し、同様にボットで通信を受け取る主要な方法は、**メッセージ** アクティビティを利用することです。 メッセージにはプレーンテキストで構成されている単純なものもあれば、カードや添付など、よりリッチなコンテンツが含まれるものもあります。 ボットのターン ハンドラーでは、ユーザーからメッセージを受け取ります。そこからユーザーに応答を送信できます。 ターン コンテキスト オブジェクトによって、ユーザーにメッセージを戻すためのメソッドが与えられます。 この記事では、単純なテキスト メッセージを送信する方法について説明します。

Markdown はほとんどのテキスト フィールドでサポートされていますが、サポート状況はチャネルごとに異なる場合があります。

メッセージを送受信する実行中のボットについては、目次の一番上にあるクイックスタートを実行するか、[ボットの動作方法に関するの記事](bot-builder-basics.md#bot-structure)を確認してください。これは、ご自身で実行する場合にご利用いただけるシンプルなサンプルへもリンクされています。

## <a name="send-a-text-message"></a>テキスト メッセージの送信

単純なテキスト メッセージを送信するには送信する文字列をアクティビティとして指定します。

# <a name="c"></a>[C#](#tab/csharp)

ボットのアクティビティ ハンドラーで、ターン コンテキスト オブジェクトの `SendActivityAsync` メソッドを使用し、メッセージ応答を 1 つ送信します。 オブジェクトの `SendActivitiesAsync` メソッドを使用して一度に複数の応答を送信することもできます。

```cs
await turnContext.SendActivityAsync($"Welcome!");
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

ボットのアクティビティ ハンドラーで、ターン コンテキスト オブジェクトの `sendActivity` メソッドを使用し、メッセージ応答を 1 つ送信します。 オブジェクトの `sendActivities` メソッドを使用して一度に複数の応答を送信することもできます。

```javascript
await context.sendActivity("Welcome!");
```

# <a name="python"></a>[Python](#tab/python)

ボットのアクティビティ ハンドラーで、ターン コンテキスト オブジェクトの `send_activity` メソッドを使用し、メッセージ応答を 1 つ送信します。

```python
await turn_context.send_activity("Welcome!")
```

---
## <a name="receive-a-text-message"></a>テキスト メッセージの受信

単純なテキスト メッセージを受信するには、"*アクティビティ*" オブジェクトの "*テキスト*" プロパティを使用します。 

# <a name="c"></a>[C#](#tab/csharp)

ボットのアクティビティ ハンドラーで、次のコードを使用してメッセージを受信します。 

```cs
var responseMessage = turnContext.Activity.Text;
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

ボットのアクティビティ ハンドラーで、次のコードを使用してメッセージを受信します。

```javascript
let text = turnContext.activity.text;
```

# <a name="python"></a>[Python](#tab/python)

ボットのアクティビティ ハンドラーで、次のコードを使用してメッセージを受信します。

```python
response = context.activity.text
```

---

## <a name="send-a-typing-indicator"></a>入力インジケーターの送信
ユーザーはメッセージに対するタイムリーな応答を期待しています。 ボットがユーザーからの指示を受け取ったということをユーザーに表示することなく、サーバーの呼び出しやクエリの実行などの実行時間の長いタスクを実行すると、ユーザーはしびれを切らして追加のメッセージを送信したり、単にボットが壊れていると推測する可能性があります。

Web チャットおよび Direct Line チャネルのボットは、メッセージが受信され、処理中であることをユーザに示す入力表示の送信をサポートできます。 ボットは 15 秒以内にターンを終了させる必要があることに注意してください。そうしないと、Connector サービスがタイムアウトになります。 より長いプロセスについては、[プロアクティブ メッセージ](bot-builder-howto-proactive-message.md)の送信に関する詳細を参照してください。 

次の例は、入力表示を送信する方法を示しています。

# <a name="c"></a>[C#](#tab/csharp)

```csharp
protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    if (string.Equals(turnContext.Activity.Text, "wait", System.StringComparison.InvariantCultureIgnoreCase))
    {
        await turnContext.SendActivitiesAsync(
            new Activity[] {
                new Activity { Type = ActivityTypes.Typing },
                new Activity { Type = "delay", Value= 3000 },
                MessageFactory.Text("Finished typing", "Finished typing"),
            },
            cancellationToken);
    }
    else
    {
        var replyText = $"Echo: {turnContext.Activity.Text}. Say 'wait' to watch me type.";
        await turnContext.SendActivityAsync(MessageFactory.Text(replyText, replyText), cancellationToken);
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
this.onMessage(async (context, next) => {
    if (context.activity.text === 'wait') {
        await context.sendActivities([
            { type: ActivityTypes.Typing },
            { type: 'delay', value: 3000 },
            { type: ActivityTypes.Message, text: 'Finished typing' }
        ]);
    } else {
        await context.sendActivity(`You said '${ context.activity.text }'. Say "wait" to watch me type.`);
    }
    await next();
});
```

# <a name="python"></a>[Python](#tab/python)

```python
async def on_message_activity(self, turn_context: TurnContext):
    if turn_context.activity.text == "wait":
        return await turn_context.send_activities([
            Activity(
                type=ActivityTypes.typing
            ),
            Activity(
                type="delay",
                value=3000
            ),
            Activity(
                type=ActivityTypes.message,
                text="Finished Typing"
            )
        ])
    else:
        return await turn_context.send_activity(
            f"You said {turn_context.activity.text}.  Say 'wait' to watch me type."
        )
```

---

## <a name="additional-resources"></a>その他のリソース

- 一般的なアクティビティ処理の詳細については、[アクティビティの処理](~/v4sdk/bot-builder-basics.md#the-activity-processing-stack)に関するページをご覧ください。
- 書式設定の詳細については、Bot Framework のアクティビティ スキーマに関するページの[メッセージ アクティビティ](https://aka.ms/botSpecs-activitySchema#message-activity)に関するセクションをご覧ください。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [メッセージにメディアを追加する](./bot-builder-howto-add-media-attachments.md)
