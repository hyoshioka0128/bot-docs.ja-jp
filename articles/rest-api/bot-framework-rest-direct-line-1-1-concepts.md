---
title: Bot Framework Direct Line API 1.1 の主要な概念 | Microsoft Docs
description: Bot Framework Direct Line API 1.1 の主要な概念を理解します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 63475546472d2305ef665fd4ab29c6f2df2b08eb
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299636"
---
# <a name="key-concepts-in-direct-line-api-11"></a>Direct Line API 1.1 の主要な概念

Direct Line API を使用すると、ボットと独自のクライアント アプリケーション間の通信を有効にできます。 

> [!IMPORTANT]
> この記事では、Direct Line API 1.1 の主要概念を紹介し、関連する開発者リソースに関する情報を提供します。 クライアント アプリケーションとボットの間の新しい接続を作成する場合は、代わりに [Direct Line API 3.0](bot-framework-rest-direct-line-3-0-concepts.md) を使用します。

## <a name="authentication"></a>認証

Direct Line API 1.1 の要求は、<a href="https://dev.botframework.com/" target="_blank">Bot Framework ポータル</a>の Direct Line チャネル構成ページから取得する**シークレット**を使用するか、実行時に取得する**トークン**を使用して認証できます。  詳細については、[認証](bot-framework-rest-direct-line-1-1-authentication.md)に関するページをご覧ください。

## <a name="starting-a-conversation"></a>会話の開始

Direct Line の会話はクライアントによって明示的に開かれ、ボットとクライアントが参加し、有効な資格情報を持っている限り続きます。 詳細については、「[会話の開始](bot-framework-rest-direct-line-1-1-start-conversation.md)」を参照してください。

## <a name="sending-messages"></a>メッセージの送信

Direct Line API 1.1 を使用すると、クライアントは `HTTP POST` 要求を発行することでメッセージを送信できます。 クライアントは、要求ごとに 1 つのメッセージを送信できます。 詳細については、「[ボットにアクティビティを送信する](bot-framework-rest-direct-line-1-1-send-message.md)」を参照してください。

## <a name="receiving-messages"></a>メッセージの受信

クライアントは Direct Line API 1.1 を使用し、`HTTP GET` 要求でポーリングしてメッセージを受信することができます。 各要求に応じて、クライアントは `MessageSet` の一部としてボットから複数のメッセージを受信することができます。 詳細については、「[Receive messages from the bot](bot-framework-rest-direct-line-1-1-receive-messages.md)」(ボットからメッセージを受信する) を参照してください。

## <a name="developer-resources"></a>開発者リソース

### <a name="client-library"></a>クライアント ライブラリ

Bot Framework では、C# からの Direct Line API 1.1 へのアクセスを容易にするクライアント ライブラリが提供されています。 Visual Studio プロジェクト内でクライアント ライブラリを使用するには、`Microsoft.Bot.Connector.DirectLine` <a href="https://www.nuget.org/packages/Microsoft.Bot.Connector.DirectLine/1.1.1" target="_blank">v1.x NuGet パッケージ</a>をインストールしてください。 

C# のクライアント ライブラリを使用する代わりに、<a href="https://docs.botframework.com/restapi/directline/swagger.json" target="_blank">Direct Line API 1.1 Swagger ファイル</a>を使用して好みの言語で独自のクライアント ライブラリを生成できます。

### <a name="web-chat-control"></a>Web チャット コントロール 

Bot Framework では、Direct Line によって強化されたボットをクライアント アプリケーションに埋め込めるようにするコントロールが提供されています。 詳細については、<a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Microsoft Bot Framework WebChat コントロール</a>に関するページを参照してください。
