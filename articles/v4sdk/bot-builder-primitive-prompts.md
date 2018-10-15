---
title: プリミティブなプロンプトを使用して会話フローを管理する | Microsoft Docs
description: Bot Builder SDK でプリミティブなプロンプトを使用して会話フローを管理する方法について説明します。
keywords: 会話フロー, プロンプト
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 7/20/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3014e6cd8b18ab44ff343373a034c392e44bca1d
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707368"
---
# <a name="prompt-users-for-input-using-your-own-prompts"></a>独自のプロンプトを使用してユーザーに入力を求める

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

多くの場合、ボットとユーザー間の会話では、ユーザーに情報の入力を求め、ユーザーの応答を解析し、その情報に基づいてアクションを実行する必要があります。 [ダイアログ ライブラリを使用したユーザーへの入力の要求](bot-builder-prompts.md)に関するトピックでは、**ダイアログ** ライブラリを使用してユーザーに入力を求める方法について詳しく説明しています。 特に、**ダイアログ** ライブラリでは、現在の会話と現在の質問の追跡を処理します。 また、テキスト、数値、日時などのさまざまな種類の情報を要求および検証するためのメソッドも用意されています。

状況によっては、組み込みの**ダイアログ**がボットに適したソリューションではない場合があります。**ダイアログ**は単純なボットに余分なオーバーヘッドを追加したり、柔軟性に欠けることがあります。また、ボットで実行する必要があることを実現できない場合もあります。 このような場合は、ライブラリをスキップし、独自のプロンプト ロジックを実装できます。 このトピックでは、独自のプロンプトを使用して会話を管理できるように、基本的な*エコー ボット*を設定する方法について説明します。

## <a name="track-prompt-states"></a>プロンプトの状態を追跡する

プロンプト駆動型の会話では、会話の進行状況と現在行われている質問を追跡する必要があります。 コードでは、これはいくつかのフラグの管理になります。

たとえば、追跡するプロパティをいくつか作成できます。

これらの状態によって、現在進行中のトピックとプロンプトが追跡されます。 クラウドにデプロイしたときに、これらのフラグが想定どおりに機能するように、[会話の状態](bot-builder-howto-v4-state.md)にフラグを保存します。 

# <a name="ctabcsharp"></a>[C# を選択した場合](#tab/csharp)

状態を追跡するために 2 つのクラスを作成します。 **TopicState** では会話のプロンプトの進行状況を追跡し、**UserProfile** ではユーザーが指定する情報を追跡します。 この情報は、この後の手順でボットの[状態](bot-builder-howto-v4-state.md)に保存します。

```csharp
/// <summary>
/// Contains conversation state information about the conversation in progress.
/// </summary>
public class TopicState
{
    /// <summary>
    /// Identifies the current "topic" of conversation.
    /// </summary>
    public string Topic { get; set; }

    /// <summary>
    /// Indicates whether we asked the user a question last turn, and
    /// if so, what it was.
    /// </summary>
    public string Prompt { get; set; }
}
```

```csharp
/// <summary>
/// Contains user state information for the user's profile.
/// </summary>
public class UserProfile
{
    public string UserName { get; set; }

    public int? Age { get; set; }

    public string WorkPlace { get; set; }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript を選択した場合](#tab/javascript)

**app.js**

```javascript
const storage = new MemoryStorage(); // Volatile memory
const conversationState = new ConversationState(storage);
const userState = new UserState(storage);
const dialogState = conversationState.createProperty('dialogState');
const userProfile = userState.createProperty('userProfile');
```

このコードをボットのメイン ロジック内に配置します。

```javascript
// Pull the state of the dialog out of the conversation state manager.
const convo = await dialogState.get(context, {
    prompt: undefined,
    topic: 'profile'
});

