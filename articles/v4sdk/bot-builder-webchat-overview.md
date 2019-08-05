---
title: Web チャットの概要 | Microsoft Docs
description: Bot Framework の Web チャットを構成する方法について学習します。
keywords: bot framework, web チャット, チャット, サンプル, react, リファレンス
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 06/07/2019
ms.openlocfilehash: 1d787d375fcd1ddade544724bb28dea9aaeb1dd6
ms.sourcegitcommit: f3fda6791f48ab178721b72d4f4a77c373573e38
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/31/2019
ms.locfileid: "68671420"
---
# <a name="web-chat-overview"></a>Web チャットの概要

この記事には、[Bot Framework の Web チャット](https://github.com/microsoft/BotFramework-WebChat) コンポーネントの詳細が含まれています。 Bot Framework の Web チャット コンポーネントは、Bot Framework V4 SDK 用の高度にカスタマイズ可能な Web ベースのクライアントです。 Bot Framework SDK v4 により、開発者は、会話をモデル化して、アプリケーションの高度なボットを構築できます。

Web チャット v3 から v4 への移行を検討している場合は、[移行のセクション](#migrating-from-web-chat-v3-to-v4)に進んでください。

## <a name="how-to-use"></a>使用方法

> [!NOTE]
> 以前のバージョンの Web チャット (v3) の場合は、[Web チャット v3 のブランチ](https://github.com/Microsoft/BotFramework-WebChat/tree/v3)を参照してください。

まず、[Azure Bot Service](https://azure.microsoft.com/services/bot-service/) を使用してボットを作成します。
ボットが作成されたら、Azure portal で[ボットの Web チャット シークレットを取得](../bot-service-channel-connect-webchat.md#step-1)する必要があります。 シークレットを使用して、[トークンを生成し](../rest-api/bot-framework-rest-direct-line-3-0-authentication.md)、それを Web チャットに渡します。

次に、Web チャット コントロールを Web サイトに追加する方法を示します。

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  token: 'YOUR_DIRECT_LINE_TOKEN'
               }),
               userID: 'YOUR_USER_ID',
               username: 'Web Chat User',
               locale: 'en-US',
               botAvatarInitials: 'WC',
               userAvatarInitials: 'WW'
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

> `userID`、`username`、`locale`、`botAvatarInitials`、`userAvatarInitials` はすべて、`renderWebChat` メソッドに渡すためのオプションのパラメーターです。 Web チャットのプロパティの詳細については、この `README` の [Web チャット API リファレンス](#web-chat-api-reference)を参照してください。
> ![Web チャットのスクリーンショット](https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/weatherquery.png.jpg)

### <a name="integrate-with-javascript"></a>JavaScript との統合

Web チャットは、JavaScript または React を使用して既存の Web サイトと統合するように設計されています。 JavaScript と統合すると、ある程度のスタイルとカスタマイズ性が得られます。

最もよく使われる機能を含んだ、完全で一般的な Web チャット パッケージを使用できます。

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  token: 'YOUR_DIRECT_LINE_TOKEN'
               }),
               userID: 'YOUR_USER_ID'
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

[完全な Web チャット バンドル](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.a.getting-started-full-bundle)の作業サンプルを参照してください。

### <a name="integrate-with-react"></a>React との統合

完全にカスタマイズできるように、React を使用して Web チャットのコンポーネントを再構成できます。

NPM から運用ビルドをインストールするには、`npm install botframework-webchat` を実行します。

```jsx
import { DirectLine } from 'botframework-directlinejs';
import React from 'react';
import ReactWebChat from 'botframework-webchat';

export default class extends React.Component {
  constructor(props) {
    super(props);

    this.directLine = new DirectLine({ token: 'YOUR_DIRECT_LINE_TOKEN' });
  }

  render() {
    return (
      <ReactWebChat directLine={ this.directLine } userID='YOUR_USER_ID' />
      element
    );
  }
}
```

> `npm install botframework-webchat@master` を実行して、Web チャットの GitHub `master` ブランチと同期されている開発ビルドをインストールすることもできます。

[React を介してレンダリングされる Web チャット](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/03.a.host-with-react/)の実稼働するサンプルを参照してください。

## <a name="customize-web-chat-ui"></a>Web チャットの UI のカスタマイズ

Web チャットは、ソース コードのフォークなしでカスタマイズできるように設計されています。 次の表は、Web チャットをさまざまな方法でインポートするときに実現可能なカスタマイズの種類を示しています。 このリストは全てを網羅しているわけではありません。

|                               | CDN バンドル         | React              |
| ----------------------------- | ------------------ | ------------------ |
| 色の変更                 | :heavy_check_mark: | :heavy_check_mark: |
| サイズの変更                  | :heavy_check_mark: | :heavy_check_mark: |
| CSS スタイルの更新/置換     | :heavy_check_mark: | :heavy_check_mark: |
| イベントのリッスン              | :heavy_check_mark: | :heavy_check_mark: |
| ホスティング Web ページとのやりとり | :heavy_check_mark: | :heavy_check_mark: |
| カスタム レンダーのアクティビティ      |                    | :heavy_check_mark: |
| カスタム レンダーの添付ファイル     |                    | :heavy_check_mark: |
| 新しい UI コンポーネントの追加         |                    | :heavy_check_mark: |
| UI 全体の再構成        |                    | :heavy_check_mark: |

カスタマイズの詳細については、[Web チャットのカスタマイズ](https://github.com/Microsoft/BotFramework-WebChat/blob/master/SAMPLES.md)の詳細を参照してください。

## <a name="migrating-from-web-chat-v3-to-v4"></a>Web チャット v3 から v4 への移行

v3 から v4 に移行する際に、移行される可能性があるパスは 3 つあります。 最初に、ご自分の始めのシナリオを比較します:

### <a name="my-current-website-integrates-web-chat-using-an-iframe-element-obtained-from-azure-bot-services-i-want-to-upgrade-to-v4"></a>私の現在の Web サイトは、Azure Bot Services から取得された `<iframe>` 要素を使用して Web チャットを統合します。 v4 へのアップグレードを希望しています。

2019 年 5 月から、`<iframe>` 要素を使用して Web チャットを統合する Web サイトに v4 を展開しています。 `<iframe>` を使用した Web チャットの統合については、[組み込みドキュメント](https://github.com/Microsoft/BotFramework-WebChat/tree/master/packages/embed)を参照してください。

### <a name="my-website-is-integrated-with-web-chat-v3-and-uses-customization-options-provided-by-web-chat-no-customization-at-all-or-very-little-of-my-own-customization-that-was-not-available-with-web-chat"></a>私の Web サイトは Web チャット v3 と統合されており、Web チャットによって提供されるカスタマイズ オプションを使用するか、カスタマイズをまったく使用しないか、または Web チャットで使用できなかった独自のカスタマイズをほとんど使用しません。

サンプルの [`01.c.getting-started-migration`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.c.getting-started-migration) の実装に従って、Web ページを Web チャットの v3 から v4 に変換してください。

### <a name="my-website-is-integrated-with-a-fork-of-web-chat-v3-i-have-implemented-a-lot-of-customization-in-my-version-of-web-chat-and-i-am-concerned-v4-is-not-compatible-with-my-needs"></a>私の Web サイトは、Web チャット v3 のフォークと統合されています。 自分のバージョンの Web チャットに多くのカスタマイズを実装しましたが、v4 が私のニーズと互換性がないことを懸念しています。

Web チャットの v4 に関して私たちのチームが気に入っていることの 1 つは、**Web チャットをフォークする必要なく**カスタマイズを追加できることです。 これにより、以前 Web チャットをフォークした v3 ユーザーに追加のオーバーヘッドが発生しますが、私たちは、バージョンアップを行うお客様をサポートするために最善を尽くします。 次の推奨事項を使用してください。

-  サンプルの実装[ `01.c.getting-started-migration`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.c.getting-started-migration)をご覧ください。 これは、Web チャットを起動して実行するための優れた出発点です。
-  次に、[サンプル リスト](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples)を調べて、カスタマイズ要件と、Web チャットが既にサポートしているものを比較してください。 これらのサンプルは、Web チャットでよく寄せられる機能で構成されています。
-  サンプルの 1 つ以上の機能が利用できない場合、[未解決の問題と終了した問題](https://github.com/Microsoft/BotFramework-WebChat/issues?utf8=%E2%9C%93&q=is%3Aissue+)、[サンプル ラベル](https://github.com/Microsoft/BotFramework-WebChat/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+label%3ASample)、[移行サポート ラベル](https://github.com/Microsoft/BotFramework-WebChat/issues?q=is%3Aissue+migrate+label%3A%22Migration+Support%22)を調べて、探している機能のサンプル リクエストやカスタマイズ サポートを検索してください。 未解決の問題にコメントを追加すると、チームは需要の高い要求に優先順位を付けるのに役立ちます。また、コミュニティへの参加をお勧めします。
-  未解決の要求のリストにご自身の機能が見つからない場合は、お気軽に[ご自分の独自の要求を提出](https://github.com/Microsoft/BotFramework-WebChat/issues/new)してください。 上記の項目と同様に、未解決の問題にコメントを追加する他のお客様によって、Web チャット ユーザー間で最も一般的に必要とされる機能を優先させることができます。
-  最後に、できるだけ早く機能が必要な場合は、Web チャットへの[プル要求](https://github.com/Microsoft/BotFramework-WebChat/compare)を歓迎します。 この機能を自分で実装するコーディング経験がある場合は、追加のサポートをいただけますと幸いです! 自分で機能を作成すると、Web チャットで迅速に使用できるようになり、同じ機能または類似の機能を探している他のお客様があなたの貢献を利用できます。
-  v4 に関する詳細については、この `README` の残りの部分を必ずご確認ください。

## <a name="samples-list"></a>サンプルのリスト

| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;サンプル&nbsp;名&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | 説明                                                                                                                                                                                                                         | Link                                                                                                                                |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| [`01.a.getting-started-full-bundle`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.a.getting-started-full-bundle)                                                                                       | CDN から埋め込まれた Web チャットを紹介し、シンプルで完全な機能を備えた Web チャットのデモンストレーションを行います。 これには、Adaptive Cards、Cognitive Services、Markdown-It の依存関係が含まれます。                                                            | [フル バンドルのデモ](https://microsoft.github.io/BotFramework-WebChat/01.a.getting-started-full-bundle)                               |
| [`01.b.getting-started-es5-bundle`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.b.getting-started-es5-bundle)                                                                                         | Web チャットの ES5 ponyfill を使用して、ES5 ブラウザーとの下位互換性を埋め込んだ、フル機能の Web チャットを紹介します。                                                                                                                | [ES5 バンドルのデモ](https://microsoft.github.io/BotFramework-WebChat/01.b.getting-started-es5-bundle)                                 |
| [`01.c.getting-started-migration`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.c.getting-started-migration)                                                                                           | Web チャット v3 のボットから v4 への移行方法を示します。                                                                                                                                                                        | [移行のデモ](https://microsoft.github.io/BotFramework-WebChat/01.c.getting-started-migration)                                   |
| [`02.a.getting-started-minimal-bundle`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/02.a.getting-started-minimal-bundle)                                                                                 | 基本的な依存関係のみで最小化された CDN を紹介します。 これには、Adaptive Cards、Cognitive Services、または Markdown-It の依存関係は含まれません。                                                                      | [最小限バンドルのデモ](https://microsoft.github.io/BotFramework-WebChat/02.a.getting-started-minimal-bundle)                         |
| [`02.b.getting-started-minimal-markdown`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/02.b.getting-started-minimal-markdown)                                                                             | 最小限バンドルの上に Markdown-It 依存関係の CDN を追加する方法を示します。                                                                                                                                            | [Markdown を使用した最小限のデモ](https://microsoft.github.io/BotFramework-WebChat/02.b.getting-started-minimal-markdown)                |
| [`03.a.host-with-react`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/03.a.host-with-react)                                                                                                               | フル機能の Web チャットをホストする React コンポーネントを作成する方法を示します。                                                                                                                                                 | [React を使用したホストのデモ](https://microsoft.github.io/BotFramework-WebChat/03.a.host-with-react)                                       |
| [`03.b.host-with-Angular`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/03.b.host-with-angular)                                                                                                           | フル機能の Web チャットをホストする Angular コンポーネントを作成する方法を示します。                                                                                                                                              | [Angular を使用したホストのデモ](https://stackblitz.com/github/omarsourour/ng-webchat-example)                                              |
| [`04.a.display-user-bot-initials-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/04.a.display-user-bot-initials-styling)                                                                           | 両方の Web チャット参加者のイニシャルを表示する方法を示します。                                                                                                                                                                | [ボットのイニシャルのデモ](https://github.com/Microsoft/BotFramework-WebChat/blob/master/samples/04.a.display-user-bot-initials-styling/)  |
| [`04.b.display-user-bot-images-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/04.b.display-user-bot-images-styling)                                                                               | 両方の Web チャット参加者のイメージとイニシャルを表示する方法を示します。                                                                                                                                                     | [ユーザー イメージのデモ](https://microsoft.github.io/BotFramework-WebChat/04.b.display-user-bot-images-styling)                           |
| [`05.a.branding-webchat-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.a.branding-webchat-styling)                                                                                             | ブランドに合わせて Web チャットをスタイル適用する機能を紹介します。 このカスタムのスタイル適用方法は、Web チャットの更新時に中断することはありません。                                                                                                   | [Web チャットのブランディングのデモ](https://microsoft.github.io/BotFramework-WebChat/05.a.branding-webchat-styling)                            |
| [`05.b.idiosyncratic-manual-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.b.idiosyncratic-manual-styling/)                                                                                    | 手動でスタイルを変更する方法を示します。これは、Web チャットのスタイルをカスタマイズするための、複雑で時間のかかる方法です。 Web チャットの更新時に手動のスタイルが壊れることがあります。                                                | [特異なスタイル適用のデモ](https://microsoft.github.io/BotFramework-WebChat/05.b.idiosyncratic-manual-styling)                    |
| [`05.c.presentation-mode-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.c.presentation-mode-styling)                                                                                           | チャット履歴は表示されますが、送信ボックスは表示されず、Adaptive Cards のインタラクティビティを無効にするプレゼンテーション モードの設定方法を示します。                                                                         | [プレゼンテーション モードのデモ](https://microsoft.github.io/BotFramework-WebChat/05.c.presentation-mode-styling)                           |
| [`05.d.hide-upload-button-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.d.hide-upload-button-styling)                                                                                         | スタイル設定を使用してファイルのアップロード ボタンを非表示にする方法を示します。                                                                                                                                                                            | [アップロード ボタンの非表示のデモ](https://microsoft.github.io/BotFramework-WebChat/05.d.hide-upload-button-styling)                         |
| [`06.a.cognitive-services-bing-speech-js`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.a.cognitive-services-bing-speech-js)                                                                           | (非推奨の) Cognitive Services Bing Speech API と JavaScript を使用した、音声テキスト変換とテキスト読み上げの機能を紹介します。                                                                                                      | [JS を使用した Bing Speech のデモ](https://microsoft.github.io/BotFramework-WebChat/06.a.cognitive-services-bing-speech-js)                 |
| [`06.b.cognitive-services-bing-speech-react`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.b.cognitive-services-bing-speech-react)                                                                     | (非推奨の) Cognitive Services Bing Speech API と React を使用した、音声テキスト変換とテキスト読み上げの機能を紹介します。                                                                                                           | [React を使用した Bing Speech のデモ](https://microsoft.github.io/BotFramework-WebChat/06.b.cognitive-services-bing-speech-react)           |
| [`06.c.cognitive-services-speech-services-js`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.c.cognitive-services-speech-services-js)                                                                   | Cognitive Services Speech Services API を使用した、音声テキスト変換とテキスト読み上げの機能を紹介します。                                                                                                                                  | [JS を使用した Speech Services のデモ](https://microsoft.github.io/BotFramework-WebChat/06.c.cognitive-services-speech-services-js)         |
| [`06.d.speech-web-browser`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.d.speech-web-browser)                                                                                                         | Web チャットのブラウザーベースの Web Speech API を使用して、テキスト読み上げを実装する方法を示します。 (サンプルの W3C 標準へのリンク)                                                                                                    | [Web Speech API のデモ](https://microsoft.github.io/BotFramework-WebChat/06.d.speech-web-browser)                                     |
| [`06.e.cognitive-services-speech-services-with-lexical-result`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.e.cognitive-services-speech-services-with-lexical-result)                                 | Cognitive Services Speech Services API から字句結果を使用する方法を示します。                                                                                                                                                 | [字句結果のデモ](https://microsoft.github.io/BotFramework-WebChat/06.e.cognitive-services-speech-services-with-lexical-result) |
| [`06.f.hybrid-speech`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.f.hybrid-speech)                                                                                                                   | 音声テキスト変換用のブラウザーベースの Web Speech API と、テキスト読み上げ用の Cognitive Services Speech Services API の両方の使用方法を示します。                                                                                        | [ハイブリッドの音声のデモ](https://microsoft.github.io/BotFramework-WebChat/06.f.hybrid-speech)                                           |
| [`07.a.customization-timestamp-grouping`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/07.a.customization-timestamp-grouping)                                                                             | タイムスタンプを表示または非表示にし、メッセージのグループ化を時間ごとに変更することによって、タイムスタンプをカスタマイズする方法を示します。                                                                                                             | [タイムスタンプのグループ化のデモ](https://microsoft.github.io/BotFramework-WebChat/07.a.customization-timestamp-grouping)                   |
| [`07.b.customization-send-typing-indicator`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/07.b.customization-send-typing-indicator)                                                                       | ユーザーが送信ボックスで入力を開始したときに入力アクティビティを送信する方法を示します。                                                                                                                                                | [ユーザーの入力インジケーターのデモ](https://microsoft.github.io/BotFramework-WebChat/07.b.customization-send-typing-indicator)             |
| [`08.customization-user-highlighting`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/08.customization-user-highlighting)                                                                                   | メッセージがユーザーからのものか、ボットからのものかに基づいて、アクティビティのスタイルをカスタマイズする方法を示します。                                                                                                                      | [ユーザーの強調表示のデモ](https://microsoft.github.io/BotFramework-WebChat/08.customization-user-highlighting)                       |
| [`09.customization-reaction-buttons`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/09.customization-reaction-buttons/)                                                                                    | ボットのニーズに固有の Web チャット用のカスタム コンポーネントを作成する機能を紹介します。 このチュートリアルでは、:thumbsup: や :thumbsdown: などの反応の絵文字を会話型アクティビティに追加する機能を説明します。 | [反応のボタンのデモ](https://microsoft.github.io/BotFramework-WebChat/09.customization-reaction-buttons)                         |
| [`10.a.customization-card-components`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/10.a.customization-card-components)                                                                                   | カスタム アクティビティ カードの添付ファイル、この場合は GitHub リポジトリ カードを作成する方法を示します。                                                                                                                                  | [カードのコンポーネントのデモ](https://microsoft.github.io/BotFramework-WebChat/10.a.customization-card-components)                         |
| [`10.b.customization-password-input`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/10.b.customization-password-input)                                                                                     | パスワード入力用のカスタム アクティビティを作成する方法を示します。                                                                                                                                                                      | [パスワード入力のデモ](https://microsoft.github.io/BotFramework-WebChat/10.b.customization-password-input)                           |
| [`11.customization-redux-actions`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/11.customization-redux-actions)                                                                                           | 高度なチュートリアル:ボットを介して redux アクションを送信することによって、Web チャット アプリに redux ミドルウェアを組み込む方法を示します。 この例では、ボットとユーザーの間のアクティビティに基づいた手動のスタイル設定を示します。             | [Redux アクションのデモ](https://microsoft.github.io/BotFramework-WebChat/11.customization-redux-actions)                               |
| [`12.customization-minimizable-web-chat`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/12.customization-minimizable-web-chat)                                                                             | 高度なチュートリアル:最小化可能な表示/非表示のチャット ボックスとして Web チャット インターフェイスを Web サイトに追加する方法を示します。                                                                                                              | [最小化可能な Web チャットのデモ](https://microsoft.github.io/BotFramework-WebChat/12.customization-minimizable-web-chat)                 |
| [`13.customization-speech-ui`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/13.customization-speech-ui)                                                                                                   | 高度なチュートリアル:ボットの主要コンポーネント (この場合は音声) を完全にカスタマイズする方法を示します。これはテキストベースのトランスクリプト UI を完全に置き換え、代わりにボットの応答を含む単純な音声ボタンを表示します。      | [音声の UI のデモ](https://microsoft.github.io/BotFramework-WebChat/13.customization-speech-ui)                                       |
| [`14.customization-piping-to-redux`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/14.customization-piping-to-redux)                                                                                       | 高度なチュートリアル:ボット アクティビティを自分の Redux ストアにパイプし、ボット アクティビティと Redux を介して自分のボットを使用してページを制御する方法を示します。                                                                          | [Redux へのパイプのデモ](https://microsoft.github.io/BotFramework-WebChat/14.customization-piping-to-redux)                           |
| [`15.a.backchannel-piggyback-on-outgoing-activities`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.a.backchannel-piggyback-on-outgoing-activities)                                                     | 高度なチュートリアル:すべての送信アクティビティにカスタム データを追加する方法を示します。                                                                                                                                                | [バックチャネルのピギーバッキングのデモ](https://microsoft.github.io/BotFramework-WebChat/15.a.backchannel-piggyback-on-outgoing-activities) |
| [`15.b.incoming-activity-event`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.b.incoming-activity-event)                                                                                               | 高度なチュートリアル:今後の処理のためにすべての受信アクティビティを JavaScript イベントに転送する方法を示します。                                                                                                                | [受信アクティビティのデモ](https://microsoft.github.io/BotFramework-WebChat/15.b.incoming-activity-event)                             |
| [`15.c.programmatic-post-activity`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.c.programmatic-post-activity)                                                                                         | 高度なチュートリアル:プログラムでメッセージを送信する方法を示します。                                                                                                                                                             | [プログラムによる投稿のデモ](https://microsoft.github.io/BotFramework-WebChat/15.c.programmatic-post-activity)                       |
| [`15.d.backchannel-send-welcome-event`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.d.backchannel-send-welcome-event)                                                                                 | 高度なチュートリアル:ブラウザー言語などのクライアント機能を使用してウェルカム イベントを送信する方法を示します。                                                                                                                        | [ウェルカム イベントのデモ](https://microsoft.github.io/BotFramework-WebChat/15.d.backchannel-send-welcome-event)                          |
| [`16.customization-selectable-activity`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/16.customization-selectable-activity)                                                                               | 高度なチュートリアル:各アクティビティにカスタム クリック動作を追加する方法を示します。                                                                                                                                                  | [選択可能なアクティビティのデモ](https://microsoft.github.io/BotFramework-WebChat/16.customization-selectable-activity)                   |
| [`17.chat-send-history`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/17.chat-send-history)                                                                                                               | 高度なチュートリアル:ユーザー入力を保存し、以前に送信されたメッセージをユーザーがさかのぼることができるようにする機能を示します。                                                                                                      | [チャット送信履歴のデモ](https://microsoft.github.io/BotFramework-WebChat/17.chat-send-history)                                     |
| [`18.customization-open-url`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/18.customization-open-url)                                                                                                     | 高度なチュートリアル:URL を開く動作をカスタマイズする方法を示します。                                                                                                                                                             | [URL を開く動作のカスタマイズのデモ](https://microsoft.github.io/BotFramework-WebChat/18.customization-open-url)                               |
| [`19.a.single-sign-on-for-enterprise-apps`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/19.a.single-sign-on-for-enterprise-apps)                                                                         | OAuth を使用してエンタープライズ アプリケーションにシングル サインオンを使用する方法を示します。                                                                                                                                                              | [OAuth を使用したシングル サインオンのデモ](https://microsoft.github.io/BotFramework-WebChat/19.a.single-sign-on-for-enterprise-apps)         |
 

## <a name="web-chat-api-reference"></a>Web チャット API リファレンス

Web チャット React コンポーネント (`<ReactWebChat>`) または `renderWebChat()` メソッドに渡すことができるプロパティがいくつかあります。 [`packages/component/src/Composer.js`](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Composer.js#L378) で始まるソース コードを自由に調べてください。 次に、使用可能なプロパティについて簡単に説明します。

| プロパティ                   | 説明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `activityMiddleware`       | [Redux ミドルウェア](https://medium.com/@jacobp100/you-arent-using-redux-middleware-enough-94ffe991e6)をモデルとした一連のミドルウェアで、開発者は現在の既存のアクティビティの DOM に新しい DOM コンポーネントを追加することができます。 ミドルウェアの署名は次のとおりです: `options => next => card => children => next(card)(children)`。                                                                                                                                                                                                                                           |
| `activityRenderer`         | Redux の[ストア エンハンサー](https://github.com/reduxjs/redux/blob/master/docs/Glossary.md#store-enhancer)の概念に似た `activityMiddleware` の "フラット化された" バージョン。                                                                                                                                                                                                                                                                                                                                                                                                                |
| `adaptiveCardHostConfig`   | Adaptive Cards ホストのカスタムの構成を渡します。使用している Adaptive Cards のバージョンでホスト構成を確認してください。 詳細は、[カスタム ホスト構成](https://github.com/microsoft/BotFramework-WebChat/issues/2034#issuecomment-501818238)を参照してください。                                                                                                                                                                                                                                                                                                                                    |
| `attachmentMiddleware`     | 開発者が添付ファイルに独自のカスタム HTML 要素を追加できるようにする一連のミドルウェア。 署名は次のとおりです: `options => next => card => next(card)`。                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `attachmentRenderer`       | `attachmentMiddleware` の "フラット化された" バージョン。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `cardActionMiddleware`     | Adaptive Cards や推奨アクションなど、開発者がカードのアクションを変更できるようにする一連のミドルウェア。 ミドルウェアの署名は次のとおりです: `cardActionMiddleware: () => next => ({ cardAction, getSignInUrl }) => next(cardAction)`                                                                                                                                                                                                                                                                                                                                           |
| `createDirectLine`         | Direct Line オブジェクトをインスタンス化するためのファクトリ メソッド。 Azure Government ユーザーは、エンドポイントを変更するために `createDirectLine({ domain: 'https://directline.botframework.azure.us/v3/directline', token });` を使用する必要があります。 パラメーターの完全なリストは、`conversationId`、`domain`、`fetch`、`pollingInterval`、`secret`、`streamUrl`、`token`、`watermark` `webSocket`です。                                                                                                                                                                                                                         |
| `createStore`              | 開発者がストアのアクションを変更できるようにする一連のミドルウェア。 ミドルウェアの署名は次のとおりです: `createStore: ({}, ({ dispatch }) => next => action => next(cardAction)`                                                                                                                                                                                                                                                                                                                                                                                                |
| `directLine`               | DirectLine オブジェクトを DirectLine トークンを使用して指定します。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `disabled`                 | Web チャットの UI (つまり、プレゼンテーション モード用) を無効にします。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `grammars`                 | 音声の文法リスト (Bing Speech または Cognitive Services Speech Services) を指定します。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `groupTimeStamp`           | タイムスタンプのグループ化の既定の設定を変更します。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `locale`                   | Web チャットの既定の言語を指定します。 4 文字のコード (`en-US` など) を強くお勧めします。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `renderMarkdown`           | 既定の Markdown レンダラー オブジェクトを変更します。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `sendTypingIndicator`      | ユーザーからボットへの入力信号を表示して、ユーザーがアイドル状態ではないことを示します。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `store`                    | プログラムによるアクティビティをボットに追加するなどのカスタム ストアを指定します。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `styleOptions`             | Web チャットのスタイル設定用のカスタマイズ値を格納するオブジェクト。 (頻繁に更新される) 既定のスタイル オプションの完全なリストについては、[defaultStyleOptions.js](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Styles/defaultStyleOptions.js) ファイルを参照してください。                                                                                                                                                                                                                                                                              |
| `styleSet`                 | スタイルをオーバーライドするための非推奨の方法。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `userID`                   | userID を指定します。 `userID` を指定するには、2 つの方法があります: プロパティで、またはトークン呼び出し (`createDirectLine()`) を生成するときにトークンで指定します。 両方の方法を使用して userID を指定した場合、トークンの userID プロパティが使用され、実行時に `console.warn` が表示されます。 `userID` がプロパティを介して提供されているが、`'dl'` が前に付いている場合 (例: `'dl_1234'`)、値はスローされ、新しい `ID` が生成されます。 `userID` が指定されていない場合は、既定でランダムなユーザー ID になります。 複数のユーザーが同じユーザー ID を共有することはお勧めできません。それらのユーザー状態は共有されます。 |
| `username`                 | ユーザー名を指定します。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `webSpeechPonyFillFactory` | テキスト読み上げおよび音声テキスト変換用に Web Speech オブジェクトを指定します。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
## <a name="browser-compatibility"></a>ブラウザーとの互換性
Web チャットは、Chrome、Edge、FireFox などの最近のブラウザーの最新の 2 つのバージョンをサポートしています。
Internet Explorer 11 で Web チャットが必要な場合は、[ES5 バンドルのデモ](https://microsoft.github.io/BotFramework-WebChat/01.b.getting-started-es5-bundle)を参照してください。

ただし、次の点に注意してください。
- Web チャットは、Internet Explorer 11 より前のバージョンをサポートしていません。
- ES5 以外のサンプルに示されているカスタマイズは、Internet Explorer ではサポートされていません。 IE11 は最新ではないブラウザーであるため、ES6 はサポートされていません。矢印関数と最新の promise を使用する多くのサンプルは、手動で ES5 に変換する必要があります。  アプリのカスタマイズを大幅に行う必要がある場合は、Google Chrome や Edge などの最新のブラウザー用にアプリを開発することを強くお勧めします。
- Web チャットは、IE11 (ES5) のサンプルをサポートする予定はありません。
   - 他のサンプルを IE11 で動作するように手動で書き換えるお客様は、[`babel`](https://babeljs.io/docs/en/next/babel-standalone.html) のようなポリフィルやトランスパイラーを使用した ES6+ から ES5 へのコードの変換を検討することをお勧めします。

## <a name="how-to-test-with-web-chats-latest-bits"></a>Web チャットの最新のビットを使用してテストする方法

*リリースされていない機能のテストは、現時点では MyGet パッケージを介してのみ可能です。*

まだリリースされていない機能やバグ修正をテストする場合は、Web チャット パッケージを、公式の npmjs フィードではなく、Web チャットの毎日のフィードをポイントするようにします。

現在は、MyGet フィードをサブスクライブすることによって、Web チャットの毎日のフィードにアクセスできます。 これを行うには、プロジェクトのレジストリを更新する必要があります。 **この変更は元に戻すことができ、指示には、公式リリースのサブスクライブに戻る方法が含まれています。** .

### <a name="subscribe-to-latest-bits-on-mygetorg"></a>`myget.org` で、最新のビットを購読します。

これを行うには、パッケージを追加してから、プロジェクトのレジストリを変更します。

1. Web チャット以外のプロジェクトの依存関係を追加します。
1. プロジェクトのルート ディレクトリに、`.npmrc` ファイルを作成します。
1. ファイルに次の行を追加します: `registry=https://botbuilder.myget.org/F/botframework-webchat/npm/`
1. プロジェクトの依存関係に Web チャットを追加します `npm i botframework-webchat --save`
1. あなたの `package-lock.json` では、指しているレジストリは MyGet になっています。 Web チャット プロジェクトでアップストリーム ソース プロキシが有効になっています。これにより、MyGet 以外のパッケージが `npmjs.com` にリダイレクトされます。

### <a name="re-subscribe-to-official-release-on-npmjscom"></a>`npmjs.com` の公式リリースを再購読します。
再購読するには、レジストリをリセットする必要があります。

1. `.npmrc file` を削除します。
1. ルート `package-lock.json` を削除します。
1. `node_modules` ディレクトリを削除します。
1. `npm i` を使用して、パッケージを再インストールします。
1. あなたの `package-lock.json` では、レジストリは再び https://npmjs.com/ を指しています。


## <a name="contributing"></a>寄与

プロジェクトのビルド方法およびプル リクエストのリポジトリのガイドラインの詳細は、[寄与のページ](https://github.com/Microsoft/BotFramework-WebChat/tree/master/.github/CONTRIBUTING.md) を参照してください。

このプロジェクトでは、[Microsoft オープン ソースの倫理規定](https://opensource.microsoft.com/codeofconduct/)を採用しています。
詳細については、[倫理規定についてよくある質問](https://opensource.microsoft.com/codeofconduct/faq/)を参照するか、[opencode@microsoft.com](mailto:opencode@microsoft.com) 宛てに質問またはコメントをお送りください。

## <a name="reporting-security-issues"></a>セキュリティの問題の報告

セキュリティの問題やバグは、[secure@microsoft.com](mailto:secure@microsoft.com) の Microsoft セキュリティ レスポンス センター (MSRC) にメールによって非公開で報告する必要があります。 24 時間以内に応答が返されます。 何らかの理由で応答がなかった場合、メールでフォローアップして、元のメッセージを私たちが受信したことを確認してください。 [MSRC PGP](https://technet.microsoft.com/security/dn606155) キーなどの詳細情報は、[Security TechCenter](https://technet.microsoft.com/security/default) を参照してください。

Copyright (c) Microsoft Corporation. All rights reserved.
