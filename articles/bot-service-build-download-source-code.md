---
title: ボットのソース コードをダウンロードして再デプロイする | Microsoft Docs
description: Bot Service のダウンロードと発行を行う方法について説明します。
keywords: ソース コードのダウンロード, 再デプロイ, デプロイ, zip ファイル, 発行
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/26/2018
ms.openlocfilehash: ee7a7a9f1b4c06f8ad762f750099383e218d98f2
ms.sourcegitcommit: b8bd66fa955217cc00b6650f5d591b2b73c3254b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326429"
---
# <a name="download-and-redeploy-bot-code"></a>ボット コードをダウンロードして再デプロイする
Azure Bot Service では、お客様のボットのソース プロジェクト全体をダウンロードできるので、好みの IDE を使用してローカルで作業できます。 コードの更新が完了したら、変更を再び Azure portal に発行することができます。 ここでは、Azure portal と `az` cli を使用してコードをダウンロードする方法について説明します。 また、Visual Studio と `az` cli ツールを使用して、更新されたボット コードを再デプロイする方法についても説明します。 お客様にとって最適な方法を選択できます。

## <a name="prerequisites"></a>前提条件
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest)
- `az extension add -n botservice` コマンドを使用して、az botservice 拡張機能をインストールする

### <a name="download-code-using-the-azure-portal"></a>Azure portal を使用してコードをダウンロードする
[Azure portal](https://portal.azure.com) からコードをダウンロードするには、以下を行います。
1. ボットのブレードを開きます。
1. **[Bot management]\(ボット管理\)** セクションの下の **[ビルド]** をクリックします。
1. **[Download source code]\(ソース コードのダウンロード\)** の下の **[Download zip file]\(zip ファイルのダウンロード\)** をクリックします。
1. Azure によってダウンロード URI が準備されるのを待ってから、通知内の **[Download zip file]\(zip ファイルのダウンロード\)** をクリックします。
1. .zip ファイルをローカル ディレクトリに保存して展開します。

C# ボットの場合は、次に示すように `appsettings.json` ファイルを更新して、.bot ファイル情報を含めます。

```
{
  "botFilePath": "yourbasicBot.bot",
  "botFileSecret": "ukxxxxxxxxxxxs="
}
```
`botFilePath` は、ボットの名前を参照します。単に "yourbasicBot.bot" をお客様独自のボット名に置き換えてください。 `botFileSecret` キーを取得するには、ボットのキーの生成について、[ボット ファイルの暗号化](https://aka.ms/bot-file-encryption)に関する記事を参照してください。


node.js ボットをお持ちの場合は、次のエントリが含まれている `.env` ファイルを追加します。
```
botFilePath=yourbasicBot.bot
botFileSecret=ukxxxxxxxxxxxxs=
```

次に、既存のソース ファイルを編集するか、プロジェクトに新しいソース ファイルを追加して、ソースを変更します。 エミュレーターを使用してコードをテストします。 変更したコードを Azure portal に再デプロイする準備ができたら、次の手順に従います。

### <a name="publish-code-using-visual-studio"></a>Visual Studio を使用してコードを発行する
1. Visual Studio でプロジェクト名を右クリックし、**[発行]** をクリックします。**[発行]** ウィンドウが開きます。

![Azure への発行](~/media/azure-bot-build/azure-csharp-publish.png)

2. お客様のプロジェクトのプロファイルを選択します。
3. プロジェクトの _publish.cmd_ ファイルに記載されているパスワードをコピーします。
4. **[発行]** をクリックします。
5. メッセージが表示されたら、手順 3. でコピーしたパスワードを入力します。   

プロジェクトが構成された後、プロジェクトの変更が Azure に発行されます。 

次に、`az` cli を使用してコードをダウンロードおよび再デプロイする方法について説明します。

### <a name="download-code-using-azure-cli"></a>Azure CLI を使用してコードをダウンロードする

最初に、az cli ツールを使用して Azure portal にログインします。

```azcli
az login
```

一意の一時的な認証コードの入力が求められます。 サインインするには、Web ブラウザーを使用して Microsoft [デバイスのログイン](https://microsoft.com/devicelogin)にアクセスし、CLI から提供されたコードを貼り付けて続行します。

`az` cli を使用してコードをダウンロードするには、次のコマンドを使用します。
```azcli
az bot download --name "my-bot-name" --resource-group "my-resource-group"`
```
コードがダウンロードされたら、以下を行います。
- C# ボットの場合は、次に示すように appsettings.json ファイルを更新して、.bot ファイル情報を含めます。

```
{
  "botFilePath": "yourbasicBot.bot",
  "botFileSecret": "ukxxxxxxxxxxxs="
}
```

- node.js ボットの場合は、次のエントリが含まれている .env ファイルを追加します。

```
botFilePath=yourbasicBot.bot
botFileSecret=ukxxxxxxxxxxxxs=
```

次に、既存のソース ファイルを編集するか、プロジェクトに新しいソース ファイルを追加して、ソースを変更します。 エミュレーターを使用してコードをテストします。 変更したコードを Azure portal に再デプロイする準備ができたら、次の手順に従います。

### <a name="login-to-azure-cli-by-running-the-following-command"></a>次のコマンドを実行して Azure CLI にログインします。
既にログインしている場合は、この手順を省略できます。

```azcli
az login
```
一意の一時的な認証コードの入力が求められます。 サインインするには、Web ブラウザーを使用して Microsoft [デバイスのログイン](https://microsoft.com/devicelogin)にアクセスし、CLI から提供されたコードを貼り付けて続行します。

### <a name="publish-code-using-azure-cli"></a>Azure CLI を使用してコードを発行する
`az` cli を使用してコードを Azure に再び発行するには、次のコマンドを使用します。
```azcli
az bot publish --name "my-bot-name" --resource-group "my-resource-group" --code-dir <path to directory> 
```

`code-dir` オプションを使用して、使用するディレクトリを指定できます。 このオプションを指定しないで `az bot publish` コマンドを実行すると、発行にローカル ディレクトリが使用されます。

## <a name="next-steps"></a>次の手順
変更を Azure に再びアップロードする方法を理解したところで、ボットの継続的デプロイを設定できます。

> [!div class="nextstepaction"]
> [継続的デプロイを設定する](bot-service-build-continuous-deployment.md)
