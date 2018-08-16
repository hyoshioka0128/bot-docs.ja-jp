---
title: Conversation Designer ボットの作成 | Microsoft Docs
description: Conversation Designer を使用して新しいボットを作成する方法について説明します。
author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 1b33d08e56bf8ae473b28cf1cf3a26d8138ce861
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301501"
---
# <a name="create-a-new-conversation-designer-bot"></a>新しい Conversation Designer ボットの作成
> [!IMPORTANT]
> Conversation Designer は、一部のお客様はまだご利用いただけません。 Conversation Designer の可用性の詳細については、今年中に提供される予定です。

このチュートリアルでは、新しい Conversation Designer ボットを作成する方法を順を追って説明します。 

## <a name="prerequisites"></a>前提条件

- Conversation Designer には、Azure サブスクリプションが必要です。 <a href="https://azure.microsoft.com/en-us/" target="_blank">ここ</a>から開始できます。
- まだ実行していない場合は、[LUIS ポータル](https://luis.ai)にアカウントを作成した後に少なくとも 1 回はサインインしていることを確認してください。
- JavaScript のプログラミングに関する知識。 カスタム スクリプト関数は JavaScript で記述します。
- Microsoft Edge または Google Chrome

## <a name="create-a-conversation-designer-bot"></a>Conversation Designer ボットの作成

Conversation Designer ボットを作成するには、次の手順を実行します。
1. https://dev.botframework.com/ に移動し、サインインします。 このプライベート プレビュー リリースに参加するには、Microsoft Corporation に指定した電子メール アドレスを使用します。
2. 右上のナビゲーション パネルで **[ボットの作成]** をクリックします。 
3. **[作成]** をクリックして、"*Conversation Designer でボットを作成*" します。
4. 多くの[サンプル ボット](conversation-designer-sample-bots.md)から、最初に使用するサンプル ボットを 1 つ選択します。 **[次へ]** をクリックします。 使用する**サンプル ボット**を決められない場合は、作成するボットに最も近いと考えられるものを 1 つ選択してください。 後で別の**サンプル ボット**に切り替えることができます。
5. すべてのフィールドに入力し、**[ボットの作成]** をクリックします。ボットのプロビジョニングが完了するまで約 2 分かかります。 

## <a name="bot-provisioning"></a>ボットのプロビジョニング

次の Azure の機能は、Conversation Designer ボットを作成するときに自動的にプロビジョニングされます。 

1. ユーザーが指定したボット名を持つ Azure リソース グループ
2. Azure App Service
3. Azure App Service プラン 
4. Azure ストレージ アカウント
5. Application Insights 
6. [LUIS.ai](https://luis.ai) の Cognitive Services サブスクリプション。 LUIS アプリは、アプリ名として**ボット ハンドル** (に加えて、ランダムに生成された文字列) を使用して作成されます。
7. Microsoft アカウントのアプリの 1 つ。 [詳細情報](https://apps.dev.microsoft.com/#/appList)

## <a name="welcome-message"></a>ウェルカム メッセージ

ボットがプロビジョニングされると、Conversation Designer によってボットの **[ビルド]** ページが開かれます。 開始するときに役立つ情報と共に、ウェルカム メッセージが表示されます。 これらのオプションを確認するか、メッセージを閉じて、ボットの作業を開始します。 ウェルカム メッセージに戻るには、左上のナビゲーション パネルで省略記号 (**...**) をクリックし、**[Welcome...]\(ようこそ...\)** オプションを選択します。

## <a name="next-step"></a>次のステップ
> [!div class="nextstepaction"]
> [ボットを保存します。](conversation-designer-save-bot.md)

## <a name="additional-resources"></a>その他のリソース
* [タスク](conversation-designer-tasks.md)について確認します。
* [Bot Builder SDK for Node](../nodejs/index.md) について確認します。 
