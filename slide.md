#### 銀座Rails#25@リンクアンドモチベーション
- - -

## Proc#using 感想戦


---

#### 自己紹介
- - -

* 名前：osyo
* Twitter : [@pink_bangbi](https://twitter.com/pink_bangbi)
* github  : [osyo-manga](https://github.com/osyo-manga)
* ブログ  : [Secret Garden(Instrumental)](http://secret-garden.hatenablog.com)
* 趣味で Ruby にパッチを投げたり bugs.ruby で気になったチケットをブログにまとめたりしてる
* Ruby で一番好きな機能は Refinements

---


## 今日のお題
### [Magic is organizing chaos](https://rubykaigi.org/2020-takeout/presentations/shugomaeda.html#sep05) の
### Proc#using について語る

---


### そもそも Refinements 使ってる？

---

#### Refinements とは
- - -

* 特定のコンテキストでのみクラス拡張を適用する構文           <!-- .element: class="fragment" -->
* 通常のモンキーパッチとは違い名前衝突の危険性を最小限に抑え、安全にクラス拡張する事ができる           <!-- .element: class="fragment" -->

```ruby
module Twice
  # String に新しいメソッドを定義する
  # これだけでは定義されない
  refine String do
    def twice
      self + self
    end
  end
end

# using を呼び出したコンテキストでのみメソッドが適用される
using Twice

p "homu".twice
# => "homuhomu"
```
<!-- .element: class="fragment" -->


---

## Proc#using って？

---

#### Proc#using とは
- - -

* Proc#using は任意のブロックに対して using を行う機能になる       <!-- .element: class="fragment" -->
* これを利用することで任意のブロック内でのみメソッド拡張を適用する事ができる       <!-- .element: class="fragment" -->

```ruby
# これを using したコンテキストのみ Proc#using が利用できる
using Proc::Refinements

block = proc {
  p "homu".twice
}

# ブロックに対して using を行うと
# そのブロック内で Twice が適用される
block.using Twice
block.call
# => "homuhomu"
```
<!-- .element: class="fragment" -->

>>>

* Proc#using を利用するとメソッドに渡したブロック内で暗黙的にクラス拡張を適用する事ができる

```ruby
# ブロックに対して任意の Refinements を適用できる
def benri(&block)
  block.using Twice
  block.call
end

using Proc::Refinements

# ブロック内で独自のメソッド拡張が呼べる
benri {
  p "homu".twice
  p "mami".twice.twice
}

# ブロックの外では使えない
# error: : undefined method `twice' for "mado":String (NoMethodError)
p "mado".twice
```

---

### Proc#using の気になるところ！！！

---

#### Proc#using 感想
- - -

* Refinements は using とワンセットで使う機能になる            <!-- .element: class="fragment" -->
    * ユーザは明示的に『どの拡張を行うのか』を指定して使う
* 一方で Proc#using の場合は using するモジュールが定義側で隠蔽されてしまう            <!-- .element: class="fragment" -->
    * ブロックを書くユーザ側ではどのモジュールが拡張されるのか不透明になる

```ruby
# ブロック内で何が using されているのかがわからない…
benri {
  p "homu".twice
}
```
<!-- .element: class="fragment" -->

* それはユーザの意図しないメソッドが呼び出される可能性があるので危険なのではないだろうか           <!-- .element: class="fragment" -->
* 個人的にはユーザ側でどのモジュールが適用されるのかを明示化したい                   <!-- .element: class="fragment" -->

---

### じゃあ Proc#using いらない？

---

## どちゃくそほしい！！！！

---

#### そもそも
- - -

* ブロック内のコンテキストが変わってしまうのは instance_eval を使ったときでも同じ    <!-- .element: class="fragment" -->
    * ブロック内で self が変わってしまうことはよくある
* なので Proc#using で呼び出されるメソッドが意図しない云々は今更感がある            <!-- .element: class="fragment" -->
* どうしても気になるなら Proc#using を使って便利メソッドを定義すればいい            <!-- .element: class="fragment" -->

```ruby
# Proc#using を使ったヘルパメソッドを定義してやる
def using_local(mod, &block)
  block.using mod
  block.call
end

using Proc::Refinements
# モジュールを渡して using を明示化する
using_local(Twice) {
  p "homu".twice
}
```
<!-- .element: class="fragment" -->

>>>

* 仮に using_local みたいなメソッドがあったとしてもメソッド内で隠蔽されてしまえば結局 Proc#using と同じ問題に突き当たる

```ruby
def benri(&block)
  # ブロック内で using_local を経由してブロックが呼びされたら
  # 結局使う側からはどのモジュールが using されるのかは不透明になる
  using_local(Twice, &block)
end

# 結局どのモジュールが適用されているのかがわからない
benri {
  p "homu".twice
  p "mami".twice.twice
}
```

* Proc#using だけを見ると使い方に関して気になる点はあるが利便性を考えるとめっちゃほしい！！！ <!-- .element: class="fragment" -->

---

## 今後こうなってほしい！！

---

## 1. Proc#using が入る

---

## 2. みんなカジュアルに Refinements を使い始める

---

## 3. Refinements のエッジケースのバグが炙り出される

---

## 4. がんばってバグが修正される

---

## 5. どんどん Refinements がよくなる

---

## 6. 幸せな世界が待っている

---

## まとめ

---

#### まとめ
- - -

* Proc#using 自体に関しては書き方や使い方に関してちょっと気になる部分がある                  <!-- .element: class="fragment" -->
* が、それ以上に Proc#using 便利なのでほしい！！！                 <!-- .element: class="fragment" -->
* Proc#using に関しては結構省いた部分があるので詳しくは元のセッションを見てください             <!-- .element: class="fragment" -->
* Refinements どちゃくそ便利なのでみんなどんどん Refinements を使ってくれー！！                 <!-- .element: class="fragment" -->
* あとバグがあったらどんどん報告してくれー！！！                 <!-- .element: class="fragment" -->

---

## 宣伝

---

## 10/03 にある [Kaigi on Rails](https://kaigionrails.org/) で ActiveRecord の話します

---

## ご清聴
## ありがとうございました

