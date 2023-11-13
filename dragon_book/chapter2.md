## 2.1

### 構文と意味

構文的には正しいが、意味的に"正しくない"例。

```ruby
def plus_one(i)
  i + 1
end

plus_one(1, 2)
```

```
9 - 5 + 2

=>

# 9から5を引いたものに2を足す(逆ポーランド記法)
9 5 - 2 +
```

最初は '+', '-', '0' - '9'だけを扱うことで、字句解析を飛ばして構文解析から話を始める。
字句解析は2.6から登場する。


## 2.2

順序が逆転したことをいうと、CFGで左辺にこないものは終端記号です。
正確には逆で、CFGとは生成文法において全生成規則の左辺が非終端記号のもののことをさします。
実務的にはlexerが生成するのが終端記号という理解。

### P.47 コラム

厳密には token = {token名, 属性値(semantic value)}. E.g. {tINTEGER("integer literal"), 1}.

```
// この全体が文法

// 開始記号
program : stmts
        ;

stmts   : stmts stmt
        ;

stmt    : ...
```

### ε(イプシロン)とは

```
arg         : arg '?' arg opt_nl ':' arg

opt_nl		: /* none */ <- これとか
            | '\n'
            ;
```

生成規則のあつまりが文法。文法が生成することのできる記号列の集合が言語。

生成規則: `arg: arg '+' arg`
文法: parse.y
言語: `1 + 1`, `bool ? a : b` ...


### 構文解析とは

`9 - 5 + 2`を例にすると。

```
// 例2.2
list
list + digit
list + 2
list - digit + 2
list - 5 + 2
digit - 5 + 2
9 - 5 + 2
```

### 木構造
接点(node), 辺(edge)

ここで
* 葉はそれより先が導出できない = 終端記号
* 内部節点は必ず導出ができる = 非終端記号


### association(結合規則)とprecedence(優先順位)

https://www.gnu.org/software/bison/manual/html_node/Why-Precedence.html

"右結合性をもつ a=b=cのような記号列"の誤りだと思う。

list & digitの例では `lsit : list +/- digit` のように左方向にしか伸びない。
例2.5ではこれをstringにまとめてしまったので `string : string +/- string` で、左にも右にも伸びるようになっており、曖昧性が生じる。
letter & rightの例では `right : letter = right`なので右方向にしか伸びない。


つまり優先順位の高い低いは生成規則の"上下"もしくは"細かさ"としてあらわれる, 非公式的な言い方だけど開始規則(program)に近いほうが上、もしくは細かくないという感覚がある。
構文木にしたときにより根に近い方から計算するので。

結合規則は生成規則のなかで演算子の左右どちらに左辺の非終端があらわれるかで表現することができる。

### 問題2.2.1

```
S : S S +
  | S S *
  | a
  ;


S
S S *
S a *
S S + a *
S a + a *
a a + a *
```

### 問題2.2.2

a) 1個以上の0がn個ならび、その後n個の1がならぶ


## 2.3

属性文法の話っぽい。

https://ja.wikipedia.org/wiki/%E5%B1%9E%E6%80%A7%E6%96%87%E6%B3%95

// post(E)
post(const) = const
post(var) = var
post(e1 op e2) = post(e1) post(e2) op
post('(' e ')') = e


### 例2.8

post('(' 9 - 5 ')' + 2)
= post('(' 9 - 5 ')') post(2) +
= post(9 - 5) post(2) +
= post(9) post(5) - post(2) +
= 9 post(5) - post(2) +
= 9 5 - post(2) +
= 9 5 - 2 +

### 合成属性

すごく雑にいうとBisonのアクションの部分。

P.59あたりがわかりにくと思うので具体例をあげると

```
# digit is_a Integer
# term  is_a String
# expr  is_a String

expr : expr + term { $$ = $1 + $2 + "+" }
     | expr - term { $$ = $1 + $2 + "-" }
     | term        { $$ = $1 }
     ;

term : digit       { $$ = $1.to_s }
     ;
```

