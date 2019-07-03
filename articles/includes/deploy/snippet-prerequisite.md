---
ms.openlocfilehash: 2f970239f7df015bf38d163dd6442a9920b21115
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405584"
---
- Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/) を作成してください。
- 最新バージョンの [Azure CLI ツール](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)をインストールします。
- 最新バージョンの [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) ツールをインストールします。
- 最新リリース バージョンの [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) をインストールします。
- [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29) をインストールして構成します。 
- [ボット リソース管理](~/v4sdk/bot-file-basics.md)に関する知識。

msbot 4.3.2 以降では、Azure CLI バージョン 2.0.54 以降が必要です。 botservice 拡張機能をインストールした場合は、次のコマンドでそれを削除します。

```cmd
az extension remove --name botservice
```
