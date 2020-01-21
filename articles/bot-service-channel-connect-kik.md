---
title: ボットを Kik に接続する - Bot Service
description: ボットの Kik への接続を構成する方法について説明します。
keywords: ボットの接続, ボットのチャネル, Kik ボット, 資格情報, 構成, 携帯電話
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 640c430786bbba6ec78d6ce3b1c02d8955a54b09
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791987"
---
# <a name="connect-a-bot-to-kik"></a>ボットを Kik に接続する

Kik メッセージング アプリを使用して、複数のユーザーとコミュニケーションするようにボットを構成できます。

## <a name="install-kik-on-your-phone"></a>Kik を携帯電話にインストールする

Kik を携帯電話にインストールしていない場合は、携帯電話のアプリ ストアまたは <a href="https://www.kik.com/" target="_blank">Kik Web サイト</a>からインストールしてください。 既存の Kik ユーザー アカウントを使用するか、または新しいアカウントにサインアップする必要があります。

![Kik サインアップ](./media/channels/kik-signup.png)

## <a name="log-into-the-dev-portal-with-your-mobile-phone"></a>携帯電話でデベロッパー ポータルにログインする

携帯電話を使用して、<a href="https://dev.kik.com" target="_blank">Kik ポータルにログイン</a>します。 _[Open this page in "Kik"?]\("Kik" でこのページを開きますか?\)_ と表示されたら、 **[Open]\(開く\)** を選択します。 

![Kik 開発ポータル](./media/channels/kik-dev-portal.png)

## <a name="follow-the-bot-setup-process"></a>ボット設定プロセスに従う

ボットの名前を指定します。

![ボットを設定する](./media/channels/kik-phone.png)

## <a name="gather-credentials"></a>資格情報を収集する

[Configuration]\(構成\) タブで、 **[Name]\(名前\)** と **[API key]\(API キー\)** をコピーします。

![ボットの情報のコピー](./media/channels/kik-configure.png)

## <a name="submit-credentials"></a>資格情報を送信する

![資格情報の貼り付け](./media/channels/kik-creds.png)

**[Submit Kik Credentials]\(KiK 資格情報の送信\)** をクリックします。

## <a name="enable-the-bot"></a>ボットを有効にする

**[Enable this bot on Kik]\(このボットを Kik で有効にする\)** をオンにします。 次に、 **[I'm done configuring Kik]\(Kik の構成が終了しました\)** をクリックします。

これらの手順を完了すると、ボットは、Kik でユーザーとコミュニケーションするように正しく構成されます。
