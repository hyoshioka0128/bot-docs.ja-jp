---
title: Cortana スキルを使用した音声認識ボットの作成 - Bot Service
description: Cortana スキルと Bot Framework SDK for Node.js を使用して音声認識ボットを作成する方法を説明します。
author: DeniseMak
manager: kamrani
ms.author: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/10/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f3a10198fc43b696017446116e5a1e8aa64fc058
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75790944"
---
# <a name="build-a-speech-enabled-bot-with-cortana-skills"></a>Cortana スキルを使用した音声認識ボットの作成

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-cortana-skill.md)
> - [Node.js](../nodejs/bot-builder-nodejs-cortana-skill.md)

Bot Framework SDK for Node.js で音声認識ボットを作成できます。そのためには、これを Cortana スキルとして Cortana チャネルに接続します。 Cortana スキルを使用すると、ユーザーからの音声入力に応じた機能を Cortana を通して提供できます。

> [!TIP]
> スキルとは何か、何ができるかについては、「[Cortana Skills Kit][CortanaGetStarted]」 (Cortana スキル キット) を参照してください。

Bot Framework を使用して Cortana スキルを作成する場合、Cortana 固有の知識はほとんど必要なく、ボットの作成が主な部分を構成します。 これまでに作成してきたかもしれない他のボットとの主な相違点の 1 つは、Cortana には視覚と音声の両方のコンポーネントがあることです。 視覚コンポーネントとしては、Cortana にはカードなどのコンテンツを表示するためのキャンバス領域があります。 音声コンポーネントとしては、ボットのメッセージにテキストまたは SSML を指定すると、それが Cortana によってユーザーに向けて読み上げられるので、ボットが話せるようになります。 

> [!NOTE]
> Cortana は多種多様なデバイスで利用できます。 画面があるものもあれば、スタンドアロン スピーカーのように画面がないものもあります。 どちらのシナリオにも対応できるようにボットを作成する必要があります。 デバイス情報を確認する方法については、[Cortana 固有のエンティティ][CortanaSpecificEntities]に関するページを参照してください。

## <a name="adding-speech-to-your-bot"></a>ボットに音声を追加する

ボットからの音声メッセージは、音声合成マークアップ言語 (SSML) として表されます。 Bot Framework SDK を使用すると、ボットが表示する内容だけでなく、話す内容も制御できるよう、応答に SSML を含めることができます。

### <a name="sessionsay"></a>session.say

ボットでは、ユーザーに話しかけるために、**session.send** メソッドの代わりに、**session.say** を使用します。 このメソッドには、SSML 出力や、カードなどの添付ファイルを送信するための省略可能なパラメーターが含まれます。 

メソッドの形式は次のとおりです。

```session.say(displayText: string, speechText: string, options?: object)```

| パラメーター | [説明] |
|------|------|
| **displayText** | Cortana の UI に表示されるテキスト メッセージ。|
| **speechText** | Cortana がユーザーに読み上げるテキストまたは SSML。 |
| **options** | 添付ファイルや入力ヒントを含めることができる [IMessage][IMessage] オブジェクト。 入力ヒントは、ボットが入力を受け入れるか、必要とするか、無視するかを示します。 カードの添付ファイルは、**displayText** 情報の下にある Cortana のキャンバスに表示されます。   |

**inputHint** プロパティは、ボットが入力を必要とするかどうかを Cortana に示すのに役立ちます。 組み込みプロンプトを使用する場合、この値は自動的に既定値の **expectingInput** に設定されます。


