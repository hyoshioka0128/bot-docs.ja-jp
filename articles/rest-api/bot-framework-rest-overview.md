---
title: Bot Framework REST API - Bot Service
description: ボットとボットに接続するクライアントを作成するために使用可能な Bot Framework REST API の概要について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/20/2020
ms.openlocfilehash: 60a5e08caf11977eed72bfba0dfeba3472194822
ms.sourcegitcommit: 772b9278d95e4b6dd4afccf4a9803f11a4b09e42
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2020
ms.locfileid: "80117452"
---
# <a name="bot-framework-rest-apis"></a>Bot Framework REST API

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.js](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

ほとんどの Bot Framework ボットは、Bot Framework SDK を使用して構築されます。これは、ボットを準備してすべての会話を処理します。 SDK を使用する代わりに、REST API を使用してボットにメッセージを直接送信することもできます。

## <a name="build-a-bot"></a>ボットを作成する

Bot Framework REST API にコーディングすることで、ボットの Azure Bot Service 登録に構成されたチャンネルで、メッセージをユーザーと送受信できます。

> [!TIP]
> Bot Framework には、C# または Node.js のいずれかでボットをビルドするのに使用できるクライアント ライブラリが用意されています。
> C# を使用してボットをビルドするには、[Bot Framework SDK for C#](../dotnet/bot-builder-dotnet-overview.md) をご利用ください。
> Node.js を使用してボッドをビルドするには、[Bot Framework SDK for Node.js](../nodejs/index.md) をご利用ください。

サービスを使用したボットの作成の詳細については、[Azure Bot Service](../bot-service-overview-introduction.md) に関する記事を参照してください。

## <a name="build-a-direct-line-or-web-chat-client"></a>Direct Line または Web チャットのクライアントを作成する

Facebook、Teams、または Slack などのほとんどのチャンネルではクライアントが提供されますが、Direct Line や Web チャットを使用すると、独自のクライアント アプリケーションでボットと通信できるようになります。 Direct Line API では、標準のシークレット/トークン パターンを使用する認証メカニズムを実装し、ボットにプロトコル バージョンの変更が生じた場合でも、安定したスキーマを提供します。 Direct Line API を使用したクライアントとボット間の通信の有効化に関する詳細については、「[Key concepts](bot-framework-rest-direct-line-3-0-concepts.md)」(重要な概念) を参照してください。

Direct Line と Web チャットのクライアントは、様々な言語とデプロイ (サービスではなく、アプリなど) で使用できます。 詳細については、「[Direct Line の概要](https://docs.microsoft.com/azure/bot-service/bot-service-channel-directline?view=azure-bot-service-4.0)」を参照してください。
