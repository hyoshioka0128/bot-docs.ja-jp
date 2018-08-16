---
title: context オブジェクトの API リファレンス | Microsoft Docs
description: Conversation Designer ボット内で context オブジェクトを参照する方法について説明します。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 5ba70189c16815539524d3c9046da03ed0344b9b
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300977"
---
# <a name="api-reference"></a>API リファレンス
> [!IMPORTANT]
> Conversation Designer は、一部のお客様はまだご利用いただけません。 Conversation Designer の可用性の詳細については、今年中に提供される予定です。

Conversation Designer では、ボットにカスタム ビジネス ロジックを追加する機能を提供しています。 これらのスクリプト関数は、**スクリプト** エディターで実装します。 関数は、条件付き応答テンプレート、タスクの "*トリガー*" や "*アクション*"、ダイアログの状態など、さまざまな場所にフックされ、すべて、関数パラメーター内の、型が **[IConversationContext]** の `context` オブジェクトを受け取ります。

次のコード サンプルは、**context** オブジェクトが渡される条件付き応答関数のシグネチャを示しています。

```javascript
/**
* @param {IConversationContext} context
*/
module.exports.NewConditionalResponse_onRun = function(context) {
    // Business logic here
    return true; // Returns a boolean
}
```

`context` オブジェクトを通じて、ユーザーとボットの会話に関する情報にアクセスできます。

## <a name="script-callback-functions"></a>スクリプト コールバック関数

カスタム スクリプト コールバック関数の作成は、さまざまなフォームを取る可能性があります。 これらにはいろいろな名前を付けることができますが、機能的には次のフォームのうちのいずれかとなります。

| フォーム | パラメーター | 戻り値の型 | 説明 |
| ---- | ---- | ---- | ---- |
| before response 関数 | context | **void** または **promise** | 応答が与えられる前に実行される関数。 |
| process 関数 | context | **void** または **promise** | ビジネス ロジックを実行する関数。 |
| decision 関数 | context | **string** または **promise** | ビジネス ロジックに基づいて決定を下す関数。 戻り値の文字列は、[decision](conversation-designer-dialogues.md#decision-state) ブロックの条件に一致する必要があります。 |
| code recognizer 関数 | context | **boolean** または **promise** | **スクリプト トリガー**が発生したときに実行されるカスタム ビジネス ロジック。 一致を示す場合は `true` を返します。 それ以外の場合、一致を取り消すには `false` を返します。 |
| onRecognize 関数 | context | **boolean** または **promise** | LUIS 認識エンジンで一致している場合にのみ実行される関数。 このコールバック関数は、LUIS エンティティを処理し、適切な **boolean** 値を返すために使用します。 一致を示す場合は `true` を返します。 それ以外の場合、一致を取り消すには `false` を返します。 |

## <a name="iconversationcontext-interface"></a>IConversationContext インターフェイス

`IConversationContext` インターフェイスは、ユーザーとボットの会話情報を追跡します。 Conversation Designer を介してフックされているすべてのカスタム関数が、パラメーターの引数として `context` オブジェクトを受け取ります。

## <a name="context-properties"></a>context プロパティ
`context` オブジェクトでは、次のプロパティが公表されます。

| Name |  コード | 説明 |
| ---- | ---- | ---- |
| `request` | `context.request` | ボットのアクティビティを格納する要求オブジェクトを取得します。  |
| | `context.request.attachment` | アダプティブ カードを格納することができる添付アクティビティ。 |
| | `context.request.text` | クライアントからの受信テキスト メッセージを格納するテキスト アクティビティ。 |
| | `context.request.speak` | クライアントからの発話テキスト (該当する場合) を格納する発話アクティビティ。 |
| | `context.request.type` | アクティビティの種類を指定します (既定は "メッセージ")。 |
| `responses` | `context.responses` | 現在の状態またはコード ビハインド実行の最後にクライアントに返すアクティビティの配列を保持します。 |
| | `context.responses.push` | 応答にアクティビティを追加します。 |
| `global` | `context.global` | ユーザーが定義した会話データを格納する JavaScript オブジェクト。 このオブジェクトは、会話を通して保持されます。 |
| `local` | `context.local` | ユーザーが定義したタスク データを格納する JavaScript オブジェクト。 このオブジェクトは、特定のタスクの間保持されます。 LUIS の意図は、常にローカル コンテキストに返されます。 LUIS の結果を保持する必要がある場合は、`context.global` コンテキストにコピーすることを検討してください。 |
| | `context.local['@description']` | LUIS から受信した生のエンティティを返します。 |
| `sticky` | `context.sticky` | 現在のタスク名を示します。 |
| `currentTemplate` | `context.currentTemplate` | 表示と発話の両方の評価のために呼び出される[条件付き応答テンプレート](conversation-designer-response-templates.md#conditional-response-templates)。 このオブジェクトには、3 つのプロパティが含まれています。 <br/>1. **name**: 現在のテンプレートの名前。 <br/>2. **modalityDisplay**: モダリティが表示の評価に関連付けられているかどうかを示すブール値。 <br/>3. **modalitySpeak**: モダリティが発話の評価に関連付けられているかどうかを示すブール値。 |

## <a name="context-methods"></a>context メソッド
`context` オブジェクトでは、次のメソッドが公開されます。

| Name | 戻り値の型 | コード | 説明 |
| ---- | ---- | ---- | ---- |
| `getCurrentTurn` | **number** | `context.getCurrentTurn();` | 再プロンプトを実行している場合は、スタックの一番上のフレームからターンを取得します。 |

## <a name="next-step"></a>次のステップ
> [!div class="nextstepaction"]
> [ボットを作成します](conversation-designer-create-bot.md)
