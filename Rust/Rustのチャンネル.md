# Rustのチャンネル

安全な並行性を保証するやり方の一つとして、**メッセージ受け渡し**がある。メッセージ送信並行性を達成するために、Rustには**チャンネル**がある。

プログラミングにおけるチャンネルは、転送機と受信機の２つに分かれる。転送機と受信機のどちらかがドロップされると、 チャンネルは閉じられる。Rustでチャンネルを作成してみる。

```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel(); //チャンネル生成

  //スレッドに所有権を渡す
    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap(); //メッセージを送信
    });
  
  let received = rx.recv().unwrap();
    // 値は{}です
    println!("Got: {}", received);
}
```

`thread::spawn`を使用して新しいスレッドを生成し、それから`move`を使用して、 立ち上げたスレッドが`tx`を所有するようクロージャに`tx`をムーブした。立ち上げたスレッドは、 メッセージをチャンネルを通して送信できるように、チャンネルの送信側を所有する必要がある。

転送側`tx`には、送信したい値を取る`send`メソッドがある。これは`Result<T, E>`型を返す。
受信側`rx`には、受信用のメソッドがふたつある。ひとつは`recv`メソッドで、これはメインスレッドの実行をブロックし、値がチャンネルを流れてくるまで待機する。もうひとつは`try_recv`メソッドで、これはメインスレッドをブロックせず、代わりに即座に`Result<T, E>`型を返す。メッセージを待つ間にスレッドでやることが他にある時に有用で、定期的に`try_recv`を呼び出し、メッセージがあったら処理してなかったら他の仕事をする、というような使い方ができる。上記の例では、メインスレッドで他にすることがないので、`recv`メソッドを選択した。



### 所有権の転送

チャンネルでは、値の所有権が転送されるため、並行コードにおけるバグが防がれる。以下のコードは、`send`した後に値を使用しているが、これはコンパイルエラーになる

```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
        // valは{}
        println!("val is {}", val);
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

一旦、値が他のスレッドに送信されたら、再度値を使用しようとする前にそのスレッドが変更したりドロップできてしまう。そのため、送信後は値を使用不能にすべき。Rustの所有権は、それを保証してくれる。`send`関数は引数の所有権を奪い、受信側が所有権を得るようになっている。



### 転送機をクローンして、複数の値を送信する

チャンネルを生成するときには、`mpsc::channel`関数を使用した。`mpsc`とは、*multiple producer, single consumer*(複数の生成機、単一の受信機)を意味する言葉。つまり、Rustのチャンネルは**複数の送信側**と、その値を消費する**ただ一つの受信側**を持つことができるということ。

`mpsc`の意味通り、`tx` の参照を引数にとる`mpsc::Sender::clone`メソッドを呼び出すことで、複数の生成機から複数のメッセージを送信することができる。

```rust
use std::thread;
use std::sync::mpsc;
fn main() {
  let (tx, rx) = mpsc::channel();
  let tx1 = mpsc::Sender::clone(&tx); //複製
  let tx2 = mpsc::Sender::clone(&tx);

// --snip--
  
  for received in rx {
    println!("受信: {}", received);
  }
}
```









