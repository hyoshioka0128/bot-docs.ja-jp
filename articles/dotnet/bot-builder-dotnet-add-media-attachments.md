---
title: メッセージへのメディア添付ファイルの追加 (v3 C#) - Bot Service
description: Bot Framework SDK for .NET を使用してメッセージにメディア添付ファイルを追加する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: ad6a1228b8fc54f8f626c07c7ed43375c249c456
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75796020"
---
# <a name="add-media-attachments-to-messages"></a>メッセージへのメディア添付ファイルの追加

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

ユーザーとボットの間のメッセージ交換には、メディア添付ファイル (イメージ、ビデオ、オーディオ、ファイルなど) を含めることができます。 

<a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity</a> オブジェクトの `Attachments` プロパティには、メッセージ内のメディア添付ファイルやリッチ カードを表す <a href="https://docs.microsoft.com/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Attachment</a> オブジェクトの配列が格納されます。 

> [!NOTE]
> 「[Add rich cards to messages](bot-builder-dotnet-add-rich-card-attachments.md)」 (メッセージへのリッチ カードの追加)。

## <a name="add-a-media-attachment"></a>メディア添付ファイルの追加  

メディア添付ファイルをメッセージに追加するには、`message` アクティビティ用の `Attachment` オブジェクトを作成し、`ContentType`、`ContentUrl`、`Name` の各プロパティを設定します。 

[!code-csharp[Add media attachment](../includes/code/dotnet-add-attachments.cs#addMediaAttachment)]

添付ファイルがイメージ、オーディオ、またはビデオの場合、コネクタ サービスにより、[チャネル](bot-builder-dotnet-channeldata.md)での会話内の添付ファイルがレンダリングできる方法で、添付ファイルのデータがチャネルに通信されます。 添付ファイルがファイルである場合は、ファイルの URL が会話内にハイパーリンクとしてレンダリングされます。

## <a name="additional-resources"></a>その他のリソース

- [チャネル リファレンス][inspector]
- [アクティビティの概要](bot-builder-dotnet-activities.md)
- [メッセージの作成](bot-builder-dotnet-create-messages.md)
- [メッセージへのリッチ カードの追加](bot-builder-dotnet-add-rich-card-attachments.md)
- <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity クラス</a>
- <a href="https://docs.microsoft.com/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Attachment クラス</a>

[inspector]: ../bot-service-channels-reference.md

