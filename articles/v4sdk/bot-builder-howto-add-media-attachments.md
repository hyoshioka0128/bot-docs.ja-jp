---
title: メッセージにメディアを追加する | Microsoft Docs
description: Bot Framework SDK を使用してメッセージにメディアを追加する方法について説明します。
keywords: メディア, メッセージ, イメージ, オーディオ, ビデオ, ファイル, MessageFactory, リッチ カード, メッセージ, アダプティブ カード, ヒーロー カード, 推奨されるアクション
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9478a3861b24746b4081ab2176486e59ccc7d4bc
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464712"
---
# <a name="add-media-to-messages"></a>メッセージにメディアを追加する

[!INCLUDE[applies-to](../includes/applies-to.md)]

ユーザーとボットの間のメッセージ交換には、イメージ、ビデオ、オーディオ、ファイルなどのメディア添付ファイルを含めることができます。 Bot Framework SDK では、ユーザーにリッチ メッセージを送信するタスクがサポートされています。 チャネル (Facebook、Skype、Slack など) がサポートするリッチ メッセージの種類を確認するには、チャネルのドキュメントで制限事項に関する情報を参照してください。

使用可能なカードの例については、[ユーザー エクスペリエンスの設計](../bot-service-design-user-experience.md)に関する記事をご覧ください。

## <a name="send-attachments"></a>添付ファイルを送信する

イメージやビデオなどのユーザー コンテンツを送信するには、添付ファイルまたは添付ファイルのリストをメッセージに追加します。

