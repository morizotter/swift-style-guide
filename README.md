# swift style guide

## 概要

Swiftのスタイルや記法に関するガイドです。個人的に気になった記法などをまとめていきます。

## 参考

- [github/swift-style-guide](https://github.com/github/swift-style-guide)
- [raywenderlich/swift-style-guide](https://github.com/raywenderlich/swift-style-guide)

## 目次

- [全般](#全般)
- [Type](#Type)

## 全般

### スペース

 * タブではなくスペースを使う。
 * ファイルの最後には改行を入れる。
 * ロジックの単位で適切に空の行を入れコードを分割する。
 * 行の最後にスペースを残さない。
   * 空の行を示すインデントも同様とする。

### varとlet

#### 基本的に `let` を使う

安全で明瞭なコードを維持するために基本的に `let` を使う。必要である場合に限り、 `var` を使う。必要である場合の例は下記の通り。

- プログラマが値が変わる可能性を知っている。

`var` が使われるときは、それが変化する値であることを前提にコードを書く。例えば、 `if let` や `variable?.doSomething()`  のようにnilを想定する。

#### 強制的アンラップは避ける

ラップされたオブジェクトはnilの可能性を持っている。強制的アンラップをすることで、nilの際に実行時にクラッシュしてしまう。これを避けるために、強制的アンラップは使わない。

`if let` 文で判定する。

```swift
if let foo = foo {
    // アンラップされた `foo` を利用する
} else {
    // 必要であれば `nil` の場合のケースを書く。
}
```

オプショナル・チェーンで判定する。

```swift
// `foo` がnilでなかった場合に、関数が呼ばれます。nilだった場合は、関数の呼び出しは無視されます。
foo?.callSomethingIfFooIsNotNil()
```

### 型指定

型が明確でなく指定をする必要がある場合は、常にコロンを付ける。型を指定する際には、常に識別子の直後にコロンを付け、スペースに続けて型名を記述します。

```swift
class SmallBatchSustainableFairtrade: Coffee { ... }

let timeToCoffee: NSTimeInterval = 2

func makeCoffee(type: CoffeeType) -> Coffee { ... }
```

### self

必要なときのみ明示的に `self` を使う。 `self` のプロパティやメソッドにアクセスする際には、明示的に `self` と記述しないようにする:

```swift
private class History {
  var events: [Event]

  func rewrite() {
    events = []
  }
}
```

クロージャの中で使われる場合、また、パラメータ名のコンフリクトが起こる場合のみ、明示的に利用する。クロージャの中でキャプ社として使われる `self` の意味が目立つため。

```swift
extension History {
  init(events: [Event]) {
    self.events = events
  }

  var whenVictorious: () -> () {
    return {
      self.rewrite()
    }
  }
}
```

## Type

### class、struct、enum

#### アクセスコントロール修飾子

トップレベルの定義には常に明示的にアクセスコントロール修飾子を付ける。トップレベルの定義を特に`internal`とするのは適切ではない場合がほとんどである。

```swift
public var whoopsGlobalState: Int
internal struct TheFez {}
private func doTheThings(things: [Thing]) {}
```

トップレベルでない場合は、適切に暗黙的なままにしておく。

```swift
internal struct TheFez {
  var owner: Person = Joshaber()
}
```

#### オペレータの定義はスペースで囲う

オペレータを定義するときにはスペースで区切ります。このように書くのではなく:

```swift
func <|(lhs: Int, rhs: Int) -> Int
func <|<<A>(lhs: A, rhs: A) -> A
```

このように書きましょう:

```swift
func <| (lhs: Int, rhs: Int) -> Int
func <|< <A>(lhs: A, rhs: A) -> A
```

_根拠_: オペレータは句読点（パンクチュエーション）で構成されます。型や値のパラメータが直後に続いていた場合、とても読みにくくなってしまいます。スペースを追加することで、それらを明確に分離することができるからです。

#### classとstruct

`struct` で十分な場合は `class` を使わない。値型としてstructを利用することにより、コードはシンプルになり、可読性も増す。

クラスによって提供される機能（同一性を確保することや、デイニシャライザーを使うことなどの）を使う必要がなければ、structを使う。

継承（それ単体として）は、普通クラスを使う適切な理由にはならない。なぜならプロトコルによってポリモーフィズムは提供されているからです。また、実装の再利用はコンポジションで可能。

例えば、このクラス階層は:

```swift
class Vehicle {
    let numberOfWheels: Int

    init(numberOfWheels: Int) {
        self.numberOfWheels = numberOfWheels
    }

    func maximumTotalTirePressure(pressurePerWheel: Float) -> Float {
        return pressurePerWheel * numberOfWheels
    }
}

class Bicycle: Vehicle {
    init() {
        super.init(numberOfWheels: 2)
    }
}

class Car: Vehicle {
    init() {
        super.init(numberOfWheels: 4)
    }
}
```

このような定義にリファクタすることができる:

```swift
protocol Vehicle {
    var numberOfWheels: Int { get }
}

func maximumTotalTirePressure(vehicle: Vehicle, pressurePerWheel: Float) -> Float {
    return pressurePerWheel * vehicle.numberOfWheels
}

struct Bicycle: Vehicle {
    let numberOfWheels = 2
}

struct Car: Vehicle {
    let numberOfWheels = 4
}
```

### class

#### できるだけ型パラメータを削るようにします

型パラメータのついたメソッドは、レシーバにとって自明であれば、受け取った型の型パラメータを削除できます。例えば:

```swift
struct Composite<T> {
  …
  func compose(other: Composite<T>) -> Composite<T> {
    return Composite<T>(self, other)
  }
}
```

これは、このように書き直せます:

```swift
struct Composite<T> {
  …
  func compose(other: Composite) -> Composite {
    return Composite(self, other)
  }
}
```

_根拠:_ 冗長な型パラメータを削除することで、意図を明確にできますし、別の型パラメータを返すときにその対比を明確にすることができるからです。

#### クラスは基本的に `final` となるようにする

クラスは、 `final` で始める。そして、継承をするもっともな理由があるという場合に限りサブクラスを許容するために `final` を外す。その場合でも、できるだけクラス内部で利用する定義には `final` を付けるべきです。

_根拠:_ 一般的にコンポジションは継承よりも優先されるべきです。継承を _利用する_ ことは、その判断をするためにより多くの思慮が必要だったことを意味するものとなることが好ましいからです。
