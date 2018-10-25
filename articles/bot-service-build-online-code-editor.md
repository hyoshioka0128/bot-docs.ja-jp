---
title: Azure オンライン コード エディターでボットをビルドする | Microsoft Docs
description: Bot Service 内のオンライン コード エディターを使用して、ボットをビルドする方法について説明します。
keywords: オンライン コード エディター、Azure portal、Functions ボット
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: tools
ms.date: 09/21/2018
ms.openlocfilehash: ba8e87073f888f21afb806221108bd6b5391744c
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997239"
---
# <a name="edit-a-bot-with-online-code-editor"></a>オンライン コード エディターでボットを編集する

IDE を必要とすることなく、オンライン コード エディターを使用してボットをビルドできます。 このトピックでは、オンライン コード エディターでボット コードを開く方法について示します。 

## <a name="edit-bot-source-code-in-online-code-editor"></a>オンライン コード エディターでボットのソース コードを編集する

オンライン コード エディターでボットのソース コードを編集するには、所有しているボットの種類に応じて以下の操作を行います。

### <a name="web-app-bot"></a>Web アプリ ボット
1. [Azure portal](http://portal.azure.com) にサインインして、ボットのブレードを開きます。
2. **[ボット管理]** セクションの下の **[ビルド]** をクリックします。
3. **[オンライン コード エディターを開く]** をクリックします。 これにより、新しいブラウザー ウィンドウにそのボットのコードが表示されます。 

   ![オンライン コード エディターを開く](~/media/azure-bot-build/open-online-code-editor.png)

   ボットの言語によって、**WWWRoot** ディレクトリの下のファイル構造は異なります。 たとえば、C# ボットを使用する場合、**WWWRoot** は次のようなものになります。

   ![C# のファイル構造](~/media/azure-bot-build/cs-wwwroot-structure.png)

   Node.js ボットを使用する場合、**WWWRoot** は次のようなものになります。

   ![Node.js のファイル構造](~/media/azure-bot-build/node-wwwroot-structure.png)

4. コードを変更します。 たとえば、C# ボットの場合、.cs ファイルから開始できます。 Node.js ボットの場合、App.js ファイルから開始できます。

   > [!NOTE]
   > プロジェクト内の現在のソース ファイルのコードを変更できるときは、オンライン コード エディターを使用して新しいソース ファイルを作成することはできません。 新しいソース ファイルをボットに追加するには、[ソース プロジェクトをダウンロード](bot-service-build-download-source-code.md)して、ご自分のファイルを追加し、変更を Azure に発行する必要があります。

5. **従量課金プラン**の C# ボットおよびすべての Node.js ボットは自動的に保存されます。 

6. **App Service** プランの C# ボットの場合、**[コンソール]** ブレードを開いて、**build.cmd** コマンドを送信します。 

   ![コンソール ブレードでプロジェクトをビルドする](~/media/azure-bot-build/cs-console-build-cmd.png)
 
   > [!NOTE]
   > このコマンドのビルドに失敗した場合、ご利用のボットの App Service を再起動してみて、もう一度ビルドを試みます。 App Service を再起動するには、ご利用のボットのブレードで **[All App service settings]\(すべての App Service の設定\)**、**[再起動]** ボタンの順にクリックします。
   > ![Web アプリを再起動する](~/media/azure-bot-build/open-online-code-editor-restart-appservice.png)

7. Azure portal に戻り、**[Test in Web Chat]\(Web チャットでテストする\)** をクリックして変更した内容をテストします。 このボット用の Web チャットが既に開いている場合、**[やり直す]** をクリックして新しい変更を表示します。

### <a name="functions-bot"></a>Functions ボット

1. [Azure portal](http://portal.azure.com) にサインインして、ボットのブレードを開きます。
2. **[ボット管理]** セクションの下の **[ビルド]** をクリックします。
3. **[Open this bot in Azure Functions]\(Azure Functions でこのボットを開く\)** をクリックします。 これにより、<a href="http://go.microsoft.com/fwlink/?linkID=747839" target="_blank">Azure Functions</a> の UI と共にボットが表示されます。 
4. コードを変更します。 たとえば、関数のメッセージ コードを更新します。 以下のスクリーン ショットでは、Node.js Functions ボットのメッセージ コードを示しています。

   ![Functions Bot のメッセージのコード エディター](~/media/azure-bot-build/functions-messages-code.png)

5. コードの変更を保存します。
6. Azure portal に戻り、**[Test in Web Chat]\(Web チャットでテストする\)** をクリックして変更した内容をテストします。 このボット用の Web チャットが既に開いている場合、**[やり直す]** をクリックして新しい変更を表示します。

## <a name="next-steps"></a>次の手順
これで、オンライン コード エディターを使用してボットのコードを編集する方法を理解し、お気に入りの IDE を使用してローカルでボットをビルドできるようになりました。

> [!div class="nextstepaction"]
> [ボットのソース コードをダウンロードする](bot-service-build-download-source-code.md)
