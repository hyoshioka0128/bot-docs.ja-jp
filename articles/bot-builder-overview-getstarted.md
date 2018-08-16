---
title: Bot Builder によるボット作成の開始 | Microsoft Docs
description: Bot Builder を使用して強力なボットの作成作業を開始します。
author: robstand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 21186c5d3b0769311e4703ca1dab2f48a0a0081a
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574888"
---
# <a name="develop-bots-with-bot-builder"></a>Bot Builder を使用したボットの開発

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Bot Builder には、SDK、ライブラリ、サンプルと、ボットの構築およびデバッグに役立つツールが用意されています。 Bot Service を使用してボットを構築する際、そのボットは Bot Builder SDK によって支援されます。 Bot Builder SDK を使用すると、C# または Node.js を使ってゼロからボットを作成することもできます。 Bot Builder には、ボットをテストする Bot Framework Emulator や、さまざまなチャネルでボットのユーザー エクスペリエンスをプレビューする Channel Inspector が含まれます。

必要な任意のプログラミング言語を使用してボットをビルドする場合は、REST API を使用できます。

## <a name="bot-builder-sdk-for-net"></a>Bot Builder SDK for .NET
Bot Builder SDK for .NET では、C# を活用して、.NET 開発者がボットを作成するための手慣れた方法を提供しています。 これは、自由形式の対話と、ユーザーが使用可能な値の中から選択するガイド付きの会話の両方に対応できるボットを作成するための強力なフレームワークです。 

[.NET クイック スタート](~/dotnet/bot-builder-dotnet-quickstart.md)では、Bot Builder SDK for .NET を使用してボットを作成する手順を案内しています。

Bot Builder SDK for .NET は、[NuGet パッケージ](https://www.nuget.org/packages/Microsoft.Bot.Builder/)として入手できます。

> [!IMPORTANT]
> Bot Builder SDK for .NET には、.NET Framework 4.6 以降が必要です。 .NET Framework の古いバージョンを対象にしている既存のプロジェクトに SDK を追加する場合は、まず、.NET Framework 4.6 を対象とするようにプロジェクトを更新する必要があります。

既存の Visual Studio プロジェクトに SDK をインストールするには、次の手順を完了します。

1. **ソリューション エクスプローラー**でプロジェクト名を右クリックし、**[NuGet パッケージの管理...]** を選択します。
2. **[参照]** タブで、検索ボックスに「Microsoft.Bot.Builder」と入力します。
3. 結果の一覧で **[Microsoft.Bot.Builder]** を選択し、**[インストール]** をクリックして、変更を確定ます。

GitHub から Bot Builder SDK for .NET の[ソース コード](https://github.com/Microsoft/BotBuilder/tree/master/CSharp)をダウンロードすることもできます。

### <a name="visual-studio-project-templates"></a>Visual Studio プロジェクト テンプレート
Visual Studio プロジェクト テンプレートをダウンロードして、ボット開発を加速します。

* [Visual Studio 用のボット テンプレート][bot-template] (C# によるボット開発用)
* [Visual Studio 用の Cortana のスキル テンプレート][cortana-template] (C# を使用した Cortana スキル開発用)

> [!TIP]
> Visual Studio 2017 プロジェクト テンプレートのインストール方法については、<a href="/visualstudio/ide/how-to-locate-and-organize-project-and-item-templates" target="_blank">こちら</a>を参照してください。

## <a name="bot-builder-sdk-for-nodejs"></a>Bot Builder SDK for Node.js
Bot Builder SDK for Node.js では、Node.js 開発者がボットを作成するための手慣れた方法を提供しています。 これを使用して、単純なメッセージから自由形式の会話まで、さまざまな会話型ユーザー インターフェイスを構築できます。 Bot Builder SDK for Node.js では、Restify という、Web サービスを構築するための一般的なフレームワークを使用して、ボットの Web サーバーを作成します。 SDK には、Express との互換性もあります。 

[Node.js クイック スタート](~/nodejs/bot-builder-nodejs-quickstart.md)では、Bot Builder SDK for Node.js を使用してボットを作成する手順を案内しています。 

Bot Builder SDK for Node.js は、npm パッケージとして入手できます。 Bot Builder SDK for Node.js とその依存関係をインストールするには、最初にボット用のフォルダーを作成し、そこに移動して、次の **npm** コマンドを実行します。

```nodejs
npm init
```

次に、以下の **npm** コマンドを実行して、Bot Builder SDK for Node.js と <a href="http://restify.com/" target="_blank">Restify</a> をインストールします。

```nodejs
npm install --save botbuilder
npm install --save restify
```

GitHub から Bot Builder SDK for Node.js の[ソース コード](https://github.com/Microsoft/BotBuilder/tree/master/Node)をダウンロードすることもできます。

## <a name="rest-api"></a>REST API

任意のプログラミング言語でボットを作成するには、Bot Framework REST API を使用できます。 [REST クイック スタート](rest-api/bot-framework-rest-connector-quickstart.md)では、REST を使用してボットを作成する手順を案内しています。

Bot Framework には、次の 3 つの REST API があります。

 - [Bot Connector REST API][connectorAPI] を使用すると、[Bot Framework Portal](https://dev.botframework.com/) 内で構成されているチャネルに、ボットからメッセージを送受信できるようになります。 

- [Bot State REST API][stateAPI]を使用すると、Bot Connector REST API を介して行われた会話に関連付けられた状態をボットで格納および取得できます。

- [Direct Line REST API][directLineAPI] を使用すると、クライアント アプリケーション、Web チャット コントロール、またはモバイル アプリなどの独自のアプリケーションを 1 つのボットに直接接続することができます。

## <a name="bot-framework-emulator"></a>Bot Framework エミュレーター
Bot Framework エミュレーターは、ローカルでもリモートでも、ボットをテストおよびデバッグすることができるデスクトップ アプリケーションです。 このエミュレーターを使用して、ボットとチャットし、ボットが送受信するメッセージを検査できます。 [Bot Framework エミュレーターの詳細を参照](~/bot-service-debug-emulator.md)し、お使いのプラットフォーム用の[エミュレーターをダウンロード](http://emulator.botframework.com)してください。

## <a name="bot-framework-channel-inspector"></a>Bot Framework Channel Inspector
Bot Framework [Channel Inspector](bot-service-channel-inspector.md) を使用してさ、さまざまなチャネルでボット機能がどのように動作するかを手早く確認します。

[bot-template]: http://aka.ms/bf-bc-vstemplate
[cortana-template]: https://aka.ms/bf-cortanaskill-template


[connectorAPI]: https://docs.botframework.com/en-us/restapi/connector/#navtitle
 
[stateAPI]: https://docs.botframework.com/en-us/restapi/state/#navtitle

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
