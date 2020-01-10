---
ms.openlocfilehash: e335e8c3da1b84ed3ce443effc6965b60c933501
ms.sourcegitcommit: a547192effb705e4c7d82efc16f98068c5ba218b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/25/2019
ms.locfileid: "75491138"
---
構成されていない [zip デプロイ API](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url) を使用してお使いのボットのコードをデプロイする場合、Web App/Kudu は次のように動作します。

"_Kudu では、zip ファイルからデプロイを実行する準備ができていること、およびデプロイ中に npm install、dotnet restore/dotnet publish などの追加ビルド ステップが不要であることを既定で想定しています。_ "

そのため、ご自身のビルドされたコードと、Web アプリに展開されている必要な依存関係すべてを zip ファイルに含めることが重要です。これを行わないと、ボットは意図したとおりに動作しません。

> [!IMPORTANT]
> プロジェクト ファイルを zip 圧縮する前に、必ずそのプロジェクト フォルダー "_内_" に移動します。 
> - C# ボットの場合は、.csproj ファイルが含まれるフォルダーです。 
> - JavaScript ボットの場合は、app.js または index.js ファイルが含まれるフォルダーです。 
> - TypeScript ボットの場合は、_src_ フォルダー (bot.ts および index.ts ファイルがある場所) が含まれるフォルダーです。 
> - Python ボットの場合は、app.py ファイルが含まれるフォルダーです。

>プロジェクト フォルダー**内**で、コマンドを実行して zip ファイルを作成する前に、zip ファイルに含めるすべてのファイルとフォルダーを選択します。これにより、選択したすべてのファイルとフォルダーを含む 1 つの zip ファイルが作成されます。
>
> お使いのルート フォルダーの場所が正しくない場合、**ボットは、Azure portal で実行できません**。
