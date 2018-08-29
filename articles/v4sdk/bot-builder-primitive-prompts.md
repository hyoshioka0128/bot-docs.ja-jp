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
ms.openlocfilehash: 1514e1bcafc87be9e8449382bca7aa156e512ed9
ms.sourcegitcommit: e8c513d3af5f0c514cadcbcd0a737a7393405afa
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/20/2018
ms.locfileid: "40228347"
---
# <a name="prompt-users-for-input-using-your-own-prompts"></a>独自のプロンプトを使用してユーザーに入力を求める
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

多くの場合、ボットとユーザー間の会話では、ユーザーに情報の入力を求め、ユーザーの応答を解析し、その情報に基づいてアクションを実行する必要があります。 [ダイアログ ライブラリ](bot-builder-prompts.md)を使用したユーザーへの入力の要求に関するトピックでは、**ダイアログ** ライブラリを使用してユーザーに入力を求める方法について詳しく説明しています。 特に、**ダイアログ** ライブラリでは、現在の会話と現在の質問の追跡を処理します。 また、テキスト、数値、日時などのさまざまな種類の情報を要求するためのメソッドも用意されています。 

状況によっては、組み込みの**ダイアログ**がボットに適したソリューションではない場合があります。**ダイアログ**は単純なボットに余分なオーバーヘッドを追加したり、柔軟性に欠けることがあります。また、ボットで実行する必要があることを実現できない場合もあります。 このような場合は、ライブラリをスキップし、独自のプロンプト ロジックを実装できます。 このトピックでは、独自のプロンプトを使用して会話を管理できるように、基本的な*エコー ボット*を設定する方法について説明します。

## <a name="track-prompt-states"></a>プロンプトの状態を追跡する

プロンプト駆動型の会話では、会話の進行状況と現在行われている質問を追跡する必要があります。 コードでは、これはいくつかのフラグの管理になります。 

たとえば、追跡するプロパティをいくつか作成できます。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ここでは、プロンプト情報に入れ子になったユーザー プロファイル クラスがあり、これらのすべてをボットの[状態](bot-builder-howto-v4-state.md)にまとめて保存できます。

