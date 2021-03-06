<!-- ## Types and Type Declarations -->
## 型と型宣言

<!--
One of the design principles of Deno is no _magical_ resolution. When TypeScript
is type checking a file, it only cares about the types for the file, and the
`tsc` compiler has a lot of logic to try to resolve those types. By default, it
expects _ambiguous_ module specifiers with an extension, and will attempt to
look for the file under the `.ts` specifier, then `.d.ts`, and finally `.js`
(plus a whole other set of logic when the module resolution is set to `"node"`).
Deno deals with explicit specifiers.
-->
Deno の設計方針の一つは _魔法の_ 解決をしないことです。TypeScript がファイルの型チェックをするとき、気にするのはファイルの型だけで、`tsc` コンパイラはその型を解決するためにたくさんのロジックを持っています。デフォルトでは、拡張子を持つ _曖昧な_ モジュール指定子を期待しており、`ts` 指定子のファイルを探し、次に `.d.ts` 最後に `.js` のファイルを探します (モジュール解決が `"node"` にセットされた場合、他にも様々なロジックがあります)。Deno は明確な指定子を扱います。

<!--
This can cause a couple problems though. For example, let's say I want to
consume a TypeScript file that has already been transpiled to JavaScript along
with a type definition file. So I have `mod.js` and `mod.d.ts`. If I try to
import `mod.js` into Deno, it will only do what I ask it to do, and import
`mod.js`, but that means my code won't be as well type checked as if TypeScript
was considering the `mod.d.ts` file in place of the `mod.js` file.
-->
これはいくつかの問題を起こすかもしれません。例えば、すでに型定義ファイルとともに JavaScript にトランスパイルされた TypeScript ファイルを読み込みm帯をします。つまり、`mod.js` と `mod.d.ts` を持っています。もし、`mod.js` を Deno にインポートしようとすると、`mod.js` をインポートしてくれますが、TypeScriptが `mod.js` ファイルの代わりに `mod.d.ts` を考慮する場合のように型チェックしてくれません。

<!--
In order to support this in Deno, Deno has two solutions, of which there is a
variation of a solution to enhance support. The two main situations you come
across would be:
-->
Deno でこれをサポートするためには2つの解決法があり、サポートを強化するための解決法がいくつかあります。あなたが出くわす主な2つの状況は:

<!--
- As the importer of a JavaScript module, I know what types should be applied to
  the module.
- As the supplier of the JavaScript module, I know what types should be applied
  to the module.
-->
- JavaScript モジュールのインポートする側として、どの型をモジュールに適応するか知っている。
- JavaScript モジュールを提供する側として、どの型をモジュールに適応するか知っている。

<!--
The latter case is the better case, meaning you as the provider or host of the
module, everyone can consume it without having to figure out how to resolve the
types for the JavaScript module, but when consuming modules that you may not
have direct control over, the ability to do the former is also required.
-->
後者は良いケースです。つまり、モジュールの提供者やホストとして、JavaScript モジュールの型を解決する方法を探すことなくすべての人がそのモジュールを使うことができますが、直接制御できないモジュールを使うとき前者の能力も二つ用となります。

<!-- ### Providing types when importing -->
### 型の提供とインポート

<!--
If you are consuming a JavaScript module and you have either created types (a
`.d.ts` file) or have otherwise obtained the types, you want to use, you can
instruct Deno to use that file when type checking instead of the JavaScript file
using the `@deno-types` compiler hint. `@deno-types` needs to be a single line
double slash comment, where when used impacts the next import or re-export
statement.
-->
もし、JavaScript モジュールを使っていて、生成された型 (`.d.ts` ファイル) を持っているか使用したい型を取得できるなら、`@deno-types` コンパイラヒントをつかって、Deno に型チェックの際 JavaScript ファイルの代わりにそのファイルを使うよう指定できます。`@deno-types` は一行ダブルスラッシュコメントである必要があり、このコメントが使われた場合次の行の import 文か re-export 文に影響があります。

<!--
For example if I have a JavaScript modules `coolLib.js` and I had a separate
`coolLib.d.ts` file that I wanted to use, I would import it like this:
-->
例えば、`coolLib.js` という JavaScript モジュールを持っていて、別の `coolLib.d.ts` ファイルを持っている場合、次のようにインポートするします:

```ts
// @deno-types="./coolLib.d.ts"
import * as coolLib from "./coolLib.js";
```

<!--
When type checking `coolLib` and your usage of it in the file, the
`coolLib.d.ts` types will be used instead of looking at the JavaScript file.
-->
ファイル内で `coolLib` の型チェックとその使用法をチェックするときに、JavaScript ファイルを見る代わりに `coolLib.d.ts` 型を見ます。

