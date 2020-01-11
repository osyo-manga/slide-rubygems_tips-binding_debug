### あなたの知ってるRubyGemsTips
- - -

#### printf デバッグが捗る gem をつくった話

---

#### 自己紹介
- - -

* なまえ  : おしょー
* Twitter : [@pink_bangbi](https://twitter.com/pink_bangbi)
* github  : [osyo-manga](https://github.com/osyo-manga)
* ブログ  : [Secret Garden(Instrumental)](http://secret-garden.hatenablog.com)
* Rails 歴 2年弱                                       <!-- .element: class="fragment" -->
* 趣味で Ruby にパッチを投げたりしてます                              <!-- .element: class="fragment" -->
  * Ruby 2.7 に [Time#floor](https://bugs.ruby-lang.org/issues/15653) / [Time#ceil](https://bugs.ruby-lang.org/issues/15772) を追加したり
* エディタは Vim                             <!-- .element: class="fragment" -->
  * <del>わしの vimrc は5000行あるぞ</del>


---

## 今日話すこと
## printf デバッグが捗る
## gem をつくった話

---

#### printf デバッグ
- - -

* 変数の内容などを標準出力して値を確認するデバッグ

```ruby
# ActiveRecord::Reflection#add_reflection
# .has_many とかの内部で呼ばれる
def add_reflection(ar, name, reflection)
  ar.clear_reflections_cache
  name = -name.to_s
  ar._reflections = ar._reflections.except(name).merge!(name => reflection)
end
```

>>>

```ruby
def add_reflection(ar, name, reflection)
  # 各値を出力する
  p __method__
  p ar
  p name
  p reflection.class.name
  ar.clear_reflections_cache
  name = -name.to_s
  ar._reflections = ar._reflections.except(name).merge!(name => reflection)
end
```

>>>

```ruby
class Blog < ActiveRecord::Base
  has_many :articles
end
```

```
:add_reflection
Blog(id: integer)
:articles
"ActiveRecord::Reflection::HasManyReflection"
```
<!-- .element: class="fragment" -->

### 何に対する出力なのかわかりづらい!!!!!                  <!-- .element: class="fragment" -->

---

#### 変数名などと一緒に表示する
- - -

```ruby
def add_reflection(ar, name, reflection)
  # 各値を出力する
  p "__method__ : #{__method__}"
  p "ar : #{ar}"
  p "name : #{name}"
  p "reflection.class.name : #{reflection.class.name}"
  ar.clear_reflections_cache
  name = -name.to_s
  ar._reflections = ar._reflections.except(name).merge!(name => reflection)
end
```

>>>


```ruby
class Blog < ActiveRecord::Base
  has_many :articles
end
```

```
"__method__ : add_reflection"
"ar : Blog"
"name : articles"
"reflection.class.name : ActiveRecord::Reflection::HasManyReflection"
```
<!-- .element: class="fragment" -->

### わかりやすいけどめんどくさい！                  <!-- .element: class="fragment" -->

---


### と、いうことで
### このあたりを便利にする
### gem をつくった

---

#### [binding-debug](https://github.com/osyo-manga/gem-binding-debug)
- - -

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
value = 114
# ブロックの中に式もかける
pp { value + value } # value + value : 228
pp { value.chr }     # value.chr : r
# ブロックの中に複数の式もかける
pp {
  value + value
  value + 3
  value * value
}
# value + value : 228
# value + 3 : 117
# value * value : 12996
```

>>>

```ruby
using BindingDebug
def add_reflection(ar, name, reflection)
  # ブロックの中に出力したい変数やメソッドを書いていく
  p {
    __method__
    ar
    name
    reflection.class.name
  }
  ar.clear_reflections_cache
  name = -name.to_s
  ar._reflections = ar._reflections.except(name).merge!(name => reflection)
end
```

>>>

```ruby
class Blog < ActiveRecord::Base
  has_many :articles
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

### 実装方法
### 説明すると長くなるので割愛                  <!-- .element: class="fragment" -->

---


#### まとめ
- - -

* こんなことができる Ruby たのしい！！！                  <!-- .element: class="fragment" -->

---

## ご清聴
## ありがとうございました
