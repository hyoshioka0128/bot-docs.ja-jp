---
title: Web チャットの概要 - Bot Service
description: Bot Framework の Web チャットを構成する方法について学習します。
keywords: bot framework, web チャット, チャット, サンプル, react, リファレンス
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 06/07/2019
ms.openlocfilehash: a85f1ddcb9a1440d8b157b3e28a7cbefb834955a
ms.sourcegitcommit: d24fe2178832261ac83477219e42606f839dc64d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/07/2020
ms.locfileid: "77071790"
---
# <a name="web-chat-overview"></a>Web チャットの概要

この記事には、[Bot Framework の Web チャット](https://github.com/microsoft/BotFramework-WebChat) コンポーネントの詳細が含まれています。 Bot Framework の Web チャット コンポーネントは、Bot Framework V4 SDK 用の高度にカスタマイズ可能な Web ベースのクライアントです。 Bot Framework SDK v4 により、開発者は、会話をモデル化して、高度なボット アプリケーションを構築できます。

Web チャット v3 から v4 への移行を検討している場合は、[移行のセクション](#migrating-from-web-chat-v3-to-v4)に進んでください。

## <a name="how-to-use"></a>使用方法

> [!NOTE]
> 以前のバージョンの Web チャット (v3) の場合は、[Web チャット v3 のブランチ](https://github.com/Microsoft/BotFramework-WebChat/tree/v3)を参照してください。

まず、[Azure Bot Service](https://azure.microsoft.com/services/bot-service/) を使用してボットを作成します。
ボットが作成されたら、Azure portal で[ボットの Web チャット シークレットを取得](../bot-service-channel-connect-webchat.md#get-your-bot-secret-key)する必要があります。 シークレットを使用して、[トークンを生成し](../rest-api/bot-framework-rest-direct-line-3-0-authentication.md)、それを Web チャットに渡します。

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

> `userID`、`username`、`locale`、`botAvatarInitials`、`userAvatarInitials` はすべて、`renderWebChat` メソッドに渡すためのオプションのパラメーターです。 Web チャットのプロパティの詳細については、この記事の「[Web チャット API リファレンス](#web-chat-api-reference)」セクションを参照してください。
> ![Web チャットのスクリーンショット](https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/weatherquery.png.jpg)

### <a name="integrate-with-javascript"></a>JavaScript との統合

Web チャットは、JavaScript または React を使用して既存の Web サイトと統合するように設計されています。 JavaScript と統合することで、一部のスタイル設定とカスタマイズ可能性を得ることができます。詳細については、「[Web チャットを Web サイトに統合する](https://aka.ms/integrate-webchat-into-site)」の記事をご覧ください。

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

完全なカスタマイズ性を得るためには、[React](https://reactjs.org/) を使用して Web チャットのコンポーネントを再構成できます。

npm から運用ビルドをインストールするには、`npm install botframework-webchat` を実行します。

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

> [!TIP]
> React と jsx を初めて使用する場合は、Reacts の「[概要](https://reactjs.org/docs/getting-started.html)」ページにトレーニングがあります。


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

> [!NOTE]
> コンテンツ配信ネットワーク (CDN) の詳細については、「[コンテンツ配信ネットワーク (CDN)](https://aka.ms/CDN-best-practices)」をご覧ください。

## <a name="migrating-from-web-chat-v3-to-v4"></a>Web チャット v3 から v4 への移行

v3 から v4 に移行する際に、移行される可能性があるパスは 3 つあります。 最初に、以下のように、ご自分の始めのシナリオを比較してください。

関連するサンプルの一覧については、「[Web チャットでホストされるサンプル](https://aka.ms/botframework-webchat-samples)」を参照してください。

### <a name="my-current-website-integrates-web-chat-using-an-iframe-element-obtained-from-azure-bot-services-i-want-to-upgrade-to-v4"></a>私の現在の Web サイトは、Azure Bot Services から取得された `<iframe>` 要素を使用して Web チャットを統合します。 v4 へのアップグレードを希望しています。

2019 年 5 月から、`<iframe>` 要素を使用して Web チャットを統合する Web サイトに v4 を展開しています。 `<iframe>` を使用した Web チャットの統合については、[組み込みドキュメント](https://github.com/Microsoft/BotFramework-WebChat/tree/master/packages/embed)を参照してください。

### <a name="my-website-is-integrated-with-web-chat-v3-and-uses-customization-options-provided-by-web-chat-no-customization-at-all-or-very-little-of-my-own-customization-that-was-not-available-with-web-chat"></a>私の Web サイトは Web チャット v3 と統合されており、Web チャットによって提供されるカスタマイズ オプションを使用するか、カスタマイズをまったく使用しないか、または Web チャットで使用できなかった独自のカスタマイズをほとんど使用しません。

サンプルの [`00.migration/a.v3-to-v4`](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/00.migration/a.v3-to-v4) の実装に従って、Web ページを Web チャットの v3 から v4 に変換してください。

### <a name="my-website-is-integrated-with-a-fork-of-web-chat-v3-i-have-implemented-a-lot-of-customization-in-my-version-of-web-chat-and-i-am-concerned-v4-is-not-compatible-with-my-needs"></a>私の Web サイトは、Web チャット v3 のフォークと統合されています。 自分のバージョンの Web チャットに多くのカスタマイズを実装しましたが、v4 が私のニーズと互換性がないことを懸念しています。

Web チャットの v4 に関して私たちのチームが気に入っていることの 1 つは、**Web チャットをフォークする必要なく**カスタマイズを追加できることです。 これにより、以前 Web チャットをフォークした v3 ユーザーに追加のオーバーヘッドが発生しますが、私たちは、バージョンアップを行うお客様をサポートするために最善を尽くします。 次の推奨事項を使用してください。

-  サンプルの実装[`00.migration/a.v3-to-v4`](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/00.migration/a.v3-to-v4)をご覧ください。 これは、Web チャットを起動して実行するための優れた出発点です。
-  次に、[サンプル リスト](https://aka.ms/botframework-webchat-samples)を調べて、カスタマイズ要件と、Web チャットが既にサポートしているものを比較してください。 これらのサンプルは、Web チャットでよく寄せられる機能で構成されています。
-  サンプルの 1 つ以上の機能が利用できない場合、[未解決の問題と終了した問題](https://github.com/Microsoft/BotFramework-WebChat/issues?utf8=%E2%9C%93&q=is%3Aissue+)、[サンプル ラベル](https://github.com/Microsoft/BotFramework-WebChat/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+label%3ASample)、[移行サポート ラベル](https://github.com/Microsoft/BotFramework-WebChat/issues?q=is%3Aissue+migrate+label%3A%22Migration+Support%22)を調べて、探している機能のサンプル リクエストやカスタマイズ サポートを検索してください。 未解決の問題にコメントを追加すると、チームは需要の高い要求に優先順位を付けるのに役立ちます。また、コミュニティへの参加をお勧めします。
-  未解決の要求のリストにご自身の機能が見つからない場合は、お気軽に[ご自分の独自の要求を提出](https://github.com/Microsoft/BotFramework-WebChat/issues/new)してください。 上記の項目と同様に、未解決の問題にコメントを追加する他のお客様によって、Web チャット ユーザー間で最も一般的に必要とされる機能を優先させることができます。
-  最後に、できるだけ早く機能が必要な場合は、Web チャットへの[プル要求](https://github.com/Microsoft/BotFramework-WebChat/compare)を歓迎します。 この機能を自分で実装するコーディング経験がある場合は、追加のサポートをいただけますと幸いです! 自分で機能を作成すると、Web チャットで迅速に使用できるようになり、同じ機能または類似の機能を探している他のお客様があなたの貢献を利用できます。
-  v4 に関する詳細については、この `README` の残りの部分を必ずご確認ください。


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

このプロジェクトは、「[Microsoft のオープン ソースの倫理規定](https://opensource.microsoft.com/codeofconduct/)」を採用しています。
詳細については[論理規定についてのよくある質問](https://opensource.microsoft.com/codeofconduct/faq/)をご覧ください。また、追加の質問やコメントがある場合は[opencode@microsoft.com](mailto:opencode@microsoft.com)にお問い合わせください。

## <a name="reporting-security-issues"></a>セキュリティの問題の報告

セキュリティの問題やバグは、[secure@microsoft.com](mailto:secure@microsoft.com) の Microsoft セキュリティ レスポンス センター (MSRC) にメールによって非公開で報告する必要があります。 24 時間以内に応答が返されます。 何らかの理由で応答がなかった場合、メールでフォローアップして、元のメッセージを私たちが受信したことを確認してください。 [MSRC PGP](https://technet.microsoft.com/security/dn606155) キーなどの詳細情報は、[Security TechCenter](https://technet.microsoft.com/security/default) を参照してください。

Copyright (c) Microsoft Corporation. All rights reserved.
