---
title: ユーザー入力を翻訳してボットを多言語対応にする | Microsoft Docs
description: ユーザー入力をボットのネイティブ言語に自動的に翻訳し、ユーザーの言語に翻訳し直す方法。
keywords: 翻訳, 翻訳する, 多言語, microsoft translator
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/06/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 13139755989afccd85b2e09267dc42619ec1f83c
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301516"
---
# <a name="translate-user-input-to-make-your-bot-multilingual"></a>ユーザー入力を翻訳してボットを多言語対応にする

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ボットでは、[Microsoft Translator](https://www.microsoft.com/en-us/translator/) を使用して、メッセージをボットが理解できる言語に自動的に翻訳できます。また、必要に応じて、ボットの応答をユーザーの言語に翻訳し直すこともできます。 ボットに翻訳機能を追加することで、ボットのコア プログラミングの重要な部分を変更せずに、より多くのユーザーを獲得できます。
<!-- 
- [Get a Text Services key](#get-a-text-services-key)
- [Installing Packages](#installing-packages)
- [Combine translation with LUIS or QnA Maker](#using-luis)
- [Combine translation with QnA Maker](#using-qna-maker)
-->

このチュートリアルでは、翻訳に Microsoft Translator サービスを使用し、簡単なボットに翻訳機能を追加してその動作を確認します。

## <a name="get-a-text-services-key"></a>テキスト サービス キーを取得する

まず、Translator Text サービスを使用するためのキーが必要です。 Azure portal で[無料試用版キー](https://www.microsoft.com/en-us/translator/trial.aspx#get-started)を取得できます。

## <a name="installing-packages"></a>パッケージをインストールする

ボットに翻訳機能を追加するために必要なパッケージを用意する必要があります。

# <a name="ctabcs"></a>[C#](#tab/cs)

次の NuGet パッケージのプレリリース バージョンへの[参照を追加](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui)します。

* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Builder.Ai.Translation` (翻訳に必要)

翻訳を Language Understanding (LUIS) と組み合わせる場合は、次のパッケージへの参照も追加します。

* `Microsoft.Bot.Builder.Ai.Luis` (LUIS に必要)

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

botbuilder-ai パッケージを使用して、これらのサービスのいずれかをボットに追加できます。 このパッケージは、npm を使用してプロジェクトに追加できます。
* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---

## <a name="configure-translation"></a>翻訳を構成する

次に、ボットのミドルウェア スタックにトランスレーターを追加して、ユーザーからメッセージを受信するたびにトランスレーターを呼び出すようにボットを構成できます。 ミドルウェアでは翻訳結果を使用し、context オブジェクトを使ってユーザーのメッセージを変更します。


# <a name="ctabcs"></a>[C#](#tab/cs)

SDK V4 の EchoBot サンプルから始め、`Startup.cs` ファイル内の `ConfigureServices` メソッドを更新して、`TranslationMiddleware` をボットに追加します。 これにより、ユーザーから受信したすべてのメッセージを翻訳するようにボットが構成されます。 <!--, by simply adding it to your bot's middleware set. The middleware stores the translation results on the context object. -->
-   using ステートメントを更新します。
-   `ConfigureServices` メソッドを更新して翻訳ミドルウェアを含めます。

    次のスニペットは、ほとんどのコメントを削除し、例外処理ミドルウェアも削除することによって簡略化されています。

**Startup.cs**
```csharp
using System;
using System.Collections.Generic;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.Ai.Translation;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
```
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        var middleware = options.Middleware;
        // Add translation middleware
        // The first parameter is a list of languages the bot recognizes
        middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR TEXT API KEY>", false));

    });
}


```

> [!TIP] 
> Bot Builder SDK では、ユーザーから送信されたメッセージに基づいてユーザーの言語を自動的に検出します。 この機能をオーバーライドするには、追加のコールバック パラメーターを指定して、ユーザーの言語を検出および変更するための独自のロジックを追加します。  



`EchoBot.cs` 内のコードを見てみましょう。このコードでは、"You sent" の後にユーザーの発言を追加して送信します。

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using System.Threading.Tasks;

namespace Microsoft.Bot.Samples
{
    public class EchoBot : IBot
    {
        public EchoBot() { }

        public async Task OnTurn(ITurnContext context)
        {
            switch (context.Activity.Type)
            {
                case ActivityTypes.Message:
                // echo back the user's input.
                    await context.SendActivity($"You sent '{context.Activity.Text}'");


                case ActivityTypes.ConversationUpdate:
                    foreach (var newMember in context.Activity.MembersAdded)
                    {
                        if (newMember.Id != context.Activity.Recipient.Id)
                        {
                            await context.SendActivity("Hello and welcome to the echo bot.");
                        }
                    }
                    break;
            }
        }
        
    }
}
```

翻訳ミドルウェアを追加するときに、省略可能なパラメーターによって、応答をユーザーの言語に翻訳し直すかどうかを指定します。 `Startup.cs` では、`false` が指定されているので、ユーザー メッセージがボットの言語に翻訳されるだけです。

```cs
// The first parameter is a list of languages the bot recognizes
// The second parameter is your API key
// The third parameter specifies whether to translate the bot's replies back to the user's language
middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR TEXT API KEY>", false));
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

エコー ボットを使用して翻訳ミドルウェアを設定するには、app.js に次のコードを貼り付けます。

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const { LanguageTranslator, LocaleConverter } = require('botbuilder-ai');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'] 
});
adapter.use(languageTranslator);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            await context.sendActivity(`You said "${context.activity.text}"`);
        } else {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

---

## <a name="run-the-bot-and-see-translated-input"></a>ボットを実行して翻訳された入力を確認する

ボットを実行し、他の言語でいくつかのメッセージを入力します。 ボットによってユーザー メッセージが翻訳され、応答で翻訳が示されていることがわかります。

![ボットによって言語が検出され、入力が翻訳される](./media/how-to-bot-translate/bot-detects-language-translates-input.png)




## <a name="invoke-logic-in-the-bots-native-language"></a>ボットのネイティブ言語でロジックを呼び出す

次に、英単語をチェックするロジックを追加します。 ユーザーが別の言語で "ヘルプ" または "キャンセル" と言った場合、ボットはそれを英語に翻訳し、英単語の "help" または "cancel" をチェックするロジックが呼び出されます。

# <a name="ctabcs"></a>[C#](#tab/cs)
`EchoBot.cs` で、ボットの `OnTurn` メソッドのメッセージ アクティビティの `case` ステートメントを更新します。
```cs
case ActivityTypes.Message:
    // check the message text before calling context.SendActivity
    var messagetext = context.Activity.Text.Trim().ToLower();
    if (string.Equals("help", messagetext))
    {
        await context.SendActivity("You asked for help.");
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
if (context.activity.type === 'message') {
    // check the message text before calling context.sendActivity
    var messagetext = context.activity.text.trim().toLowerCase();
    if(messagetext === "help"){
        await context.sendActivity("you said help");
    }
}
```

---

![ボットによってフランス語のヘルプが検出される](./media/how-to-bot-translate/bot-detects-help-french.png)



## <a name="translate-replies-back-to-the-users-language"></a>応答をユーザーの言語に翻訳し直す

最後のコンストラクター パラメーターを `true` に設定することで、応答をユーザーの言語に翻訳し直すこともできます。

# <a name="ctabcs"></a>[C#](#tab/cs)
`Startup.cs` で、`ConfigureServices` メソッドの次の行を更新します。
```cs
// Use language recognition to detect the user's language from their message, instead of providing helper callbacks.
// Last parameter indicates that we'll translate replies back to the user's language
middleware.Add(new TranslationMiddleware(new string[] { "en" }, "TRANSLATION-SUBSCRIPTION-KEY", true));
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
// Use language recognition to detect the user's language from their message, instead of providing helper callbacks.
// Last parameter indicates that we'll translate replies back to the user's language
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: true
});
adapter.use(languageTranslator);
```

---

## <a name="run-the-bot-to-see-replies-in-the-users-language"></a>ボットを実行してユーザーの言語での応答を確認する

ボットを実行し、他の言語でいくつかのメッセージを入力します。 ユーザーの言語が検出され、応答が翻訳されていることがわかります。

![ボットによって言語が検出され、応答が翻訳される](./media/how-to-bot-translate/bot-detects-language-translates-response.png)


## <a name="adding-logic-for-detecting-or-changing-the-user-language"></a>ユーザーの言語を検出または変更するためのロジックを追加する

Bot Bilder SDK によってユーザーの言語が自動的に検出されるようにするのではなく、コールバックを提供して、ユーザーの言語を特定したり、ユーザーの言語が変更されたときに判断したりする独自のロジックを追加できます。


次の例では、`CheckUserChangedLanguage` コールバックによって、特定のユーザー メッセージがチェックされ、言語が変更されます。 

# <a name="ctabcs"></a>[C#](#tab/cs)
`Startup.cs` で、翻訳ミドルウェアにコールバックを追加します。
```csharp
middleware.Add(new TranslationMiddleware(new string[] { "en" },
     "<YOUR MICROSOFT TRANSLATOR TEXT API KEY>", null,
     TranslatorLocaleHelper.GetActiveLanguage,
     TranslatorLocaleHelper.CheckUserChangedLanguage));
```
`TranslatorLocalHelper.cs` ファイルを追加し、TranslatorLocalHelper クラス定義に次のコードを追加します。
```csharp
    class CurrentUserState
    {
        public string Language { get; set; }
    }

    public static void SetLanguage(ITurnContext context, string language) =>
        context.GetConversationState<CurrentUserState>().Language = language;

    public static bool IsSupportedLanguage(string language) =>
        _supportedLanguages.Contains(language);

    public static async Task<bool> CheckUserChangedLanguage(ITurnContext context)
    {
        bool changeLang = false;
        
        // use a specific message from user to change language
        if (context.Activity.Type == ActivityTypes.Message)
        {
            var messageActivity = context.Activity.AsMessageActivity();
            if (messageActivity.Text.ToLower().StartsWith("set my language to"))
            {
                changeLang = true;
            }
            if (changeLang)
            {
                var newLang = messageActivity.Text.ToLower().Replace("set my language to", "").Trim();
                if (!string.IsNullOrWhiteSpace(newLang)
                        && IsSupportedLanguage(newLang))
                {
                    SetLanguage(context, newLang);
                    await context.SendActivity($@"Changing your language to {newLang}");
                }
                else
                {
                    await context.SendActivity($@"{newLang} is not a supported language.");
                }
                // return true to intercept message from further processesing
                return true;
            }
        }

        return false;
    }

    public static string GetActiveLanguage(ITurnContext context)
    {

        if (context.Activity.Type == ActivityTypes.Message
            && context.GetConversationState<CurrentUserState>() != null
            && context.GetConversationState<CurrentUserState>().Language != null)
        {
            return context.GetConversationState<CurrentUserState>().Language;
        }

        return "en";
    }

```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
// When the user inputs 'set my language to fr'
// The bot will automatically change all text to French
async function setUserLanguage(context) {
    let state = conversationState.get(context)
    if (context.activity.text.toLowerCase().startsWith('set my language to')) {
        state.language = context.activity.text.toLowerCase().replace('set my language to', '').trim();
        await context.sendActivity(`Setting your language to ${state.language}`);
        return Promise.resolve(true);
    } else {
        return Promise.resolve(false);
    }
}

// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "<YOUR MICROSOFT TRANSLATOR TEXT API KEY>",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    setUserLanguage: setUserLanguage,
    translateBackToUserLanguage: true
});
adapter.use(languageTranslator);

// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            await context.sendActivity(`You said: ${context.activity.text}`)
        }
    });
});

