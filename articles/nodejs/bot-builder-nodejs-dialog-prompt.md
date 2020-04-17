---
title: ユーザーに入力を求めるプロンプト - Bot Service
description: Bot Framework SDK for Node.js でプロンプトを使用してユーザー入力を収集する方法について説明します。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 723f81d5ef76eb92df74d1946532fe475c617c59
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75791094"
---
# <a name="prompt-for-user-input"></a>ユーザーに入力を求めるプロンプト

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Bot Framework SDK for Node.js には、ユーザーからの入力の収集を単純にする、組み込みプロンプトのセットがあります。 

"*プロンプト*" は、ボットがユーザーからの入力を必要とするときに常に使用されます。 ウォーターフォールの中でプロンプトを連鎖させて使用することで、一連の入力をユーザーに求めることができます。 プロンプトと[ウォーターフォール](bot-builder-nodejs-dialog-waterfall.md)を併用することで、ボットの[会話フローを管理](bot-builder-nodejs-manage-conversation-flow.md)できます。 

この記事では、プロンプトのしくみと、それらを使用してユーザーから情報を収集する方法を理解するのに役立ちます。

## <a name="prompts-and-responses"></a>プロンプトと応答

ユーザーからの入力が必要な場合は、常にプロンプトを送信し、ユーザーが入力で応答するまで待機した後、入力を処理して応答をユーザーに送信できます。

次のコード サンプルは、ユーザーに名前を入力を求め、あいさつメッセージで応答します。

```javascript
bot.dialog('greetings', [
    // Step 1
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    // Step 2
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
```

この基本的なコンストラクトを使用して、ボットが必要とする数のプロンプトと応答を追加することで、会話フローをモデル化できます。

## <a name="prompt-results"></a>プロンプトの結果 

