# 文字列

## 仕様について

D言語の文字列は文字の配列である。文字列リテラルはimmutableである。

```d
char[] str1 = "abc";                // error: 文字列リテラルは暗黙にmutableなchar[]へと変換できない。
char[] str2 = "abc".dup;            // ok: dupを使えばmutableなコピーが可能。
immutable(char)[] str3 = "abc";     // ok
immutable(char)[] str4 = str2;      // error: mutableなchar[]からimmutableへは暗黙変換できない。
immutable(char)[] str5 = str3;      // ok: immutableなchar[]は代入できる。
immutable(char)[] str6 = str2.idup; // ok: idupを使えばimmutable なコピーができる。
```

実際に、string型は `immutable(char)[]` となっている。上の処理と同等の処理は以下のようにも書ける。

```d
char[] str1 = "abc";     // error
char[] str2 = "abc".dup; // ok
string str3 = "abc";     // ok
string str4 = str2;      // error
string str5 = str3;      // ok
string str6 = str2.idup; // ok
```

`immutable(char)[]` 型は `immutable char` の配列を表現するが、文字列それ自体への参照はmutableとなる。

```d
immutable char[] s = "foo"; // もしくは immutable(char[])
s[0] = 'a';  // error: immutableなデータへの参照を書き換えようとしている
s = "bar";   // error: sはimmutable

immutable(char)[] s = "hello";  // string
s[0] = 'b';  // error: s[]はimmutable
s = null;    // ok: s自体はmutable
```

また、`char[]`はUTF-8な文字列を表し、`wchar[]` および `dchar[]` はそれぞれUTF-16/UTF-32文字列となる。

文字列はコピー・比較・結合・(末尾への)追加が可能である。

```d
str1 = str2;
if (str1 < str3) { ... }
func(str3 ~ str4);
str4 ~= str1;
```

一時変数はGC(もしくは `alloca()`)によって解放される。
これは文字列に限らず配列一般に適用される。

文字列リテラルからcharへのポインタを生成することができる。

```d
char* p = &str[3];  // 4番目の要素へのポインタ
char* p = str;      // 先頭の要素へのポインタ
```

D言語の文字列はC言語のようにnull終端になっていない。C言語の関数に文字列のポインタを渡す際に、末尾をnull終端にする必要がある。

```d
str ~= "\0";
```

あるいは標準ライブラリの `std.string.toStringz` 関数を使うこともできる。

### C言語のprintfと文字列

前述したように、C言語の関数にD言語の文字列を渡す場合はnull終端を考慮する必要がある。
しかし文字列リテラルにかんしては、すでにnull終端になっているためそのまま渡すことができる。

```d
import core.stdc.stdio;
printf("the string literal is '%s'\n", "string literal".ptr);
```

ここで最初の引数で `.ptr` を要求していないのはなぜだろうか？
最初の引数は `const(char)*` というプロトタイプ宣言がなされており、文字列リテラルは `const(char)*` へと暗黙にキャストされる。
しかしprintfの残りの引数は可変長であり、 `immutable(char)*` は可変長引数に渡すことができない。

また別の方法としてprecision specifierとして文字列長を指定するという方法もある。

```d
printf("the string is '%.*s'\n", cast(int)str.length, str.ptr);
```

最良の方法は `std.stdio.writefln` を使うことである。

```d
import std.stdio;
writefln("the string is '%s'", str);
```

## 参考資料

- D言語仕様 文字列: https://dlang.org/spec/arrays.html#string