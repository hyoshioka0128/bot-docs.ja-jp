---
title: Enterprise Bot のカスタマイズ | Microsoft Docs
description: Enterprise Template で作成したボットをカスタマイズする方法について説明します
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/7/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d472cbe7c0235862f8dcff1bcc2d53d977bb7657
ms.sourcegitcommit: 8183bcb34cecbc17b356eadc425e9d3212547e27
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2019
ms.locfileid: "55971492"
---
# <a name="enterprise-bot-template---customize-your-bot"></a>Enterprise Bot Template - ボットのカスタマイズ

> [!NOTE]
> このトピックは、SDK の v4 バージョンに適用されます。 

## <a name="net"></a>.NET
[こちら](bot-builder-enterprise-template-deployment.md)の説明に従ってデプロイを完了し、Enterprise Bot Template がすべて正常に機能していることを確認したら、シナリオやニーズに応じて、ボットを簡単にカスタマイズできます。 テンプレートの目的は、会話型エクスペリエンスを構築するための強固な基盤を提供することです。

## <a name="project-structure"></a>プロジェクト構造

ボットのフォルダー構造は次のとおりです。これは、ボット プロジェクトを構成し、受信メッセージを処理するための推奨のベスト プラクティスです。

    | - YourBot.bot         // The .bot file containing all of your Bot configuration including dependencies
    | - README.md           // README file containing links to documentation
    | - Program.cs          // Default Program.cs file
    | - Startup.cs          // Core Bot Initialisation including Bot Configuration LUIS, Dispatcher, etc. 
    | - <BOTNAME>State.cs   // The Root State class for your Bot
    | - appsettings.json    // References above .bot file for Configuration information. App Insights key
    | - CognitiveModels     
        | - LUIS            // .LU file containing base conversational intents (Greeting, Help, Cancel)
        | - QnA             // .LU file containing example QnA items
    | - DeploymentScripts   // msbot clone recipes for deployment
        | - de              // Deployment files for German
        | - en              // Deployment files for English        
        | - es              // Deployment files for Spanish
        | - fr              // Deployment files for French
        | - it              // Deployment files for Italian
        | - zh              // Deployment files for Chinese
    | - Dialogs             // All Bot dialogs sit under this folder
        | - Main            // Root Dialog for all messages
            | - MainDialog.cs       // Dialog Logic
            | - MainResponses.cs    // Dialog responses
            | - Resources           // Adaptive Card JSON, Resource File
        | - Onboarding
            | - OnboardingDialog.cs       // Onboarding dialog Logic
            | - OnboardingResponses.cs    // Onboarding dialog responses
            | - OnboardingState.cs        // Localised dialog state
            | - Resources                 Resource File
        | - Cancel
        | - Escalate
        | - Signin
    | - Middleware          // Telemetry, Content Moderator
    | - ServiceClients      // SDK libraries, example GraphClient provided for Auth example
   
## <a name="update-introduction-message"></a>紹介メッセージの更新

