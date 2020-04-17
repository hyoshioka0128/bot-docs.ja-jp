---
title: JavaScript v3 から v4 への移行のクイック リファレンス - Bot Service
description: JavaScript Bot Framework SDK の v3 と v4 の主な相違点を概説します。
keywords: JavaScript, ボットの移行, ダイアログ, v3 ボット
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ea71cfa7adb9ba4f38b3f0ec79641cf28b0f64f3
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75791074"
---
# <a name="javascript-migration-quick-reference"></a>JavaScript 移行クイック リファレンス

BotBuilder JavaScript SDK v4 では、ボットの作成方法に影響する基本的な変更がいくつか行われています。 このガイドでは、v3 と v4 の各 SDK で作業を行う場合の一般的な違いについてのクイック リファレンスを提供することを目的としています。

- ボットとチャネルの間で情報を渡す方法が変更されました。
    v3 では、_connector_ オブジェクトと _session_ オブジェクトを使用していました。
    v4 では、これらが、_adapter_ オブジェクトと _turn context_ オブジェクトに代わります。

- また、ダイアログとボット インスタンスがさらに分離されました。
    v3 では、ダイアログはボットのコンストラクターに直接登録されました。
    v4 では、ダイアログを引数としてボット インスタンスに渡すようになったため、構成における柔軟性が向上しています。

- さらに、v4 では、`ActivityHandler` クラスを提供しています。これを利用して、_メッセージ_ アクティビティ、_会話の更新_アクティビティ、_イベント_ アクティビティなど、さまざまな種類のアクティビティの処理を自動化することができます。

これらの機能強化の結果、特に、ボット オブジェクトの作成、ダイアログの定義、イベント処理ロジックのコーディング関連で、JavaScript でボットを開発するための構文が変更されています。

このトピックの残りの部分では、JavaScript Bot Framework SDK v3 の構成要素と、それに対応する v4 の構成要素を比較します。

## <a name="to-listen-for-incoming-requests"></a>受信要求をリッスンするには

### <a name="v3"></a>v3

```javascript
var connector = new builder.ChatConnector({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

server.post('/api/messages', connector.listen());
```

### <a name="v4"></a>v4

```javascript
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        await bot.run(context);
    });
});
```

## <a name="to-create-a-bot-instance"></a>ボット インスタンスを作成するには

### <a name="v3"></a>v3

```javascript
var bot = new builder.UniversalBot(connector, [ ...DIALOG_STEPS ]);
```

### <a name="v4"></a>v4

```javascript
// Define the bot class as extending ActivityHandler.
const { ActivityHandler } = require('botbuilder');

class MyBot extends ActivityHandler {
    // ...
}

// Instantiate a new bot instance.
const bot = new MyBot(conversationState, dialog);
```

## <a name="to-send-a-message-to-a-user"></a>ユーザーにメッセージを送信するには

### <a name="v3"></a>v3

```javascript
session.send('Hello and welcome to the help desk bot.');
```

### <a name="v4"></a>v4

```javascript
await context.sendActivity('Hello and welcome to the help desk bot.');
```

## <a name="to-define-a-default-dialog"></a>既定のダイアログを定義するには

### <a name="v3"></a>v3

```javascript
var bot = new builder.UniversalBot(connector, [
    // Add default dialog waterfall steps...
]);
```

### <a name="v4"></a>v4

```javascript
// In the main dialog class, define a run method.
async run(turnContext, accessor) {
    const dialogSet = new DialogSet(accessor);
    dialogSet.add(this);

    const dialogContext = await dialogSet.createContext(turnContext);
    const results = await dialogContext.continueDialog();
    if (results.status === DialogTurnStatus.empty) {
        await dialogContext.beginDialog(this.id);
    }
}

// Pass conversation state management and a main dialog objects to the bot (in index.js).
const bot = new MyBot(conversationState, dialog);

// Inside the bot's constructor, add the dialog as a member property and define a DialogState property accessor.
this.dialog = dialog;
this.dialogState = this.conversationState.createProperty('DialogState');

// Inside the bot's message handler, invoke the dialog's run method, passing in the turn context and DialogState accessor.
this.onMessage(async (context, next) => {
    // Run the Dialog with the new message Activity.
    await this.dialog.run(context, this.dialogState);
    await next();
});
```

