---
title: ユーザーおよび会話データを保存する | Microsoft Docs
description: Bot Builder SDK for .NET を使用して状態データを保存および取得する方法について説明します。
keywords: 会話状態, ユーザー状態, 会話フロー
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/14/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5698c50b167e7162ef6910b7c428dab5ceb51d0e
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645642"
---
# <a name="save-user-and-conversation-data"></a>ユーザーおよび会話データを保存する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ボットを適切に設計するための鍵は、メッセージ交換のコンテキストを追跡することです。そうすることで、以前の質問に対する回答などの事柄をボットで記憶することができます。 ボットの使用目的によっては、会話の有効期間よりも長い間、状態を追跡または情報を保存することが必要な場合もあります。 ボットの*状態*は、受信メッセージに適切に応答するためにボットが記憶する情報です。 Bot Builder SDK は、ユーザーまたは会話に関連付けられたオブジェクトとして状態データを保存および取得するための 2 つのクラスを提供します。

- **会話状態**は、ボットがユーザーと交わしている現在の会話をボットが追跡するために役立ちます。 ボットがステップのシーケンスを完了する、または会話トピックを切り替える必要がある場合、会話プロパティを使用してシーケンス内のステップを管理、または現在のトピックを追跡することができます。 

- **ユーザー状態**は、ユーザーの以前の会話がどこで中断されたかを判断したり、戻ってきたユーザーにそのユーザーの名前を使って挨拶したりするなど、さまざまな目的で使用できます。 ユーザーの設定を保存すると、次回チャットするときにその情報を使用して会話をカスタマイズできます。 たとえば、ユーザーが関心を持っているトピックに関するニュース記事についてユーザーに通知したり、予約が利用可能になった場合にユーザーに通知したりできます。 

`ConversationState` と `UserState` は状態クラスであり、格納されたオブジェクトの有効期間とスコープをコントロールするポリシーによって `BotState` クラスが特殊化されたものです。 状態を格納する必要があるコンポーネントは、型を持つプロパティを作成および登録し、プロパティ アクセサーを使用して状態にアクセスします。 ボットの状態マネージャーはメモリ ストレージ、CosmosDB、Blob Storage を使用できます。 

> [!NOTE] 
> ボットの状態マネージャーは、詳細設定、ユーザー名、または最後に指示したものを格納しますが、重要なビジネス データを格納するためには使用しないでください。 重要なデータについては、独自のストレージ コンポーネントを作成するか、[ストレージ](bot-builder-howto-v4-storage.md)に直接書き込んでください。
> メモリ内データ ストレージはテスト専用です。 このストレージは揮発性であり、一時的なものです。 ボットの再起動ごとにデータがクリアされます。

## <a name="using-conversation-state-and-user-state-to-direct-conversation-flow"></a>会話状態とユーザー状態を使用して会話フローを制御する
会話フローの設計では、状態フラグを定義して会話フローを制御すると便利です。 フラグは、単純なブール型でも、現在のトピックの名前を含む型でもかまいません。 フラグは、会話内での現在位置を追跡するのに役立ちます。 たとえば、ブール型のフラグはユーザーが会話中であるかどうかを示す一方で、トピック名のプロパティは現在どの会話に参加しているかを示すことができます。



