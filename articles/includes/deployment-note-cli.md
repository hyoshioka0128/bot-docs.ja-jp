---
ms.openlocfilehash: cc7e656d7c8a61e7bf784db579d0065c3bff331a
ms.sourcegitcommit: 4ff7a8772124a567f43e2c3e13aded368c4002e3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/03/2019
ms.locfileid: "65035765"
---
LUIS などのサービスを使用する場合は、`luisAuthoringKey` も渡す必要があります。 Azure で既存のリソース グループを使用する場合は、上のコマンドで `groupName` 引数を使用します。

ボットのデプロイ時に発生する可能性のある問題をトラブルシューティングしやすくするために、`verbose` オプションを使用することを強くお勧めします。 `msbot clone services` コマンドで使用される追加のオプションを以下に説明します。

| 引数    | 説明 |
|--------------|-------------|
| `folder`     | `bot.recipe` ファイルの場所。 既定で、レシピ ファイルは `DeploymentsScript/MSBotClone` に作成されます。 このファイルは変更しないでください。|
| `location`   | ボット サービス リソースを作成するために使用される地理的な場所。 たとえば、eastus、westus、westus2 などです。|
| `proj-file`  | C# ボットの場合は、.csproj ファイルです。 JS ボットの場合は、ローカルのボットのスタートアップ プロジェクト ファイル名 (index.js など) です。|
| `name`       | Azure でのボットのデプロイに使用される一意の名前。 ローカルのボットと同じ名前にすることができます。 名前にスペースやアンダースコアを含めないでください。|
| `luisAuthoringKey` | LUIS リソースに適した LUIS 作成リージョンの作成キー。 |

Azure リソースを作成するには、認証の完了が求められます。 画面に表示される指示に従って、この手順を完了します。

上記の手順の完了に必要な時間は "_数秒から数分_" です。Azure 内に作成されるリソースには不規則な名前が付けられます。 不規則な名前の詳細については、GitHub リポジトリで[問題 #796](https://github.com/Microsoft/botbuilder-tools/issues/796) に関するページを参照してください。
