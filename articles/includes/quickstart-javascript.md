---
ms.openlocfilehash: 77635aab0e1535a2d44ef24c3e33094b13f4a15a
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "80117832"
---
## <a name="prerequisites"></a>前提条件

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.js](https://nodejs.org/)
- [Yeoman](http://yeoman.io/) (ジェネレーターを使用してボットを作成します)
- [git](https://git-scm.com/)
- [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)
- [restify](http://restify.com/) および JavaScript での非同期プログラミングに関する知識

> [!NOTE]
> 下記の Windows ビルド ツールのインストールは、開発用オペレーティング システムとして Windows を使用している場合にのみ必要となります。
> 一部のインストールでは、restify のインストール手順を実行すると、node-gyp に関するエラーが発生します。
> 該当する場合は、管理者特権のアクセス許可を使用してこのコマンドを実行してみてください。
> お使いのシステムに Python が既にインストールされている場合、この呼び出しは終了せずにハングすることもあります。

> ```bash
> # only run this command if you are on Windows. Read the above note. 
> npm install -g windows-build-tools
> ```

## <a name="create-a-bot"></a>ボットの作成

ボットを作成してパッケージを初期化するには

1. ターミナルまたは管理者特権でのコマンド プロンプトを開きます。
1. JavaScript ボット用のディレクトリがまだない場合は、ディレクトリを 1 つ作成し、そのディレクトリに変更します (このチュートリアルではボットを 1 つだけ作成しますが、通常は JavaScript ボット用のディレクトリを作成します)。

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

```text
? What's the name of your bot? my-chat-bot
? What will your bot do? Demonstrate the core capabilities of the Microsoft Bot Framework
? What programming language do you want to use? JavaScript
? Which template would you like to start with? Echo Bot - https://aka.ms/bot-template-echo
? Looking good.  Shall I go ahead and create your new bot? (Y/n)
```

テンプレートのおかげで、プロジェクトには、このクイック スタートでボットを作成するのに必要なすべてのコードが含まれています。 実際には、追加のコードを記述する必要はありません。

> [!NOTE]
> `Core` ボットを作成する場合は、LUIS 言語モデルが必要です。 これは [luis.ai](https://www.luis.ai) で作成できます。 モデルの作成後、構成ファイルを更新します。

## <a name="start-your-bot"></a>ボットの起動

ターミナルまたはコマンド プロンプトで、ボット用に作成したディレクトリに変更し、`npm start` でボットを起動します。 この時点では、ボットはローカルで実行されています。

## <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続

1. Bot Framework Emulator を起動します。
2. エミュレーターの [Welcome] タブにある **[Create a new bot configuration]** リンクをクリックします。 
3. ボット用のフィールドに入力します。 ボットのウェルカム ページ アドレス (通常は http://localhost:3978) ) を使用し、このアドレスにルーティング情報 '/api/messages' を追加します。
4. 次に、 **[Save and connect]** をクリックします。

メッセージをボットに送信します。ボットはメッセージで応答します。
![実行中のエミュレーター](../media/emulator-v4/js-quickstart.png)
