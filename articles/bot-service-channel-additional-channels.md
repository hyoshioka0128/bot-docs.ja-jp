---
title: 追加のチャネル - Bot Service
description: 追加のチャネルをボットに構成する方法について説明します。
keywords: ボット チャネル, ハングアウト, Twilio, Facebook, Azure portal
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/08/2019
ms.openlocfilehash: 7aff1376ffa7cc720548ca34eec92acc2d2ef057
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75789150"
---
# <a name="additional-channels"></a>追加のチャネル

これらのドキュメント内にリストされているチャネルのほかに、アダプターとして使用できる追加のチャネルがいくつかあり、Botkit により[提供されるプラットフォーム](https://botkit.ai/docs/v4/platforms/)、または[コミュニティ リポジトリ](https://github.com/BotBuilderCommunity/)の両方を通じてアクセスできます。 提供される追加のチャネルは以下のとおりです。

- [Webex Teams](https://botkit.ai/docs/v4/platforms/webex.html)
- [WebSocket と Webhook](https://botkit.ai/docs/v4/platforms/web.html)
- [Google ハングアウトと Google アシスタント](https://github.com/BotBuilderCommunity/) (コミュニティを通じて利用可能)
- [Amazon Alexa](https://github.com/BotBuilderCommunity/) (コミュニティを通じて利用可能)

## <a name="currently-available-adapters"></a>現在使用できるアダプター

使用できるアダプターの完全な一覧は、[こちらで確認](https://botkit.ai/docs/v4/platforms/)できます。 いくつかのチャネルはアダプターとしても使用できることがわかります。チャネルとアダプターのどちらを使用するかはお客様しだいです。

### <a name="when-to-use-an-adapter"></a>アダプターを使用する場合

1. このサービスでは、お客様が必要とするチャネルがサポートされません
2. デプロイのセキュリティとコンプライアンスの要件により、外部のサービスを利用することができません
3. 特定のチャネルで必要なレベルの機能がサポートされない場合があります

### <a name="when-to-use-a-channel"></a>チャネルを使用する場合

1. チャネル間の互換性が必要です。お客様のボットは、利用可能な 2 つ以上のチャネルで動作する必要があります
2. 組み込みのサポート: サード パーティが更新を行うたび、Microsoft はお客様のために各チャネルのメンテナンス、パッチ適用、円滑な保守を行います
3. 追加の限定 Microsoft チャネル (急速に成長している Microsoft Teams など) にアクセスできます
4. 自分のボットで追加のチャネルを有効にするために、GUI インターフェイスを利用したい場合
