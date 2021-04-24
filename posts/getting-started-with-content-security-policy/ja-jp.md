---
title: Content Security Policyを設定してウェブサイトをXSSから守る
description: Content Security Policyを設定してウェブサイトをXSSから守る
cover_image_url: https://user-images.githubusercontent.com/4289883/115945746-e4187580-a471-11eb-982b-24945103d02d.png
tags:
  - web security
first_published_at: '2019-12-09T15:00Z'
last_published_at: '2019-12-09T15:00Z'
---
ある企業の面接を受けた時にContent Security Policyについて質問されて、あまりきちんと答えられなかったので調査しつつ、このウェブサイトで試験的に利用してみることにしました。調査している中で色々わかったので共有します。

Content Security Policyは画像、動画、JavaScript、CSSなどのアセットの読み込みや、`<iframe>` やWeb Workerの読み込み、XHRやFetchによるリソースの読み込みをポリシーの宣言によって制限できるものです。

インラインのCSSやJavaScriptの実行も制限でき、ユーザーが書き込んだコンテンツに混入してしまった `<script>` や `<button onclick="">` によるXSSも防止できます。

## ディレクティブ
Content Security Policyはディレクティブと呼ばれるキーバリュー形式のセットによって設定します。ディレクティブを設定する方法は2種類あります。

- HTTPレスポンスヘッダで設定する
    - `Content-Security-Policy: <directive>; <directive>`
- `<meta>` 要素で設定する
    - `<meta http-equiv="Content-Security-Policy" content="<directive>; <directive>">`

### ディレクティブの種類 (抜粋)
- `default-src` : 以下の設定のフォールバック先となるもの。たとえばFetchは `connect-src` を参照した後、対応するものがなければ `default-src` を参照する
- `connect-src` : XHRやFetchによる読み込み
- `script-src` : JavaScriptの読み込み
    - `<script src="...">` による読み込みだけでなく、 `<script> ... </script>` や `<button onclick="alert(1)">` のようなインラインスクリプトの実行を抑制する設定もある
- `style-src` : CSSの読み込み
    - `script-src` と同じように `<link href="..." rel="stylesheet" type="text/css">` による読み込みだけでなく `<style>...</style>` や `<button style="...">` のようなインラインスタイルもカバーする
- `img-src` : `<img>` とFaviconの読み込み
- `media-src` : `<video>` と `<audio>` による読み込み
- `prefetch-src` : `<link rel="prefetch">` や `<link rel="prerender">` による読み込み
- `frame-src` : `<iframe>` による読み込み

## `Content-Security-Policy-Report-Only`
`Content-Security-Policy` をHTTPリクエストヘッダまたは `<meta>` 要素で設定すると、ポリシーに抵触したリソースの読み込みや実行がすべてブロックされます。ポリシーの宣言が間違っていると、最低限必要なCSSやJavaScriptが実行されず、ウェブサイトそのものが利用できないということになりかねません。

例えばGoogle Analyticsを利用しているウェブサイトではGoogle Analyticsが行う通信を許可する必要があります。Google Analyticsは `<img>` 要素によるGETリクエストで通信するので、通信先のオリジンを `img-src` に追加しなければなりません。これを忘れると、Google Analyticsに一切データが集まらなくなってしまいます。

`Content-Security-Policy` の代わりに `Content-Security-Policy-Report-Only` を利用すると、ポリシーに抵触した際にブロックせず、代わりに報告するのみになります。`Content-Security-Policy-Report-Only` にはまったく同じ内容を設定できるので、HTTPリクエストヘッダで設定している場合はヘッダ名を置き換えるだけで動作します。ポリシーにどう抵触したかは `report-uri` ディレクティブに宣言したURLにJSONで送信され、ChromeであればDeveloper ToolsのConsoleタブにも表示されます。

