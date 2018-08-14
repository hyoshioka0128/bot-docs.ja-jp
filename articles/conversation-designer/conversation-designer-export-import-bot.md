---
title: Conversation Designer ボットのインポートとエクスポート | Microsoft Docs
description: Conversation Designer ボットをインポートおよびエクスポートする方法を説明します。
author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: ed055cb0f75d148e6a0dd3248366851901d9a675
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304188"
---
# <a name="export-and-import-a-conversation-designer-bot"></a>Conversation Designer ボットのエクスポートとインポート

Conversation Designer のボットは .zip ファイルとしてコンピューターにエクスポートできます。 これにより、ボットをバックアップすることができます。 エクスポートした .zip ファイルのいずれかを使用して、後でボットを以前の状態に復元できます。 

## <a name="export-a-conversation-designer-bot"></a>Conversation Designer ボットのエクスポート

エクスポートすると、Conversation Designer ボットの現在の状態をローカル コンピューターに保存することができます。 

Conversation Designer ボットをエクスポートするには、次の手順を実行します。
1. ナビゲーション パネルの右上隅にある省略記号 (**...**) をクリックします。
2. **[エクスポート]** をクリックします。 必要な情報がサーバーによって収集され、.zip ファイルを保存するためのオプションが表示されます。
3. エクスポートした .zip ファイルをローカル コンピューターに保存します。

これで、ボットはコンピューターの .zip ファイルに保存されました。 これをボットに[インポート](#import-a-conversation-designer-bot)して戻すと、後でボットをこの状態に復元することができます。

## <a name="import-a-conversation-designer-bot"></a>Conversation Designer ボットのインポート

インポートすると、Conversation Designer ボットを以前の状態に復元することができます。 インポートすることにより、現在のボットはインポートするボットに置換されます。 現在のボットにある現在の状態を保持する場合は、インポート操作を実行する前に、必ず現在のボットを[エクスポート](#export-a-conversation-designer-bot)してください。

Conversation Designer ボットをインポートするには、次の手順を実行します。
1. ナビゲーション パネルの右上隅にある省略記号 (**...**) をクリックします。
2. **[インポート]** をクリックします。 
3. 現在のボットの作業内容を保存する場合は、**[バックアップとインポート]** オプションを選択します。 このオプションを選択すると、現在のボットがローカル コンピューターに保存され、インポートする .zip ファイルの場所を指定するよう求められます。 それ以外の場合は、**[バックアップなしのインポート]** を選択します。

これで、ボットがインポートされました。

> [!NOTE]
> インポートできるのは、Conversation Designer によってエクスポートされたボットのみです。

## <a name="import-a-luis-app-into-a-conversation-designer-bot"></a>LUIS アプリの Conversation Designer ボットへのインポート

LUIS アプリがあり、Conversation Designer ボットで使用する場合は、LUIS アプリを Conversation Designer ボットにインポートできます。 概念的には、このプロセスでは、LUIS アプリと Conversation Designer ボットをエクスポートしてから、ボットの **luis.model** ファイルの内容を **luis.json** ファイルとスワップする必要があります。 次に、変更内容を Conversation Designer ボットにインポートして戻します。 基本的に、ボットの LUIS 意図を LUIS アプリの意図に置換します。 このため、ボットの LUIS 意図のカスタマイズを開始する前に、このインポート操作を実行することをお勧めします。そうしない場合、このインポート操作によってすべての作業内容が上書きされます。

> [!NOTE]
> ボットに関連付けられている [LUIS](https://luis.ai) アプリ (各 Conversation Designer ボットには対応する LUIS アプリがあります) で変更を加える場合は、これらの手順を実行する必要はありません。 代わりに、Conversation Designer ボットに移動して、[**[保存]**](conversation-designer-save-bot.md) をクリックするだけです。

LUIS アプリを Conversation Designer ボットにインポートするには、次の手順を実行します。

1. [LUIS.ai](https://luis.ai) から LUIS アプリを開き、**[設定]** をクリックします。
2. 使用するアプリの **[バージョン]** を選択して、**{ }** アクション アイコンをクリックします。 このアクションにより、**luis.json** ファイルがローカル コンピューターにダウンロードされます。 
3. Conversation Designer から、[新しいボットを作成する](conversation-designer-create-bot.md#create-a-conversation-designer-bot)か、または既存のボットを開きます。
4. ボットを[エクスポート](#export-a-conversation-designer-bot)します。 これで、ボットが .zip ファイルとしてコンピューターにエクスポートされます。
5. エクスポートした .zip ファイルに移動して、抽出します。
6. テキスト エディターで **luis.model** ファイルを開き、このファイルの内容を **luis.json** ファイルの内容に置換します。 ファイルを保存します。
7. Conversation Designer ボットのフォルダーを zip 圧縮します。
8. 次に、ボットを Conversation Designer ボットに[インポート](#import-a-conversation-designer-bot)して戻します。

これで、ボットは今インポートした新しい LUIS 意図を使用できるようになりました。

## <a name="next-step"></a>次のステップ
> [!div class="nextstepaction"]
> [タスク](conversation-designer-tasks.md)
