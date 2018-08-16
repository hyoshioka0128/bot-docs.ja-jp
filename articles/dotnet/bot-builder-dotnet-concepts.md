---
title: Bot Builder SDK for .NET の主要概念 | Microsoft Docs
description: Bot Builder SDK for .NET で利用可能な会話型ボットを構築および展開するための主要概念とツールについて説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0a8c64cced0cb6c10daeedee4edfd777ff6d8e78
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300900"
---
# <a name="key-concepts-in-the-bot-builder-sdk-for-net"></a>Bot Builder SDK for .NET の主要概念
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-concepts.md)
> - [Node.js](../nodejs/bot-builder-nodejs-concepts.md)

この記事では、Bot Builder SDK for .NET の主要な概念について説明します。

## <a name="connector"></a>コネクタ

[Bot Framework Connector](bot-builder-dotnet-connector.md) では、ボットで Skype、電子メール、Slack などの複数のチャンネルとの間で通信できるようにする単一の REST API が用意されています。 これにより、ボットからチャンネルおよびチャンネルからボットにメッセージを返信することで、ボットとユーザー間の通信が容易になります。 

Bot Builder SDK for .NET では、[Connector][connectorLibrary] ライブラリを使用してコネクタにアクセスできます。 

## <a name="activity"></a>アクティビティ

[!INCLUDE [Activity concept overview](../includes/snippet-dotnet-concept-activity.md)]

Bot Builder SDK for .NET 内のアクティビティの詳細については、「[Activities overview](bot-builder-dotnet-activities.md)」(アクティビティの概要) を参照してください。

## <a name="dialog"></a>ダイアログ

Bot Builder SDK for .NET を使用してボットを作成する場合、[ダイアログ](bot-builder-dotnet-dialogs.md)を使用して会話をモデル化し、[会話フロー](../bot-service-design-conversation-flow.md#dialog-stack)を管理できます。 ダイアログは、他のダイアログと組み合わせて最大限に再利用することができます。ダイアログ コンテキストは、任意の時点における会話内のアクティブな[ダイアログのスタック](../bot-service-design-conversation-flow.md)を保持します。 ダイアログを構成する会話はコンピューター間で移植可能です。これにより、ボットの実装をスケーリングすることができます。 

Bot Builder SDK for .NET では、[Builder][builderLibrary] ライブラリを使用してダイアログを管理できます。

## <a name="formflow"></a>FormFlow

Bot Builder SDK for .NET 内の [FormFlow](bot-builder-dotnet-formflow.md) を使用すると、ユーザーから情報を収集するボットを効率的に構築できます。 たとえば、サンドイッチの注文を取るボットは、パンの種類、トッピングの選択、サイズなど、ユーザーから情報をいくつか収集する必要があります。 基本的なガイドラインを指定すると、このようなガイド付きの会話を管理するために必要なダイアログを FormFlow で自動的に生成することができます。

## <a name="state"></a>State

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

Bot Builder SDK for .NET を使用して状態を管理する方法の詳細については、「[Manage state data](bot-builder-dotnet-state.md)」(状態データの管理) を参照してください。

## <a name="naming-conventions"></a>名前付け規則

Bot Builder SDK for .NET ライブラリは、厳密に型指定されたパスカルケースの名前付け規則を使用します。 ただし、ネットワーク経由でやり取りされる JSON メッセージは、キャメルケースの名前付け規則を使用します。 たとえば、C# プロパティ **ReplyToId** は、ネットワーク経由で転送される JSON メッセージでは **replyToId** としてシリアル化されます。

## <a name="security"></a>セキュリティ

ボットのエンドポイントは、Bot Framework Connector サービスから呼び出す必要があります。 このトピックの詳細については、「[Secure your bot](bot-builder-dotnet-security.md)」(ボットをセキュリティで保護する) を参照してください。

## <a name="next-steps"></a>次の手順

ここでは、すべてのボットの背後にある概念について学びました。 テンプレートを使用すると、[Visual Studio で簡単にボットを構築](bot-builder-dotnet-quickstart.md)できます。 次に、個々の主要概念について詳細に説明します。まずダイアログから始めます。

> [!div class="nextstepaction"]
> [Bot Builder SDK for .NET のダイアログ](bot-builder-dotnet-dialogs.md)

[connectorLibrary]: /dotnet/api/microsoft.bot.connector

[builderLibrary]: /dotnet/api/microsoft.bot.builder.dialogs