使用可能なカードの例については、[ユーザー エクスペリエンスの設計](../bot-service-design-user-experience.md)に関する記事をご覧ください。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Activity` オブジェクトの `Attachments` プロパティには、メッセージに添付するメディア添付ファイルやリッチ カードを表す `Attachment` オブジェクトが格納されます。 メディア添付ファイルをメッセージに追加するには、(`CreateReply()` を使用してアクティビティ外で作成された) `reply` アクティビティ用の `Attachment` オブジェクトを作成し、`ContentType`、`ContentUrl`、`Name` の各プロパティを設定します。

ここで示すソース コードは、[添付ファイルの処理](https://aka.ms/bot-attachments-sample-code)のサンプルに基づいています。

返信メッセージを作成するには、テキストを定義し、添付ファイルを設定します。 添付ファイルを返信に割り当てる場合、添付ファイルの種類ごとに同じ操作を行いますが、次のスニペットに示すように、添付ファイルの設定と定義はそれぞれ異なります。 次のコードでは、インラインの添付ファイル用の返信が設定されます。

**Bots/AttachmentsBot.cs**  
[!code-csharp[inline attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=105-106)]

次に、添付ファイルの種類を見ていきます。 最初はインラインの添付ファイルです。

**Bots/AttachmentsBot.cs**  
[!code-csharp[inline attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=167-178)]

次は、アップロードされた添付ファイルです。

**Bots/AttachmentsBot.cs**  
[!code-csharp[uploaded attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=181-214)]

最後は、インターネットの添付ファイルです。

**Bots/AttachmentsBot.cs**  
[!code-csharp[online attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=217-226)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ここで示すソース コードは、[JS の添付ファイルの処理](https://aka.ms/bot-attachments-sample-code-js)のサンプルに基づいています。

添付ファイルを使用するには、お使いのボットに次のライブラリを含めます。

**bots/attachmentsBot.js**  
[!code-javascript[attachments libraries](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=4)]

返信メッセージを作成するには、テキストを定義し、添付ファイルを設定します。 添付ファイルを返信に割り当てる場合、添付ファイルの種類ごとに同じ操作を行いますが、次のスニペットに示すように、添付ファイルの設定と定義はそれぞれ異なります。 次のコードでは、インラインの添付ファイル用の返信が設定されます。

**bots/attachmentsBot.js**  
[!code-javascript[attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=119,128-129)]

イメージやビデオのような単一のコンテンツを送信する場合、メディアを送信する方法は複数あります。 まず、インラインの添付ファイルとして送信できます。

**bots/attachmentsBot.js**  
[!code-javascript[inline attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=170-179)]

次は、アップロードされた添付ファイルです。

**bots/attachmentsBot.js**  
[!code-javascript[uploaded attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=197-215)]

最後は、URL に含まれるインターネットの添付ファイルです。

**bots/attachmentsBot.js**  
[!code-javascript[internet attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=184-191)]

---

添付ファイルがイメージ、オーディオ、またはビデオの場合、コネクタ サービスにより、[チャネル](bot-builder-channeldata.md)での会話内の添付ファイルがレンダリングできる方法で、添付ファイルのデータがチャネルに通信されます。 添付ファイルがファイルである場合は、ファイルの URL が会話内にハイパーリンクとしてレンダリングされます。

## <a name="send-a-hero-card"></a>ヒーロー カードを送信する

シンプルなイメージ添付ファイルまたはビデオ添付ファイルだけでなく、**ヒーロー カード**を添付することもできます。これにより、1 つのオブジェクトに含まれるイメージとボタンを結合してユーザーに送信することができます。 Markdown はほとんどのテキスト フィールドでサポートされていますが、サポート状況はチャネルごとに異なる場合があります。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

ヒーロー カードとボタンを使用してメッセージを作成するには、`HeroCard` をメッセージに添付します。 

ここで示すソース コードは、[添付ファイルの処理](https://aka.ms/bot-attachments-sample-code)のサンプルに基づいています。

**Bots/AttachmentsBot.cs**  
[!code-csharp[Hero card](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=39-58)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ヒーロー カードとボタンを使用してメッセージを作成するには、`HeroCard` をメッセージに添付します。 

ここで示すソース コードは、[JS の添付ファイルの処理](https://aka.ms/bot-attachments-sample-code-js)のサンプルに基づいています。

**bots/attachmentsBot.js**  
[!code-javascript[hero card](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=148-164)]

---

## <a name="process-events-within-rich-cards"></a>リッチ カード内のイベントを処理する

リッチ カード内のイベントを処理するには、_カード アクション_ オブジェクトを使用して、ユーザーがボタンをクリックするか、またはカードのセクションをタップしたときのアクションを指定します。 各カード アクションには _type_ と _value_ があります。

正常に機能させるために、カード上のクリック可能な各アイテムにアクションの種類を割り当てます。 この表では、使用できるアクションの種類と、関連付けられている value プロパティに含める内容を一覧にまとめ、説明しています。

| Type | 説明 | 値 |
| :---- | :---- | :---- |
| openUrl | 組み込みのブラウザーで URL を開きます。 | 開く URL。 |
| imBack | ボットにメッセージを送信し、目に見える応答をチャットに投稿します。 | 送信するメッセージのテキスト。 |
| postBack | ボットにメッセージを送信します。目に見える応答はチャットに投稿できません。 | 送信するメッセージのテキスト。 |
| 以下を呼び出します。 | 電話を発信します。 | 電話の発信先 (形式は `tel:123123123123`)。 |
| playAudio | オーディオを再生します。 | 再生するオーディオの URL。 |
| playVideo | ビデオを視聴します。 | 再生するビデオの URL。 |
| showImage | 画像を表示します。 | 表示する画像の URL。 |
| downloadFile | ファイルをダウンロードします。 | ダウンロードするファイルの URL。 |
| signin | OAuth サインイン プロセスを開始します。 | 開始する OAuth フローの URL。 |

## <a name="hero-card-using-various-event-types"></a>さまざまな種類のイベントを使用するヒーロー カード

次のコードでは、さまざまなリッチ カード イベントを使用する例を示します。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

使用可能なすべてのカードの例については、[C# カードのサンプル](https://aka.ms/bot-cards-sample-code)をご覧ください。

**Cards.cs**  
[!code-csharp[hero cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Cards.cs?range=27-40)]

**Cards.cs**  
[!code-csharp[cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Cards.cs?range=91-100)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

使用可能なすべてのカードの例については、[JS カードのサンプル](https://aka.ms/bot-cards-js-sample-code)をご覧ください。

**dialogs/mainDialog.js**  
[!code-javascript[hero cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=213-225)]

**dialogs/mainDialog.js**  
[!code-javascript[sign in cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=266-272)]

---

## <a name="send-an-adaptive-card"></a>アダプティブ カードを送信する
ユーザーとの通信に、テキスト、画像、動画、オーディオ、ファイルなど、多彩なメッセージを送信するために、アダプティブ カードと MessageFactory が使用されます。 ただし、これには違いがいくつかあります。 

まず、アダプティブ カードをサポートするのは一部のチャネルのみで、サポートしていないチャネルでは、アダプティブ カードが部分的にしかサポートされない可能性があります。 たとえば、Facebook でアダプティブ カードを送信する場合、テキストと画像は適切に機能しますが、ボタンは機能しません。 MessageFactory は、作成手順を自動化するための、Bot Framework SDK 内の単なるヘルパー クラスであり、ほとんどのチャネルでサポートされています。 

また、アダプティブ カードではカード形式でメッセージが配信され、チャネルによって、カードのレイアウトが決まります。 MessageFactory によって配信されるメッセージの形式はチャネルによって異なり、アダプティブ カードが添付ファイルに含まれていない限り、必ずしもカード形式であるとは限りません。 

チャネルでのアダプティブ カードのサポートに関する最新情報については、<a href="http://adaptivecards.io/designer/">アダプティブ カード デザイナー</a>に関するページをご覧ください。

アダプティブ カードを使用するには、必ず `AdaptiveCards` NuGet パッケージを追加してください。 


> [!NOTE]
> ご自分のボットで使用されるチャネルにおいてこの機能をテストして、それらのチャネルでアダプティブ カードがサポートされているかどうかを判断する必要があります。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

アダプティブ カードを使用するには、必ず `AdaptiveCards` NuGet パッケージを追加してください。

ここで示すソース コードは、[カード使用](https://aka.ms/bot-cards-sample-code)のサンプルに基づいています。

**Cards.cs**  
[!code-csharp[adaptive cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Cards.cs?range=13-25)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

アダプティブ カードを使用するには、必ず `adaptivecards` npm パッケージを追加してください。

ここで示すソース コードは、[JS のカード使用](https://aka.ms/bot-cards-js-sample-code)のサンプルに基づいています。 

ここでは、アダプティブ カードは独自のファイルに格納され、ボットに含まれています。

**resources/adaptiveCard.json**  
[!code-json[adaptive cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/resources/adaptiveCard.json)]

次に、CardFactory で作成されます。

**dialogs/mainDialog.js**  
[!code-javascript[adaptive cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=177-179)]

---

## <a name="send-a-carousel-of-cards"></a>カードのカルーセルを送信する

また、メッセージには複数の添付ファイルをカルーセル レイアウトで含めることもできます。このレイアウトでは、添付ファイルが左右に並べて配置され、ユーザーは全体をスクロールすることができます。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

ここで示すソース コードは、[カード サンプル](https://aka.ms/bot-cards-sample-code)に基づいています。

最初に、返信を作成し、添付ファイルをリストとして定義します。

**Dialogs/MainDialog.cs**  
[!code-csharp[carousel of cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Dialogs/MainDialog.cs?range=61-66)]

次に、添付ファイルを追加します。 ここでは 1 つずつ追加しますが、必要に応じて、リストを使ってカードを追加することもできます。

**Dialogs/MainDialog.cs**  
[!code-csharp[carousel of cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Dialogs/MainDialog.cs?range=104-113)]

添付ファイルが追加されると、他と同じように返信を送信できます。

**Dialogs/MainDialog.cs**  
[!code-csharp[carousel of cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Dialogs/MainDialog.cs?range=117-118)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ここで示すソース コードは、[JS カード サンプル](https://aka.ms/bot-cards-js-sample-code)に基づいています。

カードのカルーセルを送信するには、添付ファイルを配列として、またレイアウトの種類を `Carousel` として定義して、返信します。

**dialogs/mainDialog.js**  
[!code-javascript[carousel of cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=104-116)]

---

<!-- TODO: Add a media card, such as video or audion. Revisit which examples we put here and link to the 06 through 08 samples. -->

## <a name="additional-resources"></a>その他のリソース

使用可能なカードの例については、[ユーザー エクスペリエンスの設計](../bot-service-design-user-experience.md)に関する記事をご覧ください。

スキーマの詳細については、「[Bot Framework card schema (Bot Framework のカード スキーマ)](https://aka.ms/botSpecs-cardSchema)」と Bot Framework のアクティビティ スキーマに関するページの「[Message activity (メッセージ アクティビティ)](https://aka.ms/botSpecs-activitySchema#message-activity)」セクションをご覧ください。

| サンプル コード | C# | JS |
| :------ | :----- | :---|
| カード | [C# のサンプル](https://aka.ms/bot-cards-sample-code) | [JS のサンプル](https://aka.ms/bot-cards-js-sample-code) |
| [添付ファイル] | [C# のサンプル](https://aka.ms/bot-attachments-sample-code) | [JS のサンプル](https://aka.ms/bot-attachments-sample-code-js) |
| 推奨されるアクション | [C# のサンプル](https://aka.ms/SuggestedActionsCSharp) | [JS のサンプル](https://aka.ms/SuggestedActionsJS) |

その他のサンプルについては、[GitHub](https://aka.ms/bot-samples-readme) の Bot Builder サンプル リポジトリを参照してください。

### <a name="code-sample-for-processing-adaptive-card-input"></a>アダプティブ カード入力を処理するためのコード サンプル

このサンプル コードは、ボット ダイアログ クラス内でアダプティブ カード入力を使用する 1 つの方法を示しています。
この方法では、テキスト フィールドで受け取った、応答するクライアントからの入力を検証することで、現在のサンプル 06.using-cards が拡張されます。
まず、resources フォルダーにある adaptiveCard.json の最後の括弧の直前に次のコードを追加することにより、既存のアダプティブ カードにテキスト入力およびボタン機能を追加しました。

```json
  ,
  "actions": [
    {
      "type": "Action.ShowCard",
      "title": "Text",
      "card": {
      "type": "AdaptiveCard",
      "body": [
        {
          "type": "Input.Text",
          "id": "text",
          "isMultiline": true,
          "placeholder": "Enter your comment"
        }
      ],
      "actions": [
        {
          "type": "Action.Submit",
          "title": "OK"
        }
      ]
    }
  }
]