#1はたとえば

* digit 終端記号は属性を1つもつ、それはIntegerである
* term 非終端記号は属性を1つもつ、それはStringである
* expr 非終端記号は属性を1つもつ、それはStringである

#2は`term : digit`の意味規則では digit 終端記号に関連付けられている属性(Integer)に#to_sすることでStringに変換し、左辺のtermの属性を計算する。`expr : expr + term`の意味規則では expr, term に関連付けられている属性(どちらもString)をconcatしてさらに末尾に"+"を追加して、左辺のexprの属性を計算する。

P.63 上の例は本書でいう"構文主導翻訳スキーム"を先取りしてしまっていますね... もっと愚直に書くべきだった

P.63 この右辺の途中にアクション ({ print('+') }) をかけるようにすると、その部分をεな生成規則として切り出すような実装になるんだよなぁと思いました。

## 2.4

なんで"再帰下降法"なんだよ！とおもいました。
O(n^3)はEarley parserとかCYK algorithmですね。
https://makenowjust-labs.github.io/blog/post/2023-08-06-pike-earley

P.66のtop downとbottom upの説明、すき。

exprを終端記号にするの大胆ですき。

### P.67

つまり、開始記号はstmtなので、4つの生成規則のうちから1つを選ぶという意味。
図2.17ではforから始まる3つ目の生成規則を選んでいる。

入力が "for ( ; expr ; expr) other" のとき、最初の状態はstmtである。
いま最初の終端記号が for なので3つ目の生成規則を選ぶ。

ここではstmtの生成規則のうちforからはじまるものがちょうど一つなので問題ないが、では複数あるときはどうするのか。
いくつか選択肢があって

1. そのような文法ははじく
2. 2つ以上のtokenを先読みする
3. バックトラックをみとめる

### P.69

ここでいう"手続き"は関数とかメソッドだととらえるとよい。

### P.72

#1 非終端記号 A = stmtだとする。stmtには4つの生成規則がある。

1. FIRST(expr ;) = {expr}
2. FIRST(if '(' expr ')' stmt) = {if}
3. FIRST(for '(' optexpr ; optexpr ; optexpr ')' stmt) = {for}
4. FIRST(other) = other

next token, ここでは lookahead 変数, が

* ifであれば #2の生成規則
* forであれば #3の生成規則
* '('であれば syntax error, stmtはε-生成規則がないので

コードをコピーする話はP.81を参照。

#### 左再帰

exprの解析のためにexprをよび、それがさらにexprをよび...

```
// 左再帰している
expr : expr + term
     | term
     ;


// していない
expr : term R
     ;

R    : + term R
     | ε
     ;
```

つまり

```
expr = expr + term
expr = expr + term + term
expr = expr + term + term + term
...

=>

expr = term R
expr = term + term R
expr = term + term + term R
...
```

## 2.5

### P.75

すくなくともこのテキストでは解析木(内部節点が非終端記号な木)のことを具象構文木というのか。
なるほどとおもったけど、'('のような葉は残るのか?

あー。抽象構文木が単一生成規則やε生成規則を残さないというのは確かにそう。

> 翻訳スキームは、解析木と構文木とができるだけ接近したものになるような文法を基にすることが望ましい。

はい。そのとおりだと思います。

## 2.6

* token: tINTEGER
* 字句: 12345

のような関係。

### P.84

空白を完全に取り除いていいと思っていた時期が私にもありました(最近はparserの生成物からコメントや空白を取得したいという人が増えてきた)

構文解析器が気にしなくていいはそのとおり

よくみかける、改行なら line += 1 のコード。

文字の先読みは https://github.com/ruby/ruby/blob/2ac3e9abe98579261a21a2e33df16f1bff1ebc1d/parse.y#L10191 など

Rubyでも基本はlexerが行を保持する構造です。

### P.85

