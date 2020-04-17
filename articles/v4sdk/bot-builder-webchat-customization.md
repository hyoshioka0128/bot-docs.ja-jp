---
title: Web チャットのカスタマイズ - Bot Service
description: Bot Framework の Web チャットをカスタマイズする方法について学習します。
keywords: bot framework, web チャット, チャット, サンプル, react, リファレンス
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 06/07/2019
ms.openlocfilehash: 5016c5810d2c3623d8e8556b0a96d717ee83000d
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75791164"
---
# <a name="web-chat-customization"></a>Web チャットのカスタマイズ

この記事では、ご利用のボットに合わせて Web チャットのサンプルをカスタマイズする方法について詳しく説明します。

## <a name="integrate-web-chat-into-your-website"></a>Web チャットを Web サイトに統合する

Web チャット コントロールをご利用の Web サイトに統合するには、[概要ページ](bot-builder-webchat-overview.md)の手順に従います。

## <a name="customizing-styles"></a>スタイルのカスタマイズ

Web チャット コントロールの最新バージョンでは、豊富なカスタマイズ オプションが提供されています。色、サイズ、要素の配置を変更したり、カスタムの要素を追加したり、ホスティング Web ページとやりとりしたりすることができます。 次に、Web チャット UI のそれらの要素をカスタマイズする方法の例をいくつか示します。

[`defaultStyleOptions.js` ファイル](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Styles/defaultStyleOptions.js)で Web チャットを簡単に変更できるすべての設定の完全なリストを見つけることができます。

これらの設定では、_スタイル セット_が生成されます。これは [glamor](https://github.com/threepointone/glamor) で強化された CSS ルールのセットです。 [`createStyleSet.js` ファイル](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Styles/createStyleSet.js)のスタイル セットで生成された CSS スタイルの完全なリストを見つけることができます。

## <a name="set-the-size-of-the-web-chat-container"></a>Web チャット コンテナーのサイズを設定する

`styleSetOptions` を使用して Web チャット コンテナーのサイズを調整できるようになりました。 次の例には、Web チャット コンテナーの表示のために、`body` で background-color が `paleturquoise` になっています (白い背景のセクション)。

```js
…
<head>
  <style>
    html, body { height: 100% }
    body {
      margin: 0;
      background-color: paleturquoise;
    }

    #webchat {
      height: 100%;
      width: 100%;
    }
  </style>
</head>
<body>
  <div id="webchat" role="main"></div>
  <script>
    (async function () {
    window.WebChat.renderWebChat({
      directLine: window.WebChat.createDirectLine({ token }),
      styleOptions: {
        rootHeight: '100%',
        rootWidth: '50%'
      }
    }, document.getElementById('webchat'));
    })()
  </script>
…
```

以下に結果を示します。

<img alt="Web Chat with root height and root width set" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/rootHeightWidth.png" width="600"/>

## <a name="change-font-or-color"></a>フォントまたは色を変更する

チャットの吹き出し内で使用される既定の背景色やフォントを使用するのではなく、ターゲットの Web ページのスタイルに合うようにそれらをカスタマイズできます。 以下のコード スニペットによって、ユーザーおよびボットからのメッセージの背景色を変更できます。

<img alt="Screenshot with custom style options" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-custom-style-options.png" width="396" />

一部のシンプルなスタイル指定を行う必要がある場合、`styleOptions` によって設定できます。 スタイルのオプションは、直接変更できる定義済みのスタイルのセットであり、Web チャットではそのオプションに基づいて全体のスタイルシートが計算されます。

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         const styleOptions = {
            bubbleBackground: 'rgba(0, 0, 255, .1)',
            bubbleFromUserBackground: 'rgba(0, 255, 0, .1)'
         };

         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  secret: 'YOUR_BOT_SECRET'
               }),

               // Passing 'styleOptions' when rendering Web Chat
               styleOptions
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

## <a name="change-the-css-manually"></a>手動で CSS を変更する

色だけでなく、次のようにメッセージのレンダリングに使用されるフォントを変更できます。

<img alt="Screenshot with custom style set" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-custom-style-set.png" width="396" />

より詳細なスタイル指定については、直接 CSS ルールを設定することで、手動でスタイル セットを変更することもできます。

> CSS ルールは DOM ツリーの構造に密に結合されるため、Web チャットのより新しいバージョンを使用するには、これらのルールを更新する必要がある可能性があります。

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         // "styleSet" is a set of CSS rules which are generated from "styleOptions"
         const styleSet = window.WebChat.createStyleSet({
            bubbleBackground: 'rgba(0, 0, 255, .1)',
            bubbleFromUserBackground: 'rgba(0, 255, 0, .1)'
         });

         // After generated, you can modify the CSS rules
         styleSet.textContent = {
            ...styleSet.textContent,
            fontFamily: "'Comic Sans MS', 'Arial', sans-serif",
            fontWeight: 'bold'
         };

         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  secret: 'YOUR_BOT_SECRET'
               }),

               // Passing 'styleSet' when rendering Web Chat
               styleSet
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

## <a name="change-the-avatar-of-the-bot-within-the-dialog-box"></a>ダイアログ ボックス内でボットのアバターを変更する

最新バージョンの Web Chat では、アバターがサポートされています。これは、`styleOptions` プロパティで `botAvatarInitials` および `userAvatarInitials` を設定することによってカスタマイズできます。

<img alt="Screenshot with avatar initials" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-avatar-initials.png" width="396" />

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         const styleOptions = {
            botAvatarInitials: 'BF',
            userAvatarInitials: 'WC'
         };

         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  secret: 'YOUR_BOT_SECRET'
               }),
               styleOptions
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

