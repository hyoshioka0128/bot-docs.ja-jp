---
title: ユーザーへのウェルカム メッセージの送信 | Microsoft Docs
description: ボットでユーザーへの歓迎メッセージを表示する方法について説明します。
keywords: 概要, 開発, ユーザー エクスペリエンス, ようこそ, パーソナライズされたエクスペリエンス, C#, JS, ウェルカム メッセージ, ボット, あいさつ, グリーティング
author: DanDev33
ms.author: v-dashel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/19/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 354dcb1bf1e172609c1729690da76f3297201c0a
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/21/2019
ms.locfileid: "56591143"
---
# <a name="send-welcome-message-to-users"></a>ユーザーへのウェルカム メッセージの送信

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ボットを作成する主な目的は、有意義な会話によってユーザー エンゲージメントを高めることです。 この目標を達成するための効果的な方法の 1 つは、ユーザーが最初にアクセスした瞬間から、ボットの主な目的と機能 (ボットが作られた理由) をユーザーがわかるようにすることです。 この記事では、ボットのユーザーを歓迎するのに役立つコード サンプルを紹介します。

## <a name="prerequisites"></a>前提条件

- [ボットの基本](bot-builder-basics.md)を理解する。 
- **ウェルカム サンプル**のコピー ([C#](https://aka.ms/bot-welcome-sample-cs) または [JS](https://aka.ms/bot-welcome-sample-js))。 サンプルのコードは、ウェルカム メッセージの送信方法を説明するために使用されます。

## <a name="same-welcome-for-different-channels"></a>複数のチャネルに同じウェルカム メッセージを使用する

ウェルカム メッセージは、ユーザーがボットと最初にやり取りしたときに生成する必要があります。 そのためには、ボットのアクティビティ タイプと、新規の接続を監視します。 新規接続が検出されたときには、チャネルによって、最大 2 つの会話更新アクティビティが生成されます。

- 1 つは、ボットが会話に接続したときに生成されます。
- もう 1 つは、ユーザーが会話に参加したときに生成されます。

新しい会話更新が検出されるごとにウェルカム メッセージを生成するという方法をとりたくなるかもしれませんが、その方法だと、ボットがさまざまなチャネルからアクセスされた場合に、予期しない結果を招く恐れがあります。

チャネルによっては、ユーザーがそのチャネルに最初に接続したときに会話更新を 1 回生成し、ユーザーからの最初の入力メッセージが受信された後に、もう 1 つの会話更新を生成する場合もあります。 また、ユーザーがチャネルに最初に接続したときに、これら両方のアクティビティを生成するチャネルもあります。 単に会話更新イベントを監視し、2 つの会話更新アクティビティを使ってチャネルにウェルカム メッセージを表示した場合、ユーザーへのメッセージは次のように表示されるかもしれません。

![重複したウェルカム メッセージ](./media/double_welcome_message.png)

このようにメッセージが重複するのを避けるには、2 番目の会話更新イベントについてのみ、最初のウェルカム メッセージを生成するようにします。 2 番目のイベントは、次の両方が発生したときに検出されるようにします。

- 会話更新イベントが発生した。
- 新しいメンバー (ユーザー) が会話に追加された。

下記のコード サンプルでは、新しい "*会話の更新アクティビティ*" を監視し、会話に参加する新しいユーザーごとにウェルカム メッセージを 1 回だけ送信します。 ボットの最初の _conversationUpdate_ イベントで、ボット自体がチャネルのアクティビティの "_受信者_" として追加されます。 コードでは、追加されたメンバーが _turnContext.Activity.Recipient.Id_ かどうかを確認します。該当する場合、最初の会話更新イベントが検出されているので、新しく接続されたユーザーを探すコードはスキップされます。

[!INCLUDE [alert-await-send-activity](../includes/alert-await-send-activity.md)]

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

C# サンプル コード内では、**Startup.cs** で "WelcomeUserStateAccessors" がサービス/シングルトンとして定義されており、"UserState" がアプリケーションの状態に追加されています。 まず、これらを使用して、会話内の特定のユーザーの状態オブジェクトとそのアクセサーを作成します。

```csharp
/// The state object is used to keep track of various state related to a user in a conversation.
/// In this example, we are tracking if the bot has replied to customer first interaction.
public class WelcomeUserState
{
    public bool DidBotWelcomeUser { get; set; } = false;
}

/// Initializes a new instance of the <see cref="WelcomeUserStateAccessors"/> class.
public class WelcomeUserStateAccessors
{
    public WelcomeUserStateAccessors(UserState userState)
    {
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    public static string WelcomeUserName { get; } = $"{nameof(WelcomeUserStateAccessors)}.WelcomeUserState";

    public IStatePropertyAccessor<WelcomeUserState> WelcomeUserState { get; set; }

    public UserState UserState { get; }
}
```

**WelcomeUserBot** では、アクティビティ更新をチェックして、会話に新しいユーザーが追加されたかどうかを確認し、それらのユーザーにウェルカム メッセージを送信します。

```csharp
// Messages sent to the user.
private const string WelcomeMessage = @"This is a simple Welcome Bot sample.This bot will introduce you
                                        to welcoming and greeting users. You can say 'intro' to see the
                                        introduction card. If you are running this bot in the Bot Framework
                                        Emulator, press the 'Start Over' button to simulate user joining
                                        a bot or a channel";

private const string InfoMessage = @"You are seeing this message because the bot received at least one
                                    'ConversationUpdate' event, indicating you (and possibly others)
                                    joined the conversation. If you are using the emulator, pressing
                                    the 'Start Over' button to trigger this event again. The specifics
                                    of the 'ConversationUpdate' event depends on the channel. You can
                                    read more information at:
                                        https://aka.ms/about-botframework-welcome-user";

private const string PatternMessage = @"It is a good pattern to use this event to send general greeting
                                        to user, explaining what your bot can do. In this example, the bot
                                        handles 'hello', 'hi', 'help' and 'intro. Try it now, type 'hi'";

// The bot state accessor object. Use this to access specific state properties.
private readonly WelcomeUserStateAccessors _welcomeUserStateAccessors;

// Initializes a new instance of the <see cref="WelcomeUserBot"/> class.
public WelcomeUserBot(WelcomeUserStateAccessors statePropertyAccessor)
{
    _welcomeUserStateAccessors = statePropertyAccessor ?? throw new System.ArgumentNullException("state accessor can't be null");
}

// Every conversation turn for our WelcomeUser Bot will call this method, including
// any type of activities such as ConversationUpdate or ContactRelationUpdate which
// are sent when a user joins a conversation.
// This bot doesn't use any dialogs; it's "single turn" processing, meaning a single
// request and response.
// This bot uses UserState to keep track of first message a user sends.
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = new CancellationToken())
{
    // use state accessor to extract the didBotWelcomeUser flag
    var didBotWelcomeUser = await _welcomeUserStateAccessors.WelcomeUserState.GetAsync(turnContext, () => new WelcomeUserState());

    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Your bot should proactively send a welcome message to a personal chat the first time
        // (and only the first time) a user initiates a personal chat with your bot.
        if (didBotWelcomeUser.DidBotWelcomeUser == false)
        {
            didBotWelcomeUser.DidBotWelcomeUser = true;
            // Update user state flag to reflect bot handled first user interaction.
            await _welcomeUserStateAccessors.WelcomeUserState.SetAsync(turnContext, didBotWelcomeUser);
            await _welcomeUserStateAccessors.UserState.SaveChangesAsync(turnContext);

            // the channel should sends the user name in the 'From' object
            var userName = turnContext.Activity.From.Name;

            await turnContext.SendActivityAsync(
                $"You are seeing this message because this was your first message ever to this bot.",
                cancellationToken: cancellationToken);
            await turnContext.SendActivityAsync(
                $"It is a good practice to welcome the user and provide personal greeting. For example, welcome {userName}.",
                cancellationToken: cancellationToken);
        }
    }

    // Greet when users are added to the conversation.
    else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
    {
        if (turnContext.Activity.MembersAdded != null)
        {
            // Iterate over all new members added to the conversation
            foreach (var member in turnContext.Activity.MembersAdded)
            {
                // Greet anyone that was not the target (recipient) of this message
                // the 'bot' is the recipient for events from the channel,
                // turnContext.Activity.MembersAdded == turnContext.Activity.Recipient.Id indicates the
                // bot was added to the conversation.
                if (member.Id != turnContext.Activity.Recipient.Id)
                {
                    await turnContext.SendActivityAsync($"Hi there - {member.Name}. {WelcomeMessage}", cancellationToken: cancellationToken);
                    await turnContext.SendActivityAsync(InfoMessage, cancellationToken: cancellationToken);
                    await turnContext.SendActivityAsync(PatternMessage, cancellationToken: cancellationToken);
                }
            }
        }
    }
    else
    {
        // Default behavior for all other type of activities.
        await turnContext.SendActivityAsync($"{turnContext.Activity.Type} activity detected");
    }

    // save any state changes made to your state objects.
    await _welcomeUserStateAccessors.UserState.SaveChangesAsync(turnContext);
}
```

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

この JavaScript コードは、ユーザーが追加されたときにウェルカム メッセージを送信します。 これは、会話アクティビティをチェックし、会話に新しいメンバーが追加されたことを確認することで実行されます。

```javascript
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License.

// Import required Bot Framework classes.
const { ActivityTypes } = require('botbuilder');
const { CardFactory } = require('botbuilder');

// Adaptive Card content
const IntroCard = require('./resources/IntroCard.json');

// Welcomed User property name
const WELCOMED_USER = 'welcomedUserProperty';

class WelcomeBot {
    /**
     *
     * @param {UserState} User state to persist boolean flag to indicate
     *                    if the bot had already welcomed the user
     */
    constructor(userState) {
        // Creates a new user property accessor.
        // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
        this.welcomedUserProperty = userState.createProperty(WELCOMED_USER);

        this.userState = userState;
    }
    /**
     *
     * @param {TurnContext} context on turn context object.
     */
    async onTurn(turnContext) {
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Read UserState. If the 'DidBotWelcomedUser' does not exist (first time ever for a user)
            // set the default to false.
            const didBotWelcomedUser = await this.welcomedUserProperty.get(turnContext, false);

            // Your bot should proactively send a welcome message to a personal chat the first time
            // (and only the first time) a user initiates a personal chat with your bot.
            if (didBotWelcomedUser === false) {
                // The channel should send the user name in the 'From' object
                let userName = turnContext.activity.from.name;
                await turnContext.sendActivity('You are seeing this message because this was your first message ever sent to this bot.');
                await turnContext.sendActivity(`It is a good practice to welcome the user and provide personal greeting. For example, welcome ${ userName }.`);

                // Set the flag indicating the bot handled the user's first message.
                await this.welcomedUserProperty.set(turnContext, true);
            } else {
                // ...
            }
            // Save state changes
            await this.userState.saveChanges(turnContext);
        } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate) {
            // Send greeting when users are added to the conversation.
            await this.sendWelcomeMessage(turnContext);
        } else {
            // Generic message for all other activities
            await turnContext.sendActivity(`[${ turnContext.activity.type } event detected]`);
        }
    }

    /**
     * Sends welcome messages to conversation members when they join the conversation.
     * Messages are only sent to conversation members who aren't the bot.
     * @param {TurnContext} turnContext
     */
    async sendWelcomeMessage(turnContext) {
        // Do we have any new members added to the conversation?
        if (turnContext.activity.membersAdded.length !== 0) {
            // Iterate over all new members added to the conversation
            for (let idx in turnContext.activity.membersAdded) {
                // Greet anyone that was not the target (recipient) of this message.
                // Since the bot is the recipient for events from the channel,
                // context.activity.membersAdded === context.activity.recipient.Id indicates the
                // bot was added to the conversation, and the opposite indicates this is a user.
                if (turnContext.activity.membersAdded[idx].id !== turnContext.activity.recipient.id) {
                    await turnContext.sendActivity(`Welcome to the 'Welcome User' Bot. This bot will introduce you to welcoming and greeting users.`);
                    await turnContext.sendActivity("You are seeing this message because the bot received at least one 'ConversationUpdate'" +
                                            'event,indicating you (and possibly others) joined the conversation. If you are using the emulator, ' +
                                            "pressing the 'Start Over' button to trigger this event again. The specifics of the 'ConversationUpdate' " +
                                            'event depends on the channel. You can read more information at https://aka.ms/about-botframework-welcome-user');
                    await turnContext.sendActivity(`It is a good pattern to use this event to send general greeting to user, explaining what your bot can do. ` +
                                            `In this example, the bot handles 'hello', 'hi', 'help' and 'intro. ` +
                                            `Try it now, type 'hi'`);
                }
            }
        }
    }
}

