---
title: 音声通話を行う | Microsoft Docs
description: Node.js を使用して、ボットの Skype と音声通話を行う方法について説明します。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 147862b0fe684f9312a94fe903326ca737f5a3d6
ms.sourcegitcommit: 697a577d72aaf91a0834d4b4c2ef5aa11291f28f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/01/2019
ms.locfileid: "67496700"
---
# <a name="support-audio-calls-with-skype"></a>Skype での音声通話のサポート

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Skype は、通話ボットという高度な機能をサポートしています。  有効にすると、ボットに音声通話を発信し、インタラクティブ ボイス レスポンス (IVR) を使用してボットと対話できます。  Bot Builder SDK for Node.js SDK には、開発者がチャット ボットに通話機能を追加するために利用できる特殊な [Calling SDK][calling_sdk] が含まれています。   

Calling SDK は、[Chat SDK][chat_sdk] とよく似ています。 同様のクラスを持ち、共通の構造を共有するだけでなく、Chat SDK を使用して通話相手のユーザーにメッセージを送信することもできます。  2 つの SDK は同時に実行できるように設計されていますが、似ている点と重要な違いがあるため、2 つのライブラリのクラスを混同しないように気を付ける必要があります。  

## <a name="create-a-calling-bot"></a>通話ボットを作成する
通話ボットの "Hello World" が通常のチャットボットとよく似ていることを示すコード例を次に示します。 

```javascript
var restify = require('restify');
var calling = require('botbuilder-calling');

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log(`${server.name} listening to ${server.url}`); 
});

// Create calling bot
var connector = new calling.CallConnector({
    callbackUrl: 'https://<your host>/api/calls',
    appId: '<your bots app id>',
    appPassword: '<your bots app password>'
});
var bot = new calling.UniversalCallBot(connector);
server.post('/api/calls', connector.listen());

// Add root dialog
bot.dialog('/', function (session) {
    session.send('Watson... come here!');
});
```

> [!NOTE]
> ボットの **AppID** と **AppPassword** の確認方法については、「[MicrosoftAppID and MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)」(MicrosoftAppID と MicrosoftAppPassword) を参照してください。

現在、エミュレーターは通話ボットのテストをサポートしていません。 ボットをテストするには、ボットの発行に必要な手順のほとんどを実行する必要があります。  また、Skype クライアントを使用してボットと対話する必要があります。 

### <a name="enable-the-skype-channel"></a>Skype チャンネルを有効にする
[[Register your bot]\(ボットの登録\)](../bot-service-quickstart-registration.md) を行い、Skype チャンネルを有効にします。 ボットの登録時にメッセージング エンドポイントを指定する必要があります。 通常はチャット ボットのエンドポイントがこのフィールドに設定されるように、通話ボットをチャット ボットとペアにすることをお勧めします。  通話ボットの登録のみを行う場合は、このフィールドに呼び出し元エンドポイントを貼り付けるだけです。  

実際の通話機能を有効にするには、ボットの Skype チャンネルを開き、通話機能を有効にする必要があります。 呼び出し元エンドポイントをコピーするフィールドが表示されます。 呼び出し元エンドポイントのホスト部分に https ngrok リンクが入力されていることを確認します。

ボットの登録時に、hello world ボットのコネクタ設定に貼り付けるアプリ ID とパスワードが割り当てられます。 また、完全な呼び出し元のリンクをコピーし、callbackUrl 設定にそれを貼り付ける必要があります。

### <a name="add-bot-to-contacts"></a>連絡先にボットを追加する
開発者ポータルのボットの登録ページには、ボット Skype チャンネルの横に **[add to Skype]\(Skype に追加\)** ボタンが表示されます。 ボタンをクリックして Skype の連絡先リストにボットを追加します。  追加が完了すると、自分だけでなく、参加リンクを教えたすべてのユーザーがボットと通信できるようになります。

