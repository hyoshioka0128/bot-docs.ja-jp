---
title: ユーザーと会話データを保存する - Bot Service
description: Bot Framework SDK を使用して状態データを保存および取得する方法について説明します。
keywords: 会話状態, ユーザー状態, 会話, 状態の保存, ボットの状態の管理
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d908b807ca356494dcc808f5bacdb15520938eb0
ms.sourcegitcommit: caaf394017dbdb1cfaba32e2d0a1e32c5ab71792
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/10/2020
ms.locfileid: "75798244"
---
# <a name="save-user-and-conversation-data"></a>ユーザーおよび会話データを保存する

[!INCLUDE[applies-to](../includes/applies-to.md)]

ボットは本質的にステートレスです。 いったんご自身のボットがデプロイされると、ターンをまたいで、そのボットを同じプロセスやマシンで実行することはできません。 しかし、お使いのボットでは、会話のコンテキストの追跡が必要になることがあります。これにより、自身の動作を管理し、以前の質問に対する回答を記憶できるようになります。 Bot Framework SDK の状態とストレージの機能を使用すると、状態をお使いのボットに追加できます。 ボットは状態管理およびストレージ オブジェクトを使用して状態を管理維持します。 この状態マネージャーは、基になるストレージの種類には関係なく、プロパティ アクセサーを使用して状態プロパティにアクセスできる抽象化レイヤーを提供します。

## <a name="prerequisites"></a>前提条件

