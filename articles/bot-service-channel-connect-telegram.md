---
title: Telegram 用のボットを作成する - Bot Service
description: ボットの Telegram への接続を構成する方法について説明します。
keywords: ボットの構成, Telegram, ボット チャネル, Telegram ボット, アクセス トークン
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: fd60fbb10e9560e3df6e427f0d5193df463ec473
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75791820"
---
# <a name="connect-a-bot-to-telegram"></a>ボットを Telegram に接続する

Telegram メッセージング アプリを使用してユーザーと通信するようにボットを構成できます。

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

## <a name="visit-the-bot-father-to-create-a-new-telegram-bot"></a>Bot Father にアクセスして新しい Telegram ボットを作成する

Bot Father を使って<a href="https://telegram.me/botfather" target="_blank">新しい Telegram ボットを作成</a>します。

![Bot Father にアクセスする](~/media/channels/tg-StepVisitBotFather.png)

## <a name="create-a-new-telegram-bot"></a>新しい Telegram ボットを作成する
新しい Telegram ボットを作成するには、`/newbot` コマンドを送信します。

![新しいボットを作成する](~/media/channels/tg-StepNewBot.png)

### <a name="specify-a-friendly-name"></a>フレンドリ名を指定する

Telegram ボットにフレンドリ名を付けます。

![ボットにフレンドリ名を指定する](~/media/channels/tg-StepNameBot.png)

### <a name="specify-a-username"></a>ユーザー名を指定する

Telegram ボットに一意のユーザー名を指定します。

![ボットに一意名を指定する](~/media/channels/tg-StepUsername.png)

### <a name="copy-the-access-token"></a>アクセス トークンをコピーする

Telegram ボットのアクセス トークンをコピーします。

![アクセス トークンをコピーする](~/media/channels/tg-StepBotCreated.png)

## <a name="enter-the-telegram-bots-access-token"></a>Telegram ボットのアクセス トークンを入力する

Azure portal でご使用のボットの **[チャネル]** セクションにアクセスし、 **[Telegram]** ボタンをクリックします。 

> [!NOTE]
>  ボットを Telegram に既に接続している場合は、Azure portal UI の表示は多少異なります。 

![チャネルでの Telegram の選択](~/media/channels/tg-connectBot-Azure.png)

前にコピーしたトークンを **[アクセス トークン]** フィールドに貼り付けて、 **[保存]** をクリックします。

![Telegram のアクセス トークン](~/media/channels/tg-accessToken-Azure.png)

これで、ボットは Telegram のユーザーと通信するように正常に構成されました。 

![Telegram ボットの有効化](~/media/channels/tg-botEnabled-Azure.png)
