---
title: Web チャットでボット サービスをテストする | Microsoft Docs
description: Azure portal の webchat コントロールを使用して Bot Service をテストする方法について説明します。
keywords: Web チャットでのテスト、Azure portal
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 0c358f4e53f3fd64cce3635f644cc0f2f612e983
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300676"
---
# <a name="test-in-web-chat"></a>Web チャットでのテスト
Bot Service には、ボットを簡単にテストするために役立つ [Web チャット コントロール](bot-service-channel-connect-webchat.md)が含まれています。 

## <a name="test-a-bot-in-the-azure-portal-with-web-chat"></a>Azure portal で Web チャットを使用してボットをテストする
[Azure portal](https://portal.azure.com) にサインインして、ボットのブレードを開きます。 [Bot Management]\(ボットの管理\) セクションで、**[Test in Web Chat]\(Web チャットでのテスト\)** をクリックします。 Bot Service で Web チャット コントロールを読み込み、ボットに接続します。

![[Test in Web Chat]\(Web チャットでのテスト\) の UI](~/media/test-in-webchat/test-in-webchat.png)

ボットとチャットするテキストを入力することができます。 ボットが音声をサポートしている場合は、マイクのボタンをクリックしてボットと話すことができます。 ボットが添付ファイルをサポートしている場合は、画像などの添付ファイルをアップロードできます。 [Bot Builder SDK](bot-builder-overview-getstarted.md) でボットに機能を追加する方法を参照してください。

> [!NOTE]
> 数分経っても Web チャット コントロールの読み込みが完了しない場合は、ページを更新してみてください。

## <a name="next-steps"></a>次の手順
ここでは、Azure でボットをテストする方法について説明しました。次は、より詳しいテストとデバッグに Bot Framework Emulator を活用する方法について説明します。

> [!div class="nextstepaction"]
> [Bot Framework Emulator](bot-service-debug-emulator.md)