---
ms.openlocfilehash: 71eb85da7a18757806bb5d782723d270f8db0ac2
ms.sourcegitcommit: 4ddee4f90a07813ce570fdd04c8c354b048e22f3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/20/2020
ms.locfileid: "77479291"
---
構成されていない [zip デプロイ API](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url) を使用してお使いのボットのコードをデプロイする場合、Web App/Kudu は次のように動作します。

"_Kudu では、zip ファイルからデプロイを実行する準備ができていること、およびデプロイ中に npm install、dotnet restore/dotnet publish などの追加ビルド ステップが不要であることを既定で想定しています。_ "

ご自身のビルドされたコードを必要なすべての依存関係と共に、デプロイされる zip ファイルに含めることが重要です。そうしないと、ボットは意図したとおりに動作しません。

> [!IMPORTANT]
> プロジェクト ファイルを zip 圧縮する前に、必ずそのプロジェクト フォルダー "_内_" に移動します。
> - C# ボットの場合は、.csproj ファイルが含まれるフォルダーです。
> - JavaScript ボットの場合は、app.js または index.js ファイルが含まれるフォルダーです。
> - TypeScript ボットの場合は、_src_ フォルダー (bot.ts および index.ts ファイルがある場所) が含まれるフォルダーです。
> - Python ボットの場合は、app.py ファイルが含まれるフォルダーです。

>プロジェクト フォルダー**内**で、コマンドを実行して zip ファイルを作成する前に、zip ファイルに含めるすべてのファイルとフォルダーを選択します。これにより、選択したすべてのファイルとフォルダーを含む 1 つの zip ファイルが作成されます。
> お使いのルート フォルダーの場所が正しくない場合、**ボットは、Azure portal で実行できません**。
