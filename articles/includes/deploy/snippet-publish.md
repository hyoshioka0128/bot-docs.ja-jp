---
ms.openlocfilehash: 88732d2d5490962d7a899d936767e7dd148e94c5
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360860"
---
ローカルのボットを Azure に公開します。 この手順には時間がかかる場合があります。

```cmd
az bot publish --name <bot-resource-name> --proj-name "<project-file-name>" --resource-group <resource-group-name> --code-dir <directory-path> --verbose --version v4
```

| オプション | 説明 |
|:---|:---|
| --name | Azure 内のボットのリソース名。 |
| --proj-name | C# の場合は、公開する必要があるスタートアップ プロジェクト ファイル名 (.csproj は含めません) を使用します。 (例: `EnterpriseBot`)。 Node.js の場合は、ボットのメイン エントリ ポイントを使用します。 たとえば、「 `index.js` 」のように入力します。 |
| --resource-group | リソース グループの名前。 |
| --code-dir | ボット コードをアップロードする元のディレクトリ。 |

これが完了すると、"Deployment successful!" (デプロイに成功しました!) メッセージが表示され、ボットは Azure にデプロイされます。