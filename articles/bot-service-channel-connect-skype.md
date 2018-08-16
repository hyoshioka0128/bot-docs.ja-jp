---
title: ボットを Skype に接続する | Microsoft Docs
description: Skpye インターフェイスを介してアクセスするようにボットを構成する方法について説明します。
keywords: skype, ボット チャネル, skype の構成, 公開, チャネルへの接続
author: v-ducvo
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 2/1/2018
ms.openlocfilehash: 5dc4063125855113f813b8873b01df84c90e197e
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301857"
---
# <a name="connect-a-bot-to-skype"></a>ボットを Skype に接続する

Skype は、インスタント メッセージング、電話、ビデオ通話を使用してユーザーとの接続を維持します。 ユーザーが Skype のインターフェイスを介して検出およびやり取りできるボットを構築すると、この機能を拡張することができます。

Skype チャネルを追加するには、[Azure portal](https://portal.azure.com/) でボットを開き、**[チャネル]** ブレードをクリックして、**[Skype]** をクリックします。 これにより、**[Configure Skype]\(Skype の構成\)** 設定ページに移動します。 ボットに関して必要な情報をすべて入力し、**[保存]** をクリックして Skype チャネルに接続します。 **サービス利用規約**を受け入れると、Skype チャネルがボットに追加されます。

![Skype チャネルを追加する](~/media/channels/skype-addchannel.png)

## <a name="web-control"></a>Web control (Web コントロール)

ボットを Web サイトに埋め込むには、**[Web control]\(Web コントロール\)** セクションの **[Get embed code]\(埋め込みコードの取得\)** ボタンをクリックしてコードを取得します。

## <a name="messaging"></a>メッセージング

このセクションでは、ボットが Skype でメッセージを送受信する方法を構成します。

## <a name="calling"></a>Calling (通話)

このセクションでは、ボットでの Skype の通話機能を構成します。 ボットで**呼び出し**を有効にするかどうか、および有効にする場合は IVR 機能またはリアルタイム メディア機能を使用するかどうかを指定できます。

## <a name="groups"></a>グループ

このセクションでは、ボットをグループに追加できるかどうか、グループでのメッセージングの動作方法、ボット通話用にグループ ビデオ通話を有効にするかどうかを構成します。

## <a name="publish"></a>[発行]

このセクションでは、ボットの公開設定を構成します。 * が付いているフィールドはすべて必須フィールドです。

**プレビュー**のボットは 100 連絡先に制限されます。 100 を超える連絡先が必要な場合は、確認用にボットを送信します。 **[確認用に送信]** をクリックすると、ボットは Skype で受け付けられた場合に自動的に検索可能になります。 要求が受け付けられない場合、受け付けられるために必要な変更が通知されます。

## <a name="next-steps"></a>次の手順

* [Skype for Business](bot-service-channel-connect-skypeforbusiness.md)
