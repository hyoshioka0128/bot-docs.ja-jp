---
title: ボットのソース コードをダウンロードして再デプロイする | Microsoft Docs
description: Bot Service のダウンロードと発行を行う方法について説明します。
keywords: ソース コードのダウンロード, 再デプロイ, デプロイ, zip ファイル, 発行
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/08/2018
ms.openlocfilehash: b77e096d28f51f605db9c49d36e796553f9293ef
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756454"
---
# <a name="download-and-redeploy-bot-source-code"></a>ボットのソース コードをダウンロードして再デプロイする

Bot Service では、ボット用のソース プロジェクト全体をダウンロードできます。 これにより、好みの IDE を使用して、ボットに関する作業をローカルで実行できます。 変更が完了したら、変更を Azure に発行できます。 

このトピックでは、ボットのソース コードのダウンロード方法と、変更の Azure への発行方法を示します。 

## <a name="download-bot-source-code"></a>ボットのソース コードをダウンロードする

ボットをローカルで開発するには、次の操作を行います。

1. Azure Portal で、ボット用のブレードを開きます。
2. **[ボット管理]** セクションの下の **[ビルド]** をクリックします。
3. **[Download zip file]\(zip ファイルのダウンロード\)** をクリックします。 

   ![ソース コードのダウンロード](~/media/azure-bot-build/download-zip-file.png)

4. .zip ファイルをローカル ディレクトリに抽出します。
5. 抽出したフォルダーに移動し、お気に入りの IDE でソース ファイルを開きます。
6. ソースを変更します。 既存のソース ファイルを編集するか、新しいものをプロジェクトに追加します。

準備ができたら、Azure にソースを発行できます。

## <a name="publish-node-bot-source-code-to-azure"></a>ノードのボットのソース コードを Azure に発行する

これらのパッケージをインストールするには、コマンド プロンプトからプロジェクト ディレクトリを参照し、次の NPM コマンドを実行します。

**注:** これらのパッケージは、1 回のみ 追加する必要があります。

```console
npm install --save fs
npm install --save path
npm install --save request
npm install --save zip-folder
```

これで、Microsoft Azure にプロジェクトを発行する準備が整いました。 Microsoft Azure にプロジェクトを発行するには、コマンド プロンプトで次の NPM コマンドを実行します。

```console
npm run azure-publish
```

> [!NOTE]
> この NPM コマンドの後にエラーが発生した場合は、`"scripts": {"azure-publish": "node publish.js"}` を `package.json` ファイルに追加し、もう一度実行する必要があります。

## <a name="publish-c-bot-source-code-to-azure"></a>C# のボットのソース コードを Azure に発行する

Visual Studio を使用した C# コードの Azure への発行は、2 つの手順で構成されるプロセスです。最初に、発行設定を構成する必要があります。 その後、変更を発行できます。

Visual Studio から発行を構成するには、次の操作を行います。

1. Visual Studio で、**[ソリューション エクスプローラー]** をクリックします。
2. プロジェクト名を右クリックし、**[発行...]** をクリックします。**[発行]** ウィンドウが開きます。
3. **[新しいプロファイルの作成]**、**[プロファイルのインポート]**、**[OK]** の順にクリックします。
4. プロジェクト フォルダー、**PostDeployScripts** フォルダーの順に移動し、末尾が **.PublishSettings** のファイルを選択します。 **[開く]** をクリックします。

これで、変更を Azure に発行するようにプロジェクトが構成されました。

プロジェクトが構成されたら、次の操作を行って、ボットのソース コードを Azure に発行できます。

1. Visual Studio で、**[ソリューション エクスプローラー]** をクリックします。
2. プロジェクト名を右クリックし、**[発行...]** をクリックします。
3. **[発行]** ボタンをクリックして、変更を Azure に発行します。

## <a name="next-steps"></a>次の手順
ボットをローカルでビルドする方法がわかったので、ボットの継続的デプロイを設定できます。

> [!div class="nextstepaction"]
> [継続的デプロイを設定する](bot-service-build-continuous-deployment.md)