```csharp
// Where user information will be stored
public class UserProfile
{
    public string userName = null;
    public string workPlace = null;
    public int age = 0;
}

// Flags to keep track of our prompts, and the 
// user profile object for this conversation
public class PromptInformation
{
    public string topicTitle = null;
    public string prompt = null;
    public UserProfile userProfile = null;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

これらの状態によって、現在進行中のトピックとプロンプトが追跡されます。 クラウドにデプロイしたときに、これらのフラグが想定どおりに機能するように、[会話の状態](bot-builder-howto-v4-state.md)にフラグを保存します。 このコードをボットのメイン ロジック内に配置します。

**app.js**
```javascript
// Define a topicStates object if it doesn't exist in the convoState.
if(!convo.topicStates){
    convo.topicStates = {
        "topicTitle": undefined, // Current conversation topic in progress
        "prompt": undefined      // Current prompt state in progress - indicate what question is being asked.
    };
}
```

---

## <a name="manage-a-topic-of-conversation"></a>会話の 1 つのトピックを管理する

ユーザーから情報を収集する会話など、連続して行われる会話では、ユーザーがいつ会話に参加し、会話がどこまで進んでいるのかを把握する必要があります。 上記で定義したプロンプト フラグを設定して確認し、状況に応じてアクションを実行することで、ボットのメイン ターン ハンドラーでこれを追跡できます。 次のサンプルは、これらのフラグを使用して、会話を通じてユーザーのプロファイル情報を収集する方法を示しています。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Get our current state, as defined above
var convoState = context.GetConversationState<PromptInformation>();

if (convoState.userProfile == null)
{
    await context.SendActivity("Welcome new user, please fill out your profile information.");
    convoState.topicTitle = "profileTopic"; // Start the userProfile topic
    convoState.userProfile = new UserProfile();
}

// Start or continue a conversation if we are in one
if (convoState.topicTitle == "profileTopic")
{
    if (convoState.userProfile.userName == null && convoState.prompt == null)
    {
        convoState.prompt = "askName";
        await context.SendActivity("What is your name?");
    }
    else if (convoState.prompt == "askName")
    {
        // Save the user's response
        convoState.userProfile.userName = context.Activity.Text;

        // Ask next question
        convoState.prompt = "askAge";
        await context.SendActivity("How old are you?");
    }
    else if (convoState.prompt == "askAge")
    {
        // Save user's response
        if (!Int32.TryParse(context.Activity.Text, out convoState.userProfile.age))
        {
            // Failed to convert to int
            await context.SendActivity("Failed to convert string to int");
        }
        else
        {
            // Ask next question
            convoState.prompt = "workPlace";
            await context.SendActivity("Where do you work?");
        }

    }
    else if (convoState.prompt == "workPlace")
    {
        // Save user's response
        convoState.userProfile.workPlace = context.Activity.Text;

        // Done
        convoState.topicTitle = null; // Reset conversation flag
        convoState.prompt = null;     // Reset the prompt flag
        
        await context.SendActivity("Thank you. Your profile is complete.");
    }
    else // Somehow our flags got inappropriately set
    {
        await context.SendActivity("Looks like something went wrong, let's start over");
        convoState.userProfile = null;
        convoState.prompt = null;
        convoState.topicTitle = null;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**
```javascript
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);

        // Defined flags to manage the conversation flow and prompt states
        // convo.topicStates.topicTitle - Current conversation topic in progress
        // convo.topicStates.prompt - Current prompt state in progress - indicate what question is being asked.
        
        if (isMessage) {
            // Define a topicStates object if it doesn't exist in the convoState.
            if(!convo.topicStates){
                convo.topicStates = {
                    "topicTitle": undefined, // Current conversation topic in progress
                    "prompt": undefined      // Current prompt state in progress - indicate what question is being asked.
                };
            }
            
            // If user profile is not defined then define it.
            if(!convo.userProfile){
                
                await context.sendActivity(`Welcome new user, please fill out your profile information.`);
                convo.topicStates.topicTitle = "profileTopic"; // Start the userProfile topic
                convo.userProfile = { // Define the user's profile object
                    "userName": undefined,
                    "age": undefined,
                    "workPlace": undefined
                }; 
            }

            // Start or continue a conversation if we are in one
            if(convo.topicStates.topicTitle == "profileTopic"){
                if(!convo.userProfile.userName && !convo.topicStates.prompt){
                    convo.topicStates.prompt = "askName";
                    await context.sendActivity("What is your name?");
                }
                else if(convo.topicStates.prompt == "askName"){
                    // Save the user's response
                    convo.userProfile.userName = context.activity.text; 

                    // Ask next question
                    convo.topicStates.prompt = "askAge";
                    await context.sendActivity("How old are you?");
                }
                else if(convo.topicStates.prompt == "askAge"){
                    // Save user's response
                    convo.userProfile.age = context.activity.text;

                    // Ask next question
                    convo.topicStates.prompt = "workPlace";
                    await context.sendActivity("Where do you work?");
                }
                else if(convo.topicStates.prompt == "workPlace"){
                    // Save user's response
                    convo.userProfile.workPlace = context.activity.text;

                    // Done
                    convo.topicStates.topicTitle = undefined; // Reset conversation flag
                    convo.topicStates.prompt = undefined;     // Reset the prompt flag
                    await context.sendActivity("Thank you. Your profile is complete.");
                }
            }

            // Check for valid intents
            else if(context.activity.text && context.activity.text.match(/hi/ig)){
                await context.sendActivity(`Hi ${convo.userProfile.userName}.`);
            }
            else {
                // Default message
                await context.sendActivity("Hi. I'm the Contoso bot.");
            }
        }

    });
});

