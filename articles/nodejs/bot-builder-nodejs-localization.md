---
title: ローカライズをサポートする | Microsoft Docs
description: Bot Framework SDK for Node.js を使用して、ユーザーの在住地を判別し、ローカライズ機能を有効にする方法について説明します。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: ce1b3f073c932cd4042b91ae9afc1e332a7443f2
ms.sourcegitcommit: 6a83b2c8ab2902121e8ee9531a7aa2d85b827396
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/12/2019
ms.locfileid: "67404902"
---
# <a name="support-localization"></a>ローカライズをサポートする

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Bot Builder には、複数の言語でユーザーとコミュニケ―ションできるボットを構築するための豊富なローカライズ システムが含まれています。 ボットのすべてのプロンプトは、ボットのディレクトリ構造内に格納されている JSON ファイルを使用してローカライズできます。 LUIS などのシステムを使用して自然言語処理を実行する場合は、ボットがサポートする言語ごとに個別のモデルが提供されるように [LuisRecognizer][LUISRecognizer] を構成できます。ユーザーの優先ロケールと一致するモデルは、SDK によって自動的に選択されます。

## <a name="determine-the-locale-by-prompting-the-user"></a>ユーザーに入力を求めることでロケールを決定する
ボットをローカライズする最初の手順は、ユーザーの優先言語を識別する機能を追加することです。 SDK には、この設定をユーザーごとに保存して取得するための [session.preferredLocale()][preferredLocal] メソッドが用意されています。 次の例は、優先言語の指定をユーザーに求め、その選択を保存するダイアログです。

``` javascript
bot.dialog('/localePicker', [
    function (session) {
        // Prompt the user to select their preferred locale
        builder.Prompts.choice(session, "What's your preferred language?", 'English|Español|Italiano');
    },
    function (session, results) {
        // Update preferred locale
        var locale;
        switch (results.response.entity) {
            case 'English':
                locale = 'en';
                break;
            case 'Español':
                locale = 'es';
                break;
            case 'Italiano':
                locale = 'it';
                break;
        }
        session.preferredLocale(locale, function (err) {
            if (!err) {
                // Locale files loaded
                session.endDialog(`Your preferred language is now ${results.response.entity}`);
            } else {
                // Problem loading the selected locale
                session.error(err);
            }
        });
    }
]);
```

## <a name="determine-the-locale-by-using-analytics"></a>分析を使用してロケールを決定する
ユーザーのロケールを決定する別の方法は、[Text Analytics API](/azure/cognitive-services/cognitive-services-text-analytics-quick-start) などのサービスを使用して、ユーザーが送信したメッセージのテキストに基づいてユーザーの言語を自動的に検出することです。

次のコード スニペットは、このサービスをボットに組み込む方法を示しています。
``` javascript
var request = require('request');

bot.use({
    receive: function (event, next) {
        if (event.text && !event.textLocale) {
            var options = {
                method: 'POST',
                url: 'https://westus.api.cognitive.microsoft.com/text/analytics/v2.0/languages?numberOfLanguagesToDetect=1',
                body: { documents: [{ id: 'message', text: event.text }]},
                json: true,
                headers: {
                    'Ocp-Apim-Subscription-Key': '<YOUR API KEY>'
                }
            };
            request(options, function (error, response, body) {
                if (!error && body) {
                    if (body.documents && body.documents.length > 0) {
                        var languages = body.documents[0].detectedLanguages;
                        if (languages && languages.length > 0) {
                            event.textLocale = languages[0].iso6391Name;
                        }
                    }
                }
                next();
            });
        } else {
            next();
        }
    }
});
```

上記のコード スニペットをボットに追加したら、[session.preferredLocale()][preferredLocal] を呼び出すと、検出された言語が自動的に返されます。 `preferredLocale()` の検索順序は次のとおりです。
1. `session.preferredLocale()` の呼び出しによって保存されたロケール。 この値は、`session.userData['BotBuilder.Data.PreferredLocale']` に格納されます。
2. `session.message.textLocale` に割り当てられている、検出されたロケール。
3. ボットの構成済みの既定のロケール (例:英語 (‘en’))。

ボットの既定のロケールは、コンストラクターを使用して構成できます。

```javascript
var bot = new builder.UniversalBot(connector, {
    localizerSettings: { 
        defaultLocale: "es" 
    }
});
```

