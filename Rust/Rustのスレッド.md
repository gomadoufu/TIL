# Rustのスレッド

プログラム内の計算を複数のスレッドに分けると、パフォーマンスが改善する。

スレッドの実装には、2種類の実装がある。
ひとつは「言語がOSのAPIを呼び出してスレッドを生成する」モデルで、これをよく**1:1モデル**と呼ぶ。1つのOSスレッドに対して、1つの言語スレッドが対応するから。
もうひとつは、プログラミング言語が独自に提供するスレッドモデルで、グリーンスレッドともいう。このモデルは、言語側の複数のスレッドを複数のOSスレッドで実行するので、**M:Nモデル**と呼ばれる。

**Rustにおけるスレッドの実装は、1:1モデル**であり、M:Nモデルは非同期ランタイムとして外部crateで提供される(tokioなど)。標準で1:1モデルしか提供されないのは、低レイヤのためにできるだけ小さなランタイムしか持てないから。

## spawnで新規スレッドを生成する

新規スレッドを生成するには、`thread::spawn`関連関数を呼び出し、新規スレッドで走らせたいコードを含むクロージャを渡す。

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("立ち上げたスレッドから送信:{}", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("メインスレッドから送信:{}", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```



## joinハンドルで全スレッドの終了を待つ

上記のコードは、たいてい問題なくスレッドを立ち上げられる。しかし、複数のスレッドがどんな順番で実行されるかや、実際にスレッドが実行されるかは、全く保証されていない。

`thread::spawn`の戻り値を変数に保存することで、この問題を解決できる。`thread::spawn`の戻り値の型は`JoinHandle`で、これの`join`メソッドを呼び出すことで、スレッドを確実にその段階で終了させることができる。

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

ハンドルに対して`join`を呼び出すと、ハンドルが表すスレッドが終了するまで**現在実行中のスレッドをブロックする**。どこで`join`を呼ぶかといったほんの些細なことが、スレッドが同時に走るかどうかに影響する。



## スレッドでmoveクロージャを使用する

`move`クロージャは、`thread::spawn`とともに、あるスレッドのデータを別のスレッドで使用できるようにする目的で使用される。

例えば、以下のようにメインスレッドでベクタを生成し、立ち上げたスレッドで`move`なしにそれを使用すると、コンパイルエラーになる。

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("メインスレッドのベクタ: {:?}", v);
    });
  
  //drop(v); ここで値がドロップされるかもよ？ スレッドからの参照が不正になるよ？

    handle.join().unwrap();
}
```

このコードは動かない。理由は、メインスレッドが終わってもhandleスレッドが生きている可能性があり、新たなスレッドからの`v`の参照が必ず有効かどうか、わからないためである。(新たなスレッドが`v`の参照を取ると明示されていないが、これはRustコンパイラが`v`のキャプチャ方法を推論しているため。`println!`マクロは読み取り専用なことから。)

クロージャの前に`move`キーワードをつけることで、コンパイラに値を借用するのだと推論させるのではなく、強制的にクロージャに値の所有権を奪わせることができる。次のコードは、コンパイルが通る。

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

  //vの所有権をhandleが奪う！
    let handle = thread::spawn(move || {
        println!("メインスレッドのベクタ: {:?}", v);
    });
  
  //drop(v); そもそもドロップできなくなる

    handle.join().unwrap();
}
```

