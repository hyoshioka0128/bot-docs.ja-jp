---
title: ユーザーおよび会話データを保存する | Microsoft Docs
description: Bot Framework SDK を使用して状態データを保存および取得する方法について説明します。
keywords: 会話状態, ユーザー状態, 会話, 状態の保存, ボットの状態の管理
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/26/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4cafa3516395fb8e44d2755d0fa09e7a5bd6203c
ms.sourcegitcommit: 32615b88e4758004c8c99e9d564658a700c7d61f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/04/2019
ms.locfileid: "55711946"
---
# <a name="save-user-and-conversation-data"></a>ユーザーおよび会話データを保存する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ボットは本質的にステートレスです。 いったんご自身のボットがデプロイされると、ターンをまたいで、そのボットを同じプロセスやマシンで実行することはできません。 しかし、お使いのボットでは、会話のコンテキストの追跡が必要になることがあります。これにより、自身の動作を管理し、以前の質問に対する回答を記憶できるようになります。 SDK の状態とストレージの機能を使用すると、状態をお使いのボットに追加できます。

## <a name="prerequisites"></a>前提条件

- [ボットの基本](bot-builder-basics.md)、およびボットによる[状態の管理](bot-builder-concept-state.md)方法に関する知識が必要です。
- この記事のコードは、**StateBot** サンプルをベースにしています。 サンプルのコピー ([C#](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/StateBot) または [JS]()) が必要になります。
- ボットをローカルでテストするための [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started)。

## <a name="about-the-sample-code"></a>サンプル コードについて

この記事では、お使いのボットの状態管理における構成の側面について説明します。 状態を追加するには、状態のプロパティ、状態管理、およびストレージを構成してから、これらをボットで使用します。

- 各 "_状態プロパティ_" には、お使いのボットの状態情報が含まれています。
- 各状態プロパティ アクセサーを使用すると、関連付けられている状態プロパティの値を取得または設定できます。
- 各状態管理オブジェクトによって、関連付けられている状態情報の、ストレージからの読み取りと書き込みが自動化されます。
- ストレージ レイヤーは、インメモリ ストレージ (テスト用) や Azure Cosmos DB ストレージ (実稼働用) など、状態のバッキング ストレージに接続されます。

ボットは、状態プロパティ アクセサーを使用して構成する必要があります。アクティビティの処理中、このアクセサーを使って、実行時に状態を取得および設定できます。 状態プロパティ アクセサーは状態管理オブジェクトを使用して作成され、状態管理オブジェクトはストレージ レイヤーを使用して作成されます。 そこで、ストレージ レベルから説明を始め、上位レベルへと進んでいきます。

## <a name="configure-storage"></a>ストレージの構成

このボットはデプロイする予定がないため、"_メモリ ストレージ_" を使用します。次の手順で、これを使用して、ユーザー状態と会話状態の両方を構成します。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs** で、ストレージ レイヤーを構成します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    IStorage storage = new MemoryStorage();
    // ...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ご自身の **index.js** ファイルで、ストレージ レイヤーを構成します。

```javascript
// Define state store for your bot.
const memoryStorage = new MemoryStorage();
```

---

## <a name="create-state-management-objects"></a>状態管理オブジェクトを作成する

"_ユーザー_" 状態と "_会話_" 状態の両方を追跡し、次の手順ではこれらを使用して、"_状態プロパティ アクセサー_" を作成します。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs** で、ご自身の状態管理オブジェクトを作成するときに、ストレージ レイヤーを参照します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    ConversationState conversationState = new ConversationState(storage);
    UserState userState = new UserState(storage);
    // ...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js** ファイルで、`UserState` を require ステートメントに追加します。

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

次に、会話およびユーザーの状態管理オブジェクトを作成するときに、ストレージ レイヤーを参照します。

```javascript
// Create conversation and user state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);
```

---

## <a name="create-state-property-accessors"></a>状態プロパティ アクセサーを作成する

状態プロパティを "_宣言_" するには、まず状態管理オブジェクトのいずれかを使用して、状態プロパティ アクセサーを作成します。 次の情報を追跡するようにボットを構成します。

- ユーザーの名前。これはユーザー状態で定義します。
- ユーザーに対して、ユーザー名と、ユーザーが送信したメッセージに関する追加情報の入力を求めるかどうか。

ボットは、アクセサーを使用して、ターン コンテキストから状態プロパティを取得します。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

最初に、それぞれの状態の種類で、管理する必要があるすべての情報が格納されるようにクラスを定義します。

- ボットによって収集されるユーザー情報を表す `UserProfile` クラス。
- いつメッセージが到着し、誰がそのメッセージを送信したかを追跡する `ConversationData` クラス。

```csharp
// Defines a state property used to track information about the user.
public class UserProfile
{
    public string Name { get; set; }
}
```

```csharp
// Defines a state property used to track conversation data.
public class ConversationData
{
    // The time-stamp of the most recent incoming message.
    public string Timestamp { get; set; }

    // The ID of the user's channel.
    public string ChannelId { get; set; }

    // Track whether we have already asked the user's name
    public bool PromptedUserForName { get; set; } = false;
}
```

次に、ボット インスタンスの構成に必要な状態管理情報が格納されるクラスを定義します。

```csharp
public class StateBotAccessors
{
    public StateBotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }
  
    public static string UserProfileName { get; } = "UserProfile";

    public static string ConversationDataName { get; } = "ConversationData";

    public IStatePropertyAccessor<UserProfile> UserProfileAccessor { get; set; }

    public IStatePropertyAccessor<ConversationData> ConversationDataAccessor { get; set; }
  
    public ConversationState ConversationState { get; }
  
    public UserState UserState { get; }
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

状態管理オブジェクトをボットのコンストラクターに直接渡して、ボットが自身で状態プロパティ アクセサーを作成できるようにします。

**index.js** で、ボットを作成するときに、状態管理オブジェクトを指定します。

```javascript
// Create the bot.
const myBot = new MyBot(conversationState, userState);
```

**bot.js** で、状態の管理と追跡に必要な識別子を定義します。

```javascript
// The accessor names for the conversation data and user profile state property accessors.
const CONVERSATION_DATA_PROPERTY = 'conversationData';
const USER_PROFILE_PROPERTY = 'userProfile';
```

---

## <a name="configure-your-bot"></a>ボットを構成する

状態プロパティ アクセサーを定義して、ボットを構成する準備ができました。
会話フローの状態プロパティ アクセサーには、会話の状態管理オブジェクトを使用します。
ユーザー プロファイルの状態プロパティ アクセサーには、ユーザーの状態管理オブジェクトを使用します。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs** で、バンドルされている状態プロパティと管理オブジェクトを提供するように ASP.NET を構成します。 これは、ASP.NET Core の依存関係挿入フレームワークを介して、ボットのコンストラクターから取得されます。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    services.AddSingleton<StateBotAccessors>(sp =>
    {
        // Create the custom state accessor.
        return new StateBotAccessors(conversationState, userState)
        {
            ConversationDataAccessor = conversationState.CreateProperty<ConversationData>(StateBotAccessors.ConversationDataName),
            UserProfileAccessor = userState.CreateProperty<UserProfile>(StateBotAccessors.UserProfileName),
        };
    });
}
```

ボットのコンストラクターでは、ASP.NET によってボットが作成されるときに `StateBotAccessors` オブジェクトが提供されます。

```csharp
// Defines a bot for filling a user profile.
public class StateBot : IBot
{
    private readonly StateBotAccessors _accessors;

    public StateBot(StateBotAccessors accessors, ILoggerFactory loggerFactory)
    {
        // ...
        _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));
    }

    // The bot's turn handler and other supporting code...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

(**bot.js** ファイルの) ボットのコンストラクターで、状態プロパティ アクセサーを作成し、ボットに追加します。 また、状態管理オブジェクトへの参照も追加します。これらは、加えた状態変更を保存するうえで必要です。

```javascript
constructor(conversationState, userState) {
    // Create the state property accessors for the conversation data and user profile.
    this.conversationData = conversationState.createProperty(CONVERSATION_DATA_PROPERTY);
    this.userProfile = userState.createProperty(USER_PROFILE_PROPERTY);

    // The state management objects for the conversation and user state.
    this.conversationState = conversationState;
    this.userState = userState;
}
```

---

## <a name="access-state-from-your-bot"></a>お使いのボットから状態にアクセスする

前のセクションでは、状態プロパティ アクセサーをボットに追加するための、初期化時の手順について説明しました。
これらのアクセサーを実行時に使用すると、状態情報を読み書きできます。

1. 状態プロパティを使用する前に、各アクセサーを使用してストレージからプロパティを読み込んだうえで状態キャッシュから取得します。
   - そのアクセサーを使用して状態プロパティを取得するたびに、既定値を提供する必要があります。 そうしないと、null 値エラーが発生することがあります。
1. ターン ハンドラーを終了する前に、次を行います。
   1. アクセサーの _set_ メソッドを使用して、ボット状態に変更をプッシュします。
   1. 状態管理オブジェクトの _save changes_ メソッドを使用して、それらの変更をストレージに書き込みます。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// The bot's turn handler.
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the state properties from the turn context.
        UserProfile userProfile =
            await _accessors.UserProfileAccessor.GetAsync(turnContext, () => new UserProfile());
        ConversationData conversationData =
            await _accessors.ConversationDataAccessor.GetAsync(turnContext, () => new ConversationData());

        if (string.IsNullOrEmpty(userProfile.Name))
        {
            // First time around this is set to false, so we will prompt user for name.
            if (conversationData.PromptedUserForName)
            {
                // Set the name to what the user provided
                userProfile.Name = turnContext.Activity.Text?.Trim();

                // Acknowledge that we got their name.
                await turnContext.SendActivityAsync($"Thanks {userProfile.Name}.");

                // Reset the flag to allow the bot to go though the cycle again.
                conversationData.PromptedUserForName = false;
            }
            else
            {
                // Prompt the user for their name.
                await turnContext.SendActivityAsync($"What is your name?");

                // Set the flag to true, so we don't prompt in the next turn.
                conversationData.PromptedUserForName = true;
            }

            // Save user state and save changes.
            await _accessors.UserProfileAccessor.SetAsync(turnContext, userProfile);
            await _accessors.UserState.SaveChangesAsync(turnContext);
        }
        else
        {
            // Add message details to the conversation data.
            conversationData.Timestamp = turnContext.Activity.Timestamp.ToString();
            conversationData.ChannelId = turnContext.Activity.ChannelId.ToString();

            // Display state data
            await turnContext.SendActivityAsync($"{userProfile.Name} sent: {turnContext.Activity.Text}");
            await turnContext.SendActivityAsync($"Message received at: {conversationData.Timestamp}");
            await turnContext.SendActivityAsync($"Message received from: {conversationData.ChannelId}");
        }

        // Update conversation state and save changes.
        await _accessors.ConversationDataAccessor.SetAsync(turnContext, conversationData);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// The bot's turn handler.
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Get the state properties from the turn context.
        const userProfile = await this.userProfile.get(turnContext, {});
        const conversationData = await this.conversationData.get(
            turnContext, { promptedForUserName: false });

        if (!userProfile.name) {
            // First time around this is undefined, so we will prompt user for name.
            if (conversationData.promptedForUserName) {
                // Set the name to what the user provided.
                userProfile.name = turnContext.activity.text;

                // Acknowledge that we got their name.
                await turnContext.sendActivity(`Thanks ${userProfile.name}.`);

                // Reset the flag to allow the bot to go though the cycle again.
                conversationData.promptedForUserName = false;
            } else {
                // Prompt the user for their name.
                await turnContext.sendActivity('What is your name?');

                // Set the flag to true, so we don't prompt in the next turn.
                conversationData.promptedForUserName = true;
            }
            // Save user state and save changes.
            await this.userProfile.set(turnContext, userProfile);
            await this.userState.saveChanges(turnContext);
        } else {
            // Add message details to the conversation data.
            conversationData.timestamp = turnContext.activity.timestamp.toLocaleString();
            conversationData.channelId = turnContext.activity.channelId;

            // Display state data.
            await turnContext.sendActivity(`${userProfile.name} sent: ${turnContext.activity.text}`);
            await turnContext.sendActivity(`Message received at: ${conversationData.timestamp}`);
            await turnContext.sendActivity(`Message received from: ${conversationData.channelId}`);
        }
        // Update conversation state and save changes.
        await this.conversationData.set(turnContext, conversationData);
        await this.conversationState.saveChanges(turnContext);
    }
}
```

---

## <a name="test-the-bot"></a>ボットのテスト

1. ご自身のマシンを使ってローカルでサンプルを実行します。 手順については、README ファイルで [C#](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/StateBot) または [JS](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/js/stateBot) サンプルを参照してください。
1. 次に示すように、エミュレーターを使用してボットをテストします。

![状態テストのボットのサンプル](media/state-bot-testing-emulator.png)

## <a name="additional-resources"></a>その他のリソース

**プライバシー:** ユーザーの個人データを格納する場合は、[一般データ保護規則](https://blog.botframework.com/2018/04/23/general-data-protection-regulation-gdpr)に関するコンプライアンスを確保する必要があります。

**状態管理:** すべての状態管理呼び出しが非同期で行われます。これらの呼び出しは既定で Last-Writer-Wins です。 実際の場面では、状態の取得、設定、および保存は、ご自身のボット内で、できるだけ近いタイミングで行う必要があります。

**重要なビジネス データ:** ボットの状態には、詳細設定、ユーザー名、またはユーザーが行った最後の指示を格納しますが、重要なビジネス データを格納するためには使用しないでください。 重要なデータについては、[独自のストレージ コンポーネントを作成](bot-builder-custom-storage.md)するか、[ストレージ](bot-builder-howto-v4-storage.md)に直接書き込んでください。

**Recognizer-Text:** サンプルでは、ユーザー入力の解析および検証に Microsoft/Recognizers-Text ライブラリが使用されます。 詳細については、[概要](https://github.com/Microsoft/Recognizers-Text#microsoft-recognizers-text-overview)に関するページを参照してください。

## <a name="next-step"></a>次のステップ

ボット データをストレージに読み書きできるように状態を構成する方法がわかりました。次は、ユーザーに一連の質問を行い、その回答を検証して、入力を保存する方法を確認しましょう。

> [!div class="nextstepaction"]
> [ユーザー入力を収集するために独自のプロンプトを作成する](bot-builder-primitive-prompts.md)。
