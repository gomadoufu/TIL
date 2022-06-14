# クラス (C++ に特有の事情のみ)



## const メンバ関数

引数リストのあとに `const` をつけることで `const` メンバ関数になる。  
`const` メンバ関数ではデータメンバを変更することができない。
オブジェクトの状態を変化させないことが分かって安心。

```C++
class Rectangle {
 public:
    int Area() const;

    int height_;
    int width_;
};

int Rectangle::Area() const {
    height_ = 0;  // データメンバを変更するとコンパイルエラーになります
    return height_ * width_;
}
```



## thisポインタ

メンバ関数では `this` で自オブジェクトのポインタを取得することができる
```C++
class Rectangle {
 public:
    int Area();

    int height_;
    int width_;
};

int Rectangle::Area() {
   // this ポインタ経由でデータメンバを使用
    return this->height_ * this->width_;
}
```



## 仮想関数とオーバーライド

派生クラスで挙動を変更できるメンバ関数を仮想関数という。 仮想関数にするには基底クラスのメンバ関数に `virtual` をつける。

再定義されなくても問題はない。

オーバーライドを明確化するときには`override`をつける(@overrideと違いコンパイラのチェックはない)

```C++
class Rectangle {
 public:
    virtual void Describe() const {
        std::cout << "height = " << height_ << std::endl;
        std::cout << "width = " << width_ << std::endl;
    }

    int height_;
    int width_;
};

class Square : public Rectangle {
 public:
    void SetSize(int size) {
        height_ = size;
        width_ = size;
    }
    //Describeをオーバーライド
    void Describe() const override {
        std::cout << "size = " << height_ << std::endl;
    }
};

```



## 純粋仮想関数とインターフェース

定義を持たない(プロトタイプ宣言されているだけの)仮想関数を純粋仮想関数という。

継承先でオーバーライドして、定義を実装する必要がある。

純粋仮想関数にするには仮想関数に`= 0`をつける。

```C++
class Polygon {
 public:
    virtual int Area() const = 0;
};
```

純粋仮想関数を持つクラスは抽象クラスという。また、C++ にはインターフェースをつくるための専用の記法はないため、 メンバ関数がすべて純粋仮想関数であるクラスをインターフェースとして扱う。



## 演算子オーバーロード

クラスに対する演算子を定義することができる。

```C++
//加算演算子
Integer operator+(const Integer& rhs) const {
  // 返すためのオブジェクトtmpを作る、演算子左辺は自オブジェクトを使用する
        Integer tmp(Value() + rhs.Value());  
        return tmp;
    }

//前置インクリメント
 Integer& operator++() {
        ++value_;
        return *this;
    }

//後置インクリメント
  Integer operator++(int) {  // 引数の int は使用しない
        Integer tmp(Value());
        ++value_;
        return tmp;
    }

//コピー代入演算子
Integer& operator=(const Integer& c);
```
