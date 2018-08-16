---
title: イベント ハンドラー | Microsoft Docs
description: イベント ハンドラーの使用方法を理解します。
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 06/14/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0c054b8f4209004806e4564be45e83bdf3a8ec25
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39303641"
---
# <a name="event-handlers"></a>イベント ハンドラー

イベント ハンドラーは、[ターン](bot-builder-basics.md#defining-a-turn)内の今後のアクティビティ イベントに追加できる関数です。 これらのアクティビティは、`SendActivity`、`UpdateActivity`、および`DeleteActivity` であり、それぞれに独自のハンドラーがあります。 これらのハンドラーは、現在のコンテキスト オブジェクトに対して、その種類の今後のすべてのアクティビティに関して何かを行う必要がある場合に便利です。

各アクティビティ イベントに複数のイベント ハンドラーを追加できます。イベント ハンドラーは、それらが追加された**後**で発生するすべてのイベントを処理するため、コードのどこに追加するかが重要です。

> [!NOTE]
> この記事で使用されている "*イベント ハンドラー*" は、従来の言語固有の "*イベント*" と "*ハンドラー*" に定義に従っていません。 それらは、"*イベント*" と "*ハンドラー*" のより概念的なプログラミングの考え方を示しています。

## <a name="using-the-next-delegate"></a>*next* デリゲートの使用

各ハンドラーは、*next* デリゲートを呼び出すことで、ハンドラーが追加された順序で次のハンドラーに実行を渡します。 最後の *next* デリゲートは、アクティビティを実際に送信、更新、または削除するアダプターの呼び出しです。

他の意図が特になければ、ハンドラー内で *next()* を呼び出すことが重要です。それ以外の場合は、アクティビティを[ショート サーキット](bot-builder-create-middleware.md#short-circuit-routing)します。 アクティビティのショート サーキットとミドルウェアの違いは、ショート サーキットはアクティビティを完全にキャンセルしますが、ミドルウェアは実行できるコードを変更しますが実行は続行されるという点です。

アクティビティの送信が成功した場合、*next* デリゲートは、送信されたメッセージに割り当てられていた ID を返します。

## <a name="expected-return-value"></a>期待される戻り値

戻り値について、イベント ハンドラーは、それが next デリゲートの結果であることを期待します。 必要であれば、結果を返す前に表示と保存を実行できます。または、次に示すように直接返すことができます。

`SendActivity` ハンドラーの戻り値は、対応する next デリゲートの戻り値としてチェーンを介して渡される戻り値です。

# <a name="ctabcseventhandler"></a>[C#](#tab/cseventhandler)

用意されている 3 種類のイベント ハンドラーは、`OnSendActivities()`、`OnUpdateActivity()`、および `OnDeleteActivity()` です。 ここでは、`OnSendActivities()` ハンドラーをボットのコードに追加しますが、ハンドラーはミドルウェアのコードにも同じように追加できます。

多くの場合、ハンドラーは、次の例に示すように、ラムダ式として追加されます。 ここでは、**help** と書き込まれたユーザーの入力アクティビティをリッスンします。

```cs
public Task OnTurn(ITurnContext context)
{
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        
        context.OnSendActivities(async (handlerContext, activities, handlerNext) =>
        {
            if (handlerContext.Activity.Text == "help")
            {
                Console.WriteLine("help!");
                // Do whatever logging you want to do for this help message
            }

            return await handlerNext();
        });
        // ...
    }
}
```

# <a name="javascripttabjseventhandler"></a>[JavaScript](#tab/jseventhandler)

用意されている 3 種類のイベント ハンドラーは、`onSendActivities()`、`onUpdateActivity()`、および `onDeleteActivity()` です。 ここでは、`onSendActivities()` ハンドラーをボットのコードに追加しますが、ハンドラーはミドルウェアのコードにも同じように追加できます。

ここでは、**help** と書き込まれたユーザーの入力アクティビティをリッスンします。

```js
adapter.processActivity(req, res, async (context) => {

    if (context.activity.type === 'message') {

        context.onSendActivities(async (handlerContext, activities, handlerNext) => { 
            
            if(handlerContext.activity.text === 'help'){
                console.log('help!')
                // Do whatever logging you want to do for this help message
            }
            // Add handler logic here
        
            await handlerNext(); 
        });
        await context.sendActivity(`you said ${context.activity.text}`);
    }
});
```

---

"*送信アクティビティ*" イベントと "*更新または削除アクティビティ*" イベントを区別することが重要です。前者は完全に新しいアクティビティ イベントを作成し、後者は過去のアクティビティに対して機能します。 さらに、 すべてのチャネルが、"*更新*" または "*削除*" アクティビティをサポートしているわけではありません。 これらに対する呼び出しとハンドラーの周囲に適切な例外処理を追加して、その可能性に対応できるようにすることをお勧めします。

