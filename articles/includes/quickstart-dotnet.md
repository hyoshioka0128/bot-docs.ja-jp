---
ms.openlocfilehash: 0570ec6a44c9fe1b007c1fd1b8c335288baa63cb
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230672"
---
## <a name="prerequisites"></a>前提条件
- [Visual Studio 2017 以降](https://www.visualstudio.com/downloads)
- [C# 用 Bot Framework SDK v4 テンプレート](https://aka.ms/bot-vsix)
- [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)
- [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) および [C# での非同期プログラミング](https://docs.microsoft.com/dotnet/csharp/programming-guide/concepts/async/index)に関する知識

## <a name="create-a-bot"></a>ボットの作成
前提条件セクションでダウンロードした BotBuilderVSIX.vsix テンプレートをインストールします。

Visual Studio で、**Echo Bot (Bot Framework v4)** テンプレートを使用して、新しいボット プロジェクトを作成します。

![Visual Studio プロジェクト](~/media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> 必要な場合は、プロジェクトのビルドの種類を ``.Net Core 2.1`` に変更します。 また、必要に応じて、`Microsoft.Bot.Builder` [NuGet パッケージ](https://docs.microsoft.com/nuget/quickstart/install-and-use-a-package-in-visual-studio)を更新します。

テンプレートのおかげで、プロジェクトには、このクイック スタートでボットを作成するのに必要なすべてのコードが含まれています。 実際には、追加のコードを記述する必要はありません。

## <a name="start-your-bot-in-visual-studio"></a>Visual Studio でのボットの開始

実行ボタンをクリックすると、Visual Studio はアプリケーションを構築し、localhost に配置して、Web ブラウザーを起動してアプリケーションの `default.htm` ページを表示します。 この時点では、ボットはローカルで実行されています。

## <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続

次に、エミュレーターを起動し、エミュレーターのボットに接続します。

1. エミュレーターの [ようこそ] タブにある **[新しいボット構成を作成する]** リンクをクリックします。 
2. ボット用のフィールドに入力します。 ボットのウェルカム ページ アドレス (通常は http://localhost:3978) ) を使用し、このアドレスにルーティング情報 '/api/messages' を追加します。
3. 次に、 **[保存および接続]** をクリックします。

## <a name="interact-with-your-bot"></a>ボットでのやり取り

メッセージをボットに送信します。ボットはメッセージで応答します。

![実行中のエミュレーター](~/media/emulator-v4/emulator-running.png)

> [!NOTE]
> メッセージを送信できない場合は、ngrok がシステム上の必要な権限をまだ取得していないため、マシンの再起動が必要になることがあります (この操作は一度だけ実行する必要があります)。