| 値 | [説明] |
|------|------|
| **acceptingInput** | ボットは受動的に入力を受け入れる準備ができていますが、応答を待機しているわけではありません。 ユーザーがマイク ボタンを押したままにすると、Cortana はユーザーからの入力を受け入れます。|
| **expectingInput** | ボットがユーザーからの応答をアクティブに必要としていることを示します。 Cortana はユーザーがマイクに話すのをリッスンします。  |
||注:ヘッドレス デバイス (ディスプレイのないデバイス) で **expectingInput** を使用_しない_でください。 [Cortana Skills Kit の FAQ](https://review.docs.microsoft.com/cortana/skills/faq) をご覧ください。|
| **ignoringInput** | Cortana は入力を無視します。 ボットが要求をアクティブに処理しており、要求が完了するまでユーザーの入力を無視する場合に、このヒントを送信できます。  |

以下の例は、Cortana がプレーンテキストまたは SSML を読み取る方法を示しています。

```javascript

// Have Cortana read plain text
session.say('This is the text that Cortana displays', 'This is the text that is spoken by Cortana.');

// Have Cortana read SSML
session.say('This is the text that Cortana displays', '<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">This is the text that is spoken by Cortana.</speak>');

```

以下の例は、ユーザー入力が必要であることを Cortana に知らせる方法を示しています。 マイクは、起動した状態のままになります。

```javascript

// Add an InputHint to let Cortana know to expect user input
session.say('Hi there', 'Hi, what’s your name?', {
    inputHint: builder.InputHint.expectingInput
});

```
<!-- TODO: tip about time limit and batching -->

### <a name="prompts"></a>プロンプト

**session.say()** メソッドを使用する方法に加えて、**speak** オプションと **retrySpeak** オプションを使用して、組み込みプロンプトにテキストまたは SSML を渡すことができます。  

```javascript

builder.Prompts.text(session, 'text based prompt', {                                    
    speak: 'Cortana reads this out initially',                                               
    retrySpeak: 'This message is repeated by Cortana after waiting a while for user input',  
    inputHint: builder.InputHint.expectingInput                                              
});

```

<!-- TODO: Link to SSML library -->

ユーザーに選択項目の一覧を提示するには、**Prompts.choice** を使用します。 **synonyms** オプションを使用すると、ユーザーが話す内容を認識する点で柔軟性が高まります。 **value** オプションは、**results.response.entity** に入れて返されます。 **action** オプションは、選択内容に応じてボットが表示するラベルを指定します。

**Prompts.choice** は順序で選択項目を指定することをサポートします。 つまり、ユーザーはリスト内の項目を選択するときに「1 番目」、「2 番目」、「3 番目」などと言うことができます。 たとえば、次のプロンプトの場合、ユーザーが Cortana に「2 番目のオプション」と命じると、プロンプトは値 8 を返します。

```javascript

        var choices = [
            { value: '4', action: { title: '4 Sides' }, synonyms: 'four|for|4 sided|4 sides' },
            { value: '8', action: { title: '8 Sides' }, synonyms: 'eight|ate|8 sided|8 sides' },
            { value: '12', action: { title: '12 Sides' }, synonyms: 'twelve|12 sided|12 sides' },
            { value: '20', action: { title: '20 Sides' }, synonyms: 'twenty|20 sided|20 sides' },
        ];
        builder.Prompts.choice(session, 'choose_sides', choices, { 
            speak: speak(session, 'choose_sides_ssml') // use helper function to format SSML
        });

```

前の例の場合、プロンプトの **speak** プロパティの SSML は、ローカライズされたプロンプト ファイルに格納されている文字列を使用して書式設定されます。ファイルの形式は次のとおりです。 

```json

{
    "choose_sides": "__Number of Sides__",
    "choose_sides_ssml": [
        "How many sides on the dice?",
        "Pick your poison.",
        "All the standard sizes are supported."
    ]
}

```

その後、ヘルパー関数が、音声合成マークアップ言語 (SSML) ドキュメントの必須のルート要素を作成します。 

```javascript

module.exports.speak = function (template, params, options) {
    options = options || {};
    var output = '<speak xmlns="http://www.w3.org/2001/10/synthesis" ' +
        'version="' + (options.version || '1.0') + '" ' +
        'xml:lang="' + (options.lang || 'en-US') + '">';
    output += module.exports.vsprintf(template, params);
    output += '</speak>';
    return output;
}
```

> [!TIP]
> [Roller サンプル スキル](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill)には、ボットの SSML ベースの応答を作成するための小規模なユーティリティ モジュール (ssml.js) があります。
> 正しい形式の SSML を簡単に作成できる SSML ライブラリもいくつかあり、[npm](https://www.npmjs.com/search?q=ssml) から入手できます。

## <a name="display-cards-in-cortana"></a>Cortana でカードを表示する

音声応答に加え、Cortana ではカードの添付ファイルも表示できます。 Cortana は、次のリッチ カードをサポートしています。
* [HeroCard](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.herocard.html)
* [ReceiptCard](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.receiptcard.html)
* [ThumbnailCard](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.thumbnailcard.html)

これらのカードが Cortana の内部でどのように表示されるかについては、[カード設計のベスト プラクティス][CardDesign]に関するページを参照してください。 ボットにリッチ カードを追加する方法の例については、「[リッチ カードの送信](bot-builder-nodejs-send-rich-cards.md)」を参照してください。 

次のコードは、Hero カードが含まれるメッセージに **speak** プロパティと **inputHint** プロパティを追加する方法を示しています。

```javascript

bot.dialog('HelpDialog', function (session) {
    var card = new builder.HeroCard(session)
        .title('help_title')
        .buttons([
            builder.CardAction.imBack(session, 'roll some dice', 'Roll Dice'),
            builder.CardAction.imBack(session, 'play yahtzee', 'Play Yahtzee')
        ]);
    var msg = new builder.Message(session)
        .speak(speak(session, 'I\'m roller, the dice rolling bot. You can say \'roll some dice\''))
        .addAttachment(card)
        .inputHint(builder.InputHint.acceptingInput); // Tell Cortana to accept input
    session.send(msg).endDialog();
}).triggerAction({ matches: /help/i });

/** This helper function builds the required root element of a Speech Synthesis Markup Language (SSML) document. */
module.exports.speak = function (template, params, options) {
    options = options || {};
    var output = '<speak xmlns="http://www.w3.org/2001/10/synthesis" ' +
        'version="' + (options.version || '1.0') + '" ' +
        'xml:lang="' + (options.lang || 'en-US') + '">';
    output += module.exports.vsprintf(template, params);
    output += '</speak>';
    return output;
}

```
## <a name="sample-rollerskill"></a>サンプル:RollerSkill
以下のセクションで取り上げるコードは、サイコロを振る Cortana のサンプル スキルのものです。 [BotBuilder-Samples リポジトリ](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill)から、ボットのコード全体をダウンロードしてください。

Cortana に向かって[呼び出し名][InvocationNameGuidelines]を話しかけて、スキルを呼び出します。 Roller スキルの場合、[ボットを Cortana チャネルに追加し][CortanaChannel]、それを Cortana スキルとして登録した後、Cortana に「Ask Roller」(Roller に頼んで) または「Ask Roller to roll dice」(サイコロを振るよう Roller に頼んで) と話しかけると、これを呼び出せます。

### <a name="explore-the-code"></a>コードを調べる

RollerSkill サンプルはまず、利用可能なオプションをユーザーに示すボタンの付いたカードを開きます。

```javascript

/**
 *   Create your bot with a default message handler that receive messages from the user.
 * - This function is be called anytime the user's utterance isn't
 *   recognized by the other handlers in the bot.
 */
var bot = new builder.UniversalBot(connector, function (session) {
    // Just redirect to our 'HelpDialog'.
    session.replaceDialog('HelpDialog');
});

//...

bot.dialog('HelpDialog', function (session) {
    var card = new builder.HeroCard(session)
        .title('help_title')
        .buttons([
            builder.CardAction.imBack(session, 'roll some dice', 'Roll Dice'),
            builder.CardAction.imBack(session, 'play craps', 'Play Craps')
        ]);
    var msg = new builder.Message(session)
        .speak(speak(session, 'help_ssml'))
        .addAttachment(card)
        .inputHint(builder.InputHint.acceptingInput);
    session.send(msg).endDialog();
}).triggerAction({ matches: /help/i });
```

### <a name="prompt-the-user-for-input"></a>ユーザーに入力を求めるプロンプト

次のダイアログでは、ボットで遊ぶカスタム ゲームを設定します。  ユーザーに対し、何面のサイコロにするか、そしてサイコロをいくつ転がすかを尋ねます。 ゲームの構造体が作成されると、それは別の 'PlayGameDialog' に渡されます。

このダイアログの **triggerAction()** ハンドラーでは、ダイアログを開始するためにユーザーが「I'd like to roll some dice (サイコロをいくつか振って)」などと話せるようになっています。 ユーザーの入力内容を照合するために正規表現が使用されますが、[LUIS インテント](./bot-builder-nodejs-recognize-intent-luis.md)を使用することも同じように簡単です。 


```javascript
bot.dialog('CreateGameDialog', [
    function (session) {
        // Initialize game structure.
        // - dialogData gives us temporary storage of this data in between
        //   turns with the user.
        var game = session.dialogData.game = { 
            type: 'custom', 
            sides: null, 
            count: null,
            turns: 0
        };

        var choices = [
            { value: '4', action: { title: '4 Sides' }, synonyms: 'four|for|4 sided|4 sides' },
            { value: '6', action: { title: '6 Sides' }, synonyms: 'six|sex|6 sided|6 sides' },
            { value: '8', action: { title: '8 Sides' }, synonyms: 'eight|8 sided|8 sides' },
            { value: '10', action: { title: '10 Sides' }, synonyms: 'ten|10 sided|10 sides' },
            { value: '12', action: { title: '12 Sides' }, synonyms: 'twelve|12 sided|12 sides' },
            { value: '20', action: { title: '20 Sides' }, synonyms: 'twenty|20 sided|20 sides' },
        ];
        builder.Prompts.choice(session, 'choose_sides', choices, { 
            speak: speak(session, 'choose_sides_ssml') 
        });
    },
    function (session, results) {
        // Store users input
        // - The response comes back as a find result with index & entity value matched.
        var game = session.dialogData.game;
        game.sides = Number(results.response.entity);

        /**
         * Ask for number of dice.
         */
        var prompt = session.gettext('choose_count', game.sides);
        builder.Prompts.number(session, prompt, {
            speak: speak(session, 'choose_count_ssml'),
            minValue: 1,
            maxValue: 100,
            integerOnly: true
        });
    },
    function (session, results) {
        // Store users input
        // - The response is already a number.
        var game = session.dialogData.game;
        game.count = results.response;

        /**
         * Play the game we just created.
         * 
         * replaceDialog() ends the current dialog and start a new
         * one in its place. We can pass arguments to dialogs so we'll pass the
         * 'PlayGameDialog' the game we created.
         */
        session.replaceDialog('PlayGameDialog', { game: game });
    }
]).triggerAction({ matches: [
    /(roll|role|throw|shoot).*(dice|die|dye|bones)/i,
    /new game/i
 ]});

```

### <a name="render-results"></a>結果のレンダリング

 次のダイアログは、メイン ゲーム ループです。 ボットはゲームの構造体を **session.conversationData** に格納するので、ユーザーが「roll again (もう一度振って)」と話しかけると、そのまま同じセットのサイコロをもう一度振ることができます。

```javascript

bot.dialog('PlayGameDialog', function (session, args) {
    // Get current or new game structure.
    var game = args.game || session.conversationData.game;
    if (game) {
        // Generate rolls
        var total = 0;
        var rolls = [];
        for (var i = 0; i < game.count; i++) {
            var roll = Math.floor(Math.random() * game.sides) + 1;
            if (roll > game.sides) {
                // Accounts for 1 in a million chance random() generated a 1.0
                roll = game.sides;
            }
            total += roll;
            rolls.push(roll);
        }

        // Format roll results
        var results = '';
        var multiLine = rolls.length > 5;
        for (var i = 0; i < rolls.length; i++) {
            if (i > 0) {
                results += ' . ';
            }
            results += rolls[i];
        }

        // Render results using a card
        var card = new builder.HeroCard(session)
            .subtitle(game.count > 1 ? 'card_subtitle_plural' : 'card_subtitle_singular', game)
            .buttons([
                builder.CardAction.imBack(session, 'roll again', 'Roll Again'),
                builder.CardAction.imBack(session, 'new game', 'New Game')
            ]);
        if (multiLine) {
            //card.title('card_title').text('\n\n' + results + '\n\n');
            card.text(results);
        } else {
            card.title(results);
        }
        var msg = new builder.Message(session).addAttachment(card);

        // Determine bots reaction for speech purposes
        var reaction = 'normal';
        var min = game.count;
        var max = game.count * game.sides;
        var score = total/max;
        if (score == 1.0) {
            reaction = 'best';
        } else if (score == 0) {
            reaction = 'worst';
        } else if (score <= 0.3) {
            reaction = 'bad';
        } else if (score >= 0.8) {
            reaction = 'good';
        }

        // Check for special craps rolls
        if (game.type == 'craps') {
            switch (total) {
                case 2:
                case 3:
                case 12:
                    reaction = 'craps_lose';
                    break;
                case 7:
                    reaction = 'craps_seven';
                    break;
                case 11:
                    reaction = 'craps_eleven';
                    break;
                default:
                    reaction = 'craps_retry';
                    break;
            }
        }

        // Build up spoken response
        var spoken = '';
        if (game.turn == 0) {
            spoken += session.gettext('start_' + game.type + '_game_ssml') + ' ';
        } 
        spoken += session.gettext(reaction + '_roll_reaction_ssml');
        msg.speak(ssml.speak(spoken));

        // Increment number of turns and store game to roll again
        game.turn++;
        session.conversationData.game = game;

        /**
         * Send card and bot's reaction to user. 
         */

        msg.inputHint(builder.InputHint.acceptingInput);
        session.send(msg).endDialog();
    } else {
        // User started session with "roll again" so let's just send them to
        // the 'CreateGameDialog'
        session.replaceDialog('CreateGameDialog');
    }
}).triggerAction({ matches: /(roll|role|throw|shoot) again/i });

```

## <a name="next-steps"></a>次のステップ
ボットをローカルで実行する場合、またはクラウドにデプロイする場合、ボットを Cortana から呼び出せます。 Cortana スキルを試すために必要な手順については、「[Cortana スキルのテスト](../bot-service-debug-cortana-skill.md)」を参照してください。


## <a name="additional-resources"></a>その他のリソース
* [Cortana スキル キット][CortanaGetStarted]
* [メッセージへの音声の追加](bot-builder-nodejs-text-to-speech.md)
* [SSML リファレンス][SSMLRef]
* [Cortana の音声設計のベスト プラクティス][VoiceDesign]
* [Cortana のカード設計のベスト プラクティス][CardDesign]
* [Cortana デベロッパー センター][CortanaDevCenter]
* [Cortana のテストとデバッグのベスト プラクティス][Cortana-TestBestPractice]


[CortanaGetStarted]: /cortana/getstarted
[BFPortal]: https://dev.botframework.com/


[SSMLRef]: https://aka.ms/cortana-ssml
[IMessage]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage.html
[Send]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#send
[CortanaDevCenter]: https://developer.microsoft.com/cortana

[CortanaSpecificEntities]: https://aka.ms/cortana-channel-data
[CortanaAuth]: https://aka.ms/add-auth-cortana-skill

[InvocationNameGuidelines]: https://aka.ms/cortana-invocation-guidelines
[VoiceDesign]: https://aka.ms/cortana-design-voice
[CardDesign]: https://aka.ms/cortana-design-card
[Cortana-Debug]: https://aka.ms/cortana-enable-debug
[Cortana-Publish]: https://aka.ms/cortana-publish


[CortanaChannel]: https://aka.ms/bot-cortana-channel
[Cortana-TestBestPractice]: https://aka.ms/cortana-test-best-practice
