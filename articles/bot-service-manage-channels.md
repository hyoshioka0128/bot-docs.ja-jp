---
title: 1 つまたは複数のチャネルで実行するようにボットを構成する | Microsoft Docs
description: Bot Framework Portal を使用して、1 つまたは複数のチャネルで実行するようにボットを構成する方法について説明します。
keywords: ボット チャネル, 構成, cortana, facebook messenger, kik, slack, skype, azure portal
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: cb682bf77f801c98d00deffa0fc63249962248cd
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352921"
---
# <a name="connect-a-bot-to-channels"></a>ボットをチャネルに接続する

チャネルは、Bot Framework と通信アプリ間の接続です。 ボットを使えるようにしたいチャネルに接続するようにボットを構成します。 たとえば、Skype チャネルに接続されているボットを連絡先リストに追加すると、ユーザーが Skype でボットと対話できるようになります。 

チャネルには、[Cortana](bot-service-channel-connect-cortana.md)、[Facebook Messenger](bot-service-channel-connect-facebook.md)、[Kik](bot-service-channel-connect-kik.md)、[Slack](bot-service-channel-connect-slack.md) など、人気のあるサービスが数多く含まれています。 [Skype](https://dev.skype.com/bots) と Web チャットは、あらかじめ構成されています。 

[Azure Portal](https://portal.azure.com) では、チャネルへの接続がすばやく簡単に行えます。

## <a name="get-started"></a>作業開始

ほとんどのチャネルでは、チャネルでボットを実行するには、チャネルの構成情報を提供する必要があります。 ほとんどのチャネルでは、ボットにチャネルのアカウントがある必要があります。また、Facebook Messenger などでは、ボットにチャネルに登録されているアプリケーションも必要です。

チャネルに接続するようにボットを構成するには、次の手順を完了します。

1. <a href="https://portal.azure.com" target="_blank">Azure Portal</a> にサインインします。
1. 構成するボットを選択します。
3. [ボット サービス] ブレードの **[Bot management]\(ボット管理\)** で、**[チャネル]** をクリックします。
4. ボットに追加するチャネルのアイコンをクリックします。

![チャネルに接続する](~/media/channels/connect-to-channels.png)

チャネルを構成すると、そのチャネル上のユーザーがボットを使用できるようになります。

## <a name="publish-a-bot"></a>ボットの発行

発行プロセスは、チャネルごとに異なります。

[!INCLUDE [publishing](~/includes/snippet-publish-to-channel.md)]

