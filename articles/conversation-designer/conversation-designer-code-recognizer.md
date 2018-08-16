---
title: タスク トリガーとしてのコード認識エンジンの定義 | Microsoft Docs
description: タスク トリガーとしてカスタム コード認識エンジンを使用する方法について説明します。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 6d44cd299e46971b2218bb78d16e454d5df3c1d9
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300980"
---
# <a name="define-code-recognizer-as-task-trigger"></a>タスク トリガーとしてのコード認識エンジンの定義
> [!IMPORTANT]
> Conversation Designer は、一部のお客様はまだご利用いただけません。 Conversation Designer の可用性の詳細については、今年中に提供される予定です。

コード認識エンジンを使用すると、タスクの実行に役立つカスタム スクリプトを記述できます。 正規表現に基づく式評価や他のサービスの呼び出しを使用して、ユーザーの意図の判断に役立てることができます。 カスタム スクリプトによって、タスクをトリガーするように会話ランタイムに指示が行われます。 

## <a name="create-a-script-function"></a>スクリプト関数の作成
認識エンジンとしてスクリプトを使用するには、タスク エディターで、認識エンジンとして [スクリプトの関数] を選択し、関数の名前を指定して、**[Create/view function]/(関数の作成/表示\)** をクリックしてスクリプトの編集を開始します。 スクリプト名を指定せずに **[Create/view function]/(関数の作成/表示\)** をクリックして、空の関数を既定の名前で作成することもできます。 

## <a name="script-trigger-function-parameter"></a>スクリプト トリガー関数のパラメーター

指定したスクリプト コールバック関数は、常に [`context`](conversation-designer-context-object.md) オブジェクトと共に呼び出されます。

`context` オブジェクトには、`taskEntities` と `contextEntities` の両方が含まれます。 タスク エンティティは、このタスクに対して定義または生成されたエンティティであり、コンテキスト エンティティは、ユーザーとの会話の間にエンティティを保持できるプロパティ バッグです。

## <a name="return-value"></a>戻り値

**スクリプト トリガー**関数はブール値を返すことが予期されています。

## <a name="sample-regex-based-recognizer"></a>正規表現に基づく認識エンジンのサンプル
次のコード サンプルでは正規表現を使用して要求を処理し、実行するスクリプト トリガーを会話ランタイムが決定できるように、ブール値を返します。

```javascript
module.exports.fnFindPhoneTrigger = function(context) {
    if (context.request.text.includes("call") || context.request.text.includes("ring")) {
        return true;
    } else {
        return false;
    }
} 
```

## <a name="next-step"></a>次のステップ
> [!div class="nextstepaction"]
> [応答アクション](conversation-designer-reply.md)
