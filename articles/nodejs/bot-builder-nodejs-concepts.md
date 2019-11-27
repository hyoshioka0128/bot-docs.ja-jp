---
title: Bot Framework SDK for Node.js の主要概念 | Microsoft Docs
description: Bot Framework SDK for Node.js で利用可能な会話型ボットを構築およびデプロイするための主要概念とツールについて説明します。
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 81a4f184f2011e6a09c645e61e1d8c3dfabdebaa
ms.sourcegitcommit: dbc7eaee5c1f300b23c55abe6b60cd01c7408915
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/23/2019
ms.locfileid: "74415149"
---
# <a name="key-concepts-in-the-bot-framework-sdk-for-nodejs"></a>Bot Framework SDK for Node.js の主要概念

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-concepts.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-concepts.md)

この記事では、Bot Framework SDK for Node.js の主要な概念について説明します。 Bot Framework の概要については、[Bot Framework の概要](../overview-introduction-bot-framework.md)に関するページを参照してください。

## <a name="connector"></a>コネクタ
Bot Framework Connector は、ご利用のボットを複数の "*チャネル*" ([Teams](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-create)、Skype、Facebook、Slack、SMS などのクライアント) に接続するサービスです。 

この Connector では、ボットからチャネルへのメッセージおよびチャネルからボットへのメッセージが中継されて、ボットとユーザー間の通信が容易になります。 ご利用のボットのロジックは、ユーザーからのメッセージを Connector サービスを経由して受信する Web サービスとしてホストされます。ご利用のボットの応答は HTTPS POST を使用して Connector に送信されます。 

Bot Framework SDK for Node.js には [UniversalBot][UniversalBot] クラスおよび [ChatConnector][ChatConnector] クラスが用意されており、これらのクラスを使用することで、ボットにおけるメッセージの送受信が Bot Framework Connector を経由して行われるように構成することができます。 `UniversalBot` クラスでは、ご利用のボットの頭脳が形成されます。 このクラスの役割は、ご利用のボットで行われるユーザーとの会話をすべて管理することにあります。 `ChatConnector` クラスでは、ご利用のボットを Bot Framework Connector サービスに接続することができます。
これらのクラスの使用例については、「[Bot Framework SDK for Node.js を使用したボットの作成](bot-builder-nodejs-quickstart.md)」をご覧ください。

Connector ではボットからチャネルに送信されるメッセージの正規化も行われます。このため、プラットフォームに依存しない方法でボットを開発することができます。 メッセージを正規化するには、メッセージを Bot Framework のスキーマから、チャネルのスキーマに変換する必要があります。 フレームワークのスキーマのすべての側面がチャネルによってサポートされているわけではない場合は、そのチャネルでサポートされている形式にメッセージを変換する試みが Connector によって行われます。 たとえば、カードとアクション ボタンを含むメッセージがボットから SMS チャネルに送信される場合、Connector によってカードがイメージとしてレンダリングされ、アクションがリンクとしてメッセージのテキストに取り込まれます。 [Channel Inspector][ChannelInspector] は、Connector によってメッセージが各種のチャネル上でどのようにレンダリングされるかを示す Web ツールです。

`ChatConnector` では、ご利用のボット内で API エンドポイントを設定する必要があります。 Node.js SDK の場合、この設定は、通常、`restify` Node.js モジュールをインストールすることによって行われます。 また、[ConsoleConnector][ConsoleConnector] (API エンドポイントは不要) を使用すると、コンソール向けのボッドを作成することができます。

## <a name="messages"></a>メッセージ

メッセージは、表示するテキスト、読み上げるテキスト、添付ファイル、リッチ カード、および推奨されるアクションで構成することができます。 ユーザーからのメッセージに応答してメッセージを送信するには、[session.send][SessionSend] メソッドを使用します。 ご利用のボットでは、ユーザーからのメッセージに応答して何度でも `send` が呼び出される可能性があります。 これを示す例については、[ユーザー メッセージへの応答][RespondMessages]に関するページを参照してください。

ユーザーがアクションを開始するためにクリックするインタラクティブ ボタンが含まれるリッチ グラフィカル カードを送信する方法を示す例については、[メッセージへのリッチ カードの追加](bot-builder-nodejs-send-rich-cards.md)に関するページを参照してください。 添付ファイルの送受信の方法を示す例については、[添付ファイルの送信](bot-builder-nodejs-send-receive-attachments.md)に関するページを参照してください。 音声認識チャネル上でご利用のボットによって読み上げられるテキストを指定するメッセージを送信する方法の例については、「[メッセージへの音声の追加](bot-builder-nodejs-text-to-speech.md)」を参照してください。 推奨されるアクションを送信する方法の例については、[推奨されるアクションの送信](bot-builder-nodejs-send-suggested-actions.md)に関するページを参照してください。

## <a name="dialogs"></a>ダイアログ
ダイアログは、ご利用のボット内で会話ロジックを整理するのに役立つと共に、[会話フローを設計する](../bot-service-design-conversation-flow.md)上での基礎となります。 ダイアログの概要については、[ダイアログを使用した会話の管理](bot-builder-nodejs-dialog-manage-conversation.md)に関するページを参照してください。