## <a name="localize-prompts"></a>プロンプトをローカライズする
Bot Framework SDK の既定のローカライズ システムはファイル ベースであり、ボットは、ディスクに格納される JSON ファイルを使用して複数の言語をサポートできます。 既定では、ローカライズ システムは、ボットのプロンプトを **./locale/<IETF TAG>/index.json** ファイルで検索します。このファイルでは、プロンプトを検索するための優先ロケールを表す有効な [IETF 言語タグ][IEFT]は <IETF TAG> です。 

以下のスクリーンショットでは、次の 3 つの言語をサポートするボットのディレクトリ構造を示します:英語、イタリア語、スペイン語。

![3 つのロケール用のディレクトリ構造](../media/locale-dir.png)

ファイルの構造は、メッセージ ID とローカライズされたテキスト文字列の単純な JSON マップです。 値が文字列ではなく配列の場合は、[session.localizer.gettext()][GetText] を使用して値を取得すると、配列から 1 つのプロンプトがランダムに選択されます。 

[session.send()](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#send) の呼び出しで、言語固有のテキストではなくメッセージ ID が渡された場合、ボットは、そのメッセージのローカライズ版を自動的に取得します。

```javascript
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("greeting");
        session.send("instructions");
        session.beginDialog('/localePicker');
    },
    function (session) {
        builder.Prompts.text(session, "text_prompt");
    }
]);
```

内部的には、SDK は、[`session.preferredLocale()`][preferredLocale] を呼び出してユーザーの優先ロケールを取得した後、それを [`session.localizer.gettext()`][GetText] の呼び出しで使用して、メッセージ ID をローカライズされたテキスト文字列にマップします。  場合によっては、ローカライザーを手動で呼び出す必要があります。 たとえば、[`Prompts.choice()`][promptsChoice] に渡された列挙値は、自動的にローカライズされることはないため、プロンプトを呼び出す前にローカライズされた一覧を手動で取得する必要があります。

```javascript
var options = session.localizer.gettext(session.preferredLocale(), "choice_options");
builder.Prompts.choice(session, "choice_prompt", options);
```

既定のローカライザーは、複数のファイルでメッセージ ID を検索し、ID が見つからない (またはローカリゼーション ファイルが提供されていない) 場合は、単純に ID のテキストを返すことで、ローカライズ ファイルの使用を透過的であり省略可能なものにします。  ファイルは、次の順序で検索されます。

1. [`session.preferredLocale()`][preferredLocale] によって返されるロケールの **index.json** ファイルが検索されます。
2. **en-US** などの省略可能なサブタグがロケールに含まれている場合は、ルート タグ **en** が検索されます。
3. ボットの構成済みの既定のロケールが検索されます。

## <a name="use-namespaces-to-customize-and-localize-prompts"></a>名前空間を使用してプロンプトをカスタマイズしてローカライズする
既定のローカライザーは、メッセージ ID 間の競合を回避するために、プロンプトの名前空間の使用をサポートします。  ボットは、名前空間内のプロンプトを上書きしてカスタマイズしたり、別の名前空間内のプロンプトに言い換えたりすることができます。  この機能を活用して SDK の組み込みメッセージをカスタマイズすることで、別の言語に対するサポートの追加や、単に SDK の現在のメッセージ テキストの書き換えを実行できます。  たとえば、**BotBuilder.json** という名前のファイルをボットのロケール ディレクトリに追加し、`default_error` メッセージ ID 用のエントリを追加するだけで、SDK の既定のエラー メッセージを変更できます

![ロケール名前空間用の BotBuilder.json](../media/locale-namespacing.png)


## <a name="additional-resources"></a>その他のリソース

認識エンジンをローカライズする方法については、[意図の認識](bot-builder-nodejs-recognize-intent-messages.md)に関する記事を参照してください。


[LUIS]: https://www.luis.ai/
[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[IntentRecognizerSetOptions]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html
[LUISRecognizer]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer
[LUISSample]: https://aka.ms/v3-js-luisSample
[DisambiguationSample]: https://aka.ms/v3-js-onDisambiguateRoute
[preferredLocal]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#preferredlocale
[preferredLocale]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#preferredlocale
[promptsChoice]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#choice
[GetText]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ilocalizer.html#gettext
[IEFT]: https://en.wikipedia.org/wiki/IETF_language_tag