```

入力フィールドには "text" というラベルが付いているので、アダプティブ カードではコメント テキスト データが Value.[text] として添付されます。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
検証コントロールでは、Newtonsoft.json を使用して最初にこれを JObject に変換し、次に比較のためにトリミングされたテキスト文字列を作成します。 そのため、次のコードを
  ```csharp
  using Newtonsoft.Json.Linq;
  ```
MainDialog.cs に追加して、Newtonsoft.Json の最新の安定した nuget パッケージをインストールします。
検証コントロールのコードでは、コード コメントにロジック フローを追加しました。 この ChoiceValidator () コードは、06.using-cards サンプルにある、MainDialog の宣言用の閉じ括弧の public の直後に配置されています。

```csharp
private async Task ChoiceValidator(
  PromptValidatorContext promptContext,
  CancellationToken cancellationToken)
  {
    // Retrieves Adaptive Card comment text as JObject.
    // looks for JObject field "text" and converts that input into a trimmed text string.
    var jobject = promptContext.Context.Activity.Value as JObject;
    var jtoken = jobject?["text"];
    var text = jtoken?.Value().Trim();
    // Logic: 1. if succeeded = true, just return promptContext
    //        2. if false, see if JObject contained Adaptive Card input.
    //               No = (bad input) return promptContext
    //               Yes = update Value field with JObject text string, return "true".
    if (!promptContext.Recognized.Succeeded && text != null)
    {
       var choice = promptContext.Options.Choices.FirstOrDefault(
       c => c.Value.Equals(text, StringComparison.InvariantCultureIgnoreCase));
       if (choice != null)
       {
           promptContext.Recognized.Value = new FoundChoice
            {
               Value = choice.Value,
             };
            return true;
       }
    }
    return promptContext.Recognized.Succeeded;
  }
