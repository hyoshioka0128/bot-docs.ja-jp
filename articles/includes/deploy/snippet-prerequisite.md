---
ms.openlocfilehash: 266cdc2bbeeb140e4b601c1bf0fb3fa8eb085dda
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360924"
---
- Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/) を作成してください。
- 最新バージョンの [Azure CLI ツール](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)をインストールします。
- 最新バージョンの [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) ツールをインストールします。
- 最新リリース バージョンの [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) をインストールします。
- [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29) をインストールして構成します。
- [.bot](~/v4sdk/bot-file-basics.md) ファイルの知識。

msbot 4.3.2 以降では、Azure CLI バージョン 2.0.54 以降が必要です。 botservice 拡張機能をインストールした場合は、次のコマンドでそれを削除します。

```cmd
az extension remove --name botservice
```