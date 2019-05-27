---
title: 既存の v3 JavaScript ボットを新しい v4 プロジェクトに移行する |Microsoft Docs
description: 既存の v3 JavaScript ボットを取得し、新しいプロジェクトを使用して、これを v4 SDK に移行します。
keywords: JavaScript, ボットの移行, formflow, ダイアログ, v3 ボット
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/13/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3692c723bf2ac4d2275176bb0f7c5f8103225808
ms.sourcegitcommit: 178140eb060d71803f1c6357dd5afebd7f44fe1d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/17/2019
ms.locfileid: "65856661"
---
# <a name="migrate-a-sdk-v3-javascript-bot-to-v4"></a>SDK v3 JavaScript のボットを v4 に移行する

この記事では、v3 SDK JavaScript [core-MultiDialogs-v3](https://aka.ms/v3-js-core-multidialog-migration-sample) ボットを新しい v4 JavaScript ボットに移植します。
この変換は次のステージに分けられます。

1. 新しいプロジェクトを作成し、依存関係を追加する。
1. エントリポイントを更新し、定数を定義する。
1. ダイアログを作成し、SDK v4 を使用してこれを再実装する。
1. ダイアログを実行するようにボットのコードを更新する。
1. **store.js** ユーティリティ ファイルを移植する。

このプロセスを完了すると、正常に動作する v4 ボットを手にすることができます。 変換後のボットのコピー [core-MultiDialogs-v4](https://aka.ms/v4-js-core-multidialog-migration-sample) も、サンプル リポジトリで入手できます。

Bot Framework SDK v4 と SDK v3 は、基になる REST API を共有しています。 ただし、SDK v4 は、開発者がより柔軟にボットを制御できるように、以前のバージョンの SDK をリファクタリングしたものです。 この SDK の主な変更点は次のとおりです。

- 状態は、状態管理オブジェクトとプロパティ アクセサーを使用して管理されます。
- ターンの処理方法、つまり、ボットがユーザーのチャネルから受信アクティビティを受け取って応答する方法が変更されました。
- v4 では `session` オブジェクトを使用しません。代わりに、受信アクティビティに関する情報を格納し、応答アクティビティを返送するために使用できる _turn context_ オブジェクトを使用します。
- 新しいダイアログ ライブラリは v3 とは大きく異ります。 コンポーネント ダイアログとウォーターフォール ダイアログを使用して、古いダイアログを新しいダイアログ システムに変換する必要があります。

<!-- TODO
For more information about specific changes, see [differences between the v3 and v4 JavaScript SDK](???.md).
-->

> [!NOTE]
> 移行の一環として一部のコードのクリーンアップも行いましたが、ここでは移行プロセスの中で v3 ロジックに加えた変更のみを紹介します。

## <a name="prerequisites"></a>前提条件

- Node.js
- Visual Studio Code
- [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)

## <a name="about-this-bot"></a>このボットについて

移行対象のボットは、会話フローを管理するために複数のダイアログが使用されていることを示しています。 このボットでは、フライトやホテルの情報を検索できます。

- メインのダイアログでは、どのような種類の情報を探しているのかをユーザーに確認します。
- ホテル ダイアログでは、ユーザーに検索パラメーターの入力を求め、モック検索を実行します。
- フライト ダイアログでは、ボットがキャッチしたエラーが生成され、適切に処理されます。

## <a name="create-and-open-a-new-v4-bot-project"></a>新しい v4 ボット プロジェクトを作成して開く

1. ボット コードの移植先となる v4 プロジェクトが必要です。 ローカルでプロジェクトを作成する場合は、「[Bot Framework SDK for JavaScript を使用したボットの作成](../../javascript/bot-builder-javascript-quickstart.md)」を参照してください。

    > [!TIP]
    > Azure でプロジェクトを作成することもできます。その場合は、「[Azure Bot Service を使用してボットを作成する](../../bot-service-quickstart.md)」を参照してください。
    > ただし、これら 2 つの方法では、若干異なるサポート ファイルが生成されます。 この記事の v4 プロジェクトは、ローカルのプロジェクトとして作成されています。

1. 次に、Visual Studio Code でプロジェクトを開きます。

## <a name="update-the-packagejson-file"></a>package.json ファイルを更新する

1. Visual Studio Code のターミナル ウィンドウに「`npm i botbuilder-dialogs`」と入力して、**botbuilder-dialogs** パッケージに依存関係を追加します。

1. **./package.json** を編集し、必要に応じて、`name`、`version`、`description` などのプロパティを更新します。

## <a name="update-the-v4-app-entry-point"></a>v4 アプリのエントリ ポイントを更新する

v4 テンプレートで、アプリのエントリ ポイントを表す **index.js** ファイルと、ボット固有のロジックを表す **bot.js** ファイルを作成します。 後の手順で、**bot.js** ファイルの名前を **bots/reservationBot.js** に変更し、各ダイアログのクラスを追加します。

ボット アプリのエントリ ポイントである **./index.js** を編集します。 これには、v3 **app.js** ファイル内の、HTTP サーバーを設定する部分が含まれます。

1. `BotFrameworkAdapter` に加え、**botbuilder** パッケージから `MemoryStorage` と `ConversationState` をインポートします。 また、ボットおよびメイン ダイアログ モジュールもインポートします  (これらはこの後すぐに作成しますが、ここではこれらを参照する必要があります)。

    ```javascript
    // Import required bot services.
    // See https://aka.ms/bot-services to learn more about the different parts of a bot.
    const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');

    // This bot's main dialog.
    const { MainDialog } = require('./dialogs/main')
    const { ReservationBot } = require('./bots/reservationBot');
    ```

1. アダプターの `onTurnError` ハンドラーを定義します。

    ```javascript
    // Catch-all for errors.
    adapter.onTurnError = async (context, error) => {
        const errorMsg = error.message ? error.message : `Oops. Something went wrong!`;
        // This check writes out errors to console log .vs. app insights.
        console.error(`\n [onTurnError]: ${ error }`);
        // Clear out state
        await conversationState.delete(context);
        // Send a message to the user
        await context.sendActivity(errorMsg);
    };
    ```

    v4 では、"_ボット アダプター_" を使用して、受信アクティビティをボットにルーティングします。 アダプターを使用すると、ターンが終了する前に、エラーをキャッチして対処することができます。 ここでは、アプリケーション エラーが発生した場合、会話状態をクリアします。これにより、すべてのダイアログがリセットされ、破損した会話状態にボットがとどまらないようにします。

1. ボットを作成するためのテンプレート コードを以下で置き換えます。

    ```javascript
    // Define state store for your bot.
    const memoryStorage = new MemoryStorage();

    // Create conversation state with in-memory storage provider.
    const conversationState = new ConversationState(memoryStorage);

    // Create the base dialog and bot
    const dialog = new MainDialog();
    const reservationBot = new ReservationBot(conversationState, dialog);
    ```

    インメモリ ストレージ層は `MemoryStorage` クラスによって提供されるようになったため、会話状態管理オブジェクトを明示的に作成する必要があります。

    ダイアログ定義コードは `MainDialog` クラスに移動されています。このクラスは、この後すぐに定義します。 また、ボット定義コードも `ReservationBot` クラスに移行します。

1. 最後に、アダプターを使用してアクティビティをボットにルーティングするように、サーバーの要求ハンドラーを更新します。

    ```javascript
    // Listen for incoming requests.
    server.post('/api/messages', (req, res) => {
        adapter.processActivity(req, res, async (context) => {
            // Route incoming activities to the bot.
            await reservationBot.run(context);
        });
    });
    ```

    v4 では、ボットは `ActivityHandler` から派生します。これは、ターンのアクティビティを受け取る `run` メソッドを定義します。

## <a name="add-a-constants-file"></a>定数ファイルを追加する

ボットの識別子を保持する **./const.js** ファイルを作成します。

```javascript
module.exports = {
    MAIN_DIALOG: 'mainDialog',
    INITIAL_PROMPT: 'initialPrompt',
    HOTELS_DIALOG: 'hotelsDialog',
    INITIAL_HOTEL_PROMPT: 'initialHotelPrompt',
    CHECKIN_DATETIME_PROMPT: 'checkinTimePrompt',
    HOW_MANY_NIGHTS_PROMPT: 'howManyNightsPrompt',
    FLIGHTS_DIALOG: 'flightsDialog',
};
```

v4 では、ダイアログ オブジェクトとプロンプト オブジェクトに ID が割り当てられ、ダイアログとプロンプトは ID で呼び出されます。

## <a name="create-new-dialog-files"></a>新しいダイアログ ファイルを作成する

以下のファイルを作成します。

| ファイル名 | 説明 |
|:---|:---|
| **./dialogs/flights.js** | `hotels` ダイアログの移行済みのロジックが含まれます。 |
| **./dialogs/hotels.js** | `flights` ダイアログの移行済みのロジックが含まれます。 |
| **./dialogs/main.js** | ボットの移行済みのロジックが含まれ、_root_ ダイアログの代わりとして機能します。 |

サポート ダイアログは移行していません。 v4 でヘルプ ダイアログを実装する方法の例については、「[ユーザーによる割り込みを処理する](../bot-builder-howto-handle-user-interrupt.md?tabs=javascript)」を参照してください。

### <a name="implement-the-main-dialog"></a>メイン ダイアログを実装する

v3 では、すべてのボットがダイアログ システムを基にして構築されていました。 v4 では、ボット ロジックとダイアログ ロジックが分離されました。 v3 ボットの "_ルート ダイアログ_" の代わりに、`MainDialog` クラスを使用されます。

**./dialogs/main.js** を編集します。

1. ダイアログに必要なクラスと定数をインポートします。

    ```javascript
    const { DialogSet, DialogTurnStatus, ComponentDialog, WaterfallDialog,
        ChoicePrompt } = require('botbuilder-dialogs');
    const { FlightDialog } = require('./flights');
    const { HotelsDialog } = require('./hotels');
    const { MAIN_DIALOG,
        INITIAL_PROMPT,
        HOTELS_DIALOG,
        FLIGHTS_DIALOG
    } = require('../const');
    ```

1. `MainDialog` クラスを定義してエクスポートします。

    ```javascript
    const initialId = 'mainWaterfallDialog';

    class MainDialog extends ComponentDialog {
        constructor() {
            super(MAIN_DIALOG);

            // Create a dialog set for the bot. It requires a DialogState accessor, with which
            // to retrieve the dialog state from the turn context.
            this.addDialog(new ChoicePrompt(INITIAL_PROMPT, this.validateNumberOfAttempts.bind(this)));
            this.addDialog(new FlightDialog(FLIGHTS_DIALOG));

            // Define the steps of the base waterfall dialog and add it to the set.
            this.addDialog(new WaterfallDialog(initialId, [
                this.promptForBaseChoice.bind(this),
                this.respondToBaseChoice.bind(this)
            ]));

            // Define the steps of the hotels waterfall dialog and add it to the set.
            this.addDialog(new HotelsDialog(HOTELS_DIALOG));

            this.initialDialogId = initialId;
        }
    }

    module.exports.MainDialog = MainDialog;
    ```

    これにより、メイン ダイアログが直接参照する、他のダイアログとプロンプトが宣言されます。

    - このダイアログのステップを含むメイン ウォーターフォール ダイアログ。 コンポーネント ダイアログが開始されると、これによって "_初期ダイアログ_" が開始されます。
    - 実行するタスクをユーザーにたずねるために使用する選択プロンプト。 検証コントロールを使用して選択プロンプトを作成しました。
    - 2 つの子ダイアログ (フライトとホテル)。

1. `run` ヘルパー メソッドをクラスに追加します。

    ```javascript
    /**
     * The run method handles the incoming activity (in the form of a TurnContext) and passes it through the dialog system.
     * If no dialog is active, it will start the default dialog.
     * @param {*} turnContext
     * @param {*} accessor
     */
    async run(turnContext, accessor) {
        const dialogSet = new DialogSet(accessor);
        dialogSet.add(this);

        const dialogContext = await dialogSet.createContext(turnContext);
        const results = await dialogContext.continueDialog();
        if (results.status === DialogTurnStatus.empty) {
            await dialogContext.beginDialog(this.id);
        }
    }
    ```

    v4 では、ボットは、まずダイアログ コンテキストを作成し、次に `continueDialog` を呼び出すことによって、ダイアログ システムとやり取りします。 アクティブなダイアログが存在する場合、これに制御が渡されます。それ以外の場合、単にこの呼び出しから制御が戻ります。 結果が `empty` のときはアクティブなダイアログが存在しなかったことを示します。この場合は、もう一度メイン ダイアログから開始します。

    `accessor` パラメーターによって、ダイアログの状態プロパティのアクセサーが渡されます。 "_ダイアログ スタック_" の状態は、このプロパティに格納されます。 v4 における状態とダイアログのしくみの詳細については、それぞれ「[状態の管理](../bot-builder-concept-state.md)」と「[ダイアログ ライブラリ](../bot-builder-concept-dialog.md)」を参照してください。

1. メイン ダイアログのウォーターフォール ステップと、選択プロンプトの検証コントロールをクラスに追加します。

    ```javascript
    async promptForBaseChoice(stepContext) {
        return await stepContext.prompt(
            INITIAL_PROMPT, {
                prompt: 'Are you looking for a flight or a hotel?',
                choices: ['Hotel', 'Flight'],
                retryPrompt: 'Not a valid option'
            }
        );
    }

    async respondToBaseChoice(stepContext) {
        // Retrieve the user input.
        const answer = stepContext.result.value;
        if (!answer) {
            // exhausted attempts and no selection, start over
            await stepContext.context.sendActivity('Not a valid option. We\'ll restart the dialog ' +
                'so you can try again!');
            return await stepContext.endDialog();
        }
        if (answer === 'Hotel') {
            return await stepContext.beginDialog(HOTELS_DIALOG);
        }
        if (answer === 'Flight') {
            return await stepContext.beginDialog(FLIGHTS_DIALOG);
        }
        return await stepContext.endDialog();
    }

    async validateNumberOfAttempts(promptContext) {
        if (promptContext.attemptCount > 3) {
            // cancel everything
            await promptContext.context.sendActivity('Oops! Too many attempts :( But don\'t worry, I\'m ' +
                'handling that exception and you can try again!');
            return await promptContext.context.endDialog();
        }

        if (!promptContext.recognized.succeeded) {
            await promptContext.context.sendActivity(promptContext.options.retryPrompt);
            return false;
        }
        return true;
    }
    ```

    ウォーターフォールの最初のステップでは、選択プロンプトを開始してユーザーに選択するよう求めます。これ自体がダイアログです。 ウォーターフォールの 2 番目のステップでは、選択プロンプトの結果が使用されます。 子ダイアログが開始されるか (選択が行われた場合)、メイン ダイアログが終了されます (ユーザーが選択しなかった場合)。

    ユーザーが有効な選択を行った場合、選択プロンプトはユーザーの選択内容を返すか、もう一度選択するようユーザーに再プロンプトを出します。 検証コントロールは、ユーザーに対して何回続けてプロンプトが出されたかをチェックし、試行が 3 回失敗したらプロンプトの失敗を許可します。この場合、メインのウォーターフォール ダイアログに制御が戻されます。

### <a name="implement-the-flights-dialog"></a>フライト ダイアログを実装する

v3 のボットでは、フライト ダイアログは、ボットが会話のエラーを処理する方法を示すスタブでした。 今回も同様に処理します。

**./dialogs/flights.js** を編集します。

```javascript
const { ComponentDialog, WaterfallDialog } = require('botbuilder-dialogs');

const initialId = 'flightsWaterfallDialog';

class FlightDialog extends ComponentDialog {
    constructor(id) {
        super(id);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = initialId;

        // Define the conversation flow using a waterfall model.
        this.addDialog(new WaterfallDialog(initialId, [
            async () => {
                throw new Error('Flights Dialog is not implemented and is instead ' +
                    'being used to show Bot error handling');
            }
        ]));
    }
}

exports.FlightDialog = FlightDialog;
```

### <a name="implement-the-hotels-dialog"></a>ホテル ダイアログを実装する

ホテル ダイアログでは、同じフロー全体を保持します。つまり、目的地、日付、宿泊日数をたずね、検索条件に一致する選択肢の一覧をユーザーに示します。

**./dialogs/hotels.js** を編集します。

1. ダイアログに必要なクラスと定数をインポートします。

    ```javascript
    const { ComponentDialog, WaterfallDialog, TextPrompt, DateTimePrompt } = require('botbuilder-dialogs');
    const { AttachmentLayoutTypes, CardFactory } = require('botbuilder');
    const store = require('../store');
    const {
        INITIAL_HOTEL_PROMPT,
        CHECKIN_DATETIME_PROMPT,
        HOW_MANY_NIGHTS_PROMPT
    } = require('../const');
    ```

1. `HotelsDialog` クラスを定義してエクスポートします。

    ```javascript
    const initialId = 'hotelsWaterfallDialog';

    class HotelsDialog extends ComponentDialog {
        constructor(id) {
            super(id);

            // ID of the child dialog that should be started anytime the component is started.
            this.initialDialogId = initialId;

            // Register dialogs
            this.addDialog(new TextPrompt(INITIAL_HOTEL_PROMPT));
            this.addDialog(new DateTimePrompt(CHECKIN_DATETIME_PROMPT));
            this.addDialog(new TextPrompt(HOW_MANY_NIGHTS_PROMPT));

            // Define the conversation flow using a waterfall model.
            this.addDialog(new WaterfallDialog(initialId, [
                this.destinationPromptStep.bind(this),
                this.destinationSearchStep.bind(this),
                this.checkinPromptStep.bind(this),
                this.checkinTimeSetStep.bind(this),
                this.stayDurationPromptStep.bind(this),
                this.stayDurationSetStep.bind(this),
                this.hotelSearchStep.bind(this)
            ]));
        }
    }

    exports.HotelsDialog = HotelsDialog;
    ```

1. ダイアログのステップで使用する 2 つのヘルパー関数をクラスに追加します。

    ```javascript
    addDays(startDate, days) {
        const date = new Date(startDate);
        date.setDate(date.getDate() + days);
        return date;
    };

    createHotelHeroCard(hotel) {
        return CardFactory.heroCard(
            hotel.name,
            `${hotel.rating} stars. ${hotel.numberOfReviews} reviews. From ${hotel.priceStarting} per night.`,
            CardFactory.images([hotel.image]),
            CardFactory.actions([
                {
                    type: 'openUrl',
                    title: 'More details',
                    value: `https://www.bing.com/search?q=hotels+in+${encodeURIComponent(hotel.location)}`
                }
            ])
        );
    }
    ```

    `createHotelHeroCard` によって、ホテルに関する情報を含むヒーロー カードが作成されます。

1. ダイアログで使用するウォーターフォール ステップをクラスに追加します。

    ```javascript
    async destinationPromptStep(stepContext) {
        await stepContext.context.sendActivity('Welcome to the Hotels finder!');
        return await stepContext.prompt(
            INITIAL_HOTEL_PROMPT, {
                prompt: 'Please enter your destination'
            }
        );
    }

    async destinationSearchStep(stepContext) {
        const destination = stepContext.result;
        stepContext.values.destination = destination;
        await stepContext.context.sendActivity(`Looking for hotels in ${destination}`);
        return stepContext.next();
    }

    async checkinPromptStep(stepContext) {
        return await stepContext.prompt(
            CHECKIN_DATETIME_PROMPT, {
                prompt: 'When do you want to check in?'
            }
        );
    }

    async checkinTimeSetStep(stepContext) {
        const checkinTime = stepContext.result[0].value;
        stepContext.values.checkinTime = checkinTime;
        return stepContext.next();
    }

    async stayDurationPromptStep(stepContext) {
        return await stepContext.prompt(
            HOW_MANY_NIGHTS_PROMPT, {
                prompt: 'How many nights do you want to stay?'
            }
        );
    }

    async stayDurationSetStep(stepContext) {
        const numberOfNights = stepContext.result;
        stepContext.values.numberOfNights = parseInt(numberOfNights);
        return stepContext.next();
    }

    async hotelSearchStep(stepContext) {
        const destination = stepContext.values.destination;
        const checkIn = new Date(stepContext.values.checkinTime);
        const checkOut = this.addDays(checkIn, stepContext.values.numberOfNights);

        await stepContext.context.sendActivity(`Ok. Searching for Hotels in ${destination} from 
            ${checkIn.toDateString()} to ${checkOut.toDateString()}...`);
        const hotels = await store.searchHotels(destination, checkIn, checkOut);
        await stepContext.context.sendActivity(`I found in total ${hotels.length} hotels for your dates:`);

        const hotelHeroCards = hotels.map(this.createHotelHeroCard);

        await stepContext.context.sendActivity({
            attachments: hotelHeroCards,
            attachmentLayout: AttachmentLayoutTypes.Carousel
        });

        return await stepContext.endDialog();
    }
    ```

    v3 ホテル ダイアログのステップが v4 ホテル ダイアログのウォーターフォール ステップに移行されました。

## <a name="update-the-bot"></a>ボットを更新する

v4 のボットでは、ダイアログ システム外部のアクティビティに対処できます。 `ActivityHandler` クラスでは一般的な種類のアクティビティに対応したハンドラーを定義できるため、コードの管理が容易になります。

**./bot.js** の名前を **./bots/reservationBot.js** に変更し、これを編集します。

1. このファイルには、ボットの基本実装を提供する **ActivityHandler** が既にインポートされています。

    ```javascript
    const { ActivityHandler } = require('botbuilder');
    ```

1. クラスの名前を `ReservationBot` に変更します。

    ```javascript
    class ReservationBot extends ActivityHandler {
        // ...
    }

    module.exports.ReservationBot = ReservationBot;
    ```

1. 受信するオブジェクトが受け入れられるように、コンストラクターのシグネチャを更新します。

    ```javascript
    /**
     *
     * @param {ConversationState} conversationState
     * @param {Dialog} dialog
     * @param {any} logger object for logging events, defaults to console if none is provided
    */
    constructor(conversationState, dialog, logger) {
        super();
        // ...
    }
    ```

1. コンストラクターでは、null パラメーターのチェックを追加し、クラス コンストラクターのプロパティを定義します。

    ```javascript
    if (!conversationState) throw new Error('[DialogBot]: Missing parameter. conversationState is required');
    if (!dialog) throw new Error('[DialogBot]: Missing parameter. dialog is required');
    if (!logger) {
        logger = console;
        logger.log('[DialogBot]: logger not passed in, defaulting to console');
    }

    this.conversationState = conversationState;
    this.dialog = dialog;
    this.logger = logger;
    this.dialogState = this.conversationState.createProperty('DialogState');
    ```

    ここで、ダイアログ スタックの状態を格納するダイアログ状態プロパティ アクセサーを作成します。

1. コンストラクターで、`onMessage` ハンドラーを更新し、`onDialog` ハンドラーを追加します。

    ```javascript
    this.onMessage(async (context, next) => {
        this.logger.log('Running dialog with Message Activity.');

        // Run the Dialog with the new message Activity.
        await this.dialog.run(context, this.dialogState);

        // By calling next() you ensure that the next BotHandler is run.
        await next();
    });

    this.onDialog(async (context, next) => {
        // Save any state changes. The load happened during the execution of the Dialog.
        await this.conversationState.saveChanges(context, false);

        // By calling next() you ensure that the next BotHandler is run.
        await next();
    });
    ```

    `ActivityHandler` によってメッセージ アクティビティが `onMessage` にルーティングされます。 このボットは、ダイアログを介してすべてのユーザー入力を処理します。

    `ActivityHandler` は、ターンが終了して制御がアダプターに戻る前に `onDialog` を呼び出します。 ターンを終了する前に、明示的に状態を保存する必要があります。 そうしないと、状態の変更が保存されないため、ダイアログが正しく動作しません。

1. 最後に、コンストラクターで `onMembersAdded` ハンドラーを更新します。

    ```javascript
    this.onMembersAdded(async (context, next) => {
        const membersAdded = context.activity.membersAdded;
        for (let cnt = 0; cnt < membersAdded.length; ++cnt) {
            if (membersAdded[cnt].id !== context.activity.recipient.id) {
                await context.sendActivity('Hello and welcome to Contoso help desk bot.');
            }
        }
        // By calling next() you ensure that the next BotHandler is run.
        await next();
    });
    ```

    `ActivityHandler` は、ボット以外の参加者が会話に追加されたことを示す会話更新アクティビティを受け取ると、`onMembersAdded` を呼び出します。 ユーザーが会話に加わったときに、あいさつメッセージを送信するように、このメソッドを更新します。

## <a name="create-the-store-file"></a>ストア ファイルを作成する

ホテル ダイアログで使用される **./store.js** ファイルを作成します。 `searchHotels` はモックのホテル検索関数で、v3 ボットと同じです。

```javascript
module.exports = {
    searchHotels: destination => {
        return new Promise(resolve => {

            // Filling the hotels results manually just for demo purposes
            const hotels = [];
            for (let i = 1; i <= 5; i++) {
                hotels.push({
                    name: `${destination} Hotel ${i}`,
                    location: destination,
                    rating: Math.ceil(Math.random() * 5),
                    numberOfReviews: Math.floor(Math.random() * 5000) + 1,
                    priceStarting: Math.floor(Math.random() * 450) + 80,
                    image: `https://placeholdit.imgix.net/~text?txtsize=35&txt=Hotel${i}&w=500&h=260`
                });
            }

            hotels.sort((a, b) => a.priceStarting - b.priceStarting);

            // complete promise with a timer to simulate async response
            setTimeout(() => { resolve(hotels); }, 1000);
        });
    }
};
```

## <a name="test-the-bot-in-the-emulator"></a>エミュレーターでボットをテストする

この時点で、ボットをローカルで実行し、エミュレーターを使用してボットに接続できます。

1. ご自身のマシンを使ってローカルでサンプルを実行します。
    Visual Studio Code でデバッグ セッションを開始した場合、ボットをテストすると、ログ情報はデバッグ コンソールに送信されます。
1. エミュレーターを起動し、ボットに接続します。
1. メッセージを送信して、メイン、フライト、ホテルの各ダイアログをテストします。

## <a name="additional-resources"></a>その他のリソース

v4 の概念トピック

- [ボットのしくみ](../bot-builder-basics.md)
- [状態の管理](../bot-builder-concept-state.md)
- [ダイアログ ライブラリ](../bot-builder-concept-dialog.md)

v4 の使用方法のトピック

- [テキスト メッセージを送受信する](../bot-builder-howto-send-messages.md)
- [ユーザーと会話データを保存する](../bot-builder-howto-v4-state.md)
- [連続して行われる会話フローの実装](../bot-builder-dialog-manage-conversation-flow.md)
