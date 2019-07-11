---
title: ボットを Teams に接続する | Microsoft Docs
description: Teams を介してアクセスするようにボットを構成する方法について説明します。
keywords: Teams, ボット チャネル, Teams の構成
author: kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/05/2019
ms.openlocfilehash: d2609e4294416691e156ba3dbd09eabc8e0d3423
ms.sourcegitcommit: b498649da0b44f073dc5b23c9011ea2831edb31e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/05/2019
ms.locfileid: "67592231"
---
# <a name="connect-a-bot-to-teams"></a>ボットを Teams に接続する

Microsoft Teams チャネルを追加するには、[Azure portal](https://portal.azure.com) でボットを開き、 **[チャネル]** ブレードをクリックして、 **[Teams]** をクリックします。

![Teams チャネルの追加](media/teams/connect-teams-channel.png)

次に、 **[保存]** をクリックします。

![Teams チャネルの保存](media/teams/save-teams-channel.png)

Teams チャネルを追加した後、 **[チャネル]** ページに移動し、 **[Get bot embed code]** \(ボット埋め込みコードを取得する\) をクリックしまう。

![埋め込みコードの取得](media/teams/get-embed-code.png)

- **[Get bot embed code]** \(ボット埋め込みコードを取得する\) ダイアログに表示されたコードの _https_ 部分をコピーします。 たとえば、「 `https://teams.microsoft.com/l/chat/0/0?users=28:b8a22302e-9303-4e54-b348-343232` 」のように入力します。 

- ブラウザーにこのアドレスを貼り付け、ボットを Teams に追加するために使用する Microsoft Teams アプリ (クライアントまたは Web) を選択します。 ボットは、Microsoft Teams でメッセージの送受信を行える連絡先として一覧表示されます。 

## <a name="additional-information"></a>追加情報
Microsoft Teams 固有の情報については、Teams の[ドキュメント](https://docs.microsoft.com/en-us/microsoftteams/platform/overview)を参照してください。 
