---
title: 独自のミドルウェアを作成する | Microsoft Docs
description: 独自のミドルウェアを作成する方法について説明します。
keywords: ミドルウェア, カスタム ミドルウェア, ショート サーキット, フォールバック, アクティビティ ハンドラー
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b774f2de5856e6001d1b75c47b92aff6399d8fe3
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904352"
---
# <a name="create-your-own-middleware"></a>独自のミドルウェアを作成する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ミドルウェアを使用すると、ご利用のボット向けに豊富なプラグインを作成することができ、それらは他でも使用することができます。 ここでは、基本的なミドルウェアの追加および実装の方法と、そのしくみについて紹介します。 v4 SDK には、状態管理、LUIS、QnAMaker、翻訳などの目的に対する何らかのミドルウェアが用意されています。 詳細については、[.NET](https://github.com/Microsoft/botbuilder-dotnet) 用または [JavaScript](https://github.com/Microsoft/botbuilder-js) 用の Bot Builder SDK を参照してください。

## <a name="adding-middleware"></a>ミドルウェアの追加

[作業開始](~/bot-service-quickstart.md)エクスペリエンスで作成された基本的なボット サンプルに基づく次の例では、2 つの異なるミドルウェアが、それらの各クラスの新しいインスタンスと共に Microsoft のサービスに追加されています。

> [!IMPORTANT]
> それらがオプションに追加される順番によって、それらが実行される順番が決まることにご注意ください。 ミドルウェアを複数使用する場合に、それがどのように機能するかを必ず検討してください。

**Startup.cs**

# <a name="ctabcsaddmiddleware"></a>[C#](#tab/csaddmiddleware)

追加するミドルウェアごとに、ご利用のボット サービス オプションに `options.Middleware.Add(new MyMiddleware());` メソッド呼び出しを追加します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<HelloBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        options.Middleware.Add(new MyMiddleware());
        options.Middleware.Add(new MyOtherMiddleware());
    });
}
```
# <a name="javascripttabjsaddmiddleware"></a>[JavaScript](#tab/jsaddmiddleware)

追加するミドルウェアごとに、ご利用のアダプターに `adapter.use(MyMiddleware());` を追加します。

```javascript
// Create adapter
const adapter = new botbuilder.BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

adapter.use(MyMiddleware());
adapter.use(MyOtherMiddleware());
```

---


## <a name="implementing-your-middleware"></a>ミドルウェアの実装

各ミドルウェアは middlware インターフェイスから継承され、その処理ハンドラーが常に実装されます。これはボットに送信されるすべてのアクティビティに対して実行されます。 追加された各ミドルウェアについては、他のミドルウェアまたはボット ロジックにコンテキスト オブジェクトとのやり取りを許可する前に、パイプライン処理の継続時に処理ハンドラーによって、コンテキスト オブジェクトが変更されたりタスク (ログ記録など) が実行されたりする可能性があります。

# <a name="ctabcsetagoverwrite"></a>[C#](#tab/csetagoverwrite)

各ミドルウェアは `IMiddleware` から継承され、常に `OnTurn()` が実装されます。

**ExampleMiddleware.cs**
```csharp
public class MyMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {            
        // This simple middleware reports the request type and if we responded
        await context.SendActivity($"Request type: {context.Activity.Type}");
        
        await next();            

        // Report if any responses were recorded
        string response = context.Responded ? "yes" : "no";
        await context.SendActivity($"Responded?  {response}");
    }
}

public class MyOtherMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        // simple middleware to add an additional send activity
        await context.SendActivity($"My other middleware just saying Hi before the bot logic");

        await next();
    }
}

```

# <a name="javascripttabjsimplementmiddleware"></a>[JavaScript](#tab/jsimplementmiddleware)

各ミドルウェアは `MiddlewareSet` から継承され、常に `onTurn()` が実装されます。

**ExampleMiddleware.js**
```js
adapter.use({onTurn: async (context, next) =>{

    // This simple middleware reports the activity type and if we responded
    await context.sendActivity(`Activity type: ${context.activity.type}`); 
    await next();            

    // Report if any responses were recorded
    const response = context.activity.text ? "yes" : "no";
    await context.sendActivity(`Responded?  ${response}`);

}}, {onTurn: async (context, next) => {

     // simple middleware to add an additional send activity
     await context.sendActivity("My other middleware just saying Hi before the bot logic");

     await next();
}})
```

---

`next()` を呼び出すと、実行が次のミドルウェアに進みます。 実行が渡されるタイミングを選択する機能により、残りのミドルウェア スタックが実行された**後**で実行されるコードを記述することができます。 ボット ロジックおよびその他のミドルウェアが実行された後、処理ハンドラーの "トレーリング エッジ" で操作を実行することができます。  上記の例では、最初に実装されたミドルウェアによってそれが行われており、実行をパイプラインに戻す前に前に、このコンテキスト オブジェクトに対して応答したかどうかが報告されます。

## <a name="short-circuit-routing"></a>ショート サーキットのルーティング

場合によっては、受信したアクティビティのそれ以上の処理を停止したい場合があります。このことをショート サーキットと呼んでいます。 これは、ミドルウェアによって要求が完全に処理される場合、特定のコマンドに対して簡単な応答がミドルウェアによって用意されている場合、あるいはボルト ロジックで確認しなくてもミドルウェアによって受信要求の処理が可能な場合に便利です。

ユーザーが "ping" と発言したときに必ず、応答を送信して、要求のそれ以上のルーティングを阻止するミドルウェアを作成してみましょう。

# <a name="ctabcsmiddlewareshortcircuit"></a>[C#](#tab/csmiddlewareshortcircuit)
```cs
public class ExampleMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        var utterance = context.Activity?.AsMessageActivity()?.Text.Trim().ToLower();

        if (utterance == "ping") 
        {
            context.SendActivity("pong");
            return;
        } 
        else 
        {
            await next();
        }
    }
}
```
# <a name="javascripttabjsmiddlewareshortcircuit"></a>[JavaScript](#tab/jsmiddlewareshortcircuit)
```JavaScript
adapter.use({onTurn: async (context, next) =>{
    const utterance = (context.activity.text || '').trim().toLowerCase();
        if (utterance == "ping") 
        {
            await context.sendActivity("pong");
            return;
        } 
        else 
        {
            await next();
        }

}})

```

---

## <a name="fallback-processing"></a>フォールバック処理

行う必要があると考えられるもう 1 つの処理は、まだ応答されていない要求に対する応答です。 これは、処理ハンドラーのトレーリング エッジを使用して、`context.Responded` プロパティを確認することで、容易に実現できます。 ボットが要求の処理に失敗した場合に、自動的に "私は理解できませんでした" と発言する簡単なミドルウェアを作成してみましょう。

# <a name="ctabcsfallback"></a>[C#](#tab/csfallback)
```cs
public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
{
    await next();

    if (!context.Responded) 
    {
        context.SendActivity("I didn't understand.");
    }
}
```
# <a name="javascripttabjsfallback"></a>[JavaScript](#tab/jsfallback)
```JavaScript
adapter.use({onTurn: async (context, next) =>{
    await next();

    if (!context.responded) 
    {
       await context.sendActivity("I didn't understand.");
    }

}})
```

---

> [!NOTE] 
> これはすべての場合に機能するわけではありません。他のミドルウェアでユーザーに応答できる可能性があったり、ボットによってメッセージは受信されるが、ボットからの応答がない場合などでは機能しません。 "私は理解できません" で応答すると、ユーザーの誤解を招くでしょう。


