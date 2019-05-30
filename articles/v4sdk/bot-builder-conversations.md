---
title: Bot Framework SDK 内での会話 | Microsoft Docs
description: Bot Framework SDK 内での会話について説明します。
keywords: 会話フロー, 意図の認識, 1 ターン, 複数ターン, ボットの会話
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 167e496fa510cdf755be13f71cf3a596b0183ec1
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215474"
---
# <a name="conversation-flow"></a>会話フロー
[!INCLUDE[applies-to](../includes/applies-to.md)]

ボットの会話フローを設計するには、ユーザーがボットに何か言ったときにボットがどのように応答するかを決定する必要があります。 ボットはまず、ユーザーからのメッセージに基づいてタスクまたは会話トピックを認識します。 ユーザーのメッセージに関連付けられたタスクまたはトピック ("*意図*" とも呼びます) を決定するために、ボットはユーザーのメッセージのテキストから単語またはパターンを探したり、[Language Understanding](bot-builder-concept-luis.md) や [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/overview/overview) のようなサービスを利用したりできます。

ボットがユーザーの意図を認識すると、シナリオに応じて、ボットは 1 回の返信でユーザーの要求を満足して 1 ターンで会話を完了する場合もあれば、数ターン必要な場合もあります。 複数ターンの会話フローの場合、Bot Framework SDK では、会話を追跡するための[状態管理](./bot-builder-howto-v4-state.md)、情報を尋ねるための[プロンプト](bot-builder-prompts.md)、および、会話フローをカプセル化するための[ダイアログ](bot-builder-dialog-manage-conversation-flow.md)が提供されます。

複数のサブシステムを持つ複雑なボットでは、ボットのサブコンポーネントごとに 1 つずつ、複数のサービスを使用して意図を認識する場合があります。 会話のサブシステムを 1 つのボットに結合する場合、[ディスパッチ ツール](bot-builder-tutorial-dispatch.md)は複数のサービスの結果を 1 か所で取得します。

<!-- 
A conversation identifies a series of activities sent between a bot and a user on a specific channel and represents an interaction between one or more bots and either a _direct_ conversation with a specific user or a _group_ conversation with multiple users.
A bot communicates with a user on a channel by receiving activities from, and sending activities to the user.

- Each user has an ID that is unique per channel.
- Each conversation has an ID that is unique per channel.
- The channel sets the conversation ID when it starts the conversation.
- The bot cannot start a conversation; however, once it has a conversation ID, it can resume that conversation.
- Not all channels support group conversations.
-->

## <a name="single-turn-conversation"></a>1 ターンの会話

最も単純な会話フローは 1 ターンです。 1 ターンのフローでは、ボットはそのタスクを 1 ターンで終了し、これはユーザーからの 1 つのメッセージとボットからの 1 会の返信で構成されます。

<!-- The following isn't always true, it's a generalization -->

最も単純な種類の 1 ターンのボットでは、会話の状態を追跡する必要はありません。 メッセージを受信するたびに、ボットは現在の受信メッセージのコンテキストのみに基づいて応答し、過去の会話ターンの知識はありません。

![1 ターンのお天気ボット](./media/concept-conversation/weather-single-turn.png)

