---
title: メッセージの内容から意図を認識する - Bot Service
description: 正規表現を使用するか、メッセージの内容をチェックすることによってユーザーの意図を認識する方法について説明します。
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: cab45fa2eca7fad94ce8a66e2a17495a529935c3
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75790685"
---
# <a name="recognize-user-intent-from-message-content"></a>メッセージの内容からユーザーの意図を認識する

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

ボットがユーザーからメッセージを受信すると、ボットは**認識エンジン**を使用してメッセージを検証し、意図を判断することができます。 意図は、メッセージから、呼び出すダイアログへのマッピングを提供します。 この記事では、正規表現を使用して、またはメッセージの内容を検査することによって意図を認識する方法について説明します。 たとえば、ボットは正規表現を使用して、メッセージに "help" という単語が含まれているかどうかチェックし、ヘルプ ダイアログを呼び出すことができます。 ボットはユーザー メッセージのプロパティをチェックすることもできます。たとえば、ユーザーがテキストではなく画像を送信したかどうか確認し、画像処理ダイアログを呼び出すことができます。

> [!NOTE]
> LUIS を使用した意図の認識については、「[LUIS を使用して意図とエンティティを認識する](bot-builder-nodejs-recognize-intent-luis.md)」を参照してください。

## <a name="use-the-built-in-regular-expression-recognizer"></a>組み込みの正規表現認識エンジンの使用

正規表現を使用してユーザーの意図を検出するには、[RegExpRecognizer][RegExpRecognizer] を使用します。 複数の言語をサポートするために、複数の式を認識エンジンに渡すことができます。

> [!TIP]
> ボットのローカライズの詳細については、「[ローカライズのサポート](bot-builder-nodejs-localization.md)」を参照してください。

次のコードは、`CancelIntent` という名前の正規表現認識エンジンを作成してボットに追加します。 この例の認識エンジンは、`en_us` ロケールと `ja_jp` ロケールの両方に正規表現を提供します。

[!code-js[Add a regular expression recognizer (JavaScript)](../includes/code/node-regex-recognizer.js#addRegexRecognizer)]

認識エンジンがボットに追加されたら、認識エンジンが意図を検出したときにボットで呼び出したいダイアログに [triggerAction][triggerAction] をアタッチします。 次のコードに示すように、`matches` オプションを使用して意図名を指定します。

[!code-js[Map the CancelIntent recognizer to a cancel dialog (JavaScript)](../includes/code/node-regex-recognizer.js#bindCancelDialogToRegexRecognizer)]

意図認識エンジンはグローバルです。つまり、ユーザーから受信したすべてのメッセージに対して認識エンジンが実行されます。 `triggerAction` を使用してダイアログにバインドされた意図を認識エンジンが検出した場合、認識エンジンは、現在アクティブなダイアログの中断をトリガーすることができます。 中断を許可して処理することは、ユーザーが実際に行うことを考慮に入れた柔軟な設計です。

> [!TIP]
> `triggerAction` とダイアログの連動のしくみについては、「[Managing conversation flow](bot-builder-nodejs-manage-conversation-flow.md)」(会話フローの管理) を参照してください。 認識された意図に関連付けることができる各種アクションの詳細については、「[Handle user actions](bot-builder-nodejs-dialog-actions.md)」(ユーザー アクションの処理) を参照してください。

## <a name="register-a-custom-intent-recognizer"></a>カスタム意図認識エンジンの登録

カスタム認識エンジンを実装することもできます。 この例では、ユーザーが「help」や「goodbye」と言うのを認識する単純な認識エンジンを追加しますが、ユーザーが画像を送信したことを認識するものなど、より複雑な処理を行う認識エンジンを容易に追加できます。

[!code-js[Add a custom recognizer (JavaScript)](../includes/code/node-howto-recognize-intent.js#addCustomRecognizer)]

認識エンジンを登録したら、`matches` 句を使用して認識エンジンをアクションに関連付けることができます。

[!code-js[Bind intents to actions (JavaScript)](../includes/code/node-howto-recognize-intent.js#bindIntentsToActions)]

## <a name="disambiguate-between-multiple-intents"></a>複数の意図間のあいまいさ排除

ボットは複数の認識エンジンを登録できます。 カスタム認識エンジンの例では、各意図に数値のスコアを割り当てる必要があります。 これが行われるのは、ボットに複数の認識エンジンが存在する場合があり、複数の認識エンジンによって返された意図間のあいまいさを排除する組み込みのロジックを Bot Framework SDK が提供するからです。 意図に割り当てられるスコアは通常は 0.0 から 1.0 ですが、特定の意図が常に Bot Framework SDK のあいまいさ排除ロジックによって選ばれることを保証するために、1.1 より大きい意図をカスタム認識エンジンで定義できます。

既定では、認識エンジンは並列実行されますが、[IIntentRecognizerSetOptions][IIntentRecognizerSetOptions] で recognizeOrder を設定し、1.0 のスコアを与える意図をボットが検出したらすぐにプロセスを終了させることができます。

Bot Framework SDK には、[IDisambiguateRouteHandler][IDisambiguateRouteHandler] を実装することによってカスタムのあいまいさ排除ロジックをボットで提供する方法を示す[サンプル][DisambiguationSample]が含まれています。

## <a name="next-steps"></a>次のステップ

正規表現を使用してメッセージの内容を検査するロジックは、ボットの会話フローが拡張可能である場合は特に、複雑になる可能性があります。 ユーザーからのより多様なテキストおよび音声入力をボットが処理できるよう、[LUIS][LUIS] のような意図認識サービスを使用して、自然言語の理解をボットに追加することができます。

> [!div class="nextstepaction"]
> [LUIS を使用して意図とエンティティを認識する](bot-builder-nodejs-recognize-intent-luis.md)

[LUIS]: https://www.luis.ai/

[IDisambiguateRouteHandler]:   https://docs.microsoft.com/javascript/api/botbuilder/idisambiguateroutehandler?view=botbuilder-ts-3.0
[IIntentRecognizerSetOptions]: https://docs.microsoft.com/javascript/api/botbuilder/iintentrecognizersetoptions?view=botbuilder-ts-3.0
[RegExpRecognizer]:            https://docs.microsoft.com/javascript/api/botbuilder/regexprecognizer?view=botbuilder-ts-3.0
[triggerAction]:               https://docs.microsoft.com/javascript/api/botbuilder/dialog?view=botbuilder-ts-3.0#triggeraction-itriggeractionoptions-

[DisambiguationSample]: https://aka.ms/v3-js-onDisambiguateRoute
