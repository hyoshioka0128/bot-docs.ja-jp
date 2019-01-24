---
title: Bot Connector サービスと Bot State サービスでの主要な概念 | Microsoft Docs
description: Bot Framework の Bot Connector サービスおよび Bot State サービスでの主要な概念について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/16/2019
ms.openlocfilehash: 7464e6f19ac1cd1a5744af845bd62c3a48cd2eb8
ms.sourcegitcommit: c6ce4c42fc56ce1e12b45358d2c747fb77eb74e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/22/2019
ms.locfileid: "54453826"
---
# <a name="key-concepts"></a>主要な概念

Bot Connector サービスおよび Bot State サービスを使用すると、Skype、電子メール、Slack などの複数のチャネルでユーザーとの通信を行うことができます。 この記事では、Bot Connector サービスおよび Bot State サービスでの主要な概念について紹介します。

> [!IMPORTANT]
> Bot Framework State Service API を運用環境で使用することは推奨されていません。将来のリリースで非推奨となる可能性があります。 テストが目的の場合はメモリ内ストレージを使用するようにボット コードを更新し、運用ボットの場合はいずれかの **Azure 拡張機能**を使用することをお勧めします。 詳細については、[.NET](~/dotnet/bot-builder-dotnet-state.md) または [Node](~/nodejs/bot-builder-nodejs-state.md) の実装に関する「**状態データの管理**」トピックを参照してください。

## <a name="bot-connector-service"></a>Bot Connector サービス

Bot Connector サービスを使用すると、<a href="https://dev.botframework.com/" target="_blank">Bot Framework Portal</a> 内で構成されているチャネルとご利用のボットとの間でメッセージのやり取りができるようになります。 このサービスでは、HTTPS 経由で業界標準の REST および JSON が使用され、JWT ベアラー トークンによる認証が有効にされます。 Bot Connector サービスを使用する方法の詳細については、「[Authentication](bot-framework-rest-connector-authentication.md)」(認証) と、このセクションの残りの記事を参照してください。

### <a name="activity"></a>アクティビティ

Bot Connector を使用すると、[Activity][ Activity] オブジェクトを渡すことにより、ボットとチャネル (ユーザー) との間で情報の交換が行えるようになります。 最も一般的な種類のアクティビティは**メッセージ**ですが、さまざまな種類の情報をボットまたはチャネルに伝達するのに使用できるアクティビティの種類が他にもあります。 Bot Connector 内のアクティビティの詳細については、「[Activities overview](bot-framework-rest-connector-activities.md)」(アクティビティの概要) を参照してください。

## <a name="bot-state-service"></a>Bot State サービス

Bot State サービスを使用すると、ユーザー、会話、または特定の会話のコンテキスト内の特定のユーザーに関連付けられている状態データを、ご利用のボットで格納および取得できます。 このサービスでは、HTTPS 経由で業界標準の REST および JSON が使用され、JWT ベアラー トークンによる認証が有効にされます。 Bot State サービスを使用して保存するデータはいずれも Azure に格納され、保存時に暗号化されます。

Bot State サービスは、Bot Connector サービスと組み合わせて使用した場合にのみ有用です。 つまり、Bot State サービスは、ボッドによって行われる、Bot Connector サービスを使用した会話に関連する状態データを格納するのに使用できるのみとなります。 Bot State サービスを使用する方法の詳細については、「[Authentication](bot-framework-rest-connector-authentication.md)」(認証) と「[Manage state data](bot-framework-rest-state.md)」(状態データの管理) を参照してください。

## <a name="authentication"></a>Authentication

Bot Connector サービスと Bot State サービスの両方を使用すると、JWT ベアラー トークンによる認証が有効になります。 ご利用のボッドから Bot Framework に送信された送信要求を認証する方法、Bot Framework からご利用のボットに届いた受信要求を認証する方法などの詳細については、「[Authentication](bot-framework-rest-connector-authentication.md)」(認証) を参照してください。 

## <a name="client-libraries"></a>クライアント ライブラリ

Bot Framework には、C# または Node.js のいずれかでボットをビルドするのに使用できるクライアント ライブラリが用意されています。 

- C# を使用してボットをビルドするには、[Bot Framework SDK for C#](../dotnet/bot-builder-dotnet-overview.md) をご利用ください。 
- Node.js を使用してボッドをビルドするには、[Bot Framework SDK for Node.js](../nodejs/index.md) をご利用ください。 

Bot Connector サービスおよび Bot State サービスのモデル化に加えて、各 Bot Framework SDK では、会話ロジック、単純な入力 (はい/いいえ、文字列、数字、列挙など) を求める組み込みのプロンプト、強力な AI フレームワーク (<a href="https://www.luis.ai/" target="_blank">LUIS</a>) の組み込みサポートなどをカプセル化したダイアログを構築するための強力なシステムも提供されています。 

> [!NOTE]
> C# SDK または Node.js SDK を使用する代わりに、好みの言語で独自のクライアント ライブラリを <a href="https://aka.ms/connector-swagger-file" target="_blank">Bot Connector Swagger ファイル</a>および <a href="https://aka.ms/state-swagger-file" target="_blank">Bot State Swagger ファイル</a>を使用して作成できます。

## <a name="additional-resources"></a>その他のリソース

Bot Connector サービスおよび Bot State サービスを使用してボッドをビルドする方法の詳細については、このセクションで紹介した記事を参照してください。まず、「[Authentication](bot-framework-rest-connector-authentication.md)」(認証) を参照してください。 Bot Connector サービスおよび Bot State サービスに関する問題が発生した場合や提案がある場合は、[サポート](../bot-service-resources-links-help.md)に関する記事で利用可能なリソースの一覧をご覧ください。 

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object