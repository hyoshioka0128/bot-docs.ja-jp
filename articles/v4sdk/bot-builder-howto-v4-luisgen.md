---
title: LUIS の型指定された結果を抽出する | Microsoft Docs
description: LUIS を使用して、Bot Builder SDK でエンティティを抽出する方法について説明します。
keywords: 意図, エンティティ, LUISGen, 抽出
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 5/16/17
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 87ab8d3ceb872cdb0342458b24a9756ccb710fb6
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/23/2018
ms.locfileid: "46706988"
---
# <a name="extract-intents-and-entities-using-luisgen"></a>LUISGen を使用した意図とエンティティの抽出

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

LUIS アプリでは、意図の認識に加え、ユーザーの要求を満たすための重要な単語であるエンティティを抽出することもできます。 たとえば、レストランの予約の例では、LUIS アプリはユーザーのメッセージからグループの人数、予約日、レストランの場所などを抽出できます。 


[LUISGen ツール](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUISGen)を使用して、ボットのコードで LUIS からエンティティを簡単に抽出できるクラスを生成できます。

Node.js コマンド ラインで、`luisgen` をグローバル パスにインストールします。
```
npm install -g luisgen
```

# <a name="ctabcs"></a>[C# を選択した場合](#tab/cs)

## <a name="generate-a-luis-results-class"></a>LUIS 結果クラスを生成する

