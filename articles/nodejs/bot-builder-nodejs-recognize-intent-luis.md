---
title: LUIS を使用して意図とエンティティを認識する | Microsoft Docs
description: ボットと LUIS を統合し、Bot Builder SDK for Node.js を使用してダイアログをトリガーすることで、ユーザーの意図を検出して適切に応答します。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/28/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 82a4d0843a9aaab25779d833f2b1b1d2ab2516c2
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905104"
---
# <a name="recognize-intents-and-entities-with-luis"></a>LUIS を使用して意図とエンティティを認識する 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

この記事では、メモ作成ボットの例を使用して、自然言語入力に対してボットが適切に応答するのに Language Understanding ([LUIS][LUIS]) がどのように役立つかを示します。 ボットではユーザーの**意図**を識別することで、ユーザーが何をしたいかを検出します。 この意図は音声またはテキスト入力 (または**発話**) から決定されます。 意図により、ダイアログの呼び出しなど、ボットで行われるアクションに発話がマップされます。 ボットでは、発話の中の重要な言葉である**エンティティ**を抽出する必要もあります。 場合によっては、エンティティで意図を満たす必要があります。 メモ作成ボットの例では、`Notes.Title` エンティティによって各メモのタイトルが識別されます。

## <a name="create-a-language-understanding-bot-with-bot-service"></a>Bot Service での Language Understanding ボットの作成

