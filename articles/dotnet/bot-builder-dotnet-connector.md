---
title: アクティビティを送受信する | Microsoft Docs
description: Bot Framework SDK for .NET を介してコネクタ サービスを使用することで、さまざまなチャネルとの間でユーザーと情報を交換する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0407ec0d90c58e10aa14616e2aa9205bb8840d55
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225227"
---
# <a name="send-and-receive-activities"></a>アクティビティを送受信する

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Bot Framework Connector では、ボットで Skype、電子メール、Slack などの複数のチャネルとの間で通信できるようにする単一の REST API が用意されています。 これにより、ボットからチャネルおよびチャネルからボットにメッセージを返信することで、ボットとユーザー間の通信が容易になります。 

この記事では、ボットとチャネル上のユーザーとの間で情報を交換するために、Bot Framework SDK for .NET を介して Connector を使用する方法について説明します。 

> [!NOTE]
> この記事に示されている技術を排他的に使用することによってボットを作成できるときに、Bot Framework SDK では、[ダイアログ](bot-builder-dotnet-dialogs.md)などの追加機能、および会話のフローと状態を管理するプロセスを効率的にできる [FormFlow](bot-builder-dotnet-formflow.md) が提供され、言語の理解などの認識サービスをよりシンプルにすることができます。

## <a name="create-a-connector-client"></a>コネクタ クライアントを作成する

[ConnectorClient][ConnectorClient] クラスには、ボットによってチャネル上のユーザーと通信するために使用されるメソッドが含まれます。 ボットで Connector から <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity</a> オブジェクトを受信すると、そのアクティビティ用に指定された `ServiceUrl` を使用して、後で返信を生成するために使用されるコネクタ クライアントを作成する必要があります。 

[!code-csharp[Create connector client](../includes/code/dotnet-send-and-receive.cs#createConnectorClient)]

> [!TIP]
> チャネルのエンドポイントは安定していない可能性があるため、(キャッシュされたエンドポイントに依存せずに) いつでも可能なときに、ボットは Connector で `Activity` オブジェクト内で指定されたエンドポイントと直接通信する必要があります。 
>
> ボットで会話を開始する必要がある場合、(そのシナリオには受信する `Activity` オブジェクトがないため) 指定したチャネル用にキャッシュされたエンドポイントを使用できますが、通常はキャッシュされたエンドポイントを更新する必要があることが多いです。 

## <a id="create-reply"></a> 返信を作成する

Connector では、[Activity](bot-builder-dotnet-activities.md) オブジェクトを使用して、ボットとチャネル (ユーザー) 間で情報をやり取りします。 すべてのアクティビティに、メッセージを作成したユーザーに関する情報 (`From` プロパティ)、メッセージのコンテキスト、メッセージの受信者 (`Recipient` プロパティ) と共に、適切な送信先にメッセージをルーティングするために使用される情報が含まれます。

ボットで Connector からアクティビティを受信すると、受信するアクティビティの `Recipient` プロパティでその会話内のボットの ID が指定されます。 一部のチャネル (例: Slack) では会話に追加されるときにボットに新しい ID を割り当てるため、ボットは常にその返信内の `From` プロパティの値として、受信アクティビティの `Recipient` プロパティの値を使用する必要があります。

自分で最初から送信の `Activity` オブジェクトを作成して初期化することもできますが、Bot Framework SDK には返信を作成するよりも簡単な方法が用意されています。 受信アクティビティの `CreateReply` メソッドを使用することで、返信にメッセージ テキストを簡単に指定し、自動的に作成された `Recipient`、`From`、`Conversation` プロパティと共に送信アクティビティが作成されます。

[!code-csharp[Create reply](../includes/code/dotnet-send-and-receive.cs#createReply)]

## <a name="send-a-reply"></a>返信を送信する

返信を作成すると、コネクタ クライアントの `ReplyToActivity` メソッドを呼び出して送信することができます。 Connector では、適切なチャネル セマンティックを使用して返信が提供されます。 

[!code-csharp[Send reply](../includes/code/dotnet-send-and-receive.cs#sendReply)]

> [!TIP]
> ボットによってユーザーのメッセージに返信している場合、常に `ReplyToActivity` メソッドを使用します。

## <a name="send-a-non-reply-message"></a>(非応答) メッセージを送信する 

ボットが会話の一部になる場合、`SendToConversation` メソッドを呼び出すことで、ユーザーからの任意のメッセージに直接の返信ではないメッセージを送信できます。 

[!code-csharp[Send non-reply message](../includes/code/dotnet-send-and-receive.cs#sendNonReplyMessage)]

`CreateReply` メソッドを使用して、(メッセージに `Recipient`、`From`、`Conversation` プロパティを自動的に設定する) 新しいメッセージを初期化できます。 また、`CreateMessageActivity` メソッドを使用し、新しいメッセージを作成して、プロパティの値をすべて自分で設定することもできます。

> [!NOTE]
> Bot Framework では、ボットで送信できるメッセージ数の制限を指定していません。 しかし、ほとんどのチャネルでは、ボットが短時間で多数のメッセージを送信しないように、調整制限を強制します。 さらに、ボットで立て続けに複数のメッセージを送信する場合は、チャネルでは適切な順序でメッセージがレンダリングされない可能性があります。

## <a name="start-a-conversation"></a>会話を開始する

ご利用のボットで 1 人または複数のユーザーと会話を開始する必要がある場合があります。 `ConversationAccount` オブジェクトを取得するために、`CreateDirectConversation` メソッド (単一ユーザーとのプライベート会話の場合) または `CreateConversation` メソッド (複数のユーザーとのグループ会話の場合) のいずれかを呼び出すことで、会話を開始することができます。 次に、メッセージを作成し、`SendToConversation` メソッドを呼び出すことによって送信します。 `CreateDirectConversation` メソッドまたは `CreateConversation` メソッドのいずれかを使用するには、まず (前のメッセージから保持している場合、キャッシュから取得できる) ターゲット チャネルのサービス URL を使用することによって、[コネクタ クライアントを作成](#create-a-connector-client)する必要があります。 

> [!NOTE]
> すべてのチャネルが、グループ会話をサポートするわけではありません。 チャネルがグループ会話をサポートするかどうかを判断するには、チャネルのドキュメントを参照してください。

このコードの例では、`CreateDirectConversation` メソッドを使用して単一ユーザーとのプライベート会話を作成します。

[!code-csharp[Start private conversation](../includes/code/dotnet-send-and-receive.cs#startPrivateConversation)]

このコードの例では、`CreateConversation` メソッドを使用して複数のユーザーとのグループ会話を作成します。

[!code-csharp[Start group conversation](../includes/code/dotnet-send-and-receive.cs#startGroupConversation)]

## <a name="additional-resources"></a>その他のリソース

- [アクティビティの概要](bot-builder-dotnet-activities.md)
- [メッセージの作成](bot-builder-dotnet-create-messages.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Framework SDK for .NET リファレンス</a>
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity クラス</a>
- <a href="/dotnet/api/microsoft.bot.connector.connectorclient" target="_blank">ConnectorClient クラス</a>

[ConnectorClient]: /dotnet/api/microsoft.bot.connector.connectorclient
