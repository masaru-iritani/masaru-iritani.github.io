---
title: html2rssをWindows Subsystem for Linux (WSL)にインストールする
categories: diary
---

気付いたら、ウェブページからCSSセレクタでRSSを生成する[Feed Creator](https://blog.mochi.is/p/important-update-changes-to-feed)が有料化されてしまっていた。無料なのに高機能なサービスだったので、RSSを生成しないウェブサイトの更新情報を[newsboatで購読する](https://masaru-iritani.github.io/diary/2024/04/07/getting-onboard-newsboat.html)ために愛用していた。有料化されてしまうのは仕方無いが、月間9ドルは少々高い。

そこで、Ruby製のコンソールアプリケーションである `html2rss` に乗り換えることにする。Windows Subsystem for Linux (WSL)で動かすべく作業する。簡単かと思ったら、随分長い手順が必要になってしまった。

## 結論

多分これでWindows Subsystem for Linux (WSL)に `html2rss` をインストールできる（が、試行錯誤の上に成功した結果から逆算した手順なので、抜け漏れや不要な手順が有るかもしれない）。

```zsh
sudo apt install build-essential libffi-dev libyaml-dev libz-dev
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
~/.rbenv/bin/rbenv init
omz reload # or exec $SHELL for bash
git clone https://github.com/rbenv/ruby-build.git "$(rbenv root)"/plugins/ruby-build
rbenv install 3.4.2
rbenv global 3.4.2
gem update
gem install html2rss
```

## 試行錯誤の記録

Rubyが入っていなかったのでインストール。

```zsh
sudo apt install ruby
```

ビルドツールが足りないと言われたので、[Ruby公式サイトで紹介されていた](https://www.ruby-lang.org/en/documentation/installation/#apt) `ruby-full` を試してみる。

```zsh
sudo apt install ruby-full
```

エラーメッセージをよく見ると `gcc` で失敗している。`which gcc` も失敗するので、[検索で引っ掛かったページ](https://linuxize.com/post/how-to-install-gcc-on-ubuntu-20-04/)を参考に `build-essential` をインストールする。

```zsh
sudo apt install build-essential
```

これでインストールできた。

```zsh
sudo gem install rss2html
```

だが、`rss2html` の実行ファイルが見付からない。

```zsh
> rehash

> which html2rss
html2rss not found

> gem list -d html2rss

*** LOCAL GEMS ***

html2rss (0.9.0)
    Author: Gil Desmarais
    Homepage: https://github.com/gildesmarais/html2rss
    License: MIT
    Installed at: /var/lib/gems/3.0.0

    Returns an RSS::Rss object by scraping a URL.

> find /var/lib/gems/3.0.0 -type f -executable | grep html2rss
/var/lib/gems/3.0.0/gems/html2rss-0.9.0/bin/console
/var/lib/gems/3.0.0/gems/html2rss-0.9.0/bin/setup

> gem specification html2rss executables
--- []
```

`html2rss` のGitHub Issuesを確認すると、0.10.0で実行ファイルをインストールできるようにしたよ、と[書いてある](https://github.com/html2rss/html2rss/issues/161#issuecomment-2259043284)。バージョンが古すぎるようだ。

>Finally released a new version. You can install it with `gem install html2rss`.
>
>[rubygems.org/gems/html2rss/versions/0.10.0](https://rubygems.org/gems/html2rss/versions/0.10.0)

Rubyのバージョンについても、[3.1すら最早非対応で3.2以上が必要](https://github.com/html2rss/html2rss/pull/224)らしい。折角なら最新安定板の3.4.2を入れよう。

`sudo apt update && sudo apt upgrade` してもバージョンが変わらないので、どうやら `rbenv` とやらでインストールする必要があるらしい。

```zsh
> git clone https://github.com/rbenv/rbenv.git ~/.rbenv
...

> ~/.rbenv/bin/rbenv init
writing ~/.zprofile: now configured for rbenv.
```

`.zprofile` に追記されてしまったので、それを消して代わりに `.zshrc` の `ohmyzsh` 設定に `rbenv` を[追加する](https://github.com/masaru-iritani/dotfiles/commit/1aada9adbb3dbb92c5d638d80f387240c844db80)。

`rbenv install -l` したら入れろと言われた `ruby-build` もインストールする。

```zsh
> git clone https://github.com/rbenv/ruby-build.git "$(rbenv root)"/plugins/ruby-build
...

> rbenv install -l
3.1.6
3.2.7
3.3.7
3.4.2
jruby-9.4.12.0
mruby-3.3.0
picoruby-3.0.0
truffleruby-24.1.2
truffleruby+graalvm-24.1.2

Only latest stable releases for each Ruby implementation are shown.
Use `rbenv install --list-all' to show all local versions.
```

これでやっとRuby 3.4.2がインストールできた……と思ったらビルド失敗。

```zsh
> rbenv install 3.4.2
...
BUILD FAILED (Ubuntu 22.04 on x86_64 using ruby-build 20250215)
...
```

`zlib.h` が無いと言われている。

```
crypto/comp/c_zlib.c:36:11: fatal error: zlib.h: No such file or directory
   36 | # include <zlib.h>
      |
```

適当に当たりを付けて `libz-dev`（正しくは `libz-dev` っぽい？）をインストール。

```zsh
> sudo apt install libz-dev
...
```

再挑戦したが今度は `libffi` が無いと言われる。`libffi-dev` をインストール。

```zsh
> rbenv install 3.4.2
...
*** Following extensions are not compiled:
fiddle:
        Could not be configured. It will not be installed.
        /tmp/ruby-build.20250224210212.14900.3rXsJY/ruby-3.4.2/ext/fiddle/extconf.rb:86: missing libffi. Please install libffi or use --with-libffi-source-dir with libffi source location.
        Check /tmp/ruby-build.20250224210212.14900.3rXsJY/ruby-3.4.2/ext/fiddle/mkmf.log for more details.
psych:
        Could not be configured. It will not be installed.
        Check /tmp/ruby-build.20250224210212.14900.3rXsJY/ruby-3.4.2/ext/psych/mkmf.log for more details.

BUILD FAILED (Ubuntu 22.04 on x86_64 using ruby-build 20250215)

> sudo apt install libffi-dev
```

またエラー。

```zsh
> rbenv install 3.4.2
...
*** Following extensions are not compiled:
psych:
        Could not be configured. It will not be installed.
```

ログファイルを見ると `yaml.h` が無いと言う。

```
conftest.c:3:10: fatal error: yaml.h: No such file or directory
    3 | #include <yaml.h>
      |          ^~~~~~~~
compilation terminated.
```

`libyaml-dev` とか `libyml-dev` は一見無さそうに見えたので、どのパッケージをインストールして良いか分からなかった。ChatGPTに訊いたら `apt-file` で分かると言われる。`apt-file` をインストールして検索してみたところ、`libyaml-dev` がヒットした。あれ、やっぱり `libyaml-dev` だった。見落としていたようだ。

```shell
> sudo apt install apt-file
...

> sudo apt-file update
...

> rehash

> apt-file search yaml.h | grep 'yaml.h$'
...
libyaml-dev: /usr/include/yaml.h
...

> sudo apt install libyaml-dev
...
```

今度こそ成功した！

```shell
> rbenv install 3.4.2
...
==> Installed ruby-3.4.2 to /home/.../.rbenv/versions/3.4.2

NOTE: to activate this Ruby version as the new default, run: rbenv global 3.4.2

> rbenv global 3.4.2

> omz reload

> ruby --version
ruby 3.4.2 (2025-02-15 revision d2930f8e7a) +PRISM [x86_64-linux]
```

あれ、でもインストールされる `html2rss` は 0.9.0 のままだ。

```
> sudo gem install html2rss
Successfully installed html2rss-0.9.0
Parsing documentation for html2rss-0.9.0
Done installing documentation for html2rss after 0 seconds
1 gem installed

> which gem
/home/.../.rbenv/shims/gem

> sudo gem update
...
ERROR:  Error installing activesupport:
        activesupport-8.0.1 requires Ruby version >= 3.2.0. The current ruby version is 3.0.2.107.
...
extconf.rb:10:in `<main>':   Can't find libcurl or curl/curl.h (RuntimeError)
...
```

古いバージョンのRubyが検出されているのも気になるが、`libcurl` の問題の方が浅そうなので、先にこちらを解決する。

```
> sudo apt install libcurl-dev
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Package libcurl-dev is a virtual package provided by:
  libcurl4-openssl-dev 7.81.0-1ubuntu1.20
  libcurl4-nss-dev 7.81.0-1ubuntu1.20
  libcurl4-gnutls-dev 7.81.0-1ubuntu1.20
You should explicitly select one to install.

E: Package 'libcurl-dev' has no installation candidate
```

うーん、どれをインストールしたら良いんだろ。faraday-patronのGitHub Issuesを検索すると、`libcurl4-openssl-dev` で解決したという[報告が上がっていた](https://github.com/lostisland/faraday-patron/issues/4#issuecomment-1006909127)ので、素直にそれを採用する。

```zsh
> sudo apt-get install libcurl4-openssl-dev
```

失敗ログを眺めていると他にも足りないライブラリが有るので、インストールする。

```zsh
> sudo apt-get install libreadline-dev
```

OpenSSLでも失敗しているが、`libopenssl-dev` パッケージが無さそうなので、まずは足りないヘッダーファイル名を確認する。

```
conftest.c:3:10: fatal error: openssl/ssl.h: No such file or directory
    3 | #include <openssl/ssl.h>
      |          ^~~~~~~~~~~~~~~
```

さっきChatGPTに教えてもらった `apt-file` で検索だ。`libssl-dev` が一番普通っぽい。

```zsh
> sudo apt-file search "openssl/ssl.h"
android-libboringssl-dev: /usr/include/android/openssl/ssl.h
libnode-dev: /usr/include/node/openssl/ssl.h
libssl-dev: /usr/include/openssl/ssl.h
libwolfssl-dev: /usr/include/cyassl/openssl/ssl.h
libwolfssl-dev: /usr/include/wolfssl/openssl/ssl.h
python3-pycparser: /usr/share/python3-pycparser/fake_libc_include/openssl/ssl.h

> sudo apt install libssl-dev
...
```

やっぱり駄目だった……が、よく考えると、`sudo` で実行される `gem` は `rbenv` でインストールされたものじゃないのかも。そう思って `sudo` を外したらビンゴ！ついに `html2rss` がインストールできた。ついでにおすすめされた `gem update` もしておく。

```zsh
> gem update
...

> gem install html2rss
...
Done installing documentation for zeitwerk, concurrent-ruby, tzinfo, thor, nokogiri, crass, sanitize, reverse_markdown, regexp_parser, websocket-extensions, websocket-driver, mime-types-data, mime-types, puppeteer-ruby, parallel, kramdown, faraday-net_http, faraday, faraday-follow_redirects, public_suffix, addressable, html2rss after 8 seconds
22 gems installed

A new release of RubyGems is available: 3.6.2 → 3.6.5!
Run `gem update --system 3.6.5` to update your installation.

> gem update --system 3.6.5
...

> rehash

> html2rss --help
...
Commands:
  html2rss auto URL                                      # Automatically sources an RSS feed from the URL
  html2rss feed YAML_FILE [FEED_NAME] [param=value ...]  # Print RSS built from the YAML_FILE file to stdout
  html2rss help [COMMAND]                                # Describe available commands or one specific command
```

## 追記

この記事は、最近導入した[Obsidian](https://obsidian.md/)に記録していたノートをそのままJekyllに持ってきたものである。ObsidianはNotionとかなり近い操作感でありつつも、カーソル付近のMarkdown記法が自動展開されるので、表記の調整がやりやすい。そして何と言ってもVimキーバインディングが使えるので、一瞬で好きになってしまった。ちょろい。