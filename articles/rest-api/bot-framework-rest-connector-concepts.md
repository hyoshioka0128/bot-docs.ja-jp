---
title: Bot Connector サービスと Bot State サービスでの主要な概念 | Microsoft Docs
description: Bot Framework の Bot Connector サービスおよび Bot State サービスでの主要な概念について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/16/2019
ms.openlocfilehash: a2fa3f5e1363cb155504cdf903df2f6c25877af3
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/03/2019
ms.locfileid: "68756949"
---
# <a name="key-concepts"></a>主要な概念

Bot Connector サービスを使用すると、Skype、電子メール、Slack などの複数のチャネルでユーザーと通信できます。 この記事では、Bot Connector サービスでの主要な概念について紹介します。

## <a name="bot-connector-service"></a>Bot Connector サービス

Bot Connector サービスを使用すると、<a href="https://dev.botframework.com/" target="_blank">Bot Framework Portal</a> 内で構成されているチャネルとご利用のボットとの間でメッセージのやり取りができるようになります。 このサービスでは、HTTPS 経由で業界標準の REST および JSON が使用され、JWT ベアラー トークンによる認証が有効にされます。 Bot Connector サービスを使用する方法の詳細については、「[Authentication](bot-framework-rest-connector-authentication.md)」(認証) と、このセクションの残りの記事を参照してください。

### <a name="activity"></a>アクティビティ

Bot Connector サービスでは、`Activity` オブジェクトを渡すことで、ボットとチャネル (ユーザー) 間で情報が交換されます。 最も一般的なアクティビティの種類は**メッセージ**ですが、ボットまたはチャネルにさまざまな種類の情報を通信するために使用できる他のアクティビティの種類も存在します。 Bot Connector 内のアクティビティの詳細については、「[Activities overview](bot-framework-rest-connector-activities.md)」(アクティビティの概要) を参照してください。

## <a name="bot-state-service"></a>Bot State サービス

Microsoft Bot Framework State サービスは 2018 年 3 月 30 日時点で廃止されています。 以前は、Azure Bot Service または Bot Builder SDK で構築されたボットには、ボットの状態データを格納するために Microsoft によってホストされたこのサービスへの既定の接続がありました。 ボットは、独自の状態ストレージを使用するように更新する必要があります。

## <a name="authentication"></a>Authentication

Bot Connector サービスを使用すると、JWT ベアラー トークンによる認証が有効になります。 ご利用のボッドから Bot Framework に送信された送信要求を認証する方法、Bot Framework からご利用のボットに届いた受信要求を認証する方法などの詳細については、「[Authentication](bot-framework-rest-connector-authentication.md)」(認証) を参照してください。 

## <a name="client-libraries"></a>クライアント ライブラリ

Bot Framework には、C# または Node.js のいずれかでボットをビルドするのに使用できるクライアント ライブラリが用意されています。 

- C# を使用してボットをビルドするには、[Bot Framework SDK for C#](../dotnet/bot-builder-dotnet-overview.md) をご利用ください。 
- Node.js を使用してボッドをビルドするには、[Bot Framework SDK for Node.js](../nodejs/index.md) をご利用ください。 

Bot Connector サービスのモデル化に加えて、各 Bot Framework SDK では、会話ロジック、単純な入力 (はい/いいえ、文字列、数字、列挙など) を求める組み込みのプロンプト、強力な AI フレームワーク (たとえば、<a href="https://www.luis.ai/" target="_blank">LUIS</a>) の組み込みサポートなどをカプセル化するダイアログを構築するための強力なシステムも提供されています。 

> [!NOTE]
> C# SDK または Node.js SDK を使用する代わりに、任意の言語で独自のクライアント ライブラリを <a href="https://aka.ms/connector-swagger-file" target="_blank">Bot Connector Swagger ファイル</a>を使用して作成できます。

## <a name="additional-resources"></a>その他のリソース

Bot Connector サービスを使用してボッドをビルドする方法の詳細については、このセクションで紹介した記事を参照してください。まず、「[認証](bot-framework-rest-connector-authentication.md)」から開始します。 Bot Connector サービスに関して問題が発生した場合や提案がある場合は、[サポート](../bot-service-resources-links-help.md)を参照して、利用可能なリソースの一覧を確認してください。 
