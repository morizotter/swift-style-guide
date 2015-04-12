このスタイルガイドは、[github/swift-style-guide](https://github.com/github/swift-style-guide)をベースにしています。今後独自に発展させていく予定です。英語版も作っていきたい。


----

Swiftのスタイルや記法に関するガイドです。

このガイドは下記の目的を達成するためのパターンを奨励する試みです（リストの順番は大まかです）。

 1. 厳格さを強め、プログラマが陥りやすいエラーを減らすこと
 1. 意図をより明確にすること
 1. 冗長性を減らすこと
 1. コードの美しさに関する議論を減らすこと

提案があれば、是非、[コントリビューション・ガイドライン](CONTRIBUTING.md)をに目を通してプルリクエストを送って下さい。 :zap:

----

#### スペース

 * タブではなくスペースを使う。
 * ファイルの最後には改行を入れる。
 * ロジックの単位で適切に空の行を入れコードを分割する。
 * 行の最後にスペースを残さない。
   * 空の行を示すインデントも同様とする。

#### できるだけ`var` バインディングより `let` バインディングを優先する

可能であるならば、また、どちらを使ってよいか明瞭ではない場合には、 `var foo = …` より、 `let foo = …` を使います。絶対に必要である場合にのみ `var` を利用します（例えば、 `weak` 修飾子を付けた場合など、プログラマが値が変わる可能性を *知っている* ときです）。

_根拠:_ それぞれのキーワードの意図と意味は明瞭ですが、 *letをデフォルトとして使うこと* により、より安全で明瞭なコードを書くことに繋がります。

`let` バインディングは、その値が変わることは想定されていないということを保証し、*プログラマに明確に伝えます* 。続くコードはその値に対して、より明確な推定の下で書き進めることができます。

これはコードをより読みやすくします。値が変わらないと推定できるものに対して `var` を使った場合、必ずそれが変化したかチェックをする必要があります。

したがって、 `var` が使われているときはいつでも、それが変化するものであることを意識し、どうしてそれが使われているかを確認しましょう。

#### オプショナルの強制的アンラップは避けましょう

`FooType?` または、 `FooType!` 型の `foo` があった場合、値を取得するためにできるだけ強制的アンラッピング（ `foo!` ）をしないようにしましょう。

代わりにこのように書きましょう:

```swift
if let foo = foo {
    // Use unwrapped `foo` value in here
} else {
    // If appropriate, handle the case where the optional is nil
}
```

あるいは、場合によってはSwiftのオプショナル・チェーンを使ってこのように書きましょう:

```swift
// `foo` がnilでなかった場合に、関数が呼ばれます。nilだった場合は、関数の呼び出しは無視されます。
foo?.callSomethingIfFooIsNotNil()
```

_根拠:_ オプショナルの`if let`バインディングは安全なコードに繋がります。強制的アンラップを使うことでランタイムにクラッシュしやすいコードになってしまいます。

#### 暗黙的アンラップ・オプショナルはできるだけ使わないようにしましょう

`foo` がnilの可能性を持っている場合は、できるだけ `let foo: FooType!` の代わりに `let foo: FooType?` を使うようにします（一般的に、 `!` の代わりに `?` を使うことができます）。

_根拠:_ 明示的なオプショナルは安全なコードに繋がります。暗黙的アンラップ・オプショナルはランタイムにクラッシュする可能性を含みます。

#### read-onlyのプロパティやsubscriptsでは暗黙的なゲッタにします

read-onlyのコンピューテッド・プロパティ やsubscriptsでは、できるだけ `get` は使わないようにします。

このように書きます:

```swift
var myGreatProperty: Int {
  return 4
}

subscript(index: Int) -> T {
    return objects[index]
}
```

… このようには書きません:

```swift
var myGreatProperty: Int {
  get {
    return 4
  }
}

subscript(index: Int) -> T {
    get {
        return objects[index]
    }
}
```

_根拠:_ 最初に上げた例の意図と意味は明確で、コード量も減らせます。

#### トップレベルの定義には常に明示的にアクセスコントロール修飾子を付けるようにします

トップレベルの関数、型、変数には常に明示的にアクセスコントロール修飾子を付けます:

```swift
public var whoopsGlobalState: Int
internal struct TheFez {}
private func doTheThings(things: [Thing]) {}
```

トップレベルでない場合は、適切に暗黙的なままにしておきます。

```swift
internal struct TheFez {
  var owner: Person = Joshaber()
}
```

_根拠:_ トップレベルの定義を特に`internal`とするのは適切ではない場合がほとんどです。明示的にすることで、その判断をする際に注意深くなります。 定義の中で同様のアクセス・コントロールをつけることは単なる重複になりますし、そのままの状態のほうが一般的に合理的です。

#### 型を指定する際は、常にコロンを付けます

型を指定する際には、常に識別子の直後にコロンを付け、スペースに続けて型名を記述します。

```swift
class SmallBatchSustainableFairtrade: Coffee { ... }

let timeToCoffee: NSTimeInterval = 2

func makeCoffee(type: CoffeeType) -> Coffee { ... }
```

_根拠:_ 型指定子は _識別子_ に対して何かを語っているので、識別子と一緒に配置されるべきです。

また、辞書の型を指定する際には常に、キーの型の後にコロンを付け、スペースに続けてバリューの型を記述します。

```swift
let capitals: [Country: City] = [ Sweden: Stockholm ]
```

#### 必要なときのみ明示的に `self` を使います

`self` のプロパティやメソッドにアクセスする際には、明示的に `self` と記述しないようにします:

```swift
private class History {
  var events: [Event]

  func rewrite() {
    events = []
  }
}
```

例として必要な場合、クロージャの中で使われる場合、また、パラメータ名のコンフリクトが起こる場合のみ、明示的に利用します。

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

_根拠:_ このようにすることで、クロージャの中でキャプチャとして使われる `self` の意味が際立ちます。また、冗長性を減らすことができます。

#### classよりもstructを使うようにします

クラスに寄って提供される機能（同一性を確保することや、デイニシャライザーを使うことなどの）を使う必要がなければ、structを使いましょう。

継承（それ単体として）は、普通クラスを使う適切な理由には _ならない_ ということを覚えておきましょう。なぜならプロトコルによってポリモーフィズムは提供されているからです。また、実装の再利用はコンポジションで可能です。

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

このような定義にリファクタすることができます:

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

_根拠:_ 値型はよりシンプルで、それが表しているものを理解しやすいものとなりますし、 `let` を使った時に想定するような振る舞いをするからです。

#### クラスは基本的に `final` となるようにします

クラスは、 `final` で始めるべきです。そして、継承をするもっともな理由があるという場合に限りサブクラスを許容するために `final` を外します。その場合でも、できるだけクラス _内部で_ 利用する定義には `final` を付けるべきです。

_根拠:_ 一般的にコンポジションは継承よりも優先されるべきです。継承を _利用する_ ことは、その判断をするためにより多くの思慮が必要だったことを意味するものとなることが好ましいからです。

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

#### オペレータの定義はスペースで囲います

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
