---
title: その他のチャネル |Microsoft Docs
description: お客様のボットの他のチャネルを構成する方法について説明します。
keywords: ボットがチャネル、ハングアウト、Twilio、facebook、azure portal
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/08/2019
ms.openlocfilehash: d6cf068f3cd4d57ff9cde2be6d41e084cee6524d
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693742"
---
# <a name="additional-channels"></a>その他のチャネル

チャネルの記載だけでなくこれらのドキュメント内では、いくつかその他のチャネルを通じて両方のアダプターとして使用可能な[プラットフォームを提供](https://botkit.ai/docs/v4/platforms/)Botkit、またはからアクセスできる、[コミュニティ リポジトリ](https://github.com/BotBuilderCommunity/). 次には、提供されるその他のチャネルを示します。

- [Webex チーム](https://botkit.ai/docs/v4/platforms/webex.html)
- [Websocket と Webhook](https://botkit.ai/docs/v4/platforms/web.html)
- [Google ハングアウトと Google Assistant](https://github.com/BotBuilderCommunity/) (コミュニティから入手可能)
- [Amazon Alexa](https://github.com/BotBuilderCommunity/) (コミュニティから入手可能)

## <a name="currently-available-adapters"></a>現在使用可能なアダプター

使用可能なアダプターの完全な一覧は、[こちら](https://botkit.ai/docs/v4/platforms/)します。 一部のチャネルは、アダプターとしても使用可能なとがするアダプターとチャネルを使用するときにわかります。

### <a name="when-to-use-an-adapter"></a>アダプターを使用する場合

1. サービスが必要なチャネルをサポートしていません
2. デプロイのセキュリティとコンプライアンスの要件に基づき、外部サービスに依存することはできません。
3. 特定のチャネルで必要な機能の深さがサポートされていません

### <a name="when-to-use-a-channel"></a>チャネルを使用する場合

1. クロス チャネルの互換性が必要な: お客様のボットが利用可能なチャネルの 1 つ以上で動作する必要があります
2. 組み込みのサポート:Microsoft の維持、パッチ、およびシームレスにサービスの各チャネルのサード パーティ製更新プログラムは、毎回
3. Microsoft Teams を迅速に拡大などに、追加の排他的な Microsoft チャネルへのアクセスを許可します。
4. お客様のボットの他のチャネルを有効にする GUI インターフェイスに依存する場合