[CafeBot LUIS サンプル](https://aka.ms/contosocafebot-luis)をダウンロードし、ルート フォルダーで LUISGen を実行します。

```
luisgen Assets\LU\models\LUIS\cafeLUISModel.json -cs ContosoCafeBot.CafeLUISModel
```

## <a name="examine-the-generated-code"></a>生成されたコードを調べる
プロジェクトに追加できる **cafeLUISModel.cs** が生成されます。 このファイルには、厳密に型指定された結果を LUIS から取得するための `cafeLuisModel` クラスが用意されています。

このクラスには、LUIS アプリで定義された意図を取得するための列挙型があります。
```cs
public enum Intent {
    Book_Table, 
    Greeting, 
    None, 
    Who_are_you_intent
};
```
また、`Entities` プロパティもあります。 ユーザーのメッセージにはエンティティが複数回出現する可能性があるため、`_Entities` クラスではエンティティの種類ごとに配列が定義されます。 
```cs
public class _Entities
{
    // Simple entities
    public string[] partySize;

    // Built-in entities
    public Microsoft.Bot.Builder.Ai.LUIS.DateTimeSpec[] datetime;
    public double[] number;

    // Lists
    public string[][] cafeLocation;

    // Instance
    public class _Instance
    {
        public Microsoft.Bot.Builder.Ai.LUIS.InstanceData[] partySize;
        public Microsoft.Bot.Builder.Ai.LUIS.InstanceData[] datetime;
        public Microsoft.Bot.Builder.Ai.LUIS.InstanceData[] number;
        public Microsoft.Bot.Builder.Ai.LUIS.InstanceData[] cafeLocation;
    }
    [JsonProperty("$instance")]
    public _Instance _instance;
}
public _Entities Entities;
```

> [!NOTE]
> LUIS では、ユーザーの発話内で、指定された種類の複数のエンティティが検出される可能性があるため、エンティティの種類はすべて配列です。 たとえば、ユーザーが "make reservations for 5pm tomorrow and 9pm next Saturday" と言った場合、`datetime` の結果で "5pm tomorrow" と "9pm next Saturday" の両方が返されます。
>

|エンティティ | type | 例 | メモ |
|-------|-----|------|---|
|partySize| string[]| `four` 人のパーティ| 単純なエンティティでは文字列が認識されます。 この例では、Entities.partySize[0] は `"four"` です。
|Datetime| [DateTimeSpec](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.datetimespec?view=botbuilder-4.0.0-alpha)[]| `9pm tomorrow` に予約| 各 **DateTimeSpec** オブジェクトには、**timex** 形式で指定された時刻の使用可能な値が含まれた timex フィールドがあります。 timex の詳細については、 http://www.timeml.org/publications/timeMLdocs/timeml_1.2.1.html#timex3 を参照してください。認識を実行するライブラリの詳細については、 https://github.com/Microsoft/Recognizers-Text を参照してください。
|number| double[]| `2` 人の子供を含む `four` 人のグループ | `number` では、グループの人数だけでなく、すべての数が識別されます。 <br/> "Party of four which includes 2 children" という発話では、`Entities.number[0]` が 4 で、`Entities.number[1]` が 2 です。
|cafelocation| string[][] | `Seattle` (場所) で予約。| cafeLocation はリスト エンティティであるため、リストの認識されたメンバーが含まれます。 認識されたエンティティが複数のリストのメンバーである場合、これは配列の配列になります。 たとえば、"reservation in Washington" は、ワシントン州のリストとワシントン D.C. のリストに対応する可能性があります。

`_Instance` プロパティでは、認識された各エンティティの詳細に関する [InstanceData](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.instancedata?view=botbuilder-4.0.0-alpha) が提供されます。

## <a name="check-intents-in-your-bot"></a>ボットで意図をチェックする
**CafeBot.cs** で、`OnTurn` 内のコードを見てみましょう。 ボットがどの時点で LUIS を呼び出し、開始するダイアログを決定するために意図をチェックしているかを確認できます。 [`Recognize`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.luisrecognizer?view=botbuilder-4.0.0-alpha#methods) の呼び出しによる LUIS の結果は、`BookTable` ダイアログに引数として渡されます。



```cs
if(!context.Responded)
{
    // call LUIS and get results
    LuisRecognizer lRecognizer = createLUISRecognizer();
    // Use the generated class as the type parameter to Recognize()
    cafeLUISModel lResult = await lRecognizer.Recognize<cafeLUISModel>(utterance, ct);
    Dictionary<string,object> lD = new Dictionary<string,object>();
    if(lResult != null) {
        lD.Add("luisResult", lResult);
    }
    
    // top level dispatch
    switch (lResult.TopIntent().intent)
    {
        case cafeLUISModel.Intent.Greeting:
            await context.SendActivity("Hello!");
            if (userState.sendCards) await context.SendActivity(CreateCardResponse(context.Activity, createWelcomeCardAttachment()));
            break;

        case cafeLUISModel.Intent.Book_Table:
            await dc.Begin("BookTable", lD);
            break;

        case cafeLUISModel.Intent.Who_are_you_intent:
            await context.sendActivity("I'm the Contoso Cafe bot.");
            break;

        case cafeLUISModel.Intent.None:
        default:
            await getQnAResult(context);
            break;
    }
}
```

## <a name="extract-entities-in-a-dialog"></a>ダイアログのエンティティを抽出する

次に、`Dialogs/BookTable.cs` を見てみましょう。 `BookTable` ダイアログには、一連のウォーターフォール ステップが含まれています。各ステップでは、`args` に渡された LUIS の結果内のエンティティがチェックされます。 エンティティが見つからない場合、このダイアログでは `next()` を呼び出すことで、エンティティの要求をスキップします。 エンティティが見つかった場合は、エンティティを要求し、次のウォーターフォール ステップでプロンプトに対するユーザーの答えを受け取ります。

```cs
    Dialogs.Add("BookTable",
        new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                dc.ActiveDialog.State = new Dictionary<string, object>();
                IDictionary<string,object> state = dc.ActiveDialog.State;

                // add any LUIS entities to active dialog state.
                if(args.ContainsKey("luisResult")) {
                    cafeLUISModel lResult = (cafeLUISModel)args["luisResult"];
                    updateContextWithLUIS(lResult, ref state);
                }
                
                // prompt if we do not already have cafelocation
                if(state.ContainsKey("cafeLocation")) {
                    state["bookingLocation"] = state["cafeLocation"];
                    await next();
                } else {
                    await dc.Prompt("choicePrompt", "Which of our locations would you like?", promptOptions);
                }
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("cafeLocation")) {
                    var choiceResult = (FoundChoice)args["Value"];
                    state["bookingLocation"] = choiceResult.Value;
                }
                bool promptForDateTime = true;
                if(state.ContainsKey("datetime")) {
                    // validate timex
                    var inputdatetime = new string[] {(string)state["datetime"]};
                    var results = evaluateTimeX((string[])inputdatetime);
                    if(results.Count != 0) {
                        var timexResolution = results.First().TimexValue;
                        var timexProperty = new TimexProperty(timexResolution.ToString());
                        var bookingDateTime = $"{timexProperty.ToNaturalLanguage(DateTime.Now)}";
                        state["bookingDateTime"] = bookingDateTime;
                        promptForDateTime = false;
                    }
                }
                // prompt if we do not already have date and time
                if(promptForDateTime) {
                    await dc.Prompt("timexPrompt", "When would you like to arrive? (We open at 4PM.)",
                                    new PromptOptions { RetryPromptString = "We only accept reservations for the next 2 weeks and in the evenings between 4PM - 8PM" });
                } else {
                    await next();
                }                       
                
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("datetime")) { 
                    var timexResult = (TimexResult)args;
                    var timexResolution = timexResult.Resolutions.First();
                    var timexProperty = new TimexProperty(timexResolution.ToString());
                    var bookingDateTime = $"{timexProperty.ToNaturalLanguage(DateTime.Now)}";
                    state["bookingDateTime"] = bookingDateTime;
                }
                // prompt if we already do not have party size
                if(state.ContainsKey("partySize")) {
                    state["bookingGuestCount"] = state["partySize"];
                    await next();
                } else {
                    await dc.Prompt("numberPrompt", "How many in your party?");
                }
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("partySize")) {
                    state["bookingGuestCount"] = args["Value"];
                }

                await dc.Prompt("confirmationPrompt", $"Thanks, Should I go ahead and book a table for {state["bookingGuestCount"].ToString()} guests at our {state["bookingLocation"].ToString()} location for {state["bookingDateTime"].ToString()} ?");
            },
            async (dc, args, next) =>
            {
                var dialogState = dc.ActiveDialog.State;

                // TODO: Verify user said yes to confirmation prompt

                // TODO: book the table! 

                await dc.Context.SendActivity($"Thanks, I have {dialogState["bookingGuestCount"].ToString()} guests booked for our {dialogState["bookingLocation"].ToString()} location for {dialogState["bookingDateTime"].ToString()}.");
            }
        }
    );
}

// This helper method updates dialog state with any LUIS results
private void updateContextWithLUIS(cafeLUISModel lResult, ref IDictionary<string,object> dialogContext) {
    if(lResult.Entities.cafeLocation != null && lResult.Entities.cafeLocation.GetLength(0) > 0) {
        dialogContext.Add("cafeLocation", lResult.Entities.cafeLocation[0][0]);
    }
    if(lResult.Entities.partySize != null && lResult.Entities.partySize.GetLength(0) > 0) {
        dialogContext.Add("partySize", lResult.Entities.partySize[0][0]);
    } else {
        if(lResult.Entities.number != null && lResult.Entities.number.GetLength(0) > 0) {
            dialogContext.Add("partySize", lResult.Entities.number[0]);
        }
    }
    if(lResult.Entities.datetime != null && lResult.Entities.datetime.GetLength(0) > 0) {
        dialogContext.Add("datetime", lResult.Entities.datetime[0].Expressions[0]);
    }
}
```
## <a name="run-the-sample"></a>サンプルを実行する

Visual Studio 2017 で `ContosoCafeBot.sln` を開き、ボットを実行します。 [Bot Framework エミュレーター](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator)を使用して、サンプル ボットに接続します。

エミュレーターで、`reserve a table` と言って予約ダイアログを開始します。

![ボットを実行する](media/how-to-luisgen/run-bot.png)

# <a name="typescripttabjs"></a>[TypeScript](#tab/js)

[CafeBot LUIS サンプル](https://aka.ms/contosocafebot-typescript-luis-dialogs)をダウンロードし、ルート フォルダーで LUISGen を実行します。

```
luisgen cafeLUISModel.json -ts CafeLUISModel
```

プロジェクトに追加できる **CafeLUISModel.ts** が生成されます。 生成されたファイル内の型を使用して、LUIS 認識エンジンから型指定された結果を取得できます。


```typescript
// call LUIS and get typed results
await luisRec.recognize(context).then(async (res : any) => 
{    
    // get a typed result
    var typedresult = res as CafeLUISModel;   
    
```

## <a name="pass-the-typed-result-to-a-dialog"></a>型指定された結果をダイアログに渡す

**luisbot.ts** 内のコードを調べます。 `processActivity` ハンドラー内で、ボットから型指定された結果をダイアログに渡します。

```typescript
// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        const isMessage = context.activity.type === 'message';

        // Create dialog context 
        const state = conversationState.get(context);
        const dc = dialogs.createContext(context, state);
            
        if (!isMessage) {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }

        // Check to see if anyone replied. 
        if (!context.responded) {
            await dc.continue();
            // if the dialog didn't send a response
            if (!context.responded && isMessage) {

                
                await luisRec.recognize(context).then(async (res : any) => 
                {    
                    var typedresult = res as CafeLUISModel;                
                    let topIntent = LuisRecognizer.topIntent(res);    
                    switch (topIntent)
                    {
                        case Intents.Book_Table: {
                            await context.sendActivity("Top intent is Book_Table ");                          
                            await dc.begin('reserveTable', typedresult);
                            break;
                        }
                        
                        case Intents.Greeting: {
                            await context.sendActivity("Top intent is Greeting");
                            break;
                        }
    
                        case Intents.Who_are_you_intent: {
                            await context.sendActivity("Top intent is Who_are_you_intent");
                            break;
                        }
                        default: {
                            await dc.begin('default', topIntent);
                            break;
                        }
                    }
    
                }, (err) => {
                    // there was some error
                    console.log(err);
                }
                );                                
            }
        }
    });
});
```

## <a name="check-for-existing-entities-in-a-dialog"></a>ダイアログの既存のエンティティをチェックする

**luisbot.ts** では、`reserveTable` ダイアログによって `SaveEntities` ヘルパー関数が呼び出され、LUIS アプリによって検出されたエンティティがチェックされます。 エンティティが見つかった場合、それらのエンティティはダイアログの状態に保存されます。 ダイアログの各ウォーターフォール ステップでは、エンティティがダイアログの状態に保存されているかどうかをチェックし、保存されていない場合は、保存するよう要求します。

```typescript
dialogs.add('reserveTable', [
    async function(dc, args, next){
        var typedresult = args as CafeLUISModel;

        // Call a helper function to save the entities in the LUIS result
        // to dialog state
        await SaveEntities(dc, typedresult);

        await dc.context.sendActivity("Welcome to the reservation service.");
        
        if (dc.activeDialog.state.dateTime) {
            await next();     
        }
        else {
            await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
        }
    },
    async function(dc, result, next){
        if (!dc.activeDialog.state.dateTime) {
            // Save the dateTimePrompt result to dialog state
            dc.activeDialog.state.dateTime = result[0].value;
        }

        // If we don't have party size, ask for it next
        if (!dc.activeDialog.state.partySize) {
            await dc.prompt('textPrompt', "How many people are in your party?");
        } else {
            await next();
        }
    },
    async function(dc, result, next){
        if (!dc.activeDialog.state.partySize) {
            dc.activeDialog.state.partySize = result;
        }
        // Ask for the reservation name next
        await dc.prompt('textPrompt', "Whose name will this be under?");
    },
    async function(dc, result){
        dc.activeDialog.state.Name = result;

        // Save data to conversation state
        var state = conversationState.get(dc.context);

        // Copy the dialog state to the conversation state
        state = dc.activeDialog.state;

        // TODO: Add in <br/>Location: ${state.cafeLocation}
        var msg = `Reservation confirmed. Reservation details:             
            <br/>Date/Time: ${state.dateTime} 
            <br/>Party size: ${state.partySize} 
            <br/>Reservation name: ${state.Name}`;
            
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

`SaveEntities` ヘルパー関数では、`datetime` エンティティと `partysize` エンティティがチェックされます。 `datetime` エンティティは[事前構築済みのエンティティ](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2)です。

```typescript
// Helper function that saves any entities found in the LUIS result
// to the dialog state
async function SaveEntities( dc: DialogContext<TurnContext>, typedresult) {
    // Resolve entities returned from LUIS, and save these to state
    if (typedresult.entities)
    {
        console.log(`Entities found.`);
        let datetime = typedresult.entities.datetime;

        if (datetime) {
            // Use the first date or time found in the utterance
            if (datetime[0].timex) {
                timexValues = datetime[0].timex;
                // Datetime results from LUIS are represented in timex format:
                // http://www.timeml.org/publications/timeMLdocs/timeml_1.2.1.html#timex3                                
                // More information on the library which does the recognition can be found here: 
                // https://github.com/Microsoft/Recognizers-Text

                if (datetime[0].type === "datetime") {
                    // To parse timex, here you use the resolve and creator from
                    // @microsoft/recognizers-text-data-types-timex-expression
                    // The second parameter is an array of constraints the results must satisfy
                    var resolution = Resolver.evaluate(
                        // array of timex values to evaluate. There may be more than one: "today at 6" can be 6AM or 6PM.
                        timexValues,
                        // constrain results to times between 4pm and 8pm                        
                        [Creator.evening]);
                    if (resolution[0]) {
                        // toNaturalLanguage takes the current date into account to create a friendly string
                        dc.activeDialog.state.dateTime = resolution[0].toNaturalLanguage(new Date());
                        // You can also use resolution.toString() to format the date/time.
                    } else {
                        // time didn't satisfy constraint.
                        dc.activeDialog.state.dateTime = null;
                    }
                } 
                else  {
                    console.log(`Type ${datetime[0].type} is not yet supported. Provide both the date and the time.`);
                }
            }                                                
        }
        let partysize = typedresult.entities.partySize;
        if (partysize) {
            console.log(`partysize entity detected.${partysize}`);
            // use first partySize entity that was found in utterance
            dc.activeDialog.state.partySize = partysize[0];
        }
        let cafelocation = typedresult.entities.cafeLocation;

        if (cafelocation) {
            console.log(`location entity detected.${cafelocation}`);
            // use first cafeLocation entity that was found in utterance
            dc.activeDialog.state.cafeLocation = cafelocation[0][0];
        }
    } 
}
```

## <a name="run-the-sample"></a>サンプルを実行する

1. TypeScript コンパイラをインストールしていない場合は、次のコマンドを使用してインストールします。

```
npm install --global typescript
```

2. サンプルのルート ディレクトリで `npm install` を実行して、ボットを実行する前に依存関係をインストールします。

```
npm install
```

3. ルート ディレクトリから、`tsc` を使用してサンプルをビルドします。 これにより、`luisbot.js` が生成されます。

4. `lib` ディレクトリで `luisbot.js` を実行します。

5. [Bot Framework エミュレーター](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator)を使用してサンプルを実行します。

6. エミュレーターで、`reserve a table` と言って予約ダイアログを開始します。

![ボットを実行する](media/how-to-luisgen/run-bot.png)

---


## <a name="additional-resources"></a>その他のリソース

LUIS の詳細な背景情報については、[Language Understanding](./bot-builder-concept-luis.md) に関する記事をご覧ください。


## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [ディスパッチ ツールを使用して LUIS と QnA を組み合わせる](./bot-builder-tutorial-dispatch.md)


