---
title: Bot Builder SDK for .NET を使用したボットの作成 | Microsoft Docs
description: ボットの強力な構築フレームワークである Bot Builder SDK for .NET を使用してボットを作成します。
keywords: Bot Builder SDK, ボットの作成, クイック スタート, .NET, 使用の開始, C# ボット
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 09/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5f3a02783242697fccf267bef2d56ed453880c67
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707978"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-net"></a>Bot Builder SDK for .NET を使用したボットの作成
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

このクイック スタートでは、C# テンプレートを使用してボットを構築し、Bot Framework Emulator でテストします。 

## <a name="prerequisites"></a>前提条件
- Visual Studio [2017](https://www.visualstudio.com/downloads)
- [C#](https://botbuilder.myget.org/feed/aitemplates/package/vsix/BotBuilderV4.fbe0fc50-a6f1-4500-82a2-189314b7bea2) 用 Bot Builder SDK v4 テンプレート
- Bot Framework [Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)
- [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) および [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index) での非同期プログラミングに関する知識

## <a name="create-a-bot"></a>ボットの作成
前提条件セクションでダウンロードした BotBuilderVSIX.vsix テンプレートをインストールします。 

Visual Studio で、新しいボット プロジェクトを作成します。

![Visual Studio プロジェクト](../media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> 必要に応じて、[NuGet パッケージ](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio)を更新します。

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

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [ボットのしくみ](../v4sdk/bot-builder-basics.md) 
