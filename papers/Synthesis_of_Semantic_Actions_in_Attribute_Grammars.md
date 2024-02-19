# Synthesis of Semantic Actions in Attribute Grammars

https://arxiv.org/pdf/2208.06916.pdf

## Abstract



## 1. INTRODUCTION

いわれてみれば semantic information は非終端記号にのみ付与されるな。

> Almost no modern applications use hand-written parsers anymore

どうもこの界隈の人はすぐにこういうことを言う...

合成(sythesized)属性と継承(inherited)属性。

合成属性や継承属性を一回で決めるのは難しく試行錯誤することになるが、それが手間である。

sketchesから semantic actions を自動生成する。

Fig. 1で黄色になっている箇所が holes であり、ここを推測する方法を紹介する。

loop-free programであるという洞察。

