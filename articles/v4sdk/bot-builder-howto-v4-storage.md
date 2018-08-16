---
title: データを格納する | Microsoft Docs
description: Bot Builder SDK for .NET の V4 でストレージに直接書き込む方法について説明します。
keywords: ストレージ, 読み取りと書き込み, メモリ ストレージ, eTag
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/2/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 653ec6a1983dd59c485a91b2c08ea07d9f2a34c8
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39302828"
---
# <a name="save-data-directly-to-storage"></a>データをストレージに直接保存する

<!--
 Note for V4: You can write directly to storage without using the state manager. Therefore, this topic isn't called "managing state". State is in a separate topic.
 ## Manage state data by writing directly to storage
-->
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ストレージ オブジェクトを直接読み書きすることにより、コンテキスト オブジェクトまたはミドルウェアを使わずに、ストレージに直接書き込むことができます。

このコード例では、ストレージのデータを読み書きする方法を示します。 この例ではストレージはファイルですが、ストレージ オブジェクトを初期化して代わりに Azure Table Storage のメモリ ストレージを使うように、コードを簡単に変更できます。 

# <a name="ctabcsharpechorproperty"></a>[C#](#tab/csharpechorproperty)
オブジェクトを定義し、`IStorage.Write` と `IStorage.Read` を使って状態を保存および取得します。 

次のサンプルでは、ユーザーからのすべてのメッセージをリストに追加します。 リストを含むデータ構造は、ユーザーが `FileStorage` コンストラクターに指定したディレクトリ内のファイルに保存されます。

BotBuilder V4 SDK の EchoBot Visual Studio テンプレートを使って始めます。
`EchoBot.cs` ファイルを編集します。 次の using ステートメントを追加し、`EchoBot` クラスの定義を置き換えます。

```csharp
using System.Collections.Generic;
using System.Linq;
```

```csharp
// In the constructor initialize file storage
public class EchoBot : IBot
{
    private readonly FileStorage _myStorage;

    public EchoBot()
    {
        _myStorage = new FileStorage(System.IO.Path.GetTempPath());
    }

    // Add a class for storing a log of utterances (text of messages) as a list
    public class UtteranceLog : IStoreItem
    {
        // A list of things that users have said to the bot
        public List<string> UtteranceList { get; private set; } = new List<string>();

        // The number of conversational turns that have occurred        
        public int TurnNumber { get; set; } = 0;

        public string eTag { get; set; } = "*";
    }

    // Replace the OnTurn in EchoBot.cs with the following:
    public async Task OnTurn(ITurnContext context)
    {
        var activityType = context.Activity.Type;

        await context.SendActivity($"Activity type: {context.Activity.Type}.");

        if (activityType == ActivityTypes.Message)
        {
            // *********** begin (create or add to log of messages)
            var utterance = context.Activity.Text;
            bool restartList = false;

            if (utterance.Equals("restart"))
            {
                restartList = true;
            }

            // Attempt to read the existing property bag
            UtteranceLog logItems = null;
            try
            {
                logItems = _myStorage.Read<UtteranceLog>("UtteranceLog").Result?.FirstOrDefault().Value;
            }
            catch (System.Exception ex)
            {
                await context.SendActivity(ex.Message);
            }

            // If the property bag wasn't found, create a new one
            if (logItems is null)
            {
                try
                {
                    // add the current utterance to a new object.
                    logItems = new UtteranceLog();
                    logItems.UtteranceList.Add(utterance);

                    await context.SendActivity($"The list is now: {string.Join(", ",logItems.UtteranceList)}");

                    var changes = new KeyValuePair<string, object>[]
                    {
                        new KeyValuePair<string, object>("UtteranceLog", logItems)
                    };
                    await _myStorage.Write(changes);
                }
                catch (System.Exception ex)
                {
                    await context.SendActivity(ex.Message);
                }
            }
            // logItems.ContainsKey("UtteranceLog") == true, we were able to read a log from storage
            else
            {
                // Modify its property
                if (restartList)
                {
                    logItems.UtteranceList.Clear();
                }
                else
                {
                    logItems.UtteranceList.Add(utterance);
                    logItems.TurnNumber++;
                }

                await context.SendActivity($"The list is now: {string.Join(", ", logItems.UtteranceList)}");

                var changes = new KeyValuePair<string, object>[]
                {
                        new KeyValuePair<string, object>("UtteranceLog", logItems)
                };
                await _myStorage.Write(changes);
            }
        }

        return;
    }
}
```
# <a name="javascripttabjsechoproperty"></a>[JavaScript](#tab/jsechoproperty)

