---
title: Botbuilder テンプレートを使用してボットを作成する
description: Botbuilder ツールを使用すると、コマンド ラインからボットのリソースを直接管理できます。
keywords: botbuilder テンプレート, node.js, python, java, .net, ludown, yeoman
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/25/2018
ms.openlocfilehash: 8004389aba58b5cf79f1559b3ce65d1d66c5358c
ms.sourcegitcommit: 44f100a588ffda19c275b118f4f97029f12d1449
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/26/2018
ms.locfileid: "42928340"
---
# <a name="create-bots-with-botbuilder-templates"></a>Botbuilder テンプレートを使用してボットを作成する

> [!NOTE]
> このトピックは、SDK の v3 および v4 バージョンに適用されます。 その他のメモについては、以下をご覧ください。

各 BotBilder SDK プラットフォームで、テンプレートを使用してボットを作成できるようになりました。 

- Node.js
- Python
- Java
- .NET

## <a name="prerequisites"></a>前提条件

Node.js、Java、および Python のテンプレートで、いずれも単純なエコー ボットを作成できます。また、いずれも [Yeoman](http://yeoman.io/) を介して入手できます。

- [Node.js](https://nodejs.org/en/) v 8.5 以降
- [Yeoman](http://yeoman.io/)

Yeoman をインストールしていない場合は、グローバルにインストールする必要があります。

```shell
npm install -g yo
```

## <a name="nodejs-python-java-templates"></a>Node.js、Python、Java テンプレート
コマンド ラインから cd を実行して、選択した新しいフォルダーに移動します。 Botbuilder Node.js テンプレートには、SDK の **v3** バージョンと **v4** バージョンをそれぞれ対象とする 2 つのバージョンが用意されています。 Python と Java のテンプレートは、v4 でのみ使用できます。 

**Node.js v3 Botbuilder** テンプレート ジェネレーターをインストールするには:

```shell
npm install generator-botbuilder
```

**Node.js v4 Botbuilder** テンプレート ジェネレーターをインストールするには:
```shell
npm install generator-botbuilder@preview
```

Python と Java のテンプレートは、**v4 でのみ使用できます**。 

**Python** の場合:
```shell
npm install generator-botbuilder-python
```

**Java** の場合:
```shell
npm install generator-botbuilder-java
```

ジェネレーターがインストールされたら、CLI で **yo** コマンドを実行するだけで、Yeoman で使用できるジェネレーターの一覧を表示できます。

![Yeoman のインターフェイス](media/botbuilder-templates/yeoman-generator-botbuilder.png)

選択した作業ディレクトリに切り替え、使用するジェネレーターを選択します。 名前や説明など、ボットを作成するためのさまざまなオプションの入力を求められます。 すべてのプロンプトが完了すると、同じ作業フォルダー内にエコー ボット テンプレートが作成されます。

![Node.js Yeoman テンプレート](media/botbuilder-templates/new-template-node.png)


## <a name="net"></a>.NET

.NET 用には、SDK の **v3** バージョンと **v4** バージョンをそれぞれ対象とする 2 つのテンプレートが用意されています。 いずれも [VSIX](https://docs.microsoft.com/en-us/visualstudio/extensibility/anatomy-of-a-vsix-package) パッケージとして入手できます。[Visual Studio Marketplace](https://marketplace.visualstudio.com/) の次のリンクの 1 つをクリックしてダウンロードします。

- [BotBuilder V3 テンプレート](https://marketplace.visualstudio.com/items?itemName=BotBuilder.BotBuilderV3)
- [BotBuilder V4 テンプレート](https://aka.ms/Ylcwxk)

#### <a name="prerequisites"></a>前提条件

- [Visual Studio 2015 以降](https://www.visualstudio.com/downloads/)
- [Azure アカウント](https://azure.microsoft.com/en-us/free/)

### <a name="install-the-templates"></a>テンプレートをインストールする

保存されたディレクトリから、VSIX パッケージを開くだけで Botbuilder テンプレートが Visual Studio にインストールされ、次に開いたときから利用できます。

![VSIX パッケージ](media/botbuilder-templates/botbuilder-vsix-template.png)

テンプレートを使用して新しいボット プロジェクトを作成するには、Visual Studio を開き、**[ファイル]** > **[新規]** > **[プロジェクト]** を選択し、Visual C# から **[Bot Framework]**、Simple Echo Bot Application の順に選択します。 ローカルに新しいエコー ボット プロジェクトが作成されます。このプロジェクトは必要に応じて編集できます。 

![.NET ボット テンプレート](media/botbuilder-templates/new-template-dotnet.png)

## <a name="store-your-bot-information-with-msbot"></a>MSBot を使用してボット情報を保存する

新しい [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot) ツールを使用すると、ボットが使用するさまざまなサービスに関するメタデータをすべて 1 か所に保存する **.bot** ファイルを作成することができます。 このファイルを使用すると、CLI からこれらのサービスにボットを接続することもできます。 このツールは、npm モジュールとして使用できます。インストールして実行するには、次のコマンドを使用します。

```shell
npm install -g msbot 
```

ボット ファイルを作成するには、CLI から「**msbot init**」を入力し、続けてボットの名前とターゲット URL エンドポイントを入力します。次に例を示します。

```shell
msbot init --name TestBot --endpoint http://localhost:9499/api/messages
```
ボットをサービスに接続するには、CLI で「**msbot connect**」を入力し、続けて該当するサービスを入力します。

```shell
msbot connect [Service]
```

| Service | 説明 |
| ------ | ----------- |
| azure  |ボットを Azure Bot Service の登録に接続します|
|localhost| ボットを localhost エンドポイントに接続します|
|luis     | ボットを LUIS アプリケーションに接続します |
| qna     |ボットを QnA ナレッジ ベースに接続します|
|help [cmd]  |[cmd] のヘルプを表示します|

### <a name="connect-your-bot-to-abs-with-the-bot-file"></a>.bot ファイルを使用してボットを ABS に接続する

MSBot ツールをインストールすると、az bot **show** コマンドを実行することで、ボットを Azure Bot Service 内の既存のリソース グループに簡単に接続することができます。 

```azurecli
az bot show -n <botname> -g <resourcegroup> --msbot | msbot connect azure --stdin
```

これにより、ターゲット リソース グループから現在のエンドポイント、MSA appID とパスワードが取得され、それに応じて .bot ファイル内の情報が更新されます。 


## <a name="manage-update-or-create-luis-and-qna-services-with--new-botbuilder-tools"></a>新しいボット ビルダー ツールを使用して LUIS および QnA サービスを管理、更新、または作成する

[ボット ビルダー ツール](https://github.com/microsoft/botbuilder-tools)は、コマンド ラインから直接ボットのリソースを管理してやり取りできる新しいツールセットです。 

>[!TIP]
> すべてのボット ビルダー ツールには、グローバル ヘルプ コマンドが含まれており、コマンド ラインから **-h** または **--help** を入力してアクセスできます。 このコマンドは、任意のアクションからいつでも使用できます。実行すると、使用可能なオプションとその説明が表示されます。 

### <a name="ludown"></a>LUDown
[LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Ludown) を使用すると、**.lu** ファイルを使用してボットの強力な言語コンポーネントを記述して作成することができます。 新しい .lu ファイルは、LUDown ツールが使用して、対象サービスに固有の .json ファイルを出力するマークダウン形式の一種です。 現時点では、.lu ファイルを使用して、新しい [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-get-started-create-app) アプリケーションまたは [QnA](https://qnamaker.ai/Documentation/CreateKb) ナレッジ ベースをそれぞれ異なる形式を使用して作成することができます。 LUDown は npm モジュールとして使用でき、お使いのコンピューターにグローバルにインストールすることで使用できます。

```shell
npm install -g ludown
```
LUDown ツールは、LUIS と QnA の両方の新しい .json モデルを作成するために使用できます。  


### <a name="creating-a-luis-application-with-ludown"></a>LUDown を使用して LUIS アプリケーションを作成する

LUIS ポータルから行うのと同じように、LUIS アプリケーションの[意図](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-intents)と[エンティティ](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-entities)を定義することができます。 

'#\<intent-name\>' には、新しい意図の定義セクションを記述します。 その後の各行は、その意図を表す[発話](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-example-utterances)の一覧です。

たとえば、次のように、1 つの .lu ファイルに複数の LUIS 意図を作成できます。 

```LUDown
# Greeting
- Hi
- Hello
- Good morning
- Good evening

# Help
- help
- I need help
- please help
```

### <a name="qna-pairs-with-ludown"></a>LUDown での QnA のペア

.lu ファイル形式では、次の表記を使用して、QnA ペアもサポートしています。 

```LUDown
> comment
### ? question ?
  ```markdown
    answer
  ```

LUDown ツールでは、質問と回答が自動的に QnA Maker の JSON ファイルに分けられます。この JSON ファイルを使用して、新しい [QnaMaker.ai](http://qnamaker.ai) ナレッジ ベースを作成することができます。

```LUDown
### ? How do I change the default message for QnA Maker?
  ```markdown
  You can change the default message if you use the QnAMakerDialog. 
  See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
  ```

また、同じ回答に対して複数の質問を追加することもできます。これは、1 つの回答に対して質問のバリエーションを新しい行で追加するだけでできます。 

```LUDown
### ? What is your name?
- What should I call you?
  ```markdown
    I'm the echoBot! Nice to meet you.
  ```

### <a name="generating-json-models-with-ludown"></a>LUDown を使用して .json モデルを生成する

.lu 形式で LUIS または QnA の言語コンポーネントを定義すると、LUIS .json、QnA .json または QnA .tsv ファイルに発行できます。 LUDown ツールは、実行すると、同じ作業ディレクトリ内のすべての .lu ファイルを検索して解析します。 LUDown ツールは、.lu ファイルを使用することで、LUIS または QnA のどちらもターゲットにできるため、一般的なコマンド **ludown parse <Service> -- in <luFile>** を使用して、どの言語サービス用に生成するかを指定するだけで済みます。 

サンプルの作業ディレクトリには、解析する 2 つの .lu ファイル (LUIS モデルを作成する '1.lu' と QnA ナレッジ ベースを作成する 'qna1.lu') があります。

#### <a name="generate-luis-json-models"></a>LUIS .json モデルを生成する

LUDown を使用して LUIS モデルを生成するには、現在の作業ディレクトリで以下を入力するだけです。

```shell
ludown parse ToLuis --in <luFile> 
```

#### <a name="generate-qna-knowledge-base-json"></a>QnA ナレッジ ベース .json を生成する

同様に、QnA ナレッジ ベースを作成するには、解析の対象を変更するだけです。 

```shell
ludown parse ToQna --in <luFile> 
```

その結果として生成される JSON ファイルは、LUIS または QnA のそれぞれのポータルを通じて、または新しい CLI ツールを使用して使用することができます。 

## <a name="connect-to-luis-an-qna-maker-services-from-the-cli"></a>CLI から LUIS と QnA Maker サービスに接続する

### <a name="connect-to-luis-from-the-cli"></a>CLI から LUIS に接続する 

新しいツール セットには、LUIS リソースを個別に管理できるようにする [LUIS 拡張機能](https://github.com/Microsoft/botbuilder-tools/tree/master/LUIS)が含まれています。 これは、ダウンロード可能な npm モジュールとして使用できます。

```shell
npm install -g luis-apis
```
CLI からの LUIS ツールの基本的なコマンドの使用方法は、次のとおりです。

```shell
luis <action> <resource> <args...>
```
ボットを LUIS に接続するには、**.luisrc** ファイルを作成する必要があります。 これは、アプリケーションが発信呼び出しを行う際に、サービス エンドポイントへの LUIS appID とパスワードをプロビジョニングする構成ファイルです。 このファイルは、次のように **luis init** を実行することで作成できます。

```shell
luis init
```
ツールがファイルを生成する前に、端末で自分の LUIS オーサリング キー、リージョン、appID の入力を求められます。  

![LUIS init](media/bot-builder-tools/luis-init.png) 


このファイルが生成されると、CLI から次のコマンドを使用して、アプリケーションで (LUDown から生成された) LUIS .json ファイルを使用できるようになります。

```shell
luis import application --in luis-app.json | msbot connect luis --stdin
```

### <a name="connect-to-qna-from-the-cli"></a>CLI から QnA に接続する

新しいツール セットには、LUIS リソースを個別に管理できるようにする [QnA 拡張機能](https://github.com/Microsoft/botbuilder-tools/tree/master/QnAMaker)が含まれています。 これは、ダウンロード可能な npm モジュールとして使用できます。

```shell
npm install -g qnamaker
```
QnA Maker ツールを使用すると、ナレッジ ベースの作成、更新、発行、削除、トレーニングができます。 開始するには、サービスにエンドポイントを有効にするために必要な **.qnamakerrc** ファイルを作成する必要があります。 このファイルは、**qnamaker init** を実行し、QnA Maker のナレッジ ベース ID を指定して画面の指示に従うことで、簡単に作成できます。 

```shell
qnamaker init 
```
![QnaMaker init](media/bot-builder-tools/qnamaker-init.png)

.qnamakerrc ファイルが生成されたら、次のコマンドを使用して、自分の QnA ナレッジ ベースに接続し、(LUDown から生成された) ナレッジ ベースの .json/.tsv ファイルを使用することができます。

```shell
qnamaker create --in qnaKB.json --msbot | msbot connect qna --stdin
```

## <a name="references"></a>参照
- [ボット ビルダー ツールのソース コード](https://github.com/Microsoft/botbuilder-tools)
- [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot)
- [ChatDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown)
- [LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Ludown)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