'+'と'-'を合わせて終端記号opとして <op, +> や <op, -> とすることも可能。Rubyの場合 tOP_ASGN (&&=) などがそう。

### P.86

ソフトキーワードというのがありますね...

```ruby
def for
  p :for
end

self.for
```

Rubyでもkeywordが生成されるのは `parse_ident` (ident = identifier)です。

https://github.com/ruby/ruby/blob/52709a4862b91f2c7c0f0555d29f21ec3ed942d0/parse.y#L10337

単一表現という点ではroslynなどのGreen Nodeがそういう構造という理解。

## P.89

これ `NUM = 256` からはじまるの、8bitのcharはそのまま使う想定な気がする(Bisonもそういう仕様)。

脚注7にそう書いてあった。

## 2.7

ようするにスコープとそれを表現するためのテーブルの話。

```
int x; // 宣言
x;     // (使用)文
```

```
{
    int x;
    char y;

    {
        bool y;
        x;
        y;
    }

    x;
    y;
}
```

yはブロックの内外で異なる型をもつのでそれを管理しましょう。xは外側の宣言だけなので、ブロックのなかでその宣言をちゃんと参照しましょう。という例。

Rubyでいうとclassやmethod定義が変数のスコープをつくる。

https://github.com/ruby/ruby/blob/52709a4862b91f2c7c0f0555d29f21ec3ed942d0/parse.y#L2714



### P.94

コラムは字句解析器ではnestを扱えないという話をしている。

```
block: '{' decls stmts '}' ;
```

って宣言と文を混ぜて書けないのか。

Rubyだとlocal_push関数が参考になる。 https://github.com/ruby/ruby/blob/2ac3e9abe98579261a21a2e33df16f1bff1ebc1d/parse.y#L13184

parser_paramsは現在のlocal_varsへの参照をもち https://github.com/ruby/ruby/blob/2ac3e9abe98579261a21a2e33df16f1bff1ebc1d/parse.y#L13208

local_varsはひとつ前のlocal_varsへの参照をもつ https://github.com/ruby/ruby/blob/2ac3e9abe98579261a21a2e33df16f1bff1ebc1d/parse.y#L13191

local_pushがよばれているのは

* programの解析開始時(mainの空間)
* def_name、つまりメソッド定義
* k_class、つまりクラス定義
* k_module、つまりモジュール定義

対になる関数はlocal_pop。

### P.98

図2.38の `top` の管理が完全にRubyの `in_rescue` とかと同じで面白い。

## 2.8

### P.100

木構造に対して、線形表現という言い方をするのは面白いですね。

3アドレスコードは `x = y op z` の形をしているので3アドレス (P.45より)

基本ブロックについては P.573を参照。

### P.104

```c
int i;
```

があったときに、この宣言には

* 変数iが宣言されたこと (宣言されていない変数へのアクセスを検知するため)
* その型はintであること

の2つの効果があるが、それはどちらも記号表に書いておけば十分ということ。

### P.105

一方で `==` などは左右のオペランドの型に関する制約は等しいが、その戻り値がboolになるなどの違いがある。

コンパイラの後のフェーズからの要求が基になって決まるっていうのは、ちゃんとユースケースを調べろということですね...

### P.107

TaPLに"構文主導"という言い方が書いてあった気がする。

### P.109

急に compile.cになった

`a[i] = b` も左辺値のアドレスを計算して一時変数にいれればよいのではとおもったけど、多分そういう話ではない。

```
expr: a[i] = 2 * a[j-k]

rvalue(expr):
=> xはRel

t = rvalue(a[i]) '=' rvalue(2 * a[j-k])


  右辺:
  => xはOp

  t1 = rvalue(2) '*' rvalue(a[j-k])

    右辺の右辺:
    => xはAccess

    t2 = lvalue(a[j-k])

      lvalue(a[j-k]):
      => Access(a, rvalue(j-k))
      =  Access(a, t3 = j - k)

    t2 = Access(a, t3 = j - k)

```


