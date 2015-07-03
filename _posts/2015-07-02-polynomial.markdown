---
title:  "Polynomial class"
date:   2015-07-02 15:00:00
description: comments for qe-seminar
author:
    name: Kenji Sato
    github: kenjisato
---

### Excercise 2: Polynomialクラスと多項式の微分

QE: [Excercise 2][qeex2] は多項式のクラス `Polynomial` を作る問題.
次のような振る舞いをするように求められています.

{% highlight python %}
>>> p = Polynomial([1.0, 2.0, -3.0])
>>> p(1.0)
0.0
>>> p(2.0)
-1.0
{% endhighlight %}

多項式

\\[
  p(x) = a\_0 + a\_1 x + a\_2 x\^2 + \cdots + a\_N x\^N
\\]

について,

- 係数 \\( coefficients = [a\_0, \dots, a\_N] \\) を初期化とするクラスを作り,
- インスタンスが関数として振る舞うようにせよ,
- そのクラスには微分を計算するメソッドを実装せよ

というのが課題です.

[こちら][qesol2] に掲載されている解答を再掲しておきます (コメントは省略しました):

{% highlight python %}
class Polynomial(object):

    def __init__(self, coefficients):
        self.coefficients = coefficients

    def __call__(self, x):
        y = 0
        for i, a in enumerate(self.coefficients):
            y += a * x**i
        return y

    def differentiate(self):
        new_coefficients = []
        for i, a in enumerate(self.coefficients):
            new_coefficients.append(i * a)
        del new_coefficients[0]
        self.coefficients = new_coefficients
{% endhighlight %}

ここでの微分の定義は, もともとの多項式インスタンスを壊してしまうので,
ときによっては使いづらいと感じることもあるでしょう. (言語によっては「破壊的メソッド」と
言って特段の注意を払うのですが, Python ではそのようなことはありません)

以下で, 元のインスタンスを「破壊」しないメソッド定義で置き換えておきましょう.
その前に1点, 足し算について注意しておきます.

#### 浮動小数点数の足し算に関する注意

Quantative Economics の最初の章は Python の入門的なトピックを総ざらいすることが
目的なので, いくつか大切なトピックが省略されています. ここでは, 足し算に関する注意点を
挙げておきます.


上記の `__call__()` メソッドの内容は, 次のようなものです.

{% highlight python %}
y = 0
for i, a in enumerate(coefficients):
    y += a * x**i
{% endhighlight %}

ひょっとするとこれは特段面白みのない当たり前に正しいコードだと思うかもしれませんが,
落とし穴がないわけではありません.

次の振る舞いを手元のインタプリタで確認してみてください.

{% highlight python %}
>>> total = 0.0
>>> for _ in range(10):
...     total += 0.1
...
>>> total
0.9999999999999999
{% endhighlight %}

`sum()` 組み込み関数を使っても同様です.

{% highlight python %}
>>> sum(0.1 for _ in range(10))
0.9999999999999999
{% endhighlight %}

浮動小数点数 (float) が正確に表現できる数は

\\[
    n / 2^m, \quad n, m \in \mathbb{N}
\\]

の形の数だけです. それ以外の数はこの数表現で近似されます.


標準ライブラリの `math` モジュールに含まれる `math.fsum` はこの問題をうまく回避してくれます.

{% highlight python %}
>>> import math
>>> math.fsum(0.1 for _ in range(10))
1.0
{% endhighlight %}

浮動小数点数の足し算をするときは `math.fsum` を使うことを検討してみてください.
複素数の足し算ができない `math.fsum` を使えないケースも多々あると思いますので,
状況に応じて適切な関数を使用してください.

### (異なる条件のもとでの) 別解

次の2つの指示を無視します.

- `import` 文を使うなという指示
- 自分自身の`coefficients` を書き換えて微分を定義せよという指示

{% highlight python %}
from math import fsum

class Poly(object):

    def __init__(self, coef):
        self.coef = coef

    def __call__(self, x):
        return fsum(a * x**i for i, a
                    in enumerate(self.coef))

    def differentiate(self):
        new_coef = [i * a for i, a
                    in enumerate(self.coef[1:], 1)]
        return Poly(new_coef)
{% endhighlight %}

使用例は次の通りです.

{% highlight python %}
>>> p = Poly([1.0, 2.0, -3.0])
>>> p.coefficients
[1.0, 2.0, -3.0]
>>> p(0)
1.0
>>> q = p.differentiate()
>>> q.coefficients
[2.0, -6.0]
>>> q(0)
2.0
{% endhighlight %}




[qeex]:  http://quant-econ.net/py/python_oop.html#exercises
[qeex2]: http://quant-econ.net/py/python_oop.html#exercise-2
[qesol2]: http://nbviewer.ipython.org/github/QuantEcon/QuantEcon.py/blob/master/solutions/oop_solutions.ipynb#Exercise-2
