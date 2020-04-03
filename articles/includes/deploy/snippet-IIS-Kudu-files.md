---
ms.openlocfilehash: 3e578bcd8301bfb74946eda99d86f5e25eea17b0
ms.sourcegitcommit: 126c4f8f8c7a3581e7521dc3af9a937493e6b1df
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/01/2020
ms.locfileid: "80499866"
---

C#、Javascript、または Typescript のボットをデプロイする前に、プロジェクト ファイルを準備する必要があります。 Python のボットをデプロイする場合は、この手順をスキップして手順 5.2 に進むことができます。

<!-- **C# bots** -->
##### <a name="c"></a>[C#](#tab/csharp)

```cmd
az bot prepare-deploy --lang Csharp --code-dir "." --proj-file-path "MyBot.csproj"
```

--code-dir に関連する .csproj ファイルへのパスを指定する必要があります。 これは、--proj-file-path 引数を使用して実行できます。 コマンドによって --code-dir および --proj-file-path が "./MyBot.csproj" に解決されます。

このコマンドを実行すると、ボット プロジェクトフォルダーに `.deployment` ファイルが生成されます。

<!-- **JavaScript bots** -->
##### <a name="javascript"></a>[JavaScript](#tab/javascript)

```cmd
az bot prepare-deploy --code-dir "." --lang Javascript
```

このコマンドによって、2 つの `web.config` ファイルがプロジェクト フォルダーに生成されます。 Node.js アプリでは、Azure App Services で IIS と連携するために web.config が必要です。 web.config がお使いのボットのルートに保存されていることを確認してください。

<!-- **TypeScript bots** -->
##### <a name="typescript"></a>[TypeScript](#tab/typescript)

```cmd
az bot prepare-deploy --code-dir "." --lang Typescript
```

このコマンドは、2 つの `web.config` ファイルを生成するという点で JavaScript と同様に機能します。 1 つはプロジェクト フォルダー内にあり、もう 1 つはプロジェクト フォルダー内の **src** フォルダーにあります。

<!-- **TPython bots** -->
##### <a name="python"></a>[Python](#tab/Python)

Python ボットをデプロイする前にプロジェクト ファイルを準備する必要はありません。 手順 5.2 に進みます。

---