次のサンプルでは、ユーザーからのすべてのメッセージをリストに追加します。 リストを含むデータ構造は、ユーザーが `FileStorage` コンストラクターに指定したディレクトリ内のファイルに保存されます。

次のコードを `app.js` に貼り付けます。
``` javascript
const { BotFrameworkAdapter, FileStorage, MemoryStorage, ConversationState, BotStateSet } = require('botbuilder');
const restify = require('restify');

// Create server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter.
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Add storage.
var storage = new FileStorage("C:/temp");
const conversationState = new ConversationState(storage);
adapter.use(new BotStateSet(conversationState));

// Listen for incoming activities.
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing.
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            let utterance = context.activity.text;
            let storeItems = await storage.read(["UtteranceLog"])
            try {
                // check result
                var utteranceLog = storeItems["UtteranceLog"];
                
                if (typeof (utteranceLog) != 'undefined') {
                    // log exists so we can write to it
                    storeItems["UtteranceLog"].UtteranceList.push(utterance);
                    
                    await storage.write(storeItems)
                    try {
                        context.sendActivity('Successful write.');
                    } catch (err) {
                        context.sendActivity(`Srite failed of UtteranceLog: ${err}`);
                    }

                } else {
                    context.sendActivity(`need to create new utterance log`);
                    storeItems["UtteranceLog"] = { UtteranceList: [`${utterance}`], "eTag": "*" }
                    await storage.write(storeItems)
                    try {
                        context.sendActivity('Successful write.');
                    } catch (err) {
                        context.sendActivity(`Write failed: ${err}`);
                    }
                }
            } catch (err) {
                context.sendActivity(`Read rejected. ${err}`);
            };

            await context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```
---

## <a name="manage-concurrency-using-etags"></a>eTag を使用して同時実行を管理する

前の例では、`eTag` プロパティを `*` に設定しました。 ストア オブジェクトの `eTag` (エンティティ タグ) メンバーは、同時実行を管理するためのものです。 `eTag` は、自分のボットが書き込んでいるストレージ内のオブジェクトをボットの別のインスタンスが変更した場合の対処方法を示します。 

<!-- define optimistic concurrency -->

### <a name="last-write-wins---allow-overwrites"></a>最後の書き込みが有効 - 上書きを許可する

前に書き込まれたデータをボットの他のインスタンスが上書きするのを許可するには、eTag を `*` に設定します。 `eTag` プロパティの値にアスタリスク (`*`) を指定すると、最後の書き込みが有効であることを示します。 新しいデータ ストアを作成するときは、プロパティの `eTag` を `*` に設定して、以前に保存したことのないデータを書き込んでいること、または以前に保存されたすべてのプロパティを最後の書き込みで上書きすることを示します。 同時実行がボットにとって問題にならない場合は、書き込んでいるすべてのデータについて `eTag` プロパティを `*` に設定して上書きを許可します。

このコードの例を次に示します。

# <a name="ctabcsetagoverwrite"></a>[C#](#tab/csetagoverwrite)
前に定義した `UtteranceLog` クラスを使います。
```csharp
// Add the current utterance to a new object and save it to storage.
logItems = new UtteranceLog();
logItems.UtteranceList.Add(utterance);
logItems.eTag = "*";

var changes = new KeyValuePair<string, object>[]
{
    new KeyValuePair<string, object>("UtteranceLog", logItems)
};
await _myStorage.Write(changes);

```
# <a name="javascripttabjstagoverwrite"></a>[JavaScript](#tab/jstagoverwrite)
新しい発話をログに追加し、上書きを許可します。

