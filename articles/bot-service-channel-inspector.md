---
title: Channel Inspector を使用してボットの機能をプレビューする | Microsoft Docs
description: Channel Inspector を使用して、さまざまなチャンネル上で Bot Framework の各種機能の外観と動作を確認します。
keyword: bot, preview features, channel, display, Channel Inspector, Text formatting, markdown, XML
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: def3f811704af2e2a22612ba5755eb5dbd2d8044
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301812"
---
# <a name="preview-bot-features-with-the-channel-inspector"></a>Channel Inspector を使用してボットの機能をプレビューする

Bot Framework では、テキスト、ボタン、画像、カルーセルまたはリスト形式で表示されるリッチ カードなどのさまざまな機能を備えたボットを作成できます。 ただし、各チャンネルが、メッセージング クライアントによる機能のレンダリングを最終的に制御します。

機能が複数のチャンネルでサポートされていても、チャンネルごとに少し異なる方法でその機能をレンダリングする可能性があります。 チャンネルがネイティブにサポートしていない機能がメッセージに含まれている場合、チャンネルはメッセージのコンテンツをテキストや静的な画像としてダウンレンダリングしようとする場合があり、クライアント上でのメッセージの外観に大きな影響を及ぼす可能性があります。 チャンネルが特定の機能をまったくサポートしない場合もあります。 たとえば、GroupMe クライアントでは入力インジケーターを表示できません。

## <a name="the-channel-inspector"></a>Channel Inspector

[Channel Inspector][inspector] は、さまざまなチャンネル上で Bot Framework の各種機能の外観と動作をプレビューできるように作成されています。 さまざまなチャンネルで各機能がどのようにレンダリングされるのかを理解することによって、通信するチャンネル上で優れたユーザー エクスペリエンスを提供するようにボットを設計できるようになります。 また、Channel Inspector は、Bot Framework の機能を確認し、視覚的に探索できる優れた方法でもあります。

> [!NOTE]
> リッチ カードは、複数のチャンネルにわたって一貫した表示を確保するための、ボット情報交換の開発標準です。 リッチ カードの詳細については、[.NET][netcard] または [Node.js][nodecard] のドキュメントをご覧ください。

## <a name="text-formatting"></a>テキストの書式設定

テキストの書式設定により、テキスト メッセージを視覚的に強化できます。 ボットでは、**プレーン**テキストのほかに、**Markdown** や **XML** を使用して書式設定したテキスト メッセージを、それらをサポートしているチャンネルに送信できます。 次の各表に、**Markdown** および **XML** で最もよく使用されるテキストの書式設定を示します。 各チャンネルがサポートしているテキストの書式設定は、ここに示すものよりも多い場合もあれば、少ない場合もあります。 使用する機能が対象とするチャンネル上でサポートされているかどうかは、[Channel Inspector][inspector] で確認できます。

### <a name="markdown-text-format"></a>Markdown テキスト形式

次のスタイルは、`textFormat` が **markdown** に設定されている場合にサポートされます。  

