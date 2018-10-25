---
title: Bot Builder SDK for JavaScript を使用してボットを作成する | Microsoft Docs
description: Bot Builder SDK for JavaScript を使用してボットをすばやく作成します。
keywords: クイック スタート, Bot Builder SDK, 使用の開始
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3e721c9142ffb1511ef926f5b1caca782e0919ed
ms.sourcegitcommit: 54ed5000c67a5b59e23b667547565dd96c7302f9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/13/2018
ms.locfileid: "49315088"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-javascript"></a>Bot Builder SDK for JavaScript を使用したボットの作成

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

このクイック スタートでは、Yeoman Bot Builder ジェネレーターと Bot Builder SDK for JavaScript を使用してボットを構築し、Bot Framework Emulator でテストする方法を説明します。 

## <a name="prerequisites"></a>前提条件

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.js](https://nodejs.org/)
- [Yeoman](http://yeoman.io/) (ジェネレーターを使用してボットを作成できます)
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator)
- [restify](http://restify.com/) および JavaScript での非同期プログラミングに関する知識

> [!NOTE]
> 一部のインストールでは、restify のインストール手順を実行すると、node-gyp に関するエラーが発生します。
> この場合は、`npm install -g windows-build-tools` を実行してみてください。

## <a name="create-a-bot"></a>ボットの作成

管理者特権でのコマンド プロンプトを開き、ディレクトリを作成し、自分のボット用にパッケージを初期化します。

```bash
md myJsBots
cd myJsBots
```

お使いの npm が最新バージョンであることを確認します。
```bash
npm i npm
```

次に、Yeoman と JavaScript のジェネレーターをインストールします。

```bash
npm install -g yo
npm install -g generator-botbuilder
```

次に、ジェネレーターを使用してエコー ボットを作成します。

```bash
yo botbuilder
```

Yeoman により、作成するボットに関する情報の入力が求められます。

- ボットの名前を入力します。
- 説明を入力します。
- ボットの言語を `JavaScript` または `TypeScript` から選択します。
- `Echo` テンプレートを選択します。

テンプレートのおかげで、プロジェクトには、このクイック スタートでボットを作成するのに必要なすべてのコードが含まれています。 実際には、追加のコードを記述する必要はありません。

> [!NOTE]
> Basic ボットでは、LUIS 言語モデルが必要です。 これは [luis.ai](https://www.luis.ai) で作成できます。 モデルの作成後、.bot ファイルを更新します。 完成したボット ファイルの例については、[こちら](../v4sdk/bot-builder-service-file.md)を参照してください。 

## <a name="start-your-bot"></a>ボットの起動

PowerShell/Bash で、ボット用に作成したディレクトリに移動し、`npm start` でボットを起動します。 この時点では、ボットはローカルで実行されています。

## <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続
1. エミュレーターを起動します。
2. エミュレーターの [ようこそ] タブにある **[Open Bot]\(ボットを開く\)** リンクをクリックします。
3. プロジェクトを作成したディレクトリにある .bot ファイルを選択します。

メッセージをボットに送信します。ボットはメッセージで応答します。
![実行中のエミュレーター](../media/emulator-v4/emulator-running.png)

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [ボットのしくみ](../v4sdk/bot-builder-basics.md) 
