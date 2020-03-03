---
ms.openlocfilehash: 9c86001a51914f359163e7d755aa57e1c54127f8
ms.sourcegitcommit: dcacda776c927bcc7c76d00ff3cc6b00b062bd6b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/23/2019
ms.locfileid: "74414466"
---
## <a name="prerequisites"></a>前提条件
- [Visual Studio 2017 以降](https://www.visualstudio.com/downloads)
- [C# 用 Bot Framework SDK v4 テンプレート](https://aka.ms/bot-vsix)
- [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)
- [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) および [C# での非同期プログラミング](https://docs.microsoft.com/dotnet/csharp/programming-guide/concepts/async/index)に関する知識

## <a name="create-a-bot"></a>ボットの作成
前提条件セクションでダウンロードした [BotBuilderVSIX.vsix](https://aka.ms/bot-vsix) テンプレートをインストールします。

Visual Studio で、**Echo Bot (Bot Framework v4)** テンプレートを使用して、新しいボット プロジェクトを作成します。 検索ボックスに「_bot framework v4_」と入力して、ボット テンプレートのみを表示します。

![Visual Studio のプロジェクト新規作成ダイアログ](../media/azure-bot-quickstarts/bot-builder-dotnet-project-vs2019.png)

> [!TIP] 
> Visual Studio 2017 を使用している場合は、プロジェクトのビルドの種類が ``.Net Core 2.1`` 以降であることを確認してください。 また、必要に応じて、`Microsoft.Bot.Builder` [NuGet パッケージ](https://docs.microsoft.com/nuget/quickstart/install-and-use-a-package-in-visual-studio)を更新します。

テンプレートのおかげで、プロジェクトには、このクイック スタートでボットを作成するのに必要なすべてのコードが含まれています。 実際には、追加のコードを記述する必要はありません。

## <a name="start-your-bot-in-visual-studio"></a>Visual Studio でのボットの開始

実行ボタンをクリックすると、Visual Studio はアプリケーションを構築し、localhost に配置して、Web ブラウザーを起動してアプリケーションの `default.htm` ページを表示します。 この時点では、ボットはローカルで実行されています。

## <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続

次に、エミュレーターを起動し、エミュレーターのボットに接続します。

1. エミュレーターの [Welcome] タブにある **[Create a new bot configuration]** リンクをクリックします。 
2. ボット用のフィールドに入力します。 ボットのウェルカム ページ アドレス (通常は http://localhost:3978) ) を使用し、このアドレスにルーティング情報 '/api/messages' を追加します。
3. 次に、 **[保存および接続]** をクリックします。

## <a name="interact-with-your-bot"></a>ボットでのやり取り

メッセージをボットに送信します。ボットはメッセージで応答します。

![実行中のエミュレーター](~/media/emulator-v4/emulator-running.png)

> [!NOTE]
> メッセージを送信できない場合は、ngrok がシステム上の必要な権限をまだ取得していないため、マシンの再起動が必要になることがあります (この操作は一度だけ実行する必要があります)。
