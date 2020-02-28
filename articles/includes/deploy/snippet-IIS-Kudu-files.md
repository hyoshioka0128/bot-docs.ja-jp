---
ms.openlocfilehash: 3a3d1056e9c6f913efbf563131a5ab377a57d2d0
ms.sourcegitcommit: 4ddee4f90a07813ce570fdd04c8c354b048e22f3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/20/2020
ms.locfileid: "77479296"
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
>  C# ボットの場合、`az bot prepare-deploy` コマンドを実行すると、1 つの`.deployment` ファイルがボット プロジェクト フォルダーに生成されます。
> JavaScript ボットの場合、このコマンドで 2 つの `web.config` ファイルがプロジェクト フォルダーに生成されます。
> TypeScript ボットの場合、このコマンドで 2 つの `web.config` ファイルが生成されます。 1 つはプロジェクト フォルダー内にあり、もう 1 つはプロジェクト フォルダー内の **src** フォルダーにあります。


