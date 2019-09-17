---
ms.openlocfilehash: 80cc69ca3d863a4f96f1dd241e6471074cc9976f
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386048"
---
構成されていない [zip デプロイ API](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url) を使用してお使いのボットのコードをデプロイする場合、Web App/Kudu は次のように動作します。

"_Kudu では、zip ファイルからデプロイを実行する準備ができていること、およびデプロイ中に npm install、dotnet restore/dotnet publish などの追加ビルド ステップが不要であることを既定で想定しています。_ "

そのため、ご自身のビルドされたコードと、Web アプリに展開されている必要な依存関係すべてを zip ファイルに含めることが重要です。これを行わないと、ボットは意図したとおりに動作しません。

> [!IMPORTANT]
> プロジェクト ファイルを zip 圧縮する前に、必ずそのプロジェクト フォルダー "_内_" に移動します。 
> - C# ボットの場合は、.csproj ファイルが含まれるフォルダーです。 
> - JS ボットの場合は、app.js ファイルまたは index.js ファイルが含まれるフォルダーです。 
>
>プロジェクト フォルダー**内**ですべてのファイルを選択して zip 圧縮し、そのフォルダー内でコマンドを実行します。 
>
> お使いのルート フォルダーの場所が正しくない場合、**ボットは、Azure portal で実行できません**。
