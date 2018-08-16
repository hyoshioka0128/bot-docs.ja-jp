---
title: Do アクションとしての返信の定義 | Microsoft Docs
description: Do アクションとして返信を定義する方法について説明します。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 344a771a55cf729587863a70398ef944863ac738
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39303220"
---
# <a name="define-a-reply-as-a-do-action"></a>Do アクションとしての返信の定義

返信は、表示テキスト、音声テキスト、またはリッチ カードの任意の組み合わせにすることができます。 このアクションでは、ボットはユーザーへの返信を送信してタスクを完了します。 返信は 1 ターンの会話であり、タスクはユーザーからのフォローアップ メッセージを期待しません。 返信を *reply* アクションとして設定するには、**[DO]\(実行\)** アクション ドロップダウンから **[Give a reply]\(返信する\)** を選択します。 その後、単純な応答を **[Bot's response to user]\(ユーザーに対するボットの応答\)** フィールドに入力することができます。

必要に応じて、単純な応答の[アダプティブ カード](conversation-designer-adaptive-cards.md)を含めることができます。 必要に応じて、ユーザーに応答を送信する前に実行するカスタム スクリプト関数を提供することもできます。 これらのオプションは、タスクの作成または編集中に使用できます。 

## <a name="next-step"></a>次のステップ
> [!div class="nextstepaction"]
> [スクリプト アクション](conversation-designer-script-function.md)
