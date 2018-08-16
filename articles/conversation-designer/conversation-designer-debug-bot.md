---
title: ボットのテスト | Microsoft Docs
description: Conversation Designer ボットのテストとデバッグ。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: b08bd96bc7c413ff7cb6db4899c8c0d3ad613ab8
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304217"
---
# <a name="test-your-conversation-designer-bot"></a>Conversation Designer ボットのテスト
> [!IMPORTANT]
> Conversation Designer は、一部のお客様はまだご利用いただけません。 Conversation Designer の可用性の詳細については、今年中に提供される予定です。

Conversation Designer は、(画面下部にある) 数種類の出力ウィンドウを通じて、有用なデバッグ ツールを提供します。 ウィンドウの右上隅にある **[テスト]** ボタンをクリックして、テストとデバッグを開始できます。 

## <a name="validation-errors"></a>検証エラー
検証エラーが発生するいくつかの条件があります。 例:  
- トリガーへの応答の不足 
- フローの部分的に完了した情報
- 条件応答テンプレート、決定、およびプロセス状態の背後のトリガーおよびコードの不足
- テンプレート参照で再帰的または無限ループが検出される 
- フローに接続していない孤立状態がダイアログに存在する。
- タスクを追加したが、トリガーまたはアクションが不足している 


## <a name="javascript-output"></a>JavaScript 出力
追加の機能を提供するために、応答の実行にスクリプトをアタッチすることができます。 スクリプトにエラーがある場合、ここに表示されます。 `console.log()` または `error.log()` または `debug.log()` をボットのビジネス ロジックに追加できます。これらは出力ウィンドウにメッセージを一覧表示します。 例: 

``` javascript
module.exports.Respond_beforeResponse = function(context) {
    console.log(JSON.stringify(context));
}
```

## <a name="runtime-output"></a>ランタイム出力
ランタイムによって出力されるエラーまたは例外はここに表示されます。 たとえば、ボットが応答に失敗する場合、出力を確認して例外またはエラーの原因を調査することができます。 出力ウィンドウでメッセージが多すぎる場合、**[Clear all]**(すべてクリア) をクリックし、ボットにメッセージを送信してボットをもう一度テストし、エラーの詳細を確認することができます。 

## <a name="next-step"></a>次のステップ
> [!div class="nextstepaction"]
> [Import and export bot](conversation-designer-export-import-bot.md) (ボットのインポートとエクスポート)