1. [Azure Portal](https://portal.azure.com) のメニュー ブレードで、**[新しいリソースの作成]** を選択し、**[すべて表示]** をクリックします。

    ![新しいリソースの作成](../media/bot-builder-nodejs-use-luis/bot-service-creation.png)

2. 検索ボックスで、**Web アプリ ボット**を検索します。 

    ![新しいリソースの作成](../media/bot-builder-nodejs-use-luis/bot-service-selection.png)

3. **[ボット サービス]** ブレードで、必要な情報を指定し、**[作成]** をクリックします。 これによって、ボット サービスと LUIS アプリが作成され、Azure にデプロイされます。 
   * **[アプリ名]** にボットの名前を設定します。 この名前は、ボットがクラウドにデプロイされるときに、サブドメインとして使用されます (mynotesbot.azurewebsites.net など)。 この名前は、ボットに関連付けられる LUIS アプリの名前としても使用されます。 後で、ボットに関連付けられた LUIS アプリを探すために使用できるよう、名前をコピーしておきます。
   * サブスクリプション、[リソース グループ](/azure/azure-resource-manager/resource-group-overview)、App Service プラン、[場所](https://azure.microsoft.com/en-us/regions/)を選択します。
   * **[ボット テンプレート]** フィールドで **Language Understanding (Node.js)** テンプレートを選択します。

     ![[ボット サービス] ブレード](../media/bot-builder-nodejs-use-luis/bot-service-setting-callout-template.png)

   * チェック ボックスをオンにしてサービス利用規約に同意します。

4. ボット サービスがデプロイされたことを確認します。
    * [通知] (Azure portal の上端にあるベル アイコン) をクリックします。 通知が、**[デプロイが開始されました]** から **[デプロイメントに成功しました]** に変わります。
    * 通知が **[デプロイメントに成功しました]** に変わったら、その通知で **[リソースに移動]** をクリックします。

## <a name="try-the-bot"></a>ボットを試す

ボットがデプロイされたことを確認するには、**[通知]** をオンにします。 通知が、**[デプロイは進行中です...]** から **[デプロイメントに成功しました]** に変わります。 **[リソースに移動]** ボタンをクリックして、ボットのリソース ブレードを開きます。

ボットが登録されたら、**[Test in Web Chat]\(Web チャットでのテスト\)** をクリックして、Web チャット ウィンドウを開きます。 Web チャットに「hello」と入力します。

  ![Web チャットでのボットのテスト](../media/bot-builder-nodejs-use-luis/bot-service-web-chat.png)

ボットが "あいさつにリーチしました。 hello と言いました" と言って応答します。 これにより、ご自身のメッセージはボットによって受信され、そのボットが作成した既定の LUIS アプリに渡されたことが確認されます。 この既定の LUIS アプリによって、あいさつの意図が検出されました。

## <a name="modify-the-luis-app"></a>LUIS アプリを変更する

Azure へのログインに使用するものと同じアカウントを使用して、[https://www.luis.ai](https://www.luis.ai) にログインします。 **[My apps]\(マイ アプリ\)** をクリックします。 アプリの一覧で、ボット サービスの作成時に **[ボット サービス]** ブレードの **[アプリ名]** で指定した名前で始まるアプリを探します。 

LUIS アプリは 4 つの意図 (Cancel、Greeting、Help、None) で始まります。 <!-- picture -->

次の手順では、Note.Create、Note.ReadAloud、および Note.Delete の各意図を追加します。 

1. ページの左下にある **[Prebuit Domains]\(事前構築済みドメイン\)** をクリックします。 **Note** ドメインを探して **[ドメインの追加]** をクリックします。

2. このチュートリアルでは、事前構築済みの **Note** ドメインに含まれるすべての意図を使用するわけではありません。 **[Intents]\(意図\)** ページで、次の各意図名をクリックしてから **[Delete Intent]\(意図の削除\)** ボタンをクリックします。
   * Note.ShowNext
   * Note.DeleteNoteItem
   * Note.Confirm
   * Note.Clear
   * Note.CheckOffItem
   * Note.AddToNote

   次の意図だけを LUIS アプリに残してください。 
   * Note.ReadAloud
   * Note.Create
   * Note.Delete
   * なし
   * [Help]
   * Greeting
   * Cancel

     ![LUIS アプリに表示される意図](../media/bot-builder-nodejs-use-luis/luis-intent-list.png)


3.  右上の **[トレーニング]** ボタンをクリックして、ご利用のアプリをトレーニングします。
4.  上部のナビゲーション バーの **[PUBLISH]\(発行\)** をクリックして、**[発行]** ページを開きます。 **[Publish to production slot]\(運用スロットに発行\)** ボタンをクリックします。 発行が正常に行われた後、**[Publish App]\(アプリの発行\)** ページの **[エンドポイント]** 列 (リソース名 Starter_Key で始まる行) に表示されている URL に、LUIS アプリがデプロイされます。 URL の形式は次の例のようになります。`https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx?subscription-key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&timezoneOffset=0&verbose=true&q=` この URL のアプリ ID およびサブスクリプション キーは、**[アプリ サービスの設定] > [ApplicationSettings] > [アプリ設定]** の LuisAppId および LuisAPIKey と同じです。


## <a name="modify-the-bot-code"></a>ボット コードの変更

**[Build]\(ビルド\)**、**[Open online code editor]\(オンライン コード エディターを開く\)** の順にクリックします。

   ![オンライン コード エディターを開く](../media/bot-builder-nodejs-use-luis/bot-service-build.png)

コード エディターで、`app.js` を開きます。 これには以下のコードが含まれます。

```javascript
var restify = require('restify');
var builder = require('botbuilder');
var botbuilder_azure = require("botbuilder-azure");

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log('%s listening to %s', server.name, server.url); 
});
  
// Create chat connector for communicating with the Bot Framework Service
var connector = new builder.ChatConnector({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword,
    openIdMetadata: process.env.BotOpenIdMetadata 
});

// Listen for messages from users 
server.post('/api/messages', connector.listen());

/*----------------------------------------------------------------------------------------
* Bot Storage: This is a great spot to register the private state storage for your bot. 
* We provide adapters for Azure Table, CosmosDb, SQL Azure, or you can implement your own!
* For samples and documentation, see: https://github.com/Microsoft/BotBuilder-Azure
* ---------------------------------------------------------------------------------------- */

var tableName = 'botdata';
var azureTableClient = new botbuilder_azure.AzureTableClient(tableName, process.env['AzureWebJobsStorage']);
var tableStorage = new botbuilder_azure.AzureBotStorage({ gzipData: false }, azureTableClient);

// Create your bot with a function to receive messages from the user
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send('You reached the default message handler. You said \'%s\'.', session.message.text);
});

bot.set('storage', tableStorage);

// Make sure you add code to validate these fields
var luisAppId = process.env.LuisAppId;
var luisAPIKey = process.env.LuisAPIKey;
var luisAPIHostName = process.env.LuisAPIHostName || 'westus.api.cognitive.microsoft.com';

const LuisModelUrl = 'https://' + luisAPIHostName + '/luis/v2.0/apps/' + luisAppId + '?subscription-key=' + luisAPIKey;

// Create a recognizer that gets intents from LUIS, and add it to the bot
var recognizer = new builder.LuisRecognizer(LuisModelUrl);
bot.recognizer(recognizer);

// Add a dialog for each intent that the LUIS app recognizes.
// See https://docs.microsoft.com/en-us/bot-framework/nodejs/bot-builder-nodejs-recognize-intent-luis 
bot.dialog('GreetingDialog',
    (session) => {
        session.send('You reached the Greeting intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Greeting'
})

bot.dialog('HelpDialog',
    (session) => {
        session.send('You reached the Help intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Help'
})

bot.dialog('CancelDialog',
    (session) => {
        session.send('You reached the Cancel intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Cancel'
})

```


> [!TIP] 
> この記事で説明するサンプル コードは、[メモ ボットのサンプル][NotesSample]に関するページにも示されています。



## <a name="edit-the-default-message-handler"></a>既定のメッセージ ハンドラーを編集する
ボットには既定のメッセージ ハンドラーがあります。 これを以下と一致するように編集します。 
```javascript
// Create your bot with a function to receive messages from the user.
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send("Hi... I'm the note bot sample. I can create new notes, read saved notes to you and delete notes.");

   // If the object for storing notes in session.userData doesn't exist yet, initialize it
   if (!session.userData.notes) {
       session.userData.notes = {};
       console.log("initializing userData.notes in default message handler");
   }
});
```

## <a name="handle-the-notecreate-intent"></a>Note.Create 意図を処理する

次のコードをコピーして、app.js の末尾に貼り付けます。

```javascript
// CreateNote dialog
bot.dialog('CreateNote', [
    function (session, args, next) {
        // Resolve and store any Note.Title entity passed from LUIS.
        var intent = args.intent;
        var title = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');

        var note = session.dialogData.note = {
          title: title ? title.entity : null,
        };
        
        // Prompt for title
        if (!note.title) {
            builder.Prompts.text(session, 'What would you like to call your note?');
        } else {
            next();
        }
    },
    function (session, results, next) {
        var note = session.dialogData.note;
        if (results.response) {
            note.title = results.response;
        }

        // Prompt for the text of the note
        if (!note.text) {
            builder.Prompts.text(session, 'What would you like to say in your note?');
        } else {
            next();
        }
    },
    function (session, results) {
        var note = session.dialogData.note;
        if (results.response) {
            note.text = results.response;
        }
        
        // If the object for storing notes in session.userData doesn't exist yet, initialize it
        if (!session.userData.notes) {
            session.userData.notes = {};
            console.log("initializing session.userData.notes in CreateNote dialog");
        }
        // Save notes in the notes object
        session.userData.notes[note.title] = note;

        // Send confirmation to user
        session.endDialog('Creating note named "%s" with text "%s"',
            note.title, note.text);
    }
]).triggerAction({ 
    matches: 'Note.Create',
    confirmPrompt: "This will cancel the creation of the note you started. Are you sure?" 
}).cancelAction('cancelCreateNote', "Note canceled.", {
    matches: /^(cancel|nevermind)/i,
    confirmPrompt: "Are you sure?"
});
```

発話内のすべてのエンティティは、`args` パラメーターを使用してダイアログに渡されます。 [ウォーターフォール][waterfall]の最初の手順では [EntityRecognizer.findEntity][EntityRecognizer_findEntity] を呼び出し、LUIS 応答の `Note.Title` エンティティからメモのタイトルを取得します。 LUIS アプリで `Note.Title` エンティティが検出されなかった場合は、ボットによってユーザーにメモの名前が求められます。 ウォーターフォールの 2 番目の手順では、メモに含めるテキストが求められます。 ボットでメモのテキストが用意できたら、3 番目の手順では [session.userData][session_userData] を使用し、キーとしてタイトルを使って、`notes` オブジェクトにメモを保存します。 `session.UserData` の詳細については、「[状態データを管理する](./bot-builder-nodejs-state.md)」を参照してください。 



## <a name="handle-the-notedelete-intent"></a>Note.Delete 意図を処理する
`Note.Create` 意図の場合と同様に、ボットでタイトルについて `args` パラメーターが調べられます。 タイトルが検出されない場合、ボットによってユーザーに入力が求められます。 タイトルは `session.userData.notes` から削除するメモを検索するために使用されます。 



次のコードをコピーして、app.js の末尾に貼り付けます。
```javascript
// Delete note dialog
bot.dialog('DeleteNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify that the title is in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to delete?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to delete.");
        }
    },
    function (session, results) {
        delete session.userData.notes[results.response.entity];        
        session.endDialog("Deleted the '%s' note.", results.response.entity);
    }
]).triggerAction({
    matches: 'Note.Delete'
}).cancelAction('cancelDeleteNote', "Ok - canceled note deletion.", {
    matches: /^(cancel|nevermind)/i
});
```

`Note.Delete` を処理するコードでは `noteCount` 関数を使用して、`notes` オブジェクトにメモが含まれているかどうかが判断されます。 

`noteCount` ヘルパー関数を `app.js` の末尾に貼り付けます。

[!code-js[Add a helper function that returns the number of notes (JavaScript)](../includes/code/node-basicNote.js#CountNotesHelper)]

## <a name="handle-the-notereadaloud-intent"></a>Note.ReadAloud 意図を処理する

次のコードをコピーして、`app.js` 内の `Note.Delete` のハンドラーの後ろに貼り付けます。

```javascript
// Read note dialog
bot.dialog('ReadNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
           
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify it's in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to read?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to read.");
        }
    },
    function (session, results) {        
        session.endDialog("Here's the '%s' note: '%s'.", results.response.entity, session.userData.notes[results.response.entity].text);
    }
]).triggerAction({
    matches: 'Note.ReadAloud'
}).cancelAction('cancelReadNote', "Ok.", {
    matches: /^(cancel|nevermind)/i
});

```

`session.userData.notes` オブジェクトは 3 番目の引数として `builder.Prompts.choice` に貼り付けられるため、プロンプトではユーザーにメモの一覧が表示されます。

これで新しい意図のためのハンドラーが追加されました。`app.js` の完全なコードには以下が含まれます。

```javascript
var restify = require('restify');
var builder = require('botbuilder');
var botbuilder_azure = require("botbuilder-azure");

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log('%s listening to %s', server.name, server.url); 
});
  
// Create chat connector for communicating with the Bot Framework Service
var connector = new builder.ChatConnector({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword,
    openIdMetadata: process.env.BotOpenIdMetadata 
});

// Listen for messages from users 
server.post('/api/messages', connector.listen());

/*----------------------------------------------------------------------------------------
* Bot Storage: This is a great spot to register the private state storage for your bot. 
* We provide adapters for Azure Table, CosmosDb, SQL Azure, or you can implement your own!
* For samples and documentation, see: https://github.com/Microsoft/BotBuilder-Azure
* ---------------------------------------------------------------------------------------- */

var tableName = 'botdata';
var azureTableClient = new botbuilder_azure.AzureTableClient(tableName, process.env['AzureWebJobsStorage']);
var tableStorage = new botbuilder_azure.AzureBotStorage({ gzipData: false }, azureTableClient);

// Create your bot with a function to receive messages from the user.
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send("Hi... I'm the note bot sample. I can create new notes, read saved notes to you and delete notes.");

   // If the object for storing notes in session.userData doesn't exist yet, initialize it
   if (!session.userData.notes) {
       session.userData.notes = {};
       console.log("initializing userData.notes in default message handler");
   }
});

bot.set('storage', tableStorage);

// Make sure you add code to validate these fields
var luisAppId = process.env.LuisAppId;
var luisAPIKey = process.env.LuisAPIKey;
var luisAPIHostName = process.env.LuisAPIHostName || 'westus.api.cognitive.microsoft.com';

const LuisModelUrl = 'https://' + luisAPIHostName + '/luis/v2.0/apps/' + luisAppId + '?subscription-key=' + luisAPIKey;

// Create a recognizer that gets intents from LUIS, and add it to the bot
var recognizer = new builder.LuisRecognizer(LuisModelUrl);
bot.recognizer(recognizer);

// CreateNote dialog
bot.dialog('CreateNote', [
    function (session, args, next) {
        // Resolve and store any Note.Title entity passed from LUIS.
        var intent = args.intent;
        var title = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');

        var note = session.dialogData.note = {
          title: title ? title.entity : null,
        };
        
        // Prompt for title
        if (!note.title) {
            builder.Prompts.text(session, 'What would you like to call your note?');
        } else {
            next();
        }
    },
    function (session, results, next) {
        var note = session.dialogData.note;
        if (results.response) {
            note.title = results.response;
        }

        // Prompt for the text of the note
        if (!note.text) {
            builder.Prompts.text(session, 'What would you like to say in your note?');
        } else {
            next();
        }
    },
    function (session, results) {
        var note = session.dialogData.note;
        if (results.response) {
            note.text = results.response;
        }
        
        // If the object for storing notes in session.userData doesn't exist yet, initialize it
        if (!session.userData.notes) {
            session.userData.notes = {};
            console.log("initializing session.userData.notes in CreateNote dialog");
        }
        // Save notes in the notes object
        session.userData.notes[note.title] = note;

        // Send confirmation to user
        session.endDialog('Creating note named "%s" with text "%s"',
            note.title, note.text);
    }
]).triggerAction({ 
    matches: 'Note.Create',
    confirmPrompt: "This will cancel the creation of the note you started. Are you sure?" 
}).cancelAction('cancelCreateNote', "Note canceled.", {
    matches: /^(cancel|nevermind)/i,
    confirmPrompt: "Are you sure?"
});

// Delete note dialog
bot.dialog('DeleteNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify that the title is in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to delete?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to delete.");
        }
    },
    function (session, results) {
        delete session.userData.notes[results.response.entity];        
        session.endDialog("Deleted the '%s' note.", results.response.entity);
    }
]).triggerAction({
    matches: 'Note.Delete'
}).cancelAction('cancelDeleteNote', "Ok - canceled note deletion.", {
    matches: /^(cancel|nevermind)/i
});


// Read note dialog
bot.dialog('ReadNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
           
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify it's in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to read?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to read.");
        }
    },
    function (session, results) {        
        session.endDialog("Here's the '%s' note: '%s'.", results.response.entity, session.userData.notes[results.response.entity].text);
    }
]).triggerAction({
    matches: 'Note.ReadAloud'
}).cancelAction('cancelReadNote', "Ok.", {
    matches: /^(cancel|nevermind)/i
});


// Helper function to count the number of notes stored in session.userData.notes
function noteCount(notes) {

    var i = 0;
    for (var name in notes) {
        i++;
    }
    return i;
}
```

## <a name="test-the-bot"></a>ボットのテスト

Azure Portal で、**[Test in Web Chat]\(Web チャットでのテスト\)** をクリックしてボットをテストします。 "メモを作成する"、"自分のメモを読み取る"、"メモを削除する" などのメッセージを入力して、追加した意図を呼び出してみます。
   ![Web チャットでメモ ボットをテストする](../media/bot-builder-nodejs-use-luis/bot-service-test-notebot.png)

> [!TIP]
> 意図またはエンティティがボットによって必ずしも正しく認識されない場合は、発話の例をさらに追加して、LUIS アプリをトレーニングすることで、そのパフォーマンスを向上させます。 お使いのボットのコードを変更せずに、ご自身の LUIS アプリを再トレーニングできます。 [発話の例の追加](/azure/cognitive-services/LUIS/add-example-utterances)に関するページ、および[ご自身の LUIS アプリのトレーニングとテスト](/azure/cognitive-services/LUIS/train-test)に関するページをご覧ください。


## <a name="next-steps"></a>次の手順

ボットを試してみると、認識エンジンで現在アクティブなダイアログの中断をトリガーできることがわかります。 中断を許可して処理することは、ユーザーが実際に行うことを考慮に入れた柔軟な設計です。 認識された意図に関連付けることができる各種アクションについて詳しく学習します。

> [!div class="nextstepaction"]
> [ユーザー アクションを処理する](bot-builder-nodejs-dialog-actions.md)


[LUIS]: https://www.luis.ai/

[intentDialog]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.intentdialog.html

[intentDialog_matches]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.intentdialog.html#matches 

[NotesSample]: https://github.com/Microsoft/BotFramework-Samples/tree/master/docs-samples/Node/basics-naturalLanguage

[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[confirmPrompt]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#confirmprompt

[waterfall]: bot-builder-nodejs-dialog-manage-conversation-flow.md#manage-conversation-flow-with-a-waterfall

[session_userData]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata

[EntityRecognizer_findEntity]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#findentity

[matches]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#matches

[LUISAzureDocs]: /azure/cognitive-services/LUIS/Home

[Dialog]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html

[IntentRecognizerSetOptions]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html

[LuisRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer

[LUISConcepts]: https://docs.botframework.com/en-us/node/builder/guides/understanding-natural-language/

[DisambiguationSample]: https://github.com/Microsoft/BotBuilder/tree/master/Node/examples/feature-onDisambiguateRoute

[IDisambiguateRouteHandler]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idisambiguateroutehandler.html

[RegExpRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.regexprecognizer.html

[AlarmBot]: https://github.com/Microsoft/BotBuilder/blob/master/Node/examples/basics-naturalLanguage/app.js

[LUISBotSample]: https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-LUIS

[UniversalBot]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html
