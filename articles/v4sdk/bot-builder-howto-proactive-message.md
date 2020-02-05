---
title: ユーザーへのプロアクティブな通知の送信 - Bot Service
description: 通知メッセージを送信する方法について説明します
keywords: プロアクティブ メッセージ, 通知メッセージ, ボットの通知,
author: jonathanfingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/24/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 77222da10d69e6ad9a029a3548da66bd1a1806a2
ms.sourcegitcommit: 36d6f06ffafad891f6efe4ff7ba921de8a306a94
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/31/2020
ms.locfileid: "76895673"
---
# <a name="send-proactive-notifications-to-users"></a>ユーザーへのプロアクティブな通知の送信

[!INCLUDE[applies-to](../includes/applies-to.md)]

通常、ボットがユーザーに送信する各メッセージは、ユーザーの前の入力に直接関連しています。
場合によっては、ボットは、会話の現在のトピックまたはユーザーによって最後に送信されたメッセージに直接関連していないメッセージをユーザーに送信する必要があります。 この種のメッセージは、_プロアクティブ メッセージ_と呼ばれます。

プロアクティブ メッセージは、さまざまなシナリオで役立ちます。 たとえば、ユーザーが製品の価格を監視するようにボットに依頼していた場合、ボットは、製品の価格が 20% 下がった場合にユーザーにアラートを発行できます。 ユーザーの質問に対する回答をまとめるのに時間が必要な場合、ボットは、時間がかかることをユーザーに通知して、会話を続けられるようにします。 質問の回答がまとまったら、ボットは、その情報をユーザーと共有します。

お使いのボットでプロアクティブ メッセージを実装するときは、短い時間で複数のプロアクティブ メッセージを送信しないでください。 一部のチャネルでは、ボットがユーザーにメッセージを送信できる頻度に制限が設定され、これらの制限に違反した場合は、ボットが無効になります。

アドホック型のプロアクティブ メッセージは、最も単純な種類のプロアクティブ メッセージです。 ボットは会話が開始されると単純に会話にメッセージを差し挟みます。つまり、ユーザーが現在、ボットと別の話題で会話中であり、話題を変えるつもりがない場合でも考慮しません。

通知をより円滑に処理するには、会話フローに通知を統合するための他の方法を検討してください (会話の状態にフラグを設定する方法や、通知をキューに追加する方法など)。

## <a name="prerequisites"></a>前提条件

