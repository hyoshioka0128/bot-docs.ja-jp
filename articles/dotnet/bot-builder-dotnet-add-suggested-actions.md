---
title: メッセージへの推奨されるアクションの追加 | Microsoft Docs
description: Bot Builder SDK for .NET を使用して、推奨されるアクションをメッセージに追加する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/13/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 63e5f4b8ac86b6b0e29d09796dbe74295bf3e213
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301486"
---
# <a name="add-suggested-actions-to-messages"></a>メッセージへの推奨されるアクションの追加
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

> [!TIP]
> さまざまなチャネルにおける推奨されるアクションの外観と動作を確認するには、[Channel Inspector][channelInspector] を使用します。

## <a name="send-suggested-actions"></a>推奨されるアクションの送信

推奨されるアクションをメッセージに追加するには、アクティビティの `SuggestedActions` プロパティを、ユーザーに表示されるボタンを表す [CardAction][cardAction] オブジェクトのリストに設定します。 

次のコード例は、3 つの推奨されるアクションをユーザーに提示するメッセージの作成方法を示しています。

[!code-csharp[Add suggested actions](../includes/code/dotnet-add-suggested-actions.cs#addSuggestedActions)]

ユーザーが推奨されるアクションのいずれかをタップすると、対応するアクションの `Value` を含むメッセージがユーザーからボットに送信されます。

## <a name="additional-resources"></a>その他のリソース

- [Channel Inspector を使用して機能をプレビューする][inspector]
- [アクティビティの概要](bot-builder-dotnet-activities.md)
- [メッセージの作成](bot-builder-dotnet-create-messages.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity クラス</a>
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">IMessageActivity インターフェイス</a>
- <a href="/dotnet/api/microsoft.bot.connector.cardaction" target="_blank">CardAction クラス</a>
- <a href="/dotnet/api/microsoft.bot.connector.suggestedactions" target="_blank">SuggestedActions クラス</a>

[cardAction]: /dotnet/api/microsoft.bot.connector.cardaction

[inspector]: ../bot-service-channel-inspector.md

[channelInspector]: ../bot-service-channel-inspector.md