- [ボットの基本](bot-builder-basics.md)、およびボットによる[状態の管理](bot-builder-concept-state.md)方法に関する知識が必要です。
- この記事のコードは、**状態管理ボット** サンプルをベースにしています。 サンプルのコピー ([CSharp](https://aka.ms/statebot-sample-cs)、[JavaScript](https://aka.ms/statebot-sample-js)、または [Python](https://aka.ms/bot-state-python-sample-code)) が必要になります。

## <a name="about-this-sample"></a>このサンプルについて
このサンプルでは、ユーザー入力を受け取ったときに、保存済み会話状態を確認して、ユーザーが以前に名前を入力するように求められたかどうかを確かめます。 求められていない場合は、ユーザーの名前が要求され、入力した内容はユーザー状態に格納されます。 この場合、ユーザー状態に格納されている名前は、ユーザーとの対話に使用され、その入力データは、受信時刻および入力チャネル ID と共にユーザーに返されます。 時刻とチャネル ID の値はユーザーの会話データから取得され、会話状態に保存されます。 次の図は、ボット、ユーザー プロファイル、および会話データ クラスの間の関係を示しています。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

![状態ボットのサンプル](media/StateBotSample-Overview.png)

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

![状態ボットのサンプル](media/StateBotSample-JS-Overview.png)

## <a name="pythontabpython"></a>[Python](#tab/python)

![状態ボットのサンプル](media/StateBotSample-Python-Overview.png)

---

## <a name="define-classes"></a>クラスの定義

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

状態管理を設定するには、まず、ユーザーと会話状態で管理する情報を含むクラスを定義します。 この記事で使用している例では、次のクラスを定義しています。

- **UserProfile.cs** では、ボットによって収集されるユーザー情報を表す `UserProfile` クラスを定義します。 
- **ConversationData.cs** では、ユーザー情報を収集しているときに、会話状態を制御する `ConversationData` クラスを定義します。

次のコード例は、`UserProfile` クラスの定義を示しています。

**UserProfile.cs**  
[!code-csharp[UserProfile](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/UserProfile.cs?range=7-11)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

最初の手順で、`UserState` と `ConversationState` の定義が含まれる botbuilder サービスを要求します。

**index.js**  
[!code-javascript[BotService](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/index.js?range=7-9)]

## <a name="pythontabpython"></a>[Python](#tab/python)

状態管理を設定するには、まず、ユーザーと会話状態で管理する情報を含むクラスを定義します。 この記事で使用している例では、次のクラスを定義しています。

- **user-profile.py** には、ボットによって収集されたユーザー情報を格納する `UserProfile` クラスが含まれています。
- **conversation_data.py** には、ユーザー情報の収集中に会話状態を制御する `ConversationData` クラスが含まれています。

次のコード例は、`ConversationData` クラスの定義を示しています。 

[!code-python[conversation_data](~/../botbuilder-python/samples/python/45.state-management/data_models/conversation_data.py?range=5-14)]

---

## <a name="create-conversation-and-user-state-objects"></a>会話およびユーザー状態オブジェクトの作成

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

次に、`UserState` オブジェクトと `ConversationState` オブジェクトの作成に使用する `MemoryStorage` を登録します。 ユーザーおよび会話状態オブジェクトは `Startup` で作成され、依存関係がボット コンストラクターに挿入されます。 ボット用のサービスとして他に資格情報プロバイダー、アダプター、およびボット実装が登録されています。

**Startup.cs**  
[!code-csharp[ConfigureServices](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/Startup.cs?range=26-29)]
[!code-csharp[ConfigureServices](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/Startup.cs?range=51-57)]

**Bots/StateManagementBot.cs**

[!code-csharp[StateManagement](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/Bots/StateManagementBot.cs?range=15-22)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

次に、`UserState` オブジェクトと `ConversationState` オブジェクトの作成に使用する `MemoryStorage` を登録します。

**index.js**  
[!code-javascript[DefineMemoryStore](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/index.js?range=33-39)]

## <a name="pythontabpython"></a>[Python](#tab/python)

次に、`UserState` オブジェクトと `ConversationState` オブジェクトの作成に使用する `MemoryStorage` を登録します。 ユーザーおよび会話状態のオブジェクトはスタートアップ時に `app.py` に作成され、依存関係がボット コンストラクターに挿入されます。

**app.py** [!code-python[state](~/../botbuilder-python/samples/python/45.state-management/app.py?range=63-69)]

**bots/state_management_bot.py** [!code-python[state](~/../botbuilder-python/samples/python/45.state-management/bots/state_management_bot.py?range=14-30)]

---

## <a name="add-state-property-accessors"></a>状態プロパティ アクセサーの追加

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

ここで、`BotState` オブジェクトにハンドルを提供する `CreateProperty` メソッドを使用して、プロパティ アクセサーを作成します。 各状態プロパティ アクセサーを使用すると、関連付けられている状態プロパティの値を取得または設定できます。 状態プロパティを使用する前に、各アクセサーを使用してストレージからプロパティを読み込んだうえで状態キャッシュから取得します。 状態プロパティに関連付けられている適切に範囲指定されたキーを取得するために、`GetAsync` メソッドを呼び出します。

**Bots/StateManagementBot.cs**

[!code-csharp[StateAccessors](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/Bots/StateManagementBot.cs?range=38-46)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`UserState` および `ConversationState` 用の状態プロパティ アクセサーを作成します。 各状態プロパティ アクセサーを使用すると、関連付けられている状態プロパティの値を取得または設定できます。 各アクセサーを使用して、関連付けられているプロパティをストレージから読み込んで、現在の状態をキャッシュから取得します。

**bots/stateManagementBot.js**

[!code-javascript[BotService](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/bots/stateManagementBot.js?range=6-19)]

## <a name="pythontabpython"></a>[Python](#tab/python)

`UserProfile` および `ConversationData` 用の状態プロパティ アクセサーを作成します。 各状態プロパティ アクセサーを使用すると、関連付けられている状態プロパティの値を取得または設定できます。 各アクセサーを使用して、関連付けられているプロパティをストレージから読み込んで、現在の状態をキャッシュから取得します。

**bots/state_management_bot.py** [!code-python[state_accessors](~/../botbuilder-python/samples/python/45.state-management/bots/state_management_bot.py?range=24-30)]

---

## <a name="access-state-from-your-bot"></a>お使いのボットから状態にアクセスする

前のセクションでは、状態プロパティ アクセサーをボットに追加するための、初期化時の手順について説明しました。 これらのアクセサーを実行時に使用すると、状態情報を読み書きできます。 次のサンプル コードでは次のロジック フローを使用します。


## <a name="ctabcsharp"></a>[C#](#tab/csharp)

- userProfile.Name が空で、conversationData.PromptedUserForName が _true_ の場合は、指定したユーザー名を取得して、それをユーザー状態に格納します。
- userProfile.Name が空で、conversationData.PromptedUserForName が _false_ の場合は、ユーザーの名前を確認します。
- userProfile.Name が前に格納されていた場合は、メッセージ時間とチャネル ID をユーザー入力から取得し、すべてのデータをユーザーにエコー バックして、取得したデータを会話状態内に格納します。

**Bots/StateManagementBot.cs**

[!code-csharp[OnMessageActivityAsync](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=38-85)]

ターン ハンドラーを終了する前に、状態管理オブジェクトの _SaveChangesAsync()_ メソッドを使用して、すべての状態変更をストレージに書き戻します。

**Bots/StateManagementBot.cs**

[!code-csharp[OnTurnAsync](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=24-31)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

- userProfile.Name が空で、conversationData.PromptedUserForName が _true_ の場合は、指定したユーザー名を取得して、それをユーザー状態に格納します。
- userProfile.Name が空で、conversationData.PromptedUserForName が _false_ の場合は、ユーザーの名前を確認します。
- userProfile.Name が前に格納されていた場合は、メッセージ時間とチャネル ID をユーザー入力から取得し、すべてのデータをユーザーにエコー バックして、取得したデータを会話状態内に格納します。

**bots/stateManagementBot.js**

[!code-javascript[OnMessage](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/bots/stateManagementBot.js?range=21-58)]

各ダイアログ ターンを終了する前に、状態管理オブジェクトの _saveChanges()_ メソッドを使用して、状態をストレージに書き込むことですべての変更を保持します。

**bots/stateManagementBot.js**

[!code-javascript[OnDialog](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/bots/stateManagementBot.js?range=60-67)]

## <a name="pythontabpython"></a>[Python](#tab/python)

- `user_profile.name` が空で、`conversation_data.prompted_for_user_name` が *true* の場合、ボットは、ユーザーによって指定された名前を取得し、ユーザーの状態内に格納します。
- `user_profile.name` が空で、`conversation_data.prompted_for_user_name` が *false* の場合、ボットはユーザーの名前を要求します。
- `user_profile.name` が前に格納されていた場合、ボットは、**メッセージ時間**と**チャネル ID** をユーザー入力から取得し、データをユーザーにエコー バックして、取得したデータを会話状態内に格納します。

**bots/state_management_bot.py** [!code-python[state_message_activity](~/../botbuilder-python/samples/python/45.state-management/bots/state_management_bot.py?range=47-89)]

各ダイアログ ターンを終了する前に、ボットは、状態管理オブジェクトの `save_changes` メソッドを使用し、ストレージに状態情報を書き込んですべての変更を保持します。

**bots/state_management_bot.py** [!code-python[state_storage](~/../botbuilder-python/samples/python/45.state-management/bots/state_management_bot.py?range=32-36)]

---

## <a name="test-the-bot"></a>ボットのテスト

最新の [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) をダウンロードしてインストールします

1. ご自身のマシンを使ってローカルでサンプルを実行します。 手順については、README ファイルで [C# サンプル](https://aka.ms/statebot-sample-cs)または [JS サンプル](https://aka.ms/statebot-sample-js)を参照してください。
1. 次に示すように、エミュレーターを使用してボットをテストします。

![状態テストのボットのサンプル](media/state-bot-testing-emulator.png)

## <a name="additional-resources"></a>その他のリソース

**プライバシー:** ユーザーの個人データを格納する場合は、[一般データ保護規則](https://blog.botframework.com/2018/04/23/general-data-protection-regulation-gdpr)に関するコンプライアンスを確保する必要があります。

**状態管理:** すべての状態管理呼び出しが非同期で行われます。これらの呼び出しは既定で Last-Writer-Wins です。 実際の場面では、状態の取得、設定、および保存は、ご自身のボット内で、できるだけ近いタイミングで行う必要があります。

**重要なビジネス データ:** ボットの状態には、詳細設定、ユーザー名、またはユーザーが行った最後の指示を格納しますが、重要なビジネス データを格納するためには使用しないでください。 重要なデータについては、[独自のストレージ コンポーネントを作成](bot-builder-custom-storage.md)するか、[ストレージ](bot-builder-howto-v4-storage.md)に直接書き込んでください。

**Recognizer-Text:** サンプルでは、ユーザー入力の解析および検証に Microsoft/Recognizers-Text ライブラリが使用されます。 詳細については、[概要](https://github.com/Microsoft/Recognizers-Text#microsoft-recognizers-text-overview)に関するページを参照してください。

## <a name="next-steps"></a>次のステップ

ボット データをストレージに読み書きできるように状態を構成する方法がわかりました。次は、ユーザーに一連の質問を行い、その回答を検証して、入力を保存する方法を確認しましょう。

> [!div class="nextstepaction"]
> [ユーザー入力を収集するために独自のプロンプトを作成する](bot-builder-primitive-prompts.md)。
