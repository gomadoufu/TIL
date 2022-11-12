# DerefトレイトとDropトレイト

`Deref`トレイトと`Drop`トレイトは、スマートポインタが実装するトレイトである。

## Derefトレイトでスマートポインタを普通の参照のように扱う

`Deref`トレイトを実装することで、**参照外し演算子**`*`（掛け算やグロブ演算子とは違います）の振る舞いをカスタマイズできる。 `Deref`を実装してスマートポインタを普通の参照のように扱えるようにすれば、 参照に対して処理を行うコードを書いて、そのコードをスマートポインタに対しても使うことができるようになる。

### 参照外し演算子で値までポインタを追いかける

以下のように参照外し演算子を使えば、元のデータまで参照を辿ることができる。

```rust
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```



### Box\<T\>を参照のように使う

上記のコードを、`Deref`トレイトの実装されているスマートポインタである`Box<T>`を使って以下のように書き直すことができる。

```rust
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```



### Derefトレイトを実装して、型を参照のように扱う

`Box<T>`がポインタのように参照外し演算子を適用できるのは、ボックスに`Deref`トレイトが実装されているためである。

ここで、試しに独自のスマートポインタを定義してみる。

```rust
struct MyBox<T>(T); //1要素だけのタプル構造体

//MyBoxはジェネリクス
impl<T> MyBox<T>{
  fn new(x:T) -> MyBox<T> {
    MyBox(x)
  }
}
```

これを、標準ライブラリの`Box<T>`の代わりに使ってみると、コンパイルエラーになる

```rust
fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
  //error[E0614]: type `MyBox<{integer}>` cannot be dereferenced
  //(エラー: 型`MyBox<{integer}>`は参照外しできません)
}
```

`MyBox<T>`を参照外しするためには、`Deref`トレイトを実装する必要がある。
`Deref`トレイトは、参照外しをした際にどのような値を返すかを決定する`deref`メソッドの、具体的な実装を要求する。

```rust
//Derefトレイトを実装する
impl<T> Deref for MyBox<T> {
    type Target = T;

  //参照外しされた際には、自身のデータの0番目の参照を返す
    fn deref(&self) -> &T {
        &self.0
    }
}
```



### 関数やメソッドにおける暗黙的な参照外し型強制

`deref`メソッドの定義された型が参照外し後の状態を期待して呼び出された時、Rustコンパイラは自動的に参照外しをして型を変更する。

例えば、次のようなhello関数がある。hello関数は、引数に文字列スライスを要求する。

```rust
fn hello(name: &str) {
    println!("Hello, {}!", name);
}
```

次に、hello関数に`MyBox<T>`への参照を与えて呼び出してみる。すると、正常に動作する。

```rust
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m); //Rust
}
```

`MyBox<T>`には`Deref`トレイトが実装されているので、コンパイラは`MyBox<T>`の`deref`を呼び出すことで、`&MyBox<String>`を`&String`に暗黙に変換できる。
さらに、`String`には文字列スライス`&str`を返す`deref`の実装があるため(APIドキュメント参照)、コンパイラは続けて`deref`を呼び出して、`&String`を`&str`に変換することができる。

結果的に、`&str`の引数に対して`&MyBox<T>`を与えても、うまく動作した。



## Dropトレイトで片付け時にコードを走らせる

`Drop`トレイトは、値がスコープを抜けそうになった時に起こることをカスタマイズできるトレイトである。どんな型に対しても`Drop`トレイトの実装を提供することができ、特にファイルの使用やネットワーク接続などのリソースを解放するのに活用できる。
`Drop`トレイトの機能は、ほとんどがスマートポインタを実装する時に使われる。

```rust
//独自のスマートポインタにDropトレイトを実装する
impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("ここでデータがドロップ `{}`!", self.data);
    }
}
```



### std::mem::dropで早期に値をドロップする

通常、`drop`メソッドを手動で呼び出すことは許されていない。
また、値がスコープを抜けるときに`drop`メソッドが自動的に呼び出されるのを無効化することもできない。

値を早期に片付けさせる必要があるときは、代わりに`std::mem::drop`関数を使用できる。
`std::mem::drop`はpreludeに含まれているので、単に`drop()`とすることで呼び出せる。

```rust
let p = CustomSmartPointer { data: String::from("data") };

c.drop(); //エラー
drop(c); //OK 引数にドロップさせたいオブジェクトを渡す
```











