---
title: ユーザー データを保持する | Microsoft Docs
description: Bot Builder SDK で、ユーザー状態データをストレージに保存する方法について説明します。
keywords: ユーザー データを保持する、ストレージ、会話データ
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 28377a1e611464012df28d3edf78d1cf01351345
ms.sourcegitcommit: 44f100a588ffda19c275b118f4f97029f12d1449
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/26/2018
ms.locfileid: "42928370"
---
# <a name="persist-user-data"></a>ユーザー データを保持する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ボットがユーザーに入力を要求した場合、それは一部の情報を何らかの形式のストレージに保持する機会になります。 Bot Builder SDK を使用すると、*インメモリ ストレージ*、*ファイル ストレージ*、または *CosmosDB* や *SQL* などのデータベース ストレージを使用してユーザー入力を格納できます。ここで、ローカル ストレージの種類は主にテストまたはプロトタイプ作成に使用され、後者のストレージの種類は運用ボットに最適です。

このチュートリアルでは、ストレージ オブジェクトを定義し、保持できるようにユーザー入力をストレージ オブジェクトに保存する方法を示します。 

> [!NOTE]
> 使用することを選択したストレージの種類には関係なく、それを接続してデータを保存するためのプロセスは同じです。 このチュートリアルでは、データを保持するためのストレージとして `FileStorage` を使用します。
> 状態やその他のストレージの種類の詳細については、[会話とユーザー状態の管理](bot-builder-howto-v4-state.md)に関するページを参照してください。

## <a name="prequisite"></a>前提条件 

このチュートリアルは、[テーブルの予約](bot-builder-tutorial-waterfall.md)のチュートリアルに基づいて構築されています。 そのチュートリアルでは、ユーザーにレストランでのテーブル予約に関する 3 つの情報の入力を要求するボットを構築します。 ただし、ユーザーの入力は保持されませんでした。 このチュートリアルでは、そのボットにデータ ストレージ保持を追加します。

## <a name="add-storage-to-middleware-layer"></a>ミドルウェア レイヤーにストレージを追加する

Bot Builder V4 SDK は、状態マネージャー ミドルウェアを使用して状態とストレージを処理します。 このミドルウェアは、基になるストレージの種類には関係なく、単純なキー値ストアを使用してプロパティにアクセスできる抽象化レイヤーを提供します。 基になるストレージの種類がインメモリ、ファイル ストレージ、または Azure テーブル ストレージのいずれであるかにかかわらず、状態マネージャーはストレージへのデータの書き込みと同時実行の管理を管理します。 このチュートリアルでは、`FileStorage` を使用してユーザー入力を保持します。

`FileStorage` プロバイダーには、`bot-builder` パッケージが付属しています。 使用を開始したサンプルでは、`MemoryStorage` プロバイダーが使用されます。 そのストレージの種類は、ボットが再起動されると破棄される、ボットの揮発性*インメモリ*を使用しています。 これに対して、`FileStorage` ライブラリはデータベースと同様に動作します。 つまり、このライブラリは、ローカル コンピューター上のファイルにストレージ情報を書き込みます。 このストレージ ファイルは、後で検査できるように、その配置場所を指定できます。

`FileStorage` を使用するには、`conversationState` オブジェクトが定義されているボット内のステートメントを見つけ、`new botbuilder.FileStorage("c:/temp")` が作成されるようにそれを更新します。 また、このストレージ ファイルを書き込む場所を定義することもできます。 これにより、それを容易に見つけて、保持されたもののコンテンツを検査できます。

# <a name="ctabcstab"></a>[C#](#tab/cstab)
```cs
var storage = new FileStorage("c:/temp");

// These two classes are simply Dictionaries to store state
options.Middleware.Add(new ConversationState<MyBot.convoState>(storage));
options.Middleware.Add(new UserState<MyBot.userState>(storage));
```

Bot Builder SDK には、選択可能な異なるスコープを持つ 3 つの状態オブジェクトが用意されています。

| State | Scope (スコープ) | 説明 |
| ---- | ---- | ---- |
| `dc.ActiveDialog.State` | ダイアログ | ウォーターフォール ダイアログの手順で使用できる状態。 |
| `ConversationState` | 会話 | 現在の会話で使用できる状態。 |
| `UserState` | user | 複数の会話にまたがって使用できる状態。 |

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

**app.js**
```javascript
// Storage
const storage = new FileStorage("c:/temp");
const convoState = new ConversationState(storage);
const userState  = new UserState(storage);
adapter.use(new BotStateSet(convoState, userState));
```

`BotStateSet` は、`ConversationState` と `UserState` の両方を同時に管理できます。 ユーザー データを保存する時間が来たら、選択できます。 Bot Builder SDK には、選択可能な異なるスコープを持つ 3 つの状態オブジェクトが用意されています。

| State | Scope (スコープ) | 説明 |
| ---- | ---- | ---- |
| `dc.activeDialog.state` | ダイアログ | ウォーターフォール ダイアログの手順で使用できる状態。 |
| `ConversationState` | 会話 | 現在の会話で使用できる状態。 |
| `UserState` | user | 複数の会話にまたがって使用できる状態。 |

---
 

## <a name="persist-state"></a>状態を保持する

保存している内容と、保持する必要のある期間に応じて、ボットでこれらの 3 つの状態のいずれの場所にも書き込むことができます。  

# <a name="ctabcstab"></a>[C#](#tab/cstab)

