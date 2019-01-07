---
title: Bot Builder SDK for .NET を使用したボットの作成 | Microsoft Docs
description: ボットの強力な構築フレームワークである Bot Builder SDK for .NET を使用してボットを作成します。
keywords: Bot Builder SDK, ボットの作成, クイック スタート, .NET, 使用の開始, C# ボット
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/19/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d40b203ccd044992c026a592d5f86b0881754a41
ms.sourcegitcommit: f7a8f05fc05ff4a7212a437d540485bf68831604
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/21/2018
ms.locfileid: "53735922"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-net"></a>Bot Builder SDK for .NET を使用したボットの作成
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

このクイック スタートでは、C# テンプレートを使用してボットを構築し、Bot Framework Emulator でテストします。

## <a name="prerequisites"></a>前提条件
- Visual Studio [2017](https://www.visualstudio.com/downloads)
- [C#](https://aka.ms/bot-vsix) 用 Bot Builder SDK v4 テンプレート
- Bot Framework [Emulator](https://aka.ms/Emulator-wiki-getting-started)
- [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) および [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index) での非同期プログラミングに関する知識

## <a name="create-a-bot"></a>ボットの作成
前提条件セクションでダウンロードした BotBuilderVSIX.vsix テンプレートをインストールします。

Visual Studio で、Bot Builder エコー ボット V4 テンプレートを使用して、新しいボット プロジェクトを作成します。

![Visual Studio プロジェクト](../media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> 必要に応じて、プロジェクトのビルドの種類を ``.Net Core 2.1`` に変更します。必要に応じて、[NuGet パッケージ](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio)を更新します。

テンプレートのおかげで、プロジェクトには、このクイック スタートでボットを作成するのに必要なすべてのコードが含まれています。 実際には、追加のコードを記述する必要はありません。

## <a name="start-your-bot-in-visual-studio"></a>Visual Studio でのボットの開始

実行ボタンをクリックすると、Visual Studio はアプリケーションを構築し、localhost に配置して、Web ブラウザーを起動してアプリケーションの `default.htm` ページを表示します。 この時点では、ボットはローカルで実行されています。

## <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続

次に、エミュレーターを起動し、エミュレーターのボットに接続します。

1. エミュレーターの [ようこそ] タブにある **[Open Bot]\(ボットを開く\)** リンクをクリックします。 
2. Visual Studio ソリューションを作成したディレクトリにある .bot ファイルを選択します。

## <a name="interact-with-your-bot"></a>ボットでのやり取り

メッセージをボットに送信します。ボットはメッセージで応答します。

![実行中のエミュレーター](../media/emulator-v4/emulator-running.png)

> [!NOTE]
> メッセージを送信できない場合は、ngrok がシステム上の必要な権限をまだ取得していないため、マシンの再起動が必要になることがあります (この操作は一度だけ実行する必要があります)。

## <a name="additional-resources"></a>その他のリソース

リモートでホストされているボットに接続する方法については、[トンネリング (ngrok)](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-(ngrok)) に関する記事を参照してください。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [ボットのしくみ](../v4sdk/bot-builder-basics.md) 
