---
title: Do アクションとしてのスクリプト関数の定義 | Microsoft Docs
description: スクリプト アクション関数を Do アクションとして設定する方法について説明します。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 674781c2cb5a8700941feb59b2ce7ab902979d4f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304089"
---
# <a name="define-a-script-function-as-a-do-action"></a>Do アクションとしてのスクリプト関数の定義

[スクリプト アクション関数](conversation-designer-context-object.md#script-callback-functions)によって、タスクを完了するために定義するカスタム スクリプトが実行されます。 たとえば、ユーザーが「サーモスタットを 74 度に設定して」と言うと、実際にサーモスタットを 74 度に設定するためにサービスが呼び出されます。 

アクションとしてカスタム スクリプトを使用するには、タスク エディターで**スクリプト アクション**を、アクションの種類 **DO** として選択します。 次に、アクションを実装する関数名を入力します。 **[編集]** をクリックして**スクリプト** エディターを開き、この関数の実装を記述します。 

## <a name="script-action-function-parameter"></a>スクリプト アクション関数のパラメーター

指定したアクション コールバック関数は、常に [`context`](conversation-designer-context-object.md) オブジェクトと共に呼び出されます。

`context` オブジェクトには、`taskEntities` と `contextEntities` の両方が含まれます。 タスク エンティティはこのタスクに対して定義または生成されるエンティティであり、コンテキスト エンティティはユーザーとの会話と会話の間にエンティティを保持できるプロパティ バッグです。

## <a name="return-value"></a>戻り値
**スクリプト アクション**関数はブール値を返すことが予期されています。

## <a name="sample-script-action-function"></a>サンプル スクリプト アクション関数
次のサンプル関数では、HTTP 呼び出しを行って、一致するトリガーを返す前に応答を受け取ります。

```javascript
module.exports.NewTask_do_onRun = function(context) {
    var options =  {
        host: 'HOST',
        path: 'PATH',
        method: 'post',
        headers: {
            "HEADER1" : "VALUE"
        }, 
        body: {
            "BODY": VALUE
        }
    };

    return http.request(options).then(function(response) {
      // parse response payload
      // Send a message to user
      context.responses.push({text: "Done", type: "message"});
    });
} 
```

## <a name="next-step"></a>次のステップ
> [!div class="nextstepaction"]
> [対話アクション](conversation-designer-dialogues.md)
