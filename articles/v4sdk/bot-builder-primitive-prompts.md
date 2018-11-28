---
title: ユーザー入力を収集するために独自のプロンプトを作成する | Microsoft Docs
description: Bot Builder SDK でプリミティブなプロンプトを使用して会話フローを管理する方法について説明します。
keywords: 会話フロー, プロンプト, 会話状態, ユーザー状態, カスタム プロンプト
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/20/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e308457e9fb228dd141ec081ac3c5daa5fd54cac
ms.sourcegitcommit: 6cb37f43947273a58b2b7624579852b72b0e13ea
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/22/2018
ms.locfileid: "52288821"
---
# <a name="create-your-own-prompts-to-gather-user-input"></a>ユーザー入力を収集するために独自のプロンプトを作成する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

多くの場合、ボットとユーザー間の会話では、ユーザーに情報の入力を求め、ユーザーの応答を解析し、その情報に基づいてアクションを実行する必要があります。

お使いのボットは会話のコンテキストを追跡する必要があります。これにより、自身の動作を管理し、以前の質問に対する回答を記憶することができます。 ボットの "*状態*" は、受信メッセージに適切に応答するためにボットが追跡する情報です。

## <a name="prerequisites"></a>前提条件

- この記事のコードは、**ユーザーに入力を求める**サンプルをベースにしています。 サンプルのコピー ([C#](https://aka.ms/cs-primitive-prompt-sample) または [JS](https://aka.ms/js-primitive-prompt-sample)) が必要になります。
- [状態の管理](bot-builder-concept-state.md)と[ユーザー データおよび会話データの保存](bot-builder-howto-v4-state.md)方法に関する知識。
- ボットをローカルでテストするための [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started)。

## <a name="about-the-sample-code"></a>サンプル コードについて

この記事では、ユーザーに対して一連の質問を行い、その回答の一部を検証して、入力を保存します。
会話のフローと入力のコレクションの管理には、ボットのターン ハンドラーと、ユーザーおよび会話の状態プロパティを使用します。

1. 状態を定義して構成します
1. 状態プロパティを使用して会話を制御します
   1. ボットのターン ハンドラーを更新します。
   1. ユーザー データのコレクションを管理するヘルパー メソッドを実装します。
   1. ユーザー入力の検証メソッドを実装します。

## <a name="define-and-configure-state"></a>状態を定義して構成する

次の情報を追跡するようにボットを構成する必要があります。

- ユーザーの名前、年齢、および選択した日付。これらはユーザー状態で定義する情報です。
- ユーザーに質問した内容。これは会話状態で定義する情報です。

このボットはデプロイする予定がないため、"_メモリ ストレージ_" を使用するように、ユーザー状態と会話情報の両方を構成します。 構成コードの重要な側面についていくつか次に説明します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

次の種類を定義します。

- ボットによって収集されるユーザー情報を表す `UserProfile` クラス。
- 会話の進行状況に関する情報を追跡するための `ConversationFlow` クラス。
- 会話の進行状況を追跡するための内部 `ConversationFlow.Question` 列挙型。
- 状態管理情報をバンドルする `CustomPromptBotAccessors` クラス。

ボット アクセサー クラスには、状態管理オブジェクトと状態プロパティ アクセサー オブジェクトが含まれています。このクラスは、ASP.NET Core の依存関係の挿入を介してボットに渡されます。 ボットでは、各ターンの作成時に受信される状態プロパティ アクセサー情報を記録します。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

状態管理オブジェクトを作成し、ボットの作成時に渡します。
ボットでは、状態プロパティの識別子と、会話の進行状況を追跡するための識別子を定義し、状態管理オブジェクトを記録して、ボットのコンストラクターで状態プロパティ アクセサーを作成します。

---

## <a name="use-state-properties-to-direct-the-conversation"></a>状態プロパティを使用して会話を制御する

状態プロパティを構成したら、それをボットで使用できます。

- 状態にアクセスしてヘルパー メソッドを呼び出す[ターン ハンドラー](#the-bots-turn-handler)を定義します。
- ユーザー プロファイルのコレクションを管理する[ヘルパー メソッド](#filling-out-the-user-profile)を実装します。
- ユーザー入力を解析して検証する[検証メソッド](#parse-and-validate-input)を実装します。

### <a name="the-bots-turn-handler"></a>ボットのターン ハンドラー

状態プロパティ アクセサーを使用して、ターン コンテキストから状態プロパティを取得します。
ユーザー プロファイルを入力する必要がある場合は、ヘルパー メソッドを呼び出して、状態の変更を保存します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
   if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the state properties from the turn context.
        ConversationFlow flow = await _accessors.ConversationFlowAccessor.GetAsync(turnContext, () => new ConversationFlow());
        UserProfile profile = await _accessors.UserProfileAccessor.GetAsync(turnContext, () => new UserProfile());

        await FillOutUserProfileAsync(flow, profile, turnContext);

        // Update state and save changes.
        await _accessors.ConversationFlowAccessor.SetAsync(turnContext, flow);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        await _accessors.UserProfileAccessor.SetAsync(turnContext, profile);
        await _accessors.UserState.SaveChangesAsync(turnContext);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// The bot's turn handler.
async onTurn(turnContext) {
    // This bot listens for message activities.
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Get the state properties from the turn context.
        const flow = await this.conversationFlow.get(turnContext, { lastQuestionAsked: question.none });
        const profile = await this.userProfile.get(turnContext, {});

        await MyBot.fillOutUserProfile(flow, profile, turnContext);

        // Update state and save changes.
        await this.conversationFlow.set(turnContext, flow);
        await this.conversationState.saveChanges(turnContext);

        await this.userProfile.set(turnContext, profile);
        await this.userState.saveChanges(turnContext);
    }
}
```

---

### <a name="filling-out-the-user-profile"></a>ユーザー プロファイルを入力する

まず情報を収集します。 それぞれに同様のインターフェイスが用意されています。

- 戻り値は、入力が、この質問に対して有効な回答であるかどうか示しています。
- 検証に合格すると、解析および正規化された値が生成され、保存されます。
- 検証に失敗した場合はメッセージが生成され、ボットはこれを使用して情報を再度求めることができます。

 [次のセクション](#parse-and-validate-input)では、ユーザー入力を解析して検証するヘルパー メソッドを定義します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private static async Task FillOutUserProfileAsync(ConversationFlow flow, UserProfile profile, ITurnContext turnContext)
{
    string input = turnContext.Activity.Text?.Trim();
    string message;
    switch (flow.LastQuestionAsked)
    {
        case ConversationFlow.Question.None:
            await turnContext.SendActivityAsync("Let's get started. What is your name?");
            flow.LastQuestionAsked = ConversationFlow.Question.Name;
            break;
        case ConversationFlow.Question.Name:
            if (ValidateName(input, out string name, out message))
            {
                profile.Name = name;
                await turnContext.SendActivityAsync($"Hi {profile.Name}.");
                await turnContext.SendActivityAsync("How old are you?");
                flow.LastQuestionAsked = ConversationFlow.Question.Age;
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
        case ConversationFlow.Question.Age:
            if (ValidateAge(input, out int age, out message))
            {
                profile.Age = age;
                await turnContext.SendActivityAsync($"I have your age as {profile.Age}.");
                await turnContext.SendActivityAsync("When is your flight?");
                flow.LastQuestionAsked = ConversationFlow.Question.Date;
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
        case ConversationFlow.Question.Date:
            if (ValidateDate(input, out string date, out message))
            {
                profile.Date = date;
                await turnContext.SendActivityAsync($"Your cab ride to the airport is scheduled for {profile.Date}.");
                await turnContext.SendActivityAsync($"Thanks for completing the booking {profile.Name}.");
                await turnContext.SendActivityAsync($"Type anything to run the bot again.");
                flow.LastQuestionAsked = ConversationFlow.Question.None;
                profile = new UserProfile();
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
    }
}


```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Manages the conversation flow for filling out the user's profile.
static async fillOutUserProfile(flow, profile, turnContext) {
    const input = turnContext.activity.text;
    let result;
    switch (flow.lastQuestionAsked) {
        // If we're just starting off, we haven't asked the user for any information yet.
        // Ask the user for their name and update the conversation flag.
        case question.none:
            await turnContext.sendActivity("Let's get started. What is your name?");
            flow.lastQuestionAsked = question.name;
            break;

        // If we last asked for their name, record their response, confirm that we got it.
        // Ask them for their age and update the conversation flag.
        case question.name:
            result = this.validateName(input);
            if (result.success) {
                profile.name = result.name;
                await turnContext.sendActivity(`I have your name as ${profile.name}.`);
                await turnContext.sendActivity('How old are you?');
                flow.lastQuestionAsked = question.age;
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }

        // If we last asked for their age, record their response, confirm that we got it.
        // Ask them for their date preference and update the conversation flag.
        case question.age:
            result = this.validateAge(input);
            if (result.success) {
                profile.age = result.age;
                await turnContext.sendActivity(`I have your age as ${profile.age}.`);
                await turnContext.sendActivity('When is your flight?');
                flow.lastQuestionAsked = question.date;
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }

        // If we last asked for a date, record their response, confirm that we got it,
        // let them know the process is complete, and update the conversation flag.
        case question.date:
            result = this.validateDate(input);
            if (result.success) {
                profile.date = result.date;
                await turnContext.sendActivity(`Your cab ride to the airport is scheduled for ${profile.date}.`);
                await turnContext.sendActivity(`Thanks for completing the booking ${profile.name}.`);
                await turnContext.sendActivity('Type anything to run the bot again.');
                flow.lastQuestionAsked = question.none;
                profile = {};
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }
    }
}
```

---

### <a name="parse-and-validate-input"></a>入力を解析して検証する

次の条件を使用して、入力を検証します。

- **name** は、空でない文字列にする必要があります。 空白文字を削除することで正規化します。
- **age** は、18 から 120 の値にする必要があります。 整数を返すことで正規化します。
- **date** は、1 時間以上未来の日付または時刻にする必要があります。
  解析された入力の日付部分のみを返すことで正規化します。

> [!NOTE]
> age と date の入力については、[Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text/) ライブラリを使用して、初期解析を実行します。
> サンプル コードは提供されますが、テキスト認識エンジン ライブラリの動作方法については説明されていません。また、これは入力を解析する方法の一例にすぎません。
> これらのライブラリの詳細については、リポジトリの **README** を参照してください。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

次の検証メソッドをお使いのボットに追加します。

```csharp
private static bool ValidateName(string input, out string name, out string message)
{
    name = null;
    message = null;

    if (string.IsNullOrWhiteSpace(input))
    {
        message = "Please enter a name that contains at least one character.";
    }
    else
    {
        name = input.Trim();
    }

    return message is null;
}

private static bool ValidateAge(string input, out int age, out string message)
{
    age = 0;
    message = null;

    // Try to recognize the input as a number. This works for responses such as "twelve" as well as "12".
    try
    {
        // Attempt to convert the Recognizer result to an integer. This works for "a dozen", "twelve", "12", and so on.
        // The recognizer returns a list of potential recognition results, if any.
        List<ModelResult> results = NumberRecognizer.RecognizeNumber(input, Culture.English);
        foreach (var result in results)
        {
            // result.Resolution is a dictionary, where the "value" entry contains the processed string.
            if (result.Resolution.TryGetValue("value", out object value))
            {
                age = Convert.ToInt32(value);
                if (age >= 18 && age <= 120)
                {
                    return true;
                }
            }
        }

        message = "Please enter an age between 18 and 120.";
    }
    catch
    {
        message = "I'm sorry, I could not interpret that as an age. Please enter an age between 18 and 120.";
    }

    return message is null;
}

private static bool ValidateDate(string input, out string date, out string message)
{
    date = null;
    message = null;

    // Try to recognize the input as a date-time. This works for responses such as "11/14/2018", "9pm", "tomorrow", "Sunday at 5pm", and so on.
    // The recognizer returns a list of potential recognition results, if any.
    try
    {
        List<ModelResult> results = DateTimeRecognizer.RecognizeDateTime(input, Culture.English);

        // Check whether any of the recognized date-times are appropriate,
        // and if so, return the first appropriate date-time. We're checking for a value at least an hour in the future.
        DateTime earliest = DateTime.Now.AddHours(1.0);
        foreach (ModelResult result in results)
        {
            // result.Resolution is a dictionary, where the "values" entry contains the processed input.
            var resolutions = result.Resolution["values"] as List<Dictionary<string, string>>;
            foreach (var resolution in resolutions)
            {
                // The processed input contains a "value" entry if it is a date-time value, or "start" and
                // "end" entries if it is a date-time range.
                if (resolution.TryGetValue("value", out string dateString)
                    || resolution.TryGetValue("start", out dateString))
                {
                    if (DateTime.TryParse(dateString, out var candidate)
                        && earliest < candidate)
                    {
                        date = candidate.ToShortDateString();
                        return true;
                    }
                }
            }
        }

        message = "I'm sorry, please enter a date at least an hour out.";
    }
    catch
    {
        message = "I'm sorry, I could not interpret that as an appropriate date. Please enter a date at least an hour out.";
    }

    return false;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

次の検証メソッドをお使いのボットに追加します。

```javascript
// Validates name input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateName(input) {
    const name = input && input.trim();
    return name != undefined
        ? { success: true, name: name }
        : { success: false, message: 'Please enter a name that contains at least one character.' };
};

// Validates age input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateAge(input) {

    // Try to recognize the input as a number. This works for responses such as "twelve" as well as "12".
    try {
        // Attempt to convert the Recognizer result to an integer. This works for "a dozen", "twelve", "12", and so on.
        // The recognizer returns a list of potential recognition results, if any.
        const results = Recognizers.recognizeNumber(input, Recognizers.Culture.English);
        let output;
        results.forEach(function (result) {
            // result.resolution is a dictionary, where the "value" entry contains the processed string.
            const value = result.resolution['value'];
            if (value) {
                const age = parseInt(value);
                if (!isNaN(age) && age >= 18 && age <= 120) {
                    output = { success: true, age: age };
                    return;
                }
            }
        });
        return output || { success: false, message: 'Please enter an age between 18 and 120.' };
    } catch (error) {
        return {
            success: false,
            message: "I'm sorry, I could not interpret that as an age. Please enter an age between 18 and 120."
        };
    }
}

// Validates date input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateDate(input) {
    // Try to recognize the input as a date-time. This works for responses such as "11/14/2018", "today at 9pm", "tomorrow", "Sunday at 5pm", and so on.
    // The recognizer returns a list of potential recognition results, if any.
    try {
        const results = Recognizers.recognizeDateTime(input, Recognizers.Culture.English);
        const now = new Date();
        const earliest = now.getTime() + (60 * 60 * 1000);
        let output;
        results.forEach(function (result) {
            // result.resolution is a dictionary, where the "values" entry contains the processed input.
            result.resolution['values'].forEach(function (resolution) {
                // The processed input contains a "value" entry if it is a date-time value, or "start" and
                // "end" entries if it is a date-time range.
                const datevalue = resolution['value'] || resolution['start'];
                // If only time is given, assume it's for today.
                const datetime = resolution['type'] === 'time'
                    ? new Date(`${now.toLocaleDateString()} ${datevalue}`)
                    : new Date(datevalue);
                if (datetime && earliest < datetime.getTime()) {
                    output = { success: true, date: datetime.toLocaleDateString() };
                    return;
                }
            });
        });
        return output || { success: false, message: "I'm sorry, please enter a date at least an hour out." };
    } catch (error) {
        return {
            success: false,
            message: "I'm sorry, I could not interpret that as an appropriate date. Please enter a date at least an hour out."
        };
    }
}
```

---

## <a name="test-the-bot-locally"></a>ボットをローカルでテストする
1. ご自身のマシンを使ってローカルでサンプルを実行します。 手順については、README ファイルで [C#](https://aka.ms/cs-primitive-prompt-sample) または [JS](https://aka.ms/js-primitive-prompt-sample) サンプルを参照してください。
1. 次に示すように、エミュレーターを使用してテストします。

![primitive-prompts](media/primitive-prompts.png)

## <a name="additional-resources"></a>その他のリソース

[ダイアログ ライブラリ](bot-builder-concept-dialog.md)には、会話の管理に関するさまざまな側面を自動化するクラスが用意されています。 

## <a name="next-step"></a>次のステップ

> [!div class="nextstepaction"]
> [連続して行われる会話フローの実装](bot-builder-dialog-manage-conversation-flow.md)
