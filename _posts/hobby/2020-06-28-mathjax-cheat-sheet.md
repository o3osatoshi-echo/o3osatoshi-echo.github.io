---
layout:        post
title:         "MathJaxチートシート"
date:          2020-06-28 00:00:00 +0900
modified_date: 2020-06-28 00:00:00 +0900
categories:    [hobby]
tags:          [cheating]
---

![thumbnail](/assets/2020-06-28-mathjax-cheat-sheet/thumbnail.png)

## MathJax記法
#### 概要
本ブログ執筆用MathJax記法チートシート
#### 参照
主に下記サイトを参照
- [MathJax 公式サイト](https://www.mathjax.org)
- [LaTeXコマンド集](http://www.latex-cmd.com)
- [MathJaxの使い方](http://www.eng.niigata-u.ac.jp/~nomoto/download/mathjax.pdf)

### WordPressにおけるMathJaxの利用
WordPressのプラグイン「Simple Mathjax」を使用

### デリミター
ディスプレイ数式であれば`{空行}$$...$${空行}`、または`\\[...\\]`  
インライン数式であれば`$$...$$`、または`\\(...\\)`
##### MathJax
```

$$y = α + βx$$

\\[y = α + βx\\]
in-line $$y = α + βx$$ in-line  
in-line \\(y = α + βx \\) in-line
```
##### 表示

$$y = α + βx$$

\\[y = α + βx\\]
in-line $$y = α + βx$$ in-line  
in-line \\(y = α + βx \\) in-line

### 分数
`\frac`コマンド
##### MathJax
```
\\[y = \frac{α}{x}\\]
```
##### 表示
\\[y = \frac{α}{x}\\]

### 三角関数
`\sin`、`\cos`、`\tan`コマンド
##### MathJax
```
\\[y = \sin x\\]
\\[y = \cos x\\]
\\[y = \tan x\\]
```
##### 表示
\\[y = \sin x\\]
\\[y = \cos x\\]
\\[y = \tan x\\]

### 指数
べき乗（`^`）、指数関数（`\exp`）、平方根（`\sqrt`）、n条根（`\sqrt[]{}`）
##### MathJax
```
\\[y = x^2 \pi\\]
\\[y = x^{2 \pi}\\]
\\[y = \exp (x)\\]
\\[y = \mathrm{e}^x\\]
\\[y = \sqrt{x^2 + y^2}\\]
\\[y = \sqrt[n]{x}\\]
```
##### 表示
\\[y = x^2 \pi\\]
\\[y = x^{2 \pi}\\]
\\[y = \exp (x)\\]
\\[y = \mathrm{e}^x\\]
\\[y = \sqrt{x^2 + y^2}\\]
\\[y = \sqrt[n]{x}\\]

### 対数
`\log`コマンド、または`\ln`コマンド
##### MathJax
```
\\[y = \log{2} x\\]
\\[y = \ln{x}\\]
```
##### 表示
\\[y = \log{2} x\\]
\\[y = \ln{x}\\]

### 総和・総乗
総和（`\sum`）、総乗（`\prod`）
##### MathJax
```
\\[y = \sum\_{i=0}^{n} x\_i\\]
\\[y = \prod\_{i=0}^{n} x\_i\\]
```
##### 表示
\\[y = \sum\_{i=0}^{n} x\_i\\]
\\[y = \prod\_{i=0}^{n} x\_i\\]

### 極限
`\lim`コマンド
##### MathJax
```
\\[y = \lim_{x \to \infty} f(x)\\]
```
##### 表示
\\[y = \lim_{x \to \infty} f(x)\\]

### 微分・偏微分
##### MathJax
```
\\[y = f'(x)\\]
\\[y = \frac{dy}{dx}\\]
\\[y = \frac{\partial y}{\partial x}\\]
```
##### 表示
\\[y = f'(x)\\]
\\[y = \frac{dy}{dx}\\]
\\[y = \frac{\partial y}{\partial x}\\]

### 積分・重積分
`\int`コマンド
##### MathJax
```
\\[y = \int_a^b f(x) dx\\]
\\[y = \iint_D f(x,y) dxdy\\]
```
##### 表示
\\[y = \int_a^b f(x) dx\\]
\\[y = \iint_D f(x,y) dxdy\\]

### 複数行
`\begin{align}`と`\end{align}`で囲む  
`\\\\`により改行  
`&`により表示位置揃え
##### MathJax
```
$$
\begin{align}
    \cos 2\theta &= \cos^{2} \theta - \sin^{2} \theta \\\\
        &= 2\cos^{2} \theta - 1 \\\\
        &= 1 - 2\sin^{2} \theta
\end{align}
$$
```
##### 表示

$$
\begin{align}
    \cos 2\theta &= \cos^{2} \theta - \sin^{2} \theta \\\\
        &= 2\cos^{2} \theta - 1 \\\\
        &= 1 - 2\sin^{2} \theta
\end{align}
$$

### 等号

|MathJax| 表示                 |読み方・意味|
|:--|:-------------------|:--|
|`=`| $$=$$              |等号、イコール|
|`\neq`| $$\neq$$           |不等、ノットイコール|
|`\sim`| $$\sim$$           |ニアリイコール|
|`\simeq`| $$\simeq$$         |ニアリイコール|
|`\approx`| $$\approx$$        |ニアリイコール|
|`\fallingdotseq`| $$\fallingdotseq$$ |ニアリイコール|
|`\risingdotseq`| $$\risingdotseq$$  |ニアリイコール|
|`\equiv`| $$\equiv$$         |合同|

### 演算子

|MathJax| 表示         |読み方・意味|
|:--|:-----------|:--|
|`+`| $$+$$      |和、足す|
|`-`| $$-$$      |差、引く|
|`\times`| $$\times$$ |積、掛ける|
|`\div`| $$\div$$   |商、割る|
|`\pm`| $$\pm$$    |プラスマイナス|
|`\mp`| $$\mp$$    |マイナスプラス|

### 記号

|MathJax| 表示                      |読み方・意味|
|:--|:------------------------|:--|
|`\infty`| $$\infty$$              |無限大|
|`\to`| $$\to$$                 |右矢印|
|`\rightarrow`| $$\rightarrow$$         |右矢印|
|`\longrightarrow`| $$\longrightarrow$$     |長い右矢印|
|`\leftrightarrow`| $$\leftrightarrow$$     |右向きと左向き矢印|
|`\longleftrightarrow`| $$\longleftrightarrow$$ |長めの右向きと左向き矢印|
|`\rightleftarrows`| $$\rightleftarrows$$    |右向きと左向き矢印|

### 斜体キャンセル
`{\rm`と`}`で囲む
##### MathJax
```
\\[aa {\rm aa} aa\\]
```
##### 表示
\\[aa {\rm aa} aa\\]