```

今度は、上記の MainDialog 宣言内で、次の内容を、
  ```csharp
  // Define the main dialog and its related components.
  AddDialog(new ChoicePrompt(nameof(ChoicePrompt)));
  ```
を次のように変更します。
  ```csharp
  // Define the main dialog and its related components.
  AddDialog(new ChoicePrompt(nameof(ChoicePrompt), ChoiceValidator));
  ```
これにより、新しい ChoicePrompt が作成されるたびに、検証コントロールが呼び出されてアダプティブ カード入力が検索されます。

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
mainDialog.js を開き、実行メソッド _async run(turnContext, accessor)_ を見つけます。このメソッドでは着信アクティビティが処理されます。
呼び出し _dialogSet.add(this);_ のすぐ後に、次のコードを追加します。
```JavaScript
  // The following check looks for a non-existant text input
  // plus Adaptive Card input in _activity.value.text
  // If both conditions exist, the Activity Card text 
  // is copied into the text input field.
  if(turnContext._activity.text == null
      && turnContext._activity.value.text != null)
   {
      this.logger.log('replacing null text with Activity Card text input');
      turnContext._activity.text = turnContext._activity.value.text;
   }
```
このチェックで、クライアントからの存在しないテキストの入力が見つかった場合、アダプティブ カードからの入力があるかどうかが調べられます。
アダプティブ カードの入力が \_activity.value.text に存在する場合は、これが通常のテキスト入力フィールドにコピーされます。

---

ご自身のコードをテストするには、アダプティブ カードが表示されたら、[テキスト] ボタンをクリックし、「ヒーロー カード」などの有効な選択肢を入力して [OK] ボタンをクリックします。

![アダプティブ カードのテスト](media/adaptive-card-input.png)

1. 最初の入力は、新しいダイアログを開始するために使用されます。
2. もう一度 [OK] ボタンをクリックすると、新しいカードを選択する場合にこの入力が使用されるようになります。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [ユーザー アクションをガイドするボタンを追加する](./bot-builder-howto-add-suggested-actions.md)
