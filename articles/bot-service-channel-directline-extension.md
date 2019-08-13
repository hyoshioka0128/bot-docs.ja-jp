---
title: Direct Line App Service 拡張機能
titleSuffix: Bot Service
description: Direct Line App Service 拡張機能のフィーチャー
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 07/09/2019
ms.openlocfilehash: 27067cf2582de63cf67785cfaaa70b9a33685894
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/03/2019
ms.locfileid: "68757764"
---
## <a name="direct-line-app-service-extension"></a>Direct Line App Service 拡張機能

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

Direct Line App Service 拡張機能は、お客様が追加できる **BotAdapter** を介してボットに接続するための**永続的名前付きパイプ**のセットを確立します。

それにより、新しいストリーミング拡張機能セットが Bot Framework プロトコルに追加されます。 これらの拡張機能により、メッセージを交換するためのトランスポートとしての HTTP は、**永続的 WebSocket** での双方向要求の送信を許可するトランスポートに置き換えられます。 これにより、パフォーマンスが向上し、情報交換中の分離性が高まります。

ストリーミング拡張機能が追加される前は、Direct Line API では、クライアントが Activity を Direct Line に送信するための 1 つの方法と、クライアントが Direct Line から Activity を取得するための 2 つの方法が提供されていました。 メッセージは、HTTP POST 経由で送信され、HTTP GET (ポーリング) を使用するか、ActivitySet を受信するための WebSocket を開くことで受信されていました。
ストリーミング拡張機能により、WebSocket の使用が拡張され、**すべてのメッセージング通信**を WebSocket で送信できます。 ストリーミング拡張機能は、チャネル サービスとボット間でも使用できます。


Direct Line App Service 拡張機能は、世界中のすべてのデータ センターで Azure App Service のすべてのインスタンスにプレインストールされています。 その維持管理は Microsoft が行っており、お客様が追加のデプロイ作業を行う必要はありません。
これは、既定では Azure App Service で無効になっていますが、簡単に有効にして、ホストされているボットに接続できます。

次の図は、全体のアーキテクチャを示しています。

![Direct Line 拡張機能のアーキテクチャ](./media/channels/direct-line-extension-architecture.png)

## <a name="see-also"></a>関連項目

|EnableAdfsAuthentication|説明|
|---|---|
|[拡張機能を使用した .NET ボット](bot-service-channel-directline-extension-net-bot.md)|**名前付きパイプ**を使って機能するようにボットを更新し、ボットがホストされている **Azure App Service** リソースで Direct Line App Service 拡張機能を有効にします。  |
|[.NET クライアントと拡張機能](bot-service-channel-directline-extension-net-client.md)|Direct Line App Service 拡張機能に接続する .NET クライアントを C# で作成します|
|[拡張機能を使用した Web チャット](bot-service-channel-directline-extension-webchat-client.md)|WebChat を Direct Line App Service 拡張機能と共に使用します。|
|[VNET 内で拡張機能を使用する](bot-service-channel-directline-extension-vnet.md)|Azure Virtual Network (VNET) で Direct Line App Service 拡張機能を使用します|

## <a name="addtional-resources"></a>その他のリソース

- [ボットを Direct Line に接続する](bot-service-channel-connect-directline.md)
- [ボットを Direct Line Speech に接続する](bot-service-channel-connect-directlinespeech.md)