<!--
The pattern matching for the compiler hint is somewhat forgiving and will accept
quoted and non-question values for the specifier as well as it accepts
whitespace before and after the equals sign.
-->
コンパイラヒントのパターンマッチはやや寛容で 指定子の値にはクオートされたものとクオートされてないものを受け入れ、等号の前と後の空白も受け入れます。

<!-- ### Providing types when hosting -->
### ホストしているときの型の提供

<!--
If you are in control of the source code of the module, or you are in control of
how the file is hosted on a web server, there are two ways to inform Deno of the
types for a given module, without requiring the importer to do anything special.
-->
モジュールのソースコードを管理している場合や、ファイルがどのように web サーバーでホストされているかを管理している場合には、インポートする側が何もしなくても特定のモジュールの型を Deno に知らせる方法が2つあります。

<!-- #### Using the triple-slash reference directive -->
#### トリプルスラッシュリファレンスディレクティブの使用

<!--
Deno supports using the triple-slash reference directive, which adopts the
reference comment used by TypeScript in TypeScript files to _include_ other
files and applies it to JavaScript files.
-->
Deno はトリプルスラッシュリファレンスディレクティブの使用をサポートします。これは、TypeScript ファイルで TypeScript が他のファイルを _include_ するために使用している参照コメントを採用し、JavaScript ファイルに適用するものです。

<!--
For example, if I had create `coolLib.js` and along side of it I had created my
type definitions for my library in `coolLib.d.ts` I could do the following in
the `coolLib.js` file:
-->
例えば、`coolLib.js` を作り、`coolLib.d.ts` に型定義を作った場合、`coolLib.js` ファイルの中で次のようにすることができます:

```js
/// <reference path="./coolLib.d.ts" />

// ... the rest of the JavaScript ...
```

When Deno encounters this directive, it would resolve the `./coolLib.d.ts` file
and use that instead of the JavaScript file when TypeScript was type checking
the file, but still load the JavaScript file when running the program.

<!-- #### Using X-TypeScript-Types header -->
#### X-TypeScript-Types ヘッダーの使用

<!--
Similar to the triple-slash directive, Deno supports a header for remote modules
that instructs Deno where to locate the types for a given module. For example, a
response for `https://example.com/coolLib.js` might look something like this:
-->
トリプルスラッシュディレクティブと同様に、Deno はモジュールの型がどこにあるかを教えるリモートモジュールのヘッダーをサポートします。例えば、`https://example.com/coolLib.js` のレスポンスがこのようだったとします:

```
HTTP/1.1 200 OK
Content-Type: application/javascript; charset=UTF-8
Content-Length: 648
X-TypeScript-Types: ./coolLib.d.ts
```

<!--
When seeing this header, Deno would attempt to retrieve
`https://example.com/coolLib.d.ts` and use that when type checking the original
module.
-->
このヘッダーが見えたら、Deno は `https://example.com/coolLib.d.ts` を取得しようとし、もとのモジュールの型チェックに使います。

<!-- ### Important points -->
### 重要な点

<!-- #### Type declaration semantics -->
#### 型宣言セマンティック

<!--
Type declaration files (`.d.ts` files) follow the same semantics as other files
in Deno. This means that declaration files are assumed to be module declarations
(_UMD declarations_) and not ambient/global declarations. It is unpredictable
how Deno will handle ambient/global declarations.
-->
型宣言ファイル (`.d.ts` ファイル) は Deno の他のファイルと同じセマンティックに従います。つまり、型宣言ファイルはモジュール宣言 (_UMD 宣言_) であり、ambient/global 宣言ではないと予測されます。Deno が ambient/global 宣言をどのように扱うかは予測できません。

<!--
In addition, if a type declaration imports something else, like another `.d.ts`
file, its resolution follow the normal import rules of Deno. For a lot of the
`.d.ts` files that are generated and available on the web, they may not be
compatible with Deno.
-->
さらに、型宣言が `.d.ts` ファイルのようになにか別のものをインポートした場合、その解決は Deno の通常のインポートルールに従います。web 上で生成され利用可能な `.d.ts` ファイルの多くは Deno と五感がないかもしれません。

