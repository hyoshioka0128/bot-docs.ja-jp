---
title: Telegram 用のボットを作成する | Microsoft Docs
description: ボットの Telegram への接続を構成する方法について説明します。
keywords: ボットの構成, Telegram, ボット チャネル, Telegram ボット, アクセス トークン
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: efe38392117fb871b2b98e3f1d8d798bfaef0c41
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563759"
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

前にコピーしたトークンを **[Access Token]\(アクセス トークン\)** フィールドに貼り付けて、**[Submit]\(送信\)** をクリックします。

## <a name="enable-the-bot"></a>ボットを有効にする
**[Enable this bot on Telegram]\(このボットを Telegram で有効にする\)** をオンにします。 次に、**[I'm done configuring Telegram]\(Telegram の構成が終了しました\)** をクリックします。

これらの手順を完了すると、ボットは、Telegram でユーザーと通信するように正しく構成されます。