![Content Security Policy on Console](https://user-images.githubusercontent.com/4289883/115945811-378ac380-a472-11eb-8629-8e18e7d935ff.png)

これは宣言したポリシーが正しいか開発環境で確認する時や、テスト段階の際にプロダクションでデータを集めたりするのに役立ちます。

https://report-uri.com のような `Content-Security-Policy-Report-Only` のレポートの送信先として利用できるWebサービスもあります。集められたポリシー抵触をダッシュボードで確認できて便利です。

![The Dashboard of report-uri.io](https://user-images.githubusercontent.com/4289883/115945817-41acc200-a472-11eb-9977-cc5de96f4bd4.png)

## `default-src 'none'` が一番が厳しい設定
Content Security Policyの宣言はディレクティブごとにホワイトリストです。例えば `img-src 'self'` にしておくと画像ファイルの読み込みは同一オリジンからのみ許可する設定となりますが、そもそも `img-src` ディレクティブを宣言していなければあらゆる画像ファイルの読み込みが許可されます。

潜在的なXSSの実行を防ぐためには、まずは `default-src` ディクレクティブを宣言し、 `'none'` をソースとして指定しておくのと良いです。Content Security Policyが以下のように `default-src 'none'` のみになっていれば、これが最も厳しいポリシーです。何の読み込みも許可されません。

```
Content-Security-Policy default-src 'none'
```

`default-src` はいろいろなリソース読み込みのフォールバック先となるので、 以下のように `img-src` を追加すると、 `https://google.com` からの画像ファイルのみ許可し、他はすべてブロックされます。

```
Content-Security-Policy default-src 'none'; img-src https://google.com
```

このような形で `Content-Security-Policy-Report-Only` を駆使して、必要なディレクティブとオリジンだけを宣言していって最小の設定にするのがセキュリティの最も高い設定になります。 

## `script-src` と `default-src` にの設定にはより注意
ひとたびXSSが発生すると、あらゆる方法でのリソースの読み込みや実行が考えられます。とくに、以下のソースの利用は気をつけてください。いずれもXSSによって発生させられる可能性のある読み込み/実行のソースです。

- `'unsafe-inline'`
- `'unsafe-eval'`
- `data:`

とくに `'unsafe-inline'` をインラインスクリプトやインラインスタイルのために利用するのは避けましょう。代わりに、インラインスクリプトやインラインスタイルの中身の文字列のSHA256ハッシュ値を取得し、 `'sha256-<hashed value>'` という形でソースとして利用できます。

以下は例として、このウェブサイトのGoogle Analytics用のインラインスクリプトを許可しているディレクティブです。この形式でなら実行されてもよいインラインスクリプトを個別に指定できます。

```
Content-Security-Policy: script-src 'sha256-6DELbQJmrBPpBmoPBeNHhSHAD6sidc72qGApkgX4m0E='
```

当然ホワイトスペースや改行文字の有無でハッシュ値が変わるので注意しましょう。Google AnalyticsやFacebookなどのサードパーティのSDKのインラインスクリプトをフロントエンドで利用している時は、ひとまず何も許可せずにChromeのDeveloper ToolsのConsoleタブを見ると良いです。抵触したインラインスクリプトをハッシュ化して自動生成されたソースが確認でき、これをそのまま利用するだけで設定できて便利です。

![Content Security Policy Source Auto Generating](https://user-images.githubusercontent.com/4289883/115945831-4e311a80-a472-11eb-8c8f-a534701eeceb.png)

## Web Extensionsが抵触することがある
ChromeやSafariなどの拡張機能であるWeb ExtensionsはWebページ内のDOMに干渉でき、表示の拡張を目的として画像ファイルを利用することがあります。これもContent Security Policyのチェック項目に含まれ抵触することがあります。

多くのWeb Extensionsはオフラインでも利用できるようにローカルファイルや `data:image/png,...` といったData URIで画像ファイルを読み込みます。中でもData URIで画像ファイルを読み込んでいるものは対処が難しいです。

Data URIはMIME Typeとコンテンツを自由に設定できるので、例えば `data:text/html,<script>alert('hi');</script>` とするとXSS攻撃が可能です。そのため `data:` をContent Security Policyのソースとして許可するわけにはいきません。

![Content Security Policy Blocks Web Extensions](https://user-images.githubusercontent.com/4289883/115945836-58ebaf80-a472-11eb-8f3e-55ca79f68dd6.png)

![Web Extensions Use Data URI Images](https://user-images.githubusercontent.com/4289883/115945849-63a64480-a472-11eb-80d7-4fce221776d5.png)

これは僕も解決法を見つけていないので、知っている方がいればぜひご一報ください...。
