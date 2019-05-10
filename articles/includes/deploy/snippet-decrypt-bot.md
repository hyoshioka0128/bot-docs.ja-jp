---
ms.openlocfilehash: eac6abae509d92ea4714bc01221f180ea575950f
ms.sourcegitcommit: 4ff7a8772124a567f43e2c3e13aded368c4002e3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/03/2019
ms.locfileid: "65035739"
---
暗号化キーを取得します。

1. [Azure Portal](http://portal.azure.com/) にログインします。
1. ボットの Web App Bot リソースを開きます。
1. ボットの **[アプリケーション設定]** を開きます。
1. **[アプリケーション設定]** ウィンドウで、**[アプリケーション設定]** までスクロールします。
1. **botFileSecret** を探してその値をコピーします。

.bot ファイルを復号化します。

```cmd
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

| オプション | 説明 |
|:---|:---|
| --bot | ダウンロードした .bot ファイルの相対パス。 |
| --secret | 暗号化キー。 |

暗号化を解除した `.bot` ファイルをローカル ボット プロジェクトが含まれるディレクトリにコピーし、この新しい `.bot` ファイルを使用するようにボットを更新して、古い `.bot` ファイルを削除します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**appsettings.json** で、ローカル ディレクトリに追加した新しい `.bot` ファイルを指すように **botFilePath** プロパティを更新します。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**.env** で、ローカル ディレクトリに追加した新しい `.bot` ファイルを指すように **botFilePath** プロパティを更新します。

---

ボットを更新した後、ダウンロードしたボットの一時ディレクトリを削除します。