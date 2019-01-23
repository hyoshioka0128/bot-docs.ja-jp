---
ms.openlocfilehash: 2096aa342fe954b9f9f1d128bc080c0e0e6efdce
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360802"
---
## <a name="prerequisites"></a>前提条件
- Visual Studio [2017](https://www.visualstudio.com/downloads)
- [C#](https://aka.ms/bot-vsix) 用 Bot Framework SDK v4 テンプレート
- Bot Framework [Emulator](https://aka.ms/Emulator-wiki-getting-started)
- [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) および [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index) での非同期プログラミングに関する知識

## <a name="create-a-bot"></a>ボットの作成
前提条件セクションでダウンロードした BotBuilderVSIX.vsix テンプレートをインストールします。

Visual Studio で、Bot Framework Echo Bot V4 テンプレートを使用して、新しいボット プロジェクトを作成します。

![Visual Studio プロジェクト](~/media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> 必要な場合は、プロジェクトのビルドの種類を ``.Net Core 2.1`` に変更します。 また、必要に応じて、`Microsoft.Bot.Builder` [NuGet パッケージ](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio)を更新します。

テンプレートのおかげで、プロジェクトには、このクイック スタートでボットを作成するのに必要なすべてのコードが含まれています。 実際には、追加のコードを記述する必要はありません。

## <a name="start-your-bot-in-visual-studio"></a>Visual Studio でのボットの開始

実行ボタンをクリックすると、Visual Studio はアプリケーションを構築し、localhost に配置して、Web ブラウザーを起動してアプリケーションの `default.htm` ページを表示します。 この時点では、ボットはローカルで実行されています。

## <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続

次に、エミュレーターを起動し、エミュレーターのボットに接続します。

1. エミュレーターの [ようこそ] タブにある **[Open Bot]\(ボットを開く\)** リンクをクリックします。 
2. Visual Studio ソリューションを作成したディレクトリにある .bot ファイルを選択します。

## <a name="interact-with-your-bot"></a>ボットでのやり取り

メッセージをボットに送信します。ボットはメッセージで応答します。

![実行中のエミュレーター](~/media/emulator-v4/emulator-running.png)

> [!NOTE]
> メッセージを送信できない場合は、ngrok がシステム上の必要な権限をまだ取得していないため、マシンの再起動が必要になることがあります (この操作は一度だけ実行する必要があります)。
