---
title: Bot Connector と Bot State サービスでの主要な概念 - Bot Service
description: Bot Framework の Bot Connector サービスおよび Bot State サービスでの主要な概念について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/20/2020
ms.openlocfilehash: f68451b8bf336a8b4f3bfc026c682338c561aed0
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "77519901"
---
# <a name="key-concepts"></a>主要な概念

Bot Framework と Azure Bot Service を使用すると、ボットはTeams、Cortana、Facebook などでユーザーと通信できます。 チャンネルは、2 つの形式で (Azure Bot Service の一部として含まれているサービスとして、および Bot Framework SDK で使用するアダプター ライブラリとして) 使用できます。 この記事では、Azure Bot Service に含まれる標準化されたチャンネルに重点を置いて説明します。

## <a name="bot-framework-channels"></a>Bot Framework チャネル

Bot Framework チャンネルを使用すると、[Azure portal](https://portal.azure.com) で構成されているチャンネルとご利用のボットとの間でメッセージのやり取りができるようになります。 このサービスでは、HTTPS 経由で業界標準の REST および JSON が使用され、JWT ベアラー トークンによる認証が有効にされます。 Bot Connector サービスを使用する方法の詳細については、「[Authentication](bot-framework-rest-connector-authentication.md)」(認証) と、このセクションの残りの記事を参照してください。

### <a name="activity"></a>アクティビティ

Bot Connector サービスでは、[Activity][Activity] オブジェクトを渡すことによって、ボットとチャネル (ユーザー) 間で情報が交換されます。 最も一般的なアクティビティの種類は**メッセージ**ですが、ボットまたはチャネルにさまざまな種類の情報を通信するために使用できる他のアクティビティの種類も存在します。 Bot Connector 内のアクティビティの詳細については、「[Activities overview](https://aka.ms/botSpecs-activitySchema)」(アクティビティの概要) を参照してください。

## <a name="authentication"></a>認証

Bot Framework サービスでは、認証に JWT ベアラー トークンを使用します。 ご利用のボッドから Bot Framework に送信された送信要求を認証する方法、Bot Framework からご利用のボットに届いた受信要求を認証する方法などの詳細については、「[Authentication](bot-framework-rest-connector-authentication.md)」(認証) を参照してください。 

## <a name="client-libraries"></a>クライアント ライブラリ

Bot Framework には、C#、JavaScript、Python のいずれかでボットを作成するために使用できるクライアント ライブラリが用意されています。

- C# を使用してボットをビルドするには、[Bot Framework SDK for C#](../dotnet/bot-builder-dotnet-overview.md) をご利用ください。 
- Node.js を使用してボッドをビルドするには、[Bot Framework SDK for Node.js](../nodejs/index.md) をご利用ください。 

Bot Framework REST API の呼び出しの簡略化に加えて、各 Bot Framework SDK では、会話ロジック、単純な入力 (はい/いいえ、文字列、数字、列挙など) を求める組み込みのプロンプト、強力な AI フレームワーク (たとえば、<a href="https://www.luis.ai/" target="_blank">LUIS</a>) の組み込みサポートなどをカプセル化するダイアログを構築するための強力なシステムも提供されています。 

> [!NOTE]
> SDK を使用する代わりに、<a href="https://aka.ms/connector-swagger-file" target="_blank">Bot Connector Swagger ファイル</a> またはその REST API への直接のコードを使用して、任意の言語で独自のクライアント ライブラリを生成できます。

## <a name="bot-state-service"></a>Bot State サービス

Microsoft Bot Framework State サービスは 2018 年 3 月 30 日時点で廃止されています。 以前は、Azure Bot Service または Bot Builder SDK で構築されたボットには、ボットの状態データを格納するために Microsoft によってホストされたこのサービスへの既定の接続がありました。 ボットは、独自の状態ストレージを使用するように更新する必要があります。

## <a name="additional-resources"></a>その他のリソース

Bot Connector サービスを使用してボッドをビルドする方法の詳細については、このセクションで紹介した記事を参照してください。まず、「[認証](bot-framework-rest-connector-authentication.md)」から開始します。 Bot Connector サービスに関して問題が発生した場合や提案がある場合は、[サポート](../bot-service-resources-links-help.md)を参照して、利用可能なリソースの一覧を確認してください。 

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
