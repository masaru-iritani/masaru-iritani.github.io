---
title: 靴下とテストデータの選び方
category: diary
image: /assets/2024-02-12-choosing-socks-and-test-data.webp
locale: ja_JP
---

![DALL-E が生成したカラフルな靴下の山から目的の一足を探す人の画像](/assets/2024-02-12-choosing-socks-and-test-data.webp)

前職で得た最も価値ある学びは、良いソフトウェアテストの設計でも C++ の超絶技巧でもなく、「靴下は同じものを買うと良い」という先輩からの教えだったような気がする。曰く、靴下は全く同じものを買っていれば、干した後や着替えの際に組み合わせを考える必要が無くなり日常が効率化されるのだ、と。それまで何となくという理由だけでばらばらの靴下を買っていた自分は、その助言に深い感銘を受け、以来靴下は黒の無地と決めている。

そんな靴下同様、何となくという理由で変えてしまいがちだったものにテストデータがあった。例えばテストデータに氏名が必要なとき、何となく「福沢諭吉」「樋口一葉」「野口英世」のようにばらばらな値を設定してしまいがちだった。

最近、その無意識な選択を思い直すきっかけとなった出来事があった。現職でテストデータに関して議論した際、ランダムな文字列を使うと「どこにフォーカスしているテストなのか分かりづらくなる」という意見が出たのだ。考えてみると、確かにその通りだ。

例えば、あるテストケースで氏名のテストデータとして「福沢諭吉」が使われていたとする。ばらばらな値が使われていると、「福沢諭吉」であることに必然性があるのか、あるいはたまたま適当な値が割り振られただけなのか、判断が難しい。人名は基本「福沢諭吉」で統一すると決まっていれば、「福沢諭吉」は任意の人名を表しているに過ぎないとすぐ理解できる。

テストデータが統一されていれば、その選定に悩む必要も無くなる。かのスティーブ・ジョブズが服装を選ぶ手間を惜しんで常に同じ格好をしていたというのは有名な話だが、それと同じことがテストデータにも言える。余計な決断を減らすことで、悩むべき本質的な課題に時間的、精神的な資源を集中できる。

同一のテストデータは変更の手間も減らせる。例えば、氏名を姓と名で分けるようになった場合でも、テストデータが統一されていれば一括置換ですぐに対応可能だ。これがばらばらになると、複数回の一括置換が必要になるし、変更漏れも生じやすい。「新渡戸稲造」が 1 箇所だけで使われていることにどうやって気付けるだろう。コンパイルエラーになるような形式変更であればまだ良いが、そうでなければテストが失敗して初めて気付くことになる。

もちろん、全人類が同じ靴下を履くべきではないように、テストデータを統一するのが常に正しいわけではない。細部のお洒落を楽しみたい人が状況に応じて様々な靴下を楽しむように、理由が有れば適切にテストデータを使い分ける必要がある。名前を変更できることを確かめるテストケースでは、「福沢諭吉」と「樋口一葉」がそれぞれ変更前と変更後の名前のテストデータとして使われることになるだろう。この場合でも、人名が基本「福沢諭吉」で統一されていれば、「樋口一葉」は別の名前である必然性があるデータなのだとすぐに理解できる。

良いコードは平凡な処理と特殊処理の違いを浮き彫りにする。テストデータも決してその例外ではない。しばしば遊ばれがちな（かつ私もそれを楽しんでいる）テストデータだが、その選び方一つで見えない手間を省くことだってできるのだ。