## <a name="to-start-a-child-dialog"></a>子ダイアログを開始するには

親ダイアログは、子ダイアログの終了後に再開されます。

### <a name="v3"></a>v3

```javascript
session.beginDialog(DIALOG_ID);
```

### <a name="v4"></a>v4

```javascript
await context.beginDialog(DIALOG_ID);
```

## <a name="to-replace-a-dialog"></a>ダイアログを置換するには

現在のダイアログは、スタック上で新しいダイアログに置換されます。

### <a name="v3"></a>v3

```javascript
session.replaceDialog(DIALOG_ID);
```

### <a name="v4"></a>v4

```javascript
await context.replaceDialog(DIALOG_ID);
```

## <a name="to-end-a-dialog"></a>ダイアログを終了するには

### <a name="v3"></a>v3

```javascript
session.endDialog();
```

### <a name="v4"></a>v4

オプションの戻り値を含めることができます。

```javascript
await context.endDialog(returnValue);
```

## <a name="to-register-a-dialog-with-a-bot-instance"></a>ダイアログをボット インスタンスに登録するには

### <a name="v3"></a>v3

```javascript
// Create the bot.
var bot = new builder.UniversalBot(connector, [
    // ...
]};

// Add the dialog to the bot.
bot.dialog('myDialog', require('./mydialog'));
```

### <a name="v4"></a>v4

```javascript
// In the main dialog class constructor.
this.addDialog(new MyDialog(DIALOG_ID));

// In the app entry point file (index.js)
const { MainDialog } = require('./dialogs/main');

// Create the base dialog and bot, passing the main dialog as an argument.
const dialog = new MainDialog();
const reservationBot = new ReservationBot(conversationState, dialog);
```

## <a name="to-prompt-a-user-for-input"></a>ユーザーに入力を要求するには

### <a name="v3"></a>v3

```javascript
var builder = require('botbuilder');
builder.Prompts.text(session, 'Please enter your destination');
```

### <a name="v4"></a>v4

```javascript
const { TextPrompt } = require('botbuilder-dialogs');

// In the dialog's constructor, register the prompt.
this.addDialog(new TextPrompt('initialPrompt'));

// In the dialog step, invoke the prompt.
return await stepContext.prompt(
    'initialPrompt', {
        prompt: 'Please enter your destination'
    }
);
```

## <a name="to-retrieve-the-users-response-to-a-prompt"></a>プロンプトに対するユーザーの応答を取得するには

### <a name="v3"></a>v3

```javascript
function (session, result) {
    var destination = result.response;
    // ...
}
```

### <a name="v4"></a>v4

```javascript
// In the dialog step after the prompt step, retrieve the result from the waterfall step context.
async nextStep(stepContext) {
    const destination = stepContext.result;
    // ...
}
```

## <a name="to-save-information-to-dialog-state"></a>ダイアログの状態の情報を保存するには

### <a name="v3"></a>v3

```javascript
session.dialogData.destination = results.response;
```

### <a name="v4"></a>v4

```javascript
// In a dialog, set the value in the waterfall step context.
stepContext.values.destination = destination;
```

## <a name="to-write-changes-in-state-to-the-persistance-layer"></a>状態の変更を永続化レイヤーに書き込むには

### <a name="v3"></a>v3

```javascript
// State data is saved by default at the end of the turn.
// Do this to save it manually.
session.save();
```

### <a name="v4"></a>v4

```javascript
// State changes are not saved automatically at the end of the turn.
await this.conversationState.saveChanges(context, false);
await this.userState.saveChanges(context, false);
```

## <a name="to-create-and-register-state-storage"></a>状態ストレージを作成して登録するには

### <a name="v3"></a>v3

```javascript
var builder = require('botbuilder');

// Create conversation state with in-memory storage provider.
var inMemoryStorage = new builder.MemoryBotStorage();

// Create the base dialog and bot
var bot = new builder.UniversalBot(connector, [ /*...*/ ]).set('storage', inMemoryStorage);
```

### <a name="v4"></a>v4

```javascript
const { MemoryStorage } = require('botbuilder');

// Create the memory layer object.
const memoryStorage = new MemoryStorage();

// Create the conversation state management object, using the storage provider.
const conversationState = new ConversationState(memoryStorage);

// Create the base dialog and bot.
const dialog = new MainDialog();
const reservationBot = new ReservationBot(conversationState, dialog);
```