// Pull the user profile out of the user state manager.
const userProfile = await userProfile.get(context, {  // Define the user's profile object
        "userName": undefined,
        "age": undefined,
        "workPlace": undefined
    }
);
```

---

## <a name="manage-a-topic-of-conversation"></a>会話の 1 つのトピックを管理する

ユーザーから情報を収集する会話など、連続して行われる会話では、ユーザーがいつ会話に参加し、会話がどこまで進んでいるのかを把握する必要があります。 上記で定義したプロンプト フラグを設定して確認し、状況に応じてアクションを実行することで、ボットのメイン ターン ハンドラーでこれを追跡できます。 次のサンプルは、これらのフラグを使用して、会話を通じてユーザーのプロファイル情報を収集する方法を示しています。

ボット コードを次に示し、次のセクションで詳しく説明します。

# <a name="ctabcsharp"></a>[C# を選択した場合](#tab/csharp)

ASP.NET Core では、最初にボットと依存関係の挿入を設定する必要があります。

**EchoWithCounterBot.cs** ファイルの名前を **PrimitivePromptsBot.cs** に変更し、クラス名も更新します。 このクラスはボットのロジックを保持します。このクラスはすぐに更新されます。

**EchoBotAccessors.cs** ファイルの名前を **BotAccessors.cs** に変更し、クラス名も更新します。 このクラスは、ボットの状態管理オブジェクトと状態プロパティ アクセサーを保持します。 定義を次のように更新します。

```csharp
using Microsoft.Bot.Builder;
using System;

/// <summary>
/// Contains the state and state property accessors for the primitive prompts bot.
/// </summary>
public class BotAccessors
{
    public const string TopicStateName = "PrimitivePrompts.TopicStateAccessor";

    public const string UserProfileName = "PrimitivePrompts.UserProfileAccessor";

    public ConversationState ConversationState { get; }

    public UserState UserState { get; }

    public IStatePropertyAccessor<TopicState> TopicStateAccessor { get; set; }

    public IStatePropertyAccessor<UserProfile> UserProfileAccessor { get; set; }

    public BotAccessors(ConversationState conversationState, UserState userState)
    {
        if (conversationState is null)
        {
            throw new ArgumentNullException(nameof(conversationState));
        }

        if (userState is null)
        {
            throw new ArgumentNullException(nameof(userState));
        }

        this.ConversationState = conversationState;
        this.UserState = userState;
    }
}
```

**Startup.cs** ファイルの `ConfigureServices` メソッドを更新します。`IStorage` オブジェクトを設定した場所から開始します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<PrimitivePromptsBot>(options =>
    {
        // ...

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        var conversationState = new ConversationState(dataStore);
        options.State.Add(conversationState);

        var userState = new UserState(dataStore);
        options.State.Add(userState);
    });

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        var userState = options.State.OfType<UserState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState, userState)
        {
            TopicStateAccessor = conversationState.CreateProperty<TopicState>(BotAccessors.TopicStateName),
            UserProfileAccessor = userState.CreateProperty<UserProfile>(BotAccessors.UserProfileName),
        };

        return accessors;
    });
}
```

最後に、**PrimitivePromptsBot.cs** ファイルでボットのロジックを更新します。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

public class PrimitivePromptsBot : IBot
{
    public const string ProfileTopic = "profile";

    /// <summary>
    /// Describes a field in the user profile.
    /// </summary>
    private class UserFieldInfo
    {
        /// <summary>
        /// The ID to use for this field.
        /// </summary>
        public string Key { get; set; }

        /// <summary>
        /// The prompt to use to ask for a value for this field.
        /// </summary>
        public string Prompt { get; set; }

        /// <summary>
        /// Gets the value of the corresponding field.
        /// </summary>
        public Func<UserProfile, string> GetValue { get; set; }

        /// <summary>
        /// Sets the value of the corresponding field.
        /// </summary>
        public Action<UserProfile, string> SetValue { get; set; }
    }