module.exports.WelcomeBot = WelcomeBot;
```

---

## <a name="discard-initial-user-input"></a>最初のユーザー入力を無視する

ユーザーの入力に有用な情報が実際に含まれている場合を考慮することも重要です。これもチャネルによって状況が変わってきます。 ユーザーが使用可能なすべてのチャネルで良好なエクスペリエンスを得られるようにするために、状態フラグ _didBotWelcomeUser_ を確認し、これが "false" の場合は、この最初のユーザー入力を処理しないようにします。 代わりに、ユーザーに応答用の初期プロンプトを表示します。 その後、変数 _didBotWelcomeUser_ が "true" に設定され、コードで追加のすべてのメッセージ アクティビティからのユーザー入力を処理します。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = new CancellationToken())
{
    // use state accessor to extract the didBotWelcomeUser flag
    var didBotWelcomeUser = await _welcomeUserStateAccessors.WelcomeUserState.GetAsync(turnContext, () => new WelcomeUserState());

    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        if (didBotWelcomeUser.DidBotWelcomeUser == false)
        {
            // See previous code sample.
        }
        else
        {
            // This example hard-codes specific utterances. You should use LUIS or QnA for more advanced language understanding.
            var text = turnContext.Activity.Text.ToLowerInvariant();
            switch (text)
            {
                case "hello":
                case "hi":
                    await turnContext.SendActivityAsync($"You said {text}.", cancellationToken: cancellationToken);
                    break;
                case "intro":
                case "help":
                default:
                    await turnContext.SendActivityAsync(WelcomeMessage, cancellationToken: cancellationToken);
                    break;
            }
        }
    }

    // Greet when users are added to the conversation.
    // Note that all channels do not send the conversation update activity.
    // If you find that this bot works in the emulator, but does not in
    // another channel the reason is most likely that the channel does not
    // send this activity.
    else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
    {
        // See previous code sample.
    }

    // save any state changes made to your state objects.
    await _welcomeUserStateAccessors.UserState.SaveChangesAsync(turnContext);
}
```

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
class MainDialog
{
    // Previous Code Sample
    async onTurn(turnContext)
    {
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Previous Code Sample
            if (didBotWelcomeUser === false) {
                // Previous Code Sample
            } else  {
                // This example uses an exact match on user's input utterance.
                // Consider using LUIS or QnA for Natural Language Processing.
                let text = turnContext.activity.text.toLowerCase();
                switch (text) {
                case 'hello':
                case 'hi':
                    await turnContext.sendActivity(`You said "${ turnContext.activity.text }"`);
                    break;
                case 'intro':
                case 'help':
                    await turnContext.sendActivity({
                        text: 'Intro Adaptive Card',
                        attachments: [CardFactory.adaptiveCard(IntroCard)]
                    });
                    break;
                default :
                    await turnContext.sendActivity(`This is a simple Welcome Bot sample. You can say 'intro' to
                                                        see the introduction card. If you are running this bot in the Bot
                                                        Framework Emulator, press the 'Start Over' button to simulate user joining a bot or a channel`);
                }
            }
        }
       // Previous Sample Code
    }
}
```

---

## <a name="using-adaptive-card-greeting"></a>アダプティブ カード グリーティングの使用

ユーザーにあいさつをするもう 1 つの方法として、アダプティブ カード グリーティングがあります。 アダプティブ カード グリーティングについて詳しくは、「[アダプティブ カードを送信する](./bot-builder-howto-add-media-attachments.md)」をご覧ください。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Sends an adaptive card greeting.
private static async Task SendIntroCardAsync(ITurnContext turnContext, CancellationToken cancellationToken)
{
    var response = turnContext.Activity.CreateReply();

    var card = new HeroCard();
    card.Title = "Welcome to Bot Framework!";
    card.Text = @"Welcome to Welcome Users bot sample! This Introduction card
                    is a great way to introduce your Bot to the user and suggest
                    some things to get them started. We use this opportunity to
                    recommend a few next steps for learning more creating and deploying bots.";
    card.Images = new List<CardImage>() { new CardImage("https://aka.ms/bf-welcome-card-image") };
    card.Buttons = new List<CardAction>()
    {
        new CardAction(ActionTypes.OpenUrl,
            "Get an overview", null, "Get an overview", "Get an overview",
            "https://docs.microsoft.com/en-us/azure/bot-service/?view=azure-bot-service-4.0"),
        new CardAction(ActionTypes.OpenUrl,
            "Ask a question", null, "Ask a question", "Ask a question",
            "https://stackoverflow.com/questions/tagged/botframework"),
        new CardAction(ActionTypes.OpenUrl,
            "Learn how to deploy", null, "Learn how to deploy", "Learn how to deploy",
            "https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-deploy-azure?view=azure-bot-service-4.0"),
    };

    response.Attachments = new List<Attachment>() { card.ToAttachment() };
    await turnContext.SendActivityAsync(response, cancellationToken);
}
```

