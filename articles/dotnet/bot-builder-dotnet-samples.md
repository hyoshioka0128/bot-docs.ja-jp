---
title: Bot Builder SDK for .NET のサンプル ボット | Microsoft Docs
description: Bot Builder SDK for .NET を使用してボット開発を開始する際に役立つサンプル ボットについて説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/03/2018
ms.openlocfilehash: 7aff56dfc60d9d5cce42a5b6a2624c1364ff1b72
ms.sourcegitcommit: 44f100a588ffda19c275b118f4f97029f12d1449
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/26/2018
ms.locfileid: "42928380"
---
# <a name="bot-builder-sdk-for-net-samples"></a>Bot Builder SDK for .NET のサンプル

::: moniker range="azure-bot-service-3.0"

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

これらのサンプルでは、Bot Builder SDK for .NET 内の機能を活用する方法を示す、タスクに焦点をおいたボットを説明します。 サンプルを利用することで、豊富な機能を備えた優れたボットの作成が迅速に開始できるようになります。

## <a name="get-the-samples"></a>サンプルの入手
サンプルを入手するには、Git を使用して [BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) GitHub リポジトリを複製します。

```cmd
git clone https://github.com/Microsoft/BotBuilder-Samples.git
cd BotBuilder-Samples
```

Bot Builder SDK for .NET を使用して作成されたサンプル ボットは、**CSharp** ディレクトリにまとめられています。

サンプルを GitHub で表示し、直接 Azure にデプロイすることもできます。

## <a name="core"></a>コア
これらのサンプルでは、豊富で有効なボットを作成するための基本的な手法を説明します。

サンプル | 説明
------------ | ------------- 
[添付ファイルを送信する](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-SendAttachment) | このサンプル ボットは、簡単なメディア添付ファイル (イメージ) をユーザーに送信します。 
[添付ファイルを受信する](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-ReceiveAttachment) | このサンプル ボットは、ユーザーによって送信された添付ファイルを受信してダウンロードします。 
[新しい会話を作成する](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-CreateNewConversation)  | このサンプル ボットは、以前に格納済みユーザーのアドレスを使用して新しい会話を開始します。
[会話のメンバーを取得する](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GetConversationMembers) | このサンプル ボットは、会話のメンバー リストを取得し、変更されているかどうかを検出します。 
[Direct Line](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-DirectLine) | このサンプル ボットとカスタム クライアントは、Direct Line API を使用して相互に通信します。 
[Direct Line (WebSocket)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-DirectLineWebSockets) | このサンプル ボットとカスタム クライアントは、Direct Line API と WebSocket を使用して相互に通信します。 
[複数のダイアログ](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-MultiDialogs) | このサンプル ボットは、さまざまな種類のダイアログを表示します。
[State API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-State) | このステートレス サンプル ボットは、会話のコンテキストを追跡します。
[Custom State API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-CustomState) | このステートレス サンプル ボットは、カスタム ストレージ プロバイダーを使用して会話のコンテキストを追跡します。
[ChannelData](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-ChannelData) | このサンプル ボットは、ChannelData を使用してネイティブ メタデータを Facebook に送信します。
[AppInsights](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-AppInsights) | このサンプル ボットは、利用統計情報を Application Insights インスタンスに記録します。

## <a name="search"></a>Search
このサンプルでは、データ ドリブン ボットでの Azure Search の利用方法を示します。

サンプル | 説明
------------ | -------------
[Azure Search](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search) | この 2 つのサンプル ボットは、大量のコンテンツ内をユーザーが移動するのに役立ちます。


## <a name="cards"></a>カード
これらのサンプルでは、Bot Framework でリッチなカードを送信する方法を示します。

サンプル | 説明
------------ | -------------
[リッチ カード](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/cards-RichCards) | このサンプル ボットは、複数の異なる種類のリッチなカードを送信します。
[カードのカルーセル](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/cards-CarouselCards) | このサンプル ボットは、カルーセル レイアウトを使用して 1 つのメッセージ内で複数のリッチなカードを送信します。

## <a name="intelligence"></a>Intelligence
これらのサンプルでは、Bing と Microsoft Cognitive Services APIs を使用してボットに人工知能機能を追加する方法を示します。

サンプル | 説明
------------ | -------------
[LUIS](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-LUIS) | このサンプル ボットは、LuisDialog を使用して LUIS.ai アプリケーションと統合します。
[イメージ キャプション](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-ImageCaption) | このサンプル ボットは、Microsoft Cognitive Services Vision API を使用してイメージのキャプションを取得します。
[音声テキスト変換](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-SpeechToText)  | このサンプル ボットは、Bing Speech API を使用して音声からテキストを取得します。
[類似製品](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-SimilarProducts) | このサンプル ボットは、Bing Image Search API を使用して見た目が似た製品を検索します。 
[Zummer](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-Zummer) | このサンプル ボットは、Bing Search API を使用して、Wikipedia の記事を検索します。

## <a name="reference-implementation"></a>リファレンス実装
このサンプルは、エンド ツー エンドのシナリオを紹介するように設計されています。 ボットに複雑な機能を実装する場合の、優れたコード フラグメントのソースです。


サンプル | 説明
------------ | -------------
[Contoso の花](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-ContosoFlowers) | このサンプル ボットは、Bot Framework の多くの機能を使用します。

::: moniker-end

::: moniker range="azure-bot-service-4.0"

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

これらのサンプルでは、Bot Builder SDK v4 for .NET 内の機能を活用する方法を示す、タスクに焦点をおいたボットについて説明します。 サンプルを利用することで、豊富な機能を備えた優れたボットの作成が迅速に開始できるようになります。 

注: SDK v4 については精力的に開発を進めているところであるため、実験目的でのみのご使用となります。 

サンプルを入手するには、Git を使用して [botbuilder-dotnet](https://github.com/Microsoft/botbuilder-dotnet) GitHub リポジトリを複製します。
```cmd
git clone https://github.com/Microsoft/botbuilder-dotnet.git
cd botbuilder-dotnet\samples-final
```
Bot Builder SDK for .NET を使用して作成されたサンプル ボットは、**samples-final** ディレクトリにまとめられています。


::: moniker-end

