# Efficient Computation of LALR(1) Look-Ahead Sets

https://dl.acm.org/doi/pdf/10.1145/69622.357187

## 1. INTRODUCTION

## 2. TERMINOLOGY

## 3. LALR LOOK-AHEAD SETS

### 各種の集合

* LA(State s, Rule r) = {Token t}
  sでrをつかってreduceするときの先読みtokenの集合
  実際にreduceしてNterm Aを生成し戻った先のstate s'(この関係をlookbackという)に関して Follow(s, A) を考える必要がある
  ここで考えているのは State s = a A ... • のようなケース

* Follow(State s, Nterm A) = {Token t}
  sからAでgotoしたあとにreduceをしたとする(そのためγ =>* εが条件にはいっている), reduce後の状態s'でshift可能なtokenも加味した集合.
  Readはgotoのみを考慮しているのにたいして、Followはreduce後のことも考慮している
  Read同様、ここでも考えているのは State s = a • A ... のようなケース

* Read(State s, Nterm A) = {Token t}
  sからAでgotoしたさきの状態s'でshift可能なtokenの集合. ただしs'でnullableなNterm Bのあとに続くtokenも含める
  Readはgotoのみを考慮している
  ここで考えているのは State s = a • A ... のようなケース

* DR(State s, Nterm A) = {Token t}
  sからAでgotoしたさきの状態でshift可能なtokenの集合

### 各種のRelation

* lookback between (State s, Rule r) and (State s', Nterm A)
  r: A -> α とする。s'からαの遷移でsに遷移するという関係を表現したもの

* includes between (State s, Nterm A) and (State s', Nterm B)
  reduceした前後のstateの関係を表現したもの
  reduceをしたときにstackを一定量巻き戻すことになるが、その巻き戻しをした結果到達するstate(複数ありえる)のこと
  `(s, A) includes (s', B) iff B -> βAγ, γ =>* ε, s' -(β)-> s`

* reads    between (State s, Nterm A) and (State s', Nterm B)
  sからAでgotoしたさきの状態s'でgoto可能なBのうちnullableなもの

### 集合とRelationの関係、もしくは集合の計算方法

* LA     := (Follow & lookback)
* Follow := Read + (Follow & includes)
* Read   := DR + (Read & reads)