```

---

## <a name="combining-luis-or-qna-with-translation"></a>LUIS または QnA を翻訳と組み合わせる

ボットで LUIS や QnA Maker などの他のサービスと翻訳を組み合わせる場合は、まず翻訳ミドルウェアを追加します。これにより、ボットのネイティブ言語を必要とする他のミドルウェアに渡す前にメッセージが翻訳されます。

# <a name="ctabcs"></a>[C#](#tab/cs)
```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<AiBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        var middleware = options.Middleware;

        // Add translation middleware
        // The first parameter is a list of languages the bot recognizes. Input in these language won't be translated.
        middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR TEXT API KEY>", false));

        // Add LUIS middleware after translation middleware to pass translated messages to LUIS
        middleware.Add(
            new LuisRecognizerMiddleware(
                new LuisModel("luisAppId", "subscriptionId", new Uri("luisModelBaseUrl"))));
    });
}
```

ボット コードでは、LUIS の結果は、ボットのネイティブ言語に既に翻訳されている入力に基づいています。 LUIS アプリの結果を確認するために、ボット コードを次のように変更してみてください。

```cs
public async Task OnTurn(ITurnContext context)
{
    switch (context.Activity.Type)
    {
        case ActivityTypes.Message:
            // check LUIS results
            var luisResult = context.Services.Get<RecognizerResult>(LuisRecognizerMiddleware.LuisRecognizerResultKey);
            (string key, double score) topItem = luisResult.GetTopScoringIntent();
            await context.SendActivity($"The top intent was: '{topItem.key}', with score {topItem.score}");

            await context.SendActivity($"You sent '{context.Activity.Text}'");

        case ActivityTypes.ConversationUpdate:
            foreach (var newMember in context.Activity.MembersAdded)
            {
                if (newMember.Id != context.Activity.Recipient.Id)
                {
                    await context.SendActivity("Hello and welcome to the echo bot.");
                }
            }
            break;
        }
    }  
}          
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: false
});
adapter.use(languageTranslator);

