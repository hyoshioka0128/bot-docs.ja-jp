---
title: チャネルと Bot Connector サービス | Microsoft Docs
description: Bot Builder SDK の主要な概念。
keywords: アクティビティ、会話
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/28/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: eaf1a8f714ea8711985b732f797951d241abbca2
ms.sourcegitcommit: 44f100a588ffda19c275b118f4f97029f12d1449
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/26/2018
ms.locfileid: "42928300"
---
# <a name="channels-and-the-bot-connector-service"></a>チャネルと Bot Connector サービス

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

チャネルは、Facebook、Skype、電子メール、Slack などの、ユーザーがボットに接続している元のエンドポイントです。[Azure Portal](https://portal.azure.com) 経由で構成される Bot Connector サービスは、ボットをこれらのチャネルに接続し、ボットとユーザーの間の通信を容易にします。 

Bot Connector サービスに付属する標準チャネルに加えて、[ダイレクト ライン](bot-builder-howto-direct-line.md)をチャネルとして使用して、ボットを独自のクライアント アプリケーションに接続することもできます。

Bot Connector サービスを使用すると、ボットがチャネルに送信するメッセージを正規化することによって、チャネルにとらわれない方法でボットを開発できます。 これを行うには、メッセージを Bot Builder スキーマからチャネルのスキーマに変換する必要があります。 ただし、チャネルが Bot Builder スキーマのすべての側面をサポートしているわけではない場合、サービスはメッセージをチャネルがサポートしている形式に変換しようとします。 たとえば、アクション ボタンが含まれたカードを含むメッセージをボットが SMS チャネルに送信した場合、コネクタはそのカードをイメージとして送信し、アクションをそのメッセージのテキスト内にリンクとして含める可能性があります。

## <a name="activities-and-conversations"></a>アクティビティと会話


Bot Connector サービスは JSON を使用してボットとユーザーの間で情報を交換し、Bot Builder SDK はこれらの情報を言語固有のアクティビティ オブジェクト内にラップします。 [アクティビティの種類](../bot-service-activities-entities.md)は、「[ボットとの対話](bot-builder-basics.md#interaction-with-your-bot)」で説明しました。最も一般的なアクティビティの種類はメッセージですが、他のアクティビティの種類も重要です。 これらには、会話の更新、連絡先の関係の更新、ユーザー データの削除、会話の終了、入力、メッセージへの反応のほか、ユーザーにはほとんど表示されないボット固有のその他のいくつかのアクティビティが含まれます。 これらの各アクティビティおよびそれらがいつ発生するかの詳細は、アクティビティのリファレンス コンテンツで見つけることができます。

すべてのターンおよびそれに関連付けられたアクティビティは、1 つ以上のボットと特定のユーザーまたはユーザーのグループの間の対話を表す論理的な**会話**に属します。 会話はチャネルに固有であり、そのチャネルに固有の ID を持っています。

## <a name="next-steps"></a>次の手順

これで、ボットのいくつかの主要な概念が理解できたので、ボットが使用する可能性のある別の形式の会話を詳細に見てみましょう。

> [!div class="nextstepaction"]
> [会話の形式](bot-builder-conversations.md)