お天気ボットは 1 ターンのフローを持つ可能性があり、ユーザーに天気予報を提供するだけです。都市や日付を尋ねて処理を前後することはありません。 天気予報を表示するすべてのロジックは、ボットが受信したばかりのメッセージに基づいています。 会話の各ターンでボットは[ターン コンテキスト](bot-builder-concept-activity-processing.md#turn-context)を受け取り、ボットはそれを使って、次に行う処理や、会話がどのように進むかを決定できます。

## <a name="multiple-turns"></a>複数ターン

ほとんどの種類の会話は 1 ターンでは完了できないため、ボットは複数ターンの会話フローを持つこともあります。 複数の会話ターンが必要なシナリオには、次のようなものがあります。

* タスクを完了するために必要な追加情報をユーザーに問い合わせるボット。 ボットは、タスクを完了するためのパラメーターが全部揃っているかどうかを追跡する必要があります。
* 注文などのプロセスで、ステップ形式でユーザーを案内するボット。 ボットは、ステップのシーケンス内でのユーザーの位置を追跡する必要があります。

たとえば、お天気ボットが複数ターンのフローを持つ可能性があるのは、「天気はどうですか?」という質問に対するボットの応答で 都市を尋ねるような場合です。

![複数ターンのお天気ボット](./media/concept-conversation/weather-multi-turn.png)

都市を尋ねるボットのプロンプトにユーザーが応答し、ボットが「シアトル」を受け取った場合、現在のメッセージが前のプロンプトへの応答、かつ天気を取得する要求の一部であることを理解するためには、何らかの保存済みコンテキストをボットが持っている必要があります。 複数ターンのボットは、新しいメッセージに適切に応答するために状態を追跡します。

詳細については、[会話とユーザー状態を管理する方法](bot-builder-howto-v4-state.md)に関するページを参照してください。

> [!NOTE]
> REST API クライアントとの複数ターン会話では、たとえばデータベースまたはテーブル ストレージで、独自の状態を追跡する必要があります。

## <a name="conversation-topics"></a>会話トピック

複数の種類のタスクを処理するためにボットを設計することができます。 たとえば、ユーザーに挨拶したり、注文したり、注文をキャンセルしたり、ヘルプを取得したりするためのさまざまな会話フローを提供するボットを作成できます。 さまざまなタスクまたは会話トピックの会話間でこの切り替えを処理する 1 つの方法は、現在のメッセージから意図 (ユーザーがしたいこと) を認識することです。

### <a name="recognize-intent"></a>意図の認識

Bot Framework SDK では、メッセージを処理して意図を判断する "_認識エンジン_" が提供されているので、ボットで適切な会話フローを開始できます。 ユーザーの意図をメッセージの内容から判断するには、認識エンジンの _recognize_ 非同期メソッドを呼び出します。 次に、結果に対して _get top scoring intent_ メソッドを呼び出して、認識エンジンの最上位の予測を取得できます。

認識エンジンでは、正規表現、Language Understanding、または開発するその他のロジックを使用できます。 使用可能な認識エンジンの例を次に示します。

* QnA Maker を使用して、サポート技術情報で取り上げられている質問をユーザーが行うタイミングを検出する認識エンジン
* Language Understanding (LUIS) と、ユーザーがヘルプを求める方法の例を使用してサービスをトレーニングし、それを `Help` の意図にマップする認識エンジン
* 正規表現を使用してコマンドを検索するカスタム認識エンジン
* サービスを使用して入力を変換するカスタム認識エンジン

### <a name="consider-how-to-interrupt-conversation-flow-or-change-topics"></a>会話フローを中断またはトピックを変更する方法の検討

会話内での位置を追跡する 1 つの方法は、[会話状態](bot-builder-howto-v4-state.md)を使用して、現在アクティブなトピックや、シーケンス内で完了しているステップに関する情報を保存することです。

ボットがより複雑になると、一連の会話フローがスタック内で発生することも想像できます。たとえば、ボットが新規注文フローを呼び出し、次に商品検索フローを呼び出します。 その後、ユーザーは商品を選択して確認し、商品検索フローを完了し、注文を完了します。

しかし、会話がそのような直線的で論理的な経路をたどることはまれです。 ユーザーは "スタック" でコミュニケーションを取るわけではなく、頻繁に心変わりする傾向があります。 次の例を考えてみます。

![予期していないことをユーザーが言う](./media/concept-conversation/interruption.png)

ボットがフローのスタックを論理的に構築していたとしても、ユーザーは現在のトピックとまったく違う何かをしようと思う、または現在のトピックに関係ない質問をしようと思うかもしれません。 例では、ユーザーはフローが期待するはい/いいえの応答を返す代わりに質問をします。 フローはどのように応答すべきでしょうか? 

* まず質問に答えるよう、ユーザーに主張します。
* ユーザーの以前の行動をすべて無視し、フロー スタック全体をリセットし、ユーザーの質問に答えようとすることによって最初から始めます。
* ユーザーの質問に答えようとし、そのはい/いいえの質問に戻って、そこから再開することを試みます。

この質問に対する正解はありません。なぜなら、シナリオの詳細や、ボットの応答に対するユーザーの妥当な期待によって、最良の解決策が変わってくるからです。 会話フローを管理するには、[ダイアログを使用する](bot-builder-dialog-manage-conversation-flow.md)方法と[割り込みを処理する](bot-builder-howto-handle-user-interrupt.md)方法に関するページを参照してください。

## <a name="conversation-lifetime"></a>会話の有効期間

<!-- Note: these activities are dependent on whether the channel actually sends them. Also, we should add links -->
ボットは、ボットが会話に追加されたり、他のメンバーが会話に追加または会話から削除されたり、会話メタデータが変更されたりするたびに、"_会話更新_" アクティビティを受信します。
ユーザーに挨拶したり、自己紹介をしたりすることで、ボットを会話更新アクティビティに反応させることが必要な場合があります。

ボットは、ユーザーが会話を終了したことを示す_会話終了_アクティビティを受信します。 ボットは、"_会話終了_" アクティビティを送信して、会話を終了していることを示すことができます。
会話についての情報を保存している場合、会話が終了したらその情報を消去することができます。

<!--  Types of conversations -->

ボットは、複数ターンの対話をサポートします。このような対話では、ボットが複数の情報をユーザーに尋ねます。 ボットは限られた 1 つのタスクに集中するか、または複数の種類のタスクをサポートできます。
Bot Framework SDK には、自然言語の "質問と回答" 機能をボットに追加するための Language Understanding (LUIS) および QnA Maker のサポートが組み込まれています。

## <a name="conversations-channels-and-users"></a>会話、チャネル、ユーザー

会話は特定のユーザーとの_直接_会話と、複数のユーザーとの_グループ_会話のどちらかです。
ボットは、ユーザーとの間でアクティビティを送受信することで、チャネルのユーザーとコミュニケーションを取ります。

* 各ユーザーにはチャネルごとに一意の ID があります。
* 各会話にはチャネルごとに一意の ID があります。
* チャネルは会話を開始するときに会話 ID を設定します。
* ボットは会話を開始できません。ただし、会話 ID を取得した後であれば、その会話を再開できます。
* すべてのチャネルが、グループ会話をサポートするわけではありません。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [ダイアログを使用して単純な会話フローを管理する](bot-builder-dialog-manage-conversation-flow.md)

<!-- In addition, your bot can send activities back to the user, either _proactively_, in response to internal logic, or _reactively_, in response to an activity from the user or channel.-->
<!--TODO: Link to messaging how tos.-->
