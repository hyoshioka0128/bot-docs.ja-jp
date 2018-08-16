---
title: Bot Builder SDK for .NET | Microsoft Docs
description: ボットを作成するための強力で使いやすいフレームワークである、Bot Builder SDK for .NET の概要を説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: bd89017714a46c541468ccc9ed9990dde14719b7
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301841"
---
# <a name="bot-builder-sdk-for-net"></a>Bot Builder SDK for .NET
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.js](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

Bot Builder SDK for .NET は、自由形式の対話と、ユーザーが使用可能な値の中から選択するガイド付きの会話の両方に対応できるボットを作成するための強力なフレームワークです。 この使いやすい SDK では、C# を活用することで、.NET 開発者が使い慣れた方法でボットを作成できるようにします。

この SDK を使用すると、SDK の次の機能を利用したボットを作成できます。 

- 分離された構成可能なダイアログを備えた強力なダイアログ システム
- はい/いいえのような単純なもの、文字列、数値、列挙型に対応する組み込みプロンプト
- <a href="http://luis.ai" target="_blank">LUIS</a> などの強力な AI フレームワークを利用する組み込みダイアログ
- 必要に応じて、ヘルプ、ナビゲーション、明確化、確認を提供してユーザーとの会話を進めるボットを (C# クラスから) 自動的に生成する FormFlow

> [!IMPORTANT]
> 2017 年 7 月 31 日に、Bot Framework のセキュリティ プロトコルに破壊的変更が実装されました。 これらの変更がボットに悪影響を及ぼさないようにするには、アプリケーションが Bot Builder SDK v3.5 以降を使用していることを確認する必要があります。 2017 年 1 月 5 日 (Bot Builder SDK v3.5 のリリース日) より前に入手した SDK を使用してボットを作成した場合は、Bot Builder SDK v3.5 以降に必ずアップグレードしてください。

## <a name="get-the-sdk"></a>SDK の取得

SDK は、NuGet パッケージおよびオープン ソースとして <a href="https://github.com/Microsoft/BotBuilder" target="_blank">GitHub</a> で入手できます。

> [!IMPORTANT]
> Bot Builder SDK for .NET には、.NET Framework 4.6 以降が必要です。 .NET Framework の古いバージョンを対象にしている既存のプロジェクトに SDK を追加する場合は、まず、.NET Framework 4.6 を対象にしてプロジェクトを更新する必要があります。

Visual Studio プロジェクトに SDK をインストールするには、次の手順を実行します。

1. **ソリューション エクスプローラー**でプロジェクト名を右クリックし、**[NuGet パッケージの管理...]** を選択します。
2. **[参照]** タブで、検索ボックスに「Microsoft.Bot.Builder」と入力します。
3. 結果の一覧で **[Microsoft.Bot.Builder]** を選択し、**[インストール]** をクリックして、変更を適用します。

## <a name="get-code-samples"></a>コード サンプルの入手

この SDK には、Bot Builder SDK for .NET の機能を使用した[サンプル ソース コード](bot-builder-dotnet-samples.md)が含まれています。

## <a name="next-steps"></a>次の手順

Bot Builder SDK for .NET を使用してボットを作成する方法の詳細については、まず、次の記事をご覧ください。

- [クイック スタート](bot-builder-dotnet-quickstart.md): このチュートリアルの手順に従って、簡単なボットをすばやく作成し、テストします。
- [主要概念](bot-builder-dotnet-concepts.md): Bot Builder SDK for .NET の主要概念を確認します。

Bot Builder SDK for .NET に関する問題が発生した場合や提案がある場合は、[サポート](../bot-service-resources-links-help.md)に関する記事で利用可能なリソースの一覧をご覧ください。 
