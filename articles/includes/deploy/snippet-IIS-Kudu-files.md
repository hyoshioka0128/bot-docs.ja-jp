---
ms.openlocfilehash: 3cd58ddac050139b93eb4ee39909b04289afd821
ms.sourcegitcommit: e5bf9a7fa7d82802e40df94267bffbac7db48af7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/18/2020
ms.locfileid: "77446873"
---

C#、Javascript、または Typescript のボットをデプロイする前に、プロジェクト ファイルを準備する必要があります。 Python のボットをデプロイする場合は、この手順をスキップできます。

<!-- **C# bots** -->
##### <a name="c"></a>[C#](#tab/csharp)

```cmd
az bot prepare-deploy --lang Csharp --code-dir "." --proj-file-path "MyBot.csproj"
```

--code-dir に関連する .csproj ファイルへのパスを指定する必要があります。 これは、--proj-file-path 引数を使用して実行できます。 コマンドによって --code-dir および --proj-file-path が "./MyBot.csproj" に解決されます

<!-- **JavaScript bots** -->
##### <a name="javascript"></a>[JavaScript](#tab/javascript)

```cmd
az bot prepare-deploy --code-dir "." --lang Javascript
```

このコマンドにより、Azure App Service で IIS と連携するために Node.js アプリに必要な web.config が取り込まれます。 web.config がお使いのボットのルートに保存されていることを確認してください。

<!-- **TypeScript bots** -->
##### <a name="typescript"></a>[TypeScript](#tab/typescript)

```cmd
az bot prepare-deploy --code-dir "." --lang Typescript
```

このコマンドは、上記の JavaScript の動作と似ていますが、Typescript ボットを対象としています。

---

> [!NOTE]
>  C# ボットと JavaScript ボットの場合、`az bot prepare-deploy` コマンドを実行すると、ボット プロジェクトフォルダーに `.deployment` ファイルが生成されます。
> TypeScript ボットの場合、このコマンドは 2 つの `web.config` ファイルを生成します。 1 つはプロジェクト フォルダー内にあり、もう 1 つはプロジェクト フォルダー内の **src** フォルダーにあります。 
