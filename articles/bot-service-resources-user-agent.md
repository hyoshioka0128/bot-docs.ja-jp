---
title: Bot Framework の User-Agent 要求 - Bot Service
description: Bot Framework から Web サーバーへの要求について理解します。
author: JohnD-ms
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: f30cf7b8d86195595575260dfd2fbcd8c14c1a15
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "80648049"
---
# <a name="bot-framework-user-agent-requests"></a>Bot Framework の User-Agent 要求

このメッセージを読んでいる場合は、Microsoft Bot Framework サービスからの要求を受信している可能性があります。 このガイドでは、この要求の性質を理解し、必要な場合には停止する手順について説明します。

上記のサービスからの要求を受信している場合、User-Agent ヘッダーは次の文字列のような書式になっています。

```User-Agent: BF-DirectLine/3.0 (Microsoft-BotFramework/3.0 +https://botframework.com/ua)```

この文字列の中で最も重要なのは、Microsoft Bot Framework が使用する **Microsoft-BotFramework** 識別子です。Microsoft Bot Framework は、独立したソフトウェア開発者が独自のボットを作成して操作できるようにするツールとサービスのコレクションです。

ボットの開発者の方なら、皆さんのサービスにこの要求が送られる理由は既にご存知でしょう。 そうでない方は、引き続き以下をお読みください。

## <a name="why-is-microsoft-contacting-my-service"></a>Microsoft が皆さんのサービスにアクセスする理由

Bot Framework は、Facebook Messenger などのチャット サービスを利用しているユーザーとボットをつなぐものです。ボットは、インターネットにアクセスできるエンドポイントで実行され、REST API を使用できる Web サーバーです。 ボットに対する HTTP 呼び出し (Webhook 呼び出しとも呼ばれます) は、Bot Framework 開発者ポータルに登録されているボット開発者が指定した URL にのみ送信されます。

皆さんの Web サービスが Bot Framework サービスからの未承諾要求を受信している場合、ある開発者がボットの Webhook コールバックとして誤ってまたは故意に皆さんの URL を入力した可能性があります。

## <a name="to-stop-these-requests"></a>要求を停止するには

望まない要求が Web サービスに到達しないようにする方法の詳細については、[bf-reports@microsoft.com](mailto://bf-reports@microsoft.com) にお問い合わせください。