Web チャットの `styleOptions` プロパティ内に、`botAvatarInitials` と `userAvatarInitials` が追加されました。

```js
botAvatarInitials: 'BF',
userAvatarInitials: 'WC'
```

`botAvatarInitials` では、左側のアバター内のテキストが設定されます。 これが false に評価される値に設定されている場合、ボット側のアバターは表示されません。 その一方、`userAvatarInitials` は右側のアバターのテキストに設定されます。

## <a name="custom-rendering-activity-or-attachment"></a>アクティビティまたは添付ファイルのレンダリングをカスタマイズする

最新バージョンの Web チャットでは、Web チャットで組み込みがサポートされないアクティビティまたは添付ファイルをレンダリングすることもできます。 アクティビティと添付ファイルのレンダーは、[Redux ミドルウェア](https://redux.js.org/api/applymiddleware)の後にモデル化される、カスタマイズ可能なパイプラインを通じて送信されます。 パイプラインは十分に柔軟なので、次のタスクを簡単に行うことができます。

-  既存のアクティビティ/添付ファイルを装飾する
-  新しいアクティビティ/添付ファイルを追加する
-  既存のアクティビティ/添付ファイルを置き換える (または削除する)
-  ミドルウェアをまとめてデイジー チェーン

### <a name="show-github-repository-as-an-attachment"></a>GitHub リポジトリを添付ファイルとして表示する

GitHub リポジトリ カードのデッキを表示する必要がある場合、GitHub リポジトリに対する新しい React コンポーネントを作成して、添付ファイルにミドルウェアとして追加できます。

<img alt="Screenshot with custom GitHub repository attachment" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-custom-github-repository-attachment.png" width="396" />

```jsx
import ReactWebChat from 'botframework-webchat';
import ReactDOM from 'react-dom';

// Create a new React component that accept render a GitHub repository attachment
const GitHubRepositoryAttachment = props => (
   <div
      style={{
         fontFamily: "'Calibri', 'Helvetica Neue', Arial, sans-serif",
         margin: 20,
         textAlign: 'center'
      }}
   >
      <svg
         height="64"
         viewBox="0 0 16 16"
         version="1.1"
         width="64"
         aria-hidden="true"
      >
         <path
            fillRule="evenodd"
            d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0 0 16 8c0-4.42-3.58-8-8-8z"
         />
      </svg>
      <p>
         <a
            href={`https://github.com/${encodeURI(props.owner)}/${encodeURI(
               props.repo
            )}`}
            target="_blank"
         >
            {props.owner}/<br />
            {props.repo}
         </a>
      </p>
   </div>
);

// Creating a new middleware pipeline that will render <GitHubRepositoryAttachment> for specific type of attachment
const attachmentMiddleware = () => next => card => {
   switch (card.attachment.contentType) {
      case 'application/vnd.microsoft.botframework.samples.github-repository':
         return (
            <GitHubRepositoryAttachment
               owner={card.attachment.content.owner}
               repo={card.attachment.content.repo}
            />
         );

      default:
         return next(card);
   }
};

ReactDOM.render(
   <ReactWebChat
      // Prepending the new middleware pipeline
      attachmentMiddleware={attachmentMiddleware}
      directLine={window.WebChat.createDirectLine({ token })}
   />,
   document.getElementById('webchat')
);
```

完全なサンプルは、[customization-card-components サンプル](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/10.a.customization-card-components)で見つけることができます。

このサンプルでは、`GitHubRepositoryAttachment` と呼ばれる新しい React コンポーネントを追加します。

```jsx
const GitHubRepositoryAttachment = props => (
   <div
      style={{
         fontFamily: "'Calibri', 'Helvetica Neue', Arial, sans-serif",
         margin: 20,
         textAlign: 'center'
      }}
   >
      <svg
         height="64"
         viewBox="0 0 16 16"
         version="1.1"
         width="64"
         aria-hidden="true"
      >
         <path
            fillRule="evenodd"
            d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0 0 16 8c0-4.42-3.58-8-8-8z"
         />
      </svg>
      <p>
         <a
            href={`https://github.com/${encodeURI(props.owner)}/${encodeURI(
               props.repo
            )}`}
            target="_blank"
         >
            {props.owner}/<br />
            {props.repo}
         </a>
      </p>
   </div>
);
```

その後、ボットでコンテンツの種類 `application/vnd.microsoft.botframework.samples.github-repository` の添付ファイルを送信するときに、新しい React コンポーネントをレンダリングするミドルウェアを作成します。 それ以外の場合は、これは `next(card)` を呼び出すことにより、ミドルウェア上で続行されます。

```jsx
const attachmentMiddleware = () => next => card => {
   switch (card.attachment.contentType) {
      case 'application/vnd.microsoft.botframework.samples.github-repository':
         return (
            <GitHubRepositoryAttachment
               owner={card.attachment.content.owner}
               repo={card.attachment.content.repo}
            />
         );

      default:
         return next(card);
   }
};
```

ボットから送信されたアクティビティは、次のようになります。

```json
{
   "type": "message",
   "from": {
      "role": "bot"
   },
   "attachmentLayout": "carousel",
   "attachments": [
      {
         "contentType": "application/vnd.microsoft.botframework.samples.github-repository",
         "content": {
            "owner": "Microsoft",
            "repo": "BotFramework-WebChat"
         }
      },
      {
         "contentType": "application/vnd.microsoft.botframework.samples.github-repository",
         "content": {
            "owner": "Microsoft",
            "repo": "BotFramework-Emulator"
         }
      },
      {
         "contentType": "application/vnd.microsoft.botframework.samples.github-repository",
         "content": {
            "owner": "Microsoft",
            "repo": "BotFramework-DirectLineJS"
         }
      }
   ]
}
```
