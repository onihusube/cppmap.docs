description: C++23 以降に向けて議論が進行中のおもな提案の紹介

# C++23 以降に向けた提案
議論が進行中のおもな提案です。標準化会議によって規格化が妥当でないと判断された場合は採択されません。

## 標準ライブラリ モジュール
- [Standard Library Modules (P0581)](https://wg21.link/p0581) 

`<iostream>` や `<vector>` など、すべての C++ 標準ライブラリヘッダをモジュールとして提供する提案です。言語機能としてのモジュールは C++20 に導入されましたが、標準ライブラリへの導入は議論が間に合わず、C++23 に先送りされました。標準ライブラリのモジュールへの移行に合わせて、[標準ライブラリを再編成する提案](https://wg21.link/p1453)も出されています。


## `operator[]` に複数の引数
- [Subscripts On Parade (P1277)](https://wg21.link/p1277)

`m[i, j, k]` のように、`operator[]` の引数に複数の値を渡せるようにする提案です。多次元配列の要素に簡潔な式でアクセスできるようになります。この書き方は、C++17 規格ではコンマ演算子と解釈されて `m[k]` になってしまうため、まずは[現行の文法を deprecated にする提案](https://wg21.link/p1161)が提出されています。`std::span` の多次元版である [`std::mdspan` の提案](https://github.com/kokkos/array_ref)での活用も期待されます。


## 低レベル オーディオ API
- [A Standard Audio API for C++:
Motivation, Scope, and Basic Design (P1386)](https://wg21.link/p1386)

音声再生・録音を行う低レベル API `std::audio` を追加する提案です。以前頓挫した 2D Graphics API の標準化に比べれば、枯れている技術なので実現可能性は低くはなさそうです。波形データをオーディオデバイスとやり取りすることが主要な役割で、音声コーデックやフィルタなどの機能は提供しません。[実験的な実装を公開](https://github.com/stdcpp-audio/libstdaudio)して、設計に関するフィードバックを集めている段階です。


## ネットワーク
- [Working Draft, C++ Extensions for Networking (N4734)](https://wg21.link/n4734)

ネットワーク機能を標準ライブラリに導入する提案です。[Boost.Asio](https://www.boost.org/doc/libs/release/doc/html/boost_asio.html) をベースに、ソケット通信、名前解決、インターネットプロトコル、タイマー、バッファ、非同期処理のための各種機能を実装する予定です。


## リフレクション
- [Working Draft, C++ Extensions for Reﬂection (N4818)](https://wg21.link/n4818)

C++ にリフレクションを導入する提案です。C# のような動的なリフレクションではなく、コンパイル時に情報を取得する静的なリフレクションです。

```C++
#include <experimental/reflect>

using namespace std::experimental::reflect;

constexpr void f(int arg1, int arg2);

using f_mobj = reflexpr(f(0, 1));

auto name = get_name_v<get_element_t<0, get_parameters_t<f_mobj>>>; // "arg1"

auto is_constexpr_function = is_constexpr<f_mobj>::value; // true
```

この機能を前提とした[メタクラスの提案](https://wg21.link/p0707)も議論が進んでいます。


## パターンマッチング
- [Pattern Matching (P1371)](https://wg21.link/p1371)

`std::tuple` や `std::variant` で C++ における代数的データ型の表現が広がりましたが、それらを扱うための `std::apply` や `std::visit` は複雑で多くの制約があります。C++ 以外のプログラミング言語にならい、パターンマッチングを導入してこの問題を解決する提案です。この提案では、より宣言的で構造化された `switch` 文として、`inspect` 文/式という新しい文法の導入を提案しています。

```C++
// inspect 文の文法
inspect constexpr_opt (init-statement_opt condition)
{
    pattern guard_opt : statement
    pattern guard_opt : statement
    ...
}

// inspect 式の文法
inspect constexpr_opt (init-statement_opt condition) trailing-return-type_opt
{
    pattern guard_opt => expression ,
    pattern guard_opt => expression ,
    ...
}
```

### 整数のマッチング
```C++ tab="現在"
switch (code)
{
	case 200: std::cout << "OK\n"; break;
	case 404: std::cout << "Not Found\n"; break;
	default: std::cout << "don't care\n";
}
```

```C++ tab="提案"
inspect (code)
{
    200: std::cout << "OK\n";
	404: std::cout << "Not Found\n";
	__: std::cout << "don't care\n"; // __ はワイルドカードパターン
}
```

### 文字列のマッチング
```C++ tab="現在"
if (s == "png")
{
    std::cout << "PNG Image\n";
}
else if (s == "jpg")
{
    std::cout << "JPEG Image\n";
}
else
{
    std::cout << "Not supported\n";
}
```

```C++ tab="提案"
inspect (s)
{
    "png": std::cout << "PNG Image\n";
    "jpg": std::cout << "JPEG Image\n";
    __: std::cout << "Not supported\n";
}
```

### Tuples のマッチング
```C++ tab="現在"
auto&& [x, y] = pos;
if (x == 0 && y == 0)
{
    std::cout << "on the origin\n";
}
else if (y == 0)
{
    std::cout << "on X-axis\n";
}
else if (x == 0)
{
    std::cout << "on Y-axis\n";
}
else
{
    std::cout << x << ", " << y << '\n';
}
```

```C++ tab="提案"
inspect (pos)
{
	[0, 0]: std::cout << "on the origin\n";
	[x, 0]: std::cout << "on the X-axis\n";
	[0, y]: std::cout << "on the Y-axis\n";
	[x, y]: std::cout << x << ", " << y << '\n';
}
```

### Variants のマッチング
```C++ tab="現在"
struct Visitor
{
	void operator()(int i) const
	{
		std::cout << "int: " << i << '\n';
	}

	void operator()(float f) const
	{
		std::cout << "float: " << f << '\n';
	}
};

int main()
{
	std::variant<int, float> v = 3.14f;

	std::visit(Visitor{}, v);
}
```

```C++ tab="提案"
int main()
{
	std::variant<int, float> v = 3.14f;

	inspect (v)
	{
		<int> i: std::cout << "int: " << i << '\n';
		<float> f: std::cout << "float: " << f << '\n';
	}
}
```

### Polymorphic Types のマッチング
```C++ tab="現在"
struct Shape
{
	virtual ~Shape() = default;
	virtual double area() const = 0;
};

struct Circle : Shape
{
	double radius;
	double area() const override
	{
		return std::numbers::pi * radius * radius;
	}
};

struct Rectangle : Shape
{
	double width, height;
	double area() const override
	{
		return width * height;
	}
};
```

```C++ tab="提案"
struct Shape
{
	virtual ~Shape() = default;
};

struct Circle : Shape
{
	double radius;
};

struct Rectangle : Shape
{
	double width, height;
};

double Area(const Shape& shape)
{
	return inspect (shape)
	{
		<Circle>	[r]		=> std::numbers::pi * r * r, // inspect 式では => を使う
		<Rectangle>	[w, h]	=> w * h
	}
}
```


## 拡張浮動小数点数型
- [Extended floating-point types (P1467)](https://wg21.link/P1467)
- [Fixed-layout floating-point type aliases (P1468)](https://wg21.link/P1468)

`float`, `double`, `long double` 以外の浮動小数点数型の実装を可能にする提案です。コンピュータグラフィックスや機械学習分野で使用頻度が増えている半精度 (16-bit) 浮動小数点数型を、組み込み型として提供できるようにすることが主な目的です。


## Unicode 名による文字エスケープ
- [Named universal character escapes (P2071)](https://wg21.link/P2071)

Python などで採用されている、Unicode 名による文字エスケープを導入する提案です。不可視文字や似ている字形を文字 / 文字列リテラルで記述する際に役に立ちます。

```C++
std::string a ="\N{ZERO WIDTH SPACE}"; // "​" (不可視の空白)
std::string b = "\N{GREEK CAPITAL LETTER OMEGA}"; // "Ω"
std::string c = "\N{OHM SIGN}"; // "Ω"
```


## `std::string::operator=(char)` の問題の解消
- [String's gratuitous assignment (P2037)](https://wg21.link/P2037)

`std::string::operator=(char)` によって次のようなコードがコンパイルできてしまう問題の解決を図る提案です。この関数自体を `deprecated` にすることも解決法の候補として挙げられています。

```C++
std::string s;
s = 48.0;
std::cout << s << '\n';
```


## Executors


## コルーチンのライブラリサポート


## 契約プログラミング