*状態マネージャー ミドルウェア*は、各ターンの最後に状態の書き込みを自動的に処理します。 そのため、ボットで行う必要があるのは、選択した状態オブジェクトにデータを割り当てることだけです。 この例では、`dc.ActiveDialog.State` を使用して予約のためのユーザー入力を追跡します。 これにより、ユーザー入力をグローバル変数に保存する代わりに、一時的な状態オブジェクト スコープでダイアログに格納できます。 このオブジェクトは、ダイアログがアクティブである間のみ存在します。それ以上保持する場合は、その他のいずれかの状態オブジェクトに転送する必要があります。 この場合は、ウォーターフォールの最後の手順で、予約 `msg` を会話状態に割り当てます。

```cs
dialogs.Add("reserveTable", new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        // Prompt for the guest's name.
        await dc.Context.SendActivity("Welcome to the reservation service.");

        dc.ActiveDialog.State = new Dictionary<string, object>();

        await dc.Prompt("dateTimePrompt", "Please provide a reservation date and time.");
    },
    async(dc, args, next) =>
    {
        var dateTimeResult = ((DateTimeResult)args).Resolution.First();

        dc.ActiveDialog.State["date"] = Convert.ToDateTime(dateTimeResult.Value);
        
        // Ask for next info
        await dc.Prompt("partySizePrompt", "How many people are in your party?");

    },
    async(dc, args, next) =>
    {
        dc.ActiveDialog.State["partySize"] = (int)args["Value"];

        // Ask for next info
        await dc.Prompt("textPrompt", "Who's name will this be under?");
    },
    async(dc, args, next) =>
    {
        dc.ActiveDialog.State["name"] = args["Text"];
        string msg = "Reservation confirmed. Reservation details - " +
        $"\nDate/Time: {dc.ActiveDialog.State["date"].ToString()} " +
        $"\nParty size: {dc.ActiveDialog.State["partySize"].ToString()} " +
        $"\nReservation name: {dc.ActiveDialog.State["name"]}";

        var convo = ConversationState<convoState>.Get(dc.Context);

        // In production, you may want to store something more helpful
        convo[$"{dc.ActiveDialog.State["name"]} reservation"] = msg;

        await dc.Context.SendActivity(msg);
        await dc.End();
    }
});
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

*状態マネージャー ミドルウェア*は、各ターンの最後にファイルへの状態の書き込みを自動的に処理します。 そのため、ボットで行う必要があるのは、選択した状態オブジェクトにデータを割り当てることだけです。 この例では、`dc.activeDialog.state` を使用して `reservervationInfo` オブジェクトへのユーザー入力を追跡します。 これにより、ユーザー入力をグローバル変数に保存する代わりに、一時的な状態オブジェクト スコープでダイアログに格納できます。 このオブジェクトは、ダイアログがアクティブである間のみ存在するため、それを保持する場合は、その他のいずれかの状態オブジェクトに転送する必要があります。 この場合は、ウォーターフォールの最後の手順で、`reservationInfo` を `convo` 状態に割り当てます。

```javascript
// Reserve a table:
// Help the user to reserve a table

dialogs.add('reserveTable', [
    async function(dc, args, next){
        await dc.context.sendActivity("Welcome to the reservation service.");

        dc.activeDialog.state.reservationInfo = {}; // Clears any previous data
        await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
    },
    async function(dc, result){
        dc.activeDialog.state.reservationInfo.dateTime = result[0].value;

        // Ask for next info
        await dc.prompt('partySizePrompt', "How many people are in your party?");
    },
    async function(dc, result){
        dc.activeDialog.state.reservationInfo.partySize = result;

        // Ask for next info
        await dc.prompt('textPrompt', "Who's name will this be under?");
    },
    async function(dc, result){
        dc.activeDialog.state.reservationInfo.reserveName = result;
        
        // Persist data
        var convo = conversationState.get(dc.context);; // conversationState.get(dc.context);
        convo.reservationInfo = dc.activeDialog.state.reservationInfo;

        // Confirm reservation
        var msg = `Reservation confirmed. Reservation details: 
            <br/>Date/Time: ${dc.activeDialog.state.reservationInfo.dateTime} 
            <br/>Party size: ${dc.activeDialog.state.reservationInfo.partySize} 
            <br/>Reservation name: ${dc.activeDialog.state.reservationInfo.reserveName}`;
            
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

---

これで、このコードをボット ロジックに組み込む準備ができました。

## <a name="start-the-dialog"></a>ダイアログを開始する

ここで行う必要のあるコードの変更はありません。 単純にボットを実行し、メッセージを送信して `reserveTable` の会話を開始します。

## <a name="check-file-storage-content"></a>ファイル ストレージのコンテンツをチェックする

ボットを実行し、`reserveTable` の会話が終了したら、指定した場所 ("C:/temp" など) にあるファイルに保存された情報を見つけます。 ファイル名の先頭には、"conversation!"  または "user!" のどちらかが付加されています。 最新のファイルが簡単に見つかるように、ファイルを日付で並べ替えると役立つことがあります。

## <a name="next-steps"></a>次の手順

これで、ユーザー入力を保存する方法が分かったので、プロンプト ライブラリを使用して、ユーザーにどのような種類の入力を要求できるかを見てみましょう。

> [!div class="nextstepaction"]
> [ユーザーに入力を要求する](~/v4sdk/bot-builder-prompts.md)
