---
title: あいさつダイアログを実装する - Bot Service
description: ダイアログを使用して、会話に参加したユーザーにあいさつします。
keywords: あいさつ, ダイアログ, 会話フロー, ダイアログ セット
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b57e88dfc5133029a2a847a211c11cb187d9492a
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75798501"
---
# <a name="implement-a-greeting-dialog"></a>あいさつダイアログを実装する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ダイアログを使用して、会話に参加したユーザーを歓迎することができます。

ユーザー歓迎の詳細については、[ウェルカム メッセージをユーザーに送信][send-welcome]する方法をご覧ください。

## <a name="prerequisites"></a>前提条件

- [状態の管理][concept-state]、[ダイアログ ライブラリ][concept-dialogs]、[会話を管理する][simple-flow]方法、および[ダイアログ プロンプトを使用してユーザー入力を収集する][prompting]方法に関する知識。
- ??? サンプルのコピー ([**C#** ][cs-sample] または [**JavaScript**][js-sample])。

## <a name="task-as-in-to-do-x-do-these-things"></a>\<task> [as in to do X, do these things]

<!--The key lines of code for this task.
    here are the cool lines that do that.
    just the few lines of implementation without setup.
-->

## <a name="about-the-sample-code"></a>サンプル コードについて

<!--setup & implementation & discussion of the sample code-->

- 必要な追加パッケージ (AI.Luis、Dialogs など)

<!--Any other key elements to get the code to work.
    Include setup for only the bits critical to the task at hand.
    don't go over all the code in the sample.
-->

## <a name="to-test-the-bot"></a>ボットをテストする

1. [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) をインストールします (まだインストールしていない場合)。
1. ご自身のマシンを使ってローカルでサンプルを実行します。
1. 以下に示すように、エミュレーターを起動し、お使いのボットに接続して、メッセージを送信します。

TODO: 新しいスクリーンショットをとります。

<!--![test dialog prompt sample](~/media/emulator-v4/test-dialog-prompt.png)-->

## <a name="discussion-optional"></a>ディスカッション (省略可能)

<!--Might be short and descriptive or include additional code for scenarios not covered in the samples repo
-->

## <a name="addition-information"></a>その他の情報

<!--include cross-linking other articles about the same sample.-->

- 全体的なサンプル (CoreBot 関連の記事) については、"コア ボットについて" ランディング ページにアクセスしてください。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [ユーザーによる割り込みの処理](bot-builder-howto-handle-user-interrupt.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[send-welcome]: bot-builder-send-welcome-message.md

[simple-flow]: bot-builder-dialog-manage-conversation-flow.md
[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: ???
[js-sample]: ???