# <a name="ctabcsharp"></a>[C#](#tab/csharp)
### <a name="conversation-and-user-state"></a>会話状態とユーザー状態
このハウツーの開始点として、[カウンター付きのエコー ボットのサンプル](https://aka.ms/EchoBot-With-Counter)を使用できます。 まず、以下に示すように `TopicState` クラスを作成して、`TopicState.cs` 内の会話の現在のトピックを管理します。

```csharp
public class TopicState
{
   public string Prompt { get; set; } = "askName";
}
``` 
次に、`UserProfile` クラスを `UserProfile.cs` に作成してユーザー状態を管理します。
```csharp
public class UserProfile
{
    public string UserName { get; set; }
    public string TelephoneNumber { get; set; }
}
``` 
`TopicState` クラスには会話内のどこにいるかをトラックするフラグが用意されており、会話状態を使用してそれを格納します。 プロンプトは "askName" として初期化され、会話を開始します。 ボットがユーザーからの応答を受け取ると、プロンプトは "askNumber" として再定義されて次の会話を開始します。 `UserProfile` クラスは、ユーザーの名前と電話番号をトラックし、それをユーザー状態に格納します。

### <a name="property-accessors"></a>プロパティ アクセサー
この例における `EchoBotAccessors` クラスはシングルトンとして作成され、依存関係の挿入を通じて `class EchoWithCounterBot : IBot` コンストラクターに渡されます。 `EchoBotAccessors` クラスには、`ConversationState`、`UserState` および関連付けられた `IStatePropertyAccessor` が含まれます。 `conversationState` オブジェクトはトピック状態と、ユーザー プロファイル情報を格納する `userState` オブジェクトを格納します。 `ConversationState` オブジェクトと `UserState` オブジェクトは、後で Startup.cs ファイル内に作成されます。 会話状態オブジェクトとユーザー状態オブジェクトは、会話とユーザー スコープにおけるすべてを保持する場所です。 

以下に示すように、コンストラクターを更新して `UserState` を追加しました。
```csharp
public EchoBotAccessors(ConversationState conversationState, UserState userState)
{
    ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    UserState = userState ?? throw new ArgumentNullException(nameof(userState));
}
```
`TopicState` アクセサーと `UserProfile` アクセサー用の一意の名前を作成します。
```csharp
public static string UserProfileName { get; } = $"{nameof(EchoBotAccessors)}.UserProfile";
public static string TopicStateName { get; } = $"{nameof(EchoBotAccessors)}.TopicState";
```
次に、2 つのアクセサーを定義します。 1 つ目は会話のトピックを格納し、2 つ目はユーザーの名前を電話番号を格納します。
```csharp
public IStatePropertyAccessor<TopicState> TopicState { get; set; }
public IStatePropertyAccessor<UserProfile> UserProfile { get; set; }
```

ConversationState を取得するプロパティは既に定義されていますが、以下に示すように `UserState` を追加する必要があります。
```csharp
public ConversationState ConversationState { get; }
public UserState UserState { get; }
```
変更を行った後に、ファイルを保存します。 次に、Startup クラスを更新して `UserState` オブジェクトを作成し、ユーザー スコープにおけるすべてを保持します。 `ConversationState` は既に存在しています。 
```csharp

services.AddBot<EchoWithCounterBot>(options =>
{
    ...

    IStorage dataStore = new MemoryStorage();
    
    var conversationState = new ConversationState(dataStore);
    options.State.Add(conversationState);
        
    var userState = new UserState(dataStore);  
    options.State.Add(userState);
});
```
行 `options.State.Add(ConversationState);` と `options.State.Add(userState);` はそれぞれ会話状態とユーザー状態を追加します。 次に状態アクセサーを作成して登録します。 ここで作成されたアクセサーは、ターンごとに IBot 派生クラスに渡されます。 以下に示すように、コードを変更してユーザー状態を追加します。
```csharp
services.AddSingleton<EchoBotAccessors>(sp =>
{
   ...
    var userState = options.State.OfType<UserState>().FirstOrDefault();
    if (userState == null)
    {
        throw new InvalidOperationException("UserState must be defined and added before adding user-scoped state accessors.");
    }
   ...
 });
```

次に、`TopicState` と `UserProfile` を使用して 2 つのアクセサーを作成し、依存関係の挿入を通じて `class EchoWithCounterBot : IBot` クラスに渡します。
```csharp
services.AddSingleton<EchoBotAccessors>(sp =>
{
   ...
    var accessors = new EchoBotAccessors(conversationState, userState)
    {
        TopicState = conversationState.CreateProperty<TopicState>(EchoBotAccessors.TopicStateName),
        UserProfile = userState.CreateProperty<UserProfile>(EchoBotAccessors.UserProfileName),
     };

     return accessors;
 });
```

会話状態とユーザー状態は `services.AddSingleton` コード ブロックを介してシングルトンにリンクされ、`var accessors = new EchoBotAccessor(conversationState, userState)` で始まるコード内の状態ストア アクセサーを介して保存されます。

### <a name="use-conversation-and-user-state-properties"></a>会話とユーザーの状態プロパティを使用する 
`EchoWithCounterBot : IBot` クラスの `OnTurnAsync` ハンドラーで、ユーザーに名前、その後電話番号を求めるようにコードを変更します。 会話内でどこにいるかをトラックするには、TopicState で定義した Prompt プロパティを使用します。 このプロパティは初期化された "askName" でした。 ユーザー名を取得した後は、それを "askNumber" に設定し、UserName をユーザーが入力した名前に設定します。 電話番号を取得した後は、会話の最後であるため、確認メッセージを送信し、プロンプトを 'confirmation' に設定します。

```csharp
if (turnContext.Activity.Type == ActivityTypes.Message)
{
    // Get the conversation state from the turn context.
    var convo = await _accessors.TopicState.GetAsync(turnContext, () => new TopicState());
    
    // Get the user state from the turn context.
    var user = await _accessors.UserProfile.GetAsync(turnContext, () => new UserProfile());
    
    // Ask user name. The Prompt was initialiazed as "askName" in the TopicState.cs file.
    if (convo.Prompt == "askName")
    {
        await turnContext.SendActivityAsync("What is your name?");
        
        // Set the Prompt to ask the next question for this conversation
        convo.Prompt = "askNumber";
        
        // Set the property using the accessor
        await _accessors.TopicState.SetAsync(turnContext, convo);
        
        //Save the new prompt into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
    else if (convo.Prompt == "askNumber")
    {
        // Set the UserName that is defined in the UserProfile class
        user.UserName = turnContext.Activity.Text;
        
        // Use the user name to prompt the user for phone number
        await turnContext.SendActivityAsync($"Hello, {user.UserName}. What's your telephone number?");
        
        // Set the Prompt now that we have collected all the data
        convo.Prompt = "confirmation";
                 
        await _accessors.TopicState.SetAsync(turnContext, convo);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        await _accessors.UserProfile.SetAsync(turnContext, user);
        await _accessors.UserState.SaveChangesAsync(turnContext);
    }
    else if (convo.Prompt == "confirmation")
    { 
        // Set the TelephoneNumber that is defined in the UserProfile class
        user.TelephoneNumber = turnContext.Activity.Text;

        await turnContext.SendActivityAsync($"Got it, {user.UserName}. I'll call you later.");

        // reset initial prompt state
        convo.Prompt = "askName"; // Reset for a new conversation.
        
        await _accessors.TopicState.SetAsync(turnContext, convo);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
}
```   

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="conversation-and-user-state"></a>会話状態とユーザー状態

このハウツーの開始点として、[カウンター付きのエコー ボットのサンプル](https://aka.ms/EchoBot-With-Counter-JS)を使用できます。 このサンプルは既に `ConversationState` を使用してメッセージの数を格納しています。 `TopicStates` オブジェクトを追加して会話状態をトラックし、`UserState` を追加して `userProfile` オブジェクト内のユーザー状態をトラックする必要があります。 

メインのボットの `index.js` ファイルで、`UserState` を要求のリストに追加します。

**index.js**

```javascript
// Import required bot services. See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

次に、`MemoryStorage` をストレージ プロバイダーとして使用して `UserState` を作成し、その後それを `MainDialog` クラスの 2 番目の引数として渡します。

**index.js**

```javascript
// Create conversation state with in-memory storage provider. 
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);
// Create the main bot.
const bot = new EchBot(conversationState, userState);
```

`bot.js` ファイルで、`userState` を 2 番目の引数として受け入れるようにコンストラクターを更新します。 次に、`topicState` プロパティを `conversationState` から作成し、`userProfile` プロパティを `userState` から作成します。

**bot.js**

```javascript
const TOPIC_STATE = 'topic';
const USER_PROFILE = 'user';

constructor (conversationState, userState) {
    // creates a new state accessor property.see https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors 
    this.conversationState = conversationState;
    this.topicState = this.conversationState.createProperty(TOPIC_STATE);

    // User state
    this.userState = userState;
    this.userProfile = this.userState.createProperty(USER_PROFILE);
}
```

### <a name="use-conversation-and-user-state-properties"></a>会話とユーザーの状態プロパティを使用する

`MainDialog` クラスの `onTurn` ハンドラーで、ユーザーに名前、その後電話番号を求めるようにコードを変更します。 会話内でどこにいるかをトラックするには、`topicState` で定義した `prompt` プロパティを使用します。 このプロパティは "askName" に初期化されます。 ユーザー名を取得した後は、それを "askNumber" に設定し、UserName をユーザーが入力した名前に設定します。 電話番号を取得した後は、会話の最後であるため、確認メッセージを送信し、プロンプトを `undefined` に設定します。

**dialogs/mainDialog/index.js**

```javascript
// see https://aka.ms/about-bot-activity-message to learn more about the message and other activity types
if (turnContext.activity.type === 'message') {
    // read from state and set default object if object does not exist in storage.
    let topicState = await this.topicState.get(turnContext, {
        //Define the topic state object
        prompt: "askName"
    });
    let userProfile = await this.userProfile.get(turnContext, {  
        // Define the user's profile object
        "userName": "",
        "telephoneNumber": ""
    });

    if(topicState.prompt == "askName"){
        await turnContext.sendActivity("What is your name?");

        // Set next prompt state
        topicState.prompt = "askNumber";

        // Update state
        await this.topicState.set(turnContext, topicState);
    }
    else if(topicState.prompt == "askNumber"){
        // Set the UserName that is defined in the UserProfile class
        userProfile.userName = turnContext.activity.text;

        // Use the user name to prompt the user for phone number
        await turnContext.sendActivity(`Hello, ${userProfile.userName}. What's your telephone number?`);

        // Set next prompt state
        topicState.prompt = "confirmation";

        // Update states
        await this.topicState.set(turnContext, topicState);
        await this.userProfile.set(turnContext, userProfile);
    }
    else if(topicState.prompt == "confirmation"){
        // Set the phone number
        userProfile.telephoneNumber = turnContext.activity.text;

        // Sent confirmation
        await turnContext.sendActivity(`Got it, ${userProfile.userName}. I'll call you later.`)

        // reset initial prompt state
        topicState.prompt = "askName"; // Reset for a new conversation

        // Update states
        await this.topicState.set(turnContext, topicState);
        await this.userProfile.set(turnContext, userProfile);
    }
    
    // Save state changes to storage
    await this.conversationState.saveChanges(turnContext);
    await this.userState.saveChanges(turnContext);
    
}
else {
    await turnContext.sendActivity(`[${turnContext.activity.type} event detected]`);
}
```

---

## <a name="start-your-bot"></a>ボットの起動
- JavaScript の場合: ターミナルまたはコマンド プロンプトで、ボット用に作成したディレクトリに変更し、`npm start` でボットを起動します。 この時点では、ボットはローカルで実行されています。

- C# ボットの場合: Visual Studio を使用して、お使いのボットをローカルで実行します。 実行ボタンをクリックすると、Visual Studio によってアプリケーションが構築され、localhost に配置されます。その後、Web ブラウザーが起動されアプリケーションの ``default.htm`` ページが表示されます。 この時点では、ボットはローカルで実行されています。

### <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続
次に、エミュレーターを起動し、エミュレーターのボットに接続します。

1. エミュレーターの [ようこそ] タブにある **[Open Bot]\(ボットを開く\)** リンクをクリックします。 
2. Visual Studio ソリューションを作成したディレクトリにある .bot ファイルを選択します。

### <a name="interact-with-your-bot"></a>ボットでのやり取り

お使いのボットに "Hi" というメッセージを送信すると、そのボットによって、名前と電話番号を指定するように求められます。 その情報の入力後、ボットにより確認メッセージが送信されます。 操作を続行すると、ボットによって同じサイクルがもう一度繰り返されます。

![実行中のエミュレーター](../media/emulator-v4/emulator-running-manage-state.png)

状態を自身で管理することを選択した場合は、[独自のプロンプトで会話フローを管理する](bot-builder-primitive-prompts.md)方法に関する記事をご覧ください。 または、ウォーターフォール ダイアログを使用します。 ダイアログは会話の状態を自動的に追跡するので、ユーザーは状態を追跡するためのフラグを作成する必要はありません。 詳しくは、[ダイアログを使用した簡単な会話の管理](bot-builder-dialog-manage-conversation-flow.md)に関する記事をご覧ください。

## <a name="next-steps"></a>次の手順
状態を使用してストレージのボット データを読み書きする方法がわかったので、ストレージを直接読み書きする方法を確認してください。

> [!div class="nextstepaction"]
> [ストレージに直接書き込む方法](bot-builder-howto-v4-storage.md)。
