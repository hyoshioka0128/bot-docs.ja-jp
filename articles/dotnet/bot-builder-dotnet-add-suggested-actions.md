---
title: メッセージへの推奨されるアクションの追加 (v3 C#) - Bot Service
description: Bot Framework SDK for .NET を使用して、推奨されるアクションをメッセージに追加する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 03/13/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 61aaff340a235f7b9b552d79205fab8e772216f9
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75796684"
---
# <a name="add-suggested-actions-to-messages"></a>メッセージへの推奨されるアクションの追加

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

## <a name="send-suggested-actions"></a>推奨されるアクションの送信

推奨されるアクションをメッセージに追加するには、アクティビティの `SuggestedActions` プロパティを、ユーザーに表示されるボタンを表す [CardAction][cardAction] オブジェクトのリストに設定します。 

次のコード例は、3 つの推奨されるアクションをユーザーに提示するメッセージの作成方法を示しています。

[!code-csharp[Add suggested actions](../includes/code/dotnet-add-suggested-actions.cs#addSuggestedActions)]

ユーザーが推奨されるアクションのいずれかをタップすると、対応するアクションの `Value` を含むメッセージがユーザーからボットに送信されます。

## <a name="additional-resources"></a>その他のリソース

- [アクティビティの概要](bot-builder-dotnet-activities.md)
- [メッセージの作成](bot-builder-dotnet-create-messages.md)
- [Activity クラス](https://aka.ms/ActivityClass-dotnet-API)
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">IMessageActivity インターフェイス</a>
- <a href="/dotnet/api/microsoft.bot.connector.cardaction" target="_blank">CardAction クラス</a>
- <a href="/dotnet/api/microsoft.bot.connector.suggestedactions" target="_blank">SuggestedActions クラス</a>

[cardAction]: /dotnet/api/microsoft.bot.connector.cardaction

[inspector]: ../bot-service-channel-inspector.md


