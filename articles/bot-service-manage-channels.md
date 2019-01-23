---
title: 1 つまたは複数のチャネルで実行するようにボットを構成する | Microsoft Docs
description: Bot Framework Portal を使用して、1 つまたは複数のチャネルで実行するようにボットを構成する方法について説明します。
keywords: ボット チャネル, 構成, cortana, facebook messenger, kik, slack, skype, azure portal
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/22/2018
ms.openlocfilehash: a00e1d0d70e48b1af9cf10fd4e47ea86ae4908e9
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224257"
---
# <a name="connect-a-bot-to-channels"></a>ボットをチャネルに接続する

チャネルは、ボットと通信アプリ間の接続です。 ボットを使えるようにしたいチャネルに接続するようにボットを構成します。 Azure portal 経由で構成される Bot Framework サービスは、ボットをこれらのチャネルに接続し、ボットとユーザーの間の通信を容易にします。 [Cortana](bot-service-channel-connect-cortana.md)、[Facebook Messenger](bot-service-channel-connect-facebook.md)、[Kik](bot-service-channel-connect-kik.md)、[Slack](bot-service-channel-connect-slack.md) など、人気のあるさまざまなサービスに接続できます。 [Skype](https://dev.skype.com/bots) と Web チャットは、あらかじめ構成されています。 Bot Connector サービスに付属する標準チャネルに加えて、ダイレクト ラインをチャネルとして使用して、ボットを独自のクライアント アプリケーションに接続することもできます。

Bot Framework サービスを使用すると、ボットがチャネルに送信するメッセージを正規化することによって、チャネルにとらわれない方法でボットを開発できます。 これを行うには、メッセージを Bot Framework スキーマからチャネルのスキーマに変換する必要があります。 ただし、チャネルで Bot Framework スキーマのすべての側面がサポートされているわけではない場合、サービスによってチャネルでサポートされている形式へのメッセージの変換が試みられます。 たとえば、アクション ボタンが含まれたカードを含むメッセージをボットが SMS チャネルに送信した場合、コネクタはそのカードをイメージとして送信し、アクションをそのメッセージのテキスト内にリンクとして含める可能性があります。



ほとんどのチャネルでは、チャネルでボットを実行するには、チャネルの構成情報を提供する必要があります。 ほとんどのチャネルでは、ボットにチャネルのアカウントがある必要があります。また、Facebook Messenger などでは、ボットにチャネルに登録されているアプリケーションも必要です。

チャネルに接続するようにボットを構成するには、次の手順を完了します。

1. <a href="https://portal.azure.com" target="_blank">Azure Portal</a> にサインインします。
1. 構成するボットを選択します。
3. [ボット サービス] ブレードの **[Bot management]\(ボット管理\)** で、**[チャネル]** をクリックします。
4. ボットに追加するチャネルのアイコンをクリックします。

![チャネルに接続する](./media/channels/connect-to-channels.png)

チャネルを構成すると、そのチャネル上のユーザーがボットを使用できるようになります。

## <a name="publish-a-bot"></a>ボットの発行

発行プロセスは、チャネルごとに異なります。

[!INCLUDE [publishing](./includes/snippet-publish-to-channel.md)]

## <a name="additional-resources"></a>その他のリソース
SDK には、ボットの構築に使用できるサンプルが含まれています。 サンプルの一覧については、[GitHub](https://github.com/Microsoft/BotBuilder-samples) リポジトリを参照してください。
