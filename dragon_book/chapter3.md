# 3

* 字句解析器生成系
  * Flexなど
  * CRubyは手書きのlexer(parser_yylex in parse.y)
    * https://github.com/ruby/ruby/blob/f1b95095d6635567cc5820b3eb40d9618faa73ed/parse.y#L10392
  * Lramaも手書きのlexerではあるが、StringScannerが強力なので字句解析器生成系に近い書き心地だと思う
    * https://github.com/ruby/lrama/blob/master/lib/lrama/lexer.rb
* 字句パターン
  * `int [0-9]+` のような記述のこと
  * https://github.com/akimd/bison/blob/25b3d0e1a3f97a33615099e4b211f3953990c203/src/scan-gram.l#L155-L158

## 3.1

### 概要

"場合によっては、構文解析器を支援するために、字句解析器が、識別子の種類に関する情報を記号表から読み取って、構文解析器に引き渡す適切なトークンを決定することもある。"
  * 近いことをやっている例 https://github.com/ruby/ruby/blob/f1b95095d6635567cc5820b3eb40d9618faa73ed/parse.y#L10375


(Cのような行指向の)マクロ展開がある場合はpreprocessをもう一段前に挟むかなぁ


### 3.1.1

どうしてLexerとparserを分けるのか?

1. これは2つのことなるロジックを用意することはサポートするけど、2つに分離することはサポートしない気がする。Rubyの場合 heredoc をparserでやるのは大変なので同意はする。
2. この実行速度の改善は現代でどのくらい効果があるのか
3. これはfile以外のケース or デバイスが特殊なケース or 文字コード?


### 3.1.2

* トークン
  * トークン名: tINTEGER
  * 属性値: 1
* パターン: `[0-9]+`
* 字句: "1 + 2"

### 3.1.3

このFortranの例、厳しい...

### 3.1.4

字句解析器レベルでエラーが出せる例としては

```
% ruby -e "@ 1"
-e:1: `@' without identifiers is not allowed as an instance variable name
-e:1: syntax error, unexpected integer literal, expecting end-of-input
-e: compile error (SyntaxError)
```

修復方法の4は思いつかなかった(実装したくない)。大体のケースでIDENTIFIERになりうるんだよなぁ

## 3.2

CRubyの場合は lex_getline で次の1行を取得する。
https://github.com/ruby/ruby/blob/f1b95095d6635567cc5820b3eb40d9618faa73ed/parse.y#L7475

ファイルのときと文字列(evalや -e option)のときの差を吸収するために実際の処理は`p->lex.gets`関数の呼び出しになっている。
`lex_get_str`や`lex_io_gets`を参照。

RubyのLexer Buffer
https://github.com/ruby/ruby/blob/f1b95095d6635567cc5820b3eb40d9618faa73ed/parse.y#L417-L425


## 3.3

### 3.3.1

文字種のことをアルファベットと言いがち。
|s| で文字列長、ε(イプシロン)で長さがゼロの記号列というのもよくある書き方。

### 3.3.5

先読みとかじゃなくてよかった。

## 3.4

### 3.4.1

この1文字戻す操作はRubyでいうと`pushback`。

### 3.4.2

Rubyは#2を採用していて、一番最後に`parse_ident`が呼ばれる。
https://github.com/ruby/ruby/blob/f1b95095d6635567cc5820b3eb40d9618faa73ed/parse.y#L11108

## 3.5

Flexへの入力ファイルの例: https://github.com/akimd/bison/blob/25b3d0e1a3f97a33615099e4b211f3953990c203/src/scan-gram.l

`%{%}`ではBisonなどの生成するheader fileを読み込むことがおおい。これはheader fileにtokenの定義が書かれているため。

https://github.com/ruby/lrama/blob/7b9313067011fd7f4babe972b1ca53ef1b84a52d/spec/fixtures/integration/named_references.l#L7

例3.13 は流石に先読みしすぎではないか...

## 3.6

### 3.7.1 NFAからDFA

P.166

DFAの開始状態をAとする。
AはNFAの状態0を含む。

A = {0}

状態0から1回のε遷移で移動できる{1, 7}を含む。
{1, 7}から1回のε遷移で移動できる{2, 4}を含む。
{2, 4}から1回のε遷移で移動できるところはない。

よって A = {0, 1, 2, 4, 7}

DAFの状態Aから文字aで遷移した状態、状態Bについて考える。

B = {3, 8}

状態3から1回のε遷移で移動できる{6}を含む。
{6}から1回のε遷移で移動できる{1, 7}を含む。
{1, 7}から1回のε遷移で移動できる{2, 4}を含む。

よって B = {1, 2, 3, 4, 6, 7, 8}

### 3.7.3

これは k(n+m) が十分に小さいときに有効(P. 163のNFA-DFA変換が直接のシミュレーションよりも時間が掛かるという状況)

### 3.7.4

正規表現 -> NFA

基底ステップの部分式aって文字? 文字列?
"正規表現技術入門" P.124をみたところ、このaは文字。