## <a name="to-catch-an-error-thrown-from-a-dialog"></a>ダイアログからスローされたエラーをキャッチするには

### <a name="v3"></a>v3

```javascript
// Set up the error handler.
session.on('error', function (err) {
    session.send('Failed with message:', err.message);
    session.endDialog();
});

// Throw an error.
session.error('An error has occurred.');
```

### <a name="v4"></a>v4

```javascript
// Set up the error handler, using the adapter.
adapter.onTurnError = async (context, error) => {
    const errorMsg = error.message

    // Clear out conversation state to avoid keeping the conversation in a corrupted bot state.
    await conversationState.delete(context);

    // Send a message to the user.
    await context.sendActivity(errorMsg);
};

// Throw an error.
async () => {
    throw new Error('An error has occurred.');
}
```

## <a name="to-register-activity-handlers"></a>アクティビティ ハンドラーを登録するには

### <a name="v3"></a>v3

```javascript
bot.on('conversationUpdate', function (message) {
    // ...
}

bot.on('contactRelationUpdate', function (message) {
    // ...
}
```

### <a name="v4"></a>v4

```javascript
// In the bot's constructor, add the handlers.
this.onMessage(async (context, next) => {
    // ...
}
  
this.onDialog(async (context, next) => {
    // ...
}

this.onMembersAdded(async (context, next) => {
    // ...
}
```

## <a name="to-send-attachments"></a>添付ファイルを送信するには

### <a name="v3"></a>v3

```javascript
var message = new builder.Message()
    .attachments(hotelHeroCards
    .attachmentLayout(builder.AttachmentLayout.carousel));
```

### <a name="v4"></a>v4

```javascript
await context.sendActivity({
    attachments: hotelHeroCards,
    attachmentLayout: AttachmentLayoutTypes.Carousel
});
```

## <a name="to-use-natural-language-recognition-luis"></a>自然言語認識 (LUIS) を使用するには

### <a name="v3"></a>v3

```javascript
// The LUIS recognizer was part of the 'botbuilder' library
var builder = require('botbuilder');

var recognizer = new builder.LuisRecognizer(LUIS_MODEL_URL);
bot.recognizer(recognizer);
```

### <a name="v4"></a>v4

```javascript
// The LUIS recognizer is now part of the 'botbuilder-ai' library
const { LuisRecognizer } = require('botbuilder-ai');

const luisApp = { applicationId: LuisAppId, endpointKey: LuisAPIKey, endpoint: `https://${ LuisAPIHostName }` };
const recognizer = new LuisRecognizer(luisApp);

const recognizerResult = await recognizer.recognize(context);
const intent = LuisRecognizer.topIntent(recognizerResult);
```

## <a name="v3-intent-dialog-and-v4-equivalent"></a>v3 意図ダイアログと v4 の同等

### <a name="v3"></a>v3

```javascript
// Create a 'greetings' RegExpRecognizer that can be turned off
var greetings = new builder.RegExpRecognizer('Greetings', /hello|hi|hey|greetings/i)
    .onEnabled(function (session, callback) {
        // Check to see if this recognizer should be enabled
        if (session.conversationData.useGreetings) {
            callback(null, true);
        } else {
            callback(null, false);
        }
    });

// Create our IntentDialog and add recognizers
var intents = new builder.IntentDialog({ recognizers: [greetings] });

bot.dialog('/', intents);

// If no intent is recognized, direct user to Recognizer Menu
intents.onDefault('RecognizerMenu');

// Match our "Greetings" and "Farewell" intents with their dialogs
intents.matches('Greetings', 'Greetings');

// Add a greetings dialog
bot.dialog('Greetings', [
    function (session) {
        session.endDialog('Greetings!');
    }
]);
```

### <a name="v4"></a>v4

```javascript
this.onMessage(async (context, next) => {

    const recognizerResult = {
        text: context.activity.text,
        intents: []
    };

    const greetingRegex = RegExp(/hello|hi|hey|greetings/i);

    if (greetingRegex.test(context.activity.text)) {
      // greeting intent identified
      recognizerResult.intents.push('Greeting');
    }

    if (recognizerResult.intents.includes('Greeting')) {
        // Run the 'Greeting' dialog
        await context.beginDialog(GREETING_DIALOG_ID);
    }

    await next();
});
```