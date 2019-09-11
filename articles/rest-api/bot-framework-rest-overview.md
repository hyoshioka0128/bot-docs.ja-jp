---
title: Bot Framework REST API | Microsoft Docs
description: ボットとボットに接続するクライアントを作成するために使用可能な Bot Framework REST API の概要について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 9f9acf2dc2b1aa0981196bf316a3f1c351467d32
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299537"
---
# <a name="bot-framework-rest-apis"></a>Bot Framework REST API
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

Bot Framework REST API を使用すると、<a href="https://dev.botframework.com/" target="_blank">Bot Framework Portal</a> で構成されたチャネルを使ってメッセージをやり取りするボットを作成し、状態データを格納および取得し、固有のクライアント アプリケーションをボットに接続することが可能になります。 すべての Bot Framework サービスが、HTTPS 経由で業界標準の REST および JSON を使用します。

## <a name="build-a-bot"></a>ボットを作成する

Bot Connector サービスを使用して任意のプログラミング言語でボットを作成し、Bot Framework Portal で構成されたチャネルを使ってメッセージをやり取りできます。 

> [!TIP]
> Bot Framework には、C# または Node.js のいずれかでボットをビルドするのに使用できるクライアント ライブラリが用意されています。 C# を使用してボットをビルドするには、[Bot Framework SDK for C#](../dotnet/bot-builder-dotnet-overview.md) をご利用ください。 Node.js を使用してボッドをビルドするには、[Bot Framework SDK for Node.js](../nodejs/index.md) をご利用ください。 

Bot Connector サービスを使用したボットの作成の詳細については、「[Key concepts](bot-framework-rest-connector-concepts.md)」 (重要な概念) を参照してください。

## <a name="build-a-client"></a>クライアントを構築する

Direct Line API を使用することによって、独自のクライアント アプリケーションとボットの通信を有効にできます。 Direct Line API では、標準のシークレット/トークン パターンを使用する認証メカニズムを実装し、ボットにプロトコル バージョンの変更が生じた場合でも、安定したスキーマを提供します。 Direct Line API を使用したクライアントとボット間の通信の有効化に関する詳細については、「[Key concepts](bot-framework-rest-direct-line-3-0-concepts.md)」(重要な概念) を参照してください。 