---
title: Bot Builder SDK for Node.js のサンプル ボット | Microsoft Docs
description: Bot Builder SDK for Node.js を使用してボット開発を開始する際に役立つさまざまなサンプル ボットについて説明します。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 43c178c4bbdf0bb04384bb8ada397066e6f7dd12
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574618"
---
# <a name="bot-builder-sdk-for-nodejs-samples"></a>Bot Builder SDK for Node.js のサンプル

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

これらのサンプルでは、Bot Builder SDK for Node.js 内の機能を活用する方法を示す、タスクに焦点をおいたボットを説明します。 サンプルを利用することで、豊富な機能を備えた優れたボットの作成が迅速に開始できるようになります。

## <a name="get-the-samples"></a>サンプルの入手
サンプルを入手するには、Git を使用して [BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) GitHub リポジトリを複製します。

```cmd
git clone https://github.com/Microsoft/BotBuilder-Samples.git
cd BotBuilder-Samples
```

Bot Builder SDK for Node.js を使用して作成されたサンプル ボットは、**Node** ディレクトリにまとめられています。

サンプルを GitHub で表示し、直接 Azure にデプロイすることもできます。

## <a name="core"></a>コア
これらのサンプルでは、豊富で有効なボットを作成するための基本的な手法を説明します。

サンプル | 説明
------------ | ------------- 
[添付ファイルを送信する](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-SendAttachment) | このサンプル ボットは、簡単なメディア添付ファイル (イメージ) をユーザーに送信します。 
[添付ファイルを受信する](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-ReceiveAttachment) | このサンプル ボットは、ユーザーによって送信された添付ファイルを受信してダウンロードします。 
[新しい会話を作成する](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-CreateNewConversation)  | このサンプル ボットは、以前に格納済みユーザーのアドレスを使用して新しい会話を開始します。
[会話のメンバーを取得する](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-GetConversationMembers) | このサンプル ボットは、会話のメンバー リストを取得し、変更されているかどうかを検出します。 
[Direct Line](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-DirectLine) | このサンプル ボットとカスタム クライアントは、Direct Line API を使用して相互に通信します。 
[Direct Line (WebSocket)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-DirectLineWebSockets) | このサンプル ボットとカスタム クライアントは、Direct Line API と WebSocket を使用して相互に通信します。 
[複数のダイアログ](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-MultiDialogs) | このサンプル ボットは、さまざまな種類のダイアログを表示します。
[State API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-State) | このステートレス サンプル ボットは、会話のコンテキストを追跡します。
[Custom State API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-CustomState) | このステートレス サンプル ボットは、カスタム ストレージ プロバイダーを使用して会話のコンテキストを追跡します。
[ChannelData](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-ChannelData) | このサンプル ボットは、ChannelData を使用してネイティブ メタデータを Facebook に送信します。
[AppInsights](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-AppInsights) | このサンプル ボットは、利用統計情報を Application Insights インスタンスに記録します。

## <a name="search"></a>Search
このサンプルでは、データ ドリブン ボットでの Azure Search の利用方法を示します。

サンプル | 説明
------------ | -------------
[Azure Search](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-Search) | この 2 つのサンプル ボットは、大量のコンテンツ内をユーザーが移動するのに役立ちます。


## <a name="cards"></a>カード
これらのサンプルでは、Bot Framework でリッチなカードを送信する方法を示します。

サンプル | 説明
------------ | -------------
[リッチ カード](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/cards-RichCards) | このサンプル ボットは、複数の異なる種類のリッチなカードを送信します。
[カードのカルーセル](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/cards-CarouselCards) | このサンプル ボットは、カルーセル レイアウトを使用して 1 つのメッセージ内で複数のリッチなカードを送信します。

## <a name="intelligence"></a>Intelligence
これらのサンプルでは、Bing と Microsoft Cognitive Services APIs を使用してボットに人工知能機能を追加する方法を示します。

サンプル | 説明
------------ | -------------
[LUIS](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-LUIS) | このサンプル ボットは、LuisDialog を使用して LUIS.ai アプリケーションと統合します。
[イメージ キャプション](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-ImageCaption) | このサンプル ボットは、Microsoft Cognitive Services Vision API を使用してイメージのキャプションを取得します。
[音声テキスト変換](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-SpeechToText)  | このサンプル ボットは、Bing Speech API を使用して音声からテキストを取得します。
[類似製品](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-SimilarProducts) | このサンプル ボットは、Bing Image Search API を使用して見た目が似た製品を検索します。 
[Zummer](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-Zummer) | このサンプル ボットは、Bing Search API を使用して、Wikipedia の記事を検索します。

## <a name="reference-implementation"></a>リファレンス実装
このサンプルは、エンド ツー エンドのシナリオを紹介するように設計されています。 ボットに複雑な機能を実装する場合の、優れたコード フラグメントのソースです。


サンプル | 説明
------------ | -------------
[Contoso の花](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-ContosoFlowers) | このサンプル ボットは、Bot Framework の多くの機能を使用します。

