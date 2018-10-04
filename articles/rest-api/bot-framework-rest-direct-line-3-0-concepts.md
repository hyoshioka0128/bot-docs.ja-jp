---
title: Bot Framework Direct Line API 3.0 の主要な概念 | Microsoft Docs
description: Bot Framework Direct Line API 3.0 の主要な概念を理解します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/28/2018
ms.openlocfilehash: 3e9756f08690820950d0f6d0b8128521cb94f60b
ms.sourcegitcommit: d4afc924b0e1907c4d6f7a6fc5ac1fe521aeef7e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/28/2018
ms.locfileid: "47447377"
---
# <a name="key-concepts-in-direct-line-api-30"></a>Direct Line API 3.0 の主要な概念

Direct Line API を使用すると、ボットと独自のクライアント アプリケーション間の通信を有効にできます。 この記事では、Direct Line API 3.0 の主要概念を紹介し、関連する開発者リソースに関する情報を提供します。

## <a name="authentication"></a>Authentication

Direct Line API 3.0 の要求は、<a href="https://dev.botframework.com/" target="_blank">Bot Framework ポータル</a>の Direct Line チャネル構成ページから取得する**シークレット**を使用するか、実行時に取得する**トークン**を使用して認証できます。 詳細については、[認証](bot-framework-rest-direct-line-3-0-authentication.md)に関するページをご覧ください。

## <a name="starting-a-conversation"></a>会話の開始

Direct Line の会話はクライアントによって明示的に開かれ、ボットとクライアントが参加し、有効な資格情報を持っている限り続きます。 詳細については、「[会話の開始](bot-framework-rest-direct-line-3-0-start-conversation.md)」を参照してください。

## <a name="sending-messages"></a>メッセージの送信

Direct Line API 3.0 を使用すると、クライアントは `HTTP POST` 要求を発行することでメッセージを送信できます。 クライアントは、要求ごとに 1 つのメッセージを送信できます。 詳しくは、「[ボットにアクティビティを送信する](bot-framework-rest-direct-line-3-0-send-activity.md)」をご覧ください。

## <a name="receiving-messages"></a>メッセージの受信

Direct Line API 3.0 を使用すると、クライアントは、`WebSocket` ストリームを通して、または `HTTP GET` 要求を発行してボットからメッセージを受信できます。 これらの手法のいずれかを使用することによって、クライアントは、`ActivitySet` の一部としてボットから一度に複数のメッセージを受信できます。 詳細については、「[ボットからアクティビティを受信する](bot-framework-rest-direct-line-3-0-receive-activities.md)」を参照してください。

## <a name="developer-resources"></a>開発者リソース

### <a name="client-libraries"></a>クライアント ライブラリ

Bot Framework では、C# と Node.js からの Direct Line API 3.0 へのアクセスを容易にするクライアント ライブラリが提供されています。 

- Visual Studio プロジェクト内で .NET クライアント ライブラリを使用するには、`Microsoft.Bot.Connector.DirectLine` <a href="https://www.nuget.org/packages/Microsoft.Bot.Connector.DirectLine" target="_blank">NuGet パッケージ</a>をインストールしてください。 

- Node.js クライアント ライブラリを使用するには、<a href="https://www.npmjs.com/package/botframework-directlinejs" target="_blank">NPM</a> を使用して `botframework-directlinejs` ライブラリをインストール (またはソースを<a href="https://github.com/Microsoft/BotFramework-DirectLineJS" target="_blank">ダウンロード</a>) してください。

C# または Node.js のクライアント ライブラリを使用する代わりに、<a href="https://docs.botframework.com/en-us/restapi/directline3/swagger.json" target="_blank">Direct Line API 3.0 Swagger ファイル</a>を使用して好みの言語で独自のクライアント ライブラリを生成できます。

::: moniker range="azure-bot-service-3.0"

### <a name="sample-code"></a>サンプル コード

<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples" target="_blank">BotBuilder-Samples</a> GitHub リポジトリには、C# と Node.js で Direct Line API 3.0 を使用する方法を示す複数のサンプルが含まれています。

| サンプル | Language | 説明 |
|----|----|----|
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-DirectLine" target="_blank">Direct Line Bot のサンプル</a> | C#の場合 | Direct Line API を使用して相互に通信するサンプル ボットとカスタム クライアント。 |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-DirectLineWebSockets" target="_blank">Direct Line Bot のサンプル (クライアント WebSocket を使用)</a> | C#の場合 | Direct Line API と WebSocket を使用して相互に通信するサンプル ボットとカスタム クライアント。 |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/Node/core-DirectLine" target="_blank">Direct Line Bot のサンプル</a> | JavaScript | Direct Line API を使用して相互に通信するサンプル ボットとカスタム クライアント。 |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/Node/core-DirectLineWebSockets" target="_blank">Direct Line Bot のサンプル (クライアント WebSocket を使用)</a> | JavaScript | Direct Line API と WebSocket を使用して相互に通信するサンプル ボットとカスタム クライアント。 |

::: moniker-end

### <a name="web-chat-control"></a>Web チャット コントロール 

Bot Framework では、Direct Line によって強化されたボットをクライアント アプリケーションに埋め込めるようにするコントロールが提供されています。 詳細については、<a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Microsoft Bot Framework WebChat コントロール</a>に関するページを参照してください。