    /// <summary>
    /// The prompts for the user profile, indexed by field name.
    /// </summary>
    private static List<UserFieldInfo> UserFields { get; } = new List<UserFieldInfo>
    {
        new UserFieldInfo {
            Key = nameof(UserProfile.UserName),
            Prompt = "What is your name?",
            GetValue = (profile) => profile.UserName,
            SetValue = (profile, value) => profile.UserName = value,
        },
        new UserFieldInfo {
            Key = nameof(UserProfile.Age),
            Prompt = "How old are you?",
            GetValue = (profile) => profile.Age.HasValue? profile.Age.Value.ToString() : null,
            SetValue = (profile, value) =>
            {
                if (int.TryParse(value, out int age))
                {
                    profile.Age = age;
                }
            },
        },
        new UserFieldInfo {
            Key = nameof(UserProfile.WorkPlace),
            Prompt = "Where do you work?",
            GetValue = (profile) => profile.WorkPlace,
            SetValue = (profile, value) => profile.WorkPlace = value,
        },
    };

    /// <summary>
    /// The state and state accessors for the bot.
    /// </summary>
    private BotAccessors Accessors { get; }

    public PrimitivePromptsBot(BotAccessors accessors)
    {
        Accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));
    }

    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        if (turnContext.Activity.Type is ActivityTypes.Message)
        {
            // Use the state property accessors to get the topic state and user profile.
            TopicState topicState = await Accessors.TopicStateAccessor.GetAsync(
                turnContext,
                () => new TopicState { Topic = ProfileTopic, Prompt = null },
                cancellationToken);
            UserProfile userProfile = await Accessors.UserProfileAccessor.GetAsync(
                turnContext,
                () => new UserProfile(),
                cancellationToken);

            // Check whether we need more information.
            if (topicState.Topic is ProfileTopic)
            {
                // If we're expecting input, record it in the user's profile.
                if (topicState.Prompt != null)
                {
                    UserFieldInfo field = UserFields.First(f => f.Key.Equals(topicState.Prompt));
                    field.SetValue(userProfile, turnContext.Activity.Text.Trim());
                }

                // Determine which fields are not yet set.
                List<UserFieldInfo> emptyFields = UserFields.Where(f => f.GetValue(userProfile) is null).ToList();

                if (emptyFields.Any())
                {
                    // If all the fields are empty, send a welcome message.
                    if (emptyFields.Count == UserFields.Count)
                    {
                        await turnContext.SendActivityAsync("Welcome new user, please fill out your profile information.");
                    }

                    // We have at least one empty field. Prompt for the next empty field,
                    // and update the prompt flag to indicate which prompt we just sent,
                    // so that the response can be captured at the beginning of the next turn.
                    UserFieldInfo field = emptyFields.First();
                    await turnContext.SendActivityAsync(field.Prompt);
                    topicState.Prompt = field.Key;
                }
                else
                {
                    // Our user profile is complete!
                    await turnContext.SendActivityAsync($"Thank you, {userProfile.UserName}. Your profile is complete.");
                    topicState.Prompt = null;
                    topicState.Topic = null;
                }
            }
            else if (turnContext.Activity.Text.Trim().Equals("hi", StringComparison.InvariantCultureIgnoreCase))
            {
                await turnContext.SendActivityAsync($"Hi. {userProfile.UserName}.");
            }
            else
            {
                await turnContext.SendActivityAsync("Hi. I'm the Contoso cafe bot.");
            }

            // Use the state property accessors to update the topic state and user profile.
            await Accessors.TopicStateAccessor.SetAsync(turnContext, topicState, cancellationToken);
            await Accessors.UserProfileAccessor.SetAsync(turnContext, userProfile, cancellationToken);

            // Save any state changes to storage.
            await Accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
            await Accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript を選択した場合](#tab/javascript)

**app.js**

```javascript

server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === ActivityTypes.Message);

        // Set up a list of fields that we need to collect from the user.
        const fields = {
            userName: "What is your name?",
            age: "How old are you?",
            workPlace: "Where do you work?"
        }

        // Pull the state of the dialog out of the conversation state manager.
        const convo = await dialogState.get(context, {
            topic: 'profile',
            prompt: undefined
        });

        // Pull the user profile out of the user state manager.
        const userProfile = await userProfile.get(context, {  // Define the user's profile object
                "userName": undefined,
                "age": undefined,
                "workPlace": undefined
            }
        );

        if (isMessage) {

            if (convo.topic === 'profile') {
                // If a prompt flag is set in the conversation state, use it to capture the incoming value
                // into the appropriate field of the user profile.
                if (convo.prompt) {
                    userProfile[convo.prompt] = context.activity.text;
                }

                 // Determine which fields are not yet set.
                 const empty_fields = [];
                 Object.keys(fields).forEach(function(key) {
                    if (!userProfile[key]) {
                        empty_fields.push({
                           key: key,
                           prompt: fields[key]
                        });
                     }
                 });

                 if (empty_fields.length) {

                    // If all the fields are empty, send a welcome message.
                    if (empty_fields.length == Object.keys(fields).length) {
                        await context.sendActivity('Welcome new user, please fill out your profile information.');
                    }

                    // We have at least one empty field. Prompt for the next empty field.
                    await context.sendActivity(empty_fields[0].prompt);

                    // update the flag to indicate which prompt we just sent
                    // so that the response can be captured at the beginning of the next turn.
                    convo.prompt = empty_fields[0].key;

                 } else {
                    // Our user profile is complete!
                    await context.sendActivity('Thank you. Your profile is complete.');
                    convo.prompt = null;
                    convo.topic = null;

                 }
            } else if (context.activity.text && context.activity.text.match(/hi/ig)) {
                // Check to see if the user said "hi" and respond with a greeting
                await context.sendActivity(`Hi ${ userProfile.userName }.`);
            } else {
                // Default message
                await context.sendActivity("Hi. I'm the Contoso bot.");
            }
        }

        // End the turn by writing state changes back to storage
        await conversationState.saveChanges(context);
        await userState.saveChanges(context);
    });
});

```

---

上記のサンプル コードでは、プロファイル収集の会話を開始するために _topic_ フラグを `profile` に初期化します。 ボットは、_prompt_ フラグを現在の質問を表す値に更新することによって会話を進めます。 このフラグが適切な値に設定されると、ボットはユーザーから受信する次のメッセージの処理方法を認識し、適宜処理します。

最後に、ボットによる情報の要求が完了したら、フラグがリセットされます。 そうしないと、ボットがループに陥って最後の質問から次に進めなくなります。

他のフラグを定義するか、ユーザー入力に基づいて会話を分岐させて、このパターンを拡張することで、より複雑な会話フローを管理できます。

## <a name="manage-multiple-topics-of-conversations"></a>会話の複数のトピックを管理する

会話の 1 つのトピックを処理できたら、ボットのロジックを拡張して会話の複数のトピックを処理できます。 複数のトピックを処理するには、追加の条件をチェックして、適切なパスを選択します。

他の機能や会話のトピック (テーブルの予約や注文など) を使用できるように上記の例を拡張できます。 会話を追跡するために、必要に応じてフラグをトピックの状態に追加します。

また、コードを個別の関数またはメソッドに分割すると、会話のフローに従いやすくなり、便利な場合もあります。 一般的なパターンとしては、メッセージと状態をボットに評価させて、機能の詳細を実装する関数の制御を委任します。

ユーザーが会話の複数のトピックを適切に処理できるようにするには、メイン メニューの使用を検討してください。 たとえば、[推奨されるアクション](bot-builder-howto-add-suggested-actions.md)を使用して、ボットで行うことのできる処理の内容を推測するのではなく、参加するトピックをユーザーが選択できるようにします。

## <a name="validate-user-input"></a>ユーザー入力の検証

**ダイアログ** ライブラリには、ユーザーの入力を検証する方法が組み込まれていますが、独自のプロンプトを使用して検証を行うこともできます。 たとえば、ユーザーの年齢をたずねる場合、応答として "Bob" などではなく、数値を確実に取得する必要があります。

数値や日時の解析は複雑なタスクであり、このトピックの範囲外となります。 ただし、利用できるライブラリがあります。 この情報を解析するには、[Microsoft のテキスト認識エンジン](https://github.com/Microsoft/Recognizers-Text) ライブラリを使用します。 このパッケージは、NuGet から入手することも、リポジトリからダウンロードすることもできます。 このパッケージは**ダイアログ** ライブラリにも含まれています。これは注目すべき点ですが、このライブラリはここでは使用しません。

このライブラリは、日付、時刻、電話番号などの複雑な入力を解析する場合に特に役立ちます。 このサンプルでは、ディナー予約のグループの人数を検証していますが、同じ概念を拡張してさらに複雑な操作を行うこともできます。

次のサンプルでは、`RecognizeNumber` の使用方法のみを示しています。 ライブラリの他の Recognizer メソッドの使用方法の詳細については、その[リポジトリのドキュメント](https://github.com/Microsoft/Recognizers-Text/blob/master/README.md)をご覧ください。

# <a name="ctabcsharp"></a>[C# を選択した場合](#tab/csharp)

**Microsoft.Recognizers.Text.Number** ライブラリを使用するには、NuGet パッケージを含めて、そのパッケージを using ステートメントに追加します。

```csharp
using Microsoft.Recognizers.Text.Number;
using Microsoft.Recognizers.Text;
using System.Linq;
```

次に、実際に検証を実行する関数を定義します。

```csharp
private async Task<bool> ValidatePartySize(ITurnContext context, string value)
{
    try
    {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = NumberRecognizer.RecognizeNumber(value, Culture.English);

        // Attempt to convert the Recognizer result to an integer
        int.TryParse(result.First().Text, out int partySize);

        if (partySize < 6)
        {
            throw new Exception("Party size too small.");
        }
        else if (partySize > 20)
        {
            throw new Exception("Party size too big.");
        }

        // If we got through this, the number is valid
        return true;
    }
    catch (Exception)
    {
        await context.SendActivityAsync("Error with your party size. < br /> Please specify a number between 6 - 20.");
        return false;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript を選択した場合](#tab/javascript)

**認識エンジン** ライブラリを使用するには、**app.js** で要求します。

```javascript
// Required packages for this bot
var Recognizers = require('@microsoft/recognizers-text-suite');
```

次に、実際に検証を実行する関数を定義します。

```javascript
// Support party size between 6 and 20 only
async function validatePartySize(context, input){
    try {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = Recognizers.recognizeNumber(input, Recognizers.Culture.English);
        var value = parseInt(results[0].resolution.value);

        if (value < 6) {
            throw new Error(`Party size too small.`);
        } else if(value > 20){
            throw new Error(`Party size too big.`);
        }
        return true; // Return the valid value
    } catch (err){
        await context.sendActivity(`${err.message} <br/>Please specify a number between 6 - 20.`);
        return false;
    }
}
```

---

プロンプトに対するユーザーの応答を処理する場合は、次のプロンプトに進む前に検証関数を呼び出します。 検証に失敗した場合は、もう一度その質問を繰り返します。

# <a name="ctabcsharp"></a>[C# を選択した場合](#tab/csharp)

```csharp
if (topicState.Prompt == "partySize")
{
    if (await ValidatePartySize(turnContext, turnContext.Activity.Text))
    {
        // Save user's response in our state, ReservationInfo, which
        // is a new class we've added to our state
        // UserFieldInfo partySize;
        partySize.SetValue(userProfile, turnContext.Activity.Text);

        // Ask next question.
        topicState.Prompt = "reserveName";
        await turnContext.SendActivityAsync("Who's name will this be under?");
    }
    else
    {
        // Ask again.
        await turnContext.SendActivityAsync("How many people are in your party?");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript を選択した場合](#tab/javascript)

**app.js**

```javascript
// ...
if (convo.prompt == "partySize") {
    if (await validatePartySize(context, context.activity.text)) {
        // Save user's response
        reservationInfo.partySize = context.activity.text;

        // Ask next question
        convo.prompt = "reserveName";
        await context.sendActivity("Who's name will this be under?");
    } else {
        // Ask again
        await context.sendActivity("How many people are in your party?");
    }
}
// ...
```

---

## <a name="next-step"></a>次のステップ

ここでは、プロンプトの状態を自分で管理する方法を説明しました。次に、**ダイアログ** ライブラリを利用してユーザーに入力を求める方法を見てみましょう。

> [!div class="nextstepaction"]
> [ダイアログを使用してユーザーに入力を求める](bot-builder-prompts.md)
