# Finding Counterexamples from Parsing Conflicts

https://www.cs.cornell.edu/andru/papers/cupex/cupex.pdf

## 1. Introduction

* unifying counterexample
  これは1つのsymbol列で、2つの異なるparseが可能なもの

* nonunifying counterexample
  これは2つsymbol列で、コンフリクトする地点まで共通のprefixをもつもの

> A key insight behind this efficiency is to expand the search frontier from the conflict state instead of the start state.


## 2. Background

## 3. Counterexamples

### 3.1

### 3.2 Properties of Good Counterexamples

コンフリクトと直接関係のない箇所では非終端記号を使って表示したほうがわかりやすい.

unifying counterexampleにおいてはsentential form全体を表示するよりも、最も内側の(曖昧さを引き起こしている箇所に最も近い)非終端記号を取り上げるのがよい.

```
expr + expr • + expr ? stmt stmt
```

例えばこのようにstmtの3つめのRule全体を表示するよりも、以下のようにexprに絞って表示したほうが注目すべきポイントがわかりやすい.

```
expr + expr • + expr
```

nonunifying counterexample

## 4. Constructing Nonunifying Counterexamples

stateの遷移ではなくlookahead-sensitive graphでの遷移を考え、そこでの最短パス(the shortest lookahead-sensitive path)の検索を行う.

lookahead-sensitive graph ではstateにおける非カーネル項の生成を明示的にあつかう.
頂点は (s, itm, L) の組で、

* s: state number
* itm: s内のitem
* L: precise lookahead set (preciseとは?)

辺は以下のようにして構成する

* transition
  2.1に書いてあるようにここでのtransitionはshiftとgotoの両方を含んだもの.
  transitionから生成される辺においては L が維持される.

* production step
  • の次が非終端記号のとき同じstate sで異なるitmの頂点への辺を生成する.
  precise lookahead setは followL(itm) により決定する.


最短パス(the shortest lookahead-sensitive path)の検索は

## 5. Constructing Unifying Counterexamples