## <a name="actions"></a>Actions
会話フロー中に任意のタイミングで発生するキャンセル要求やヘルプ要求などの割り込みを処理できるようにご自分のボットを設計する必要があります。 Bot Framework SDK for Node.js では、キャンセルやその他のダイアログの呼び出しのようなアクションをトリガーするグローバル メッセージ ハンドラーが提供されています。 [triggerAction][triggerAction] ハンドラーの使用方法を示す例については、「[ユーザー アクションを処理する](bot-builder-nodejs-dialog-actions.md)」を参照してください。
<!--[Handling cancel](bot-builder-nodejs-manage-conversation-flow.md#handling-cancel), [Confirming interruptions](bot-builder-nodejs-manage-conversation-flow.md#confirming-interruptions) and-->


## <a name="recognizers"></a>認識エンジン
"ヘルプ" や "ニュースの検索" のようなことがユーザーから求められた場合、ご利用のボットによって、ユーザーからの要求の内容が把握され、適切なアクションが実行される必要があります。 ユーザーの入力に基づいて意図が認識され、その意図がアクションに関連付けられるように、ご自分のボットを設計することができます。 

Bot Framework SDK で提供されている組み込みの正規表現認識エンジンを使用するか、LUIS API などの外部サービスを呼び出すか、またはカスタム認識エンジンを実装することで、ユーザーの意図を判断することもできます。 ご利用のボットに認識エンジンを追加し、それらを使用してアクションをトリガーする方法を示す例については、[ユーザーの意図の認識](bot-builder-nodejs-recognize-intent-messages.md)に関するページを参照してください。


## <a name="saving-state"></a>状態の保存

ボットを適切に設計するための鍵は、会話のコンテキストを追跡することです。そうすることで、ユーザーからの最後の質問などの事柄をボットで記憶することができます。 Bot Framework SDK を使用して構築するボットはステートレスとなるように設計されます。そのため、複数の計算ノード間で実行する場合にボッドを容易にスケーリングすることができます。 Bot Framework にはボットのデータを格納するストレージ システムが用意されているため、ボッド Web サービスをスケーリングすることができます。 そのことが理由で、通常は、グローバル変数または関数クロージャを使用して状態を保存しないようにする必要があります。 それを行うと、ご利用のボットをスケール アウトする場合に問題が発生します。 代わりに、ご利用のボットの[session][Session] オブジェクトの次のプロパティを使用します。

* **userData** では、すべての会話でのユーザーの情報がグローバルに格納されます。
* **conversationData** では、単一の会話に関する情報がグローバルに格納されます。 このデータは、会話に参加しているすべてのユーザーに表示されるので、このプロパティにデータを格納する場合は注意が必要です。 既定では有効になっています。ボットの [persistConversationData][PersistConversationData] 設定を使用することで、無効にすることができます。
* **privateConversationData** では単一の会話に関する情報がグローバルに格納されますが、これは現在のユーザーに固有のプライベート データとなります。 このデータはすべてのダイアログにまたがっています。そのため、会話の終了時にクリーンアップされるようにしたい一時的な状態を格納する場合に役に立ちます。
* **dialogData** では単一のダイアログ インスタンスに関する情報が保持されます。 これは、ダイアログ内の[ウォーターフォール](bot-builder-nodejs-dialog-waterfall.md)のステップ間における一時的な情報を格納するのに不可欠です。

これらのプロパティを使用してデータを格納および取得する方法を示す例については、「[Manage state data](bot-builder-nodejs-state.md)」(状態データの管理) を参照してください。

## <a name="natural-language-understanding"></a>自然言語の理解

Bot Builder では、LUIS を使用して、ご利用のボットに自然言語の理解機能を追加することができます。[LuisRecognizer][LuisRecognizer] クラスを使用します。 **LuisRecognizer** のインスタンスを追加することができます。これによって、ご利用の公開された言語モデルが参照され、ユーザーの発話に応答してアクションを実行するハンドラーが追加されます。 実行中の LUIS を確認するには、次の 10 分間チュートリアルをご覧ください。

* [Microsoft LUIS チュートリアル][LUISVideo] (ビデオ)

## <a name="next-steps"></a>次の手順
> [!div class="nextstepaction"]
> [ダイアログの概要](bot-builder-nodejs-dialog-overview.md)



[PersistConversationData]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iuniversalbotsettings.html#persistconversationdata
[UniversalBot]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html
[ChatConnector]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[ConsoleConnector]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.consoleconnector.html

[ChannelInspector]: ../bot-service-channel-inspector.md

[Session]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html
[SessionSend]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#send

[triggerAction]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction
[waterfall]: bot-builder-nodejs-prompts.md

[RespondMessages]:bot-builder-nodejs-use-default-message-handler.md

[LUISRecognizer]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer
[LUISVideo]: https://vimeo.com/145499419