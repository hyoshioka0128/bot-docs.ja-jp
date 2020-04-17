---
title: ダイアログの概要 (v3 JS) - Bot Service
description: Bot Framework SDK for Node.js 内でダイアログを使用して会話をモデル化し、会話フローを管理する方法を学習します。
author: DucVo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: be983482273eaaa5be79f7c4100dcd4741f62469
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75790924"
---
# <a name="dialogs-in-the-bot-framework-sdk-for-nodejs"></a>Bot Framework SDK for Node.js のダイアログ

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-dialogs.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-overview.md)

Bot Framework SDK for Node.js 内でダイアログを使用すると、会話をモデル化し、会話フローを管理できます。 ボットは会話を通じてユーザーとコミュニケーションを取ります。 会話はダイアログに編成されます。 ダイアログには、ウォーターフォール手順とプロンプトを含めることができます。 ユーザーがボットとやりとりすると、ボットはユーザーのメッセージに応じてさまざまなダイアログの開始、停止、切り替えを行います。 ダイアログのしくみを理解することは、優れたボットを適切に設計し、作成するために重要です。 

この記事では、ダイアログの概念を紹介します。 この記事を読んだ後は、「[次の手順](#next-steps)」セクションのリンクをたどって、このような概念の理解を深めてください。

## <a name="conversations-through-dialogs"></a>ダイアログによる会話

Bot Framework SDK for Node.js は、1 つまたは複数のダイアログを介してボットとユーザー間の通信として会話を定義します。 最も基本的なレベルのダイアログは、操作を実行する、またはユーザーから情報を収集する再利用可能なモジュールです。 ボットの複雑なロジックを再利用可能なダイアログ コードにカプセル化することができます。

会話は、さまざまな方法で構成および変更できます。

- [既定ダイアログ](#default-dialog)から始めることができます。
- あるダイアログから別のダイアログにリダイレクトすることができます。
- 再開することができます。
- [ウォーターフォール](bot-builder-nodejs-dialog-waterfall.md) パターンに従うことができます。ウォーターフォール パターンで、ユーザーに一連の手順を案内したり、ユーザーに一連の質問への回答を求める[プロンプト](bot-builder-nodejs-dialog-prompt.md)を表示できます。
- 異なるダイアログをトリガーする単語やフレーズを聞き取る[アクション](bot-builder-nodejs-dialog-actions.md)を使用できます。 

会話は、ダイアログの親のように考えることができます。 そのため、会話には*ダイアログ スタック*が含まれ、独自の状態データ セット (つまり `conversationData` と `privateConversationData`) が維持されます。 一方、ダイアログは `dialogData` を維持します。 状態データの詳細については、「[Manage state data](bot-builder-nodejs-state.md)」(状態データの管理) を参照してください。

## <a name="dialog-stack"></a>ダイアログ スタック

ボットは、ダイアログ スタック上で維持される一連のダイアログを介してユーザーとやりとりします。 会話中にダイアログはスタックにプッシュされ、スタックから削除されます。 スタックは通常の LIFO スタックと同様に動作します。つまり最後に追加されたダイアログから完了します。 ダイアログが完了すると、コントロールはスタック上の前のダイアログに戻ります。

ボットの会話が初めて開始されたとき、または会話が終了したとき、ダイアログ スタックは空です。 この時点で、ユーザーがボットにメッセージを送信すると、ボットは*既定のダイアログ*で応答します。

## <a name="default-dialog"></a>既定のダイアログ

Bot Framework 3.5 より前のバージョンでは、*ルート* ダイアログは、`/` という名前のダイアログを追加して定義されます。そのため、URL と似た名前付け規則になります。 この名前付け規則は、ダイアログの命名には適切ではありませんでした。 

> [!NOTE]
> Bot Framework バージョン 3.5 以降、*既定のダイアログ*は [`UniversalBot`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html#constructor) のコンストラクターに 2 つ目のパラメーターとして登録されています。  

次のコード スニペットは、`UniversalBot` オブジェクトの作成時に既定のダイアログを定義する方法を示しています。

```javascript
var bot = new builder.UniversalBot(connector, [
    //...Default dialog waterfall steps...
    ]);
```

ダイアログ スタックが空で、LUIS または別の[認識エンジン](bot-builder-nodejs-recognize-intent-messages.md)を介して他のダイアログが[トリガー](bot-builder-nodejs-dialog-actions.md)されない場合は、常に既定のダイアログが実行されます。 既定のダイアログはユーザーに対するボットの最初の応答なので、既定のダイアログは、使用できるコマンドの一覧やボットの機能の概要など、コンテキスト情報をユーザーに提供する必要があります。

## <a name="dialog-handlers"></a>ダイアログ ハンドラー

ダイアログ ハンドラーは、会話のフローを管理します。 会話を進めるために、ダイアログ ハンドラーは、ダイアログを開始および終了することでフローを指示します。 

## <a name="starting-and-ending-dialogs"></a>ダイアログの開始と終了

新しいダイアログを開始する (そしてスタックにプッシュする) には、[`session.beginDialog()`](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#begindialog) を使用します。 ダイアログを終了する (そしてスタックから削除し、呼び出し元のダイアログにコントロールを戻す)には、[`session.endDialog()`](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#enddialog) または [`session.endDialogWithResult()`](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#enddialogwithresult) を使用します。 

## <a name="using-waterfalls-and-prompts"></a>ウォーターフォールとプロンプトを使用する

[ウォーターフォール](bot-builder-nodejs-dialog-waterfall.md)は、会話フローをモデル化して管理する簡単な方法です。 ウォーターフォールには一連の手順が含まれています。 各手順では、ユーザーに代わってアクションを完了するか、[プロンプト](bot-builder-nodejs-dialog-prompt.md)でユーザーに情報を提供することができます。

ウォーターフォールは、一連の関数で構成されるダイアログを使用して実装されます。 各関数で、ウォーターフォールの手順を定義します。 次のコード サンプルは、2 段階のウォーターフォールを使用してユーザーに名前の入力を求め、名前を呼んであいさつする簡単な会話の例です。

```javascript
// Ask the user for their name and greet them by name.
bot.dialog('greetings', [
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
```

ボットがダイアログを終了することなくウォーターフォールの最後に到達した後に、ユーザーから次のメッセージが入力されると、ウォーターフォールの手順 1 からそのダイアログは再開されます。 その結果、ユーザーがループに閉じ込められているように感じ、不満につながる可能性があります。 このような状況を回避するには、会話またはダイアログが終了したときは、`endDialog`、`endDialogWithResult`、または `endConversation` を明示的に呼び出すことをお勧めします。

## <a name="next-steps"></a>次のステップ

ダイアログの知識を深めるには、ウォーターフォール パターンのしくみや、それを使用してプロセスを介してユーザーを誘導する方法を理解することが重要です。

> [!div class="nextstepaction"]
> [ウォーターフォールを使用した会話のステップの定義](bot-builder-nodejs-dialog-waterfall.md)
