---
title: Flutter製のテキサスホールデムポーカーのiOS/Androidアプリをリリースしました
description: Flutterで作ったiOS/Androidアプリを初めてストアに出しました。テキサスホールデムというポーカーの計算機で、複数人のハンドやハンドレンジからそれぞれ誰がどれくらいの勝率があるかを計算できるものです。
cover_image_url: https://user-images.githubusercontent.com/4289883/115946128-feebe980-a473-11eb-966c-0ebd54c3e569.png
tags:
  - flutter
first_published_at: '2019-11-07T00:00Z'
last_published_at: '2019-11-07T00:00Z'
---
[Flutter](https://flutter.dev/)で作ったiOS/Androidアプリを初めてストアに出しました。テキサスホールデムというポーカーの計算機で、複数人のハンドやハンドレンジからそれぞれ誰がどれくらいの勝率があるかを計算できるものです。

![UIインタラクション](https://user-images.githubusercontent.com/4289883/115946161-2ba00100-a474-11eb-9556-d9bc73d00db6.png)

## ダウンロード

[:google-play](https://play.google.com/store/apps/details?id=app.axross.aqua&hl=ja) [:app-store](https://apps.apple.com/jp/app/odds-calculator-for-poker/id1485519383`)

オープンソースにしています。**IssueやStarなどでのフィードバックは励みになるので是非よろしくお願いします。**

[](https://github.com/axross/aqua)

## 実装について

### マッチアップの計算

マッチアップは所謂モンテカルロシミュレーションで、一定回数の計算をランダムに試行して平均勝率を算出します。

ポーカーでは相手が持っているカードが何なのかをプレイ中に知ることはできず、複数の読みの候補をもとに判断を下していきます。このアプリはその判断を下す手助けをするツールなので、複数の読みを考慮した計算ができるようになっています。

たとえば相手の持っているハンドがAとKの組み合わせ、AとQの組み合わせ、AとJの同じスート (マーク) の組み合わせのどれかであるという読みの場合、AKが16通り、AQが16通り、AJが4通りで合計36通りあります。このうちAQとAJに対しては勝っていてAKには負けているのであれば勝率は16/36で約44.4%です。

この計算に次のファクターを加えます。

- コミュニティカード (中央に開かれる全プレイヤー共通のカード) の枚数
- 相手プレイヤーが持っていうるカードの組み合わせ数
- 相手プレイヤーの人数

マッチアップを全通り計算する時間複雑度は次のようになり、全通り試行するのはおよそ現実的ではないことがわかります。マッチアップの選択をランダムにしているのはこのためです。

```
n = カードの枚数 (52枚)
m = プレイヤーの人数

時間複雑度 = O((n^5)(n^2^m))
```

### 勝率計算

記事を書いた時点から何回か改善を加えているので大まかな方針だけですが、次のようになっています。

1. コミュニティカードをランダムに選択する
2. 各プレイヤーの持っているカードにコミュニティカードを加え、最強になる5枚の組み合わせをフォールスルーで探し、強さを数値化する
3. プレイヤーの強さの数値の中から最強のものを探し、勝利フラグを立てる
4. プレイヤーごとに勝利した回数を全試行回数で割り、勝率とする

勝率計算の1回はマイクロ秒単位で終わりますが、充分な精度で求めようとすると膨大な回数の試行が必要になります。数百回単位でチャンクとし、チャンクごとに別スレッドで計算するようにしてUIスレッドを止めないようにする必要があります。この手の並列処理には[`Isolate`](https://api.dartlang.org/stable/2.6.0/dart-isolate/dart-isolate-library.html)を使うと便利です。

### UIインタラクション

![Flutter Tween Animation Example](https://videos.ctfassets.net/2mfcuy3p355s/4jPAo53HynNcYyE32mgJhV/1f83b17f483853f2bb903cd61ec05043/RPReplay_Final1574145129__2___1_.mp4) ![Flutter Tween Animation Example](https://videos.ctfassets.net/2mfcuy3p355s/5bER0UCiqkHJH1JyOx7f38/903960c488a9f719f6ab3063b51adfe4/RPReplay_Final1574145112__2___1_.mp4)

Flutterは逐次的なシーケンスアニメーションと、値のグラデーションによるツウィーンアニメーションの2種類が扱えます。UIインタラクションに使うのは通常ツウィーンアニメーションの方で、これはFlutterの `UI=F(S)` モデルとうまく協調して動くAPIで作られています。

```dart
class SlidingButton extends StatefulWidget {
  _SlidingButtonState createState() => _SlidingButtonState();
}

class _SlidingButtonState extends State<SlidingButton> with SingleTickerProviderStateMixin {
  Animation<double> animation;
  AnimationController controller;

  @override
  void initState() {
    super.initState();

    controller = AnimationController(duration: const Duration(milliseconds: 300), vsync: this);
    animation = CurvedAnimation(parent: controller, curve: Curves.easeInOut);
  }

  @override
  Widget build(BuildContext context) =>
      SlideTransition(
        position: Tween<Offset>(
          begin: const Offset(0, 0.125),
          end: const Offset(0, 0),
        ).animate(curvedAnimation),
        child: Button(
          child: Text("Tap to slide"),
        ),
      );

  @override
  void dispose() {
    controller.dispose();

    super.dispose();
  }
}
```

### メディアクエリ

Flutterには[MediaQuery](https://api.flutter.dev/flutter/widgets/MediaQuery-class.html)というAPIがあり、これによって端末やOSの色々な情報が取得できます。

- ディスプレイのサイズや向き
- 太いフォントを好んで利用するかどうか (iOS)
- 24時間のタイムフォーマットを利用しているかどうか

これを利用することでダークモードもUIに簡単に反映できます。

![Dark theme](https://user-images.githubusercontent.com/4289883/115946173-470b0c00-a474-11eb-8fb5-efd2029c6567.png)

実際に利用する際は、アプリケーションのできるだけ先祖に近い箇所でMediaQueryを取得するようにして [`InheritedWidget`](https://api.flutter.dev/flutter/widgets/InheritedWidget-class.html) などを使って子孫ウィジェットに伝搬していく作りにするとよいと思います。 `InheritedWidget` は[ReactのContext](https://ja.reactjs.org/docs/context.html)に似た伝播モデルを司るウィジェットです。

### リリース

さほど難しくないですが、やっぱりiOSとAndroidでの開発は経験しておいた方がいいです。コーディングに関する部分はFlutterによってうまく抽象化されているのでまったくiOS/Androidの知識がなくても扱えるようになっています。ですがリリースに際しての証明書での署名や難読化、デバイスのアーキテクチャごとのビルドなど、各プラットフォームの知識があった方がいい箇所もあります。
