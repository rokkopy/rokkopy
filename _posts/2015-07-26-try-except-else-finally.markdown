---
title:  "try-except-else-finally"
date:   2015-07-26 18:00:00
description: comments for qe-seminar
author: kenjisato
---

Python は [EAFP (Easier to ask for forgiveness than permission)](https://docs.python.org/3/glossary.html#term-eafp) と呼ばれるコーディングスタイルが
推奨されています. 例えば, 次のようなコードを書きたいとしましょう.

```python
>>> scores = {'Alice': [80, 100], 'Bob': [90, 60]}
>>> def math_score(name):
...     if name in scores:
...         return scores[name][0]
...     else:
...         print('No data for {}'.format(name))
...         return None

```

もちろんこれはこれで問題ないのですが, `math_score`
関数は次のように書くこともできます.

```python
>>> def english_score(name):
...     try:
...         return scores[name][1]
...     except KeyError:
...         print('No data for {}'.format(name))
...         return None
```

それぞれ, 特定の動作がエラーにならないことを先に確認するコーディングスタイル
(LBYL: Look Before You Leap) と,
エラーが起きたら対処するというコーディングスタイル (EAFP) です.

どちらが望ましいという訳ではないと思いますが,
EAFP スタイルに慣れておきましょう.

極端なことを言えば, Python では例外処理は条件分岐に使われる構文の一種と言えます.
次のコードは, `x` が 0 でないことをチェックする代わりに,
ゼロ割エラーの発生を検知して条件分岐させます.

```python
def f(x):
    try:
        y = 1.0 / x
    except ZeroDivisionError:
        print('an exception raised')
    else:
        print('no exception')
    finally:
        print('always printed')
```

次の `str_to_num()` 関数は, 文字列を受け取り数値を返します.
整数への変換が失敗した場合に発生する `ValueError` は関数内部で捕捉され,
浮動小数点数への変換を試みます. さらに, これが失敗した場合は呼び出し元に
`ValueError` 例外が渡されます.

```python
def str_to_num(s):
    try:
        return int(s)
    except ValueError:
        return float(s)
```

`input_number()` 関数を実行するとインタプリタは入力待ちになります.
Ctrl+C (強制終了コマンド) を規定の終了動作にするために
`KeyboardInterrupt` 例外を捕捉しています. (1つ目の try-except)

数字を入力すると, 文字列を数値型に変換しようとします. 変換が失敗すると
`ValueError` が発生するので, これを補足して入力に誤りがある旨を表示します.

```python
def input_number(msg='Enter a number (^C to cancel)'):
    while True:
        try:
            s = input(msg + ': ')
        except KeyboardInterrupt:
            print('\nBye!')
            break

        try:
            num = str_to_num(s)
        except ValueError:
            print('Not a number')
            continue
        else:
            print('Your number is {}'.format(s))
            print('Bye!')
            break
```

実行結果:

```python
>>> input_number()
Enter a number (^C to cancel): 1
Your number is 1
Bye!
>>> input_number()
Enter a number (^C to cancel): -1
Your number is -1
Bye!
>>> input_number()
Enter a number (^C to cancel): 1.23
Your number is 1.23
Bye!
>>> input_number()
Enter a number (^C to cancel): 1e-10
Your number is 1e-10
Bye!
>>> input_number()
Enter a number (^C to cancel): 1.2.3
Not a number
Enter a number (^C to cancel): abc
Not a number
Enter a number (^C to cancel): ^C
Bye!
```

最後のコードは, Python スクリプト実行時のオプション有無で振る舞いを変わります.

実行時のオプションは `sys.argv` によって取得できます.
`sys.argv` は長さが１以上のリストで, オプションなしで実行された場合には,
実行されたスクリプトのファイル名のみを格納した長さが１のリストになります.
オプションが渡された場合には `sys.argv[1]`, `sys.arg[2]` .... でオプションを文字列として取得できます.

`len(sys.argv) > 1` の真偽でオプション有無を判定してもよいのですが,
次のようにしても同じことです.

```python
# hello.py
import sys

try:
    name = sys.argv[1]
except IndexError:
    print('Hello, world!')
else:
    print('Hello, {}'.format(name))
finally:
    print('Bye')
```

シェルで実行してみます.

```bash
$ python hello.py
Hello, world!
Bye
$ python hello.py Kenji
Hello, Kenji
Bye











