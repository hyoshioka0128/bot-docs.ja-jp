---
ms.openlocfilehash: bbe74a9a82d3bd04593384d825d373bfab35e3db
ms.sourcegitcommit: 7e901f5f39a0cfb0d37e532321b90a1dcf4baadd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/08/2019
ms.locfileid: "72039787"
---
ボットをデプロイする前に、プロジェクト ファイルを準備する必要があります。 
<!-- **C# bots** -->
##### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cmd
az bot prepare-deploy --lang Csharp --code-dir "." --proj-file-path "MyBot.csproj"
```

--code-dir に関連する .csproj ファイルへのパスを指定する必要があります。 これは、--proj-file-path 引数を使用して実行できます。 コマンドによって --code-dir および --proj-file-path が "./MyBot.csproj" に解決されます

<!-- **JavaScript bots** -->
##### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```cmd
az bot prepare-deploy --code-dir "." --lang Javascript
```

このコマンドにより、Azure App Service で IIS と連携するために Node.js アプリに必要な web.config が取り込まれます。 web.config がお使いのボットのルートに保存されていることを確認してください。

<!-- **TypeScript bots** -->
##### <a name="typescripttabtypescript"></a>[TypeScript](#tab/typescript)

```cmd
az bot prepare-deploy --code-dir "." --lang Typescript
```

このコマンドは、上記の JavaScript の動作と似ていますが、Typescript ボットを対象としています。

---

> [!NOTE]
>  C# ボットと JavaScript ボットの場合、`az bot prepare-depoloy` コマンドを実行すると、ボット プロジェクトフォルダーに `.deployment` ファイルが生成されます。
> TypeScript ボットの場合、このコマンドは 2 つの `web.config` ファイルを生成します。 1 つはプロジェクト フォルダー内にあり、もう 1 つはプロジェクト フォルダー内の **src** フォルダーにあります。 
