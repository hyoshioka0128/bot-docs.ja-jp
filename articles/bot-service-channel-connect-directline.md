---
title: ボットを Direct Line に接続する - Bot Service
description: Direct Line へのボットの接続を構成する方法について説明します。
keywords: Direct Line, ボット チャネル, カスタム クライアント, チャネルへの接続, 構成
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/7/2019
ms.openlocfilehash: a7c52ec4a6ab8d1744e2c4d9a94ec2ca5623329a
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "77479220"
---
# <a name="connect-a-bot-to-direct-line"></a>ボットを Direct Line に接続する

Direct Line チャネルを使用することによって、独自のクライアント アプリケーションとボットの通信を有効にできます。

## <a name="add-the-direct-line-channel"></a>Direct Line チャネルを追加する

Direct Line チャネルを追加するには、[Azure portal](https://portal.azure.com/) でボットを開き、 **[チャネル]** ブレードをクリックして、 **[Direct Line]\(Direct Line\)** をクリックします。

![Direct Line チャネルを追加する](media/bot-service-channel-connect-directline/directline-addchannel.png)

## <a name="add-new-site"></a>新しいサイトを追加する

次に、ボットに接続するクライアント アプリケーションを表す新しいサイトを追加します。 **[Add new site]\(新しいサイトの追加\)** をクリックし、サイトの名前を入力して、 **[完了]** をクリックします。

![Direct Line サイトを追加する](media/bot-service-channel-connect-directline/directline-addsite.png)

## <a name="manage-secret-keys"></a>秘密鍵を管理する

サイトが作成されると、Bot Framework によって秘密鍵が生成されます。クライアント アプリケーションは、この秘密鍵を使って、ボットと通信するために発行する Direct Line API 要求の[認証](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md)を行うことができます。 鍵をプレーン テキストで表示するには、対応する鍵の **[表示]** をクリックします。

![Direct Line の鍵を表示する](media/bot-service-channel-connect-directline/directline-showkey.png)

表示された鍵をコピーし、安全に保管します。 その後、この鍵を使って、クライアントがボットと通信するために発行する Direct Line API 要求を[認証](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md)します。
または、Direct Line API を使って[鍵をトークンと交換](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md#generate-token)します。クライアントはこのトークンを使って、1 つの会話のスコープ内で後続の要求を認証できます。

![Direct Line の鍵をコピーする](media/bot-service-channel-connect-directline/directline-copykey.png)

## <a name="configure-settings"></a>設定の構成

最後に、サイトの設定を構成します。

- クライアント アプリケーションがボットとの通信に使用する Direct Line プロトコルのバージョンを選択します。

> [!TIP]
> クライアント アプリケーションとボットの間の新しい接続を作成する場合は、Direct Line API 3.0 を使います。

終わったら、 **[完了]** をクリックしてサイトの構成を保存します。 ボットに接続するクライアント アプリケーションごとに、このプロセスを [[Add new site]\(新しいサイトの追加\)](#add-new-site) から繰り返すことができます。

**拡張認証が有効になっている**場合は、信頼されている配信元が使用される次の動作を確認できます。

- 信頼されている配信元を構成 UI ページの一部として構成した場合、これらは**常に**唯一のセットとして使用されます。 トークンの生成や会話の開始時に、信頼されている配信元を送信しない場合、または追加で送信する場合は、無視されます (つまり、それらはリストに**追加されず**、クロス検証もされません)。
- 信頼されている配信元を構成 UI の一部として構成していない場合は、API 呼び出しの一部として送信した値が使用されます。