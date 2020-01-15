---
ms.openlocfilehash: 9a2618533dfefe86be1a15fb5d88740182e04b6d
ms.sourcegitcommit: 46fbb8982144c66864b83889b6457187e890badd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2020
ms.locfileid: "75752433"
---
## <a name="prerequisites"></a>前提条件
- Python [3.6](https://www.python.org/downloads/release/python-369/) または [3.7](https://www.python.org/downloads/release/python-375/)
- [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)
- [git](https://git-scm.com/)
- Python での非同期プログラミングに関する知識

## <a name="create-a-bot"></a>ボットの作成
1. タターミナルを開き、ボットをローカルに保存するフォルダーに移動します。 次のコマンドを実行して、必要なパッケージをインストールします。
- `pip install botbuilder-core`
- `pip install asyncio`
- `pip install -r requirements.txt`
- `pip install cookiecutter`

最後のパッケージである cookiecutter は、ボットの生成に使用されます。 `cookiecutter --help` を実行して、cookiecutter が正しくインストールされたことを確認します。

2. ボットを作成するには、次を実行します。

```cmd
cookiecutter https://github.com/microsoft/botbuilder-python/releases/download/Templates/echo.zip
```

このコマンドは、Python の [echo テンプレート](https://github.com/microsoft/botbuilder-python/tree/master/generators/app/templates/echo)に基づいてエコー ボットを作成します。

3. ボットの "*名前*" と "*説明*" を入力するよう求められます。 次に示すように、ボットに `echo-bot` という名前を付け、説明を `A bot that echoes back user response.` に設定します。

![名前と説明の設定](~/media/python/quickstart/set-name-description.png)

最後の行にあるアドレスの最後の 4桁 (通常は 3978) をコピーします。これは、次の手順で使用します。 これで、ボットを開始する準備ができました。

## <a name="start-you-bot"></a>ボットの開始
1. ターミナルから、ボットを保存した `echo-bot` フォルダーに移動します。 `pip install -r requirements.txt` を実行して、ボットを実行するために必要なパッケージをインストールします。

2. パッケージがインストールされたら、`python app.py` を実行してボットを開始します。 次のスクリーンショットに示されている最後の行が表示されたら、ボットのテストを行う準備ができていることがわかります。

![ローカルで実行されているボット](~/media/python/quickstart/bot-running-locally.png)

## <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続
1. エミュレーターを起動し、 **[Open Bot]\(ボットを開く\)** ボタンをクリックします。

2. このボタンをクリックすると、ボックス ウィンドウが開きます。ここで、ボットを実行するために必要な値を設定します。 前に保存した数字を使用し、次に示すように **[Bot URL]\(ボット URL\)** を `http://localhost:<saved number>/api/messages` に設定します。

![ボットを開く画面](~/media/python/quickstart/open-bot.png)

3. **[Connect]\(接続\)** ボタンをクリックすると、ボットが開始されます。 次に示すように、任意の文字を入力し *Enter* キーをクリックして、ボットをテストしてみてください。

![接続とテスト](~/media/python/quickstart/connect-and-start.png)