- [ボットの基本](bot-builder-basics.md)を理解する。
- プロアクティブ メッセージ サンプルのコピー ([**C#** ](https://aka.ms/proactive-sample-cs)、[**JavaScript**](https://aka.ms/proactive-sample-js)、または [**Python**](https://aka.ms/bot-proactive-python-sample-code))。 このサンプルは、この記事でプロアクティブ メッセージングを説明するために使用します。

## <a name="about-the-proactive-sample"></a>プロアクティブ サンプルについて

次の図に示すように、このサンプルには、プロアクティブ メッセージをボットに送信するときに使用されるボットと追加コントローラーが含まれています。

![プロアクティブ ボット](media/proactive-sample-bot.png)

## <a name="retrieve-and-store-conversation-reference"></a>会話の参照を取得して格納する

エミュレーターがボットに接続するとき、ボットは 2 つの会話更新アクティビティを受け取ります。 次に示すように、ボットの会話更新アクティビティ ハンドラーで、会話の参照が取得され、ディクショナリに格納されます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\ProactiveBot.cs**

[!code-csharp[OnConversationUpdateActivityAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/16.proactive-messages/Bots/ProactiveBot.cs?range=26-37&highlight=3-4,9)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/proactiveBot.js**

[!code-javascript[onConversationUpdateActivity](~/../botbuilder-samples/samples/javascript_nodejs/16.proactive-messages/bots/proactiveBot.js?range=13-17&highlight=2)]

[!code-javascript[onConversationUpdateActivity](~/../botbuilder-samples/samples/javascript_nodejs/16.proactive-messages/bots/proactiveBot.js?range=41-44&highlight=2-3)]

# <a name="pythontabpython"></a>[Python](#tab/python)

**bots/proactive_bot.py** [!code-python[on_conversation_update_activity](~/../botbuilder-python/samples/python/16.proactive-messages/bots/proactive_bot.py?range=14-16&highlight=2)]

[!code-python[on_conversation_update_activity](~/../botbuilder-python/samples/python/16.proactive-messages/bots/proactive_bot.py?range=35-45)]

---

注:実際のシナリオでは、会話の参照はデータベースに保持されます。メモリ内のオブジェクトは使用されません。

会話の参照には、アクティビティが存在する会話を表す _conversation_ プロパティが含まれます。 会話には、会話の参加ユーザーを一覧表示する _user_ プロパティと、現在のアクティビティに対する返信が送信される可能性のある URL を表すために、チャネルによって使用される _service URL_ プロパティが含まれます。 プロアクティブ メッセージをユーザーに送信するには、有効な会話の参照が必要です。

## <a name="send-proactive-message"></a>プロアクティブ メッセージを送信する

2 番目のコント ローラーである "_通知_" コントローラーは、プロアクティブ メッセージをボットに送信する役割を果たします。 プロアクティブ メッセージを生成するには、次の手順に従います。

1. プロアクティブ メッセージを送信する先の会話の参照を取得します。
1. アダプターの _continue conversation_ メソッドを呼び出して、使用するターン ハンドラー デリゲートと会話の参照を指定します。 会話続行メソッドによって、参照されている会話に対してターン コンテキストが生成され、指定したターン ハンドラー デリゲートが呼び出されます。
1. デリゲートで、ターン コンテキストを使用して、プロアクティブ メッセージを送信します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Controllers\NotifyController .cs**

ボットの通知ページが要求されるたびに、通知コントローラーは、ディクショナリから会話の参照を取得します。
次に、コントローラーは `ContinueConversationAsync` メソッドと `BotCallback` メソッドを使用して、プロアクティブ メッセージを送信します。

[!code-csharp[Notify logic](~/../botbuilder-samples/samples/csharp_dotnetcore/16.proactive-messages/Controllers/NotifyController.cs?range=17-62&highlight=28,40-44)]

プロアクティブ メッセージを送信するには、アダプターにボット用のアプリ ID が必要です。 ボットのアプリ ID は、運用環境で使用できます。 ローカル テスト環境では、任意の GUID を使用できます。 ボットにアプリ ID 割り当てられていない場合は、通知コントローラーによってプレースホルダー ID が自己生成され、これが呼び出しに使用されます。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

サーバーの `/api/notify` ページが要求されるたびに、サーバーは、ディクショナリから会話の参照を取得します。
次に、サーバーは `continueConversation` メソッドを使用して、プロアクティブ メッセージを送信します。
`continueConversation` のパラメーターは、このターンのボットのターン ハンドラーとして機能する関数です。

[!code-javascript[Notify logic](~/../botbuilder-samples/samples/javascript_nodejs/16.proactive-messages/index.js?range=68-82&highlight=4-8)]

# <a name="pythontabpython"></a>[Python](#tab/python)

ボットの通知ページが要求されるたびに、サーバーは、ディクショナリから会話の参照を取得します。
次に、サーバーは `_send_proactive_message` を使用して、プロアクティブ メッセージを送信します。

[!code-python[Notify logic](~/../botbuilder-python/samples/python/16.proactive-messages/app.py?range=97-105&highlight=5-9)]

---

## <a name="test-your-bot"></a>ボットをテストする

1. [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) をインストールします (まだインストールしていない場合)。
1. ご自身のマシンを使ってローカルでサンプルを実行します。
1. エミュレーターを起動し、お使いのボットに接続します。
1. お使いのボットの API/通知ページに読み込みます。 これによりエミュレーターでプロアクティブ メッセージが生成されます。

## <a name="additional-information"></a>関連情報

この記事のサンプル以外の追加サンプルについては、[GitHub](https://github.com/Microsoft/BotBuilder-Samples/) の C# および JS で入手できます。

### <a name="avoiding-401-unauthorized-errors"></a>401 "未承認" エラーの防止

BotAuthentication によって受信要求が認証されている場合、既定では、BotBuilder SDK は、信頼されたホスト名の一覧に `serviceUrl` を追加します。 これらはメモリ内キャッシュに保持されています。 お使いのボットを再起動すると、再起動したボットにもう一度情報を伝達しない限り、プロアクティブ メッセージを待っているユーザーがそのメッセージを受信できません。

これを回避するには、次を使用して、信頼されたホスト名の一覧に `serviceUrl` を手動で追加する必要があります。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
MicrosoftAppCredentials.TrustServiceUrl(serviceUrl);
```

プロアクティブなメッセージングについては、`serviceUrl` はプロアクティブ メッセージの受信者が使用しているチャネルの URL で、`Activity.ServiceUrl` にあります。

上記のコードを、プロアクティブ メッセージを送信するコードの直前に追加します。 [プロアクティブ メッセージ サンプル](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/16.proactive-messages)では、`NotifyController.cs` 内で `await turnContext.SendActivityAsync("proactive hello");` の直前に配置します。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```js
MicrosoftAppCredentials.trustServiceUrl(serviceUrl);
```

プロアクティブなメッセージングについては、`serviceUrl` はプロアクティブ メッセージの受信者が使用しているチャネルの URL で、`activity.serviceUrl` にあります。

上記のコードを、プロアクティブ メッセージを送信するコードの直前に追加します。 [プロアクティブ メッセージ サンプル](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/16.proactive-messages)では、`index.js` 内で `await turnContext.sendActivity('proactive hello');` の直前に配置します。

# <a name="pythontabpython"></a>[Python](#tab/python)

```python
MicrosoftAppCredentials.trustServiceUrl(serviceUrl)
```

プロアクティブなメッセージングについては、`serviceUrl` はプロアクティブ メッセージの受信者が使用しているチャネルの URL で、`activity.serviceUrl` にあります。

上記のコードを、プロアクティブ メッセージを送信するコードの直前に追加します。 [プロアクティブ メッセージ サンプル](https://aka.ms/bot-proactive-python-sample-code)では、*proactive hello* メッセージを送信する前に、`app.py` にそれを追加します。

---

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [連続して行われる会話フローの実装](bot-builder-dialog-manage-conversation-flow.md)