<!--
To overcome this problem, some solution providers, like the
[Skypack CDN](https://www.skypack.dev/), will automatically bundle type
declarations just like they provide bundles of JavaScript as ESM.
-->
この問題を克服するために、[Skypack CDN](https://www.skypack.dev/) などのプロバイダーは JavaScript バンドルを ESM として提供しているように型宣言を自動的にバンドルしてくれます。

<!-- #### Deno Friendly CDNs -->
#### Deno フレンドリーな CDN

<!-- There are CDNs which host JavaScript modules that integrate well with Deno. -->
Deno と相性のいいJavaScript モジュールをホストしている CDN があります。

<!--
- [Skypack.dev](https://docs.skypack.dev/skypack-cdn/code/deno) is a CDN which
  provides type declarations (via the `X-TypeScript-Types` header) when you
  append `?dts` as a query string to your remote module import statements. For
  example:

  ```ts
  import React from "https://cdn.skypack.dev/react?dts";
  ```
-->
- [Skypack.dev](https://docs.skypack.dev/skypack-cdn/code/deno) はリモートモジュールのインポート文クエリ文字列に `?dts` を追加した際、(`X-TypeScript-Types` ヘッダーによる) 型宣言を提供します。例えば:

  ```ts
  import React from "https://cdn.skypack.dev/react?dts";
  ```

<!-- ### Behavior of JavaScript when type checking -->
### 型チェックの際の JavaScript の振る舞い

<!--
If you import JavaScript into TypeScript in Deno and there are no types, even if
you have `checkJs` set to `false` (the default for Deno), the TypeScript
compiler will still access the JavaScript module and attempt to do some static
analysis on it, to at least try to determine the shape of the exports of that
module to validate the import in the TypeScript file.
-->
Deno で型がない JavaScript を TypeScript にインポートする場合、`checkJs` を `false` (Deno のデフォルト) にしていても、TypeScript コンパイラは JavaScript モジュールにアクセスし静的解析しようとし、少なくとも TypeScript ファイルへのインポートを検証するためにエクスポートの形を決定しようとします。

<!--
This is usually never a problem when trying to import a "regular" ES module, but
in some cases if the module has special packaging, or is a global _UMD_ module,
TypeScript's analysis of the module can fail and cause misleading errors. The
best thing to do in this situation is provide some form of types using one of
the methods mention above.
-->
これは通常 "普通の" ES モジュールをインポートしようとしているときは問題にはなりませんが、モジュールが特別なパッケージを持っていたり、グローバル _UMD_ モジュールの場合 TypeScript のモジュール解析は失敗し誤解を招くエラーが発生することがあります。このような問題の解決策で最善のものは、上記で示したような手法の一つを使い型を提供することです。

<!-- #### Internals -->
#### 内部

<!--
While it isn't required to understand how Deno works internally to be able to
leverage TypeScript with Deno well, it can help to understand how it works.
-->
Deno で TypeScript をうまく活用するためには、Deno の内部での動作を理解する必要はありませんが、Deno がどのように動作するのかを理解するのに役立ちます。

<!--
Before any code is executed or compiled, Deno generates a module graph by
parsing the root module, and then detecting all of its dependencies, and then
retrieving and parsing those modules, recursively, until all the dependencies
are retrieved.
-->
なにかのコードが実行やコンパイルされる前、Deno はルートモジュールをパースすることでモジュールグラフを作り、すべての依存関係を検出し、それらのモジュールを取得しパースします。

<!--
For each dependency, there are two potential "slots" that are used. There is the
code slot and the type slot. As the module graph is filled out, if the module is
something that is or can be emitted to JavaScript, it fills the code slot, and
type only dependencies, like `.d.ts` files fill the type slot.
-->
それぞれの依存関係に対して、使用される可能性のある "スロット" が2つあります。これはコードスロットと型スロットです。モジュールグラフが書かれていくにつれ、モジュールが JavaScript を出力できるのであれば、コードスロットを埋め、`.d.ts` ファイルなどの型依存関係のみであれば型スロットを埋めます。

<!--
When the module graph is built, and there is a need to type check the graph,
Deno starts up the TypeScript compiler and feeds it the names of the modules
that need to be potentially emitted as JavaScript. During that process, the
TypeScript compiler will request additional modules, and Deno will look at the
slots for the dependency, offering it the type slot if it is filled before
offering it the code slot.
-->
モジュールグラフが作られグラフの型チェックが必要なとき、Deno は TypeScript コンパイラを起動し JavaScript として出力される可能性があるモジュールの名前を TypeScript コンパイラに与えます。このプロセス中、TypeScript コンパイラは追加のモジュールを要求し、Deno は依存関係のためのスロットをみて、それが満たせれていればコードスロットを提供する前に型スロットを提供します。

<!--
This means when you import a `.d.ts` module, or you use one of the solutions
above to provide alternative type modules for JavaScript code, that is what is
provided to TypeScript instead when resolving the module.
-->
これは、`.d.ts` モジュールをインポートしたり、上記の解決法を使って JavaScript コードの代替型モジュールを提供したりした場合に、モジュールを解決する際に代わりに TypeScript に提供されるものを意味します。
