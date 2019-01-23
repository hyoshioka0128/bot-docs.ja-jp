---
title: メッセージを作成する | Microsoft Docs
description: Bot Framework SDK for Node.js を使用したメッセージの作成方法を学習します。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/7/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3a4f9e1dc3c5598c3aa79996b01f11e8b1339fe2
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225217"
---
# <a name="create-messages"></a>メッセージを作成する

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

ボットとユーザー間の通信は、メッセージを通じて行われます。 ボットは、メッセージ アクティビティを送信してユーザーに情報を伝達し、同様にユーザーからのメッセージ アクティビティも受信します。 プレーン テキストだけで構成されているシンプルなメッセージもあれば、読み上げテキスト、推奨されるアクション、メディアの添付ファイル、リッチ カード、チャネル固有のデータなど、よりリッチなコンテンツを含むメッセージもあります。

この記事では、ユーザー エクスペリエンスを強化するために使用できる、よく使用されるメッセージ メソッドについていくつか説明します。

## <a name="default-message-handler"></a>既定のメッセージ ハンドラー

Bot Framework SDK for Node.js には、[`session`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html) オブジェクトに組み込まれている既定のメッセージ ハンドラーが付属しています。 このメッセージ ハンドラーを使用すると、ボットとユーザー間でテキスト メッセージの送受信を行うことができます。

### <a name="send-a-text-message"></a>テキスト メッセージの送信

既定のメッセージ ハンドラーを使用してテキスト メッセージを送信するのは簡単です。`session.send` を呼び出して、**文字列**で渡すだけです。

このサンプルでは、ユーザーにあいさつするテキスト メッセージを送信する方法を示しています。
```javascript
session.send("Good morning.");
```

このサンプルでは、JavaScript 文字列テンプレートを使用してテキスト メッセージを送信する方法を示しています。
```javascript
var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
session.send(msg); //msg: "You ordered: Potato Salad for a total of $5.99."
```

### <a name="receive-a-text-message"></a>テキスト メッセージの受信

ユーザーがボットにメッセージを送信すると、ボットは `session.message` プロパティを通じてそのメッセージを受信します。

このサンプルでは、ユーザーのメッセージにアクセスする方法を示しています。
```javascript
var userMessage = session.message.text;
```

## <a name="customizing-a-message"></a>メッセージのカスタマイズ

メッセージのテキストの書式設定をさらに制御するため、カスタム [`message`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html) オブジェクトを作成して、ユーザーに送信する前に必要なプロパティを設定することができます。

このサンプルでは、カスタム `message` オブジェクトを作成し、`text`、`textFormat`、`textLocale` のプロパティを設定する方法を示しています。

```javascript
var customMessage = new builder.Message(session)
    .text("Hello!")
    .textFormat("plain")
    .textLocale("en-us");
session.send(customMessage);
```

スコープ内に `session` オブジェクトがない場合は、`bot.send` メソッドを使用して書式設定されたメッセージをユーザーに送信することができます。

メッセージの `textFormat` プロパティを使用して、テキストの書式を指定することができます。 `textFormat` プロパティは、**plain**、**markdown**、**xml** のいずれかに設定できます。 `textFormat` の既定値は **markdown** です。 

## <a name="message-property"></a>メッセージ プロパティ

[`Message`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html) オブジェクトには、送信されるメッセージを管理するために使用する内部の**データ** プロパティがあります。 設定するその他のプロパティは、このオブジェクトによって公開されるさまざまなメソッドを通じて提供されます。 

## <a name="message-methods"></a>メッセージ メソッド

メッセージ プロパティは、オブジェクトのメソッドを通じて設定および取得されます。 さまざまな**メッセージ** プロパティを設定/取得するために呼び出すことができるメソッドの一覧を次の表に示します。

| 方法 | 説明 |
| ---- | ---- | 
| [`addAttachment(attachment:AttachmentType)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#addattachment) | メッセージに添付ファイルを追加します|
| [`addEntity(obj:Object)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#addentity) | メッセージにエンティティを追加します。 |
| [`address(adr:IAddress)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#address) | メッセージのルーティング情報をアドレス指定します。 ユーザーにプロアクティブ メッセージを送信するには、メッセージのアドレスを userData バッグに保存します。 |
| [`attachmentLayout(style:string)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#attachmentlayout) | クライアントが複数の添付ファイルをどのようにレイアウトすべきかのヒント。 既定値は "list" です。 |
| [`attachments(list:AttachmentType)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#attachments) | ユーザーに送信するカードまたはイメージの一覧。 |
| [`compose(prompts:string[], ...args:any[])`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#compose) | ユーザーに対して複雑なランダム化された応答を作成します。 |
| [`entities(list:Object[])`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#entities) | ボットまたはユーザーに渡される構造化されたオブジェクト。 |
| [`inputHint(hint:string)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#inputhint) | ボットがさらなる入力を想定しているかどうかをユーザーに知らせるために送信されるヒント。 送信メッセージのこの値は、組み込みのプロンプトによって自動的に入力されます。 |
| [`localTimeStamp((optional)time:string)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#localtimestamp) | メッセージが送信されたときのローカル時刻 (クライアントまたはボットによって設定、例:2016-09-23T13:07:49.4714686-07:00)。 |
| [`originalEvent(event:any)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#originalevent) | 受信メッセージのチャネルの元の形式またはネイティブ形式のメッセージ。 |
| [`sourceEvent(map:ISourceEventMap)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#sourceevent) | カスタムの添付ファイルのようなソース固有のイベント データを渡すために使用できる送信メッセージ。 |
| [`speak(ssml:TextType, ...args:any[])`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#speak) | メッセージの音声フィールドを*音声合成マークアップ言語 (SSML)* として設定します。 これはサポートされているデバイスで、ユーザーに対して読み上げられます。 |
| [`suggestedActions(suggestions:ISuggestedActions `&#124;` IIsSuggestedActions)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#suggestedactions) | ユーザーに送信する省略可能な推奨されるアクション。 推奨されるアクションは、推奨されるアクションをサポートするチャネルでのみ表示されます。 |
| [`summary(text:TextType, ...argus:any[])`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#summary) | フォールバックとして、およびメッセージのコンテンツの短い説明として表示されるテキスト (例:最近の会話のリスト)。 |
| [`text(text:TextType, ...args:any[])`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#text) | メッセージ テキストを設定します。 |
| [`textFormat(style:string)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#textformat) | テキスト形式を設定します。 既定の形式は **markdown** です。 |
| [`textLocale(locale:string)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#textlocale) | メッセージの対象言語を設定します。 |
| [`toMessage()`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#tomessage) | メッセージの JSON を取得します。 |
| [`composePrompt(session:Session, prompts:string[], args?:any[])`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#composeprompt-1) | プロンプトの配列を 1 つのローカライズされたプロンプトに結合し、必要に応じて、渡された引数をプロンプト テンプレートのスロットに入力します。 |
| [`randomPrompt(prompts:TextType)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#randomprompt) | 渡される **prompts* の配列からランダムなプロンプトを取得します。 |

## <a name="next-step"></a>次のステップ

> [!div class="nextstepaction"]
> [添付ファイルを送受信する](bot-builder-nodejs-send-receive-attachments.md)

