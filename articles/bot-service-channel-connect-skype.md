---
title: ボットを Skype に接続する - Bot Service
description: Skype インターフェイスを介してアクセスするようにボットを構成する方法について説明します。
keywords: skype, ボット チャネル, skype の構成, 公開, チャネルへの接続
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 10/11/2018
ms.openlocfilehash: 5fb665bf8f6857f42b977a5f0126ce2e0426586d
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791897"
---
# <a name="connect-a-bot-to-skype"></a>ボットを Skype に接続する

Skype は、インスタント メッセージング、電話、ビデオ通話を使用してユーザーとの接続を維持します。 ユーザーが Skype のインターフェイスを介して検出およびやり取りできるボットを構築すると、この機能を拡張することができます。

Skype チャネルを追加するには、[Azure portal](https://portal.azure.com/) でボットを開き、 **[チャネル]** ブレードをクリックして、 **[Skype]** をクリックします。

![Skype チャネルを追加する](~/media/channels/skype-addchannel.png)

これにより、 **[Configure Skype]\(Skype の構成\)** 設定ページに移動します。

![Skype チャネルを構成](~/media/channels/skype_configure.png)

**[Web control]\(Web コントロール\)** 、 **[Messaging]\(メッセージング\)** 、 **[Calling]\(通話\)** 、 **[Groups]\(グループ\)** 、 **[Publish]\(発行\)** の設定を構成する必要があります。 1 つずつ見ていきましょう。

## <a name="web-control"></a>Web control (Web コントロール)

ボットを Web サイトに埋め込むには、 **[Web control]\(Web コントロール\)** セクションの **[Get embed code]\(埋め込みコードの取得\)** ボタンをクリックします。 これにより [開発者向け Skype] ページに誘導されます。 画面の指示に従って埋め込みコードを取得してください。

## <a name="messaging"></a>メッセージング

このセクションでは、ボットが Skype でメッセージを送受信する方法を構成します。

## <a name="calling"></a>呼び出し

このセクションでは、ボットでの Skype の通話機能を構成します。 ボットで**呼び出し**を有効にするかどうか、および有効にする場合は IVR 機能またはリアルタイム メディア機能を使用するかどうかを指定できます。

## <a name="groups"></a>グループ

このセクションでは、ボットをグループに追加できるかどうか、グループでのメッセージングの動作方法、ボット通話用にグループ ビデオ通話を有効にするかどうかを構成します。

## <a name="publish"></a>発行

このセクションでは、ボットの公開設定を構成します。 \* が付いているフィールドはすべて必須フィールドです。

**プレビュー**のボットは 100 連絡先に制限されます。 100 を超える連絡先が必要な場合は、確認用にボットを送信します。 **[確認用に送信]** をクリックすると、ボットは Skype で受け付けられた場合に自動的に検索可能になります。 要求が受け付けられない場合、受け付けられるために必要な変更が通知されます。

> [!TIP]
> 確認用にボットを送信する場合、受け付けられるには、[skype の認定チェックリスト](https://github.com/Microsoft/skype-dev-bots/blob/master/certification/CHECKLIST.md)を満たす必要があることに注意してください。

構成が完了したら、 **[保存]** をクリックして**サービス利用規約**に同意します。 ボットに Skype チャネルが追加されました。

## <a name="next-steps"></a>次のステップ

* [Skype for Business](bot-service-channel-connect-skypeforbusiness.md)
