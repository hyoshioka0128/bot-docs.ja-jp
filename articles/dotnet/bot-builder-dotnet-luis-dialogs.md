---
title: LUIS を使用して意図とエンティティを認識する | Microsoft Docs
description: Bot Framework SDK for .NET の LUIS ダイアログを使用して、ボットが自然言語を理解できるようにする方法について説明します。
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 7e7eb36d0cb3cdbf18037f05b4960b240cb70d8d
ms.sourcegitcommit: eacf1522d648338eebefe2cc5686c1f7866ec6a2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/30/2019
ms.locfileid: "70167400"
---
# <a name="recognize-intents-and-entities-with-luis"></a>LUIS を使用して意図とエンティティを認識する 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

この記事では、メモ作成ボットの例を使用して、自然言語入力に対してボットが適切に応答するのに Language Understanding ([LUIS][LUIS]) がどのように役立つかを示します。 ボットではユーザーの**意図**を識別することで、ユーザーが何をしたいかを検出します。 この意図は音声またはテキスト入力 (または**発話**) から決定されます。 意図は、ボットが実行するアクションに発話をマップします。 たとえば、ノート記録ボットは `Notes.Create` 意図を認識して、ノートを作成するための機能を呼び出します。 ボットは、発話の中の重要な言葉である**エンティティ**を抽出する必要もあります。 メモ作成ボットの例では、`Notes.Title` エンティティによって各メモのタイトルが識別されます。

## <a name="create-a-language-understanding-bot-with-bot-service"></a>Bot Service での Language Understanding ボットの作成