| Style | Markdown | 例 |
| ---- | ---- | ---- |
| 太字 | \*\*text\*\* | **text** |
| 斜体 | \*text\* | *text* |
| ヘッダー (1 から 5) | # H1 | # H1 |
| 取り消し線 | \~\~text\~\~ | ~~text~~ |
| 横罫線 | ---- | ---- |
| 記号付きリスト | \* text |  <ul><li>text</li></ul> |
| 番号付きリスト | 1. text | 1. text |
| 整形済みテキスト | \`text\` | `text`  |
| ブロック引用 | \> text | <blockquote>text</blockquote> |
| ハイパーリンク | \[bing](http://www.bing.com) | [bing](http://www.bing.com) |
| 画像リンク| !\[duck](http://aka.ms/Fo983c) | ![duck](http://aka.ms/Fo983c) |

> [!NOTE]
> **Markdown** の HTML タグは、Microsoft Bot Framework Web チャット チャンネルではサポートされていません。 **Markdown** で HTML タグを使用する必要がある場合は、それらをサポートする [Direct Line](~/bot-service-channel-connect-directline.md) チャンネルでレンダリングできます。 また、`textFormat` を **xml** に設定して下記の HTML タグを使用し、ボットを Skype チャンネルに接続することもできます。

### <a name="xml-text-format"></a>XML テキスト形式

次のスタイルは、`textFormat` が **xml** に設定されている場合にサポートされます。

|      Style      |                       XML                       |                   例                   |
|-----------------|-------------------------------------------------|---------------------------------------------|
|      太字       |                 \<b\>text\</b\>                 |            <strong>text</strong>            |
|     斜体      |                 \<i\>text\</i\>                 |                <em>text</em>                |
|    下線    |                 \<u\>text\</u\>                 |                 <u>text</u>                 |
|  取り消し線  |                 \<s\>text\</s\>                 |                 <s>text</s>                 |
|    ハイパーリンク    |   \<a href="http://www.bing.com"\>bing\</a\>    |   <a href="http://www.bing.com">bing</a>    |
|    段落    |                 \<p\>text\</p\>                 |                 <p>text</p>                 |
|   改行    |                     \<br/\>                     |             line 1 <br/>line 2              |
| 横罫線 |                     \<hr/\>                     |                    <hr/>                    |
|  ヘッダー (1 から 4)   |                \<h1\>text\</h1\>                |             <h1>Heading 1</h1>              |
|      code       |           \<code\>code block\</code\>           |           <code>code block</code>           |
|      image      | \<img src="<http://aka.ms/Fo983c>" alt="Duck"\> | <img src="http://aka.ms/Fo983c" alt="Duck"> |

> [!NOTE]
> `textFormat` **xml** は、Skype チャンネルでのみサポートされています。

## <a name="preview-features-across-various-channels"></a>さまざまなチャンネルで機能をプレビューする

チャンネルで特定の機能がどのようにレンダリングされるのかを確認するには、[Channel Inspector][inspector] に移動し、**[Channel]** 一覧からチャンネルを、**[Feature]** 一覧から機能を選択します。 たとえば、Skype でヒーロー カードがどのようにレンダリングされるのかを確認するには、**[Channel]** を *[Skype]* に設定し、**[Feature]** を *[HeroCard]* に設定します。

![Skype チャンネルとヒーロー カードを示す Channel Inspector](~/media/bot-service-channel-inspector.png)

Channel Inspector には、指定したチャンネルでレンダリングされるときの選択した機能のプレビューが表示されます。 **[Notes]** セクションには、メッセージの制限事項や表示の変更に関する重要な情報が示されます。 たとえば、リッチ カードには 1 つの画像しかサポートしていないものがあり、一部の機能は特定のチャンネル上でダウンレンダリングされる場合があります。

> [!NOTE]
> 現在、Channel Inspector では次のチャンネルがサポートされています。
> * Cortana
> * 電子メール
> * Facebook
> * GroupMe
> * Kik
> * Skype
> * Skype for Business
> * Slack
> * sms
> * Microsoft Teams
> * Telegram
> * WeChat
> * Web チャット
>
> その他のチャネルは、今後追加される可能性があります。

## <a name="features-that-can-be-previewed"></a>プレビューできる機能

現在、Channel Inspector では次の機能をプレビューできます。

|Feature | 説明|
| --- | ----|
| Adaptive Cards (アダプティブ カード) | テキスト、音声、画像、ボタン、入力フィールドの任意の組み合わせを含めることができるカード。 |
| Buttons (ボタン)| ユーザーがクリックできるボタン。 ボタンは、それらが属しているメッセージと共に会話キャンバスに表示されます。 |
| Carousel (カルーセル)| コンパクトでスクロール可能な、カードの横並びのリスト。 縦レイアウトの場合は "リスト" を使用します。|
| ChannelData (チャンネル データ)| カード、テキスト、添付ファイル以外のチャンネル固有の機能にアクセスするためのメタデータを渡す方法。|
| Codesample (コード サンプル)| コンピューター コードを表示するように設計された、複数行の整形済みテキスト ブロック。|
| DirectMessage (ダイレクト メッセージ)| グループ会話の 1 人のメンバーに送信されるメッセージ。
| ドキュメント| メディア以外の添付ファイルを送受信します。 |
| Emoji (絵文字)| サポートされている絵文字を表示します。
| GroupChat (グループ チャット)| ボットによって、グループ会話に参加するためのメッセージが送信されます。 |
| HeroCard (ヒーロー カード)| 説明テキスト コンテンツと共に、通常は 1 つの大きな画像、タップ アクション、ボタン (任意) が含まれた書式付き添付ファイルです。 |
| イメージ| 画像添付ファイルを表示します。 |
| List Layout (リスト レイアウト)| カードの縦並びのリスト。 横並びのスクロール可能なレイアウトには、カルーセルを使用します。|
| Location| ユーザーの物理的な場所をボットと共有します。 |
| Markdown| Markdown で書式設定されたテキストをレンダリングします。|
| [メンバー]| グループ チャット中に会話のメンバーのリストをボットと共有します。 |
| Receipt Card (領収書カード)| ユーザーに領収書を表示します。 |
| Signin Card (サインイン カード)| ユーザーに認証資格情報の入力を要求します。|
| Suggested Actions (推奨されるアクション) | ユーザーが入力を提供するためにタップできるボタンとして表示されるアクション。 |
| Thumbnail Card (サムネイル カード)| 説明テキスト コンテンツと共に、1 つの小さな画像 (サムネイル)、タップ アクション、ボタン (任意) が含まれた書式付き添付ファイルです。 |
| Typing (入力)| 入力インジケーターを表示します。 これは、ボットが引き続き機能しているが、バックグラウンドで何らかのアクションを実行中であることをユーザーに通知する際に役立ちます。|
| ビデオ| 動画添付ファイルと再生コントロールを表示します。|

## <a name="additional-resources"></a>その他のリソース

* [Channel Inspector][inspector]
* [Node.js][nodecard] および [.NET][netcard] のリッチ カード
* [Node.js][nodemedia] および [.NET][netmedia] のメディア添付ファイル
* [Node.js][nodebutton] および [.NET][netbutton] の推奨されるアクション

[inspector]: https://docs.botframework.com/en-us/channel-inspector/channels/Skype/

[syntax]: https://daringfireball.net/projects/markdown/syntax

[netcard]: ~/dotnet/bot-builder-dotnet-add-rich-card-attachments.md
[nodecard]: ~/nodejs/bot-builder-nodejs-send-rich-cards.md

[netmedia]: ~/dotnet/bot-builder-dotnet-add-media-attachments.md
[nodemedia]: ~/nodejs/bot-builder-nodejs-send-receive-attachments.md

[netbutton]: ~/dotnet/bot-builder-dotnet-add-suggested-actions.md
[nodebutton]: ~/nodejs/bot-builder-nodejs-send-suggested-actions.md