// Add Luis recognizer middleware
const luisRecognizer = new LuisRecognizer({
    appId: "xxxxxx",
    subscriptionKey: "xxxxxxx"
});
adapter.use(luisRecognizer);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            let results = luisRecognizer.get(context);
            for (const intent in results.intents) {
                await context.sendActivity(`intent: ${intent.toString()} : ${results.intents[intent.toString()]}`)
            }
        }
    });
});
```

---

## <a name="bypass-translation-for-specified-patterns"></a>指定したパターンの翻訳を省略する
固有名詞など、ボットで翻訳する必要のない特定の単語が存在する場合があります。 翻訳する必要のないパターンを示す正規表現を指定できます。 たとえば、ユーザーがボットのネイティブ言語以外の言語で「私の名前は...です」と言ったときに、名前が翻訳されないようにする場合は、パターンを使用してこれを指定できます。

# <a name="ctabcs"></a>[C#](#tab/cs)
Startup.cs
```cs
// Pattern representing input to not translate
Dictionary<string, List<string>> patterns = new Dictionary<string, List<string>>();
// single pattern for fr language, to avoid translating what follows "my name is"
patterns.Add("fr", new List<string> { "mon nom est (.+)" });

middleware.Add(new TranslationMiddleware(new string[] { "en" },
    "<YOUR API KEY>", patterns,
    TranslatorLocaleHelper.GetActiveLanguage,
    TranslatorLocaleHelper.CheckUserChangedLanguage));

