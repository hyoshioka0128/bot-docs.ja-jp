---
title: Bot Framework SDK for .NET の主要概念 | Microsoft Docs
description: Bot Framework SDK for .NET で利用可能な会話型ボットを構築および展開するための主要概念とツールについて説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0b0493d9975e58dda0f2195c03d887e4681ca0bf
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298387"
---
# <a name="key-concepts-in-the-bot-framework-sdk-for-net"></a>Bot Framework SDK for .NET の主要概念

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-concepts.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-concepts.md)

この記事では、Bot Framework SDK for .NET の主要な概念について説明します。

## <a name="connector"></a>コネクタ

[Bot Framework Connector](bot-builder-dotnet-connector.md) では、ボットで Skype、電子メール、Slack などの複数のチャンネルとの間で通信できるようにする単一の REST API が用意されています。 これにより、ボットからチャンネルおよびチャンネルからボットにメッセージを返信することで、ボットとユーザー間の通信が容易になります。 

Bot Framework SDK for .NET では、[Connector][connectorLibrary] ライブラリを使用してコネクタにアクセスできます。 

## <a name="activity"></a>アクティビティ

[!INCLUDE [Activity concept overview](../includes/snippet-dotnet-concept-activity.md)]

Bot Framework SDK for .NET 内のアクティビティの詳細については、「[アクティビティの概要](bot-builder-dotnet-activities.md)」を参照してください。

## <a name="dialog"></a>ダイアログ

Bot Framework SDK for .NET を使用してボットを作成する場合、[ダイアログ](bot-builder-dotnet-dialogs.md)を使用して会話をモデル化し、[会話フロー](../bot-service-design-conversation-flow.md#dialog-stack)を管理できます。 ダイアログは、他のダイアログと組み合わせて最大限に再利用することができます。ダイアログ コンテキストは、任意の時点における会話内のアクティブな[ダイアログのスタック](../bot-service-design-conversation-flow.md)を保持します。 ダイアログを構成する会話はコンピューター間で移植可能です。これにより、ボットの実装をスケーリングすることができます。 

Bot Framework SDK for .NET では、[Builder][builderLibrary] ライブラリを使用してダイアログを管理できます。

## <a name="formflow"></a>FormFlow

Bot Framework SDK for .NET 内の [FormFlow](bot-builder-dotnet-formflow.md) を使用すると、ユーザーから情報を収集するボットを効率的に構築できます。 たとえば、サンドイッチの注文を取るボットは、パンの種類、トッピングの選択、サイズなど、ユーザーから情報をいくつか収集する必要があります。 基本的なガイドラインを指定すると、このようなガイド付きの会話を管理するために必要なダイアログを FormFlow で自動的に生成することができます。

## <a name="state"></a>State

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

Bot Framework SDK for .NET を使用して状態を管理する方法の詳細については、「[状態データの管理](bot-builder-dotnet-state.md)」を参照してください。

## <a name="naming-conventions"></a>名前付け規則

Bot Framework SDK for .NET ライブラリでは、厳密に型指定されたパスカルケースの名前付け規則が使用されます。 ただし、ネットワーク経由でやり取りされる JSON メッセージは、キャメルケースの名前付け規則を使用します。 たとえば、C# プロパティ **ReplyToId** は、ネットワーク経由で転送される JSON メッセージでは **replyToId** としてシリアル化されます。

## <a name="security"></a>セキュリティ

ボットのエンドポイントは、Bot Framework Connector サービスから呼び出す必要があります。 このトピックの詳細については、「[Secure your bot](bot-builder-dotnet-security.md)」(ボットをセキュリティで保護する) を参照してください。

## <a name="next-steps"></a>次の手順

ここでは、すべてのボットの背後にある概念について学びました。 テンプレートを使用すると、[Visual Studio で簡単にボットを構築](bot-builder-dotnet-quickstart.md)できます。 次に、個々の主要概念について詳細に説明します。まずダイアログから始めます。

> [!div class="nextstepaction"]
> [Bot Framework SDK for .NET のダイアログ](bot-builder-dotnet-dialogs.md)

[connectorLibrary]: /dotnet/api/microsoft.bot.connector

[builderLibrary]: /dotnet/api/microsoft.bot.builder.dialogs
