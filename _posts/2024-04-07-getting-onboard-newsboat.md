---
title: newsboat を用いた RSS 購読環境構築
category: diary
locale: ja_JP
---

![newsboat のスクリーンショット](/assets/2024-04-07-getting-onboard-newsboat.png)


RSS (Really Simple Syndication) 購読に長らく使っていた Feedly から [newsboat](https://newsboat.org/) に乗り換えた。newsboat の痒い所に手が届く設定が私の要件を満たし、結果としてより快適な RSS 購読環境を構築することができた。その顛末をここに記す。

## 背景

事の発端は Feedly の有料サービスである Feedly Pro+ の請求がクレジットカードの海外ショッピング限度額を超えてしまったことだ。年課金なのでそんな事も有るだろうと請求額を見てみると、その額 24,000円。以前は $144 で 15,000円前後であったような気がしていたので、急な高騰に驚いた。今調べたら米ドルでは年間 $144 のままのようなので、ここ最近の円安が大きく響いている格好だ。

ここで契約を更新するべきかを考え直す。Feedly Pro+ を契約したのは、次の機能を享受するためだった。

- RSS を提供しないサイトの購読を可能にする RSS Builder
- 特定単語を含む記事を非表示にするフィルター
- 電子メールアドレス発行によるメールマガジンの受信

しかし、よくよく考えてみると次のような問題に悩まされていた。

- RSS Builder や電子メールアドレスで購読している配信元は合わせても片手で数える程度で、費用対効果が低い。
- RSS Builder は XPath や CSS セレクターによる調整ができず、ウェブページによっては件名が上手く抽出できない。
- フィルターはカテゴリー単位の設定であり、意図せず同一カテゴリー内の別ウェブサイトの記事が非表示になる。
- 完全一致や正規表現による厳密なフィルター設定はできず、日本語の処理も不正確で、非表示にしたい記事が表示されてしまう。

総じて詳細を隠して賢くやろうという方向性が見えて、それが嵌る人にとっては簡便なのだろう。一方、開発者気質で何もかも自身の手で制御したい自分とは相性が合わなくなっているように思われた。

そこで私は重い腰を上げて、自分にとって最適な RSS 購読環境を再構築することにしたのだった。

## 要件

実現したいと思っていたことは次の通り。

- 全フィードからの記事を一覧表示できる。
- [Pocket](https://getpocket.com/ja/) に記事を追加できる。
- RSS を提供しないウェブサイトを解析して購読できる。
- 特定の文字列を件名に含む記事を非表示にできる。
- できれば無料が良いが、納得できる額なら支払う用意は有る。

## 候補

まず最初に移行先の候補となったのは、他の RSS 購読ウェブサービスだ。

[Inoreader](https://www.inoreader.com/?lang=ja_JP) が有名なようで、実際にアカウントを作って少しばかり試してみた。見た目は確かに綺麗なのだが、無料アカウントのせいか動作にもっさり感がある。随分前の記憶になるが Feedly では無料版でもかなり高速で、それに慣れてしまった自分には少々常用が厳しいと思えた。彼らもそれを知ってか知らずか日本のミラーサーバーを提供しているが、そちらに切り替えても差は感じられなかった。

他にも幾つか同様の機能を提供するウェブサービスは在るが、いずれも私が Feedly Pro+ を契約してまで欲しかった RSS 構築やフィルターのような機能は大抵有料版にのみ搭載されている。そもそも、自分は RSS を購読を専ら単一端末から行っているため、ウェブサービスである必要性が薄い。複数端末から使うのであればフィードの一覧や既読状態を同期してくれるウェブサービスの強みが活かせるが、そうでないなら寧ろ費用や拡張性の低さといった欠点が目立つ。

そこでローカル環境で動く RSS リーダーを物色し始めた。GUI アプリケーションも視認性が高くて便利だが、結局キーボード操作や拡張性を考えるとコンソールアプリケーションに軍配が上がる。

[russ](https://github.com/ckampfe/russ) はその中でも興味深いクライアントで、Rust で書かれたコンソールアプリケーションだ。明示的に指示されたことだけを行う (<q>Russ will only do these things when you tell it to</q>) という方針は好感が持てる。Vi キーバインドによる動作もかなり軽快だ。ただ、機能については実装方針も含めてまだまだ発展途上という状況で、採用は見送った。

## newsboat との邂逅

最終的に移行先探しの決め手になったのは ChatGPT だった。[ローカル環境で RSS を構築する方法を尋ねた](https://chat.openai.com/share/c6b47621-3845-4807-a5bc-2fa0fd4e363f)ところ返ってきたのが newsboat だった。回答の詳細は不正確だったが、名前から手掛かりを得て調べたところ、高い拡張性で要求をほぼ全て満たせることが分かった。

### 全フィード一覧表示

newsboat では、特定条件に引っ掛かる記事の検索結果を 1 つのフィードとして登録できる。ここで全記事に合致する条件を記述すれば、全フィードを一覧表示することができる。現時点では自分の全購読フィードはウェブにあるので、`urls` に次の 1 行を足せば全記事を含む "\*" フィードが誕生する。

```
"query:*:rssurl =~ \"^http\""
```

幸い現在の購読フィードに文字コード順で "\*" より先に来るものは無いので、フィード一覧を名前順にすれば常に "*" が先頭に来る。起動時に `open` コマンドを自動実行すれば、自動で先頭の "\*" フィードが開かれて全記事一覧が表示されるようになる。

```
feed-sort-order "title"
prepopulate-query-feeds yes
run-on-startup reload-all;open
```

### Pocket 連携

任意のプログラムを実行できるブックマーク機能を活用する。Pocket に記事を登録するシェルスクリプト [pocket-sh-add](https://github.com/riichard/pocket-sh-add) を登録すれば、Ctrl + B で記事を Pocket に送れる。

```
bookmark-autopilot yes
bookmark-cmd "~/pocket-sh-add/add.sh"
```

### RSS 構築

fivefilters が提供する [Feed Creator](https://createfeed.fivefilters.org/) を利用することにした。無料版では記事数やフィルター数にこそ制限は有るものの、CSS セレクターによる正確な記事抽出が可能で、困ることは無かった。

newsboat は [RSS を吐くプログラムをフィードとして登録できる](https://newsboat.org/releases/2.31/docs/newsboat.html#_scripts_and_filters_snownews_extensions)ので、いざという時にはウェブページを解析するプログラムを登録してやれば良いという安心感が有る。

### フィルター

私の購読している YouTube チャンネルは通常動画の切り抜きをショート動画として公開していて、内容が重複している。ショート動画を一律非表示にするため次のようなフィルターを定義した。

```
ignore-article "*" "title =~ \"#Shorts\""
ignore-article "*" "title =~ \"#shorts\""
```

### 見た目と操作感の調整

次の設定でかなり自分好みに仕上げた結果が冒頭のスクリーンショットである。フォントは 0xProto Nerd Font を利用している。

```
articlelist-format "%?n?&? %D %t%?T? &?%T"
bind-key ; cmdline
bind-key g home
bind-key h prev-dialog
bind-key j next
bind-key k prev
bind-key l next-dialog
bind-key t toggle-show-read-feeds
bind-key z set-tag
bind-key G end
bind-key LEFT bookmark
bind-key ^d down
bind-key ^u up
browser "'/mnt/c/Program\ Files/Mozilla\ Firefox/firefox.exe' --new-tab %u"
color listnormal color244   default
color listnormal_unread color15 default
datetime-format "%F"
feedlist-format "%?n?&? %t"
feedlist-title-format "%N %V (%u)"
articlelist-title-format "%N %V - %T (%u)"
goto-next-feed no
highlight article "^(Feed|Title|Author|Link|Date):.*$" color244 default
unbind-key D
```

### メールマガジン購読

これは流石にメールアドレスを自分で用意して、そのメールを取得するプログラムを準備する必要がある。自分の場合はそこまで必要性の無い購読が 1 件だけだったので、これを機に購読を止めてしまった。

## 結論

newsboat に移行して 1 週間経つが、年間 1 万円以上を支払っていた Feedly Pro+ よりも快適に RSS を購読できるようになったと感じている。使い慣れた環境を切り替えるのは億劫だが、慣れてしまった日々の問題を解消できる余地は無いか、これからも時折見直していきたい。