```
<!-- TODO: ADD more explanation (both of these callbacks are run every time), fix image by debugging regex for l'etat -->

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "<YOUR API KEY>",
    // single pattern for fr language, to avoid translating what follows "my name is"
    noTranslatePatterns: new Set("fr", "/mon nom est (.+)/"),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: false,
    
});
adapter.use(languageTranslator);

```

---

![ボットでパターンの翻訳を省略する](./media/how-to-bot-translate/bot-no-translate-name-fr.png)

## <a name="localize-dates"></a>日付をローカライズする

日付をローカライズする必要がある場合は、`LocaleConverterMiddleware` を追加します。 たとえば、ボットで `MM/DD/YYYY` 形式の日付が想定されており、他のロケールのユーザーが `DD/MM/YYYY` 形式で日付を入力する可能性があることがわかっている場合は、ロケール コンバーター ミドルウェアによって、日付をボットで想定されている形式に自動的に変換できます。

> [!NOTE]
> ロケール コンバーター ミドルウェアは、日付のみを変換することを目的としています。 翻訳ミドルウェアの結果は認識されません。 翻訳ミドルウェアを使用している場合は、ロケール コンバーターとどのように組み合わせるかに注意してください。 翻訳ミドルウェアでは、テキスト形式の一部の日付は他のテキスト入力と共に翻訳されますが、日付は翻訳されません。

