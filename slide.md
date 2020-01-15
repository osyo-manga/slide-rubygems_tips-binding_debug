### あなたの知ってるRubyGemsTips
- - -

## printf デバッグが捗る
## gem をつくった話

---

#### 自己紹介
- - -

* なまえ  : おしょー
* Twitter : [@pink_bangbi](https://twitter.com/pink_bangbi)
* github  : [osyo-manga](https://github.com/osyo-manga)
* ブログ  : [Secret Garden(Instrumental)](http://secret-garden.hatenablog.com)
  * [ピュア Ruby で Ruby 2.7 の Numbered parameter を実装してみよう！ - Secret Garden(Instrumental)](http://secret-garden.hatenablog.com/entry/2019/12/01/154607)   <!-- .element: class="fragment" -->
* Rails 歴 2年弱                                       <!-- .element: class="fragment" -->
* 趣味で Ruby にパッチを投げたりしてます                              <!-- .element: class="fragment" -->
  * Ruby 2.7 に [Time#floor](https://bugs.ruby-lang.org/issues/15653) / [Time#ceil](https://bugs.ruby-lang.org/issues/15772) を追加したり


---

## 今日話すこと
- - -
## printf デバッグが捗る
## gem をつくった話

---

#### printf デバッグ
- - -

* 変数を標準出力して値を確認するデバッグ

```ruby
class Blog < ActiveRecord::Base
  has_many :articles    # ←これの実装を調べる
end
```

```ruby
# ActiveRecord::Reflection#add_reflection
def add_reflection(ar, name, reflection)
  ar.clear_reflections_cache
  name = -name.to_s
  ar._reflections = ar._reflections.except(name).merge!(name => reflection)
end
```
<!-- .element: class="fragment" -->

>>>

```ruby
def add_reflection(ar, name, reflection)
  p __method__
  p ar
  p name
  p reflection.class.name
  # ...省略...
end
```

```
:add_reflection
Blog(id: integer)
:articles
"ActiveRecord::Reflection::HasManyReflection"
```
<!-- .element: class="fragment" -->

---

### 何に対する出力なのかわかりづらい!!!!!

---

#### 変数名などと一緒に表示する
- - -

```ruby
def add_reflection(ar, name, reflection)
  p "__method__ : #{__method__}"
  p "ar : #{ar}"
  p "name : #{name}"
  p "reflection.class.name : #{reflection.class.name}"
  # ...省略...
end
```

```
"__method__ : add_reflection"
"ar : Blog"
"name : articles"
"reflection.class.name : ActiveRecord::Reflection::HasManyReflection"
```
<!-- .element: class="fragment" -->

---

### わかりやすいけどめんどくさい!!!!

---


### と、いうことで
### このあたりを便利にする
### gem をつくった

---

#### [binding-debug](https://github.com/osyo-manga/gem-binding-debug)
- - -

* github : https://github.com/osyo-manga/gem-binding-debug
* rubygems : https://rubygems.org/gems/binding-debug
* install : $ gem install binding-debug

>>>

```ruby
require "binding/debug"

# refinements で実装しているので要 using
using BindingDebug

value = 42
# 普通に使うと値だけが出力される
p value         # 42

# pp にブロックを渡すと変数名も一緒に出力される
p { value }     # value : 42
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="4"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="8"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="11"></span>

>>>

```ruby
value = 42
# ブロックの中に式もかける
p { value + value }    # => value + value : 84
p { value.to_s.size }  # => value.to_s.size : 2
```

```ruby
# ブロックの中に複数の式もかける
pp {
  value + value
  value + 3
  value * value
}
# => value + value : 84
#    value + 3 : 45
#    value * value : 1764
```
<!-- .element: class="fragment" -->

>>>

```ruby
def add_reflection(ar, name, reflection)
  p {
    __method__
    ar
    name
    reflection.class.name
  }
  # ...省略...
end
```

```ruby
__method__ : :add_reflection
ar : Blog(id: integer)
name : :articles
reflection.class.name : "ActiveRecord::Reflection::HasManyReflection"
```
<!-- .element: class="fragment" -->

---

#### 実装の経緯
- - -

* 元々は Binding#p に文字列を渡す実装だった    <!-- .element: class="fragment" -->
  * 渡した文字列を binding.eval で評価する

```ruby
binding.p "value"
binding.p %{
  value + value
  value + 3
  value * value
}
```
<!-- .element: class="fragment" -->

* binding.p は長いので闇の力をつかって短くした     <!-- .element: class="fragment" -->

---

### 実装方法
### 説明すると長くなるので割愛                  <!-- .element: class="fragment" -->

---


#### まとめ
- - -

* こんなことができる Ruby たのしい！！！                  <!-- .element: class="fragment" -->
* みんなもどんどん雑に闇の力をつかって gem をつくっていこう             <!-- .element: class="fragment" -->

---

## ご清聴
## ありがとうございました
