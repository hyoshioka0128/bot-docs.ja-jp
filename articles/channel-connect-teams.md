---
title: ボットを Teams に接続する - Bot Service
description: Teams を介してアクセスするようにボットを構成する方法について説明します。
keywords: Teams, ボット チャネル, Teams の構成
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/26/2019
ms.openlocfilehash: 8c2fdc87c45009a6d46b641d8d8aca2a31967c03
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75795843"
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

> [!IMPORTANT] 
> テスト以外の目的のために、GUID によってボットを追加することはお勧めできません。 そのようにすると、ボットの機能が厳しく制限されます。 運用環境のボットは、アプリの一部として Teams に追加する必要があります。 「[ボットの作成](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-create)」と[「Microsoft Teams のボットのテストとデバッグ](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-test)」を参照してください。


## <a name="additional-information"></a>関連情報
Microsoft Teams 固有の情報については、Teams の[ドキュメント](https://docs.microsoft.com/microsoftteams/platform/overview)を参照してください。 