```

---

上記のサンプル コードでは、ユーザーのプロファイルがメモリ内に定義されているかどうかを確認します。 定義されておらず、これが新しいユーザーの場合は、そのトピックを自動的に開始するようにフラグを設定します。 次に、`prompt` フラグを現在の質問の値に設定して会話を進める方法を示しています。 このフラグが適切な値に設定されると、条件句が各ターンでユーザーの応答をキャッチし、適宜処理します。 

最後に、情報の要求が完了したら、これらのフラグをリセットして、ボットがループに陥らないようにする必要があります。 他のフラグを定義するか、ユーザー入力に基づいて会話を分岐させて、この基本的な設定を拡張することで、ボットに必要なより複雑な会話フローを管理できます。

## <a name="manage-multiple-topics-of-conversations"></a>会話の複数のトピックを管理する

会話の 1 つのトピックを処理できたら、ボットのロジックを拡張して会話の複数のトピックを処理できます。 会話の 1 つのトピックと同様に、特定のトピックをトリガーする条件をチェックし、適切なパスを選択するだけで、複数のトピックを処理できます。

上記の例では、他の関数やトピック (レストランの予約やディナーの注文など) を使用できるようにリファクタリングできます。 会話を追跡するために、必要に応じてユーザー プロファイルやトピックの状態フラグに情報を追加できます。

会話の複数のトピックを適切に管理できるようにするには、1 つの方法としてメイン メニューを提供します。 [推奨されるアクション](bot-builder-howto-add-suggested-actions.md)を使用して、参加するトピックをユーザーが選択できるようにし、そのトピックの会話をすぐに開始できます。 また、コードを個別の関数に分割すると、会話のフローに従いやすくなり、便利な場合もあります。

## <a name="validate-user-input"></a>ユーザー入力の検証

**ダイアログ** ライブラリには、ユーザーの入力を検証する方法が組み込まれていますが、独自のプロンプトを使用して検証を行うこともできます。 たとえば、ユーザーの年齢をたずねる場合、応答として "Bob" などではなく、数値を確実に取得する必要があります。

数値や日時の解析は複雑なタスクであり、このトピックの範囲外となります。 ただし、利用できるライブラリがあります。 この情報を適切に解析するには、[Microsoft のテキスト認識エンジン](https://github.com/Microsoft/Recognizers-Text) ライブラリを使用します。 このパッケージは、NuGet から入手することも、リポジトリからダウンロードすることもできます。 さらに、このパッケージは**ダイアログ** ライブラリにも含まれています。これは注目すべき点ですが、このライブラリはここでは使用しません。

テキスト認識エンジン ライブラリは、共通言語や、日付、時刻、電話番号などに関する複雑な応答を解析する際に特に役立ちます。 このサンプルでは、ディナー予約のグループの人数だけを検証していますが、同じ概念を拡張してさらに詳しい検証を行うこともできます。 このライブラリを使い慣れていない場合は、該当のサイトのコンテンツを見直して、提供される内容を確認してください。

このサンプルでは、`RecognizeNumber` の使用方法のみを示しています。 ライブラリの他の Recognizer メソッドの使用方法の詳細については、その[リポジトリのドキュメント](https://github.com/Microsoft/Recognizers-Text/blob/master/README.md)をご覧ください。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

認識エンジン ライブラリを使用するには、using ステートメントに追加します。

```csharp
using Microsoft.Recognizers.Text.Number;
using Microsoft.Recognizers.Text;
using System.Linq; // Used to get the first result from the recognizer
```

次に、実際に検証を実行する関数を定義します。

```csharp
private async Task<bool> ValidatePartySize(ITurnContext context, string value)
{
    try
    {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = NumberRecognizer.RecognizeNumber(input, Culture.English);

        // Attempt to convert the Recognizer result to an integer
        Int32.TryParse(result.First().Text, out int partySize);
        
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
        await context.SendActivity("Error with your party size. < br /> Please specify a number between 6 - 20.");
        return false;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

認識エンジン ライブラリを使用するには、**app.js** で要求します。

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

        if(value < 6) {
            throw new Error(`Party size too small.`);
        }
        else if(value > 20){
            throw new Error(`Party size too big.`);
        }
        return true; // Return the valid value
    }
    catch (err){
        await context.sendActivity(`${err.message} <br/>Please specify a number between 6 - 20.`);
        return false;
    }
}
```

---

検証するプロンプト ステップ内で、次のプロンプトに進む前に検証関数を呼び出します。 検証に失敗した場合は、もう一度その質問を繰り返します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
if (convoState.prompt == "partySize")
{
    if (await ValidatePartySize(context, context.Activity.Text))
    {
        // Save user's response in our state, ReservationInfo, which 
        // is a new class we've added to our state
        convoState.ReservationInfo.partySize = context.Activity.Text;

        // Ask next question
        convoState.prompt = "reserveName";
        await context.SendActivity("Who's name will this be under?");
    }
    else
    {
        // Ask again
        await context.SendActivity("How many people are in your party?");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**

```javascript
// ...
if(convo.topicStates.prompt == "partySize"){
    if(await validatePartySize(context, context.activity.text)){
        // Save user's response
        convo.reservationInfo.partySize = context.activity.text;
        
        // Ask next question
        topicStates.prompt = "reserveName";
        await context.sendActivity("Who's name will this be under?");
    }
    else {
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
