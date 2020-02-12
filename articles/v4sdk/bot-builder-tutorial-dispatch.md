---
title: 複数の LUIS と QnA モデルを使用する - Bot Service
description: ボットで LUIS や QnA Maker を使用する方法について説明します。
keywords: Luis, QnA, ディスパッチ ツール, 複数のサービス, 意図のルーティング
author: diberry
ms.author: diberry
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/27/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: bea377055010077d964f639518556e12e212f1ad
ms.sourcegitcommit: 36d6f06ffafad891f6efe4ff7ba921de8a306a94
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/31/2020
ms.locfileid: "76895708"
---
# <a name="use-multiple-luis-and-qna-models"></a>複数の LUIS および QnA モデルを使用する

[!INCLUDE[applies-to](../includes/applies-to.md)]

ボットで複数の LUIS モデルおよび QnA Maker ナレッジ ベース (ナレッジ ベース) が使用されている場合は、ディスパッチ ツールを使用して、どの LUIS モデルまたは QnA Maker ナレッジ ベースがユーザー入力に最も適しているかを判断できます。 ディスパッチ ツールでは、これを行うために、ユーザー入力を正しいモデルにルーティングする 1 つの LUIS アプリが作成されます。 ディスパッチ (CLI コマンドを含む) の詳細については、[README][dispatch-readme] を参照してください。

## <a name="prerequisites"></a>前提条件