次に、下記の await コマンドを使用してカードを送信します。 このコマンドを、ボットの _switch (text) case "help"_ 内に配置します。

```csharp
switch (text)
{
    case "hello":
    case "hi":
        await turnContext.SendActivityAsync($"You said {text}.", cancellationToken: cancellationToken);
        break;
    case "intro":
    case "help":
        await SendIntroCardAsync(turnContext, cancellationToken);
        break;
    default:
        await turnContext.SendActivityAsync(WelcomeMessage, cancellationToken: cancellationToken);
        break;
}
```

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

まず、ボットの Imports の下にある _index.js_ の先頭に、アダプティブ カードを追加します。

```javascript
// Adaptive Card content
const IntroCard = require('./Resources/IntroCard.json');
```

次に、ボットの _switch (text)_ _case "help"_ セクションで下記のコードを使用して、ユーザー プロンプトにアダプティブ カードで応答します。

```javascript
switch (text)
{
    case "hello":
    case "hi":
        await turnContext.sendActivity(`You said "${turnContext.activity.text}"`);
        break;
    case "intro":
    case "help":
        await turnContext.sendActivity({
            text: 'Intro Adaptive Card',
            attachments: [CardFactory.adaptiveCard(IntroCard)]
        });
        break;
    default :
        await turnContext.sendActivity(`This is a simple Welcome Bot sample. You can say 'intro' to 
                                        see the introduction card. If you are running this bot in the Bot 
                                        Framework Emulator, press the 'Start Over' button to simulate user joining a bot or a channel`);
}
```

---

## <a name="test-the-bot"></a>ボットのテスト

ボットを実行およびテストする手順については、[README](https://aka.ms/bot-welcome-sample-cs) ファイルを参照してください。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [ユーザー入力の収集](bot-builder-prompts.md)