```javascript
storeItems["UtteranceLog"] = { UtteranceList: [`${utterance}`], "eTag": "*" }
await storage.write(storeItems)
```
---

### <a name="maintain-concurrency-and-prevent-overwrites"></a>同時実行を維持して上書きを禁止する
プロパティへの同時アクセスを禁止し、変更をボットの別のインスタンスによって上書きされないようにする場合は、`*` 以外の値を `eTag` に使います。 ボットが状態データを保存しようとして、`eTag` がストレージ内の値と同じでない場合、ボットは `etag conflict key=` というメッセージのエラー応答を受け取ります。 <!-- To control concurrency of data that is stored using `IStorage`, the BotBuilder SDK checks the entity tag (ETag) for `Storage.Write()` requests. -->

既定では、ストアはボットがストレージ オブジェクトを書き込むたびにその項目の `eTag` プロパティが等しいかどうかを確認し、各書き込みの後で新しい一意の値に更新します。 書き込みの `eTag` プロパティがストレージの `eTag` と一致しない場合は、別のボットまたはスレッドがデータを変更したことを意味します。 

たとえば、保存されたメモをボットで編集しますが、ボットの別のインスタンスが行った変更を上書きしないようにする必要があるものとします。 ボットの別のインスタンスが編集を行った場合は、ユーザーに最新の更新でバージョンを編集させます。

# <a name="ctabcsetag"></a>[C#](#tab/csetag)
最初に、`IStoreItem` を実装するクラスを作成します。
```csharp
public class Note : IStoreItem
{
    public string Name { get; set; }
    public string Contents { get; set; }
    public string eTag { get; set; }
}
```
次に、ストレージ オブジェクトを作成することによって最初のメモを作成し、オブジェクトをストアに追加します。
```csharp
// create a note for the first time, with a non-null, non-* eTag.
var note = new Note { Name = "Shopping List", Contents = "eggs", eTag = "x" };

var changes = new KeyValuePair<string, object>[]
{
    new KeyValuePair<string, object>("Note", note)
};
await NoteStore.Write(changes);
```
後でメモにアクセスして更新し、ストアから読み取ったその `eTag` を維持します。
```csharp
var note = NoteStore.Read<Note>("Note").Result.FirstOrDefault().Value;

if (note != null)
{
    note.Contents += ", bread";
    var changes = new KeyValuePair<string, object>[]
    {
        new KeyValuePair<string, object>("Note1", note)
    };
    await NoteStore.Write(changes);
}
```
変更を書き込む前にストアのメモが更新されていた場合、`Write` の呼び出しでは例外がスローされます。

# <a name="javascripttabjsetag"></a>[JavaScript](#tab/jsetag)

最初に、`myNote` オブジェクトを作成します。
```javascript
var myNote = {
    name: "Shopping List",
    contents: "eggs",
    eTag: "*"
}
```
次に、`changes` オブジェクトを初期化し、"*メモ*" をそれに追加してから、ストレージに書き込みます。

```javascript
// Write a note
var changes = {};
changes["Note"] = myNote;
await storage.write(changes);
```
後でメモにアクセスして更新し、ストアから読み取ったその `eTag` を維持します。
```javascript
// Read in a note
var note = await storage.read(["Note"]);
try {
    // Someone has updated the note. Need to update our local instance to match
    if(myNote.eTag != note.eTag){
        myNote = note;
    }

    // Add any updates to the note and write back out
    myNote.contents += ", bread";   // Add to the note
    changes["Note"] = note;
    await storage.write(changes); // Write the changes back to storage
    try {
        context.sendActivity('Successful write.');
    } catch (err) {
        context.sendActivity(`Write failed: ${err}`);
    }
}
catch (err) {
    context.sendActivity(`Unable to read the Note: ${err}`);
}
```
変更を書き込む前にストアのメモが更新されていた場合、`Write` の呼び出しでは例外がスローされます。

---