- [ボットの基本](bot-builder-basics.md)、[LUIS][howto-luis]、および [QnA Maker][howto-qna] に関する知識。
- [ディスパッチ ツール](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch)
- **ディスパッチによる NLP** のコピー ([C# サンプル][cs-sample]、[JS サンプル][js-sample]、または [Python サンプル][python-sample]のコード リポジトリ)。
- LUIS アプリを発行するための [luis.ai](https://www.luis.ai/) アカウント。
- QnA ナレッジ ベースを発行するための [QnA Maker](https://www.qnamaker.ai/) アカウント。

## <a name="about-this-sample"></a>このサンプルについて

このサンプルは、LUIS および QnA Maker アプリの定義済みセットに基づいています。

## <a name="ctabcs"></a>[C#](#tab/cs)

![コード サンプル ロジック フロー](./media/tutorial-dispatch/dispatch-logic-flow.png)

ユーザー入力を受け取るたびに、`OnMessageActivityAsync` が呼び出されます。 このモジュールでは、最上位スコアのユーザーの意図を検索し、その結果を `DispatchToTopIntentAsync` に渡します。 次に、DispatchToTopIntentAsync によって、適切なアプリ ハンドラーが呼び出されます

- `ProcessSampleQnAAsync` - ボットの FAQ の質問。
- `ProcessWeatherAsync` -天気クエリ。
- `ProcessHomeAutomationAsync` -家庭用照明コマンド。

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

![コード サンプル ロジック フロー](./media/tutorial-dispatch/dispatch-logic-flow-js.png)

ユーザー入力を受け取るたびに、`onMessage` が呼び出されます。 このモジュールでは、最上位スコアのユーザーの意図を検索し、その結果を `dispatchToTopIntentAsync` に渡します。 次に、dispatchToTopIntentAsync によって、適切なアプリ ハンドラーが呼び出されます

- `processSampleQnA` - ボットの FAQ の質問。
- `processWeather` -天気クエリ。
- `processHomeAutomation` -家庭用照明コマンド。

## <a name="pythontabpython"></a>[Python](#tab/python)

![コード サンプル ロジック フロー](./media/tutorial-dispatch/dispatch-logic-flow-python.png)

ユーザー入力を受け取るたびに、`on_message_activity` が呼び出されます。 このモジュールでは、最上位スコアのユーザーの意図を検索し、その結果を `_dispatch_to_top_intent` に渡します。 次に、_dispatch_to_top_intent によって、適切なアプリ ハンドラーが呼び出されます

- `_process_sample_qna` - ボットの FAQ の質問。
- `_process_weather` -天気クエリ。
- `_process_home_automation` -家庭用照明コマンド。

---

ハンドラーによって LUIS または QnA Maker サービスが呼び出され、生成された結果がユーザーに返されます。

## <a name="create-luis-apps-and-qna-knowledge-base"></a>LUIS アプリと QnA ナレッジ ベースを作成する

ディスパッチ モデルを作成するには、ご自身の LUIS アプリと QnA ナレッジ ベースを作成し、発行しておく必要があります。 この記事では、`\CognitiveModels` フォルダーの "_ディスパッチによる NLP_" サンプルに含まれる次のモデルを公開します。

| Name | [説明] |
|------|------|
| HomeAutomation | 関連付けられているエンティティ データによってホーム オートメーションの意図を認識する LUIS アプリ。|
| Weather | 場所データによって天気関連の意図を認識する LUIS アプリ。|
| QnA Maker  | ボットに関するシンプルな質問への回答を提供する QnA Maker ナレッジ ベース。 |

### <a name="create-luis-apps"></a>LUIS アプリを作成する

1. [LUIS Web ポータル](https://www.luis.ai/)にログインします。 _[マイ アプリ]_ セクションの _[Import new app]\(新しいアプリのインポート\)_ タブを選択します。 次のダイアログ ボックスが表示されます。

    ![LUIS JSON ファイルをインポートする](./media/tutorial-dispatch/import-new-luis-app.png)

2. _[Choose app file]\(アプリ ファイルの選択\)_ ボタンを選択し、ご自身のサンプル コードの CognitiveModel フォルダーに移動して、"HomeAutomation.json" ファイルを選択します。 省略可能な名前のフィールドは空白のままにします。

3. _[完了]_ を選択します。

4. LUIS で Home Automation アプリが開いたら、 _[Train]\(トレーニング\)_ ボタンを選択します。 これで、'home-automation.json' ファイルでインポートした一連の発話を使用してアプリがトレーニングされます。

5. トレーニングが完了したら、 _[Publish]\(公開\)_ ボタンを選択します。 次のダイアログ ボックスが表示されます。

    ![LUIS アプリを公開する](./media/tutorial-dispatch/publish-luis-app.png)

6. '運用' 環境を選択し、 _[Publish]\(公開\)_ ボタンを選択します。

7. 新しい LUIS アプリが公開されたら、 _[MANAGE]\(管理\)_ タブを選択します。[アプリケーション情報] ページから、`Application ID` の値として "_app-id-for-app_" を、また `Display name` の値として "_name-of-app_" を記録しておきます。 [キーとエンドポイント] ページから、`Authoring Key` の値として "_your-luis-authoring-key_" を、また `Region` の値として "_your-region_" を記録しておきます。 これらの値は、お使いの appsetting.json ファイル内で後で使用されます。

8. 完了したら、"Weather.json" ファイルに対して上記の手順を繰り返して、ご自身の LUIS **Home Automation** アプリと LUIS **Weather** アプリの両方を "_トレーニング_" し、"_公開_" します。

### <a name="create-qna-maker-knowledge-base"></a>QnA Maker ナレッジ ベースの作成

QnA Maker ナレッジ ベースを設定する最初の手順は、Azure で QnA Maker サービスを設定することです。 そのためには、[こちら](https://aka.ms/create-qna-maker)で説明されている手順に従います。

Azure で QnA Maker サービスを作成したら、QnA Maker サービス用に提供される Cognitive Services の "_キー 1_" を記録しておく必要があります。 これは、QnA Maker アプリをご利用のディスパッチ アプリケーション追加する際に、\<azure-qna-service-key1> として使用します。

QnA Maker で使用される [2 つの異なる種類のキー](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure#types-of-keys-in-qna-maker)の詳細については、こちらを参照してください。

以降の手順では、このキーが提供されます。

![コグニティブ サービスを選択する](./media/tutorial-dispatch/select-qna-cognitive-service.png)

1. ご利用の Azure portal 内で、QnA Maker コグニティブ サービスを選択します。

    ![コグニティブ サービス キーを選択する](./media/tutorial-dispatch/select-cognitive-service-keys.png)

1. 左側のメニューで、 _[リソース管理]_ セクションの下にあるキー アイコンを選択します。

    ![コグニティブ サービス キー 1 を選択する](./media/tutorial-dispatch/select-cognitive-service-key1.png)

1. "_キー 1_" の値をクリップボードにコピーして、ローカルに保存します。 これは、後で QnA Maker アプリをご利用のディスパッチ アプリケーションに追加する際に、(-k) キー値 \<azure-qna-service-key1> の代わりに使用します。

1. 次に [QnAMaker Web ポータル](https://qnamaker.ai)にサインインします。

1. 手順 2 では、次のように選択します。

    - Azure AD アカウント。
    - Azure サブスクリプション名。
    - QnA Maker サービス用に作成した名前 (最初の段階でこのプル ダウン リストに Azure QnA サービスが表示されない場合は、ページを更新してみてください)。

    ![QnA の作成手順 2](./media/tutorial-dispatch/create-qna-step-2.png)

1. 手順 3 では、QnA Maker ナレッジベースの名前を指定します。 この例では、名前 'sample-qna' を使用します。

    ![QnA の作成手順 3](./media/tutorial-dispatch/create-qna-step-3.png)

1. 手順 4 では、 _[+ ファイルの追加]_ オプションを選択し、ご自身のサンプル コードの CognitiveModel フォルダーに移動して、"QnAMaker.tsv" ファイルを選択します。 ナレッジ ベースには _Chit-chat_ の性格を追加する選択がもう 1 つありますが、この例にはこのオプションが含まれていません。

    ![QnA の作成手順 4](./media/tutorial-dispatch/create-qna-step-4.png)

1. 手順 5 では、 _[Create your knowledge base]\(ナレッジ ベースの作成\)_ を選択します。

1. アップロードしたファイルからナレッジ ベースが作成されたら、 _[Save and train]\(保存してトレーニング\)_ を選択し、完了したら、 _[PUBLISH]\(公開\)_ タブを選択してアプリを公開します。

1. QnA Maker アプリが公開されたら、 _[SETTINGS]\(設定\)_ タブを選択し、[Deployment details]\(デプロイの詳細\) まで下にスクロールします。 _Postman_ サンプル HTTP 要求の次の値を書き留めます。

    ```text
    POST /knowledge bases/<knowledge-base-id>/generateAnswer
    Host: <your-hostname>  // NOTE - this is a URL.
    Authorization: EndpointKey <qna-maker-resource-key>
    ```

    ホスト名の完全な URL 文字列は、"https://<host-id>.azure.net/qnamaker" のようになります。 これらの値は、お使いの `appsettings.json` ファイルまたは `.env` ファイルで後で使用されます。

## <a name="dispatch-app-needs-read-access-to-existing-apps"></a>ディスパッチ アプリには既存のアプリへの読み取りアクセスが必要

LUIS アプリおよび QnA Maker アプリにディスパッチする新しい親 LUIS アプリを作成するために、ディスパッチ ツールには既存の LUIS アプリと QnA Maker アプリを読み取るための作成アクセスが必要です。 このアクセスには、アプリ ID と作成キーが提供されます。

### <a name="service-authoring-keys"></a>サービス作成キー

**作成キー**は、モデルの作成と編集にのみ使用されます。 2 つの LUIS アプリのそれぞれ、および QnA Maker アプリに対して ID とキーが必要です。

|アプリ|情報の場所|
|--|--|
|LUIS|**アプリ ID** - アプリごとに [LUIS ポータル](https://www.luis.ai)に表示されます ([管理] -> [アプリケーション情報])。<br>**作成キー** - LUIS ポータルの右上隅に表示されます。自分の [ユーザー]、[設定] の順に選択します。|
|QnA Maker| **アプリ ID** - アプリを公開した後、[設定] ページ上の [QnA Maker ポータル](https://http://qnamaker.ai)に表示されます。 これは、knowledgebase の後にくる POST コマンドの最初の部分にある ID です。 アプリ ID を検査する場所の例として、`POST /knowledgebases/<APP-ID>/generateAnswer` が挙げられます。<br>**作成キー** - Azure portal で QnA Maker リソースの **[キー]** の下にあります。 複数のキーのうち、必要なのは 1 つだけです。|

作成キーは、発行されたアプリケーションから予測スコアまたは信頼スコアを取得するためには使用されません。 この操作にはエンドポイント キーが必要です。 **[エンドポイント キー](#service-endpoint-keys)** は、このチュートリアルで後ほど取り上げて使用します。

QnA Maker で使用される [2 つの異なる種類のキー](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure#types-of-keys-in-qna-maker)の詳細については、こちらを参照してください。

## <a name="create-the-dispatch-model"></a>ディスパッチ モデルを作成する

ディスパッチ ツールの CLI インターフェイスにより、適切な LUIS アプリまたは QnA Maker アプリにディスパッチするためのモデルが作成されます。

1. コマンド プロンプトまたはターミナル ウィンドウを開き、ディレクトリを **CognitiveModels** ディレクトリに変更します
1. 最新バージョンの npm とディスパッチ ツールがインストールされていることを確認します。

    ```cmd
    npm i -g npm
    npm i -g botdispatch
    ```

1. `dispatch init` を使用して初期化し、ご自身のディスパッチ モデル用に `.dispatch` ファイルを作成します。 これは、あなたが見てわかるファイル名を使用して作成します。

    ```cmd
    dispatch init -n <filename-to-create> --luisAuthoringKey "<your-luis-authoring-key>" --luisAuthoringRegion <your-region>
    ```

1. `dispatch add` を使用して、ご自身の LUIS アプリと QnA Maker ナレッジ ベースを `.dispatch` ファイルに追加します。

    ```cmd
    dispatch add -t luis -i "<app-id-for-weather-app>" -n "<name-of-weather-app>" -v <app-version-number> -k "<your-luis-authoring-key>" --intentName l_Weather
    dispatch add -t luis -i "<app-id-for-home-automation-app>" -n "<name-of-home-automation-app>" -v <app-version-number> -k "<your-luis-authoring-key>" --intentName l_HomeAutomation
    dispatch add -t qna -i "<knowledge-base-id>" -n "<knowledge-base-name>" -k "<azure-qna-service-key1>" --intentName q_sample-qna
    ```

1. `dispatch create` を使用して、`.dispatch` ファイルからディスパッチ モデルを生成します。

    ```cmd
    dispatch create
    ```

1. 先ほど作成したディスパッチ LUIS アプリを公開します。

## <a name="use-the-dispatch-luis-app"></a>ディスパッチ LUIS アプリを使用する

生成された LUIS アプリによって、子アプリおよびナレッジ ベースごとに意図が定義されます。発話に適合する意図がない場合は、_none_ が定義されます。

- `l_HomeAutomation`
- `l_Weather`
- `None`
- `q_sample-qna`

ボットを適切に動作させるには、これらのサービスが正しい名前で公開されている必要があります。
公開されたサービスにボットがアクセスできるようにするには、アクセス対象サービスに関する情報がそのボットに必要です。

### <a name="service-endpoint-keys"></a>サービス エンドポイント キー

このボットには、3 つの LUIS アプリ (ディスパッチ、天気、およびホーム オートメーション) 用のクエリ予測エンドポイントと、単一の QnA Maker ナレッジ ベースが必要です。 次の表を使用してエンドポイント キーを見つけます。

|アプリ|クエリ エンドポイント キーの場所|
|--|--|
|LUIS|LUIS ポータルで各 LUIS アプリの [管理] セクションで、 **[Keys and Endpoint settings]\(キーとエンドポイントの設定\)** を選択して、各アプリに関連付けられているキーを見つけます。 このチュートリアルを行う場合、エンドポイント キーは `<your-luis-authoring-key>` と同じキーです。 作成キーでは、1000 回のエンドポイント ヒットが許可され、その後有効期限が切れます。|
|QnA Maker|QnA Maker ポータルのナレッジ ベースの [管理] 設定で、**Authorization** ヘッダー用の [Postman]\(Postman\) 設定に表示されているキー値を、テキスト `EndpointKey` なしで使用します。|

これらの値は、C# の場合は **appsettings.json** ファイルで使用され、javascript の場合は **.env** ファイルで使用されます。

## <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="installing-packages"></a>パッケージのインストール

このアプリを初めて実行する前に、いくつかの NuGet パッケージがインストールされていることを確認します。

- **Microsoft.Bot.Builder**
- **Microsoft.Bot.Builder.AI.Luis**
- **Microsoft.Bot.Builder.AI.QnA**

### <a name="manually-update-your-appsettingsjson-file"></a>お使いの appsettings.json ファイルを手動で更新する

すべてのサービス アプリが作成されたら、それぞれの情報を "appsettings.json" ファイルに追加する必要があります。 初期 [C# サンプル][cs-sample] コードには、空の appsettings.json ファイルが含まれています。

**appsettings.json**

[!code-json[AppSettings](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/AppSettings.json?range=8-17)]

次に示すエンティティごとに、前の手順で記録した値を追加します。

**appsettings.json**

```json
"MicrosoftAppId": "",
"MicrosoftAppPassword": "",
  
"QnAKnowledgebaseId": "<knowledge-base-id>",
"QnAEndpointKey": "<qna-maker-resource-key>",
"QnAEndpointHostName": "<your-hostname>",

"LuisAppId": "<app-id-for-dispatch-app>",
"LuisAPIKey": "<your-luis-endpoint-key>",
"LuisAPIHostName": "<your-dispatch-app-region>",
```

すべての変更が完了したら、このファイルを保存します。

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="installing-packages"></a>パッケージのインストール

このアプリを初めて実行する前に、npm パッケージをいくつかインストールする必要があります。

```powershell
npm install --save botbuilder
npm install --save botbuilder-ai
```

.env 構成ファイルを使用するには、お使いのボットに追加のパッケージが含まれている必要があります。

```powershell
npm install --save dotenv
```

### <a name="manually-update-your-env-file"></a>.env ファイルを手動で更新する

すべてのサービス アプリが作成されたら、それぞれの情報を ".env" ファイルに追加する必要があります。 初期 [JavaScript サンプル][js-sample] コードには、空の .env ファイルが含まれています。

**.env**  
[!code-file[EmptyEnv](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/.env?range=1-10)]

次に示すように、ご自身のサービス接続の値を追加します。

**.env**

```file
MicrosoftAppId=""
MicrosoftAppPassword=""

QnAKnowledgebaseId="<knowledge-base-id>"
QnAEndpointKey="<qna-maker-resource-key>"
QnAEndpointHostName="<your-hostname>"

LuisAppId=<app-id-for-dispatch-app>
LuisAPIKey=<your-luis-endpoint-key>
LuisAPIHostName=<your-dispatch-app-region>
```

すべての変更が完了したら、このファイルを保存します。

## <a name="pythontabpython"></a>[Python](#tab/python)

### <a name="installing-packages"></a>パッケージのインストール

このアプリを初めて実行する前に、pypi パッケージをいくつかインストールする必要があります。

```powershell
pip install azure
pip install botbuilder-core
pip install botbuilder-ai
```

### <a name="manually-update-your-configpy-file"></a>config.py ファイルを手動で更新する
すべてのサービス アプリが作成されたら、それぞれの情報を "config.py" ファイルに追加する必要があります。 この最初の [Python サンプル][python-sample] コードには、空の config.py ファイルが含まれています。

**config.py**

[!code-python[config.py](~/../botbuilder-python/samples/python/14.nlp-with-dispatch/config.py?range=10-24)]

次に示すエンティティごとに、前の手順で記録した値を追加します。

```python
APP_ID = os.environ.get("MicrosoftAppId", "")
APP_PASSWORD = os.environ.get("MicrosoftAppPassword", "")

QNA_KNOWLEDGEBASE_ID = os.environ.get("QnAKnowledgebaseId", "<knowledge-base-id>")
QNA_ENDPOINT_KEY = os.environ.get("QnAEndpointKey", "<qna-maker-resource-key>")
QNA_ENDPOINT_HOST = os.environ.get("QnAEndpointHostName", "<your-hostname>")

LUIS_APP_ID = os.environ.get("LuisAppId", "<app-id-for-dispatch-app>")
LUIS_API_KEY = os.environ.get("LuisAPIKey", "<your-luis-endpoint-key>")
# LUIS endpoint host name, ie "westus.api.cognitive.microsoft.com"
LUIS_API_HOST_NAME = os.environ.get("LuisAPIHostName", "<your-dispatch-app-region>")
```

すべての変更が完了したら、このファイルを保存します。

---

### <a name="connect-to-the-services-from-your-bot"></a>ボットからサービスに接続する

Dispatch、LUIS、および QnA Maker サービスに接続するために、ボットは設定ファイルから情報をプルします。

## <a name="ctabcs"></a>[C#](#tab/cs)

**BotServices.cs** では、ご自身のディスパッチ ボットと `Dispatch` サービスおよび `SampleQnA` サービスとの接続に、構成ファイル _appsettings.json_ に含まれている情報が使用されます。 コンストラクターでは、ご自身で指定した値が、これらのサービスへの接続に使用されます。

**BotServices.cs**

[!code-csharp[ReadConfigurationInfo](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/BotServices.cs?range=14-45)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**dispatchBot.js** では、ご自身のディスパッチ ボットと _LuisRecognizer(dispatch)_ サービスおよび _QnAMaker_ サービスとの接続に、構成ファイル _.env_ に含まれている情報が使用されます。 コンストラクターでは、ご自身で指定した値が、これらのサービスへの接続に使用されます。

**bots/dispatchBot.js**

[!code-javascript[ReadConfigurationInfo](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=11-26)]

## <a name="pythontabpython"></a>[Python](#tab/python)

**dispatch_bot.py** では、ディスパッチ ボットを _QnAMaker_ および _LuisRecognizer_ サービスに接続するために、構成ファイル _config.py_ 内に含まれている情報が使用されます。 コンストラクターでは、ご自身で指定した値が、これらのサービスへの接続に使用されます。

**bots/dispatch_bot.py**

[!code-python[ReadConfigurationInfo](~/../botbuilder-python/samples/python/14.nlp-with-dispatch/bots/dispatch_bot.py?range=14-34)]

---

> [!NOTE]
> 既定では、`includeApiResults` パラメーターは false に設定されています。つまり、レコグナイザーはエンティティ/意図に関する基本情報のみを返します。 LUIS からの完全な応答 (このチュートリアルで後ほど使用する `ConnectedServiceResult` など) が必要な場合は、このパラメーターを true に設定します。 これにより、LUIS サービスからの完全な応答が、`RecognizerResult` の Properties コレクションに追加されます。

### <a name="call-the-services-from-your-bot"></a>ボットからのサービスを呼び出す

ボット ロジックでは、ユーザーからの入力ごとに、結合されたディスパッチ モデルに対してユーザー入力が確認され、返された最上位の意図が検索されます。その後、その情報を使用して、その入力に適したサービスが呼び出されます。

## <a name="ctabcs"></a>[C#](#tab/cs)

**DispatchBot.cs** ファイルでは、`OnMessageActivityAsync` メソッドが呼び出されるたびに、受信ユーザー メッセージをディスパッチ モデルに対して確認します。 その後、ディスパッチ モデルの `topIntent` と `recognizerResult` を適切なメソッドに渡して、サービスを呼び出し、結果を返します。

**bots\DispatchBot.cs**

[!code-csharp[OnMessageActivity](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/bots/DispatchBot.cs?range=26-36)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**dispatchBot.js** の `onMessage` メソッドでは、ディスパッチ モデルに対してユーザー入力メッセージを確認し、_topIntent_ を検索します。次に、_dispatchToTopIntentAsync_ を呼び出して、これを渡します。

[!code-javascript[onMessage](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=31-44)]

## <a name="pythontabpython"></a>[Python](#tab/python)

**dispatch_bot.py** ファイルでは、`on_message_activity` メソッドが呼び出されるたびに、受信ユーザー メッセージをディスパッチ モデルに対して確認します。 その後、ディスパッチ モデルの `top_intent` と `recognize_result` を適切なメソッドに渡して、サービスを呼び出し、結果を返します。

**bots/dispatch_bot.py**

[!code-python[on_message](~/../botbuilder-python/samples/python/14.nlp-with-dispatch/bots/dispatch_bot.py?range=46-54)]

---

### <a name="work-with-the-recognition-results"></a>認識結果を操作する

## <a name="ctabcs"></a>[C#](#tab/cs)

モデルによって結果が生成されるとき、どのサービスが発話を最も適切に処理できるかが示されます。 このボットのコードでは、要求を対応するサービスにルーティングし、呼び出されたサービスからの応答を要約します。 Dispatch から返された "_意図_" によっては、このコードでは、返された意図を使用して、適切な LUIS モデルまたは QnA サービスへのルーティングが行われます。

**bots\DispatchBot.cs**

[!code-csharp[DispatchToTop](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/bots/DispatchBot.cs?range=51-69)]

`ProcessHomeAutomationAsync` または `ProcessWeatherAsync` のいずれかのメソッドが呼び出されると、_luisResult.ConnectedServiceResult_ 内のディスパッチ モデルから結果が渡されます。 その後、指定されたメソッドによって、ユーザー フィードバックが提供されます。このフィードバックには、ディスパッチ モデルの最上位の意図と、検出された意図およびエンティティの一覧がランク付けされて示されています。

`q_sample-qna` メソッドが呼び出されると、turnContext 内に含まれるユーザー入力が、ナレッジ ベースからの応答の生成と、ユーザーへの結果の表示に使用されます。

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

モデルによって結果が生成されるとき、どのサービスが発話を最も適切に処理できるかが示されます。 このサンプルのコードでは、対応するサービスへの要求のルーティング方法を示すために、認識された _topIntent_ が使用されます。

**bots/dispatchBot.js**  
[!code-javascript[dispatchToTopIntentAsync](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=61-77)]

`processHomeAutomation` または `processWeather` のいずれかのメソッドが呼び出されると、_recognizerResult.luisResult_ 内のディスパッチ モデルから結果が渡されます。 その後、指定されたメソッドによって、ユーザー フィードバックが提供されます。このフィードバックには、ディスパッチ モデルの最上位の意図と、検出された意図およびエンティティの一覧がランク付けされて示されています。

`q_sample-qna` メソッドが呼び出されると、turnContext 内に含まれるユーザー入力が、ナレッジ ベースからの応答の生成と、ユーザーへの結果の表示に使用されます。

## <a name="pythontabpython"></a>[Python](#tab/python)

モデルによって結果が生成されるとき、どのサービスが発話を最も適切に処理できるかが示されます。 このサンプルのコードでは、対応するサービスへの要求のルーティング方法を示すために、認識された最上位の "_意図_" が使用されます。

**bots\dispatch_bot.py**

[!code-python[dispatch top intent](~/../botbuilder-python/samples/python/14.nlp-with-dispatch/bots/dispatch_bot.py?range=56-70)]

`_process_home_automation` または `_process_weather` のいずれかのメソッドが呼び出されると、_recognizer_result.properties["luisResult"]_ 内のディスパッチ モデルから結果が渡されます。 その後、指定されたメソッドによって、ユーザー フィードバックが提供されます。このフィードバックには、ディスパッチ モデルの最上位の意図と、検出された意図およびエンティティの一覧がランク付けされて示されています。

`q_sample-qna` メソッドが呼び出されると、turnContext 内に含まれるユーザー入力が、ナレッジ ベースからの応答の生成と、ユーザーへの結果の表示に使用されます。

---

> [!NOTE]
> これが運用環境のアプリケーションの場合は、ここで、選択した LUIS メソッドが、指定されたサービスに接続されたうえで、そのメソッドによってユーザー入力が渡され、返された LUIS の意図とエンティティ データが処理されます。

## <a name="test-your-bot"></a>ボットをテストする

1. ご自身の開発環境を使用して、サンプル コードを開始します。 ご自身のアプリによって開かれたブラウザー ウィンドウのアドレス バーに表示されている次の _localhost_ アドレスをメモしてください: "https://localhost:<Port_Number>" 
1. ご利用の Bot Framework Emulator を開き、[`Create a new bot configuration`] を選択します。 `.bot` ファイルを使用すると、ボット エミュレーター内の "_インスペクター_" を使用して、LUIS および QnA Maker から返された JSON を確認できます。
1. **[New bot configuration]\(新しいボット構成\)** ダイアログ ボックスで、ご自身のボット名とエンドポイント URL (`http://localhost:3978/api/messages` など) を入力します。 ご自身のボット サンプル コード プロジェクトのルートにファイルを保存します。
1. ボット ファイルを開き、ご自身の LUIS アプリおよび QnA Maker アプリのセクションを追加します。 [このサンプル ファイル](https://github.com/microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/sample-bot-file.json)を設定用のテンプレートとして使用します。 変更を保存します。
1. **[マイ ボット]** リストでボット名を選択して、実行中のボットにアクセスします。 参考のために、ご自身のボット用に作成されたサービスに含まれる質問とコマンドの一部を次に示します。

    - QnA Maker
      - `hi`, `good morning`
      - `what are you`, `what do you do`
    - LUIS (ホーム オートメーション)
      - `turn on bedroom light`
      - `turn off bedroom light`
      - `make some coffee`
    - LUIS (天気)
      - `whats the weather in redmond washington`
      - `what's the forecast for london`
      - `show me the forecast for nebraska`

## <a name="dispatch-for-user-utterance-to-qna-maker"></a>QnA Maker へのユーザー発話のためのディスパッチ

1. ボット エミュレーターで、テキスト `hi` を入力して発話を送信します。 ボットから、このクエリがディスパッチ LUIS アプリに送信されると、さらなる処理のためにどの子アプリがこの発話を取得する必要があるかを示す応答がボットに届きます。

1. ログ内の `LUIS Trace` 行を選択すると、ボット エミュレーター内で LUIS 応答を確認できます。 ディスパッチ LUIS アプリからの LUIS 結果がインスペクターに表示されます。

    ```json
    {
      "luisResponse": {
        "entities": [],
        "intents": [
          {
            "intent": "q_sample-qna",
            "score": 0.9489713
          },
          {
            "intent": "l_HomeAutomation",
            "score": 0.0612499453
          },
          {
            "intent": "None",
            "score": 0.008567564
          },
          {
            "intent": "l_Weather",
            "score": 0.0025761195
          }
        ],
        "query": "Hi",
        "topScoringIntent": {
          "intent": "q_sample-qna",
          "score": 0.9489713
        }
      }
    }
    ```

    発話 `hi` は、ディスパッチ LUIS アプリの **q_sample-qna** 意図の一部であり、`topScoringIntent` として選択されているため、ボットでは同じ発話を使用して今回は QnA Maker アプリに 2 番目の要求が行われます。

1. ボット エミュレーター ログ内で `QnAMaker Trace` 行を選択します。 インスペクターに QnA Maker の結果が表示されます。

```json
{
    "questions": [
        "hi",
        "greetings",
        "good morning",
        "good evening"
    ],
    "answer": "Hello!",
    "score": 1,
    "id": 96,
    "source": "QnAMaker.tsv",
    "metadata": [],
    "context": {
        "isContextOnly": false,
        "prompts": []
    }
}
```

## <a name="resolving-incorrect-top-intent-from-dispatch"></a>ディスパッチによる誤っている最上位の意図の解決

ボットが起動したら、ディスパッチされたアプリ間での類似の発話または重複する発話を削除することで、ボットのパフォーマンスを向上することができます。
<!--For example, let's say that in the `Home Automation` LUIS app requests like "turn my lights on" map to a "TurnOnLights" intent, but requests like "Why won't my lights turn on?" map to a "None" intent so that they can be passed on to QnA Maker. These two utterances are too close for the dispatch LUIS app to determine if the correct child app is the LUIS app or the QnA Maker app.

When you combine the LUIS app and the QnA Maker app using dispatch, you need to do _one_ of the following:

- Remove the "None" intent from the child `Home Automation` LUIS app, and instead add the utterances from that intent to the "None" intent in the dispatcher app.
- Add logic in your bot to pass the messages that match the Dispatch LUIS app's "None" intent on to the QnA maker service. Compare the score of the Dispatch LUIS app's score and the score of the QnA Maker app. Use the highest score. This effectively removes QnA Maker from the Dispatch cycle.

Either of the above two actions will reduce the number of times that your bot responds back to your users with the message, 'Couldn't find an answer.'
-->
[Dispatch][dispatch-readme] コマンドライン ツールを使用して、ディスパッチ モデルをテストおよび評価することができます。

### <a name="to-update-or-create-a-new-luis-model"></a>LUIS モデルを更新するか、新しく作成する

このサンプルは、事前構成済みの LUIS モデルが基になっています。 このモデルの更新または新しい LUIS モデルの作成に役立つ追加情報については、[こちら](https://aka.ms/create-luis-model#updating-your-cognitive-models)をご覧ください。

基になるモデル (QnA または LUIS) を更新した後、`dispatch refresh` を実行して Dispatch LUIS アプリを更新します。 `dispatch refresh` は基本的に `dispatch create` と同じコマンドですが、新しい LUIS アプリ ID は作成されません。 

LUIS で直接追加された発話は、`dispatch refresh` の実行時に保持されないことに注意してください。 これらの追加の発話を Dispatch アプリで保持するには、テキスト ファイルでそれらの発話を (1 行に 1 発話ずつ) 追加し、次のコマンドを実行してファイルを Dispatch に追加します。

```powershell
dispatch add -t file -f <file path> --intentName <target intent name, ie l_General>
```

追加の発話を含むファイルが Dispatch に追加されると、更新してもそれらの発話は残ります。

### <a name="to-delete-resources"></a>リソースを削除する

このサンプルでは、次の手順を使用して削除できるアプリケーションとリソースが多数作成されますが、"*他のアプリやサービス*" が依存しているリソースは削除しないでください。

LUIS リソースを削除するには:

1. [luis.ai](https://www.luis.ai) ポータルにサインインします。
1. _[マイ アプリ]_ ページに移動します。
1. このサンプルによって作成されたアプリを選択します。
   - `Home Automation`
   - `Weather`
   - `NLP-With-Dispatch-BotDispatch`
1. _[削除]_ をクリックし、 _[OK]_ をクリックして確認します。

QnA Maker リソースを削除するには:

1. [qnamaker.ai](https://www.qnamaker.ai/) ポータルにサインインします。
1. _[My knowledge bases]\(マイ ナレッジ ベース\)_ ページに移動します。
1. `Sample QnA` ナレッジ ベースの削除ボタンをクリックし、 _[削除]_ をクリックして確認します。

### <a name="best-practice"></a>ベスト プラクティス

このサンプルで使用されているサービスを向上させるには、[LUIS](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-best-practices) および [QnA Maker](https://docs.microsoft.com/azure/cognitive-services/qnamaker/concepts/best-practices) 向けのベスト プラクティスを参照してください。

<!-- Foot-note style links -->



[howto-luis]: bot-builder-howto-v4-luis.md
[howto-qna]: bot-builder-howto-qna.md

[cs-sample]: https://aka.ms/dispatch-sample-cs
[js-sample]: https://aka.ms/dispatch-sample-js
[python-sample]: https://aka.ms/dispatch-sample-python

[dispatch-readme]: https://aka.ms/dispatch-command-line-tool
<!--[dispatch-evaluate]: https://aka.ms/dispatch-command-line-tool#evaluating-your-dispatch-model-->