1. [Azure Portal](https://portal.azure.com) のメニュー ブレードで、 **[新しいリソースの作成]** を選択し、 **[すべて表示]** をクリックします。

    ![新しいリソースの作成](../media/bot-builder-dotnet-use-luis/bot-service-creation.png)

2. 検索ボックスで、**Web アプリ ボット**を検索します。 

    ![新しいリソースの作成](../media/bot-builder-dotnet-use-luis/bot-service-selection.png)

3. **[ボット サービス]** ブレードで、必要な情報を指定し、 **[作成]** をクリックします。 これによって、ボット サービスと LUIS アプリが作成され、Azure にデプロイされます。 
   * **[アプリ名]** にボットの名前を設定します。 この名前は、ボットがクラウドにデプロイされるときに、サブドメインとして使用されます (mynotesbot.azurewebsites.net など)。 この名前は、ボットに関連付けられる LUIS アプリの名前としても使用されます。 後で、ボットに関連付けられた LUIS アプリを探すために使用できるよう、名前をコピーしておきます。
   * サブスクリプション、[リソース グループ](/azure/azure-resource-manager/resource-group-overview)、App Service プラン、[場所](https://azure.microsoft.com/regions/)を選択します。
   * **[ボット テンプレート]** フィールドで **Language Understanding (C#)** テンプレートを選択します。

     ![[ボット サービス] ブレード](../media/bot-builder-dotnet-use-luis/bot-service-setting-callout-template.png)

   * チェック ボックスをオンにしてサービス利用規約に同意します。

4. ボット サービスがデプロイされたことを確認します。
    * [通知] (Azure portal の上端にあるベル アイコン) をクリックします。 通知が、 **[デプロイが開始されました]** から **[デプロイメントに成功しました]** に変わります。
    * 通知が **[デプロイメントに成功しました]** に変わったら、その通知で **[リソースに移動]** をクリックします。

## <a name="try-the-bot"></a>ボットを試す

ボットがデプロイされたことを確認するには、 **[通知]** をオンにします。 通知が、 **[デプロイは進行中です...]** から **[デプロイメントに成功しました]** に変わります。 **[リソースに移動]** ボタンをクリックして、ボットのリソース ブレードを開きます。

ボットが登録されたら、 **[Test in Web Chat]\(Web チャットでのテスト\)** をクリックして、Web チャット ウィンドウを開きます。 Web チャットに「hello」と入力します。

  ![Web チャットでのボットのテスト](../media/bot-builder-dotnet-use-luis/bot-service-web-chat.png)

ボットが "あいさつにリーチしました。 hello と言いました" と言って応答します。 これにより、ご自身のメッセージはボットによって受信され、そのボットが作成した既定の LUIS アプリに渡されたことが確認されます。 この既定の LUIS アプリによって、あいさつの意図が検出されました。

## <a name="modify-the-luis-app"></a>LUIS アプリを変更する

Azure へのログインに使用するものと同じアカウントを使用して、[https://www.luis.ai](https://www.luis.ai) にログインします。 **[My apps]\(マイ アプリ\)** をクリックします。 アプリの一覧で、ボット サービスの作成時に **[ボット サービス]** ブレードの **[アプリ名]** で指定した名前で始まるアプリを探します。 

LUIS アプリは次の 4 つの意図で始まります:Cancel、Greeting、Help、None。 <!-- picture -->

次の手順では、Note.Create、Note.ReadAloud、および Note.Delete の各意図を追加します。 

1. ページの左下にある **[Prebuit Domains]\(事前構築済みドメイン\)** をクリックします。 **Note** ドメインを探して **[ドメインの追加]** をクリックします。

2. このチュートリアルでは、事前構築済みの **Note** ドメインに含まれるすべての意図を使用するわけではありません。 **[Intents]\(意図\)** ページで、次の各意図名をクリックしてから **[Delete Intent]\(意図の削除\)** ボタンをクリックします。
   * Note.ShowNext
   * Note.DeleteNoteItem
   * Note.Confirm
   * Note.Clear
   * Note.CheckOffItem
   * Note.AddToNote

   次の意図だけを LUIS アプリに残してください。 
   * Note.ReadAloud
   * Note.Create
   * Note.Delete
   * なし
   * [Help]
   * Greeting
   * Cancel 

     ![LUIS アプリに表示される意図](../media/bot-builder-dotnet-use-luis/luis-intent-list.png)

3. 右上の **[トレーニング]** ボタンをクリックして、ご利用のアプリをトレーニングします。
4. 上部のナビゲーション バーの **[PUBLISH]\(発行\)** をクリックして、 **[発行]** ページを開きます。 **[Publish to production slot]\(運用スロットに発行\)** ボタンをクリックします。 発行に成功した後、 **[Publish App]\(アプリの発行\)** ページの **[エンドポイント]** 列に表示されている URL を、リソース名 Starter_Key で始まる行にコピーします。 後からボットのコードで使用するために、この URL を保存しておきます。 URL の形式は次の例のようになります。`https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx?subscription-key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&timezoneOffset=0&verbose=true&q=`

## <a name="modify-the-bot-code"></a>ボット コードの変更

**[Build]\(ビルド\)** 、 **[Open online code editor]\(オンライン コード エディターを開く\)** の順にクリックします。
    ![オンライン コード エディターを開く](../media/bot-builder-dotnet-use-luis/bot-service-build.png)

コード エディターで、`BasicLuisDialog.cs` を開きます。 これには、LUIS アプリから意図を処理するための次のコードが含まれています。
```cs
using System;
using System.Configuration;
using System.Threading.Tasks;

using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Luis;
using Microsoft.Bot.Builder.Luis.Models;

namespace Microsoft.Bot.Sample.LuisBot
{
    // For more information about this template visit http://aka.ms/basic-luis-dialog
    [Serializable]
    public class BasicLuisDialog : LuisDialog<object>
    {
        public BasicLuisDialog() : base(new LuisService(new LuisModelAttribute(
            ConfigurationManager.AppSettings["LuisAppId"], 
            ConfigurationManager.AppSettings["LuisAPIKey"], 
            domain: ConfigurationManager.AppSettings["LuisAPIHostName"])))
        {
        }

        [LuisIntent("None")]
        public async Task NoneIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        // Go to https://luis.ai and create a new intent, then train/publish your luis app.
        // Finally replace "Greeting" with the name of your newly created intent in the following handler
        [LuisIntent("Greeting")]
        public async Task GreetingIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        [LuisIntent("Cancel")]
        public async Task CancelIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        [LuisIntent("Help")]
        public async Task HelpIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        private async Task ShowLuisResult(IDialogContext context, LuisResult result) 
        {
            await context.PostAsync($"You have reached {result.Intents[0].Intent}. You said: {result.Query}");
            context.Wait(MessageReceived);
        }
    }
}
```
### <a name="create-a-class-for-storing-notes"></a>ノートを格納するクラスの作成

次の `using` ステートメントを BasicLuisDialog.cs に追加します。

```cs
using System.Collections.Generic;
```

`BasicLuisDialog` クラス内で、コンストラクター定義の後に次のコードを追加します。

```cs
        // Store notes in a dictionary that uses the title as a key
        private readonly Dictionary<string, Note> noteByTitle = new Dictionary<string, Note>();
        
        [Serializable]
        public sealed class Note : IEquatable<Note>
        {

            public string Title { get; set; }
            public string Text { get; set; }

            public override string ToString()
            {
                return $"[{this.Title} : {this.Text}]";
            }

            public bool Equals(Note other)
            {
                return other != null
                    && this.Text == other.Text
                    && this.Title == other.Title;
            }

            public override bool Equals(object other)
            {
                return Equals(other as Note);
            }

            public override int GetHashCode()
            {
                return this.Title.GetHashCode();
            }
        }

        // CONSTANTS        
        // Name of note title entity
        public const string Entity_Note_Title = "Note.Title";
        // Default note title
        public const string DefaultNoteTitle = "default";
```

### <a name="handle-the-notecreate-intent"></a>Note.Create 意図の処理
Note.Create 意図を処理するには、次のコードを `BasicLuisDialog` クラスに追加します。

```cs
        private Note noteToCreate;
        private string currentTitle;
        [LuisIntent("Note.Create")]
        public Task NoteCreateIntent(IDialogContext context, LuisResult result)
        {
            EntityRecommendation title;
            if (!result.TryFindEntity(Entity_Note_Title, out title))
            {
                // Prompt the user for a note title
                PromptDialog.Text(context, After_TitlePrompt, "What is the title of the note you want to create?");
            }
            else
            {
                var note = new Note() { Title = title.Entity };
                noteToCreate = this.noteByTitle[note.Title] = note;

                // Prompt the user for what they want to say in the note           
                PromptDialog.Text(context, After_TextPrompt, "What do you want to say in your note?");
            }
            
            return Task.CompletedTask;
        }
        
        
        private async Task After_TitlePrompt(IDialogContext context, IAwaitable<string> result)
        {
            EntityRecommendation title;
            // Set the title (used for creation, deletion, and reading)
            currentTitle = await result;
            if (currentTitle != null)
            {
                title = new EntityRecommendation(type: Entity_Note_Title) { Entity = currentTitle };
            }
            else
            {
                // Use the default note title
                title = new EntityRecommendation(type: Entity_Note_Title) { Entity = DefaultNoteTitle };
            }

            // Create a new note object 
            var note = new Note() { Title = title.Entity };
            // Add the new note to the list of notes and also save it in order to add text to it later
            noteToCreate = this.noteByTitle[note.Title] = note;

            // Prompt the user for what they want to say in the note           
            PromptDialog.Text(context, After_TextPrompt, "What do you want to say in your note?");

        }

        private async Task After_TextPrompt(IDialogContext context, IAwaitable<string> result)
        {
            // Set the text of the note
            noteToCreate.Text = await result;
            
            await context.PostAsync($"Created note **{this.noteToCreate.Title}** that says \"{this.noteToCreate.Text}\".");
            
            context.Wait(MessageReceived);
        }
```

### <a name="handle-the-notereadaloud-intent"></a>Note.ReadAloud 意図の処理
ボットは `Note.ReadAloud` 意図を使用して、1 つのノート (または、ノートのタイトルが検出されない場合はすべてのノート) の内容を表示できます。

次のコードを `BasicLuisDialog` クラスに貼り付けます。
```cs
        [LuisIntent("Note.ReadAloud")]
        public async Task NoteReadAloudIntent(IDialogContext context, LuisResult result)
        {
            Note note;
            if (TryFindNote(result, out note))
            {
                await context.PostAsync($"**{note.Title}**: {note.Text}.");
            }
            else
            {
                // Print out all the notes if no specific note name was detected
                string NoteList = "Here's the list of all notes: \n\n";
                foreach (KeyValuePair<string, Note> entry in noteByTitle)
                {
                    Note noteInList = entry.Value;
                    NoteList += $"**{noteInList.Title}**: {noteInList.Text}.\n\n";
                }
                await context.PostAsync(NoteList);
            }

            context.Wait(MessageReceived);
        }
        
        public bool TryFindNote(string noteTitle, out Note note)
        {
            bool foundNote = this.noteByTitle.TryGetValue(noteTitle, out note); // TryGetValue returns false if no match is found.
            return foundNote;
        }
        
        public bool TryFindNote(LuisResult result, out Note note)
        {
            note = null;

            string titleToFind;

            EntityRecommendation title;
            if (result.TryFindEntity(Entity_Note_Title, out title))
            {
                titleToFind = title.Entity;
            }
            else
            {
                titleToFind = DefaultNoteTitle;
            }

            return this.noteByTitle.TryGetValue(titleToFind, out note); // TryGetValue returns false if no match is found.
        }
```

### <a name="handle-the-notedelete-intent"></a>Note.Delete 意図の処理
次のコードを `BasicLuisDialog` クラスに貼り付けます。

```cs
        [LuisIntent("Note.Delete")]
        public async Task NoteDeleteIntent(IDialogContext context, LuisResult result)
        {
            Note note;
            if (TryFindNote(result, out note))
            {
                this.noteByTitle.Remove(note.Title);
                await context.PostAsync($"Note {note.Title} deleted");
            }
            else
            {                             
                // Prompt the user for a note title
                PromptDialog.Text(context, After_DeleteTitlePrompt, "What is the title of the note you want to delete?");                         
            }           
        }

        private async Task After_DeleteTitlePrompt(IDialogContext context, IAwaitable<string> result)
        {
            Note note;
            string titleToDelete = await result;
            bool foundNote = this.noteByTitle.TryGetValue(titleToDelete, out note);

            if (foundNote)
            {
                this.noteByTitle.Remove(note.Title);
                await context.PostAsync($"Note {note.Title} deleted");
            }
            else
            {
                await context.PostAsync($"Did not find note named {titleToDelete}.");
            }

            context.Wait(MessageReceived);
        }
```

## <a name="build-the-bot"></a>ボットのビルド
コード エディターで **build.cmd** を右クリックし、 **[Run from Console]\(コンソールから実行\)** を選択します。

   ![build.cmd の実行](../media/bot-builder-dotnet-use-luis/bot-service-run-console.png)

## <a name="test-the-bot"></a>ボットのテスト

Azure Portal で、 **[Test in Web Chat]\(Web チャットでのテスト\)** をクリックしてボットをテストします。 「Create a note」、「read my notes」、「delete notes」のようなメッセージを入力してみます。
   ![Web チャットでメモ ボットをテストする](../media/bot-builder-dotnet-use-luis/bot-service-test-notebot.png)

> [!TIP]
> 意図またはエンティティがボットによって必ずしも正しく認識されない場合は、発話の例をさらに追加して、LUIS アプリをトレーニングすることで、そのパフォーマンスを向上させます。 お使いのボットのコードを変更せずに、ご自身の LUIS アプリを再トレーニングできます。 [発話の例の追加](/azure/cognitive-services/LUIS/add-example-utterances)に関するページ、および[ご自身の LUIS アプリのトレーニングとテスト](/azure/cognitive-services/LUIS/train-test)に関するページをご覧ください。

> [!TIP]
> ボットのコードで問題が発生した場合は、次のことを確認してください。
> * [ボットをビルド](./bot-builder-dotnet-luis-dialogs.md#build-the-bot)したこと。
> * ボットのコードは、LUIS アプリ内のすべての意図のハンドラーを定義します。

## <a name="next-steps"></a>次の手順

ボットを試してみると、LUIS の意図によってタスクが呼び出されるしくみが分かります。 ただし、この単純な例では、現在アクティブなダイアログの中断はできません。 "help" や "cancel" のような中断を許可して処理することは、ユーザーが実際に行うことを考慮に入れた柔軟な設計です。 スコアラブル ダイアログを使用してダイアログで中断を処理できるようにする方法を理解します。

> [!div class="nextstepaction"]
> [Global message handlers using scorables](bot-builder-dotnet-scorable-dialogs.md)(スコアラブルを使用したグローバル メッセージ ハンドラー)

## <a name="additional-resources"></a>その他のリソース

- [ダイアログ](bot-builder-dotnet-dialogs.md)
- [ダイアログを使用して会話フローを管理する](bot-builder-dotnet-manage-conversation-flow.md)
- <a href="https://www.luis.ai" target="_blank">LUIS</a>
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Framework SDK for .NET リファレンス</a>

[LUIS]: https://www.luis.ai/
[NotesSample]: https://github.com/Microsoft/BotFramework-Samples/tree/master/docs-samples/CSharp/Simple-LUIS-Notes-Sample
[NotesSampleJSON]: https://github.com/Microsoft/BotFramework-Samples/blob/master/docs-samples/CSharp/Simple-LUIS-Notes-Sample/Notes.json