同時実行を維持するには、常にストレージからプロパティを読み取った後、読み取ったプロパティを変更して、`eTag` が維持されるようにします。 ストアからユーザー データを読み取る場合、応答には eTag プロパティが含まれます。 データを変更して更新後のデータをストアに書き込む場合、要求に含まれる eTag プロパティでは、前に読み取ったのと同じ値が指定されている必要があります。 ただし、`eTag` を `*` に設定してオブジェクトを書き込むと、書き込みで他の変更を上書きできます。

<!-- If the ETag specified in your `Storage.Write()` request matches the current value in the store, the server will save the data and specify a new eTag value in the body of the response, that indicates that the data has been updated. If the ETag specified in your Storage.Write() request does not match the current value in the store, the bot responds with an error indicating an eTag conflict, to indicate that the user's data in the store has changed since you last saved or retrieved it. -->

<!-- TODO: new snippet -->

<!-- jf: I think this section can be cut entirely.

## Save a conversation reference using storage

You can use IStorage to save BotBuilder SDK objects as well as user-defined data. This code snippet uses `Storage.Write()` to save a ConversationReference object for use in sending a proactive message later.

# [C#](#tab/csharpwriteconvref)
```csharp
            var reference = context.ConversationReference;
            var userId = reference.User.Id;

            // save the ConversationReference to a global variable of this class
            conversationReference = reference;

            StoreItems storeItems = new StoreItems();
            StoreItem conversationReferenceToStore = new StoreItem();
            // set the eTag to "*" to indicate you're overwriting previous data
            conversationReferenceToStore.eTag = "*";
            conversationReferenceToStore.Add("ref", reference);
            storeItems[$"ConversationReference/{userId}"] = conversationReferenceToStore;

            await storage.Write(storeItems).ConfigureAwait(false);

            SubscribeUser(userId);

            await context.SendActivity("Thank You! We will message you shortly.");
        }

public void SubscribeUser(string userId)
{
    CreateContextForUserAsync(userId, async (ITurnContext context) =>
    {
        await context.SendActivity("You've been notified.");
        await Task.Delay(2000);
    });
                
}
```
# [JavaScript](#tab/jswriteconvref)
```javascript
const { MemoryStorage } = require('botbuilder');

const storage = new MemoryStorage();

// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const utterances = (context.activity.text || '').trim().toLowerCase()
            if (utterances === 'subscribe') {
                const reference = context.activity;
                const userId = reference.id;
                const changes = {};
                changes['reference/' + userId] = reference;
                await storage.write(changes)
                await subscribeUser(userId)
                await context.sendActivity(`Thank You! We will message you shortly.`);
               
            } else{
                await context.sendActivity("Say 'subscribe'");
            }
    
        }
    });
});
```
---

To read the saved object from storage, call `Storage.Read()`.

# [C#](#tab/csharpread)
```csharp


        private async void CreateContextForUserAsync(String userId,Func<ITurnContext, Task> onReady)
        {
            var referenceKey = $"ConversationReference/{userId}";
            
            ConversationReference localRef = null;
            StoreItems conversationReferenceFromStorage = await storage.Read(referenceKey);
            foreach (var item in conversationReferenceFromStorage)
            {
                StoreItem value = item.Value as StoreItem;
                localRef = value["ref"];
            }
            
            await bot.CreateContext(this.conversationReference, onReady);

        }
```
# [JavaScript](#tab/jsread)
```javascript
async function subscribeUser(userId) {
    setTimeout(() => {
        createContextForUser(userId, (context) => {
            context.sendActivity("You have been notified");
        })
    }, 2000);
}

async function createContextForUser(userId, callback) {
    const referenceKey = 'reference/' + userId;
    var rows = await storage.read([referenceKey])
    var reference = await rows[referenceKey]
    await callback(adapter.createContext(reference))
          
}
```
---

-->

## <a name="next-steps"></a>次の手順

ストレージを直接読み書きする方法がわかったので、状態マネージャーを使ってそれを行う方法を確認してください。

> [!div class="nextstepaction"]
> [会話とユーザーのプロパティを使用して状態を保存する](bot-builder-howto-v4-state.md)

## <a name="additional-resources"></a>その他のリソース

- [概念: Bot Builder SDK でのストレージ](bot-builder-storage-concept.md)