紹介メッセージには[アダプティブ カード](https://www.adaptivecards.io)が使用されます。 ボットの紹介メッセージをカスタマイズするには、Dialogs/Main/Resources フォルダー内の JSON ファイル (```Intro.json```) を検索します。 [アダプティブ カード ビジュアライザー](http://adaptivecards.io/visualizer)を使用して、ボットの要件に合うようにアダプティブ カードを変更します。

## <a name="update-bot-responses"></a>ボットの応答の更新

プロジェクト内の各ダイアログには一連の応答セットがあり、それらはサポート リソース (.resx) ファイル内に格納されています。 これらのファイルは、Dialog の下の Resources フォルダー内にあります。

Visual Studio のリソース エディター (下記) で応答を変更すれば、ボットの応答方法を調整することができます。

![ボットの応答のカスタマイズ](media/enterprise-template/EnterpriseBot-CustomisingResponses.png)

このアプローチは、標準のリソース ファイル ローカリゼーション アプローチを使用した多言語応答に対応しています。 詳しくは、[こちら](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/localization?view=aspnetcore-2.1)をご覧ください。

## <a name="updating-your-cognitive-models"></a>コグニティブ モデルの更新

Enterprise Template には、サンプルの FAQ QnA Maker ナレッジ ベースと、一般的な意図 (あいさつ、ヘルプ、キャンセルなど) に対応した LUIS モデルの 2 つのコグニティブ モデルが、既定で用意されています。 これらのモデルは、ニーズに応じてカスタマイズできます。 また、新しい LUIS モデルや QnA Maker ナレッジ ベースを追加して、ボットの機能を拡張することもできます。

### <a name="updating-an-existing-luis-model"></a>既存の LUIS モデルの更新
Enterprise Template 用の既存の LUIS モデルを更新するには、次の手順を実行します。
1. [LUIS ポータル](http://luis.ai)にアクセスするか、CLI ツール [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) と [Luis](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS) を使用して、LUIS モデルに変更を加えます。 
2. 次のコマンドを実行してディスパッチ モデルを更新し、変更を反映します (適切なメッセージ ルーティングを確保します)。
```shell
    dispatch refresh --bot "YOUR_BOT.bot" --secret YOUR_SECRET
```
3. 更新された各モデルについて、プロジェクトのルートから次のコマンドを実行し、関連付けられている LuisGen クラスを更新します。 
```shell
    luis export version --appId [LUIS_APP_ID] --versionId [LUIS_APP_VERSION] --authoringKey [YOUR_LUIS_AUTHORING_KEY] | luisgen --cs [CS_FILE_NAME] -o "\Dialogs\Shared\Resources"
```

### <a name="updating-an-existing-qna-maker-knowledge-base"></a>既存の QnA Maker ナレッジ ベースの更新
既存の QnA Maker ナレッジ ベースを更新するには、以下の手順を実行します。
1. [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) および [QnA Maker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker) CLI ツールまたは [QnA Maker ポータル](https://qnamaker.ai)を使用して、ご自身の QnA Maker ナレッジ ベースに変更を加えます。
2. 次のコマンドを実行してディスパッチ モデルを更新し、変更を反映します (適切なメッセージ ルーティングを確保します)。
```shell
    dispatch refresh --bot "YOUR_BOT.bot" --secret YOUR_SECRET
```

### <a name="adding-a-new-luis-model"></a>新しい LUIS モデルの追加

新しい LUIS モデルをプロジェクトに追加する場合は、ボット構成とディスパッチャーを更新して、追加のモデルを認識させる必要があります。 
1. CLI ツール LuDown/LUIS を使用するか、LUIS ポータルにアクセスして、LUIS モデルを作成します
2. 次のコマンドを実行して、新しい LUIS アプリを .bot ファイルに接続します。
```shell
    msbot connect luis --appId [LUIS_APP_ID] --authoringKey [LUIS_AUTHORING_KEY] --subscriptionKey [LUIS_SUBSCRIPTION_KEY] 
```
3. 次のコマンドを使用して、新しい LUIS モデルをディスパッチャーに追加します
```shell
    dispatch add -t luis -id LUIS_APP_ID -bot "YOUR_BOT.bot" --secret YOURSECRET
```
4. 次のコマンドを使用してディスパッチ モデルを更新し、LUIS モデルの変更を反映します
```shell
    dispatch refresh -bot "YOUR_BOT.bot" --secret YOUR_SECRET
```

### <a name="adding-an-additional-qna-maker-knowledge-base"></a>他の QnA Maker ナレッジ ベースの追加

シナリオによっては、追加の QnA Maker ナレッジベースをお使いのボットに追加したくなることがあります。その場合は、次の手順を実行します。

1. 次のコマンドを使用して、JSON ファイルから新しい QnA Maker ナレッジ ベースを作成します。このコマンドは、お使いのアシスタント ディレクトリで実行します
```shell
qnamaker create kb --in <KB.json> --msbot | msbot connect qna --stdin --bot "YOUR_BOT.bot" --secret YOURSECRET
```
2. 次のコマンドを実行してディスパッチ モデルを更新し、変更を反映します
```shell
dispatch refresh --bot "YOUR_BOT.bot" --secret YOUR_SECRET
```
3. 厳密に型指定されたディスパッチ クラスを更新し、新しい QnA ソースを反映します
```shell
msbot get dispatch --bot "YOUR_BOT.bot" | luis export version --stdin > dispatch.json
luisgen dispatch.json -cs Dispatch -o Dialogs\Shared
```
4.  提供された例に従って、ご自身の新しい QnA ソースに対応するディスパッチ意図が含まれるように、`Dialogs\Main\MainDialog.cs` ファイルを更新します。

お使いのボットの一部として、複数の QnA ソースを活用できるようになりました。

## <a name="adding-a-new-dialog"></a>新しいダイアログの追加

新しいダイアログをボットに追加するには、まず Dialogs の下に新しいフォルダーを作成し、そのクラスが `EnterpriseDialog` から派生していることを確認する必要があります。 その後、ダイアログ インフラストラクチャを接続する必要があります。 オンボード ダイアログは、この操作の簡単な例として参考にできます。次に示すのは、コードの抜粋と手順の概要です。

- ウォーターフォール ダイアログをコンストラクターに追加します
- ウォーターフォールのステップを定義します
- ウォーターフォール ステップを作成します
- AddDialog を呼び出して、ウォーターフォールを渡します
- AddDialog を呼び出して、ウォーターフォールで使用するプロンプトを渡します
- InitialDialogId を、コンポーネントで最初に実行するダイアログに設定します

```
    InitialDialogId = nameof(OnboardingDialog);

    var onboarding = new WaterfallStep[]
    {
        AskForName,
        AskForEmail,
        AskForLocation,
        FinishOnboardingDialog,
    };

    AddDialog(new WaterfallDialog(InitialDialogId, onboarding));
    AddDialog(new TextPrompt(DialogIds.NamePrompt));
    AddDialog(new TextPrompt(DialogIds.EmailPrompt));
    AddDialog(new TextPrompt(DialogIds.LocationPrompt));
```

次に、応答を処理するためのテンプレート マネージャーを作成する必要があります。 新しいクラスを作成し、TemplateManager を継承します。この操作の例は OnboardingResponses.cs ファイルで示されています。次に示すのはその抜粋です。

```    
 ["default"] = new TemplateIdMap
            {
                { ResponseIds.EmailPrompt,
                    (context, data) =>
                    MessageFactory.Text(
                        text: OnboardingStrings.EMAIL_PROMPT,
                        ssml: OnboardingStrings.EMAIL_PROMPT,
                        inputHint: InputHints.ExpectingInput)
                },
                { ResponseIds.HaveEmailMessage,
                    (context, data) =>
                    MessageFactory.Text(
                        text: string.Format(OnboardingStrings.HAVE_EMAIL, data.email),
                        ssml: string.Format(OnboardingStrings.HAVE_EMAIL, data.email),
                        inputHint: InputHints.IgnoringInput)
                },
                { ResponseIds.HaveLocationMessage,
                    (context, data) =>
                    MessageFactory.Text(
                        text: string.Format(OnboardingStrings.HAVE_LOCATION, data.Name, data.Location),
                        ssml: string.Format(OnboardingStrings.HAVE_LOCATION, data.Name, data.Location),
                        inputHint: InputHints.IgnoringInput)
                },
                
                ...
```

応答をレンダリングするには、テンプレート マネージャー インスタンスを使用し、プロンプトの `ReplyWith` または `RenderTemplate` を使用して、それらの応答にアクセスします。 次に示すのは例です。

```
Prompt = await _responder.RenderTemplate(sc.Context, sc.Context.Activity.Locale, OnboardingResponses.ResponseIds.NamePrompt)
await _responder.ReplyWith(sc.Context, OnboardingResponses.ResponseIds.HaveNameMessage, new { name });
```

ダイアログ インフラストラクチャの最後の構成要素として、ダイアログのみをスコープとした State クラスを作成します。 新しいクラスを作成し、`DialogState` から派生していることを確認します

ダイアログが作成できたら、`AddDialog` を使用して、ダイアログを `MainDialog` コンポーネントに追加する必要があります。 新しいダイアログを使用するには、`RouteAsync` メソッド内から `dc.BeginDialogAsync()` を呼び出します。必要な場合は、適切な LUIS インテントを使用してトリガーします。

## <a name="conversational-insights-using-powerbi-dashboard-and-application-insights"></a>PowerBI ダッシュ ボードと Application Insights を使用した会話型インサイト
- 会話型インサイトの取得を開始するには、「[Enterprise Bot Template - PowerBI ダッシュ ボードと Application Insights を使用した会話型分析](bot-builder-enterprise-template-powerbi.md)」から続行してください。