### <a name="test-your-bot"></a>ボットをテストする
Skype クライアントを使用してボットをテストできます。 ボットの連絡先エントリをクリックすると、呼び出しアイコンが点灯します (ボットを表示するには、必要に応じてボットを検索します)。既存のボットの呼び出しを追加した場合は、呼び出しアイコンが点灯するまでに数分かかることがあります。  

呼び出しボタンを押すと、ボットが呼び出され、「Watson… come here!」(ワトソン...ここに来てくれ!) という声が聞こえ、 通話が終了します。

## <a name="calling-basics"></a>通話の基本
[UniversalCallBot](http://docs.botframework.com/node/builder/calling-reference/classes/_botbuilder_d_.universalcallbot) クラスと [CallConnector](http://docs.botframework.com/node/builder/calling-reference/classes/_botbuilder_d_.callconnector) クラスを使用すると、チャット ボットと同じように通話ボットを作成できます。 [チャット ダイアログ](bot-builder-nodejs-manage-conversation-flow.md)と基本的に同じボットにダイアログを追加します。 ボットには[ウォーターフォール](bot-builder-nodejs-prompts.md)を追加できます。 セッション オブジェクト [CallSession](http://docs.botframework.com/node/builder/calling-reference/classes/_botbuilder_d_.callsession) クラスがあり、このクラスには、現在の通話を管理するための、追加された [answer()](http://docs.botframework.com/node/builder/calling-reference/classes/_botbuilder_d_.callsession#answer)、[hangup()](http://docs.botframework.com/node/builder/calling-reference/classes/_botbuilder_d_.callsession#hangup)、および [reject()](http://docs.botframework.com/node/builder/calling-reference/classes/_botbuilder_d_.callsession#reject) メソッドが含まれています。 一般的に、CallSession には自動的に通話を管理するロジックが用意されているので、このような通話コントロール方法について心配する必要はありません。 メッセージの送信や組み込みプロンプトの呼び出しなどの操作を行うと、セッションは自動的に通話に応答します。 [endConversation ()](http://docs.botframework.com/node/builder/calling-reference/classes/_botbuilder_d_.callsession#endconversation) を呼び出すか、呼び出し元への質問を停止したことが検出された場合 (組み込みのプロンプトを呼び出さなかった場合)、通話は自動的に終了または拒否されます。

通話ボットとチャット ボットのもう 1 つの違いは、通常、チャット ボットはメッセージ、カード、キーボードをユーザーに送信しますが、通話ボットはアクションと結果を処理する点です。 Skype の通話ボットは、1 つまたは複数の[アクション](http://docs.botframework.com/node/builder/calling-reference/interfaces/_botbuilder_d_.iaction)から構成される[ワークフロー](http://docs.botframework.com/node/builder/calling-reference/interfaces/_botbuilder_d_.iworkflow)を作成するために必要です。  Bot Builder の通話 SDK が管理をほとんど自動的に行うので、実際には、この点もあまり心配する必要がありません。 [CallSession.send()](http://docs.botframework.com/node/builder/calling-reference/classes/_botbuilder_d_.callsession#send) メソッドを使用すると、アクションまたは文字列を渡し、[PlayPromptActions](http://docs.botframework.com/node/builder/calling-reference/classes/_botbuilder_d_.playpromptaction) に変換することができます。  セッションには、呼び出し元サービスに送信される単一のワークフローに複数のアクションをまとめる自動バッチ処理ロジックがあるので、安心して send() を複数回呼び出すことができます。  また、すべての結果を処理するときに、SDK の組み込み[プロンプト](bot-builder-nodejs-prompts.md)を利用してユーザーの入力を収集することをお勧めします。  

[calling_sdk]: http://docs.botframework.com/node/builder/calling-reference/modules/_botbuilder_d_
[chat_sdk]: http://docs.botframework.com/node/builder/chat-reference/modules/_botbuilder_d_
