---
title: Azure CLI を使用したボットの作成
description: ボット ビルダー ツールを使用すると、コマンド ラインから直接ボットのリソースを管理できます。
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/25/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1eb47e76ef1bd6765d5ba93c27b97a8d9e6143db
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905306"
---
# <a name="create-bots-with-azure-cli"></a>Azure CLI を使用したボットの作成

[!INCLUDE [pre-release-label](./includes/pre-release-label-v3.md)]

[ボット ビルダー ツール](https://github.com/microsoft/botbuilder-tools)は、コマンド ラインから直接ボットのリソースを管理してやり取りできる新しいツールセットです。 

このチュートリアルでは、次の操作方法について説明します。

- Azure CLI のボット拡張機能を有効にする
- Azure CLI を使用して新しいボットを作成する 
- 開発用にローカル コピーをダウンロードする
- 新しい MSBot ツールを使用して、すべてのボット リソース情報を格納する
- LUDown を使用して、LUIS および QnA のモデルを管理、作成、または更新する
- CLI から LUIS と QnA Maker サービスに接続する
- CLI から Azure にボットをデプロイする

## <a name="prerequisites"></a>前提条件

コマンド ラインからこれらのツールを有効にするには、お使いのコンピューターに Node.js がインストールされている必要があります。 

- [Node.js (v8.5 以上)](https://nodejs.org/en/)

## <a name="1-enable-azure-cli"></a>1.Azure CLI を有効にする

他の Azure リソースと同じように、[Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest) を使用してボットを管理できるようになりました。 Azure CLI を有効にするには、次の手順を完了します。

1. Azure CLI をまだお持ちでない場合は、[ダウンロード](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)します。 

2. 次のコマンドを入力して、Azure Bot Extension 配布パッケージをダウンロードします。

```azurecli
az extension add -n botservice
```

>[!TIP]
> Azure Bot Extension では現在、v3 ボットのみをサポートしています。
  
3. 次のコマンドを実行して Azure CLI に[ログイン](https://docs.microsoft.com/en-us/cli/azure/authenticate-azure-cli?view=azure-cli-latest)します。

```azurecli
az login
```
一意の一時的な認証コードの入力が求められます。 サインインするには、Web ブラウザーを使用して Microsoft [デバイスのログイン](https://microsoft.com/devicelogin)にアクセスし、CLI から提供されたコードを貼り付けて続行します。 

![MS デバイスのログイン](media/bot-builder-tools/ms-device-login.png)

正常にログインすると、アカウントとリソースの管理に使用できるオプションの一覧と共に、Azure CLI のようこそ画面が表示されます。

![Azure Bot CLI](media/bot-builder-tools/az-cli-bot.png)


 Azure CLI コマンドの完全な一覧については、[こちら](https://docs.microsoft.com/cli/azure/reference-index?view=azure-cli-latest)をクリックしてください。


## <a name="2-create-a-new-bot-from-azure-cli"></a>2.Azure CLI から新しいボットを作成する

Azure CLI と新しいボット拡張機能を使用すると、コマンド ラインから新しいボットを完全に作成できます。 

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
az bot create --resource-group "my-resource-group" --name "my-bot-name" --kind "my-resource-type" --description "description-of-my-bot"
```
要求が成功すると、確認メッセージが表示されます。
```
obtained msa app id and password. Provisioning bot now.
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

既定では、新しい .NET ボットが作成されます。 **--lang** 引数を使用して言語を指定することで、プラットフォーム SDK を指定できます。 現時点では、ボットの拡張機能パッケージでは、C# および Node.js ボットの SDK がサポートされています。 たとえば、**Node.js ボットを作成**するには、次のようにします。

```azurecli
az bot create --resource-group "my-resource-group" --name "my-bot-name" --kind "my-resource-type" --description "description-of-my-bot" --lang Node 
```
Azure のリソース グループに新しいエコー ボットがプロビジョニングされます。それをテストするには、Web アプリ ボット ビューのボット管理ヘッダーの下で **[Test in Web Chat]\(Web チャットでのテスト\)** を選択するだけです。 

![Azure のエコー ボット](media/bot-builder-tools/az-echo-bot.png) 

## <a name="3-download-the-bot-locally"></a>手順 3.ボットをローカルにダウンロードする

新しいボットのソース コードをダウンロードするには、2 つの方法があります。
- Azure Portal からダウンロードする。
- 新しい Azure CLI を使用してダウンロードする。

ポータルからボットのソース コードをダウンロードするには、単純にボット リソースを選択して、[Bot management]\(ボット管理\) の下で **[ビルド]** を選択します。 ボットのソース コードをローカルで管理または取得するために使用できる異なるオプションがいくつかあります。 

![Azure Portal のボットのダウンロード](media/bot-builder-tools/az-portal-manage-code.png)

CLI を使用してボットのソースをダウンロードするには、次のコマンドを入力します。 ボットは、サブディレクトリにダウンロードされます。 サブディレクトリがまだない場合は、このコマンドにより作成されます。

```azurecli
az bot download --name "my-bot-name" --resource-group "my-resource-group"
```
ただし、ボットをダウンロードするディレクトリを指定することもできます。
例: 

![CLI のダウンロード コマンド](media/bot-builder-tools/cli-bot-download-command.png)

![CLI のボットのダウンロード](media/bot-builder-tools/cli-bot-download.png)

上記のコマンドを使用すると、ボットのソース コードを指定した場所に直接ダウンロードすることができ、ローカルでボットの開発ができるようになります。


## <a name="4-store-your-bot-information-with-msbot"></a>4.MSBot を使用してボット情報を保存する

新しい [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot) ツールを使用すると、ボットが使用するさまざまなサービスに関するメタデータをすべて 1 か所に保存する **.bot** ファイルを作成することができます。 このファイルを使用すると、CLI からこれらのサービスにボットを接続することもできます。 このツールは、npm モジュールとして使用できます。インストールして実行するには、次のコマンドを使用します。

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

### <a name="connect-your-bot-to-abs-with-the-bot-file"></a>.bot ファイルを使用してボットを ABS に接続する

MSBot ツールをインストールすると、az bot **show** コマンドを実行することで、ボットを Azure Bot Service 内の既存のリソース グループに簡単に接続することができます。 

```azurecli
az bot show -n my-bot-name -g my-resource-group --msbot | msbot connect azure --stdin
```

これにより、ターゲット リソース グループから現在のエンドポイント、MSA appID とパスワードが取得され、それに応じて .bot ファイル内の情報が更新されます。 


## <a name="5-manage-update-or-create-luis-and-qna-services-with--new-botbuilder-tools"></a>5.新しいボット ビルダー ツールを使用して LUIS および QnA サービスを管理、更新、または作成する

[ボット ビルダー ツール](https://github.com/microsoft/botbuilder-tools)は、コマンド ラインから直接ボットのリソースを管理してやり取りできる新しいツールセットです。 

>[!TIP]
> すべてのボット ビルダー ツールには、グローバル ヘルプ コマンドが含まれており、コマンド ラインから **-h** または **--help** を入力してアクセスできます。 このコマンドは、任意のアクションからいつでも使用できます。実行すると、使用可能なオプションとその説明が表示されます。

### <a name="ludown"></a>LUDown
[LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Ludown) を使用すると、**.lu** ファイルを使用してボットの強力な言語コンポーネントを記述して作成することができます。 新しい .lu ファイルは、LUDown ツールが使用して、対象サービスに固有の .json ファイルを出力するマークダウン形式の一種です。 現時点では、.lu ファイルを使用して、新しい [LUIS](https://docs.microsoft.com/azure/cognitive-services/luis/luis-get-started-create-app) アプリケーションまたは [QnA](https://qnamaker.ai/Documentation/CreateKb) ナレッジ ベースをそれぞれ異なる形式を使用して作成することができます。 LUDown は npm モジュールとして使用でき、お使いのコンピューターにグローバルにインストールすることで使用できます。

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

新しいツール セットには、LUIS リソースを個別に管理できるようにする [LUIS 拡張機能](https://github.com/Microsoft/botbuilder-tools/tree/master/LUIS)が含まれています。 これは、ダウンロード可能な npm モジュールとして使用できます。

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

## <a name="7-publish-to-azure-from-the-cli"></a>7.CLI から Azure に発行する

ボットのソース コードを変更したら、次のコマンドを実行して、変更内容をシームレスに発行できます。

```azurecli
az bot publish --name "my-bot-name" --resource-group "my-resource-group"
```

## <a name="references"></a>参照
- [ボット ビルダー ツールのソース コード](https://github.com/Microsoft/botbuilder-tools)
- [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot)
- [ChatDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown)
- [LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/ludown)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)