組み込みプロンプトは、ユーザーの応答を `results.response` フィールドに返す[ダイアログ](bot-builder-nodejs-dialog-overview.md)として実装されます。 JSON オブジェクトでは、応答は、`results.response.entity` フィールドに返されます。 プロンプトの結果は、任意の種類の[ダイアログ ハンドラー](bot-builder-nodejs-dialog-overview.md#dialog-handlers)で受信できます。 ボットで応答を受信したときに、応答をそのまま使用するか、[`session.endDialogWithResult`][EndDialogWithResult] メソッドを呼び出して呼び出し元のダイアログに渡すことができます。

次のコード サンプルは、`session.endDialogWithResult` メソッドを使用して、呼び出し元のダイアログにプロンプトの結果を返す方法を示しています。 この例では、`askName` ダイアログが返す結果を使用して、`greetings` ダイアログがユーザーの名前を使用してあいさつを返します。

```javascript
// Ask the user for their name and greet them by name.
bot.dialog('greetings', [
    function (session) {
        session.beginDialog('askName');
    },
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
bot.dialog('askName', [
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

## <a name="prompt-types"></a>プロンプトの種類
Bot Framework SDK for Node.js には、さまざまな種類の組み込みプロンプトが含まれています。 

|**プロンプトの種類**     | **説明** |     
| ------------------ | --------------- |
|[Prompts.text](#promptstext) | テキスト文字列を入力することをユーザーに求めます。 |     
|[Prompts.confirm](#promptsconfirm) | アクションを確認することをユーザーに求めます。| 
|[Prompts.number](#promptsnumber) | 数値を入力することをユーザーに求めます。     |
|[Prompts.time](#promptstime) | 時間または日付/時刻を入力することをユーザーに求めます。      |
|[Prompts.choice](#promptschoice) | オプションの一覧から選択することをユーザーに求めます。    |
|[Prompts.attachment](#promptsattachment) | ピクチャまたはビデオをアップロードすることをユーザーに求めます。|       

次のセクションで、プロンプトの種類ごとに詳しく説明します。

### <a name="promptstext"></a>Prompts.text

**テキスト文字列**の入力をユーザーに求める場合は、[Prompts.text()][PromptsText] メソッドを使用します。 このプロンプトでは、ユーザーの応答を [IPromptTextResult][IPromptTextResult] として返します。

```javascript
builder.Prompts.text(session, "What is your name?");
```

### <a name="promptsconfirm"></a>Prompts.confirm

**はい/いいえ**応答でアクションを確認することをユーザーに求める場合は、[Prompts.confirm()][PromptsConfirm] メソッドを使用します。 このプロンプトは、ユーザーの応答を [IPromptConfirmResult](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptconfirmresult.html) として返します。

```javascript
builder.Prompts.confirm(session, "Are you sure you wish to cancel your order?");
```

### <a name="promptsnumber"></a>Prompts.number

**数値**の入力をユーザーに求める場合は、[Prompts.number()][PromptsNumber] メソッドを使用します。 このプロンプトは、ユーザーの応答を [IPromptNumberResult](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptnumberresult.html) として返します。

```javascript
builder.Prompts.number(session, "How many would you like to order?");
```

### <a name="promptstime"></a>Prompts.time

**時間**または**日付/時刻**の入力をユーザーに求める場合は、[Prompts.time()][PromptsTime] メソッドを使用します。 このプロンプトは、ユーザーの応答を [IPromptTimeResult](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html) として返します。 フレームワークは、[Chrono](https://github.com/wanasit/chrono) ライブラリを使用してユーザーの応答を解析し、相対的な応答 (例: "5 分後") と相対的でない応答 (例: "6 月 6 日午後 2 時") の両方をサポートします。

ユーザーの応答を表す [Results.response](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html#response) フィールドには、日付と時刻を指定する[entity](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ientity.html) オブジェクトが格納されます。 日付と時刻を JavaScript の`Date` オブジェクトに解決するには、[EntityRecognizer.resolveTime()](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#resolvetime) メソッドを使用します。

> [!TIP] 
> ユーザーが入力した時刻は、ボットがホストされているサーバーのタイム ゾーンに基づいて UTC 時間に変換されます。 サーバーは、ユーザーとは別のタイム ゾーンに配置されている場合があるため、必ずタイム ゾーンを考慮してください。 日付と時刻をユーザーのローカル時刻に変換するには、ユーザーが属しているタイム ゾーンを確認することを検討してください。

```javascript
bot.dialog('createAlarm', [
    function (session) {
        session.dialogData.alarm = {};
        builder.Prompts.text(session, "What would you like to name this alarm?");
    },
    function (session, results, next) {
        if (results.response) {
            session.dialogData.name = results.response;
            builder.Prompts.time(session, "What time would you like to set an alarm for?");
        } else {
            next();
        }
    },
    function (session, results) {
        if (results.response) {
            session.dialogData.time = builder.EntityRecognizer.resolveTime([results.response]);
        }

        // Return alarm to caller  
        if (session.dialogData.name && session.dialogData.time) {
            session.endDialogWithResult({ 
                response: { name: session.dialogData.name, time: session.dialogData.time } 
            }); 
        } else {
            session.endDialogWithResult({
                resumed: builder.ResumeReason.notCompleted
            });
        }
    }
]);
```

### <a name="promptschoice"></a>Prompts.choice

**オプションの一覧から選択する**ことをユーザーに求める場合は、[Prompts.choice()][PromptsChoice] メソッドを使用します。 ユーザーは、選択するオプションに関連付けられている番号を入力するか、選択するオプションの名前を入力することで、選択肢を伝えることができます。 オプションの名前の全部または一部の一致の両方がサポートされます。 このプロンプトは、ユーザーの応答を [IPromptChoiceResult](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptchoiceresult.html) として返します。 

ユーザーに表示される一覧のスタイルを指定するには、[IPromptOptions.listStyle](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptoptions.html#liststyle) プロパティを設定します。 次の表は、このプロパティの `ListStyle` 列挙値を示しています。


`ListStyle` 列挙値は次のとおりです。

| インデックス | 名前 | 説明 |
| ---- | ---- | ---- |
| 0 | なし | 一覧は表示されません。 これは、一覧がプロンプトの一部として含まれる場合に使用されます。 |
| 1 | インライン | 選択肢は、次の形式のインラインリストとして表示されます。"1. 赤、2. 緑、3. 青" |
| 2 | list | 選択肢は、番号付きリストとして表示されます。 |
| 3 | ボタン | 選択肢は、ボタンをサポートするチャネルでは、ボタンとして表示されます。 その他のチャネルでは、テキストとして表示されます。 |
| 4 | 自動 | スタイルは、チャネルとオプションの数に基づいて自動的に選択されます。 | 

`builder` オブジェクトからこの列挙にアクセスできます。または、`ListStyle` を選択するインデックスを提供できます。 たとえば、次のコード スニペット内の 2 つのステートメントは、同じことを実現します。

```javascript
// ListStyle passed in as Enum
builder.Prompts.choice(session, "Which color?", "red|green|blue", { listStyle: builder.ListStyle.button });

// ListStyle passed in as index
builder.Prompts.choice(session, "Which color?", "red|green|blue", { listStyle: 3 });
```

オプションの一覧を指定するには、パイプ (`|`) で区切られた 文字列、文字列の配列、またはオブジェクト マップを使用できます。

パイプで区切られた文字列: 

```javascript
builder.Prompts.choice(session, "Which color?", "red|green|blue");
```

文字列の配列:

```javascript
builder.Prompts.choice(session, "Which color?", ["red","green","blue"]);
```

オブジェクト マップ: 

```javascript
var salesData = {
    "west": {
        units: 200,
        total: "$6,000"
    },
    "central": {
        units: 100,
        total: "$3,000"
    },
    "east": {
        units: 300,
        total: "$9,000"
    }
};

bot.dialog('getSalesData', [
    function (session) {
        builder.Prompts.choice(session, "Which region would you like sales for?", salesData); 
    },
    function (session, results) {
        if (results.response) {
            var region = salesData[results.response.entity];
            session.send(`We sold ${region.units} units for a total of ${region.total}.`); 
        } else {
            session.send("OK");
        }
    }
]);
```

### <a name="promptsattachment"></a>Prompts.attachment

イメージやビデオなどのファイルをアップロードすることをユーザーに求める場合は、[Prompts.attachment()][PromptsAttachment] メソッドを使用します。 このプロンプトは、ユーザーの応答を [IPromptAttachmentResult](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptattachmentresult.html) として返します。

```javascript
builder.Prompts.attachment(session, "Upload a picture for me to transform.");
```

## <a name="next-steps"></a>次のステップ

ウォーターフォールとプロンプトを使用してユーザーから情報を収集する方法がわかったので、次に会話フローをさらに管理するために役立つ情報を確認できます。

> [!div class="nextstepaction"]
> [会話フローを管理する](bot-builder-nodejs-dialog-manage-conversation-flow.md)


[SendAttachments]: bot-builder-nodejs-send-receive-attachments.md
[SendCardWithButtons]: bot-builder-nodejs-send-rich-cards.md
[RecognizeUserIntent]: bot-builder-nodejs-recognize-intent-messages.md
[SaveUserData]: bot-builder-nodejs-save-user-data.md

[UniversalBot]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html
[ChatConnector]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[Session]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session


[SendTyping]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping

[EndDialogWithResult]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#enddialogwithresult

[IPromptResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html

[Result_Response]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#response

[ResumeReason]: https://docs.botframework.com/node/builder/chat-reference/enums/_botbuilder_d_.resumereason.html

[Result_Resumed]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#resumed

[entity]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ientity.html

[ResolveTime]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#resolvetime

[PromptsRef]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html

[PromptsText]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#text

[IPromptTextResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttextresult.html

[PromptsConfirm]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#confirm

[IPromptConfirmResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptconfirmresult.html

[PromptsNumber]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#number

[IPromptNumberResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptnumberresult.html

[PromptsTime]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#time

[IPromptTimeResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html

[PromptsChoice]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#choice

[IPromptChoiceResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptchoiceresult.html

[PromptsAttachment]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#attachment

[IPromptAttachmentResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptattachmentresult.html
