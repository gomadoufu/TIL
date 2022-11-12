# CargoとCrates.io



## Cargoのプロファイル

Cargoには主な2つのプロファイルが存在する。
`dev`プロファイルは、`cargo build`コマンドを実行したときに使用され、 `release`プロファイルは、`cargo build --release`コマンドを実行したときに使用される。`dev`プロファイルは、開発中に役に立つデフォルト設定がなされていて、`release`プロファイルは、 リリース用の設定がされている。

Cargoは、Cargo.tomlに**[profile]**セクションが存在しない場合に、デフォルト設定を使用する。自分でセクションを追加することで、デフォルト設定の一部を上書きすることができる。例えば、コンパイルの最適化度合い(gccでいうoオプションみたいな)を制御する`opt-level`設定のデフォルト値は以下

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

`opt-level`に異なる値を以下することで、これを上書きすることができる

```toml
[profile.dev]
opt-level = 1
```



## Crates.ioにクレートを公開する

自分のパッケージを公開することで、他の人とコードを共有することができる。自分のパッケージのソースコードを配布することになるので、主にオープンソースのコードをホストする。

### ドキュメンテーションコメント

Rustにはドキュメンテーション用の**ドキュメンテーションコメント**がある。ドキュメンテーションコメントはHTMLドキュメントを生成する。

ドキュメンテーションコメントは、3連スラッシュ`///`を使用する。Markdown記法もサポートされている。

```rust
/// Adds one to the number given.
/// 与えられた数値に1を足す。
///
/// # Examples
///
/// ```
/// let five = 5;
///
/// assert_eq!(6, my_crate::add_one(5));
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

この状態で`cargo doc`を実行すれば、ドキュメントを生成することができる。`cargo doc`は`rustdoc`ツールを実行し、生成されたドキュメントを**target/doc**に格納する。

ドキュメンテーションコメントの別のスタイルとして、`//!`がある。これは、このコメントを含む要素(モジュールやクレート)の全体についての説明をするためのもの。

```rust
//! # My Crate
//!
//! `my_crate` is a collection of utilities to make performing certain
//! calculations more convenient.

//! #自分のクレート
//!
//! `my_crate`は、ユーティリティの集まりであり、特定の計算をより便利に行うことができます。

/// Adds one to the number given.
// --snip--
```