たとえば、次の画像は、英語からフランス語への翻訳後に、ユーザー入力をエコー バックするボットを示しています。 このボットでは、`LocaleConverterMiddleware` を使用せずに `TranslationMiddleware` を使用しています。

![日付を変換せずに日付を翻訳するボット](./media/how-to-bot-translate/locale-date-before.png)

次の画像は、`LocaleConverterMiddleware` が追加された場合の同じボットを示しています。

![日付を変換せずに日付を翻訳するボット](./media/how-to-bot-translate/locale-date-after.png)

ロケール コンバーターは、英語、フランス語、ドイツ語、中国語のロケールをサポートできます。 <!-- TODO: ADD DETAIL ABOUT SUPPORTED LOCALES -->

# <a name="ctabcs"></a>[C#](#tab/cs)
Startup.cs
```cs
// Add locale converter middleware
middleware.Add(new LocaleConverterMiddleware(
    TranslatorLocaleHelper.GetActiveLocale,
    TranslatorLocaleHelper.CheckUserChangedLocale,
     "en-us", LocaleConverter.Converter));
```
TranslatorLocaleHelper.cs
```cs
// TranslatorLocaleHelper.cs
public static async Task<bool> CheckUserChangedLocale(ITurnContext context)
{
    bool changeLocale = false;//logic implemented by developper to make a signal for language changing 
    //use a specific message from user to change language
    if (context.Activity.Type == ActivityTypes.Message)
    {
        var messageActivity = context.Activity.AsMessageActivity();
        if (messageActivity.Text.ToLower().StartsWith("set my locale to"))
        {
            changeLocale = true;
        }
        if (changeLocale)
        {
            var newLocale = messageActivity.Text.ToLower().Replace("set my locale to", "").Trim(); //extracted by the user using user state 
            if (!string.IsNullOrWhiteSpace(newLocale)
                    && IsSupportedLanguage(newLocale))
            {
                SetLocale(context, newLocale);
                await context.SendActivity($@"Changing your language to {newLocale}");
            }
            else
            {
                await context.SendActivity($@"{newLocale} is not a supported locale.");
            }
            //intercepts message
            return true;
        }
    }

    return false;
}
public static string GetActiveLocale(ITurnContext context)
{
    if (currentLocale != null)
    {
        //the user has specified a different locale so update the bot state
        if (context.GetConversationState<CurrentUserState>() != null
            && currentLocale != context.GetConversationState<CurrentUserState>().Locale)
        {
            SetLocale(context, currentLocale);
        }
    }
    if (context.Activity.Type == ActivityTypes.Message
        && context.GetConversationState<CurrentUserState>() != null && context.GetConversationState<CurrentUserState>().Locale != null)
    {
        return context.GetConversationState<CurrentUserState>().Locale;
    }

    return "en-us";
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

<!-- this snippet only works if the user doesn't actually try to change their locale.  Emailed Mostafa about the issue 
It should change the locale after you type in 'set my locale to....' -->

```javascript
// Delegates for getting and setting user locale
function getUserLocale(context) {
    const state = conversationState.get(context)
    if (state.locale == undefined) {
        return 'en-us';
    } else {
        return state.locale;
    }
}

// Function to change the user's locale.
async function setUserLocale(context) {
    let state = conversationState.get(context)
    if (context.activity.text.toLowerCase().startsWith('set my locale to')) {        
        state.locale = context.activity.text.toLowerCase().replace('set my locale to', '').trim();
        await context.sendActivity(`Setting your locale to ${state.locale}`);
        return Promise.resolve(true);
    } else {
        return Promise.resolve(false);
    }
}

// Add locale converter middleware
// Will convert input to fr-fr
const localeConverter = new LocaleConverter({
    toLocale: 'fr-fr',
    setUserLocale: setUserLocale,
    getUserLocale: getUserLocale
});
adapter.use(localeConverter);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            await context.sendActivity(`The message that you sent was:  ${context.activity.text}`)
        }
    });
});

```

---
