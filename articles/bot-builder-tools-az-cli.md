---
title: Azure CLI を使用したボットの作成
description: ボット ビルダー ツールを使用すると、コマンド ラインから直接ボットのリソースを管理できます。
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: tools
ms.date: 10/31/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8a59c0a8b7ee664cdb38ab9d0cb186114938d73f
ms.sourcegitcommit: 782b3a2e788c25effd7d150a070bd2819ea92dad
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/01/2018
ms.locfileid: "50743666"
---
# <a name="create-bots-with-azure-cli"></a>Azure CLI を使用したボットの作成

[!INCLUDE [pre-release-label](./includes/pre-release-label-v3.md)]

このチュートリアルで学習する内容は次のとおりです。 
- Azure CLI を使用して新しいボットを作成する 
- 開発用にローカル コピーをダウンロードする
- 新しい MSBot ツールを使用して、すべてのボット リソース情報を格納する
- LUDown を使用して、LUIS および QnA のモデルを管理、作成、または更新する
- CLI から LUIS と QnA Maker サービスに接続する
- CLI から Azure にボットをデプロイする

## <a name="prerequisites"></a>前提条件

コマンド ラインからこれらのツールを使用するには、お使いのマシンに Node.js がインストールされている必要があります。 
- [Node.js (v8.5 以上)](https://nodejs.org/en/)

## <a name="1-install-tools"></a>1.ツールをインストールする
1. 最新バージョンの Azure CLI を[インストール](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)します。
2. Bot Builder ツールを[インストール](https://aka.ms/botbuilder-tools-readme)します。

他の Azure リソースと同じように、Azure CLI を使用してボットを管理できるようになりました。

>[!TIP]
> Azure Bot Extension では現在、v3 ボットのみをサポートしています。
  
3. 次のコマンドを実行して Azure CLI にログインします。

```azurecli
az login
```
ブラウザー ウィンドウが開いて、サインインできます。 サインインすると、次のメッセージが表示されます。

![MS デバイスのログイン](media/bot-builder-tools/az-browser-login.png)

コマンド ライン ウィンドウに次の情報が表示されます。

![Azure ログイン コマンド](media/bot-builder-tools/az-login-command.png)

## <a name="2-create-a-new-bot-from-azure-cli"></a>2.Azure CLI から新しいボットを作成する

Azure CLI を使用して、コマンド ラインから新しいボットを完全に作成できます。 

```azurecli
az bot [command]
```
|コマンド|  |
|----|----|
| create      |リソースを追加します|
| 削除     |リソースを複製します|
| ダウンロード   | ボットのソース コードをダウンロードします|
| [発行]   |既存のボット サービスに発行します|
| show |既存のボット リソースを表示します|
| update| 既存のボット サービスを更新します|

CLI から新しいボットを作成するには、既存の[リソース グループ](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)を選択するか、新規に作成する必要があります。 

```azurecli
az bot create --resource-group "my-resource-group" --name "my-bot-name" --kind "my-resource-type" --version v3 --description "description-of-my-bot" --lang "programming-language"
```
`--kind` に使用できる値は `function, registration, webapp`、`--version` に使用できる値は `v3, v4` です。  `--lang` 引数を指定しないと、.NET ボットが作成されます。 ノードのボットを作成するには、`Node` を使用します。

要求が成功すると、確認メッセージが表示されます。
```
Obtained msa app id and password. Provisioning bot now.
```

> [!TIP]
> **リソース グループが見つかりませんでした**というエラー メッセージが表示された場合は、Azure CLI で[サブスクリプション](https://docs.microsoft.com/en-us/azure/architecture/cloud-adoption-guide/adoption-intro/subscription-explainer)を設定する必要がある場合があります。 Azure サブスクリプションは、リソース グループの作成時に入力したものと一致している必要があります。 設定するには、次を入力します。
> ```azurecli
> az account set --subscription "your-subscription-name"
> ```
> 自分のアカウントのサブスクリプションのリストを確認するには、次のコマンドを入力します。
> ```azurecli
> az account list
> ```

Azure のリソース グループに新しいエコー ボットがプロビジョニングされます。それをテストするには、Web アプリ ボット ビューのボット管理ヘッダーの下で **[Test in Web Chat]\(Web チャットでのテスト\)** を選択するだけです。 

![Azure のエコー ボット](media/bot-builder-tools/az-echo-bot.png) 

## <a name="3-download-the-bot-locally"></a>手順 3.ボットをローカルにダウンロードする

ソース コードをダウンロードするには、2 とおりの方法があります。
- Azure portal から
- 新しい Azure CLI を使用

[Azure portal](http://portal.azure.com) からボットのソース コードをダウンロードするには、単純にボット リソースを選択して、ボット管理の **[ビルド]** を選択します。 ボットのソース コードをローカルで管理または取得するために使用できる異なるオプションがいくつかあります。

![Azure Portal のボットのダウンロード](media/bot-builder-tools/az-portal-manage-code.png)

CLI を使用してボットのソースをダウンロードするには、次のコマンドを入力します。 ボットはローカルにダウンロードされます。

```azurecli
az bot download --name "my-bot-name" --resource-group "my-resource-group"
```

![CLI のダウンロード コマンド](media/bot-builder-tools/cli-bot-download-command.png)

## <a name="4-store-your-bot-information-with-msbot"></a>4.MSBot を使用してボット情報を保存する

新しい MSBot ツールを使用すると、**.bot ファイル**を作成できます。これにより、ボットが使用するさまざまなサービスに関するすべてのメタデータを 1 か所に格納できます。 このファイルを使用すると、CLI からこれらのサービスにボットを接続することもできます。 MSBot ツールはいくつかのコマンドをサポートしています。詳細については、[readme](https://aka.ms/botbuilder-tools-msbot-readme) ファイルを参照してください。 

MSBot をインストールするには、次のコマンドを実行します。

```shell
npm install -g msbot
```

ボット ファイルを作成するには、CLI から「**msbot init**」を入力し、続けてボットの名前とターゲット URL エンドポイントを入力します。次に例を示します。

```shell
msbot init --name name-of-my-bot --endpoint http://localhost:bot-port-number/api/messages
```

ボットをサービスに接続するには、CLI で「**msbot connect**」を入力し、続けて該当するサービスを入力します。

```shell
msbot connect service-type
```

| サービスの種類 | 説明 |
| ------ | ----------- |
| azure  |ボットを Azure Bot Service の登録に接続します|
|endpoint| ボットを localhost などのエンドポイントに接続します|
|luis     | ボットを LUIS アプリケーションに接続します |
| qna     |ボットを QnA ナレッジ ベースに接続します|
|help [cmd]  |[cmd] のヘルプを表示します|

サポートされているサービスの完全な一覧については、[readme](https://aka.ms/botbuilder-tools-msbot-readme) ファイルを参照してください。

### <a name="connect-your-bot-to-abs-with-the-bot-file"></a>.bot ファイルを使用してボットを ABS に接続する

MSBot ツールをインストールすると、az bot **show** コマンドを実行することで、ボットを Azure Bot Service 内の既存のリソース グループに簡単に接続することができます。

```azurecli
az bot show -n my-bot-name -g my-resource-group --msbot | msbot connect azure --stdin
```

これにより、ターゲット リソース グループから現在のエンドポイント、MSA appID とパスワードが取得され、それに応じて .bot ファイル内の情報が更新されます。


## <a name="5-manage-update-or-create-luis-and-qna-services-with--new-botbuilder-tools"></a>5.新しいボット ビルダー ツールを使用して LUIS および QnA サービスを管理、更新、または作成する

[ボット ビルダー ツール](https://aka.ms/botbuilder-tools)は、コマンド ラインから直接ボットのリソースを管理してやり取りできる新しいツールセットです。

>[!TIP]
> すべてのボット ビルダー ツールには、グローバル ヘルプ コマンドが含まれており、コマンド ラインから **-h** または **--help** を入力してアクセスできます。 このコマンドは、任意のアクションからいつでも使用できます。実行すると、使用可能なオプションとその説明が表示されます。

### <a name="ludown"></a>LUDown

[LUDown](https://aka.ms/botbuilder-ludown) を使用すると、**.lu** ファイルを使用してボットの強力な言語コンポーネントを記述して作成することができます。 新しい .lu ファイルは、LUDown ツールが使用して、対象サービスに固有の .json ファイルを出力するマークダウン形式の一種です。 現時点では、.lu ファイルを使用して、新しい [LUIS](https://docs.microsoft.com/azure/cognitive-services/luis/luis-get-started-create-app) アプリケーションまたは [QnA](https://qnamaker.ai/Documentation/CreateKb) ナレッジ ベースをそれぞれ異なる形式を使用して作成することができます。 LUDown は npm モジュールとして使用でき、お使いのコンピューターにグローバルにインストールすることで使用できます。

```shell
npm install -g ludown
```

LUDown ツールは、LUIS と QnA の両方の新しい .json モデルを作成するために使用できます。  

### <a name="creating-a-luis-application-with-ludown"></a>LUDown を使用して LUIS アプリケーションを作成する

LUIS ポータルから行うのと同じように、LUIS アプリケーションの[意図](https://docs.microsoft.com/azure/cognitive-services/luis/add-intents)と[エンティティ](https://docs.microsoft.com/azure/cognitive-services/luis/add-entities)を定義することができます。

`# \<intent-name\>` は、新しい意図の定義セクションを説明します。 それ以降の行には、その意図を説明する[発話](https://docs.microsoft.com/azure/cognitive-services/luis/add-example-utterances)が含まれます。

たとえば、次のように、1 つの .lu ファイルに複数の LUIS 意図を作成できます。 

```ludown
# Greeting
Hi
Hello
Good morning
Good evening

# Help
help
I need help
please help
```

### <a name="qna-pairs-with-ludown"></a>LUDown での QnA のペア

.lu ファイル形式では、次の表記を使用して、QnA ペアもサポートしています。 

  ```ludown
  > This is a comment. QnA definitions have the general format:
  ### ? this-is-the-question-string
  - this-is-an-alternate-form-of-the-same-question
  - this-is-another-one
    ```markdown
    this-is-the-answer
    ```
  ```
LUDown ツールでは、質問と回答が自動的に QnA Maker の JSON ファイルに分けられます。この JSON ファイルを使用して、新しい [QnaMaker.ai](http://qnamaker.ai) ナレッジ ベースを作成することができます。

  ```ludown
  ### ? How do I change the default message for QnA Maker?
    ```markdown
    You can change the default message if you use the QnAMakerDialog. 
    See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
    ```
  ```

また、同じ回答に対して複数の質問を追加することもできます。これは、1 つの回答に対して質問のバリエーションを新しい行で追加するだけでできます。 

  ```ludown
  ### ? What is your name?
  - What should I call you?
    ```markdown
    I'm the echoBot! Nice to meet you.
    ```
  ```

### <a name="generating-json-models-with-ludown"></a>LUDown を使用して .json モデルを生成する

.lu 形式で LUIS または QnA の言語コンポーネントを定義すると、LUIS .json、QnA .json または QnA .tsv ファイルに発行できます。 LUDown ツールは、実行すると、同じ作業ディレクトリ内のすべての .lu ファイルを検索して解析します。 LUDown ツールは、.lu ファイルを使用することで、LUIS または QnA のどちらもターゲットにできるため、一般的なコマンド **ludown parse <Service> --in <luFile>** を使用して、どの言語サービス用に生成するかを指定するだけで済みます。 

サンプルの作業ディレクトリには、解析する 2 つの .lu ファイル (LUIS モデルを作成する 'luis-sample.lu' と QnA ナレッジ ベースを作成する ' qna-sample.lu') があります。


#### <a name="generate-luis-json-models"></a>LUIS .json モデルを生成する

**luis sample.lu** 
```ludown
# Greeting
- Hi
- Hello
- Good morning
- Good evening
```

LUDown を使用して LUIS モデルを生成するには、現在の作業ディレクトリで以下を入力するだけです。

```shell
ludown parse ToLuis --in ludown-file-name.lu
```

#### <a name="generate-qna-knowledge-base-json"></a>QnA ナレッジ ベース .json を生成する

**qna-sample.lu**
  ```ludown
  > This is a sample ludown file for QnA Maker.

  ### ? How do I change the default message
    ```markdown
    You can change the default message if you use the QnAMakerDialog. 
    See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
    ```
  ```

同様に、QnA ナレッジ ベースを作成するには、解析の対象を変更するだけです。 

```shell
ludown parse ToQna --in ludown-file-name.lu
```

その結果として生成される JSON ファイルは、LUIS または QnA のそれぞれのポータルを通じて、または新しい CLI ツールを使用して使用することができます。 

## <a name="6-connect-to-luis-an-qna-maker-services-from-the-cli"></a>6.CLI から LUIS と QnA Maker サービスに接続する

### <a name="connect-to-luis-from-the-cli"></a>CLI から LUIS に接続する 

新しいツール セットには、LUIS リソースを個別に管理できるようにする [LUIS 拡張機能](https://aka.ms/botbuilder-luis-cli)が含まれています。 これは、ダウンロード可能な npm モジュールとして使用できます。

```shell
npm install -g luis-apis
```
CLI からの LUIS ツールの基本的なコマンドの使用方法は、次のとおりです。

```shell
luis action-name resource-name arguments-list
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

新しいツール セットには、LUIS リソースを個別に管理できるようにする [QnA 拡張機能](https://aka.ms/botbuilder-tools-qnaMaker)が含まれています。 これは、ダウンロード可能な npm モジュールとして使用できます。

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

## <a name="7-publish-to-azure-from-the-cli"></a>7.CLI から Azure に発行する

ボットのソース コードを変更したら、次のコマンドを実行して、変更内容をシームレスに発行できます。

```azurecli
az bot publish --name "my-bot-name" --resource-group "my-resource-group"
```

## <a name="references"></a>参照
- [Bot Builder ツール](https://aka.ms/botbuilder-tools-readme)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
