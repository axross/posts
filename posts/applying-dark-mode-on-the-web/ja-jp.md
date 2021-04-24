---
title: Webでダークモードに対応する
description: ダークモードは2018年にmacOSで実装されたのを皮切りに、今では主要なOSすべてに実装されています。有機ELディスプレイでの寿命やバッテリーの持ちを改善する効果が期待でき、これから徐々に一般的になっていくことが予想できます。この記事ではウェブサイトをダークモードに対応させる方法と注意点を紹介します。
cover_image_url: https://user-images.githubusercontent.com/4289883/115945142-1d9bb180-a46f-11eb-9a22-8c5bb0d0351e.png
tags:
  - web frontend
first_published_at: '2019-08-24T00:00Z'
last_published_at: '2019-08-24T00:00Z'
---
:::callout{variant="warning"}
このウェブサイトは何回かのアップデートを重ねており、以下で紹介するスクリーンショットの中には古いものがあります。ですが技術的な内容そのものには変更ありません。
:::

axross.devをダークモードに対応させました。

## ダークモードとは

ダークモードはユーザーのOS・ブラウザの設定または端末に応じて表示を黒背景に変える仕組みです。iOS、Android、macOS、WindowsなどのOSの設定でダークモードに設定している人は既にこのページが黒背景で表示されていると思います。ダークモードをオフにすると白背景で表示されます。

![ダークモードに対応しました](https://user-images.githubusercontent.com/4289883/115945189-5b98d580-a46f-11eb-940e-43999f2ea52d.png)

### ダークモードはアクセシビリティの機能

ダークモードは単なる「Webサイトの色設定」ではありません。**ユーザーと端末が求めている色でWebサイトを表示する**機能と考えることができます。

OSのUIが全体的に黒背景を貴重としているのにWebサイトだけ白背景なのは不自然です。また、黒背景で慣れた目に急に白背景が飛び込んでくるとダメージを感じるかもしれません。

![macOS Catalinaでのダークモードの設定](https://user-images.githubusercontent.com/4289883/115945235-969b0900-a46f-11eb-925d-ac467ae848c4.png)

最新のiOSやmacOSの設定には日の入りから日の出まで自動的にダークモードをオンにするオプションがあります。多くのモバイルデバイスのOSに搭載されているNight LightやNight Shift (ブルーライトの軽減を目的として時間帯によってディスプレイの色味を変える機能) と同じように生活に馴染んでいくと思います。

### ダークモードはユーザーの電池持ちを良くする

昨今のモバイルデバイスにはディスプレイに有機ELを使っているものが多いです。有機ELはバックライトを用いずドットごとに素子が発光するので、暗い色ほど消費エネルギーが少なくなります。Googleの調査によるとダークモードの利用によって端末の電池消費を50%以上削減できた例もあります。

[](https://japanese.engadget.com/2018/11/09/el-google/)

## ダークモードを実装する

メディアクエリで [`prefers-color-scheme`](https://developer.mozilla.org/ja/docs/Web/CSS/@media/prefers-color-scheme) を利用するとダークモードがオンになっているかどうかを判別できます。次のコードはダークモードがオンになっている時は背景が `#000` 、オフの時は
`#fff` になります。

```css
html {
  background-color: #fff;
}

@media (prefers-color-scheme: dark) {
  html {
    background-color: #000;
  }
}
```

ダークモードを実装する時はまずカラースキームを作りましょう。実際には背景だけではなくテキストやアイコンの色、ボタンやテキスト入力部品の背景色やボーダー、UI部品によってはドロップシャドウの色など、それまで白背景を貴重に作られていたすべての色に対して、黒背景用のものを用意しましょう。

## JavaScriptで判別する

メディアクエリなので [`window.matchMedia()`](https://developer.mozilla.org/ja/docs/Web/API/Window/matchMedia) で判別できます。

```js
// ダークモードならtrue、そうでなければfalse
window.matchMedia("(prefers-color-scheme: dark)").matches
```

ダークモードのオン・オフの切り替えを監視するには、 `window.matchMedia()` から得られる [`MediaQueryList`](https://developer.mozilla.org/ja/docs/Web/API/MediaQueryList) の `.addEventListener()` を利用します。次のコードはReactのContextを通じてダークモードかどうかを子孫コンポーネントに伝搬するコードです。

```jsx
const DarkModeContext = React.useContext();

function DarkModeProvider({ children }) {
  const [isDarkMode, setIsDarkMode] = React.useState(false);

  React.useEffect(() => {
    // "prefers-color-scheme: dark" に対するMediaQueryListを取得
    const mediaQueryList = window.matchMedia("(prefers-color-scheme: dark)");

    // 次の行で使うイベントハンドラ
    // ダークモードがオンならe.matchesはtrueになり、オフならfalseになる
    const listener = e => setDarkMode(e.matches);

    // ダークモードのオン・オフが切り替わった時にchangeイベントが発火される
    mediaQueryList.addEventListener("change", listener);

    // このコンポーネントがレンダーツリーから外れた時にはchangeイベントへの監視を解除する
    () => mediaQueryList.removeEventListener("change", listener);
  });

  // Contextを通じて子孫要素にダークモードかどうかを伝搬する
  return (
    <DarkModeContext.Provider value={isDarkMode}>
      {children}
    </DarkModeContext.Provider>
  );
}

function SomeComponent() {
  // コンテキストを通じてダークモードかどうかを判別する
  const isDarkMode = React.useContext(DarkModeContext);

  return isDarkMode ? <AnotherDarkComponent /> : <AnotherLightComponent />;
}
```

:::callout{variant="warning"}
JavaScriptのみでダークモードを実装するのはできるだけ控えてください。なぜならダークモードは端末やOSの設定に依存し、サーバーサイドレンダリングの際にその情報を取得することができず、サーバーでのレンダリング結果とリハイドレート後のレンダリングが異なる結果になってしまうからです。

できるだけCSSのメディアクエリ (`@media`) を利用してダークモードを実装するのをおすすめします。
:::

## `prefers-color-scheme` の対応状況

モダンブラウザはEdgeを除き対応しています。例のごとくInternet Explorerは非対応です。

![prefers-color-schemeの対応状況](https://user-images.githubusercontent.com/4289883/115945328-12955100-a470-11eb-8231-757522d533c1.png)

## 関連リンク

::embed{href="https://web.dev/prefers-color-scheme"}
