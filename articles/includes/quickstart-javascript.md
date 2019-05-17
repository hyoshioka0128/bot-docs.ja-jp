---
ms.openlocfilehash: ee2c261f6b028542b20771801b114d0b23ab21ca
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033204"
---
## <a name="prerequisites"></a>前提条件

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.js](https://nodejs.org/)
- [Yeoman](http://yeoman.io/) (ジェネレーターを使用してボットを作成します)
- [git](https://git-scm.com/)
- [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)
- [restify](http://restify.com/) および JavaScript での非同期プログラミングに関する知識

> [!NOTE]
> 下記の Windows ビルド ツールのインストールは、開発用オペレーティング システムとして Windows を使用している場合にのみ必要となります。 一部のインストールでは、restify のインストール手順を実行すると、node-gyp に関するエラーが発生します。
> 該当する場合は、管理者特権のアクセス許可を使用してこのコマンドを実行してみてください。
> お使いのシステムに Python が既にインストールされている場合、この呼び出しは終了せずにハングすることもあります。

> ```bash
> npm install -g windows-build-tools
> ```

## <a name="create-a-bot"></a>ボットの作成

ボットを作成してパッケージを初期化するには

1. ターミナルまたは管理者特権でのコマンド プロンプトを開きます。
1. JavaScript ボット用のディレクトリがまだない場合は、ディレクトリを 1 つ作成し、そのディレクトリに変更します  (このチュートリアルではボットを 1 つだけ作成しますが、通常は JavaScript ボット用のディレクトリを作成します)。

   ```bash
   mkdir myJsBots
   cd myJsBots
   ```

1. お使いの npm が最新バージョンであることを確認します。

   ```bash
   npm install -g npm
   ```

1. 次に、Yeoman と JavaScript のジェネレーターをインストールします。

   ```bash
   npm install -g yo generator-botbuilder
   ```

1. 次に、ジェネレーターを使用してエコー ボットを作成します。

   ```bash
   yo botbuilder
   ```

Yeoman により、作成するボットに関する情報の入力が求められます。 このチュートリアルでは、既定値を使います。

- ボットの名前を入力します。 (myChatBot)
- 説明を入力します。 (Microsoft Bot Framework の主要な機能を示します)
- ボットの言語を選択します。 (JavaScript)
- 使用するテンプレートを選択します。 (Echo)

テンプレートのおかげで、プロジェクトには、このクイック スタートでボットを作成するのに必要なすべてのコードが含まれています。 実際には、追加のコードを記述する必要はありません。

> [!NOTE]
> `Basic` ボットを作成する場合は、LUIS 言語モデルが必要です。 これは [luis.ai](https://www.luis.ai) で作成できます。 モデルの作成後、構成ファイルを更新します。

## <a name="start-your-bot"></a>ボットの起動

ターミナルまたはコマンド プロンプトで、ボット用に作成したディレクトリに変更し、`npm start` でボットを起動します。 この時点では、ボットはローカルで実行されています。

## <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続

1. Bot Framework Emulator を起動します。
2. エミュレーターの [ようこそ] タブにある **[新しいボット構成を作成する]** リンクをクリックします。 
3. ご自身のボットのフィールドに入力し、**[保存および接続]** をクリックします。

メッセージをボットに送信します。ボットはメッセージで応答します。
![実行中のエミュレーター](../media/emulator-v4/js-quickstart.png)
